## 1、Spring 框架

> 问：Spring 框架的优点/好处？

`Spring` 是一款**开源的轻量级 Java 开发框架**，旨在提高开发人员的开发效率以及系统的可维护性。一般说 **Spring 框架**指的都是 `Spring Framework`，它是**很多模块的集合，**使用这些模块可以很方便地协助我们进行开发。

**Spring 最核心的思想就是不重新造轮子，开箱即用！**

## 2、Spring 框架的重要模块

`Spring` 各个模块的依赖关系如下：

![image-20220802101850532](https://img.zxdmy.com/2022/202208021018722.png)

### 2.1 核心模块（IoC 模块）

**Spring 框架的核心模块（基础模块），主要提供 IoC 依赖注入功能的支持**。

> **Spring** 其他所有的功能基本都需要依赖于该模块。

- **spring-core** ：**Spring 框架基本的核心工具类**。
- **spring-beans** ：**提供对 bean 的创建、配置和管理等功能的支持**。
- **spring-context** ：提供对国际化、事件传播、资源加载等功能的支持。
- **spring-expression** ：提供对表达式语言（Spring Expression Language） SpEL 的支持，只依赖于 core 模块，不依赖于其他模块，可以单独使用

### 2.2 AOP 模块

- **spring-aspects** ：**该模块为与 AspectJ 的集成提供支持。**
- **spring-aop** ：**提供了面向切面的编程实现**。
- **spring-instrument** ：提供了为 JVM 添加代理（agent）的功能。 具体来讲，它为 Tomcat 提供了一个织入代理，能够为 Tomcat 传递类文 件，就像这些文件是被类加载器加载的一样。没有理解也没关系，这个模块的使用场景非常有限。

### 2.3数据库连接与整合模块

- **spring-jdbc** ：**提供了对数据库访问的抽象 JDBC。不同的数据库都有自己独立的 API 用于操作数据库，而 Java 程序只需要和 JDBC API 交互，这样就屏蔽了数据库的影响**。
- **spring-tx** ：提供对事务的支持。
- **spring-orm** ： 提供对 Hibernate、JPA 、iBatis 等 ORM 框架的支持。
- **spring-oxm** ：提供一个抽象层支撑 OXM(Object-to-XML-Mapping)，例如：JAXB、Castor、XMLBeans、JiBX 和 XStream 等。
- **spring-jms** ：Java 消息服务。自 Spring Framework 4.1 以后，它还提供了对 spring-messaging 模块的继承。

### 2.4 Web 模块

- **spring-web** ：**对 Web 功能的实现提供一些最基础的支持**。
- **spring-webmvc** ： **提供对 Spring MVC 的实现**。
- **spring-websocket** ： 提供了对 WebSocket 的支持，WebSocket 可以让客户端和服务端进行双向通信。
- **spring-webflux** ：提供对 WebFlux 的支持。WebFlux 是 Spring Framework 5.0 中引入的新的响应式框架。与 Spring MVC 不同，它不需要 Servlet API，是完全异步。

### 2.5 Test 模块

Spring 的测试模块对 JUnit（单元测试框架）、TestNG（类似 JUnit）、Mockito（主要用来 Mock 对象）、PowerMock（解决 Mockito 的问题比如无法模拟 final, static， private 方法）等等常用的测试框架支持的都比较好。

## 3、Spring、Spring MVC、Spring Boot 的区别

如前文，**Spring 包含了多种功能模块**。

> 其中最重要的是 `Spring-Core` 模块（主要提供 IoC 依赖注入功能的支持） ，**Spring 中的其他模块的功能实现基本都需要依赖于该模块**。

**Spring MVC 是 Spring 中的一个很重要的模块，主要赋予 Spring 快速构建 MVC 架构的 Web 程序的能力**。

> MVC 是模型(Model)、视图(View)、控制器(Controller)的简写，其核心思想是通过将业务逻辑、数据、显示分离来组织代码。

**Spring Boot 简化了 Spring MVC 的各种配置，做到开箱即用。**

## 4、Spring IoC/DI（控制反转与依赖注入）

`IoC`（全写：`Inverse of Control`），即 **控制反转** 的意思。

`DI`（全写：`Dependency Injection`），即 **依赖注入** 的意思。

> Spring 官方文档中有这样一句话：**IoC is also known as dependency injection (DI)**，何出此言？

### 4.1 起源

众所周知，一个对象，需要实现某个功能，往往需要依赖于其他的对象的功能，比如：

```java
class A{
    private B b;
	public void doSomething(){
        b.doSomething;
	}	
}
```

但是，上面的这个代码，B 没有实例化，无法运行。

所以需要手动 new 一个 B 的实例：

```java
class A{
    private B b;
    public void doSomething(){
        // 手动 new 出 B 的实例
		b = new B;
        b.doSomething;
    }
}
```

但是，企业项目中使用传统的方式，会**带来各种问题**：

+ 比如 B 需要是一个单例，而在 A 以外的地方也需要使用 B；
+ 比如在多人开发中，B 还只是一个接口，并没有实现类，根本 new 不出来；
+ 比如 B 本身也有依赖的对象需要先实例化；
+ 比如在 A 中硬编码的方式造成了过高的耦合……

所以，我们需要使用 **控制反转** 的方式，将 **所依赖的对象实例化权限，交给其他地方控制**。

即 A 不再负责 B 的实例化，而是通过某种方式，直接获得 B 的实例即可。

其实，**依赖注入 是 Spring 框架实现 控制反转 思想的具体实现**。即依赖注入是一种设计模式，可以作为控制反转的一种实现方式。

> **所谓 依赖注入，就是将被依赖的对象传入到依赖对象中。**

### 4.2 控制反转

**IoC 是一种设计思想，而不是一个具体的技术实现。**

IoC 的思想就是**将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理**。不过， IoC 并非 Spring 特有，在其他语言中也有应用。

IoC 实现了松散耦合。

**控制反转 包含两层含义：**

- **控制** ：指的是对象创建（实例化、管理）的权力
- **反转** ：控制权交给外部环境（Spring 框架、IoC 容器）

![image-20220802104734352](https://img.zxdmy.com/2022/202208021047758.png)

**如上图，将对象之间的相互依赖关系，交给 IoC 容器管理，并由 IoC 容器完成对象的注入。**

![image-20220810173349215](https://img.zxdmy.com/2022/202208101733336.png)

> `IoC 容器` 就像工厂，当我们需要创建一个对象的时候，只需要配置好 `配置文件/注解`，无需考虑对象如何被创建，把应用从复杂的依赖关系中解放出来，极大简化开发难度。
>
> 比如：实际项目中的 `Service` 类可能依赖很多其他的类。如果需要实例化这个 Service，可能每次都需要搞清这个 Service 所有底层类的构造函数……。如果使用 IoC 的话，只需要配置好，然后在需要的地方引用即可，这极大增加了项目的可维护性，降低了开发难度。

在 Spring 中， `IoC 容器`是 Spring 用来实现 IoC 的载体， `IoC 容器`实际上就是个 `Map`（key，value），Map 中存放的是各种对象。

> Spring 时代一般通过 `XML` 文件来配置 `Bean`，后来开发人员觉得 `XML` 文件来配置不太好，于是 `SpringBoot 注解配置`就开始流行。

### 4.3 依赖注入

> **官方描述**：
> 由于某**客户类只依赖于服务类的一个接口**，而不依赖于具体服务类，所以客户类只定义一个注入点。在程序运行过程中，客户类不直接实例化具体服务类实例，而是客户类的运行上下文环境或专门组件负责实例化服务类，然后将其注入到客户类中，保证客户类的正常运行。

通常，`依赖注入` 可以通过三种方式完成，即：

- 构造函数注入
- Setter 注入
- 接口注入

在 Spring Framework 中，仅使用 **构造函数** 和 **Setter 注入**。

|        构造函数注入        |        Setter 注入         |
| :------------------------: | :------------------------: |
|        没有部分注入        |         有部分注入         |
|    不会覆盖 setter 属性    |     会覆盖 Setter 属性     |
| 任意修改都会创建一个新实例 | 任意修改不会创建一个新实例 |
|     适用于设置很多属性     |     适用于设置少量属性     |

#### 构造函数注入

即：用构造函数的形式来实现注入。

实例：

比如有参数的构造函数如下：

```java
public class AccountServiceImpl implements IAccountService {
    private String name;
    private Integer age;
    private Date birthday;
    
    // 定义 构造函数，实现依赖注入
    public AccountServiceImpl(String name, Integer age, Date birthday) {
        this.name = name;
        this.age = age;
        this.birthday = birthday;
    }
    
    public void saveAccount() {
        System.out.println("Service 中的 saveAccount 方法执行了。。。");
    }
}
```

配置 bean.xml：

```xml
	<bean id="accountService" class="com.xxx.service.impl.AccountServiceImpl">
        <constructor-arg name="age" value="18"></constructor-arg>
        <constructor-arg name="name" value="Zhang San"></constructor-arg>
        <constructor-arg name="birthday" ref="now"></constructor-arg>
    </bean>
    <bean id="now" class="java.util.Date"></bean>
```

使用：

```java
// 1. 获取核心容器对象
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
// 2. 根据 id 获取 Bean 对象
IAccountService accountService = (IAccountService)ac.getBean("accountService");
// 3. 使用 Bean 的方法
accountService.saveAccount();
```

#### Setter 注入

方法如下：

```java
public class AccountServiceImpl implements IAccountService{

    private String name;
    private Integer age;
    private Date birthday;

    public String getName() {
        return name;
    }

    // 使用 Setter 进行依赖注入
    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }
    
    public void saveAccount() {
        System.out.println("Service 中的 saveAccount 方法执行了。。。");
    }
}
```

配置 bean.xml 进行属性注入：

```xml
    <bean id="accountService2" class="com.xxx.service.impl.AccountServiceImpl">
        <property name="name" value="Bruce"></property>
        <property name="age" value="21"></property>
        <property name="birthday" ref="now"></property>
    </bean>
```

使用：

```java
// 1. 获取核心容器对象
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
// 2. 根据 id 获取 Bean 对象
IAccountService accountService = (IAccountService)ac.getBean("accountService");
// 3. 使用 Bean 的方法
accountService.saveAccount();
```

## 5、Spring Bean

简单来说，`Bean` 代指的就是那些被 `IoC 容器`所管理的 `对象`。

可以通过配置元数据定义对象，进而交给 IoC 容器管理。

配置元数据的方式可以是 XML、注解或Java配置类。

`org.springframework.beans`和 `org.springframework.context` 这两个包是 `IoC` 实现的基础。

### 5.1 将类声明为 Bean 的注解

一般使用 `@Component`、`@Repository` 、`@Service`、`@Controller`将类声明为 Bean，详情如下：

+ `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
+ `@Repository` : 对应**持久层**即 Dao 层，主要用于数据库相关操作。
+ `@Service` : 对应**服务层**，主要涉及一些复杂的逻辑，需要用到 Dao 层。
+ `@Controller` : 对应 Spring MVC **控制层**，主要接受用户请求并调用 Service 层返回数据给前端页面。

#### @Component 和 @Bean 的区别

- `@Component` 注解作用于类，而`@Bean`注解作用于方法。
- `@Component`通常是通过类路径扫描来自动侦测以及自动装配到 Spring 容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了 Spring 这是某个类的实例，当我需要用它的时候还给我。
- `@Bean` 注解比 `@Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册 bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

### 5.2 注入 Bean 的注解

Spring 内置的 `@Autowired` 以及 JDK 内置的 `@Resource` 和 `@Inject` 都可以注入（自动装配） Bean，`@Autowired` 和`@Resource`使用的比较多。

|     注解     |  来源  |         优先注入方式         |
| :----------: | :----: | :--------------------------: |
| `@Autowired` | Spring | `byType`（根据类型进行匹配） |
| `@Resource`  |  JDK   | `byName`（根据名称进行匹配） |

详细如下。

#### @Autowired

**`@Autowired` 属于 Spring 内置的注解，默认的注入方式为`byType`（根据类型进行匹配），即优先根据接口类型去匹配并注入 Bean （接口的实现类）。**

但是：当一个接口存在多个实现类的话，`byType`这种方式就无法正确注入对象，会变成 `byName`（根据名称进行匹配），这个名称通常就是类名（首字母小写）。

比如： `SmsService` 接口有两个实现类: `SmsServiceImpl1`和 `SmsServiceImpl2`，且它们都已经被 Spring 容器所管理，那么下面的第一种方式就无法注入 Bean。

```java
// 报错，byName 和 byType 都无法匹配到 bean
@Autowired
private SmsService smsService;

// 正确注入 SmsServiceImpl1 对象对应的 bean，通过名称匹配
@Autowired
private SmsService smsServiceImpl1;

// 正确注入  SmsServiceImpl1 对象对应的 bean
// smsServiceImpl1 就是我们上面所说的名称
@Autowired
@Qualifier(value = "smsServiceImpl1")
private SmsService smsService;
```

> 对于一个接口多个实现类的情况，建议通过 `@Qualifier` 注解来显示指定名称。

#### @Resource

`@Resource` 属于 JDK 提供的注解，默认注入方式为 `byName`。如果无法通过名称匹配到对应的 Bean 的话，注入方式会变为 `byType`。

`@Resource` 注解有两个比较重要且日常开发常用的属性：`name`（名称）、`type`（类型）。

+ 如果不指定`name` 和 `type` ，则先 `byName` ，再 `byType` ；
+ 如果仅指定 `name` ，则注入方式为`byName`；
+ 如果仅指定 `type` ，则注入方式为`byType`；
+ 如果同时指定 `name`  和 `type` 属性，则注入方式为`byType`+`byName`（TM吃饱了撑的写这么多）

比如： `SmsService` 接口有两个实现类: `SmsServiceImpl1`和 `SmsServiceImpl2`：

```java
// 报错，byName 和 byType 都无法匹配到 bean
@Resource
private SmsService smsService;

// 正确注入 SmsServiceImpl1 对象对应的 bean
@Resource
private SmsService smsServiceImpl1;

// 正确注入 SmsServiceImpl1 对象对应的 bean（比较推荐这种方式）
@Resource(name = "smsServiceImpl1")
private SmsService smsService;
```

### 5.3 Bean 的作用域

Spring 中 Bean 的作用域通常有下面几种：

- **singleton** : IoC 容器中只有唯一的 bean 实例。Spring 中的 bean 默认都是单例的，是对单例设计模式的应用。
- **prototype** : 每次获取都会创建一个新的 bean 实例。也就是说，连续 `getBean()` 两次，得到的是不同的 Bean 实例。
- **request** （仅 Web 应用可用）: 每一次 HTTP 请求都会产生一个新的 bean（请求 bean），该 bean 仅在当前 HTTP request 内有效。
- **session** （仅 Web 应用可用） : 每一次来自新 session 的 HTTP 请求都会产生一个新的 bean（会话 bean），该 bean 仅在当前 HTTP session 内有效。
- **application/global-session** （仅 Web 应用可用）： 每个 Web 应用在启动时创建一个 Bean（应用 Bean），，该 bean 仅在当前应用启动时间内有效。
- **websocket** （仅 Web 应用可用）：每一次 WebSocket 会话产生一个新的 bean。

**可以通过 `XML` 或 `注解方式` 配置 Bean 的作用域：**

XML：

```xml
<bean id="..." class="..." scope="singleton"></bean>
```

注解：

```java
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Person personPrototype() {
    return new Person();
}
```

### 5.4 Bean 的生命周期

![image-20220802151417390](https://img.zxdmy.com/2022/202208021514870.png)

可以简单理解为5步：**实例化 — 设置属性 — 初始化 — 使用 — 销毁**。

**详细生命周期如下**：

- Bean 容器找到配置文件中 Spring Bean 的定义。
- Bean 容器利用 Java Reflection API 创建一个 Bean 的实例。
- 如果涉及到一些属性值 利用 `set()`方法设置一些属性值。
- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入 Bean 的名字。
- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
- 如果 Bean 实现了 `BeanFactoryAware` 接口，调用 `setBeanFactory()`方法，传入 `BeanFactory`对象的实例。
- 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
- 如果 Bean 实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
- 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法
- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

### 5.5 单例 Bean 的线程安全问题

单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候是存在资源竞争的。不过，大部分 bean 实际都是无状态（没有实例变量）的（比如 Dao、Service），这种情况下， bean 是线程安全的。

常见的有两种解决办法：

1. 在 bean 中尽量避免定义可变的成员变量。
2. 在类中定义一个 `ThreadLocal` 成员变量，将需要的可变成员变量保存在 `ThreadLocal` 中（推荐的一种方式）。

不过，**大部分 Bean 实际都是无状态**（没有实例变量）的（比如 Dao、Service），这种情况下， Bean 是线程安全的。

### 5.6 Bean 的循环依赖问题

#### 产生情况

**循环依赖的产生** 可能有很多情况，比如：

+ A 的构造方法中依赖了 B 的实例对象，同时 B 的构造方法中依赖了 A 的实例对象；
+ A 的构造方法中依赖了 B 的实例对象，同时 B 的某个 field 或者 setter 需要 A 的实例对象，以及反之；
+ A 的某个 field 或者 setter 依赖了B的实例对象，同时 B 的某个 field 或者 setter 依赖了 A 的实例对象，以及反之……

```java
@Service
public class TestService1 {

    @Autowired
    private TestService2 testService2;

    public void test1() {
    }
}

@Service
public class TestService2 {

    @Autowired
    private TestService1 testService1;

    public void test2() {
    }
}
```

#### 解决方案

Spring 对循环依赖的处理有三种情况：

1. **构造器的循环依赖**：这种依赖 Spring **无法处理**，抛出 BeanCurrentlyInCreationException 异常；
2. **单例模式**下的 Setter 循环依赖：通过 “**三级缓存**” 处理循环依赖；
3. 非单例循环依赖：无法处理。

![image-20220811172926799](https://img.zxdmy.com/2022/202208111729512.png)

#### 单例模式下的 Setter 循环依赖解决方案

Spring 的单例对象的初始化主要分为如下三步：

![image-20220811102121676](https://img.zxdmy.com/2022/202208111023305.png)

1. createBeanInstance：实例化，其实也就是调用对象的构造方法实例化对象；
2. populateBean：填充属性，这一步主要是多bean的依赖属性进行填充；
3. initializeBean：调用 Spring xml 中的 init 方法。

**循环依赖** 主要发生在第一、第二步。这就需要用到下面的 **三级缓存** 解决。

|         缓存          | 用途                                                 |              结构               |
| :-------------------: | ---------------------------------------------------- | :-----------------------------: |
|   singletonObjects    | 存放完全初始化的 Bean，可直接使用                    |      `Map<String, Object>`      |
| earlySingletonObjects | 存放原始的 Bean 对象（未填充属性），用于解决循环依赖 |      `Map<String, Object>`      |
|  singletonFactories   | 存放 Bean的工厂对象，用于解决循环依赖                | `Map<String, ObjectFactory<?>>` |

比如：**A 的某个 field 或 Setter 依赖了 B 的实例对象，同时 B 的某个 field 或 Setter 依赖了 A 的实例对象**：

+ 最初：A 不在缓存中，需要初始化
+ A 实例化（create）：A 的构造函数创建 Bean A，并通过 `ObjectFactory` 提前曝光在 `singletonFactories`（三级缓存）；
+ A 填充属性：发现自己依赖于 B，就尝试 get(B) ，发现 B 没有在缓存中，则需要对 B 进行实例化：
  + B 实例化：B 的构造函数创建 Bean B，并通过 `ObjectFactory` 提前曝光在 `singletonFactories`；
  + B 填充属性：发现自己依赖于 A，就尝试 get(B) ：
    + 尝试一级缓存：不在，因为 A 没有完全初始化
    + 尝试二级缓存：不在
    + 尝试三级缓存：可以通过 `ObjectFactory.getObject` 拿到 A 的对象
  + B 初始化：B 拿到 A 的对象，顺利完成初始化，并将自己放入 **一级缓存**
+ A 初始化：返回 A，此时 A 顺利拿到了 B 的对象，完成初始化，同样放入 **一级缓存**。

> 虽然通过缓存 ObjectFactory 拿到的 Bean 实例不是完整的 Bean 实例，由于 Bean 是单例的，所以后续初始化完成后，该 Bean 实例的引用地址不变，故可以正常使用。

总之，在实际项目中，不要使用基于构造函数的依赖注入，可以通过以下方式解决：

1. 在字段上使用 `@Autowired` 注解，让 Spring 决定在合适的时机注入；

2. 用基于 Setter 方法的依赖注入。

### 5.7 Bean 的同名问题

- 同一个配置文件内同名的Bean，以最上面定义的为准
- 不同配置文件中存在同名Bean，后解析的配置文件会覆盖先解析的配置文件
- 同文件中`@ComponentScan`和`@Bean`出现同名Bean。同文件下`@Bean`的会**生效**，@ComponentScan扫描进来不会生效。通过@ComponentScan扫描进来的优先级是最低的，原因就是它扫描进来的Bean定义是最先被注册的。

## 6、Spring AOP

### 6.1 AOP 简介

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

### 6.2 AOP 的实现方式（实现技术）

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

### 6.3 Spring AOP 与 AspectJ AOP 的区别

|         Spring AOP         |              AspectJ AOP              |
| :------------------------: | :-----------------------------------: |
| 基于动态代理（运行时增强） |      基于静态代理（编译时增强）       |
|     基于代理(Proxying)     | 基于字节码操作(Bytecode Manipulation) |
| 仅支持方法级别的 PointCut  |        支持属性级别的 PointCut        |
|         相对较简单         |     功能更强大，提供完全的AOP支持     |

### 6.4 AspectJ 切面注解通知

- `@Before`：**前置通知**，目标对象的方法调用之前触发；
- `@After`：**后置通知**，目标对象的方法调用之后触发；
- `@AfterRunning`：**返回通知**，目标对象的方法调用完成，在返回结果值之后触发；
- `@AfterThrowing`：**异常通知**，目标对象的方法运行中抛出 / 触发异常后触发。AfterReturning 和 AfterThrowing 两者互斥。如果方法调用成功无异常，则会有返回值；如果方法抛出了异常，则不会有返回值；
- `@Around`：**环绕通知**，围绕着方法执行，是所有通知类型中可操作范围最大的一种。

### 6.5 多个切面执行顺序

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

## 7、Spring MVC

### 7.1 Spring MVC 核心组件

**MVC 是模型（Model）、视图（View）、控制器（Controller）的简写**，其核心思想是通过将业务逻辑、数据、显示分离来组织代码。

**Spring MVC 的核心组件有：**

|           组件           | 说明                                                         |
| :----------------------: | ------------------------------------------------------------ |
|  **DispatcherServlet**   | Spring MVC 的**核心组件**，是请求的入口，负责协调各个组件工作 |
|    MultipartResolver     | 内容类型( `Content-Type` )为 `multipart/*` 的请求的解析器，例如解析处理文件上传的请求，便于获取参数信息以及上传的文件 |
|    **HandlerMapping**    | 请求的**处理器匹配器**（映射器），负责为请求找到合适的 `HandlerExecutionChain` 处理器执行链，包含处理器（`handler`）和拦截器们（`interceptors`） |
|    **HandlerAdapter**    | **处理器的适配器**。因为处理器 `handler` 的类型是 Object 类型，需要有一个调用者来实现 `handler` 是怎么被执行。Spring 中的处理器的实现多变，比如用户处理器可以实现 Controller 接口、HttpRequestHandler 接口，也可以用 `@RequestMapping` 注解将方法作为一个处理器等，这就导致 Spring MVC 无法直接执行这个处理器。所以这里需要一个处理器适配器，由它去执行处理器 |
| HandlerExceptionResolver | 处理器异常解析器，将处理器（ `handler` ）执行时发生的异常，解析( 转换 )成对应的 ModelAndView 结果 |
|       **Handler**        | **请求处理器**                                               |
|     **ViewResolver**     | **视图解析器**，根据视图名和国际化，获得最终的视图 View 对象 |

### 7.2 工作原理

**Spring MVC 原理如下图所示：**

![](https://img.zxdmy.com/2022/202207062051678.png)

1. **客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。**
2. **`DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。**
3. **解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。**
4. **`HandlerAdapter` 会根据 `Handler`来调用真正的处理器开处理请求，并处理相应的业务逻辑。**
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）

## 8、Spring 事务

### 8.1 Spring 事务的管理方式（实现方式）

- **编程式事务** ： 在代码中硬编码(不推荐使用) : 通过 `TransactionTemplate`或者 `TransactionManager` 手动管理事务，实际应用中很少使用，但是对于你理解 Spring 事务管理原理有帮助。
- **声明式事务** ： 在 XML 配置文件中配置或者直接基于注解（推荐使用） : 实际是通过 AOP 实现（基于`@Transactional` 的全注解方式使用最多）

### 8.2 Spring 框架的事务管理优点

- 提供了跨不同事务 api（如JTA、JDBC、Hibernate、JPA和JDO）的一致编程模型。
- 为编程事务管理提供了比JTA等许多复杂事务API更简单的API。
- 支持声明式事务管理。
- 很好地集成了 Spring 的各种数据访问抽象。

### 8.3 Spring 事务定义的传播行为（传播规则）

**当事务方法被另一个事务方法调用时，必须指定事务应该如何传播**。

事务的传播行为主要有一下 7 种：

|          事务名称           |             如果当前存在事务             | 如果当前没有事务 |
| :-------------------------: | :--------------------------------------: | :--------------: |
|    **REQUIRED**（默认）     |                加入该事务                | 创建一个新的事务 |
|      **REQUIRES_NEW**       |   创建新事务（互相独立），挂起当前事务   | 创建一个新的事务 |
|          SUPPORTS           |                加入该事务                | 以非事务方式运行 |
|        NOT_SUPPORTED        |               挂起当前事务               | 以非事务方式运行 |
| **MANDATORY** (*mandatory*) |               加入当前事务               |     抛出异常     |
|    **NESTED** (*nested*)    | 创建一个事务作为当前事务的嵌套事务来运行 | 创建一个新的事务 |
|            NEVER            |                 抛出异常                 | 以非事务方式运行 |

使用示例：

```java
@Override
@Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)
public int saveRole(SysRole role, Integer[] menusIds) {
    // TODO 
}
```

### 8.4 Spring 事务中的隔离级别有哪几种?

> 这个问题和 MySQL 数据库的基本一致。

- default : 使用后端数据库默认的隔离级别
- read_uncommitted：读未提交
- read_committed：读已提交
- repeatable_read：可重复的
- serializable：串行化

## 7、Spring 框架中用到的设计模式

- **工厂设计模式** : Spring 使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** : Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。





