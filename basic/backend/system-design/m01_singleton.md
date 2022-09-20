## 1、模式定义

> Ensure a class has only one instance,and provide a global point of access to it.

单例模式**确保一个类只有一个实例，而且自行实例化并向整个系统提供这个实例**。

## 2、类图

**饿汉式**：

![image-20220913104831098](https://img.zxdmy.com/2022/202209131048347.png)

**懒汉式**：

![image-20220913105644149](https://img.zxdmy.com/2022/202209131056872.png)

## 3、代码实现

> 基于Java语言

**饿汉式**：

```java
public class Singleton {
    private static Singleton m_instance = new Singleton();
    
    // 构造方法私有，保证外界无法直接实例化
    private Singleton(){
    }
    
    // 通过该方法获得实例对象
    public static Singleton getInstance() {
        return m_instance;
    }
}
```

**懒汉式**：

```java
public class Singleton {
    private static Singleton _instance = null;
    
    //构造方法私有，保证外界无法直接实例化
    private Singleton(){
    }
    
    //方法同步
    synchronized public static Singleton getInstance() {
        if (_instance == null){
            _instance = new Singleton();
        }
        return _instance;
    }
}
```

饿汉式单例类在被加载时实例化，而懒汉式单例类在第一次引用时实例化。

## 4、应用示例

下面代码中创建一个饿汉式单例类`GlobalNum`，其中`getNum()`方法用于返回访问次数，并且使用`synchronized`对该方法进行线程同步：

```java
public class GlobalNum {
    
    private static GlobalNum gn = new GlobalNum();
    
    private int num = 0;
    public static GlobalNum getInstance() {
        return gn;
    }
    public synchronized int getNum() {
        return ++num;
    }
}
```

