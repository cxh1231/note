`Spring` 旨在简化 `J2EE` 企业应用程序开发。

但是 ，`Spring` 的组件代码是轻量级的，但它的配置却是**重量级**的（需要大量 `XML` 配置） 。

所以，`Spring Boot`  旨在简化 `Spring` 开发（减少配置文件，开箱即用）。

`Spring Boot` 基于 Spring 开发，Spirng Boot 本身并不提供 Spring 框架的核心特性以及扩展功能，只是 **用于快速、敏捷地开发新一代基于 Spring 框架的应用程序**。

`Spring Boot` 并不是用来替代 Spring 的解决方案，而是 **和 Spring 框架紧密结合，用于提升 Spring 开发者体验的工具**。

`Spring Boot` 以 **约定大于配置** 核心思想开展工作，相比Spring具有如下优势：

+ Spring Boot 可以**快速创建**独立的Spring应用程序。
+ Spring Boot 项目所需的开发或工程时间明显减少，通常会提高整体生产力。
+ Spring Boot **不需要编写大量样板代码、XML 配置和注释**。
+ Spring 引导应用程序可以很容易地与 Spring 生态系统集成，如 Spring JDBC、Spring ORM、Spring Data、Spring Security 等。
+ Spring Boot 遵循“固执己见的默认配置”，即**约定大于配置，减少开发工作**（默认配置可以修改）。
+ Spring Boot 应用程序**提供嵌入式 HTTP 服务器**，如 Tomcat 和 Jetty，可以轻松地开发和测试 web 应用程序。（这点很赞！普通运行 Java 程序的方式就能运行基于 Spring Boot web 项目，省事很多）
+ Spring Boot 提供命令行接口(CLI)工具，用于开发和测试 Spring Boot 应用程序，如 Java 或 Groovy。
+ Spring Boot 提供了多种插件，可以使用内置工具(如 Maven 和 Gradle)开发和测试 Spring Boot 应用程序