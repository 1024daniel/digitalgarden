---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/python/decorator/","noteIcon":"3"}
---

https://www.runoob.com/w3cnote/python-func-decorators.html

### 1.what is decortaor

#decorator
1. 装饰器快速为函数增加操作和逻辑，
2. warp可以保证装饰器不会覆盖被装饰对象的函数名称，注释文档以及参数列表等等
3. 

```py
from functools import wraps
 
def a_new_decorator(a_func):
    @wraps(a_func)
    def wrapTheFunction():
        print("I am doing some boring work before executing a_func()")
        a_func()
        print("I am doing some boring work after executing a_func()")
    return wrapTheFunction
 
@a_new_decorator
def a_function_requiring_decoration():
    """Hey yo! Decorate me!"""
    print("I am the function which needs some decoration to "
          "remove my foul smell")
 
print(a_function_requiring_decoration.__name__)
# Output: a_function_requiring_decoration

print(a_function_requiring_decoration())
# Output:
#I am doing some boring work before executing a_func()
#I am the function which needs some decoration to remove my foul smell
#I am doing some boring work after executing a_func()


```

### 2. property decorator
#property
property decorator可以把对应的函数当做属性一样获取，不需要加上()获取值

```py
class Money:
    def __init__(self, dollars, cents):
        self.total_cents = dollars * 100 + cents

    # Getter and setter for dollars...
    @property
    def dollars(self):
        return self.total_cents // 100
    
    @dollars.setter
    def dollars(self, new_dollars):
        self.total_cents = 100 * new_dollars + self.cents

    # And the getter and setter for cents.
    @property
    def cents(self):
        return self.total_cents % 100
    
    @cents.setter
    def cents(self, new_cents):
        self.total_cents = 100 * self.dollars + new_cents

m = Money(5,10)
print(f"now with dollars:{m.dollars},cents:{m.cents}, total cents:{m.total_cents}")
# now with dollars:5,cents:10, total cents:510
m.dollars+=2;
m.cents+=2;
print(f"now with dollars:{m.dollars},cents:{m.cents}, total cents:{m.total_cents}")
# now with dollars:7,cents:12, total cents:712

```

### 3. dataclass decorator

https://stackoverflow.com/a/74406814


自动添加__init__方法

```py
from dataclasses import dataclass
@dataclass
class Point:
	x: int
	y: int

a = Point(10,20)
#print(a)
print(a.__repr__())

```