---
{"dg-publish":true,"permalink":"/AI/torch/","noteIcon":"3"}
---

#torch #tensor
tensor
### 1. tensor save and load
```bash
model = torch.load('model.pth', map_location=torch.device('cpu'))
model = torch.load('model.pth', map_location='cpu')
model = torch.load('model.pth', map_location='npu:0')
# 将存储在 GPU 0 上的数据加载到 CPU。
model = torch.load('model.pth', map_location={'cuda:0': 'cpu'})
# model 将被加载到 CPU 上
model.to(torch.device('cuda:0'))
# model 将被移动到 GPU 0 上
```


### 1. tensor num_groups num_channels
num_groups，第一个维度数
num_channels，第2个维度数


![Pasted image 20231228201032.png](/img/user/pics/Pasted%20image%2020231228201032.png)