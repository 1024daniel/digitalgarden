---
{"dg-publish":true,"permalink":"/submodules/pve/docker/calibre-web/","noteIcon":"3"}
---

#docker 
calibre-web is self-host library which can help organize you books.
## 1. Set up calibre-web using docker compose
create a calibre-web.yml file in folder /opt/containers/calibre-web/
```yml
version: "2.1"
services:
  calibre-web:
    image: lscr.io/linuxserver/calibre-web:latest
    container_name: calibre-web
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - DOCKER_MODS=linuxserver/mods:universal-calibre #optional
      - OAUTHLIB_RELAX_TOKEN_SCOPE=1 #optional
    volumes:
      - /opt/containers/calibre-web/config:/config
      - /opt/containers/calibre-web/library:/books
    ports:
      - 8083:8083
    restart: unless-stopped
```
We should create /opt/containers/calibre-web/config and /opt/containers/calibre-web/library before we set up the container.
```sh
cd /opt/containers/calibre-web/
docker compose -f calibre-web.yml up -d
```
![Pasted image 20230513184649.png](/img/user/submodules/pve/docker/pics/Pasted%20image%2020230513184649.png)

## 2. Permission Configuration of the calibre-web
Firstly we should create user admin in host because default user in calibre container is admin.
```sh
useradd admin
groups admin
```
copy the metadata.db in current folder to host folder:/opt/containers/calibre-web/library/ and adjust the permission.
![Pasted image 20230513184632.png](/img/user/submodules/pve/docker/pics/Pasted%20image%2020230513184632.png)

## 3. Login in and make some adjustment
login in with admin/admin123

go to admin->edit basic configuration->Feature Configuration->enable upload
