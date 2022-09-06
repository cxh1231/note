## 1、Spring 事务的管理方式（实现方式）

- **编程式事务** ： 在代码中硬编码(不推荐使用) : 通过 `TransactionTemplate`或者 `TransactionManager` 手动管理事务，实际应用中很少使用，但是对于你理解 Spring 事务管理原理有帮助。
- **声明式事务** ： 在 XML 配置文件中配置或者直接基于注解（推荐使用） : 实际是通过 AOP 实现（基于`@Transactional` 的全注解方式使用最多）

## 2、Spring 框架的事务管理优点

- 提供了跨不同事务 api（如JTA、JDBC、Hibernate、JPA和JDO）的一致编程模型。
- 为编程事务管理提供了比JTA等许多复杂事务API更简单的API。
- 支持声明式事务管理。
- 很好地集成了 Spring 的各种数据访问抽象。

## 3、Spring 事务定义的传播行为（传播规则）

**当事务方法被另一个事务方法调用时，必须指定事务应该如何传播**。

事务的传播行为主要有一下 7 种：

|          事务名称           |             如果当前存在事务             | 如果当前没有事务 |
| :-------------------------: | :--------------------------------------: | :--------------: |
|    **REQUIRED**（默认）     |                加入该事务                | 创建一个新的事务 |
|      **REQUIRES_NEW**       |   创建新事务（互相独立），挂起当前事务   | 创建一个新的事务 |
|          SUPPORTS           |                加入该事务                | 以非事务方式运行 |
|        NOT_SUPPORTED        |               挂起当前事务               | 以非事务方式运行 |
| **MANDATORY** (*mandatory*) |               加入当前事务               |     抛出异常     |
|    **NESTED** (*nested*)    | 创建一个事务作为当前事务的嵌套事务来运行 | 创建一个新的事务 |
|            NEVER            |                 抛出异常                 | 以非事务方式运行 |

使用示例：

```java
@Override
@Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)
public int saveRole(SysRole role, Integer[] menusIds) {
    // TODO 
}
```

## 4、Spring 事务中的隔离级别有哪几种?

> 这个问题和 MySQL 数据库的基本一致。

- default : 使用后端数据库默认的隔离级别
- read_uncommitted：读未提交
- read_committed：读已提交
- repeatable_read：可重复的
- serializable：串行化