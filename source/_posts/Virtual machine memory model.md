---
top: true
title: java虚拟机内存模型
categories: [java]
date: 2019-12-04
mathjax: true
cover: https://gschaos.club/ico/img/huifeng-huang-pose-cl05.jpg
description: java虚拟机内存模型
---

java虚拟机内存模型



<!-- more -->



> 作者：土豆是我的最爱
>
> 原文链接：https://blog.csdn.net/qq_37141773/article/details/103138476

# 虚拟机

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191119111545984.png)


同样的java代码在不同平台生成的机器码肯定是不一样的，因为不同的操作系统底层的硬件指令集是不同的。

同一个java代码在windows上生成的机器码可能是0101.......，在linux上生成的可能是1100......，那么这是怎么实现的呢？

不知道同学们还记不记得，在下载jdk的时候，我们在oracle官网，基于不同的操作系统或者位数版本要下载不同的jdk版本，也就是说针对不同的操作系统，jdk虚拟机有不同的实现。

那么虚拟机又是什么东西呢，如图是从软件层面屏蔽不同操作系统在底层硬件与指令上的区别，也就是跨平台的由来。

说到这里同学们可能还是有点不太明白，说的还是太宏观了，那我们来了解下java虚拟机的组成。

# 虚拟机组成

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125152256267.png)

## 栈
我们先讲一下其中的一块内存区域栈，大家都知道栈是存储局部变量的，也是线程独有的区域，也就是每一个线程都会有自己独立的栈区域。



```java
public class Math {
    public static int initData = 666;
    public static User user = new User();
public int compute() {
    int a = 1;
    int b = 2;
    int c = (a+b) * 10;
    return c;
}
 
public static void main(String[] args) {
    Math math = new Math();
    math.compute();
    System.out.println("test");
	}
}
```

说起栈大家都不会陌生，数据结构中就有学，这里线程栈中存储数据的部分使用的就是栈，先进后出。

大家都知道每个方法都有自己的局部变量，比如上图中<font color=#98F5FF>main</font>方法中的<font color=#FF6A6A>math</font>，<font color=#FF6A6A>compute</font>方法中的<font color=#98F5FF>a b c</font>，那么java虚拟机为了区分不同方法中局部变量作用域范围的内存区域，每个方法在运行的时候都会分配一块独立的栈帧内存区域，我们试着按上图中的程序来简单画一下代码执行的内存活动。

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125161338493.png)



执行<font color=#FF69B4>main</font>方法中的第一行代码是，栈中会分配<font color=#FF6A6A>main()</font>方法的栈帧，并存储<font color=#FFA500>math</font>局部变量,，接着执行<font color=#FF6A6A>compute()</font>方法，那么栈又会分配<font color=#FFA500>compute()</font>的栈帧区域。

这里的栈存储数据的方式和数据结构中学习的栈是一样的，先进后出。当<font color=#FFA500>compute()</font>方法执行完之后，就会出栈被释放，也就符合先进后出的特点，后调用的方法先出栈。

**栈帧**
那么栈帧内部其实不只是存放局部变量的，它还有一些别的东西，主要由四个部分组成。

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/2019112516224297.png)



那么要讲这个就会涉及到更底层的原理--字节码。我们先看下我们上面代码的字节码文件。

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125164159994.png)



看着就是一个16字节的文件，看着像乱码，其实每个都是有对应的含义的，oracle官方是有专门的jvm字节码指令手册来查询每组指令对应的含义的。那我们研究的，当然不是这个。

jdk有自带一个<font color=#FFA500>javap</font>的命令，可以将上述class文件生成一种更可读的字节码文件。

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125170812635.png)



我们使用<font color=#FFA500>javap -c</font>命令将class文件反编译并输出到TXT文件中。

```c++
Compiled from "Math.java"
public class com.example.demo.test1.Math {
  public static int initData;

  public static com.example.demo.bean.User user;

  public com.example.demo.test1.Math();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public int compute();
    Code:
       0: iconst_1
       1: istore_1
       2: iconst_2
       3: istore_2
       4: iload_1
       5: iload_2
       6: iadd
       7: bipush        10
       9: imul
      10: istore_3
      11: iload_3
      12: ireturn

  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class com/example/demo/test1/Math
       3: dup
       4: invokespecial #3                  // Method "<init>":()V
       7: astore_1
       8: aload_1
       9: invokevirtual #4                  // Method compute:()I
      12: pop
      13: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
      16: ldc           #6                  // String test
      18: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      21: return

  static {};
    Code:
       0: sipush        666
       3: putstatic     #8                  // Field initData:I
       6: new           #9                  // class com/example/demo/bean/User
       9: dup
      10: invokespecial #10                 // Method com/example/demo/bean/User."<init>":()V
      13: putstatic     #11                 // Field user:Lcom/example/demo/bean/User;
      16: return
}
```



此时的jvm指令码就清晰很多了，大体结构是可以看懂的，类、静态变量、构造方法、compute()方法、main()方法。

其中方法中的指令还是有点懵，我们举compute()方法来看一下：

```c++
Code:
       0: iconst_1
       1: istore_1
       2: iconst_2
       3: istore_2
       4: iload_1
       5: iload_2
       6: iadd
       7: bipush        10
       9: imul
      10: istore_3
      11: iload_3
      12: ireturn
```



这几行代码就是对应的我们代码中compute()方法中的四行代码。大家都知道越底层的代码，代码实现的行数越多，因为他会包含一些java代码在运行时底层隐藏的一些细节原理。

那么一样的，这个jvm指令官方也是有手册可以查阅的，网上也有很多翻译版本，大家如果想了解可自行百度。

这里我只讲解本博文设计代码中的部分指令含义：

0. 将int类型常量1压入操作数栈

```c++
0: iconst_1     
```

这一步很简单，就是将1压入操作数栈 

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125173802589.png)



1.  将int类型值存入局部变量1 

```c++
1: istore_1     

```



局部变量1，在我们代码中也就是第一个局部变量a，先给a在局部变量表中分配内存，然后将int类型的值，也就是目前唯一的一个1存入局部变量a

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125174002271.png)



2. 将int类型常量2压入操作数栈

```c++
2: iconst_2
```



3.  将int类型值存入局部变量2 

```c++
3: istore_2
```



这两行代码就和前两行类似了。



![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125175335780.png)

4. 从局部变量1中装载int类型值

```c++
4: iload_1
```



5. 从局部变量2中装载int类型值 

```c++
5: iload_2
```



这两个代码是将局部变量1和2，也就是a和b的值装载到操作数栈中

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125195538208.png)



6. 执行int类型的加法

```c++
6: iadd
```



<font color=#FFA500>iadd</font>指令一执行，会将操作数栈中的1和2依次从栈底弹出并相加，然后把运算结果3在压入操作数栈底。

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125195857649.png)



7. 将一个8位带符号整数压入栈

```c++
7: bipush        10
```



 这个指令就是将10压入栈

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125200118182.png)



8. 执行int类型的乘法

```c++
9: imul
```



这里就类似上面的加法了，将3和10弹出栈，把结果30压入栈

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125200401195.png)




9. 将将int类型值存入局部变量3 

```c++
10: istore_3
```



这里大家就不陌生了吧，和第二步第三步是一样的，将30存入局部变量3，也就是c

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125200811132.png)



10. 从局部变量3中装载int类型值

```c++
11: iload_3
```



这个前面也说了

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125201027757.png)

11. 返回int类型值

```c++
12: ireturn
```

这个就不用多说了，就是将操作数栈中的30返回

 

到这里就把我们<font color=#FF69B4>compute()</font>方法讲解完了，讲完有没有对局部变量表和操作数栈的理解有所加深呢？说白了赋值号=后面的就是操作数，在这些操作数进行赋值，运算的时候需要内存存放，那就是存放在操作数栈中，作为临时存放操作数的一小块内存区域。

接下来我们再说说方法出口。

方法出口说白了不就是方法执行完了之后要出到哪里，那么我们知道上面<font color=#FF69B4>compute()</font>方法执行完之后应该回到<font color=#FFA500>main()</font>方法第三行那么当<font color=#FF69B4>main()</font>方法调用<font color=#FF34B3>compute()</font>的时候，<font color=#FF34B3>compute()</font>栈帧中的方法出口就存储了当前要回到的位置，那么当<font color=#FF34B3>compute()</font>方法执行完之后，会根据方法出口中存储的相关信息回到<font color=#FFA500>main()</font>方法的相应位置。

那么<font color=#FFA500>main()</font>方同样有自己的栈帧，在这里有些不同的地方我们讲一下。

我们上面已经知道局部变量会存放在栈帧中的局部变量表中，那么<font color=#FFA500>main()</font>方法中的<font color=#FF6A6A>math</font>会存入其中，但是这里的math是一个对象，我们知道new出来的对象是存放在堆中的

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125202605931.png)



那么这个math变量和堆中的对象有什么联系呢？是同一个概念么？

当然不是的，局部变量表中的math存储的是堆中那个math对象在堆中的内存地址

## 程序计数器
程序计数器也是线程私有的区域，每个线程都会分配程序计数器的内存，是用来存放当前线程正在运行或者即将要运行的<font color=#1E90FF>jvm指令码对应的地址</font>，或者说行号位置。

上述代码中每个指令码前面都有一个行号，你就可以把它看作当前线程执行到某一行代码位置的一个标识，这个值就是程序计数器的值。

那么jvm虚拟机为什么要设置程序计数器这个结构呢？就是为了多线程的出现，多线程之间的切换，当一个程序被挂起的时候，总是要恢复的，那么恢复到哪个位置呢，总不能又重新开始执行吧，那么程序计数器就解决了这个问题。

## 方法区
在jdk1.8之前，有一个名称叫做持久带/永久代，很多同学应该听过，在jdk1.8之后，oracle官方改名为元空间。存放常量、静态变量、类元信息。

```java
public static int initData = 666;
```



这个initData就是静态变量，毋庸置疑是存放在方法区的

```java
public static User user = new User();
```



那么这个user就有点不一样了，user变量放在方法区，new的User是存放在堆中的

到这里我们就能意识到栈，堆，方法区之间都是有联系的。

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191125204225518.png)

栈中的局部变量，方法区中的静态变量，如果是对象类型的话都会指向堆中new出来中的对象，那么红色的联系代表什么呢？我们先来了解一下对象。

对象组成
你对对象的了解有多少呢，天天用对象，你是否知道对象在虚拟机中的存储结构呢？

对象在内存中存储的布局可以分为3块区域：<font color=#1E90FF>对象头（Header）、实例数据（Instance Data）和对齐填充（Padding）</font>。下图是普通对象实例与数组对象实例的数据结构：

![è¿éåå¾çæè¿°](https://gitee.com/MysticalYu/pic/raw/master/hexo/20170419212953720)

**对象头**

HotSpot虚拟机的对象头包括两部分信息：

**Mark Word**

> 第一部分<font color=#FF69B4>markword</font>,用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，这部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit，官方称它为“MarkWord”。

**Klass Pointer**

>    对象头的另外一部分是<font color=#FF69B4>klass</font>类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例.

**数组长度（只有数组对象有）**

>    如果对象是一个数组, 那在对象头中还必须有一块数据用于记录数组长度.

**实例数据**

> 实例数据部分是对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容。无
> 论是从父类继承下来的，还是在子类中定义的，都需要记录起来。

**对齐填充**

> 第三部分对齐填充并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。由于 
> HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说，就是对象
> 的大小必须是8字节的整数倍。而对象头部分正好是8字节的倍数（1倍或者2倍），因此，当对象> 实例数据部分没有对齐时，就需要通过对齐填充来补全。

其中的klass类型指针就是那条红色的联系，那是怎么联系的呢？

```java
new Thread().start();
```




类加载其实最终是以类元信息的形式存储在方法区中的，math和math2都是由同一个类new出来的，当对象被new时，都会在对象头中存储一个指向类元信息的指针，这就是<font color=#FF69B4>Klass  Pointer</font>.

到这里我们就讲解了栈，程序计数器和方法区，下面我们简单介绍一下本地方法区，最后再终点讲解堆。

## 本地方法栈
实际上现在本地方法栈已经用的比较少了，大家应该都有听过本地方法吧

如何经常用的线程类

```java
new Thread().start();
public synchronized void start() {
        if (threadStatus != 0)
            throw new IllegalThreadStateException();
        group.add(this);
        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
            }
        }
    }
```



其中底层调用了一个<font color=#FF69B4>start0()</font>的方法

```java
private native void start0();
```



这个方法没有实现，但又不是接口，是使用native修饰的，是属于本地方法，底层通过C语言实现的，那java代码里为什么会有C语言实现的本地方法呢？

大家都知道JAVA是问世的，在那之前一个公司的系统百分之九十九都是使用C语言实现的，但是java出现后，很多项目都要转为java开发，那么新系统和旧系统就免不了要有交互，那么就需要本地方法来实现了，底层是调用C语言中的<font color=#FF69B4>dll</font>库文件，就类似于java中的jar包，当然，如今跨语言的交互方式就很多了，比如thrift，http接口方式，webservice等，当时并没有这些方式，就只能通过本地方法来实现了。

那么本地方法始终也是方法，每个线程在运行的时候，如果有运行到本地方法，那么必然也要产生局部变量等，那么就需要存储在本地方法栈了。如果没有本地方法，也就没有本地方法栈了。

## 堆
最后我们讲堆，堆是最重要的一块内存区域，我相信大部分人对堆都不陌生。但是对于它的内部结构，运作细节想要搞清楚也没那么简单。

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191126141228830.png)

对于这个基本组成大家应该都有所了解，对就是由<font color=#FF69B4>年轻代</font>和<font color=#87CEFF>老年代</font>组成，年轻代又分为<font color=#EE82EE>伊甸园区</font>和<font color=#90EE90>survivor</font>区，survivor区中又有<font color=#EEA2AD>from区</font>和<font color=#FF7256>to区</font>.

我们new出来的对象大家都知道是放在堆中，那具体放在堆中的哪个位置呢？

其实new出来的对象一般都放在<font color=#EE82EE>Eden区</font>，那么为什么叫伊甸园区呢，伊甸园就是亚当夏娃住的地方，不就是造人的地方么？所以我们new出来的对象就是放在这里的，那当Eden区满了之后呢？

假设我们给对分配600M内存，这个是可以通过参数调节的，我们后文再讲。那么<font color=#87CEFF>老年代</font>默认是占2/3的，也就是差不多400M，那<font color=#FF69B4>年轻代</font>就是200M，<font color=#EE82EE>Eden区</font>160M，<font color=#90EE90>Survivor区</font>40M。

## GC

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191126142038375.png)


一个程序只要在运行，那么就不会不停的new对象，那么总有一刻Eden区会放满，那么一旦Eden区被放满之后，虚拟机会干什么呢？没错，就是gc，不过这里的gc属于<font color=#EE82EE>minor gc</font>，就是垃圾收集，来收集垃圾对象并清理的，那么什么是垃圾对象呢？

好比我们上面说的math对象，我们假设我们是一个web应用程序，main线程执行完之后程序不会结束，但是main方法结束了，那么main()方法栈帧会被释放，局部变量会被释放，但是局部变量对应的堆中的对象还是依然存在的，但是又没有指针指向它，那么它就是一个垃圾对象，那就应该被回收掉了，之后如果还会new Math对象，也不会用这个之前的了，因为已经无法找到它了，如果留着这个对象只会占用内存，显然是不合适的。

这里就涉及到了一个<font color=#EE82EE>GC Root</font>根以及可达性分析算法的概念，也是面试偶尔会被问到的。

可达性分析算法是将GC Roots对象作为起点，从这些起点开始向下搜索引用的对象，找到的对象都标记为非垃圾对象，其余未标记的都是垃圾对象。

那么<font color=#EE82EE>GC Roots</font>根对象又是什么呢，<font color=#EE82EE>GC Roots</font>根就是判断一个对象是否可以回收的依据，只要能通过<font color=#EE82EE>GC Roots</font>根向下一直搜索能搜索到的对象，那么这个对象就不算垃圾对象，而可以作为<font color=#EE82EE>GC Roots</font>根的有线程栈的本地变量，静态变量，本地方法栈的变量等等，说白了就是找到和根节点有联系的对象就是有用的对象，其余都认为是垃圾对象来回收。

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191126145400951.png)

经历了第一次<font color=#EE82EE>minor gc</font>后，没有被清理的对象就会被移到<font color=#EEA2AD>From区</font>，如上图。

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191126145647900.png)

上面在说对象组成的时候有写到，在对象头的Mark Word中有存储<font color=#EEA2AD>GC分代年龄</font>，一个对象每经历一次gc，那么它的gc分代年龄就会<font color=red>+1</font>，如上图。

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191126151319582.png)

那么如果第二次新的对象又把Eden区放满了，那么又会执行<font color=#EE82EE>minor gc</font>，但是这次会连着From区一起gc，然后将<font color=#EE82EE>Eden区</font>和<font color=#EEA2AD>From区</font>存活的对象都移到<font color=#FF7256>To区域</font>，对象头中分代年龄都<font color=red>+1</font>，如上图。

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191126151616936.png)

那么当第三次Eden区又满的时候，<font color=#EE82EE>minor gc</font>就是回收<font color=#EE82EE>Eden区</font>和<font color=#FF7256>To区</font>域了，<font color=#EE82EE>Eden区</font>和<font color=#FF7256>To区</font>域还活着的对象就会都移到From区，如上图。说白了就是<font color=#90EE90>Survivor区</font>中总有一块区域是空着的，存活的对象存放是在<font color=#EEA2AD>From区</font>和<font color=#FF7256>To区</font>轮流存放，也就是互相复制拷贝，这也就是垃圾回收算法中的<font color=red>复制-回收算法</font>。

如果一个对象经历了一个限值15次gc的时候，就会移至老年代。那如果还没有到限值，From区或者To区域也放不下了，就会直接挪到老年代，这只是举例了两种常规规则，还有其他规则也是会把对象存放至老年代的。

那么随着应用程序的不断运行，老年代最终也是会满的，那么此时也会gc，此时的gc就是<font color=#FF7256>Full gc</font>了。

**GC案例**
下面我们通过一个简单的演示案例来更加清楚的了解GC。

```java
public class HeapTest {
    byte[] a = new byte[1024*100];
    public static void main(String[] args) throws InterruptedException {
        ArrayList<HeapTest> heapTest = new ArrayList<>();
        while(true) {
            heapTest.add(new HeapTest());
            Thread.sleep(10);
        }
    }
}
```



这块代码很明显，就是一个死循环，不断的往list中添加new出来的对象。

我们这里使用jdk自带的一个jvm调优工具<font color=#87CEFF>jvisualvm</font>来观察一下这个代码执行的的内存结构。

运行代码打开之后我们可以看到这样的界面：

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/2019112616032778.png)

我们在左边的应用程序中可以看到我们运行的这个代码，右边是它的一些jvm，内存信息，我们这里不关注，我们需要用到的是最后一个Visual GC面板，这是一个插件，如果你的打开没有这一栏的话，可以再工具栏的插件中进行下载安装。

 打开visual GC，我们先看一下界面大概的布局，

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/2019112616134712.png)

其中<font color=#87CEFF>老年代(Olc)</font>，<font color=#EE82EE>伊甸园区(Eden)</font>，<font color=#EEA2AD>S0(From)</font>，<font color=#FF7256>S1(To)</font>几个区域的内存和动态分配图都是清晰可见，以一对应的。

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191126164339736.png)

我们选择中间一张图给大家对应一下上面所讲的内容：

> 1：对象放入<font color=#EE82EE>Eden区</font>
>
> 2：<font color=#EE82EE>Eden区</font>满发生<font color=#EE82EE>minor gc</font>
>
> 3：第二步的存活对象移至<font color=#EEA2AD>From(Survivor 0)</font>区
>
> 4：<font color=#EE82EE>Eden区</font>再满发生<font color=#EE82EE>minor gc</font>
>
> 5：第四步存活的对象移至<font color=#FF7256>To(Survivor 1)</font>区

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191126171445401.png)

这里可以注意到<font color=#EEA2AD>From</font>和<font color=#FF7256>To区域</font>和我们上面所说移至，总有一个是空的。 

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/20191126164912890.png)

大家还可以注意到老年代这里，都是一段一段的直线，中间是突然的增加，这就是在<font color=#EE82EE>minor gc</font>中一批一批符合规则的对象被批量移入<font color=#87CEFF>老年代</font>。

 那当我们老年代满了会发生什么呢？当然是我们上面说过的<font color=red>Full GC</font>，但是你仔细看我们写的这个程序，我们所有new出来的HeapTest对象都是存放在heapLists中的，那就会被这个局部变量所引用，那么<font color=red>Full GC</font>就不会有什么垃圾对象可以回收，可是内存又满了，那怎么办？

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/2019112617183192.png)

没错，就是我们就算没见过也总听过的<font color=#87CEFF>OOM</font>。

到这里jvm内存模型简单介绍就结束了，看到这里还不点个赞嘛！ 



