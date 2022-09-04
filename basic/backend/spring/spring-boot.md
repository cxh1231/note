## 1、使用 Spring Boot 的原因

### 1.1 Spring 的不足

Spring 的组件代码是轻量级的，但它的配置却是重量级的（需要大量 XML 配置） 。

### 1.2 Spring Boot 的优点

Spring 旨在简化 J2EE 企业应用程序开发。

Spring Boot  旨在简化 Spring 开发（减少配置文件，开箱即用！）。

1. 开发基于 Spring 的应用程序很容易。
2. Spring Boot 项目所需的开发或工程时间明显减少，通常会提高整体生产力。
3. Spring Boot **不需要编写大量样板代码、XML 配置和注释**。
4. Spring 引导应用程序可以很容易地与 Spring 生态系统集成，如 Spring JDBC、Spring ORM、Spring Data、Spring Security 等。
5. Spring Boot 遵循“固执己见的默认配置”，即**约定大于配置，减少开发工作**（默认配置可以修改）。
6. Spring Boot 应用程序**提供嵌入式 HTTP 服务器**，如 Tomcat 和 Jetty，可以轻松地开发和测试 web 应用程序。（这点很赞！普通运行 Java 程序的方式就能运行基于 Spring Boot web 项目，省事很多）
7. Spring Boot 提供命令行接口(CLI)工具，用于开发和测试 Spring Boot 应用程序，如 Java 或 Groovy。
8. Spring Boot 提供了多种插件，可以使用内置工具(如 Maven 和 Gradle)开发和测试 Spring Boot 应用程序

## 2、@SpringBootApplication与自动装配

### 3.1 核心注解：@SpringBootApplication

在 IDEA 中 `XXXApplication.java` 启动类中，进入 `@SpringBootApplication` 注解，可以看到其详细实现：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    ……
}
```

然后继续进入上面的 `@SpringBootConfiguration` 注解，可以看到其实现如下：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
@Indexed
public @interface SpringBootConfiguration {
  @AliasFor(
          annotation = Configuration.class
  )
  boolean proxyBeanMethods() default true;
}
```

由上面的实现看出，`@SpringBootApplication`是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。

**根据 SpringBoot 官网，这三个注解的作用：**

+ `@EnableAutoConfiguration`：启用 SpringBoot 的**自动配置**机制；
+ `@ComponentScan`：扫描被 `@Component`（`@Service`，`@Controller`）注解的 `bean`，注解默认会扫描该类所在的包下所有的类；
+ `@Configuration`：允许在上下文中注册额外的 `bean` 或导入其他配置类。

### 3.2 Spring Boot 自动配置

已知 `@SpringBootApplication` 看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。

**`@EnableAutoConfiguration` 是启动自动配置的关键**。

通过 IDEA 进入 `@EnableAutoConfiguration` 注解的接口类中，可以看到如下代码：

![image-20220808160057353](https://img.zxdmy.com/2022/202208081718252.png)

即 `@EnableAutoConfiguration` 注解通过 Spring 提供的 `@Import` 注解导入了`AutoConfigurationImportSelector.class` 类

> `@Import` 注解可以导入配置类或者 Bean 到当前类中。

接着进入`AutoConfigurationImportSelector.class` 类中的 `getCandidateConfigurations()`方法，如下图所示

![image-20220808160305371](https://img.zxdmy.com/2022/202208081718979.png)

该方法会将所有**自动配置类的信息以 List 的形式返回**。这些配置信息会被 `Spring` 容器作 `bean` 来管理。

有了**自动配置信息**，接着通过**条件装配注解** `@Conditional` ，用于限制 `@Bean` 注解在什么时候才生效。

##  3、Spring Boot Starters 的工作原理

`Spring Boot Starters` 是一系列 **依赖关系的集合**，因为它的存在，项目的依赖之间的关系对我们来说变的更加简单了。

比如进行 Web 项目开发，只需添加一个 `spring-boot-starter-web` 依赖，则该依赖的子依赖中包含了我们开发 REST 服务需要的所有依赖。

**Spring Boot 在启动的时候会干这几件事情**：

- Spring Boot 在启动时会去依赖的 `Starter` 包中寻找 `resources/META-INF/spring.factories` 文件，然后根据文件中配置的 `Jar` 包去扫描项目所依赖的 `Jar` 包。
- 根据 `spring.factories` 配置加载 `AutoConfigure` 类
- 根据 `@Conditional` 注解的条件，进行自动配置并将 `Bean` 注入 `Spring Context`

总结一下，其实就是 Spring Boot 在启动的时候，**按照约定去读取 Spring Boot Starter 的配置信息，再根据配置信息对资源进行初始化，并注入到 Spring 容器中**。这样 Spring Boot 启动完毕后，就已经准备好了一切资源，使用过程中直接注入对应 Bean 资源即可。

## 4、 RESTful Web 服务常用注解

#### Bean 相关

+ `@Autowired` ：自动导入对象到类中，被注入进的类同样要被 Spring 容器管理。

+ `@Component` ：通用的注解，可标注任意类为 Spring 组件。如果一个 Bean 不知道属于哪个层，可以使用@Component 注解标注。

+ `@Repository`、`@Mapping` ：对应**持久层**，即 Dao 层，主要用于数据库相关操作。

  > `@Repository` 标注的接口，需要在启动类使用 `@MapperScan` 扫描该接口。

+ `@Service` ：对应**服务层**，主要涉及一些复杂的逻辑，需要用到 Dao 层。

+ `@Controller` ：对应 Spring MVC **控制层**，主要用于接受用户请求并调用 Service 层返回数据给前端页面。

+ `@RestController` ： `@RestController` 注解是 `@Controller` 和 `@ResponseBody` 的合集，表示这是个控制器 `bean`，并且是将函数的返回值，直接填入 HTTP 响应体中，是 `REST` 风格的控制器。

#### HTTP 请求相关

+ `@GetMapping` : GET 请求
+ `@PostMapping` : POST 请求
+ `@PutMapping` : PUT 请求
+ `@DeleteMapping` : DELETE 请求。

> 注：`@PostMapping` 多次相同POST会产生多份相同的数据，不具有幂等性；`@PutMapping` 多次相同的PUT请求和第一次相同，具有幂等性。

#### 前后端传值

+ `@RequestParam` ：获取查询参数
+ `@Pathvairable` ：获取路径参数
+ `@RequestBody` ：用于读取 Request 请求（可能是 POST,PUT,DELETE,GET 请求）的 body 部分并且 Content-Type 为 application/json 格式的数据，接收到数据之后会自动将数据绑定到 Java 对象上去。系统会使用HttpMessageConverter或者自定义的HttpMessageConverter将请求的 body 中的 json 字符串转换为 java 对象。

![img](https://img.zxdmy.com/2022/202208081718707.jpg)