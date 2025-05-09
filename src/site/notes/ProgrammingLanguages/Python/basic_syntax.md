---
{"dg-publish":true,"permalink":"/ProgrammingLanguages/Python/basic_syntax/","noteIcon":"3"}
---

#format #print
## format
[python format deatils](https://pyformat.info/)
f-stirng
```python

```
1. new style
```python
stat='well'
ret=0
print("the stat is {}, ret code is {}".format(stat,ret))
print("the stat is {0}, ret code is {1}".format(stat,ret))
print("the stat is {stat}, ret code is {ret}".format(stat=stat,ret=ret))

```

2. old style
```python
stat='well'
ret=0
print("the stat is %s, ret code is %d" %(stat,ret))
```

## string
```python
name='George'
print(f"my name is {name}")
```

join string
```python
# Joining with empty separator
list1 = ['g', 'e', 'e', 'k', 's']
print("".join(list1))

# Joining with string

print("-".join(list1))

# Joining with string
list1 = " geeks "
print("$".join(list1))

```