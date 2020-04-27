

#### Throwable

![](C:\Users\kyrie\Pictures\java异常.PNG)

**RuntimeException**

| 序号 | 异常名称                       | 异常描述         |
| :--- | :----------------------------- | :--------------- |
| 1    | ArrayIndexOutOfBoundsException | 数组越界异常     |
| 2    | NullPointerException           | 空指针异常       |
| 3    | IllegalArgumentException       | 非法参数异常     |
| 4    | NegativeArraySizeException     | 数组长度为负异常 |
| 5    | IllegalStateException          | 非法状态异常     |
| 6    | ClassCastException             | 类型转换异常     |

**UncheckedException**

| 序号 | 异常名称               | 异常描述                         |
| :--- | :--------------------- | :------------------------------- |
| 1    | NoSuchFieldException   | 表示该类没有指定名称抛出来的异常 |
| 2    | NoSuchMethodException  | 表示该类没有指定方法抛出来的异常 |
| 3    | IllegalAccessException | 不允许访问某个类的异常           |
| 4    | ClassNotFoundException | 类没有找到抛出异常               |

**throws 和 throw**

```java
public exception() throws Exception{
    ...
    throw new Exception();
}
```

throw 语句用在方法体内，表示抛出异常，由方法体内的语句处理。

throws 语句用在方法声明后面，表示再抛出异常，由该方法的调用者来处理。

throws 主要是声明这个方法会抛出这种类型的异常，使它的调用者知道要捕获这个异常。throw 是具体向外抛异常的动作，所以它是抛出一个异常实例。



#### 异常处理的原则

* 不要捕获类似 `Exception` 之类的异常，而应该捕获类似特定的异常，比如 `InterruptedException`，方便排查问题
* 不要生吞异常。这是异常处理中要特别注重的事情。如果我们不把异常抛出来，或者也没有输出到 Logger 日志中，程序可能会在后面以不可控的方式结束
* 不要在函数式编程中使用 `checkedException`

#### Error

Error 是程序无法处理的错误，表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM（Java 虚拟机）出现的问题。比如 `OutOfMemoryError` 和 `StackOverflowError`

- 虚拟机栈：如果线程请求的栈深度大于虚拟机栈所允许的深度，将会出现 `StackOverflowError` 异常；如果虚拟机动态扩展无法申请到足够的内存，将出现 `OutOfMemoryError`。
- 本地方法栈和虚拟机栈一样
- 堆：Java 堆可以处于物理上不连续，逻辑上连续，就像我们的磁盘空间一样，如果堆中没有内存完成实例分配，并且堆无法扩展时，将会抛出 OutOfMemoryError。
- 方法区：方法区无法满足内存分配需求时，将抛出 OutOfMemoryError 异常。



##### 一道非常经典的面试题，**NoClassDefFoundError 和 ClassNotFoundException 有什么区别**？

ClassNotFoundException 和 NoClassDefFoundError 都是由 CLASSPATH 中缺少类引起的，通常是由于缺少 JAR 文件而引起的，但是如果 JVM 认为应用运行时找不到相应的引用，就会抛出 NoClassDefFoundError 错误；当你在代码中显示的加载类比如 `Class.forName()` 调用时却没有找到相应的类，就会抛出 `java.lang.ClassNotFoundException`。

* NoClassDefFoundError 是 JVM 引起的错误，是 unchecked，未经检查的。因此不会使用 try-catch 或者 finally 语句块；另外，ClassNotFoundException 是受检异常，因此需要 try-catch 语句块或者 try-finally 语句块包围，否则会导致编译错误。
* 调用 **Class.forName()、ClassLoader.findClass() 和 ClassLoader.loadClass()** 等方法时可能会引起 `java.lang.ClassNotFoundException`
* 调用 **Class.forName()、ClassLoader.findClass() 和 ClassLoader.loadClass()** 等方法时可能会引起 `java.lang.ClassNotFoundException`