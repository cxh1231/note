## 1、数据类型与基本运算

`Java` 语言的数据类型分为两种：`基本数据类型`和`引用数据类型`。

![img](https://img.zxdmy.com/2022/202208091535838.png)

### 1.1 八种基本数据类型

- 4 种整数型：`byte`、`short`、`int`、`long`
- 2 种浮点型：`float`、`double`
- 1 种字符类型：`char`
- 1 种布尔型：`boolean`

| 基本类型  | 位数 | 字节 | 默认值  | 取值范围                                   |
| :-------: | :--: | :--: | :-----: | ------------------------------------------ |
|  `byte`   |  8   |  1   |    0    | -128 ~ 127                                 |
|  `short`  |  16  |  2   |    0    | -32768 ~ 32767                             |
|   `int`   |  32  |  4   |    0    | -2147483648 ~ 2147483647                   |
|  `long`   |  64  |  8   |   0L    | -9223372036854775808 ~ 9223372036854775807 |
|  `char`   |  16  |  2   | 'u0000' | 0 ~ 65535                                  |
|  `float`  |  32  |  4   |   0f    | 1.4E-45 ~ 3.4028235E38                     |
| `double`  |  64  |  8   |   0d    | 4.9E-324 ~ 1.7976931348623157E308          |
| `boolean` |  1   |      |  false  | true、false                                |

>  `boolean`的 字节数，官方文档未明确定义。
>
>  `long` 类型的数据要在数值后面加 **L** 。

这**八种基本类型**都有对应的包装类分别为：`Byte`、`Short`、`Integer`、`Long`、`Float`、`Double`、`Character`、`Boolean` 。

> Integer 的大小：16 字节（含 int 4 字节，其他是指针等信息）。

### 1.2 基本类型和包装类型的区别

- 包装类型不赋值就是 `null` ，而基本类型有默认值且不是 `null`。
- 包装类型可用于泛型，而基本类型不可以。
- 基本数据类型的**局部变量存放在 Java 虚拟机栈中的局部变量表中**，基本数据类型的**成员变量（未被 `static` 修饰 ）存放在 Java 虚拟机的堆中**。包装类型属于对象类型，我们知道几乎所有对象实例都存在于堆中。
- 相比于对象类型， 基本数据类型占用的空间非常小。

### 1.3 包装类型的缓存机制（常量池）与比较

Java 基本类型的包装类的大部分都实现了常量池技术。

`Byte`、`Short`、`Integer`、`Long` 这 4 种包装类默认创建了数值 `[-128，127]` 的相应类型的缓存数据，`Character` 创建了数值在 `[0,127]`范围的缓存数据，`Boolean` 直接返回 `True` or `False`。

如果超出对应范围仍然会去创建新的对象，缓存的范围区间的大小只是在性能和资源之间的权衡。

所以 `==` 比较大小，有问题，强制使用 `equals` 。

```java
Integer a = 33;
Integer b1 = 33;
Integer b2 = new Integer(33);
System.out.println(a == b1); // true
System.out.println(a == b2); // false
Integer c = 1024;
Integer d = 1024;
System.out.println(c == d); // false
System.out.println(c.equals(d)); // true

Float i1 = 333f;
Float i2 = 333f;
System.out.println(i1 == i2); // false

Double i3 = 1.2;
Double i4 = 1.2;
System.out.println(i3 == i4); // false
```

下图是阿里巴巴Java开发手册给出的说明。

![image-20220728142703256](https://img.zxdmy.com/2022/202208101056842.png)

### 1.4 自动装箱与自动拆箱

- **装箱**：将`基本类型`用它们对应的`引用类型`包装起来；调用了包装类的`valueOf()`方法，
- **拆箱**：将`包装类型`转换为`基本数据类型`；调用了 `xxxValue()`方法。

```java
Integer i = 10; //装箱
int n = i;   	//拆箱
```

即：

- `Integer i = 10` 等价于 `Integer i = Integer.valueOf(10)`
- `int n = i` 等价于 `int n = i.intValue()`;

> 频繁拆装箱会严重影响系统的性能，应该尽量避免不必要的拆装箱操作。

### 1.5 自动类型转换与强制类型转换

`Java` 所有的数值型变量可以相互转换，当把一个表数范围小的数值或变量直接赋给另一个表数范围大的变量时，可以进行 **自动类型转换**；反之，需要 **强制转换**。

![image-20220920144058090](https://img.zxdmy.com/2022/202209201440118.png)

比如，Java 中 `3.14` 默认是**双精度**（`double`）的，当写 `float f = 3.14` 时，会造成精度损失，需要强制类型转换：`float f = 3.14f`。

### 1.5 浮点运算精度问题

由于计算机保存浮点数的机制问题，其使用二进制保存数字，宽度有限，**无限循环小数只能被截断**，所以导致小数精度丢失。

示例：

```java
float a = 2.0f - 1.9f;
float b = 1.8f - 1.7f;
System.out.println(a);		// 0.100000024
System.out.println(b);		// 0.099999905
System.out.println(a == b);	// false
```

**解决精度丢失问题，可以使用`BigDecimal` 实现对浮点数的运算，不会造成精度丢失**。

示例：

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");

BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);

System.out.println(x); /* 0.1 */
System.out.println(y); /* 0.1 */
System.out.println(Objects.equals(x, y)); /* true */
```

> `BigDecimal` 传入的值，建议使用 String 类型。

### 1.6 大数表示

超过 `Long.MAX_VALUE` 的数，可以使用 `BigInteger` 来表示运算。但运算效率相对较低。

> `BigInteger` 内部使用 `int[]` 数组来存储任意大小的整形数据。

## 2、基本语法

### 2.1 标识符与关键字

**标识符就是一个名字** ，而**关键字是被赋予特殊含义的标识符**。

> 所有的关键字都是小写的。

### 2.2 自增自减运算符

**前缀：先自增/减，再赋值**

当 `b = ++a` 时，先自增（自己增加 1），再赋值（赋值给 b）

**后缀：先赋值，再自增/减**

当 `b = a++` 时，先赋值(赋值给 b)，再自增（自己增加 1）

### 2.3 成员变量与局部变量

|        | 成员变量                                                     | 局部变量                                                    |
| :----: | ------------------------------------------------------------ | ----------------------------------------------------------- |
|  语法  | 属于类的，可以被 `public`,`private`,`static` 等修饰符所修饰，能被 `final` 所修饰 | 代码块或方法中定义的变量或方法的参数，只能被 `final` 所修饰 |
|  存储  | `static` 修饰，属于类，**堆**内存；否则属于实例，**栈**内存  | **栈**内存的局部变量表中                                    |
|  生存  | 与对象生存周期一致                                           | 方法调用结束则消亡                                          |
| 默认值 | 以类型的默认值而赋值（ `final` 修饰的成员变量必须显式地赋值） | 不会自动赋值                                                |

### 2.4 Static 关键字

“`static`”关键字表明一个**成员变量** 或 **成员方法**可以在没有所属的类的实例变量的情况下被访问，即**在没有创建对象的情况下也可以去调用方法**。

> Java中 `static` 方法不能被覆盖。
>
> 因为**方法覆盖是基于运行时动态绑定**的，而static方法是编译时静态绑定的。

### 2.5 静态变量

`静态变量` 可以被类的所有实例共享。无论一个类创建了多少个对象，它们都共享同一份静态变量。

通常情况下，静态变量会被 `final` 关键字修饰成为常量。

### 2.6 静态方法与非静态成员

+ `静态方法`是属于类的，在类加载的时候就会分配内存，可以通过类名直接访问。

+ `非静态成员`属于实例对象，只有在对象实例化之后才存在，需要通过类的实例对象去访问。

> **问：静态方法为什么不能调用非静态成员?**
>
> 在类的`非静态成员`不存在的时候`静态成员`就已经存在了，此时调用在内存中还不存在的非静态成员，属于非法操作。
>
> **静态方法可以调用静态资源。**

### 2.7 静态方法与实例方法

**静态方法：**

+ 在外部被调用时，可以使用 `类名.方法名` 和 `对象.方法名` 两种方式（但推荐用前者）；
+ 在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），不允许访问实例成员。

**实例方法：**

+ 被调用时，需要先实例化（创建对象），使用 `对象.方法名` 的方式来调用；
+ 在访问本类的成员时，没有任何限制。

### 2.9 & 与&& 及 | 与 ||

+ `&` 表示 **逻辑与** 运算， `1 & 1 = 1` ； `1 & 0 = 0` ；`7 & 3 = 3`（即二进制 `111 & 11 = 11`）
+ `|` 表示 **逻辑或** 运算，`7 | 3 = 7` ，`8 | 7 = 15`
+ `&&` 表示 **短路与** 运算
+ `||` 表示 **短路或** 运算

### 2.10 位运算（左移、右移）

+ `a << b` ：表示 **左移** 运算，即将 `a` 的 **二进制** 向 **左** 移动 `b` 位，末尾补 `0`，比如 `8 << 2 = 32`；
+ `a >> b` ：表示 **右移** 运算，即将 `a` 的 **二进制** 向 **右** 移动 `b` 位，符号位不动，其他位补 `1`，比如 `-8 >> 2 = -2`；
+ `a >>> b` ：表示 **无符号右移** 运算，即将 `a` 的 **二进制** 向 **右** 移动 `b` 位（含符号位），并补 `1`，比如 `-32 >>> 2 = 1073741816`，即对应二进制 `111111111111111111111111111000`

### 2.11 switch 语法

`Java 5` 以前 `switch(expr)` 中，expr 只能是 `byte`、`short`、`int`、`char`。

从 `Java 5` 开始，Java 中引入了枚举类型， expr 也可以是 `enum` 类型。

从 `Java 7` 开始，expr 还可以是字符串(`String`)。

**但是长整型(`long`)在目前所有的版本中都是不可以的。**



### 2.13 代码块的执行顺序

Java 中的代码块包含以下三种类型。

1. **静态代码块**（静态快、静态初始化块）：在类加载 `JVM` 时运行；
2. **构造代码块**（构造初始化块）：创建对象时被调用，在类中定义；
3. **普通代码块**（代码块、初始化块）：在方法体中定义。

代码块执行顺序： **静态代码块 —> 构造代码块 —> 构造函数—> 普通代码块**，示例如下：

User.java：

```java
public class User {

    public String name;

    // 构造代码块
    {
        name = "张三";
        System.out.println("User：构造代码块");
    }

    // 无参构造代码块
    public User() {
        System.out.println("User：无参构造代码块");
    }

    // 有参构造代码块
    public User(String name) {
        this.name = name;
        System.out.println("User：有参构造代码块");
    }

    // 静态代码块
    static {
        System.out.println("User：静态代码块");
    }

    // 普通代码块
    public String getName() {
        return name;
    }
}
```

执行结果：

```java
public class Main {
    public static void main(String[] args) {
        User user = new User("用户李四");
        System.out.println(user.getName());

        System.out.println("----------");

        User user2 = new User();
        System.out.println(user2.getName());
    }
}
/**
User：静态代码块
User：构造代码块
User：有参构造代码块
用户李四
----------
User：构造代码块
User：无参构造代码块
张三
*/
```

继承中代码块执行顺序：**父类静态块 —> 子类静态块 —> 父类代码块 —> 父类构造器 —> 子类代码块 —> 子类构造器**，示例如下：

父类是上面的User，子类为Student.java：

```java
public class Student extends User {

    // 构造代码块
    {
        name = "李四";
        System.out.println("Student：构造代码块");
    }

    // 无参构造代码块
    public Student() {
        System.out.println("Student：无参构造代码块");
    }

    // 有参构造代码块
    public Student(String name) {
        System.out.println("Student：有参构造代码块");
    }


    // 静态代码块
    static {
        System.out.println("Student：静态代码块");
    }
}
```

执行结果：

```java
public class Main {
    public static void main(String[] args) {
        Student student = new Student("学生王五");
        System.out.println(student.getName());

        System.out.println("----------");

        Student student2 = new Student();
        System.out.println(student2.getName());
    }
}
/**
User：静态代码块
Student：静态代码块
User：构造代码块
User：无参构造代码块
Student：构造代码块
Student：有参构造代码块
学生王五
----------
User：构造代码块
User：无参构造代码块
Student：构造代码块
Student：无参构造代码块
李四
*/
```

## 3、Java 常用类：Object 类

`Object` 类是一个特殊的类，**是所有类的父类**。它主要提供了以下 `11` 个方法：

![image-20220920150919760](https://img.zxdmy.com/2022/202209201509195.png)

其方法的详细定义与描述如下：

```java
// native 方法，用于返回当前运行时对象的 Class 对象，使用了 final 关键字修饰，故不允许子类重写。
public final native Class<?> getClass();
    
// native 方法，用于返回对象的哈希码，主要使用在哈希表中，比如 JDK 中的HashMap。
public native int hashCode();

// 用于比较 2 个对象的内存地址是否相等，String 类对该方法进行了重写以用于比较字符串的值是否相等。
public boolean equals(Object obj);

// naitive 方法，用于创建并返回当前对象的一份拷贝。
protected native Object clone() throws CloneNotSupportedException;

//返回类的名字实例的哈希码的 16 进制的字符串。建议 Object 所有的子类都重写这个方法。
public String toString();

// native 方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。
public final native void notify();

// native 方法，并且不能重写。跟 notify 一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。
public final native void notifyAll();

// native方法，并且不能重写。暂停线程的执行。注意：sleep 方法没有释放锁，而 wait 方法释放了锁 ，timeout 是等待时间。
public final native void wait(long timeout) throws InterruptedException;

// 多了 nanos 参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上 nanos 毫秒。。
public final void wait(long timeout, int nanos) throws InterruptedException;

// 跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念
public final void wait() throws InterruptedException;

// 实例被垃圾回收器回收的时候触发的操作
protected void finalize() throws Throwable { };
```

### 3.1 hashcode()

`hashCode()` 的作用是获取**哈希码**，也称为散列码；它实际上是返回一个`int`整数。

这个**哈希码**的作用是确定该对象在**哈希表**中的索引位置，比如 `JDK` 中的 `HashMap`。

### 3.2 equals() 和 ==

**`==`** 对于基本类型和引用类型的作用效果是不同的：

- 对于**基本数据**类型来说，`==` 比较的是值。
- 对于**引用数据**类型来说，`==` 比较的是对象的内存地址。

> 因为 `Java` 只有 `值传递`，所以，对于 `==` 来说，不管是比较基本数据类型，还是引用数据类型的变量，其本质比较的都是值，只是 `引用类型变量存的值是对象的地址`。

`equals()` 不能用于判断**基本数据类型**的变量，只能用来判断两个对象是否相等。

`equals()` 方法存在两种使用情况：

- **类没有重写 `equals()`方法** ：通过`equals()`比较该类的两个对象时，等价于通过“==”比较这两个对象，使用的默认是 `Object`类`equals()`方法。
- **类重写了 `equals()`方法** ：一般都重写 `equals()`方法来比较两个对象中的**属性**是否相等；若它们的**属性相等**，则返回 true(即，认为这两个对象相等)。

比如，`String` 类就对 `equals()` 方法进行了 **重写**，以判断字符串的对象的值是否相同。

### 3.3 hashCode()与equals()的关系

- 如果两个对象相等，则`hashcode`一定也是相同的；
- 如果两个对象相等，对两个对象分别调用`equals`方法都返回`true`；
- 如果两个对象有相同的`hashcode`值，它们也不一定是相等的。

### 3.4 重写 equals 方法必须重写 hashcode 方法

判断的时候先根据`hashcode`进行的判断，相同的情况下再根据`equals()`方法进行判断。如果只重写了`equals`方法，而不重写`hashcode`的方法，会造成`hashcode`的值不同，而`equals()`方法判断出来的结果为true。

在`Java`中的一些容器中，不允许有两个完全相同的对象，插入的时候，如果判断相同则会进行覆盖。

这时候如果只重写了`equals` 的方法，而不重写`hashcode`的方法，`Object`中`hashcode`是根据对象的存储地址转换而形成的一个哈希值。

这时候就有可能因为没有重写`hashcode`方法，造成相同的对象散列到不同的位置而造成对象的不能覆盖的问题。

## 4、Java常用类：String 类

### 4.1 String 不可变

**为什么不可变**：

+ `String` 类中使用 `final` 关键字修饰字符数组来保存字符串，导致其**不能被继承**，进而避免了子类破坏 `String` 不可变。所以**线程安全**。
+ 保存字符串的数组被 `final` 修饰且为**私有**的，并且`String` 类没有提供/暴露修改这个字符串的方法。

**为什么设计成不可变**：

1. 便于实现字符串池（String pool）
2. 使多线程安全
3. 避免安全问题
4. 加快字符串处理速度

### 4.2 字符串常量池

**字符串常量池** 是 `JVM` 为了提升性能和减少内存消耗针对字符串（`String` 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

示例如下：

```java
// 在堆中创建字符串对象”ab“
// 将字符串对象”ab“的引用保存在字符串常量池中
String a = "ab";
// 直接返回字符串常量池中字符串对象”ab“的引用
String b = "ab";
System.out.println(a == b);// true
```

### 4.3 字符串对象创建

`String s = new String("abc")` 会创建几个字符串对象？会创建 `1` 或 `2` 个字符串对象。

1. 如果字符串常量池中**不存在**字符串对象“`abc`”的引用，那么会在堆中创建 2 个字符串对象“`abc`”。

```java
String s = new String("abc");
```

1. 如果字符串常量池中**已存在**字符串对象“`abc`”的引用，则只会在堆中创建 1 个字符串对象“`abc`”。

```java
// 字符串常量池中已存在字符串对象“abc”的引用
String s1 = "abc";
// 下面这段代码只会在堆中创建 1 个字符串对象“abc”
String s = new String("abc");
```

### 4.4 String.intern() 方法

`String.intern()` 是一个 `native`（本地）方法，其作用是**将指定的字符串对象的引用保存在字符串常量池中**，可以简单分为两种情况：

- 如果字符串常量池中**保存了对应的字符串对象的引用，就直接返回该引用**。
- 如果字符串常量池中没有保存了对应的字符串对象的引用，那就在常量池中**创建一个指向该字符串对象的引用并返回**。

示例：

```java
// 在堆中创建字符串对象”Java“
// 将字符串对象”Java“的引用保存在字符串常量池中
String s1 = "Java";
// 直接返回字符串常量池中字符串对象”Java“对应的引用
String s2 = s1.intern();
// 会在堆中在单独创建一个字符串对象
String s3 = new String("Java");
// 直接返回字符串常量池中字符串对象”Java“对应的引用
String s4 = s3.intern();
// s1 和 s2 指向的是堆中的同一个对象
System.out.println(s1 == s2); // true
// s3 和 s4 指向的是堆中不同的对象
System.out.println(s3 == s4); // false
// s1 和 s4 指向的是堆中的同一个对象
System.out.println(s1 == s4); //true
```

### 4.5 常量折叠

对于编译器可以确定值的字符串，也就是**常量字符串** ，jvm 会将其存入字符串常量池。并且，**字符串常量拼接得到的字符串常量**在编译阶段就已经被存放字符串常量池，这个得益于编译器的优化。

比如，对于 `String str3 = "str" + "ing";` 编译器会给你优化成 `String str3 = "string";` 。

> 并不是所有的常量都会进行折叠，**只有编译器在程序编译期就可以确定值的常量才可以**：
>
> - 基本数据类型( `byte`、`boolean`、`short`、`char`、`int`、`float`、`long`、`double`)以及字符串常量。
> - `final` 修饰的基本数据类型和字符串变量
> - 字符串通过 “+”拼接得到的字符串、基本数据类型之间算数运算（加减乘除）、基本数据类型的位运算（<<、>>、>>> ）

### 4.6 String 的 + 运算

Java 语言本身并不支持运算符重载，`+` 和 `+=` 是专门为 `String` 类重载过的运算符，也是 Java 中仅有的两个重载过的运算符。

对于示例代码：

```java
String str1 = "he";
String str2 = "llo";
String str3 = "world";
String str4 = str1 + str2 + str3;
```

**字符串对象** 通过 `+` 的形式拼接字符串，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。

即：

```java
String str4 = new StringBuilder().append(str1).append(str2).toString();
```

> 注意：在循环内使用 `+` 或 `+=` 进行字符串拼接，编译器会创建多个 `StringBuilder` 对象（即在循环内创建）。

### 4.7 String.equals() 方法与字符串比较

`String` 中的 `equals` 方法是被重写过的，比较的是 String 字符串的值是否相等。

`Object` 的 `equals` 方法是比较的对象的内存地址。

对于字符串的各种创建与比较，直接上代码，详见如下示例。

**示例：常量与常量比较**

```java
String a = "ab"; // 放在常量池中
String b = "ab"; // 从常量池中查找
System.out.println(a == b); // true
System.out.println(a.equals(b)); // true
```

**示例：引用与引用比较**

```java
String a = new String("ab"); // a 为一个引用
String b = new String("ab"); // b为另一个引用,对象的内容一样
System.out.println(a == b); // false，比较的是内存地址
System.out.println(a.equals(b)); // true，比较的是对象的值
```

**示例：常量与引用的比较**

```java
String a = "ab"; // 放在常量池中
String b = new String("ab");
System.out.println(a == b); // false
System.out.println(a.equals(b)); // true
```

**示例：+ 运算后比较**

```java
String str1 = "str";
String str2 = "ing";
String str3 = "str" + "ing";	// 常量池
String str4 = str1 + str2;		// 创建了新的对象
System.out.println(str3 == str4); //false

String str5 = "string";
System.out.println(str3 == str5); //true
System.out.println(str4 == str5); //false
```

**示例：final 修饰后比较**

```java
final String str1 = "str";
final String str2 = "ing";
// 下面两个表达式其实是等价的
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2;  // 常量池中的对象
System.out.println(c == d); // true
```

> 被 `final` 关键字修改之后的 `String` 会被编译器当做常量来处理，编译器在程序编译期就可以确定它的值，其效果就相当于访问常量。

**示例：运行时才确定的值的比较**

```java
final String str1 = "str";    	// 编译时确定
final String str2 = getStr();	// 运行时确定

String c = "str" + "ing";	// 常量池中的对象
String d = str1 + str2; 	// 在堆上创建的新的对象
System.out.println(c == d);	// false

public static String getStr() {
      return "ing";
}
```

**示例：intern() 方法后获取的引用的比较**

```java
// 在堆中创建字符串对象”Java“
// 将字符串对象”Java“的引用保存在字符串常量池中
String s1 = "Java";
// 直接返回字符串常量池中字符串对象”Java“对应的引用
String s2 = s1.intern();

// s1 和 s2 指向的是堆中的同一个对象
System.out.println(s1 == s2); // true

// 在堆中在单独创建一个字符串对象
String s3 = new String("Java");
// 直接返回字符串常量池中字符串对象”Java“对应的引用
String s4 = s3.intern();

// s3 和 s4 指向的是堆中不同的对象
System.out.println(s3 == s4); // false

// s1 和 s4 指向的是堆中的同一个对象
System.out.println(s1 == s4); //true
```

### 4.8 String、StringBuffer、StringBuilder 的区别

可变性：

+ `String` 是不可变的。
+ `StringBuilder` 与 `StringBuffer` 没有使用 `final` 和 `private` 关键字修饰，提供了修改字符串的方法比如 `append` 。

线程安全性：

+ `String` 中的对象是不可变的，也就可以理解为常量，线程安全。
+ `StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。
+ `StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的。

性能：

+ 对 `String` 类型进行改变的时候，会生成新的 `String` 对象，然后将指针指向新的 `String` 对象。
+ `StringBuffer` 会对 `StringBuffer` 对象本身进行操作，而不生成新的对象并改变对象引用。
+ 相同情况下使用 `StringBuilder` 相比使用 `StringBuffer` 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

**对于三者使用的总结：**

1. 操作少量的数据: 适用 `String`
2. 单线程操作字符串缓冲区下操作大量数据: 适用 `StringBuilder`
3. 多线程操作字符串缓冲区下操作大量数据: 适用 `StringBuffer`