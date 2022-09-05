

**反射** 赋予了框架在`运行时分析类以及执行类中方法的能力`，即通过反射可以获取任何一个类的所有属性和方法，进而调用这些方法和属性，是框架的灵魂。

## 1、反射机制优缺点

- **优点** ：可以让代码更加灵活，为各种框架的开箱即用功能提供了便利;
- **缺点** ：在运行时有了分析操作类的能力，这同样也 `增加了安全问题`。比如可以无视泛型参数的安全检查（泛型参数的安全检查发生在编译时）。另外，反射的性能也要稍差点，不过，对于框架来说实际是影响不大的。

## 2、反射的使用示例

**可以通过如下方法获取反射中的 Class 对象。**

+ `Class.forName(“类的路径”)`：知道该类的全路径名，可使用此方法

```java
Class clz = Class.forName("java.lang.String");
```

+ `类名.class`：在编译前就知道操作的 Class

```java
Class clz = String.class;
```

+ `对象名.getClass()`

```java
String str = new String("Hello");
Class clz = str.getClass();
```

+ 如果是基本类型的包装类，可以调用包装类的`Type属性`来获得该包装类的Class对象。

**下面是一个完整的示例：**

```java
import java.lang.reflect.Method;

public class Book {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public static void main(String[] args) {
        // 正常调用
        Book book = new Book();
        book.setName("西游记");
        System.out.println(book.getName());

        // 使用反射调用
        try {
            // 1. 获取类的 Class 对象实例
            Class<?> clazz = Class.forName("test2.Book");
            // 2. 使用对象的 newInstance 方法获取反射类对象
            Object bookObj = clazz.newInstance();
            // 3. 获取[setName]方法的 Method 对象
            Method setPriceMethod = clazz.getMethod("setName", String.class);
            // 4. 利用 invoke 方法调用[setName]方法
            setPriceMethod.invoke(bookObj, "三国演义");
            // 5. 获取[getName]方法的 Method 对象，并通过 invoke 方法调用[getName]方法
            System.out.println(clazz.getMethod("getName").invoke(bookObj));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 3、反射的应用场景

业务代码一般用不到反射机制。

+ 像 Spring/Spring Boot、MyBatis 等，**这些框架中大量使用了动态代理，而动态代理的实现也依赖反射。**

+ **注解** 的实现也用到了反射。基于反射分析类，获取到类/属性/方法/方法的参数上的注解，然后做进一步的处理。

- 反射让开发人员可以**通过外部类的全路径名创建对象**，并使用这些类，实现一些扩展的功能。
- 反射让开发人员可以**枚举出类的全部成员**，包括构造函数、属性、方法。以帮助开发者写出正确的代码。
- 测试时可以**利用反射 API 访问类的私有成员**，以保证测试代码覆盖率。

#### 场景1：JDBC 的数据库的连接

在JDBC 的操作中，第一步就是通过 `Class.forName()` 加载数据库的驱动程序 （通过反射加载，前提是引入相关了Jar包）

```java
 Class.forName("com.mysql.jdbc.Driver"); // 使用CLASS 类加载驱动程序 ,反射机制的体现 
```

#### 场景2：Spring 框架的 xml 配置模式

Spring 通过 XML 配置模式装载 Bean 的过程中，通过对 XML 内容进行解析，得到对应实体类的字节码字符串，以及相关的属性信息，然后**使用反射机制，根据这个字符串获得某个类的Class实例**，进而动态配置实例的属性。

## 4、注解

`Annotation` （注解） 是 Java5 开始引入的新特性，可以看作是一种特殊的注释，主要用于修饰类、方法或者变量。

> 注解本质是一个继承了`Annotation` 的特殊接口。

**注解只有被解析之后才会生效**，常见的解析方法有两种：

- **编译期直接扫描** ：编译器在编译 Java 代码的时候扫描对应的注解并处理，比如某个方法使用`@Override` 注解，编译器在编译的时候就会检测当前的方法是否重写了父类对应的方法。
- **运行期通过反射处理** ：像框架中自带的注解（比如 Spring 框架的 `@Value` 、`@Component`）都是通过反射来进行处理的。