## 1、简介

`ThreadLocal`，即 **线程本地变量**。

如果创建了一个ThreadLocal变量，那么访问这个变量的每个线程都会有这个变量的一个本地拷贝，多个线程操作这个变量的时候，实际是操作自己本地内存里面的变量，从而起到线程隔离的作用，避免了线程安全问题。

```java
// 创建一个ThreadLocal变量
static ThreadLocal<String> localVariable = new ThreadLocal<>();
```

应用场景：

- 数据库连接池
- 会话管理中使用

## 2、实现原理

- `Thread` 类有一个类型为`ThreadLocal.ThreadLocalMap` 的实例变量 `threadLocals`，即每个线程都有一个属于自己的 `ThreadLocalMap`。
- `ThreadLocalMap` 内部维护着 `Entry` 数组，每个`Entry`代表一个完整的对象，`key` 是 `ThreadLocal` 本身，`value` 是 `ThreadLocal` 的泛型值。
- 每个线程在往`ThreadLocal`里设置值的时候，都是往自己的`ThreadLocalMap`里存，读也是以某个`ThreadLocal`作为引用，在自己的`map`里找对应的`key`，从而实现了线程隔离。

![image-20220815155259875](https://img.zxdmy.com/2022/202208151553447.png)

## 3、内存泄露问题

> 内存泄漏：不会被使用的对象或者变量无法被回收。

通过 `ThreadLocal` 实现原理可以看出，`ThreadLocalMap` 对象的 `Entry` 中 `value` 是强引用，`key` 为弱引用。

当 `CG` 回收 `ThreadLocal` 后，由于 `value` 是强引用，无法被回收，所以造成了内存泄漏。

即：由于`ThreadLocalMap`的生命周期跟`Thread`一样长，如果没有手动删除对应的`key`就会导致内存泄漏，而并不是因为弱引用。

解决：

+ 每次使用完 `ThreadLocal` 对象后都调用它的 `remove()` 的方法清除数据；

+ 将`ThreadLocal`引用定义为`private static`，这样就一直存在`ThreadLocal`的强引用，也就保证能通过`ThreadLocal`弱引用访问到`Entry`中的`value`值，进而清除数据。

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

