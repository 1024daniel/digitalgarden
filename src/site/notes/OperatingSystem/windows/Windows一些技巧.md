---
{"dg-publish":true,"permalink":"/OperatingSystem/windows/Windows一些技巧/","noteIcon":"3"}
---

### 1. 快速跳转多个tab
针对跳转，可以直接全程使用三指来快速跳转当前desktop打开的应用tab，可以从多个方向跳转
![Pasted image 20231004224402.png](/img/user/pics/Pasted%20image%2020231004224402.png)
<font color="#ffc000">alt tab</font>快捷键可以实现类似的效果，但是效率不高，因为只能依次从左往右跳转
<font color="#92d050">win tab</font>可以打开当前所有desktop以及对应的应用tab，然后单指选择跳转tab
<font color="#d99694">ctl tab</font>可以针对当前应用内打开的tabs进行跳转
<font color="#c4bd97">ctl win</font> ->/<-跳转desktop
### 2.  拷贝文件到剪切板
```bash
#在需要拷贝文件所在的文件夹右键打开git
cat file | clip
```
#alias
### 3. 设置alias
```powershell
#获取系统的profile文件
echo $PROFILE
#没有该文件的话直接创建，在该文件加入如下格式
function kctl {
	D:/k8s/kubectl.exe -n kubemaker --kubeconfig D:/k8s/config
}
#保存文件之后关闭打开的所有powershell terminals然后打开powershell terminal验证是否生效
kctl get pods

```

### 4. 查看设置开机启动的程序
```
shell:startup
```
### 5. windows11右键默认展开所有选项

```powershell
reg.exe add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve 
taskkill /f /im explorer.exe & start explorer.exe
```
