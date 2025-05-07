---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Python/syntactic sugar/decorator/","noteIcon":"3"}
---

https://www.runoob.com/w3cnote/python-func-decorators.html
#decorator
# 1.what is decortaor
在 Python 中，装饰器（Decorator）是一种用于快速修改或扩展函数/方法行为的高级语法糖

按实现方式进行分类，装饰器可以分为
1. 内置装饰器
2. 函数装饰器
3. 类装饰器
# 内置装饰器
## 1.@wrap

warp可以保证装饰器不会覆盖被装饰对象的函数名称(`__name__`)，注释文档(`__doc__`)以及参数列表等等

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

## 2. @property
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


## 3.@staticmethod

## 4.@classmethod
## 5.@lru_cache


# 函数装饰器

# 类装饰器

## 自定义类装饰器
这里参考evalscope的代码示例
evalscope支持openai tgi等等多个接口对接后端大模型，对于输入指定的url可以判断是什么类型的接口，这里通过传入的url来选择指定的接口类进行输入处理，组装request
![Pasted image 20250507145059.png](/img/user/ProgrammingLanguages/Python/syntactic%20sugar/attachments/Pasted%20image%2020250507145059.png)
以openai接口处理类为例，如下图，第13行使用自定义装饰器进行装饰第14行定义的接口类
展开来看的话，
- `@register_api([...])`是一个带有参数的类装饰器，本质是一个装饰器工厂，接受一个参数列表，返回一个装饰器函数
- 上面返回的装饰器函数是`register_api`的内层函数，这个装饰器接受类作为参数进行修改/扩展或者增强

> [!NOTE] 注意
> 这里装饰器函数实现使用<font color="#00b050">闭包(嵌套函数，内部函数引用外部函数的作用域，外部函数返回这个内部函数)</font>的方式主要是将外层函数参数列表传递下去，因为在第13行会立即返回装饰器，在第14行才传入实际的类，通过虽然传入的函数列表已经脱离作用域，但是闭包是一种函数对象，可以<font color="#00b050">捕获并保存其定义时所在环境中的变量</font>，即使该环境已经离开作用域，这些变量任然可以被访问。这些被捕获的环境变量主要是存储在函数的`__closure__`中

![Pasted image 20250507145739.png](/img/user/ProgrammingLanguages/Python/syntactic%20sugar/attachments/Pasted%20image%2020250507145739.png)
![Pasted image 20250507150612.png](/img/user/ProgrammingLanguages/Python/syntactic%20sugar/attachments/Pasted%20image%2020250507150612.png)
## @dataclass

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