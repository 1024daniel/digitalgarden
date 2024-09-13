---
{"dg-publish":true,"permalink":"/Applications/VSCODE/","noteIcon":"3"}
---


## PLUGINS:
1. better comment
![Pasted image 20230228125744.png](/img/user/Applications/Pasted%20image%2020230228125744.png)
2. Material Icon Theme 

## style
设置使用谷歌style

![Pasted image 20230728211157.png](/img/user/pics/Pasted%20image%2020230728211157.png)

## vscode-leetcode设置做题代码文[件的相对路径
![Pasted image 20230728211340.png](/img/user/pics/Pasted%20image%2020230728211340.png)
![Pasted image 20230728211421.png](/img/user/pics/Pasted%20image%2020230728211421.png)
在这里我们根据问题的tag，难易度还有语言进行嵌套放置我们的代码文件
[参考](https://github.com/LeetCode-OpenSource/vscode-leetcode/wiki/Customize-the-Relative-Folder-and-the-File-Name-of-the-Problem-File)

## firacode字体设置
[参考](https://github.com/tonsky/FiraCode/wiki/Installing)
![Pasted image 20230728212423.png](/img/user/pics/Pasted%20image%2020230728212423.png)
## 设置autosave
![Pasted image 20230728212036.png](/img/user/pics/Pasted%20image%2020230728212036.png)
## coderunner设置输出在terminal中而非output中
![Pasted image 20230728231602.png](/img/user/pics/Pasted%20image%2020230728231602.png)

## 制定g++编译使用c++17标准，m1 默认是c98标准
![Pasted image 20230728234451.png](/img/user/pics/Pasted%20image%2020230728234451.png)
如果没有生效，可以直接编辑code runner的json配置文件
![Pasted image 20230730090226.png](/img/user/pics/Pasted%20image%2020230730090226.png)

![Pasted image 20230730090306.png](/img/user/pics/Pasted%20image%2020230730090306.png)


### remote-ssh
如果需要远程连接server进行编程需要保证远程主机的sshd_config设置中这两个转发设置
```
AllowTcpForwarding yes
AllowAgentForwarding yes
```
### user code snippet
命令面板输入snippet来打开Configure User Snippet
使用网页服务快速定制code snippet来直接粘贴到vscode的snippet文件
https://snippet-generator.app/?description=&tabtrigger=&snippet=&mode=vscode
![Pasted image 20240202001714.png](/img/user/pics/Pasted%20image%2020240202001714.png)
word wrap on

highlight
	


```
code --disable-extensions
```