---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/cpp/build-set/gcc/gcc/","noteIcon":"3"}
---

<font color="#00b050">recommend links:</font>
https://developer.apple.com/library/archive/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/lldb-command-examples.html

#gcc
### 1.查看当前gcc支持那些C++ standards
```bash
gcc -v --help 2> /dev/null | sed -n '/^ *-std=\([^<][^ ]\+\).*/ {s//\1/p}'

$(gcc -print-prog-name=cc1) --help | grep std
```

### 2.gcc查看具体的so的还有可执行文件的绝对路径
```bash
gcc --print-file-name=libstdc++.so
gcc --print-prog-name=cc1
```

### 3. 查看符号定义位置
https://stackoverflow.com/questions/57866267/how-does-the-n-option-to-image-lookup-in-lldb-operate-compared-to-the-s
```bash
# im loo -r -s hello
image lookup -r -s hello

```
![Pasted image 20240121233343.png](/img/user/pics/Pasted%20image%2020240121233343.png)

### 4. 查看gcc/g++默认搜索路径
- `-E`: Preprocess only; do not compile, assemble or link.
- `-xc++`: Specify the language (C++)
- `-`: Take input from stdin instead of a file
- `-v`: Display the programs invoked by the compiler.
- `< /dev/null`: Send emptiness to stdin.

```bash
#print当前库文件搜索路径
g++ -print-search-dirs
# 预处理
g++ -E -xc++ - <<< "#include <iostream>"
g++ -E -xc++ - -v < /dev/null
man g++ | col -b | grep -B 2 -e '-std=.* This is the default'
```

### 5. glibc相关操作

```bash
#glibc降级，比较危险的操作，可能导致多个基础命令无法使用
yum downgrade glibc\*
#查看当前glibc版本
ldd --version
```