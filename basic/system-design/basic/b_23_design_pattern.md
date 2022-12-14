## 1、创建型模式（5种）

|     模式名称     |                             描述                             |                             类图                             |
| :--------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|   **单例模式**   | 确保一个类只有一个实例，而且自行实例化并向整个系统提供这个实例 | <img src="https://img.zxdmy.com/2022/202209061856185.png" alt="image-20220906185654178" style="zoom:50%;" /> |
| **工厂方法模式** | 定义一个用于创建对象的接口，让子类决定实例化哪个类。工厂方法使一个类的实例化延迟到其子类 | ![image-20220906185824769](https://img.zxdmy.com/2022/202209061858979.png) |
| **抽象工厂模式** | 为创建一组相关或相互依赖的对象提供一个接口，而且无须指定它们的具体类 | ![image-20220906190246679](https://img.zxdmy.com/2022/202209061902130.png) |
|  **建造者模式**  | 将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示 | ![image-20220906190337842](https://img.zxdmy.com/2022/202209061903213.png) |
|     原型模式     | 用原型实例指定创建对象的种类，并且通过复制这些原型创建新的对象 | ![image-20220906190410766](https://img.zxdmy.com/2022/202209061904787.png) |

## 2、结构型模式（7种）

|    模式名称    |                             描述                             |                             类图                             |
| :------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  **代理模式**  |         为其他对象提供一种代理以控制对这个对象的访问         | ![image-20220906190555815](https://img.zxdmy.com/2022/202209061905030.png) |
|  **装饰模式**  | 动态地给一个对象添加一些额外的职责。就增加功能来说，装饰模式相比生成子类更为灵活 | ![image-20220906190626641](https://img.zxdmy.com/2022/202209061906655.png) |
| **适配器模式** | 将一个类的接口变换成客户端所期待的另一种接口，从而使原本因接口不匹配而无法在一起工作的两个类能够在一起工作 | ![image-20220906190656911](https://img.zxdmy.com/2022/202209061906961.png) |
|    组合模式    | 将对象组合成树形结构以表示“部分—整体”的层次结构，使得用户对单个对象和组合对象的使用具有一致性 | ![image-20220906190727402](https://img.zxdmy.com/2022/202209061907627.png) |
|  **桥梁模式**  |           将抽象和实现解耦，使得两者可以独立地变化           | ![image-20220906190756333](https://img.zxdmy.com/2022/202209061907394.png) |
|    外观模式    | 要求一个子系统的外部与其内部的通信必须通过一个统一的对象进行。外观模式提供一个高层次的接口，使得子系统更易使用 | ![image-20220906190828443](https://img.zxdmy.com/2022/202209061908495.png) |
|    享元模式    |          使用共享对象可有效地支持大量的细粒度的对象          | ![image-20220906190859639](https://img.zxdmy.com/2022/202209061909864.png) |

## 3、行为型模式（11 种）

|    模式名称    |                             描述                             |                             类图                             |
| :------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  模板方法模式  | 定义一个操作中的算法的框架，而将一些步骤延迟到子类中。使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤 | ![image-20220906191022769](https://img.zxdmy.com/2022/202209061910198.png) |
|    命令模式    | 将一个请求封装成个对象，从而让你使用不同的请求把客户端参数化，对请求排队或者记录请求日志，可以提供命令的撤销和恢复功能 | ![image-20220906191057753](https://img.zxdmy.com/2022/202209061910027.png) |
| **责任链模式** | 使多个对象都有机会处理请求，从而避免了请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有对象处理它为止 | ![image-20220906191136734](https://img.zxdmy.com/2022/202209061911927.png) |
|  **策略模式**  |  定义一组算法，将每个算法都封装起来，并且使它们之间可以互换  | ![image-20220906191219115](https://img.zxdmy.com/2022/202209061912076.png) |
| **迭代器模式** | 提供一种方法访问个容器对象中各个元素，而又不需暴露该对象的内部细节 | ![image-20220906191243558](https://img.zxdmy.com/2022/202209061912548.png) |
|   中介者模式   | 用一个中介对象封装一系列对象（同事）的交互，中介者使各对象不需要显式地相互作用，从而使其耦合松散，而且可以独立地改变它们之间的交互 | ![image-20220906191321327](https://img.zxdmy.com/2022/202209061913328.png) |
| **观察者模式** | 定义对象间一种一对多的依赖关系，使得每当一个对象改变状态，则所有依赖于它的对象都会得到通知并被自动更新 | ![image-20220906191350026](https://img.zxdmy.com/2022/202209061913168.png) |
|   备忘录模式   | 在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可以将该对象恢复到原先保存的状态 | ![image-20220906191424553](https://img.zxdmy.com/2022/202209061914662.png) |
|   访问者模式   | 封装一些作用于某种数据结构中的各元素的操作，它可以在不改变数据结构的前提下定义作用于这些元素的新的操作 | ![image-20220906191459196](https://img.zxdmy.com/2022/202209061915199.png) |
|    状态模式    | 当一个对象内在状态改变时允许改变行为，这个对象看起来像改变了其类型 | ![image-20220906191544669](https://img.zxdmy.com/2022/202209061915105.png) |
|   解释器模式   | 给定一门语言，定义它的文法的一种表示，并定义一个解释器，该解释器使用该表示来解释语言中的句子 | ![image-20220906191614391](https://img.zxdmy.com/2022/202209061916467.png) |

