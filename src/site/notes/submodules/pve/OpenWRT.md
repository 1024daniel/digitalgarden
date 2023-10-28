---
{"dg-publish":true,"permalink":"/submodules/pve/OpenWRT/","noteIcon":"3"}
---

#network/openwrt #network/clash #network/proxy
# Deploy OpenWRT
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
#dnsmasq
references:
https://openwrt.org/docs/guide-user/base-system/dhcp.dnsmasq
对于我们局域网内的服务，为了能在网页直接输入域名而非ip进入服务，我们可以利用openwrt和dnsmasq来进行静态域名和ip绑定
首先在openwrt的hosts文件指定局域网内部主机名

```bash
# 这里只用openwrt的服务举例
192.168.66.100 openwrt
![Pasted image 20231024232907.png](app://54cfc47ada3a33b97e8c13b83fa8976748ae/Users/daniel/code/blog/Pasted%20image%2020231024232907.png?1698161347355)```
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

然后我们直接网页访问openwrt.lan就可以志杰访问我们192.168.66.100上的openwrt网页
![Pasted image 20231024223051.png|100%](/img/user/pics/Pasted%20image%2020231024223051.png)

通过域名登录openwrt然后登录openclash服务，如何想要打开openclash的yacd面板的话需要输入secret(直接使用ip的话可以不用secret)
如下图我们可以看到登录dashboard的地址由external-controller指定
![Pasted image 20231024230642.png|100%](/img/user/pics/Pasted%20image%2020231024230642.png)]
如果使用域名登录的话，需要add对应的ip:端口，然后填写secret登录
secret可在openclash查看
![Pasted image 20231024231127.png|100%](/img/user/pics/Pasted%20image%2020231024231127.png)]
这里的密钥可以在Config Magager里面的yaml文件的secret字段进行配置
![Pasted image 20231024231415.png|100%](/img/user/pics/Pasted%20image%2020231024231415.png)]

# Deploy Clash in OpenWRT
## 1.config dns and region

![Pasted image 20230428231323.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428231323.png)]

![Pasted image 20230428232919.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428232919.png)

![Pasted image 20230428232548.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428232548.png)

![Pasted image 20230428232951.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428232951.png)

apply the change
![Pasted image 20230428231603.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428231603.png)

check change in openwrt shell

![Pasted image 20230428232852.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230428232852.png)

now we can ping baidu.com in the openwrt shell

![Pasted image 20230429003130.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230429003130.png)

## 2.download clash

![open clash for openwrt](https://github.com/vernesong/OpenClash)

after install openclash, reboot
add clash subcription
![Pasted image 20230429013630.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230429013630.png)


## 3.config end device
config the client device network: make sure gateway and dns is the ip of openWRT
![c5837995c81f57ff3a0441748641eb2.jpg|90%](/img/user/submodules/pve/pics/c5837995c81f57ff3a0441748641eb2.jpg)


## 3. some additional settings
### 3.1 visit bing.com instead of cn.bing.com
As I wanna use new bing's chat AI in clash rule mode(it work in clash global mode), I need to overwrite clash default rule mode to make bing.com does not go to cn.bing.com.
go to openclash and download clash rule file
![Pasted image 20230711215316.png|90%](/img/user/submodules/pve/pics/Pasted%20image%2020230711215316.png)

we can see all the aviable proxy node
![Pasted image 20230711215607.png|90%](/img/user/submodules/pve/pics/Pasted%20image%2020230711215607.png)

use one node which area bing is not blocked
![Pasted image 20230711215746.png|90%](/img/user/submodules/pve/pics/Pasted%20image%2020230711215746.png)
click apply setting.
now go into clash yacd web and check if the new rule is working
![Pasted image 20230711215915.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230711215915.png)


now I can use bing's chat AI
![Pasted image 20230711220000.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230711220000.png)
