## 1、堆内存调优

#### 显式指定堆内存

**根据应用程序要求初始化堆内存**可以优化性能（推荐显示指定大小）。以下参数可指定最小、最大堆内存：

```shell
-Xms<heap size>[unit] 
-Xmx<heap size>[unit]
```

- **heap size** 表示要初始化内存的具体大小。
- **unit** 表示要初始化内存的单位。单位为**“ g”**(GB) 、**“ m”**（MB）、**“ k”**（KB）。

比如，为JVM分配最小2 GB和最大5 GB的堆内存大小：

```shell
-Xms2G -Xmx5G
```

#### 显示新生代内存

除了堆内存，新生代内存在堆内存所占比例是第二大影响因素。

默认情况下，新生代内存 的最小为 1310 *MB*，最大为 *无限制*。

**1、通过`-XX:NewSize`和`-XX:MaxNewSize`指定**

```shell
-XX:NewSize=<young size>[unit] 
-XX:MaxNewSize=<young size>[unit]
```

**2、通过`-Xmn<young size>[unit] `指定**

```shell
-Xmn256m
```

> 将新对象预留在新生代，由于 Full GC 的成本远高于 Minor GC，因此尽可能将对象分配在新生代是明智的做法，实际项目中**根据 GC 日志分析新生代空间大小分配是否合理，适当通过“-Xmn”命令调节新生代大小，最大限度降低新对象直接进入老年代的情况。**

**3、通过`-XX:NewRatio=<int>`设置新生代和老年代内存的比值**

比如，新生代与老年代所占比值为1:1，新生代占整个堆栈的 1/2，可设置为：

```shell
-XX:NewRatio=1
```

#### 显示指定永久代/元空间大小

> **从Java 8开始，如果我们没有指定 Metaspace 的大小，随着更多类的创建，虚拟机会耗尽所有可用的系统内存（永久代并不会出现这种情况）。**

**永久代**（Java 8 之前）：

```shell
// 方法区 (永久代) 初始大小
-XX:PermSize=N
// 方法区 (永久代) 最大大小
// 超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
-XX:MaxPermSize=N 
```

**元空间**（Java 8 及之后）：

```shell
// 设置 Metaspace 的初始（和最小大小）
-XX:MetaspaceSize=N
// 设置 Metaspace 的最大大小，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。
-XX:MaxMetaspaceSize=N
```

## 2、垃圾收集调优（GC）

#### 垃圾收集器

为了提高应用程序的稳定性，选择正确的 **垃圾收集算法** 至关重要。

JVM具有四种类型的*GC*实现：

- 串行垃圾收集器
- 并行垃圾收集器
- CMS垃圾收集器
- G1垃圾收集器

可以使用以下参数声明这些实现：

```shell
-XX:+UseSerialGC
-XX:+UseParallelGC
-XX:+UseParNewGC
-XX:+UseG1GC
```

#### GC 记录

为了严格监控应用程序的运行状况，我们应该始终检查JVM的**垃圾回收性能**。最简单的方法是以人类可读的格式**记录GC活动**。

使用以下参数，我们可以记录 **GC活动**：

```shell
-XX:+UseGCLogFileRotation 
-XX:NumberOfGCLogFiles=<number of log files> 
-XX:GCLogFileSize=<file size>[ unit ]
-Xloggc:/path/to/gc.log
```

