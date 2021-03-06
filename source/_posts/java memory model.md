---

title: Java 内存模型
categories: java
tags: [内存模型] 
date: 2019-11-11
cover: https://gschaos.club/ico/img/20200409221112-hyejin-jo-3.jpg


---



在并发编程中，我们需要处理两个关键问题：线程之间如何通信及线程之间如何同步（这里的线程是指并发执行的活动实体）。通信是指线程之间以何种机制来交换信息。在命令式编程中，线程之间的通信机制有两种：共享内存和消息传递。



 <!-- more -->



> 作者：程晓明
>
> 出处:https://www.infoq.cn/article/java-memory-model-1

## <font color=  #8EE5EE  >并发编程模型的分类</font>

在并发编程中，我们需要处理两个关键问题：线程之间如何通信及线程之间如何同步（这里的线程是指并发执行的活动实体）。通信是指线程之间以何种机制来交换信息。在命令式编程中，线程之间的通信机制有两种：共享内存和消息传递。

在共享内存的并发模型里，线程之间共享程序的公共状态，线程之间通过写 - 读内存中的公共状态来隐式进行通信。在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过明确的发送消息来显式进行通信。

同步是指程序用于控制不同线程之间操作发生相对顺序的机制。在共享内存并发模型里，同步是显式进行的。程序员必须显式指定某个方法或某段代码需要在线程之间互斥执行。在消息传递的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是隐式进行的。

Java 的并发采用的是共享内存模型，Java 线程之间的通信总是隐式进行，整个通信过程对程序员完全透明。如果编写多线程程序的 Java 程序员不理解隐式进行的线程之间通信的工作机制，很可能会遇到各种奇怪的内存可见性问题。

## <font color=  #8EE5EE  >Java 内存模型的抽象</font>

在 java 中，所有实例域、静态域和数组元素存储在堆内存中，<font color= #FF1493    >堆内存在线程之间共享</font>（本文使用“共享变量”这个术语代指实例域，静态域和数组元素）。<font color= #FF1493    >局部变量（Local variables），方法定义参数（java 语言规范称之为 formal method parameters）和异常处理器参数（exception handler parameters）不会在线程之间共享，它们不会有内存可见性问题，也不受内存模型的影响</font>。

Java 线程之间的通信由 Java 内存模型（本文简称为 JMM）控制，JMM 决定一个线程对共享变量的写入何时对另一个线程可见。从抽象的角度来看，JMM 定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读 / 写共享变量的副本。本地内存是 JMM 的一个抽象概念，并不真实存在。它涵盖了<font color= #FF1493    >缓存</font>，<font color=#EE82EE >写缓冲区</font>，<font color= #8A2BE2 >寄存器</font>以及其他的硬件和编译器优化。Java 内存模型的抽象示意图如下：

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/b098a84eb7598d70913444a991d1759b.png)

从上图来看，线程 A 与线程 B 之间如要通信的话，必须要经历下面 2 个步骤：

1. 首先，线程 A 把本地内存 A 中更新过的共享变量刷新到主内存中去。
2. 然后，线程 B 到主内存中去读取线程 A 之前已更新过的共享变量。

下面通过示意图来说明这两个步骤：

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/2c452d147bf0d09b14b770d3990740cb.png)

如上图所示，本地内存 A 和 B 有主内存中共享变量 x 的副本。假设初始时，这三个内存中的 x 值都为 0。线程 A 在执行时，把更新后的 x 值（假设值为 1）临时存放在自己的本地内存 A 中。当线程 A 和线程 B 需要通信时，线程 A 首先会把自己本地内存中修改后的 x 值刷新到主内存中，此时主内存中的 x 值变为了 1。随后，线程 B 到主内存中去读取线程 A 更新后的 x 值，此时线程 B 的本地内存的 x 值也变为了 1。

从整体来看，这两个步骤实质上是线程 A 在向线程 B 发送消息，而且这个通信过程必须要经过主内存。JMM 通过控制主内存与每个线程的本地内存之间的交互，来为 java 程序员提供内存可见性保证。

## <font color=  #8EE5EE  >重排序</font>

在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序。重排序分三种类型：

1. 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
2. 指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-Level Parallelism， ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
3. 内存系统的重排序。由于处理器使用缓存和读 / 写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

从 java 源代码到最终实际执行的指令序列，会分别经历下面三种重排序：

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/9026b8f4b6c1fae4270615e0aadc7cdf.png)

上述的 1 属于编译器重排序，2 和 3 属于处理器重排序。这些重排序都可能会导致多线程程序出现内存可见性问题。对于编译器，JMM 的编译器重排序规则会禁止特定类型的编译器重排序（不是所有的编译器重排序都要禁止）。对于处理器重排序，JMM 的处理器重排序规则会要求 java 编译器在生成指令序列时，插入特定类型的内存屏障（memory barriers，intel 称之为 memory fence）指令，通过内存屏障指令来禁止特定类型的处理器重排序（不是所有的处理器重排序都要禁止）。

JMM 属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证。

## <font color=  #8EE5EE  >处理器重排序与内存屏障指令</font>

现代的处理器使用写缓冲区来临时保存向内存写入的数据。写缓冲区可以保证指令流水线持续运行，它可以避免由于处理器停顿下来等待向内存写入数据而产生的延迟。同时，通过以批处理的方式刷新写缓冲区，以及合并写缓冲区中对同一内存地址的多次写，可以减少对内存总线的占用。虽然写缓冲区有这么多好处，但每个处理器上的写缓冲区，仅仅对它所在的处理器可见。这个特性会对内存操作的执行顺序产生重要的影响：处理器对内存的读 / 写操作的执行顺序，不一定与内存实际发生的读 / 写操作顺序一致！为了具体说明，请看下面示例：

|       Processor A       |       Processor B       |
| :---------------------: | :---------------------: |
| a = 1; //A1 x = b; //A2 | b = 2; //B1 y = a; //B2 |

初始状态：a = b = 0 处理器允许执行后得到结果：x = y = 0

假设处理器 A 和处理器 B 按程序的顺序并行执行内存访问，最终却可能得到 x = y = 0 的结果。具体的原因如下图所示：

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/7004695e6bacd62e916f2f5ece3d7c9d.png)

这里处理器 A 和处理器 B 可以同时把共享变量写入自己的写缓冲区（A1，B1），然后从内存中读取另一个共享变量（A2，B2），最后才把自己写缓存区中保存的脏数据刷新到内存中（A3，B3）。当以这种时序执行时，程序就可以得到 x = y = 0 的结果。

从内存操作实际发生的顺序来看，直到处理器 A 执行 A3 来刷新自己的写缓存区，写操作 A1 才算真正执行了。虽然处理器 A 执行内存操作的顺序为：A1->A2，但内存操作实际发生的顺序却是：A2->A1。此时，处理器 A 的内存操作顺序被重排序了（处理器 B 的情况和处理器 A 一样，这里就不赘述了）。

这里的关键是，由于写缓冲区仅对自己的处理器可见，它会导致处理器执行内存操作的顺序可能会与内存实际的操作执行顺序不一致。由于现代的处理器都会使用写缓冲区，因此现代的处理器都会允许对写 - 读操做重排序。

下面是常见处理器允许的重排序类型的列表：

|           | Load-Load | Load-Store | Store-Store | Store-Load | 数据依赖 |
| :-------: | :-------: | :--------: | :---------: | :--------: | :------: |
| sparc-TSO |     N     |     N      |      N      |     Y      |    N     |
|    x86    |     N     |     N      |      N      |     Y      |    N     |
|   ia64    |     Y     |     Y      |      Y      |     Y      |    N     |
|  PowerPC  |     Y     |     Y      |      Y      |     Y      |    N     |

上表单元格中的“N”表示处理器不允许两个操作重排序，“Y”表示允许重排序。

从上表我们可以看出：常见的处理器都允许 Store-Load 重排序；常见的处理器都不允许对存在数据依赖的操作做重排序。sparc-TSO 和 x86 拥有相对较强的处理器内存模型，它们仅允许对写 - 读操作做重排序（因为它们都使用了写缓冲区）。

※注 1：sparc-TSO 是指以 TSO(Total Store Order) 内存模型运行时，sparc 处理器的特性。

※注 2：上表中的 x86 包括 x64 及 AMD64。

※注 3：由于 ARM 处理器的内存模型与 PowerPC 处理器的内存模型非常类似，本文将忽略它。

※注 4：数据依赖性后文会专门说明。

为了保证内存可见性，java 编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序。JMM 把内存屏障指令分为下列四类：

|      屏障类型       |          指令示例          |                             说明                             |
| :-----------------: | :------------------------: | :----------------------------------------------------------: |
|  LoadLoad Barriers  |   Load1; LoadLoad; Load2   | 确保 Load1 数据的装载，之前于 Load2 及所有后续装载指令的装载。 |
| StoreStore Barriers | Store1; StoreStore; Store2 | 确保 Store1 数据对其他处理器可见（刷新到内存），之前于 Store2 及所有后续存储指令的存储。 |
| LoadStore Barriers  |  Load1; LoadStore; Store2  | 确保 Load1 数据装载，之前于 Store2 及所有后续的存储指令刷新到内存。 |
| StoreLoad Barriers  |  Store1; StoreLoad; Load2  | 确保 Store1 数据对其他处理器变得可见（指刷新到内存），之前于 Load2 及所有后续装载指令的装载。StoreLoad Barriers 会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令。 |

StoreLoad Barriers 是一个“全能型”的屏障，它同时具有其他三个屏障的效果。现代的多处理器大都支持该屏障（其他类型的屏障不一定被所有处理器支持）。执行该屏障开销会很昂贵，因为当前处理器通常要把写缓冲区中的数据全部刷新到内存中（buffer fully flush）。

## <font color=  #8EE5EE  >happens-before</font>

从 JDK5 开始，java 使用新的 JSR -133 内存模型（本文除非特别说明，针对的都是 JSR- 133 内存模型）。JSR-133 提出了 happens-before 的概念，通过这个概念来阐述操作之间的内存可见性。如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在 happens-before 关系。这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。 与程序员密切相关的 happens-before 规则如下：

- 程序顺序规则：一个线程中的每个操作，happens- before 于该线程中的任意后续操作。
- 监视器锁规则：对一个监视器锁的解锁，happens- before 于随后对这个监视器锁的加锁。
- volatile 变量规则：对一个 volatile 域的写，happens- before 于任意后续对这个 volatile 域的读。
- 传递性：如果 A happens- before B，且 B happens- before C，那么 A happens- before C。

注意，两个操作之间具有 happens-before 关系，并不意味着前一个操作必须要在后一个操作之前执行！happens-before 仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前（the first is visible to and ordered before the second）。happens- before 的定义很微妙，后文会具体说明 happens-before 为什么要这么定义。

happens-before 与 JMM 的关系如下图所示：

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/bc22eaae1a77f9e1a6c09f4b6a833163.png)

如上图所示，一个 happens-before 规则通常对应于多个编译器重排序规则和处理器重排序规则。对于 java 程序员来说，happens-before 规则简单易懂，它避免程序员为了理解 JMM 提供的内存可见性保证而去学习复杂的重排序规则以及这些规则的具体实现。

## 参考文献

1. [Programming Language Pragmatics, Third Edition](http://www.amazon.com/Programming-Language-Pragmatics-Third-Michael/dp/0123745144/ref=sr_1_1?s=books&ie=UTF8&qid=1347794595&sr=1-1&keywords=Michael+L.Scott)
2. [The Java Language Specification, Third Edition](http://docs.oracle.com/javase/specs/jls/se5.0/jls3.pdf)
3. [JSR-133: Java Memory Model and Thread Specification](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr133.pdf)
4. [Java theory and practice: Fixing the Java Memory Model, Part 2](http://www.ibm.com/developerworks/java/library/j-jtp03304/index.html)
5. [Understanding POWER Multiprocessors](http://www.cl.cam.ac.uk/~pes20/ppc-supplemental/pldi105-sarkar.pdf)
6. [Concurrent Programming on Windows](http://www.amazon.com/Concurrent-Programming-Windows-Joe-Duffy/dp/032143482X/ref=sr_1_1?ie=UTF8&s=books&qid=1262571776&sr=1-1)
7. [The Art of Multiprocessor Programming](http://www.amazon.com/Art-Multiprocessor-Programming-Maurice-Herlihy/dp/0123705916/ref=sr_1_1?ie=UTF8&s=books&qid=1248834374&sr=1-1)
8. [Intel® 64 and IA-32 ArchitecturesvSoftware Developer’s Manual Volume 3A: System Programming Guide, Part 1](http://download.intel.com/products/processor/manual/253668.pdf)
9. [Java Concurrency in Practice](http://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601/ref=pd_sim_b_1)
10. [The JSR-133 Cookbook for Compiler Writers](http://gee.cs.oswego.edu/dl/jmm/cookbook.html)

## 关于作者

**程晓明**，Java 软件工程师，国家认证的系统分析师、信息项目管理师。专注于并发编程，就职于富士通南大。个人邮箱：[asst2003@163.com](mailto:asst2003@163.com)。