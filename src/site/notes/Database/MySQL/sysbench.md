---
{"dg-publish":true,"permalink":"/Database/MySQL/sysbench/","noteIcon":"3"}
---

## a example script using sysbench to test mysql
### prepare
1. install pdsh, if we use pdcp, you need install pdsh in every node
> ./configure --enable-static-modules --without-rsh --with-ssh --without-ssh-connect-timeout-option --with-dshgroups
2. prepare data for oltp_read_write and oltp_update_index
3. network interface tuning
set_all_eth.sh
```bash
sh set_eth.sh enp133s0f1
sh set_eth.sh enp133s0f0
sh set_eth.sh enp125s0f0
sh set_eth.sh enp125s0f1
sh set_eth.sh enp125s0f2
sh set_eth.sh enp125s0f3
```
set_eth.sh
```bash
systemctl stop irqbalance
systemctl disable irqbalance
ethtool -G $1 tx 8192
ethtool -G $1 rx 8192
ethtool -C $1 adaptive-rx off
ethtool -C $1 adaptive-tx off
ethtool -C $1 rx-usecs 0     ## more interrupts but low latency
ethtool -C $1 tx-usecs 0
ethtool -C $1 rx-frames 0
ethtool -C $1 tx-frames 0
ethtool -C $1 rx-gro-hw off

```
reset_all_eth.sh
```bash
sh reset_eth.sh enp133s0f1
sh reset_eth.sh enp133s0f0
sh reset_eth.sh enp125s0f0
sh reset_eth.sh enp125s0f1
sh reset_eth.sh enp125s0f2
sh reset_eth.sh enp125s0f3
```
reset_eth.sh
```bash
ethtool -C $1 adaptive-rx on
ethtool -C $1 adaptive-tx on
ethtool -C $1 rx-usecs 20
ethtool -C $1 tx-usecs 20
ethtool -C $1 rx-gro-hw on
```

auto_test.sh
```bash
#!/bin/bash
set -x
function sysbench-test {
	for i in 50 100 120 160 200; do
		numactl -C 48-95 sysbench --db-driver=mysql --mysql-host=192.168.3.10 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench_375 --table_size=2000000 --tables=375 --events=0 --time=900 --thread=$i --percentile=95 --report-interval=10 oltp_read_write run &>> $logfile
	done
	for i in 50 100 120 160 200; do
		numactl -C 48-95 sysbench --db-driver=mysql --mysql-host=192.168.3.10 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench_375 --table_size=2000000 --tables=375 --events=0 --time=900 --thread=$i --percentile=95 --report-interval=10 oltp_update_index run &>> $logfile
	done
	for i in 50 100 120 160 200; do
		sysbench --db-driver=mysql --mysql-host=192.168.3.10 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench_insert --table_size=2000000 --tables=375 --events=0 --time=900 --thread=$i --percentile=95 --report-interval=10 oltp_insert cleanup &>> $logfile
		sysbench --db-driver=mysql --mysql-host=192.168.3.10 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench_insert --table_size=0 --tables=375 --events=0 --time=900 --thread=$i --percentile=95 --report-interval=10 oltp_insert prepare &>> $logfile
		numactl -C 48-95 sysbench --db-driver=mysql --mysql-host=192.168.3.10 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench_375 --table_size=2000000 --tables=375 --events=0 --time=900 --thread=$i --percentile=95 --report-interval=10 oltp_update_index run &>> $logfile
	done
}

function configswitch {
	logfile=sysbench-log-$1.txt
	echo > $logfile
	# change configuration
	{
		date
		echo ===== start to change configuration to $1 ======
		pdcp -w ceph[1-3] ceph.$1.conf /etc/ceph/ceph.conf
		pdsh -w ceph[1-3] "systemctl restart ceph.target"
		sleep 5
		local configeth=reset_all_eth.sh
		if [ "$1" = "huawei" ]; then
		configeth=set_all_eth.sh
		fi
		pdsh -w ceph[1-3] "cd /home/xxl && bash ${configeth}"
		sleep 5
		pdsh -w ceph[1-3] "ethtool -c enp133s0f1"
		echo ===== finish changing configuration to $1 =====
	} &>> $logfile
}

configswitch huawei
sysbench-test
configswitch suyan
sysbench-test
```
