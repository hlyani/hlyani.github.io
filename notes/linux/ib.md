# IB

# 一、介绍

InfiniBand ，一种高性能计算和数据中心网络技术。它提供了一种低延迟、高带宽和可靠性。

RDMA(Remote Direct Memory Access)

# 二、安装

https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/8/html-single/configuring_infiniband_and_rdma_networks/index#renaming-ipoib-devices_configuring-ipoib

# 三、测试

## 1、查看状态

##### 确认 IB  网卡硬件设备是否正常识别

```
ip a|grep ib

6: ib0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 2044 qdisc mq state UP group default qlen 256
    link/infiniband 00:00:10:29:fe:80:00:00:00:00:00:00:04:3f:72:03:00:da:21:30 brd 00:ff:ff:ff:ff:12:40:1b:ff:ff:00:00:00:00:00:00:ff:ff:ff:ff
    inet 192.168.1.11/8 brd 11.255.255.255 scope global ib0

```

##### 查看 IB 网卡状态

```
ibstat

CA 'mlx5_0'
	CA type: MT4119
	Number of ports: 1
	Firmware version: 16.32.1010
	Hardware version: 0
	Node GUID: 0x043f720300da2130
	System image GUID: 0x043f720300da2130
	Port 1:
		State: Active
		Physical state: LinkUp
		Rate: 100
		Base lid: 25606
		LMC: 0
		SM lid: 117
		Capability mask: 0x2651e848
		Port GUID: 0x043f720300da2130
		Link layer: InfiniBand
```

> - CA 'mlx5_0': 表示设备的逻辑名称
> - CA类型：HCA型号为 MT4119。
> - 端口数量：HCA有1个端口。
> - 固件版本：HCA固件版本为 16.32.1010。
> - 硬件版本：HCA硬件版本为0。
> - 节点GUID：此HCA的节点全局唯一标识符(GUID)为0x043f720300da2130。
> - 系统镜像GUID：此HCA的系统镜像GUID为0x043f720300da2130。
> - 端口1：这提供了有关HCA端口1的信息：
> - 状态：端口1的状态为“活动”。
> - 物理状态：端口1的物理状态为“LinkUp”。
> - 速率：端口1的数据传输速率为 100 Gbps。
> - 基本LID：端口1的基本LID(本地标识符)为25606。
> - LMC：端口1的LMC(LID掩码计数)为0，表示仅使用基本LID。
> - SM LID：端口1的SM LID(子网管理器LID)为117。
> - 能力掩码：端口1的能力掩码为0x2651e848。
> - 端口GUID：端口1的端口GUID为0x043f720300da2130。
> - 链接层：端口1使用的链接层协议是InfiniBand。

##### 查看所有 InfiniBand 设备

```
ibv_devices

    device          	   node GUID
    ------          	----------------
    mlx5_0          	043f720300da2130
```

##### 详细检查设备信息

```
ibv_devinfo

hca_id:	mlx5_0
	transport:			InfiniBand (0)
	fw_ver:				16.32.1010
	node_guid:			043f:7203:00da:2130
	sys_image_guid:			043f:7203:00da:2130
	vendor_id:			0x02c9
	vendor_part_id:			4119
	hw_ver:				0x0
	board_id:			MT_0000000010
	phys_port_cnt:			1
		port:	1
			state:			PORT_ACTIVE (4)
			max_mtu:		4096 (5)
			active_mtu:		4096 (5)
			sm_lid:			117
			port_lid:		25606
			port_lmc:		0x00
			link_layer:		InfiniBand
```

##### 查看 PCI 设备

```
lspci | grep -i mellanox

61:00.0 Infiniband controller: Mellanox Technologies MT27800 Family [ConnectX-5]
```

##### 检测内核驱动

```
lsmod | grep mlx

mlx5_ib               375821  0 
ib_uverbs             132749  2 mlx5_ib,rdma_ucm
ib_core               357685  10 rdma_cm,ib_cm,iw_cm,mlx5_ib,ib_umad,ib_uverbs,rdma_ucm,ib_ipoib
mlx5_core            1277550  1 mlx5_ib
mlxfw                  22321  1 mlx5_core
psample                13526  1 mlx5_core
devlink                48345  1 mlx5_core
mlx_compat             56599  10 rdma_cm,ib_cm,iw_cm,mlx5_ib,ib_core,ib_umad,ib_uverbs,mlx5_core,rdma_ucm,ib_ipoib
ptp                    19231  3 igb,i40e,mlx5_core
```

## 2、使用 iperf3 测试

> iperf3 可以用于测试 InfiniBand 组网的性能和带宽。为此，需要在 InfiniBand 网络中确认 InfiniBand 适配器已启用 `IPoIB` 功能。`IPoIB`是一种在`InfiniBand`网络上传输`IP数据`的方法，它允许使用标准的`TCP/IP`协议栈和网络应用程序。

##### 1 在启用`IPoIB`之前，确保已正确配置InfiniBand适配器和子网管理器(SM)。

```
lsmod | grep ib_ipoib
modprobe  ib_ipoib
```

### 2 给IB卡配置IP

```
ip a add 192.168.1.11/8 dev ib0
ip l set ib0 up
```

```
 ping 192.168.1.11
 ping 192.168.1.11 -I ib0
```

### 3 使用 iperf3 测试

```
iperf3 -s

------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  4] local 192.168.1.11 port 5001 connected with 192.168.1.13 port 57327
[ ID] Interval       Transfer     Bandwidth
[  4]  0.0-10.0 sec  14.7 GBytes  12.6 Gbits/sec
```

> 5201

```
iperf3 -c 192.168.1.11

------------------------------------------------------------
Client connecting to 192.168.1.11, TCP port 5001
TCP window size:  408 KByte (default)
------------------------------------------------------------
[  3] local 192.168.1.13 port 57327 connected with 192.168.1.11 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-10.0 sec  14.7 GBytes  12.6 Gbits/sec
```

> - ID：测试流的唯一标识符。
> - Interval：测试的时间间隔，以秒为单位。
> - Transfer：在测试过程中传输的总字节数。
> - Bitrate：传输速率，以比特每秒(bps)为单位。
> - Retr：在测试期间发生的重传次数。
> - Sender：表示此行所列出的结果来自iperf3客户端。
> - Receiver：表示此行所列出的结果来自iperf3服务器。

## 3、带宽测试

```
FROM ubuntu:22.04

RUN apt update && apt install -y perftest iperf3 infiniband-diags net-tools iputils-ping wrk vim iproute2 && apt clean
```

```
docker build -t test.com:5000/k8s/perftest .
```

```
cat <<EOF |kubectl apply -f -  
apiVersion: v1
kind: Pod
metadata:
  name: perf-test-server
  namespace: iperf
spec:
  tolerations:
  - key: "CriticalAddonsOnly"
    operator: "Exists"
  - operator: "Exists"
    effect: "NoSchedule"
  - operator: "Exists"
    effect: "NoExecute"
  nodeSelector:
    kubernetes.io/hostname: 10.0.0.11
  containers:
  - name: iperf3
    image: test.com:5000/k8s/perftest:latest
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "sleep infinity"]
    ports:
    - containerPort: 5201
    resources:
      requests:
        rdma/rdma_shared_device_a: 1
      limits:
        rdma/rdma_shared_device_a: 1
    securityContext:
      capabilities:
        add:
        - IPC_LOCK
---
apiVersion: v1
kind: Pod
metadata:
  name: perf-test-client1
  namespace: iperf
spec:
  tolerations:
  - key: "CriticalAddonsOnly"
    operator: "Exists"
  - operator: "Exists"
    effect: "NoSchedule"
  - operator: "Exists"
    effect: "NoExecute"
  nodeSelector:
    kubernetes.io/hostname: 10.0.0.11
  containers:
  - name: iperf3
    image: test.com:5000/k8s/perftest:latest
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "sleep infinity"]
    resources:
      requests:
        rdma/rdma_shared_device_a: 1
      limits:
        rdma/rdma_shared_device_a: 1
    securityContext:
      capabilities:
        add:
        - IPC_LOCK
---
apiVersion: v1
kind: Pod
metadata:
  name: perf-test-client1
  namespace: iperf
spec:
  tolerations:
  - key: "CriticalAddonsOnly"
    operator: "Exists"
  - operator: "Exists"
    effect: "NoSchedule"
  - operator: "Exists"
    effect: "NoExecute"
  nodeSelector:
    kubernetes.io/hostname: 10.0.0.12
  containers:
  - name: iperf3
    image: test.com:5000/k8s/perftest:latest
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "sleep infinity"]
    resources:
      requests:
        rdma/rdma_shared_device_a: 1
      limits:
        rdma/rdma_shared_device_a: 1
    securityContext:
      capabilities:
        add:
        - IPC_LOCK
EOF
```

##### 带宽测试 `ib_send_bw`  ,在server端执行

```
ib_send_bw -a -c UD -d mlx5_0 -i 1

************************************
* Waiting for client to connect... *
************************************
 Max msg size in UD is MTU 4096
 Changing to this MTU
---------------------------------------------------------------------------------------
                    Send BW Test
 Dual-port       : OFF		Device         : mlx5_0
 Number of qps   : 1		Transport type : IB
 Connection type : UD		Using SRQ      : OFF
 PCIe relax order: ON
 ibv_wr* API     : ON
 RX depth        : 1000
 CQ Moderation   : 100
 Mtu             : 4096[B]
 Link type       : IB
 Max inline data : 0[B]
 rdma_cm QPs	 : OFF
 Data ex. method : Ethernet
---------------------------------------------------------------------------------------
 local address: LID 0x6406 QPN 0x1652 PSN 0xb6e71d
 remote address: LID 0x62ca QPN 0x19e1 PSN 0xb814bc
---------------------------------------------------------------------------------------
 #bytes     #iterations    BW peak[MB/sec]    BW average[MB/sec]   MsgRate[Mpps]
 2          1000             0.00               5.39   		   2.825338
 4          1000             0.00               10.67  		   2.797672
 8          1000             0.00               21.54  		   2.823264
 16         1000             0.00               30.30  		   1.985466
 32         1000             0.00               86.60  		   2.837765
 64         1000             0.00               166.40 		   2.726281
 128        1000             0.00               339.47 		   2.780945
 256        1000             0.00               494.67 		   2.026178
 512        1000             0.00               1015.35		   2.079434
 1024       1000             0.00               2066.23		   2.115820
 2048       1000             0.00               3615.16		   1.850961
 4096       1000             0.00               5706.22		   1.460792
---------------------------------------------------------------------------------------
```

> - `-a`：这个选项通常用于自动选择最佳的包大小来进行测试。
> - `-c UD`：指定使用 Unreliable Datagram (UD) 传输类型进行测试。UD 是一种无连接、不可靠的数据报服务，它适用于那些不需要保证数据可靠传输的场景，例如一些高性能计算或大数据处理任务。
> - `-d mlx5_0`：指定使用 mlx5_0这个 InfiniBand 设备来进行测试。mlx5_0是 InfiniBand 设备的名称，通常与特定的硬件适配器相关联。
> - `-i `1：这个选项指定了迭代次数，即测试会运行 1 次。

##### 在client端执行

```
ib_send_bw -a -c UD -d mlx5_0 -i 1 192.168.1.11

 Max msg size in UD is MTU 4096
 Changing to this MTU
---------------------------------------------------------------------------------------
                    Send BW Test
 Dual-port       : OFF		Device         : mlx5_0
 Number of qps   : 1		Transport type : IB
 Connection type : UD		Using SRQ      : OFF
 PCIe relax order: ON
 ibv_wr* API     : ON
 TX depth        : 128
 CQ Moderation   : 100
 Mtu             : 4096[B]
 Link type       : IB
 Max inline data : 0[B]
 rdma_cm QPs	 : OFF
 Data ex. method : Ethernet
---------------------------------------------------------------------------------------
 local address: LID 0x62ca QPN 0x19e1 PSN 0xb814bc
 remote address: LID 0x6406 QPN 0x1652 PSN 0xb6e71d
---------------------------------------------------------------------------------------
 #bytes     #iterations    BW peak[MB/sec]    BW average[MB/sec]   MsgRate[Mpps]
 2          1000             5.33               4.87   		   2.553039
 4          1000             10.72              10.44  		   2.738001
 8          1000             21.43              21.18  		   2.776313
 16         1000             42.56              40.84  		   2.676230
 32         1000             85.48              85.19  		   2.791503
 64         1000             170.97             163.61 		   2.680534
 128        1000             339.56             333.17 		   2.729332
 256        1000             513.44             486.25 		   1.991675
 512        1000             1005.73            1001.93		   2.051956
 1024       1000             2049.45            2041.78		   2.090782
 2048       1000             4052.13            3222.50		   1.649920
 4096       1000             7301.40            6170.13		   1.579554
---------------------------------------------------------------------------------------
```

## 4、延时测试

##### 延时测试 `ib_send_lat`, 在server端执行

```
ib_send_lat -a -c UD -d mlx5_0 -i 1

************************************
* Waiting for client to connect... *
************************************
 Max msg size in UD is MTU 4096
 Changing to this MTU
---------------------------------------------------------------------------------------
                    Send Latency Test
 Dual-port       : OFF		Device         : mlx5_0
 Number of qps   : 1		Transport type : IB
 Connection type : UD		Using SRQ      : OFF
 PCIe relax order: ON
 ibv_wr* API     : ON
 RX depth        : 1000
 Mtu             : 4096[B]
 Link type       : IB
 Max inline data : 188[B]
 rdma_cm QPs	 : OFF
 Data ex. method : Ethernet
---------------------------------------------------------------------------------------
 local address: LID 0x6406 QPN 0x1653 PSN 0x35c980
 remote address: LID 0x62ca QPN 0x19e2 PSN 0x705b26
---------------------------------------------------------------------------------------
 #bytes #iterations    t_min[usec]    t_max[usec]  t_typical[usec]    t_avg[usec]    t_stdev[usec]   99% percentile[usec]   99.9% percentile[usec] 
 2       1000          1.65           6.25         1.67     	       1.70        	0.28   		3.52    		6.25   
 4       1000          1.83           7.17         1.86     	       1.90        	0.33   		2.19    		7.17   
 8       1000          1.76           6.06         1.80     	       1.84        	0.26   		2.08    		6.06   
 16      1000          1.65           6.21         1.68     	       1.70        	0.21   		1.97    		6.21   
 32      1000          1.78           12.80        1.81     	       1.94        	0.77   		6.75    		12.80  
 64      1000          1.73           6.09         1.76     	       1.78        	0.22   		1.95    		6.09   
 128     1000          1.84           5.93         1.87     	       1.90        	0.28   		2.17    		5.93   
 256     1000          2.69           6.57         2.72     	       2.74        	0.24   		4.00    		6.57   
 512     1000          2.44           6.21         2.48     	       2.52        	0.25   		3.71    		6.21   
 1024    1000          2.56           6.55         2.60     	       2.63        	0.29   		4.54    		6.55   
 2048    1000          2.98           6.56         3.02     	       3.05        	0.25   		4.48    		6.56   
 4096    1000          3.71           7.94         3.73     	       3.76        	0.16   		5.04    		7.94   
---------------------------------------------------------------------------------------
```

##### 在client端执行

```
ib_send_lat -a -c UD -d mlx5_0 -i 1 192.168.1.11

 Max msg size in UD is MTU 4096
 Changing to this MTU
---------------------------------------------------------------------------------------
                    Send Latency Test
 Dual-port       : OFF		Device         : mlx5_0
 Number of qps   : 1		Transport type : IB
 Connection type : UD		Using SRQ      : OFF
 PCIe relax order: ON
 ibv_wr* API     : ON
 TX depth        : 1
 Mtu             : 4096[B]
 Link type       : IB
 Max inline data : 188[B]
 rdma_cm QPs	 : OFF
 Data ex. method : Ethernet
---------------------------------------------------------------------------------------
 local address: LID 0x62ca QPN 0x19e2 PSN 0x705b26
 remote address: LID 0x6406 QPN 0x1653 PSN 0x35c980
---------------------------------------------------------------------------------------
 #bytes #iterations    t_min[usec]    t_max[usec]  t_typical[usec]    t_avg[usec]    t_stdev[usec]   99% percentile[usec]  99.9% percentile[usec] 
 2       1000          1.65           6.28         1.67     	       1.70        	0.28   		3.52    		6.28   
 4       1000          1.83           7.47         1.86     	       1.90        	0.36   		2.19    		7.47   
 8       1000          1.77           7.19         1.79     	       1.84        	0.28   		2.09    		7.19   
 16      1000          1.65           7.43         1.68     	       1.70        	0.24   		1.97    		7.43   
 32      1000          1.78           12.79        1.81     	       1.94        	0.77   		6.74    		12.79  
 64      1000          1.72           7.50         1.76     	       1.78        	0.24   		1.96    		7.50   
 128     1000          1.84           6.91         1.87     	       1.90        	0.30   		2.19    		6.91   
 256     1000          2.70           8.32         2.72     	       2.75        	0.26   		3.99    		8.32   
 512     1000          2.44           8.03         2.48     	       2.52        	0.27   		3.77    		8.03   
 1024    1000          2.56           8.78         2.60     	       2.64        	0.31   		4.55    		8.78   
 2048    1000          2.98           8.01         3.02     	       3.05        	0.27   		4.50    		8.01   
 4096    1000          3.70           7.95         3.73     	       3.76        	0.19   		5.04    		7.95   
---------------------------------------------------------------------------------------
```

## 5、CPU 频率不一致问题

> Conflicting CPU frequency values detected: 1200.000000 != 2000.000000. CPU Frequency is not max.

```
cat /proc/cpuinfo | grep "cpu MHz"
lscpu | grep "CPU MHz"
```

Linux 内核提供了多种调度器（governors）来动态调整 CPU 频率，基于负载和功耗进行优化。常见的 CPU governor 包括：

- **performance**：始终运行在最大频率，适用于需要高性能的任务。
- **powersave**：始终运行在最低频率，适用于节能。
- **ondemand**：根据系统负载动态调整频率，适用于平衡性能和功耗。
- **conservative**：类似于 `ondemand`，但调整频率的步骤更小。
- **userspace**：允许用户手动设置频率。

```
# 查看
cpupower frequency-info
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

```
cpupower frequency-set --governor performance
```

```
# 检查 CPU 频率调节驱动
lsmod | grep acpi_cpufreq
modprobe acpi_cpufreq
```

```
# 调整 CPU 频率在多核系统上的影响
echo "performance" | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

## 其他命令

```
ibstat
ibstatus
ibv_devinfo
ibv_devices
rdma link
mlnx_perf -i ib0 | grep rdma
ibstat: 查询 InfiniBand 设备的基本状态
ibstatus： 网卡信息
ibv_devinfo：网卡设备信息（ibv_devinfo -d mlx5_0 -v）
ibv_devices：查看本主机的 infiniband 设备
ibnodes：查看网络中的 infiniband 设备
show_gids：看看网卡支持的 roce 版本
show_counters: 网卡端口统计数据，比如发送接受数据大小
mlxconfig: 网卡配置（mlxconfig -d mlx5_1 q 查询网卡配置信息）
```

**perftest 工具集**：

- 包括 `ib_read_bw`、`ib_write_bw`、`ib_send_bw` 等，用于 RDMA 的点对点测试。

> ### **连接配置**
>
> - `-d <dev_name>`：指定 RDMA 设备（如 `mlx5_0`）。
> - `-i <port>`：指定设备端口号（如 1）。
> - `-R`：使用 RDMA 读写模式。
> - `-F`：强制使用 TCP 连接的 IB 地址。
>
> ### **消息大小**
>
> - `-s <size>`：设置数据包大小（默认 2KB）。可以测试特定消息大小的带宽性能。
> - `--range <min>:<max>`：设置消息大小的范围，测试多种消息大小的性能。
>
> ### **测试控制**
>
> - `-n <iters>`：设置发送消息的迭代次数（默认 1000）。
> - `-t <duration>`：设置测试时长（秒），可以让测试持续更长时间。
> - `-x <sl>`：设置服务等级（Service Level）。
>
> ### **性能优化**
>
> - `--cpu_util`：启用 CPU 使用率统计。
> - `--report_gbps`：结果以 Gbps 为单位报告。
>
> ### **结果输出**
>
> - `-o <file>`：将测试结果保存到指定文件。
> - `--report_format <format>`：指定输出格式，例如 CSV。

## 6、pod中使用

#### 步骤一：安装和配置InfiniBand硬件和驱动

在物理机上安装InfiniBand网卡，并安装相应的驱动程序。

#### 步骤二：部署InfiniBand CNI插件

首先，在Kubernetes集群中安装InfiniBand CNI插件，例如`rdma-cni`插件。

```bash
kubectl apply -f https://raw.githubusercontent.com/Mellanox/rdma-cni/master/k8s-rdma-cni.yaml
```

#### 步骤三：创建IB网络资源

创建一个IB网络资源，例如名为`ib-network`的网络。

```yaml
apiVersion: "rdma.cni.k8s.io/v1"
kind: RDMAConfiguration
metadata:
name: ib-network
spec:
ibDevices: ["mlx5_1"]
```

保存为`ib-network.yaml`文件，并执行以下命令：

```bash
kubectl apply -f ib-network.yaml
```

#### 步骤四：部署Pod并指定使用IB网络

创建一个Pod，并在Pod的配置中指定使用`ib-network`网络资源。

```yaml
apiVersion: v1
kind: Pod
metadata:
name: ib-pod
spec:
containers:
- name: ib-container
image: nginx
resources:
limits:
rdma/ib-network: 1
```

保存为`ib-pod.yaml`文件，并执行以下命令：

```bash
kubectl apply -f ib-pod.yaml
```
