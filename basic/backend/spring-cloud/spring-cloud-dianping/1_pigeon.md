## 1、Pigeon 概述

Pigeon 是一个分布式服务通信框架（RPC），是美团点评最基础的底层框架之一。

开源链接：https://github.com/dianping/pigeon

官方的用户手册明确其具有如下特点：

- 除了支持spring schema等配置方式，也支持代码annotation方式发布服务、引用远程服务，并提供原生api接口的用法。
- 支持http协议，方便非java应用调用pigeon的服务。
- 序列化方式除了hessian，还支持thrift等。
- 提供了服务器单机控制台pigeon-console，包含单机服务测试工具。
- 创新的客户端路由策略，提供服务预热功能，解决线上流量大的service重启时大量超时的问题。
- 记录每个请求的对象大小、返回对象大小等监控信息。
- 服务端可对方法设置单独的线程池进行服务隔离，可配置客户端应用的最大并发数进行限流。

## 2、Bean 注入

### 2.1 XML注入

Pigeon 支持基于 Spring Schema 方式进行配置，其自定义了一个 XML 命名空间解析器，用来解析以`pigeon:xxx` 标志的 XML 配置，具体实例配置如下：

provider.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans …………>
    <bean id="defaultEchoServiceImpl" class="com.dianping.pigeon.demo.EchoServiceDefaultImpl"/>
    <!-- 定义独立的线程池 -->
    <pigeon:pool id="poolEcho" corePoolSize="5" maxPoolSize="20" workQueueSize="50"/>
    <!-- 以服务方视角，定义一个 Pigeon 服务，供客户端调用-->
    <pigeon:service
            url="com.dianping.pigeon.demo.EchoService"
            interface="com.dianping.pigeon.demo.EchoService"
            useSharedPool="false"
            ref="defaultEchoServiceImpl">
        <pigeon:method name="echo" pool="poolEcho"/>
    </pigeon:service>
</beans>
```

之后，该XML配置信息将被实现自 `BeanDefinitionParserLoader#loadBeanDefinitionParsers` 的方法进行解析。

对于 `provider` 来说，其解析器有如下三种类型：

![image-20230904220444438](https://img.zxdmy.com/2022/202309042204319.png)

invoker.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans …………>
    <bean id="echoServiceCallback" class="com.dianping.pigeon.demo.EchoServiceCallback"/>
    <!-- 以调用方视角，定义一个 Pigeon 服务引用，通过RPC调用远程服务方服务-->
    <pigeon:reference id="echoServiceWithCallback"
                      interface="com.dianping.pigeon.demo.EchoService"
                      url="com.dianping.pigeon.demo.EchoService"
                      timeout="1000" protocol="default" serialize="hessian" callType="callback"
                      callback="echoServiceCallback"/>
</beans>
```

之后，该XML配置信息将被实现自 `BeanDefinitionParser#parse` 的 `ReferenceBeanDefinitionParser#parse` 方法进行解析，注册到Spring容器`BeanDefinition`注册中心，实现Bean的实例化和依赖注入。

### 2.2 注解注入

Pigeon 可以配置 `<pigeon:annotation package="com.dianping.pigeon.demo.annotation"/>` 实现注解扫描配置的方式，其中package属性是具体扫描的包，默认扫描com.dianping下所有包。

在Pigeon中主要使用了 `@Reference` 和 `@Service` 注解，用来标志服务 **调用方接口** 和 **服务提供方** 实现类，其内部定义的配置分别和`InvokerConfig`和`ProviderConfig`相对应，具体定义如下：

![image-20230904221615854](https://img.zxdmy.com/2022/202309042216719.png)

使用时，

+ 服务提供方：只需要实现特定接口，然后在实现类上标注 `@Service` 注解
+ 服务调用方：只需要在field或method上加上 `@Reference` 注解

![image-20230904221911361](https://img.zxdmy.com/2022/202309042219637.png)

在Spring加载`BeanDefinition`时，识别到 `XML` 配置了pigeon命名空间，会根据 `META-INF/spring.handlers` 文件下的配置找到`CommonNamespaceHandler`：

![image-20230904222237381](https://img.zxdmy.com/2022/202309042222873.png)

然后调用 `CommonNamespaceHandler#init` 方法，加载类路径 `META-INF.services`目录下的`com.dianping.pigeon.config.spring.BeanDefinitionParserLoader` 文件中定义的所有 `loader`：

![image-20230904222410494](https://img.zxdmy.com/2022/202309042224272.png)

其中，`AnnotationBeanDefinitionParserLoader` 将获取 XML 配置的属性值，这里主要获取`package`属性，用来定义扫描的包空间，同时注册当前`BeanDefinition`到Spring容器`BeanDefinition`注册中心，在后续初始化Spring容器Bean的时候，会初始化当前`BeanDefinition`对应的实例类`AnnotationBean`。

![image-20230904222726270](https://img.zxdmy.com/2022/202309042227395.png)

#### 包扫描

在Spring初始化容器时，会调用`AbstractApplicationContext#refresh`方法完成整个初始化逻辑，其中有两步：

![image-20230904224353545](https://img.zxdmy.com/2022/202309042243729.png)

其中：

+ `postProcessBeanFactory` 会调用`beanFactory.getBeanNamesForType`拿到所有`BeanDefinition`中的`BeanClass`属性实现了`BeanFactoryPostProcessor`接口的`BeanName`，进而会对其初始化，调用其中的`postProcessBeanFactory`方法，如下图所示。
+ `invokeBeanFactoryPostProcessors` 会注册Spring容器中所有`BeanClass`实现了`BeanPostProcessorts`接口的`BeanName`，同样会进一步初始化，完成注册，在所有容器Bean初始化的前后前置和后置调用，以拓展Bean功能。

![image-20230904224011901](https://img.zxdmy.com/2022/202309042240457.png)

上图中，`AnnotationBean#postProcessBeanFactory` 方法分为三步：

+ 加载 `ClassPathBeanDefinitionScanner` 并初始化 `scanner`
+ 调用 `scanner.addIncludeFilter` 添加对 `Service` 注解扫描支持
+ 调用`scanner.scan`进行扫描，并将满足过滤要求（即有@Service注解）的类封装成 `BeanDefinition`，并注册到 `BeanDefinitionRegistry` 中。

通过以上三个流程，完成了特定包空间下pigeon注解类的扫描，并注册到`BeanDefinitionRegistry`中。

#### 依赖注入

对于如`url`、`version`等属性，在初始化Bean结束后的**依赖注入过程**里完成：

![image-20230904224849139](https://img.zxdmy.com/2022/202309042248328.png)

`@Reference` 注解是在Bean通过构造方法实例化后，但在进行初始化(依赖注入)之前进行 **依赖注入**：

![image-20230904224943256](https://img.zxdmy.com/2022/202309042249362.png)



参考资料：

+ RPC框架pigeon源码分析 - 洪墨水：https://www.cnblogs.com/hongmoshui/p/11187692.html
+ Pigeon源码阅读_jeanheo：https://blog.csdn.net/qwe6112071/category_9284723.html
