---
top: true
title: UNSAFE和Java 内存布局
categories: [java]
mathjax: true
date: 2019-11-15
cover: https://mysticalyu.gitee.io/pic/img/huifeng-huang-upset1-5k.jpg
sticky: 99
author:
  name: staconfree
  avatar: https://cdn2.jianshu.io/assets/default_avatar/15-a7ac401939dd4df837e3bbf82abaa2a8.jpg
  url: https://www.jianshu.com/p/cb5e09facfee
description: 在看CAS中经常会遇到unsafe.compareAndSwapInt(this, stateOffset, expect, update);很久很久以前看着就当眼熟;现在再看，结果对这个偏移量完全未知，于是有了这篇文章
---

![](https://mysticalyu.gitee.io/pic/img/huifeng-huang-upset1-5k.jpg)

在看CAS中经常会遇到unsafe.compareAndSwapInt(this, stateOffset, expect, update);很久很久以前看着就当眼熟;现在再看，结果对这个偏移量完全未知，于是有了这篇文章



<!-- more -->



# UNSAFE和Java 内存布局（深入理解：锁/反射/线程挂起/内存回收等）

最近在翻ReentrantLock源码的时候，看到AQS（AbstractQueuedSynchronizer.java）里面有一段代码



```java
    protected final boolean compareAndSetState(int expect, int update) {
        // See below for intrinsics setup to support this
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
```

这就是经典的CAS的算法，这里包含两个陌生的东西，unsafe，stateOffset。



```kotlin
private static final Unsafe unsafe = Unsafe.getUnsafe();

stateOffset = unsafe.objectFieldOffset
                (AbstractQueuedSynchronizer.class.getDeclaredField("state"));
```

又发现stateOffset是跟AQS里面的state字段相关



```cpp
private volatile int state;
```

然后我们又发现state是volatitle类型的，当然这是实现LOCK必备的。

## 思考

这个stateOffset是什么，值是多少，由stateOffset能得到什么？由CAS的算法我们知道需要跟原值进行对比，所以大胆推测通过stateOffset可以得到state字段的值。

另外还有一个东西很让人好奇，UNSAFE是什么，能做什么？

## 粗略认识

带着这两个问题，查了不少资料，这里我希望尽量能用白话的方式说明一下。
 **UNSAFE**，顾名思义是不安全的，他的不安全是因为他的权限很大，可以调用操作系统底层直接操作内存空间，所以一般不允许使用。
 可参考：[java对象的内存布局(二):利用sun.misc.Unsafe获取类字段的偏移地址和读取字段的值](https://blog.csdn.net/aitangyong/article/details/46439375)
 我们注意到上面有一个方法

- stateOffset=unsafe.objectFieldOffset(field)  从方法名上可以这样理解：获取object对象的属性Field的偏移量。

要理解这个偏移量，需要先了解java的内存模型

## Java内存模型



![offset_1](/images/storage/offset_1.jpg)


 此文章值得认真阅读几遍： [java对象在内存中的结构（HotSpot虚拟机）](https://www.cnblogs.com/duanxz/p/4967042.html)



Java对象在内存中存储的布局可以分为三块区域：对象头（Header）、实例数据（Instance Data）和对齐填充（Padding），简单的理解：

- **对象头**，对象是什么？

- **实例数据**，对象里有什么？

- 对齐填充

  ，不关键，目的是补齐位数达到8的倍数。

  参考：

  对象的内存布局

  ![offset_2](/images/storage/offset_2.jpg)

  image.png

举个简单的例子，如下类：



```cpp
class VO {
    public int a = 0;
    public int b = 0;
}
```

VO vo=new VO();的时候，Java内存中就开辟了一块地址，包含一个固定长度的对象头（假设是16字节，不同位数机器/对象头是否压缩都会影响对象头长度）+实例数据（4字节的a+4字节的b）+padding。

这里直接说结论，我们上面说的偏移量就是在这里体现，如上面a属性的偏移量就是16，b属性的偏移量就是20。

在unsafe类里面，我们发现一个方法unsafe.getInt(object, offset);
 通过unsafe.getInt(vo, 16) 就可以得到vo.a的值。是不是联想到反射了？其实java的反射底层就是用的UNSAFE（***具体如何实现，预留到以后研究\***）。

## 进一步思考

**如何知道一个类里面每个属性的偏移量？只根据偏移量，java怎么知道读取到哪里为止是这个属性的值？**

查看属性偏移量，推荐一个工具类jol：http://openjdk.java.net/projects/code-tools/jol/
 用jol可以很方便的查看java的内存布局情况，结合一下代码讲解



```xml
        <dependency>
            <groupId>org.openjdk.jol</groupId>
            <artifactId>jol-core</artifactId>
            <version>0.9</version>
        </dependency>
```



```java
public class VO {
    public int a = 0;
    public long b = 0;
    public String c= "123";
    public Object d= null;
    public int e = 100;
    public static int f= 0;
    public static String g= "";
    public Object h= null;
    public boolean i;
}
```



```kotlin
    public static void main(String[] args) throws Exception {
        System.out.println(VM.current().details());
        System.out.println(ClassLayout.parseClass(VO.class).toPrintable());
        System.out.println("=================");
        Unsafe unsafe = getUnsafeInstance();
        VO vo = new VO();
        vo.a=2;
        vo.b=3;
        vo.d=new HashMap<>();
        long aoffset = unsafe.objectFieldOffset(VO.class.getDeclaredField("a"));
        System.out.println("aoffset="+aoffset);
        // 获取a的值
        int va = unsafe.getInt(vo, aoffset);
        System.out.println("va="+va);
    }

    public static Unsafe getUnsafeInstance() throws Exception {
        // 通过反射获取rt.jar下的Unsafe类
        Field theUnsafeInstance = Unsafe.class.getDeclaredField("theUnsafe");
        theUnsafeInstance.setAccessible(true);
        // return (Unsafe) theUnsafeInstance.get(null);是等价的
        return (Unsafe) theUnsafeInstance.get(Unsafe.class);
    }
```

在我本地机器测试结果如下：



```csharp
# Running 64-bit HotSpot VM.
# Using compressed oop with 0-bit shift.
# Using compressed klass with 3-bit shift.
# Objects are 8 bytes aligned.
# Field sizes by type: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
# Array element sizes: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]

com.ha.net.nsp.product.VO object internals:
 OFFSET  SIZE               TYPE DESCRIPTION                               VALUE
      0    12                    (object header)                           N/A
     12     4                int VO.a                                      N/A
     16     8               long VO.b                                      N/A
     24     4                int VO.e                                      N/A
     28     1            boolean VO.i                                      N/A
     29     3                    (alignment/padding gap)                  
     32     4   java.lang.String VO.c                                      N/A
     36     4   java.lang.Object VO.d                                      N/A
     40     4   java.lang.Object VO.h                                      N/A
     44     4                    (loss due to the next object alignment)
Instance size: 48 bytes
Space losses: 3 bytes internal + 4 bytes external = 7 bytes total

=================
aoffset=12
va=2
```

在结果中，我们发现：

- 1、我本地的虚拟机环境是64位并且开启了compressed压缩，对象都是8字节对齐
- 2、VO类的内存布局包含12字节的对象头，4字节的int数据，8字节的long数据，其他String和Object是4字节，最后还有4字节的对齐。
- 3、VO类属性的内存布局跟属性声明的顺序不一致。
- 4、VO类的static属性不在VO的内存布局中，因为他是属于class类。
- 5、通过VO类就可以确定一个对象占用的字节数，这个占用空间在编译阶段就已经确定（注：此占用空间并不是对象的真实占用空间，）。
- 6、如上，通过偏移量12就可以读取到此处存放的值是2。

引申出新的问题：
 **1、这里的对象头为什么是12字节？对象头里都具体包含什么？**
 答：正常情况下，对象头在32位系统内占用一个机器码也就是8个字节，64位系统也是占用一个机器码16个字节。但是在我本地环境是开启了reference（指针）压缩，所以只有12个字节。
 **2、这里的String和Object为什么都是4字节？**
 答：因为String或者Object类型，在内存布局中，都是reference类型，所以他的大小跟是否启动压缩有关。未启动压缩的时候，32位机器的reference类型是4个字节，64位是8个字节，但是如果启动压缩后，64位机器的reference类型就变成4字节。
 **3、Java怎么知道应该从偏移量12读取到偏移量16呢，而不是读取到偏移量18或者20？**
 答：这里我猜测，虚拟机在编译阶段，就已经保留了一个VO类的偏移量数组，那12后的偏移量就是16，所以Java知道读到16为止。

更多内存布局问题请参考：
 [java对象的内存布局(一)：计算java对象占用的内存空间以及java object layout工具的使用](https://blog.csdn.net/aitangyong/article/details/46416667)
 [Java对象内存结构](http://www.importnew.com/1305.html)
 [JVM内存堆布局图解分析](https://www.cnblogs.com/SaraMoring/p/5713732.html)

## 对象头包含什么内容

[java中的对象头的解析](https://blog.csdn.net/yinbucheng/article/details/72524482)



![offset_3](/images/storage/offset_3.jpg)

- 1、对象头有几位是锁标志位
   可以参考如下文章，对象头跟锁有很重要的关联，并且文章中提到另外一个概念：**Monitor，预留到以后研究**
   [死磕Java并发：深入分析synchronized的实现原理](https://mp.weixin.qq.com/s/wHz0uL_LEe4OgLsSFGEZEg)
- 2、对象头有几位代表分代年龄，与回收算法有关
   CMS标记-清除回收算法，标记阶段的大概过程是从栈中查找所有的reference类型，递归可达的所有堆内对象的对象头都标记为数据可达，清除阶段是对堆内存从头到尾进行线性遍历，如果发现有对象没有被标识为可到达对象，那么就将此对象占用的内存回收，并且将原来标记为可到达对象的标识清除。
   在 gc回收的时候，会更新还存活的对象的对象头的分代年龄，同时如果这些对象还有发生位置移动（碎片清理），那么还要重新计算对象头的hash值，以及栈中相应的reference引用的值。

说到回收算法，再参考下这篇也更能理解对象的创建和回收：
 [垃圾回收机制中，引用计数法是如何维护所有对象引用的？](https://www.zhihu.com/question/21539353/answer/95667088)

## UNSAFE与线程的关系

unsafe中有一个park方法，与线程挂起有关，**预留到以后研究**

## 参考资料

[一个Java对象到底占多大内存？](http://www.importnew.com/14948.html)

[JVM内存模型及String对象内存分配](https://www.imooc.com/article/20818)