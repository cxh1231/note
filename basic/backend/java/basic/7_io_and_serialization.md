## 1、IO 流的分类

流按照不同的特点，有很多种划分方式。主要有以下几种划分方式：

- 按照**流的流向**划分，可以划分为：**输入流**和**输出流**；
- 按照**操作单元**划分，可以划分为：**字节流**和**字符流**；
- 按照**流的角色**划分，可以划分为：**节点流**和**处理流**。

Java 中的流种类多达 40 多种，可以按如下划分：

![image-20220920160921671](https://img.zxdmy.com/2022/202209201609169.png)

![image-20220809203111696](https://img.zxdmy.com/2022/202208092031968.png)

![image-20220729145125664](https://img.zxdmy.com/2022/202207291451500.png)

## 2、IO 流体系的设计模式：装饰器模式

Java的 **IO流** 体系用到的了一个设计模式——**装饰器模式**。

比如，`InputStream` 相关的部分类图如下：

![image-20220920161207946](https://img.zxdmy.com/2022/202209201612171.png)

关于 **装饰器模式** 的详情，见 **设计模式** 篇章。

## 3、字节流与字符流的区别

> 不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，那为什么 I/O 流操作要分为字节流操作和字符流操作呢？

**字符流** 是由 Java 虚拟机将字节转换得到的，问题就出在**这个过程还算是非常耗时**，并且，如果我们不知道编码类型就很**容易出现乱码**问题。

所以， `I/O` 流就干脆提供了一个 **直接操作字符** 的接口，方便我们平时 **对字符进行流操作**。

字节流：适合音频文件、图片等媒体文件；

字符流：涉及到字符时，推荐。

## 4、三种 IO 模型：BIO、NIO、AIO

概述如下：

|        BIO         |           NIO            |                 AIO                  |
| :----------------: | :----------------------: | :----------------------------------: |
|    同步阻塞 IO     |      同步非阻塞 IO       |            异步非阻塞 IO             |
|  一个连接一个线程  |     一个请求一个线程     |         一个有效请求一个线程         |
| 发起请求，一直阻塞 | 多个连接多路复用一个线程 | IO请求立即返回，操作结束后，回调通知 |
| 一般通过连接池改善 |                          |                                      |

详述如下：

### 4.1 BIO（blocking I/O）

`BIO` 是传统的 IO，**同步并阻塞**，在服务器中实现的模式为 **一个连接一个线程**。

也就是说，客户端有连接请求的时候，服务器就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然这也可以通过线程池机制改善。

![image-20220920162302495](https://img.zxdmy.com/2022/202209201623671.png)

`BIO` **一般适用于连接数目小且固定的架构**，这种方式对于服务器资源要求比较高，而且并发局限于应用中，是`JDK1.4`之前的唯一选择，但好在程序直观简单，易理解。

### 4.2 NIO（non-blocking IO，New IO）

`NIO`，即**多路复用IO**，**同步并非阻塞**，在服务器中实现的模式为 **一个请求一个线程**。

也就是说，**客户端**发送的连接请求都会注册到**多路复用器**上，**多路复用器**轮询到有连接IO请求时才会启动一个线程进行处理。

![image-20220920162349982](https://img.zxdmy.com/2022/202209201623300.png)

`NIO` 的数据是面向**缓冲区** `Buffer` 的，必须从 `Buffer` 中读取或写入。

所以完整的`NIO`示意图：

![image-20220920163125577](https://img.zxdmy.com/2022/202209201631783.png)

可以看出，`NIO`的**运行机制**如下：

+ 每个`Channel`对应一个`Buffer`；
+ `Selector`对应一个线程，一个线程对应多个`Channel`；
+ `Selector`会根据不同的事件，在各个通道上切换；
+ `Buffer`是内存块，底层是数据。

`NIO` **一般适用于连接数目多且连接比较短（轻操作）的架构**，并发局限于应用中，编程比较复杂，从JDK1.4开始支持。

### 4.3 AIO（Asynchronous I/O）

`AIO`，即**异步IO**，**异步并非阻塞**，在服务器中实现的模式为**一个有效请求一个线程**。

也就是说，客户端的IO请求都是通过操作系统先完成之后，再通知服务器应用去启动线程进行处理。

在进行 I/O 编程中，常用到两种模式：`Reactor` 和 `Proactor`。

`Java` 的 `NIO` 就是 `Reactor`，当有事件触发时，服务器端得到通知，进行相应的处理，完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连接时间较长的应用。

`AIO` 一般适用于**连接数目多且连接比较长（重操作）的架构**，充分调用操作系统参与并发操作，编程比较复杂，从`JDK1.7`开始支持。

## 5、序列化与反序列化

### 5.1 简介

需要持久化 Java 对象，或在网络传输 Java 对象，就需要序列化。比如将 Java 对象保存在文件、Redis中。

即：**序列化的主要目的是通过网络传输对象或者说是将对象存储到文件系统、数据库、内存中。**

序列化与反序列化的基本过程：

- **序列化**： 将数据结构或对象 **转换成二进制字节流** 的过程
- **反序列化**：将在序列化过程中所生成的二进制字节流 **转换成数据结构或者对象** 的过程

> Java 序列化的都是对象（Object），也就是实例化后的类(Class)。

> 序列化的时候是不包含静态变量的。

### 5.2 优点

1. **对象序列化可以实现分布式对象。**
2. **java 对象序列化不仅保留一个对象的数据，而且递归保存对象引用的每个对象的数据。**即进行对象的`深复制`。
3. **序列化可以将内存中的类写入文件或数据库中。**
4. 对象、文件、数据，有许多不同的格式，很难统一传输和保存。**序列化后就变成字节流，可以进行通用的格式传输或保存**。

### 5.3 序列化方式

Java 序列化方式有很多，常见的有三种：

![image-20220920164641386](https://img.zxdmy.com/2022/202209201646352.png)

**Java对象流列化** ：Java **原生序列化** 方法即通过Java原生流（`InputStream`和`OutputStream`之间的转化）的方式进行转化，一般是**对象输出流**`ObjectOutputStream` 和**对象输入流**`ObjectInputStream` 。

**Json序列化**：这个可能是我们**最常用**的序列化方式，Json序列化的选择很多，一般会使用`jackson`包，通过`ObjectMapper`类来进行一些操作，比如**将对象转化为byte数组或者将json串转化为对象**。

**ProtoBuff序列化**：`ProtocolBuffer`是一种轻便高效的**结构化数据存储格式**，ProtoBuff序列化对象可以很大程度上将其压缩，可以大大减少数据传输大小，提高系统性能。

### 5.4 序列化与反序列化示例

> 通常将需要序列化的类实现 `Serializable` 接口或者 `Externalizable` 接口。

**两种序列化的对比**：

|                     实现Serializable接口                     |  实现Externalizable接口  |
| :----------------------------------------------------------: | :----------------------: |
|                    系统自动存储必要的信息                    |  程序员决定存储哪些信息  |
| Java内建支持，易于实现，只需要实现该接口即可，无需任何代码支持 | 必须实现接口内的两个方法 |
|                           性能略差                           |         性能略好         |

1、**可以通过实现 `java.io.Serializable` 接口以启用其序列化功能**

`Phone.jav`a：

```java
import java.io.Serializable;

public class Phone implements Serializable {

    private String name;


    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

`Main.java`：

```java
import java.io.*;

public class Main {
    public static void main(String[] args) {
        // 创建 Phone 对象
        Phone phone = new Phone();
        phone.setName("华为");

        // 序列化输出至文件
        try {
            FileOutputStream fos = new FileOutputStream("phone.txt");
            ObjectOutputStream oos = new ObjectOutputStream(fos);
            oos.writeObject(phone);
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 从文件中反序列化输入
        File file = new File("phone.txt");
        try {
            FileInputStream fis = new FileInputStream(file);
            ObjectInputStream ois = new ObjectInputStream(fis);
            Phone phone2 = (Phone) ois.readObject();
            System.out.println(phone2.getName()); // 输出：华为
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

2、**可以通过实现 `Externalizable` 接口实现序列化**

`Computer.java`：

```java
import java.io.Externalizable;
import java.io.IOException;
import java.io.ObjectInput;
import java.io.ObjectOutput;

public class Computer implements Externalizable {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(name);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        name = (String) in.readObject();
    }
}
```

`Main.java`：

```java
import java.io.*;

public class Main {
    public static void main(String[] args) {
        // 创建Computer对象
        Computer computer = new Computer();
        computer.setName("联想");

        // 序列化输出至文件
        try {
            FileOutputStream fos = new FileOutputStream("computer.txt");
            ObjectOutputStream oos = new ObjectOutputStream(fos);
            oos.writeObject(computer);
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 从文件中反序列化输入
        File file = new File("computer.txt");
        try {
            FileInputStream fis = new FileInputStream(file);
            ObjectInputStream ois = new ObjectInputStream(fis);
            Computer computer2 = (Computer) ois.readObject();
            System.out.println(computer2.getName()); // 输出：联想
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

### 5.5 serialVersionUID

Java 的序列化机制是通过 **在运行时判断类的`serialVersionUID`来验证版本一致性** 的。

在进行反序列化时，JVM会把传来的字节流中的 `serialVersionUID` 与本地相应实体（类）的 `serialVersionUID` 进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常。

**显示指定 serialVersionUID 值的原因**：

1. 若不指定，则 `JVM` 在序列化时会根据属性自动生成一个`serialVersionUID`。当反序列化时，JVM 会再根据属性自动生成一个**新版** `serialVersionUID`。这个`新版serialVersionUID`与序列化时生成的`旧版serialVersionUID`进行比较。
   1. 如果序列化的这个类不做任何修改，那新旧`serialVersionUID`一致，没有问题；
   2. 但是，如果这个类进行了迭代更新，新旧`serialVersionUID`将不致，旧对象反序列化就会报错。
2. 如果显示指定，JVM在序列化和反序列化时仍然都会生成一个serialVersionUID，但**值为我们显示指定的值**，这样在反序列化时新旧版本的serialVersionUID就一致了。

![image-20220809202546014](https://img.zxdmy.com/2022/202208092025532.png)

## 6、transient  关键字（阻止序列化）

对于不想进行序列化的变量，使用 `transient` 关键字修饰。

`transient` 关键字的作用是：阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被 `transient` 修饰的变量值不会被持久化和恢复。

关于 `transient` 还有几点注意：

- `transient` 只能修饰变量，不能修饰类和方法。
- `transient` 修饰的变量，在反序列化后变量值将会被置成类型的默认值。例如，如果是修饰 `int` 类型，那么反序列后结果就是 `0`。
- `static` 变量因为不属于任何对象(Object)，所以无论有没有 `transient` 关键字修饰，均不会被序列化，即 **静态变量不会被序列化**。

