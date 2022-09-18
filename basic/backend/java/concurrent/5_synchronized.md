## 1、synchronized 简介

`synchronized` 关键字是**同步**的意思，主要**解决多个线程之间访问资源的同步性**，`synchronized`关键字可以保证被它修饰的方法或者代码块**在任意时刻只能有一个线程执行**。

早期的 `synchronized` 属于 **重量级锁**，效率低下。但 Java 6 ，Java官方对其进行优化，引入了如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。

## 2、synchronized 的使用方式

`synchronized` 关键字最主要的三种使用方式：

1. 修饰**实例方法**
2. 修饰**静态方法**
3. 修饰**代码块**

#### 方式 1：修饰实例方法

> 锁当前对象实例

给当前对象实例加锁，进入同步代码前要获得 **当前对象实例的锁** 。

```java
synchronized void method() {
    //业务代码
}
```

> 注意：**构造方法不能使用 synchronized 关键字修饰。**构造方法本身就属于线程安全的，不存在同步的构造方法一说。

#### 方式 2：修饰静态方法

> 锁当前类

给当前类加锁，会作用于类的所有对象实例 ，进入同步代码前要获得 **当前 class 的锁**。

```java
synchronized static void method() {
    //业务代码
}
```

#### 方式 3：修饰代码块

> 锁指定对象/类

对括号里指定的对象/类加锁：

- `synchronized(object)` 表示进入同步代码库前要获得 **给定对象的锁**。
- `synchronized(类.class)` 表示进入同步代码前要获得 **给定 Class 的锁**

```java
synchronized(this) {
    //业务代码
}
```

> 尽量不要使用 `synchronized(String a)`，因为 JVM 中，字符串常量池具有缓存功能。

## 3、synchronized 的使用示例

下面是一个使用的示例。

```java
class MyThread extends Thread {

    String name;

    MyThread(String name) {
        this.name = name;
    }

    public void run() {
        // 同步代码块
        synchronized (MyThread.class) {
            System.out.println(this.name + " 线程开始");
            // 休眠 2 秒后打印
            for (int i = 1; i < 4; i++) {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                System.out.println(this.name + " - " + i);
            }
            System.out.println(this.name + " 线程结束");
        }
    }
}

public class SynchronizedTest {
    public static void main(String[] args) {
        MyThread thread_a = new MyThread("A");
        MyThread thread_b = new MyThread("B");
        thread_a.start();
        thread_b.start();
    }
}
/**
A 线程开始
A - 1
A - 2
A - 3
A 线程结束
B 线程开始
B - 1
B - 2
B - 3
B 线程结束
*/
```

## 4、synchronized 的底层原理

> `synchronized` 关键字底层原理属于 JVM 层面。

#### 修饰同步语句块

如代码：

```java
public class SynchronizedDemo {
    public void method() {
        synchronized (this) {
            System.out.println("synchronized 代码块");
        }
    }
}
```

其在 JVM 中的字节码为：

![image-20220801162432996](https://img.zxdmy.com/2022/202208011624904.png)

可以看出：**`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。**

当执行 `monitorenter` 指令时，线程试图获取锁也就是获取 **对象监视器 `monitor`** 的持有权。

在执行`monitorenter`时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。

![image-20220801163059853](https://img.zxdmy.com/2022/202208011631456.png)

对象锁的的拥有者线程才可以执行 `monitorexit` 指令来释放锁。在执行 `monitorexit` 指令后，将锁计数器设为 0，表明锁被释放，其他线程可以尝试获取锁。

![image-20220801163103539](https://img.zxdmy.com/2022/202208011631595.png)



如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

#### 修饰方法

如代码：

```java
public class SynchronizedDemo2 {
    public synchronized void method() {
        System.out.println("synchronized 方法");
    }
}
```

其在 JVM 中的字节码为：

![image-20220801162455652](https://img.zxdmy.com/2022/202208011624507.png)

`synchronized` 修饰的方法并没有 `monitorenter` 指令和 `monitorexit` 指令，取得代之的确实是 `ACC_SYNCHRONIZED` 标识，该标识指明了该方法是一个同步方法。JVM 通过该 `ACC_SYNCHRONIZED` 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

如果是实例方法，JVM 会尝试获取实例对象的锁。如果是静态方法，JVM 会尝试获取当前 class 的锁

#### 总结

`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。

`synchronized` 修饰的方法并没有 `monitorenter` 指令和 `monitorexit` 指令，取得代之的确实是 `ACC_SYNCHRONIZED` 标识，该标识指明了该方法是一个同步方法。

**不过两者的本质都是对对象监视器 monitor 的获取。**

> synchronized 的锁优化

## 5、synchronized 与 JMM 的原子性、可见性、有序性

`JMM` 的三大特性：

+ 原子性 - 保证指令不会受到线程上下文切换的影响
+ 可见性 - 保证指令不会受 cpu 缓存的影响
+ 有序性 - 保证指令不会受 cpu 指令并行优化的影响

#### synchronized 和 原子性

`synchronized` 通过 `悲观锁` 机制保证代码块中的代码的`原子性`，即要么全部执行完再由其他线程执行，要么都不执行。

#### synchronized 和 可见性

在JMM规范中，关于`synchronized`有如下规定：

线程加锁时，必须清空工作内存中共享变量的值，从而使用共享变量时需要从主内存重新读取；

线程在解锁时，需要把工作内存中最新的共享变量的值写入到主存，以此来保证共享变量的`可见性`。

#### synchronized 和 有序性

首先，`synchronized` 并不能保证代码块中不会发生指令重排，但是我们知道，`as-if-serial`语义中不管怎么重排序（编译器和处理器为了提高并行度），单线程程序的执行结果都不能被改变。编译器和处理器无论如何优化，都必须遵守`as-if-serial`语义。

同一时间只能被同一线程访问，那么也就是单线程执行的。所以，可以保证其`有序性`。

