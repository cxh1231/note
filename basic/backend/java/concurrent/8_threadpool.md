## 1、使用线程池的原因

> **池化技术** 的思想，在项目中已被广泛应用，比如数据库连接处、HTTP 连接池等等，池化技术的思想主要是为了减少每次获取资源的消耗，提供对资源的利用率。

**线程池** 提供了一种限制和管理资源（包括执行一个任务）的方式。

每个**线程池** 维护了一些基本统计信息，例如已完成任务的数量

**使用线程池的好处**：

- **降低资源消耗**。通过`重复利用已创建的线程`降低线程创建和销毁造成的消耗。
- **提高响应速度**。当任务到达时，任务可以`不需要等到线程创建就能立即执行`。
- **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，`使用线程池可以进行统一的分配，调优和监控` 。

## 2、Executor 框架结构

`Executor` 框架是 Java5 之后引进的，通过 `Executor` 来启动线程比使用 `Thread` 的 `start` 方法更好，除了更易管理，效率更好（用线程池实现，节约开销）外，还有关键的一点：有助于避免 this 逃逸问题。

> 补充：this 逃逸是指在构造函数返回之前其他线程就持有该对象的引用. 调用尚未构造完全的对象的方法可能引发令人疑惑的错误。

`Executor` 框架不仅包括了**线程池的管理**，还提供了**线程工厂**、**队列**以及**拒绝策略**等，`Executor` 框架让并发编程变得更加简单。

#### 结构1：任务（Runnable / Callable）

执行任务需要实现 **`Runnable` 接口** 或 **`Callable`接口**。**`Runnable` 接口 **或 **`Callable` 接口** 的实现类都可以被 **`ThreadPoolExecutor`** 或 **`ScheduledThreadPoolExecutor`** 执行。

> **`Runnable` 接口和 `Callable` 接口的区别：**
>
> + **`Runnable` 接口** 不会返回结果或抛出检查异常，但是 **`Callable` 接口** 可以。
> + 工具类 `Executors` 可以实现将 `Runnable` 对象转换成 `Callable` 对象。

#### 结构2：任务的执行（Executor）

任务执行机制的**核心接口**是 **`Executor`** ，以及继承自 `Executor` 接口的 `ExecutorService` 接口。

![image-20220804180215789](https://img.zxdmy.com/2022/202208041802505.png)

**`ThreadPoolExecutor`** 和 **`ScheduledThreadPoolExecutor`** 这两个**关键类**实现了 `ExecutorService` 接口。

> 上图的各个底层类关系中， `ThreadPoolExecutor` 类在实际使用线程池的过程中，使用频率非常高。

> `Executors` 是一个工具类，不同的方法按照不同需求创建不同的线程池，来满足业务的需求。

#### 结构3：异步计算的结果（Future）

**`Future`** 接口以及 `Future` 接口的实现类 **`FutureTask`** 类都可以代表异步计算的结果。

当我们把 **`Runnable`接口** 或 **`Callable` 接口** 的实现类提交给 **`ThreadPoolExecutor`** 或 **`ScheduledThreadPoolExecutor`** 执行，调用 `submit()` 方法时会返回一个 **`FutureTask`** 对象。

> **`ExecutorService` 接口有两种执行线程任务的方法：`execute()` 和 `submit()`，其区别如下**：
>
> + `execute()`方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；
> + `submit()`方法用于提交需要返回值的任务。线程池会返回一个 `Future` 类型的对象，通过这个 `Future` 对象可以判断任务是否执行成功，并且可以通过 `Future` 的 `get()`方法来获取返回值，`get()`方法会阻塞当前线程直到任务完成，而使用 `get(long timeout，TimeUnit unit)`方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

## 3、线程池流程

#### Executor 的执行流程

如下图所示：

![image-20220804181040377](https://img.zxdmy.com/2022/202208041810573.png)

1. **主线程**创建实现 `Runnable` 或 `Callable` 接口的**任务对象**；
2. 可以把实现 `Runnable`接口的 **任务对象** 直接交给 `ExecutorService` 的`execute(Runnable command)` 方法执行；
3. 或者可以把实现 `Runnable/Callable` 接口的 **任务对象** 提交给 `ExecutorService` 的`submit(Runnable task)` 或  `submit(Callable <T> task)` 执行，并返回一个实现`Future`接口的`FutureTask` 对象。
4. 最后，主线程可以执行 `FutureTask.get()`方法来等待任务执行完成。
5. 主线程也可以执行 `FutureTask.cancel（boolean mayInterruptIfRunning）`来取消此任务的执行。

#### 线程池的运行（运作）流程

![image-20220727202905097](https://img.zxdmy.com/2022/202207272029608.png)

![image-20220815162522421](https://img.zxdmy.com/2022/202208151625353.png)

#### 线程池的线程复用

源码中 ThreadPoolExecutor 中有个内置对象Worker，每个worker都是一个线程，worker线程数量和参数有关，每个worker会while死循环从阻塞队列中取数据，**通过置换worker中Runnable对象，运行其run方法起到线程置换的效果**，这样做的好处是避免多线程频繁线程切换，提高程序运行性能。

## 4、ThreadPoolExecutor 类

> 线程池实现类 `ThreadPoolExecutor` 是 `Executor` 框架最核心的类。

#### ThreadPoolExecutor 类分析

`ThreadPoolExecutor` 类中提供的四个构造方法。其中最基础的构造方法如下：

```java
/**
 * 用给定的初始参数创建一个新的ThreadPoolExecutor。
 */
public ThreadPoolExecutor(int corePoolSize,//线程池的核心线程数量
                          int maximumPoolSize,//线程池的最大线程数
                          long keepAliveTime,//当线程数大于核心线程数时，多余的空闲线程存活的最长时间
                          TimeUnit unit,//时间单位
                          BlockingQueue<Runnable> workQueue,//任务队列，
                          ThreadFactory threadFactory,//线程工厂，用来创建线程，一般默认即可
                          RejectedExecutionHandler handler//拒绝策略，
                           ) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

其他构造方法均基于此，对一些默认参数进行指定。

**构造方法的 7 个参数：**

1. **`corePoolSize`** ：**核心线程数**，定义最小可以同时运行的线程数量。
2. **`maximumPoolSize`** ：线程池的**最大线程数**。当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
3. **`workQueue`**：任务队列，用来储存等待执行任务的队列。新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。
4. **`keepAliveTime`** ：当线程池中的线程数量大于 `corePoolSize` 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`才会被回收销毁；
5. **`unit`** ：`keepAliveTime` 参数的时间单位。
6. **`threadFactory`** ：线程工厂，`executor` 创建新线程的时候会用到，一般默认即可。
7. **`handler`** ：**饱和策略（拒绝策略）**，当提交的任务过多而不能及时处理时，定制策略来处理任务：
   + `ThreadPoolExecutor.AbortPolicy` ：（默认策略）抛出 `RejectedExecutionException`来拒绝新任务的处理。
   + `ThreadPoolExecutor.CallerRunsPolicy` ：（推荐使用）调用执行自己的线程运行任务，也就是直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
   + `ThreadPoolExecutor.DiscardPolicy` ：不处理新任务，直接丢弃掉。
   + `ThreadPoolExecutor.DiscardOldestPolicy` ：此策略将丢弃最早的未处理的任务请求。

> **不允许使用 `Executors` 去创建，而是通过 `ThreadPoolExecutor` 构造函数的方式创建线程池的原因：规避资源耗尽的风险**。
>
> `Executors` 返回线程池对象的弊端如下：
>
> - `FixedThreadPool` 和 `SingleThreadExecutor` ：允许**请求的队列长度**为 `Integer.MAX_VALUE` ，可能堆积大量的请求，从而导致 OOM。
> - `CachedThreadPool` 和 `ScheduledThreadPool` ：允许**创建的线程数量**为 `Integer.MAX_VALUE` ，可能会创建大量线程，从而导致 OOM。

#### ThreadPoolExecutor 类的使用

**1、创建任务的实现类**

以`Runnable` 接口为例：

```java
import java.util.Date;

public class MyRunnable implements Runnable {

    private String userName;

    public MyRunnable(String username) {
        this.userName = username;
    }


    @Override
    public void run() {
        System.out.println("当前线程：" + Thread.currentThread().getName() + " 开始时间：" + new Date() + " 当前用户：" + this.userName);
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("当前线程：" + Thread.currentThread().getName() + " 结束时间：" + new Date());
    }
}
```

以`Callable` 接口为例：

```java
import java.util.Date;
import java.util.concurrent.Callable;

public class MyCallable implements Callable<String> {

    private String userName;

    public MyCallable(String username) {
        this.userName = username;
    }

    @Override
    public String call() throws Exception {
        System.out.println("当前线程：" + Thread.currentThread().getName() + " 开始时间：" + new Date() + " 当前用户：" + this.userName);
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("当前线程：" + Thread.currentThread().getName() + " 结束时间：" + new Date());
        return "用户 " + this.userName + " 执行完成";
    }
}
```

**2、创建线程池和任务并执行**

```java
package xianchengchi;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        // 创建线程池
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2, 4, 100,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(10),
                new ThreadPoolExecutor.DiscardPolicy()

        );
        // MyCallable 执行结果返回列表
        // List<Future<String>> futureList = new ArrayList<>();

        // 循环
        for (int i = 0; i < 20; i++) {
            // 通过 Runnable创建任务并执行：
            MyRunnable myRunnable = new MyRunnable("张三" + i);
            threadPoolExecutor.execute(myRunnable);

            // 通过 Callable创建任务并执行
            // MyCallable myCallable = new MyCallable("李四" + i);
            // Future<String> future = threadPoolExecutor.submit(myCallable);
            // futureList.add(future);
        }
        // 打印 MyCallable 的执行结果
        // for (Future<String> future : futureList) {
        //     try {
        //         System.out.println(future.get());
        //     } catch (Exception e) {
        //         e.printStackTrace();
        //     }
        // }

        // 任务执行完成后，结束线程池
        threadPoolExecutor.shutdown();
        while (!threadPoolExecutor.isTerminated()) {
        }
        System.out.println("Finished all threads");
    }
}

```

执行结果：

```java
当前线程：pool-1-thread-3 开始时间：Fri Aug 05 10:49:23 CST 2022 当前用户：张三12
当前线程：pool-1-thread-4 开始时间：Fri Aug 05 10:49:23 CST 2022 当前用户：张三13
当前线程：pool-1-thread-1 开始时间：Fri Aug 05 10:49:23 CST 2022 当前用户：张三0
当前线程：pool-1-thread-2 开始时间：Fri Aug 05 10:49:23 CST 2022 当前用户：张三1
当前线程：pool-1-thread-4 结束时间：Fri Aug 05 10:49:28 CST 2022
当前线程：pool-1-thread-3 结束时间：Fri Aug 05 10:49:28 CST 2022
当前线程：pool-1-thread-1 结束时间：Fri Aug 05 10:49:28 CST 2022
当前线程：pool-1-thread-2 结束时间：Fri Aug 05 10:49:28 CST 2022
当前线程：pool-1-thread-1 开始时间：Fri Aug 05 10:49:28 CST 2022 当前用户：张三3
当前线程：pool-1-thread-3 开始时间：Fri Aug 05 10:49:28 CST 2022 当前用户：张三2
当前线程：pool-1-thread-2 开始时间：Fri Aug 05 10:49:28 CST 2022 当前用户：张三5
当前线程：pool-1-thread-4 开始时间：Fri Aug 05 10:49:28 CST 2022 当前用户：张三4
当前线程：pool-1-thread-2 结束时间：Fri Aug 05 10:49:33 CST 2022
当前线程：pool-1-thread-3 结束时间：Fri Aug 05 10:49:33 CST 2022
当前线程：pool-1-thread-2 开始时间：Fri Aug 05 10:49:33 CST 2022 当前用户：张三6
当前线程：pool-1-thread-3 开始时间：Fri Aug 05 10:49:33 CST 2022 当前用户：张三7
当前线程：pool-1-thread-1 结束时间：Fri Aug 05 10:49:33 CST 2022
当前线程：pool-1-thread-4 结束时间：Fri Aug 05 10:49:33 CST 2022
当前线程：pool-1-thread-1 开始时间：Fri Aug 05 10:49:33 CST 2022 当前用户：张三8
当前线程：pool-1-thread-4 开始时间：Fri Aug 05 10:49:33 CST 2022 当前用户：张三9
当前线程：pool-1-thread-3 结束时间：Fri Aug 05 10:49:38 CST 2022
当前线程：pool-1-thread-1 结束时间：Fri Aug 05 10:49:38 CST 2022
当前线程：pool-1-thread-2 结束时间：Fri Aug 05 10:49:38 CST 2022
当前线程：pool-1-thread-1 开始时间：Fri Aug 05 10:49:38 CST 2022 当前用户：张三11
当前线程：pool-1-thread-4 结束时间：Fri Aug 05 10:49:38 CST 2022
当前线程：pool-1-thread-3 开始时间：Fri Aug 05 10:49:38 CST 2022 当前用户：张三10
当前线程：pool-1-thread-1 结束时间：Fri Aug 05 10:49:43 CST 2022
当前线程：pool-1-thread-3 结束时间：Fri Aug 05 10:49:43 CST 2022
Finished all threads

Process finished with exit code 0
```

如上是执行流程的输出结果，如下图所示。

![image-20220805104411091](https://img.zxdmy.com/2022/202208051044558.png)

> **核心线程池** 对应参数：`corePoolSize = 2`，大于则加入等待队列
>
> **线程池** 对应参数：`maximumPoolSize = 4`，等待队列满了，则创建新的线程池，进行执行

#### ThreadPoolExecutor 类的几个方法

`execute()` vs `submit()`

- `execute()`方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；
- `submit()`方法用于提交需要返回值的任务。

`shutdown()`VS`shutdownNow()`

- **`shutdown（）`** :关闭线程池，线程池的状态变为 `SHUTDOWN`。线程池不再接受新任务了，但是队列里的任务得执行完毕。
- **`shutdownNow（）`** :关闭线程池，线程的状态变为 `STOP`。线程池会终止当前正在运行的任务，并停止处理排队的任务并返回正在等待执行的 List。

 `isTerminated()` VS `isShutdown()`

- **`isShutDown`** 当调用 `shutdown()` 方法后返回为 true。
- **`isTerminated`** 当调用 `shutdown()` 方法后，并且所有提交的任务完成后返回为 true

> 线程池核心线程空闲时怎么销毁？
>
> 超过corePoolSize的空闲线程由线程池回收，线程池Worker启动跑第一个任务之后就一直循环遍历线程池任务队列，超过指定超时时间获取不到任务就remove Worker，最后由垃圾回收器回收。
>
> https://zhuanlan.zhihu.com/p/510240849

## 5、更多线程池（ThreadExecutor）

表格左侧是线程池，右侧为它们对应的阻塞队列， 5 种线程池对应了 3 种阻塞队列。

![image-20220815163618386](https://img.zxdmy.com/2022/202208151636728.png)

#### FixedThreadPool

**实现源码：**

```java
/**
* 创建一个可重用固定数量线程的线程池
*/
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(nThreads, 
                                  nThreads,
                                  0L, 
                                  TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>(),
                                  threadFactory);
}
```

> 创建的 `FixedThreadPool` 的 `corePoolSize` 和 `maximumPoolSize` 都被设置为 nThreads。

**不足：**

1. 当线程池中的线程数达到 `corePoolSize` 后，新任务将在无界队列中等待，因此线程池中的线程数不会超过 corePoolSize；
2. 由于使用无界队列时 `maximumPoolSize` 将是一个无效参数，因为不可能存在任务队列满的情况。所以，通过创建 `FixedThreadPool`的源码可以看出创建的 `FixedThreadPool` 的 `corePoolSize` 和 `maximumPoolSize` 被设置为同一个值。
3. 由于 1 和 2，使用无界队列时 `keepAliveTime` 将是一个无效参数；
4. 运行中的 `FixedThreadPool`（未执行 `shutdown()`或 `shutdownNow()`）不会拒绝任务，在任务比较多的时候会导致 OOM（内存溢出）

#### SingleThreadExecutor 

**实现源码**：

```java
/**
 *返回只有一个线程的线程池
 */
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 
                                1,
                                0L, 
                                TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>(),
                                threadFactory));
}
```

**不足：**

`SingleThreadExecutor` 使用无界队列 `LinkedBlockingQueue` 作为线程池的工作队列（队列的容量为 Intger.MAX_VALUE）。`SingleThreadExecutor` 使用无界队列作为线程池的工作队列会对线程池带来的影响与 `FixedThreadPool` 相同。说简单点就是可能会导致 OOM，

#### CachedThreadPool 

**实现源码**：

```java
/**
 * 创建一个线程池，根据需要创建新线程，但会在先前构建的线程可用时重用它。
 */
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(0, 
                                  Integer.MAX_VALUE,
                                  60L, 
                                  TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>(),
                                  threadFactory);
}
```

**不足：**

`CachedThreadPool`允许创建的线程数量为 `Integer.MAX_VALUE` ，可能会创建大量线程，从而导致 OOM。

## 6、线程池的 7 种阻塞队列

关于阻塞队列：

超出**核心线程数**的任务时，将任务加入在此任务**阻塞队列** 。

当任务队列塞满时，创建新的线程执行任务，直到线程数达到**最大线程数**，触发**拒绝策略**。

|        队列名称         |                 结构与读写                 |               大小               |
| :---------------------: | :----------------------------------------: | :------------------------------: |
| **ArrayBlockingQueue**  |   基于 **数组** 的 **先进先出** 有界队列   |            创建需指定            |
| **LinkedBlockingQueue** | 基于 **链表** 的 **先进先出** 可选有界队列 | 默认 `Integer.MAX_VALUE`，可指定 |
|  **SynchronousQueue**   |         **不存储元素** 的阻塞队列          |          0，插入会阻塞           |
|  **DelayedWorkQueue**   |     基于 **堆(完全二叉树)** 的无界队列     |                                  |
|  PriorityBlockingQueue  |       支持 **优先级排序** 的无界队列       |                                  |
|   LinkedTransferQueue   |          基于 **链表** 的无界队列          |                                  |
|   LinkedBlockingDeque   |       基于 **链表** 的 **双向** 队列       |                                  |

## 7、线程池大小确定

自定义线程池就需要我们自己配置最大线程数 `maximumPoolSize` ，为了高效的并发运行，这时需要看我们的业务是 **IO密集型** 还是 **CPU密集型**。

#### IO 密集型

该任务需要大量的IO，即大量的阻塞。

在单线程上运行IO密集型的任务会导致大量的CPU运算能力浪费在等待。

所以在IO密集型任务中使用多线程可以大大的加速程序运行，即使在单核CPU上这种加速主要就是利用了被浪费掉的阻塞时间

IO 密集型时，大部分线程都阻塞，故需要多配制线程数。公式为：

+ `CPU核数 * 2` ；
+ `CPU核数 / (1 - 阻塞系数)` ，阻塞系数在0.8~0.9之间。

#### CPU 密集型

CPU密集的意思是该任务需要最大的运算，而没有阻塞，CPU一直全速运行。

CPU密集任务只有在真正的多核CPU上才能得到加速(通过多线程)。而在单核CPU上，无论你开几个模拟的多线程该任务都不可能得到加速，因为CPU总的运算能力就那么多。

> 更多与线程池相关的实战，参考：
>
> [https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)





多线程同步怎么做？