---

title: 动态代理在代码中的应用
categories: java
tags: [java] 
date: 2020-01-11
cover: https://mysticalyu.gitee.io/pic/img/cat-lazy-fengmian.jpg

---
动态代理在代码中的应用



<!-- more -->



## <font face="楷体" color=#FF0000>动态代理在代码中的应用.</font>

 <font color=#008000  >先看一段代码</font> 

<font color=#008000>MixedRedisHelperFactoryBean</font>:

```java
@Component
public class MixedRedisHelperFactoryBean implements FactoryBean<MixedRedisHelper>, InitializingBean, InvocationHandler {

    @Autowired
    private ShardedJedisTemplate shardedJedisTemplate;

    @Autowired
    private JedisClusterTemplate jedisClusterTemplate;

    private MixedRedisHelper helper;


    @Override
    public void afterPropertiesSet() throws Exception {
        helper = (MixedRedisHelper) Proxy.newProxyInstance(MixedRedisHelperFactoryBean.class.getClassLoader(),
                new Class<?>[]{MixedRedisHelper.class}, this);
    }

    @Override
    public MixedRedisHelper getObject() throws Exception {
        return helper;
    }

    @Override
    public Class<?> getObjectType() {
        return MixedRedisHelper.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        if (Object.class.equals(method.getDeclaringClass())) {
            return method.invoke(this, args);
        }
        Object target;
        //根据开关来决定使用啥shard：cluster
        if (ConfigManager.getBoolean("redis.cluster.standalone", false)) {
            target = jedisClusterTemplate;
        } else {
            target = shardedJedisTemplate;
        }
        return method.invoke(target, args);
    }
}

```

<font color=#008000>MixedRedisHelper</font>:

```java
public interface MixedRedisHelper extends JedisCommands {
}

```

<font color=#008000>ShardedJedisTemplate</font>:

```java
public interface ShardedJedisTemplate extends JedisCommands, Closeable, BinaryJedisCommands {
	List<?> pipelined(PipelineHandler handler);
}

```

<font color=#008000>JedisClusterTemplate</font>:

```java
public interface JedisClusterTemplate extends BasicCommands, BinaryJedisClusterCommands, MultiKeyBinaryJedisClusterCommands,
        JedisClusterBinaryScriptingCommands, JedisCommands, MultiKeyJedisClusterCommands, JedisClusterScriptingCommands {

}

```

<font color=#FF0000 >InitializingBean</font>就是一个初始化类需要重写afterPropertiesSet()；就当是@postconstruct；

所以此处的代码是在类初始化的时候初始化了一个MixedRedisHelperFactoryBean，从它实现FactoryBean<MixedRedisHelper>可以知道它是用来生成MixedRedisHelper，就是说在使用@Autowired注解使用MixedRedisHelper的时候会使用此Factory生产的MixedRedisHelper的实例。

在来看它的初始方法

```java
  @Override
    public void afterPropertiesSet() throws Exception {
        helper = (MixedRedisHelper) Proxy.newProxyInstance(MixedRedisHelperFactoryBean.class.getClassLoader(),
                new Class<?>[]{MixedRedisHelper.class}, this);
    }
```

这里使用了动态代理。。Proxy的用法请google。所以此Factory中的MixedRedisHelper对象是有代理生成的，也即我们执行MixedRedisHelper这个对象实例的时候会执行invoke方法。

```java
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        if (Object.class.equals(method.getDeclaringClass())) {
            return method.invoke(this, args);
        }
        Object target;
        //根据开关来决定使用啥shard：cluster
        if (ConfigManager.getBoolean("redis.cluster.standalone", false)) {
            target = jedisClusterTemplate;
        } else {
            target = shardedJedisTemplate;
        }
        return method.invoke(target, args);
    }
```

这里的invoke又使用了注入JedisClusterTemplate，ShardedJedisTemplate，这样就达到了通过配置文件来区分使用哪个，其中还有其他的请等等。。

待续！