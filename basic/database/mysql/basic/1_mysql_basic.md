## 1、数据库基础

### 1.1 数据库设计的基本步骤

1. **需求分析** : 分析用户的需求，包括数据、功能和性能需求。
2. **概念结构设计** : 主要采用 E-R 模型进行设计，包括画 E-R 图。
3. **逻辑结构设计** : 通过将 E-R 图转换成表，实现从 E-R 模型到关系模型的转换。
4. **物理结构设计** : 主要是为所设计的数据库选择合适的存储结构和存取路径。
5. **数据库实施** : 包括编程、测试和试运行
6. **数据库的运行和维护** : 系统的运行与数据库的日常维护。

### 1.2 数据库三范式概念

> **`范式`（ Normal Form）**，是英国人 E.F.Codd（关系数据库的老祖宗）在上个世纪70年代提出关系数据库模型后总结出来的，范式是关系数据库理论的基础，也是我们在设计数据库结构过程中所要遵循的规则和指导方法。目前有迹可寻的共有8种范式，依次是：1NF，2NF，3NF，BCNF，4NF，5NF，DKNF，6NF。

通常所用到的只是前三个范式，即：**第一范式（1NF），第二范式（2NF），第三范式（3NF）。**

+ **第一范式（1NF）**：要求属性不能被继续分割，是**数据库最基本的要求**。

  + 比如下面这个表不符合 1NF，将所有属性分解为最基本的属性即可。

    <img src="https://img.zxdmy.com/2022/202207221527511.png" alt="img" style="zoom:67%;" />

+ **第二范式（2NF）**：**消除非主属性对于码的部分函数依赖** ，即在满足 1NF 的前提下，要求表必须有主键，并且没有包含在主键中的列必须完全依赖于主键，不能只依赖于主键的一部分。

  + 比如下面这个表不符合 2NF，主码为【学号，课程号】，而关系模式中存在【姓名】等属性依赖于【学号】的关系，即【姓名】等属性对【学号，课程号】存在部分函数依赖。

    <img src="https://img.zxdmy.com/2022/202207221529500.png" alt="img" style="zoom: 80%;" />

  + 将其转化为如下 2 个表，即可符合 2NF：

    + 学生信息表【学号，姓名，性别，系名，公寓名称】
    + 成绩表【学号，课程号，成绩】

  + *但是，如果一个系一栋或几栋公寓的前提下，上面这种转化方式不符合 3NF。*

+ **第三范式（3NF）**： **消除非主属性对于码的函数依赖** ，即在满足 2NF 的前提下，要求非主键列必须直接依赖于主键，不能存在传递依赖，即不能存在【非主键列 A 依赖于非主键列 B ，非主键列 B 依赖于主键】的情况。

  + 比如一个 订单表【订单ID，订单日期，客户ID，客户名称，客户地址】，其主键是【订单ID】：
    + 订单表中非主键列，均完全依赖于主键【订单ID】，符合 2NF；
    + 但非主键列中存在一种直接依赖关系，即【客户名称，客户地址】直接依赖于【客户ID】；
    + 【客户名称，客户地址】是通过【客户ID】传递依赖于主键【订单ID】，故不符合 3NF。
  + 将其转化为如下 2 个表，即可符合 3NF：
    + 订单表【订单ID，订单日期，客户ID】
    + 客户表【客户ID，客户名称，客户地址】

## 2、MySQL 执行过程

1. **建立连接与权限校验**：**客户端**通过 `TCP` 连接发送连接请求到 `MySQL` **连接器**，**连接器**会对该请求进行**权限验证及连接资源分配**；
2. **查询缓存**：判断缓存是否在哈希表中。在查询命中时，`MySQL` 不会进行解析查询语句，而是**直接使用** SQL 语句和客户端发送过来的其他原始信息。所以，任何字符上的不同，例如空格、注解等都会导致缓存的不命中。
3. **解析与预处理**：即**解析器**通过**词法分析**和**语法分析**验证`SQL`的合法性，并生成**解析树**；**预处理器**则根据 MySQL 规则进一步检查解析树的合法性，以及检查数据表和数据列是否存在，解析别名看是否存在歧义。
4. **优化**：**优化器**负责将语法树转化成执行计划，比如根据 SQL 语句，决定使用哪个索引，或者决定表的连接顺序等。
5. **执行**：**执行器**负责根据执行计划，调用**存储引擎**的API接口来完成整个查询工作。
6. **返回结果**：将执行的结果返回给客户端。如果开启了查询缓存，同时会将数据缓存到查询缓存中。

![image-20220825174256805](https://img.zxdmy.com/2022/202208251742112.png)

![image-20220819200441740](https://img.zxdmy.com/2022/202208192004163.png)

更新语句执行会复杂一点。需要检查表是否有排它锁，写 binlog，刷盘，是否执行 commit。