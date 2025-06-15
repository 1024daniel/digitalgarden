---
{"dg-publish":true,"permalink":"/OperatingSystem/macos/Automator用法/","noteIcon":"3"}
---


#automator
Automator 是 macOS 内置的一款强大的自动化工具，它能帮助你无需编写代码即可创建各种自动化流程，从而大大提高工作效率。简单来说，你可以把它看作是一个“拖放式”的编程工具。
这里使用几个简单的例子来讲述Automator的使用，同时也展现几个能提高Mac工作效率的场景

### 1. 为应用创建别名,更快得使用spotlight打开
抖音.app使用spotlight打开的话需要输入中文抖音才能触发，比较低效，如果想通过直接输入dy来索引到抖音.app,可以通过Automator来进行快速操作
首先打开Automator点击任务类别为Application
![Pasted image 20250615112937.png](/img/user/OperatingSystem/macos/attachments/Pasted%20image%2020250615112937.png)

之后在搜索框搜索script，这里直接选择shell脚本，将其拖入到右侧，之后在右侧可以编辑脚本内容，这里直接键入`open -a 抖音.app`
![Pasted image 20250615113049.png](/img/user/OperatingSystem/macos/attachments/Pasted%20image%2020250615113049.png)
之后直接`Command S`保存到Application文件夹中

### 2. Finder增加选项快速打开当前folder的Terminal

Finder如果打开一个folder之后打开terminal到当前folder路径，首先需要finder先设置好默认展示当前Path，之后才能在folder下方看到当前的路径位置，之后右键选择对应的Path来选择Open in Terminal 
![Pasted image 20250615113357.png](/img/user/OperatingSystem/macos/attachments/Pasted%20image%2020250615113357.png)
这种效率比较低，通过Automator可以在Finder的工具栏加上相关操作按钮直接单击完成任务
首先还是先打开automator之后选择application之后搜索选中Run AppleScript，拖入右边，键入
```sh
on run {input, parameters}
    tell application "Finder"
        set targetFolder to (insertion location as alias)
    end tell

    tell application "Terminal"
        activate
        do script "cd " & quoted form of POSIX path of targetFolder
    end tell
end run

```
![Pasted image 20250615113745.png](/img/user/OperatingSystem/macos/attachments/Pasted%20image%2020250615113745.png)
直接保存到application命名为OpenTerminal
默认新建的application的图标和Automator一样的，这里我们可以替换这个新建的OpenTerminal的图标和Terminal图标保持一致，这样更加和他的功能保持一致
复制图标的话只需要在Finder先找到Terminal.app，之后右键选择Get Info，之后鼠标单击选择左上角的图标，可以看到有蓝色表示选中了，之后直接command C复制之后同样找到我们需要更改图标的OpenTerminal.app明知后同样操作选中图标之后command V粘贴，这样就可以完成图标替换
![Pasted image 20250615114119.png](/img/user/OperatingSystem/macos/attachments/Pasted%20image%2020250615114119.png)

之后可以把这个新建的app导入到Finder的工具栏，这样可以完成单击打开当前Folder的Terminal
导入工具栏需要我们在finder先找到OpenTerminal.app，之后先单击选中app，之后按照command的同时将应用拖入到Finder的右上角的工具栏区域完成添加
![Pasted image 20250615114624.png](/img/user/OperatingSystem/macos/attachments/Pasted%20image%2020250615114624.png)
