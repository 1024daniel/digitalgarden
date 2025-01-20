---
{"dg-publish":true,"permalink":"/submodules/pve/proxmox setup note/","noteIcon":"3"}
---

#deploy #pve

![Pasted image 20231119161843.png|100%](/img/user/pics/Pasted%20image%2020231119161843.png)

## My Home Lab Server Topology
![pve.png](/img/user/pics/pve.png)
## 1. use rufus to make a boot usb from a installed proxmox iso

## 2. start the machine and press del to get into bios,  set usb as the most prority device to load os
## config source.list
## restore the usb to normal
1. use cmd to clean usb
```powershell
diskpart
list disk
select disk 1 
clean
```
2. go to disk manager to create a new filesystem

## 3. prepare some deb packages in usb for some elemental functions for proxmox
1. install wpa_supplicant 

## 4. install some fundamental debs in usb
1. deb website
2. install prepared deb from the usb stick
wpa_supplicant depends libnl and libpcslite
```sh
dpkg -i wpasupplicant_2.9.0-21_amd64.deb
dpkg -i libnl-genl-3-200_3.4.0-1+b1_amd64.deb
dpkg -i libpcsclite1_1.8.20-1_amd64.deb
```
## 5. config network
1. use wpa_supplicant 
```sh
wpa_passphrase _MYSSID_ _passphrase_ > /etc/wpa_supplicant/example.conf
```
you can add multiple networks and add priority option to give different network different prorities.
```sh
wpa_passphrase _MYSSID_ _passphrase_ >> /etc/wpa_supplicant/example.conf
```


commands to list networks:

```
wpa_cli -i wlan0 list_networks
```

to get a list of your configured networks. And to connect to first network:

```
wpa_cli -i wlan0 select_network 0
```

To shift from `wifi_A` to `wifi_B` use:

```
wpa_cli -i wlan0 select_network 1
```
3. wpa_supplicant usage reference
[wpa_supplicant usage reference](https://wiki.archlinux.org/title/wpa_supplicant)


## 6. config ssh
1. copy windows ssh public key to pve
```powershell
type $env:USERPROFILE\.ssh\id_rsa.pub | ssh {IP-ADDRESS-OR-FQDN} "cat >> /root/.ssh/authorized_keys"
```

3. install openssh-client, openssh-server
4. setup authenication between windows and proxmox
5. config beautiful banner
## 7. install apcupsd
## 8. install SDN tool
### 8.1 zerotier
### 8.2 headscale or tailscale
[deploy tutorial](https://icloudnative.io/posts/how-to-set-up-or-migrate-headscale/)
[download tailscale for linux](https://pkgs.tailscale.com/stable/#static)
after copy file to system path start the service and connect to our network with the help of google sso
```sh
systemctl start tailscaled
tailscale up
tailscale status
```


## 9.install fancontrol
1. apt install lm-sensors gcc make git
2. get modified it87 for pwm support
```shell
cp /lib/modules/`uname -r`/kernel/drivers/hwmon/it87.ko /home # backup it87
git clone https://github.com/a1wong/it87.git
wget http://download.proxmox.com/debian/pve/dists/bullseye/pvetest/binary-amd64/pve-headers-5.15.102-1-pve_5.15.102-1_amd64.deb
apt install ./pve-headers-5.15.102-1-pve_5.15.102-1_amd64.deb
cd it87 && make -j
make install
```
you can then check whether the we can detect fan speed and temptation
> sensors

if we can see fan speed is normal, then we install fancontrol
> apt install fancontrol

finallly we use pwmconfig to config the options for fancontrol
> pwmconfig

make sure it87 module it loaded before fancontrol service startup
![Pasted image 20230423224402.png](/img/user/submodules/pve/pics/Pasted%20image%2020230423224402.png)






