---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/disk/from disk to lvm/","tags":["lvm","vgextend","disk"],"noteIcon":"3"}
---


### 1.逻辑卷创建
- 裸盘分区
- 物理分区创建pv
- pv纳入vg
- lv将vg中空闲的空间全部纳入
![Pasted image 20240205110736.png](/img/user/OperatingSystem/Linux/disk/attachments/Pasted%20image%2020240205110736.png)

### 2.逻辑卷挂载
- 逻辑卷创建文件系统(<font color="#00b050">挂载到linux所在的文件系统前必须先格式化创建文件系统</font>)
- 挂载逻辑卷到挂载点文件夹
- /etc/fstab加入机器启动自动挂载逻辑卷
![Pasted image 20240229113509.png](/img/user/OperatingSystem/Linux/disk/attachments/Pasted%20image%2020240229113509.png)
/etc/fstab文件加入挂载lvm卷
#fstab
```bash
/dev/mapper/vgruntime-lvruntime /runtime ext4 default 0 2
# 路径可能不准确，部分/dev/sda /dev/sdb重启可能会变化，可以指定uuid，对于对应磁盘的uuid和fs type可以从blkid /dev/sda获取
UUID='' /runtime ext4 default 0 0
```
