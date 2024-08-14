---
{"dg-publish":true,"permalink":"/AI/torch/embedding/","noteIcon":"3"}
---

#embedding

https://blog.csdn.net/qq_43391414/article/details/120783887

https://qiankunli.github.io/2022/03/02/embedding.html

https://allenwind.github.io/blog/8912/

https://stackoverflow.com/questions/55276504/different-methods-for-initializing-embedding-layer-weights-in-pytorch

```py title:embedding指定初始化weight
import torch
import torch.nn as nn

torch.manual_seed(3)
emb1 = nn.Embedding(5,5)
emb1.weight.data.uniform_(-1, 1)

torch.manual_seed(3)
emb2 = nn.Embedding(5,5)
nn.init.uniform_(emb2.weight, -1.0, 1.0)

assert torch.sum(torch.abs(emb1.weight.data - emb2.weight.data)).numpy() == 0

```
