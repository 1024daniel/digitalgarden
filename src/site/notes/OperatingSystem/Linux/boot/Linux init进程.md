---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/boot/Linux init进程/","noteIcon":"3"}
---

https://blog.csdn.net/cougar_mountain/article/details/9798191

https://blog.csdn.net/cougar_mountain/article/details/9798191

https://www.cnblogs.com/ShineLeBlog/p/17582366.html

https://fedoraproject.org/wiki/SysVinit_to_Systemd_Cheatsheet

#init #runlevel
linux在/etc/rc<\Level>.d文件夹中level数值对应以下几个模式，文件夹有symbolic link到/etc/intit.d文件中指定的服务文件

![Pasted image 20240317233542.png](/img/user/OperatingSystem/Linux/boot/attachments/Pasted%20image%2020240317233542.png)

![Pasted image 20240317235235.png](/img/user/OperatingSystem/Linux/boot/attachments/Pasted%20image%2020240317235235.png)

```sh
[root@node5 ~]# init 0   #关机
[root@node5 ~]# init 3   #进入3级别字符界面
[root@node5 ~]# init 5   #进入5级别图形界面
[root@node5 ~]# init 6   #重启
#设置默认第三启动级别
[root@node5 ~]# systemctl set-default multi-user.target

#设置默认第五启动级别
[root@node5 ~]# systemctl set-default graphical.target
[root@node5 ~]# runlevel
3 5   #表示从3级别切换到了5级别

#查看当前默认的启动级别
[root@node5 ~]# systemctl get-default
graphical.target

[root@node5 ~]# systemctl get-default
multi-user.target
# 两种写法
systemctl graphical.target
systemctl isolate graphical.target    #**切换为第五启动级别-图形界面**

```
