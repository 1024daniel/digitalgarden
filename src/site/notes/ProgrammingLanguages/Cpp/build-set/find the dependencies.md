---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Cpp/build-set/find the dependencies/","noteIcon":"3"}
---

https://blog.csdn.net/zhuzitop/article/details/80388159

gcc:

### 在PATH中找到可执行文件程序的路径。  
```sh
export PATH =\$PATH:\$HOME/bin  
```
  
### gcc找到头文件的路径  
```sh
C_INCLUDE_PATH=/usr/include/libxml2:/MyLib  
export C_INCLUDE_PATH  
```
  
### g++找到头文件的路径  
```sh
CPLUS_INCLUDE_PATH=$CPLUS_INCLUDE_PATH:/usr/include/libxml2:/MyLib  
export CPLUS_INCLUDE_PATH  
```
  
### 找到动态链接库的路径  
```sh
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/MyLib  
export LD_LIBRARY_PATH  
```
  
### 找到静态库的路径  
```sh
LIBRARY_PATH=$LIBRARY_PATH:/MyLib  
export LIBRARY_PATH
```


库文件依赖文件
```sh
objdump -T <static_library.a> | grep NEEDED
nm --undefined-only <static_library.a>

ldd dynamic_library.so


```