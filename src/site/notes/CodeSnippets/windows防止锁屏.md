---
{"dg-publish":true,"permalink":"/CodeSnippets/windows防止锁屏/","noteIcon":"3"}
---

#windows #vbs
windows设置从不关闭，从不休眠还是会锁屏导致一些需要连夜跑的进程异常
这里收集到一个脚本模仿隔段时间按NUMLOCK来防止锁屏
将以下脚本加入到文本保存更改后缀名为vbs，双击之后进程可在任务管理中搜索wscript.exe进程
需要结束进程的话直接杀死wscript.exe进程即可

```vb {3,10-11}
' 先定义一个Shell对象
Set vbObj = WScript.CreateObject("WScript.Shell")
WScript.Echo "buxiuxi"
WScript.Sleep 5000
vbObj.SendKeys "{NUMLOCK}"
WScript.Sleep 500
vbObj.SendKeys "{NUMLOCK}"
for i=1 to 5
' 一个迭代1h
    WScript.Sleep 60 * 60 * 1000
    vbObj.SendKeys "{NUMLOCK}"
    WScript.Sleep 500
    vbObj.SendKeys "{NUMLOCK}"
next

```

vbs脚本直接双击或者在cmd中`cscript   test.vbs`运行