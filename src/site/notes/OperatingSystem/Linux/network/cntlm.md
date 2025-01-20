---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/network/cntlm/","noteIcon":"3"}
---

#proxy #network
## background
We need proxy to visit outside network, and it need verification.
But a lot of software, like git, nodejs, npm and so on sometimes cannot connect to outside network without proxy.
## Solution
Install proxy software like cntlm to make local pc a proxy server if local pc has the verification to connect outside network.
## Installation and configuration
### 1. Install cnltm
install setup.exe from below link
[cntlm sourceforge](https://sourceforge.net/projects/cntlm)
### 2. Config cntlm.ini

```json
Username x30029338
Domain china.huawei.com
Proxy proxyhk.huawei.com:8080
NoProxy localhost,127.0.0.*,10.*,192.168.*,*huawei.com
Listen 3128
Gateway yes
Allow 90.90.0.0/16


```

### 3. Generate password
Execute command ".\cntlm.exe -H -u x30029338", and type in your password in the prompt. You wil get password which is needed in the cntlm.ini from the output of previous command.
```json
Auth NTLM
PassLM *****
PassNT *****
```
### 4. Start cntlm
execute ""regedit", open registry editor, enter HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\cntlm\\Parameters,
change AppArgs' value to "-f -c "C:\Program Files (x86)\\Cntlm\\cntlm.ini"
open admin powershell, execute command "net start cntlm"

## Config Proxy using cntlm proxy server.
after we start the cntlm in the pc, we can set proxy in our develop environment.
```bash
export http_proxy=http://$pc_ip:3218 
export https_proxy=$http_proxy
```
