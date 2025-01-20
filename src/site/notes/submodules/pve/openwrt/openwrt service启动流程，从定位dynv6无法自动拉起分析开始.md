---
{"dg-publish":true,"permalink":"/submodules/pve/openwrt/openwrt service启动流程，从定位dynv6无法自动拉起分析开始/","noteIcon":"3"}
---


```bash
ifstatus wan6 | jsonfilter -e '@.l3_device' 2>/dev/null

```

在 OpenWrt 中，**service 脚本** 负责控制服务的启动和停止。脚本中包含一个名为 **start** 的数值，它指示服务的启动优先级。

**start 数值的含义:**

* **数值越小，优先级越高**。这意味着服务将在系统启动过程中更早启动。
* **数值越大，优先级越低**。意味着服务将在系统启动过程中稍后启动。

**start 数值的常见取值范围:**

* **10-99**: 通常用于系统核心服务，例如网络、日志记录等。
* **100-199**: 通常用于非核心服务，例如 SSH、Web 服务器等。
* **200-299**: 通常用于用户自定义服务。

**设置 start 数值的注意事项:**

* 确保服务的 start 数值与其他服务的 start 数值不冲突。
* 优先启动依赖于其他服务的服务。
* 避免为非必要服务设置过高的优先级。

**以下是一些示例:**

* **sshd** 服务的 start 数值通常为 **20**，因为它是一个重要的服务，需要在系统启动时尽早启动。
* **httpd** 服务的 start 数值通常为 **40**，因为它不是一个核心服务，可以稍后启动。
* **my-custom-service** 服务的 start 数值可以设置为 **250**，因为它是一个用户自定义服务。

**总结:**

* start 数值是 OpenWrt service 脚本中一个重要的参数，用于控制服务的启动顺序。
* 了解 start 数值的含义和注意事项可以帮助您更好地管理 OpenWrt 服务。


在 OpenWrt 系统中，`/etc/rc.d` 目录下存放着各种服务的启动脚本。这些脚本的文件名以 **S** 或 **K** 开头，分别表示服务的 **启动** 和 **停止**。

**S 和 K 的含义:**

* **S**: 表示 **Start**，即服务的启动脚本。
* **K**: 表示 **Kill**，即服务的停止脚本。

**文件名格式:**

* S 或 K 后面跟着一个数字，数字越小，服务的启动或停止优先级越高。
* 数字后面通常是服务的名称，例如 `S10sshd` 表示 SSH 服务的启动脚本，`K20httpd` 表示 HTTP 服务的停止脚本。

**示例:**

* `/etc/rc.d/S10sshd`: SSH 服务的启动脚本，将在系统启动时较早启动。
* `/etc/rc.d/K20httpd`: HTTP 服务的停止脚本，将在系统关机时较早停止。

**注意:**

* 不要随意修改 `S` 或 `K` 脚本，否则可能会导致服务启动或停止失败。
* 如果您需要添加或删除服务，请参考 OpenWrt 官方文档。


部分需要关注的命令`ifstatus`,`ubus`,`jsonfilter`

1.分析启动优先级比较高，启动阶段，依赖的device还未完全初始化完成
2.最末优先级时刻依赖的device还未初始化完成，需要在start入口函数先主动sleep 30s
3.reboot 路由器，dynv6服务sleep 30s即可正常，在这个时间段wan6获取到公网ipv6地址
4.下电上电的情况的话等到dynv6服务(start=299)启动阶段到wan6获取到公网ipv6地址需要7mins左右，所以设置dynv6服务器在start函数入口先`sleep 10m`保证在机器获取到公网ipv6地址之后再进行ipv6地址同步到dynv6

5. 服务器启动时候的命令是bash /etc/rc.common servicepath boot



![Pasted image 20240406085614.png](/img/user/submodules/pve/openwrt/attachments/Pasted%20image%2020240406085614.png)

考虑到/usr/bin/dynv6-update.sh脚本是bash -e，所以不能直接

![Pasted image 20240406094925.png](/img/user/submodules/pve/openwrt/attachments/Pasted%20image%2020240406094925.png)



几个注意的点
1. init启动的几个service是同步进行的，A依赖B的话，A的START值可以设置大于B，对于在init.d中需要执行的一些耗时的操作需要在后台进行，否者存在依赖关系可能导致一些进程没法正常启动(之前没有后台启动导致网页服务没有启动，也一直没发获取到ipv6)
2. 对于脚本中的shellbang中加上了-e的选项之后，命令中存在返回值非0的操作都会导致进程退出，所以不能使用`echo $?`的方式来判断上一个命令是否
3. 对于日志定位的话可以在脚本开头加上`exec > /root/dynv6.log 2&1> 