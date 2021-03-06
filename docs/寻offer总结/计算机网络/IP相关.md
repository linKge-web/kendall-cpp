
- [ARP请求报文组成](#arp请求报文组成)
  - [ARP是地址解析协议工作原理](#arp是地址解析协议工作原理)
- [IP（网咯层）和 MAC（数据链路层）之间的区别和联系](#ip网咯层和-mac数据链路层之间的区别和联系)
- [IP 地址的分类](#ip-地址的分类)
- [广播地址作用](#广播地址作用)
  - [广播地址分类](#广播地址分类)
  - [IP 分类的缺点](#ip-分类的缺点)
    - [无分类地址 CIDR](#无分类地址-cidr)
    - [怎么划分网络号和主机号](#怎么划分网络号和主机号)
    - [子网掩码](#子网掩码)
    - [为什么要分离网络号和主机号](#为什么要分离网络号和主机号)
  - [怎么进行子网划分](#怎么进行子网划分)
- [介绍一下`ping`的过程，分别用到了哪些协议](#介绍一下ping的过程分别用到了哪些协议)

------------

## ARP请求报文组成

![](./img/01tcpip04.png)

- 硬件类型：定义物理地址的类型，它的值为`1`表示`MAC`地址
- 协议类型字段表示要映射的映射的协议地址类型，它的值 `0x800`, 表示IP地址。
- 硬件地址长度字段和协议地址长度字段，单位是字节。对`MAC`地址类说，其长度是`6`，`IPv4`地址来说，其长度为`4`.
- 操作字段支出`4`种操作字段，`ARP`请求（值为`1`）、`ARP` 应答（值为`2`）、`RARP` 请求（值为`3`）和 `RARP` 应答（值为`4`）
- 最后`4`个字段指定通信双方的以太网地址和`IP`地址,发送端填充除目的端口`IP`地址是自己，就把自己的以太网地址填进去，然后交换两个目的端地址和两个发送端地址，以构建`ARP`应答返回值。


### ARP是地址解析协议工作原理


1：首先，每个主机都会在自己的ARP缓冲区中建立一个ARP列表，以表示IP地址和MAC地址之间的对应关系。

2：当源主机要发送数据时，首先检查ARP列表中是否有对应IP地址的目的主机的MAC地址，如果有，则直接发送数据，如果没有，就向本网段的所有主机发送ARP数据包，该数据包包括的内容有：源主机 IP地址，源主机MAC地址，目的主机的IP 地址。

3：当本网络的所有主机收到该ARP数据包时，首先检查数据包中的IP地址是否是自己的IP地址，如果不是，则忽略该数据包，如果是，则首先从数据包中取出源主机的IP和MAC地址写入到ARP列表中，如果已经存在，则覆盖，然后将自己的MAC地址写入ARP响应包中，告诉源主机自己是它想要找的MAC地址。

4：源主机收到ARP响应包后。将目的主机的IP和MAC地址写入ARP列表，并利用此信息发送数据。如果源主机一直没有收到ARP响应数据包，表示ARP查询失败。

广播发送ARP请求，单播发送ARP响应。

## IP（网咯层）和 MAC（数据链路层）之间的区别和联系

IP 层的作用是负责两个主机之间的通信，而 MAC 的作用是实现**直连**的两个设备之间通信，而 IP 则负责在 **没有直连** 的两个网络之间进行通信传输。

> IP 最主要的作用是在复杂网络中 将数据包发送给最终的 目标主机

## IP 地址的分类

IP 地址分类成了 5 种类型，分别是 A 类、B 类、C 类、D 类、E 类

![](https://cdn.jsdelivr.net/gh/kendall-cpp/blogPic@main/寻offer总结02/IP地址地址的分类.60s9tr9hwuw0.png)

其中对于 A、B、C 类主要分为两个部分，分别是网络号和主机号.

因为在 IP 地址中，有两个 IP 是特殊的，分别是主机号全为 1 和 全为 0 地址。

![](https://cdn.jsdelivr.net/gh/kendall-cpp/blogPic@main/寻offer总结02/IP地址地址的02.33bso2jske00.png)

- 主机号全为 1 指定某个网络下的所有主机，用于广播
- 主机号全为 0 指定某个网络

而 D 类和 E 类地址是没有主机号的，所以不可用于主机 IP，D 类常被用于多播，E 类是预留的分类，暂时未使用。

> 多播用于将包发送给特定组内的所有主机。

## 广播地址作用

广播地址用于在同一个链路中相互连接的主机之间发送数据包。

当主机号全为 1 时，就表示该网络的广播地址。例如把 `172.20.0.0/16` 用二进制表示如下：

`10101100.00010100.00000000.00000000`

将这个地址的主机部分全部改为 1，则形成广播地址：

`10101100.00010100.11111111.11111111`

再将这个地址用十进制表示，则为 `172.20.255.255`。


### 广播地址分类

- 在本网络内广播的叫做**本地广播**。例如网络地址为 `192.168.0.0/24` 的情况下，广播地址是 `192.168.0.255` 。因为这个广播地址的 IP 包会被路由器屏蔽，所以不会到达 `192.168.0.0/24` 以外的其他链路上。
- 在不同网络之间的广播叫做**直接广播**。例如网络地址为 `192.168.0.0/24` 的主机向 `192.168.1.255/24` 的目标地址发送 IP 包。收到这个包的路由器，将数据转发给 192.168.1.0/24，从而使得所有 192.168.1.1~192.168.1.254 的主机都能收到这个包（由于直接广播有一定的安全问题，多数情况下会在路由器上设置为不转发。）。
### IP 分类的缺点

- 同一网络下没有地址层次，比如一个公司里用了 B 类地址，但是可能需要根据生产环境、测试环境、开发环境来划分地址层次，而这种 IP 分类是没有地址层次划分的功能，所以这就缺少地址的灵活性。

- A、B、C 类有个尴尬处境，就是不能很好的与现实网络匹配。
  - C 类地址能包含的最大主机数量实在太少了，只有 254 个，估计一个网吧都不够用。
  - 而 B 类地址能包含的最大主机数量又太多了，6 万多台机器放在一个网络下面，一般的企业基本达不到这个规模，闲着的地址就是浪费。
  - 
这两个缺点，都可以在 CIDR 无分类地址解决。

#### 无分类地址 CIDR

正因为 IP 分类存在许多缺点，所以后面提出了无分类地址的方案，即 CIDR。

这种方式不再有分类地址的概念，32 比特的 IP 地址被划分为两部分，前面是**网络号**，后面是**主机号**。

#### 怎么划分网络号和主机号

表示形式 `a.b.c.d/x`，其中 `/x `表示前 `x` 位属于网络号， `x` 的范围是 `0 ~ 32`，这就使得 IP 地址更加具有灵活性。

比如 `10.100.122.2/24`，这种地址表示形式就是 CIDR，`/24` 表示前 `24` 位是网络号，剩余的 `8` 位是主机号。

#### 子网掩码

还有另一种划分网络号与主机号形式，那就是**子网掩码**，掩码的意思就是掩盖掉主机号，剩余的就是网络号。

将子网掩码和 IP 地址按位计算 AND (`&`)，就可得到网络号

![](https://cdn.jsdelivr.net/gh/kendall-cpp/blogPic@main/寻offer总结02/IP划分01.352m6u36dcw0.png)


#### 为什么要分离网络号和主机号

因为两台计算机要通讯，首先要判断是否处于同一个广播域内，即网络地址是否相同。如果网络地址相同，表明接受方在本网络上，那么可以把数据包直接发送到目标主机。

路由器寻址工作中，也就是通过这样的方式来找到对应的网络号的，进而把数据包转发给对应的网络内。

![](https://cdn.jsdelivr.net/gh/kendall-cpp/blogPic@main/寻offer总结02/IP划分02.45dtok4gssw0.png)

### 怎么进行子网划分

通过子网掩码划分出网络号和主机号，那实际上子网掩码还有一个作用，那就是划分子网。

子网划分实际上是将主机地址分为两个部分：子网网络地址和子网主机地址。形式如下：

![](https://cdn.jsdelivr.net/gh/kendall-cpp/blogPic@main/寻offer总结02/子网划分.115ndqdhbui8.png)

- 未做子网划分的 ip 地址：网络地址＋主机地址
- 做子网划分后的 ip 地址：网络地址＋（子网网络地址＋子网主机地址）

> 具体分析：https://blog.csdn.net/qq_34827674/article/details/105930929


## 介绍一下`ping`的过程，分别用到了哪些协议

详见：[`Ping`原理与`ICMP`协议](https://www.cnblogs.com/Akagi201/archive/2012/03/26/2418475.html)

`ping`命令基于网络层的命令，是基于`ICMP`协议工作的。 

`ICMP`:网络控制报文协议

* 首先，`ping`命令会构建一个`ICMP`请求数据包，然后由`ICMP`协议将这个数据包连同目的`IP`地址源的`IP`地址一起交给`IP`协议。
* 然后`IP`协议就会构建一个`IP`数据报，并且在映射表中查找目的`IP`对应的`mac`地址，将其交给数据链路层。
* 然后数据链路层就会构建一个数据帧，附上源`mac`地址和目的`mac`地址发送出去。

* 目的主机接收到数据帧后，就会检查包上的`mac`地址与本机`mac`是否相符，如果相符，就接收并把其中的信息提取出来交给`IP`协议，`IP`协议就会将其中的信息提取出来交给`ICMP`协议。然后构建一个`ICMP`应答包，用相同的过程发送回去。

> 具体参考：https://blog.csdn.net/qq_34827674/article/details/105106807