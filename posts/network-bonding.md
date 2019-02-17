# 网卡绑定

## 七种bond模式

### mod=0 (balance-rr) Round-robin policy（平衡抡循环策略）

特点：传输数据包顺序是依次传输（即：第 1 个包走 eth0，下一个包就走 eth1….一直循环下去，直到最后一个传输完毕），此模式提供负载平衡和容错能力；但是我们知道如果一个连接或者会话的数据包从不同的接口发出的话，中途再经过不同的链路，在客户端很有可能会出现数据包无序到达的问题，而无序到达的数据包需要重新要求被发送，这样网络的吞吐量就会下降

### mod=1 (active-backup) Active-backup policy（主-备份策略）

特点：只有一个设备处于活动状态，当一个宕掉另一个马上由备份转换为主设备。mac地址是外部可见得，从外面看来，bond 的 MAC 地址是唯一的，以避免 switch (交换机) 发生混乱。此模式只提供了容错能力；由此可见此算法的优点是可以提供高网络连接的可用性，但是它的资源利用率较低，只有一个接口处于工作状态，在有 N 个网络接口的情况下，资源利用率为 `1/N`

### mod=2 (balance-xor) XOR policy（平衡策略）

特点：基于指定的传输HASH策略传输数据包。缺省的策略是：(源 MAC 地址 XOR 目标 MAC 地址) % slave 数量。其他的传输策略可以通过 xmit_hash_policy 选项指定，此模式提供负载平衡和容错能力

### mod=3 broadcast（广播策略）

特点：在每个 slave 接口上传输每个数据包，此模式提供了容错能力

### mod=4 (802.3ad) IEEE 802.3ad Dynamic link aggregation（IEEE 802.3ad 动态链接聚合）

特点：创建一个聚合组，它们共享同样的速率和双工设定。根据 `802.3ad` 规范将多个 slave 工作在同一个激活的聚合体下。

外出流量的 slave 选举是基于传输 hash 策略，该策略可以通过 xmit_hash_policy 选项从缺省的 XOR 策略改变到其他策略。需要注意的是，并不是所有的传输策略都是 `802.3ad` 适应的，尤其考虑到在 `802.3ad` 标准包乱序问题。不同的实现可能会有不同的适应性。

必要条件：

条件1：ethtool 支持获取每个 slave 的速率和双工设定

条件2：switch(交换机)支持 IEEE 802.3ad Dynamic link aggregation

条件3：大多数 switch (交换机)需要经过特定配置才能支持 `802.3ad` 模式

### mod=5 (balance-tlb) Adaptive transmit load balancing（适配器传输负载均衡）

特点：不需要任何特别的 switch (交换机)支持的通道 bonding。在每个 slave 上根据当前的负载（根据速度计算）分配外出流量。如果正在接受数据的 slave 出故障了，另一个 slave 接管失败的 slave 的 MAC 地址。

该模式的必要条件：ethtool 支持获取每个 slave 的速率

### mod=6 (balance-alb) Adaptive load balancing（适配器适应性负载均衡）

特点：该模式包含了 balance-tlb 模式，同时加上针对 IPv4 流量的接收负载均衡(receive load balance, rlb)，而且不需要任何 switch (交换机)的支持。接收负载均衡是通过 ARP 协商实现的。bonding 驱动截获本机发送的 ARP 应答，并把源硬件地址改写为 bond 中某个 slave 的唯一硬件地址，从而使得不同的对端使用不同的硬件地址进行通信。

### 哪些模式需要交换机的支持？

mod 1, 5, 6 完全不需要交换机支持；mod 0, 2, 3 需要在交换机上配置静态聚合；mod 4 需要交换机支持 802.3ad 协议

## CentOS 配置

### 第一步：创建一个ifcfg-bondX

vi /etc/sysconfig/network-scripts/ifcfg-bond0

```ini
DEVICE=bond0
BONDING_OPTS="mode=0 miimon=100"
BOOTPROTO=none
ONBOOT=yes
BROADCAST=192.168.0.255
IPADDR=192.168.0.180
NETMASK=255.255.255.0
NETWORK=192.168.0.0
USERCTL=no
BONDING_OPTS="mode=0 miimon=100"
```

### 第二步：修改/etc/sysconfig/network-scripts/ifcfg-ethX

vi /etc/sysconfig/network-scripts/ifcfg-eth0

```ini
DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
USERCTL=no
```

vi /etc/sysconfig/network-scripts/ifcfg-eth1

```ini
DEVICE=eth1
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
USERCTL=no
```

### 第三步：配置/etc/modprobe.conf，添加alias bond0 bonding

vi /etc/modprobe.conf

```
alias eth0 e1000e
alias eth1 e1000e
alias scsi_hostadapter mptbase
alias scsi_hostadapter1 mptspi
alias bond0 bonding
```

### 第四步：重启网络服务

`service network restart`

通过查看 `/proc/net/bonding/bond0`，查看当前是用什么 mode，如果是主备的话，当前是哪个网卡工作。

cat /proc/net/bonding/bond0

```
Ethernet ChannelBonding Driver: v3.0.3 (March 23, 2006)
Bonding Mode: fault-tolerance (active-backup)
Primary Slave:None
Currently Active Slave: eth0
MII Status: up
MII PollingInterval (ms): 100
Up Delay (ms): 0
Down Delay (ms):0
Slave Interface:eth0
MII Status: up
Link FailureCount: 0
Permanent HWaddr: 00:0c:29:01:4f:77
Slave Interface:eth1
MII Status: up
Link FailureCount: 0
Permanent HWaddr: 00:0c:29:01:4f:8b
```
