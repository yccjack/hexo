---
title: 跨域问题
categories: java
tags: [java] 
date: 2020-06-05
update: 2020-09-10
cover: https://mysticalyu.gitee.io/pic/img/20200407235505-www-ycfcg-com-2.jpg


---

前后端分离跨域问题的几种解决方案.为什么会出现跨域问题?出于浏览器的同源策略限制。 同源策略（Sameoriginpolicy）是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。可以说Web是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现。同源策略会阻止一个域的javascript脚本和另外一个域的内容进行交互。所谓同源（即指在同一个域）就是两个页面具有相同的协议（protocol），主机（host）和端口号（port） 。



 <!-- more -->



# 跨域问题

> 作者：cnsyear
>
> 链接 ：https://cnsyear.com/posts/a74bc789.html

# 前后端分离跨域问题的几种解决方案

> 前后端分离跨域问题的几种解决方案

## 一、为什么会出现跨域问题?

出于浏览器的同源策略限制。

> 同源策略（Sameoriginpolicy）是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。可以说Web是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现。同源策略会阻止一个域的javascript脚本和另外一个域的内容进行交互。所谓同源（即指在同一个域）就是两个页面具有相同的协议（protocol），主机（host）和端口号（port） 。

## 二、什么是跨域?

当一个请求url的协议、域名、端口三者之间任意一个与当前页面url不同即为跨域。

| 当前页面url               | 被请求页面url                   | 是否跨域 | 原因                           |
| ------------------------- | ------------------------------- | -------- | ------------------------------ |
| http://www.test.com/      | http://www.test.com/index.html  | 否       | 同源（协议、域名、端口号相同） |
| http://www.test.com/      | https://www.test.com/index.html | 跨域     | 协议不同（http/https）         |
| http://www.test.com/      | http://www.baidu.com/           | 跨域     | 主域名不同（test/baidu）       |
| http://www.test.com/      | http://blog.test.com/           | 跨域     | 子域名不同（www/blog）         |
| http://www.test.com:8080/ | http://www.test.com:7001/       | 跨域     | 端口号不同（8080/7001）        |

两个相同的源之间浏览器默认其是可以相互访问资源和操作DOM的。
两个不同的源之间 若想要相互访问资源或者操作DOM，那么会有一套基础的安全策略的制约。
具体有如下两方面的限制:

- 1.安全性：浏览器要防止当前站点的私密数据不会向其他站点发送 如当前站点的Cookie,LocalStorage,IndexDb不会被发送到其他站点或被其他站点脚本读取到无法跨域获取Dom，无法发送Ajax请求。
- 2.可用性：大型站点的图片，音视频等资源，希望部署在独立服务器上，为缓解当前服务的压力，开放某些特定的方式，访问非同源站点 如：`<img><iframe><link><vedio>`等，可以同src属性跨域访问允许跨域提交表单/或重定向请求。

## 三、解决方案

### 1.后台服务端解决方案

- 方法一：@CrossOrigin

```
复制/**
注意：
1、springMVC的版本要在4.2或以上版本才支持@CrossOrigin
2、非@CrossOrigin没有解决跨域请求问题，而是不正确的请求导致无法得到预期的响应，导致浏览器端提示跨域问题。
3、在Controller注解上方添加@CrossOrigin注解后，仍然出现跨域问题，
解决方案之一就是：在@RequestMapping注解中没有指定Get、Post方式，具体指定后，问题解决。

其中@CrossOrigin中的2个参数：
origins  ： 允许可访问的域列表
maxAge：准备响应前的缓存持续的最大时间（以秒为单位）。

可以配置在Controller上 也可以配置在方法上
*/
@CrossOrigin
@RestController
public class person{
    
    @RequestMapping(method = RequestMethod.GET)
    public String add() {
        // 若干代码
    }
}

@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }
}
```

- 方法2

```
复制package cn.pconline.pcloud.admin.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@Configuration
public class CorsConfig {
    private CorsConfiguration buildConfig() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*"); // 1允许任何域名使用
        corsConfiguration.addAllowedHeader("*"); // 2允许任何头
        corsConfiguration.addAllowedMethod("*"); // 3允许任何方法（post、get等）
        return corsConfiguration;
    }

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", buildConfig()); // 4
        return new CorsFilter(source);
    }
}
```

### 2.Nginx代理服务器，反向代理接口请求

```
复制location /api{ 
  rewrite ^/api/(.*)$ /$1 break;
  proxy_pass http://localhost:8081/;
}
```

### 3.jsonp方式

[推荐文章：说说JSON和JSONP，也许你会豁然开朗，含jQuery用例](https://www.cnblogs.com/dowinning/archive/2012/04/19/json-jsonp-jquery.html)

**JSONP是怎么产生的**：

- 1、浏览器的同源策略限制，Ajax直接请求普通文件存在跨域无权限访问的问题，甭管你是静态页面、动态网页、web服务、WCF，只要是跨域请求，一律不准；
  - 2、不过Web页面上调用js文件时则不受是否跨域的影响（不仅如此，我们还发现凡是拥有”src”这个属性的标签都拥有跨域的能力，比如`、<img>、<iframe>`）；
  - 3、于是可以判断，当前阶段如果想通过纯web端（ActiveX控件、服务端代理、属于未来的HTML5之Websocket等方式不算）跨域访问数据就只有一种可能，那就是在远程服务器上设法把数据装进js格式的文件里，供客户端调用和进一步处理；
  - 4、恰巧我们已经知道有一种叫做JSON的纯字符数据格式可以简洁的描述复杂数据，更妙的是JSON还被js原生支持，所以在客户端几乎可以随心所欲的处理这种格式的数据；
  - 5、这样子解决方案就呼之欲出了，web客户端通过与调用脚本一模一样的方式，来调用跨域服务器上动态生成的js格式文件（一般以JSON为后缀），显而易见，服务器之所以要动态生成JSON文件，目的就在于把客户端需要的数据装入进去。
  - 6、客户端在对JSON文件调用成功之后，也就获得了自己所需的数据，剩下的就是按照自己需求进行处理和展现了，这种获取远程数据的方式看起来非常像AJAX，但其实并不一样。
  - 7、为了便于客户端使用数据，逐渐形成了一种非正式传输协议，人们把它称作JSONP，该协议的一个要点就是允许用户传递一个callback参数给服务端，然后服务端返回数据时会将这个callback参数作为函数名来包裹住JSON数据，这样客户端就可以随意定制自己的函数来自动处理返回数据了。

**具体原理：**

服务端提供的js脚本是动态生成的，这样调用者可以传一个参数过去告诉服务端“我想要一段调用XXX函数的js代码，请你返回给我”，于是服务器就可以按照客户端的需求来生成js脚本并响应了。

**JS调用实例：**

```
复制<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title></title>
    <script type="text/javascript">
    // 得到航班信息查询结果后的回调函数
    var flightHandler = function(data){
        alert('你查询的航班结果是：票价 ' + data.price + ' 元，' + '余票 ' + data.tickets + ' 张。');
    };
    // 提供jsonp服务的url地址（不管是什么类型的地址，最终生成的返回值都是一段javascript代码）
    var url = "http://flightQuery.com/jsonp/flightResult.aspx?code=CA1998&callback=flightHandler";
    // 创建script标签，设置其属性
    var script = document.createElement('script');
    script.setAttribute('src', url);
    // 把script标签加入head，此时调用开始
    document.getElementsByTagName('head')[0].appendChild(script); 
    </script>
</head>
<body>
</body>
</html>
```

我们看到调用的url中传递了一个code参数，告诉服务器我要查的是CA1998次航班的信息，而callback参数则告诉服务器，我的本地回调函数叫做flightHandler，所以请把查询结果传入这个函数中进行调用。
OK，服务器很聪明，这个叫做flightResult.aspx的页面生成了一段这样的代码提供给jsonp.html（服务端的实现这里就不演示了，与你选用的语言无关，说到底就是拼接字符串）：

```
复制flightHandler({
    "code": "CA1998",
    "price": 1780,
    "tickets": 5
});
```

我们看到，传递给flightHandler函数的是一个json，它描述了航班的基本信息。运行一下页面，成功弹出提示窗口，jsonp的执行全过程顺利完成！

**JQuery调用实例：**

```
复制<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
 <html xmlns="http://www.w3.org/1999/xhtml" >
 <head>
     <title>Untitled Page</title>
      <script type="text/javascript" src=jquery.min.js"></script>
      <script type="text/javascript">
     jQuery(document).ready(function(){ 
        $.ajax({
             type: "get",
             async: false,
             url: "http://flightQuery.com/jsonp/flightResult.aspx?code=CA1998",
             dataType: "jsonp",
             jsonp: "callback",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(一般默认为:callback)
             jsonpCallback:"flightHandler",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名，也可以写"?"，jQuery会自动为你处理数据
             success: function(json){
                 alert('您查询到航班信息：票价： ' + json.price + ' 元，余票： ' + json.tickets + ' 张。');
             },
             error: function(){
                 alert('fail');
             }
         });
     });
     </script>
     </head>
  <body>
  </body>
 </html>
```

是不是有点奇怪？为什么我这次没有写flightHandler这个函数呢？而且竟然也运行成功了！哈哈，这就是jQuery的功劳了，jquery在处理jsonp类型的ajax时（还是忍不住吐槽，虽然jquery也把jsonp归入了ajax，但其实它们真的不是一回事儿），自动帮你生成回调函数并把数据取出来供success属性方法来调用，是不是很爽呀？