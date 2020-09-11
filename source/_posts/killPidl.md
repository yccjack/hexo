---
title: 杀死进程
categories: linux
tags: [linux,shell] 
date: 2020-09-07
cover: https://mysticalyu.gitee.io/pic/img/wayne-wan-(4).jpg
---

![](https://mysticalyu.gitee.io/pic/img/wayne-wan-(4).jpg)



 <!-- more -->



# linux 查看某进程 并杀死进程 ps grep kill

[Linux](http://lib.csdn.net/base/linux) 中使用top 或 ps 查看进程使用kill杀死进程

1.使用top查看进程：

$top

进行执行如上命令即可查看top！但是难点在如何以进程的cpu占用量进行排序呢？

cpu占用量排序执行下操作：

按大写O再按k再敲回车，然后使用R就可以以cpu占用量进行查看了！下面贴出top的技巧命令：

“更改显示内容
通过 f 键可以选择显示的内容。按 f 键之后会显示列的列表，按 a-z 即可显示或隐藏对应的列，最后按回车键确定。

按 o 键可以改变列的显示顺序。按小写的 a-z 可以将相应的列向右移动，而大写的 A-Z 可以将相应的列向左移动。最后按回车键确定。

按大写的 F 或 O 键，然后按 a-z 可以将进程按照相应的列进行排序。而大写的 R 键可以将当前的排序倒转。”

然后还是顶部一参数的含义：

“ 150 total 进程总数
2 running 正在运行的进程数
148 sleeping 睡眠的进程数
0 stopped 停止的进程数
0 zombie 僵尸进程数
Cpu0: 67.4% us 用户空间占用CPU百分比
2.0% sy 内核空间占用CPU百分比
0.0% ni 用户进程空间内改变过优先级的进程占用CPU百分比
30.2% id 空闲CPU百分比
0.0% wa 等待输入输出的CPU时间百分比
0.0% hi
0.0% si
0.0% st


进程信息区
统计信息区域的下方显示了各个进程的详细信息。首先来认识一下各列的含义。

序号 列名 含义
a PID 进程id
b PPID 父进程id
c RUSER Real user name
d UID 进程所有者的用户id
e USER 进程所有者的用户名
f GROUP 进程所有者的组名
g TTY 启动进程的终端名。不是从终端启动的进程则显示为 ?
h PR 优先级
i NI nice值。负值表示高优先级，正值表示低优先级
j P 最后使用的CPU，仅在多CPU环境下有意义
k %CPU 上次更新到现在的CPU时间占用百分比
l TIME 进程使用的CPU时间总计，单位秒
m TIME+ 进程使用的CPU时间总计，单位1/100秒
n %MEM 进程使用的物理内存百分比
o VIRT 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
p SWAP 进程使用的虚拟内存中，被换出的大小，单位kb。
q RES 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
r CODE 可执行代码占用的物理内存大小，单位kb
s DATA 可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb
t SHR 共享内存大小，单位kb
u nFLT 页面错误次数
v nDRT 最后一次写入到现在，被修改过的页面数。
w S 进程状态。
D=不可中断的睡眠状态
R=运行
S=睡眠
T=跟踪/停止
Z=僵尸进程
x COMMAND 命令名/命令行
y WCHAN 若该进程在睡眠，则显示睡眠中的系统函数名
”

2.使用ps命令查看进程

$ ps -ef

……
smx    1822   1 0 11:38 ?    00:00:49 gnome-terminal
smx    1823 1822 0 11:38 ?    00:00:00 gnome-pty-helper
smx    1824 1822 0 11:38 pts/0  00:00:02 bash
smx    1827   1 4 11:38 ?    00:26:28 /usr/lib/firefox-3.6.18/firefox-bin
smx    1857 1822 0 11:38 pts/1  00:00:00 bash
smx    1880 1619 0 11:38 ?    00:00:00 update-notifier
……
smx   11946 1824 0 21:41 pts/0  00:00:00 ps -ef

或者：

$ ps -aux

……

smx    1822 0.1 0.8 58484 18152 ?    Sl  11:38  0:49 gnome-terminal
smx    1823 0.0 0.0  1988  712 ?    S  11:38  0:00 gnome-pty-helper
smx    1824 0.0 0.1  6820 3776 pts/0  Ss  11:38  0:02 bash
smx    1827 4.3 5.8 398196 119568 ?    Sl  11:38 26:13 /usr/lib/firefox-3.6.18/firefox-bin
smx    1857 0.0 0.1  6688 3644 pts/1  Ss  11:38  0:00 bash
smx    1880 0.0 0.6 41536 12620 ?    S  11:38  0:00 update-notifier
……
smx   11953 0.0 0.0  2716 1064 pts/0  R+  21:42  0:00 ps -aux

3.下面演示如何杀死进程

此时如果我想杀了火狐的进程就在终端输入：

$ kill -s 9 1827

其中-s 9 制定了传递给进程的信号是９，即强制、尽快终止进程。各个终止信号及其作用见附录。

1827则是上面ps查到的火狐的PID。

简单吧，但有个问题，进程少了则无所谓，进程多了，就会觉得痛苦了，无论是ps -ef 还是ps -aux，每次都要在一大串进程信息里面查找到要杀的进程，看的眼都花了。

进阶篇：

改进１：

把ps的查询结果通过管道给grep查找包含特定字符串的进程。管道符“|”用来隔开两个命令，管道符左边命令的输出会作为管道符右边命令的输入。

$ ps -ef | grep firefox
smx    1827   1 4 11:38 ?    00:27:33 /usr/lib/firefox-3.6.18/firefox-bin
smx   12029 1824 0 21:54 pts/0  00:00:00 grep --color=auto firefox

这次就清爽了。然后就是

$kill -s 9 1827

还是嫌打字多？

改进２——使用pgrep：

一看到pgrep首先会想到什么？没错，grep！pgrep的p表明了这个命令是专门用于进程查询的grep。

$ pgrep firefox
1827

看到了什么？没错火狐的PID，接下来又要打字了：

$kill -s 9 1827


改进３——使用pidof：

看到pidof想到啥？没错pid of xx，字面翻译过来就是 xx的PID。

$ pidof firefox-bin
1827
和pgrep相比稍显不足的是，pidof必须给出进程的全名。然后就是老生常谈：


$kill -s 9 1827

无论使用ps 然后慢慢查找进程PID 还是用grep查找包含相应字符串的进程，亦或者用pgrep直接查找包含相应字符串的进程ＰＩＤ，然后手动输入给ｋｉｌｌ杀掉，都稍显麻烦。有没有更方便的方法？有！

改进４：


$ps -ef | grep firefox | grep -v grep | cut -c 9-15 | xargs kill -s 9


说明：

“grep firefox”的输出结果是，所有含有关键字“firefox”的进程。

“grep -v grep”是在列出的进程中去除含有关键字“grep”的进程。

“cut -c 9-15”是截取输入行的第9个字符到第15个字符，而这正好是进程号PID。

“xargs kill -s 9”中的xargs命令是用来把前面命令的输出结果（PID）作为“kill -s 9”命令的参数，并执行该命令。“kill -s 9”会强行杀掉指定进程。

难道你不想抱怨点什么？没错太长了

改进５：

知道pgrep和pidof两个命令，干嘛还要打那么长一串！


$ pgrep firefox | xargs kill -s 9

改进６：


$ ps -ef | grep firefox | awk '{print $2}' | xargs kill -9
kill: No such process


有一个比较郁闷的地方，进程已经正确找到并且终止了，但是执行完却提示找不到进程。

其中awk '{print $2}' 的作用就是打印（print）出第二列的内容。根据常规篇，可以知道ps输出的第二列正好是PID。就把进程相应的PID通过xargs传递给kill作参数，杀掉对应的进程。

改进７：


难道每次都要调用xargs把PID传递给kill？答案是否定的：

$kill -s 9 `ps -aux | grep firefox | awk '{print $2}'`

改进８：

没错，命令依然有点长，换成pgrep。


$kill -s 9 `pgrep firefox`

改进9——pkill：

看到pkill想到了什么？没错pgrep和kill！pkill＝pgrep+kill。

$pkill -９ firefox

说明："-9" 即发送的信号是9，pkill与kill在这点的差别是：pkill无须 “ｓ”，终止信号等级直接跟在 “-“ 后面。之前我一直以为是 "-s 9"，结果每次运行都无法终止进程。

改进10——killall：

killall和pkill是相似的,不过如果给出的进程名不完整，killall会报错。pkill或者pgrep只要给出进程名的一部分就可以终止进程。

$killall -9 firefox

OK,讲到这里大家应该了解了吧！