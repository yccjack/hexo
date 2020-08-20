---
title: redissionq启动报错涉及安全检查问题排查
categories: java
tags: [java] 
description: redissionq启动报错涉及安全检查问题排查
---





[size=medium]初涉spring boot/cloud，最近有个项目启动时报redis无法连接，但又不影响正常使用
检查日志发现有如下报错信息：[/size]

	at org.springframework.boot.actuate.health.RedisHealthIndicator.doHealthCheck(RedisHealthIndicator.java:52)
	at org.springframework.boot.actuate.health.AbstractHealthIndicator.health(AbstractHealthIndicator.java:43)
	at org.springframework.boot.actuate.health.CompositeHealthIndicator.health(CompositeHealthIndicator.java:68)
	at org.springframework.boot.actuate.endpoint.HealthEndpoint.invoke(HealthEndpoint.java:85)


[size=medium]项目中没有明确指定Actuator监控Redis，因此怀疑是Actuator发现项目有redis时，默认自动监控的。

有两个解决方案：
第一种：允许Actuator监控Redis连接
在application.yml中增加配置：[/size]

spring:
  redis:
    database: 0
    host: 127.0.0.1
    port: 6379
    password: 
    timeout: 0
    pool:
      max-active: 8
      max-wait: -1
      max-idle: 8
      min-idle: 0


[size=medium]第二种：禁止Actuator监控Redis连接
在application.yml中增加配置：[/size]

management:
  health:
    redis:
      enabled: false



[size=medium]完整的报错信息如下：[/size]

org.springframework.data.redis.RedisConnectionFailureException: Cannot get Jedis connection; nested exception is redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool
	at org.springframework.data.redis.connection.jedis.JedisConnectionFactory.fetchJedisConnector(JedisConnectionFactory.java:204)
	at org.springframework.data.redis.connection.jedis.JedisConnectionFactory.getConnection(JedisConnectionFactory.java:348)
	at org.springframework.data.redis.core.RedisConnectionUtils.doGetConnection(RedisConnectionUtils.java:129)
	at org.springframework.data.redis.core.RedisConnectionUtils.getConnection(RedisConnectionUtils.java:92)
	at org.springframework.data.redis.core.RedisConnectionUtils.getConnection(RedisConnectionUtils.java:79)
	at org.springframework.boot.actuate.health.RedisHealthIndicator.doHealthCheck(RedisHealthIndicator.java:52)
	at org.springframework.boot.actuate.health.AbstractHealthIndicator.health(AbstractHealthIndicator.java:43)
	at org.springframework.boot.actuate.health.CompositeHealthIndicator.health(CompositeHealthIndicator.java:68)
	at org.springframework.boot.actuate.endpoint.HealthEndpoint.invoke(HealthEndpoint.java:85)
	at org.springframework.boot.actuate.endpoint.HealthEndpoint.invoke(HealthEndpoint.java:35)
	at org.springframework.boot.actuate.endpoint.jmx.DataEndpointMBean.getData(DataEndpointMBean.java:46)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at sun.reflect.misc.Trampoline.invoke(MethodUtil.java:71)
	at sun.reflect.GeneratedMethodAccessor186.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at sun.reflect.misc.MethodUtil.invoke(MethodUtil.java:275)
	at javax.management.modelmbean.RequiredModelMBean$4.run(RequiredModelMBean.java:1252)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.security.ProtectionDomain$JavaSecurityAccessImpl.doIntersectionPrivilege(ProtectionDomain.java:80)
	at javax.management.modelmbean.RequiredModelMBean.invokeMethod(RequiredModelMBean.java:1246)
	at javax.management.modelmbean.RequiredModelMBean.invoke(RequiredModelMBean.java:1085)
	at org.springframework.jmx.export.SpringModelMBean.invoke(SpringModelMBean.java:90)
	at javax.management.modelmbean.RequiredModelMBean.getAttribute(RequiredModelMBean.java:1562)
	at org.springframework.jmx.export.SpringModelMBean.getAttribute(SpringModelMBean.java:109)
	at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.getAttribute(DefaultMBeanServerInterceptor.java:647)
	at com.sun.jmx.mbeanserver.JmxMBeanServer.getAttribute(JmxMBeanServer.java:678)
	at javax.management.remote.rmi.RMIConnectionImpl.doOperation(RMIConnectionImpl.java:1445)
	at javax.management.remote.rmi.RMIConnectionImpl.access$300(RMIConnectionImpl.java:76)
	at javax.management.remote.rmi.RMIConnectionImpl$PrivilegedOperation.run(RMIConnectionImpl.java:1309)
	at javax.management.remote.rmi.RMIConnectionImpl.doPrivilegedOperation(RMIConnectionImpl.java:1401)
	at javax.management.remote.rmi.RMIConnectionImpl.getAttribute(RMIConnectionImpl.java:639)
	at sun.reflect.GeneratedMethodAccessor85.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at sun.rmi.server.UnicastServerRef.dispatch(UnicastServerRef.java:357)
	at sun.rmi.transport.Transport$1.run(Transport.java:200)
	at sun.rmi.transport.Transport$1.run(Transport.java:197)
	at java.security.AccessController.doPrivileged(Native Method)
	at sun.rmi.transport.Transport.serviceCall(Transport.java:196)
	at sun.rmi.transport.tcp.TCPTransport.handleMessages(TCPTransport.java:568)
	at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run0(TCPTransport.java:826)
	at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.lambda$run$0(TCPTransport.java:683)
	at java.security.AccessController.doPrivileged(Native Method)
	at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run(TCPTransport.java:682)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool
	at redis.clients.util.Pool.getResource(Pool.java:53)
	at redis.clients.jedis.JedisPool.getResource(JedisPool.java:226)
	at redis.clients.jedis.JedisPool.getResource(JedisPool.java:16)
	at org.springframework.data.redis.connection.jedis.JedisConnectionFactory.fetchJedisConnector(JedisConnectionFactory.java:194)
	... 50 common frames omitted
Caused by: redis.clients.jedis.exceptions.JedisConnectionException: java.net.ConnectException: Connection refused: connect
	at redis.clients.jedis.Connection.connect(Connection.java:207)
	at redis.clients.jedis.BinaryClient.connect(BinaryClient.java:93)
	at redis.clients.jedis.BinaryJedis.connect(BinaryJedis.java:1767)
	at redis.clients.jedis.JedisFactory.makeObject(JedisFactory.java:106)
	at org.apache.commons.pool2.impl.GenericObjectPool.create(GenericObjectPool.java:888)
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:432)
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:361)
	at redis.clients.util.Pool.getResource(Pool.java:49)
	... 53 common frames omitted
Caused by: java.net.ConnectException: Connection refused: connect
	at java.net.DualStackPlainSocketImpl.waitForConnect(Native Method)
	at java.net.DualStackPlainSocketImpl.socketConnect(DualStackPlainSocketImpl.java:85)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
	at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:172)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
	at java.net.Socket.connect(Socket.java:589)
	at redis.clients.jedis.Connection.connect(Connection.java:184)
	... 60 common frames omitte