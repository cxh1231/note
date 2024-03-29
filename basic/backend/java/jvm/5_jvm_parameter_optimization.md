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

还有其他配置命令：

```shell
-XX:NewSize=n # 设置年轻代大小
-XX:NewRatio=n # 设置年轻代和年老代的比值。如：为3表示年轻代和年老代比值为1：3，年轻代占整个年轻代年老代和的1/4
-XX:SurvivorRatio=n # 年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如3表示Eden： 3 Survivor：2，一个Survivor区占整个年轻代的1/5
-XX:MaxPermSize=n # 设置持久代大小
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

JVM具有四种类型的`GC`实现：

- 串行垃圾收集器
- 并行垃圾收集器
- CMS垃圾收集器
- G1垃圾收集器

可以使用以下参数声明这些实现：

```shell
-XX:+UseSerialGC  		# 设置串行收集器
-XX:+UseParallelGC		# 设置并行收集器
-XX:+UseParNewGC		
-XX:+UseG1GC			# 设置 G1 收集器
```

当然，还可以对并发收集器进行设置：

```shell
-XX:ParallelGCThreads=n 	# 设置并行收集器收集时使用的CPU数。并行收集线程数
-XX:MaxGCPauseMillis=n 		# 设置并行收集最大的暂停时间（如果到这个时间了，垃圾回收器依然没有回收完，也会停止回收）
-XX:GCTimeRatio=n			# 设置垃圾回收时间占程序运行时间的百分比。公式为：1/(1+n)
-XX:+CMSIncrementalMode		# 设置为增量模式。适用于单CPU情况
-XX:ParallelGCThreads=n		# 设置并发收集器年轻代手机方式为并行收集时，使用的CPU数。并行收集线程数
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

## 3、各种疑难杂症解决方案

### 3.1 JVM 疑难杂症与调优的基本流程

1. 分析系统运行情况：
   1. 实时监控：Linux 命令、JDK 命令、实时监控平台
   2. 事后分析：GC 日志分析、堆转储快照分析
2. 确定 JVM 调优量化目标
   1. 内存使用率、停顿时间、回收频率等
3. 确定 JVM 调优参数
4. 依次确定调优内存、延迟、吞吐量等指标
5. 对比观察调优前后的差异
6. 分析和调整
7. 找到合适的参数
8. 持续跟踪监控

### 3.2 频繁 Minor GC 的解决方案

频繁发生**新生代的垃圾收集**，可能原因与解决方案如下。

#### 原因：

通常情况下，由于**新生代空间较小**，`Eden`区很快被填满，

#### 解决方案：

**增大新生代空间**`-Xmn` 来降低Minor GC的频率。

### 3.3 频繁 Full GC 的解决方案

频繁发生 **整体的垃圾收集**，可能原因与解决方案如下。

#### 原因：

`Full GC` 可能由以下几种情况导致：

+ **大对象**：系统一次性加载了过多数据到内存中（比如SQL查询未做分页），导致大对象进入了老年代；
+ **内存泄漏**：频繁创建了大量对象，但是无法被回收（比如`IO`对象使用完后未调用`close`方法释放资源），先引发`FGC`，最后导致`OOM`；
+ 程序频繁生成一些 **长生命周期的对象**，当这些对象的存活年龄超过分代年龄时便会进入老年代，最后引发`FGC`；
+ 程序`BUG`；
+ 代码中显式调用了 `gc` 方法，包括自己的代码甚至框架中的代码；
+ **JVM参数设置问题**：包括总内存大小、新生代和老年代的大小、Eden区和S区的大小、元空间大小、垃圾回收算法等。

#### 解决方案：

1. 查看监控，以了解出现问题的**时间点**以及当前**FGC的频率**（可对比正常情况看频率是否正常）
2. 了解该时间点之前有没有程序上线、基础组件升级等情况。
3. 了解`JVM`的**参数设置**，包括：堆空间各个区域的大小设置，新生代和老年代分别采用了哪些垃圾收集器，然后分析JVM参数设置是否合理。
4. 再对步骤1中列出的可能原因做排除法，其中元空间被打满、内存泄漏、代码显式调用gc方法比较容易排查。
5. 针对**大对象**或者**长生命周期对象**导致的`FGC`，可通过 `jmap -histo` 命令并结合`dump`堆内存文件作进一步分析，需要先定位到可疑对象。
6. 通过可疑对象定位到具体代码再次分析，这时候要结合GC原理和JVM参数设置，弄清楚可疑对象是否满足了进入到老年代的条件才能下结论。

### 3.4 CPU 占用过高排查方案

#### 原因：

`CPU` 高一定是 **某个程序长期占用了CPU资源**。

#### 解决方案：

1. 找出占用 CPU 资源高的**进程**：使用 `top` 命令
2. 找到该进程中，占用 CPU 资源高的**线程ID**：使用 `top -Hp 进程ID` 命令
3. 找到该线程`PID`后，打印**堆栈信息**：
   1. `printf "%x\n" PID` 命令：把线程ID转换为16进制
   2. `jstack PID` 打印出进程的所有线程信息，从打印出来的线程信息中找到上一步转换为16进制的**线程ID对应的线程信息**。
4. 根据**堆栈信息**定位到具体的**业务方法**中，从**代码逻辑**中找出问题所在。

最后一步主要是查看是否有线程长时间的`watting` 或`blocked`，如果线程长期处于`watting`状态下， 关注`watting on xxxxxx`，说明线程在等待这把锁，然后根据锁的地址找到持有锁的线程。

### 3.5 内存占用过高排查方案

#### 原因：

内存飚高如果是发生在java进程上，一般是因为**创建了大量对象**所导致，持续飚高说明**垃圾回收跟不上对象创建的速度**，或者**内存泄露**导致对象无法回收。

#### 解决方案：

1. 先观察垃圾回收的情况：
   1. `jstat -gc PID 1000` 命令：查看`GC`次数，时间等信息，每隔一秒打印一次
   2. `jmap -histo PID | head -20` 查看堆内存占用空间最大的前20个对象类型，可初步查看是哪个对象占用了内存。
2. 导出堆内存文件快照：命令 `jmap -dump:live,format=b,file=/home/myheapdump.hprof PID` 导出堆内存信息至 `dump` 文件中
3. 使用`visual VM`对`dump`文件进行离线分析，找到占用内存高的对象，再找到创建该对象的业务代码位置，从代码和业务场景中定位具体问题。

### 3.6 内存泄露/溢出问题的定位与解决方案

> 内存泄漏和内存溢出二者关系非常密切，内存溢出可能会有很多原因导致，内存泄漏最可能的罪魁祸首之一。

#### 内存泄露的可能表现：

+ 应用程序长时间**连续运行时性能严重下降**
+ `CPU` **使用率飙升**，甚至到 100%
+ 频繁 `Full GC`，各种报警，例如接口超时报警等
+ 应用程序抛出 `OutOfMemoryError` 错误
+ 应用程序偶尔会**耗尽连接对象**

#### 内存泄露的定位方案：

严重**内存泄漏**往往伴随频繁的 `Full GC`，所以分析排查**内存泄漏**问题首先还得从查看 `Full GC` 入手。主要有以下操作步骤：

1. 使用 `jps` 查看运行的 Java **进程 ID**

2. 使用 `top -p [pid]` 查看**进程**使用 `CPU` 和 `MEM` 的情况

3. 使用 `top -Hp [pid]` 查看进程下的所有**线程**占 `CPU` 和 `MEM` 的情况

4. 将**线程 ID** 转换为 16 进制： `printf "%x\n" [pid]` ，输出的值就是线程栈信息中的 nid。
   例如： `printf "%x\n" 29471` ，换行输出 731f。

5. 抓取线程栈： `jstack 29452 > 29452.txt` ，可以多抓几次做个对比。
   在线程栈信息中找到对应线程号的 16 进制值，如下是 731f 线程的信息。线程
   栈分析可使用 Visualvm 插件 TDA。

6. 使用 `jstat -gcutil [pid] 5000 10` 每隔 5 秒输出 GC 信息，输出 10 次，查看 `YGC` 和 `Full GC` 次数。通常会出现 YGC 不增加或增加缓慢，而 Full GC增加很快。

   1. 或使用 `jstat -gccause [pid] 5000` ，同样是输出 **GC 摘要信息**。
   2. 或使用 `jmap -heap [pid]` 查看 **堆的摘要信息**，关注老年代内存使用是否达到阀值，若达到阀值就会执行 `Full GC`。

7. 如果发现 `Full GC` 次数太多，就很大概率存在内存泄漏了

8. 使用 `jmap -histo:live [pid]` 输出每个类的对象数量，内存大小(字节单位)及全限定类名。

9. 生成 `dump` 文件，借助工具分析哪 个对象非常多，基本就能定位到问题在那了使用 `jmap` 生成 `dump` 文件。

10. `dump` 文件分析：

11. 可以使用 `jhat` 命令分析： `jhat -port 8000 29471.dump` ，浏览器访问 `jhat`服务，端口是 `8000`。

12. 通常使用图形化工具分析，如 JDK 自带的 `jvisualvm`，从 菜单 > 文件 > 装入dump 文件。

13. 或使用第三方式具分析的，如 `JProfiler` 也是个图形化工具，`GCViewer` 工具。Eclipse 或以使用 `MAT` 工具查看。或使用在线分析平台 `GCEasy`。

    注意：如果 dump 文件较大的话，分析会占比较大的内存。

14. 在 `dump` 文析结果中查找存在大量的对象，再查对其的引用。


基本上就可以定位到代码层的逻辑了。

