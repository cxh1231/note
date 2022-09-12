## 1、@SpringBootApplication 注解

`Spring Boot` 项目 `XXXApplication.java` 启动类，有一个 `@SpringBootApplication` 注解。

进入 `@SpringBootApplication` 注解，可以看到如下实现：

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

然后继续进入上面的 `@SpringBootConfiguration` 注解，可以看到如下实现：

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

通过上面的两段代码可以看出，`@SpringBootApplication` 是一个**复合注解**，由 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 三大注解组成。

**根据 SpringBoot 官网，这三个注解的作用：**

+ `@Configuration`：允许在上下文中注册额外的 `bean` 或导入其他**配置类**；
+ `@ComponentScan`：扫描被 `@Component`（`@Service`，`@Controller`）注解的 `bean`，注解默认会扫描该类所在的包下所有的类；
+ `@EnableAutoConfiguration`：启用 `Spring Boot` **自动配置/装配** 机制的核心注解。

其中

## 2、@EnableAutoConfiguration 自动装配

![image-20220912202044295](https://img.zxdmy.com/2022/202209122020191.png)

`SpringBoot` 开启 **自动配置** 的注解是 `@EnableAutoConfiguration`。

进入 `@EnableAutoConfiguration` 注解的接口类中，可以看到如下代码：

![image-20220912202305667](2、Spring Boot 自动装配.assets/image-20220912202305667.png)

`@EnableAutoConfiguration` 注解包含 **两个关键的注解**，其主要功能如下：

+ `@AutoConfigurationPackage`：将`main`同级的包下的所有组件注册到容器中；
+ `@Import(AutoConfigurationImportSelector.class)`：加载自动装配类 `xxxAutoconfiguration`。

`AutoConfigurationImportSelector` 类的实现如下图所示：

![image-20220912202655635](https://img.zxdmy.com/2022/202209122026192.png)

`AutoConfigurationImportSelector` 实现了`DeferredImportSelector` 接口，而 `DeferredImportSelector` 接口实现了 `ImportSelector` 接口。

`ImportSelector` 接口如下图所示。

![image-20220912202829357](https://img.zxdmy.com/2022/202209122028481.png)

`ImportSelector` 接口的作用就是**收集需要导入的配置类**，配合 `@Import()` 就可以将相应的类导入到**Spring容器**中。

**获取注入类**的方法是 `selectImports()`，它实际调用的是 `getAutoConfigurationEntry` ，**这个方法是获取自动装配类的关键**，主要流程可以分为这么几步：

1. **获取注解的属性**，用于后面的排除；
2. **获取所有需要自动装配的配置类的路径**：这一步是最关键的，从 `METAINF/spring.factories` 获取自动配置类的路径；
3. **去重与排除**：去掉重复的配置类和需要排除的重复类
4. 把需要自动加载的配置类的路径存储起来

![image-20220912203715136](https://img.zxdmy.com/2022/202209122037245.png)

然后通过 `selectImports` 方法返回所有需要被应用配置的类：

![image-20220912205853929](https://img.zxdmy.com/2022/202209122058948.png)

最后 `ConfigurationClassPostProcessor.postProcessBeanFactory()` 将识别这些配置类中定义的 `Bean` ，并将它们 **注册到容器** 中。

**总结图**：

![image-20220912211250071](https://img.zxdmy.com/2022/202209122112035.png)



---



这里有几个存放配置信息文件再详细介绍一下：

+ `spring-autoconfigure-metadata.properties`：属性文件，包含自动配置元数据AutoConfigurationMetadata
+ `spring.factories`：自动配置候选类
+ `spring-autoconfigure-metadata.properties`：候选类的过滤规则

具体如下：

**`spring.factories` **：

存储了 Spring Boot 所有默认支持的待自动装配候选类。

通过 `AutoConfigurationImportSelector` 类的导入包列表，可以看到导入了一个类 `SpringFactoriesLoader`，这是 **Spring 组件工厂**，其中有一个参数就是  `METAINF/spring.factories` 文件的路径：

![image-20220912205007669](https://img.zxdmy.com/2022/202209122050432.png)

`spring.factories` 文件包含**所有需要配置的类路径**：

![image-20220912205142889](https://img.zxdmy.com/2022/202209122051588.png)

**`spring-autoconfigure-metadata.properties`**：

该文件存储的是 **待自动装配候选类** **过滤的计算规则**，框架会**根据里面的规则逐一对候选类进行计算看是否需要被自动装配进容器**，**并不是全部加载**。

内容格式为：`自动配置的类全名.条件Condition = 值`

如下图所示：

![image-20220912211648078](https://img.zxdmy.com/2022/202209122116206.png)

## 3、自动装配扩展开发

如果想依赖 Spring Boot 自动装配扩展点扩展，那么需要做如下工作：

1、新建 `spring.factories` 文件放在自己工程下，结构如下：

![image-20220912211920427](https://img.zxdmy.com/2022/202209122119474.png)

写入如下代码：

```factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
x.xx.xxAutoConfiguration,\
x.xxx.xxxAutoConfiguration
```

2、在 `xxAutoConfiguration` 类里配合 `@ConditionalOnProperty`、`@ConditionalOnClass` 等注入我们能所需的类即可。

比如：

![image-20220912212448738](https://img.zxdmy.com/2022/202209122124955.png)

再如：

```java
/**
 * Author: bamboo
 * Time: 2018/11/25/025
 * Describe: 自动配置类
 * 根据条件判断是否要自动配置，创建Bean
 */
@Configuration
@EnableConfigurationProperties(BambooServerProperties.class)//声明开启属性注入BambooServerProperties
@ConditionalOnClass(BambooServer.class)//判断BambooServer这个类在类路径中是否存在,只有存在时才符合条件
@ConditionalOnProperty(prefix = "bamboo",value = "enabled",matchIfMissing = true)
public class BmbooServiceAutoConfiguration {

    @Autowired
    private BambooServerProperties mistraServiceProperties;

    @Bean(name = "bambooServer")
    @ConditionalOnMissingBean(BambooServer.class)//当容器中没有这个Bean时(BambooServer)就自动配置这个Bean，Bean的参数来自于BambooServerProperties
    public BambooServer mistraService(){
        BambooServer mistraService = new BambooServer();
        mistraService.setName(mistraServiceProperties.getName());
        return mistraService;
    }
}
```

## 4、自动装配扩展开发示例

1、**创建**一个名为 `demo-spring-boot-starter` 的**项目**，并引入 Spring Boot 相关依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configurationprocessor</artifactId>
    <optional>true</optional>
</dependency>
```

2、编写**配置文件**，并自定义属性配置的前缀：

```java
@ConfigurationProperties(prefix = "demo")
public class DemoProperties {
    private String name;
    //省略getter、setter
}
```

3、创建**自动配置类** `DemoPropertiesConfigure`：

```java
@Configuration
@EnableConfigurationProperties(DemoProperties.class)
public class DemoPropertiesConfigure {
    
}
```

4、在 `/resources/META-INF/spring.factories` 文件中添加**自动配置类路径**：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.xxx.demo.starter.configure.DemoPropertiesConfigure
```

5、测试：

任意创建一个工程，引入前文自定义的 starter 依赖：

```xml
<dependency>
    <groupId>cn.xxx</groupId>
    <artifactId>demo-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

在配置文件里添加配置：

```properties
demo.name=张三
```

测试类：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloTest {
    
    @Autowired
    DemoProperties demoProperties;
    
    @Test
    public void demo(){
        System.out.println("你好，"+ demoProperties.getName());
    }
}
```

运行结果：

![image-20220912214750025](https://img.zxdmy.com/2022/202209122147140.png)

至此，本简单 Demo 已经实现了 **自动装配** 能力。

## 5、总结

Spring Boot 的自动装配机制是 Spring Boot 的核心特性之一，它可以让我们在开发中更加专注于业务逻辑的开发，而不用关心各种配置的问题。

Spring Boot 的自动装配机制是通过 `@EnableAutoConfiguration` 注解实现的，它会扫描 `META-INF/spring.factories` 文件中的 `org.springframework.boot.autoconfigure.EnableAutoConfiguration` 配置项，然后根据 `spring-autoconfigure-metadata.properties` 文件中的配置规则，对候选类进行过滤，最后将过滤后的类加载到 Spring 容器中。
