## 1、Spring 简介

> 问：Spring 框架的优点/好处？

`Spring` 是一款**开源的轻量级 Java 开发框架**，旨在提高开发人员的开发效率以及系统的可维护性。一般说 **Spring 框架**指的都是 `Spring Framework`，它是**很多模块的集合，**使用这些模块可以很方便地协助我们进行开发。

**Spring 最核心的思想就是不重新造轮子，开箱即用！**

## 2、Spring 的重要模块

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