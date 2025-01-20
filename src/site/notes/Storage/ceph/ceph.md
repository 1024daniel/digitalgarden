---
{"dg-publish":true,"permalink":"/Storage/ceph/ceph/","noteIcon":"3"}
---

#storage/ceph
## config
show a config of ceph, ceph-osd@0 service should  be in the node where we execute below code.
> ceph daemon osd.0 config show |grep mon_max

```sh
ceph daemon osd.1 perf dump
```


## remove osd

```sh
i=$1
ceph osd down osd.$1 # if the osd is up
ceph osd crush reweight osd.$i 0.0 # need to execute in the host that osd belong to, or would ecounter the asok is not found error 
systemctl stop ceph-osd@$i.service # or ceph osd stop osd.$i
ceph osd out osd.$i
ceph osd crush remove osd.$i
ceph osd rm osd.$i
ceph auth del osd.$i
```

## take out osd from cluster

```sh
i=$1
ceph osd out osd.$i
ceph crush remove osd.$i
ceph crush remove <wronghost>

```


## add in the outed osd
```sh
for i in $(seq $1 $2)
do
	ceph osd in $i
	ceph osd crush add osd.$i 1 host=$3 ## host shoule be the host where the osd was created. you can check use ceph osd find $i
done
```

## remove related rpms in client side
```bash
set -x
function remove_rpm {
	temp_rpm=$(rpm -qa|grep $1)
	for item in $temp_rpm
	do
		rpm -e $item --nodeps
	done
}
remove_rpm ceph
remove_rpm rbd
remove_rpm rados
remove_rpm rgw

```
## create pool
```bash
## pg_num is a exponet of 2 that is not smaller than to 100*osd_num/replicate_num
ceph osd pool create poolname 128
```
## delete pool
```bash
ceph osd pool rm poolname poolname true
ceph osd pool delete poolname poolname --yes-i-really-really-mean-it
```

## cleanceph.sh
pdsh -w ceph[1-3] "bash cleanceph.sh"
```bash
set -x
set +e
ps aux|grep ceph|grep -v cleanceph|awk '{print $2}'|xargs kill -9

umount /var/lib/ceph/osd/*
rm -rf /var/lib/ceph/osd/*
rm -rf /var/lib/ceph/mon/*
rm -rf /var/lib/ceph/mds/*
rm -rf /var/lib/ceph/bootstrap-mds/*
rm -rf /var/lib/ceph/bootstrap-osd/*
rm -rf /var/lib/ceph/bootstrap-rgw/*
rm -rf /var/lib/ceph/bootstrap-mgr/*
rm -rf /var/lib/ceph/tmp/*
rm -rf /etc/ceph/*
rm -rf /var/run/ceph/*
```

## clean backend partition of osd
#cut #dmsetup
```bash
dmsetup ls | awk '{$1=$1}1' |cut -d ' ' -f 1 | grep ceph | xargs dmsetup remove
lsblk | grep -e -nvme | cut -d '-' -f 2 | awk '{$1=$1}1' | cut -d ' ' -f 1 | xargs -i dd if=/dev/zero of=/dev/{} bs=512k count=1
```

## save ceph pool in local filesystem
```bash
ceph osd pool create test 1024
rbd create test/foo --size 1G
rados -p test export test
ceph osd pool rm test test true
ceph osd pool create test test 1024
rados -p test import test
```

## get perf metric info

```bash
ceph daemon osd.0 perf dump prioritycache:kv

```

查看rbd image当前已经已使用的空间
```bash
rbd du pool/image
```
查看磁盘使用情况
```bash
iostat -xmt 3
```
ceph线程



tp_osd_tp
kv_sync
kv_finally
asyncmessage


ceph配置参数
```bash
rbd_op_threads = 4 #fio操作进程创建的librbd_tp线程个数，不是越多越好
```