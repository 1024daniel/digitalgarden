---
{"dg-publish":true,"permalink":"/CloudNative/deploy virtual machine/","noteIcon":"3"}
---


![alessandro-armignacco-UATaDiqcmV0-unsplash.jpg|100%](/img/user/banner/alessandro-armignacco-UATaDiqcmV0-unsplash.jpg)
#virsh #virt-install 
## create vm
基于已有虚拟机的qcow2镜像文件创建虚拟机
```bash
virt-install --name master --memory 61440 --vcpus 16 --disk path=openEuler2-20-03-LTS-SP1.x64.qcow2,format=qcow2,bus=virtio\
 --import --nographics --network bridge=br0
 # --boot uefi会生成nvram文件
```
如果虚拟机内存需要大页内存，虚拟机xml中对应内存的配置应类似如下
```xml
<memory unit=GiB>60</memory>
<currentMemory uit=GiB>60</memory>
<memoryBacking>
	<hugepages>
		<page size='2048' unit='KiB'/> <!---预先在node进行分配选定的页大小-->
	</hugepages>
	<source type='file'/>
	<access mode='shared'/>
	<allocation mode='immediate'/>
<memoryBacking>
```

虚拟机通过NAT通外网
```bash
# -----  宿主机执行 ----------
# 这里192.168.1.0/24为网桥网络地址，enp125s0f0为90.90.64.0/24网络地址网卡，宿主机使用cntlm通过工位机90.253.0.0/24的ip与外网通信
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUING -s '192.168.1.0/24' -o enp125s0f0 -j MASQUERADE
# 如果宿主机也是通过代理与外网连接，则虚拟机内部需要配置一样的proxy
#  ------  虚拟机执行 -------
export http_proxy=90.253.xx.xx:3128
export https_proxy=$http_proxy
#如果虚拟机内有多个网卡，且没有和代理90.253.0.0/16网段的，则可能packet不一定从桥接宿主机的192.168.1.0/24地址的网卡发错，导致packet的source ip不是我们上面设置的nat的网络地址，没法使能NAT，所以需要我们添加一个路由规则，是的走代理的packet走192.168.1.0/24的网桥网卡
# 写法： ip route add <network_ip>/<cidr> via <gateway_ip> dev <interface>
ip route add 90.253.0.0/16 via 192.168.3.2 dev eth0 #这里的gateway是宿主机网桥地址
```
虚拟机挂载rbd image:[参考](https://docs.ceph.com/en/latest/rbd/libvirt/)

```bash
ceph osd pool create mysql-origin
rbd create mysql-origin/data --size 1T
tee secret.xml << EOF
<secret ephemeral='no' private='no'>
	<usage type='ceph'>
		<name>client.admin secret</name>
	</usage>
</secret>
EOF
virsh secret-define --file secret.xml ## this will generate a uuid
ceph auth get-key client.admin | tee client.admin.key
virsh secret-set-value --secret {uuid} --base64 $(cat client.admin.key)
cat > rbd-disk.xml << EOF
<disk type='network' device='disk'>
	<auth username='admin'>
		<secret type='ceph' uuid='your uuid generated above'/>
	</auth>
	<source protocol='rbd' name='mysql-origin/data'>
		<host name='monhost1_ip' port='6789'/>
		<host name='monhost2_ip' port='6789'/>
		<host name='monhost3_ip' port='6789'/>
	</source>
	<target protocol='vdb' bus='virtio'/>
</disk>
EOF
virsh attach-device master rbd-disk.xml --persistent

## or just command without xml
### [Replace `<domain>` with the name of the domain, `<pool>` with the name of the pool, `<image>` with the name of the image, and `<target>` with the target device name](https://raymii.org/s/tutorials/KVM_add_disk_image_or_swap_image_to_virtual_machine_with_virsh.html) [1](https://raymii.org/s/tutorials/KVM_add_disk_image_or_swap_image_to_virtual_machine_with_virsh.html).
virsh attach-device <domain> /dev/rbd/<pool>/<image> <target> --driver=rbd --subdriver=none --config /etc/ceph/ceph.conf --cache=none --live --persistent

```

## 直通网卡
#passthrough
首先确保直通的物理网卡是正常可以up，否者透传之后功能异常
### method 1
1. 首先确定iommu是否enabled, bios vt-d是否开启
`dmesg|grep -iE"DMAR|IOMMU"|grep enabled`
在/boot/efi/EFI/centos/grud.cfg 找到两个kernel的位置行末加上intel_iommu=on 或者amd_iommu=on之后reboot
3. 加载vfio驱动
`modprobe vfio && modprobe vfio-pci`
5. host解绑网卡
```bash
ethtool -i ens7|grep bus
echo "0000:3b:00.0" > /sys/bus/pci/devices/0000\:3b\:00.0/driver/unbind
```
7. 绑定网卡到vfio
```bash
lspci -s 0000:3b:00.0 -n ## get vendor and driver code
echo "8086 154d" > /sys/bus/pci/drivers/vfio-pci/new_id
```
9. 透传网卡至虚拟机
```bash
## method 1
cat > interface.xml << EOF
<hostdev mode='subsystem' type='pci' managed='yes'>
	<source>
		<address domain='0x0000' bus='0xdd' slot='0x00' function='0x0'/>
	</source>
</hostdev>
EOF
virsh attach-device master interface.xml --persistence
```
### method 2
```bash
virsh nodedev-list
virsh nodedev-dettach
virsh nodedev-reattach
virsh attach-interface master hostdev ens7 --persistent --live
```