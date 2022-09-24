## 1、网络体系结构

计算机网络中常见的体系结构主要有以下三种：

|     结构名称      | 层数 |                      简介                       |
| :---------------: | :--: | :---------------------------------------------: |
|  `OSI` 体系结构   | 七层 |           理想化结构，效率低，未采用            |
|   原理体系结构    | 五层 | 结合了`OSI`和 `TCP/IP` 结构的优点，理论学习结构 |
| `TCP/IP` 体系结构 | 四层 |                 真实采用的结构                  |

![image-20220829160605330](https://img.zxdmy.com/2022/202209011126451.png)

## 2、OSI 七层结构

- **应用层** ：为特定应用程序提供数据传输服务，例如 HTTP、DNS 等协议。数据单位为报文。
- *表示层* ：数据压缩、加密以及数据描述，这使得应用程序不必关心在各台主机中数据内部格式不同的问题。
- *会话层* ：建立及管理会话
- **传输层** ：为进程提供通用数据传输服务。由于应用层协议很多，定义通用的传输层协议就可以支持不断增多的应用层协议。运输层包括两种协议：传输控制协议 TCP，提供面向连接、可靠的数据传输服务，数据单位为报文段；用户数据报协议 UDP，提供无连接、尽最大努力的数据传输服务，数据单位为用户数据报。TCP 主要提供完整性服务，UDP 主要提供及时性服务。
- **网络层** ：为主机提供数据传输服务。而传输层协议是为主机中的进程提供数据传输服务。网络层把传输层传递下来的报文段或者用户数据报封装成分组。
- **数据链路层** ：网络层针对的还是主机之间的数据传输服务，而主机之间可以有很多链路，链路层协议就是为同一链路的主机提供数据传输服务。数据链路层把网络层传下来的分组封装成帧。
- **物理层** ：考虑的是怎样在传输媒体上传输数据比特流，而不是指具体的传输媒体。物理层的作用是尽可能屏蔽传输媒体和通信手段的差异，使数据链路层感觉不到这些差异。

|      各层      |            设备            |               协议                |     数据单元      |
| :------------: | :------------------------: | :-------------------------------: | :---------------: |
|   **应用层**   |          应用程序          |    DHCP、FTP、HTTP、SMTP、RPC     |                   |
|     表示层     |             ——             |                ——                 |        ——         |
|     会话层     |             ——             |                ——                 |        ——         |
|   **传输层**   |         进程和端口         |             TCP、UDP              | 数据段（Segment） |
|   **网络层**   | 路由器，防火墙、多层交换机 | `IP`、`ARP`、RARP、`ICMP`、ICMPv6 | 数据包（Packet）  |
| **数据链路层** |     网卡，网桥，交换机     |           CSMA/CD、ARQ            |    帧（Frame）    |
|   **物理层**   | 中继器，集线器、网线、HUB  |                ——                 |    比特（Bit）    |

![image-20220916155345855](https://img.zxdmy.com/2022/202209161553211.png)

> 传输层协议和网络层协议有什么区别：
>
> **网络层**协议负责提供**主机**间的逻辑通信；**传输层**协议负责提供**进程**间的逻辑通信。

## 3、TCP/IP 体系结构

![image-20220829161927784](https://img.zxdmy.com/2022/202208291619526.png)

它只有四层，相当于五层协议中数据链路层和物理层合并为网络接口层。

TCP/IP 体系结构不严格遵循 OSI 分层概念，应用层可能会直接使用 IP 层或者网络接口层。

## 4、数据传输

在**向下**的过程中，需要**添加下层协议所需要的首部或者尾部**，而在**向上**的过程中不断**拆开首部和尾部**。

路由器只有下面三层协议，因为路由器位于网络核心中，不需要为进程或者应用程序提供服务，因此也就不需要传输层和应用层。

![image-20220916155437821](https://img.zxdmy.com/2022/202209161554588.png)