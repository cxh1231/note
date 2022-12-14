## 1、CAP 与 最终一致

`CAP` 是分布式系统的理论基石：

+ `C` 代表 **一致性** （Consistent）；
+ `A` 代表 **可用性** （Availability）；
+ `P` 代表 **分区容忍性**（Partition tolerance）。

分布式系统一般都是在不同的机器上通过网络进行交互，但是网络一般都会有断开的风险，网络断开的场景一般称之为 **网络分区** 。

当发生 **网络分区** 时，**一致性** 和 **可用性** 很难两全：

+ 发生`网络分区`，两个分布式节点无法通信，其中一个节点的修改操作无法同步到另一个节点中，两个节点中的数据不同，此时数据的 `一致性` 无法满足；
+ 发生`网络分区`，如果要满足 `一致性`，就需要暂停分布式节点对外的服务，等待网络分区恢复，两个节点正常通信数据保持一致后，再对外提供服务，但此时的 `可用性` 将无法满足。

## 2、Redis 的最终一致性

Redis 的主从节点之间，数据是异步同步的，所以分布式的 Redis 系统并不符合一致性的要求。

即：主从断开连接，主节点依旧可以正常对外提供服务，所以 Redis 符合的是可用性。

Redis 无法保证实时一致性，为保证 `最终一致性`，`从节点`会努力追赶`主节点`的数据，最终在某个时间数据将和`主节点`相同。

如果发生网络分区，主从数据将不一致。当网络恢复后，`从节点`将会通过`多种同步策略`保证与主节点数据一致。

## 3、Redis 主从复制

**主从复制** 是指将一台 Redis 服务器的数据，复制到其他的 Redis 服务器。

前者称为 `主节点`(`master`)，后者称为 `从节点`(`slave`)，且数据的复制是 **单向** 的，只能由主节点到从节点。

Redis **主从复制** 支持 `主从同步` 和 `从从同步` 两种，后者是 Redis 后续版本新增的功能，以减轻主节点的同步负担。

![image-20220911200306797](https://img.zxdmy.com/2022/202209112003406.png)

## 4、Redis 主从复制的作用

1. **数据冗余**：主从复制实现了数据的热备份，是**持久化**之外的一种数据冗余方式。
2. **故障恢复**：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复 (实际上是一种服务的冗余) 。
3. **负载均衡**：在主从复制的基础上，配合**读写分离**，可以由主节点提供写服务，由从节点提供读服务 （即写 Redis 数据时应用连接主节点，读 Redis 数据时应用连接从节点） ，**分担服务器负载**。尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高 Redis 服务器的并发量。
4. **高可用基石**： 除了上述作用以外，**主从复制是哨兵和集群能够实施的基础**，因此说主从复制是 Redis 高可用的基础。

## 5、Redis 主从复制拓扑结构

Redis 的复制**拓扑结构**可以支持**单层**或**多层**复制关系，根据拓扑复杂性可以分为以下三种：

+ **一主一从**：用于主节点出现宕机时从节点提供故障转移支持；
+ **一主多从**：对于读占比大的场景，将读命令发送至从节点，分担主节点压力，实现 **读写分离**；
+ **树状主从**：有效 **降低主节点负载** 和需要传送给从节点的数据量。

![image-20220911201514989](https://img.zxdmy.com/2022/202209112015100.png)

## 6、Redis 主从复制工作流程

Redis主从复制的 **工作流程** 大概可以分为如下几步：

![image-20220911201903730](https://img.zxdmy.com/2022/202209112019098.png)

1. **保存主节点（master）信息**：只保存主节点的`ip`和`port`；
2. **主从建立连接**：从节点（slave）发现新的主节点后，会尝试`和主节点建立网络连接`；
3. **发送ping命令**：从节点发送`ping`请求进行首次通信，主要是检测主从之间网络套接字是否可用、主节点当前是否可接受处理命令；
4. **权限验证**：如果主节点要求`密码验证`，从节点必须正确的密码才能通过验证；
5. **同步数据集**：主从复制连接正常通信后，`主节点会把持有的数据全部发送给从节点`；
6. **命令持续复制**：主节点会`持续`地把写命令发送给从节点，保证`主从数据一致性`。

## 7、Redis 主从数据同步策略

Redis在2.8及以上版本使用psync命令完成主从数据同步。

Redis 持久化有两种方式，一种是**全量持久化**（RDB），一种是**增量持久化**（AOF）。

Redis 节点之间同步方式也是两种：

+ `全量同步`（全量复制、快照同步）
+ `增量同步`（增量复制）

![image-20220911202358364](https://img.zxdmy.com/2022/202209112023565.png)

#### 全量同步

**全量同步**（全量复制）一般用于初次复制场景，将 `主节点` 的全部数据，一次性发送给 `从节点`。

`全量同步`与快照备份（RDB）类似，首先 `主节点` 执行一次 `bgsave` 命令，在后台异步保存当前数据库的数据到磁盘，然后将文件传送给`从节点`，从节点接收完毕快照文件后，先将当前内存中的数据清空，然后加载 `快照文件` 中的数据，加载完毕以后通知`主节点`继续进行`增量同步`。

当数据量较大时，会对主从节点和网络造成很大的开销。

> 在 **快照同步** 的过程中，主节点的指令buffer还在不停的增加，如果同步的时间过长或者buffer过小，会导致buffer中的指令被新的指令覆盖，这样快照同步完成后依然无法使用增量同步保证一致，此时将继续发起快照同步，这样会出现死循环的情况。
>
> 因此**需要配置一个合适的buffer大小，保证不会出现指令覆盖，防止出现死循环**。

全量复制的完整运行流程如下：

![image-20220911202639071](https://img.zxdmy.com/2022/202209112026105.png)

1. **从节点** 发送 `psync` 命令进行数据同步，由于是第一次进行复制，**从节点** 没有 **复制偏移量** 和 **主节点**的运行ID，所以发送`psync -1`；
2. **主节点** 根据 `psync-1` 解析出当前命令为 `全量复制`，回复 `+FULLRESYNC` 响应；
3. **从节点** 接收 **主节点** 的响应数据保存 `运行ID` 和 `偏移量offset`；
4. **主节点** 执行 `bgsave` 保存 `RDB` 文件到本地；
5. **主节点** 发送 `RDB` 文件给**从节点**，**从节点** 把接收的 `RDB` 文件保存在本地并直接作为**从节点**的数据文件；
6. 在 **从节点** 开始接收`RDB`快照到接收完成期间，**主节点** 仍然响应`读写命令`，因此 **主节点** 会把这期间写命令数据保存在 `复制客户端缓冲区` 内，当 **从节点** 加载完`RDB`文件后，**主节点** 再把`缓冲区`内的数据发送给**从节点**，保证主从之间数据一致
7. **从节点** 接收完 **主节点** 传送来的全部数据后会清空自身旧数据
8. **从节点** 清空数据后开始加载`RDB`文件
9. **从节点** 成功加载完`RDB`后，如果当前节点开启了`AOF`持久化功能， 它会立刻做`bgrewriteaof`操作，为了 `保证全量复制后AOF持久化文件立刻可用`。

#### 增量同步

**增量同步**（增量复制）是针对全量复制的过高开销作出的一项优化。

**增量备份** 是将 Redis 具有修改的指令存储至 AOF 日志中。增量同步与增量备份类似：

+ `增量同步` 是将 `Redis` 具有修改的指令存在`主节点`本地的内存 `Buffer` 中（一个定长的**环形数组**），然后`异步`将 `Buffer` 中的指令同步到`从节点`。
+ `从节点` 通过执行 `主节点` 同步过来的指令流，来达到和 `主节点` 一样的状态，同时向`主节点`反馈自己同步到的位置（偏移量）。
+ `Buffer` 的内存空间是有限的，无限的指令不会一直存在有限的 `Buffer` 中。
+ 如果 `Buffer` 数组满了，将会从头开始覆盖前面的内容。

如果因为网络状况不好，从节点短期内无法和主节点进行同步，当网络状况恢复时，`buffer`中没有被同步的指令很有可能已经被后续的指令覆盖了，从节点无法通过指令流来进行同步追赶保证一致，这时候就需要快照（全量）同步来保证一致性。

**增量同步** 使用 `psync {runId} {offset}` 命令实现。

当**从节点**（slave）正在复制**主节点** （master）时，如果出现网络闪断或者命令丢失等**异常**情况时，**从节点**会向主节点**要求补发丢失的命令数据**，如果**主节点**的复制积压缓冲区 **存在这部分数据则直接发送给从节点**，这样就可以`保持主从节点复制的一致性`。

**增量复制** 的运行流程如下：

![image-20220911203542349](https://img.zxdmy.com/2022/202209112035661.png)

1. 当**主从节点**之间网络出现**中断**时，如果超过`repl-timeout`时间，主节点会认为从节点故障并中断复制连接；
2. **主从连接中断**期间主节点依然响应命令，但因复制连接中断命令无法发送给从节点，不过主节点内部存在 `复制积压缓冲区`，依然可以保存最近一段时间的写命令数据，默认最大缓存`1MB`；
3. 当主从节点网络恢复后，从节点会再次连上主节点；
4. 当**主从连接恢复**后，由于**从节点**之前保存了自身`已复制的偏移量`和`主节点的运行ID`。因此会把它们当作`psync`参数发送给主节点，**要求进行部分复制操作**。
5. **主节点** 接到`psync`命令后首先核对参数`runId`是否与自身一致，如果一 致，说明之前复制的是当前主节点；之后根据参数`offset`在自身`复制积压缓冲区`查找，如果 **偏移量之后的数据存在缓冲区** 中，则对从节点发送`+CONTINUE`响应，表示可以进行部分复制。
6. **主节点**根据`偏移量`把`复制积压缓冲区`里的数据发送给**从节点**，保证主从复制进入正常状态。

#### 无盘复制同步

主节点进行快照同步时，将文件保存到本地磁盘是一个很耗时的文件IO操作，如果是非SSD磁盘存储，快照同步对系统的负载会造成较大的影响。

从 `Redis2.8.18` 开始，`Redis` 支持无盘复制，`快照同步`将**不会**进行磁盘操作，生产快照是一个遍历内存的过程，`主节点`一边遍历一边将序列化的内容发送给`从节点`，`从节点`将接收的内容存在磁盘文件中，最后进行一次性加载。

> 当有新的从节点加入时，**将进行一次快照同步，后续则是增量同步**。

## 8、Redis 主从复制存在的问题

主从复制虽好，但也存在一些问题：

+ 一旦**主节点出现故障**，需要 **手动将一个从节点晋升为主节点**，同时需要修改应用方的主节点地址，还需要命令其他从节点去复制新的主节点，整个过程都需要人工干预。
+ **主节点**的 **写能力** 受到单机的限制。
+ **主节点**的**存储能力**受到单机的限制。

> 第一个问题是Redis的高可用问题，第二、三个问题属于Redis的分布式问题。
