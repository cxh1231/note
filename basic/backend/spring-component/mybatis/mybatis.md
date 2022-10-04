## 1、MyBatis 基础

### 1.1 MyBatis 简介

`Mybatis` 是一个开源的 **持久层框架**，是一个 `半ORM`（Object Relational Mapping，**对象关系映射**）框架。

> Mybatis 在查询关联对象或关联集合对象时，需要手动编写 SQL 来完成（并非根据对象关系模型直接获取），所以，称之为半自动ORM映射工具。

`MyBatis` 内部封装了 `JDBC` （Java DataBase Connectivity，Java 数据库连接）的繁杂过程：**加载驱动、创建连接、创建Statement对象、执行SQL、获取结果集、处理结果集和关闭资源**，使得开发者开发项目时主需要关注如何编写 `SQL` 语句，`MyBatis` 能够严格控制 `SQL` 的性能和执行，灵活度高。

> Statement 对象：用于执行静态 SQL 语句并返回其生成的结果对象。
>
> Statement 对象执行 SQL 语句，存在 SQL 注入风险。

作为一个 **半ORM框架**，`MyBatis` 可以使用 `XML` 或 `注解` 来配置和映射原生信息，将 `POJO` （Java 对象）映射成数据库中的记录，有效避免了几乎所有的 `JDBC` 代码、手动参数设置以及获取结果集。

**MyBatis 从执行 SQL 到返回结果集的大致过程**：

1. 通过 xml 文件或注解的方式将要执行的各种 Statement 配置起来，
2. 通过 Java 对象和 Statement 中 SQL 的动态参数进行映射生成最终执行的sql语句，
3. 由 MyBatis 框架执行 SQL 并将结果映射为 Java 对象，然后返回。

由于 MyBatis 专注于 SQL 本身，灵活度高，所以比较适合对性能的要求很高，或者需求变化较多的项目，如互联网项目。

### 1.2 MyBatis 的优缺点

#### 优点：

1、基于 SQL 语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任 何影响，SQL 写在 XML 里，解除 sql 与程序代码的耦合，便于统一管理；提供 XML 标签，支持编写动态 SQL 语句，并可重用。

2、与 JDBC 相比，减少了 50%以上的代码量，消除了 JDBC 大量冗余的代码，不 需要手动开关连接；

3、很好的与各种数据库兼容（因为 MyBatis 使用 JDBC 来连接数据库，所以只要 JDBC 支持的数据库 MyBatis 都支持）。

4、能够与 Spring 很好的集成；

5、提供映射标签，支持对象与数据库的 ORM 字段关系映射；提供对象关系映射 标签，支持对象关系组件维护

#### 缺点：

1、SQL 语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写 SQL 语句的功底有一定要求。

2、SQL 语句依赖于数据库，导致数据库移植性差，不能随意更换数据库。

### 1.3 MyBatis 解决的 JDBC 问题

|                         JDBC 的问题                          |                      MyBatis 的解决方案                      |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|   频繁进行数据库的连接和释放造成系统资源浪费，影响系统性能   | 在 `SqlMapConfig.xml` 中配置 **数据连接池**，使用连接池管理数据库链接 |
|              SQL 语句写在代码中造成代码不易维护              |        SQL语句配置在 `*.xml` 文件中，与 Java 代码分离        |
| 向 SQL 传参数麻烦，SQL 语句的 where 条件不唯一，占位符需要和参数一一对应 |               自动将 Java 对象映射至 SQL 语句                |
|  结果集解析麻烦，SQL 变化导致解析代码变化，且解析前需要遍历  |             自动将 SQL 执行结果映射至 Java 对象              |

### 1.4 MyBatis 和 Hibernate 的异同点

**相同**：

都是对 JDBC 的封装，都是持久层的框架，都用于 Dao 层的开发。

**不同**：

|                        |                           MyBatis                            |                          Hibernate                           |
| :--------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|        映射关系        | 半自动映射框架，Java对象与sql语句执行结果的对应关系，多表关联关系配置简单 | 全表映的框架，Java对象与数据库表的对应关系，多表关联关系配置复杂 |
|    SQL优化和移植性     | 需要手动编写 SQL，支持动态 SQL、处理列表、动态生成表名、支持存储过程。开发工作量相对大些。直接使用SQL语句操作数据库，不支持数据库无关性，但sql语句优化容易。 | 对SQL语句封装，提供了日志、缓存、级联（级联比 MyBatis 强大）等特性，此外还提供 HQL（Hibernate Query Language）操作数据库，数据库无关性支持好，但会多消耗性能。如果项目需要支持多种数据库，代码开发量少，但SQL语句优化困难 |
| 开发难易程度和学习成本 | 轻量级框架，学习使用门槛低，适合于需求变化频繁，大型的项目，比如：互联网电子商务系统 | 重量级框架，学习使用门槛高，适合于需求相对稳定，中小型的项目，比如：办公自动化系统 |
|          总结          |   是一个小巧、方便、高效、简单、直接、半自动化的持久层框架   |   是一个强大、方便、高效、复杂、间接、全自动化的持久层框架   |

### 1.5 \#\{\} 和 \$\{\} 的区别

|          |                            \#\{\}                            |                            \$\{\}                            |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  占位符  |               是 `Properties` 文件的变量占位符               |         是 `SQL` 的参数占位符（拼接符，字符串替换）          |
|  预编译  |                            预编译                            |                          没有预编译                          |
| 替换方式 | 参数以字符串传入，然后将 `SQL` 中的`# {}` 换为 `?` 号，调用`PreparedStatement` 的 `set` 方法来赋值。 | 参数以原值传入，将 将 `SQL` 中的`$ {}` 直接替换为变量的值，相当于JDBC中的Statement 编译 |
|  替换后  |                 变量**自动加上**单引号 `‘ ’`                 |                 变量**不会**加上单引号 `‘ ’`                 |
| SQL 注入 |                  可以**有效防止 SQL 注入**                   |                           不能防止                           |
| 替换位置 |                           DBMS 内                            |                           DBMS 外                            |

### 1.6 MyBatis 一级、二级缓存

#### 一级缓存

**一级缓存**：基于 `PerpetualCache` 的 `HashMap` 本地缓存，其存储作用域为 `SqlSession`，各个`SqlSession` 之间的缓存相互**隔离**，当 `Session flush` 或 `close` 之后，该 `SqlSession` 中的所有 `Cache` 就将清空，MyBatis 默认打开一级缓存。

![image-20220929214800414](https://img.zxdmy.com/2022/202209292148489.png)

#### 二级缓存

**二级缓存**与一级缓存其机制相同，默认也是采用 `PerpetualCache`，`HashMap` 存储，不同之处在于其存储**作用域**为 `Mapper(Namespace)`，可以在多个`SqlSession`之间**共享**，并且可自定义存储源，如 `Ehcache`。

默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现`Serializable`序列化接口（可用来保存对象的状态），可在它的映射文件中配置。

![image-20220929214806788](https://img.zxdmy.com/2022/202209292148951.png)

## 2、MyBatis 原理

### 2.1 MyBatis 工作流程

MyBatis的基本工作流程如下：

1. 创建 `SqlSessionFactory`
2. 通过 SqlSessionFactory 创建 `SqlSession`
3. 通过 `sqlsession` 执行数据库操作
4. 调用 `session.commit()` 提交事务
5. 调用 `session.close()` 关闭会话

按工作原理，可以将工作流程分为两大步： **生成会话工厂**、**会话运行**。

![image-20220929215223073](https://img.zxdmy.com/2022/202209292152027.png)

**构造会话工厂**也可以分为两步：

1. **获取配置**
2. **构建SqlSessionFactory**

**会话运行** 是MyBatis最复杂的部分，它的运行离不开四大组件的配合：

+ Executor（执行器）
+ StatementHandler（数据库会话器）
+ ParameterHandler （参数处理器）
+ ResultSetHandler（结果处理器）

具体的源码就不分析了，完整的工作流程图如下：

![image-20220929215540501](MyBatis.assets/image-20220929215540501.png)

1. 读取 `MyBatis` 配置文件 `mybatis-config.xml` 、加载 **映射文件**（映射文件即 SQL 映射文件，文件中配置了操作数据库的 SQL 语句），最后生成一个配置对象。
2. 构造 **会话工厂**：通过 MyBatis 的环境等配置信息构建会话工厂 `SqlSessionFactory`。
3. 创建 **会话对象**：由会话工厂创建 `SqlSession` 对象，该对象中包含了执行 SQL 语句的所有方法。
4. Executor **执行器**：MyBatis 底层定义了一个 `Executor` 接口来操作数据库，它将根据 `SqlSession` 传递的参数动态地生成需要执行的 `SQL` 语句，同时负责查询缓存的维护。
5. StatementHandler：**数据库会话器**，串联起参数映射的处理和运行结果映射的处理。
6. **参数处理**：对输入参数的类型进行处理，并 **预编译**。
7. **结果处理**：对返回结果的类型进行处理，根据对象映射规则，返回相应的对象。

### 2.2 MyBatis 功能架构

一般把Mybatis的功能架构分为三层：

![image-20220930152606197](https://img.zxdmy.com/2022/202209301526195.png)

+ **API接口层**：提供给外部使用的接口API，开发人员通过这些本地API来操纵数据库。接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理。
+ **数据处理层**：负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作。
+ **基础支撑层**：负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑。









### Dao 接口的工作原理与注意事项

#### 工作原理

通常一个 `XML` 映射文件，都与一个 `Dao` 接口相对应。

`Dao 接口` 即 `Mapper 接口`。

`MyBatis` 是采用 **代理开发方式** 实现 `Dao` 层的开发，即 `MyBatis` 框架根据接口定义，创建接口的**动态代理对象**，**动态代理对象** 的 **方法体** 即 **Dao 接口实现类** 的方法。

为了实现上面的 **动态代理对象的方法体**，`Mapper.xml` 接口开发需要遵循以下规范：

+ Mapper 接口 的全限定名 和 Mapper.xml 文件中的 namespace 相同；
+ Mapper 接口方法名 和 Mapper.xml 中定义的每个 `statement 的 id 相同`；
+ Mapper接口方法 的 `输入参数类型` 和 mapper. xml 中定义的 `每个 SQL 的 parameterType 类型相同`；
+ Mapper接口方法 的 `输出参数类型` 和 mapper.xml 中定义的 `每个 SQL 的 resultType 的类型相同`。



Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名的拼接字符串作为key值，可唯一定位一个MapperStatement。

#### 注意事项

Dao接口里的方法，是不能重载的，因为是全限名+方法名的保存和寻找策略。

Dao接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，然后将sql执行结果返回。













## 3、MyBatis 插件









## 4、MyBatis 实战

### 4.1 实体类属性名与表字段名的一致性

当实体类属性名和表中字段名不一致时，可以采用如下方案解决：

1. 通过在查询的SQL语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。
2. 通过修改 `Mapper.xml` 文件中的 `resultMap`  部分的 `<result>` 来映射字段名和实体类属性名的对应的关系。

### 4.2 Mapper 传参

从 `Mapper.java` 接口，至 `Mapper.xml` ，参数可以通过如下方式传递：

+ 顺序传参法：`where user_name = #{0}`
+ @Param 注解传参法：`where user_name = #{userName}`
+ Map 传参法
+ Java Bean 传参法

### 4.3 MyBatis 映射 Enum 枚举类

Mybatis当然可以映射枚举类，不单可以映射枚举类，Mybatis可以映射任何对象到表的一列上。映射方式为自定义一个TypeHandler，实现TypeHandler的setParameter()和getResult()接口方法。

TypeHandler有两个作用，一是完成从javaType至jdbcType的转换，二是完成jdbcType至javaType的转换，体现为setParameter()和getResult()两个方法，分别代表设置sql问号占位符参数和获取列查询结果。

### 4.4 模糊查询 like 语句

模糊查询like语句有以下实现方式：

1. `'%${question}%'` ，可能引起SQL注入，不推荐
2. `"%"#{question}"%"` ，需要注意，因为 `#{…}` 解析成 `sql` 语句时候，会在变量外侧自动加**单引号**，所以这里 `%` 需要使用**双引号**，不能使用**单引号**，不然会查不到任何结果。
3. `CONCAT('%',#{question},'%')`，即使用 `CONCAT()` 函数，（推荐✨）
4. 使用`bind`标签（不推荐）

```xml
<select id="listUserLikeUsername" resultType="com.jourwon.pojo.User">
    <bind name="pattern" value="'%' + username + '%'" />
    select id,sex,age,username,password 
    from person
    where
    username LIKE #{pattern}
</select>
```

### 4.5 关联查询

MyBatis 支持一对一、一对多、多对多、多对一的关联查询。

#### 一对一

**一对一** 关联查询使用 `<association>` 标签实现。

比如订单和支付是一对一的关系，这种关联的实现如下。

实体类：

```java
public class Order {
    private Integer orderId;
    private String orderDesc;
    /**
    * 支付对象
    */
    private Pay pay;
    //……
}
```

结果映射：

```xml
<!-- 订单resultMap -->
<resultMap id="peopleResultMap" type="cn.xxx.entity.Order">
    <id property="orderId" column="order_id" />
    <result property="orderDesc" column="order_desc"/>
    <!-- 一对一结果映射 -->
    <association property="pay" javaType="cn.xxx.entity.Pay">
        <id column="payId" property="pay_id"/>
        <result column="account" property="account"/>
    </association>
</resultMap>
```

查询SQL，就是普通的关联查询：

```xml
<select id="getTeacher" resultMap="getTeacherMap" parameterType="int">
    select * from order o
    left join pay p on o.order_id=p.order_id
    where o.order_id=#{orderId}
</select>
```

#### 一对多

一对多通过 `<collection>` 标签实现。

比如商品分类和商品，是一对多的关系。

实体类：

```java
public class Category {
    private int categoryId;
    private String categoryName;
    /**
    * 商品列表
    **/
    List<Product> products;
    //……
}
```

结果映射：

```xml
<resultMap type="Category" id="categoryBean">
    <id column="categoryId" property="category_id" />
    <result column="categoryName" property="category_name" />
    <!-- 一对多的关系 -->
    <!-- property: 指的是集合属性的值, ofType：指的是集合中元素的类型 -->
    <collection property="products" ofType="Product">
        <id column="product_id" property="productId" />
        <result column="productName" property="productName" />
        <result column="price" property="price"/>
    </collection>
</resultMap>
```

### 4.6 MyBatis 回填主键

新增标签中添加：`keyProperty="ID"` 即可

```xml
<insert id="insert" useGeneratedKeys="true" keyProperty="userId" >
    insert into user(user_name, user_password, create_time)
    values(#{userName}, #{userPassword} , #{createTime, jdbcType= TIMESTAMP})
</insert>
```







### 4.7 MyBatis 批量操作













### 1.2 Xml 映射文件中，除了常见的 select|insert|update|delete 标签之外，还有哪些标签

`<resultMap>` 、 `<parameterMap>` 、 `<sql>` 、 `<include>` 、 `<selectKey>` ，加上动态 sql 的 9 个标签， `trim|where|set|foreach|if|choose|when|otherwise|bind` 等，其中 `<sql>` 为 sql 片段标签，通过 `<include>` 标签引入 sql 片段， `<selectKey>` 为不支持自增的主键生成策略标签。





## maven 和 gradle 有了解吗

> [https://www.javalearn.cn/#/doc/Mybatis/面试题](https://www.javalearn.cn/#/doc/Mybatis/面试题)