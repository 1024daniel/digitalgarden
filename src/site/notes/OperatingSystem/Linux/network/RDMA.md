---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/network/RDMA/","noteIcon":"3"}
---


#RDMA

为了拉满网络带宽，降低网络延时
10G网络，开巨帧
10G以上网络，开RDMA 


Introduction to RDMA tech

---

What is RDMA: 
- new standards
- new protocols
- new hardware interface cards and switches
- new software

---

Three Type of RDMA

- <font color="#e5b9b7">Infiniband: 需要交换机和网卡支持</font>
- <font color="#76923c">RoCE：以以太网报文内嵌Infiniband报文，支持以太网网络，网卡需要支持RDMA</font>
- <font color="#f79646">iWARP：一个允许在TCP上执行RDMA的网络协议。 IB和RoCE中存在的功能在iWARP中不受支持。 这支持在标准以太网基础设施（交换机）上使用RDMA</font>


https://blog.51cto.com/liangchaoxi/5223656


![Pasted image 20240222120320.png](/img/user/OperatingSystem/Linux/network/attachments/Pasted%20image%2020240222120320.png)


RDMA (Remote Direct Memory Access) 技术允许网络中的计算机直接访问另一台计算机的内存，而无需CPU介入，从而提高网络通信的性能。DSCP (Differentiated Services Code Point) 是IP数据包中的一个字段，用于标记数据包的服务质量（QoS），以便网络设备可以对不同优先级的流量进行区分处理。

在RDMA环境中，DSCP设置通常用于确保关键任务流量得到优先处理。以下是一些关于RDMA DSCP设置的步骤和方法：

1. **设置QoS参数**：可以通过命令行工具如 `mlnx_qos` 来设置网络接口上的QoS参数，将DSCP设置为网卡的信任模式。例如，使用以下命令设置DSCP信任模式：
   ```
   # mlnx_qos -i <interface> --trust dscp
   ```
   其中 `<interface>` 是指父接口，例如 `ens2f0` [^1^][^3^]。

2. **设置ToS值**：可以为所有RoCE流量设置ToS值，例如将ToS设置为106（DSCP 26），这可以通过以下命令完成：
   ```
   # echo 106 > /sys/class/infiniband/<mlx-device>/tc/1/traffic_class
   ```
   这里 `<mlx-device>` 是指Mellanox设备，例如 `mlx5_0` [^1^]。

3. **使用RDMA-CM设置ToS/DSCP**：通过 `cma_roce_tos` 脚本来设置RDMA-CM QP的ToS/DSCP值。例如，要设置DSCP值为24，需要将该值乘以4（即96）：
   ```
   # cma_roce_tos -d <ib_device> -t 96
   ```
   注意，ToS字段是8位，而DSCP字段是6位，所以设置DSCP值时需要进行相应的转换 [^1^][^9^]。

4. **启用PFC（Priority Flow Control）**：PFC是一种流控机制，可以减少网络拥塞和丢包。可以通过 `mlnx_qos` 工具启用特定队列的PFC。例如，启用第四个队列的PFC：
   ```
   # mlnx_qos -i mlx5_0 --trust=dscp --pfc=0,0,0,1,0,0,0,0
   ```
   这表示除了第四个队列外，其他队列的PFC都被禁用 [^2^]。

5. **配置环境变量**：如果业务在容器中运行，需要在容器中配置环境变量，例如设置 `HCCL_RDMA_TC` 来定义发送的RoCE报文的DSCP值：
   ```
   export HCCL_RDMA_TC=100
   ```
   这将DSCP配置为25 [^3^]。

6. **使用 `hccn_tool` 命令**：在某些系统中，可能需要使用特定的工具来配置DSCP到TC（Traffic Class）的映射关系。例如，将DSCP 25映射到TC 2：
   ```
   hccn_tool -i 0 -dscp_to_tc -s dscp 25 tc 2
   ```
   这需要根据具体的系统和配置指南来执行 [^3^]。

请注意，上述步骤可能需要根据您的具体硬件和软件环境进行调整。在进行配置时，建议参考硬件供应商提供的文档和指南。