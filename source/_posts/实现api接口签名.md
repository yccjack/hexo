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

### 实现

springboot +Annotation +aop 实现对请求的拦截处理。

对于算法，其实客户端和服务端达成一致即可。

实现之前需要知道以下知识点。

#### Annotation 

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



#### AOP

## 什么是 AOP

AOP （Aspect Orient Programming）,直译过来就是 面向切面编程。AOP 是一种编程思想，是面向对象编程（OOP）的一种补充。面向对象编程将程序抽象成各个层次的对象，而面向切面编程是将程序抽象成各个切面。
![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/758949-20190529225344641-152289598.png)

从该图可以很形象地看出，所谓切面，相当于应用对象间的横切点，我们可以将其单独抽象为单独的模块。



我们在写代码逻辑的时候，都是自上而下，顺序执行(不考虑重排序的问题)；而我们需要一种能在顺序执行的时候横跨这些逻辑执行别的通用逻辑，这个时候就需要AOP，那么AOP如何实现？

## AOP 实现分类



AOP 要达到的效果是，保证开发者不修改源代码的前提下，去为系统中的业务组件添加某种通用功能。AOP 的本质是由 AOP 框架修改业务组件的多个方法的源代码，看到这其实应该明白了，AOP 其实就是前面一篇文章讲的代理模式的典型应用。
按照 AOP 框架修改源代码的时机，可以将其分为两类：

- 静态 AOP 实现， AOP 框架在编译阶段对程序源代码进行修改，生成了静态的 AOP 代理类（生成的 *.class 文件已经被改掉了，需要使用特定的编译器），比如 AspectJ。
- 动态 AOP 实现， AOP 框架在运行阶段对动态生成代理对象（在内存中以 JDK 动态代理，或 CGlib 动态地生成 AOP 代理类），如 SpringAOP。

关于Spring提供的AOP方案的实现的很多概念大家自行百度。

这里通过代码来熟悉。



1. 首先定义一个抽象类使用@Pointcut来定义切点，再定义一个前置处理 @Before("mapping() ")来做前置增强,我们这里不做后置或者环绕，原理类似。

```java
public abstract class SignedService {
    @Value("${signature.algorithm:HmacSHA1}")
    protected String ALGORITHM;
    @Value("${signature.time-diff-max:100}")
    protected long TIME_DIFF_MAX;

    @Value("${share.key}")
    protected String shareKey;
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

 有这集中切点匹配方式

![image-20201124174554525](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20201124174554525.png)

我们这里使用@within(com.mystical.cloud.auth.signature.annotation.SignedMapping)和@annotation(com.mystical.cloud.auth.signature.annotation.SignedMapping)来匹配，这两个注解的意思就是 类含有某种类型的注解或者含有某个注解类型的方法。




----





