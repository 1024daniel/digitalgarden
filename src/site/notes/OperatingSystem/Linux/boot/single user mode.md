---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/boot/single user mode/","noteIcon":"3"}
---

<font color="#00b050">单用户模式可以不使用密码登录root用户，可以解决忘记root密码或者错误修改root login shell到一个不存在的路径等等导致无法登录root用户等问题</font>


https://www.alibabacloud.com/help/en/ecs/use-cases/boot-a-linux-ecs-instance-into-single-user-mode

## 步骤如下
### 1. 重启机器，选择对应的内核，输入e
### 2.编辑内核参数

在linux这一行加上 `rw init=/bin/sh crashkernel=auto`, 编辑完成之后ctrl x或者F10保存退出回到正常启动流程

![Pasted image 20240408092008.png](/img/user/OperatingSystem/Linux/boot/attachments/Pasted%20image%2020240408092008.png)

### 3. root单用户模式下进行一些修改
比如重置密码，修改root用户login shell等等
### 4.退出单用户模式
```sh
exec /sbin/init
```