---
{"dg-publish":true,"permalink":"/OperatingSystem/windows/wsl/","noteIcon":"3"}
---

#wsl #windows
在不了解wsl的时候我感觉既然windows已经有了强大的hyperv，可以有多个亲和性很好的虚拟机，完全不需要这种四不像的wsl了，但是在接触了wsl之后才发现我错了，因为wsl的使能相当于使得windows的最后一个短板完全解决了，就是wsl的bash可以让熟悉linux命令行操作还有一系列的强大的packages开发者直接在windows原生filesystem中使用，可以说有了wsl的windows没有任一项落于macos。
### 1.安装wsl和ubuntu lts

![Pasted image 20231217002217.png](/img/user/pics/Pasted%20image%2020231217002217.png)


### 2. 打开安装virtual machine platform的feature
![Pasted image 20231217002433.png](/img/user/pics/Pasted%20image%2020231217002433.png)

### 3. 使用wsl 的bash
直接win R输入bash，cmd或者powershell进入之后在直接输出bash或者直接在资源管理器上方的路径直接输入bash回车
### 4. 继承使用windows的ssh密钥和配置来直接使用ssh还有git
安装wsl-ubuntu之后ubuntu中的/etc/hosts会继承windows的hosts文件，所以我们可以直接将windows中的的ssh的config文件拷贝到ubuntu的.ssh文件夹
```
cp /mnt/c/User/daniel/.ssh/config  ~/.ssh/
```

拷贝完之后注意将windows路径转换成ubuntu所见的路径，
使用软链接形式复用windows本地的ssh key来往github提交代码和登录之前和windows互信免密的机器
![Pasted image 20231217022334.png](/img/user/pics/Pasted%20image%2020231217022334.png)
同时还需要修改ubuntu所见的权限,.ssh文件夹内文件设置600，直接chmod没法修改ubuntu所见windows上的文件权限的话需要进行一下设置之后重新打开bash，如果还是没有生效的话直接重启机器:https://superuser.com/questions/1323645/unable-to-change-file-permissions-on-ubuntu-bash-for-windows-10
![Pasted image 20231217012328.png|100%](/img/user/pics/Pasted%20image%2020231217012328.png)

复用ssh配置之后直接push到github之后发现一直要求username和password，查阅资料发现wsl中当前仓库的配置和windows不一样，虽然使用同一套密钥，windows能直接push，我们需要在wsl设置一下认证程序:https://stackoverflow.com/questions/66503781/git-asks-for-password-to-push-on-wsl-2-ubuntu-but-not-on-windows-why-is-that
```
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/libexec/git-core/git-credential-manager.exe"
```
### 5.更改wsl-ubuntu所在的磁盘从C盘到D盘，防止系统盘空间不够
默认所在的位置:/mnt/c/Users/daniel/AppData/Local/Packages/CanonicalGroupLimited.Ubuntu22.04LTS_79rhkp1fndgsc
可以看到wsl也是使用将linux distro系统map到本地文件的vhdk文件，类似virtual box还有hyperv的方式
![Pasted image 20231217013440.png|100%](/img/user/pics/Pasted%20image%2020231217013440.png)
https://blog.csdn.net/x356982611/article/details/108641601
references:https://github.com/microsoft/WSL/issues/10623