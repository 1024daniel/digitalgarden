---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/linux任务后台运行的几种方法/","noteIcon":"3"}
---


#disown #screen
1. disown
```sh
bash task.sh
ctl z
bg
# 加上-h选项之后当前会话jobs能看到对应的job，但是关闭当前会话，job的父进程会切换到1，不会收到SIGHUP信号
disown -h %1
#直接切换对应pid的owner
disown pid

```
2. screen
```sh
# create session with specific name
screen -S session_name
# resume from the default session
screen -r
# resume from the specific session name
screen -r <name>
# list all screen sessions
screen -ls
# detach from current screen session
ctl a d
# exit and kill current screen session
ctl d
```
3. nohup
```sh
nohup bash task.sh &> task.log &

```