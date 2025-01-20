---
{"dg-publish":true,"permalink":"/Database/MySQL/basic expressions/","noteIcon":"3"}
---

# SQL基础语句
变量
```bash
# user-defined variable is session specific
set @id = 10;
set @id = @id + 1;
# 可以使用以下引号来包含特殊字符，注意sql中变量名是非case-sensitive
set @'my-var' = 10;
set @"my-var" = 10;
set @`my-var` = 10;
#全局变量set的时候不需要加前缀的@，两个@@表示全局变量，一个@代表自定义变量
set gtid_next = xxxx;
select @@gtid_next;

```

### 1. 查看所有数据库，查看数据库中的表

>show databases; show tables;

### 2. 执行sql脚本

>source path\xxx.sql;

### 3. 登录mysql并在test数据库执行脚本

```sql

mysql –uroot –p123456 --default-character-set=utf8  -Dtest<d:\test\ss.sql #如果ss.sql中含有use test，就不需要-Dtest了

```

### 4. 查看当前所在数据库

>select database();
