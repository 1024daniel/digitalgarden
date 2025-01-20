---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/nginx/","noteIcon":"3"}
---

#nginx #portforward
容器内启动nginx并进行端口转发

```sh
# 容器内没有systemctl相关命令，没法通过systemd将nginx像service服务这种进行启动
nginx -g "damon off;"
# 查看nginx的配置文件路径
/usr/sbin/nginx -t

```

```sh
cd /etc/nginx/`/etc/nginx/sites-available/`
cp sites-available/default sites-available/mindie
ln -s sites-enabled/mindie sites-available/minide

```

编辑`/etc/nginx/site-available/minide`
```json

    server {
        listen 81; # 监听 81 端口, default里面是80，这里设置不同的监听端口
        server_name example.com; # 服务器名称

        # 访问控制
        location / {
            # 允许来自特定 IP 地址的访问
            allow 192.168.1.0/24;
            deny all;

            # 代理设置
            proxy_pass http://127.0.0.1:1025; #转发到特定的端口
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # 过滤特定请求
        location ~* \.(jpg|jpeg|png|gif)$ {
            # 拒绝图片文件的直接访问
            deny all;
        }
        

        # 其他配置...
    }
```

完成编辑之后直接reload
```sh
nginx -s reload
# 查看对应端口是否已经监听成功了
netstat -tunlp|grep nginx

```