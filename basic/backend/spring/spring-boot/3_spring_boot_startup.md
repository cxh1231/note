`SpringBootApplication` 这个类主要做了以下四件事情：

1. 推断应用的类型是普通的项目还是 Web 项目
2. 查找并加载所有可用初始化器 ， 设置到 initializers 属性中
3. 找出所有的应用程序监听器，设置到 listeners 属性中
4. 推断并设置 `main` 方法的定义类，找到运行的主类`SpringBoot` 启动大致流程如下 ：

![image-20220912215148581](https://img.zxdmy.com/2022/202209122152593.png)

