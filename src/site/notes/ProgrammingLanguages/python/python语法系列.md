---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/python/python语法系列/","noteIcon":"3"}
---

### 1. list comprehension
reference:https://realpython.com/list-comprehension-python/

```python
# new_list = [expression for member in iterable if conditional]

matrix=[
		 [0,0,0],
		 [1,1,1],
		 [2,2,2]
]
# 注意先迭代行，comprehension从左至右
a = [number for row in matrix for number in row]
print(a)
```

|                                              |                 |                                                            |
| -------------------------------------------- | --------------- | ---------------------------------------------------------- |
| Expression                                   | Purpose         | Output                                                     |
| [number for row in matrix for number in row] | Flattens matrix | Single list containing all elements in row-major order     |
| [number for number in row for row in matrix] | Likely error    | Error due to attempting to iterate over undefined elements |

### 2. generator

```python
# now python3 can store a huge number only limited to the memory instead of the system architecture, the integer in python3 can grow dynamically, sys.maxsize represent the maxint in python based on the system's architecture
a=sum(number * number for number in range(1_000_000_000))

```
### 3.unpack
```python
print(*range(0,16,8)

```