---
{"dg-publish":true,"permalink":"/AI/ascend/大ep/openeuler大ep部署/","noteIcon":"3"}
---

#docker #skopeo
### 1.安装k8s
#### 首先配置好k8s相关的yum源
```sh
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-aarch64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

```

```sh
yum install -y kubelet-1.23.0-00 kubeadm-1.23.0-00 kubectl-1.23.0-00
kubelet --version
```

#### 安装k8s基础镜像

linux主机如果没法连接到k8s镜像仓库
可以先本地机器下载好相关镜像之后导入到远程服务器

```sh
#!/bin/bash

IMAGES=(
  "registry.k8s.io/kube-apiserver:v1.23.0"
  "registry.k8s.io/kube-controller-manager:v1.23.0"
  "registry.k8s.io/kube-scheduler:v1.23.0"
  "registry.k8s.io/kube-proxy:v1.23.0"
  "registry.k8s.io/etcd:3.5.0-0"
  "registry.k8s.io/pause:3.6"
  "registry.k8s.io/coredns/coredns:v1.8.6"
  "docker.io/calico/node:v3.23.5"
  "docker.io/calico/cni:v3.23.5"
  "docker.io/calico/kube-controllers:v3.23.5"
)

PLATFORM=linux/arm64

for IMAGE in "${IMAGES[@]}"; do
  echo "Pulling $IMAGE for $PLATFORM"
  docker pull --platform=$PLATFORM "$IMAGE"
done

echo "Saving to k8s-arm64.tar"
docker save -o k8s-arm64.tar "${IMAGES[@]}"

echo "Removing local images"
for IMAGE in "${IMAGES[@]}"; do
  docker image rm "$IMAGE"
done

```

上面安装涉及到先导入到本地机器系统,也需要安装docker desktop，下面可以通过skopeo来直接下载到本地，不需要导入到系统
这里直接下载，但是会存在问题是镜像文件会丢失repo和tag信息
```sh
#!/bin/bash

# 镜像列表
IMAGES=(
  "registry.k8s.io/kube-apiserver:v1.23.0"
  "registry.k8s.io/kube-controller-manager:v1.23.0"
  "registry.k8s.io/kube-scheduler:v1.23.0"
  "registry.k8s.io/kube-proxy:v1.23.0"
  "registry.k8s.io/etcd:3.5.1-0"
  "registry.k8s.io/pause:3.6"
  "registry.k8s.io/coredns/coredns:v1.8.6"
  "docker.io/calico/node:v3.23.5"
  "docker.io/calico/cni:v3.23.5"
  "docker.io/calico/kube-controllers:v3.23.5"
)

ARCH=arm64
OS=linux

mkdir -p k8s-arm64-images

for IMAGE in "${IMAGES[@]}"; do
  # 提取镜像名称和版本（用于生成文件名）
  NAME=$(basename "$(echo $IMAGE | cut -d':' -f1)")
  TAG=$(echo $IMAGE | cut -d':' -f2)
  FILE="k8s-arm64-images/${NAME}-${TAG}.tar"

  echo "📦 正在保存 $IMAGE 为 $FILE"
  skopeo copy \
    --override-arch=$ARCH \
	--override-os=$OS \
    docker://$IMAGE \
    docker-archive:$FILE
done

echo "✅ 所有镜像已保存到 k8s-arm64-images/ 目录中"

```

使用下面脚本来显式指定写入repo和tag信息到镜像文件
```sh
#!/bin/bash

# 镜像列表
IMAGES=(
  "registry.k8s.io/kube-apiserver:v1.23.0"
  "registry.k8s.io/kube-controller-manager:v1.23.0"
  "registry.k8s.io/kube-scheduler:v1.23.0"
  "registry.k8s.io/kube-proxy:v1.23.0"
  "registry.k8s.io/etcd:3.5.1-0"
  "registry.k8s.io/pause:3.6"
  "registry.k8s.io/coredns/coredns:v1.8.6"
  "docker.io/calico/node:v3.23.5"
  "docker.io/calico/cni:v3.23.5"
  "docker.io/calico/kube-controllers:v3.23.5"
)

ARCH=arm64
OS=linux

mkdir -p k8s-arm64-images

for IMAGE in "${IMAGES[@]}"; do
  # 分离 repo 和 tag
  REPO=$(echo "$IMAGE" | cut -d':' -f1)
  TAG=$(echo "$IMAGE" | cut -d':' -f2)
  
  # 只取镜像名最后一段用于文件名，如 "kube-apiserver"
  NAME=$(basename "$REPO")

  FILE="k8s-arm64-images/${NAME}-${TAG}.tar"

  echo "📦 正在保存 $IMAGE 为 $FILE"

  skopeo copy \
    --override-arch=$ARCH \
    --override-os=$OS \
    docker://$IMAGE \
    "docker-archive:${FILE}:${REPO}:${TAG}"
done

echo "✅ 所有镜像已保存到 k8s-arm64-images/ 目录中"

```


拉起ubuntu基础镜像
```sh
skopeo copy \
  --override-arch=arm64 \
  --override-os=linux \
  docker://docker.io/library/ubuntu:22.04 \
  docker-archive:ubuntu-22.04-arm64.tar:ubuntu:22.04

```

#### k8s初始化

```sh
mkdir /var/lib/kubelet
swapoff -a
kubeadm reset -f
rm -rf /etc/cni/net.d /root/.kube/
unset http_proxy
unset https_proxy
unset HTTP_PROXY
unset HTTPS_PROXY

```
以下只需要主节点执行
```sh
kubelet_version='1.23.0'
host_ip=175.99.1.2
kubeadm init --kubernetes-version=v${kubelet_version} --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=${host_ip}
mkdir -p $HOME/.kube
cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

```


all_label.sh
```sh
master=

```