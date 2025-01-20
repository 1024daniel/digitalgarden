---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/symbol/","noteIcon":"3"}
---

## nm
for example an elf file named test
show nm dynamical symbols
> nm -D test

[nm output message meaning](https://man7.org/linux/man-pages/man1/nm.1p.html)


## symver


readelf  -Ws libcares.so.2.4.3 |grep "socket\\|Ndx\\|Symbol"

objcopy --redefine-sym socket@GLIBC_2.2.5=socket libcares.so.2.4.3

## LD_PRELOAD VS LD_LIBRARY_PATH
LD_PRELOAD 直接指定预先加载哪一个so文件，从而加载了里面的符号，会覆盖可执行文件需要的默认so所调用的符号的实现
LD_LIBRARY_PATH指的是为可执行文件提供标准路径之外的so搜索路径，保障可执行文件可找到制定的so，这里先找的是so文件名，之后在所有的需要找到的so列表里面找到对应的符号
总之，LD_PRELOAD指定的是so，以替换符号，LD_LIBRARY_PATH是路径，以提供所需要可执行文件需要找到的so所在的路径
