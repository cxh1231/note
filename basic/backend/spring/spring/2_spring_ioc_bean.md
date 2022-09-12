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

> 以前是我们想要什么，就自己创建什么，现在是我们需要什么，IoC 容器就给我们送来什么。

其实，**依赖注入 是 Spring 框架实现 控制反转 思想的具体实现**。即依赖注入是一种设计模式，可以作为控制反转的一种实现方式。

> **所谓 依赖注入，就是将被依赖的对象传入到依赖对象中。**

## 2、IoC 概述

**IoC 是一种设计思想，而不是一个具体的技术实现。**

IoC 的思想就是**将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理**。不过， IoC 并非 Spring 特有，在其他语言中也有应用。

> IoC 实现了松散耦合。

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

#### 容器分类

Spring 提供了两种**容器**类型：基本容器 `BeanFactory` 和 扩展容器 `ApplicationContext`。

|  异同  |                  BeanFactory                   |               ApplicationContext               |
| :----: | :--------------------------------------------: | :--------------------------------------------: |
|  接口  | org.springframework.beans.factory.BeanFactory  | org.springframework.context.ApplicationContext |
|        |               IoC 容器的基本实现               |    BeanFactory 的子接口，提供了更高级的特性    |
|        |    Spring 框架的基础设施，面向 Spring 本身     |          面向使用 Spring 框架的开发者          |
| 初始化 | 默认 **延迟初始化**，首次 `getBean()` 时初始化 |          默认**全部初始化**并绑定完成          |
|        |                所需要的资源有限                |               需要更多的系统资源               |

#### 容器启动

Spring 的 `IoC` 容器的工作过程可以分为两个阶段：`容器启动阶段` 和 `Bean 实例化阶段`。

容器启动阶段主要做的工作是加载和解析配置文件，保存到对应的Bean定义中：

+ 首先通过某种途径加载 Congiguration MetaData，在大部分情况下，容器需要依赖某些工具类（BeanDefinitionReader）对加载的Congiguration MetaData进行解析和分析，并将分析后的信息组为相应的BeanDefinition。
+ 然后把这些保存了Bean定义必要信息的BeanDefinition，注册到相应的BeanDefinitionRegistry，这样容器启动就完成了。

## 4、Bean 的简介

简单来说，`Bean` 代指的就是那些被 `IoC 容器`所管理的 `对象`，比如MVC中的 `service` 。

可以通过配置元数据定义对象，进而交给 IoC 容器管理。

配置元数据的方式可以是 `XML`、`注解` 或 `Java` 配置类。

`org.springframework.beans`和 `org.springframework.context` 这两个包是 `IoC` 实现的基础。

## 5、Bean 的定义方式（声明方式）

`Bean` 的 **定义方式**（声明方式）主要有以下三种：

+ 直接编码方式：最底层，其他方式最终都是通过直接编码实现；
+ **配置文件方式**：通过 `xml`、`properties` 类型的配置文件，配置相应的依赖关系，Spring 读取配置文件，完成依赖关系的注入；
+ **注解方式**：在指定的类、方法上使用`注解`修饰，Spring 会扫描注解，完成依赖关系的注入。

将类声明为 `Bean` 的**注解**主要有`@Component`、`@Repository` 、`@Service`、`@Controller`，具体如下：

+ `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
+ `@Repository` : 对应**持久层**即 Dao 层，主要用于数据库相关操作。
+ `@Service` : 对应**服务层**，主要涉及一些复杂的逻辑，需要用到 Dao 层。
+ `@Controller` : 对应 Spring MVC **控制层**，主要接受用户请求并调用 Service 层返回数据给前端页面。

`@Component` 和 `@Bean` 的区别：

|        |                      @Component                      |              @Bean              |
| :----: | :--------------------------------------------------: | :-----------------------------: |
| 作用域 |                          类                          |              方法               |
|        | 通过类路径扫描来自动侦测以及自动装配到 Spring 容器中 | 标有该注解的方法中定义产生 bean |
|        | 可以在 `@ComponentScan` 中定义要扫描的路径，进而装配 |                                 |
|        |                                                      |          自定义行更强           |
|        |                                                      |   第三方类库 Bean 的装配常用    |

## 6、Bean 的依赖注入（属性注入）

> **依赖注入的官方描述**：
> 由于某**客户类只依赖于服务类的一个接口**，而不依赖于具体服务类，所以客户类只定义一个注入点。在程序运行过程中，客户类不直接实例化具体服务类实例，而是客户类的运行上下文环境或专门组件负责实例化服务类，然后将其注入到客户类中，保证客户类的正常运行。

**依赖注入** 指的是在 Spring 创建对象的过程中，把对象依赖的属性注入到对象中。

`Spring` 支持构造如下三种 `依赖注入` 方式：

+ **构造方法注入**（推荐）
+ **Setter 注入**
+ 工厂方法注入
  + 非静态工厂方法
  + 静态工厂方法

**构造函数** 和 **Setter 注入** 的简单对比如下：

|        构造函数注入        |        Setter 注入         |
| :------------------------: | :------------------------: |
|        没有部分注入        |         有部分注入         |
|    不会覆盖 setter 属性    |     会覆盖 Setter 属性     |
| 任意修改都会创建一个新实例 | 任意修改不会创建一个新实例 |
|     适用于设置很多属性     |     适用于设置少量属性     |

#### 构造函数注入

构造函数注入即：用构造函数的形式来实现注入。

> `Spring` 中的 `@AllArgsConstructor` 注解就是默认生成 **以所有属性为入参的构造函数**。

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

配置 `bean.xml`：

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

配置 `bean.xml` 进行属性注入：

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

#### 工厂方法注入之：静态工厂

`静态工厂`就是通过调用静态工厂的方法来获取自己需要的对象，为了让 Spring 管理所有对象，我们不能直接通过"`工程类.静态方法()`"来获取对象，而是依然通过 Spring 注入的形式获取。

#### 工厂方法注入之：实例工厂

`非静态工厂`，也叫`实例工厂`，意思是工厂方法不是静态的，所以我们需要首先 new 一个工厂实例，再调用普通的实例方法。

## 7、Bean 的自动装配

### 7.1 自动装配简介

在前期，Spring IoC 容器已经通过 XML 等知道了 Bean 的`配置信息`，然后通过 Java `反射机制`获知实现`类的结构信息`，如构造方法的结构、属性等信息。

在掌握所有 Bean 的这些信息后，Spring IoC 容器就可以按照某种规则对容器中的 Bean 进行自动装配，而无须通过显式的方式进行依赖配置。

Spring 提供的这种方式，可以按照某些规则进行 Bean 的自动装配，元素提供了一个指定自动装配类型的属性：`autowire="<自动装配类型>"`

### 7.2 自动装配类型

+ `byName`：根据名称自动匹配。
+ `byType`：根据类型自动匹配。
+ `constructor`：针对构造函数。比如 Boss类 的构造函数有一个参数为 Car 类型，容器中也有一个 Car 类型的 Bean，则该 Bean 将作为 Boss类 的构造函数的入参。如果容器中没有与构造函数入参匹配的 Bean，将抛出异常。
+ `autodetect`：根据 Bean 的自省机制决定采用 byType 还是constructor进行自动装配。如果 Bean 提供了默认的构造函数，则采用 byType，否则采用constructor。

### 7.3 自动装配的注解

Spring 内置的 `@Autowired` 以及 JDK 内置的 `@Resource` 和 `@Inject` 都可以注入（自动装配） Bean，`@Autowired` 和`@Resource`使用的比较多。

|     注解     |  来源  |         优先注入方式         |
| :----------: | :----: | :--------------------------: |
| `@Autowired` | Spring | `byType`（根据类型进行匹配） |
| `@Resource`  |  JDK   | `byName`（根据名称进行匹配） |

详细如下。

#### @Autowired

> 已经不推荐使用 属性注入 方式了。

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

## 8、Bean 的作用域

Spring 中 `Bean` 的 **作用域** 通常有下面 6 种：

|           作用域           |                             简介                             |
| :------------------------: | :----------------------------------------------------------: |
|     singleton（默认）      |        IoC 容器中只有唯一的 Bean 实例，以单例方式存在        |
|         prototype          |       每次获取 `getBean()`  都会创建一个新的 bean 实例       |
|        ↓ Web 适用 ↓        |                                                              |
|          request           | 每次 HTTP 请求产生一个新的 Bean（请求 bean），仅在当前 HTTP Request 内有效 |
|          session           | 同一个 HTTP Session 共享一个Bean（会话 bean），不同 Session 的Bean 不同 |
| application/global-session | 同一个全局 Session 共享一个Bean（应用 Bean），只用于基于 Protlet 的Web应用，仅在当前应用启动时间内有效，Spring 5 中已经不存在了 |
|         websocket          |            每一次 WebSocket 会话产生一个新的 bean            |

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

## 9、Bean 的生命周期

![image-20220802151417390](https://img.zxdmy.com/2022/202208021514870.png)

可以简单理解为 5 步：

1. **实例化**（Instantiation）
2. **设置对象属性**（Populate）
3. **初始化**（Initialization）
4. **使用**（Use）
5. **销毁**（Destruction）

具体是执行方法细节、流程如下：

1. 调用构造方法实例化
2. 设置属性
3. 【可选】Bean 内可以通过实现某些 `Aware` 接口，设置属性：
  1. 【可选】实现 `BeanNameAware` 接口的 `setBeanName()` 方法，设置 Bean 的名称
  2. 【可选】实现 `BeanFactoryAware` 接口的 `setBeanFactory()` 方法，传入 `BeanFactory` 对象的实例
  3. 【可选】实现 `BeanClassLoaderAware` 接口的 `setBeanClassLoader()` 方法，传入 `ClassLoader` 对象的实例
  4. 【可选】……
4. 【可选】实现 `BeanPostProcessor` 接口的 `postProcessBeforeInitialization()` 方法，在**初始化**之**前**实现某些功能
5. 【可选】Bean 内实现 InitializingBean 接口的 `afterPropertiesSet()` 方法，在初始化过程中，填充对象属性后，实现某些功能
6. 【可选】Bean 内自定义 init() 方法，以实现某些功能
7. 【可选】实现 `BeanPostProcessor` 接口的 `postProcessAfterInitialization()` 方法，在**初始化**之**后**实现某些功能
8. 使用 Bean
9. 【可选】Bean 内实现 `DisposableBean` 接口的 `destroy()` 方法，销毁 Bean
10. 【可选】 Bean 内自定义 destroy-method 属性，执行销毁后的操作

**代码示例**：

![image-20220908180416697](https://img.zxdmy.com/2022/202209081804976.png)

1、定义一个 `UserBean` 类，实现`DisposableBean`、`InitializingBean`、`BeanFactoryAware` 、`BeanNameAware` 4 个接口，同时自定义 `init-method` 和 `destroy-method` 方法。

`UserBean.java`：

```java
package com.zxdmy.excite.admin;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.*;

public class UserBean implements DisposableBean, InitializingBean, BeanFactoryAware, BeanNameAware {

    private String id;

    private String name;

    public String getId() {
        System.out.println("com.zxdmy.excite.admin.UserBean.getId");
        return id;
    }

    public void setId(String id) {
        System.out.println("com.zxdmy.excite.admin.UserBean.setId");
        this.id = id;
    }

    public String getName() {
        System.out.println("com.zxdmy.excite.admin.UserBean.getName");
        return name;
    }

    public void setName(String name) {
        this.name = name;
        System.out.println("com.zxdmy.excite.admin.UserBean.setName");
    }

    public UserBean() {
        System.out.println("com.zxdmy.excite.admin.UserBean.UserBean");
    }

    /**
     * BeanNameAware 接口的 setBeanName 方法
     *
     * @param s
     */
    @Override
    public void setBeanName(String s) {
        System.out.println("com.zxdmy.excite.admin.UserBean.setBeanName");
    }

    /**
     * BeanFactoryAware 接口的 setBeanFactory 方法
     *
     * @param beanFactory
     * @throws BeansException
     */
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("com.zxdmy.excite.admin.UserBean.setBeanFactory");
    }

    /**
     * InitializingBean 接口的 afterPropertiesSet 方法
     */
    @Override
    public void afterPropertiesSet() {
        System.out.println("com.zxdmy.excite.admin.UserBean.afterPropertiesSet");
    }

    /**
     * 自定义的初始化方法
     */
    public void myInitMethod() {
        System.out.println("com.zxdmy.excite.admin.UserBean.initMethod");
    }

    public void work() {
        System.out.println("com.zxdmy.excite.admin.UserBean.work");
    }

    /**
     * DisposableBean 接口的 destroy 方法
     *
     * @throws Exception
     */
    @Override
    public void destroy() throws Exception {
        System.out.println("com.zxdmy.excite.admin.UserBean.destroy");
    }

    /**
     * 自定义的销毁方法
     */
    public void myDestroyMethod() {
        System.out.println("com.zxdmy.excite.admin.UserBean.destroyMethod");
    }
}
```

2、然后实现 `BeanPostProcessor` 接口，并覆盖两个方法。

`MyBeanPostProcessor.java`：

```java
package com.zxdmy.excite.admin;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class MyBeanPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("com.zxdmy.excite.admin.MyBeanPostProcessor.postProcessBeforeInitialization");
        return bean;
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("com.zxdmy.excite.admin.MyBeanPostProcessor.postProcessAfterInitialization");
        return bean;
    }
}
```

3、配置 `xml` 文件，指定 `init-method` 和 `destroy-method` 属性。

`bean.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 指定 MyBeanPostProcessor -->
    <bean name="myBeanPostProcessor"
          class="com.zxdmy.excite.admin.MyBeanPostProcessor"
    />
    <!-- 设置 UserBean 的名称、类路径、初始化方法、销毁时的方法-->
    <bean name="userBean"
          class="com.zxdmy.excite.admin.UserBean"
          init-method="myInitMethod"
          destroy-method="myDestroyMethod"
    >
        <property name="id" value="123456"/>
        <property name="name" value="张三"/>
    </bean>
</beans>
```

4、测试执行：

```java
package com.zxdmy.excite.admin;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestMain {

    public static void main(String[] args) {
        System.out.println("========");
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
        UserBean userBean = (UserBean) context.getBean("userBean");
        userBean.work();
        context.destroy();
        System.out.println("========");
    }
}
```

5、执行结果：

```txt
========
18:25:10.212 [main] DEBUG org.springframework.context.support.ClassPathXmlApplicationContext - Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@1888ff2c
18:25:10.380 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loaded 2 bean definitions from class path resource [bean.xml]
18:25:10.430 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'myBeanPostProcessor'
18:25:10.459 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'userBean'
com.zxdmy.excite.admin.UserBean.UserBean
com.zxdmy.excite.admin.UserBean.setId
com.zxdmy.excite.admin.UserBean.setName
com.zxdmy.excite.admin.UserBean.setBeanName
com.zxdmy.excite.admin.UserBean.setBeanFactory
com.zxdmy.excite.admin.MyBeanPostProcessor.postProcessBeforeInitialization
com.zxdmy.excite.admin.UserBean.afterPropertiesSet
com.zxdmy.excite.admin.UserBean.myInitMethod
com.zxdmy.excite.admin.MyBeanPostProcessor.postProcessAfterInitialization
com.zxdmy.excite.admin.UserBean.work
18:25:10.548 [main] DEBUG org.springframework.context.support.ClassPathXmlApplicationContext - Closing org.springframework.context.support.ClassPathXmlApplicationContext@1888ff2c, started on Thu Sep 08 18:25:10 CST 2022
com.zxdmy.excite.admin.UserBean.destroy
com.zxdmy.excite.admin.UserBean.myDestroyMethod
========
```

## 10、Bean 的各种问题

### 10.1 单例 Bean 的线程安全问题

> Spring中的单例 Bean **不是线程安全** 的。

单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候是存在资源竞争的。

常见的有两种解决办法：

1. 在 Bean 中尽量避免定义可变的成员变量。
2. 在类中定义一个 `ThreadLocal` 成员变量，将需要的可变成员变量保存在 `ThreadLocal` 中（推荐的一种方式）。

不过，**大部分 Bean 实际都是无状态**（没有实例变量）的（比如 Dao、Service），这种情况下， Bean 是线程安全的。

### 10.2 Bean 的循环依赖问题

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

#### 只用二级缓存不可行

主要是为了 **生成代理对象**。

如果是没有代理的情况下，使用二级缓存解决循环依赖也是OK的。

但是如果存在代理，三级没有问题，二级就不行了。

因为三级缓存中放的是⽣成具体对象的匿名内部类，获取Object的时候，它可以生成代理对象，也可以返回普通对象。使⽤三级缓存主要是为了保证不管什么时候使用的都是⼀个对象。

假设只有⼆级缓存的情况，往⼆级缓存中放的显示⼀个普通的Bean对象，Bean初始化过程中，通过 BeanPostProcessor 去⽣成代理对象之后，覆盖掉⼆级缓存中的普通Bean对象，那么可能就导致取到的Bean对象不一致了。

![image-20220908215315727](https://img.zxdmy.com/2022/202209082153006.png)

### 10.3 Bean 的同名问题

- 同一个配置文件内同名的Bean，以最上面定义的为准
- 不同配置文件中存在同名Bean，后解析的配置文件会覆盖先解析的配置文件
- 同文件中`@ComponentScan`和`@Bean`出现同名Bean。同文件下`@Bean`的会**生效**，@ComponentScan扫描进来不会生效。通过@ComponentScan扫描进来的优先级是最低的，原因就是它扫描进来的Bean定义是最先被注册的。

## 11、手动实现 IoC

Spring `IoC` 的本质是一个大工厂，即采用 `工厂模式` 的方式，来实现 `Bean` 的创建和管理，采用 `反射` 机制来实现 `Bean` 的实例化。

简单来说，Spring IoC主要有以下功能和组成：

+ Bean 的定义
+ IoC 容器
+ Bean 对象的管理

![image-20220908111534521](https://img.zxdmy.com/2022/202209081115107.png)

接下来就基于此，实现一个简单的IoC容器。

![image-20220908111836378](https://img.zxdmy.com/2022/202209081118754.png)

#### Bean 的定义

在 Spring 中，`Bean` 都是通过`注解`或者 `XML` 配置文件进行定义。

这里为了简化操作，我们使用一个 `key-value` 形式的 `properties 文件`定义 `Bean`。

其中 `key` 就是 Bean 的名称，`value` 就是 Bean 的类。

`beans.properties`：

```properties
userDao:com.bean.diy.dao.UserDao
bookDao:com.bean.diy.dao.BookDao
```

接下来使用一个 `实体类` 来表示properties文件中定义的 `Bean`：

`BeanDefinition.java`：

```java
package com.bean.diy.bean;

/**
 * Bean 的定义类，properties 配置文件中 Bean 定义对应的实体
 *
 * @author 拾年之璐
 * @since 2022/9/8 10:04
 */
public class BeanDefinition {

    private String beanName;

    private Class beanClass;

    public String getBeanName() {
        return beanName;
    }

    public void setBeanName(String beanName) {
        this.beanName = beanName;
    }

    public Class getBeanClass() {
        return beanClass;
    }

    public void setBeanClass(Class beanClass) {
        this.beanClass = beanClass;
    }
}
```

#### 配置文件加载 Bean 对象

对于 `properties` 配置文件中定义的 `Bean`，采用一个 `资源加载器` 进行加载。

`ResourceLoader.java`：

```java
package com.bean.diy.bean;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

/**
 * 资源加载器，用来完成properties配置文件中配置的加载
 *
 * @author 拾年之璐
 * @since 2022/9/8 10:05
 */
public class ResourceLoader {

    /**
     * 加载默认的properties配置文件中的Bean
     *
     * @return 配置文件的Map，KEY是Bean的名称，Value 是 Bean 的实体类
     */
    public static Map<String, BeanDefinition> getResource() {
        // 存放 Bean 的 KEY 和 定义类
        Map<String, BeanDefinition> beanDefinitionMap = new HashMap<>(16);
        // 从文件中逐行读取
        Properties properties = new Properties();
        try {
            // 读取配置文件
            BufferedReader bufferedReader = new BufferedReader(new FileReader("src/com/bean/diy/beans.properties"));
            // 转换成 properties 对象
            properties.load(bufferedReader);
            // 遍历加载
            for (String key : properties.stringPropertyNames()) {
                // 从 properties 文件中，根据 key 读取类名称（路径.类名）
                String className = properties.getProperty(key);
                // Bean 实例
                BeanDefinition beanDefinition = new BeanDefinition();
                // 设置名称
                beanDefinition.setBeanName(key);
                // 通过路径读取类，通过反射加载 Bean
                Class<?> clazz = Class.forName(className);
                beanDefinition.setBeanClass(clazz);
                // 放 Bean 至 Bean 的缓存 Map
                beanDefinitionMap.put(key, beanDefinition);
                System.out.println("com.bean.diy.bean.ResourceLoader.getResource : Bean[" + className + "]读取完成");
            }
            bufferedReader.close();
        } catch (IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
        return beanDefinitionMap;
    }
}
```

#### Bean 对象的注册

Spring 中的 Bean，用到了三级缓存来处理（比如循环依赖），其中一级缓存存放的是实例化完成的，可以直接使用的 Bean 对象。

Spring 中的一级缓存本质是一个ConCurrentHashMap，定义如下：

```java
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
```

这里我们只模拟一级缓存，也采用 Map 来存放实例化的 Bean：

+ key 表示 Bean的名称
+ value 表示 Bean 的实例化对象

对于一级缓存，需要有相应的注册器进行写入和读取 Bean 对象。

`BeanRegister.java`：

```java
package com.bean.diy.bean;

import java.util.HashMap;
import java.util.Map;

/**
 * 对象注册器
 *
 * @author 拾年之璐
 * @since 2022/9/8 10:09
 */
public class BeanRegister {

    /**
     * 存放单例 Bean 的一级缓存容器
     */
    private Map<String, Object> singletonMap = new HashMap<>(32);

    /**
     * 获取单例 Bean
     *
     * @param beanName bean名称
     * @return Bean 的对象
     */
    public Object getSingletonBean(String beanName) {
        return singletonMap.get(beanName);
    }

    /**
     * 注册单例 Bean
     *
     * @param beanName Bean 的名称
     * @param bean Bean 的对象
     */
    public void registerSingletonBean(String beanName, Object bean) {
        // 容器中没有此 Bean 时，才缓存至 Map 中
        if (!singletonMap.containsKey(beanName)) {
            singletonMap.put(beanName, bean);
        }
    }
}
```

#### Bean 对象的工厂

对象工厂，是 IoC 最核心的一个类，在初始化时，需要完成两项主要的工作：

+ 创建 Bean 注册器，初始化缓存容器
+ 并完成 properties 配置文件中的 Bean 实体资源的加载

同时，我们需要从对象工厂中获取 Bean，其基本流程是：

+ 从单例缓存（一级缓存）中读取
+ 单例缓存不存在，则注册该 Bean，并写入B单例缓存，然后返回 Bean

`BeanFactory.java`：

```java
package com.bean.diy.bean;

import java.util.Map;

/**
 * 对象工厂，最核心的类，在初始化时，创建 Bean 注册器，以及进行资源的加载。
 *
 * @author 拾年之璐
 * @since 2022/9/8 10:11
 */
public class BeanFactory {

    /**
     * 配置文件中定义的 Bean
     */
    private Map<String, BeanDefinition> beanDefinitionMap;

    /**
     * Bean 注册器，用来缓存和获取已经实例化的Bean对象
     */
    private BeanRegister beanRegister;

    /**
     * Bean 工厂构造函数
     */
    public BeanFactory() {
        // 创建 bean 注册器
        beanRegister = new BeanRegister();
        // 加载资源
        this.beanDefinitionMap = ResourceLoader.getResource();
        System.out.println("com.bean.diy.bean.BeanFactory.BeanFactory : Bean 工厂构造完成");
    }

    /**
     * 获取 Bean，先从单例缓存中取，如果没有取到，就创建并注册一个 Bean
     *
     * @param beanName Bean 的名称
     * @return Bean 实例
     */
    public Object getBean(String beanName) {
        // 从 Bean 缓存中取
        Object bean = beanRegister.getSingletonBean(beanName);
        if (bean != null) {
            System.out.println("com.bean.diy.bean.BeanFactory.getBean : 从缓存中获取 Bean[" + beanName + "]");
            return bean;
        }
        // 配置文件中已经对 Bean 进行定义，则根据名称进行创建、缓存
        if (beanDefinitionMap.containsKey(beanName)) {
            // 根据 Bean 定义，创建 Bean
            return createBean(beanDefinitionMap.get(beanName));
        } else {
            System.out.println("com.bean.diy.bean.BeanFactory.getBean : Bean[" + beanName + "]未在properties文件中定义");
            return null;
        }
    }

    /**
     * 创建 Bean
     *
     * @param beanDefinition Bean 的定义
     * @return 创建完成后的 Bean
     */
    private Object createBean(BeanDefinition beanDefinition) {
        System.out.println("com.bean.diy.bean.BeanFactory.createBean : 创建 Bean[" + beanDefinition.getBeanName() + "]");
        try {
            Object bean = beanDefinition.getBeanClass().newInstance();
            // 缓存 Bean
            beanRegister.registerSingletonBean(beanDefinition.getBeanName(), bean);
            return bean;
        } catch (InstantiationException | IllegalAccessException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

#### 测试

先通过两个 Dao 文件，模拟Spring MVC中的 Dao层。

`BookDao.java`：

```java
package com.bean.diy.dao;

import java.util.ArrayList;
import java.util.List;

/**
 * @author 拾年之璐
 * @since 2022/9/8 9:59
 */
public class BookDao {

    public List<String> getBookNames() {
        List<String> list = new ArrayList<>();
        list.add("深入理解计算机系统");
        list.add("Spring 终极讲义");
        return list;
    }
}
```

`UserDao.java`:

```java
package com.bean.diy.dao;

import java.util.HashMap;
import java.util.Map;

/**
 * @author 拾年之璐
 * @since 2022/9/8 9:59
 */
public class UserDao {

    public String getNameById(Integer id) {
        Map<Integer, String> map = new HashMap<>(16);
        map.put(1, "张三");
        map.put(2, "李四");
        return map.getOrDefault(id, "None");
    }
}
```

然后通过一个 Main 方法测试：

`Main.java`：

```java
package com.bean.diy;

import com.bean.diy.bean.BeanFactory;
import com.bean.diy.dao.BookDao;
import com.bean.diy.dao.UserDao;

import java.util.List;

public class Main {
    static BeanFactory beanFactory = new BeanFactory();

    public static void main(String[] args) {
        test1();
        test2();
        test3();
        test4();
    }

    static void test1() {
        // 创建 Bean 工厂，同时完成资源加载，创建注册单例 Bean 注册器的操作
        UserDao userDao = (UserDao) beanFactory.getBean("userDao");
        System.out.println(userDao.getNameById(1));
    }

    static void test2() {
        UserDao userDao2 = (UserDao) beanFactory.getBean("userDao");
        System.out.println(userDao2.getNameById(3));
    }

    static void test3() {
        BookDao bookDao = (BookDao) beanFactory.getBean("bookDao");
        List<String> bookNameList = bookDao.getBookNames();
        for (String name : bookNameList) {
            System.out.println(name);
        }
    }

    static void test4() {
        UserDao userDao2 = (UserDao) beanFactory.getBean("carDao");
        System.out.println(userDao2.getNameById(3));
    }
}
```

运行结果：

```txt
略

```



