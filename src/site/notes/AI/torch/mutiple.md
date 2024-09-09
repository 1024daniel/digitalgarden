---
{"dg-publish":true,"permalink":"/AI/torch/mutiple/","noteIcon":"3"}
---

https://blog.csdn.net/da_kao_la/article/details/87484403

#matmul
tensor有四种乘法
### 1. `*`
对于`a = b * c`来说，如果b和c的size不一样，则会以某种方式，比如`expand`将两者`shape`对齐进行`element-wise`的简单乘法
```py
import torch
b = torch.ones(3,4)
c = 2
print(b * c)

# 行向量expand
c = torch.tensor([1,2,3,4])
print(b * c)

# 列向量expand
c = torch.tensor([1,2,3]).reshape((3,1))
print(b * c)

# 都是矩阵的话要求两者shape一致
b = torch.tensor([[1,2],[2,3]])
print(b * b)
```

### 2. `torch.mul`
`torch.mul`和`*`的用法相同，也是element-wise的，支持broadcasting[[AI/torch/semantics/broadcasting semantics\|broadcasting semantics]]

```py
a = torch.ones(3,4)
b = torch.tensor([1,2,3]).reshape((3,1))

c = torch.mul(a,b)
print(c)

```

### 3. `torch.mm`

终于到了我们数学上的矩阵乘法了，这里限制两个tensor的shape需要满足矩阵乘法的要求
```py

a = torch.ones(3,4)
b = torch.ones(4,3)
c = torch.mm(a, b)
print(c)

```
### 4. `torch.matmul`
`torch.matmul`为`torch.mm`的支持broadcast版本

```py
a = torch.ones(3,4)
# broadcast从trailing dimension对齐，这里b的后两维可以和a做矩阵乘
b = torch.ones(5,4,2)

c = torch.matmul(a,b)
print(c)

```