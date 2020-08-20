---

title: 傅里叶变换
categories:  傅里叶变换
tags: [数学] 
description: 傅里叶变换，拉普斯变换，z变换

---

作者：DBinary



![img](https://pic4.zhimg.com/80/v2-441dc7886c2d1651975a8d8dec827526_hd.jpg)

   作为一个资深信号狗,必须强答一波这个问题,想当年也是被一堆变换公式折磨的要死要活的,多年过去了,用的多了发现也就是那么回事,尽管其内部的数学推论是复杂的(其实也就那样),但真的要说,仍然可以用最简单的几句话和最通俗易懂的语言把它的原理和作用讲清楚.

   既然要讲,我就从最基础的东西开始说一说,首先我们先来认识下三角函数,要说三角函数这个东西,我们首先要来说说弧度,什么是弧度呢,你可以在纸上画一个圆,选取圆的一段边,边长和这个圆半径的比值,就是该边与圆心对应夹角的弧度,不好理解是不是,没关系,看个图你就懂了

 ![img](https://pic2.zhimg.com/50/v2-3f115013d83be51c9a9ad1c4006609c6_hd.webp) 

  弧度的单位是rad,你会发现,所有的圆边长和半径的比值都是2πRad,而π是一个无限不循环的常数,它约等于3.1415926,可以发现弧度和角度是一个对应的关系,如果按角度制而言绕圆一周是360°,弧度制而言,就是2π了

 现在,我们引入另一个在信号处理中极为极为极为重要的一个函数,三角函数,之所以叫做三角函数,是因为它的计算方式和直角三角密切相关

![img](https://pic2.zhimg.com/50/v2-71c4df5642e4123c6a024cb4ae0f21ee_hd.jpg)![img](https://pic2.zhimg.com/80/v2-71c4df5642e4123c6a024cb4ae0f21ee_hd.jpg)

三角函数又常常叫正弦函数常用的主要有sin和cos两种,在高中的书本上,常常叫它们正弦函数和余弦函数,但实际在使用中,不管是sin还是cos都常常被统称为正弦函数,看上面的直角三角形, 以sin函数为例,关于这个函数的求法,可以用下面的公式来表述

![[公式]](https://www.zhihu.com/equation?tex=sin%28%5Cangle+A%29%3D%5Cfrac%7Ba%7D%7Bc%7D)

也就是说sin角a的值,等于其对应直角三角形的对边比斜边,实际上我们常常用 ![[公式]](https://www.zhihu.com/equation?tex=sin%28x%29) 来表示这个正弦函数,而 ![[公式]](https://www.zhihu.com/equation?tex=x) 则表示某一弧度,如果你把这个三角形画在一个二维坐标系的圆上面,比如下面的这种形式

![img](https://pic2.zhimg.com/50/v2-4de7aeea265dfeb1b10597005e3802a7_hd.jpg)![img](https://pic2.zhimg.com/80/v2-4de7aeea265dfeb1b10597005e3802a7_hd.jpg)

那么 ![[公式]](https://www.zhihu.com/equation?tex=sin%28%5Ctheta%29%3D%5Cfrac%7By%7D%7Br%7D) ,当然,这个时候,正弦值还仅仅是一个"正弦值",现在你可以开始想象假如圆上的这个点现在开始动了起来,并开始绕圆逆时针旋转, ![[公式]](https://www.zhihu.com/equation?tex=sin%28%5Ctheta%29) 的值会如何变化呢?下面的图会告诉你答案

 ![img](https://pic2.zhimg.com/50/v2-bad99139a4a97a51fcb01ad36e9fa5c4_hd.webp) 





显然的,当我们引入动态的概念后,正弦函数随之而动,从一个定值变成了一个波,在信号处理中,我们称之为正弦波,高中的课本会告诉你正弦函数的性质和和差化积积化和差之类的公式,而我会告诉你正弦函数和其所对应的正弦波估计是信号处理中最重要最常用没有之一的重要工具

到这里既然我们说到波了,那么就不得不提几个问题和其对应的概念,现在你再看看上图,如果我们需要描述一个正弦波是不是需要下面几个问题,而这个问题的答案,对应了几个概念

1. 这个点围绕的圆到底有多大---->波幅
2. 这个点旋转的速度有多快---->角速度--->频率
3. 这个点最初的位置在哪里---->相位

当然,如果我们描述正弦波只能用上面的文字来说,未免显得不够专业,于是乎,我们用一个更加通用的公式来描述正弦波

![[公式]](https://www.zhihu.com/equation?tex=f%28t%29%3DAsin%28%5Comega+t%2B%5Cvarphi%29%2Bk)

其中,A表示振福,A越大,振福越大.

![[公式]](https://www.zhihu.com/equation?tex=w) 表示角速度,当然,角速度和频率 ![[公式]](https://www.zhihu.com/equation?tex=f) 是对应关系,所以信号处理中常常也用角频率这种俗语来描述

![[公式]](https://www.zhihu.com/equation?tex=%5Cvarphi) 表示相位,sin和cos两个正弦函数的差别其实也仅仅是相位不同

![[公式]](https://www.zhihu.com/equation?tex=k) 是这个正弦波的偏移,你可以理解为这个波在y轴上如何的上下移动,在信号处理中,这个会被统一到直流分量中(频率为0的波的波幅)

科普完上面的概念之后,要说傅里叶变换是怎么回事其实已经很容易了,现在我们来看傅里叶说过的一句话

***“任何”周期信号都可以用一系列成谐波关系的正弦曲线来表示。\***

我们先不讨论这句话的适用条件(狄里赫利条件),这句话简直牛逼大了,这表示下面这些信号

![img](https://pic2.zhimg.com/50/v2-e42e6174a024074fe086d57cb2c6e42e_hd.jpg)![img](https://pic2.zhimg.com/80/v2-e42e6174a024074fe086d57cb2c6e42e_hd.jpg)

全部可以用下面这个式子来表示

![[公式]](https://www.zhihu.com/equation?tex=f%5Cleft%28+t+%5Cright%29+%3D+c_%7B0%7D+%2B+%5Csum_%7Bn+%3D+1%7D%5E%7B%5Cinfty%7D%7Bc_%7Bn%7D%5Ccos%7D%28n%5Comega+t+%2B+%5Cvarphi%29+)      (式1.0)

如果看不明白没关系,下面这张图能让你看个清楚,如何用正弦波组成一个近似的方波

 ![img](https://pic3.zhimg.com/50/v2-bb1427097bb4a91d4a78e384641ab8fa_hd.webp) 

图像来自wiki百科

***那么,有什么意义呢,要知道,如果可以将信号分解为正弦函数的累加和,不就等于知道了这个信号是由哪些频率的正弦波构成了的么,同时,我们还能知道对应频率的波在信号中的能量和相位信息.\***

举个很简单的声学例子,如果我们直接看一段声音信号的波形图,我们很难看出他是男声还是女声(别说男声的嗓门比较大波幅宽,河东狮吼了解下)但是从频域中我们就能够很容易分辨出来,毕竟女声的频域中,高频的能量占比会比较高

再举个很简单的图形学例子,如果把一张图像做频域分析,图像的低频代表着轮廓信息,高频代表着细节信息,相位代表位置信息,你要是想让图像变模糊,简单,把高频的能量压下来就行了,想让图像变尖锐,高频能量加上去就行了.

那么问题又来了,已知 ![[公式]](https://www.zhihu.com/equation?tex=f%28t%29) ,我们如何把它分解为 ![[公式]](https://www.zhihu.com/equation?tex=%7Bc_%7B1%7D%5Ccos%7D%28%5Comega+t+%2B+%5Cvarphi%29%2B%7Bc_%7B2%7D%5Ccos%7D%282%5Comega+t+%2B+%5Cvarphi%29%2B%7Bc_%7B3%7D%5Ccos%7D%283%5Comega+t+%2B+%5Cvarphi%29.....)

的形式呢,实际上傅里叶变换需要解决的就是这一点,它的最终目的就是要将信号分解为上面这样的形式,好让我们把别通频率的正弦波信息给剥离出来

要说这个,我们就不得不谈谈三角函数的正交性了,

***首先我们知道,对正弦波正无穷到负无穷内进行积分,其结果必定是0(主值积分，取周期)\***

所以根据三角函数的积化和差公式,下面的推论都是成立的

![[公式]](https://www.zhihu.com/equation?tex=%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7B%5Ccos%28+%5Ctext%7Bmx%7D+%29%5Ccos%28+%5Ctext%7Bnx%7D+%29%7Ddx+%3D+%5Cfrac%7B1%7D%7B2%7D%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7B%7B+%5Ccos%5Clbrack+%28+m+-+n+%29x+%5Crbrack+%2B+%5Ccos%5Clbrack+%28+m+%2B+n+%29x+%5Crbrack+%7D%5Ctext%7Bdx%7D%7D+%3D+%5Cfrac%7B1%7D%7B2%7D%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7B%5Ccos%5Clbrack+%28+m+-+n+%29x+%5Crbrack%5Ctext%7Bdx%7D%7D+%2B+%5Cfrac%7B1%7D%7B2%7D%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7B%5Ccos%5Clbrack+%28+m+%2B+n+%29x+%5Crbrack%5Ctext%7Bdx%7D%7D+%3D+0+)

![[公式]](https://www.zhihu.com/equation?tex=%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7B%5Csin%28+%5Ctext%7Bmx%7D+%29%5Csin%28+%5Ctext%7Bnx%7D+%29%7Ddx+%3D+%5Cfrac%7B1%7D%7B2%7D%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7B%7B+%5Ccos%5Clbrack+%28+m+-+n+%29x+%5Crbrack+-+%5Ccos%5Clbrack+%28+m+%2B+n+%29x+%5Crbrack+%7D%5Ctext%7Bdx%7D%7D%3D%5Cfrac%7B1%7D%7B2%7D%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7B%5Ccos%5Clbrack+%28+m+-+n+%29x+%5Crbrack%5Ctext%7Bdx%7D%7D+-+%5Cfrac%7B1%7D%7B2%7D%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7B%5Ccos%5Clbrack+%28+m+%2B+n+%29x+%5Crbrack%5Ctext%7Bdx%7D%7D+%3D+0)

![[公式]](https://www.zhihu.com/equation?tex=%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7B%5Csin%28+%5Ctext%7Bmx%7D+%29%5Ccos%28+%5Ctext%7Bnx%7D+%29%7Ddx+%3D+%5Cfrac%7B1%7D%7B2%7D%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7B%7B+%5Csin%5Clbrack+%28+m+-+n+%29x+%5Crbrack+%2B+%5Csin%5Clbrack+%28+m+%2B+n+%29x+%5Crbrack+%7D%5Ctext%7Bdx%7D%7D+%3D+%5Cfrac%7B1%7D%7B2%7D%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7B%5Csin%5Clbrack+%28+m+-+n+%29x+%5Crbrack%5Ctext%7Bdx%7D%7D+%2B+%5Cfrac%7B1%7D%7B2%7D%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7B%5Csin%5Clbrack+%28+m+%2B+n+%29+%5Crbrack%5Ctext%7Bdx%7D%7D+)

这导致了一个很重要的概念

***不同频率的正弦波相乘,对其周期积分后,其结果是0!\***


=======================================================

*(我知道有人肯定会说,作者你胡说八道,* ![[公式]](https://www.zhihu.com/equation?tex=%5Cint_%7B-%5Cinfty%7D%5E%7B%5Cinfty%7Dsin%28nx%29dx) *怎么会是0,老师告诉我它明明是发散的,你又忽悠我,关于这点我要说明一下,首先你的老师没说错,不过我也没有忽悠你,首先在大学高数求极限那些知识中,这个函数确实积分后是发散的,这个发散的具体原因是建立在* ![[公式]](https://www.zhihu.com/equation?tex=%5Clim_%7BM+%5Crightarrow+%5Cinfty+%2C+N+%5Crightarrow+-%5Cinfty+%7D%7B%5Cint_%7BN%7D%5E%7BM%7Dsin%28nx%29dx%7D+) *这种情况下的,也就是我们正常说的无穷积分,但是如果按这种玩法,基本上大半的信号处理函数都没法玩了,因此在信号处理的公式中比如傅里叶变换,默认都以柯西主值积分作为钦定的积分方式,打个比方定义* ![[公式]](https://www.zhihu.com/equation?tex=%5Clim_%7BM+%5Crightarrow+%5Cinfty+%7D%7B%5Cint_%7B-M%7D%5E%7BM%7Dsin%28nx%29dx%7D+)*,* ![[公式]](https://www.zhihu.com/equation?tex=sin%28nx%29)*这种情况下,负无穷到正无穷的积分不就是0了么,所以这里我说明一下,傅里叶变换中使用的是柯西主值积分,整个无穷区间取周期倍)*

*=======================================================*

这个概念我们又叫做波的相干性,比如给你一段信号,问你信号里有没有100HZ频率的正弦波信号,怎么办?简单,把这个信号和100hz的正弦波信号相乘,然后对其周期内积分,如果结果不是0,那么这个信号就含有100HZ的信号

那么剩下的问题就是如何求得该频率正弦波对应的幅度和相位了,实际上就是求式1.0的 ![[公式]](https://www.zhihu.com/equation?tex=c_%7Bn%7D) 和 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvarphi) 下面我要甩点公式了,如果感到不适,可以选择跳过

利用三角函数的变换公式，(式1.0)可变形为

![[公式]](https://www.zhihu.com/equation?tex=f%5Cleft%28+t+%5Cright%29+%3D+c_%7B0%7D+%2B+%5Csum_%7Bn+%3D+1%7D%5E%7B%5Cinfty%7D%7B%5Clbrack+c_%7Bn%7Dcos%5Cvarphi+cos%28n%5Comega+t%29+-+c_%7Bn%7Dsin%5Cvarphi+sin%28n%5Comega+t%29%5Crbrack%7D+)

设![[公式]](https://www.zhihu.com/equation?tex=+a_%7Bn%7D+%3D+c_%7Bn%7D%5Ctext%7Bcos%CF%86%7D+) ,![[公式]](https://www.zhihu.com/equation?tex=+b_%7Bn%7D+%3D+-+c_%7Bn%7D%5Ctext%7Bsin%CF%86%7D+) 那么，上式变为

![[公式]](https://www.zhihu.com/equation?tex=f%5Cleft%28+t+%5Cright%29+%3D+c_%7B0%7D+%2B+%5Csum_%7Bn+%3D+1%7D%5E%7B%5Cinfty%7D%7B%5Clbrack+a_%7Bn%7D%5Ccos%5Cleft%28+%5Ctext%7Bn%CF%89t%7D+%5Cright%29+%2B+b_%7Bn%7D%5Csin%5Cleft%28+%5Ctext%7Bn%CF%89t%7D+%5Cright%29%5Crbrack%7D+)

现在，让我们正式的引入正交性的性质，还记得检波手段么，这里，我们假设对 ![[公式]](https://www.zhihu.com/equation?tex=f%28t%29) 用![[公式]](https://www.zhihu.com/equation?tex=sin%28k%5Ctext%7B%CF%89t%7D%29) 进行检波(说人话就是乘起来,然后为了方便计算对其在一个周期内积分)，那么就有

![[公式]](https://www.zhihu.com/equation?tex=%5Cint_%7B0%7D%5E%7BT%7D%7Bf%5Cleft%28+t+%5Cright%29%5Csin%5Cleft%28+%5Ctext%7Bk%CF%89t%7D+%5Cright%29%5Ctext%7Bdt%7D%7D+)

假设f(x)中含有 ![[公式]](https://www.zhihu.com/equation?tex=%5Ctext%7Bn%CF%89%7D) 角频率的正弦波系数为 ![[公式]](https://www.zhihu.com/equation?tex=b_%7Bn%7D) ，那么根据三角函数的正交性，上式就有![[公式]](https://www.zhihu.com/equation?tex=%5Cint_%7B0%7D%5E%7BT%7D%7Bf%5Cleft%28+t+%5Cright%29%5Csin%5Cleft%28+%5Ctext%7Bn%CF%89t%7D+%5Cright%29%5Ctext%7Bdt%7D%7D%3Db_%7Bn%7D%5Cint_%7B0%7D%5E%7BT%7D%7B%5Csin%7B%5Cleft%28+%5Ctext%7Bn%CF%89t%7D+%5Cright%29%5Csin%5Cleft%28+k%5Ctext%7B%CF%89t%7D+%5Cright%29%7D%5Ctext%7Bdt%7D%7D)

为什么会这样?你想啊,别的频率的波积分后全变0了,不就是剩下( ![[公式]](https://www.zhihu.com/equation?tex=k%3Dn) )频率一样的情况了么.因此

![[公式]](https://www.zhihu.com/equation?tex=%5Cint_%7B0%7D%5E%7BT%7D%7Bf%5Cleft%28+t+%5Cright%29%5Csin%5Cleft%28+%5Ctext%7Bn%CF%89t%7D+%5Cright%29%5Ctext%7Bdt%7D%7D%3Db_%7Bn%7D%5Cint_%7B0%7D%5E%7BT%7D%7B%5Csin%7B%5Cleft%28+%5Ctext%7Bn%CF%89t%7D+%5Cright%29%5Csin%5Cleft%28+k%5Ctext%7B%CF%89t%7D+%5Cright%29%7D%5Ctext%7Bdt%7D%7D%3Db_%7Bn%7D%5Cint_%7B0%7D%5E%7BT%7Dsin%28nwt%29%5E%7B2%7Ddt)

进一步计算，可得

![[公式]](https://www.zhihu.com/equation?tex=%5Cint_%7B0%7D%5E%7BT%7D%7Bf%5Cleft%28+t+%5Cright%29%5Csin%5Cleft%28+%5Ctext%7Bn%CF%89t%7D+%5Cright%29%5Ctext%7Bdt%7D%7D%3Db_%7Bn%7D%5Cfrac%7BT%7D%7B2%7D)

![[公式]](https://www.zhihu.com/equation?tex=+b_%7Bn%7D+%3D+%5Cfrac%7B2%7D%7BT%7D%5Cint_%7B0%7D%5E%7BT%7D%7Bf%5Cleft%28+t+%5Cright%29%5Csin%5Cleft%28+%5Ctext%7Bn%CF%89t%7D+%5Cright%29%5Ctext%7Bdt%7D%7D+)

同样，![[公式]](https://www.zhihu.com/equation?tex=a_%7Bn%7D) 也可以使用相同的方式进行推导

因此，通过 ![[公式]](https://www.zhihu.com/equation?tex=a_%7Bn%7Db_%7Bn%7D) 我们可以知道这个波的波幅与相位：

![[公式]](https://www.zhihu.com/equation?tex=+c_%7Bn%7D+%3D+%5Csqrt%7B%7Ba_%7Bn%7D%7D%5E%7B2%7D+%2B+%7Bb_%7Bn%7D%7D%5E%7B2%7D%7D+)

![[公式]](https://www.zhihu.com/equation?tex=%5Cvarphi+%3D+arctan%28+-+%5Cfrac%7Bb_%7Bn%7D%7D%7Ba_%7Bn%7D%7D%29)

好了,这个基本就是傅里叶变换中最核心的傅里叶级数了

不是很复杂吧,你是不是很疑惑,为什么长得和傅里叶变换的标准公式差的有点多呢,标准公式不是长得是这样么:

![[公式]](https://www.zhihu.com/equation?tex=+F%5Cleft%28+f%5Cleft%28+t+%5Cright%29+%5Cright%29+%3D+%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7Bf%28t%29e%5E%7B-+iwt%7D%5Ctext%7Bdt%7D%7D)

没关系,看看我们的欧拉公式

![[公式]](https://www.zhihu.com/equation?tex=+e%5E%7B%5Ctext%7Bi%CE%B8%7D%7D+%3D+cos%5Ctheta+%2B+isin%5Ctheta+)

然后把欧拉公式代入傅里叶变换

![[公式]](https://www.zhihu.com/equation?tex=+F%5Cleft%28+f%5Cleft%28+t+%5Cright%29+%5Cright%29+%3D+%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7Bf%5Cleft%28+t+%5Cright%29%5Clbrack%5Ccos%5Cleft%28+wt+%5Cright%29+-+isin%28wt%29%5Crbrack%5Ctext%7Bdt%7D%7D+)

![[公式]](https://www.zhihu.com/equation?tex=+F%5Cleft%28+f%5Cleft%28+t+%5Cright%29+%5Cright%29+%3D+%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7Bf%5Cleft%28+t+%5Cright%29%5Ccos%5Cleft%28%5Comega+t+%5Cright%29%5Ctext%7Bdt%7D%7D+-+%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7Bf%5Cleft%28+t+%5Cright%29%5Coperatorname%7Bisin%7D%5Cleft%28+%5Comega+t+%5Cright%29%5Ctext%7Bdt%7D%7D+)

你看,最终还不是换汤不换药,无非就是多了个复数,这个复数其实没有别的其它意义,作用就是在计算中和cos区分开来,扯到复平面上绕圈圈?没必要!

现在傅里叶变换讲完了,我们来看看拉普拉斯变换

真的,傅里叶搞懂了拉普拉斯变换基本上一句话就能讲完,如果不扯点傅里叶变换的东西,我估计会因为回答问题过于简短待会答案都被折叠了

先看看拉普拉斯变换公式

![[公式]](https://www.zhihu.com/equation?tex=+F%28s%29+%3D%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7Bf%28t%29e%5E%7B-+st%7D%5Ctext%7Bdt%7D%7D%3D+%5Cint_%7B-+%5Cinfty%7D%5E%7B%5Cinfty%7D%7Bf%28t%29e%5E%7B-+%28%5Csigma+%2Biw%29t%7D%5Ctext%7Bdt%7D%7D)

这搞毛呢,不就是傅里叶变换的公式乘以一个 ![[公式]](https://www.zhihu.com/equation?tex=e%5E%7B-%5Csigma%7D) 么,只要搞懂为什么要这么干,我们就能理解拉普拉斯变换了

我们来看看下面这个信号图

![img](https://pic1.zhimg.com/50/v2-853336361ecc499d208fd815b9cad2a3_hd.jpg)![img](https://pic1.zhimg.com/80/v2-853336361ecc499d208fd815b9cad2a3_hd.jpg)

是的,这个信号的毛病在于,他已经上天了,是的,它增长的速度太快了,而我们却要使用不能够"上天"的正弦函数去拟合它,这不是为难我胖虎么,这个时候,我们就得想起一句名言,要么解决问题,要么解决制造问题的人(信号),既然傅里叶变换无法制造一个同样上天的正弦信号来拟合,我们就把它原本的信号"掰弯",那么如何"掰弯"呢,简单,乘以一个 ![[公式]](https://www.zhihu.com/equation?tex=e%5E%7B-%5Csigma%7D) 就行了

然后图像就变成了这样

![img](https://pic4.zhimg.com/50/v2-e99d5a465d0c8d5045cf767c359305a0_hd.jpg)![img](https://pic4.zhimg.com/80/v2-e99d5a465d0c8d5045cf767c359305a0_hd.jpg)

你看,这不就皆大欢喜了么,搞来搞去,拉普拉斯变换的意义无非就是把那些想要上天的函数掰弯,好最终变成那种适合做变换的函数,但是掰弯听起来不太专业,所以我们又管![[公式]](https://www.zhihu.com/equation?tex=e%5E%7B-%5Csigma%7D)叫衰减因子

好了,现在能解决 的信号我们有傅里叶变换解决了,不能解决的信号有拉普拉斯变换解决了,感觉上是不是皆大欢喜,写个软件跑跑看呗

这时你一拍脑袋!不好,信号是连续的,而计算机上存储的数据是离散的,这可咋办好,没关系,我们可以这样,每隔一小段距离,取一个点,最后用的时候把这些点连起来,不就能变成原来的的信号了么,当然我们还得研究研究,这个一小段距离究竟得多小,才不至于让原信号失真,这个就得参考参考香农采样定律了

好的,现在我们把连续的信号换一下,换成离散的"点",首先积分是不能用了,既然换成离散的了,积分对应的就应该变成累加符号 ![[公式]](https://www.zhihu.com/equation?tex=%5CSigma) ,当然,![[公式]](https://www.zhihu.com/equation?tex=f%28t%29) 也是不能用了,这是一个连续信号的写法,而离散的一个一个的点得换成 ![[公式]](https://www.zhihu.com/equation?tex=x%5Bn%5D) ,其中的n表示第n个点,实际上就是时间变来的,当然 ![[公式]](https://www.zhihu.com/equation?tex=e%5E%7B-j%5Comega+t%7D) 也不能用了,你想啊,我们要具体到某个点,这个点怎么表示,当然了,首先把 ![[公式]](https://www.zhihu.com/equation?tex=t) 时间换成 ![[公式]](https://www.zhihu.com/equation?tex=n) 索引号,然后 ![[公式]](https://www.zhihu.com/equation?tex=w) 这个动态的角速度值换成具体的角度 ![[公式]](https://www.zhihu.com/equation?tex=%5Cphi)

好了,我们终于把连续信号的傅里叶变换变成了离散信号的傅里叶变换,写写看

![[公式]](https://www.zhihu.com/equation?tex=A%5Csum_%7B-%5Cinfty%7D%5E%7B%5Cinfty%7D%7Bx%5Bn%5De%5E%7B-j%5Cphi+n%7D%7D)

令 ![[公式]](https://www.zhihu.com/equation?tex=z%3DAe%5E%7Bj%5Cphi%7D) 得到

![[公式]](https://www.zhihu.com/equation?tex=X%28z%29%3D%5Csum_%7B-%5Cinfty%7D%5E%7B%5Cinfty%7D%7Bx%5Bn%5Dz%5E%7B-n%7D%7D+)

哎呀,一不小心把Z变换的公式也写出来了,原来搞了半天,不就是傅里叶变换的离散形式么.

最后总结一下

![img](https://pic3.zhimg.com/50/v2-3d77916a96663ed60abe33ceabed16f1_hd.jpg)![img](https://pic3.zhimg.com/80/v2-3d77916a96663ed60abe33ceabed16f1_hd.jpg)

数学分析工具就是这样,当出现解决不了的问题之后,随之就会出现改进的方案,我们可以说,拉普拉斯变换是为了解决一些"太飘了"或者专业说法叫不收敛的信号,而z变换则用于解决了信号的存储和编码问题,那么,那么还有没有别的问题?

有的,从时域到频域,频域的时间信息消失了,你有没觉得之前我们分析的信号都太理想化了,现实中的信号往往随着时间而变化并非一成不变的,比如一辆车向你开来然后远去,你会听到声音从尖锐逐渐变得浑浊,这是多普勒效应造成的,而你收到的声音信号也由高频逐渐变为低频,而傅里叶变换只能告诉你信号中存在某种频率的信号,但却不能告诉你这个频率的信号是在什么时候出现的.它可能一直存在,或者只存在前半段信号里,可能存在后半段信号里.或者别的区间.

这个时候,又出现了傅里叶变换的改进版本,叫短时傅里叶变换.简单来说就是一段信号,假如这个信号长度是1秒,那么就每隔0.1秒就做一次傅里叶变换,总共做10次,这样,第一个变换的结果对应0-0.1s的信号频谱,第二个变换结果对应0.1-0.2s的信号频谱

虽然短时傅里叶提供了一个粗糙版本的方案把时间的概念引入频域,但无法解决信号拟合的问题,我们使用正弦波去拟合方波,我们就需要用无穷多个不同频率的正弦波去拟合以抵消时频间的能量差异,简单来说,一个方波我们用正弦波去拟合,最终会拟合成这个样子

![img](https://pic4.zhimg.com/50/v2-ee38d87de684b7108a0c8a107c4a0514_hd.jpg)![img](https://pic4.zhimg.com/80/v2-ee38d87de684b7108a0c8a107c4a0514_hd.jpg)



为了解决上面两个问题,小波变换诞生了,要使用小波变换,在进行变换前首先需要挑选合适的母小波(也常常叫基波函数,以前实验室里经常被用来调侃:喲,你搞个基波啊),然后通过对母小波的平移和缩放,最终去拟合原信号,在平移的过程中,最终也把时间信息带入了频域(小波域)中,同时不同的母小波也更好解决了信号的拟合问题,当然,大多小波变换的核心原理,最终和傅里叶变换一样,利用了正交性来检波(有的基波没有正交性,例如morlet和mexican hat,这类小波在用于离散小波变换时有限制性)

那么如何挑选母小波呢?不用担心,数学大佬们为我们总结了一堆好用的母小波,按照响应的情况挑选就行了

![img](https://pic1.zhimg.com/50/v2-b98eee21c5fcbc31c77fa3b6c0d2faa3_hd.jpg)![img](https://pic1.zhimg.com/80/v2-b98eee21c5fcbc31c77fa3b6c0d2faa3_hd.jpg)



什么,太简略了不过瘾?

[DBinary：傅里叶变换推导详解zhuanlan.zhihu.com![图标](https://pic2.zhimg.com/equation_ipico.jpg)](https://zhuanlan.zhihu.com/p/77345128)[DBinary：离散傅里叶变换DFT详解及应用zhuanlan.zhihu.com![图标](https://pic3.zhimg.com/equation_ipico.jpg)](https://zhuanlan.zhihu.com/p/77347644)[DBinary：从三角函数到离散傅里叶变换到语音识别再到图像频域鲁棒性水印zhuanlan.zhihu.com![图标](https://pic1.zhimg.com/equation_ipico.jpg)](https://zhuanlan.zhihu.com/p/72644228)[DBinary：详解离散余弦变换（DCT）zhuanlan.zhihu.com![图标](https://pic1.zhimg.com/equation_ipico.jpg)](https://zhuanlan.zhihu.com/p/85299446)

references:

奥本海姆大佬<<信号与系统>> <<离散时间信号处理>> 大佬是我心中永远的神

<<小波十讲>>他们说这本书是本好书,可惜我真没几章看的懂

<<数字语音处理理论与应用>> 声控福利,宅男必备,论如何秒变萝莉音忽悠沙雕网友