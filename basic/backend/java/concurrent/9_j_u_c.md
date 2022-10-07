> 本文的内容，都是 `java.util.concurrent` 包下的。

## 1、AQS

### 1.1 AQS 简介

`AQS` 的全称为 `AbstractQueuedSynchronizer` ，翻译过来的意思就是**抽象队列同步器**。这个类在 `java.util.concurrent.locks` 包下面。

![image-20220815202754761](https://img.zxdmy.com/2022/202208152027352.png)

`AQS` 就是一个**抽象类**，主要 **用来构建锁和同步器**。

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable {
}
```

`AQS` 为构建 **锁** 和 **同步器** 提供了一些 **通用功能的是实现**。

并发包中的 **锁**，就是基于 `AQS` 实现的。

因此，使用 `AQS` 能简单且高效地构造出应用广泛的大量的同步器，比如后续的 `ReentrantLock`，`Semaphore`，其他的诸如 `ReentrantReadWriteLock`，`SynchronousQueue`，`FutureTask` (jdk1.7) 等等皆是基于 `AQS` 的。

### 1.2 AQS 的结构与原理

`AQS`是基于一个`FIFO`的**双向队列**，其内部定义了一个节点类 `Node`，`Node` 节点内部的：

+ `SHARED`【共享】用来标记该线程是 **获取共享资源时被阻塞挂起** 后放入`AQS` 队列的
+ `EXCLUSIVE` 【独占】用来标记线程是 **获取独占资源时被挂起** 后放入`AQS` 队列

`AQS` 使用一个由 `volatile` 修饰的 `int` 类型的成员变量 `state` 来表示 **同步状态**，修改同步状态成功即为获得锁。

![image-20221007204104596](https://img.zxdmy.com/2022/202210072041658.png)

获取`state`的方式分为两种，**独占方式**和**共享方式**：

+ 一个线程使用**独占方式**获取了资源，其它线程就会在获取失败后被阻塞。
+ 一个线程使用**共享方式**获取了资源，另外一个线程还可以通过`CAS`的方式进行获取。

如果**共享资源**被占用，需要一定的**阻塞等待唤醒机制**来保证锁的分配，`AQS` 中会将竞争共享资源失败的线程添加到一个变体的 `CLH` 队列中。

`CLH` 队列，是 **单向链表** 实现的 **队列**。

申请线程只在本地变量上自旋，其**不断轮询前驱的状态**，如果发现 **前驱节点释放了锁就结束自旋**。

![image-20221007204145836](B-9、并发包类 JUC.assets/image-20221007204145836.png)

`AQS` 中的队列是 `CLH` 变体的**虚拟双向队列**，通过**将每条请求共享资源的线程封装成一个节点来实现锁的分配**：

![image-20221007204335546](B-9、并发包类 JUC.assets/image-20221007204335546.png)

![image-20220815202958637](https://img.zxdmy.com/2022/202208152029944.png)

`AQS` 中的 `CLH` **变体** **等待队列** 拥有以下 **特性**：

+ `AQS` 中队列是个 **双向链表**，也是 `FIFO` 先进先出的特性；
+ 通过 `Head`、`Tail` 头尾两个节点来组成队列结构，通过 `volatile` 修饰保证可见性；
+ `Head` 指向节点为 **已获得锁的节点**，是一个虚拟节点，节点本身不持有具体线程；
+ 获取不到同步状态，会将节点进行**自旋**获取锁，自旋一定次数失败后会将线程**阻塞**，相对于 `CLH` 队列性能较好。

## 2、Lock 接口及其实现类（ReentrantLock等）

> https://blog.csdn.net/xyy1028/article/details/107333451

### 2.1 简介

`java.util.concurrent.locks.Lock` 是一个类似于`synchronized` 关键字的线程同步机制。

`Lock` 比 `synchronized` 块更加灵活。

`Lock` 是个**接口**，其 **实现类** 有 `ReentrantLock` ，**内部类** 有`ReentrantReadWriteLock.ReadLock`，`ReentrantReadWriteLock.WriteLock` 。

`ReetrantLock` 是一个可重入的**独占锁**，主要有两个特性：

1. 支持公平锁和非公平锁
2. 支持重入锁

`ReetrantLock` 实现依赖于 `AQS`（AbstractQueuedSynchronizer），即依靠 `AQS` 维护一个**阻塞队列**，多个线程对共享资源加锁时，失败的线程就会进入**阻塞队列**。等待唤醒，重新尝试加锁。

### 2.2 常用方法

**获取锁的方法**：

+ `lock()` ：用来**获取锁**。如果锁已被其他线程获取，则进行等待。
+ `tryLock()` ：用来**尝试获取锁**。如果获取成功，则返回 **true**，如果获取失败（即锁已被其他线程获取），则返回 **false** 。
+ `tryLock(long time, TimeUnit unit)` ：如果一开始或在**等待时间内拿到锁**，则返回 **true**，否则会等待一段时间，直至时间结束，返回 **false**。
+ `lockInterruptibly()` ：如果该线程正在等待获取锁，可以通过 `thread.interrupt()` 中断该线程的等待状态。（即**可中断锁**）

**释放锁的方法**：

+ `unLock()` 方法来释放锁，一般放在 `finally` 块中，以保证锁被释放。

```java
    private static final Lock lock = new ReentrantLock();

    public static void main(String[] args) {
        // 加锁
        lock.lock();
        try {
            System.out.println("获取到了锁");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            System.out.println("释放了锁");
            // 释放锁
            lock.unlock();
        }
    }
```

### 2.3 ReentrantLock 的高级功能

`ReentrantLock` 是可重入的**独占锁**，只能有一个线程可以获取该锁，其它获取该锁的线程会被阻塞而被放入该锁的阻塞队列里面。

#### 等待可中断

指当前有锁的线程长时间不释放锁时，正在等待的线程可以选择放弃等待，改做其他事情。

可中断特性对处理执行时间非常长的同步块很有帮助。

**主要通过 `lock.lockInterruptibly()` 来实现这个机制**。

#### 可实现公平锁

+ **公平锁** 指多个线程等待同一个锁时，必须 **按照申请锁的时间顺序依次获得锁**。
+ **非公平锁** 则在锁被释放时，任何一个等待锁的线程**都有机会获得锁**。

`new ReentrantLock()` **构造函数** 默认创建的是**非公平锁** `NonfairSync()`。

**可以使用 `ReentrantLock` 类的 `ReentrantLock(boolean fair)` 构造方法来制定是否是公平的**，进而创建公平锁 `FairSync()`。

`FairSync()`、`NonfairSync()` 代表 **公平锁** 和 **非公平锁**，两者都是 `ReentrantLock` **静态内部类**，只不过 **实现不同锁语义**。

> 注意：
>
> **非公平锁**在调用 `lock` 后，首先就会调用 `CAS` 进行一次抢锁，如果这个时候恰巧锁没有被占用，那么直接就获取到锁返回了，性能较高。
>
> 如果使用了**公平锁**，会判断等待队列是否有线程处于等待状态，有则不抢锁，排到队列中。这将导致 **吞吐量明显降低**， `ReentrantLock` 的性能急剧下降。

#### 锁绑定多个条件（实现选择性通知）

指的是 `ReentrantLock` 对象可以同时绑定多个 `Condition` 对象。

**用ReentrantLock类结合Condition实例可以实现“选择性通知”**

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ConditionTest {

    public static void main(String[] args) throws InterruptedException {
        ConditionService conditionService = new ConditionService();

        ThreadA a = new ThreadA(conditionService);
        a.setName("A");
        a.start();
        ThreadB b = new ThreadB(conditionService);
        b.setName("B");
        b.start();
        Thread.sleep(3000);
        // 唤醒 A 线程
        conditionService.signalA();
    }
}

class ConditionService {

    private static final Lock lock = new ReentrantLock();

    public Condition conditionA = lock.newCondition();

    public Condition conditionB = lock.newCondition();

    public void awaitA() {
        lock.lock();
        try {
            System.out.println("准备调用[conditionA.await()]方法，当前线程将被阻塞");
            conditionA.await();
            System.out.println("已调用[conditionA.signalAll()]方法，此时 awaitA 被唤醒");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void awaitB() {
        lock.lock();
        try {
            System.out.println("准备调用[conditionB.await()]方法，当前线程将被阻塞");
            conditionB.await();
            System.out.println("已调用[conditionB.signalAll()]方法，当前 awaitB 被唤醒");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void signalA() {
        lock.lock();
        try {
            System.out.println("准备唤醒 conditionA 下的所有线程");
            conditionA.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void signalB() {
        lock.lock();
        try {
            System.out.println("准备唤醒 conditionB 下的所有线程");
            conditionB.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}


class ThreadA extends Thread {

    private ConditionService conditionService;

    public ThreadA(ConditionService conditionService) {
        this.conditionService = conditionService;
    }

    @Override
    public void run() {
        conditionService.awaitA();
    }
}

class ThreadB extends Thread {

    private ConditionService conditionService;

    public ThreadB(ConditionService conditionService) {
        this.conditionService = conditionService;
    }

    @Override
    public void run() {
        conditionService.awaitB();
    }
}
```

### 2.4 synchronized 与 Lock 的异同

|  区别点  |           synchronized 关键词            |         Lock / ReentrantLock 接口          |
| :------: | :--------------------------------------: | :----------------------------------------: |
|  锁类型  |                 可重入锁                 |                  可重入锁                  |
|  锁实现  |      基于 **JVM**，对象头监视器模式      |          基于 **API**，依赖 `AQS`          |
| 等待中断 |                 不可中断                 |   `lock.lockInterruptibly()` 可实现中断    |
|  公平锁  |                 非公平锁                 | `ReentrantLock(boolean fair)` 可实现公平锁 |
| 关联通知 |         关联一个通知：`notify()`         | 关联多个通知：多次调用 `newCondition` 实现 |
|   用法   |   给 **类**、**方法**、**代码块** 加锁   |           只能给 **代码块** 加锁           |
| 锁的取舍 | **自动**获取和释放锁，异常释放锁，无死锁 |    **手动**加锁和释放锁，处理不当会死锁    |
|  锁状态  |             无法判断锁的状态             |   可通过 `tryLock()` 验证线程是否拿到锁    |
| 读读操作 |                 依次进行                 |   `ReentrantReadWriteLock()` 可多线程读    |

**关于读读操作**：

`ReadWriteLock` 接口的实现类 `ReentrantReadWriteLock` **读写锁**提供了两个方法：`readLock()` 和 `writeLock()` 用来获取**读锁**和**写锁**，也就是说**将文件的读写操作分开**，分成2个锁来分配给线程，从而使得多个线程可以同时进行读操作。

- `writeLock()`：获取写锁。
- `readLock()`：获取读锁。

**多个读锁之间不互斥**，读锁与写锁互斥，写锁与写锁互斥。

## 3、Atomic（原子类）

### 3.1 原子类的简介与类型

`Atomic` （英：原子的）是指一个**操作是不可中断的**。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。

所以，**所谓原子类说简单点就是具有原子 / 原子操作特征的类**。

并发包 `java.util.concurrent` 的原子类都存放在 `java.util.concurrent.atomic` 下：

![image-20220815190343685](https://img.zxdmy.com/2022/202208151903612.png)

`Atomic`包里的类基本都是使用`Unsafe`实现的**包装类**。

根据 **操作的数据类型**，可以将 `JUC` 包中的**原子类**分为 4 类。

**基本类型**：使用原子的方式更新基本类型

+ `AtomicInteger`：整型原子类
+ `AtomicLong`：长整型原子类
+ `AtomicBoolean` ：布尔型原子类

**数组类型**：使用原子的方式更新数组里的某个元素

- `AtomicIntegerArray`：整型数组原子类
- `AtomicLongArray`：长整型数组原子类
- `AtomicReferenceArray` ：引用类型数组原子类

**引用类型**：如果要原子更新多个变量，则使用原子更新引用类型提供的类

- `AtomicReference`：引用类型原子类
- `AtomicMarkableReference`：原子更新带有标记的引用类型。该类将 boolean 标记与引用关联起来。（不能解决 ABA 问题）
- **`AtomicStampedReference`** ：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，**可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题**。

**对象的属性修改类型**：如果需要原子地更新某个类里的某个字段，需要使用原子更新字段类

- `AtomicIntegerFieldUpdater`:原子更新整型字段的更新器
- `AtomicLongFieldUpdater`：原子更新长整型字段的更新器
- `AtomicReferenceFieldUpdater`：原子更新引用类型里的字段

### 3.2 基本类型原子类的原理与使用示例

>  基本类型原子类使用`CAS`实现。

`AtomicInteger` 类的部分源码如下：

```java
    // setup to use Unsafe.compareAndSwapInt for updates
    //（更新操作时提供“比较并替换”的作用）
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
```

`AtomicInteger` 类主要利用 `CAS` (compare and swap) + `volatile` 和 `native` 方法来保证原子操作，从而避免 `synchronized` 的高开销，执行效率大为提升。

> CAS 的原理是拿期望的值和原本的一个值作比较，如果相同则更新成新的值。

`UnSafe` 类的 `objectFieldOffset()` 方法是一个**本地方法**，这个方法是用来拿到“原来的值”的内存地址。

另外 `value` 是一个 `volatile` 变量，在**内存中可见**，因此 JVM 可以**保证任何时刻任何线程总能拿到该变量的最新值**。

再如：

以`AtomicInteger`的**添加方法**为例：

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

通过`Unsafe` 类的实例来进行**添加**操作，来看看具体的`CAS`操作：

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5,var5 + var4));
    return var5;
}
```

`compareAndSwapInt` 是一个`native`方法，基于`CAS`来操作`int`类型变量。

其它的原子操作类基本都是大同小异。

---

下面是 **多线程环境使用基本数据类型原子类保证线程安全** 的示例：

```java
public class AtomicTest {

    public static void main(String[] args) {
        // 创建10000个线程，每个线程自增1，并且输出结果
        AtomicInteger atomicInteger = new AtomicInteger(0);
        Test test = new Test();
        for (int i = 0; i < 100000; i++) {
            new Thread(() -> {
                // 自增
                atomicInteger.incrementAndGet();
                System.out.println(atomicInteger.get());
            }).start();
        }
    }
}
```

最终输出的结果是 **100000**。

### 3.3 引用类型原子类的使用示例

`AtomicReference` 类使用示例：

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReference;

public class AtomicReferenceTest {

    public static void main(String[] args) {
        AtomicReference<User> atomicReference = new AtomicReference<>();
        // 创建线程池
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(20, 40, 1000L, TimeUnit.SECONDS, new ArrayBlockingQueue<>(100));
        // 创建用户类
        User user = new User(0, "name:" + 0);
        atomicReference.set(user);
        // 多个任务并发执行
        for (int i = 1; i <= 100; i++) {
            // 设置对象属性
            user.setId(i);
            user.setName("name:" + i);
            // 添加至线程池并执行
            threadPoolExecutor.execute(() -> atomicReference.compareAndSet(user, user));
        }
        threadPoolExecutor.shutdown();
        while (!threadPoolExecutor.isTerminated()) {
        }
        // 输出结果
        System.out.println(atomicReference.get().getName());
    }
}

class User {
    int id;
    String name;
    // 省略
}
```

最终输出结果：`name:100` 。

如果将for 循环里的代码修改为如下：

```java
// 设置对象属性
// user.setId(i);
// user.setName("name:" + i);
User newUser = new User(i, "name:" + i);
// 创建一个线程并添加至线程池
threadPoolExecutor.execute(() -> atomicReference.compareAndSet(user, newUser));
```

则输出结果为：`name:1` 。

因为在第 `1` 次循环时，引用原子类 `atomicReference` 内的 `User` 的值为 `user`，与方法 `compareAndSet(user, newUser)` 的第一个值 `user` 匹配成功，则执行方法，将原子类 的 `User` 值设置为 `newUser` 。

在第 `2` 次及以后的循环中，引用原子类内的 `User` 值为 `newUser`，与方法 `compareAndSet(user, newUser)` 的第一个参数匹配失败，无法执行该方法。

### 3.4 对象的属性修改类型原子类的使用示例

要想原子地更新对象的属性需要两步。

第一步，因为 **对象的属性修改类型原子类** 都是抽象类，所以每次使用都必须使用静态方法 `newUpdater()` 创建一个更新器，并且需要**设置想要更新的类和属性**。

第二步，**更新的对象属性**必须使用 `public volatile` 修饰符。

示例如下：

```java
import java.util.concurrent.atomic.AtomicIntegerFieldUpdater;

public class AtomicIntegerFieldUpdaterTest {
    public static void main(String[] args) {
        AtomicIntegerFieldUpdater<People> atomicIntegerFieldUpdater = AtomicIntegerFieldUpdater.newUpdater(People.class, "age");
        People people = new People("张三", 18);

        System.out.println(atomicIntegerFieldUpdater.incrementAndGet(people)); // 19
        System.out.println(people.getAge()); // 19
        System.out.println(atomicIntegerFieldUpdater.getAndIncrement(people));// 19
        System.out.println(people.getAge()); // 20
        System.out.println(atomicIntegerFieldUpdater.addAndGet(people, 5));// 25

        atomicIntegerFieldUpdater.compareAndSet(people, 20, 30);
        System.out.println(people.getAge()); // 25

        atomicIntegerFieldUpdater.compareAndSet(people, 25, 35);
        System.out.println(people.getAge()); // 35
    }
}

class People {
    private String name;
    public volatile int age;

    // 以下省略
}
```

## 4、并发工具类

### 4.1 CountDownLatch（倒计数器）

`CountDownLatch`，倒计数器。

场景 1：协调子线程**结束**动作：等待所有子线程运行结束

`CountDownLatch`允许一个或多个线程等待其他线程完成操作。

场景 2：协调子线程**开始**动作：统一各线程动作开始的时机

`CountDownLatch` 的核心方法也不多：

+ `await()` ：等待 latch 降为 0；
+ `boolean await(long timeout, TimeUnit unit)` ：等待latch降为0，但是可以设置超时时间。比如有玩家超时未确认，那就重新匹配，总不能为了某个玩家等到天荒地老。
+ `countDown()` ：latch数量减1；
+ `getCount()` ：获取当前的latch数量。

### 4.2 CyclicBarrier（同步屏障）

`CyclicBarrier`的字面意思是可循环使用（Cyclic）的屏障（Barrier）。

它要做的事情是，让一组线程到达一个屏障（也可以叫**同步点**）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。

### 4.3 Semaphore（信号量）

`Semaphore`（信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源。

### 4.4 Exchanger（交换者）

`Exchanger`（交换者）是一个用于线程间协作的工具类。

Exchanger用于进行**线程间的数据交换**。

它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。

