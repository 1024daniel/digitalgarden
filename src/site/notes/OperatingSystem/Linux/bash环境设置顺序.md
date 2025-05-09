---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/bash环境设置顺序/","noteIcon":"3"}
---

#bash #profile #bashrc
在 Linux 系统中，当用户登录时，会依次执行以下配置文件：

1. `/etc/profile`：此文件为系统的每个用户设置环境信息，当用户第一次登录时，该文件被执行。
2. `/etc/bashrc`：为每一个运行 bash shell 的用户执行此文件。
3. `~/.bash_profile`：每个用户都可使用该文件输入专用于自己使用的 shell 信息，当用户登录时，该文件仅仅执行一次!
4. `~/.bashrc`：每次用户打开一个新的交互式 shell 时都会执行此文件。
5. `~/.bash_login`：如果 `~/.bash_profile` 不存在，则执行此文件。

**执行顺序图：**

```
                                   +--------------+
                                   |  /etc/profile  |
                                   +--------------+
                                       / \
                                      /   \
                                   +-------+     +--------------+
                                   |  bashrc  |     |  .bash_profile  |
                                   +-------+     +--------------+
                                       / \             / \
                                      /   \           /   \
                                  +---------+     +-------+     +---------+
                                  | .bash_login |     | .bashrc |     | .profile |
                                  +---------+     +-------+     +---------+
```

**注意：**

* 如果 `~/.bash_profile` 文件存在，则会执行 `~/.bashrc` 文件。
* 如果 `~/.bash_profile` 文件不存在，则会执行 `~/.bash_login` 文件。
* `~/.profile` 文件仅在用户第一次登录时执行。

**以下是一些常见的配置文件设置：**

* 设置环境变量，例如 PATH、LANG、EDITOR 等。
* 设置别名，例如 ls -al 等。
* 设置函数，例如 history、cd 等。

**您可以根据自己的需要修改这些配置文件。**

设置Linux默认的交互式终端使用bash而非sh
在~/.profile文件最后加上
```sh
exec /bin/bash

```

**参考资料：**

* Linux bashrc profile等一系列文件的执行顺序: [https://www.cnblogs.com/lizm166/p/10662371.html](https://www.cnblogs.com/lizm166/p/10662371.html)
* Linux 之 /etc/profile、~/.bash_profile 等几个文件的执行过程: [https://cloud.tencent.com/developer/article/1114357](https://cloud.tencent.com/developer/article/1114357)