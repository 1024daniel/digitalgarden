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

```cardlink
url: https://stackoverflow.com/a/6561160
title: "Difference between \"git checkout <filename>\" and \"git checkout -​- <filename>\""
description: "http://norbauer.com/notebooks/code/notes/git-revert-reset-a-single-fileI have found a post. But still don't know what is the difference betweengit checkout &lt;filename&gt;git checkout -- &lt;"
host: stackoverflow.com
image: https://cdn.sstatic.net/Sites/stackoverflow/Img/apple-touch-icon@2.png?v=73d79a89bded
```

```bash
git checkout README     # would normally discard uncommitted changes
                        # to the _file_ "README"

git checkout master     # would normally switch the working copy to
                        # the _branch_ "master"

git checkout -- master  # discard uncommitted changes to the _file_ "master"

# 本地checkout到指定仓库的分支，个人仓库同步分支
git checkout pr-flashcommv1 lyh/flashcommv1
git push origin pr-flashcommv1

#acl 
git fecth --tags
git checkout tags/v5.0 #防止和分支 tag有冲突
git checkout -b my-v5.0 v5.0


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

## ignore
忽略所有层级的`__pycache__`文件夹内容以及pyc文件,对应的ignore文件
```sh
**/__pycache__/
*.pyc

```
如果vscode显示对应的文件git还是存在，可能是gitignore文件中添加ignore的文件之前就已经git add进行追踪了，需要主动进行remove from track
```sh

git rm -r --cached **/__pycache__/
git rm -r --cached *.pyc
git commit -m "Remove __pycache__ and .pyc from tracking"

```
## commit

```sh
# 修改上一次的提交信息
git commit --amend
# 修改上一次的提交人信息
git commit --amend --author="new name <>new.email@example.com" --no-edit
## use pre-commit to format the files you add before commit
pre-commit run --file files/you/add


```

## log

```bash
git log --oneline
# 展示前三个commit
git log -3

```

## clone
针对比较大的项目比如linux,ceph或者hf上面的权重，如果直接clone的话，数据量会比较大，主要的数据是源码还有就是分支，commit信息之类的。其中commit信息等元数据占的比重并不少，克隆的时候可以考虑将这些元数据不要进行下载

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
克隆的时候可能存在一些因为网络问题导致的克隆失败，可以尝试设置以下参数:
```sh
git config --global http.postBuffer 2097152000

git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
git clone  --depth 1
git fetch --unshallow

```

```bash
git pull --unshallow
```
对于git lfs下载hf上面权重，如果在执行git lfs install; git clone repo之后出现网络中断或者意外退出，为了断点续传可以执行以下命令
```sh
cd repo
git lfs pull
# 查看lfs文件状态，带*号的表示还未下载完
git lfs ls-files
git reset --hard ## 文件夹大小正确，文件缺失的话可以考虑执行该命令，注意有可能会丢失文件

```
针对特定组织需要指定密钥进行拉取代码
```sh
GIT_SSH_COMMAND="ssh -i ~/.ssh/id_ed25519" git clone <>
# 或者通过指定脚本，这里指定的sh脚本里面就是上面GIT_SSH_COMMAND的内容
GIT_SSH=/Users/daniel/code/ascend/ssh-ascend.sh git clone
```

## pull


```sh

GIT_SSH_COMMAND="ssh -i ~/.ssh/id_ed25519" git pull

```

## proxy
git在windows设置代理,注意<font color="#00b050">这里设置的代理要么不加引号或者加双引号，不能用单引号，单引号会当做代理的字符串的一部分</font>

```bash
git config --global http.proxy http://user:passwd@proxyserverdomain:8080
# 如果对于公司内部分域名不需要代理的，可以针对指定域名来禁用代理的同时不会影响其他的域名访问
git config --global http.https://codehub-g.huawei.com.proxy ""
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --global --unset core.gitproxy
```
<font color="#00b050">对于用户名或者密码存在特殊字符的话需要用URL编码，比如\*需要换成%2A</font>

## remote
```sh
git remote add upstream git@github.com:koalamer/vsc-labeled-bookmarks.git
git remote set-url origin new.example.git
git checkout -b selectable-storage
git reset upstream/selectable-storage --hard
git push origin selectable-storage

git remote remove upstream

```
![Pasted image 20240607212834.png](/img/user/OperatingSystem/Linux/git/attachments/Pasted%20image%2020240607212834.png)

## branch
#branch

```bash
#查看当前和remote有哪些分支
git branch -r
# 查看本地分支和远端分支对应关系
git branch -vv

# 查看包特定commit的本地分支
git branch --contains a1b2c3d4
# 查看包特定commit的remote分支
git branch -r --contains a1b2c3d4
# 查看包特定commit的local和remote分支
git branch -a --contains a1b2c3d4
```

## tag
#tag
```sh
# 打标签
git tag <tagname>
# 附注标签
git tag -a v1.0 -m "Release message"

# 查看当前所有的标签
git tag
#过滤查看指定的标签
git tag -l "v1.0*"
#查看标签详情
git show <tagname>

# 删除本地标签
git tag -d v1.0
# 删除远程标签
git push origin --delete <tagname>

# 推送标签到远程仓库，默认情况标签不会推送
git push origin <tagname>
#推送所有的标签
git push origin --tags

```

## stash
[参考](https://www.atlassian.com/git/tutorials/saving-changes/git-stash)
<svg id="Layer_1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 800 500"><style>.st0{fill:none;stroke:#414141;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st1{fill:#FFFFFF;stroke:#414141;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st2{fill:#A58BC0;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;} .st3{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;} .st4{fill:#000100;} .st5{font-family:&apos;CircularPro-Book&apos;;} .st6{font-size:15px;} .st7{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;} .st8{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:5.773,9.6216;} .st9{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.4378,10.7297;} .st10{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.017,10.0283;} .st11{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.1571,10.2619;} .st12{fill:#FFFFFF;} .st13{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.3663,10.6105;} .st14{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.9923,11.6539;} .st15{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.7154,11.1923;} .st16{fill:#979797;} .st17{fill:#B5E0F7;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;} .st18{fill:#B18BE8;stroke:#404040;stroke-width:4;stroke-miterlimit:10;} .st19{fill:none;stroke:#404040;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;} .st20{fill:#FC8363;stroke:#404040;stroke-width:4;stroke-miterlimit:10;} .st21{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.0329,10.0548;} .st22{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:5.9778,9.963;} .st23{fill:#B18BE8;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st24{fill:#FC8363;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st25{fill:#B5E0F7;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st26{fill:none;stroke:#404040;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;} .st27{fill:#B5E0F7;stroke:#404040;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;} .st28{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;stroke-dasharray:6,10;} .st29{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:6.1001,10.1669;} .st30{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:5.9247,9.8746;} .st31{fill:#59ABDD;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st32{fill:#61C19B;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st33{fill:#61C19B;stroke:#404040;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10;} .st34{fill:none;stroke:#414141;stroke-width:4;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:5.977,9.9616;} .st35{fill:#DADFE2;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;} .st36{fill:none;stroke:#404040;stroke-width:4;stroke-linejoin:round;stroke-miterlimit:10;}</style><path class="st0" d="M30 51.2H770V490H30z"/><path class="st0" d="M50 71.2H523.3V450H50z"/><path class="st0" d="M70 91.2H276.7V400H70z"/><path class="st1" d="M179.7,148.3v35.1H216v77.2h-85.4V148.3H179.7"/><path class="st2" d="M179.7,148.3l36.3,35.2l-36.3-0.1V148.3z"/><path class="st3" d="M166.8,229.2l-15-15.1l14.5-14.5"/><path class="st3" d="M179.9,199.6l15,15.1l-14.5,14.5"/><text transform="translate(141.33 293.98)"><tspan x="0" y="0" class="st4 st5 st6">Tracked </tspan><tspan x="15.1" y="18" class="st4 st5 st6">Files</tspan></text><path class="st7" d="M689.4 183.4L689.4 186.4"/><path class="st8" d="M689.4 196L689.4 252.8"/><path class="st7" d="M689.4 257.6L689.4 260.6 686.4 260.6"/><path class="st9" d="M675.6 260.6L612.3 260.6"/><path class="st7" d="M607 260.6L604 260.6 604 257.6"/><path class="st10" d="M604 247.6L604 156.3"/><path class="st7" d="M604 151.3L604 148.3 607 148.3"/><path class="st11" d="M617.2 148.3L644.9 148.3"/><path class="st7" d="M650.1 148.3L653.1 148.3"/><path class="st12" d="M653.1 148.3L689.4 183.5 653.1 183.4z"/><path class="st7" d="M653.1 151.3L653.1 148.3 655.2 150.4"/><path class="st13" d="M662.8 157.8L683.4 177.7"/><path class="st7" d="M687.2 181.4L689.4 183.5 686.4 183.5"/><path class="st14" d="M674.7 183.4L661.9 183.4"/><path class="st7" d="M656.1 183.4L653.1 183.4 653.1 180.4"/><path class="st15" d="M653.1 169.2L653.1 156.9"/><text transform="translate(359.785 293.98)"><tspan x="0" y="0" class="st4 st5 st6">Untracked </tspan><tspan x="23.3" y="18" class="st4 st5 st6">Files</tspan></text><text transform="translate(618.25 293.98)"><tspan x="0" y="0" class="st4 st5 st6">Ignored</tspan><tspan x="11.5" y="18" class="st4 st5 st6">Files</tspan></text><text transform="translate(193.967 380)" class="st4 st5 st6">git stash</text><text transform="translate(420.73 430)" class="st4 st5 st6">git stash -u</text><text transform="translate(668.04 470)" class="st4 st5 st6">git stash -a</text><text transform="translate(344.983 21.163)" class="st16 st5 st6">git stash options</text><path class="st1" d="M406.4,148.3v35.1h36.3v77.2h-85.4V148.3H406.4"/><path class="st2" d="M406.4,148.3l36.3,35.2l-36.3-0.1V148.3z"/><path class="st3" d="M393.4,229.2l-15-15.1l14.5-14.5"/><path class="st3" d="M406.6,199.6l15,15.1L407,229.2"/></svg>

```bash
# include untracked files
git stash -u
git stash save "stash note"

## stash部分指定文件
git stash push -m "msg" path/to/file1 path/to/file2
git stash push -um 'msg' path/to/newfile

git stash list
## apply the latest stash, and keep the stash
git stash apply
## apply the specific stash and keep it
git stash apply stash@{2}
## apply the lastest stash but drop it
git stash pop
## apply the specific stash but drop it
	git stash pop stash@{3}win`
	
## drop the specific stash
git stash drop stash@{2}
## clear all stashes
git stash clear


git stash show stash@{0}
git stash show -p stash@{0}
git log -p -1 stash@{0}
git diff stash@{0} -- src/main.py

git restore path/to/file

```


## rebase


```bash
## 针对最近4个commit进行交互式rebase修改，
git rebase -i HEAD~4
# merge模式: 当本地和远程都有一个commit没有同步，rebase之后会额外创建一个merge commit
git config pull.rebase false
# rebase模式: 当本地和远程都有一个commit没有同步，rebase之后不会创建merge commit
git config pull.rebase true
# 对于部分社区提交代码的话DCO一般需commit带上邮箱，如果已提交的commit可通过以下代码进行修改
git rebase HEAD~5 --signoff
git push --force-with-lease
# 设置自动commit提交带上email签名
git config commit.gpgsign false  # 确保不要干扰签名
git config format.signoff true   # 默认添加 Signed-off-by


### 如果远程只有一个commit，想要将本地新增的一个commit合入到最初的commit，需要指定root才能获取到全量的commit信息进行rebase
git rebase -i --root

```
![Pasted image 20250207000434.png](/img/user/OperatingSystem/Linux/git/attachments/Pasted%20image%2020250207000434.png)

## reset

```sh
git reset 

```

## push
当前github开始支持PAT替换密码进行安全校验.对于提交代码需要输入用户名，密码需要用PAT
生成PAT参考:
https://github.com/settings/tokens
注意PAT只会明文显示一次，需要自行保存好
如果不想每次输入PAT需要进行设置:
```sh
git config --global credential.helper store


# 把本地 local_feature 推送到远程 new_feature
git push origin local_feature:new_feature

# 如果本地分支是新分支，直接push origin，远程会新建同步分支
git push origin

```


### apply

```sh
git apply diff.patch

git apply -R diff.patch

```

# 案例 
### git账户隔离

比如针对gitee有多个账户，其中某个组织下面的仓库需要指定的账户进行登录才能查看，对于这类组织仓库可以单独维护一个.gitconfig来保持我们提交代码或者是拉取代码都是用的指定的账户信息和对应的密钥
其中一个关键信息是git的用户名和email其实对于认证作用不大，主要作用是提交代码中commit带上相关信息，对于拉取代码提交代码都是需要用到指定密钥或者是输入密码之类的
对于指定组织使用单独的git用户信息和ssh密钥，我们可以通过以文件夹形式进行隔离，在指定文件夹只下载指定组织的仓库代码，配置gitconfig使得该文件夹下面使用单独的认证信息，独立于全局的git设置

如下图所示，在全局gitconfig设置文件中新增10-11行，这里表示在指定文件夹下面递归子文件夹，如果存在git仓库，会将11行指定的.gitconfig文件生效
![Pasted image 20250411093418.png](/img/user/OperatingSystem/Linux/git/attachments/Pasted%20image%2020250411093418.png)

注意这里局部gitconfig中第5行，这里指定密钥的原因在于，由于我们我们可能在本机存在多公钥密钥对，在gitee上面针对不同账户上传了不同的公钥，默认从Terminal登录gitee是从~/.ssh/下面每个密钥进行尝试，比如我们个人账户(全局账户)使用的是rsa密钥，但是在指定组织使用的是ed25519密钥，不指定密钥的话可能以错误的密钥进行登录gitee，导致无法提交或者拉起代码
![Pasted image 20250411094020.png](/img/user/OperatingSystem/Linux/git/attachments/Pasted%20image%2020250411094020.png)

### 本地代码对比

如果想要对比两个文件夹内容更改
```shell
cd /path/to/base
git init
git add .
git commit -m 'base'

git clone /path/to/base /path/to/workspace
cp -rf /path/to/base/* /path/to/workspace/*
```

### 代码提交量

```sh
uname=$1
author_stats=$(git log --author="$uname" --numstat --pretty="%H" \
  | awk 'NF==3 {adds+=$1; subs+=$2} END {print adds,subs}')

coauthor_stats=$(git log --grep="Co-authored-by: $uname" --numstat --pretty="%H" \
  | awk 'NF==3 {adds+=$1; subs+=$2} END {print adds,subs}')

author_add=$(echo $author_stats | cut -d' ' -f1)
author_del=$(echo $author_stats | cut -d' ' -f2)
coauthor_add=$(echo $coauthor_stats | cut -d' ' -f1)
coauthor_del=$(echo $coauthor_stats | cut -d' ' -f2)

echo "Author added: $author_add, deleted: $author_del"
echo "Co-author added: $coauthor_add, deleted: $coauthor_del"
echo "Total added: $((author_add+coauthor_add)), deleted: $((author_del+coauthor_del))"

```


### 修改内容生成patch文件

```sh
# -N会将修改中新增文件这些untract的文件让git文件存在，之后git diff就会看到新增文件内容
git add -N . 
# 这里diff会忽略.gitignore指定的文件，但是会包含除此之外的文件包含二进制文件
git diff HEAD --binary > n8.0.patch

```





