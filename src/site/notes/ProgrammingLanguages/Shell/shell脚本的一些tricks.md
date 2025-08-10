---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Shell/shell脚本的一些tricks/","noteIcon":"3"}
---

#shell
### 1. 字符串内替换

```bash
os="Mac OS X"
# 语法: ${variable//pattern/replacement}
echo ${os// /}
```

### 2. 文件拆分和压缩
#split 

```cardlink
url: https://stackoverflow.com/a/50606414
title: "How can I split a large text file into smaller files with an equal number of lines?"
description: "I've got a large (by number of lines) plain text file that I'd like to split into smaller files, also by number of lines.  So if my file has around 2M lines, I'd like to split it up into 10 files t..."
host: stackoverflow.com
image: https://cdn.sstatic.net/Sites/stackoverflow/Img/apple-touch-icon@2.png?v=73d79a89bded
```


```bash
#默认aa,ab这种后置
split -l 100 large_file.log part_
#以400行作为一个part拆分，使用数字后缀，前缀指定为part_,并显示分割进度
split -l 400 -d large_file.log part_ --verbose
#按文件大小拆分
split -b 1G large_file.log part_
# 只拆分出一部分文件，当使用机器可用空间不足的时候可以参考
split -n l1/10 input output
#拆分数据还原
cat part_* > merge_file.log
#压缩文件
tar -zcvf large_file.tar.gz large_file
#解压
tar -zxvf latrge_file.tar.gz
zip -r large.zip large
# 压缩的文件比较大的话可以使用pv显示进度,这里-表示stdout or stdin，取决于命令中的位置，需要明确的是这不是shell内置的而是取决于具体命令的实现
tar cf - $path | pv -s $(du -sb $path|awk '{print $1}') | gzip > file.tar.gz

```

#file
### 3.查找文件
#find #file
[参考](https://www.myfreax.com/find-large-files-in-linux/)

```shell
# 查找大文件
find . -xdev -type f -size +100M -print | xargs ls -lh | sort -k5,5 -h -r | head

fd -g config.json -x cat

# 文件夹大小排序
du -h --max-depth=1 | sort -h

# 查看文件夹内文件个数
find . -type f |wc -l
tree | tail -1

# find and delete
find /path/to/directory -type f -size +100M -delete

# find and delete
find . -xdev -type f -size +100M -print

# find and execute command, + means all find results can be passed as arguments to comamnd
find /path/to/directory -type f -name "*.txt" -exec ls -l {} +

```

### 4. 使用alias
#alias #shopt

```bash
#一些软件安装是直接拉取指定链接文件直接本地执行，这些脚本文件中可能会使用wget和curl来拉取依赖
#这些工具可能会存在ssl认证的问题，为了能顺利执行，我们可以将脚本拉取到本地，在脚本内第一行使用alias
# 第一行务必使用shopt开启alias展开。。
shopt -s expand_aliases
alias curl='curl -k'
alias wget='wget --no-check-certificate'

```

### 5.日志按时间排序
#sort #log
```bash
#文件夹中日志文件有多个，按：分割之后包含时间信息的在第4~6列
grep -rn ERROR /home/plog|sort -t ":" -k4,6

```

### 6.性能计算
#performance #awk #average
> 日志样式:
>  iteration       70/    5000 | consumed samples:         8960 | consumed tokens:     36700160 | elapsed time per iteration (ms): 1905.5 | learning rate: 1.250E-05 | global batch size:   128 | lm loss: 4.108219E+00 | loss scale: 1.0 | grad norm: 5.817 | actual seqlen:  4096 | number of skipped iterations:   0 | number of nan iterations:   0 | samples per second: 67.175 | TFLOPs: 98.24 | time (ms)
```bash
grep -rn "iteration (ms):" 7b_8m.log|awk -F ":" '{print $5}'|awk -F "|" '{print $1}' |awk '{sum+=$1} END {printf "average performance= %f tokens/s/p",sum/NR}'

# 排除第三个参数指定的个数数据, awk传递变量
grep -rn "iteration (ms):" $1|awk -F ":" '{print $5}'|awk -F "|" 'NR>dismiss {print $1}' dismiss=$3 |awk '{sum+=$1} END {printf ",avg iteration time: %fms, average performance= %f tokens/s/p\n",sum/NR, perbz*4096*1000*NR/sum}' perbz=$2
# 对于删除第一个数据，可以在同一个awk中，计算平均数时总数是NR-1，或者只直接下一个awk，总数还是NR
awk -F',' 'NR > 1 {sum+=$3} END {print sum / (NR - 1)}' cpu.csv 
```

### 7.设置ssh登录时的当前路径位置
#login #ssh #pwd
```bash
for i in {1..10}
do
  ssh node${i} "tee -a /etc/profile <<< 'cd /home/ModelLink/examples/llama2"
done

```

### 8.指定环境变量执行脚本
#env
```bash
env -i var1=$var1 var2=$var2 test.sh options

```

### 9. 遍历解析获取命令选项
```bash
while [ $# -gt 0 ];do
        ...
        if []; then
	        var1=$1
	    elif []; then
	        var2=$1
        OPTIONS="$OPTIONS $1"
        shift
done

```

### 10.校验文件是否存在
```bash
#校验文件存在并且是个regular file, excludes directories, symbolic links, block devices, character devices, etc
[ -f /usr/bin/curl ]
#校验文件是否存在，不管类型，如文件夹，链接文件等等
[ -e /usr/bin/curl ]
#在-f的基础上校验文件是否具有x权限
[ -x /usr/bin/curl ]
```

### 11.日志重定向
#redirect
```bash
# 脚本开头加上这一行将脚本输出重定向到指定文件
exec >>/root/log 2>&1

```


### 12.字符串分割

```sh
    read -ra parts <<< "$1"
    model_type="${parts[0]}"
    if [ "$model_type" == "pa" ]; then
        data_type="${parts[1]}"
    fi

```

### 13.数组
#array

```bash
arr=(1 2 3)
arr+=( 4 )
echo ${arr[@]}
echo "=========" element
for i in ${arr[@]}
do
	echo $i
done
echo "=========" index
for i in ${!arr[@]}
	echo ${arr[$i]}
done

```
### 14.文件过滤
```sh
ll |cut -d ' ' -f 10|awk '{$1=$1}1'
#awk -F指定空格，输入字符串多个空格连在一起会被当成一个间隔符，和cut不一样
ll|awk -F ' ' '{print $10}'

```

### 15. 路径

```sh
# 保证从什么路径执行当前脚本，脚本环境的当前路径为脚本所在路径文件夹
SCRIPT_DIR=$(
    cd $(dirname ${BASH_SOURCE:-$0})
    pwd
)
CALLER_DIR=$PWD


```

### 16.文件压缩 解压
```sh
# 指定密码压缩
zip -r -e logs.zip logs
unzip

```


### 17. 拷贝软连接
```sh
rsync -l link_file w1:`pwd`

```


### 18.patch

```sh
patch -p1 < diff.patch
patch -p1 -R diff.patch
# 保持交互模式，键盘可用
patch -p1 -R -i diff.patch
git apply  -R diff.patch

```