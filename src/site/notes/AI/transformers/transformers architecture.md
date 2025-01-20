---
{"dg-publish":true,"permalink":"/AI/transformers/transformers architecture/","noteIcon":"3"}
---

#transformer
![Pasted image 20241124144006.png](/img/user/AI/transformers/attachments/Pasted%20image%2020241124144006.png)

https://arxiv.org/pdf/1706.03762

https://zhuanlan.zhihu.com/p/82312421

[[AI/transformers/Autoregressive Models\|Autoregressive Models]]

![Pasted image 20241124140104.png](/img/user/AI/transformers/attachments/Pasted%20image%2020241124140104.png)

https://www.youtube.com/watch?v=wjZofJX0v4M



<div style="position: relative; width: 100%; max-width: 800px; height: 0; padding-bottom: 56.25%; margin: 0 auto;">
    <iframe 
        src="https://www.youtube.com/embed/wjZofJX0v4M?rel=0&modestbranding=1&autoplay=0&showinfo=0&fs=1&disablekb=1" 
        style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: none;" 
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
        allowfullscreen>
    </iframe>
</div>




![Pasted image 20250119001634.png](/img/user/AI/transformers/attachments/Pasted%20image%2020250119001634.png)


### 1. embedding层
[[AI/torch/nn/embedding\|embedding]]
主要将词表表达为vector并映射到高维词表空间，对于英文来说有大概5k+左右的向量，除了表示token的含义之外，还通过位置编码将token在context的位置也包含了进去，将位置编码进去的主要目的是为了可以并行计算不会像RNN网络一样存在前后依赖，可以最大化并行利用好gpu的效率
### 2. attention 层
### 3. feed forward layer