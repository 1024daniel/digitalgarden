---
{"dg-publish":true,"permalink":"/AI/nvidia/nvidia相关资源/","noteIcon":"3"}
---

#nvidia #docker #cuda
https://blog.csdn.net/seasermy/article/details/105298646

[nvidia申请账号密钥密码](https://ngc.nvidia.com/setup/api-key)

[nvidia相关镜像tag](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/pytorch/tags)

nvidia cuda toolkit各个版本下载：
https://developer.nvidia.com/cuda-toolkit-archive
```sh
# 环境中存在多个cuda toolkit版本的时候，指定特定的cuda版本, 注意环境中的CUDA_HOME环境变量可能优先级更高，需要先unset
export CUDA_PATH={path_of_cuda_11.8}
export GCC_HOME={path_of_gcc_10.2.0}
export MPFR_HOME={path_of_mpfr_4.1.0}
export LD_LIBRARY_PATH=${GCC_HOME}/lib64:${MPFR_HOME}/lib:${CUDA_PATH}/lib64:$LD_LIBRARY_PATH
export PATH=${GCC_HOME}/bin:${CUDA_PATH}/bin:$PATH
export CC=${GCC_HOME}/bin/gcc
export CXX=${GCC_HOME}/bin/c++


```

如果当前系统cuda版本和想要的cuda版本不一致，为了不破坏系统的cuda版本，可以使用conda进行环境隔离(比如PyTorch 1.11 官方仅支持到 CUDA 11.x，当前系统环境cuda版本是12.1)
```sh
conda install pytorch torchvision torchaudio cudatoolkit=11.3 -c pytorch
conda install -c conda-forge cudatoolkit-dev=11.3

export PATH=$CONDA_PREFIX/bin:$PATH
export CUDA_HOME=$CONDA_PREFIX
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH

python -c "import torch; print(torch.cuda.is_available()); print(torch.version.cuda)"
nvcc --version

```

查看cuda版本
https://www.cnblogs.com/wuliytTaotao/p/11453265.html
```sh
nvcc --verison
```



```sh

docker login nvcr.io
Username: $oauthtoken
Password: Z2k3YWdzbmpydDhiZ205Mjdkcm1xZjFmYjoxYTI1NzAxMy01YzA0LTRmNTctYjk4Zi1iMTE3N2MxOTQ1Mjk
```


nvidia docker container setup

```sh
docker run -it  --gpus all --net host --name xxl-dahua1 \
--shm-size=64G \
-v /data:/data \
nvcr.io/nvidia/pytorch:23.04-py3 /bin/bash
```

查看当前nvidia服务器是否有nvlink，以及nvlink的速率

```sh
$ nvidia-smi -q | grep -i nvlink
        NVLink 0: Enabled2
        NVLink 1: Enabled
        Nvlink Speed: 25.78 GB/s
        Nvlink Counter: 0
        Nvlink Counter: 0

nvidia nvlink --status
```


指定卡训练
```sh
export CUDA_VISIBLE_DEVICES=7

```