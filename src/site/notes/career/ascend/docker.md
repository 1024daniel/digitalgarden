---
{"dg-publish":true,"permalink":"/career/ascend/docker/","noteIcon":"3"}
---

```dockerfile
# syntax = docker/dockerfile:experimental
FROM ubuntu:18.04
ARG PYTHON_VERSION=3.8.12
ARG PYTHON_VERSION_SHOT=3.8
ARG TOOLKIT_PATH=/usr/local/Ascend/ascend-toolkit/latest
WORKDIR /tmp


RUN echo "deb [trusted=yes] https://repo.huaweicloud.com/ubuntu/ bionic main restricted universe multiverse\n\
    deb-src [trusted=yes] https://repo.huaweicloud.com/ubuntu/ bionic main restricted universe multiverse\n\
    deb [trusted=yes] https://repo.huaweicloud.com/ubuntu/ bionic-updates main restricted universe multiverse\n\
    deb-src [trusted=yes] https://repo.huaweicloud.com/ubuntu/ bionic-updates main restricted universe multiverse\n\
    deb [trusted=yes] https://repo.huaweicloud.com/ubuntu/ bionic-backports main restricted universe multiverse\n\
    deb-src [trusted=yes] https://repo.huaweicloud.com/ubuntu/ bionic-backports main restricted universe multiverse\n\
    deb [trusted=yes] https://repo.huaweicloud.com/ubuntu/ bionic-security main restricted universe multiverse\n\
    deb-src [trusted=yes] https://repo.huaweicloud.com/ubuntu/ bionic-security main restricted universe multiverse\n" > /etc/apt/sources.list && \
    apt-get -o "Acquire::https::Verify-Peer=false" update && \
    apt-get -o "Acquire::https::Verify-Peer=false" install -y --no-install-recommends ca-certificates && \
    apt-get -o "Acquire::https::Verify-Peer=false" install -y --no-install-recommends \
    wget vim sudo bzip2 wget make tar curl g++ pkg-config unzip numactl \
    libsqlite3-dev libzip-dev liblzma-dev zlib1g-dev libbz2-dev libopenblas-dev libblas3 liblapack3 \
    liblapack-dev libblas-dev gfortran libhdf5-dev libffi-dev libicu60 \
    libxml2 libssl-dev git patch libfreetype6-dev libpng-dev libgl1-mesa-glx less htop bc \
    gcc cmake zlib1g pciutils libxslt1-dev net-tools  libxml2-dev libxslt-dev libsqlite3-dev openssl openssh-server git-lfs && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*


ENV LD_LIBRARY_PATH=/usr/local/python${PYTHON_VERSION}/lib: \
    PATH=/usr/local/python${PYTHON_VERSION}/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# 安装python和pip
RUN curl -k https://mirrors.huaweicloud.com/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tar.xz -o Python-${PYTHON_VERSION}.tar.xz && \
    tar -xf /tmp/Python-${PYTHON_VERSION}.tar.xz && \
    cd Python-${PYTHON_VERSION} && ./configure --prefix=/usr/local/python${PYTHON_VERSION} --enable-shared --enable-loadable-sqlite-extensions --enable-optimizations && \
    make -j8 && make install && \
    ln -sf /usr/local/python${PYTHON_VERSION}/bin/python3 /usr/bin/python3 && \
    ln -sf /usr/local/python${PYTHON_VERSION}/bin/python3 /usr/bin/python && \
    ln -sf /usr/local/python${PYTHON_VERSION}/bin/pip3 /usr/bin/pip3 && \
    ln -sf /usr/local/python${PYTHON_VERSION}/bin/pip3 /usr/bin/pip && \
    cd .. && \
    rm -rf Python* && \
    mkdir -p ~/.pip  && \
    echo '[global] \n\
    index-url=https://mirrors.aliyun.com/pypi/simple\n\
    trusted-host=mirrors.aliyun.com' >> ~/.pip/pip.conf && \
    pip3 install pip -U


RUN git config --global http.sslVerify false 

#安装cmake
RUN wget https://github.com/Kitware/CMake/releases/download/v3.28.1/cmake-3.28.1-linux-x86_64.sh --no-check-certificate  && \
    mkdir /usr/local/cmake  && \
    bash cmake-3.28.1-linux-x86_64.sh  --skip-license --prefix=/usr/local/cmake 

ENV PATH=/usr/local/cmake/bin:$PATH \
LD_LIBRARY_PATH=/usr/local/cmake/lib:/$LD_LIBRARY_PATH


#torch.whl   https://download.pytorch.org/whl/cpu/torch-2.1.0%2Bcpu-cp38-cp38-linux_x86_64.whl#sha256=9e5cfd931a65b38d222755a45dabb53b836be31bc620532bc66fee77e3ff67dc
#torch_npu   https://gitee.com/ascend/pytorch/releases/download/v5.0.0-pytorch2.1.0/torch_npu-2.1.0-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
COPY ./pkgs/torch*.whl  /tmp/
# RUN pip3 install torch-* --force-reinstall  && \
#     pip3 install --upgrade torch_npu*.whl  --force-reinstall  && \
#     git config --global http.sslVerify false && \
#     rm -rf /tmp/*  && \
#     rm -rf /root/.cache/pip
#编译安装torch 和npu
RUN pip3 install torch-* --force-reinstall    && \
    rm -rf /tmp/*
RUN cd /root/ && \
    git clone https://gitee.com/ascend/pytorch.git -b v2.1.0-5.0.0 --depth 1 && \
    cd pytorch && \
    bash ci/build.sh --python=${PYTHON_VERSION_SHOT} && \
    pip3 install --upgrade dist/torch_npu-*  --force-reinstall  && \

#安装pip包
RUN pip3 install pip -U && \
    pip3 install decorator numpy sympy pyyaml pathlib2 grpcio grpcio-tools protobuf scipy \
    requests attrs wheel Pillow wheel setuptools matplotlib opencv-python \
    sklearn==0.0 pandas lxml pytest xdoctest  torchvision==0.16.0 && \
    pip3 install cffi pycocotools  easydict psutil pathlib2 absl-py && \
    git clone https://gitee.com/mirrors_NVIDIA/dllogger && \
    pip3 install /tmp/dllogger && \
    rm -rf /tmp/* \
    rm -rf /root/.cache/pip



#set env
ENV LD_LIBRARY_PATH=$TOOLKIT_PATH/fwkacllib/lib64/:/usr/local/python${PYTHON_VERSION}/lib/python${PYTHON_VERSION_SHOT}/site-packages/torch/lib:/usr/local/Ascend/driver/lib64/:/usr/local/Ascend/driver/lib64/common/:/usr/local/Ascend/driver/lib64/driver/:/usr/local/Ascend/add-ons/:/usr/lib/aarch64_64-linux-gnu:$LD_LIBRARY_PATH \
PATH=$PATH:$TOOLKIT_PATH/fwkacllib/ccec_compiler/bin/:$TOOLKIT_PATH/toolkit/tools/ide_daemon/bin/ \
ASCEND_OPP_PATH=$TOOLKIT_PATH/opp/ \
OPTION_EXEC_EXTERN_PLUGIN_PATH=$TOOLKIT_PATH/fwkacllib/lib64/plugin/opskernel/libfe.so:$TOOLKIT_PATH/fwkacllib/lib64/plugin/opskernel/libaicpu_engine.so:$TOOLKIT_PATH/fwkacllib/lib64/plugin/opskernel/libge_local_engine.so \
PYTHONPATH=$TOOLKIT_PATH/fwkacllib/python/site-packages/:$TOOLKIT_PATH/fwkacllib/python/site-packages/auto_tune.egg/auto_tune:$TOOLKIT_PATH/fwkacllib/python/site-packages/schedule_search.egg:$PYTHONPATH \
ASCEND_AICPU_PATH=$TOOLKIT_PATH


COPY ./pkgs/Ascend-cann-toolkit*.run  /tmp/
COPY ./pkgs/Ascend-cann-kernels-*.run  /tmp/
#安装toolkit
RUN echo y | bash ./Ascend-cann-toolkit*.run --install-path=/usr/local/Ascend/ --install --quiet --install-for-all && \
    echo y | bash ./Ascend-cann-kernels-*.run  --install --install-for-all && \
    rm -rf /tmp/* 



#apex包编译安装 
RUN git clone -b master https://gitee.com/ascend/apex.git && \
    cd apex && \
    bash scripts/build.sh --python=${PYTHON_VERSION_SHOT} && \
    cd apex/dist/ && \
    pip3 uninstall apex && \
    pip3 install --upgrade apex* 
# COPY ./pkgs/apex-*.whl  /tmp/
# RUN pip3 install --upgrade /tmp/apex*  && \
#     rm -rf /tmp/* 


# mindx_toolbox
COPY ./pkgs/Ascend-mindx*  /tmp/
RUN echo y | bash ./Ascend-mindx*.run --install-path=/usr/local/Ascend/ --install --quiet  && \
    rm -rf /tmp/* 


# mpich 安装   (hccl_test前置)
#https://www.hiascend.com/document/detail/zh/canncommercial/70RC1/devtools/auxiliarydevtool/HCCLpertest_16_0002.html
RUN wget https://www.mpich.org/static/downloads/3.2.1/mpich-3.2.1.tar.gz --no-check-certificate && \
    tar -xzvf mpich-3.2.1.tar.gz  && \
    cd mpich-3.2.1  && \
    ./configure --disable-fortran --prefix=/usr/local/mpich-3.2.1 && \
    make -j8 && \
    make install &&\
    echo "export PATH=\$PATH:/usr/local/mpich-3.2.1/bin" >> /root/.bashrc  &&\
    echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:/usr/local/mpich-3.2.1/lib" >> /root/.bashrc  && \
    rm -rf /tmp/* 


# cann 环境变量
RUN  echo "source /usr/local/Ascend/ascend-toolkit/set_env.sh">>/root/.bashrc
RUN  echo "source /usr/local/Ascend/ascend-toolkit/latest/bin/setenv.bash">>/root/.bashrc

WORKDIR /root/

#Model-link迁移
#https://gitee.com/ascend/AscendSpeed/tree/master/examples/llama2
RUN pip3 install --no-use-pep517 -e git+https://github.com/NVIDIA/Megatron-LM.git@23.05#egg=megatron-core && \
    pip install deepspeed==0.9.2 && \
    git clone https://gitee.com/ascend/DeepSpeed.git -b v0.9.2 deepspeed_npu && \
    cd deepspeed_npu && \
    pip3 install -e ./ && \
    cd ..  && \
    git clone https://gitee.com/ascend/AscendSpeed.git && \
    cd AscendSpeed && \
    pip install -r requirements.txt  && \
    mkdir logs && \
    mkdir ckpt && \
    mkdir dataset && \
    mkdir dataset/llama/ && \
    cd dataset 
# wget https://huggingface.co/datasets/tatsu-lab/alpaca/resolve/main/data/train-00000-of-00001-a09b74b3ef9c3b56.parquet --no-check-certificate


# Modelzoo 适配
RUN git clone https://gitee.com/ascend/ModelZoo-PyTorch.git && \
    cd ModelZoo-PyTorch/PyTorch/built-in/foundation/ChatGLM2-6B/ && \
    find ./ -name "requirements.txt" -exec sed -i 's/transformers==4.29.0/ /g' {} +   && \
    pip install -r requirements.txt   && \
    pip3 install -U transformers && \
    pip3 install deepspeed 


```