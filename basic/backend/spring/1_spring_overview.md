## 1、Spring 简介

> 问：Spring 框架的优点/好处？

`Spring` 是一款**开源的轻量级 Java 开发框架**，旨在提高开发人员的开发效率以及系统的可维护性。一般说 **Spring 框架**指的都是 `Spring Framework`，它是**很多模块的集合，**使用这些模块可以很方便地协助我们进行开发。

**Spring 是一个轻量级、非入侵式的控制反转 (IoC) 和面向切面（AOP）的框架**。

**Spring 最核心的思想就是不重新造轮子，开箱即用！**

![image-20220907161248435](https://img.zxdmy.com/2022/202209071612721.png)

## 2、Spring 的特性与优点

#### IoC 与 DI 的支持

Spring 的核心就是一个大的工厂容器，可以维护所有对象的创建和依赖关系，Spring 工厂用于生成 Bean，并且管理 Bean 的生命周期，实现 **高内聚低耦合** 的设计理念。

#### AOP 编程的支持

Spring 提供了 **面向切面编程**，可以方便的实现对程序进行权限拦截、运行监控等切面功能。

#### 声明式事务的支持

不需要通过编码，只需要**通过配置即可完成对事物的管理**，进而实现事务的提交、回滚等。

#### 快捷测试的支持

Spring 对 Junit 提供支持，可以通过 **注解** 快捷地测试 Spring 程序。

#### 快速集成功能

Spring 方便快速集成各种优秀的框架，提供了对各种优秀框架的支持，比如MyBatis等。

#### 复杂API模板封装

Spring 对 JavaEE 开发中非常难用的一些 API（JDBC、JavaMail、远程调用等）都提供了模板化的封装，这些封装 API 的提供使得应用难度大大降低。

## 3、Spring 的主要组成模块

`Spring` 框架由大约 20 种模块组成，如下图所示：

![image-20220907162014287](https://img.zxdmy.com/2022/202209071620346.png)

其中重要、**必选**的模块是 `Core Container`，其他的模块都是可选的。

`Spring` 各个模块的依赖关系如下：

![image-20220802101850532](https://img.zxdmy.com/2022/202208021018722.png)

> UML：父类 ← 子类

### 3.1 核心模块（IoC 模块）

`Core Container` 是 Spring 框架的**核心模块**（基础模块），主要提供 IoC 依赖注入功能的支持。

> **Spring** 其他所有的功能基本都需要依赖于该模块。

该模块下主要有的组成如下：

|       模块        | 功能                                                         |
| :---------------: | ------------------------------------------------------------ |
|   `Spring-Core`   | **Spring 框架基本的核心工具类**                              |
|  `Spring-Beans`   | **提供对 bean 的创建、配置和管理等功能的支持**               |
|  Spring-Context   | 提供对国际化、事件传播、资源加载等功能的支持                 |
| Spring-Expression | 提供对表达式语言（Spring Expression Language） SpEL 的支持，只依赖于 core 模块，不依赖于其他模块，可以单独使用 |

### 3.2 AOP 模块

即**面向切面编程**，它提供了与 AOP 联盟兼容的编程实现。

|       模块        | 功能                                  |
| :---------------: | ------------------------------------- |
| `Spring-Aspects`  | **该模块为与 AspectJ 的集成提供支持** |
|   `Spring-Aop`    | **提供了面向切面的编程实现**          |
| spring-instrument | 提供了为 JVM 添加代理（agent）的功能  |

具体来讲，它为 Tomcat 提供了一个织入代理，能够为 Tomcat 传递类文 件，就像这些文件是被类加载器加载的一样。没有理解也没关系，这个模块的使用场景非常有限。

### 3.3 数据库连接与整合模块

提供对 JDBC 抽象层，简化了 JDBC 编码，同时，编码更具有健壮性。

|      模块       | 功能                                                         |
| :-------------: | ------------------------------------------------------------ |
| **Spring JDBC** | **提供了对数据库访问的抽象 JDBC**。Java 程序只需要和 JDBC API 交互，屏蔽数据库的影响 |
|    Spring-tx    | 提供对事务的支持                                             |
|   Spring-orm    | 提供对 Hibernate、JPA 、iBatis 等 ORM 框架的支持             |
|   Spring-oxm    | 提供一个抽象层支撑 OXM(Object-to-XML-Mapping)，例如：JAXB、Castor、XMLBeans 等 |
|   Spring-jms    | Java 消息服务。自 Spring Framework 4.1 以后，它还提供了对 spring-messaging 模块的继承 |

### 3.4 Web 模块

针对 Web 应用中 MVC 思想的实现。

|       模块       | 功能                                                        |
| :--------------: | ----------------------------------------------------------- |
|   `spring-web`   | **对 Web 功能的实现提供一些最基础的支持**                   |
| `spring-webmvc`  | **提供对 Spring MVC 的实现**                                |
| spring-websocket | 提供了对 WebSocket 的支持，可以让客户端和服务端进行双向通信 |
|  spring-webflux  | 提供对 WebFlux 的支持                                       |

> WebFlux 是 Spring Framework 5.0 中引入的新的响应式框架。与 Spring MVC 不同，它不需要 Servlet API，是完全异步。

### 3.5 Test 模块

Spring 的测试模块对 JUnit（单元测试框架）、TestNG（类似 JUnit）、Mockito（主要用来 Mock 对象）、PowerMock（解决 Mockito 的问题比如无法模拟 final, static， private 方法）等等常用的测试框架支持的都比较好。

## 4、Spring 的常用注解

Spring 的常用注解，可以根据模块来划分：

#### Web 模块的注解

![image-20220907164254169](https://img.zxdmy.com/2022/202209071642199.png)

|      注解       | 简介                                                         |
| :-------------: | ------------------------------------------------------------ |
|  @RequestBody   | 允许 request 请求的参数在 Request 体中，而不是在直接连接在地址后面 |
|  @ResponseBody  | 支持将返回值放在 Response 内，而不是一个页面，通常用户返回 JSON 数据 |
|  @PathVariable  | 用于接收路径参数，通常作为Restful的接口实现方法              |
| @RequestMapping | 用于映射Web请求，包括访问路径和参数。如果是 Restful 风格接口，还可以根据请求类型使用不同的注解：<br />　　@GetMapping <br/>　　@PostMapping <br/>　　@PutMapping <br/>　　@DeleteMapping |
|   @Controller   | 组合注解（组合了 @Component 注解），应用在 MVC 的 C 层（控制层） |
| @RestController | 组合注解（@Controller和@ResponseBody 的组合）注解在 **类** 上，意味着该Controller的所有方法都默认加上了 @ResponseBody。 |

#### 容器相关的注解

与 Bean 的生命、注入、作用域等相关的注解：

|    注解     | 简介                                                         |
| :---------: | ------------------------------------------------------------ |
| @Component  | 表示该**类**是一个“组件”，成为 Spring 管理的Bean。<br />当使用基于注解的配置和类路径扫描时，这些类被视为自动检测的候选对象。<br />@Component 是一个**元注解**。 |
|    @Bean    | 作用在**方法**上，声明当前方法的返回值为一个Bean。<br />返回的Bean对应的类中可以定义`init()`方法和`destroy()`方法，然后在`@Bean(initMethod="init", destroyMethod="destroy")`定义，在构造之后执行`init`，在销毁之前执行 `destroy`。 |
| @Autowired  | 由 Spring 的 **依赖注入** 工具（BeanPostProcessor、BeanFactoryPostProcessor）自动注入，默认的注入方式为 `byType`。 |
| *@Resource* | 由 JDK 提供的依赖注入注解，默认注入方式为 `byName`。         |
| @Qualifier  | 指定依赖注入时的实现**类的名称**。                           |
|   @Value    | 将常量、配置文件中的值、其它 Bean 的属性值注入到变量中，配合 SpEL 使用。 |
|   @Scope    | 配置 Bean 实例的作用域，作用在方法上，与 @Bean 配合使用。    |

> **元注解的作用就是负责注解其他注解**。

基于 `@Component` 的注解：

|      注解      | 简介                                                         |
| :------------: | ------------------------------------------------------------ |
|    @Service    | 组合注解（组合了@Component注解），应用在 Service层（业务逻辑层） |
|  @Repository   | 组合注解（组合了@Component注解），应用在 Dao层（数据访问层） |
| @Configuration | 组合注解（组合了@Component注解），声明当前类是一个配置类（相当于一个Spring配置的xml文件） |

#### AOP 模块的注解

|   注解    | 简介                                                         |
| :-------: | ------------------------------------------------------------ |
|  @Aspect  | 作用于**类**上，声明一个切面，使用@After、@Before、@Around 定义建言（advice），可直接将拦截规则（切点）作为参数 |
| @PointCut | 作用于**方法**上，声明切点，在java配置类中使用@EnableAspectJAutoProxy注解开启 Spring 对 AspectJ代理的支持 |
|  @After   | 作用于**方法**上，在方法执行之后执行                         |
|  @Before  | 作用于**方法**上，在方法执行之前执行                         |
|  @Around  | 作用于**方法**上，在方法执行之前与之后执行                   |

#### 事务注解

`@Transactional`：在要开启事务的 **方法** 上使用@Transactional注解，即可 **声明式开启事务**。

## 5、Spring、Spring MVC、Spring Boot 的区别

如前文，**Spring 包含了多种功能模块**。

> 其中最重要的是 `Spring-Core` 模块（主要提供 IoC 依赖注入功能的支持） ，**Spring 中的其他模块的功能实现基本都需要依赖于该模块**。

**Spring MVC 是 Spring 中的一个很重要的模块，主要赋予 Spring 快速构建 MVC 架构的 Web 程序的能力**。

> MVC 是模型(Model)、视图(View)、控制器(Controller)的简写，其核心思想是通过将业务逻辑、数据、显示分离来组织代码。

**Spring Boot 简化了 Spring MVC 的各种配置，做到开箱即用。**

## 6、Spring 的设计模式

|   设计模式   | 具体应用                                                     |
| :----------: | ------------------------------------------------------------ |
|   工厂模式   | Spring 容器本质是一个大工厂，使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象 |
|   代理模式   | Spring AOP 功能是通过代理模式来实现的，分为**动态代理**和**静态代理** |
|   单例模式   | Spring 中的 Bean 默认都是单例的，有利于容器对Bean的管理      |
| 模板方法模式 | Spring 中 `JdbcTemplate`、`RestTemplate` 等以 `Template` 结尾的对数据库、网络等进行操作的类，都使用到了模板方法模式 |
|  观察者模式  | Spring 事件驱动模型（ApplicationEvent等）就是观察者模式很经典的一个应用 |
|  适配器模式  | Spring AOP 的增强或通知（Advice）使用了适配器模式，Spring MVC 中也是用到了适配器模式适配`Controller`。 |
|  包装器模式  | 当需要动态连接多个数据库时，根据访问需要去访问不同的数据库，动态切换数据源就用到了该模式 |
|   策略模式   | Spring中有一个Resource接口，它的不同实现类，会根据不同的策略去访问资源。 |

