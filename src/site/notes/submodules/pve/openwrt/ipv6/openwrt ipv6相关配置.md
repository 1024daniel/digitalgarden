---
{"dg-publish":true,"permalink":"/submodules/pve/openwrt/ipv6/openwrt ipv6相关配置/","noteIcon":"3"}
---

#ipv6 #adb

https://www.right.com.cn/forum/thread-8320635-1-1.html
禁止科学上网

### 1. 设置op ipv6网关
```sh
# 1 登录 ssh
        sendat 2 'AT+QADBKEY?' #(获取密钥
        #浏览器打开 https://onecompiler.com/python/3znepjcsq 把第一部获取的密钥粘贴进去点击 RUN 将得到 key
        sendat 2 'AT+QADBKEY="获取的key"' #(写入key
        sendat 2 'AT+QCFG="usbcfg",0x2C7C,0x0801,1,1,1,1,1,1,0' #(开启 ADB
        sendat 2 'AT+CFUN=1,1' #(重启模块
# 2 安装ADB

        opkg update && opkg install adb
        adb devices #(查看 ADB 是否开启成功)
        #List of devices attached 6af80141 device（出现类似内容既开启成功

#3 添加开机脚本
        vim /etc/hotplug.d/iface/90-ipv6gwadd
        # 写入如下内容
        #!/bin/sh
        [ "$ACTION" = ifup ] || exit 0
        [ "$INTERFACE" = wan ] || exit 0
        sleep 15s
        ipv6_gateway="$(ip -6 route show default | awk '/default/ {print $3}')"
        adb shell ip -6 route add $ipv6_gateway dev bridge0 metric 100 pref medium
        ip -6 route add $ipv6_gateway dev br-lan

#4 重启 660
ip -6 route
```
![Pasted image 20240407161517.png](/img/user/submodules/pve/openwrt/attachments/Pasted%20image%2020240407161517.png)

### 2. wan
DHCP client
![Pasted image 20240407160912.png](/img/user/submodules/pve/openwrt/attachments/Pasted%20image%2020240407160912.png)

![Pasted image 20240407161000.png](/img/user/submodules/pve/openwrt/attachments/Pasted%20image%2020240407161000.png)
### 3. wan6
DHCPv6 client
![Pasted image 20240407160733.png](/img/user/submodules/pve/openwrt/attachments/Pasted%20image%2020240407160733.png)

![Pasted image 20240407161018.png](/img/user/submodules/pve/openwrt/attachments/Pasted%20image%2020240407161018.png)
### 4. lan

![Pasted image 20240407161124.png](/img/user/submodules/pve/openwrt/attachments/Pasted%20image%2020240407161124.png)


### 5. 设置openclash ipv6 dns解析


![Pasted image 20240424123408.png](/img/user/submodules/pve/openwrt/attachments/Pasted%20image%2020240424123408.png)




wired connection


https://forum.proxmox.com/threads/how-to-setup-proxmox-for-ipv6-in-my-homenet.135475/post-599624
https://www.saudiqbal.com/blog/ipv6-home-server-with-dynamic-prefix-for-vpn-web-server-rdp-and-firewall-setup-guide.php
https://wiki.debian.org/IPv6PrefixDelegation
https://openwrt.org/docs/guide-user/network/ipv6/configuration

https://sysctl-explorer.net/net/ipv6/accept_ra/

![Pasted image 20240412192126.png](/img/user/submodules/pve/openwrt/attachments/Pasted%20image%2020240412192126.png)

### 6.网线连接客户端接受ipv6分配
设置网线直连的proxmox网卡获取到公网ipv6
```yaml hl:52,53
auto lo
iface lo inet loopback

allow-bond0 enp4s0
iface enp4s0 inet manual
iface enp4s0 inet6 manual

#auto bond0
iface bond0 inet manual
        bond-slaves enp4s0
        bond-miimon 1000
        bond-mode active-backup
        bond-primary enp4s0
#       dns-nameserver 202.101.172.35
#       dns-nameserver 8.8.8.8
#       dns-nameserver 114.114.114.114

iface bond0 inet6 manual
        bond-slaves enp4s0
        bond-miimon 1000
        bond-mode active-backup
        bond-primary enp4s0

allow-bond0 wlp4s0
iface wlp4s0 inet manual
        bond_miimon 1000
        bond_mode active-backup
        wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
        wpa-bridge bond0
        bond-master bond0
        bond-give-a-chance 10

iface wlp4s0 inet6 manual
        bond_miimon 1000
        bond_mode active-backup
        wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
        wpa-bridge bond0
        bond-master bond0
        bond-give-a-chance 10

auto br
#iface br inet6 dhcp
iface br inet manual
        address 192.168.66.2/24
        gateway 192.168.66.1
        bridge-ports bond0
        bridge-stp off
        bridge-fd 0
        dns-nameservers 202.101.172.35 8.8.8.8 114.114.114.114

iface br inet6 dhcp
        accept_ra 2
        request_prefix 1
#        bridge-ports bond0
#        bridge-stp off
#        bridge-fd 0 


#auto br:0
#iface br:0 inet6 dhcp
#        bridge-ports bond0
#        bridge-stp off
#        bridge-fd 0


auto vmbr0
iface vmbr0 inet static
        address 10.3.24.111/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
        post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up   iptables -t nat -A POSTROUTING -s '10.3.24.0/24' -o br -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.3.24.0/24' -o br -j MASQUERADE
```







