![对象引用](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/对象引用_3a17300dab71fbcea29bb38fca3ba04f.jpg)



> 哈喽，大家好🎉，我是**世杰**。
>
>
> 本文我为大家介绍面试官经常考察的**「Java对象引用相关内容」**



照例在开头留一些面试考察内容~~

### 面试连环call

1. Java对象引用都有哪些类型?
2. Java参数传递是值传递还是引用传递? 为什么?
3. Java对象引用访问方式有哪些?



上一篇文章讲了 JVM 在执行 new 对象的过程中都执行了那些操作。现在我来讲一下创建对象之后的的使用

## 1. Java对象引用



> 《Java编程思想》： 每种编程语言都有自己的数据处理方式。有些时候，程序员必须注意将要处理的数据是什么类型。你是直接操纵元素，还是用某种基于特殊语法的间接表示（例如C/C++里的指针）来操作对象。所有这些在 Java 里都得到了简化，**一切都被视为对象**。因此，我们可采用一种统一的语法。尽管将一切都“看作”对象，但操纵的标识符实际是指向一个**对象的“引用”（reference）**。



Java程序需要通过**栈上的引用数据**来操作**堆上的具体对象**。对象的访问方式取决于虚拟机实现，目前主流的访问方式有使用句柄和直接指针两种。

- **直接指针**：指向对象，代表对象的内存地址。
- **句柄**：句柄是一种特殊的指针，可以理解为指向指针的指针，维护指向对象的指针变化，而对象的句柄本身不发生变化

### 1.1 直接指针

如果使用直接指针访问，那么Java堆对象的布局中就必须考虑如何放置**访问类型数据的相关信息**，而引用中存储的直接就是**对象地址**。

- **优势**：速度更快，节省了一次指针定位的时间开销。当对象的访问非常频繁，此开销积也可节省非常可观的成本。

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/2443180-20211126100056767-1083651541-20240707123734735_2496ba2cca025017ac1eb48d53e3befa.png)



### 1.2 句柄

Java堆中划分出一块内存来作为**句柄池**，引用中存储对象的句柄地址，而句柄中包含了对象**实例数据**与**类型数据**各自的具体地址信息。

- 优势：引用中存储的是稳定的句柄地址，在**对象被移动**(垃圾收集时移动对象是非常普遍的行为)时只会改变句柄中的实例数据指针，而引用**本身不需要修改**。
- 缺点：需要**两次指针访问**才能访问到对象数据。

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/2443180-20211126095946429-579451745_006729e3eae2908d3455bc70603a7abb.png)



-----



## 2. 对象引用类型



> 在Java中一共有四种引用类型，分为了：**强引用（Strong Reference）、软引用（Soft Reference）、弱引用（Weak Reference）、虚引用（Phantom Reference）4 种**，这 4 种引用的强度依次减弱。



### 2.1 强引用

> 强引用是最常见的引用类型。

- 当我们创建一个对象并将其赋值给一个引用变量时，这个引用就被称为强引用。只要强引用存在，**垃圾回收器就不会回收被引用的对象**。即使系统内存空间不足，JVM宁愿抛出**OutOfMemoryError**错误，使程序异常终止，也不会靠随意回收具有强引用的存活对象来解决内存不足问题。
- 强引用的特性是只要有强引用存在，被引用的对象就不会被垃圾回收。
- 可以将强引用赋值为 null，这样一来，JVM 就可以适时的回收对象了(**注意不是立刻回收**)

```java
public class StrongReferenceUsage {

    @Test
    public void stringReference(){
        Object obj = new Object();
    }
}
```

上面我们new了一个Object对象，并将其赋值给obj，这个obj就是new Object()的强引用。





### 2.2 软引用

> 软引用是用来描述一些还有用但并非必需的对象。

- 对于软引用关联着的对象，在系统将要**发生内存溢出异常前**，将会把这些对象列进回收范围进行**第二次回收**。如果这次回收还没有足够的内存，才会抛出内存溢出异常。在Java中，软引用通过**java.lang.ref.SoftReference**类来实现。

- 这种特性常常被用来实现**缓存技术**，比如网页缓存，图片缓存等。

```java
    @Test
    public void softReference(){
        Object obj = new Object();
        SoftReference<Object> soft = new SoftReference<>(obj);
        obj = null;
        log.info("{}",soft.get());
        System.gc();
        log.info("{}",soft.get());
    }
```

输出结果：

```
22:50:43.733 [main] INFO com.flydean.SoftReferenceUsage - java.lang.Object@71bc1ae4
22:50:43.749 [main] INFO com.flydean.SoftReferenceUsage - java.lang.Object@71bc1ae4
```

可以看到在内存充足的情况下，SoftReference引用的对象是不会被回收的。



### 2.3 弱引用

> 弱引用也是用来描述非必需对象的，不过它的强度比软引用更弱一些。

- 被弱引用关联的对象只能生存到**下一次垃圾收集发生之前**。当垃圾收集器工作时，无论当前内存是否足够，都会回收被弱引用的对象。

- 在Java中，弱引用通过**java.lang.ref.WeakReference**类来实现。

```java
    @Test
    public void weakReference() throws InterruptedException {
        Object obj = new Object();
        WeakReference<Object> weak = new WeakReference<>(obj);
        obj = null;
        log.info("{}",weak.get());
        System.gc();
        log.info("{}",weak.get());
    }
```

输出结果：

```
22:58:02.019 [main] INFO com.flydean.WeakReferenceUsage - java.lang.Object@71bc1ae4
22:58:02.047 [main] INFO com.flydean.WeakReferenceUsage - null
```

可以看到即便在内存充足的情况下，WeakReference引用的对象也被回收了。

  

### 2.4 虚引用

> 虚引用是最弱的一种引用关系。

- 一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也**无法通过虚引用来取得一个对象实例**。
- 为一个对象设置虚引用的唯一目的就是能在**这个对象被收集器回收时收到一个系统通知**。
- 在Java中，虚引用通过**java.lang.ref.PhantomReference**类来实现。
- 虚引用必须和**引用队列**（ReferenceQueue）联合使用



PhantomReference的作用是**跟踪垃圾回收器收集对象的活动**，在GC的过程中，如果发现有PhantomReference，GC则会将引用放到ReferenceQueue中，由程序员自己处理，当程序员调用ReferenceQueue.pull()方法，将引用队列中的对象引用移除之后，Reference对象会变成**Inactive**状态，意味着被引用的**对象可以被回收**了。

```java
@Slf4j
public class PhantomReferenceUsage {

    @Test
    public void usePhantomReference(){
        ReferenceQueue<Object> rq = new ReferenceQueue<>();
        Object obj = new Object();
        PhantomReference<Object> phantomReference = new PhantomReference<>(obj,rq);
        obj = null;
        log.info("{}",phantomReference.get());
        System.gc();
        Reference<Object> r = (Reference<Object>)rq.poll();
        log.info("{}",r);
    }
}
```

输出结果：

```
07:06:46.336 [main] INFO com.flydean.PhantomReferenceUsage - null
07:06:46.353 [main] INFO com.flydean.PhantomReferenceUsage - java.lang.ref.PhantomReference@136432db
```

我们看到get的值是null，而GC过后，poll是有值的。

因为PhantomReference引用的是**需要被垃圾回收的对象**，所以在类的定义中，get一直都是返回null：

```java
    public T get() {
        return null;
    }
```



**『总结』**

- **强引用是最常见的类型**，但如果内存空间不足，可能会导致OutOfMemoryError错误。
- 软引用的对象在系统内存即将溢出时会被回收，适用于**缓存**等情况。
- 弱引用的对象无论当前内存空间足够与否都会被回收，适用于实现对象的**单例模式**等。
- 虚引用的主要目的是为了能**收到系统通知**，以便在对象被回收时进行相应的处理。

在程序设计中一般很少使用弱引用与虚引用，使用软引用的情况较多，这是因为软引用可以加速 JVM 对垃圾内存的回收速度，可以维护系统的运行安全，防止内存溢出（OOM）等问题的产生。



-----



## 3. 值传递 vs 引用传递



### 3.1 定义



1. **值传递**：方法参数传入的是**参数的备份**，对参数的修改并不会影响本身
2. **引用传递**：方法参数传入的都是**参数的引用**，对参数的修改会影响本身



> Java的参数传递是值传递还是引用传递?

先说结论：Java本质上都是**值传递**



### 3.2 Java对象类型

说到值传递和引用传递我们不得不提到两个概念：**值类型和引用类型**。



> 基本类型是值类型。JVM赋值时，直接在栈上生成值

![java_value_type](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/java_value_type_82a96745d68d77a41b65539621ee10f2.png)



> 类、接口、数组、对象、基本类型的包装类都是引用类型。JVM赋值时，在栈上生成引用，在堆中生成数据

![java_reference_type](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/java_reference_type_32c04e2a32c478533d94a8405dd88b36.png)



### 3.3 为什么?



先看一段代码

```java
    @Test
    public void testReference() {
        Base base = new Base();
        base.setName("main");
        set2(base);
        System.out.println(base.getName());
    }

    private void set1(Base base) {
        base.setName("set1");
    }

    private void set2(Base base) {
        base = new Base();
        base.setName("set2");
    }
```

上述的方法，调用set1的执行结果是【set1】，调用set2的执行结果是【main】



由此可以看出，方法入参传入的都是**副本**

1. 当是基本类型时，传入的是基本类型的拷贝副本，修改基本类型的【入参】并不会影响【实参】
2. 当是引用类型时，传入的引用类型依旧是拷贝副本，但是此时的副本是**拷贝地址**，因此修改入参会影响到实现的值，因此**入参和实参的指向都是堆中同一个内存区域**，例如set1方法。但当为入参重新**赋值一个新的地址**，并在修改此**地址指向的内存区域**，就不会影响实参中的值，例如set2方法。



**所以Java是值传递**



----



> 参考文章

1. https://mp.weixin.qq.com/s/r5cMTmQjNwYwiX7pLx_52Q

2. [JAVA对象的创建及内存分配详解](https://www.cnblogs.com/xfeiyun/p/15600513.html)
3. [对象和引用](https://alexyyek.github.io/2014/12/29/Reference/)

4. [理解java对象的引用](https://www.cnblogs.com/better-farther-world2099/articles/17064423.html)

5. [一文读懂java中的Reference和引用类型](https://www.cnblogs.com/flydean/p/java-reference-referencetype.html)

6. [这一次，彻底解决Java的值传递和引用传递](https://segmentfault.com/a/1190000016773324)

