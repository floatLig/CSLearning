你知道怎么查看IP地址吗？

1. Windows上是`ipconfig`
2. Linux上可以是`ifconfig`,也可以是`ip addr`

在Linux中，我们输入`ip addr`命令，会看到一下结果，让我们来分析一下：

```bash
[zzl@localhost ~]$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:a7:21:09 brd ff:ff:ff:ff:ff:ff
    inet 192.168.142.140/24 brd 192.168.142.255 scope global noprefixroute dynamic ens33
       valid_lft 1738sec preferred_lft 1738sec
    inet6 fe80::ad55:5961:fd93:b0e7/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:3b:4d:74 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:3b:4d:74 brd ff:ff:ff:ff:ff:ff
```

### 读懂IP

这个命令显示这台机器上所有的网卡。大部分网卡都会有一个IP地址，但是，这不是必须的。

IP地址是`网卡`在网络世界中的`通讯地址`，相当于我们现实世界的门牌号码。

IPv4总共有32位，一开始IP被分为A，B，C，D，E四类。

这五类地址中，还有一类`D类是组播地址`。使用这一类地址，属于某个组的机器都可以收到。这点有点类似在公司里面大家都加入一个`邮件组`。发送邮件，加入这个组的都能收到。组播地址在后面的VXLAN协议会提到。A，B，C类的详细情况如下：

![IPv4.jpg](../../_img/IPv4.jpg)

这里面有个尴尬的事情，就是`C类地址能包含的最大主机数量实在太少了`，只有254个。当时设计的时候恐怕没想到，现在估计一个网吧都不够用吧。`而B类地址能包含的最大主机数量又太多了`。6万多台机器放在一个网络下面，一般的企业基本达不到这个规模，闲着的地址就是浪费。

所以现在常用的是`CIDR（无类型域间选路）`的解决办法，这种方式打破了原来设计的几类地址的做法，将32位的IP地址一分为二，`前面是网络号，后面是主机号`。

看到我们`ip adrr`命令的查询结果中有:`192.168.142.140/24`,这个IP地址中有一个斜杠，斜杠后面有个数字24。这种地址表示形式，就是CIDR。后面24的意思是，32位中，前24位是网络号，后8位是主机号。

伴随CIDR存在的，一个是`广播地址：192.168.142.255(主机号全是1)`，如果发送这个地址，所有的192。168.142网络里面的机器都可以收到。

另一个是`子网掩码：255.255.255.0`。IP地址和子网掩码相`与&`，就得到`了网络号：192.168.142`，剩下的`140就是主机号`。

### 一个容易“犯错”的CIDR

看一下16.158.165.91/22这个CIDR。求一下这个网络的第一个地址、子网掩码和广播地址。

- 子网掩码有22位，2 * 8 + 6 = 22，即165（1010 0101）的前6位是属于网络号，故网络号为：16.158.164.0（16.158.10100100.00000000）。
- 所以第一个网络号为：16.158.164.1
- 子网掩码为：255.255.252.0（255.255.1111 1100.0000 0000）
- 广播地址为：16.158.167.255（16.158.1010 0111.1111 1111）

### 共有IP和私有IP

在日常的工作中，几乎不用划分A类、B类或者C类，所以时间长了，很多人就忘记了这个分类，而只记得CIDR。但是有一点还是要注意的，就是`公有IP地址和私有IP地址`。

![私有IP和共有IP.jpg](../../_img/私有IP和共有IP.jpg)

我们继续看上面的表格。表格最右列是私有IP地址段。平时我们看到的数据中心里面、办公室、`家里或学校的IP地址，一般都是私有IP地址`。因为这些地址允许组织内部的`管理人员`进行分配和管理，甚至可以重复。因此，你学校的私有IP地址段和我学校`可以是一样的`。

表格中的`192.168.0.x是最常用的私有IP地址`。你家里有Wi-Fi，对应就会有一个IP地址。一般你家里地上网设备不会超过256个，所以/24基本就够了。

在整个网络里面的第一个地址192.168.0.1，往往就是你这个私有网络的出口地址。例如，你家`的WIFI路由器的地址就是192.168.0.1`，而`192.168.0.255就是广播地址`。这个发送这个地址，整个192.168.0网络里面的所有机器都可以收到。

### 细谈ip addr的参数

#### lo

lo全称是`loopback`，又称`环回接口`，往往被分配`127.0.0.1`这个地址。这个地址用于`本机通信`，经过内核处理后直接返回，不会在任何网络中出现。

使用lo,即发给本机的报文，它的路径是这么走的：

1. 应用层-->socket接口-->传输层（TCP/UDP报文）-->网络层-->back to传输层-->backto socket接口-->传回应用程序
2. `在网络层`，会在`路由表查询路由`，路由表初始化时会保存主机路由（host route，or环回路由），查询（先匹配`本机的mask`，再匹配ip，localhost路由再路由表的最顶端，最优先查到）后`发现不用转发就不用走中断到网络适配器`，所以就不用发给数据链路层了。
3. 参考资料：<https://www.zhihu.com/question/43590414>

#### <LOOPBACK,UP,LOWER_UP>

这一串参数是：net_device flags,网络设备的状态标识。

- `LOOPBACK`是环回接口
- `UP`代表网卡属于启动状态
- `LOWER_UP`表示L1是启动的，也即网线是插着的
- `BROADCAST`表示这个网卡有广播地址，可以发送广播包
- `MULTICAST`表示网卡可以发送多播包

#### mtu

MTU是最大传输单元，1500是以太网的默认值

#### qdisc

qdisc全称是`queueing discipline`，中文叫排队规则。

内核如果需要通过某个网络接口发送方数据包，它都需要按照为这个接口配置的qdisc(排队规则)把数据包加入队列。

最简单的qdisc是pfifo，它不对进入的数据包做任何处理，数据包采用FIFO（先进先出）的方式通过队列。

pfifi_fast稍微复杂点，它的队列包括三个波段（band）。在每个波段里面，使用先进先出规则。

三个波段（band）的优先级也不相同。band 0的优先级最高，band 2的最低。如果band 0里面有数据包，系统就不会处理band 1里面的数据包，band 1和band 2之间也一样。

数据包是按照服务类型（Type of Service,TOS）被分配到三个波段（band）里面的。TOS是IP头里面的一个字段，代表了当前的包是最高优先级的，还是最低优先级的。

云计算中，队列的作用很大。

#### link/ether

`link/ether`是`MAC地址`，是网卡的物理地址，用16进制，6个byte表示。

MAC地址地址像身份证，是一个唯一的标识。不同网卡放在一个网络里面的时候，可以不用担心冲突。从硬件角度，保证不同的网卡有不同的标识。

MAC地址的通信范围很小，局限在一个子网里面。

#### scope

在IP地址的后面有一个scope，对于lo来说，scope 后面是host lo，说明这张网卡仅仅可以供本机相互通信。

对于ens33来说，是global，说明这张网卡是可以对外的，可以接收来自各个地方的包。

brd代表广播地址。
