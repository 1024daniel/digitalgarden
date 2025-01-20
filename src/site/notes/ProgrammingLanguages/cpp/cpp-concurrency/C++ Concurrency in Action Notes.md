---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/cpp/cpp-concurrency/C++ Concurrency in Action Notes/","noteIcon":"3"}
---

# 1. Hello, world of concurrency in C++!
task switching vs hardware concurrecny
> in order to do the interleaving, the system has to perform a context switch every time it changes from one task to another, and this takes time. In order to perform a context switch, the OS has to save the state and instruction pointer for the currently running task, work out which task to switch to, and reload the CPU state for the task being switched to. The CPU will then potentially have to load the memory for the instructions and data for the new task into cache, which can prevent the CPU from executing any instructions, causing further delay.

 上下文切换：操作系统保存当前任务的状态和指令指针，准备好需要切换的任务，加载该任务的cpu state和指令和数据所需的memory到cache
 以下灰色的部分代表的是context switch导致的cpu delay
![Pasted image 20230226215638.png](/img/user/ProgrammingLanguages/cpp/cpp-concurrency/pics/Pasted%20image%2020230226215638.png)
process and thread
thread可以类比于一个developer，process可以类比于一个office，内部可以有一个或者多个developer，有developer需要的manual等一些resource
one thread per process vs multiple threads per process
```ad-note
进程多线程的prons：
- 一份资源就足够

cons
- thread比较难以专注任务，需要管理好一些shared resource
```

