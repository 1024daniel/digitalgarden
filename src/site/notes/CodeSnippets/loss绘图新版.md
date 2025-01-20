---
{"dg-publish":true,"permalink":"/CodeSnippets/loss绘图新版/","noteIcon":"3"}
---



```python
from matplotlib import rcParams
import matplotlib.pyplot as plt
import re
import pandas as pd


def collect_loss(file_path):
	text = ''
	with open(file_path, 'r', encoding='utf-8', errors='ignore') as file:
		for line in file:
			text += line
	all_list = re.findall('step_loss=.*[0-9]]', text)
	train_loss = []
	for i in range(0, len(all_list), 2)
		train_loss.append(float(all_list[i].split('step_loss=')[1][:-1]))
	return train_loss

# init pciture with specific dimsion
plt.figure(figuresize=(12,6))
train_loss_npu = collect_loss("path/to/npu_log")
train_loss_gpu = collect_loss("path/to/gpu_log")
plt.plot(train_loss_npu, label = 'npu_loss', alpha = 0.5, color = 'blue')
plt.plot(train_loss_gpu, label = 'gpu_loss', alpha = 0.5, color = 'red')
plt.legend(loc='best')
plt.savefig('./compare.png')

# for the 2nd picture
plt.show()
# clear axes
plt.cla()
# clear figure including axes
plt.clf()
plt.figure(figure=(12,6))

train_diff = []
train_diff_rate = []
for i in range(0, len(train_loss_npu)):
	diff = train_loss_npu[i] - train_loss_gpu[i]
	train_diff.append(diff)
	train_diff_rate.append(abs(diff)/train_loss_gpu[i])
plt.plot(train_diff, label = 'loss_gap')
plt.legend(loc = 'best')
plt.savefig('./loss_gap.png')
plt.show()

# save diff data
data = {
		'train_loss_npu' : train_loss_npu,
		'train_loss_gpu' : train_loss_gpu,
		'train_loss_gap' : train_diff
}
data = pd.DataFrame(data)
data.to_excel(r'./compare.xlsx')


```

新版绘图基于[[CodeSnippets/读取训练loss趋势并且对比绝对差值\|读取训练loss趋势并且对比绝对差值]]，但是<font color="#ffc000">显示图从方方正正的图片改成了矩形更好展示数据，同时绘图时只给y轴数据，这样两组数据就算长度不一样也能正常绘制</font>，这样可以在运行前期就可以对比数据是否异常，获取关键数据没有使用match group而是使用findall来直接找子字符串

- 指定figsize为2:1来使得数据更全面的展示
- 这里使用re.findall函数来提取每一行匹配的字符串，最后用split来将数字部分拆分出来
- alpha = 0.5 for transparency
- legend函数中给定loc = 'best'来让绘图工具自动调整最佳位置展示数据