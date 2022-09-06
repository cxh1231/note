## 1、Spring IoC&DI 起源

`IoC`（全写：`Inverse of Control`），即 **控制反转** 的意思。

`DI`（全写：`Dependency Injection`），即 **依赖注入** 的意思。

> Spring 官方文档中有这样一句话：**IoC is also known as dependency injection (DI)**，何出此言？

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

## 2、IoC 概述

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

所以，`IoC` 的底层，主要基于：

+ xml 解析
+ 工厂模式
+ 反射机制

## 3、IoC 容器

#### 两种 IoC 容器

Spring 提供了两种**容器**类型：`BeanFactory`和`ApplicationContext`。

| 异同 |                  BeanFactory                  |               ApplicationContext               |
| :--: | :-------------------------------------------: | :--------------------------------------------: |
| 接口 | org.springframework.beans.factory.BeanFactory | org.springframework.context.ApplicationContext |
|      |              IoC 容器的基本实现               |    BeanFactory 的子接口，提供了更高级的特性    |
|      |    Spring 框架的基础设施，面向 Spring 本身    |          面向使用 Spring 框架的开发者          |
|      |      默认采用延迟初始化策略（lazy-load）      |            默认全部初始化并绑定完成            |
|      |               所需要的资源有限                |               需要更多的系统资源               |

#### ApplicationContext 初始化方式







## 4、依赖注入方式

> **官方描述**：
> 由于某**客户类只依赖于服务类的一个接口**，而不依赖于具体服务类，所以客户类只定义一个注入点。在程序运行过程中，客户类不直接实例化具体服务类实例，而是客户类的运行上下文环境或专门组件负责实例化服务类，然后将其注入到客户类中，保证客户类的正常运行。

通常，`依赖注入` 可以通过三种方式完成，即：

- **构造函数注入**
- **Setter 注入**
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