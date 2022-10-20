## 1、工厂模式

**工厂方法模式**（Factory Method Pattern）又叫**虚拟构造函数**（Virtual Constructor）模式或者**多态性工厂**（Polymorphic Factory）模式。

**工厂方法模式** 的用意是定义一个创建产品对象的工厂接口，**将实际创建性工作推迟到子类中**。

**工厂模式**可分为**简单工厂**、**工厂方法**和**抽象工厂**模式。

+ **简单工厂模式** 中，一个工厂类处于对产品类实例化的中心位置上，它知道每一个产品，它决定哪一个产品类应当被实例化。这个模式的优点是允许客户端相对独立于产品创建的过程，并且在系统引入新产品的时候无须修改客户端，即在某种程度上支持“开-闭”原则。这个模式的缺点是对“开-闭”原则的支持不够，因为如果有新的产品加入到系统中，则需要修改工厂类，将必要的逻辑加入到工厂类中。
+ **工厂方法模式** 是简单工厂模式的进一步抽象和推广。由于使用了多态性，工厂方法模式保持了简单工厂模式的优点，且克服了它的缺点。首先，在工厂方法模式中，核心的工厂类不再负责所有产品的创建，而是将具体创建的工作交给子类去做。这个核心类则成为一个抽象工厂角色，仅负责给出具体工厂子类必须实现的接口，而不接触哪个产品类应当被实例化的细节。此种进一步抽象化的结果，使这种工厂方法模式可以用来允许系统在不修改具体工厂角色的情况下引进新的产品，这一特点使得工厂模式具有超过简单工厂的优越性。
+ **抽象工厂模式** 是所有形态的工厂模式中最为抽象和最具有一般性的一种形态。抽象工厂模式与工厂方法模式的最大区别就在于，工厂方法模式针对的是一个产品等级结构；而抽象工厂模式则需要面对多个产品等级结构。

## 2、工厂方法模式定义

**工厂模式** 定义一个用于创建对象的接口，让子类决定实例化哪个类。工厂方法使一个类的实例化延迟到其子类。

## 2、类图

![](https://img.zxdmy.com/2022/202209131127318.jpeg)

在工厂方法模式中，共涉及以下4个角色：

+ **抽象工厂**（Creator）角色：该角色是工厂方法模式的核心，与应用系统无关，任何在创建对象的工厂类必须实现这个接口。
+ **具体工厂**（Concrete Creator）角色：该角色实现了抽象工厂接口，含有与应用密切相关的逻辑，并且受到应用程序的调用以创建产品对象。
+ **抽象产品**（Product）角色：该角色负责定义产品的共性，实现对产品最抽象的定义。
+ **具体产品**（Concrete Product）角色：该角色实现抽象产品角色所声明的接口，工厂方法模式所创建的每一个对象都是某个具体产品角色的实例

## 3、代码实现

> 基于Java语言

**抽象工厂** `Creator.java`：

```java
public interface Creator {
    /**
    * 工厂方法
    * 创建一个产品对象，其输入参数类型可以自行设置
    */
    public <T extends Product> T factory(Class<T> c);
}
```

**具体工厂**`ConcreteCreator.java`

```java
public class ConcreteCreator implements Creator {
    public <T extends Product> T factory(Class<T> c) {
        Product product = null;
        try {
            product = (Product) Class.forName(c.getName()).newInstance();
        } catch (Exception e) {
        }
        return (T) product;
    }
}
```

**抽象产品**`Product.java`：

```java
public interface Product {
    
    //产品类的公共方法
    public void method1();
    public void method2();
}
```

**具体产品**`ConcreteProduct.java`

```java
public class ConcreteProduct implements Product {
    public void method1() {
        // 业务逻辑处理代码
    }
    public void method2() {
        // 业务逻辑处理代码
    }
}
```

**应用代码** `Demo.java`：

```java
public class Demo {
    public static void main(String args[]) {
        Creator creator=new ConcreteCreator();
        Product product=creator.factory(ConcreteProduct.class);
        /* 继续业务处理 */
    }
}
```


## 4、应用示例

