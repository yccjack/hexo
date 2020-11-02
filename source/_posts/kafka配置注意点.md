---
title: kafka配置注意点
cover: https://gschaos.club/ico/img/wayne-wan-.jpg
---



 kafka无法启动,Cannot assign requested address.

# Cannot assign requested address.

搭建kafka时，需要对配置文件进行修改，在服务器上搭建时候，配置文件中需要配置

> ：listeners
> ：advertised.host.name
> ：advertised.listeners

这几项，其中 listeners 配置的ip和其他两者相同时启动kafka会报这样的错误：
![kafka](https://img-blog.csdnimg.cn/20200514142306551.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE3MjM4NDQ5,size_16,color_FFFFFF,t_70#pic_center)
Cannot assign requested address。这个错误的原因是
基于OpenStack的机器的情况下的虚拟机对外ip[暴露的ip]和真实ip[ifconfig显示的ip]可能只是映射关系，用户访问对外ip时，OpenStack会转发到对应的真实ip实现访问。但此时如果 Kafka server.properties配置中的listeners=PLAINTEXT://对外IP:9092中的ip配置为[对外ip]的时候无法启动，因为socket无法绑定监听；

## 解决

查看内网IP地址：ifconfig
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020051414254043.png#pic_center)
修改配置文件：

```
listeners=PLAINTEXT://内网:9092               
advertised.host.name=外网ip或者域名
advertised.listeners=PLAINTEXT://外网ip或者域名:9092    
123
```

重新启动：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200514142624528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE3MjM4NDQ5,size_16,color_FFFFFF,t_70#pic_center)

启动成功。。。