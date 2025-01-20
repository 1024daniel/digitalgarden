---
{"dg-publish":true,"permalink":"/CodeSnippets/获取存储进程当前iops/","noteIcon":"3"}
---

#shell/numfmt

```bash
#!/bin/bash
demoHost=171.6.63.20
# start fio test
function start(){
	nohup sh /home/xxl/test/main.sh block test rand 4k > nohup.out  2>&1 &  
}
## start fio test
start
isRunning=true
function gc_status(){
	local result=`systemctl is-active globalcache`
	if [ $result != "active" ];then
		echo "gc is not running"
		isRunning=false
		return 1
	fi
	echo "gc is running"
	isRunning=true
	return 0

}
rValue=0
wValue=0
total=0

function get_iops(){
        local latestMessage=`tail -n 1 nohup.out|grep IOPS`
        echo latestMessage is $latestMessage
        local ioData="${latestMessage##*r=}"
        ioData="${ioData%%IOPS*}" # 175k,w=75.4k
        rValue="${ioData%,*}"
        rValue="${rValue/k/K}"
        if [[ -z $rValue ]];then
                rValue=0
        fi
        rValue=`numfmt --to=none --from=si $rValue`
        wValue="${ioData#*w=}"
        wValue="${wValue/k/K}"
        if [[ -z $wValue ]];then
                wValue=0
        fi
        wValue=`numfmt --to=none --from=si $wValue`
        echo rValue is $rValue
        echo wValue is $wValue
	total=`expr $rValue + $wValue`
	echo ------------------------------------------ total is $total ------------------------------------------------------
        #total=`echo|awk -v total=$total 'BEGIN{printf "%0.0f",total/10}'`
        total=`expr $total / 10`
        echo ------------------------------------------ value to write: $total -----------------------------------------------

}

sleep 1
MAX_TIME=120
runTime=0
while ([ "$isRunning" = "true" ] && [ $runTime -le $MAX_TIME ])
do
	echo globalcache is running: $isRunning
	sleep 1
	runTime=$(($runTime+1))
	echo runTime is $runTime
	## get iops value
	get_iops

	ssh $demoHost "echo ${runTime} >> /opt/hall/storage-boostkit/time"
	if [ ${total} -ne 0 ];then
		ssh $demoHost "echo ${total} >> /opt/hall/storage-boostkit/query.result"
	fi
	gc_status
done

ps -ef|grep fio|grep -v grep|awk '{print $2}'|xargs kill -9
ps -ef|grep "sh /home/xxl/test/main.sh block test rand "|grep -v grep|awk '{print $2}'|xargs kill -9

ssh $demoHost "echo '1' > /opt/hall/storage-boostkit/status"

```
