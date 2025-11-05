---
{"dg-publish":true,"permalink":"/AI/concepts/Speculative Decoding/","noteIcon":"3"}
---


Aggresive 大模型前向推理比较消耗算力资源，make every token expenise
投机推理主要是使用小模型(Draft Model)先批量生成k个tokens，耗时为$T_{s}$
之后直接用大模型(Target Model)进行验证是否符合，符合的话直接采纳，如果存在不符合，直接从
	最前面不符合的位置开始自行逐token补充生成，假如在k个tokens中验证有效的前缀个数平均概率为`p`，则能够复用的前缀个数为$p*k$
不过对于验证draft model生产的k个tokens是否符合消耗的时间和前向生成一个token消耗的时间差不多，记为$T_{L}$
则性能提升为
$$
SeepUp =  \frac{T_{L}}{(\frac{T_{s}+T_{L}}{p*k})} =  \frac{p*k*T_{L}}{T_{s}+T_{L}} \approx  {\color{Green}{p*k}}
$$


因为$T_{s} \ll T_{L}$, 所以提升按上述公式推导约等于$p*k$比如概率p为0.75， k为4的话，提升性能约等于3x,提升效果还是比较明显的