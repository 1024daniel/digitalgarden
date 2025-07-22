---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/security/ssl/","noteIcon":"3"}
---

#ssl 

```cardlink
url: https://www.linkedin.com/pulse/beginning-ssl-platform-engineer-isaac-carrington/?trackingId=B9b8rRTIQRWO8FtCLnQnGA%3D%3D
title: "Beginning with SSL for a Platform Engineer"
description: "As an aspiring Software Engineer at Uni the infra stuff was mostly a mystical dark art to me. We covered it, but not enough for me to feel comfortable with it and I thought I'd be able to skip through my career without ever needing solid infra skills."
host: www.linkedin.com
favicon: https://static.licdn.com/aero-v1/sc/h/al2o9zrvru7aqj8e1x2rzsrca
image: https://media.licdn.com/dms/image/v2/C5612AQEa9clxy0lCvA/article-cover_image-shrink_720_1280/article-cover_image-shrink_720_1280/0/1647679243384?e=2147483647&v=beta&t=wTJKCE1c9Qb0_vJwGL_vEM3qcB2yMUyGCM8ivkPDrIg
```


对于公司内网服务器环境，如果存在获取指定网站域名的时候存在错误`curl: (60) SSL certificate problem: self signed certificate in certificate chain`, 可能是由于公司网关的隔离，需要客户端安装网关中间人的证书才能与外网建立SSL连接，即使不配置SSL证书，通过关闭SSL验证，设置可信站点或者指定使用的证书也可以建立SSL连接，但不是所有的应用都提供类似功能(比如一些python代码第三方包request之类的调用链比较长的，可能很难避免)
Git可以通过使用如下方式关闭SSL验证，但是Git LFS不共享此配置
```sh
git config --global htt.sslVerify false

```
对于以上问题可以通过将工位机windows上的证书拷贝到具体服务器环境，一般公司的windows镜像已经内置了SSL证书，通过在浏览器网址栏把正在使用的SSL证书另存为\<company\>-gateway-ca-cer.
对于Linux环境一般是需要crt格式的证书文件，可以通过以下命令将证书进行转换
```sh
openssl x509 -inform DER -in <company>-gateway-ca.cer -out <company>-gateway-ca.crt

```

然后直接将对应的crt文件放入到linux CA证书管理器文件夹
```sh
mv <company>-gateway-ca.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

```
如果是windows上得git bash环境，证书不是单独的一个一个文件，而是多个证书统一放在一个文件里面，可以直接把crt文件追加到对应的证书文件
```sh
cat <company>-gateway-ca.crt >> mingw64/ssl/certs/ca-bundle.crt

```
