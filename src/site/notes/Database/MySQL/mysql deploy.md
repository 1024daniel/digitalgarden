---
{"dg-publish":true,"permalink":"/Database/MySQL/mysql deploy/","noteIcon":"3"}
---

---
dg-publish: true
---
## deploy mysql
[install mysql in ubuntu 2204](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04)
```sh
apt install mysql-server
systemctl start mysql
sudo mysql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
mysql> use mysql;
mysql> update user set user.Host='%' where user.User='root';
确保配置文件中bind-address不为127.0.0.1，否则无法无法remote connect
select user,host from user;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
mysql> flush privileges;
mysql> exit;
```


## sysbench test

```sh
apt install -y sysbench;

sysbench oltp_insert --tables=5 --table_size=100 --mysql-host=localhost --mysql-port=3306 --mysql-db=sysbench --mysql-user=root --mysql-password=123456  prepare
```


## master and slave mode


