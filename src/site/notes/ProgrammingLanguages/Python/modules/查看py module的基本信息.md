---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Python/modules/查看py module的基本信息/","noteIcon":"3"}
---

#module

### 1. 查看module的路径
1. 对于大的一个module的话可以直接通过`pip show <module_name> 来查看`
2. 对于一些模块可能存在pip无法查看的现象，可能是由于下载名和实际python环境内可见名不一致导致的，如果需要通过py环境实际module名来查看module的定义路径，可通过如下方法:

通过module的内置属性`__file__`

```py
import numpy
# 通过module的内置属性
print(numpy.__file__)
print(numpy.fft.__file__)


```
通过注册到`sys.modules`的信息
```py
import sys
import numpy
from numpy.fft import fft

# 通过module在sys.modules注册的信息
print(sys.modules['numpy'])
print(sys.modules['numpy.fft'])

```


判断一个module是否已经安装(包含requirement)
```py
import importlib
import pkg_resources
from pkg_resources import get_distribution

def is_installed(package: str)-> bool:
	importlib.reload(pkg_resources)
	try:
		get_distribution(package)
		return True
	except pkg_resources.DistributionNotFound:
		return False
	except:
		print('get distribution error')
		return False

package='numpy==1.26.4'
if is_installed(package):
	print(package, 'is installed')
else:
	print(package, 'is not installed')

```

### 2. 查看module的版本

```py
import mmcv
print(mmcv.__version__)

```