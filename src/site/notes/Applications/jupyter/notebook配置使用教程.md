---
{"dg-publish":true,"permalink":"/Applications/jupyter/notebook配置使用教程/","noteIcon":"3"}
---

#jupyter
### 1.安装
```sh
pip install jupyter
# 配置jupyter登录密码
jupyter notebook password

# 外网访问常用配置
vim ~/.jupyter/jupyter_ntoebook_config.py
c.NotebookApp.allow_remote_access = True
c.NotebookApp.allow_root = True
c.NotebookApp.allow_ip = '0..0.0.0'
c.NotebookApp.port = 8888

```

### 2.配置自动服务自动拉起
```sh
crontab -e
```

![Pasted image 20250119013727.png](/img/user/Applications/jupyter/attachments/Pasted%20image%2020250119013727.png)