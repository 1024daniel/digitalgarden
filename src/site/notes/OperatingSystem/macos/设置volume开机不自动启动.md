---
{"dg-publish":true,"permalink":"/OperatingSystem/macos/设置volume开机不自动启动/","noteIcon":"3"}
---


https://discussions.apple.com/docs/DOC-7942
#mac #diskutil #fstab
### 1. 获取卷的uuid

mac设置卷的挂载行为和linux的fstab类似，我们首先需要获取指定卷的UUID
MacOS中的Volume一般都会挂载在`/Volumes/`文件夹下

如果想要开机不自动挂载的卷当前已经挂载了
```sh
# 当前卷已经挂载
diskutil info /Volumes/private
# 当前卷未挂载，先找到对应的/dev/路径
diskutil list
diskutil info /dev/disk3s7

```
![Pasted image 20240910095047.png](/img/user/OperatingSystem/macos/attachments/Pasted%20image%2020240910095047.png)

![Pasted image 20240910095335.png](/img/user/OperatingSystem/macos/attachments/Pasted%20image%2020240910095335.png)

### 2.编辑fstab
vifs是内置的能安全编辑fstab的工具
```sh
sudo vifs
```
![Pasted image 20240910095520.png](/img/user/OperatingSystem/macos/attachments/Pasted%20image%2020240910095520.png)
