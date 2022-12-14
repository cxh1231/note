## 1、泛型概述

**Java 泛型（`Generics`）** 是 `JDK 5` 中引入的一个新特性。

使用泛型参数，可以增强代码的可读性以及稳定性。

**泛型就是将类型参数化，其在编译时才确定具体的参数。**

**泛型只存在于编译阶段，而不存在于运行阶段。**

使用泛型，具有如下好处：

**类型安全**：

- 泛型的主要目标是提高 Java 程序的类型安全
- 编译时期就可以检查出因 Java 类型不正确导致的 ClassCastException 异常
- 符合越早出错代价越小原则

**消除强制类型转换**：

- 泛型的一个附带好处是，使用时直接得到目标类型，消除许多强制类型转换
- 所得即所需，这使得代码更加可读，并且减少了出错机会

**潜在的性能收益**：

- 由于泛型的实现方式，支持泛型（几乎）不需要 JVM 或类文件更改
- 所有工作都在编译器中完成
- 编译器生成的代码跟不使用泛型（和强制类型转换）时所写的代码几乎一致，只是更能确保类型安全而已

## 2、泛型的使用

泛型一般有三种使用方式：

+ 泛型类
+ 泛型接口
+ 泛型方法

![image-20220920165330509](https://img.zxdmy.com/2022/202209201653765.png)

### 2.1 泛型类

泛型类在实例化时，必须指定 `T` 的具体类型。

定义与使用：

```java
// 定义泛型类
// 此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
public class Generic<T>{

    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey(){
        return key;
    }
}
// 使用
// 在实例化泛型类时，必须指定 T 的具体类型
Generic<Integer> genericInteger = new Generic<Integer>(123456);
```

### 2.2 泛型接口

定义与使用：

```java
// 定义接口
public interface Generator<T> {
    public T method();
}
// 实现泛型接口，不指定类型：
class GeneratorImpl<T> implements Generator<T>{
    @Override
    public T method() {
        return null;
    }
}
// 实现泛型接口，指定类型：
class GeneratorImpl<T> implements Generator<String>{
    @Override
    public String method() {
        return "hello";
    }
}
```

### 2.3 泛型方法

定义与使用：

```java
// 定义
public static <E> void printArray(E[] inputArray) {
	for (E element : inputArray) {
		System.out.printf("%s ", element);
	}
	System.out.println();
}
// 使用
Integer[] intArray = { 1, 2, 3 };
String[] stringArray = { "Hello", "World" };
printArray(intArray);
printArray(stringArray);
```

## 3、泛型通配符

常用的通配符为： T，E，K，V，？

+ `?` ：表示**不确定**的 java 类型
+ `T` (type) ：表示**具体**的 java 类型
+ `K`、 `V` (key value) ：分别代表 java 键值中的 Key Value
+ `E` (element) ：代表 `Element`

#### 限定通配符

**限定通配符** 对类型进行了限制。

有两种限定通配符：

1. `<? extends T>`，其**类型必须是 `T` 的子类来设定类型的上界**，即可以接受任何继承自 T 的类型；
2. `<? super T>`，其**类型必须是 `T` 的父类来设定类型的下界**，即可以接受任何 T 的父类的类型。

泛型类型必须用限定内的类型来进行初始化，否则会导致编译错误。

#### 非限定通配符

**非限定通配符** `?` ，可以用任意类型来替代。

如`List<?>` 的意思是这个集合是一个可以持有任意类型的集合，它可以是`List<A>`，也可以是`List<B>`,或者`List<C>`。

> 不可以把 `List<String>` 传递给一个接受 `List<Object>` 参数的方法。

> 不可以在 `Array` 中使用泛型，而经过使用 `List` 代替。

## 4、类型擦除（泛型擦除）

Java 的泛型是 **伪泛型**，这是因为 Java 在**编译**期间，所有的泛型信息都会被擦掉，即在运行的时候是没有泛型的。

这也就是通常所说 **类型擦除** 。

使用泛型的时候加上的类型参数，编译器在编译的时候去掉类型参数。

例如这段代码，往一群猫里放条狗：

```java
LinkedList<Cat> cats = new LinkedList<Cat>();
LinkedList list = cats; // 注意我在这里把范型去掉了，但是 list 和 cats 是同一个链表！
list.add(new Dog()); 	// 完全没问题！
```

因为Java的范型只存在于源码里，编译的时候给你静态地检查一下范型类型是否正确，而到了运行时就不检查了。

上面这段代码在JRE（Java运行环境）看来和下面这段没区别

```java
LinkedList cats = new LinkedList(); // 注意：没有范型！
LinkedList list = cats;
list.add(new Dog());
```

为什么要 **类型擦除** 呢？

主要是为了 **向下兼容**，因为 `JDK5` 之前是没有泛型的，为了让`JVM`保持向下兼容，就出了类型擦除这个策略。