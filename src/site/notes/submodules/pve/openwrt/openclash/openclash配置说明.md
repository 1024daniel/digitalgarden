---
{"dg-publish":true,"permalink":"/submodules/pve/openwrt/openclash/openclash配置说明/","noteIcon":"3"}
---

#clash

https://github.com/Aethersailor/Custom_OpenClash_Rules?tab=readme-ov-file

### 1.域名指定使用节点
As I wanna use new bing's chat AI in clash rule mode(it work in clash global mode), I need to overwrite clash default rule mode to make bing.com does not go to cn.bing.com.
go to openclash and download clash rule file
![Pasted image 20230711215316.png|90%](/img/user/submodules/pve/pics/Pasted%20image%2020230711215316.png)

we can see all the aviable proxy node
![Pasted image 20230711215607.png|90%](/img/user/submodules/pve/pics/Pasted%20image%2020230711215607.png)

use one node where bing is not blocked
![Pasted image 20230711215746.png|90%](/img/user/submodules/pve/pics/Pasted%20image%2020230711215746.png)
click apply setting.
now go into clash yacd web and check if the new rule is working
![Pasted image 20230711215915.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230711215915.png)


now I can use bing's chat AI
![Pasted image 20230711220000.png|100%](/img/user/submodules/pve/pics/Pasted%20image%2020230711220000.png)


### 2.域名不走代理
#dns
连上开启openclash的路由器wifi之后bilibili没法登录，可能是使用了错误代理之类导致的，在openclash的config manger设置界面可以添加一个rule，将域名包含bilibili.com的都直接走DIRECT，即不走代理
![Pasted image 20241214190107.png](/img/user/submodules/pve/openwrt/openclash/attachments/Pasted%20image%2020241214190107.png)

如果以上设置不生效，fake-ip-filter中加入b站相关域名，来设置这些域名不走fake ip模式
![Pasted image 20250401223857.png](/img/user/submodules/pve/openwrt/openclash/attachments/Pasted%20image%2020250401223857.png)

注意看上面的nameserver-policy主要是为特定域名设置推荐的DNS服务器

• **阿里系域名**（tmall.com、taobao.com、alipay.com 等）使用 **阿里 DNS**（223.5.5.5）。

• **腾讯系域名**（qq.com、tencent.com、weixin.com）使用 **DNSPod DNS**（119.29.29.29）。

• **百度** 相关域名使用 **阿里 DNS**（223.5.5.5）。

• **B 站（bilivideo.+）、爱奇艺（iqiyi.com）** 走 **腾讯 DNS**（119.29.29.29）。

• **抖音（douyin.com、douyincdn.com）** 走 **专用 DNS 180.184.1.1**。

• **飞书（feishu.cn）** 走 **180.184.1.1**。