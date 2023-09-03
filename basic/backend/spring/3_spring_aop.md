## 1、AOP 简介

`AOP（Aspect-Oriented Programming，面向切面编程）`能够将那些与业务无关，却为 **业务模块** 所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）**封装** 起来，便于**减少系统的重复代码，降低模块间的耦合度**，并有利于未来的可拓展性和可维护性。

![image-20220909203139605](https://img.zxdmy.com/2022/202209092031912.png)

AOP 可以将遍布应用各处的功能分离出来形成可重用的组件。在编译期间、装载期间或运行期间实现在不修改源代码的情况下给程序动态添加功能。从而实现对业务逻辑的隔离，提高代码的模块化能力。

![image-20220909203359879](https://img.zxdmy.com/2022/202209092034171.png)

## 2、AOP 核心概念

`AOP` 切面编程涉及到的专业术语如下：

+ **切面**（Aspect）：类是对物体特征的抽象，切面就是对横切关注点的抽象
+ **连接点**（Join Point）：被拦截到的点，因为 Spring 只支持方法类型的连接点，所以在 Spring中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器
+ **切点**（Pointcut）：对连接点进行拦截的定位
+ **通知**（Advice）：所谓通知指的就是指拦截到连接点之后要执行的代码，也可以称作增强
+ **目标对象** （Target）：代理的目标对象
+ **织入**（Weabing）：织入是将增强添加到目标类的具体连接点上的过程。
    + **编译期织入**：切面在目标类编译时被织入
    + **类加载期织入**：切面在目标类加载到JVM时被织入。需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码。
    + **运行期织入**：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。SpringAOP就是以这种方式织入切面。
+ **引介**（introduction）：引介是一种特殊的增强，可以动态地为类添加一些属性
  和方法

> `Spring`采用**运行期织入**，而 `AspectJ` 采用 **编译期织入** 和 **类加载期织入**。
>
> `Spring AOP` 已经集成了 `AspectJ` ，**AspectJ 基本是 Java 生态系统中最完整的 AOP 框架**。

## 3、AOP 的核心实现：动态代理

**静态代理（编译时增强）**:

**静态代理** 指使用 AOP 框架提供的命令进行编译，使其在编译阶段生成 AOP 代理类，也被称为**编译时增强**。

- 编译时编织（特殊编译器实现）
- 类加载时编织（特殊的类加载器实现）

=  =  =  =

Spring的`AOP`是通过`动态代理`来实现的。

`动态代理` 主要有两种方式：

+ `JDK` 动态代理
+ `CgLib` 动态代理

### 3.1 JDK 动态代理

**JDK 动态代理** 是指通过反射来接收被代理的类，并且要求 **被代理的类必须实现一个接口** 。

JDK 动态代理的核心是 `InvocationHandler 接口` 和 `Proxy 类` ：

+ `InvocationHandler 接口`：通过实现这个接口，定义横切逻辑，再通过反射机制（`invoke`）调用目标类的代码，在次过程，可以包装逻辑，对目标方法进行前置后置处理；
+ `Proxy` ：利用 `InvocationHandler` 动态创建一个符合目标类实现的接口的实例，生成目标类的代理对象。

![image-20220909210826676](https://img.zxdmy.com/2022/202209092108806.png)

下面基于 `JDK 动态代理` 实现一个简单的 demo。

![image-20220909210407363](https://img.zxdmy.com/2022/202209092104656.png)

1、首先实现业务逻辑的 接口 和 具体实现：

`IMyService.java` ：

```java
package com.aop.diy.service;

public interface IMyService {

    /**
     * 业务逻辑处理
     *
     * @return 业务处理结果
     */
    String doSomething();
}
```

2、实现目标类：需要实现对应接口

`MyServiceImpl.java` ：

```java
package com.aop.diy.service.impl;

import com.aop.diy.service.IMyService;

public class MyServiceImpl implements IMyService {
    /**
     * 业务逻辑处理
     *
     * @return 业务处理结果
     */
    @Override
    public String doSomething() {
        System.out.println("com.aop.diy.service.impl.MyServiceImpl.doSomething : 正在处理业务逻辑");
        return "SUCCESS";
    }
}
```

3、创建动态代理工厂 ProxyFactory，直接用反射方式生成一个目标对象的代理对象，这里用了一个匿名内部类方式重写 `InvocationHandler` 方法：

`ProxyFactory.java` ：

```java
package com.aop.diy.factory;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class ProxyFactory {

    /**
     * 维护一个目标对象
     */
    private Object target;

    public ProxyFactory(Object target) {
        this.target = target;
    }

    /**
     * 给目标对象生成一个代理对象
     *
     * @return 代理对象
     */
    public Object getProxyInstance() {
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        // 业务逻辑处理前执行的操作
                        System.out.println("java.lang.reflect.InvocationHandler.invoke : 业务逻辑处理前执行的操作");
                        // 调用目标对象方法，执行业务逻辑
                        Object returnValue = method.invoke(target, args);
                        // 查看业务执行后的结果
                        System.out.println("java.lang.reflect.InvocationHandler.invoke : 业务执行后的结果为" + returnValue);
                        // 业务逻辑处理后执行的操作
                        System.out.println("java.lang.reflect.InvocationHandler.invoke : 业务逻辑处理后执行的操作");
                        return returnValue;
                    }
                }
        );
    }
}
```

4、主函数测试：

```java
package com.aop.diy;

import com.aop.diy.factory.ProxyFactory;
import com.aop.diy.service.IMyService;
import com.aop.diy.service.impl.MyServiceImpl;

public class Main {

    public static void main(String[] args) {
        // 创建目标对象（只能是接口）
        IMyService target = new MyServiceImpl();
        // 给目标对象，创建代理对象（被代理的类，必须实现一个接口）
        IMyService proxy = (IMyService) new ProxyFactory(target).getProxyInstance();
        // 执行代理对象的方法，触发intercept方法，从而实现对目标对象的调用
        String result = proxy.doSomething();
        System.out.println("result = " + result);
    }
}
```

5、输出结果：

```java
java.lang.reflect.InvocationHandler.invoke : 业务逻辑处理前执行的操作
com.aop.diy.service.impl.MyServiceImpl.doSomething : 正在处理业务逻辑
java.lang.reflect.InvocationHandler.invoke : 业务执行后的结果为SUCCESS
java.lang.reflect.InvocationHandler.invoke : 业务逻辑处理后执行的操作
result = SUCCESS
```

> 通过上面的示例，可以看出，JDK 动态代理 的局限性，是只能为接口创建代理实例。

### 3.2 CGLIB 动态代理

如果目标类没有实现接口，那么 `Spring AOP` 会选择使用 `CGLIB` 来动态代理目标类 。

`CGLIB` （ Code Generation Library ），是一个**代码生成**的类库，可以在运行时动态的生成某个类的子类。

其原理是**通过字节码技术为一个类创建子类**，并在子类中**采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑**。

注意， `CGLIB` 是通过**继承**的方式做的**动态代理**，因此如果某个类被标记为 `final` ，那么它是无法使用 `CGLIB` 做动态代理的

下面基于 `CGLIB 动态代理` 实现一个简单的 demo。

![image-20220909212511251](https://img.zxdmy.com/2022/202209092125743.png)

1、创建目标类，不再实现接口，实现具体的业务逻辑

`MyService.java` ：

```java
package com.zxdmy.demo.aop.cglab.service;

public class MyService {

    public String doSomething() {
        System.out.println("com.cxhit.aop.cglab.MyService.doSomething : 进行业务逻辑处理");
        return "SUCCESS";
    }
}
```

2、创建动态代理工厂：实现 `org.springframework.cglib.proxy 包` 的 `MethodInterceptor` 接口

`CgLibProxyFactory.java` ：

```java
package com.zxdmy.demo.aop.cglab.factory;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class CgLibProxyFactory implements MethodInterceptor {

    /**
     * 维护目标对象
     */
    private Object target;

    public CgLibProxyFactory(Object target) {
        this.target = target;
    }

    /**
     * 给目标对象创建一个代理对象
     *
     * @return 代理对象
     */
    public Object getProxyInstance() {
        // 1. 工具类
        Enhancer en = new Enhancer();
        // 2. 设置父类
        en.setSuperclass(target.getClass());
        // 3. 设置回调函数
        en.setCallback(this);
        // 4. 创建子类(代理对象)
        return en.create();
    }

    /**
     * 重写intercept方法，会调用目标对象的方法
     *
     * @param o           代理对象
     * @param method      目标对象的方法
     * @param objects     方法参数
     * @param methodProxy 代理对象的方法
     * @return 方法返回值
     * @throws Throwable 异常
     */
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("com.zxdmy.demo.aop.cglab.factory.CgLibProxyFactory.intercept : 业务逻辑处理前执行的操作");

        // 执行目标对象的方法
        Object returnValue = method.invoke(target, objects);

        System.out.println("com.zxdmy.demo.aop.cglab.factory.CgLibProxyFactory.intercept : 业务执行后的结果为 " + returnValue);
        System.out.println("com.zxdmy.demo.aop.cglab.factory.CgLibProxyFactory.intercept : 业务逻辑处理后执行的操作");

        return returnValue;
    }
}
```

3、主函数中调用测试：

```java
package com.zxdmy.demo.aop.cglab;

import com.zxdmy.demo.aop.cglab.factory.CgLibProxyFactory;
import com.zxdmy.demo.aop.cglab.service.MyService;

public class Main {

    public static void main(String[] args) {
        // 创建目标对象
        MyService myService = new MyService();
        // 生成代理对象
        MyService proxyService = (MyService) new CgLibProxyFactory(myService).getProxyInstance();
        // 执行代理对象的方法，触发 intercept 方法，从而实现对目标对象的调用
        String result = proxyService.doSomething();
        System.out.println("result = " + result);
    }
}
```

执行结果：

```text
com.zxdmy.demo.aop.cglab.factory.CgLibProxyFactory.intercept : 业务逻辑处理前执行的操作
com.cxhit.aop.cglab.MyService.doSomething : 进行业务逻辑处理
com.zxdmy.demo.aop.cglab.factory.CgLibProxyFactory.intercept : 业务执行后的结果为 SUCCESS
com.zxdmy.demo.aop.cglab.factory.CgLibProxyFactory.intercept : 业务逻辑处理后执行的操作
result = SUCCESS
```

## 4、Spring AOP 与 AspectJ AOP 的区别

|              Spring AOP              |                       AspectJ AOP                        |
| :----------------------------------: | :------------------------------------------------------: |
|             纯 Java 实现             |                   Java 语言的扩展实现                    |
|             编译器 Javac             |                       一般需要 ajc                       |
|    基于动态代理（**运行时增强**）    |              基于静态代理（**编译时增强**）              |
|         只支持**运行时**织入         |        支持**编译时**、**编译后**、**加载时**织入        |
|        只支持**方法级**的编织        | 支持**字段**、**方法**、**构造函数**、**静态初始值**编织 |
|  只支持 **Spring 管理的Bean**上实现  |                  **所有**域的对象可实现                  |
|       只支持**方法执行**切入点       |                    支持**所有**切入点                    |
|             速度较**慢**             |                       速度很**快**                       |
| 创建目标对象的代理，切面在代理中执行 |             执行程序前，各方面直接织入代码中             |
|              相对较简单              |              功能更强大，提供完全的AOP支持               |

## 5、AspectJ  环绕注解与使用示例

#### 注解详解：

AOP 一般有 5 种环绕方式：

|       注解       |   简介   |                         详情                         |
| :--------------: | :------: | :--------------------------------------------------: |
|    `@Before`     | 前置通知 |              目标对象的方法调用之前触发              |
|     `@After`     | 后置通知 |              目标对象的方法调用之后触发              |
| `@AfterRunning`  | 返回通知 |     目标对象的方法调用完成，在返回结果值之后触发     |
| `@AfterThrowing` | 异常通知 |     目标对象的方法运行中抛出 / 触发异常后触发。      |
|    `@Around`     | 环绕通知 | 围绕着方法执行，是所有通知类型中可操作范围最大的一种 |

> `@AfterReturning` 和 `@AfterThrowing` 两者互斥。
>
> 如果方法调用成功无异常，则会有返回值；如果方法抛出了异常，则不会有返回值。

**多个切面的情况下，可以通过 `@Order` 指定先后顺序，数字越小，优先级越高**。详见后文

![image-20220909213914338](https://img.zxdmy.com/2022/202209092139063.png)

#### 使用示例：

下面是详细的使用示例。

1、引入AOP 依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

2、自定义注解：自定义一个注解作为切点

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
@Documented
public @interface WebLog {
}
```

3、配置AOP切面：

+ @Aspect：标识切面
+ @Pointcut：设置切点，这里以自定义注解为切点，定义切点有很多其它种方式，自定义注解是比较常用的一种
+ @Before：在切点之前织入，打印了一些入参信息
+ @Around：环绕切点，打印返回参数和接口执行时间

```java
package com.zxdmy.demo.aop.diy;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.util.Objects;

@Aspect
@Component
public class WebLogAspect {

    /**
     * 以自定义 @WebLog 注解为切点
     **/
    @Pointcut("@annotation(com.zxdmy.demo.aop.diy.WebLog)")
    public void webLog() {
    }

    /**
     * 前置通知，方法调用前被调用
     *
     * @param joinPoint 切点
     */
    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {
        // 获取请求日志
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = Objects.requireNonNull(attributes).getRequest();
        // 打印请求相关参数
        System.out.println("URL : " + request.getRequestURL().toString());
        System.out.println("HTTP_METHOD : " + request.getMethod());
        System.out.println("IP : " + request.getRemoteAddr());
        // 打印请求入参
        System.out.println("Request Args : " + new ObjectMapper().writeValueAsString(joinPoint.getArgs()));
    }

    /**
     * 在切点之后织入
     */
    @After("webLog()")
    public void doAfter() {
        System.out.println("WebLogAspect.doAfter()");
    }

    /**
     * 环绕
     */
    @Around("webLog()")
    public Object doAround(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        //开始时间
        long startTime = System.currentTimeMillis();
        // 执行方法，获取返回值
        Object result = proceedingJoinPoint.proceed();
        // 打印出参
        System.out.println("Response Args : " + new ObjectMapper().writeValueAsString(result));
        // 执行耗时
        System.out.println("Time-Consuming : " + (System.currentTimeMillis() - startTime));
        return result;
    }
}
```

4、测试：只需要在接口上加上自定义注解即可

```java
@GetMapping("/hello")
@WebLog(desc = "这是一个欢迎接口")
public String hello(String name){
    return "Hello "+name;
}
```

#### 多个切面执行顺序：

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

