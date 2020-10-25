---

title: 限流原理解读之guava中的RateLimiter
categories:  RateLimiter限流
tags: [Guava] 
date: 2019-10-19
cover: https://gschaos.club/ico/img/eric-lee-psb.jpg


---

<font color= #EE82EE>限流原理解读之guava中的RateLimiter</font>



<!-- more -->




# <font color= #EE82EE>限流原理解读之guava中的RateLimiter</font>

RateLimiter有两种新建的方式

1. 创建Bursty方式
2. 创建WarmingUp方式

> 以下源码来自 guava-17.0

# <font color= #EE82EE>Bursty</font>

```java
//初始化
RateLimiter r = RateLimiter.create(1); 
//不阻塞
r.tryAcquire();
//阻塞
r.acquire()

```

RateLimiter.create做了两件事情创建Bursty对象和设置了速率,至次初始化过程结束

```java
RateLimiter rateLimiter = new Bursty(ticker, 1.0 /* maxBurstSeconds */); //ticker默认使用自己定义的
rateLimiter.setRate(permitsPerSecond);

```

1. 新建Bursty对象。它指定的是能够存储的最大时间是多长，比如设置的时间是<font color= #EE82EE>1s</font>,那么假设允许每秒钟发放的令牌数量为<font color= #EE82EE>2</font>，能存储的最大量为<font color= #EE82EE>2</font>；

2. setRate。 内部通过私有锁来保证速率的修改是线程安全的

   ```java
   synchronized (mutex) {
     //1：查看当前的时间是否比预计下次可发放令牌的时间要大，如果大，更新下次可发放令牌的时间为当前时间
     resync(readSafeMicros());
     //2：计算两次发放令牌之间的时间间隔，比如1s中需要发放5个，那它就是 200000.0微秒
     double stableIntervalMicros = TimeUnit.SECONDS.toMicros(1L) / permitsPerSecond;
     this.stableIntervalMicros = stableIntervalMicros;
     //3：设置maxPermits和storedPermits
     doSetRate(permitsPerSecond, stableIntervalMicros);
   }
   
   ```

> resync源码
>
> ```java
> private void resync(long nowMicros) {
>   // 查看当前的时间是否比预计下次可发放令牌的时间要大，如果大，更新下次可发放令牌的时间为当前时间
>   if (nowMicros > nextFreeTicketMicros) {
>     storedPermits = Math.min(maxPermits,
>         storedPermits + (nowMicros - nextFreeTicketMicros) / stableIntervalMicros);
>     nextFreeTicketMicros = nowMicros;
>   }
> }
> 
> ```
>
> doSetRate源码
>
> ```java
> @Overridevoid doSetRate(double permitsPerSecond, double stableIntervalMicros) {
>  double oldMaxPermits = this.maxPermits;
>  maxPermits = maxBurstSeconds * permitsPerSecond;
>  storedPermits = (oldMaxPermits == 0.0)
>      ? 0.0 // 初始条件存储的是没有
>     storedPermits * maxPermits / oldMaxPermits;
> }
> 
> ```

在整个的初始化过程中，关键信息是：

- <font color=  #00FF7F >nextFreeTicketMicros</font> 预计下次发放令牌的时间, stableIntervalMicros 两次发放令牌之间的时间间隔
- <font color=  #00FF7F>maxPermits</font> 最大能存储的令牌的数量 storedPermits 已经存储的令牌数

## <font color= #EE82EE>为什么是nextFreeTicketMicros?</font>

最简单的维持QPS速率的方式就是记住最后一次请求的时间，然后确保再次有请求过来的时候，已经经过了 1/QPS 秒。比如QPS是5 次/秒，只需要确保两次请求时间经过了200ms即可，如果刚好在100ms到达，就会再等待100ms,也就是说，如果一次性需要15个令牌，需要的时间为为3s。但是对于一个长时间没有请求的系统，这样的的设计方式有一定的不合理之处。考虑一个场景：如果一个RateLimiter,每秒产生1个令牌,它一直没有使用过，突然来了一个需要100个令牌的请求，选择等待100s再执行这个请求，显得不太明智，更好的处理方式为立即执行它，然后把接下来的请求推迟100s。

因而RateLimiter本身并不记下最后一次请求的时间，而是记下下一次期望运行的时间（nextFreeTicketMicros）。

> 这种方式带来的一个好处是，可以去判断等待的超时时间是否大于下次运行的时间，以使得能够执行，如果等待的超时时间太短，就能立即返回。

## <font color= #EE82EE>为什么会有一个标记代表存储了多少令牌？</font>

同样的考虑长时间没有使用的场景。如果长时间没有请求，突然间来了，这个时候是否应该立马放行这些请求？长时间没有使用可能意味着两件事：

1. 很多资源是存在空闲的情况，比如说网络请求长时间没有，它的缓冲区很有可能是空的，此时是可以加速传输，提高它的利用率
2. 一些时候，瞬间的爆发会导致溢出，比如说服务上的缓存过期了，需要去查询库，这个花销是非常“昂贵”的，过多的请求会导致数据库撑不住

RateLimiter就使用storedPermits来给过去请求的不充分程度建模。它的存储规则如下： 假设RateLimiter每秒产生一个令牌,每过去一秒如果没有请求，RateLimter也就没有消费，就使storedPermits增长1。假设10s之内都没有请求过来,storedPermits就变成了10（假设maxPermits>10），此时如果要获取3个令牌，会使用storedPermits来中的令牌来处理，然后它的值变为了7，片刻之后，如果调用了acquire(10),部分的会从storedPermits拿到7个权限，剩余的3个则需要重新产生。

总的来说RateLimiter提供了一个<font color=  #00FF7F>storedPermits</font>变量，当资源利用充分的时候，它就是0，最大可以增长到 <font color=  #00FF7F>maxStoredPermits</font>。请求所需的令牌来自于两个地方：stored permits(空闲时存储的令牌)和fresh permits（现有的令牌）

## <font color= #EE82EE>怎么衡量从storedPermits中获取令牌这个过程？</font>

同样假设每秒RateLimiter只生产一个令牌，正常情况下，如果一次来了3个请求，整个过程会持续3秒钟。考虑到长时间没有请求的场景：

1. 资源空闲。这种时候系统是能承受住一定量的请求的，当然希望在承受范围之内能够更快的提供请求，也就是说，如果有存储令牌，相比新产生令牌，此时希望能够更快的获取令牌，也就是此时从存储令牌中获取令牌的时间消耗要比产生新令牌要少，从而更快相应请求
2. 瞬时流量过大。这时候就不希望过快的消耗存储的令牌，希望它能够相比产生新的令牌的时间消耗大些，从而能够使请求相对平缓。

分析可知，针对不同的场景，需要对获取storedPermits做不同的处理，Ratelimiter的实现方式就是 storedPermitsToWaitTime 函数，它建立了从storedPermits中获取令牌和时间花销的模型函数,而衡量时间的花销就是通过对模型函数进行积分计算，比如原来存储了10个令牌，现在需要拿3个令牌，还剩余7个，那么所需要的时间花销就是该函数从7-10区间中的积分。

> 这种方式保证了任何获取令牌方式所需要的时间都是一样的，好比 每次拿一个和先拿两个再拿一个，从时间上来讲并没有分别。

## <font color= #EE82EE>storedPermitsToWaitTime实现原理</font>

storedPermits本身是用来衡量没有使用的时间的。在没有使用令牌的时候存储，存储的速率（单位时间内存储的令牌的个数）是 每没用1次就存储1次: rate=permites/time 。也就是说 1 / rate = time / permits，那么可得到 (1/rate)*permits 就可以来衡量时间花销。

选取(1/rate)作为基准线

- 如果选取一条在它之上的线，就做到了比从fresh permits中获取要慢；
- 如果在基准线之下，则是比从fresh permits中获取要快；
- 刚好是基准线，那么从storedPermits中获取和新产生的速率一模一样；

## <font color= #EE82EE>Bursty的storedPermitsToWaitTime函数实现</font>

```java
long storedPermitsToWaitTime(double storedPermits, double permitsToTake) {
  return 0L;
}

```

它直接返回了0，也就是在基准线之下，获取storedPermits的速率比新产生要快，立即能够拿到存储的量

# <font color= #EE82EE>WarmingUp</font>

```java
//初始化
RateLimiter r =RateLimiter.create(1,1,TimeUnit.SECONDS);
//不阻塞
r.tryAcquire();
//阻塞
r.acquire()

```

create方法创建了WarmingUp对象，并这只了对应的速率

```java
RateLimiter rateLimiter = new WarmingUp(ticker, warmupPeriod, unit);
rateLimiter.setRate(permitsPerSecond);
复制代码
```

相比Bursty，它多了个参数warmupPeroid,它会以提供的unit为时间单位，转换成微秒存储。setRate类似于Bursty，只是在doSetRate提供不同的实现

```java
void doSetRate(double permitsPerSecond, double stableIntervalMicros) {
  double oldMaxPermits = maxPermits;
  //1:最大的存储个数为需要预热的时间除以两个请求的时间间隔，比如设定预热时间为1s,每秒有5个请求，那么最大的存储个数为1000ms/200ms=5个
  maxPermits = warmupPeriodMicros / stableIntervalMicros; 
  //2:计算最大存储permits的一半
  halfPermits = maxPermits / 2.0;
  //3:初始化稳定时间间隔的3倍作为冷却时间间隔
  double coldIntervalMicros = stableIntervalMicros * 3.0;
  //4:设置基准线的斜率
  slope = (coldIntervalMicros - stableIntervalMicros) / halfPermits;
  if (oldMaxPermits == Double.POSITIVE_INFINITY) {
    storedPermits = 0.0;
  } else {
    storedPermits = (oldMaxPermits == 0.0)
        ? maxPermits // 初始条件下，认为就是存储满的，以达到缓慢消费的效果
        : storedPermits * maxPermits / oldMaxPermits;
  }
}

```

在这个过程中可以看到Warmup方式新增了一个halfPermits的设计，以及通过公式 `slope=(coldIntervalMicros-stableIntervalMicros)/halfPermits`，他们在函数 storedPermitsToWaitTime中得到了运用

```java
@Overridelong storedPermitsToWaitTime(double storedPermits, double permitsToTake) {
  //1:计算储存的令牌中超过了最大令牌一半的数量
  double availablePermitsAboveHalf = storedPermits - halfPermits;
  long micros = 0;
  // 计算超过一半的部分所需要的时间花销(对于函数来说，就是积分计算)
  if (availablePermitsAboveHalf > 0.0) {
    double permitsAboveHalfToTake = Math.min(availablePermitsAboveHalf, permitsToTake);
    micros = (long) (permitsAboveHalfToTake * (permitsToTime(availablePermitsAboveHalf)
        + permitsToTime(availablePermitsAboveHalf - permitsAboveHalfToTake)) / 2.0);
    permitsToTake -= permitsAboveHalfToTake;
  }
  // 计算函数的尚未超过一半的部分所需要的时间花销
  micros += (stableIntervalMicros * permitsToTake);
  return micros;
}

private double permitsToTime(double permits) {
  return stableIntervalMicros + permits * slope;
}

```

## <font color= DarkSlateGray1 >WarmingUp的设计理念</font>

WarmingUp对时间花销衡量方式为下图

```java
   *          ^ throttling
   *          |
   * 3*stable +                  /
   * interval |                 /.
   *  (cold)  |                / .
   *          |               /  .   <-- "warmup period" is the area of the trapezoid between
   * 2*stable +              /   .       halfPermits and maxPermits
   * interval |             /    .
   *          |            /     .
   *          |           /      .
   *   stable +----------/  WARM . }
   * interval |          .   UP  . } <-- this rectangle (from 0 to maxPermits, and
   *          |          . PERIOD. }     height == stableInterval) defines the cooldown period,
   *          |          .       . }     and we want cooldownPeriod == warmupPeriod
   *          |---------------------------------> storedPermits
   *              (halfPermits) (maxPermits)

```

> 横轴表示存储的令牌个数，纵轴表示时间，这样函数的积分就可以表示所要消耗的时间。
> 在程序刚开始运行的时候，warmingup方式会存满所有的令牌，而根据从存储令牌中的获取方式，可以实现从存储最大令牌中到降到一半令牌所需要的时间为存储同量令牌时间的2倍，从而使得刚开始的时候发放令牌的速度比较慢，等消耗一半之后，获取的速率和生产的速率一致，从而也就实现了一个‘热身’的概念

从storedPermits中获取令牌所需要的时间，它分为两部分，以maxPetmits的一半为分割点

- storedPermits <= halfPermits 的时候,存储和消费storedPermits的速率与产生的速率一模一样
- storedPermits>halfPermits, 存储storePermites所需要的时间和产生的速率保持一致，但是消费storePermites从maxPermits到halfPermits所需要的时间为从halfPermits增长到maxPermits所需要时间的2被，也就是比新令牌产生要慢 为什么在分隔点计算还有斜率方面选了3倍和一半的位置 对函数做积分计算(图形面积)，刚好可以保证，超过一半的部分，如果要拿掉一半的存储令牌所需要的时间恰好是存储同样量（或者说是新令牌产生）时间花销的两倍，对应场景如果过了很长一段时间没有使用(存储的令牌会达到maxPermits),刚开始能接收请求的速率相对比较慢，然后再增长到稳定的消费速率

> 关键在于存储的速率是和新令牌产生的速率一样，但是消费的速率，当存储的超过一半时，会慢于新令牌产生的速率，小于一半则速率是一样的

# <font color= #EE82EE>TryAcquire</font>

它会尝试去获取令牌，如果无法获取就立即返回，否则再超时时间之内返回给定的令牌。 源码如下

```java
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) {
  //1：使用微秒来转换超时时间
  long timeoutMicros = unit.toMicros(timeout);
  checkPermits(permits);
  long microsToWait;
  synchronized (mutex) {
    long nowMicros = readSafeMicros();
    //2.1：如果下次能够获取令牌的时间超过超时时间范围，立马返回；
    if (nextFreeTicketMicros > nowMicros + timeoutMicros) {
      return false;
    } else {
      //2.2：获取需要等待的时间，本次获取的时间肯定不会超时
      microsToWait = reserveNextTicket(permits, nowMicros);
    }
  }
  //3：实行等待
  ticker.sleepMicrosUninterruptibly(microsToWait);
  return true;
}

```

第一次运行的时候，nextFreeTicketMicros是创建时候的时间，必定小于当前时间，所以第一次肯定会放过，允许执行，只是需要计算要等待的时间。

```java
private long reserveNextTicket(double requiredPermits, long nowMicros) {
  //1：如果下次可以获取令牌的时间在过去，更新
  resync(nowMicros);
  //2：计算距离下次获取令牌需要的时间，如果nextFreeTikcetMicros>nowMicros，这个时间段必定在超时时间之内,假如入超时时间是0，那么必定是microsToNextFreeTicket趋近于0，也就是立马能够放行；
  long microsToNextFreeTicket = Math.max(0, nextFreeTicketMicros - nowMicros);
  //3：计算需要消耗的存储的令牌
  double storedPermitsToSpend = Math.min(requiredPermits, this.storedPermits);
  //4：计算需要新产生的令牌
  double freshPermits = requiredPermits - storedPermitsToSpend;
  //5：计算消耗存储令牌所需要的时间和新产生令牌所需要的时间。对于Bursty来讲，消耗存储的令牌所需要时间为0，WarmingUp方式则是需要根据不同的场景有不同的结果
  long waitMicros = storedPermitsToWaitTime(this.storedPermits, storedPermitsToSpend)
      + (long) (freshPermits * stableIntervalMicros);
  //6：下次能够获取令牌的时间，需要延迟当前已经等待的时间，也就是说，如果立马有请求过来会放行，但是这个等待时间将会影响后续的请求访问，也就是说，这次的请求如果当前的特别的多，下一次能够请求的能够允许的时间必定会有很长的延迟
  this.nextFreeTicketMicros = nextFreeTicketMicros + waitMicros;
  //7：扣除消耗的存储令牌
  this.storedPermits -= storedPermitsToSpend;
  //8：返回本次要获取令牌所需要的时间,它肯定不会超过超时时间
  return microsToNextFreeTicket;
}

```

# <font color= #EE82EE>Acquire</font>

它会阻塞知道允许放行，返回值为阻塞的时长 源码如下

```java
public double acquire(int permits) {
  long microsToWait = reserve(permits); //也就是调用reserveNextTicket
  ticker.sleepMicrosUninterruptibly(microsToWait); //阻塞住需要等待的时长
  return 1.0 * microsToWait / TimeUnit.SECONDS.toMicros(1L);
}

```

# <font color= #EE82EE>TryAcquire 运行案例</font>

程序设置10个线程,使得并发数为10，模拟线上的场景,任务内容如下

```java
class MyTask implements Runnable{
    private CountDownLatch latch;
    private RateLimiter limiter;

    public MyTask(CountDownLatch latch, RateLimiter limiter) {
        this.latch = latch;
        this.limiter = limiter;
    }

    @Override    public void run() {
        try {
            //使得线程同时触发            
           latch.await();
            System.out.println("time "+System.currentTimeMillis()+"ms :"+limiter.tryAcquire());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}

```

## <font color= DarkSlateGray1 >Bursty-TryAcquire</font>

这里设置限制每秒的流量为5，也就是说第一次请求过后，下次请求需要等200ms

```java
RateLimiter r =RateLimiter.create(5);
ExecutorService service = Executors.newFixedThreadPool(10);
CountDownLatch latch = new CountDownLatch(10);

for (int i=0;i<10;i++)
{

    service.submit(new MyTask(latch, r));
    latch.countDown();
    System.out.println("countdown:" + latch.getCount());
}
System.out.println("countdown over");
service.shutdown();

```

结果如下

```java
countdown:9
countdown:8
countdown:7
countdown:6
countdown:5
countdown:4
countdown:3
countdown:2
countdown:1
countdown:0
countdown over
time 1538487195698ms :true
time 1538487195699ms :false
time 1538487195699ms :false
time 1538487195699ms :false
time 1538487195699ms :false
time 1538487195699ms :false
time 1538487195699ms :false
time 1538487195698ms :false
time 1538487195698ms :false
time 1538487195699ms :false

```

如果使得线程等待401ms,那么程序会存储的令牌为2个

> 注意刚开始存储的时候，不是慢的，这里的存储量是慢慢增长，并且能够立马拿到

```java
RateLimiter r =RateLimiter.create(5);
ExecutorService service = Executors.newFixedThreadPool(10);
CountDownLatch latch = new CountDownLatch(10);

for (int i=0;i<10;i++)
{

    service.submit(new MyTask(latch, r));
    if (i==9){
        TimeUnit.MILLISECONDS.sleep(401);
        System.out.println("sleep 10 seconds over");
    }
    latch.countDown();
    System.out.println("countdown:" + latch.getCount());
}
System.out.println("countdown over");
service.shutdown();

```

运行结果刚好允许3个运行

```java
countdown:9
countdown:8
countdown:7
countdown:6
countdown:5
countdown:4
countdown:3
countdown:2
countdown:1
sleep 10 seconds over
countdown:0
countdown over
time 1538487297981ms :true
time 1538487297981ms :false
time 1538487297981ms :false
time 1538487297981ms :false
time 1538487297981ms :true
time 1538487297981ms :true
time 1538487297981ms :false
time 1538487297981ms :false
time 1538487297981ms :false
time 1538487297981ms :false

```

如果等待时间超过1秒，允许放行的流量也不会超过6个，存储的令牌+第一个令牌

```java
RateLimiter r =RateLimiter.create(5);
ExecutorService service = Executors.newFixedThreadPool(10);
CountDownLatch latch = new CountDownLatch(10);

for (int i=0;i<10;i++)
{

    service.submit(new MyTask(latch, r));
    if (i==9){
        TimeUnit.MILLISECONDS.sleep(1001);
        System.out.println("sleep 10 seconds over");
    }
    latch.countDown();
    System.out.println("countdown:" + latch.getCount());
}
System.out.println("countdown over");
service.shutdown();
```

结果为

```java
countdown:9
countdown:8
countdown:7
countdown:6
countdown:5
countdown:4
countdown:3
countdown:2
countdown:1
sleep 10 seconds over
countdown:0
countdown over
time 1538487514780ms :true
time 1538487514780ms :true
time 1538487514780ms :true
time 1538487514780ms :false
time 1538487514780ms :true
time 1538487514780ms :false
time 1538487514780ms :false
time 1538487514780ms :false
time 1538487514780ms :true
time 1538487514780ms :true

```

## <font color= DarkSlateGray1 >WarmingUp-TryAcquire</font>

使用warmingUp的方式由于默认已经存储满了令牌，那么，它在第一次请求执行完之后，必须等待一定的时间才会让下一次请求开始，而这个请求放行的时间则是会超过存储所需要的时间

> 注意这里的不同，默认是存储满的，也就是刚开始的消费要慢很多

```java
RateLimiter r =RateLimiter.create(5,1,TimeUnit.SECONDS);
ExecutorService service = Executors.newFixedThreadPool(10);
CountDownLatch latch = new CountDownLatch(10);

for (int i=0;i<10;i++)
{

    service.submit(new MyTask(latch, r));
    latch.countDown();
    System.out.println("countdown:" + latch.getCount());
}
System.out.println("countdown over");
service.shutdown();

```

运行结果如下

```cmake
countdown:9
countdown:8
countdown:7
countdown:6
countdown:5
countdown:4
countdown:3
countdown:2
countdown:1
countdown:0
countdown over
time 1538487677462ms :true
time 1538487677462ms :false
time 1538487677462ms :false
time 1538487677462ms :false
time 1538487677462ms :false
time 1538487677462ms :false
time 1538487677462ms :false
time 1538487677462ms :false
time 1538487677462ms :false
time 1538487677462ms :false

```

# <font color= #EE82EE>Acquire运行案例</font>

所需要的task源码如下

```java
class MyTask implements Runnable{
    private CountDownLatch latch;
    private RateLimiter limiter;
    private long start;

    public MyTask(CountDownLatch latch, RateLimiter limiter,long start) {
        this.latch = latch;
        this.limiter = limiter;
       this.start=start;
    }

    @Override    public void run() {
        try {
            //使得线程同时触发            
           latch.await();
            System.out.printf("result:"+limiter.acquire(2));
            System.out.println(" time "+(System.currentTimeMillis()-start)+"ms");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}

```

## <font color= DarkSlateGray1 >Busty-Acquire</font>

Acquire会阻塞运行的结果，而且会提前消费

```java
RateLimiter r =RateLimiter.create(1); 
r.acquire();
System.out.println("time cost:"+(System.currentTimeMillis()-start)+"ms");
r.acquire();
System.out.println("time cost:"+(System.currentTimeMillis()-start)+"ms");
r.acquire(3);
System.out.println("time cost:"+(System.currentTimeMillis()-start)+"ms");
r.acquire();
System.out.println("time cost:"+(System.currentTimeMillis()-start)+"ms");

```

第一次会立马运行，然后因为请求了一次，下次发放令牌的时间往后迁移,获取的令牌越多，下次能够运行需要等待的时间越长

运行结果为

```cmake
time cost:0ms
time cost:1005ms
time cost:2004ms
time cost:5001ms

```

在多线程背景运行如下

```java
RateLimiter r =RateLimiter.create(1);
long start=System.currentTimeMillis();
r.acquire(3);
System.out.println("time cost:"+(System.currentTimeMillis()-start)+"ms");
ExecutorService service = Executors.newFixedThreadPool(10);
CountDownLatch latch = new CountDownLatch(10);

for (int i=0;i<10;i++)
{

    service.submit(new MyTask(latch, r,start));
    latch.countDown();
    System.out.println("countdown:" + latch.getCount());
}
System.out.println("countdown over");
service.shutdown();

```

结果如下

```SAS
time cost:1ms
countdown:9
countdown:8
countdown:7
countdown:6
countdown:5
countdown:4
countdown:3
countdown:2
countdown:1
countdown:0
countdown over
result:2.995732 time 3024ms
result:4.995725 time 5006ms
result:6.995719 time 7007ms
result:8.995716 time 9006ms
result:10.995698 time 11004ms
result:12.995572 time 13006ms
result:14.995555 time 15007ms
result:16.995543 time 17005ms
result:18.995516 time 19005ms
result:20.995463 time 21005ms

```

## <font color= DarkSlateGray1 >WarmingUp-acquire</font>

warmingUp通过acquire的方式获取的令牌，同样会被按照同步的方式获取

```java
RateLimiter r =RateLimiter.create(1,1,TimeUnit.SECONDS);
long start=System.currentTimeMillis();
r.acquire(3);
System.out.println("time cost:"+(System.currentTimeMillis()-start)+"ms”);
ExecutorService service = Executors.newFixedThreadPool(10);
CountDownLatch latch = new CountDownLatch(10);

for (int i=0;i<10;i++)
{

    service.submit(new MyTask(latch, r,start));
    latch.countDown();
    System.out.println("countdown:" + latch.getCount());
}
System.out.println("countdown over");
service.shutdown();

```

结果如下

```cmake
time cost:0ms
countdown:9
countdown:8
countdown:7
countdown:6
countdown:5
countdown:4
countdown:3
countdown:2
countdown:1
countdown:0
countdown over
result:3.496859 time 3521ms
result:5.496854 time 5506ms
result:7.49685 time 7505ms
result:9.496835 time 9504ms
result:11.496821 time 11505ms
result:13.496807 time 13502ms
result:15.496793 time 15504ms
result:17.496778 time 17506ms
result:19.496707 time 19506ms
result:21.496699 time 21506ms

```

> RateLimiter本身实现的就是一个[令牌桶算法](https://en.wikipedia.org/wiki/Token_bucket)

来自： https://juejin.im/post/5bb48d7b5188255c865e31bc 