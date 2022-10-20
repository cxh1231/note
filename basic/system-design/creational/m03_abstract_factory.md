## 1、模式定义

**抽象工厂模式** 指的是：为创建一组相关或相互依赖的对象提供一个接口，而且无须指定它们的具体类。

抽象工厂模式是工厂方法模式的升级版本。在有多个业务品种、业务分类时，通过抽象工厂模式产生需要的对象是一种非常好的解决方式。

## 2、类图

![img](https://img.zxdmy.com/2022/202209191926038.jpeg)

抽象工厂模式中的 4 个角色如下：

+ **抽象工厂**（Abstract Factory）角色：该角色是抽象工厂模式的核心，与应用系统无关，任何创建对象的工厂类必须实现这个接口。
+ **具体工厂**（Concrete Factory）角色：该角色实现了抽象工厂接口，含有选择合适的产品对象的逻辑，并且受到应用程序的调用以创建产品对象。
+ **抽象产品**（Abstract Product）角色：该角色负责定义产品的共性，实现对产品最抽象的定义。
+ **具体产品**（Concrete Product）角色：该角色实现抽象产品角色所声明的接口，抽象工厂模式所创建的任何产品对象都是某个具体产品角色的实例。

## 3、代码实现

> 基于Java语言

**抽象工厂** `AbstractFactory.java`：

```java
public interface AbstractFactory {
    // 创建产品A
    public ProductA factoryA();
    // 创建产品B
    public ProductB factoryB();
}
```

**具体工厂**`ConcreteFactory1.java`

```java
public class ConcreteFactory1 implements AbstractFactory {
    // 创建等级为1的产品A
    public ProductA factoryA() {
        return new ProductA1();
    }
    // 创建等级为1产品B
    public ProductB factoryB() {
        return new ProductB1();
    }
}
```

`ConcreteFactory2.java`：

```java
public class ConcreteFactory2 implements AbstractFactory {
    // 创建等级为2的产品A
    public ProductA factoryA() {
        return new ProductA2();
    }
    // 创建等级为2产品B
    public ProductB factoryB() {
        return new ProductB2();
    }
}
```

`ProductA` 的 **抽象产品接口**：

`ProductA.java`

```java
public interface ProductA {
    //产品A的公共方法
    public void method1();
    public void method2();
}
```

`ProductA` 的两个**具体产品** `ProductA1`和`ProductA2`：

`ProductA1.java`

```java
public class ProductA1 implements ProductA {
    public void method1() {
        System.out.println("等级为1的产品A的实现方法");
    }
    public void method2() {
        // 业务逻辑处理代码
    }
}
```

`ProductA2.java`

```java
public class ProductA2 implements ProductA {
    public void method1() {
        System.out.println("等级为2的产品A的实现方法");
    }
    public void method2() {
        // 业务逻辑处理代码
    }
}
```

**应用代码** `Demo.java`：

```java
public class ClientDemo {
    public static void main(String args[]) {
        // 定义两个工厂
        AbstractFactory factory1 = new ConcreteFactory1();
        AbstractFactory factory2 = new ConcreteFactory2();
        // 生产等级为1的产品A
        ProductA a1 = new ProductA1();
        // 生产等级为2的产品A
        ProductA a2 = new ProductA2();
        // 生产等级为1的产品B
        ProductB b1 = new ProductB1();
        // 生产等级为2的产品B
        ProductB b2 = new ProductB2();
        // 业务处理
        a1.method1();
        a2.method1();
        b1.method1();
        b2.method1();
    }
}
```

## 4、应用示例









