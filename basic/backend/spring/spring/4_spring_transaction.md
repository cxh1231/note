## 1、Spring 事务简介

Spring 事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，Spring 是无法提供事务功能的。

Spring 只提供统一事务管理接口，具体实现都是由各数据库自己实现，数据库事务的提交和回滚是通过数据库自己的事务机制实现。

Spring 事务具有如下优点：

- 提供了跨不同事务 api（如JTA、JDBC、Hibernate、JPA和JDO）的一致编程模型。
- 为编程事务管理提供了比JTA等许多复杂事务API更简单的API。
- 支持声明式事务管理。
- 很好地集成了 Spring 的各种数据访问抽象。

## 2、Spring 事务的管理方式（实现方式）

Spring 支持 **编程式事务** 管理和 **声明式事务** 管理方式。

#### 编程式事务

**编程式事务** 是指在代码中，通过 `TransactionTemplate`或者 `TransactionManager` 手动管理事务，需要显示执行事务。

不推荐使用编程式事务。

编程式事务 实际应用中很少使用，但对于理解 Spring 事务管理原理有帮助。

#### 声明式事务

**声明式事务** 管理建立在 `AOP` 之上。

即通过 `AOP` 的功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，在目标方法开始之前启动一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

**声明式事务** 的具体使用方法有两种：

+ 在 `XML` 配置文件中配置
+ 直接基于 **注解**（推荐使用）（基于`@Transactional` 的全注解方式使用最多）

**声明式事务** 的优点是不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明或通过 `@Transactional` 注解的方式，便可以将事务规则应用到业务逻辑中，减少业务代码的污染。

唯一不足地方是，**最细粒度只能作用到方法级别**，无法做到像编程式事务那样可以作用到代码块级别。

## 3、Spring 事务的隔离级别

> 这个问题和 MySQL 数据库的基本一致。

Spring 的接口 `TransactionDefinition` 中定义了表示隔离级别的常量，其主要还是对应数据库的事务隔离级别。

|       事务       | 简介                                               |
| :--------------: | -------------------------------------------------- |
|     default      | 使用后端数据库默认的隔离级别（MySQL 默认可重复的） |
| read_uncommitted | 读未提交                                           |
|  read_committed  | 读已提交                                           |
| repeatable_read  | 可重复读                                           |
|   serializable   | 串行化                                             |

![image-20220912162201176](https://img.zxdmy.com/2022/202209121622586.png)

## 4、Spring 事务的传播机制（传播规则）

`Spring` 事务的**传播机制**，指的是**多个事务同时存在时（一般指多个事务方法互相调用），Spring 如何处理这些事务的行为**。

**当事务方法被另一个事务方法调用时，必须指定事务应该如何传播**。

**事务传播机制** 是使用简单的 `ThreadLocal` 实现的。

如果调用的方法是在 **新线程调用** 的，事务传播实际上是会 **失效** 的。

`Spring` 支持以下 `7` 种事务传播机制：

|          事务名称           |             如果当前存在事务             | 如果当前没有事务 |
| :-------------------------: | :--------------------------------------: | :--------------: |
|    **REQUIRED**（默认）     |                加入该事务                | 创建一个新的事务 |
|      **REQUIRES_NEW**       |   创建新事务（互相独立），挂起当前事务   | 创建一个新的事务 |
|          SUPPORTS           |                加入该事务                | 以非事务方式运行 |
|        NOT_SUPPORTED        |               挂起当前事务               | 以非事务方式运行 |
| **MANDATORY** (*mandatory*) |               加入当前事务               |     抛出异常     |
|    **NESTED** (*nested*)    | 创建一个事务作为当前事务的嵌套事务来运行 | 创建一个新的事务 |
|            NEVER            |                 抛出异常                 | 以非事务方式运行 |

`Spring` 默认的事务传播行为是 `PROPAFATION_REQUIRED`，它适合绝大多数情况。

比如有多个 `ServiceX#methodX()` 都工作在事务环境下（均被 Spring 事务增强），且程序中存在调用链`Service1#method1() -> Service2#method2() -> Service3#method3()`，那么这 3 个服务类的三个方法通过 Spring 的事务传播机制都工作在同一个事务中。

具体使用示例：

```java
@Override
@Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)
public int saveRole(SysRole role, Integer[] menusIds) {
    // TODO 
}
```

## 4、Spring 声明式事务的实现原理

> Spring 声明式事务 通过AOP/动态代理实现。

#### 在Bean初始化阶段创建代理对象

Spring 容器在初始化每个单例bean的时候，会遍历容器中的所有 `BeanPostProcessor` 实现类，并执行其
`postProcessAfterInitialization` 方法；

在执行 `AbstractAutoProxyCreator` 类的 `postProcessAfterInitialization` 方法时，会遍历容器中所有的切面，查找与当前实例化bean匹配的切面；

这时，会获取到 **事务属性切面**，查找 `@Transactional` 注解及其属性值，然后根据得到的切面创建一个代理对象；

默认是使用JDK动态代理创建代理，如果目标类是**接口**，则使用`JDK`动态代理，否则使用`Cglib`。

#### 在执行目标方法时进行事务增强操作

当通过 **代理对象** 调用 `Bean` 方法的时候，会触发对应的 `AOP` 增强拦截器；

**声明式事务**是一种**环绕增强**，对应接口为`MethodInterceptor` ，事务增强对该接口的实现为`TransactionInterceptor` ，类图如下：

![image-20220912161755196](https://img.zxdmy.com/2022/202209121617233.png)

**事务拦截器**`TransactionInterceptor` 在`invoke` 方法中，通过调用父类`TransactionAspectSupport` 的 `invokeWithinTransaction` 方法进行事务处理，包括**开启事务**、**事务提交**、**异常回滚**。

## 5、Spring 声明式事务的失效场景

主要在以下四个场景会**失效**：

+ @Transactional 应用在非 public 修饰的方法上
+ @Transactional 注解的属性 propagation 设置错误
+ @Transactional 注解的属性 rollbackFor 设置错误
+ 同一个类中方法调用，导致@Transactional失效

#### @Transactional 应用在非 public 修饰的方法上

如果 `@Transactional` 注解应用在非 `public` 修饰的方法上，`Transactional` 将会失效。

这时因为在 `Spring AOP` 代理时，`TransactionInterceptor` （事务拦截器）在目标方法执行**前后**进行拦截，`DynamicAdvisedInterceptor`（`CglibAopProxy` 的内部类）的`intercept`方法或 `JdkDynamicAopProxy`的`invoke`方法会间接调用`AbstractFallbackTransactionAttributeSource`的 `computeTransactionAttribute` 方法，获取 `Transactional` 注解的事务配置信息。

![image-20220912164900972](https://img.zxdmy.com/2022/202209121649272.png)

此方法会检查**目标方法的修饰符**是否为 `public`，不是 public 则**不会**获取 `@Transactional` 的**属性配置**信息。

#### @Transactional 注解属性 propagation 设置错误

在事务的隔离级别中，有几项是会让事务失效（不以事务方式执行），具体如下：

+ `TransactionDefinition.PROPAGATION_SUPPORTS`：如果当前存在事务，则加入该事务；如果当前没有事务，则以**非事务的方式继续运行**。
+ `TransactionDefinition.PROPAGATION_NOT_SUPPORTED`：以**非事务方式运行**，如果当前存在事务，则把当前事务挂起。
+ `TransactionDefinition.PROPAGATION_NEVER`：以**非事务方式运行**，如果当前存在事务，则抛出异常。

#### @Transactional 注解属性 rollbackFor 设置错误

属性 `rollbackFor` 可以**指定能够触发事务回滚的异常类型**。

Spring 默认抛出了未检查`unchecked`异常（继承自 `RuntimeException`的异常）或者 `Error`才回滚事务，其他异常不会触发回滚事务。

若在目标方法中抛出的异常是 `rollbackFor` 指定的**异常的子类**，事务同样**会回滚**。

![image-20220912165303615](https://img.zxdmy.com/2022/202209121653647.png)

#### 同一个类中方法调用，导致@Transactional失效

比如 MyService 类中有方法 A 和方法 B：

+ 方法 A 没有声明事务，方法 B 声明了事务
+ 方法 A 中调用了方法 B
+ MyService 类外部调用 方法 A

这时，方法 B 的事务不会生效。

这是因为 Spring AOP 的动态代理呆滞的，只有事务方法被当前类以外的代码调用时，才会由 Spring 生成代理对象来管理。