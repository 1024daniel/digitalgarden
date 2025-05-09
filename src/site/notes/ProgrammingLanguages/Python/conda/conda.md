---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Python/conda/conda/","noteIcon":"3"}
---

conda和pip的区别:https://stackoverflow.com/a/20994790/14637253
conda和miniconda获取链接
https://repo.anaconda.com/archive/
https://repo.anaconda.com/miniconda/


### conda 安装

```sh
# 安装之后需要执行conda init，这会在linux的~/.bashrc，mac的~/.bash_profile里面加上conda需要的一些初始化设置好让终端打开就能初始化相关环境变量，如果删除anaconda之后重新安装的话需要先清理对应的bashrc的代码片段
conda init
# 对于mac需要设置打开terminal生效默认环境，需要在~/.zshrc中设置,~/.zprofile中设置没有生效
conda config --set auto_activate_base true	# 默认进入base环境, 对应的设置会写入~/.condarc

```


### 1.环境创建

将指定目录创建conda环境
https://blog.metman.top/index.php/archives/85/
更换conda环境优先安装位置，对于根目录所在磁盘满了，但是其他磁盘所在目录空间足够比较友好
```yaml title:~/.condarc
envs_dirs:
   - /path/to/new/envs/dir

```


```bash
conda create --prefix=/path/to/your/env
conda activate /path/to/your/env
```

创建环境一般方法
```sh
conda create -n modellink python=3.8
conda info
conda env list
conda activate modellink



```

基于当前环境配置安装pip包

```bash
pip freeze > requirements.txt
conda create --name new_env requirements.txt

```


### 2.环境复用
#conda
#### 在线安装方式
**创建一个新的Python 3.8环境：** 首先，在Conda中创建一个新的Python 3.8环境。你可以使用以下命令来创建一个名为`py38env`的环境：

- `conda create --name py38env python=3.8`
    
- **激活新环境：** 激活新创建的Python 3.8环境：
    
- `conda activate py38env`
- **复制包列表：** 现在，你需要从Python 3.7环境中导出已安装包的列表。可以使用以下命令导出包列表：
- `conda list --export > package_list.txt`

- **安装包到新环境：** 使用以下命令，将导出的包列表安装到新的Python 3.8环境中：   

`conda install --name py38env --file package_list.txt`

这将安装与Python 3.7环境相同的包到新的Python 3.8环境中，可能存在部分包不支持3.8，需要手动升级了
#### 离线拷贝方式

克隆已有环境
```bash
conda create -n env2 --clone env1

```
**通过硬链接复制已安装的包：**

1. 首先，在Python 3.8环境中创建一个新的虚拟环境（如果尚未创建），并激活它：
- `conda create --name py38env python=3.8 
   `conda activate py38env` 
- 找到Python 3.7环境中的`site-packages`目录，这是已安装包的位置。你可以使用以下命令查找它：
- `conda list --name py37env`
    查找输出中的 `site-packages` 目录。
- 切换到Python 3.8环境，然后创建硬链接以复制Python 3.7环境中的包到Python 3.8环境。假设你找到的`site-packages`目录是 `/path/to/py37env/lib/python3.7/site-packages`，可以使用以下命令：
	`ln -s /path/to/py37env/lib/python3.7/site-packages/* /path/to/py38env/lib/python3.8/site-packages/` 
- 这会在Python 3.8环境中创建硬链接到Python 3.7环境中的包。 
- 验证已经复制了包，你可以运行以下命令检查：

 `conda list --name py38env` 
这将显示已经安装的包列表。
这种方法可以帮助你在不重新下载包的情况下复用Python 3.7环境中的包到Python 3.8环境中。然而，要注意的是硬链接可能会在不同的环境之间引入一些潜在问题，因此在执行此操作之前请谨慎考虑。确保你理解这种方法可能带来的潜在风险。