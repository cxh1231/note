## 1、面向对象概述

### 1.1 面向对象与面向过程

- `面向过程` **把解决问题的过程拆成一个个方法，通过一个个方法的执行解决问题**。比如单片机、嵌入式开发一般采用面向过程开发，性能比较高。
- `面向对象` 会**先抽象出对象，然后用对象执行方法的方式解决问题**。有封装、继承、多态性的特性，可以设计出低耦合的系统，使系统更加灵活、更加易于维护。易维护、易复用、易扩展。但性能比面向过程低。

### 1.2 面向对象三大特征

+ 封装
+ 继承
+ 多态

#### 封装

**封装** : 把一个对象的状态信息（也就是属性）隐藏在对象内部，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性。

+ 比如`实体类`。

#### 继承

**继承** : 不同类型的对象，相互之间经常有一定数量的共同点。继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承，可以快速地创建新的类，可以提高代码的重用，程序的可维护性，节省大量创建新类的时间 ，提高我们的开发效率。

+ 子类拥有父类对象所有的属性和方法，但无法访问，子类的对象可以访问。
+ 子类可以拥有自己属性和方法；
+ 子类可以用自己的方式实现父类的方法。

示例：

```java
// 父类
public class Animal { 
    private String name;  
    private int id; 
    // 构造方法
    public Animal(String myName, int myid) { 
        name = myName; 
        id = myid;
    } 
    public void eat(){ 
        System.out.println(name+"正在吃"); 
    }
    public void sleep() {
		System.out.println(name + "正在睡觉");
	}
}
// 子类
public class Mouse extends Animal { 
    // 继承父类的构造方法
    public Mouse(String myName, int myid) { 
        super(myName, myid); 
    }
}
// 使用
public static void main(String[] args) {
	Animal mouse = new Mouse("张三", 0);
    // 或：Mouse mouse = new Mouse("张三", 0);
	mouse.eat();
    mouse.sleep();
}
// 结果
张三正在吃
张三正在睡觉
```

#### 多态

**多态** : 表示一个对象具有多种的状态，具体表现为父类的引用指向子类的实例。

+ `多态是方法的多态`，属性没有多态；
+ `方法可重写`，不能实现重写的也就没有多态可言；
+ 父类引用指向子类对象：`Parent p = new Child();`

> 即三个必要条件：继承、重写和向上转型

示例：

```java
//父类
class Shape {
	void draw() {
	}
}
//继承
class Circle extends Shape {
	// 多态：重写父类方法
	void draw() {
		System.out.println("Circle.draw()");
	}
}
//继承
class Square extends Shape {
	// 多态：重写父类方法
	void draw() {
		System.out.println("Square.draw()");
	}
}
// 使用
Shape shape = new Circle();
shape.draw(); // 输出：Circle.draw()
Shape shape2 = new Square();
shape2.draw(); // 输出：Square.draw()
Circle circle = new Circle();
circle.draw(); // 输出：Circle.draw()
```

多态又分为 **编译时多态（又称静态多态）** 和 **运行时多态（又称动态多态）**。

**重载（overload）就是编译时多态**的一个例子，编译时多态在编译时就已经确定，运行的时候调用的是确定的方法。

**通常所说的多态指的都是运行时多态，也就是编译时不确定究竟调用哪个具体方法，一直延迟到运行时才能确定。**

## 2、对象概述

### 2.1 对象简介

**对象**是**类的一个实例**，有状态和行为。

> **对象实例** 在 **堆** 内存中，**对象引用** 存放在 **栈** 内存中。

一个**对象引用**可以指向 `0` 个或 `1` 个对象，一个对象可以有 `n` 个引用指向它。

- **对象相等** 一般比较的是内存中存放的**内容**是否相等。
- **引用相等** 一般比较的是他们指向的**内存地址**是否相等。

如表达式 `A a1 = new A();` ：

+ `A` 是**类**；
+ `a1` 是对象**引用**，`a1`不是对象；
+ `new A()` 是**对象**，`a1`引用 指向 `new A()` 这个对象。

> 一个类的实例从`new` 开始的过程？详见 JVM 篇章，类的生命周期。

### 2.2 创建的创建方式

`Java` 中提供了以下四种创建对象的方式:

- `new` 创建新对象：创建 **类的实例**，**对象引用** 指向类的实例。
- 通过**反射**机制
- 采用 `clone` 机制
- 通过 **序列化** 机制

### 2.3 不可变对象

**不可变对象** 指对象一旦被创建，状态就不能再改变，任何修改都会创建一个新的对象。

如 `String`、`Integer`及其它包装类就是不可变对象。

**不可变对象最大的好处是线程安全**。

> 可以创建一个包含可变对象的不可变对象。
>
> 比如`final Person[] persons = new Persion[]{}`，`persons`是不可变对象的引用，但其数组中的 `Person` **实例** 却是可变的。
>
> 这种情况下需要特别谨慎，不要共享可变对象的引用。
>
> 这种情况下，如果数据需要变化时，就返回原对象的一个拷贝。

## 3、构造方法

**构造方法** 是一种特殊的方法，主要作用是 **完成对象的初始化工作**。

- 名字与类名相同。
- 没有返回值，但不能用 `void` 声明构造函数。
- 生成类的对象时自动执行，无需调用。

如果类不写构造方法，则Java 默认有一个**无参构造方法**。

**如果写了构造方法，则Java不再默认添加无参构造方法**。

**构造方法不能被 override（重写），但是可以 overload（重载）**。

> 推荐：如果**重载**了有参的构造方法，要把无参的构造方法也写出来。

## 4、方法的重载与重写

方法的**重载**和**重写**都是实现**多态**的方式，区别在于：

+ **重载** 实现的是**编译时**的多态性；
+ **重写** 实现的是**运行时**的多态性。

![image-20220809163417776](https://img.zxdmy.com/2022/202208091634574.png)

#### 重载

指的是同样的一个方法，能够根据输入数据的不同，做出不同的处理，如：

```java
StringBuilder sb = new StringBuilder();
StringBuilder sb2 = new StringBuilder("HelloWorld");
```

即：**同一个类中多个同名方法根据不同的传参来执行不同的逻辑处理**。

> 不能根据返回值类型来区分重载的方法。

#### 重写

重写发生在运行期，是 **子类对父类的允许访问的方法的实现过程进行重新编写**。

+ 方法名、参数列表必须相同；
+ 子类方法**返回值**类型应比父类方法返回值类型更小（即可以是引用类型的子类）或相等；
+ 抛出的异常范围**小于等于**父类；
+ 访问修饰符范围**大于等于**父类；
+ 如果父类方法访问修饰符为 `private/final/static` 则子类就不能重写该方法，但是被 `static` 修饰的方法能够被再次声明；
+ 构造方法无法被重写。

即：**重写就是子类对父类方法的重新改造，外部样子不能改变，内部逻辑可以改变**。

## 5、访问修饰符 public、private、protected

- `public` ：**对所有类可见**。使用对象：类、接口、变量、方法
- `protected` ：**对同一包内的类和所有子类可见**。使用对象：变量、方法。 **注意：不能修饰类（外部类）**。
- `default` (即默认，什么也不写）: **在同一包内可见**，不使用任何修饰符。使用对象：类、接口、变量、方法。
- `private` ：**在同一类内可见**。使用对象：变量、方法。 **注意：不能修饰类（外部类）**

![image-20220809154320321](https://img.zxdmy.com/2022/202208091543673.png)

## 6、this 关键字

`this` 是**自身的一个对象**，代表**对象本身**，可以理解为：**指向对象本身的一个指针**。

`this` 的用法在Java 中大体可以分为`3`种：

1. 普通的直接引用，`this` 相当于是指向当前对象本身；
2. 形参与成员变量名字重名，用`this`来区分；
3. 引用本类的构造函数。

```java
public Person(String name,int age){
    this.name=name;
    this.age=age;
}
```

## 7、接口 interface 和抽象类 abstract

|        |                   interface                    |                      abstract                       |
| :----: | :--------------------------------------------: | :-------------------------------------------------: |
| **同** |                  不能被实例化                  |                    不能被实例化                     |
|        |                 可包含抽象方法                 |                   可包含抽象方法                    |
|  JDK8  |      可以有默认方法（`default`修饰方法 ）      |        可以有默认方法（`default`修饰方法 ）         |
| **异** | 主要对类的行为进行约束，实现接口即具有全部功能 |          主要用于代码的复用，强调所属关系           |
|        |      接口是对行为的抽象，是一种行为的规范      |          抽象是对类的抽象，是⼀种模板设计           |
|        |         不能有非 `default` 方法的实现          |                 可以有非抽象的方法                  |
|        |   除了 `static`、`final` 变量，不能有其他的    |      不一定，成员变量默认 `default` 子类随便用      |
|        |           **一个类可以实现多个接口**           |            **一个类只能实现一个抽象类**             |
|        |     接口可以通过 `extends` 关键字扩展接口      |                                                     |
|        |            接口方法默认是 `public`             | **抽象方法**可以是 `public`、`protected`、`default` |
|        |                                                |   **抽象方法** 为了能够被重写，不能使用 `private`   |
|  JDK8  |          接口可以有默认方法和静态方法          |                                                     |
|  JDK9  |       接口可以引入私有方法和私有静态方法       |                                                     |

## 8、引用拷贝、深拷贝与浅拷贝

- **引用拷贝**：引用拷贝就是两个不同的引用指向同一个对象。
- **浅拷贝**：浅拷贝会在**堆**上创建一个**新的对象**（区别于引用拷贝的一点），不过，如果**原对象内部**的属性是**引用类型**的话，浅拷贝会**直接复制内部对象的引用地址**，也就是说**拷贝对象和原对象共用同一个内部对象**。
    - `Object`类提供的`clone()`方法可以非常简单地实现对象的**浅拷贝**。

- **深拷贝** ：深拷贝会**完全复制**整个对象，包括这个对象所包含的内部对象。
    - **重写克隆方法**：重写克隆方法，引用类型变量单独克隆，这里可能会涉及多层递归。
    - **序列化**：可以先对**原对象**序列化，再 **反序列化成拷贝对象**。


![image-20220729110217967](https://img.zxdmy.com/2022/202207291102912.png)

## 9、方法调用的值传递与引用传递

> **Java 中只有值传递**。

+ **值传递**：指的是在方法调用时，**传递的参数是按值的拷贝传递**，传递的是值的拷贝，也就是说传递后就互不相关了。
+ **引用传递**：指的是在方法调用时，传递的参数是按引用进行传递，其实**传递的是引用的地址**，也就是变量所对应的内存空间的地址。传递的是值的引用，也就是说传递前和传递后都指向同一个引用（也就是同一个内存空间）。

**基本类型**作为参数被传递时肯定是值传递；

**引用类型**作为参数被传递时也是值传递，只不过“值”为对应的引用，并不是对象本身。