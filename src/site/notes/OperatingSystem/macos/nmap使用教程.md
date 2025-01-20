---
{"dg-publish":true,"permalink":"/OperatingSystem/macos/nmap使用教程/","noteIcon":"3"}
---

### 1.扫描端口
```bash
nmap localhost
nmap -F remote_ip
```
### 2.扫描网络
```bash
nmap -sP 192.168.0.1-25
nmap 192.168.0.*
```

3.traceroute
```bash
nmap --dns-servers 8.8.8.8 yahoo.com
sudo nmap --traceroute yahoo.com
```