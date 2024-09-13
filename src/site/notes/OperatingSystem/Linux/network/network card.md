---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/network/network card/","noteIcon":"3"}
---

查看网卡的pci地址(新版的linux的网卡一般可以通过网卡名识别)
`ethtool -i ens7`
![Pasted image 20230717213002.png](/img/user/OperatingSystem/Linux/network/pics/Pasted%20image%2020230717213002.png)
查看对应的内核模块
`lspci -s 0000:04:00.0 -vv`
![Pasted image 20230717213217.png](/img/user/OperatingSystem/Linux/network/pics/Pasted%20image%2020230717213217.png)
查看内核模块对应的ko文件
`modinfo r8169`
![Pasted image 20230717213341.png](/img/user/OperatingSystem/Linux/network/pics/Pasted%20image%2020230717213341.png)
查看文件对应的deb包或者rpm包
`dpkg -S filepath` or `rpm -qf filepath`
![Pasted image 20230717213446.png](/img/user/OperatingSystem/Linux/network/pics/Pasted%20image%2020230717213446.png)

#network/bond 
查看当前所有的bond
`cat /proc/net/bonding/*`
删除bond
`ip link delete bond0 type bond`
将bond的slave网卡删除
`ip link set eth0 nomaster`
查看bond的所有slave interfaces
`cat /proc/net/bonding/bond0|grep -i slave`

查看网络包
```bash
tcpdump -i <interface> host <host_ip>
arping <hostip>
```
查看当前主机的arp cache
如下可以看见openwrt的ip为192.168.66.100，对应的mac地址是56:35:9d:10:0c:c8
![Pasted image 20230825001905.png](/img/user/pics/Pasted%20image%2020230825001905.png)