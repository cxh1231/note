## 1、简介

`ThreadLocal`，即 **线程本地变量**。

如果创建了一个`ThreadLocal`变量，那么访问这个变量的每个线程都会有这个变量的一个本地拷贝，多个线程操作这个变量的时候，实际是**操作自己本地内存里面的变量**，从而起到**线程隔离**的作用，避免了线程安全问题。

```java
// 创建一个ThreadLocal变量，任何线程都可以并发访问
public static ThreadLocal<String> localVariable = new ThreadLocal<>();
// 写入，线程可以在任何地方使用、写入
localVariable.set("张三");
// 读取，线程在任何地方读取的都是本线程写入的变量
localVariable.get();
```

应用场景：

- 数据库连接池
- 会话管理中使用

## 2、实现原理

`ThreadLocal` 的 `set()` 方法如下：

![image-20221005205337743](https://img.zxdmy.com/2022/202210052053844.png)

该方法的操作很简单：

1. 获取到当前线程 `t`；
2. 获取当前线程的 `ThreadLocalMap` ；
3. 最后再将元素 `value` 存入该 `map` 中。

其实，`ThreadLocal` 实现线程隔离的关键点，都在这个 `ThreadLocal.ThreadLocalMap` 中。

`Thread` 类中，定义了一个类型为 `ThreadLocalMap` 的成员变量 `threadLocals`。

![image-20221005210147793](https://img.zxdmy.com/2022/202210052101765.png)

`ThreadLocalMap` 以 `Map` 结尾，必然是一个 `key-value` 型的数据结构，其节点结构如下：

![image-20221005210531337](https://img.zxdmy.com/2022/202210052105261.png)

通过如上代码可以简单看出，`key` 为 `ThreadLocal`，`value` 为 代码中写入的**元素**。

但是，`key` 并不是 `ThreadLocal` 本身，而是 `ThreadLocal` 的一个**弱引用**，通过上面的代码也可以看出，`Entry` 继承自 `WeakReference`。

所以，`key` 的赋值，使用的是 `WeakReference` 的赋值。

![image-20220815155259875](https://img.zxdmy.com/2022/202208151553447.png)

**总结**：

- `Thread` 类有一个类型为 `ThreadLocalMap` 的实例变量 `threadLocals`，即每个线程都有一个属于自己的 `ThreadLocalMap`。
- `ThreadLocalMap` 内部维护着 `Entry` 数组，每个`Entry`代表一个完整的对象，`key` 是 `ThreadLocal` 的 **弱引用**，`value` 是 `ThreadLocal` 的泛型值。
- 每个线程在往`ThreadLocal`里设置值的时候，都是往自己的`ThreadLocalMap`里存，读也是以某个`ThreadLocal`作为引用，在自己的`map`里找对应的`key`，从而实现了**线程隔离**。
- `ThreadLocal` 本身不存储值，它只是作为一个`key`来让线程往 `ThreadLocalMap` 里存取值。

## 3、内存泄露问题

> 内存泄漏：不会被使用的对象或者变量无法被回收。

已知 `JVM` 中，**栈**内存是线程私有的，存储了对象的引用；**堆**内存线程共享，存储了对象的实例。

对于 `ThreadLocal` 来说，栈 中存储了 `ThreadLocal`、`Thread` 的**引用**，堆中存储了它们的**具体实例**。

![image-20221005212804853](https://img.zxdmy.com/2022/202210052128893.png)

通过 `ThreadLocal` 实现原理可以看出，`ThreadLocalMap` 对象的 `Entry` 中 `value` 是强引用，`key` 为 `ThreadLocal` 的 **弱引用**。

> 弱引用：只要垃圾回收机制运行，不管JVM的内存空间是否充足，都会回收该对象占用的内存。

当 `JVM` 回收 `ThreadLocal` 时：

+ 由于 `value` 是强引用，无法被回收；
+ 而 `ThreadLocal`（`ThreadLocalMap` 中的 `Key`）很容易被垃圾回收器回收。

同时，由于 `ThreadLocalMap` 的生命周期跟 `Thread` 一样长，不会被回收，就会出现`ThreadLocalMap`的`key` 没了，`value` 还在的情况，这就造成了 **内存泄露**。

**解决内存泄露**：

+ 每次使用完 `ThreadLocal` 对象后都调用它的 `remove()` 的方法清除数据；

+ 将 `ThreadLocal` 引用定义为 `private static`，这样就一直存在 `ThreadLocal` 的强引用，也就保证能通过 `ThreadLocal` 弱引用访问到 `Entry` 中的 `value` 值，进而清除数据。

> 为什么 `key` 要设计成**弱引用**？
>
> `key` 设计成 **弱引用** 同样是为了 **防止内存泄漏**。
>
> 假如 `key` 被设计成**强引用**，如果 `ThreadLocal Reference` 被销毁，此时它指向`ThreadLocal` 的 **强引用** 就没有了，但是此时`key`还 **强引用** 指向`ThreadLocal`，就会导致`ThreadLocal`不能被回收，这时候就发生了 **内存泄漏** 的问题。

## 4、结构、冲突与扩容

### 4.1 ThreadLocalMap 的结构

`ThreadLocalMap` 的结构与 `HashMap` 类似，主要有 **元素数组** 和 **散列方法** 两个要素。

![image-20221005214136392](https://img.zxdmy.com/2022/202210052141680.png)

#### 元素数组

一个 `table` 数组，存储 `Entry` 类型的元素。

```java
private Entry[] table;
```

`Entry` 是以 `ThreaLocal` 弱引用作为 `key`，`Object` 作为 `value` 的结构。

#### 散列方法

**散列方法** 就是怎么把对应的`key`映射到table数组的相应下标。

`ThreadLocalMap`用的是**哈希取余法**，取出 `key` 的 `threadLocalHashCode`，然后和`table`数组长度减一`&` 运算（相当于取余）。

```java
int i = key.threadLocalHashCode & (table.length - 1);
```

这里的 `threadLocalHashCode` 的计算有技术含量。

每创建一个 `ThreadLocal` 对象，`threadLocalHashCode` 就会新增`0x61c88647` ，这个值很特殊，它是 **斐波那契数** 也叫 **黄金分割数**。

hash 增量为这个数字，带来的好处就是 hash 分布非常均匀。

```java
private static final int HASH_INCREMENT = 0x61c88647;

private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```

### 4.2 ThreadLocalMap 的哈希冲突解决

`ThreadLocalMap` 使用 **开放定址法** 来解决冲突。

> **开放定址法** 简单来说，就是这个坑被占了，那就接着去找空着的坑。

在`get`的时候，也会根据`ThreadLocal`对象的`hash`值，定位到`table`中的位置，然后判断该槽位`Entry`对象中的`key`是否和`get`的`key`一致，如果不一致，就判断下一个位置。

### 4.3 ThreadLocalMap 的扩容机制

在 `ThreadLocalMap.set()` 方法的最后，如果执行完启发式清理工作后：

+ 未清理到任何数据；
+ 且当前散列数组中 `Entry` 的数量已经达到了列表的扩容阈值 `(len * 2 / 3)` ；

就开始执行 `rehash()` 逻辑：

```java
if (!cleanSomeSlots(i, sz) && sz >= threshold)
    rehash();
```

接下来是执行 `rehash()` 方法：

+ 先清理过期的Entry，
+ 然后还要根据条件判断 `size >= threshold - threshold / 4` 也就是 `size >= threshold* 3/4` ，来决定是否需要扩容。

```java
private void rehash() {
    //清理过期Entry
    expungeStaleEntries();
    //扩容
    if (size >= threshold - threshold / 4)
        resize();
}
```

接下来是执行 `resize()` 方法：

+ 先将新的 `table` 数组扩容为原来数组的 `2` 倍；
+ 然后遍历**老**的`table`数组，通过散列方法重新计算位置，开放地址解决冲突，
+ 然后放到新的 `table`；
+ 遍历完成之后， `oldTable` 中所有的`entry` 数据都已经放入到 `newTable` 中了，然后`table`引用指向 `newTable`。

![image-20221005215314110](https://img.zxdmy.com/2022/202210052153137.png)

## 5、父子线程共享数据

父线程 **不能** 用 `ThreadLocal` 来给子线程传值。

但是，可以通过另一个类 `InheritableThreadLocal` 实现父子线程的数据共享。

使用起来很简单，在主线程的 `InheritableThreadLocal` 实例设置值，在子线程中就可以拿到了。

其原理是，在`Thread`类里还有另外一个变量：

```java
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

在 `Thread.init()` 的时候，如果**父线程**的 `inheritableThreadLocals` 不为空，就把它赋给当前线程（子线程）的`inheritableThreadLocals` 。

![image-20221005215852658](https://img.zxdmy.com/2022/202210052158798.png)

## 4、使用示例

```java
public class ThreadLocalTest {

    // 创建 ThreadLocal 对象
    public static ThreadLocal<Integer> localVariable = new ThreadLocal<>();

    static {
        localVariable.set(1);
    }

    public static class RunnableImpl implements Runnable {

        @Override
        public void run() {
            // 生成1000到5000的随机值
            int random = (int) (Math.random() * (5000 - 1000 + 1)) + 1000;
            // 设置 ThreadLocal 对象的值
            localVariable.set(random);
            // 随机睡眠
            try {
                Thread.sleep(random);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 输出 ThreadLocal 对象的值
            System.out.println(Thread.currentThread().getName() + "生成的随机值为：" + localVariable.get());
            // 使用完成后，及时调用 该方法释放内存，以防止内存泄露
            localVariable.remove();
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 3; i++) {
            new Thread(new RunnableImpl()).start();
        }
        System.out.println(localVariable.get());
    }
}
```

输出结果：

```
1
Thread-2生成的随机值为：1313
Thread-0生成的随机值为：2353
Thread-1生成的随机值为：4251
```

