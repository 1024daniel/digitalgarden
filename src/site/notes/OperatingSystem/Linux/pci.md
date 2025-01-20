---
dg-publish: true
---
#lspci #bus #device #pcie
pci拓扑结构，pci总线链路信息
`lspci -vt`
![B64F1025-1283-4184-BF08-1BBF76685801.png](/img/user/pics/B64F1025-1283-4184-BF08-1BBF76685801.png)
对于这里的的pci bridge，能扇出pci bridge和pci设备，新的pci bridge下的有新的总线
`lspci -vvvs d6:01.0|head -n2`确认为pci bridge设备