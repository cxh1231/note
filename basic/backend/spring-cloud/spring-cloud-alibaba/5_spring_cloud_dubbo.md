## 1、RPC 概述

**分布式**是促使 `RPC` 诞生的领域，**RPC 是一种编程模型，并没有规定你具体要怎样实现**，无论使用 HTTP 或是 RMI 都是可以的。

在**单体应用**中，各个方法和函数都在相同的地址空间中，方法之间的调用，直接通过虚拟机的**栈**和**参数栈**即可实现。

而**分布式应用**中，也可以将各个服务共享的功能方法、函数独立出来，让其他服务去调用。

所以，**RPC 要解决的两个问题**

- 解决分布式系统中，服务之间的调用问题
- 远程调用时，要能够像本地调用一样方便，让调用者感知不到远程调用的逻辑

实际情况下，**RPC 很少用到 HTTP 协议来进行数据传输**，毕竟使用文本传输的应用层协议，是在浪费资源。

相反，可以直接使用**二进制传输**，比如直接用 Java 的 Socket 协议进行传输。

> `RPC` 可以基于`HTTP`协议实现，也可以直接在`TCP`协议上实现。

RPC 传输过程中，最重要的是 **序列化** 和 **反序列化** 。因为数据传输的数据包必须是**二进制**的，即把 Java 对象**序列化**为二进制格式，传给 Server 端，Server 端接收到之后，再**反序列化**为 Java 对象。

**RPC 是面向过程**，**Restful 是面向资源**，并且使用了 HTTP 动词。从这个维度上看，Restful 风格的 URL 在表述的精简性、可读性上都要更好。

## 2、Dubbo 概述

`Apache Dubbo` (incubating) |ˈdʌbəʊ| 是一款高性能、轻量级的开源 Java `RPC` 分布式服务框架，它提供了三大核心能力：

+ **面向接口的远程方法调用**
+ 智能容错和负载均衡
+ 服务自动注册和发现

`Dubbo` 最大的特点是按照分层的方式来架构，使用这种方式可以使各个层之间解耦合（或者最大限度地松耦合）。

从服务模型的角度来看，`Dubbo` 采用的是一种非常简单的模型，要么是**提供方提供服**务，要么是**消费方消费服务**，所以基于这一点可以抽象出两个角色：

+ 服务提供方（Provider）
+ 服务消费方（Consumer）

## 3、Dubbo 架构

Dubbo 架构的节点组成以及示意图如下：

|   节点    | 角色说明                               |
| :-------: | :------------------------------------- |
| Provider  | 暴露服务的服务提供方                   |
| Consumer  | 调用远程服务的服务消费方               |
| Registry  | 服务注册与发现的注册中心               |
|  Monitor  | 统计服务的调用次数和调用时间的监控中心 |
| Container | 服务运行容器                           |

示意图：

![image-20220923162802457](https://img.zxdmy.com/2022/202209231628400.png)

调用关系说明：

- 服务容器负责启动，加载，运行服务提供者
- 服务提供者在启动时，向注册中心注册自己提供的服务
- 服务消费者在启动时，向注册中心订阅自己所需的服务
- 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者
- 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用
- 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心

## 4、Dubbo 序列化

Dubbo `RPC` 是 Dubbo 体系中最核心的一种高性能、高吞吐量的远程调用方式，可以称之为多路复用的 `TCP` 长连接调用：

- **长连接：** 避免了每次调用新建 TCP 连接，提高了调用的响应速度
- **多路复用：** 单个 TCP 连接可交替传输多个请求和响应的消息，降低了连接的等待闲置时间，从而减少了同样并发数下的网络连接数，提高了系统吞吐量

Dubbo RPC 主要用于两个 Dubbo 系统之间的远程调用，特别适合高并发、小数据的互联网场景。

而序列化对于远程调用的响应速度、吞吐量、网络带宽消耗等同样也起着至关重要的作用，是我们提升分布式系统性能的最关键因素之一。

Dubbo 中支持的序列化方式：

- **dubbo 序列化：** 阿里尚未开发成熟的高效 Java 序列化实现，阿里不建议在生产环境使用它
- **hessian2 序列化：** hessian 是一种跨语言的高效二进制序列化方式。但这里实际不是原生的 hessian2 序列化，而是阿里修改过的 hessian lite，它是 dubbo RPC 默认启用的序列化方式
- **json 序列化：** 目前有两种实现，一种是采用的阿里的 fastjson 库，另一种是采用 dubbo 中自己实现的简单 json 库，但其实现都不是特别成熟，而且 json 这种文本序列化性能一般不如上面两种二进制序列化。
- **java 序列化：** 主要是采用 JDK 自带的 Java 序列化实现，性能很不理想。

在通常情况下，这四种主要序列化方式的性能从上到下依次递减。对于 dubbo RPC 这种追求高性能的远程调用方式来说，实际上只有 1、2 两种高效序列化方式比较般配，而第 1 个 dubbo 序列化由于还不成熟，所以实际只剩下 2 可用，所以 dubbo RPC 默认采用 hessian2 序列化。

但 hessian 是一个比较老的序列化实现了，而且它是跨语言的，所以不是单独针对 Java 进行优化的。而 dubbo RPC 实际上完全是一种 Java to Java 的远程调用，其实没有必要采用跨语言的序列化方式（当然肯定也不排斥跨语言的序列化）。

最近几年，各种新的高效序列化方式层出不穷，不断刷新序列化性能的上限，最典型的包括：

- 专门针对 Java 语言的：**Kryo**，FST 等等
- 跨语言的：Protostuff，ProtoBuf，**Thrift**，Avro，MsgPack 等等

这些序列化方式的性能多数都显著优于 hessian2（甚至包括尚未成熟的 dubbo 序列化），有鉴于此，我们为 dubbo 引入 `Kryo` 和 FST 这两种高效 Java 序列化实现，来逐步取代 hessian2。

其中，**`Kryo` 是一种非常成熟的序列化实现**，已经在 Twitter、Groupon、Yahoo 以及多个著名开源项目（如 Hive、Storm）中广泛的使用。

而 FST 是一种较新的序列化实现，目前还缺乏足够多的成熟使用案例。

> **注意：** 在面向生产环境的应用中，目前更优先选择 `Kryo`

![image-20220923163856936](https://img.zxdmy.com/2022/202209231639036.png)

## 5、Dubbo 负载均衡

在集群负载均衡时，Dubbo 提供了多种均衡策略：

1. **随机**（`Random LoadBalance`）：按权重设置随机概率，在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。
2. **轮循**（`RoundRobin LoadBalance`）：按公约后的权重设置轮询比率，存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。
3. **最少活跃调用数**（`LeastActive LoadBalance`）：相同活跃数的随机，活跃数指调用前后计数差，使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。
4. **一致性 Hash**（`ConsistentHash LoadBalance`）：相同参数的请求总是发到同一提供者。当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。

> 缺省为 `random` 随机调用。

下面是简单的使用示例：

- 修改 `dubbo-provider` 项目的负载均衡策略，默认的负载均衡策略是 **随机**，我们修改为 **轮循**，可配置的值分别是：`random`，`roundrobin`，`leastactive`，`consistenthash`

```yaml
dubbo:
  provider:
    loadbalance: roundrobin
```

- 修改 `dubbo-provider` 的协议端口为 20880 和 20881，并启动多个实例，IDEA 中依次点击 **Run** -> **Edit Configurations** 并勾选 **Allow parallel run** 以允许 IDEA 多实例运行项目

![img](https://img.zxdmy.com/2022/202209231642661.png)

- Nacos Server 控制台可以看到 `dubbo-provider` 有 2 个实例

![img](https://img.zxdmy.com/2022/202209231642278.png)

- 修改 `dubbo-provider` 项目的 `EchoServiceImpl` 中的测试方法

```java
package com.funtl.apache.dubbo.provider.service;

import com.funtl.apache.dubbo.provider.api.EchoService;
import org.apache.dubbo.config.annotation.Service;
import org.springframework.beans.factory.annotation.Value;

@Service(version = "1.0.0")
public class EchoServiceImpl implements EchoService {
    
    @Value("${dubbo.protocol.port}")
    private String port;
    
    @Override
    public String echo(String string) {
        return "Echo Hello Dubbo " + string + " i am from port: " + port;
    }
    
}
```

- 重启服务，通过浏览器访问 [http://localhost:8080/echo/hi](http://www.qfdmy.com/wp-content/themes/quanbaike/go.php?url=aHR0cDovL2xvY2FsaG9zdDo4MDgwL2VjaG8vaGk=) ，反复刷新浏览器，浏览器交替显示

```
Echo Hello Dubbo hi i am from port: 20880
Echo Hello Dubbo hi i am from port: 20881
```

