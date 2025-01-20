---
{"dg-publish":true,"permalink":"/CodeSnippets/读取训练loss趋势并且对比绝对差值/","noteIcon":"3"}
---

#python #re #numpy #matplotlib #ai

```python
import re
import matplotlib.pyplot as plt
import numpy as np
def read_loss(file_path):
	str1 = "'loss': "
	str2 = ", 'learning_rate'"
	numbers = []
	pattern = r'[+-]?(\d+(\.\d*)?|\.\d+)([eE][+-]?\d+)?' #匹配浮点数
	with open(file_path, 'r', encoding='utf-8', errors='ignore') as file:
		for line in file:
			match = re.search(f'{str1}({pattern}){str2}', line)
			if match:
				numbers.append(float(match.group(1)))
	return numbers
def read_epoch(file_path)
	str1 = "'epoch': "
	str2 = "}"
	numbers = []
	pattern = r'[+-]?(\d+(\.\d*)?|\.\d+)([eE][+-]?\d+)?' #匹配浮点数
	with open(file_path, 'r', encoding='utf-8', errors='ignore') as file:
		for line in file:
			match = re.search(f'{str1}({pattern}){str2}', line)
			if match:
				numbers.append(float(match.group(1)))
	return numbers
def av_time(file_path)
	str1 = "'time cost': "
	numbers = []
	pattern = r'[+-]?(\d+(\.\d*)?|\.\d+)([eE][+-]?\d+)?' #匹配浮点数
	with open(file_path, 'r', encoding='utf-8', errors='ignore') as file:
		for line in file:
			match = re.search(f'{str1}({pattern})', line)
			if match:
				numbers.append(float(match.group(1)))
	nums =  np.array(numbers)
	return nums.mean(nums)
loss_value_a800 = read_loss(.\\train_7B_a800.log)
loss_value_npu = read_loss(.\\train_7B_npu.log)
epoch = read_epoch(.\\train_7B_a800.log)
epoch.pop() #最后一个是汇总，去除以保持数量一致

# first picture
plt.figure()
plt.plot(epoch,loss_value_a800,color='b',label='A800')
plt.plot(epoch,loss_value_npu,color='g',label='npu')
plt.title('Loss Trend')
plt.xlabel('Epochs')
plt.ylabel('Loss')

# second picture
plt.figure()
a800 = np.array(loss_value_a800)
npu = np.array(loss_value_npu)
lossDiff = np.abs(a800-npu)
plt.plot(epoch,lossDiff,color='r',label='diff')
plt.title('Absolute Loss Diff Trend')
plt.xlabel('Epochs')
plt.ylabel('Absolute Loss Diff')
plt.ylim(top=0.1)

# some addition info
a800_t = av_time(.\\train_7B_a800.log)
npu_t = av_time(.\\train_7B_npu.log)
print(f'a800 mean step time: {a800_t}')
print(f'npu mean step time: {npu_t}')
proportion = np.sum(lossDiff<0.001)/len(lossDiff)
print(f"小于0.001的比例: {proportion}")

plt.show()
```

- 这里的match.group(1)的参数1表示正则表达式匹配的第一个子组，一个小括号代表一个子组, 也称为capture group
- 这里使用open打开文件的时候指定了文件编码，防止一些由于错误解码导致的错误，这里还指定了忽略解析文件的错误error='ignore'主要是防止文件内部出现给定编码之外的特殊字符而导致错误退出，如表示进度的符号
- plt绘图时候制定了ylim(top=0.1)来制定y轴的最大值来约束绘图效果图
- 使用numpy来进行一些对于数组的高效操作，比如计算数组平均值，数组内小于某一个值的比例等等