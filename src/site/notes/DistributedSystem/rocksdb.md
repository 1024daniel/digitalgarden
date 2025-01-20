---
{"dg-publish":true,"permalink":"/DistributedSystem/rocksdb/","noteIcon":"3"}
---

---
dg-publish: true
---
![Pasted image 20231119165350.png|100%](/img/user/pics/Pasted%20image%2020231119165350.png)
## key feature
高效发挥存储硬件性能的嵌入式KV存储引擎
针对写多读少的场景
基于leveldb开发
分布式领域的三大文章GFS，BigTable，MapReduce
redis->pika
mysql->myrocks
TiDB
MySQL InnoDB B+ tree, 读多写少
LSM-Tree非数据结构，数据一种组织方式

 内存顺序io  >> 内存随机IO  ≈ 磁盘顺序IO >> 磁盘随机IO

1. 查询，有序
2. 数据冗余 拆分，压缩合并