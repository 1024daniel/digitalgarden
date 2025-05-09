---
{"dg-publish":true,"permalink":"/submodules/pve/openwrt/OpenWRT/","noteIcon":"3"}
---

#network/openwrt #network/clash #network/proxy
![jess-bailey-l3N9Q27zULw-unsplash.jpg|100%](/img/user/banner/jess-bailey-l3N9Q27zULw-unsplash.jpg)
# 一、Deploy OpenWRT in pve vm
## 1.upload openwrt img file to pve
you can upload from pve web or download can put to /var/lib/vz/template/iso/
## 2.create openwrt vm
![Pasted image 20230428130225.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428130225.png)

detach hard disk
![Pasted image 20230428130704.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428130704.png)

remove unused ide2 and disk
![Pasted image 20230428130931.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428130931.png)

execute command
```sh
qm importdisk 100 /var/lib/vz/template/iso/openwrt-22.03.4-x86-64-generic-ext4-combined-efi.img local-lvm
```
![Pasted image 20230428131243.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428131243.png)

set disk
![Pasted image 20230428131440.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428131440.png)

adjust boot order and make sata0 the first one to be booted
![Pasted image 20230428131644.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428131644.png)

## 3.add a linux bridge for openWRT
before we add bridge, network configuration is like this below:
![Pasted image 20230427235816.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230427235816.png)

after we add a network bridge on bond0
![Pasted image 20230428224131.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428224131.png)
start the openwrt vm and modify ip in /etc/config/network
![Pasted image 20230428224209.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428224209.png)

## 4.make openwrt use the bridge on bond0, so our laptop, ipad can be in the same network with openwrt.
proxmox web cannot detect what a bridge on a bond is
![Pasted image 20230428224416.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428224416.png)

Add we cannot edit the network device of openwrt in proxmox web to switch the base device from vmbr0 to br0.

![Pasted image 20230428224622.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428224622.png)


So we can only edit the opemwrt vm conf in pve shell
![Pasted image 20230428224754.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428224754.png)

##  5.login openwrt
![Pasted image 20230428224925.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428224925.png)



## 6.配置局域网域名解析
### 6.1 openwrt.lan访问
#dnsmasq
references:
https://openwrt.org/docs/guide-user/base-system/dhcp.dnsmasq
对于我们局域网内的服务，为了能在网页直接输入域名而非ip进入服务，我们可以利用openwrt和dnsmasq来进行静态域名和ip绑定
首先在openwrt的hosts文件指定局域网内部主机名

```bash
# 这里只用openwrt的服务举例
192.168.66.100 openwrt
```

然后在/etc/dnsmasq.conf中指定局域网内部域名后缀
默认情况下，Dnsmasq配置为将主机放入.lan域。

```bash
local=/lan/
domain=lan
```

对应dnsmasq其他指定域名的可以参考
![Pasted image 20231024232907.png|100%](/img/user/pics/Pasted%20image%2020231024232907.png)
配置完之后可以直接重载dnsmasq

```bash
/etc/init.d/dnsmasq reload
```

然后我们直接网页访问openwrt.lan就可以直接访问我们192.168.66.100上的openwrt网页
![Pasted image 20231024223051.png|100%](/img/user/pics/Pasted%20image%2020231024223051.png)

通过域名登录openwrt然后登录openclash服务，如何想要打开openclash的yacd面板的话需要输入secret(直接使用ip的话可以不用secret)
如下图我们可以看到登录dashboard的地址由external-controller指定
![Pasted image 20231024230642.png|100%](/img/user/pics/Pasted%20image%2020231024230642.png)
如果使用域名登录的话，需要add对应的ip:端口，然后填写secret登录
secret可在openclash查看
![Pasted image 20231024231127.png|100%](/img/user/pics/Pasted%20image%2020231024231127.png)
这里的密钥可以在Config Magager里面的yaml文件的secret字段进行配置
![Pasted image 20231024231415.png|100%](/img/user/pics/Pasted%20image%2020231024231415.png)

>我在一个lxc内部署了很多的容器，有calibre，个人多媒体中心，每个容器都有对应一个网页服务，且端口不是一样的，但是所在的ip都是这个lxc分配的ip，为了在浏览器登录的时候不需要输入domain:port的方式登录具体的服务，尝试使用不同的domain映射到同一个ip上的不同的端口
关于实现这个功能，牵涉到reverse proxy:


```cardlink
url: https://forum.openwrt.org/t/using-domain-name-instead-of-ip-port/158811/18
title: "Using domain name instead of ip:port"
description: "lighttpd documentation      As others have noted, you need to configure DNS for domain names.  If you configure DNS with  myrouter.com 192.168.10.1  mycloud.com 192.168.10.1  then you can configure lighttpd to respond to them separately  $SERVER[\"socket\"] == \":80\" {     $HTTP[\"host\"] == \"mycloud.com\" {         ...     }     #else {     #    ...     #} } $SERVER[\"socket\"] == \":81\" {     $HTTP[\"host\"] == \"myrouter.com\" {         ...     }     #else {     #    ...     #} }  lighttpd will still be l..."
host: forum.openwrt.org
favicon: https://forum.openwrt.org/uploads/default/optimized/3X/0/b/0be226be0af76ed16c229fa402d72b8a7f7266d5_2_32x32.png
image: https://forum.openwrt.org/uploads/default/original/3X/2/9/2965b316403db302c535cae40139e8c49bbad6e3.png
```
![Pasted image 20231111220949.png](/img/user/submodules/pve/pics/Pasted%20image%2020231111220949.png)
### 6.2 lxc上多个service通过反向代理配置域名访问
openwrt的磁盘是从之前的img文件导入制备的，空间只有124M，我们这里先将openwrt对应的lvm进行扩容，以备后面使用
![Pasted image 20231112222405.png](/img/user/submodules/pve/pics/Pasted%20image%2020231112222405.png)
首先openwrt需要下载haproxy, openwrt下载软件前需要先更新available list，因为这些可下载包都存在memory中，reboot之后想要在网页看到需要先update

```bash
opkg update
opkg list|grep haproxy
opkg install haproxy
/etc/init.d/haproxy status

```

# Deploy Clash in OpenWRT
## 1.config dns and region

![Pasted image 20230428231323.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428231323.png)

![Pasted image 20230428232919.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428232919.png)

![Pasted image 20231118031258.png](/img/user/pics/Pasted%20image%2020231118031258.png)


![Pasted image 20230428232951.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428232951.png)

apply the change
![Pasted image 20230428231603.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428231603.png)

check change in openwrt shell

![Pasted image 20230428232852.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428232852.png)

now we can ping baidu.com in the openwrt shell

![Pasted image 20230429003130.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230429003130.png)

## 2.download clash
openclash依赖dnsmasq-full，默认的openwrt镜像内已经包含dnsmasq，我们需要先remove dnsmasq防止冲突
[open clash for openwrt](https://github.com/vernesong/OpenClash)

after install openclash, reboot
add clash subcription
![Pasted image 20230429013630.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230429013630.png)


## 3.config end device
config the client device network: make sure gateway and dns is the ip of openWRT
![c5837995c81f57ff3a0441748641eb2.jpg|90%](/img/user/submodules/pve/pics/c5837995c81f57ff3a0441748641eb2.jpg)



# 二、 Deploy OpenWRT in NRadio 5G CPE

openclash core
