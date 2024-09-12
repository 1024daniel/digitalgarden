---
{"dg-publish":true,"permalink":"/AI/torch/embedding/","noteIcon":"3"}
---

#embedding

https://blog.csdn.net/qq_43391414/article/details/120783887

https://qiankunli.github.io/2022/03/02/embedding.html

https://allenwind.github.io/blog/8912/

https://stackoverflow.com/questions/55276504/different-methods-for-initializing-embedding-layer-weights-in-pytorch
embedding相当于一个简单的查找表，内容存储在属性weight中，该属性可以通过指定算法进行初始化，模型也可以训练更新weight, 
<font color="#f79646">embeddding的weight是一个torch.nn.Parameter类型，该类型是Tensor的子类，但具有torch.nn.Module的属性，这种类型的参数会参与模型梯度计算，在调用optimizer.step()的时候会自动更新</font>
embedding的输入的每个数字相当于从weight中的具体的index，一个数字取出来的维度为(1, embedding_dim)，其中embedding_dim就是每个embedding vector的长度

![Pasted image 20240912121756.png](/img/user/AI/torch/attachments/Pasted%20image%2020240912121756.png)

```py title:embedding指定初始化weight
import torch
import torch.nn as nn

# 直接使用tensor的uniform方法
torch.manual_seed(3)
emb1 = nn.Embedding(5,5)
emb1.weight.data.uniform_(-1, 1)

# 使用nn的uniformer方法
torch.manual_seed(3)
emb2 = nn.Embedding(5,5)
nn.init.uniform_(emb2.weight, -1.0, 1.0)

# 两个初始化权重方法等价
assert torch.sum(torch.abs(emb1.weight.data - emb2.weight.data)).numpy() == 0

```
