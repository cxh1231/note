## 1、单体应用的局限性

+ 随着时间的推移，业务正价，导致**应用臃肿**，进而导致**开发、交付困难**，BUG 难以排查（滚雪球）
+ 应用程序的规模变大，**启动时间增长**，单体应用将**减缓发展**；
+ 复杂的单体应用本身就是**持续部署的障碍**；
+ 不同模块存在 **资源需求冲突** 时，单体应用可能**难以扩展**；
+ 单体应用的 **可靠性** 无法保证；
+ 单体应用使得采用新框架和语言变得非常困难。

## 2、微服务概述

### 2.1 微服务解决复杂问题

微服务的基本思路是将应用程序分解成一套较小的互连服务。一个服务通常实现了一组不同的特性或功能，例如订单管理、客户管理等。每一个微服务都是一个迷你应用。

一些微服务会暴露一个供其他微服务或应用客户端消费的 API。其他微服务可能实现了一个 web UI。在运行时，每个实例通常是一个云虚拟机 (virtual machine， VM) 或者一个 Docker 容器。

应用程序的每个功能区域现在都由自己的微服务实现。

![image-20220923153845443](https://img.zxdmy.com/2022/202209231538294.png)

**微服务架构是一种架构思想，旨在通过将功能分解到各个离散的服务中以实现对解决方案的解耦**。

它的主要作用是将功能分解到离散的各个服务当中，从而降低系统的耦合性，并提供更加灵活的服务支持。

**概念**：把一个大型的单个应用程序和服务拆分为数个甚至数十个的支持微服务，它可扩展单个组件而不是整个的应用程序堆栈，从而满足服务等级协议。

**定义**：微服务围绕业务领域组件来创建应用，这些应用可独立地进行开发、管理和迭代。在分散的组件中使用云架构和平台式部署、管理和服务功能，使产品交付变得更加简单。

**本质：** 用一些功能比较明确、业务比较精练的服务去解决更大、更实际的问题。

总之，使用微服务，需要遵循三个标准：

- 提高敏捷性：及时响应业务需求，促进企业发展
- 提升用户体验：提升用户体验，减少用户流失
- 降低成本：降低增加产品、客户或业务方案的成本

### 2.2 微服务的特征

#### 官方的定义

- 一系列的独立的服务共同组成系统
- 单独部署，跑在自己的进程中
- 每个服务为独立的业务开发
- 分布式管理
- 非常强调隔离性

#### 大概的标准

- 分布式服务组成的系统
- 按照业务，而不是技术来划分组织
- 做有生命的产品而不是项目
- 强服务个体和弱通信（ Smart endpoints and dumb pipes ）
- 自动化运维（ DevOps ）
- 高度容错性
- 快速演化和迭代

### 2.3 微服务的优点

+ 微服务解决了复杂问题，将庞大的单体应用分解成一套服务，实现模块化；
+ 微服务架构使得每个服务都可以由一个独立的团队开发；
+ 微服务架构模式可以实现每个微服务独立部署；
+ 微服务架构模式使得每个服务能够独立扩展。

### 2.4 微服务的缺点

+ 由于微服务是一个分布式系统，其使得整体变得复杂。比如发者需要选择和实现基于消息或者 RPC 的进程间通信机制，或目标请求可能很慢或者不可用，开发者必须要编写代码来处理局部故障……
+ 微服务的另一个挑战是分区数据库架构。
+ 测试微服务应用程序也很复杂。
+ 实现跨越多服务变更难度大，需要协调多服务的变更
+ 部署基于微服务的应用程序也是相当复杂的。

## 3、微服务需要解决的问题

实际的微服务应用，需要解决以下问题：

- 服务很多，客户端怎么访问（提供对外网关）
- 这么多服务，服务之间如何通信（HTTP or RPC）
- 这么多服务，如何治理（服务的注册和发现）
- 服务挂了，如何解决（备份方案，应急处理机制）

总结一下，即：

- API Gateway
- 服务间调用
- 服务发现
- 服务容错
- 服务部署
- 数据调用

### 3.1 客户端如何访问各个服务

按功能拆分成独立的服务后，一般每个Java虚拟机上跑的是互相独立的进程，客户端如何访问他们？

后台有 N 个服务，前台就需要记住管理 N 个服务，一个服务 **下线**、**更新**、**升级**，前台就要重新部署，这明显不服务我们拆分的理念，特别当前台是移动应用的时候，通常业务变化的节奏更快。

另外，N 个小服务的调用也是一个不小的网络开销。还有一般微服务在系统内部，通常是无状态的，用户登录信息和权限管理最好有一个统一的地方维护管理（OAuth）。

所以，一般在后台 N 个服务和 UI 之间一般会一个代理或者叫 `API Gateway`，他的作用包括：

- 提供统一服务入口，让微服务对前台透明
- 聚合后台的服务，节省流量，提升性能
- 提供安全，过滤，流控等API管理功能

其实这个 `API Gateway` 可以有很多广义的实现办法，可以是一个软硬一体的盒子，也可以是一个简单的 MVC 框架，甚至是一个 `Node.js` 的服务端。

他们最重要的作用是为前台（通常是移动应用）提供后台服务的聚合，提供一个统一的服务出口，解除他们之间的耦合，不过 `API Gateway` 也有可能成为 **单点故障** 点或者性能的瓶颈。

![image-20220923154701051](https://img.zxdmy.com/2022/202209231547459.png)

### 3.2 服务之间如何通信

所有的微服务都是独立的 Java 进程跑在独立的虚拟机上，所以服务间的通信就是 IPC（Inter Process Communication），已经有很多成熟的方案。现在基本最通用的有两种方式：

+ 同步调用
+ 异步消息调用

#### 同步调用

**同步调用**比较简单，一致性强，但是容易出调用问题，性能体验上也会差些，特别是调用层次多的时候。

实现同步调用的方式主要有：

- REST（JAX-RS，Spring Boot）
- RPC（Thrift，Dubbo）

一般 `REST` 基于 HTTP，更容易实现，更容易被接受，服务端实现技术也更灵活些，各个语言都能支持，同时能跨客户端，对客户端没有特殊的要求，只要封装了 HTTP 的 SDK 就能调用，所以相对使用的广一些。

`RPC` 也有自己的优点，传输协议更高效，安全更可控，特别在一个公司内部，如果有统一个的开发规范和统一的服务框架时，开发效率优势更明显些。

#### 异步消息调用

**异步消息** 的方式在分布式系统中有特别广泛的应用，既能减低调用服务之间的耦合，又能成为调用之间的缓冲，确保消息积压不会冲垮被调用方，同时能保证调用方的服务体验，继续干自己该干的活，不至于被后台性能拖慢。

实现异步消息的方案主要有：

- Kafka
- Notify
- MessageQueue

不过需要付出的代价是一致性的减弱，需要接受数据 **最终一致性**；

还有就是后台服务一般要实现 **幂等性**，因为消息发送出于性能的考虑一般会有重复（保证消息的被收到且仅收到一次对性能是很大的考验）；

最后就是必须引入一个独立的 `Broker`。

![image-20220923154952643](https://img.zxdmy.com/2022/202209231549548.png)

### 3.3 多个服务，如何治理

在微服务架构中，一般每一个服务都是有多个拷贝，来做负载均衡。一个服务随时可能下线，也可能应对临时访问压力增加新的服务节点。服务之间如何相互感知？服务如何管理？

这就是服务发现的问题了。一般有两类做法，也各有优缺点。基本都是通过 Zookeeper 等类似技术做服务注册信息的分布式管理。当服务上线时，服务提供者将自己的服务信息注册到 ZK（或类似框架），并通过心跳维持长链接，实时更新链接信息。服务调用者通过 ZK 寻址，根据可定制算法，找到一个服务，还可以将服务信息缓存在本地以提高性能。当服务下线时，ZK 会发通知给服务客户端。

#### 基于客户端的服务注册与发现

优点是架构简单，扩展灵活，只对服务注册器依赖。缺点是客户端要维护所有调用服务的地址，有技术难度，一般大公司都有成熟的内部框架支持，比如 Dubbo。

![img](https://img.zxdmy.com/2022/202209231555387.png)

#### 基于服务端的服务注册与发现

优点是简单，所有服务对于前台调用方透明，一般在小公司在云服务上部署的应用采用的比较多。

![img](https://img.zxdmy.com/2022/202209231555941.png)

### 3.4 服务挂了，如何解决

分布式最大的特性就是 **网络是不可靠的**。

通过微服务拆分能降低这个风险，如果没有特别的保障，结局肯定是噩梦。

所以当我们的系统是由一系列的服务调用链组成的时候，我们必须确保任一环节出问题都不至于影响整体链路。

相应的手段有很多：

- 重试机制
- 限流
- 熔断机制
- 负载均衡
- 降级（本地缓存）

## 4、Spring Cloud 核心组件

![image-20220912215633376](https://img.zxdmy.com/2022/202209122156978.png)