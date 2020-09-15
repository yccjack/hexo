---
title: 解决jar包冲突的简单办法
categories: java
tags: [java]
date: 2020-05-07 
cover: https://mysticalyu.gitee.io/pic/img/cg-.jpg
description: 解决jar包冲突的简单办法-- 在使用log4j.properties时，pom中导入的一些jar会产生log4j类的冲突报错，以下是一个简单的pom配置：
---

![](https://mysticalyu.gitee.io/pic/img/cg-.jpg)

解决jar包冲突的简单办法-- 在使用log4j.properties时，pom中导入的一些jar会产生log4j类的冲突报错，以下是一个简单的pom配置：



<!-- more -->



# 解决jar包冲突的简单办法

场景：在使用log4j.properties时，pom中导入的一些jar会产生log4j类的冲突报错，以下是一个简单的pom配置：

```
复制<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis-reactive</artifactId>

        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-quartz</artifactId>

        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.kudu</groupId>
            <artifactId>kudu-client</artifactId>
            <version>1.7.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>1.2.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.5.6</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.10.0</version>
        </dependency>

        <dependency>
            <groupId>com.nari</groupId>
            <artifactId>front-param</artifactId>
            <version>2.9</version>

        </dependency>

        <dependency>
            <groupId>com.oracle</groupId>
            <artifactId>ojdbc6</artifactId>
            <version>11.2.0.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

<!--        <dependency>-->
<!--            <groupId>com.alibaba.boot</groupId>-->
<!--            <artifactId>nacos-discovery-spring-boot-starter</artifactId>-->
<!--            <version>0.2.4</version>-->
<!--        </dependency>-->
    </dependencies>
```

运行项目会出现一下冲突：

![image-20200519094027451](/images/storage/image-20200519094027451.png)

这里提示org-slf4j 冲突； 使用mvn dependency:tree 查看依赖树：

```
复制mvn dependency:tree > tree.txt
```

tree.txt:(信息比较多，就截取一点。)

```
复制
[INFO] com.nari:bgservice-task:jar:1.2.1
[INFO] +- org.springframework.boot:spring-boot-starter-data-redis-reactive:jar:2.0.4.RELEASE:compile
[INFO] |  \- org.springframework.boot:spring-boot-starter-data-redis:jar:2.0.4.RELEASE:compile
[INFO] |     +- org.springframework.data:spring-data-redis:jar:2.0.9.RELEASE:compile
[INFO] |     |  +- org.springframework.data:spring-data-keyvalue:jar:2.0.9.RELEASE:compile
[INFO] |     |  \- org.springframework:spring-oxm:jar:5.0.8.RELEASE:compile
[INFO] |     \- io.lettuce:lettuce-core:jar:5.0.4.RELEASE:compile
[INFO] |        \- io.projectreactor:reactor-core:jar:3.1.8.RELEASE:compile
[INFO] |           \- org.reactivestreams:reactive-streams:jar:1.0.2:compile
[INFO] +- org.springframework.boot:spring-boot-starter-quartz:jar:2.0.4.RELEASE:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter:jar:2.0.4.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot:jar:2.0.4.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-autoconfigure:jar:2.0.4.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-starter-logging:jar:2.0.4.RELEASE:compile
[INFO] |  |  |  +- ch.qos.logback:logback-classic:jar:1.2.3:compile
[INFO] |  |  |  |  \- ch.qos.logback:logback-core:jar:1.2.3:compile
[INFO] |  |  |  +- org.apache.logging.log4j:log4j-to-slf4j:jar:2.10.0:compile
[INFO] |  |  |  |  \- org.apache.logging.log4j:log4j-api:jar:2.10.0:compile
[INFO] |  |  |  \- org.slf4j:jul-to-slf4j:jar:1.7.25:compile
[INFO] |  |  +- javax.annotation:javax.annotation-api:jar:1.3.2:compile
[INFO] |  |  +- org.springframework:spring-core:jar:5.0.8.RELEASE:compile
[INFO] |  |  |  \- org.springframework:spring-jcl:jar:5.0.8.RELEASE:compile
[INFO] |  |  \- org.yaml:snakeyaml:jar:1.19:runtime
[INFO] |  +- org.springframework:spring-context-support:jar:5.0.8.RELEASE:compile
[INFO] |  |  +- org.springframework:spring-beans:jar:5.0.8.RELEASE:compile
[INFO] |  |  \- org.springframework:spring-context:jar:5.0.8.RELEASE:compile
[INFO] |  |     \- org.springframework:spring-expression:jar:5.0.8.RELEASE:compile
[INFO] |  +- org.springframework:spring-tx:jar:5.0.8.RELEASE:compile
[INFO] |  \- org.quartz-scheduler:quartz:jar:2.3.0:compile
[INFO] |     \- com.mchange:mchange-commons-java:jar:0.2.11:compile
[INFO] +- org.springframework.boot:spring-boot-starter-data-jpa:jar:2.0.4.RELEASE:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter-aop:jar:2.0.4.RELEASE:compile
[INFO] |  |  +- org.springframework:spring-aop:jar:5.0.8.RELEASE:compile
[INFO] |  |  \- org.aspectj:aspectjweaver:jar:1.8.13:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter-jdbc:jar:2.0.4.RELEASE:compile
[INFO] |  |  +- com.zaxxer:HikariCP:jar:2.7.9:compile
[INFO] |  |  \- org.springframework:spring-jdbc:jar:5.0.8.RELEASE:compile
[INFO] |  +- org.hibernate:hibernate-core:jar:5.2.17.Final:compile
[INFO] |  |  +- org.jboss.logging:jboss-logging:jar:3.3.2.Final:compile
[INFO] |  |  +- org.hibernate.javax.persistence:hibernate-jpa-2.1-api:jar:1.0.2.Final:compile
[INFO] |  |  +- org.javassist:javassist:jar:3.22.0-GA:compile
[INFO] |  |  +- antlr:antlr:jar:2.7.7:compile
[INFO] |  |  +- org.jboss:jandex:jar:2.0.3.Final:compile
[INFO] |  |  +- com.fasterxml:classmate:jar:1.3.4:compile
[INFO] |  |  +- dom4j:dom4j:jar:1.6.1:compile
[INFO] |  |  \- org.hibernate.common:hibernate-commons-
.....
```

看到这里发现不是方便查找需要的jar包，这里可以使用mvn dependency:tree -Dincludes 限制；

```
复制mvn dependency:tree -Dincludes=org.slf4j
```

![image-20200519112242971](/images/storage/image-20200519112242971.png)

从图中发现org-slf4j的版本是一样的，先不管这个，先排除所有试试；

```
复制<dependency>
           <groupId>org.apache.zookeeper</groupId>
           <artifactId>zookeeper</artifactId>
           <version>3.5.6</version>
           <exclusions>
               <exclusion>
                   <groupId>org.slf4j</groupId>
                   <artifactId>slf4j-log4j12</artifactId>
               </exclusion>
               <exclusion>
                   <groupId>log4j</groupId>
                   <artifactId>log4j</artifactId>
               </exclusion>
           </exclusions>
       </dependency>
```

再次运行 如果发现依然报错：

![image-20200519112516682](/images/storage/image-20200519112516682.png)

再次寻找冲突问题：

这次把焦点放在logback上；

执行

```
复制mvn dependency:tree -Dverbose -Dincludes=ch.qos.logback
```

![image-20200519112939624](/images/storage/image-20200519112939624.png)

发现这个logback 1.2.3的包，将其排除：

```
复制<dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-quartz</artifactId>
          <exclusions>
              <exclusion>
                  <groupId>ch.qos.logback</groupId>
                  <artifactId>logback-classic</artifactId>
              </exclusion>
          </exclusions>
      </dependency>
```

此次运行将正常运行不在报jar报冲突；；

处理jar冲突：

> 简介:处理jar包依赖冲突,首先,对于多个jar包都引用同一jar包的情况,最好是在程序中显式定义被共同引用的jar包的依赖,来统一版本号,方便维护
>
> 如果A和B都依赖同一jar包C,可能会出现两种情况
>
> 1.A和B引用的C版本相同,这时按照pom定义顺序选择第一个即可,没有冲突问题,如果在项目的maven中显示定义了C依赖,那么用选择项目定义的依赖,反正version都一样,没有影响
>
> 2.A和B依赖的C版本不同,选择版本高的那个,这时会出现两种结果
>
> (1) 高版本兼容低版本,所以不会出现问题
>
> (2)高版本不兼容低版本,假如A依赖C2版本,B依赖C3版本,C3不兼容C2,maven选择了高版本C3,对A来说会出现问题
>
> 有3种解决方法
>
> 　　[1]提升A版本,找到依赖C3的A版本
>
> 　　[2]如果B版本也可依赖C2,在项目的maven中显示定义对C2的依赖,这样所有都使用C2版本
>
> 　　[3]如果B版本不支持C2版本,只能降低B版本,找到依赖C2的B版本
>
> 　　从功能性和可维护性考虑,高版本提供的功能更多,bug更少,优先考虑1
>
> 　　再考虑2
>
> 　　最后考虑3

- **作者:** MysticalYcc