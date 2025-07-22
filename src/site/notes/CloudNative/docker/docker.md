---
{"dg-publish":true,"permalink":"/CloudNative/docker/docker/","tags":["docker"],"noteIcon":"3"}
---

![timo-stern-EvcUtLF12XQ-unsplash.jpg|100%](/img/user/banner/timo-stern-EvcUtLF12XQ-unsplash.jpg)

### 1.pull
对于一个使用buidx构建的同时支持x86和arm的镜像在hub上拉取下来之后其实会根据你本地的架构拉取不同的layer
对于hub上面的镜像有不同的层，有manifest等原数据文件记录了层之间的联系，类似下面
```
📦 Manifest Index: yourname/app:latest
  ├── linux/amd64: layers (sha256:aaa, sha256:bbb)
  └── linux/arm64: layers (sha256:ccc, sha256:ddd)
```
通过`docker manifest inspect yourname/app:latest`可以查看hub上面的镜像支持哪些架构
![Pasted image 20250424100413.png](/img/user/CloudNative/docker/attachments/Pasted%20image%2020250424100413.png)
如果想在x86下载arm的镜像到本地，需要强制指定arm
```sh
docker pull --platform linux/amd64 nginx
```
镜像拉取下来可以查看是否是指定架构的镜像文件

> [!NOTE] 注意
> docker manifest inspect看的是远程hub上的镜像元数据，docker image inspect看的是本地镜像的元数据

```sh
docker image inspect swr.cn-south-1.myhuaweicloud.com/ascendhub/mindie:1.0.0-800I-A2-py311-openeuler24.03-lts |grep Architecture

```
![Pasted image 20250424101959.png](/img/user/CloudNative/docker/attachments/Pasted%20image%2020250424101959.png)

### 1. docker的网络

#### 1.1代理
docker和maven，npm类似，代理和系统代理不一样需要单独配置，具体配置参考[docker docs](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)
关于docker的配置代理的文件主要是两个，可以根据需要编辑其中一个或者两个一起
/etc/docker/daemon.json
![Pasted image 20230922224955.png|100%](/img/user/pics/Pasted%20image%2020230922224955.png)
/etc/systemd/system/docker.service.d/http-proxy.conf

   
编辑添加代理之后可以通过以下命令生效并查看是否成功生效

```bash
systemctl daemon-reload
# 一般不需要restart docker即可看到生效systemctl cat docker
# systemctl restart docker
systemctl show --property=Environment docker
```
其中daemon.json的方式只能docker高版本支持(23.0以上)
代理生效之后docker login, docker pull相关命令可以正常联网解析域名
#### 1.2 mtu and dns
`docker build`的时候如果存在`timeout`或者`cannot resolve host`之类的问题可以考虑mtu和dns的配置

对于timeout可能是我们docker的mtu和宿主机的网卡的mtu不一致，docker默认的是1500
```sh
# 查看物理机mtu网卡
ifconfig eth0 | grep mtu
# 设置物理机网卡mtu
ifconfig eth0 mtu 1500
# 设置docker的mtu?
ifconfig docker0 mtu 1500

```

/etc/docker/daemon.json
```json
{
	"mtu": 1500,
	"dns": ["8.8.8.8"]
}

```
/etc/default/docker
```sh
DOCKER_OPTS="--mtu=1500"

```
docker-compose
```yaml
networks:
	default:
		driver: bridge
		driver_opts:
			com.docker.network.driver.mtu: 1500

```


### 2. Dockerfile的编写

[[CloudNative/docker/docker build/dockerfile\|dockerfile]]
```bash
ARG base=debian:v0  # 指定基础镜像，给出默认值，可以从docker build指定值覆盖
FROM $bash
ARG arg1 # 声明参数, 注意docker build指定的build arg最好得在dockerfile中声明
ARG arg2 # 声明参数
ARG WORKDIR=/home/pkgs
USER root
# 在镜像内部创建文件夹
WORKDIR $WORKDIR
CP *.deb /home
RUN command1 $arg1 && \
command2 $arg2
```
在Dockerfile所在的路径执行

```bash
docker build -t . --build-arg arg1='a' --build-arg='b'
#设置clean build，不使用上一次build的cache
docker build -t base:v0 --no-cache .

```
dockerfile中两个连续的RUN之间的状态并不连续一致的，每一个RUN开始都会进入到/home/currentUser目录下，而非后一个RUN命令开始会在前一个RUN最后所在的目录下
### 3. Docker的镜像
docker镜像可以直接由dockerfile制备，或者可以将运行中的容器当前的文件系统和状态保存下来

```bash
docker commit container_name repo:tag #将当前容器状态保存到镜像repo:tag
docker save repo:tag > repo.tar # 将镜像保存到本地文件repo.tar
docker load < repo.tar # 从本地文件导入镜像


docker export container-name > ex.tar # 将容器当前文件系统保存，不包含之前的layer
docker import ex.tar new_img:tag # 从export出来的文件导出成本地镜像

# 查看具体一个image的大小
docker images image:tag 

# alias a image tag
docker tag repo:tag newrepo:newtag
docker tag imagehash newrepo:newtag

# 查看容器的基础镜像
docker inspect --format='{{.Config.Image}}' <container_name>
docker ps --format "table {{.Names}}\t{{.ID}}\t{{.Image}}" 
# 查看镜像
docker image inspect <image-name> --format='{{.Architecture}}'
```

export/import与save/load的区别
> export是从容器导出，save是从镜像导出，save会保存所有layer的信息，export不会保存所有layer的信息，因此导出文件更小，但是没法回滚

<font color="#ffc000">对于想在镜像中保存文件，需要在docker run -v指定映射物理机文件夹子路径之外创建文件夹并保存，因为-v指定的物理机的路径之后将容器导出成镜像文件并不会保存下来，只是一个空文件夹</font>
### 4.容器的进出

退出当前container而不stop当前container(exit会stop容器)

```bash
ctrl p q # 不kill当前session
ctrl q   # kill当前session
```


进入到容器
```bash
docker exec -it container_name bash
docker attach container_name 
```
> [具体来说，`docker exec`命令会在容器内部启动一个新的进程，以执行指定的命令](https://www.zhihu.com/question/276485274)[1](https://www.zhihu.com/question/276485274). [而`docker attach`命令则会将标准输入和输出连接到容器内部的PID 1，实现对容器内正在执行的终端的附着](https://www.zhihu.com/question/276485274)[1](https://www.zhihu.com/question/276485274)

<font color="#ff0000">如果是使用docker attach 到一个容器终端，如果关闭电脑或者关闭终端可能导致容器exit，按ctrl d也会导致容器推出，因为docker attach上的是一个主会话进程</font>
<font color="#ff0000">docker exec -it登录的话ctrl q只会将docker exec创建的会话kill掉，不会影响容器主进程</font>
### 5. 容器文件互传

```bash
docker cp container_name:path host_path
docker cp host_path container_name:path
```

### 6. 更改docker存放image位置
```bash
# 查看当前的docker的data root
docker info | grep "Docker Root Dir"
docker info -f '{{ .DockerRootDir }}'
```

首先先停掉docker之后开始备份数据，<font color="#ff0000">数据备份不能丢失文件权限等信息</font>，否则容器进入存在没有权限等问题
```sh
rsync -aP /var/lib/docker /data/

```
`vim /etc/docker/daemon.json`
```json
{ 
   "data-root": "/data/docker"
}
```

```bash
# restart and check whether the modifies is avtivated
systemctl daemon-reload
systemctl restart docker
docker info -f '{{ .DockerRootDir }}'
docker info | grep "Docker Root Dir"

```

### 7.容器路径映射


> [!NOTE] 注意
> 对于容器中映射系统路径需要注意最好不好映射和容器镜像中已有的文件夹路径一样的宿主机路径，否者容器内部的该路径会被系统路径覆盖，比如pip editable project所在路径被覆盖导致缺少包

```sh
# 查看当前容器映射的物理机路径
docker inspect --format {{.HostConfig.Binds}} <container_name>

```
查看物理机中/var/lib/docker/overlay2/中长hash文件夹对应的容器
```sh
containers=$(docker ps -aq)
for i in $containers;do
	docker inspect $i| grep $i || echo $i
done

```

### 8.容器配置
#shm
修改shm大小


```sh
#获取容器的配置路径
docker inspect <container name> |grep <container id>
#到对应路径修改hostconfig.json的ShmSize的配置

```

### 9. 更名

```sh
docker rename docker_old_name docker_new_name

```

### 10.容器大小排序
```sh
docker ps -a --format '{{.ID}}\t{{.Names}}\t{{.Size}}' | sort -h -k 3

```


### 11. 特权
#privileged
```sh
docker ps -q | xargs docker inspect --format '{{.Id}}: {{.HostConfig.Privileged}}' | grep 'true'

```
### 11.查看容器空间大小

```sh
docker ps -s

```


##  Docker Volume