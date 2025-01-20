---
{"dg-publish":true,"permalink":"/submodules/pve/docker/mediacms/","noteIcon":"3"}
---

## Installation
[install tutorial](https://github.com/mediacms-io/mediacms/blob/main/docs/admins_docs.md#3-docker-installation)
config user/passwd.
![Pasted image 20230521133051.png](/img/user/submodules/pve/docker/pics/Pasted%20image%2020230521133051.png)


config restart policy
![Pasted image 20230521133133.png](/img/user/submodules/pve/docker/pics/Pasted%20image%2020230521133133.png)

## 更新版本
port change
![Pasted image 20231124003510.png](/img/user/pics/Pasted%20image%2020231124003510.png)

![Pasted image 20231123020919.png](/img/user/pics/Pasted%20image%2020231123020919.png)

postgre升级之后已有数据格式存在不匹配，需要进行调整
docker compose up报错显示我的数据格式应该使用版本为13的postgre，但是我现在docker compose的yaml文件制定了使用postgre 15，所以存在故障。
我们需要将代码仓中有关postgre的版本都还原成原始的13
[对应的仓库修改参考](https://github.com/mediacms-io/mediacms/commit/487e098b9672ec110dfd8abd8c1df918c37622fa)
![Pasted image 20231123234933.png](/img/user/pics/Pasted%20image%2020231123234933.png)


```bash
find . -type f -exec sed -i 's/image: postgres:15.2-alpine/image: postgre:13/g' {} +
```

![Pasted image 20231124002509.png](/img/user/pics/Pasted%20image%2020231124002509.png)


