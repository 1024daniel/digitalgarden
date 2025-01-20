---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/python/built-in functions/","noteIcon":"3"}
---

https://www.pythontutorial.net/python-oop/python-__repr__/

### 1. `__str__` and `__repr__`


|     | `__str__`                                                                                                                                                      | `__repr__`                                                                                                                                                                                                                                                                                                                                            |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 含义  | returns a string representation of an object <br>that is human-readable,<br>通常如果未自定义实现,会调用`__repr__`<br>`str()`最后调用此方法，通常`print(a)`会调用`str()`,最后调用到a的`__str__` | returns a string representation of an object that is machine-readable<br>Typically, the `__repr__()` returns a string that can be executed and yield the same value as the object.<br>In other words, if you pass the returned string of the `object_name.__repr__()` method to the `eval()` function, you’ll get the same value as the `object_name` |
|     |                                                                                                                                                                |                                                                                                                                                                                                                                                                                                                                                       |


```py

class Person:
    def __init__(self, first_name, last_name, age):
        self.first_name = first_name
        self.last_name = last_name
        self.age = age

    def __repr__(self):
        return f'Person("{self.first_name}","{self.last_name}",{self.age})'

person = Person("John", "Doe", 25)
print(repr(person))
# using str()
print(person)

```

### 2. `__call__`

The `__call__` method allows an instance of a class to be called as if it were a function. This can be useful for objects that represent operations or transformations.
```py

class Adder:
    def __init__(self, amount):
        self.amount = amount

    def __call__(self, value):
        return value + self.amount

add_five = Adder(5)
print(add_five(10))  # Output: 15
```
3. 