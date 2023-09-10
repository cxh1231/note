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

## 3、服务注册发布

如下是最简服务提供者的注册方法示例：

```java
public static void main(String[] args) throws Exception {
    // 初始化ProviderConfig，服务接口是EchoService，具体服务提供者是EchoServiceDefaultImpl
    ProviderConfig<EchoService> providerConfig = new ProviderConfig<EchoService>(EchoService.class, new EchoServiceDefaultImpl());
    // 调用URL
    String url = "com.dianping.pigeon.demo.EchoService";
    providerConfig.setUrl(url);
    // 注册服务
    ServiceFactory.addService(providerConfig);
}
```

每个接口服务都以 `ProviderConfig` 类作为元数据，映射一个服务调用接口。

其中定义了相关属性，如提供服务的抽象接口类型`serviceInterface`、访问服务的`url`等等，如下图所示：

![image-20230905230239897](https://img.zxdmy.com/2022/202309052302173.png)

`ServerConfig` 则对应一个服务器进程，其具体服务器，可为基于`netty`的`RPC`服务器，也可为基于`jetty`的`HTTP`服务器。

![image-20230905230937942](https://img.zxdmy.com/2022/202309052313829.png)

### 3.1 服务提供方静态初始化流程

示例中，`ServiceFactory.addService(providerConfig);` 完成服务注册，在调用 `addService()` 方法之前，会触发 `ServiceFactory` 的静态初始化，如下图所示：

![image-20230905231305256](https://img.zxdmy.com/2022/202309052313412.png)

其具体初始化工作，主要包括如下三部分：

1. 初始化`serviceProxy`，提供作为调用方的信息注册，和服务代理获取；
2. 初始化`publishPolicy`，针对特定的`providerConfig`进行服务注册；
3. 调用`ProviderBootStrap.init()`初始化调用`ProviderBootStrap`。

`ProviderBootStrap` 是服务提供者的核心启动类，内部如下：

![image-20230905231710504](https://img.zxdmy.com/2022/202309052317200.png)

主要包括以下初始化流程：

1. `ProviderProcessHandlerFactory.init()` 初始化所有相关拦截器
2. `SerializerFactory.init()` 初始化所有序列化方式
3. `ClassUtils.loadClasses(“com.dianping.pigeon”)` 预加载所有Pigeon相关的类
4. 注册关闭钩子`ShutdownHookListener`，内部调用`ServiceFactory.unpublishAllServices()`进程完成相关清理工作
5. `RegistryManager.getInstance()` 初始化注册管理器
6. 加载服务器，默认包括 `NettyServer` 和 `JettyHttpServer`，当前会注册HTTP服务器
    1. 启动`http`服务器，默认注册到4080
    2. 设置`consoleServer`，初始化注册配置

### 3.2 服务提供方发布服务

调用 `ServiceFactory#addService` 方法，会进一步调用 `DefaultPublishPolicy#doAddService`，最后调用的是抽象父类 `AbstractPublishPolicy#doAddService` 方法。

`AbstractPublishPolicy#doAddService` 方法具体源码如下：

![image-20230905232419334](https://img.zxdmy.com/2022/202309052324616.png)

#### 检查服务名

如果自定义了访问url，和默认服务名（接口全限定名）不一致，会检查服务是否为存量服务，如果不是，会判断配置是否允许自定义服务名，如果允许，则使用自定义服务名，否则会抛出异常或强制转换为类路径服务名。

![image-20230910205820086](https://img.zxdmy.com/2022/202309102058984.png)

#### 解析服务方法

+ 判断是否配置了版本， 如果配置了，生成带版本的`urlWithVersion`，更新`key=urlWithVersion`的服务，同时如果大于`key=url`的对应服务版本，会用新版本覆盖默认url版本;

+ 如果服务实现了`InitializingService`接口，调用实现的`initialize`方法

+ 调用`ServiceMethodFactory.init(url)`方法，用来初始化调用`ServiceMethodFactory`的`ServiceMethodCache`：
    + 遍历`ServicePublisher`的所有服务提供类，建立`ServiceMethodCache`，存储该类下所有满足要求的方法和方法id的映射关系
    + 首先会忽略掉Object和Class的所有方法
    + 过滤方法后，判断是否需要压缩，根据`url+"#"+方法名` 的方式进行`hash`

![image-20230910211432063](https://img.zxdmy.com/2022/202309102114507.png)

#### 启动netty RPC服务器

根据protocol+port判断当前服务器列表中是否存在相关的服务器实例，如果不存在先创建一个，然后再添加服务，再创建新的服务器实例后，会先预启动内部的请求处理器核心线程；

![image-20230910210907471](https://img.zxdmy.com/2022/202309102109410.png)

#### 发布服务到注册中心

具体流程如下：

1. 先遍历缓存的服务，根据url判断服务是否已存在，记录在`existingService`变量，如果不存在服务，直接结束，否则继续进行下面流程；
2. 读取配置判断是否允许自动发布；
3. 调用`publishServiceToRegistry`方法将服务发布到注册中心；
4. 如果注册数量大于0，判断配置创建线程进行健康检查，健康检查的原理是通过`serviceAddress`建立映射关系，定期将当前时间戳更新到注册中心（`zookeeper`）上；
5. 读取配置判断是否需要调用`notifyServicePublished`方法通知所有的`serviceChangeListener`服务已发布；
6. 读取配置判断是否允许自动注册，如果允许，启动`ServiceOnlineTask`线程，会定时调用`ServiceFactory.online()`方法，更新服务权重，以提供向外调用；
7. 设置`providerConfig`发布成功标志。

![image-20230910210411945](https://img.zxdmy.com/2022/202309102104930.png)

## 4、服务发现

最基本的**服务调用者**代码如下所示：

```java
InvokerConfig<EchoService> invokerConfig = new InvokerConfig<>(EchoService.class);
EchoService echoService = ServiceFactory.getService( invokerConfig);
```

其大致流程：

1. 创建一个`InvokerConfig`，至少传入一个接口类全限定名
2. 调用`ServiceFactory.getService(invokerConfig)`获取具体的代理对象
3. 以接口引用，直接通过代理对象调用远程方法

每个`InvokerConfig`对应一个调用服务，其内部属性定义了协议、服务访问url、调用超时时间、泳道等等配置，如下图所示。

![image-20230910212341871](https://img.zxdmy.com/2022/202309102123340.png)

#### 静态初始化

`ServiceFactory`静态初始化流程类似于服务提供方，大致流程如下：

在首次访问ServiceFactory，会触发ServiceProxy和PublishPolicy的加载

1. ServiceProxy提供作为调用方的信息注册，和服务代理获取
2. PublishPolicy针对特定的providerConfig进行服务注册

![image-20230910212829426](https://img.zxdmy.com/2022/202309102128164.png)

随后会调用ProviderBootStrap.init()初始化对外提供服务的基础依赖

1. ProviderProcessHandlerFactory.init()初始化所有相关拦截器
2. SerializerFactory.init()初始化所有序列化方式
3. ClassUtils.loadClasses(“com.dianping.pigeon”)预加载所有pigeon相关类
4. 注册关闭钩子ShutdownHookListener，内部调用ServiceFactory.unpublishAllServices()进程完成相关清理工作
5. RegistryManager.getInstance() 初始化注册管理器
6. 加载服务器，默认包括NettyServer和JettyHttpServer，当前会注册HTTP服务器
7. 启动http服务器，默认注册到4080
8. 设置consoleServer，初始化注册配置

![image-20230910213144918](https://img.zxdmy.com/2022/202309102131045.png)

#### 初始化调用

在具体调用`ServiceFactory.getService`时，实际会调用`serviceProxy.getProxy(invokerConfig)`，进一步调用 `AbstractServiceProxy#getProxy` 方法，这个方法的实现逻辑大致如下：

1. 检查interface,url,protocol等参数合法性，如果url没有设置，默认使用接口全限定名，如果protocol不为空，且等于default,则修改成@DEFAULT@，同时更新url加上这个协议前缀
2. 双重加锁检查invokeConfig对应的服务是否以存在，不存在先尝试启动调用方启动类InvokerBootStrap
3. 先根据配置的序列方式获取相应的序列化类，再根据invokerConfig创建动态代理：ServiceInvocationProxy
4. 如果配置中的负载均衡配置存在，注册服务到负载均衡管理器中
5. 注册区域策略服务
6. ClientManager.getInstance().registerClients(invokerConfig)注册客户端到注册中心（默认为zk）
7. 缓存服务

如下图所示：

![image-20230910214239090](https://img.zxdmy.com/2022/202309102142092.png)


## 5、服务调用

前文中的最基本的**服务调用者**代码如下所示：

```java
InvokerConfig<EchoService> invokerConfig = new InvokerConfig<>(EchoService.class);
EchoService echoService = ServiceFactory.getService( invokerConfig);
```

在通过`ServiceFactory#getService`方法拿到的接口引用，

![image-20230910215425875](https://img.zxdmy.com/2022/202309102154390.png)

实际是一个`Proxy`代理类，其内部持有一个`ServiceInvocationProxy`实例；

![image-20230910215519586](https://img.zxdmy.com/2022/202309102155847.png)

而`ServiceInvocationProxy`实例持有两个属性，分别是`InvokerConfig`和`InvokerProcessHandlerFactory`的一个匿名内部类`ServiceInvocationHandler`实例对象，

![image-20230910215702574](https://img.zxdmy.com/2022/202309102157330.png)

而`ServiceInvocationHandler`内部持有一个**拦截器链**，来完成具体的链式调用。

当调用服务 `EchoService` 的方法时，实际执行的是 `ServiceInvocationProxy#invoke` 方法，如下图所示。

![image-20230910215805250](https://img.zxdmy.com/2022/202309102158262.png)

其中，执行`handler.handle()`会执行`InvokerProcessHandlerFactory`中创建的匿名内部类的方法，

![image-20230910220347403](https://img.zxdmy.com/2022/202309102203302.png)

这时，开始了诸多拦截器链的链式调用：

![image-20230910220613941](https://img.zxdmy.com/2022/202309102206014.png)

各个拦截器的功能大致如下：

+ RemoteCallMonitorInvokeFilter：添加监控事务打点的相关信息
+ TraceFilter：添加链路调用相关监控信息
+ FaultInjectionFilter：故障注入拦截器，更改配置信息，模拟服务故障时应用的表现
+ DegradationFilter：降级处理拦截器
+ ClusterInvokeFilter：集群策略，根据配置返回策略配置结果
+ GatewayInvokeFilter：网关流量拦截器，用于限流
+ ContextPrepareInvokeFilter：初始化远程调用请求和准备远程调用链路传递的上下文自定义参数
+ SecurityFilter：鉴权拦截器
+ RemoteCallInvokeFilter：实际触发远程调用请求的拦截器，也是整个调用流程的最后一个拦截器链。

## 6、 服务响应

服务提供方需要接收来自调用方的请求，这一步在`NettyServerHandler`内完成。`NettyServerHandler`继承了`Netty`的`SimpleChannelUpstreamHandler`接口：

![image-20230910221835940](https://img.zxdmy.com/2022/202309102218975.png)

`NettyServerHandler` 在`NettyServerPipelineFactory`中创建，作为netty处理连接pipeline的最后一个处理器，如下图所示：

![image-20230910221918894](https://img.zxdmy.com/2022/202309102219559.png)

请求从进入`Netty`服务器，到`NettyServerHandler`之前，已经完成了请求数据解压缩、校验、解密等流程。

最终调用了`NettyServerHandler#messageReceived`方法，来处理消息接收请求：

![image-20230910222052377](https://img.zxdmy.com/2022/202309102220270.png)

上图中，`this.server.processRequest(request, invocationContext);` 进行请求处理，最终是子类 RequestThreadPoolProcessor 的doProcessRequest 执行：

![image-20230910222836762](https://img.zxdmy.com/2022/202309102228855.png)

上图中，`invocationHandler.handle(providerContext);` 中的 拦截器链 ，主要有：

+ WriteResponseProcessFilter：处理完方法调用后，执行pigeon服务提供方自定义拦截器的postInvoke逻辑；
+ ContextTransferProcessFilter：调用方->服务方上下文数据的传递
+ ExceptionProcessFilter：捕捉后续拦截器调用和实际方法调用抛出的异常，对异常进一步封装后返回。
+ SecurityFilter：统一鉴权，如IP、APP名称、Token等鉴权
+ GateWayProcessFilter：用于流量计数和根据限流配置，对应用app进行限流处理
+ BusinessProcessFilter：会通过反射调用我们实际的业务方法，在调用之前，需要根据服务名、服务版本、方法名、方法参数等信息找到最佳匹配方法。最后执行。

## 7、客户端集群访问策略

在pigeon中，目前实现了4种集群策略：

+ failfast：调用服务的一个节点失败后抛出异常返回，可以同时配置重试timeoutRetry和retries属性
+ failover：调用服务的一个节点失败后会尝试调用另外的一个节点，可以同时配置重试timeoutRetry和retries属性
+ failsafe：调用服务的一个节点失败后不会抛出异常，返回null，后续版本会考虑按配置默认值返回
+ forking：同时调用服务的所有可用节点，返回调用最快的节点结果数据

![image-20230910223418449](https://img.zxdmy.com/2022/202309102234080.png)

## 8、区域路由策略





## 9、负载均衡策略

在pigeon默认实现中，有4种负载均衡策略，包括：

+ RandomLoadBalance：基于不同的权重，随机从多个连接实例种选择一个，权重越大，被选中的可能性也越大
+ AutoawareLoadBalance：实时获取当前客户端对每个服务端连接的请求数，获取请求数最小的进行请求
+ RoundRobinLoadBalance：基于不同的权重，从大到小轮询所有的连接实例进行请求
+ WeightedAutoawareLoadBalance：基于不同的权重，在实时获取每个连接实例的请求量基础上，除以权重计算一个相对的负载量，取最小的连接实例完成请求。

![image-20230910223741646](https://img.zxdmy.com/2022/202309102237730.png)

## 10、熔断降级

pigeon提供三种降级开关，来分别支持不同的降级策略：

+ **强制降级开关**：在远程服务大量超时或其他不可用情况时，紧急时候进行设置，开启后，调用端会根据上述降级策略直接返回默认值或抛出降级异常，当远程服务恢复后，建议关闭此开关。对应配置pigeon.invoker.degrade.force=true，默认为false
+ **失败降级开关**：失败降级开关便于客户端在服务端出现非业务异常(比如网络失败，超时，无可用节点等)时进行降级容错，而在出现业务异常(比如登录用户名密码错误)时不需要降级。对应配置pigeon.invoker.degrade.failure=true，默认为false
+ **自动降级开关**：自动降级开关是在调用端设置，开启自动降级后，调用端如果调用某个服务出现连续的超时或不可用，当一段时间内（10秒内）失败率超过一定阀值（默认1%）会触发自动降级，调用端会根据上述降级策略直接返回默认值或抛出降级异常；当服务端恢复后，调用端会自动解除降级模式，再次发起请求到远程服务。对应配置pigeon.invoker.degrade.auto=true，默认为false

若同时开启了多个开关，会根据下面优先级使用相应降级策略：**强制降级 > 自动降级 > 失败降级**，其中自动降级包含失败降级策略。








参考资料：

+ RPC框架pigeon源码分析 - 洪墨水：https://www.cnblogs.com/hongmoshui/p/11187692.html
+ Pigeon源码阅读_jeanheo：https://blog.csdn.net/qwe6112071/category_9284723.html
