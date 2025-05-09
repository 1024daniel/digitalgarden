---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/packages/rpm/","noteIcon":"3"}
---

#rpm #yum

#### 设置yum安装包时不进行ssl校验

#ssl
```bash
# 编辑/etc/yum.conf,在[main]下加上
sslverify=false

```


#### 安装指定版本的package
![Pasted image 20240510111951.png](/img/user/OperatingSystem/Linux/packages/attachments/Pasted%20image%2020240510111951.png)

#### 查看rpm包,例如test安装的文件

```bash
rpm -qa|grep test
rpm -ql test
```

#### 查看rpm包的信息
`rpm -qi test`

#### yum 更新index

```bash
#更新包源的索引，类似apt update。需要注意的事yum update等同于apt upgrade
yum clean all
yum makecache
```

#### 查看安装包在那个repo中包含
```bash
repoquery -i gazelle
```

#### glibc降级
```bash
yum downgrade glibc\* --allowerasing
```
#### 查看文件对应的rpm包
`rpm -qf filepath`

#### 下载rpm到本地

```bash
yum install libtirpc-gen --downloadonly --destdir .
#使用yum-utils中的命令
yumdownloader package_name

```

#### rpm包解压到本地

```bash
rpm2cpio xxx.rpm | cpio -div
```


### 查看本地有哪些repo
```sh
yum repolist

```


