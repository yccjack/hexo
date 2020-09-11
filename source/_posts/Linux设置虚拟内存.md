---
title: Linux设置虚拟内存
categories: 服务器
tags: [java,服务器,内存] 
date: 2020-08-28
cover: https://mysticalyu.gitee.io/pic/img/20200605013054948.jpg
description: 
---

![img](https://mysticalyu.gitee.io/pic/img/a-i-20.jpg)

在我们自己的购买的服务器环境中，一般是买的1g的内存，但是当服务器里面的东西装的比较多的时候就会导致内存不够用了,这个时候可以通过增加虚拟内存来夸大内存容量。



 <!-- more -->



# Linux设置虚拟内存

## 交换技术

交换（Swapping）技术它的主要特点是：打破了一个程序一旦进入内存，就一直驻留在内存直到运行结束的限制。

在多道程序环境下，内存中可以同时存在多个进程（程序），其中的一部分进程由于等待某些事件而处于阻塞状态，但这些处于阻塞状态的进程仍然驻留内存，并占据着内存空间；另一方面，外存上可能有许多等待装入内存运行的程序，却因内存不足而未能装入。显然，这是一种严重的系统资源浪费，它会使系统的吞吐量下降。为了解决这个问题，可以在操作系统中增加交换（对换）功能，即由操作系统根据需要，将内存中暂时不具备运行条件的部分程序或数据移到外存（换出），以便腾出足够的内存空间，将外存中需要运行的程序或数据调入内存（换入）投入运行。在操作系统中引入交换（对换）技术，可以显著提高内存资源的利用率并改善系统的性能。

以交换的单位不同来划分，则有以下两种交换方式。

- 以进程为单位的交换。每次换入/换出的是整个进程，我们称这种交换为进程交换（进程对换）或整体交换（整体对换）。进程交换广泛应用于分时系统，主要解决内存紧张问题。

- 以页（此处不多做介绍）或段（此处不多做介绍）为单位的交换。这种交换分别称为页置换（页交换或页对换）或段置换（段交换或段对换），页置换和段置换是以进程中的某一部分为交换单位，因此又称为部分交换（部分对换）。部分交换广泛应用于现代操作系统中，是实现虚拟存储器的基础。

我们这里所说的交换是指进程交换，为了实现进程交换，操作系统需要解决以下两个问题。

- 对换空间的管理。在具有交换功能的操作系统中，一般将外存空间分为文件区和交换区（对换区）。文件区用来存放文件，而交换区则用来存放从内存中换出的进程，或等待换入内存的进程。尽管文件区一般采用离散分配方式来分配外存存储空间，但交换区的存储空间分配则宜采用连续分配方式，这是因为交换区中存放的是换入/换出的进程，为了提高交换速度，有必要采用连续分配方式，并且交换区可以采用与可变分区存储管理类似的方法进行管理。例如，使用空闲分区表或空闲分区链来记录外存交换区的使用情况，利用首次适应算法、最佳适应算法或最差适应算法来进行外存交换区的分配。

- 交换的时机以及选择哪些进程交换。交换时机一般选择在进程的时间片用完，以及进程等待输入/输出时，或者在进程要求扩充其内存空间而得不到满足时。换出到外存的进程一般选择处于阻塞状态，或优先级低且短时间内不会再次投入运行的进程；换入到内存的进程则应选择换出时间最久且已处于就绪状态的进程。

**《操作系统原理》**



## 介绍

在我们自己的购买的服务器环境中，一般是买的1g的内存，但是当服务器里面的东西装的比较多的时候就会导致内存不够用了

## 创建swap文件

1. 进入`/usr`目录

```bash
[root@localhost usr]$ pwd
/usr
[root@localhost usr]$ 

```

1. 创建`swap`文件夹,并进入该文件夹

```bash
[root@localhost usr]# mkdir swap
[root@localhost usr]# cd swap/
[root@localhost swap]# pwd
/usr/swap
[root@localhost swap]# 

```

1. 创建`swapfile`文件,使用命令`dd if=/dev/zero of=/usr/swap/swapfile bs=1M count=4096`

```bash
[root@localhost swap]# dd if=/dev/zero of=/usr/swap/swapfile bs=1M count=4096
记录了4096+0 的读入
记录了4096+0 的写出
4294967296字节(4.3 GB)已复制，15.7479 秒，273 MB/秒
[root@localhost swap]#
```

## 查看swap文件

1. 使用命令`du -sh /usr/swap/swapfile`,可以看到我们创建的这个swap文件为4g

```bash
[root@localhost swap]# du -sh /usr/swap/swapfile
4.1G	/usr/swap/swapfile
[root@localhost swap]# 
```

## 将目标设置为swap分区文件

1. 使用命令`mkswap /usr/swap/swapfile`将swapfile文件设置为swap分区文件

```bash
[root@localhost swap]# mkswap /usr/swap/swapfile
mkswap: /usr/swap/swapfile: warning: don't erase bootbits sectors
        on whole disk. Use -f to force.
Setting up swapspace version 1, size = 4194300 KiB
no label, UUID=5bd241ff-5375-449d-9975-5fdd429df784
[root@localhost swap]#
```

## 激活swap区，并立即启用交换区文件

1. 使用命令`swapon /usr/swap/swapfile`

```bash
[root@localhost swap]# swapon /usr/swap/swapfile
[root@localhost swap]#
```

1. 使用命令`free -m` 来查看现在的内存,可以看到里面的Swap分区变成了4095M，也就是4G内存。

```bash
[root@localhost swap]# free -m
             total       used       free     shared    buffers     cached
Mem:           980        910         70          3          8        575
-/+ buffers/cache:        326        654
Swap:         4095          0       4095
[root@localhost swap]#
```

## 设置开机自动启用虚拟内存，在`etc/fstab`文件中加入如下命令

1. 使用vim编辑器打开/etc/fstab文件
2. 在文件中加入如下内容

```bash
/usr/swap/swapfile swap swap defaults 0 0
```

## 使用reboot命令重启服务器

1. 输入`reboot` 命令来重启

```bash
	[root@localhost swap]# reboot

	Broadcast message from liaocheng@localhost.localdomain
		(/dev/pts/1) at 3:56 ...

	The system is going down for reboot NOW!
	[root@localhost swap]# Connection to 192.168.136.142 closed by remote host.
	Connection to 192.168.136.142 closed.
	[进程已完成]
```

1. 重启完成过后使用free -m 命令来查看现在的内存是否挂在上了。

```bash
[root@localhost swap]# free -m
             total       used       free     shared    buffers     cached
Mem:           980        910         70          3          8        575
-/+ buffers/cache:        326        654
Swap:         4095          0       4095
```



