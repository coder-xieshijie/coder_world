![Java类被加载到内存](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/Java类被加载到内存_ff8a1debd72fc665a4813234ad42fdec.jpg)




## 面试连环call

1. Java类是如何被加载到内存中的？
2. Java类的生命周期都有哪些阶段？
3. JVM加载的class文件都有哪些来源？
4. JVM在加载class文件时，何时判断class文件的格式是否符合要求？



## 类生命周期

一个类从被加载到虚拟机内存开始，到卸载出内存为止，它的整个生命周期将会经历**加载、验证、准备、解析、初始化、使用和卸载**七个阶段，其中**验证、准备、解析**三个部分统称为**连接**。

![类的生命周期](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/01n2cq6mb3_733d56d3e5746842d033197e8afb9ee1.png)



- **加载、验证、准备、初始化和卸载**这五个阶段的顺序是确定的，类型的加载过程必须按照这种顺序按步就班地开始，
- **解析阶段**顺序则不固定，它可以在初始化之前，而在某些情况下也可以在初始化阶段之后，这是为了支持Java语言的**运行时绑定特性**。



## 类加载过程

系统加载 Class 类型的文件主要三步：**加载->连接->初始化**。连接过程又可分为三步：**验证->准备->解析**。



![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/ioh3f64wc7_0ea4e83ff9a1c35fc004929672e3cc3b.png)



## 加载



> 加载是类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情

- 通过一个类的全限定名来获取其定义的**二进制字节流**。
- 将这个字节流所代表的静态存储结构转化为**方法区的运行时数据结构**。
- 在**Java堆**中生成一个代表这个**类的java.lang.Class对象**，作为对方法区中这些数据的访问入口。

![java_jvm_classload_1](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/java_jvm_classload_1_76a7c3c9c71306c2ba8916fbb55510b2.png)



> 加载.class文件的方式

- 从**本地文件**系统中直接加载
- 通过**网络下载**.class文件
- 从**zip，jar等归档文件**中加载.class文件
- 从专有**数据库**中提取.class文件
- 将**Java源文件**动态编译为.class文件



## 链接

### 验证

> 确保被加载的类的正确性

验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

- **文件格式验证**（Class 文件格式检查）
- **元数据验证**（字节码语义检查）
- **字节码验证**（程序语义检查）
- **符号引用验证**（类的正确性检查）

![验证阶段示意图](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/class-loading-process-verification_57598d0f5f9156f13c08e9345649e19d.png)



### 准备

> 为类的静态变量分配内存，并将其初始化为默认值

- 这时候进行内存分配的仅包括**类变量(`static`)**，而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在Java堆中。
- 这里所设置的初始值通常情况下是**数据类型默认的零值**(如`0`、`0L`、`null`、`false`等)，而不是被在Java代码中被显式地赋予的值。

![基本数据类型的零值](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/基本数据类型的零值_b6ba2b9f75814d879d3251463dcabe8b.png)

**注意**：JDK 7 之前，HotSpot 使用永久代来实现方法区的时候，类变量所使用的内存都应当在 **永久的** 中进行分配。 而在 JDK 7 及之后，HotSpot 已经把原本放在永久代的**字符串常量池、静态变量**等移动到**堆**中，这个时候类变量则会随着 Class 对象一起存放在 **Java 堆**中。

### 解析

> 虚拟机将常量池内的符号引用替换为直接引用的过程

解析动作主要针对**类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用限定符** 7 类符号引用进行



1. **符号引用**

- 符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能够无歧义的定位到目标即可



2.  **直接引用**

- 直接指向目标的指针（比如，指向Class类型、类变量、类方法的直接引用可能是指向方法区的指针）
- 相对偏移量（比如，指向实例变量、实例方法的直接引用都是偏移量）
- 一个能间接定位到目标的句柄



## 类的初始化

> 当一个JVM在启动之后，其中可能包含的类非常多，是不是每个类都会被初始化呢？答案是**否定**的。

JVM对类的初始化是一个**延迟机制**，当一个类在首次使用的时候才会被初始化，在同一个运行时package下，一个Class只会被初始化一次。



> 《Java虚拟机规范》则是严格规定了**有且只有6种情况**下必须立即对类进行**初始化**（而加载、验证、准备自然需要在此之前开始）

1. 通过**new关键字**会导致类的初始化

2. 访问**类的静态变量**，包括读取和更新会导致类的初始化

3. 访问**类的静态方法**，也会导致类初始化

4. **初始化子类**会导致父类被初始化

5. 对某个类进行**反射操作**。会导致类被初始化

6. **启动类**，就是执行main函数所在的类会导致该类被初始化



**注意：构造某个类的数组时并不会导致该类的初始化**



> 初始化步骤

- 如果这个类还没有被加载和链接，那先进行加载和链接
- 如果这个类存在直接父类，且这个类还没有被初始化，那就初始化直接的父类
- 执行类中初始化语句（如static变量和static块）



----




> 参考内容

- 《深入理解 Java 虚拟机》

- [十分钟搞懂Java类加载](https://mp.weixin.qq.com/s?__biz=MzIxNzM0NjA1OQ==&mid=2247483698&idx=1&sn=6ab24416042310b91f9363fff3402370&chksm=97fa7856a08df140e0a97ad31cbfcc3b99d13741e1db365ea08ab307392f3d1b1535f2041fdb&token=471240037&lang=zh_CN#rd)

- [Java 类加载机制](https://pdai.tech/md/java/jvm/java-jvm-classload.html)

- [类加载过程详解](https://javaguide.cn/java/jvm/class-loading-process.html)

- [JVM：类加载过程](https://cloud.tencent.com/developer/article/1787638)

