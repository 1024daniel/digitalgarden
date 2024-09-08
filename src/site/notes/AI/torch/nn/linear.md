---
{"dg-publish":true,"permalink":"/AI/torch/nn/linear/","noteIcon":"3"}
---

#linear

https://pytorch.org/docs/stable/generated/torch.nn.Linear.html

Applies an affine linear transformation to the incoming data: 
$$
y=xA^{T} + b
$$
其中A和b分别是Linear中的weight和bias，这两个值会根据定义linear的in_features和out_features时确定shape，之后随机初始化，当然后也可以指定初始化算法，之后随模型训练更新weight

```py
import torch
from torch import nn
m = nn.Linear(3,4)
print(m.weight)
print(m.bias)

input = torch.randn(3,3)
output1 = m(input)

output2 = torch.matmul(input, m.weight.t()) + m.bias

assert output1 == output2

```