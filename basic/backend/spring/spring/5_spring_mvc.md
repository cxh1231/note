## 1、Spring MVC 核心组件

**MVC 是模型（Model）、视图（View）、控制器（Controller）的简写**，其核心思想是通过将业务逻辑、数据、显示分离来组织代码。

**Spring MVC 的核心组件有：**

|           组件           | 说明                                                         |
| :----------------------: | ------------------------------------------------------------ |
|   `DispatcherServlet`    | **前置控制器**，Spring MVC 的**核心组件**，是请求的入口，负责协调各个组件工作 |
|        `Handler`         | **请求处理器**，完成具体的**业务逻辑**，相当于 `Servlet` 或 `Action` |
|     `HandlerMapping`     | 请求的**处理器匹配器**（映射器），`DispatcherServlet` 接收到请求之后，通过 `HandlerMapping` 将不同的请求映射到不同的 `Handler` |
|   `HandlerInterceptor`   | 请求的**处理器拦截器**，是接口，实现该接口将进行拦截处理     |
|  HandlerExecutionChain   | **处理器执行链**，包括两部分内容：`Handler` 和 `HandlerInterceptor`（系统有默认的 `HandlerInterceptor`，如果需要额外设置拦截，可以添加拦截器） |
|     `HandlerAdapter`     | **处理器的适配器**。`Handler` 执行业务方法之前，需要进行一系列的操作，包括表单数据的验证、数据类型的转换、将表单数据封装到 `JavaBean` 等，这些操作都是由 `HandlerApater` 来完成，开发者只需将注意力集中业务逻辑的处理上，`DispatcherServlet` 通过 `HandlerAdapter` 执行不同的 `Handler`。 |
|     `HandlerAdapter`     | **处理器的适配器**。因为处理器 `handler` 的类型是 Object 类型，需要有一个调用者来实现 `handler` 是怎么被执行。Spring 中的处理器的实现多变，比如用户处理器可以实现 Controller 接口、HttpRequestHandler 接口，也可以用 `@RequestMapping` 注解将方法作为一个处理器等，这就导致 Spring MVC 无法直接执行这个处理器。所以这里需要一个处理器适配器，由它去执行处理器 |
| HandlerExceptionResolver | 处理器异常解析器，将处理器（ `handler` ）执行时发生的异常，解析( 转换 )成对应的 ModelAndView 结果 |
|    MultipartResolver     | 内容类型( `Content-Type` )为 `multipart/*` 的请求的解析器，例如解析处理文件上传的请求，便于获取参数信息以及上传的文件 |
|      `ModelAndView`      | 装载了模型数据和视图信息，作为 `Handler` 的处理结果，返回给 `DispatcherServlet`。 |
|       ViewResolver       | **视图解析器**，`DispatcheServlet` 通过它将逻辑视图解析为物理视图，最终将渲染结果响应给客户端。 |

## 2、Spring MVC 工作流程

**Spring MVC 原理如下图所示：**

![image-20220912171617491](https://img.zxdmy.com/2022/202209121716364.png)

1. 客户端（浏览器）向服务端**发起请求**，请求首先到达 **前置控制器**（`DispatcherServlet`，也叫中央控制器）；
2. `DispatcherServlet` 收到请求，根据请求信息调用 **处理器映射器**（`HandlerMapping`），进而得知该请求对应的 `Handler`（即 `Controller` 控制器）（但此时并未调用 `Controller`，只是得知）；
3. `DispatcherServlet` 接着调用 **处理器适配器**（`HandlerAdapter`），告知 `HandlerAdapter` 需要执行的 `Controller`；
4. `HandlerAdapter` 执行 `Controller`，得到处理结果 `ModelAndView`（数据和视图），并层层返回给`DispatcherServlet`；
5. `DispatcherServlet` 将 `ModelAndView` 交给 **视图解析器**（`ViewReslover`）进行解析，然后返回真正的视图；
6. `DispatcherServlet` 将 `Model` 模型数据填充到 `View` 视图中；
7. `DispatcherServlet` 将结果响应给客户端。

Spring MVC 虽然整体流程复杂，但是真正需要开发人员进行处理的只有 `Handler`（`Controller`） 、`View` 、`Model`。

## 3、Spring MVC Restful 接口工作流程

`Restful`接口的响应格式是 `json`，这就用到了一个常用注解：`@ResponseBody`。

加入这个注解后，整体的流程上和使用`ModelAndView`大体上相同，但是细节上有一些不同：

![image-20220912172743122](https://img.zxdmy.com/2022/202209121727022.png)

1. 客户端向服务端发送一次请求，这个请求会先到**前端控制器 **（`DispatcherServlet`）；
2. `DispatcherServlet` 接收到请求后会调用`HandlerMapping`处理器映射器。由此得知，该请求该由哪个`Controller`来处理；
3. `DispatcherServlet`调用`HandlerAdapter`**处理器适配器**，告诉处理器适配器应该要去执行哪个Controller；
4. `Controller`被封装成了`ServletInvocableHandlerMethod`，`HandlerAdapter`**处理器适配器**去执行`invokeAndHandle`方法，完成对`Controller`的请求处理；
5. `HandlerAdapter`执行完对`Controller`的请求，会调用`HandlerMethodReturnValueHandler`去处理返回值，主要的过程：
   1. 调用`RequestResponseBodyMethodProcessor`，创建`ServletServerHttpResponse`（Spring对原生 `ServerHttpResponse` 的封装）实例；
   2. 使用`HttpMessageConverter`的`write`方法，将返回值写入`ServletServerHttpResponse`的`OutputStream`输出流中；
   3. 在写入的过程中，会使用`JsonGenerator`（默认使用`Jackson`框架）对返回值进行`Json`序列化；
   4. 执行完请求后，返回的 `ModealAndView` 为null，`ServletServerHttpResponse`里也已经写入了响应，所以不用关心`View`的处理

## 4、Spring MVC 拦截器工作流程

**拦截器** 会对处理器进行拦截，这样通过拦截器就可以**增强处理器的功能**。

`Spring MVC` 中，所有的**拦截器**都需要实现 `HandlerInterceptor` 接口，该接口包含如下三个方法：

+ `preHandle()`：**在控制器方法前执行**，其返回值为 `true` 表示**继续**往下执行，`false` 表示中断后续所有操作；
+ `postHandle()`：**在控制器方法调用之后，且解析视图之前执行**，对请求域中的模型和视图做出进一步的修改；
+ `afterCompletion()`：**在请求完成之后、视图渲染结束之后执行**，可以实现一些资源清理、记录日志信息等工作。

![image-20220912190528394](https://img.zxdmy.com/2022/202209121905649.png)

![image-20220912190550952](https://img.zxdmy.com/2022/202209121905067.png)

1. **开发拦截器**：实现`handlerInterceptor`接口，从三个方法中选择合适的方法，实现拦截时要执行的具体业务逻辑。
2. **注册拦截器**：定义配置类，并让它实现`WebMvcConfigurer`接口，在接口的`addInterceptors`方法中，注册拦截器，并定义该拦截器匹配哪些请求路径。

