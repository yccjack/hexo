---
title: Spring是如何启用aop切面
categories: spring
tags: [java] 
date: 2019-10-17
cover: https://gschaos.club/ico/img/d-long-5k.jpg
description: Spring是如何启用aop切面（比如声明式事务），而对我们的bean实现代理的呢？
---

Spring是如何启用aop切面（比如声明式事务），而对我们的bean实现代理的呢？



<!-- more -->



# Spring是如何启用aop切面（比如声明式事务），而对我们的bean实现代理的呢？

首先从后面说实现原理，通过aop包下的AspectJAwareAdvisorAutoProxyCreator 继承自AbstractAdvisorAutoProxyCreator 又继承 ***AbstractAutoProxyCreator\***类，而***AbstractAutoProxyCreator\***中有个方法

```java
/**
* Create an AOP proxy for the given bean.
* @param beanClass the class of the bean
* @param beanName the name of the bean
* @param specificInterceptors the set of interceptors that is
* specific to this bean (may be empty, but not null)
* @param targetSource the TargetSource for the proxy,
* already pre-configured to access the bean
* @return the AOP proxy for the bean
* @see #buildAdvisors
*/
protected Object createProxy(Class<?> beanClass, @Nullable String beanName,
			@Nullable Object[] specificInterceptors, TargetSource targetSource)
```

创建一个类的代理，其中创建ProxyFactory类的对象，调用其getProxy(ClassLoader classLoader)方法，如下：

```java
/**
* Create a new proxy according to the settings in this factory.
* <p>Can be called repeatedly. Effect will vary if we've added
* or removed interfaces. Can add and remove interceptors.
* <p>Uses the given class loader (if necessary for proxy creation).
* @param classLoader the class loader to create the proxy with
* (or {@code null} for the low-level proxy facility's default)
* @return the proxy object
*/
public Object getProxy(@Nullable ClassLoader classLoader) {
    return createAopProxy().getProxy(classLoader);
}
```

其中createAopProxy()来自其父类ProxyCreatorSupport，具体如下：

```java
/**
* Subclasses should call this to get a new AOP proxy. They should <b>not</b>
* create an AOP proxy with {@code this} as an argument.
*/
protected final synchronized AopProxy createAopProxy() {
    if (!this.active) {
        activate();
    }
    return getAopProxyFactory().createAopProxy(this);
}
```

getAopProxyFactory()返回了AopProxyFactory的唯一实现DefaultAopProxyFactory，然后其调用createAopProxy方法，如下：

```java
@Override
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
        Class<?> targetClass = config.getTargetClass();
        if (targetClass == null) {
            throw new AopConfigException("TargetSource cannot determine target class: " +
                    "Either an interface or a target is required for proxy creation.");
        }
        if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
            return new JdkDynamicAopProxy(config);
        }
        return new ObjenesisCglibAopProxy(config);
    }
    else {
        return new JdkDynamicAopProxy(config);
    }
}
```

可以看到，根据目标类的条件，选择创建其Jdk动态代理或者基于Cglib的代理。查看DefaultAopProxyFactory类的说明可以看到：

```java
Default AopProxyFactory implementation, creating either a CGLIB proxy or a JDK dynamic proxy.
Creates a CGLIB proxy if one the following is true for a given AdvisedSupport instance:
the optimize flag is set
the proxyTargetClass flag is set
no proxy interfaces have been specified
In general, specify proxyTargetClass to enforce a CGLIB proxy, or specify one or more interfaces to use a JDK dynamic proxy.
```

默认的AopProxyFactory实现，创建CGLIB代理或JDK动态代理。

如果给定的AdvisedSupport实例满足以下条件之一，则创建CGLIB代理：

- 设置了优化标志
- 设置了proxyTargetClass标志
- 没有指定代理接口

通常，指定proxyTargetClass来强制执行CGLIB代理，或者指定一个或多个接口来使用JDK动态代理。

符合if条件逻辑。

## 实现过程理清楚了，那么，spring是如何启用aop功能的呢？

如果是使用springboot的情况下，可以看到spring-boot-autoconfiguer包有一个配置类:

```java
@Configuration
@ConditionalOnClass({ EnableAspectJAutoProxy.class, Aspect.class, Advice.class,
		AnnotatedElement.class })
@ConditionalOnProperty(prefix = "spring.aop", name = "auto", havingValue = "true", matchIfMissing = true)
public class AopAutoConfiguration {

	@Configuration
	@EnableAspectJAutoProxy(proxyTargetClass = false)
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false", matchIfMissing = false)
	public static class JdkDynamicAutoProxyConfiguration {

	}

	@Configuration
	@EnableAspectJAutoProxy(proxyTargetClass = true)
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true", matchIfMissing = true)
	public static class CglibAutoProxyConfiguration {
	}
}
```



可以看到，该配置类会读取配置文件中的spring.aop.auto属性，如果配置为true，或者没有配置，且路径下存在EnableAspectJAutoProxy.class, Aspect.class,

Advice.class，AnnotatedElement.class这些类的时候，该配置类则生效（matchIfMissing = true），然后再读取spring.aop.proxy-target-class的值来确定

使用Cglib还是Jdk动态代理。然后启用注解@**EnableAspectJAutoProxy**，该注解如下：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {

	/**
	 * Indicate whether subclass-based (CGLIB) proxies are to be created as opposed
	 * to standard Java interface-based proxies. The default is {@code false}.
	 */
	boolean proxyTargetClass() default false;

	/**
	 * Indicate that the proxy should be exposed by the AOP framework as a {@code ThreadLocal}
	 * for retrieval via the {@link org.springframework.aop.framework.AopContext} class.
	 * Off by default, i.e. no guarantees that {@code AopContext} access will work.
	 * @since 4.3.1
	 */
	boolean exposeProxy() default false;
}
```

可以看到该注解导入了**AspectJAutoProxyRegistrar**类（@Import(AspectJAutoProxyRegistrar.class)），而AspectJAutoProxyRegistrar类最终将***AnnotationAwareAspectJAutoProxyCreator\***类注入了

spring中，它是AspectJAwareAdvisorAutoProxyCreator的子类。所以最终回到前文所述的，完成对bean的代理实现。

## 那么非springboot环境是如何启用的呢？

通常在spring xml配置文件加入<aop:aspectj-autoproxy/>标签启用，而这个标签对应的解析器为：AopNamespaceHandler，它是位于spring aop包下。

```java
public class AopNamespaceHandler extends NamespaceHandlerSupport {

	/**
	 * Register the {@link BeanDefinitionParser BeanDefinitionParsers} for the
	 * '{@code config}', '{@code spring-configured}', '{@code aspectj-autoproxy}'
	 * and '{@code scoped-proxy}' tags.
	 */
	@Override
	public void init() {
		// In 2.0 XSD as well as in 2.1 XSD.
		registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
		registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
		registerBeanDefinitionDecorator("scoped-proxy", new ScopedProxyBeanDefinitionDecorator());

		// Only in 2.0 XSD: moved to context namespace as of 2.1
		registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
	}
}
```

该类注册了AspectJAutoProxyBeanDefinitionParser类，而AspectJAutoProxyBeanDefinitionParser类又注册了（具体代码就不贴了）AnnotationAwareAspectJAutoProxyCreator，达到了同样的效果。