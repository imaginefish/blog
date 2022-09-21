---
title: Spark 的序列化问题总结
date: 2022-09-20 16:24:06
categories:
- 大数据
tags: 
- 序列化
- Spark
- Java
toc: true
---
## Java 序列化
Java 序列化就是指将一个对象转化为二进制的 `byte[]` 数组，然后以文件的方式进行保存或通过网络传输，等待被反序列化读取出来。序列化常被用于数据存取和通信过程中。

一个 Java 对象要能序列化，必须实现一个特殊的 `java.io.Serializable` 接口，它的定义如下：
```java
public interface Serializable {
}
```
<!--more-->
Serializable 接口没有定义任何方法，它是一个空接口。我们把这样的空接口称为“标记接口”（Marker Interface）。

但实现该接口不保证该对象一定可以序列化，因为序列化必须保证该对象的所有属性可以序列化。

并且 static 和 transient 修饰的变量不会被序列化，这也是解决序列化问题的方法之一，让不能序列化的引用用 static 和 transient 来修饰。（transient 修饰的变量，是不会被序列化到文件中，在被反序列化后，transient 变量的值被设为初始值，如 int 是0，对象是 null）

此外还可以实现 readObject() 方法和 writeObject() 方法来自定义实现序列化。

### 序列化
```java
import java.io.*;
import java.util.Arrays;
public class Main {
    public static void main(String[] args) throws IOException {
        ByteArrayOutputStream buffer = new ByteArrayOutputStream();
        try (ObjectOutputStream output = new ObjectOutputStream(buffer)) {
            // 写入int:
            output.writeInt(12345);
            // 写入String:
            output.writeUTF("Hello");
            // 写入Object:
            output.writeObject(Double.valueOf(123.456));
        }
        System.out.println(Arrays.toString(buffer.toByteArray()));
    }
}
```
### 反序列化
```java
try (ObjectInputStream input = new ObjectInputStream(...)) {
    int n = input.readInt();
    String s = input.readUTF();
    Double d = (Double) input.readObject();
}
```
为了避免因 class 定义变动导致的反序列不兼容，抛出 `InvalidClassException` 类不匹配异常，Java 的序列化允许 class 定义一个特殊的 serialVersionUID 静态变量，用于标识 Java 类的序列化“版本”，通常可以由 IDE 自动生成。如果增加或修改了字段，可以改变 serialVersionUID 的值，这样就能自动阻止不匹配的 class 版本：
```java
public class Person implements Serializable {
    private static final long serialVersionUID = 2709425275741743919L;
}
```
## Spark 序列化
Spark 是分布式执行引擎，其核心抽象是弹性分布式数据集 RDD，其代表了分布在不同节点的数据。Spark 的计算是在 executor 上分布式执行的，故用户开发的对于 RDD 的 `map`、`flatMap`、`reduceByKey` 等 transformation 操作会有如下的执行过程：

1. 代码中的对象在 driver 本地序列化
2. 对象序列化后传输到远程 executor 节点
3. 远程 executor 节点反序列化对象
4. 最终远程节点执行运算

故对象在 transformation 操作中需要序列化后通过网络传输，然后在 executor 节点反序列化执行运算，则要求对象必须可序列化。
## 如何解决 Spark 项目中的序列化问题
###  Java 对象
如果 RDD 保存的是 Java 对象，则要求使用 Java 机制，实现该对象 class 的序列化，即 class 实现 `Serializable` 接口。对于不可序列化对象，如果本身不需要存储或传输，则可使用 `static` 或 `trarnsient` 修饰；如果需要存储传输，则实现 `writeObject()/readObject()` 使用自定义序列化方法。
### Scala 对象
对于 scala 开发 Spark 程序，可以定义样例类（case class）来创建对象，实例化后的对象直接可序列化。

此外还需注意哪些操作在 driver，哪些操作在 executor 执行，因为在driver 端（foreachRDD）实例化的对象，很可能不能在 foreach 中运行，因为对象不能从 drive 序列化传递到 executor 端（有些对象有 TCP 链接，一定不可以序列化）。所以这里一般在 foreachPartitions 或 foreach 算子中来实例化对象，这样对象在 executor 端实例化，没有从 driver 传输到 executor 的过程。

>参考链接：
https://blog.csdn.net/weixin_38653290/article/details/84503295
