## 1、异常类别

![image-20220729145309400](https://img.zxdmy.com/2022/202207291453708.png)

![image-20220809202657051](https://img.zxdmy.com/2022/202208092026349.png)

在 Java 中，所有的异常都有一个共同的祖先 `java.lang` 包中的 `Throwable` 类。`Throwable` 类有两个重要的子类:

**`Exception`** ：程序本身可以处理的异常，可以通过 `catch` 来进行捕获。`Exception` 又可以分为 ：

- Checked Exception (受检查异常，必须处理) ，
- Unchecked Exception (不受检查异常，可以不处理)。

**`Error`** ：程序无法处理的错误。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止：

- Java 虚拟机运行错误（`Virtual MachineError`），
- 虚拟机内存不够错误(`OutOfMemoryError`)，
- 类定义错误（`NoClassDefFoundError`）等 。

## 2、受检查异常与不受检查异常

#### 受检查异常（一般异常）

`Checked Exception` 即 **受检查异常（一般异常）** 。Java 代码在编译过程中，如果**受检查异常**没有被 `catch`或者`throws` 关键字处理的话，就没办法通过编译。比如：

-  IO 相关的异常

-  `ClassNotFoundException` 

-  `SQLException` ...

#### 不受检查异常（运行时异常）

`Unchecked Exception` 即 **不受检查异常（运行时异常）** ，Java 代码在编译过程中 ，即使不处理`不受检查异常`也可以正常通过编译。比如：

- `NullPointerException`(空指针错误)
- `IllegalArgumentException`(参数错误比如方法入参类型错误)
- `NumberFormatException`（字符串转换为数字格式错误，`IllegalArgumentException`的子类）
- `ArrayIndexOutOfBoundsException`（数组越界错误）
- `ArithmeticException`（算术错误）……

## 3、try-catch-finally

- `try` 块：用于捕获异常。其后可接多个 `catch` 块，若没有 `catch` 块，则必须接一个 `finally` 块。
- `catch` 块：用于处理 `try` 捕获到的异常。
- `finally` 块：无论是否捕获或处理异常，`finally` 块里的语句都会被执行。
  - 当在 `try` 块或 `catch` 块中遇到 `return` 语句时，`finally` 语句块将在方法返回之前被执行。

#### 不要在 finally 语句块中使用 return

 因为：try 语句遇到 return 时，其返回值会暂存本地变量中，然后执行 finally 语句，然后由于遇到了 finally 中的 return，就执行该 return，进而忽略了 try 中的 return，变为了 finally 语句中的 return 返回值。

示例：

```java
public static int getInt() {
    int a = 10;
    try {
        System.out.println(a / 0);
        a = 20;
    } catch (ArithmeticException e) {
        a = 30;
        return a;
    } finally {
        a = 400;
        //如果这样，就又重新形成了一条返回路径，由于只能通过一个 return 返回，所以这里直接返回 400
        return a; 
    }
}
// 返回结果：400
```

## 4、finally 的代码一定会执行吗？

不一定，有以下几种情况，不会执行：

-  `finally` 之前虚拟机被终止运行（`System.exit(1);`）

-  程序所在的线程死亡

-  关闭 CPU

## 5、异常使用注意事项

+ 不要把异常定义为静态变量，因为这样会导致异常栈信息错乱。
+ 抛出的异常信息一定要有意义。
+ 抛出更加具体的异常
+ 使用日志打印异常之后就不要再抛出异常