## 1、垃圾回收（GC）概述

> 如果没有特殊说明，都是针对的是 `HotSpot` 虚拟机。

当需要排查各种内存溢出问题、当垃圾收集成为系统达到更高并发的瓶颈时，我们就需要对这些“自动化”的技术实施必要的监控和调节。

Java 的自动内存管理主要是针对对象内存的回收和对象内存的分配。同时，Java 自动内存管理最核心的功能是 **堆** 内存中对象的分配与回收。

**Java 堆是垃圾收集器管理的主要区域**，因此也被称作 **GC 堆（Garbage Collected Heap）**。

从垃圾回收的角度来说，由于现在收集器基本都采用 **分代垃圾收集算法**，所以 Java 堆被划分为了几个不同的区域，这样我们就可以根据各个区域的特点选择合适的垃圾收集算法。

在 JDK 7 版本及 JDK 7 版本之前，堆内存被通常分为下面三部分：

1. 新生代（Young Generation）
2. 老生代（Old Generation）
3. 永久代（Permanent Generation）

**下图所示的 Eden 区、两个 Survivor 区 S0 和 S1 都属于`新生代`，中间一层属于`老年代`，最下面一层属于`永久代`，但 JDK 1.8 中是 `元空间`**。

![image-20220731165419362](https://img.zxdmy.com/2022/202208010958045.png)

## 2、对象死亡

### 2.1 对象死亡的判断

**堆** 中几乎放着所有的**对象实例**，对 **堆** 垃圾回收前的第一步就是要判断哪些对象已经死亡（即不能再被任何途径使用的对象）

#### 引用计数法

给对象中添加一个引用计数器：

- 每当有一个地方引用它，计数器就加 1；
- 当引用失效，计数器就减 1；
- 任何时候计数器为 0 的对象就是不可能再被使用的。

> **这个方法实现简单，效率高，但是目前主流的虚拟机中`并没有`选择这个算法来管理内存，其最主要的原因是它很难解决对象之间`相互循环引用`的问题。**

相互引用示例：

```java
public class ReferenceCountingGc {
    Object instance = null;
    public static void main(String[] args) {
        ReferenceCountingGc objA = new ReferenceCountingGc();
        ReferenceCountingGc objB = new ReferenceCountingGc();
        objA.instance = objB;
        objB.instance = objA;
        objA = null;
        objB = null;
    }
}
```

除了对象 `objA` 和 `objB` 相互引用着对方之外，这两个对象之间再无任何引用。但是他们因为互相引用对方，导致它们的引用计数器都不为 `0`，无法被回收。

#### 可达性分析法

即通过一系列被称为 **“GC Roots”** 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为**引用链**，当一个对象到 **GC Roots** 没有任何**引用链**相连的话，则证明此对象是不可用的，需要被回收。

下图中的 `Object 6 ~ Object 10` 之间虽有引用关系，但它们到 **GC Roots** 不可达，因此需要被回收的对象。

![image-20220801095635336](https://img.zxdmy.com/2022/202208010956419.png)

### 2.2 GC ROOT 对象类型

- 虚拟机栈（栈帧中的本地变量表）中引用的对象
- 本地方法栈（Native 方法）中引用的对象
- 方法区中**类静态属性**引用的对象
- 方法区中**常量引用**的对象
- 所有被**同步锁**持有的对象

### 2.3 临死前的最后挽留：finalize()方法

如果对象在进行 **可达性分析** 后发现没有与 `GC Roots` 相连接的引用链，那它将会被**第一次标记**，随后进行一次筛选，筛选的条件是此对象**是否有必要执行`finalize()`方法**。

如果对象在在`finalize()`中成功拯救自己——只要重新与引用链上的任何一个对象建立关联即可，譬如把自己 （this关键字）赋值给某个类变量或者对象的成员变量，那在第二次标记时它就“逃过一劫”。

但是如果没有抓住这个机会，那么对象就真的要被回收了。

## 3、类型引用

> 无论是通过 **引用计数法** 判断对象引用数量，还是通过 **可达性分析法** 判断对象的引用链是否可达，**判定对象的存活都与“引用”有关。**

`JDK1.2` 以后，`Java` 对引用的概念进行了扩充，将引用分为**强引用**、**软引用**、**弱引用**、**虚引用**四种（引用**强度逐渐减弱**）。

|  引用  |      用途      |    生存时间    |  GC 时间   |
| :----: | :------------: | :------------: | :--------: |
| 强引用 | 对象的一般状态 | JVM 停止时终止 |  从来不会  |
| 软引用 |    对象缓存    | 内存不足时终止 | 内存不足时 |
| 弱引用 |    对象缓存    |   GC 运行后    |   GC 时    |
| 虚引用 |    Unknown     |   随时 Over    |  任何时候  |

![image-20220923212804219](https://img.zxdmy.com/2022/202209232128365.png)

![image-20220918213102357](https://img.zxdmy.com/2022/202209182131569.png)

#### 强引用

**强引用** 是使用最普遍的引用，我们使用的大部分引用都属于强引用。

**如果一个对象具有强引用，那该对象必不可少，垃圾回收器 绝不会回收它。**

当内存不足时，也不会回收，而是抛出  `OutOfMemoryError` 错误使程序异常终止。

#### 软引用

具有 **软引用** 的对象可有可无。

内存足够时，垃圾回收器 不会回收它，内存不足时，垃圾回收器 会回收这些对象的内存。

**软引用** 可用来实现内存敏感的高速缓存。

> **软引用** 可以和一个**引用队列**（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，JAVA 虚拟机就会把这个软引用加入到与之关联的引用队列中。

#### 弱引用

具有 **弱引用** 的对象可有可无。

**弱引用** 的对象拥有更短暂的生命周期，垃圾回收器线程一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。（垃圾回收器 是一个优先级很低的线程， 不一定会很快发现那些只具有弱引用的对象。）

> **弱引用** 可以和一个**引用队列**（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java 虚拟机就会把这个弱引用加入到与之关联的引用队列中。

#### 虚引用

如果一个对象仅持有**虚引用**，那么它就和**没有任何引用**一样，在任何时候都可能被垃圾回收。

## 4、对象分配机制

#### 对象优先在 Eden 区分配

大多数情况下，对象在**新生代**中 Eden 区分配。

`当 Eden 区没有足够空间进行分配时`，虚拟机将发起一次 **Minor GC（新生代垃圾回收）**。

#### 大对象直接进入老年代

**大对象** 就是需要大量连续内存空间的对象（比如：字符串、数组）。

大对象直接进入老年代主要是为了避免为大对象分配内存时由于**分配担保机制**带来的复制而降低效率。

> **空间分配担保** 是为了确保在 Minor GC 之前老年代本身还有容纳新生代所有对象的剩余空间。

#### 长期存活的对象将进入老年代

由于 Java 虚拟机采用分代收集的思想来管理内存，为了内存回收时识别哪些对象放在新生代，哪些对象放在老年代，虚拟机给每个对象一个**对象年龄计数器**（Age）。

1. 大多数情况下，对象首先被分配在 `Eden` 区域；
2. `Eden` 出生的对象，经过一次 `Minor GC` 后仍存活，并且能被 `Survivor` 容纳：将被移动到 `Survivor` 空间（`s0` 或者 `s1`）中，并将对象年龄设为 `1`
3. 对象在 `Survivor` 中每熬过一次 `MinorGC`,年龄就增加 `1` 岁
4. 当该对象年龄增加到一定程度（默认为 15 岁，但有出入，详见下文），就会被晋升到`老年代`中。

#### 动态对象年龄判定

为了能更好地适应不同程序的内存状况，HotSpot虚拟机并不是永远要求对象的年龄必须达到- XX：MaxTenuringThreshold才能晋升老年代，如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代。

> 对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

> `Hotspot` 遍历所有对象时，按照**年龄从小到大**对其所占用的大小进行累积，当累积的某个年龄大小超过了 `survivor` 区的 `50%` 时（默认值是 `50%`，可以通过参数 `-XX:TargetSurvivorRatio=percent` 来设置），取这个年龄和 MaxTenuringThreshold 中更小的一个值，作为新的晋升年龄阈值。

#### 空间分配担保

假如在Young GC之后，新生代仍然有大量对象存活，就需要老年代进行分配担保，把Survivor无法容纳的对象直接送入老年代。

## 5、分代垃圾回收

### 5.1 垃圾回收类型

**部分收集**（`Partial GC`）：指目标不是完整收集整个 Java 堆的垃圾收集。

其中又分为：

- **新生代收集**（`Minor GC / Young GC`）：只对新生代进行垃圾收集。
- **老年代收集**（`Major GC / Old GC`）：只对老年代进行垃圾收集。需要注意的是 Major GC 在有的语境中也用于指代整堆收集。目前只有 **CMS收集器** 会有单独收集老年代的行为。
- **混合收集**（`Mixed GC`）：对整个新生代和部分老年代进行垃圾收集。目前只有 **G1收集器** 会有这种行为。

**整堆收集** (`Full GC`)：收集整个 Java **堆**和 **方法区** 的垃圾收集。

### 5.2 新生代收集触发条件

新创建的对象优先在新生代Eden区进行分配，

如果`Eden`区没有足够的空间时，就会触发`Young GC`来清理**新生代**。

### 5.3 整堆收集触发条件

触发 `Full GC` 的条件主要有如下几点：

1、**Young GC 之前检查老年代**

在要进行 `Young GC` 时，如果 **老年代可用的连续内存空间 < 新生代历次Young GC后升入老年代的对象总和的平均大小**，说明本次 `Young GC` 后升入 **老年代** 的对象大小，可能会超过 **老年代** 当前可用内存空间，那就会触发 `Full GC`。

2、**Young GC 之后老年代空间不足**

执行 `Young GC` 之后有一批对象需要放入 **老年代**，此时 **老年代** 没有足够的内存空间存放这些对象，就必须触发一次 `Full GC`。

3、**老年代空间不足**

老年代内存使用率过高，**达到一定比例**，也会触发 `Full GC`。

4、**空间分配担保失败**（ Promotion Failure），

新生代的 To 区放不下从 Eden 和 From 拷贝过来对象，或者新生代对象 GC 年龄到达阈值需要晋升这两种情况，老年代如果放不下的话都会触发 Full GC。

5、**方法区内存空间不足**

如果方法区由永久代实现，永久代空间不足就会触发 Full GC。

6、`System.gc()` **等命令触发**

`System.gc()`、`jmap -dump` 等命令会触发 `full gc`。

## 6、废弃常量与无用类

#### 废弃常量

> **运行时常量池** 主要回收的是**废弃常量**。

假如在字符串常量池中存在字符串 "abc"，如果当前**没有任何 String 对象引用该字符串常量**的话，就说明常量 "abc" 就是废弃常量，如果这时发生内存回收的话而且有必要的话，"abc" 就会被系统清理出常量池了。

#### 无用类

> **方法区**主要回收的是**无用的类**。但不是必然回收！

**无用类** 满足以下 3 个条件：

- 该类所有的实例都已经被回收，也就是 Java 堆中**不存在该类的任何实例**。
- **加载该类的 `ClassLoader` 已经被回收。**
- 该类对应的 `java.lang.Class` 对象**没有在任何地方被引用**，无法在任何地方通过反射访问该类的方法。

## 7、垃圾收集算法

> 常用的 GC 算法（方式）。

#### 标记-清除 算法

该算法包含 **标记** 和 **清除** 两个阶段。

1. 标记 所有不需要回收的对象；
2. 统一回收所有 未被标记 的对象。

该算法存在两个问题：

+ 效率问题
+ 空间问题：标记清除后会产生大量不连续的空间碎片。

> 这是最基础的收集算法，后续算法均对其不足进行改进。

#### 标记-复制 算法

为了**解决 效率问题**，该算法出现。

1. 首先将内存分为大小相同的两块，每次使用其中的一块；
2. 当一块内存使用完后，将存活的对象复制到另一块；
3. 清理掉使用完成的内存块。

#### 标记-整理 算法

根据**老年代**的特点提出的一种标记算法，标记过程与“标记-清除”算法一致。

但是：**不直接对可回收对象回收，而是让所有存活的对象向一端移动，然后直接清理掉端边界以外的内存。**

#### 分代收集算法

> 当前虚拟机的垃圾收集都采用分代收集算法。
>
> 其实就是根据对象存活周期的不同，将内存分为几块。

一般将 java 堆分为**新生代**和**老年代**，这样我们就可以**根据各个年代的特点选择合适的垃圾收集算法**。

+ **新生代**：每次收集，都将有大量的对象死去，故可选择 **标记-复制 算法**。
+ **老年代**：对象存活几率较高，没有额外的空间对其进行分配担保，故选择 “**标记-清除**”或“**标记-整理**”算法进行垃圾收集。

> **这也是为什么要分 新生代 和 老年代 的原因之一。**

## 8、垃圾收集器

|       收集器        |            简介             |    算法     |    更多    |
| :-----------------: | :-------------------------: | :---------: | :--------: |
|      `Serial`       |  **串行新生代** 垃圾收集器  |  标记-复制  | **单线程** |
|    `Serial` Old     |  **串行老年代** 垃圾收集器  |  标记-整理  |   单线程   |
|      `ParNew`       | `Serial` 的 **多线程** 版本 | 同 `Serial` | **多线程** |
| `Parallel` Scavenge |  **并行新生代**垃圾收集器   |  标记-复制  |   多线程   |
|   `Parallel` Old    |  **并行老年代**垃圾收集器   |  标记-整理  |   多线程   |
|        `CMS`        |      获取最短停顿时间       |  标记-清除  |            |
|        `G1`         |     简单可行的性能调优      |     ==      |            |
|         ZGC         |                             |  标记-复制  |            |

![img](https://img.zxdmy.com/2022/202209182142217.jpeg)



#### Serial 收集器

**Serial 收集器**（串行收集器）是最基本、历史最悠久的垃圾收集器。

**优点：**

+ 简单高效（与其他收集器的单线程相比）

**缺点：**

+ **这是一个 单线程收集器，在进行垃圾收集工作时，必须暂停其他所有的工作线程，直至收集结束。**

![image-20220801105010498](https://img.zxdmy.com/2022/202208011050864.png)

+ 新生代采用 标记-复制 算法
+ 老年代采用 标记-整理 算法

#### ParNew 收集器

**ParNew 收集器** 是 Serial 收集器的**多线程**版本，除了使用多线程进行垃圾收集外，其余行为（控制参数、收集算法、回收策略等等）和 Serial 收集器完全一样。

![image-20220801105221618](https://img.zxdmy.com/2022/202208011052557.png)

+ 新生代采用 标记-复制 算法
+ 老年代采用 标记-整理 算法

> 除了 `Serial 收集器`，只有`ParNew 收集器` 能与 `CMS 收集器`（真正意义上的并发收集器）配合工作。

#### Parallel Scavenge 收集器

**Parallel Scavenge 收集器** 与 **ParNew 收集器** 几乎一样。

**Parallel Scavenge 收集器 关注点是吞吐量（高效率的利用 CPU）**：提供很多参数供用户找到最合适的停顿时间或最大吞吐量。

![image-20220801110846918](https://img.zxdmy.com/2022/202208011108941.png)

+ 新生代采用标记-复制算法
+ 老年代采用标记-整理算法

> **JDK1.8** 默认使用的是 **Parallel Scavenge** + **Parallel Old**。
>
> 如果指定了-XX:+UseParallelGC 参数，则默认指定了-XX:+UseParallelOldGC，可以使用-XX:-UseParallelOldGC 来禁用该功能

#### Serial Old 收集器

**Serial 收集器的老年代版本**，是一个单线程收集器。主要有两大用途：

1. 在 JDK1.5 以及以前的版本中与 Parallel Scavenge 收集器搭配使用
2. 作为 CMS 收集器的后备方案

#### Parallel Old 收集器

**Parallel Scavenge 收集器的老年代版本**，使用多线程和“标记-整理”算法。

在注重吞吐量以及 CPU 资源的场合，都可以优先考虑 Parallel Scavenge 收集器和 Parallel Old 收集器。

#### CMS 收集器

**CMS（Concurrent Mark Sweep，并发标记清除）收集器是一种以获取最短回收停顿时间为目标的收集器。**它非常符合在注重用户体验的应用上使用。

**CMS（Concurrent Mark Sweep）收集器是 HotSpot 虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。**

**CMS 收集器** 使用 **“标记-清除”算法** 实现。

**CMS 收集器** 运作流程：

- **初始标记：** 暂停所有的其他线程，并记录下直接与 root 相连的对象，速度很快 ；
- **并发标记：** 同时开启 GC 和用户线程，用一个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以 GC 线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方。
- **重新标记：** 重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短
- **并发清除：** 开启用户线程，同时 GC 线程开始对未标记的区域做清扫。

![image-20220801143558892](https://img.zxdmy.com/2022/202208011436695.png)

**优点：**

+ 并发收集、低停顿

**缺点：**

- 对 CPU 资源敏感；
- 无法处理浮动垃圾；
- 它使用的回收算法-“标记-清除”算法会导致收集结束时会有大量空间碎片产生。

#### G1 收集器

**G1 (Garbage-First)** 是一款**面向服务器**的垃圾收集器，主要针对配备多颗处理器及大容量内存的机器，以极高概率满足 GC 停顿时间要求的同时，还具备高吞吐量性能特征。

被视为 JDK1.7 中 HotSpot 虚拟机的一个重要进化特征，它具备以下特点：

- **并行与并发**：G1 能充分利用 CPU、多核环境下的硬件优势，使用多个 CPU（CPU 或者 CPU 核心）来缩短 Stop-The-World 停顿时间。部分其他收集器原本需要停顿 Java 线程执行的 GC 动作，G1 收集器仍然可以通过并发的方式让 java 程序继续执行。
- **分代收集**：虽然 G1 可以不需要其他收集器配合就能独立管理整个 GC 堆，但是还是保留了分代的概念。
- **空间整合**：与 CMS 的“标记-清理”算法不同，G1 从整体来看是基于“标记-整理”算法实现的收集器；从局部上来看是基于“标记-复制”算法实现的。
- **可预测的停顿**：这是 G1 相对于 CMS 的另一个大优势，降低停顿时间是 G1 和 CMS 共同的关注点，但 G1 除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内。

G1 收集器的运作大致分为以下几个步骤：

- **初始标记**
- **并发标记**
- **最终标记**
- **筛选回收**

**G1 收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的 Region(这也就是它的名字 Garbage-First 的由来)** 。这种使用 Region 划分内存空间以及有优先级的区域回收方式，保证了 G1 收集器在有限时间内可以尽可能高的收集效率（把内存化整为零）

#### ZGC 收集器

与 CMS 中的 ParNew 和 G1 类似，ZGC 也采用**标记-复制**算法，不过 ZGC 对该算法做了重大改进。

## 9、HotSpot的 GC 算法细节实现

#### 9.1 根节点枚举

迄今为止，所有收集器在 **根节点枚举** 这一步骤时都是 **必须暂停用户线程** 的。

现在 **可达性分析算法** 耗时最长的 **查找引用链** 的过程已经可以做到**与用户线程一起并发**，**但** **根节点枚举**始终还是必须在一个能保障一致性的快照中才得以进行。

这里 **一致性** 的意思是整个枚举期间，执行子系统看起来就像被冻结在某个时间点上，不会出现分析过程中， 根节点集合的对象引用关系还在不断变化的情况，若这点不能满足的话，分析结果准确性也就无法保证。这是导致垃圾收集过程必须停顿所有用户线程的其中一个重要原因。

即使是号称停顿时间可控，或者（几乎）不会发生停顿 的`CMS`、`G1`、`ZGC` 等收集器，**枚举根节点时也是必须要停顿的**。

由于目前主流 Java 虚拟机使用的都是 **准确式垃圾收集**，所以**当用户线程停顿**下来之后，其实并不需要一个不漏地检查完所有执行上下文和全局的引用位置，虚拟机是有办法**直接得到哪些地方存放着对象引用**的。

在 `HotSpot` 的解决方案里，是使用一组称为`OopMap`的 **数据结构** 来达到这个目的。

类加载动作完成的时候，`HotSpot`就会把 **对象内什么偏移量上是什么类型的数据** 计算出来，在即时编译过程中，也会在特定的位置记录下栈里和寄存器里哪些位置是引用。这样**收集器在扫描时就可以直接得知这些信息**了，并不需要真正一个不漏地从方法区等 `GC Roots` 开始查找。

#### 9.2 STW 与 安全点

> **STW**：

进行**垃圾回收**的过程中，会涉及**对象的移动**。

为了**保证对象引用更新的正确性**，必须**暂停所有的用户线程**。

像这样的停顿，虚拟机设计者形象描述为`Stop The World` ，简称为 `STW`。

在 `HotSpot` 中，有个数据结构（映射表）称为`OopMap` 。

一旦类加载动作完成时，`HotSpot` 会把**对象内什么偏移量上是什么类型的数据计算出来**，记录到`OopMap`。

在`OopMap`的协助下，`HotSpot`可以快速准确地完成`GC Roots`枚举。

> **存在的问题**：

但一个很现实的问题随之而来：可能导致引用关系变化，或者说导致`OopMap`内容变化的指令非常多，如果为每一条指令都生成对应的`OopMap`，那将会需要大量的额外存储空间，这样垃圾收集伴随而来的空间成本就会变得无法忍受的高昂。

> **解决方法**：

在即时编译过程中，也会在**特定的位置**生成 `OopMap`，记录下**栈**上和**寄存器**里哪些位置是**引用**。

这些 **特定的位置** 主要在：

1. 循环的末尾（非 counted 循环）
2. 方法临返回前 / 调用方法的 `call` 指令后
3. 可能抛异常的位置

这些位置就叫作**安全点**（`safepoint`）。

用户程序执行时并非在代码指令流的任意位置都能够在停顿下来开始垃圾收集，而是必须是执行到安全点才能够暂停。

**安全点** 决定了用户程序执行时并非在代码指令流的任意位置都能够停顿下来开始垃圾收集，而是强制要求必须执行到达安全点后才能够暂停。

因此，**安全点**的选定**既不能太少以至于让收集器等待时间过长**，**也不能太过频繁以至于过分增大运行时的内存负荷**。

> **实现安全点**：

如何在垃圾收集发生时让所有线程都跑到最近的安全点，然后停顿下来？主要有两种解决方案：

+ **抢先式中断** （Preemptive Suspension）：抢先式中断不需要线程的执行代码主动去配合，在垃圾收集发生时，系统**首先把所有用户线程全部中断**，如果发现有用户线程中断的地方不在安全点上，就恢复这条线程执行，让它一会再重新中断，直到跑到安全点上。现在几乎没有虚拟机实现采用抢先式中断来暂停线程响应GC事件。（涉及到线程上下文的切换，比较消耗资源）
+ 【实际采用】**主动式中断**（Voluntary Suspension）：主动式中断的思想是当垃圾收集需要中断线程的时候，不直接对线程操作，仅仅简单地设置一个标志位，**各个线程执行过程时会不停地主动去轮询这个标志，一旦发现中断标志为真时就自己在最近的安全点上主动中断挂起**。轮询标志的地方和安全点是重合的，另外还要加上所有创建对象和其他需要在Java堆上分配内存的地方，这是为了检查是否即将要发生垃圾收集，避免没有足够内存分配新对象。

#### 9.3 安全区域

> **安全点机制的不足**：

**安全点机制**保证了 **程序执行** 时，在不太长的时间内就会遇到可进入垃圾收集过程的安全点。

但是，程序“**不执行**” 的时候呢？所谓的程序不执行就是没有分配处理器时间，典型的场景便是**用户线程处于Sleep状态或者Blocked状态**，这时候线程无法响应虚拟机的中断请求，不能再走到安全的地方去中断挂起自己，虚拟机也显然不可能持续等待线程重新被激活分配处理器时间。

对于这种情况，就必须引入**安全区域**（`Safe Region`）来解决。

> **解决方案**：

**安全区域** 是指能够**确保在某一段代码片段之中，引用关系不会发生变化**，因此，在这个区域中任意地方开始垃圾收集都是安全的。

我们也可以把安全区域看作被扩展拉伸了的安全点。

+ 当**用户线程**执行到**安全区域里面的代码**时，首先会**标识自己已经进入了安全区域**，那样当这段时间里虚拟机要发起垃圾收集时就不必去管这些已声明自己在安全区域内的线程了。
+ 当线程要**离开安全区域**时，它要**检查虚拟机是否已经完成了根节点枚举**（或者垃圾收集过程中其他需要暂停用户线程的阶段），如果完成了，那线程就当作没事发生过，继续执行；否则它就必须一直等待，直到收到可以离开安全区域的信号为止

> 垃圾回收是**如何处理跨代引用**问题的

跨代引用举例：假如要现在进行一次只局限于新生代区域内的收集（`Minor GC`），但新生代中的对象是完全有可能被老年代所引用的，为了找出该区域中的存活对象，不得不在固定的`GC Roots`之外，再额外遍历整个老年代中所有对象来确保可达性分析结果的正确性，反过来也是一样。

并不只是新生代、老年代之间才有跨代引用的问题，**所有涉及部分区域收集（Partial GC）行为的垃圾收集器，典型的如G1、ZGC和Shenandoah收集器，都会面临相同的问题**。

`JVM` 为了用尽量少的资源消耗解决跨代引用下的垃圾回收问题，引入了**记忆集**。

**记忆集**是一种用于**记录从非收集区域指向收集区域的指针集合的抽象数据结构**。

在垃圾收集的场景中，收集器只需要通过记忆集判断出某一块非收集区域是否存在有指向了收集区域的指针就可 以了，并不需要了解这些跨代指针的全部细节。

目前最常用的一种记忆集实现形式种称为“**卡表**”，**卡表** 中的**每个记录精确到一块内存区域**（每块内存区域称之为卡页），该区域内有**对象**含有**跨代指针**。

一个卡页的内存中通常包含不止一个对象，只要卡页内有一个（或更多）对象的字段存在着跨代指针，那就将对应卡表的数组元素的值标识为`1`，称为这个**元素变脏**（Dirty），没有则标识为`0`。

在垃圾收集发生时，只要**筛选出卡表中变脏的元素，就能轻易得出哪些卡页内存块中包含跨代指针**，把它们加入`GC Roots`中一并扫描。

#### 9.4 写屏障

解决了如何 **使用记忆集来缩减GC Roots扫描范围** 的问题，但还没有解决**卡表元素如何维护**的问题，例如它们何时变脏、谁来把它们变脏等。

> 卡表元素**何时变脏**

+ 有其他分代区域中对象引用了本区域对象时，其对应的卡表元素就应该变脏
+ 变脏时间点原则上应该发生在引用类型字段赋值的那一刻

> 卡表元素**如何变脏**，即**如何在对象赋值的那一刻去更新维护卡表**

+ **解释执行的字节码**，那相对好处理，虚拟机负责每条字节码指令的执行，有充分的介入空间
+ **编译执行**是通过**写屏障**（Write Barrier）技术维护卡表状态的，写屏障可以看作在虚拟机层面对“引用类型字段赋值”这个动作的AOP切面

#### 9.5 并发的可达性分析（垃圾回收器的三色标记理论）

当前主流编程语言的**垃圾收集器**基本上都是**依靠可达性分析算法来判定对象是否存活**的，可达性分析算法理 论上要求全过程都基于一个能保障一致性的快照中才能够进行分析，这意味着必须全程冻结用户线程的运行。

在根节点枚举这个步骤中，由于GC Roots相比起整个Java堆中全部的对象毕竟还算是极少数，且在各种优化技 巧（如OopMap）的加持下，它带来的停顿已经是非常短暂且相对固定（不随堆容量而增长）的了

“**标记**”阶段是**所有追踪式垃圾收集算法的共同特征**，如果这个阶段会随着堆变大而等比例增加停顿时间，其影响就会波及几乎所有的垃圾收集器。

+ 白色：表示**对象尚未被垃圾收集器访问过**。显然在可达性分析刚刚开始的阶段，所有的对象都是白色的，若在分析结束的阶段，仍然是白色的对象，即代表不可达。
+ 黑色：表示**对象已经被垃圾收集器访问过**，且这个对象的所有引用都已经扫描过。黑色的对象代表已经扫描过， 它是安全存活的，如果有其他对象引用指向了黑色对象，无须重新扫描一遍。黑色对象不可能直接（不经过灰 色对象）指向某个白色对象。
+ 灰色：表示**对象已经被垃圾收集器访问过**，但这个对象上**至少存在一个引用还没有被扫描过**。

![img](https://img.zxdmy.com/2022/202209271545898.png)

#### 9.6 对象消失

“**对象消失**”的问题，即**原本应该是黑色的对象被误标为白色**。

当且仅当以下两个条件同时满足时，会产生“对象消失”的问题:

+ 赋值器插入了一条或多条从黑色对象到白色对象的新引用；
+ 赋值器删除了全部从灰色对象到该白色对象的直接或间接引用

解决并发扫描时的对象消失问题，只需破坏这两个条件的任意一个即可。

由此分别产生了两种解决方案：**增量更新**（Incremental Update）和**原始快照**（Snapshot At The Beginning，SATB），具体如下：

+ **增量更新**要破坏的是**第一个条件**，当黑色对象插入新的指向白色对象的引用关系时，就将这个新插入的引用记录下来，等并发扫描结束之后，再将这些记录过的引用关系中的黑色对象为根，重新扫 描一次。这可以简化理解为，黑色对象一旦新插入了指向白色对象的引用之后，它就变回灰色对象了。
+ **原始快照**要破坏的是**第二个条件**，当灰色对象要删除指向白色对象的引用关系时，就将这个要删除的引用记录下来，在并发扫描结束之后，再将这些记录过的引用关系中的灰色对象为根，重新扫描 一次。这也可以简化理解为，无论引用关系删除与否，都会按照刚刚开始扫描那一刻的对象图快照来进行搜索

以上无论是对**引用关系记录的插入还是删除**，虚拟机的记录操作都是通过**写屏障**实现的。

在`HotSpot`虚拟机中， 增量更新和原始快照这两种解决方案都有实际应用，譬如，`CMS`是基于增量更新来做并发标记的，`G1`、 `Shenandoah`则是用原始快照来实现。

## 补充：元空间的内存管理

在**元空间**中，**类和其元数据的生命周期**与其对应的**类加载器**相同，只要**类的类加载器是存活**的，在`Metaspace`中的**类元数据也是存活**的，不能被回收（所以基本上不存在类回收）。

**元空间**的内存，由**元空间虚拟机**（`Metaspace VM`）管理。

`Metaspace VM`使用一个**块分配器**（`chunking allocator`）来管理`Metaspace`空间的内存分配，块的大小依赖于类加载器的类型。

`Metaspace VM`中有一个**全局的可使用的块列表**（`a global free list of chunks`）：`VirtualSpaceList`。

当**类加载器**需要一个块的时候，类加载器从全局块列表中取出一个块，添加到它自己维护的块列表中。

当**类加载器**死亡，它的块将会被释放，归还给全局的块列表。

**块**（`chunk`）会进一步被划分成 `blocks`，每个 `block` 存储一个**元数据单元**（a unit of metadata）。

`Chunk` 中`Blocks`的是分配线性的（pointer bump）。

这些`chunks`被分配在内存映射空间（memory mapped(mmapped) spaces）之外。

在一个全局的虚拟内存映射空间（global virtual mmapped spaces）的链表，当任何虚拟空间变为空时，就将该虚拟空间归还回操作系统。

![image-20220924205325840](https://img.zxdmy.com/2022/202209242053543.png)


