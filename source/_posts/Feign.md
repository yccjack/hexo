---
title: Feign的请求参数绑定机制
categories: java
tags: [java]
date: 2020-11-12
cover: https://gschaos.club/ico/img/jin-kyoung-shim-pose6-resize.jpg
---



# 从 Feign 使用注意点到 RESTFUL 接口设计规范



场景：在gateway拦截请求获取token调用认证服务认证token正确性。



1. 在auth-service服务端提供验证token的服务接口，它是这个样子的

```java
@RestController
@RequestMapping("auth")
public class AuthController {

    @RequestMapping(value = "/info", method = RequestMethod.GET)
    public CommonResponse<String> auth(String token){
        System.out.println(token);
        String s = JwtTokenUtil.parseToken(token);
        System.out.println(s);
        if("ycc".equals(s)){
            return new CommonResponse<>(CommonResultEnum.SUCCESS);
        }
        return new CommonResponse<>(CommonResultEnum.FAILED_INSUFFICIENT_AUTHORITY);

    }

}
```

这个接口写起来非常简单，但实际 springmvc 做了非常多的兼容，使得这个接口可以接受多种请求方式。

> RequestMapping 代表映射的路径，使用 GET,POST,PUT,DELETE 方式都可以映射到该端点。

> SpringMVC 中常用的请求参数注解有（@RequestParam,@RequestBody,@PathVariable）等。token 被默认当做 @RequestParam。形参 `String token` 由框架使用字节码技术获取 token 这个名称，自动检测请求参数中 key 值为 token 的参数，也可以使用 @RequestParam(“token”) 覆盖变量本身的名称。当我们在 url 中携带 token 参数或者 form 表单中携带 token 参数时，会被获取到。

```bash
POST /hello HTTP/1.1
Host: localhost:8987
Content-Type: application/x-www-form-urlencoded

token=xxxxxxxxxxx
```

或

```bash
GET /auth/info?token=xxxxxx HTTP/1.1
Host: localhost:8987
```



2. 在gateway的一端需要拿到token进行Feign调用auth服务，它是这个样子的

<font color=red>AuthService</font>

```java
@FeignClient(name = "auth-service")
public interface AuthService {

    @RequestMapping(value = "/auth/info", method = RequestMethod.GET)
    CommonResponse<String> getAuthInfo(String token);
}
```

<font color=red>TokenFilter调用处</font>

```java
  log.info("authenticate token start...");
            if(token.contains("Bearer")){
                token = token.substring(token.indexOf("Bearer ")+7);
            }
            CommonResponse<String> authInfo = authService.getAuthInfo(token);
            if (!"200".equals(authInfo.getCode())) {
                exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
                return exchange.getResponse().setComplete();
            }
```



一切看上去都那么对称美好，没有瑕疵。

我们按照写 SpringMVC 的 RestController 的习惯写了一个 FeignClient，按照我们的一开始的想法，由于指定了请求方式是 GET，那么 token应该会作为 QueryString 拼接到 Url 中吧？发出一个这样的 GET 请求：

```bash
GET /auth/info?token=xxx HTTP/1.1
Host: localhost:8987
```



当我们启动项目开始调用的时候，

1. gateway调用处断点

![image-20201112164809208](https://gitee.com/MysticalYu/pic/raw/master/202011/image-20201112164809208.png)

2. 出现的结果是：

![image-20201112164840132](https://gitee.com/MysticalYu/pic/raw/master/202011/image-20201112164840132.png)

并没有按照期望使用 GET 方式发送请求，而是 POST 方式.

既然这样，那我们就让auth-service支持POST请求；

![image-20201112165127155](https://gitee.com/MysticalYu/pic/raw/master/202011/image-20201112165127155.png)



再来调用一次；

1. gateway调用处断点

![image-20201112165218376](https://gitee.com/MysticalYu/pic/raw/master/202011/image-20201112165218376.png)

2. 出现的结果是：

![image-20201112165225995](https://gitee.com/MysticalYu/pic/raw/master/202011/image-20201112165225995.png)



这个时候显示成功了，那么现在去auth-service查看结果；

![image-20201112165306725](https://gitee.com/MysticalYu/pic/raw/master/202011/image-20201112165306725.png)



看了个寂寞，参数并未接收到。





----

## Feign 的请求参数绑定机制

查看文档发现，如果不加默认的注解，Feign 则会对参数默认加上 @RequestBody 注解，而 RequestBody 一定是包含在请求体中的，GET 方式无法包含。所以上述两个现象得到了解释。Feign 在 GET 请求包含 RequestBody 时强制转成了 POST 请求，而不是报错。

理解清楚了这个机制我们就可以在开发 Feign 接口避免很多坑。而解决上述这个问题也很简单

- 在 Feign 接口中为 token添加 @RequestParam(“token”) 注解，token必须指定，Feign 的请求参数不会利用 SpringMVC 字节码的机制自动给定一个默认的名称。
- 由于 Feign 默认使用 @RequestBody，也可以改造 RestController，使用 @RequestBody 接收。但是，请求参数通常是多个，推荐使用上述的 @RequestParam，而 @RequestBody 一般只用于传递对象。



## Feign 绑定复合参数

指定请求参数的类型与请求方式，上述问题的出现实际上是由于在没有理清楚 Feign 内部机制的前提下想当然的和 SpringMVC 进行了类比。同样，在使用对象作为参数时，也需要注意这样的问题。

对于这样的接口

```java
@FeignClient("book")
public interface BookApi {

    @RequestMapping(value = "/book",method = RequestMethod.POST)
    Book book(@RequestBody Book book); // <1>
  
    @RequestMapping(value = "/book",method = RequestMethod.POST)
    Book book(@RequestParam("id") String id,@RequestParam("name") String name); // <2>
  
    @RequestMapping(value = "/book",method = RequestMethod.POST)
    Book book(@RequestParam Map map); // <3>
  
    // 错误的写法
  	@RequestMapping(value = "/book",method = RequestMethod.POST)
    Book book(@RequestParam Book book); // <4>

}
```

> 使用 @RequestBody 传递对象是最常用的方式。

> 如果参数并不是很多，可以平铺开使用 @RequestParam

> 使用 Map，这也是完全可以的，但不太符合面向对象的思想，不能从代码立刻看出该接口需要什么样的参数。

> 错误的用法，Feign 没有提供这样的机制自动转换实体为 Map。



按照这个说法修改我们的接口和FeignClient

<font color=red>AuthController</font>

```java
@RestController
@RequestMapping("auth")
public class AuthController {

    @RequestMapping(value = "/info", method = RequestMethod.GET)
    public CommonResponse<String> auth(String token){
        System.out.println(token);
        String s = JwtTokenUtil.parseToken(token);
        System.out.println(s);
        if("ycc".equals(s)){
            return new CommonResponse<>(CommonResultEnum.SUCCESS);
        }
        return new CommonResponse<>(CommonResultEnum.FAILED_INSUFFICIENT_AUTHORITY);

    }

}
```

<font color=red>AuthService</font>

```java
@FeignClient(name = "auth-service")
public interface AuthService {

    @RequestMapping(value = "/auth/info", method = RequestMethod.GET)
    CommonResponse<String> getAuthInfo(@RequestParam("token") String token);
}
```



再次调试：

![image-20201112170943409](https://gitee.com/MysticalYu/pic/raw/master/202011/image-20201112170943409.png)





![image-20201112171025912](https://gitee.com/MysticalYu/pic/raw/master/202011/image-20201112171025912.png)



调用成功......



## Feign 中使用 @PathVariable 与 RESTFUL 规范

这涉及到一个如何设计 RESTFUL 接口的话题，我们知道在自从 RESTFUL 在 2000 年初被提出来之后，就不乏文章提到资源，契约规范，CRUD 对应增删改查操作等等。下面笔者从两个实际的接口来聊聊自己的看法。

根据 id 查找用户接口：

```java
@FeignClient("user")
public interface UserApi {

    @RequestMapping(value = "/user/{userId}",method = RequestMethod.GET)
    String findById(@PathVariable("id") String userId);

}
```

这应该是没有争议的，注意前面强调的，@PathVariable(“id”) 括号中的 id 不可以忘记。那如果是“根据邮箱查找用户呢”? 很有可能下意识的写出这样的接口：

```java
@FeignClient("user")
public interface UserApi {
  
    @RequestMapping(value = "/user/{email}",method = RequestMethod.GET)
    String findByEmail(@PathVariable("email") String email);

}
```

- 首先看看 Feign 的问题。email 中通常包含’.‘这个特殊字符，如果在路径中包含，会出现意想不到的结果。我不想探讨如何去解决它（实际上可以使用 {email:.+} 的方式), 因为我觉得这不符合设计。
- 再谈谈规范的问题。这两个接口是否是相似的，email 是否应该被放到 path 中？这就要聊到 RESTFUL 的初衷，为什么 userId 这个属性被普遍认为适合出现在 RESTFUL 路径中，因为 id 本身起到了资源定位的作用，他是资源的标记。而 email 不同，它可能是唯一的，但更多的，它是资源的属性，所以，笔者认为不应该在路径中出现非定位性的动态参数。而是把 email 作为 @RequestParam 参数。

## RESUFTL 结构化查询

笔者成功的从 Feign 的话题过度到了 RESTFUL 接口的设计问题，也导致了本文的篇幅变长了，不过也不打算再开一片文章谈了。

再考虑一个接口设计，查询某一个月某个用户的订单，可能还会携带分页参数，这时候参数变得很多，按照传统的设计，这应该是一个查询操作，也就是与 GET 请求对应，那是不是意味着应当将这些参数拼接到 url 后呢？再思考 Feign，正如本文的第二段所述，是不支持 GET 请求携带实体类的，这让我们设计陷入了两难的境地。而实际上参考一些 DSL 语言的设计如 elasticSearch，也是使用 POST JSON 的方式来进行查询的，所以在实际项目中，笔者并不是特别青睐 CRUD 与四种请求方式对应的这种所谓的 RESTFUL 规范，如果说设计 RESTFUL 应该遵循什么规范，那大概是另一些名词，如契约规范和领域驱动设计。

```java
@FeignClient("order")
public interface BookApi {

    @RequestMapping(value = "/order/history",method = RequestMethod.POST)
    Page<List<Orders>> queryOrderHistory(@RequestBody QueryVO queryVO);

}
```

## RESTFUL 行为限定

在实际接口设计中，我遇到了这样的需求，用户模块的接口需要支持修改用户密码，修改用户邮箱，修改用户姓名，而笔者之前阅读过一篇文章，也是讲舍弃 CRUD 而是用领域驱动设计来规范 RESTFUL 接口的定义，与项目中我的想法不谋而合。看似这三个属性是同一个实体类的三个属性，完全可以如下设计：

```java
@FeignClient("user")
public interface UserApi {

    @RequestMapping(value = "/user",method = RequestMethod.POST)
    User update(@RequestBody User user);

}
```

但实际上，如果再考虑多一层，就应该产生这样的思考：这三个功能所需要的权限一致吗？真的应该将他们放到一个接口中吗？实际上，笔者并不希望接口调用方传递一个实体，因为这样的行为是不可控的，完全不知道它到底是修改了什么属性，如果真的要限制行为，还需要在 User 中添加一个操作类型的字段，然后在接口实现方加以校验，这太麻烦了。而实际上，笔者觉得规范的设计应当如下：

```java
@FeignClient("user")
public interface UserApi {

    @RequestMapping(value = "/user/{userId}/password/update",method = RequestMethod.POST)
    ResultBean<Boolean> updatePassword(@PathVariable("userId) String userId,@RequestParam("password") password);
    
    @RequestMapping(value = "/user/{userId}/email/update",method = RequestMethod.POST)
    ResultBean<Boolean> updateEmail(@PathVariable("userId) String userId,@RequestParam("email") String email);
    
    @RequestMapping(value = "/user/{userId}/username/update",method = RequestMethod.POST)
    ResultBean<Boolean> updateUsername(@PathVariable("userId) String userId,@RequestParam("username") String username);

}
```

- 一般意义上 RESTFUL 接口不应该出现动词，这里的 update 并不是一个动作，而是标记着操作的类型，因为针对某个属性可能出现的操作类型可能会有很多，所以我习惯加上一个 update 后缀，明确表达想要进行的操作，而不是仅仅依赖于 GET，POST，PUT，DELETE。实际上，修改操作推荐使用的请求方式应当是 PUT，这点笔者的理解是，已经使用 update 标记了行为，实际开发中不习惯使用 PUT。
- password，email，username 都是 user 的属性，而 userId 是 user 的识别符号，所以 userId 以 PathVariable 的形式出现在 url 中，而三个属性出现在 ReqeustParam 中。

顺带谈谈逻辑删除，如果一个需求是删除用户的常用地址，这个 api 的操作类型，我通常也不会设计为 DELETE 请求，而是同样使用 delete 来标记操作行为

```
@RequestMapping(value = "/user/{userId}/address/{addressId}/delete",method = RequestMethod.POST)
    ResultBean<Boolean> updateEmail(@PathVariable("userId") String userId,@PathVariable("userId") String email);
```



---

参考

[徐靖峰/阿里巴巴中间件研发](https://www.cnkirito.moe/feign-1/)