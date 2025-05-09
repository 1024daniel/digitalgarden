---
{"dg-publish":true,"permalink":"/OperatingSystem/macos/macos配置/","noteIcon":"3"}
---


#macos
1. macos键盘配置
设置page up page down terminal history macth
设置成功之后可以直接在terminal先输入命令前缀之后点击向上arrow就可以调出匹配的历史命令记录
![Pasted image 20230729125944.png](/img/user/pics/Pasted%20image%2020230729125944.png)

#keyboard #hotkey
### 1.terminal行首行尾快捷键设置
mac终端默认到行首是ctrl A，行尾是ctrl E，fn <-和fn ->没有作用(离谱的设计)
如果想要像其他软件一样使用fn加方向键来navigate in terminal，需要设置terminal
这里设置了home(fn <-)和end(fn ->)会发送\\001(ctrl A)和\\005(ctrl E)到终端
https://stackoverflow.com/a/57179327/14637253
![Pasted image 20240130233704.png](/img/user/pics/Pasted%20image%2020240130233704.png)
home: fn + <-
end: fn + ->
### 如果是使用使用terminal登录到linux，
pageup :     shift + fn+ up
pagedown: shift + fn + down
home:         shift + fn + left
end:            shift + fn + right
### 在编辑器内如vscode内
home: command + left
end:    command +right
体验起来macos的键盘还有快捷键确实比win的差好多，对于自带的terminal登录到linux和不登录的快捷键和是不同的，且terminal和editor中的快捷键也不一样？？
3. 截屏快捷键
shift+command+5, ,之后选择截取的是当前window或者指定区域，如果在选择的时候不按住control键，截屏会自动保存在桌面，如果在选择的时候按住control键，截屏会保存到clipboard
5. 快速打开桌面
![Pasted image 20230808004913.png](/img/user/pics/Pasted%20image%2020230808004913.png)

### vscode添加code到系统路径
windows安装vscode会默认将code可执行文件添加到系统路径，但是在mac中需要额外执行命令添加code到系统路径从而实现直接terminal中输入code  path打开制定路径的项目

![Pasted image 20230906130054.png](/img/user/pics/Pasted%20image%2020230906130054.png)

vscode shortcut for mac
[[keyboard-shortcuts-macos.pdf]]
[shortcut](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-macos.pdf)

mac高效率
快捷键
command H: 隐藏当前应用，比minimize更高效
option command H：隐藏其他应用，类似windows的抖动当前window快速隐藏其他windows，需要show all隐藏的应用的话可以点击当前应用左上角标识选择show all
commmand tab: 跳转到其他应用
command Q： 退出当前应用
shift command . :隐藏/显示设置为hidden的文件，文件夹或者app
隐藏文件或者文件夹
```bash
chflags hidden path
chflags nohidden path
```

### vscode设置dash快速查找当前变量或者函数文档
```txt
安装dash插件
选中vscode中文本，
ctrl h(根据当前文件编程语言限制搜索)/ctrl option h(不限制语言)快速跳转

```

### 切换shell
```bash
chsh -s bash
chsh -s zsh
```

### 应用残留文件删除

```bash
# shift command G
~/Library/Application Support/
~/Library/Preferences/
~/Library/Caches/

```

### 安装free工具

#free
```
# 代码仓库和安装教程见如下
https://gitlab.com/parseltongueio/macos-free

```

### 关闭开机启动
#boot #autostart

开机启动有两种，一种是应用程序，一个是系统服务，以zerotier举例，zerotier ui只是一个app，它会和后台系统服务zerotier进行交互，对于禁止zertoier ui只需要spotlight🔍打开login item把它移除就行，对于zertoeir系统服务，需要执行
```bash

sudo launchctl unload /Library/LaunchDaemons/com.zerotier.one.plist
# sudo launchctl load /Library/LaunchDaemons/com.zerotier.one.plist
```
一般对于一些程序如果在login item没法静止，可以再查看一下
1. `/User/user_name/Library/LaunchDaemons`
2. `/User/user_name/Library/LaunchAgents`
3. `/Library/LaunchDaemons`
4. `/Library/LaunchAgents`

### mac查看磁盘总read write数据量
#disk #usage #smartctl
```sh
brew install smartmontools
smartctl -a disk0

```
![Pasted image 20240404140224.png](/img/user/OperatingSystem/macos/attachments/Pasted%20image%2020240404140224.png)




### 查看当前充电功率
#power #
```sh
system_profiler SPPowerDataType | grep Wattage -C 5

```

### 防止mac息屏之后亮度回到default brightness
![Pasted image 20240408104234.png](/img/user/OperatingSystem/macos/attachments/Pasted%20image%2020240408104234.png)


### 截屏
#cut 

```
# 全屏截图
command shift 3
# 指针截图
command shift 4
# full function 截图
command shift 5
```

对于非官方或者未识别developer的工具安装或者使用时出现被拦截的时候可以选择信任使能

### 窗口
- 窗口最小化 `command M`
- 窗口全屏 `fn F`


### 查看packages下载位置
```sh
brew info pybind11
brew --cellar pybind

```

### 编译器支持

https://stackoverflow.com/questions/70168181/apple-clang-13-c20-module-support-missing
https://clang.llvm.org/cxx_status.html


### 安装解压工具

```sh
brew install carlocab/personal/unrar

```

