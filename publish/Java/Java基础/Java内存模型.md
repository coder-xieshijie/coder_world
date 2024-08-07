

![面试官](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/面试官_861e323fc0616911b75a408abc40af09.jpg)




> 哈喽，大家好🎉，我是**世杰**。
> 
> 
> 本文我为大家介绍面试官经常考察的**「Java内存模型JMM相关内容」**


## 面试连环call

1. 什么是Java内存模型(JMM)? 为什么需要JMM?
2. Java线程的工作内存和主内存各自的作用?
3. Java缓存一致性问题?
4. Java的并发编程问题?



要想理解透彻 JMM（Java 内存模型），我们先要从 **『硬件内存结构』** 说起。让我们开始吧！🎉🎉🎉



## 1. 硬件内存结构

### 1.1 CPU 缓存模型

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/java-memory-model-4_bfec7abd0e07f348c4ef51812b5331b0.png)





**(1)CPU Register**

CPU Register  也就是 **CPU 寄存器**。CPU 寄存器是 **CPU 内部集成**的，在寄存器上执行操作的效率要比在主存上高出几个数量级。

**(2)CPU Cache Memory**

CPU Cache Memory 也就是 **CPU 高速缓存**，相对于寄存器来说，通常也可以成为 L2 二级缓存。相对于硬盘读取速度来说内存读取的效率非常高，但是与 CPU 还是相差数量级，所以在 CPU 和主存间引入了多级缓存。CPU 缓存则是为了**解决 CPU 处理速度和内存处理速度不对等**的问题。

**(3)Main Memory**

Main Memory 就是**主存**，主存比 L1、L2 缓存要大很多。



### 1.2 缓存一致性问题

由于主存与 CPU 处理器的运算能力之间有**数量级的差距**，所以在传统计算机内存架构中会引入**高速缓存**来作为主存和处理器之间的缓冲。先复制一份数据到 CPU Cache 中，当 CPU 需要用到的时候就可以直接从 CPU Cache 中读取数据，当运算完成后，再将运算得到的数据写回 Main Memory 中。

但是，这样存在 **内存缓存不一致性的问题** ！比如我执行一个 i++ 操作的话，如果两个线程同时执行的话，假设两个线程从 CPU Cache 中读取的 i=1，两个线程做了 i++ 运算完之后再写回 Main Memory 之后 i=2，而正确结果应该是 i=3。

CPU 为了解决内存缓存不一致性问题可以通过**制定缓存一致协议（比如 [MESI 协议open in new window]）**或者其他手段来解决。 这个缓存一致性协议指的是在 CPU 高速缓存与主内存交互的时候需要遵守的原则和规范。不同的 CPU 中，使用的缓存一致性协议通常也会有所不同。



![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/20d39e9d5cc81017f0ae660877718450_565d8b08159d7afecaaba504813039ee.png)



### 1.3 指令重排序

> **什么是指令重排序？** 简单来说就是系统在执行代码的时候并不一定是按照你写的代码的顺序依次执行。

常见的指令重排序有下面 2 种情况：

- **编译器优化重排**：编译器（包括 JVM、JIT 编译器等）在不改变单线程程序语义的前提下，重新安排语句的执行顺序。
- **指令并行重排**：现代处理器采用了指令级并行技术(Instruction-Level Parallelism，ILP)来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
- **内存系统的重排序**：由于处理器使用缓存和读写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/8659b033108f907b9ef4f60940feb6e9_148b5a7e0cc167e333d6f5a7997fcf1a.png)

Java 源代码会经历 **编译器优化重排 —> 指令并行重排 —> 内存系统重排** 的过程，最终才变成操作系统可执行的指令序列。



----



## 2. Java内存模型



### 2.1 Java内存与硬件内存



之前讲 JVM 运行时内存区域时，聊到 **JVM 分为栈、堆**等，这些都是 JVM 定义的概念。在传统的**硬件内存架构中是没有栈和堆**这种概念。从图中可以看出**栈和堆既存在于高速缓存中又存在于主内存**中，所以Java内存和硬件内存没有直接的关系。



![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/java-memory-model-5_15c2ce1606ccea46e1e95c392f853ad0.png)



### 2.2 Java内存与主内存



(1) Java 内存模型规定所有的变量都**存储在主内存**中，每条线程都有自己的工作内存。

(2) 线程的**工作内存**中保存了被该线程使用到的变量的**主内存副本拷贝**，线程对变量的所有操作（读取、赋值等）都**必须在工作内存中**进行，而不能直接读写主内存中的变量。

(3) 不同线程之间也**无法直接访问对方工作内存**中的变量，线程间变量值的传递都需要通过主内存来完成。





![jmm](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/jmm_2bb08f233da3cb09dcee7c9072892528.png)



- **主内存**：所有线程创建的实例对象都存放在主内存中，不管该实例对象是**成员变量**，还是**局部变量**，**类信息**、**常量**、**静态变量**都是放在主内存中。为了获取更好的运行速度，虚拟机及硬件系统可能会让工作内存优先存储于寄存器和高速缓存中。

- **本地内存**：每个线程都有一个私有的本地内存，本地内存**存储了该线程以读 / 写共享变量的副本**。每个线程只能操作自己本地内存中的变量，无法直接访问其他线程的本地内存。如果线程间需要通信，必须通过主内存来进行。本地内存是 **JMM 抽象出来的一个概念**，并不真实存在，它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化。



> 为了更好的控制主内存和本地内存的交互，Java 内存模型定义了**八种操作**来实现:

**『主内存』**

- **lock：锁定**。作用于主内存的变量，把一个变量标识为一条线程独占状态

- **unlock：解锁**。作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定

- **read：读取**。作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用

- **write：写入**。作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中

**『工作内存』**

- **load：载入**。作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中

- **use：使用**。作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作

- **assign：赋值**。作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作

- **store：存储**。作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作

  

----

## 3. 再聊并发编程

熟悉 Java 并发编程的同学肯定对这三个问题很熟悉：**『可见性问题』**、**『原子性问题』**、**『有序性问题』**。

### 3.1 有序性

> 由于指令重排序问题，代码的执行顺序未必就是编写代码时候的顺序。我们上面讲重排序的时候也提到过：**指令重排序可以保证串行语义一致，但是没有义务保证多线程间的语义也一致** ，所以在多线程下，指令重排序可能会导致一些问题。

在 Java 中，`volatile` 关键字可以禁止指令进行重排序优化。



### 3.2 原子性

> 一次操作或者多次操作，要么**所有的操作全部都得到执行**并且不会受到任何因素的干扰而中断，要么**都不执行**。

在 Java 中，可以借助`synchronized`、各种 `Lock` 以及各种原子类实现原子性。



### 3.3 可见性

> 当一个线程对共享变量进行了修改，那么另外的线程都是**立即可以看到**修改后的最新值。

在 Java 中，可以借助`synchronized`、`volatile` 以及各种 `Lock` 实现可见性。



----



> 参考文章

1. [Java 内存模型引入](https://pdai.tech/md/java/jvm/java-jvm-x-introduce.html)

2. [JMM（Java 内存模型）详解](https://javaguide.cn/java/concurrent/jmm.html)

3. [说说什么是Java内存模型？](https://www.51cto.com/article/658158.html)

4. [从 CPU 讲起，深入理解 Java 内存模型！ ](https://www.cnblogs.com/chanshuyi/p/deep-insight-of-java-memory-model.html)

