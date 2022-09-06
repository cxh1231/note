## 1、AOP 简介

`AOP（Aspect-Oriented Programming，面向切面编程）`能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）**封装** 起来，便于**减少系统的重复代码，降低模块间的耦合度**，并有利于未来的可拓展性和可维护性。

AOP 切面编程设计到的一些专业术语：

|        术语         | 含义                                                         |
| :-----------------: | :----------------------------------------------------------- |
|   目标（Target）    | 被通知的对象                                                 |
|    代理（Proxy）    | 向目标对象应用通知之后创建的代理对象                         |
| 连接点（JoinPoint） | 目标对象的所属类中，定义的所有方法均为连接点                 |
| 切入点（Pointcut）  | 被切面拦截 / 增强的连接点（切入点一定是连接点，连接点不一定是切入点） |
|   通知（Advice）    | 增强的逻辑 / 代码，也即拦截到目标对象的连接点之后要做的事情  |
|   切面（Aspect）    | 切入点(Pointcut)+通知(Advice)                                |
|   Weaving（织入）   | 将通知应用到目标对象，进而生成代理对象的过程动作             |

Spring AOP 已经集成了 AspectJ ，AspectJ 基本是 Java 生态系统中最完整的 AOP 框架。

## 2、AOP 的实现方式（实现技术）

实现 AOP 的技术，主要分为两大类：

#### 静态代理（编译时增强）

**静态代理** 指使用 AOP 框架提供的命令进行编译，使其在编译阶段生成 AOP 代理类，也被称为**编译时增强**。

- 编译时编织（特殊编译器实现）
- 类加载时编织（特殊的类加载器实现）

#### 动态代理（运行时增强）

**动态代理** 指运行时在内存中 **临时** 生成 AOP 动态代理类，也被称为 **运行时增强**。

动态代理主要分两种：

- **`JDK` 动态代理：通过反射来接收被代理的类**，并且要求被代理的类必须实现一个接口 。JDK 动态代理的核心是 InvocationHandler 接口和 Proxy 类 。
- `CGLIB`动态代理： 如果目标类没有实现接口，那么 `Spring AOP` 会选择使用 `CGLIB` 来动态代理目标类 。`CGLIB` （ Code Generation Library ），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意， `CGLIB` 是通过继承的方式做的动态代理，因此如果某个类被标记为 `final` ，那么它是无法使用 `CGLIB` 做动态代理的

## 3、Spring AOP 与 AspectJ AOP 的区别

|         Spring AOP         |              AspectJ AOP              |
| :------------------------: | :-----------------------------------: |
| 基于动态代理（运行时增强） |      基于静态代理（编译时增强）       |
|     基于代理(Proxying)     | 基于字节码操作(Bytecode Manipulation) |
| 仅支持方法级别的 PointCut  |        支持属性级别的 PointCut        |
|         相对较简单         |     功能更强大，提供完全的AOP支持     |

## 4、AspectJ 切面注解通知

- `@Before`：**前置通知**，目标对象的方法调用之前触发；
- `@After`：**后置通知**，目标对象的方法调用之后触发；
- `@AfterRunning`：**返回通知**，目标对象的方法调用完成，在返回结果值之后触发；
- `@AfterThrowing`：**异常通知**，目标对象的方法运行中抛出 / 触发异常后触发。AfterReturning 和 AfterThrowing 两者互斥。如果方法调用成功无异常，则会有返回值；如果方法抛出了异常，则不会有返回值；
- `@Around`：**环绕通知**，围绕着方法执行，是所有通知类型中可操作范围最大的一种。

## 5、多个切面执行顺序

方法1、通常使用 `@Order` 注解直接定义切面顺序

```java
// 值越小优先级越高
@Order(3)
@Component
@Aspect
public class LoggingAspect implements Ordered {
}
```

方法2、实现`Ordered` 接口重写 `getOrder` 方法。

```java
@Component
@Aspect
public class LoggingAspect implements Ordered {

    // ....

    @Override
    public int getOrder() {
        // 返回值越小优先级越高
        return 1;
    }
}
```

