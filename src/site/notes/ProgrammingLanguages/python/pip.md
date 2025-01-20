---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/python/pip/","noteIcon":"3"}
---

#python/pip#pip
![freestocks-flOVXZWbjJ4-unsplash.jpg|100%](/img/user/banner/freestocks-flOVXZWbjJ4-unsplash.jpg)
### pip常用命令
pip配置文件:~/.pip/pip.config(linux), %APPDATA/pip/pip.ini(windows)，配置示例，pip使用系统代理就行

```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
extra-index-url=
    https://pypi.mirrors.ustc.edu.cn/simple/
    https://pypi.douban.com/simple/
    https://pypi.python.org/simple/
[install]
trusted-host = tuna.tsinghua.edu.cn
    pypi.douban.com
    mirrors.ustc.edu.cn
    python.org


```



```bash
pip show rich # 显示当前已安装包的信息
pip install rich==13.5.2 -i http://... --trusted-host mirrors.huaweicloud.com
# 查看config
pip config list
pip -v config list # 可查看到具体加载的那些配置文件和内容
pip config set global.proxy ....
pip uninstall rich # 卸载包的缓存文件在~/.cache/pip中，之后重新install会从缓存中获取
pip cache purge # 清除所有缓存文件

pip install --no-index --find-links=deps --no-build-isolation psycopg[c]

# 升级pip包
pip install --upgrade transformers
# 查看pip包有什么版本
pip install datasets==
#设置超时时间
pip install datasets --default-time=200
#安装包，不使用cache文件
pip install torch --no-cache-dir

#保存当前环境配置
pip freeze > requirements.txt
# 下载指定包到本地，对于requirements里面可以不用指定版本的限定，可以直接用仓库里面那种空格后面接一个版本的形式
pip download -r requirements.txt --dest . --no-deps
```

editable project
执行以下命令会执行当前目录的setup.py，将当前项目代码作为模块引入到环境的
对于无法联网的环境需要安装这个文件夹相关的module，可以先在外部环境pip install -e .当前目录生成egg-link文件。打包当前文件夹传到无法联网环境直接执行pip install -e .安装


```bash
pip install -e .
# or
python setup.py install

```

![Pasted image 20231028220804.png|80%](/img/user/pics/Pasted%20image%2020231028220804.png)



pip下载包到本地

```bash
pip download megatron==0.1 --no-deps
# pip默认会下载当前pip对应的本地python版本的whl包，我们可以指定别的python版本
pip download megatron==0.1 --python-version 3.9 --no-deps

```


conda环境使用参考:[[ProgrammingLanguages/python/conda\|conda]]