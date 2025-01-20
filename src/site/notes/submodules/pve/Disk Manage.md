---
{"dg-publish":true,"permalink":"/submodules/pve/Disk Manage/","noteIcon":"3"}
---

![Pasted image 20231119154239.png|100%](/img/user/pics/Pasted%20image%2020231119154239.png)
#storage/disk #lvs #pvs #extend #disk
I give a small size to proxmox when a install it, which make the local-lvm size not enough to create many vm and store iso. So I need to extend the local-lvm to the full size of my os disk.
## 1. Extend the partition
before my operation, the partition and lvm is like below:
![Pasted image 20230424235158.png](/img/user/submodules/pve/pics/Pasted%20image%2020230424235158.png)

```sh
fdisk /dev/sda    ## make sure the disk is the last partition of the disk
>> p              ## rember the fist sector number
>> d
>> 3
>> n              ## make sure the first sector number is the same before
>> p              ## primary partition
>> w
```
now lsblk output will show that /dev/sda3 is 930.5G, but still we need to extend the pve-data logical volume
## 2.Resize the physical volume
use pvdisplay, the result show that the pv /dev/sda3 is still not change(200G).
![Pasted image 20230425000225.png](/img/user/submodules/pve/pics/Pasted%20image%2020230425000225.png)
```sh
pvresize /dev/sda3
```
after resize the pg, the vg is also back to normal
![Pasted image 20230425000328.png](/img/user/submodules/pve/pics/Pasted%20image%2020230425000328.png)

## 3.extend the logical volume
With the vg having enough space, we can extend the local-lvm
![Pasted image 20230425001038.png](/img/user/submodules/pve/pics/Pasted%20image%2020230425001038.png)

```sh
lvextend -r -l +100%FREE /dev/pve/data
```

以上是针对lvm进行扩容，对于挂载在物理分区上的文件系统，比如根目录，如果需要进行扩容，需要按一下操作
![Pasted image 20231102015046.png|90%](/img/user/pics/Pasted%20image%2020231102015046.png)
对于根目录所在的分区所在的磁盘![Pasted image 20231102015046.png](/img/user/pics/Pasted%20image%2020231102015046.png)
对于新建磁盘不能删除ext4 signature?
![Pasted image 20231115002235.png](/img/user/submodules/pve/pics/Pasted%20image%2020231115002235.png)
[openwrt extend disk](https://blog.csdn.net/ls0111/article/details/128769859)
![Pasted image 20231115014303.png](/img/user/submodules/pve/pics/Pasted%20image%2020231115014303.png)
### 物理盘内逻辑卷扩展

```bash
fdisk /dev/sda
#删除根目录所在分区及后面的分区
#创建新分区，新分区的范围应为根目录所在的起始位置加上盘的最终位置
## 需要注意新建分区必须起始位置对齐，最终位置需要不小于之前的结尾，不然保存修改之后reboot会导致系统不可逆的崩溃
#保存修改之后
partprobe /dev/sda
df -Th
# 更改没有通知到内核，可能需要重启生效更改
reboot

```

根目录扩展到新加的逻辑卷
![Pasted image 20231114000739.png](/img/user/submodules/pve/pics/Pasted%20image%2020231114000739.png)

## 4.精简池和精简卷
#thinpool
Thin Pool实际上也是一个LV
精简卷（Thin Volume）是一种逻辑卷管理技术，旨在提高存储效率。与传统逻辑卷不同，精简卷使用按需分配（Thin Provisioning），即在实际使用数据时才分配存储空间，而不是在创建时预先分配全部空间。
![Pasted image 20250101043445.png](/img/user/submodules/pve/attachments/Pasted%20image%2020250101043445.png)
这里可以看到base-102-disk-0等都是data这个精简池里面的，而这个data精简池也是一个LV
从Attr中看到关键字t则表示是精简lv的含义
> [!NOTE] 注意
> 精简池和普通的逻辑卷不同的是精简池不直接存储用户数据，只为Thin volume提供支持，因此没有类似普通逻辑卷在/dev/下面有挂载路径，不过对于精简池的操作可以不通过挂载路径而是通过vg_name/thin_pool方式
精简池和精简卷相关操作:
```sh
#创建一个大小为 50GB 的精简池：
lvcreate -L 50G --thinpool thin_pool thin_vg
#从精简池中创建一个大小为 10GB 的精简卷：
lvcreate -V 10G --thin -n thin_lv thin_vg/thin_pool
#扩展精简池的大小：
lvextend -L +10G thin_vg/thin_pool
```



## Related posts
[resize partition](https://www.ibm.com/docs/en/cloud-pak-system-w3550/2.3.3?topic=images-extending-partition-file-system-sizes)

[lv, vg and pv](https://networklessons.com/uncategorized/extend-lvm-partition)

[resize lv](https://unix.stackexchange.com/questions/138090/cant-resize-a-partition-using-resize2fs)
