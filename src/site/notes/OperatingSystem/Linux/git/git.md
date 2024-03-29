---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/git/git/","noteIcon":"3"}
---

#git
## 校验
#ssl
```bash
git config --global http.sslVerify "false"
gitc clone xxx.git
```

或者
```bash
git -c http.sslVerify=false clone xxx.git
```

## checkout
#checkout
https://stackoverflow.com/a/6561160
```bash
git checkout README     # would normally discard uncommitted changes
                        # to the _file_ "README"

git checkout master     # would normally switch the working copy to
                        # the _branch_ "master"

git checkout -- master  # discard uncommitted changes to the _file_ "master"

```


## submodule
1. add submodules
`git submodule add giturl path-to-existedfolder/your-favor-repository-name`
2. update submodule

```bash
git submodule init 
git submodule update --recursive
```
对于一个包含子模块的仓库，如果在本地子模块和本仓库都进行了修改，需要进行以下操作进行同步子模块的本仓库

```bash
cd path/to/submodule
git add -A
git commit -m "submodule commit"
git push
cd path/to/base
git add -A
git add path/to/submodule # 更新base索引的子模块的最新commit
git commit -m "base commit and update submodule"
git push

```

## 日志

```bash
git log --oneline

```

## 克隆
针对比较大的项目比如linux和ceph，如果直接clone的话，数据量会比较大，主要的数据是源码还有就是分支，commit信息之类的。其中commit信息等元数据占的比重并不少，克隆的时候可以考虑将这些元数据不要进行下载

```bash
git clone --depth 1 --branch v4.10-rc4  \
git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git linux-4.10-rc4
```

git默认最后走的是系统代理，我们也可以指定代理

```bash
git config --global http.proxy proxyurl
git config --global https.proxy proxyurl

```
clone解决了代理的问题也有可能遇到ssl认证相关的问题，我们可以通过设置不进行ssl校验

```bash
git config --global http.sslVerify false
git clone repourl
# or
git config -c http.sslVerify=false clone repourl
# or
GIT_SSL_NO_VERIFY=true git clone repourl

```


```bash
git pull --unshallow
```

## 代理
git在windows设置代理,注意<font color="#00b050">这里设置的代理要么不加引号或者加双引号，不能用单引号，单引号会当做代理的字符串的一部分</font>

```bash
git config --global http.proxy http://user:passwd@proxyserverdomain:8080
```
<font color="#00b050">对于用户名或者密码存在特殊字符的话需要用URL编码，比如\*需要换成%2A</font>

## 分支

```bash
#查看当前和remote有哪些分支
git branch -r

```

## stash
[参考](https://www.atlassian.com/git/tutorials/saving-changes/git-stash)
<svg id="Layer_1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 800 500"><style>.st0{fill:none;stroke:#414141;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st1{fill:#FFFFFF;stroke:#414141;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st2{fill:#A58BC0;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;} .st3{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;} .st4{fill:#000100;} .st5{font-family:&apos;CircularPro-Book&apos;;} .st6{font-size:15px;} .st7{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;} .st8{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:5.773,9.6216;} .st9{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.4378,10.7297;} .st10{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.017,10.0283;} .st11{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.1571,10.2619;} .st12{fill:#FFFFFF;} .st13{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.3663,10.6105;} .st14{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.9923,11.6539;} .st15{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.7154,11.1923;} .st16{fill:#979797;} .st17{fill:#B5E0F7;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;} .st18{fill:#B18BE8;stroke:#404040;stroke-width:4;stroke-miterlimit:10;} .st19{fill:none;stroke:#404040;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;} .st20{fill:#FC8363;stroke:#404040;stroke-width:4;stroke-miterlimit:10;} .st21{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.0329,10.0548;} .st22{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:5.9778,9.963;} .st23{fill:#B18BE8;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st24{fill:#FC8363;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st25{fill:#B5E0F7;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st26{fill:none;stroke:#404040;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;} .st27{fill:#B5E0F7;stroke:#404040;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;} .st28{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;stroke-dasharray:6,10;} .st29{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.1001,10.1669;} .st30{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:5.9247,9.8746;} .st31{fill:#59ABDD;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st32{fill:#61C19B;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st33{fill:#61C19B;stroke:#404040;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;} .st34{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:5.977,9.9616;} .st35{fill:#DADFE2;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st36{fill:none;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;}</style><path class="st0" d="M30 51.2H770V490H30z"/><path class="st0" d="M50 71.2H523.3V450H50z"/><path class="st0" d="M70 91.2H276.7V400H70z"/><path class="st1" d="M179.7,148.3v35.1H216v77.2h-85.4V148.3H179.7"/><path class="st2" d="M179.7,148.3l36.3,35.2l-36.3-0.1V148.3z"/><path class="st3" d="M166.8,229.2l-15-15.1l14.5-14.5"/><path class="st3" d="M179.9,199.6l15,15.1l-14.5,14.5"/><text transform="translate(141.33 293.98)"><tspan x="0" y="0" class="st4 st5 st6">Tracked </tspan><tspan x="15.1" y="18" class="st4 st5 st6">Files</tspan></text><path class="st7" d="M689.4 183.4L689.4 186.4"/><path class="st8" d="M689.4 196L689.4 252.8"/><path class="st7" d="M689.4 257.6L689.4 260.6 686.4 260.6"/><path class="st9" d="M675.6 260.6L612.3 260.6"/><path class="st7" d="M607 260.6L604 260.6 604 257.6"/><path class="st10" d="M604 247.6L604 156.3"/><path class="st7" d="M604 151.3L604 148.3 607 148.3"/><path class="st11" d="M617.2 148.3L644.9 148.3"/><path class="st7" d="M650.1 148.3L653.1 148.3"/><path class="st12" d="M653.1 148.3L689.4 183.5 653.1 183.4z"/><path class="st7" d="M653.1 151.3L653.1 148.3 655.2 150.4"/><path class="st13" d="M662.8 157.8L683.4 177.7"/><path class="st7" d="M687.2 181.4L689.4 183.5 686.4 183.5"/><path class="st14" d="M674.7 183.4L661.9 183.4"/><path class="st7" d="M656.1 183.4L653.1 183.4 653.1 180.4"/><path class="st15" d="M653.1 169.2L653.1 156.9"/><text transform="translate(359.785 293.98)"><tspan x="0" y="0" class="st4 st5 st6">Untracked </tspan><tspan x="23.3" y="18" class="st4 st5 st6">Files</tspan></text><text transform="translate(618.25 293.98)"><tspan x="0" y="0" class="st4 st5 st6">Ignored</tspan><tspan x="11.5" y="18" class="st4 st5 st6">Files</tspan></text><text transform="translate(193.967 380)" class="st4 st5 st6">git stash</text><text transform="translate(420.73 430)" class="st4 st5 st6">git stash -u</text><text transform="translate(668.04 470)" class="st4 st5 st6">git stash -a</text><text transform="translate(344.983 21.163)" class="st16 st5 st6">git stash options</text><path class="st1" d="M406.4,148.3v35.1h36.3v77.2h-85.4V148.3H406.4"/><path class="st2" d="M406.4,148.3l36.3,35.2l-36.3-0.1V148.3z"/><path class="st3" d="M393.4,229.2l-15-15.1l14.5-14.5"/><path class="st3" d="M406.6,199.6l15,15.1L407,229.2"/></svg>

```bash
git stash -u
git stash save "stash note"
git stash list
## apply the latest stash, and keep the stash
git stash apply
## apply the specific stash and keep it
git stash apply stash@{2}
## apply the lastest stash but drop it
git stash pop
## apply the specific stash but drop it
git stash pop stash@{3}
## drop the specific stash
git stash drop stash@{2}
## clear all stashes
git stash clear

```


## rebase


```bash
## 针对最近4个commit进行交互式rebase修改，
git rebase -i HEAD~4

```

