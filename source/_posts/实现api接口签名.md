---
title: 实现api接口签名
cover: https://gschaos.club/ico/img/taeyong-choi-1-2.jpg
categories: 设计
tags: [设计,接口]
date: 2020-11-18

---
[TOC]







## API签名是什么

 API签名可以理解为就是对API的调用进行签名保护。是在进行API调用时，加了一个调用者及其调用行为的指纹信息，以帮助服务端更好的识别用户及其调用行为的合法性。

## 怎么做

### 设计API签名

#### 签名算法选择

 在密码学中，有对称加密算法、非对称加密算法、 希运算消息认证码等等几种方案可以很好保护用户密钥的同时，验证用户的身份。那么，我们应该如何选择呢？

​        （1）首先排除的是非对称加密算法，理由是耗时长，性能差。

​         通过实测，非对接加密算法（RSA）相对加密算法（AES）和 希运算消息认证码算法（HmacSHA256）的加解密耗时要高2~3个数量级，对于一个服务端来说，性能也是很重要的考虑标准，故一般不选择非对称加密算法。

| 算法类型   | 加密耗时     | 解密耗时    |
| :--------- | :----------- | :---------- |
| RSA        | 380317 ns/op | 34427 ns/op |
| AES        | 885 ns/op    | 938 ns/op   |
| HmacSHA256 | 1458 ns/op   | 1458 ns/op  |

​        （2）从以上结果看，Hmac和AES似乎都不错，而且AES更优。但考虑到签名的目的，除了明确用户身份外，还要明确调用者的调用行为；也就是说，为了需要保证整个请求的完整性，需要加密整个请求的所有关键内容，这时，Hmac算法的防伪造性(即修改一个字节，签名信息就完全不一样)的优势就突显出来了，在性能差不多的情况下，当然，选择Hmac算法了。

​        （3）Hmac支持的hash算法非常多，但一般不建议使用MD5和SHA1，因其有哈希长度扩展攻击(Hash Length Extension Attacks)的风险，故一般推荐使用HmacSHA256或HmacSHA512。

​         若服务端支持多种算法，则请求时，需明确带上使用的签名方法：**SignatureMethod**。

#### 明确调用者的调用行为

把调用行为涉及的关键信息都放到签名内容中进行签名

>即每个请求Method和URL，这是每个请求都有的信息，且最为关键的信息。
>
>请求的关键内容，即这部分内容改变这调用发生改变的内容，不过最好不要设计到字段，不然服务端需要为每个接口判断字段，不便于编码维护。

#### 防止重放

像一些签到，打卡，订阅，转账等请求是一次性的，即这些请求发送2次导致的结果会发生变化。

为了防止这样的请求重放就需要 调用者生成并带上一个随机数**Nonce** ,服务端该随机数是否已出现，有则拒绝，无则存储该随机数并放过请求.

 这里服务端要保证Nonce唯一，就得存储已经用过的Nonce，但长期保持会带来两个问题.

（1）存储成本增加，日积月累，这里要存储的Nonce会越来越多，需要的存储空间就越大

（2）碰撞概率增加，正常服务被拒绝概率增大；这里随着生成Nonce值越来越多，碰撞的概率一定越来越大，若通过增加Nonce值的长度，有增加存储成本。

​         那么，另一个可行的办法，就是调用者每次请求时带上当前请求时间点Timestamp，然后由服务端限制请求的时效性。

## 实现

springboot +Annotation +aop 实现对请求的拦截处理。

对于算法，其实客户端和服务端达成一致即可。

实现之前需要知道以下知识点。

### Annotation 

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/28123151-d471f82eb2bc4812b46cc5ff3e9e6b82.jpg)

从中，我们可以看出：

1. 1个Annotation 和 1个RetentionPolicy关联。
       可以理解为：每1个Annotation对象，都会有唯一的RetentionPolicy属性。

2. 1个Annotation 和 1~n个ElementType关联。
       可以理解为：对于每1个Annotation对象，可以有若干个ElementType属性。

3. Annotation 有许多实现类，包括：Deprecated, Documented, Inherited, Override等等。
       Annotation 的每一个实现类，都“和1个RetentionPolicy关联”并且“和1~n个ElementType关联”。

那这些都是什么意思呢，我们通过例子来说明。

```java
package java.lang.annotation;

/**
 * Annotation 接口
 “每1个Annotation” 都与 “1个RetentionPolicy”关联，并且与 “1～n个ElementType”关联。可以通俗的理解为：每1个Annotation对象，都会有唯一的RetentionPolicy属性；至于ElementType属性，则有1~n个。
 
 * The {@link java.lang.reflect.AnnotatedElement} interface discusses
 * compatibility concerns when evolving an annotation type from being
 * non-repeatable to being repeatable.
 *
 * @author  Josh Bloch
 * @since   1.5
 */
public interface Annotation {
        boolean equals(Object obj);
       	int hashCode();
        String toString();
        Class<? extends Annotation> annotationType();
}
```

```java
package java.lang.annotation;

/**
 *  对出现注解的地方做了枚举，分别有 
 1. TYPE(类、接口（包括注释类型）或枚举声明) ,   
 2. FIELD (字段声明（包括枚举常量）)
 3. METHOD (方法声明)
 4. PARAMETER (参数声明)
 5. CONSTRUCTOR（构造方法声明）
 6. LOCAL_VARIABLE (局部变量声明) 
 7. ANNOTATION_TYPE (注释类型声明)
 8. PACKAGE (包声明)
      ElementType它用来指定Annotation的类型。
      “每1个Annotation” 都与 “1～n个ElementType”关联。当Annotation与某个ElementType关联时，就意味着：Annotation有了某种用途。
      例如，若一个Annotation对象是METHOD类型，则该Annotation只能用来修饰方法。
      
 * <p>The constants {@link #ANNOTATION_TYPE} , {@link #CONSTRUCTOR} , {@link
 * #FIELD} , {@link #LOCAL_VARIABLE} , {@link #METHOD} , {@link #PACKAGE} ,
 * {@link #PARAMETER} , {@link #TYPE} , and {@link #TYPE_PARAMETER} correspond
 * to the declaration contexts in JLS 9.6.4.1.
 */
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}

```

```java
package java.lang.annotation;

/**
 * 注解的策略
  RetentionPolicy 是Enum枚举类型，它用来指定Annotation的策略。通俗点说，就是不同RetentionPolicy类型的Annotation的作用域不同。
  “每1个Annotation” 都与 “1个RetentionPolicy”关联。
     a) 若Annotation的类型为 SOURCE，则意味着：Annotation仅存在于编译器处理期间，编译器处理完之后，该Annotation就没用了。
          例如，“ @Override ”标志就是一个Annotation。当它修饰一个方法的时候，就意味着该方法覆盖父类的方法；并且在编译期间会进行语法检查！编译器处理完	  后，“@Override”就没有任何作用了。 
      b) 若Annotation的类型为 CLASS，则意味着：编译器将Annotation存储于类对应的.class文件中，它是Annotation的默认行为。
      c) 若Annotation的类型为 RUNTIME，则意味着：编译器将Annotation存储于class文件中，并且可由JVM读入。
 * @author  Joshua Bloch
 * @since 1.5
 */
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

例子：

申明一个注解 使用@inteface

```java

/**
 * 1. @Documented
 *                  类和方法的Annotation在缺省情况下是不出现在javadoc中的。如果使用@Documented修饰该Annotation，则表示它可以出现在javadoc中。
 *                  定义Annotation时，@Documented可有可无；若没有定义，则Annotation不会出现在javadoc中。
 * 2. @Target(ElementType.TYPE)
 *                    前面我们说过，ElementType 是Annotation的类型属性。而@Target的作用，就是来指定Annotation的类型属性。
 *                    @Target(ElementType.TYPE) 的意思就是指定该Annotation的类型是ElementType.TYPE。这就意味着，MyAnnotation1是来修饰“类、接口（包括注释类型）或枚举声明”的注解。
 *                    定义Annotation时，@Target可有可无。若有@Target，则该Annotation只能用于它所指定的地方；若没有@Target，则该Annotation可以用于任何地方。
 * 3. @Retention(RetentionPolicy.RUNTIME)
 *                      RetentionPolicy 是Annotation的策略属性，而@Retention的作用，就是指定Annotation的策略属性。
 *                      @Retention(RetentionPolicy.RUNTIME) 的意思就是指定该Annotation的策略是RetentionPolicy.RUNTIME。这就意味着，编译器会将该Annotation信息保留在.class文件中，并且能被虚拟机读取。
 *                      定义Annotation时，@Retention可有可无。若没有@Retention，则默认是RetentionPolicy.CLASS。
 */
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation{

}
```

```java
/**
 * @author ycc
 * @time 21:26
 * 在增加了 @Target(ElementType.TYPE) 限制注解只能使用在 TYPE(类、接口（包括注释类型）或枚举声明) 上，所以这里的test方法会提示报错。
 */
@TestAnnotation
public class Test {

    @TestAnnotation
    public void test(){

    }
}

```

![image-20201119212955159](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20201119212955159.png)



在看@Retention(RetentionPolicy.RUNTIME) 。

修改注解内容和测试类

```java
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.CLASS)
public @interface TestAnnotation{

}
```

```java
@TestAnnotation
public class Test {
    public void test(){

    }
    public static void main(String[] args) throws Exception{
        Class<? extends Test> aClass = Test.class;
   		 // Test类是否具有 TestAnnotation Annotation
        boolean annotationPresent = aClass.isAnnotationPresent(TestAnnotation.class);
        System.out.println(annotationPresent);
    }
}
```

除了 @Retention(RetentionPolicy.) 为RUNTIME的时候，我们得到的结果都是false，所以当我们想要动态判断自定义注解的时候，需要将设置成RUNTIME,编译器将Annotation存储于Class文件中，并且可由JVM读入。

由以上我们可以看到， 使用 @interface 定义一个注解  ，并使用 @Target(ElementType.TYPE) 限制注解使用的地方，如果不是编译器做提醒

像 @Override注解,在程序运行的时候Override是没有任何用途的,我们需要设置 @Retention(RetentionPolicy.) 为RUNTIME。

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```



### AOP

#### 什么是 AOP

AOP （Aspect Orient Programming）,直译过来就是 面向切面编程。AOP 是一种编程思想，是面向对象编程（OOP）的一种补充。面向对象编程将程序抽象成各个层次的对象，而面向切面编程是将程序抽象成各个切面。
![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/758949-20190529225344641-152289598.png)

从该图可以很形象地看出，所谓切面，相当于应用对象间的横切点，我们可以将其单独抽象为单独的模块。



我们在写代码逻辑的时候，都是自上而下，顺序执行(不考虑重排序的问题)；而我们需要一种能在顺序执行的时候横跨这些逻辑执行别的通用逻辑，这个时候就需要AOP，那么AOP如何实现？

#### AOP 实现分类



AOP 要达到的效果是，保证开发者不修改源代码的前提下，去为系统中的业务组件添加某种通用功能。AOP 的本质是由 AOP 框架修改业务组件的多个方法的源代码，看到这其实应该明白了，AOP 其实就是前面一篇文章讲的代理模式的典型应用。
按照 AOP 框架修改源代码的时机，可以将其分为两类：

- 静态 AOP 实现， AOP 框架在编译阶段对程序源代码进行修改，生成了静态的 AOP 代理类（生成的 *.class 文件已经被改掉了，需要使用特定的编译器），比如 AspectJ。
- 动态 AOP 实现， AOP 框架在运行阶段对动态生成代理对象（在内存中以 JDK 动态代理，或 CGlib 动态地生成 AOP 代理类），如 SpringAOP。

关于Spring提供的AOP方案的实现的很多概念大家自行百度。

这里通过代码来熟悉。



1. 首先定义一个抽象类使用@Pointcut来定义切点，再定义一个前置处理   @Before("addAdvice()")来做前置增强,我们这里不做后置或者环绕，原理类似。

```java

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Aspect
@Component
@Slf4j
public class DemoAspect {


    @Pointcut(" @within(com.mystical.cloud.entrance.sign.TestAnnotation)|| @annotation(com.mystical.cloud.entrance.sign.TestAnnotation)")
    public void addAdvice(){}

    @Before("addAdvice()")
    public void before(JoinPoint joinPoint){
    log.info("前置处理");
    }
}
```



**@Pointcut** 

此注解为切点注解，就是我们需要切入的类型的配置点，这里会进行模式匹配。

```java
/**
 * Pointcut declaration
 *
 * @author Alexandre Vasseur
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Pointcut {

    /**
     * @return the pointcut expression
     * We allow "" as default for abstract pointcut
     */
    String value() default "";
    
    /**
     * When compiling without debug info, or when interpreting pointcuts at runtime,
     * the names of any arguments used in the pointcut are not available.
     * Under these circumstances only, it is necessary to provide the arg names in 
     * the annotation - these MUST duplicate the names used in the annotated method.
     * Format is a simple comma-separated list.
     * 
     * @return argNames the argument names (should match those in the annotated method)
     */
    String argNames() default "";
}
```

 有这几种切点匹配方式

![image-20201124174554525](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20201124174554525.png)

我们这里使用@within和@annotation来匹配，这两个注解的意思就是 目标类含有某种类型的注解或者含有某个注解类型的方法。

我们先看一SpringAop组件的图，

![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/SouthEast.png)



请忽略上面的图......



#### Spring AOP代理对象的生成

Spring提供了两种方式来生成代理对象: JDKProxy和Cglib，具体使用哪种方式生成由AopProxyFactory根据AdvisedSupport对象的配置来决定。默认的策略是如果目标类是接口，则使用JDK动态代理技术，否则使用Cglib来生成代理。下面我们来研究一下Spring如何使用JDK来生成代理对象，具体的生成代码放在JdkDynamicAopProxy这个类中，直接上相关代码：

```java
/**
    * <ol>
    * <li>获取代理类要实现的接口,除了Advised对象中配置的,还会加上SpringProxy, Advised(opaque=false)
    * <li>检查上面得到的接口中有没有定义 equals或者hashcode的接口
    * <li>调用Proxy.newProxyInstance创建代理对象
    * </ol>
    */
   public Object getProxy(ClassLoader classLoader) {
       if (logger.isDebugEnabled()) {
           logger.debug("Creating JDK dynamic proxy: target source is " +this.advised.getTargetSource());
       }
       Class[] proxiedInterfaces =AopProxyUtils.completeProxiedInterfaces(this.advised);
       findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
       return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
}
```



那这个其实很明了，注释上我也已经写清楚了，不再赘述。

下面的问题是，代理对象生成了，那切面是如何织入的？

我们知道InvocationHandler是JDK动态代理的核心，生成的代理对象的方法调用都会委托到InvocationHandler.invoke()方法。而通过JdkDynamicAopProxy的签名我们可以看到这个类其实也实现了InvocationHandler，下面我们就通过分析这个类中实现的invoke()方法来具体看下Spring AOP是如何织入切面的。

```java
publicObject invoke(Object proxy, Method method, Object[] args) throwsThrowable {
       MethodInvocation invocation = null;
       Object oldProxy = null;
       boolean setProxyContext = false;
 
       TargetSource targetSource = this.advised.targetSource;
       Class targetClass = null;
       Object target = null;
 
       try {
           //eqauls()方法，具目标对象未实现此方法
           if (!this.equalsDefined && AopUtils.isEqualsMethod(method)){
                return (equals(args[0])? Boolean.TRUE : Boolean.FALSE);
           }
 
           //hashCode()方法，具目标对象未实现此方法
           if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)){
                return newInteger(hashCode());
           }
 
           //Advised接口或者其父接口中定义的方法,直接反射调用,不应用通知
           if (!this.advised.opaque &&method.getDeclaringClass().isInterface()
                    &&method.getDeclaringClass().isAssignableFrom(Advised.class)) {
                // Service invocations onProxyConfig with the proxy config...
                return AopUtils.invokeJoinpointUsingReflection(this.advised,method, args);
           }
 
           Object retVal = null;
 
           if (this.advised.exposeProxy) {
                // Make invocation available ifnecessary.
                oldProxy = AopContext.setCurrentProxy(proxy);
                setProxyContext = true;
           }
 
           //获得目标对象的类
           target = targetSource.getTarget();
           if (target != null) {
                targetClass = target.getClass();
           }
 
           //获取可以应用到此方法上的Interceptor列表
           List chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method,targetClass);
 
           //如果没有可以应用到此方法的通知(Interceptor)，此直接反射调用 method.invoke(target, args)
           if (chain.isEmpty()) {
                retVal = AopUtils.invokeJoinpointUsingReflection(target,method, args);
           } else {
                //创建MethodInvocation
                invocation = newReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
                retVal = invocation.proceed();
           }
 
           // Massage return value if necessary.
           if (retVal != null && retVal == target &&method.getReturnType().isInstance(proxy)
                    &&!RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
                // Special case: it returned"this" and the return type of the method
                // is type-compatible. Notethat we can't help if the target sets
                // a reference to itself inanother returned object.
                retVal = proxy;
           }
           return retVal;
       } finally {
           if (target != null && !targetSource.isStatic()) {
                // Must have come fromTargetSource.
               targetSource.releaseTarget(target);
           }
           if (setProxyContext) {
                // Restore old proxy.
                AopContext.setCurrentProxy(oldProxy);
           }
       }
    }
```

主流程可以简述为：获取可以应用到此方法上的通知链（Interceptor Chain）,如果有,则应用通知,并执行joinpoint; 如果没有,则直接反射执行joinpoint。而这里的关键是通知链是如何获取的以及它又是如何执行的，下面逐一分析下。

首先，从上面的代码可以看到，通知链是通过Advised.getInterceptorsAndDynamicInterceptionAdvice()这个方法来获取的,我们来看下这个方法的实现:

```java
public List<Object>getInterceptorsAndDynamicInterceptionAdvice(Method method, Class targetClass) {
                   MethodCacheKeycacheKey = new MethodCacheKey(method);
                   List<Object>cached = this.methodCache.get(cacheKey);
                   if(cached == null) {
                            cached= this.advisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice(
                                               this,method, targetClass);
                            this.methodCache.put(cacheKey,cached);
                   }
                   returncached;
         }
```

可以看到实际的获取工作其实是由AdvisorChainFactory. getInterceptorsAndDynamicInterceptionAdvice()这个方法来完成的，获取到的结果会被缓存。

下面来分析下这个方法的实现：

```java
/**
    * 从提供的配置实例config中获取advisor列表,遍历处理这些advisor.如果是IntroductionAdvisor,
    * 则判断此Advisor能否应用到目标类targetClass上.如果是PointcutAdvisor,则判断
    * 此Advisor能否应用到目标方法method上.将满足条件的Advisor通过AdvisorAdaptor转化成Interceptor列表返回.
    */
    publicList getInterceptorsAndDynamicInterceptionAdvice(Advised config, Methodmethod, Class targetClass) {
       // This is somewhat tricky... we have to process introductions first,
       // but we need to preserve order in the ultimate list.
       List interceptorList = new ArrayList(config.getAdvisors().length);
 
       //查看是否包含IntroductionAdvisor
       boolean hasIntroductions = hasMatchingIntroductions(config,targetClass);
 
       //这里实际上注册一系列AdvisorAdapter,用于将Advisor转化成MethodInterceptor
       AdvisorAdapterRegistry registry = GlobalAdvisorAdapterRegistry.getInstance();
 
       Advisor[] advisors = config.getAdvisors();
        for (int i = 0; i <advisors.length; i++) {
           Advisor advisor = advisors[i];
           if (advisor instanceof PointcutAdvisor) {
                // Add it conditionally.
                PointcutAdvisor pointcutAdvisor= (PointcutAdvisor) advisor;
                if(config.isPreFiltered() ||pointcutAdvisor.getPointcut().getClassFilter().matches(targetClass)) {
                    //TODO: 这个地方这两个方法的位置可以互换下
                    //将Advisor转化成Interceptor
                    MethodInterceptor[]interceptors = registry.getInterceptors(advisor);
 
                    //检查当前advisor的pointcut是否可以匹配当前方法
                    MethodMatcher mm =pointcutAdvisor.getPointcut().getMethodMatcher();
 
                    if (MethodMatchers.matches(mm,method, targetClass, hasIntroductions)) {
                        if(mm.isRuntime()) {
                            // Creating a newobject instance in the getInterceptors() method
                            // isn't a problemas we normally cache created chains.
                            for (intj = 0; j < interceptors.length; j++) {
                               interceptorList.add(new InterceptorAndDynamicMethodMatcher(interceptors[j],mm));
                            }
                        } else {
                            interceptorList.addAll(Arrays.asList(interceptors));
                        }
                    }
                }
           } else if (advisor instanceof IntroductionAdvisor){
                IntroductionAdvisor ia =(IntroductionAdvisor) advisor;
                if(config.isPreFiltered() || ia.getClassFilter().matches(targetClass)) {
                    Interceptor[] interceptors= registry.getInterceptors(advisor);
                    interceptorList.addAll(Arrays.asList(interceptors));
                }
           } else {
                Interceptor[] interceptors =registry.getInterceptors(advisor);
                interceptorList.addAll(Arrays.asList(interceptors));
           }
       }
       return interceptorList;
}
```

这个方法执行完成后，Advised中配置能够应用到连接点或者目标类的Advisor全部被转化成了MethodInterceptor.

接下来我们再看下得到的拦截器链是怎么起作用的。

```java
if (chain.isEmpty()) {
                retVal = AopUtils.invokeJoinpointUsingReflection(target,method, args);
            } else {
                //创建MethodInvocation
                invocation = newReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
                retVal = invocation.proceed();
            }
```

​     从这段代码可以看出，如果得到的拦截器链为空，则直接反射调用目标方法，否则创建MethodInvocation，调用其proceed方法，触发拦截器链的执行，来看下具体代码

```java
public Object proceed() throws Throwable {
       //  We start with an index of -1and increment early.
       if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size()- 1) {
           //如果Interceptor执行完了，则执行joinPoint
           return invokeJoinpoint();
       }
 
       Object interceptorOrInterceptionAdvice =
           this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
       
       //如果要动态匹配joinPoint
       if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher){
           // Evaluate dynamic method matcher here: static part will already have
           // been evaluated and found to match.
           InterceptorAndDynamicMethodMatcher dm =
                (InterceptorAndDynamicMethodMatcher)interceptorOrInterceptionAdvice;
           //动态匹配：运行时参数是否满足匹配条件
           if (dm.methodMatcher.matches(this.method, this.targetClass,this.arguments)) {
                //执行当前Intercetpor
                returndm.interceptor.invoke(this);
           }
           else {
                //动态匹配失败时,略过当前Intercetpor,调用下一个Interceptor
                return proceed();
           }
       }
       else {
           // It's an interceptor, so we just invoke it: The pointcutwill have
           // been evaluated statically before this object was constructed.
           //执行当前Intercetpor
           return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
       }
}
```

代码也比较简单，这里不再赘述。



好了，我们回到之前自定义的DemoAspect，这里就是使用AOP的原理在要执行目标方法之前拦截处理。

### 签名实现

1.  @SignedMapping，我们的@within/@annotation类型

   ```java
   /**
    * 注释表示需要对RESTful API进行签名
    */
   @Target({ElementType.TYPE, ElementType.METHOD})
   @Retention(value = RetentionPolicy.RUNTIME)
   @Inherited
   public @interface SignedMapping {
       Class<?> value() default BaseSignedService.class;
       boolean resubmit() default false;//允许重复请求
   }
   ```

2. 为了方便判断实体类型，我们为entity增加一个@SignedEntity注解，为需要待验证的签名增加@Signature注解

```java
/**
 * API签名封装的实体类
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface SignedEntity {
}

```

```java
/**
 * 签名字段
 */
@Target({ElementType.FIELD})
@Retention(value = RetentionPolicy.RUNTIME)
@SignedIgnore
public @interface Signature {
}
```

3. 为了方便拓展，有些时候不需要把数据一起加密，这里设计一个忽略字段的注解@SignedIgnore

```java
/**
 * 忽略字段
 */
@Target({ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(value = RetentionPolicy.RUNTIME)
public @interface SignedIgnore {
}

```

4. 针对于可能出现定义字段时与我们预知的名称不一致，我们设计每个需要使用字段的注解

```java
 /**
 *  标识该注解的字段为appId
 */
@Target({ElementType.FIELD})
@Retention(value = RetentionPolicy.RUNTIME)
public @interface SignedAppId {
}



/**
 * 标识该注解的字段为Nonce
 */
@Target({ElementType.FIELD})
@Retention(value = RetentionPolicy.RUNTIME)
public @interface SignedNonce {
}


/**
 *  标识该注解的字段为timestamp
 */
@Target({ElementType.FIELD})
@Retention(value = RetentionPolicy.RUNTIME)
public @interface SignedTimestamp {
}

```



5. SignedParam，需要接收api请求封装的entity

```java
@SignedEntity
@ToString
@Setter
@Getter
public class SignedParam {
    @SignedAppId
    private String appId;
    
    //这里把请求参数忽略
	@SignedIgnore
    private String data;
    
    @SignedTimestamp
    private long timestamp;
    
    @SignedNonce
    private int nonce;
    
    @Signature
    private String signature;

}

```



6.设计切面类, 这里的SignedService为抽象类，具体的获取实体参数和解析由子类实现

```java

public abstract class SignedService {
    @Value("${signature.algorithm:HmacSHA1}")
    protected String ALGORITHM;
    @Value("${signature.time-diff-max:100}")
    protected long TIME_DIFF_MAX;
    @Autowired(required = false)
    protected RedisUtil redisUtil;
    protected static final String PREFIX = "atpapi:signature:";

    @Pointcut("@within(com.mystical.cloud.auth.signature.annotation.SignedMapping) || @annotation(com.mystical.cloud.auth.signature.annotation.SignedMapping) ")
    public void mapping() {
    }


    @Before("mapping() ")
    public void before(JoinPoint joinPoint) throws Exception {
        MethodSignature sign = (MethodSignature) joinPoint.getSignature();
        Method method = sign.getMethod();

        //  Get the annotations on the class or method, and get the parameter
        SignedMapping signedMapping = AnnotationUtils.findAnnotation(method, SignedMapping.class);
        if (signedMapping == null) {
            signedMapping = AnnotationUtils.findAnnotation(joinPoint.getTarget().getClass(), SignedMapping.class);
        }

        //允许重复请求
        if (signedMapping == null || signedMapping.resubmit()) {
            return;
        }
        Class<?> clazz = signedMapping.value();

        //  If there are other implementation classes, prevent multiple executions here
        if (!this.getClass().equals(clazz)) {
            return;
        }

        Object[] args = joinPoint.getArgs();
        boolean paramHasSignedEntity = false;
        //获取SignedEntity 的数据
        if (args != null&& args.length>0) {
            for (Object obj : args) {
                if (obj==null){
                    continue;
                }
                SignedEntity signedEntity = AnnotationUtils.findAnnotation(obj.getClass(), SignedEntity.class);
                if (signedEntity != null) {
                    paramHasSignedEntity = true;
                    entry(obj);
                    break;
                }
            }
        }

        //注解的方法没有增加SignedEntity的形参获取 request的请求头信息.
        if (!paramHasSignedEntity) {
            signedValidate();
        }

    }

    protected abstract void entry(Object obj) throws Exception;

    /**
     * 获取待验证的集合
     *
     * @throws NoSuchAlgorithmException NoSuchAlgorithmException {@link NoSuchAlgorithmException}
     * @throws InvalidKeyException      InvalidKeyException {@link InvalidKeyException}
     */
    protected abstract void signedValidate() throws NoSuchAlgorithmException, InvalidKeyException;

    /**
     * Convert obj to map and verify that the field is not empty
     *
     * @param obj 代转换的对象
     * @return 将对象转换成key value集合
     * @throws IllegalAccessException InvalidKeyException {@link InvalidKeyException}
     */
    public Map<String, Object> object2Map(Object obj) throws IllegalAccessException {
        Map<String, Object> map = new HashMap<>();
        if (obj == null) {
            return map;
        }
        Class<?> clazz = obj.getClass();
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            field.setAccessible(true);
            //  If the field is not ignored, put it in the map
            //  If the field is null, throw an exception
            SignedIgnore signedIgnore = AnnotationUtils.findAnnotation(field, SignedIgnore.class);
            if (signedIgnore == null) {
                if (field.get(obj) == null) {
                    throw new SignedException.NullParam(field.getName());
                }
                map.put(field.getName(), field.get(obj));
            }
        }
        return map;
    }


    /**
     * Get the field value of obj through annotation
     *
     * @param obj            目标实体
     * @param annotationType 获取元素的注解类型
     * @return 标识 annotationType的 元素
     * @throws IllegalAccessException InvalidKeyException {@link InvalidKeyException}
     */
    public final Object getParamByAnnotation(Object obj, Class<? extends Annotation> annotationType) throws IllegalAccessException {
        Class<?> clazz = obj.getClass();
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            field.setAccessible(true);
            Annotation annotation = AnnotationUtils.findAnnotation(field, annotationType);
            if (annotation != null) {
                return field.get(obj);
            }
        }
        return null;
    }


    /**
     * Get the appSecret by appId
     *
     * @param appId appId
     * @return 根据appId获取appSecret
     */
    public String getAppSecret(String appId) {
        Object appSecret = redisUtil.get(appId);
        if (appSecret == null)
            throw new SignedException.AppIdInvalid(appId);
        return appSecret.toString();
    }

    /**
     * If the time difference between the server and the client is too large, throw an exception
     *
     * @param timestamp 超时时间戳。单位秒
     */
    public void isTimeDiffLarge(long timestamp) {
        long diff = timestamp - System.currentTimeMillis() / 1000;
        if (Math.abs(diff) > TIME_DIFF_MAX) {
            throw new SignedException.TimestampError(diff + "");
        }
    }


    /**
     * Judge whether it is a replay attack by time stamp and nonce
     */
    public void isReplayAttack(String appId, long timestamp, Object nonce, String signature) {
        String key = PREFIX + appId + "_" + timestamp + "_" + nonce;
        Object obj = redisUtil.get(key);
        if (obj != null && signature.equals(obj.toString()))
            throw new SignedException.ReplayAttack(appId, timestamp, nonce);
        else
            redisUtil.set(key, signature, TIME_DIFF_MAX);
    }

}

```



```java
@Aspect
@Component
@Primary
@Slf4j
public class BaseSignedService extends SignedService {
    /**
     * 组装验证sign
     *
     * @throws NoSuchAlgorithmException NoSuchAlgorithmException {@link NoSuchAlgorithmException }
     * @throws InvalidKeyException      InvalidKeyException {@link InvalidKeyException}
     */
    protected void signedValidate() throws NoSuchAlgorithmException, InvalidKeyException {
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        String appId = request.getHeader("aptapi-appid");
        long timestamp = Long.parseLong(request.getHeader("aptapi-timestamp"));
        int nonce = Integer.parseInt(request.getHeader("aptapi-nonce"));
        String signatureParam = request.getHeader("aptapi-signature");
        String data = request.getHeader("aptapi-data");
        Map<String, Object> headerMap = new HashMap<>(4);
        //处理header name
        headerMap.put("appId", appId);
        headerMap.put("timestamp", timestamp);
        headerMap.put("nonce", nonce);
        if (!StringUtils.isNotBlank(data)) {
            headerMap.put("data", data);
        }
        isTimeDiffLarge(timestamp);
        isReplayAttack(appId, timestamp, nonce, signatureParam);
        String signature = getSignature(appId, headerMap);
        //  If the signatures are inconsistent, throw an exception
        if (!signatureParam.equals(signature)) {
            throw new SignedException.SignatureError(signatureParam);
        }

    }
    /**
     * Entry method, used to call other methods uniformly
     *
     * @param obj
     * @throws Exception
     */
    public void entry(Object obj) throws Exception {

        Map map = object2Map(obj);

        String appId = (String) getParamByAnnotation(obj, SignedAppId.class);
        long timestamp = (Long) getParamByAnnotation(obj, SignedTimestamp.class);
        int nonce = (Integer) getParamByAnnotation(obj, SignedNonce.class);
        String signatureParam = (String) getParamByAnnotation(obj, Signature.class);

        isTimeDiffLarge(timestamp);

        isReplayAttack(appId, timestamp, nonce, signatureParam);

        String signature = getSignature(appId, map);

        //  If the signatures are inconsistent, throw an exception
        if (!signatureParam.equals(signature))
            throw new SignedException.SignatureError(signatureParam);
    }




    /**
     * Signature calculation and verification
     *
     * @param appId
     * @param map
     */
    public String getSignature(String appId, Map map) throws NoSuchAlgorithmException, InvalidKeyException {

        String appSecret = getAppSecret(appId);

        //  Sort the parameters by ascending ASCII
        SortedMap<String, Object> sortedMap = new TreeMap<>(map);

        //  Splice the parameters
        //  e.g. "key1=value1&key2=value2"
        StringBuilder plainText = new StringBuilder();
        for (Map.Entry<String, Object> entry : sortedMap.entrySet()) {
            if (entry.getKey().equals("signature")) {
                continue;
            }
            if (entry.getValue() == null) {
                continue;
            }
            plainText.append(entry.getKey()).append("=").append(entry.getValue());
            plainText.append("&");
        }
        plainText.deleteCharAt(plainText.length() - 1);

        //  The plain text is encrypted by algorithm and converted to Base64
        SecretKeySpec secretKeySpec = new SecretKeySpec(org.apache.commons.codec.binary.StringUtils.getBytesUtf8(appSecret), ALGORITHM);
        Mac mac = Mac.getInstance(ALGORITHM);
        mac.init(secretKeySpec);

        return Base64.encodeBase64String(mac.doFinal(org.apache.commons.codec.binary.StringUtils.getBytesUtf8(plainText.toString())));
    }
}

```



以上实现了两种认证方式，一种是以形参的形式进行解析验证，另外一种是在没有形参却使用了api签名注解的话，使用请求头信息进行验证。


### 实测



1. 我们这里没有客户端实，就直接请求接口获取签名数据。

```java
@RestController
@RequestMapping("/signature")
@Slf4j
public class signatureController {

    private final BaseSignedService baseSignedService;

    @Autowired
    public signatureController(BaseSignedService baseSignedService) {
        this.baseSignedService = baseSignedService;
    }

    @RequestMapping("base")
    public SignedParam base(@RequestBody(required = false) String data) {

        SignedParam signedParam = new SignedParam();
        signedParam.setAppId(MyApplicationRunner.APP_ID);
        signedParam.setData(data);
        signedParam.setTimestamp(System.currentTimeMillis() / 1000);
        signedParam.setNonce(new Random().nextInt());

        Map<String, Object> map = null;
        try {
            map = baseSignedService.object2Map(signedParam);
            String signature = baseSignedService.getSignature(MyApplicationRunner.APP_ID, map);
            signedParam.setSignature(signature);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return signedParam;
    }


    @PostMapping("param")
    @SignedMapping
    public CommonResponse<String> testHasSignatureParam(@RequestBody(required = false) SignedParam signedParam) {
        String data = signedParam.getData();

        return new CommonResponse<>(CommonResultEnum.SUCCESS, data);
    }

    @GetMapping("head")
    @SignedMapping
    public CommonResponse<String> testHasSignatureHead(@RequestBody(required = false) String data) {

        return new CommonResponse<>(CommonResultEnum.SUCCESS, data);
    }

    @GetMapping("resubmit")
    @SignedMapping(resubmit = true)
    public CommonResponse<String> testResubmit(@RequestBody(required = false) String data) {

        return new CommonResponse<>(CommonResultEnum.SUCCESS, data);
    }

}
```



2. 启动初始化APP_ID 和APP_SECRET

```java
@Component
@Slf4j
public class MyApplicationRunner implements ApplicationRunner {

    @Autowired
    private RedisUtil redisUtil;
    public static String APP_ID;
    public static String APP_SECRET;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        initAppId();
    }

    public void initAppId() {
        // You can initialize the cache here,
        // for example, write the appIds and appSecrets to redis from db
        log.info("Initialization the appId and appSecret...");
        redisUtil.set(APP_ID, APP_SECRET);
    }

    @Value("${signature.app.id:APP_ID_TEST}")
    public void setAppId(String appId) {
        APP_ID = appId;
    }

    @Value("${signature.app.secret:APP_SECRET_TEST}")
    public void setAppSecret(String appSecret) {
        APP_SECRET = appSecret;
    }
}

```



3. 获取签名数据

![image-20201125153643470](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20201125153643470.png)

以 形参数据进行请求, 第一次结果

![image-20201125153701039](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20201125153701039.png)



第二次提示重复请求

![image-20201125153740412](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20201125153740412.png)



以请求头的形式请求,(这里需要重新获取签名数据，客户端即重新生成)，第一次结果

![image-20201125153944735](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20201125153944735.png)



第二次：

![image-20201125154028056](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20201125154028056.png)

重复的请求我们配置@SignedMapping(resubmit = true)即可。



至此，API签名的设计到实现就完成了。

源码：[Cloud脚手架](https://gitee.com/MysticalYu/cloud)的Auth模块里面

