---
title: maven多配置
categories: java 
tags: [maven,java] 
description: 开发的时候总会遇到不同环境需要的配置文件是不同的，maven提供的多配置文件打包所需要的plugin，配置一下即可搞定不同环境打包不同配置。
---


**resources下面增加多配置的文件夹；如下：**

![](/images/storage/17H9FFPRJAVAC347UDD600FV8I.png)

---
apollo下的application.yml
```ymal
app:
  id: DataProcess
apollo:
  #apollo meta server地址
  meta: http://192.168.6.205:8080/
  #apollo本地缓存路径  默认: windows C:\opt\data\  Linux: \opt\data\,  在Apollo服务不可用时,会从本地恢复配置
  cacheDir: doc
```
local下的appliction.yml
```yaml
#端口号
server:
  port: 8688

spring:
  application:
    name: Content-FrontSwitch
  #数据库
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      url: jdbc:mysql://192.168.6.226:3306/Ti1CmdProcess?useUnicode=true&characterEncoding=UTF-8&useSSL=true
      username: root
      password: 123456
      driver-class-name: com.mysql.jdbc.Driver
      #连接池配置
      initialSize: 5
      max-active: 100
      min-idle: 5
      max-wait: 60000
      timeBetweenEvictionRunsMillis: 60000
      pool-prepared-statements: true
      max-pool-prepared-statement-per-connection-size: 100
      minEvictableIdleTimeMillis: 300000
      validationQuery: SELECT 1 FROM DUAL
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false

mybatis-plus:
  mapper-locations: classpath:mapper/*.xml
#服务注册
eureka:
  instance:
    hostname: 192.168.6.205
    prefer-ip-address: true
    lease-expiration-duration-in-seconds: 3
    lease-renewal-interval-in-seconds: 1
  client:
    registryFetchIntervalSeconds: 5
    service-url:
      defaultZone: http://${eureka.instance.hostname}:10010/eureka/

management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: ALWAYS
logging:
  level:
    root: info
    com.jsc.content.frontswitch.mapper: debug
socket:
  ti3:
    port: 8889
    filePath: /u02/ycc/
    timeOut: 1000
    #白名单
    whitelist:
      - 192.168.16.201
      - 192.168.16.134
      - 192.168.99.166
      - 192.168.6.17
      - 127.0.0.1
      - 192.168.6.99
    ki: {ki1: abcdefghijklmnop,ki2: 1234567890ABCDEF,ki3: 1234567890ABCDEF}
    kiPassword: {ki1-password: 1234567890123456,ki2-password: 1234567890123456,ki3-password: 1234567890123456}
    kiKey: 0X27,0X3b,0X9e,0X52,0Xc0,0x4e,0xde,0x75,0x00,0xcb,0x4c,0x4d,0x57,0xfb,0x08,0x9e,0x27,0x3b,0x9e,0x52,0xc0,0x4e,0xde,0x75
    licId: LIC0
    leaId: IMS_YD_LEA
    #设备id
    equId: 630000-020601
    #更新数据库时间
    shedulTime: 30000
    #线程池大小
    maxNumPoolSize: 100
    #socket等待超时时间
    timer: 20000



seaweedfs:
  host: '192.168.6.226'
  port: 9333
```

---

修改pom文件：
```pom
 <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <excludes>
                    <exclude>apollo/*</exclude>
                    <exclude>local/*</exclude>
                </excludes>
            </resource>
            <resource>
                <directory>src/main/resources/${profileActive}</directory>
            </resource>
        </resources>
        <finalName>Content-FrontSwitch</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <includeSystemScope>true</includeSystemScope>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <!--profile -->
    <profiles>
        <profile>
            <id>apollo</id>
            <properties>
                <profileActive>apollo</profileActive>
            </properties>
        </profile>
        <profile>
            <id>local</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <profileActive>local</profileActive>
            </properties>

        </profile>

    </profiles>

```

![](/images/storage/13LRJJGJA6I8CCIPC97GJQ265I.png)

![](/images/storage/38GIFCQ5RD2CLPBEFJAGEOQRN5.png)

**打包时选择需要的配置勾选并取消另外一个profile即可打包对应的配置文件，同样也可以增加mvn 
package -P apollo,!local 来使用需要的配置文件(jenkins打不同环境的包)**