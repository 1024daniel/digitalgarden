---
{"dg-publish":true,"permalink":"/submodules/pve/wireshark和tcpdump使用实战/","noteIcon":"3"}
---

![Pasted image 20231119150253.png|100%](/img/user/pics/Pasted%20image%2020231119150253.png)
Linux抓包是通过注册一种虚拟的底层网络协议来完成对网络报文(准确的说是网络设备)消息的处理权。当网卡接收到一个网络报文之后，它会遍历系统中所有已经注册的网络协议，例如以太网协议、x25协议处理模块来尝试进行报文的解析处理，这一点和一些文件系统的挂载相似，就是让系统中所有的已经注册的文件系统来进行尝试挂载，如果哪一个认为自己可以处理，那么就完成挂载。

当抓包模块把自己伪装成一个网络协议的时候，系统在收到报文的时候就会给这个伪协议一次机会，让它来对网卡收到的报文进行一次处理，此时该模块就会趁机对报文进行窥探，也就是把这个报文完完整整的复制一份，假装是自己接收到的报文，汇报给抓包模块


## wireshark使用
#tcpdump #wireshark #dns

```bash
dns and (dns.qry.name == "www.x.com")
```




```bash
ip.addr == 192.168.1.1
ipv6.addr == fe80::2e0:67ff:fe27:fef1
```

![Pasted image 20231118043924.png](/img/user/pics/Pasted%20image%2020231118043924.png)

## tcpdump工具使用

抓取限定条件的报文

```bash
tcpdump src host 192.168.66.179 and dst port 443

tcpdump src host 192.168.66.179 and dst host 192.168.66.101 and ! dst port 22
```


### 查看openclash bypass过程
gateway，也就是openclash机器监听域名解析报文

```bash
tcpdump -nt port domain
```

客户端使用dns查询命令
`host -t A youtube.com`



```bash
tcpdump -v host 192.168.66.101 and  port 7895 and ! dst port 22 and ! src port 22
```
