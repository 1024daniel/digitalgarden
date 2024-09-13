---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/python/context manager/","noteIcon":"3"}
---


https://stackoverflow.com/questions/1984325/explaining-pythons-enter-and-exit

> [!NOTE] With statement
> **with** statement only works with objects that support the context management protocol (i.e. they have `__enter__` and `__exit__` methods). A class which implement both methods is known as context manager class.

with语句可以支持context manager的类，也就是实现了`__enter__`和`__exit__`的类，通过在with block的开始的时候会调用`__enter__`方法来申请资源，在with block结束的时候调用`__exit__`来进行释放资源

```py
 class Log:
    def __init__(self,filename):
        self.filename=filename
        self.fp=None    
    def logging(self,text):
        self.fp.write(text+'\n')
    def __enter__(self):
        print("__enter__")
        self.fp=open(self.filename,"a+")
        return self    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("__exit__")
        self.fp.close()

with Log(r"C:\Users\SharpEl\Desktop\myfile.txt") as logfile:
    print("Main")
    logfile.logging("Test1")
    logfile.logging("Test2")

```