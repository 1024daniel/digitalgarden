---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/python/modules/查看py module路径/","noteIcon":"3"}
---

#module
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