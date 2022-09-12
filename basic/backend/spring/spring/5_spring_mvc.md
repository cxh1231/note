## 1、Spring MVC 核心组件

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

## 2、工作原理

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