---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/ssh/","noteIcon":"3"}
---

https://feihu.me/blog/2014/env-problem-when-ssh-executing-command-on-remote/


#ssh

### 1.跨机容器之间ssh登录
```bash
#两端容器设置ssh通信收发端口(/etc/ssh/ssh_config发,/etc/ssh/sshd_config收)，与系统默认22不一样
Port 10222
#两端机器设置允许root密码登录(/etc/ssh/sshd_config)
PermitRootLogin yes
#修改使能
service ssh reload
#互信
ssh-copy-id node1

```

ssh connection remote operations
environment variables
remote ssh nohup no default output file

bashrc disable ps1 
https://unix.stackexchange.com/questions/32096/why-is-bashs-prompt-variable-called-ps1


如果想在集群中的多个节点分发任务或者修改文件，一个一个登录操作比较麻烦，可以通过在已和其他子节点免密的主节点通过ssh向集群中的子节点进行分发任务来减少工作量。
ssh连接执行命令与正常ssh交互式登录是存在差别的，系统一般将ssh执行命令视作风险比较高的操作，一些环境变量是缺失的，比如~/.bashrc第一行就判断当前会话是否是交互式登录，如果不是的话直接return
为了将我们任务分发时必要的环境变量保存，我们可以打开/etc/ssh/sshd_config文件中的PermitUserEnvironment选项，然后restart ssh
将每个子节点的必要的环境变量导入到~/.ssh/environment文件中，这样主节点命令行方式对于子节点的ssh会话都会读取该文件中的环境变量
```bash
PermitUserEnvironment yes
service ssh restart
env >> ~/.ssh/environment

```
多节点交互式登录设置的环境变量
https://tldp.org/LDP/abs/html/x17837.html

```bash
## here-string syntax
for i in {1..10}
do
  ssh node${i} "tee -a ~/.bashrc <<< 'export PATH=/usr/local/python3.8.12/bin:\$PATH'"
done
```

远程ssh连接进行多任务分发

```bash
#这里主节点进行任务分发时将子节点的train.sh的stdout输出到/dev/null，不输出到主节点的屏幕中，注意这里的train.sh最好本身就使用tee将输出导入到本地日志文件中
# 每次ssh命令之后会卡住，需要ctrl c 10次，注意不能ctrl c掉当前for循环命令，最后ctrl z加上bg来使得命令后台运行，最后使用disown来将for循环的后台任务与当前会话分离，防止ssh中断导致任务中断
# 这里使用&>的语法将stderr和stdout都导入到指定的日志文件可以防止出现错误时错误日志没法打屏(ssh连接形式无法打屏)导致ssh连接中断
for i in {1..10}
do
  nohup ssh node{i} "nohup bash train.sh &> nohup.out  &" > node${i}.log &
done

```


使用sshpass免重复输入密码登录服务器
#sshpass
```bash
sshpass -f psd_file ssh pve
sshpass -p psd ssh pve

```

### 2.通过中间节点登录远端服务器

> [!NOTE] Note
> 不需要保证中间节点能免密登录目标节点，只需要保证当前节点能链接中间节点，中间节点能链接目标节点，链接过程中可以使用密码，依次是当前节点登录中间节点使用中间节点的密码登录中间节点，当前节点使用目标节点密码登录目标节点，如果当前节点对于中间节点和目标节点都进行过免密的话最快了


```bash
# 通过配置文件配置， 这里我们配置登录6wrt_t_pve通过中间节点wrt,6wrt_t_pve中配置的hostname pve应为wrt的hosts文件里面配置有指定ip，同样6wrt里面配置的Hostname应为本机hosts文件已经配置的节点，不使用name的话可以直接使用ip
# 这里使用指定inet6主要是避免有的时候域名解析错误没到对应的ipv6地址而是localhost?
Host 6wrt
    Hostname ipv6url.com
    User root
    AddressFamily inet6
Host 6wrt_t_pve
    Hostname pve
    User root
    ProxyJump 6wrt

# 配置号之后可以直接登录target server
ssh 6wrt_t_pve

# 不使用配置文件的话可以直接使用如下方式直接登录
ssh -J middle_server target_server

```

```
Host 6wrt_t_pve
    HostName pve
    User root
    ProxyCommand ssh -W %h:%p root@6wrt
```
• -W %h:%p: Forwards the target host and port (%h and %p) through the jump host.

#forward
### 3.端口转发和隧道

```sh
# 其中 `-N` 表示不执行远程命令，只是建立隧道。
ssh -L [本地IP:]本地端口:目标服务器IP:目标端口 用户名@SSH服务器 -N
# 建立一个隧道本地的2777端口的包转发到172.31.224.191上的22端口，然后转发到remote的2777端口
ssh -L 2777:localhost:2777 root@172.31.224.191 -p 22
```