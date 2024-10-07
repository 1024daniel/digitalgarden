---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/python/pybind11/pybind11 build tutorial/","noteIcon":"3"}
---

#pybind11
https://bayareanotes.com/pybind-tutorial/
对于以下的模块示例代码，我们可以通过多种方法将这个module编译生成python可以直接调用的module，这里我们简单介绍三种比较常用的方法
```cpp

#include <pybind11/pybind11.h>

int add(int i, int j) {
    return i + j;
}

PYBIND11_MODULE(example, m) {
    m.def("add", &add, "A function which adds two numbers");
}
```

### 1. build with python setup

setup.py
```py
from setuptools import setup, Extension

module = Extension('example', sources=['example.cpp'])

setup(
    name='example',
    version='1.0',
    ext_modules=[module],
)

```
编译生成对应的模块`python setup.py build_ext --inplace`


### 2. build with cmake
创建CMakeLists.txt文件
```CMake
cmake_minimum_required(VERSION 3.0)
project(pybind11_example)

# Find Python
find_package(PythonLibs REQUIRED)
include_directories(${PYTHON_INCLUDE_DIRS})

# Find Pybind11
find_package(pybind11 REQUIRED)

# Add the example module
pybind11_add_module(example example.cpp)

# Link the module with Python
target_link_libraries(example PRIVATE ${PYTHON_LIBRARIES})

```

编译模块`mkdir build && cd build && cmake .. && make`

### 3. build manually
https://pybind11.readthedocs.io/en/stable/compiling.html#building-manually

```sh
## for linux
c++ -O3 -Wall -shared -std=c++11 -fPIC $(python3 -m pybind11 --includes) example.cpp -o example$(python3-config --extension-suffix)
## for macOS
c++ -undefined dynamic_lookup  -O3 -Wall -shared -std=c++11 -fPIC $(python3 -m pybind11 --includes) example.cpp -o example$(python3-config --extension-suffix)
```

- 这里的`-undefined dynamic_lookup`旨在macos平台allows the linker to defer the resolution of undefined symbols until runtime
- python3 -m pybind11 --includes可以为gcc加上pybind11相关的头文件路径


