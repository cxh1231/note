## 1、序列化与反序列化

#### 简介

需要持久化 Java 对象，或在网络传输 Java 对象，就需要序列化。比如将 Java 对象保存在文件、Redis中。

即：**序列化的主要目的是通过网络传输对象或者说是将对象存储到文件系统、数据库、内存中。**

序列化与反序列化的基本过程：

- **序列化**： 将数据结构或对象 `转换成二进制字节流` 的过程
- **反序列化**：将在序列化过程中所生成的二进制字节流 `转换成数据结构或者对象` 的过程

> Java 序列化的都是对象（Object），也就是实例化后的类(Class)。

#### 优点

1. **对象序列化可以实现分布式对象。**
2. **java 对象序列化不仅保留一个对象的数据，而且递归保存对象引用的每个对象的数据。**即进行对象的`深复制`。
3. **序列化可以将内存中的类写入文件或数据库中。**
4. 对象、文件、数据，有许多不同的格式，很难统一传输和保存。**序列化后就变成字节流，可以进行通用的格式传输或保存**。

#### 序列化实现方式

> 实现 `Serializable` 接口或者 `Externalizable` 接口。

1、**可以通过实现 `java.io.Serializable` 接口以启用其序列化功能**

Phone.java：

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

Main.java：

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

Computer.java：

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

Main.java：

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

**两种序列化的对比**

|                     实现Serializable接口                     |  实现Externalizable接口  |
| :----------------------------------------------------------: | :----------------------: |
|                    系统自动存储必要的信息                    |  程序员决定存储哪些信息  |
| Java内建支持，易于实现，只需要实现该接口即可，无需任何代码支持 | 必须实现接口内的两个方法 |
|                           性能略差                           |         性能略好         |

#### serialVersionUID 

Java 的序列化机制是通过 **在运行时判断类的`serialVersionUID`来验证版本一致性** 的。

在进行反序列化时，JVM会把传来的字节流中的 `serialVersionUID` 与本地相应实体（类）的 `serialVersionUID` 进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常。

**显示指定 serialVersionUID 值的原因**：

1. 若不指定，则 `JVM` 在序列化时会根据属性自动生成一个`serialVersionUID`。当反序列化时，JVM 会再根据属性自动生成一个**新版** `serialVersionUID`。这个`新版serialVersionUID`与序列化时生成的`旧版serialVersionUID`进行比较。
   1. 如果序列化的这个类不做任何修改，那新旧`serialVersionUID`一致，没有问题；
   2. 但是，如果这个类进行了迭代更新，新旧`serialVersionUID`将不致，旧对象反序列化就会报错。
2. 如果显示指定，JVM在序列化和反序列化时仍然都会生成一个serialVersionUID，但**值为我们显示指定的值**，这样在反序列化时新旧版本的serialVersionUID就一致了。

![image-20220809202546014](https://img.zxdmy.com/2022/202208092025532.png)

## 2、transient  关键字（阻止序列化）

对于不想进行序列化的变量，使用 `transient` 关键字修饰。

`transient` 关键字的作用是：阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被 `transient` 修饰的变量值不会被持久化和恢复。

关于 `transient` 还有几点注意：

- `transient` 只能修饰变量，不能修饰类和方法。
- `transient` 修饰的变量，在反序列化后变量值将会被置成类型的默认值。例如，如果是修饰 `int` 类型，那么反序列后结果就是 `0`。
- `static` 变量因为不属于任何对象(Object)，所以无论有没有 `transient` 关键字修饰，均不会被序列化，即 **静态变量不会被序列化**。

## 3、Java 中 IO 流的分类

- 按照流的流向分，可以分为输入流和输出流；
- 按照操作单元划分，可以划分为字节流和字符流；
- 按照流的角色划分为节点流和处理流。

![image-20220809203111696](https://img.zxdmy.com/2022/202208092031968.png)

![image-20220729145125664](https://img.zxdmy.com/2022/202207291451500.png)

## 4、既然有了字节流，为什么还要有字符流？

> 不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，那为什么 I/O 流操作要分为字节流操作和字符流操作呢？

字符流是由 Java 虚拟机将字节转换得到的，问题就出在这个过程还算是非常耗时，并且，如果我们不知道编码类型就很容易出现乱码问题。所以， I/O 流就干脆提供了一个直接操作字符的接口，方便我们平时对字符进行流操作。如果音频文件、图片等媒体文件用字节流比较好，如果涉及到字符的话使用字符流比较好。

## 5、三种常见 IO 模型：BIO、NIO、AIO

#### BIO

**同步阻塞IO**：**同步并阻塞**，在服务器中实现的模式为**一个连接一个线程**。也就是说，客户端有连接请求的时候，服务器就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然这也可以通过线程池机制改善。BIO**一般适用于连接数目小且固定的架构**，这种方式对于服务器资源要求比较高，而且并发局限于应用中，是JDK1.4之前的唯一选择，但好在程序直观简单，易理解。

#### NIO

 **多路复用IO**：**同步并非阻塞**，在服务器中实现的模式为**一个请求一个线程**，也就是说，客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到有连接IO请求时才会启动一个线程进行处理。**NIO一般适用于连接数目多且连接比较短（轻操作）的架构**，并发局限于应用中，编程比较复杂，从JDK1.4开始支持。

#### AIO

**异步IO**：异步并非阻塞，在服务器中实现的模式为**一个有效请求一个线程**，也就是说，客户端的IO请求都是通过操作系统先完成之后，再通知服务器应用去启动线程进行处理。AIO一般适用于连接数目多且连接比较长（重操作）的架构，充分调用操作系统参与并发操作，编程比较复杂，从JDK1.7开始支持。