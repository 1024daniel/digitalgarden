---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/Build System/","noteIcon":"3"}
---

#make
## cmake
cmake ..
cmake --build  .
cmake --install .
## make
```sh
make -j
make install -n     //  just print what files will install in where but does not install
make install
make target-name -j // make specific target
```

Makefile
- $< 表示源文件，这里指代SRC
- $@表示makefile中target name，这里也就是OBJ，目标文件
```sh
SRC = main.cpp
OBJ = main.o

all: $(OBJ)

$(OBJ): $(SRC)
    $(CXX) $(CXXFLAGS) $(CPPFLAGS) $< -o $@



```

https://makefiletutorial.com/#targets
https://unix.stackexchange.com/questions/116547/what-do-and-in-a-makefile-mean
https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html
基于megatron-lm 23.05学习python调用cpp

初步解释:
第一个target是default，前提依赖是也就是help.cpython-311-darwin.so，这里环境以macos举例
所有以.cpython-311-darwin.so后缀的target的依赖文件是当前目录下的所有cpp文件，生成这些so的方法是第9行指定的使用cpp编译器加上对应的flag，源文件就是%.cpp,输出结果就是指定的target，%$(LIBEXT):


1. `$@` The file name of the target of the rule. If the target is an archive member, then ‘\$\@’ is the name of the archive file. In a pattern rule that has multiple targets (see Introduction to Pattern Rules), ‘$@’ is the name of whichever target caused the rule's recipe to be run.
2. `$<` The name of the first prerequisite. If the target got its recipe from an implicit rule, this will be the first prerequisite added by the implicit rule.

```sh info:9,10
CXXFLAGS += -O3 -Wall -shared -std=c++11 -fPIC -fdiagnostics-color
CPPFLAGS += $(shell python3 -m pybind11 --includes)
LIBNAME = helpers
LIBEXT = $(shell python3-config --extension-suffix)

default: $(LIBNAME)$(LIBEXT)
# 注意这里这里表示通配符的%，在匹配target和prerequisites的除去后缀的文件名的部分是一样的，
#，helpers.cpython-38-x86_64-linux-gnu.so匹配helpers.cpp，不能匹配其他的cpp文件
%$(LIBEXT): %.cpp
	$(CXX) $(CXXFLAGS) $(CPPFLAGS) $< -o $@

```

当前python函数所在文件编译对应的so
```py
def compile_helpers():
    """Compile C++ helper functions at runtime. Make sure this is invoked on a single process.
    """
    import os
    import subprocess

    command = ["make", "-C", os.path.abspath(os.path.dirname(__file__))]
    if subprocess.run(command).returncode != 0:
        import sys

        log_single_rank(logger, logging.ERROR, "Failed to compile the C++ dataset helper functions")
        sys.exit(1)

```