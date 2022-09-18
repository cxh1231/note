## 1、线程的 3 种实现方式

通常来说，可以认为有三种方式：

1. 继承 `Thread` 类；
2. 实现 `Callable` 接口；
3. 实现 `Runnable` 接口。

其中，`Thread` 其实是实现了 `Runable` 接口。

> 还有一说法，可以通过**线程池**创建，其**线程池**也就是封装了上面的方法。

`Runnable` 和 `Callable` 的区别：

|   区别   |           Callable           | Runnable |
| :------: | :--------------------------: | :------: |
| 重写方法 |           `call()`           | `run()`  |
|   异常   |           可以抛出           |    无    |
|  返回值  |           有返回值           |    无    |
| 返回对象 | Future对象，表示异步计算结果 |    无    |

## 2、Thread 类调用 start() 方法的流程

1. 当 `new` 一个 `Thread` 时，线程即进入 **新建** （NEW）状态；
2. 当 `Thread` 调用 `start()` 方法时，会启动一个线程，并进入 **就绪** （READY）状态；
3. 当 该线程分配到 CPU 时间片时，就开始 **运行**（RUNNING）。

> `start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。

## 3、Thread 类调用 start() 和 run() 方法的区别

**先说结论**

`Thread.run()` 方法：就是**普通的方法调用**，即在 Main 线程中调用 `run()` 方法，不新建线程；

`Thread.star()` 方法：启动一个**新线程执行 run() 方法**。次数该线程处于就绪（可运行）状态，并没有运行，一旦得到 CPU 时间片，就开始执行 `run()` 方法。

**代码示例：**

`Thread.run()` 方法：

```java
public class ThreatTest {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            System.out.println("A");
        });
        // 调用 run 方法
        thread.run();
        System.out.println("B");
    }
}
/**
A
B
*/
```

`Thread.star()` 方法：

```java
public class ThreatTest {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            System.out.println("A");
        });
        // 调用 start 方法
        thread.start();
        System.out.println("B");
    }
}
/**
B
A
*/
```

## 4、Thread 类调用 sleep() 和 yield() 方法的区别

`Thread.sleep()` 方法：线程进入 **超时等待**（TIMED_WAITING）状态；换言之，即暂停当前线程**让出 CPU**，给其他线程运行的机会，**不考虑优先级**。

`Thread.yield()` 方法：线程进入 **就绪**（READY）状态。只给相同或更高优先级的线程运行机会。也就是说，该方法**只能保证当前线程放弃 CPU 占用，但不能保证其他线程能占用 CPU**，当前线程也可能进入暂停状态后马上被执行。

## 5、Thread.sleep() 和 class.wait() 方法的区别

两者都可以**暂停线程的执行**，使线程进入**等待**（`WAITING`）或**超时等待**（`TIMED_WAITING`）状态。

但其详细区别如下：

|      区别点      |             sleep()              |                            wait()                            |
| :--------------: | :------------------------------: | :----------------------------------------------------------: |
|       来源       |      `Thread` 类的静态方法       |                    `Object` 类的成员方法                     |
|    **锁释放**    |             没有释放             |                           释放了锁                           |
|       用途       |             暂停执行             |                      线程间交互 / 通信                       |
| 唤醒 / 恢复方式  |         时间结束自动恢复         | 其他线程调用同一对象的 `notify()/nofityAll()` 才能重新恢复，或使用 `wait(long timeout)` 超时后线程会自动苏醒。 |
|     使用范围     |             任何地方             | **同步控制方法或者同步控制块内**，否则抛出 `IllegalMonitorStateException`异常 |
| **同步锁的影响** | 没有影响，已拥有的**锁不会释放** |   **释放同步锁**，让其他线程进入 `synchronized` 代码块执行   |

代码示例：

```java
class SleepTest implements Runnable {

    @Override
    public void run() {
        try {
            System.out.println("sleep 开始并睡眠 5 秒");
            // 调用 sleep 方法，休眠5秒
            Thread.sleep(5000);
            System.out.println("sleep 结束睡眠");
            // 唤醒 wait
            synchronized (WaitTest.class) {
                System.out.println("唤醒 wait");
                WaitTest.class.notify();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class WaitTest implements Runnable {

    @Override
    public void run() {
        synchronized (WaitTest.class) {
            try {
                System.out.println("wait 开始并等待");
                // 调用 wait 方法，持续等待
                WaitTest.class.wait();
                // 调用 wait 方法，等待时间超过 5 秒，自动唤醒
                // this.wait(5000);
                System.out.println("wait 被唤醒并结束");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class SleepWaitTest {
    public static void main(String[] args) {
        // sleep 使用示例
        Thread thread = new Thread(new SleepTest());
        thread.start();
        // wait 使用示例
        Thread thread1 = new Thread(new WaitTest());
        thread1.start();
    }
}
/**
sleep 开始并睡眠 5 秒
wait 开始并等待
sleep 结束睡眠
唤醒 wait
wait 被唤醒并结束
*/
```

## 6、线程的 join() 方法的使用

用于等待当前线程终止，即使当前 **主线程** 进入 **等待**（WAITING）状态。

> 注意：这里的**主线程**，不是 Main 方法，而是**调用了其他子线程的线程**。

一句话：**让当前线程等待子线程执行结束之后再运行当前线程**。

即：如果一个 **线程A** 执行了 `threadB.join()` 语句，其含义是：当前 **线程A**等待 `threadB` 线程终止之后才从 `threadB.join()` 的位置返回，继续往下执行 **线程A** 的代码。

代码示例：

```java
class MyThread_A extends Thread {

    public void run() {
        System.out.println("A 线程开始执行");
        // 创建子线程 B 并执行
        MyThread_B thread_b = new MyThread_B();
        thread_b.start();
        try {
            // 调用子线程 B 的 join 方法，等待子线程 B 结束
            thread_b.join();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        System.out.println("A 线程执行结束");
    }
}

class MyThread_B extends Thread {

    public void run() {
        for (int i = 1; i < 4; i++) {
            // 休眠 2 秒后打印
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println("B - " + i);
        }
    }
}

public class JoinTest {
    public static void main(String[] args) throws InterruptedException {
        MyThread_A thread_a = new MyThread_A();
        thread_a.start();
    }
}
/**
 A 线程开始执行
 B - 1
 B - 2
 B - 3
 A 线程执行结束
 */
```

> `thread.join()` 方法只会使主线程进入等待池并等待 `thread` 线程执行完毕后才会被唤醒，并不影响同一时刻处在运行状态的其他线程。
>
> 比如上面的 Main 方法中，再来个 C 线程，则不影响 C 线程的运行。