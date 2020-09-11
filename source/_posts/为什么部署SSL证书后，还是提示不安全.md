---
title: 为什么部署SSL证书后还是提示不安全
categories: 网站
tags: [ssl,网站,服务器,https]
date: 2020-09-09
---



![img](https://mysticalyu.gitee.io/pic/img/20200407235705-www-ycfcg-com-3.jpg)

很多朋友可能都会遇到这个让人抓狂的问题：为什么按要求部署SSL证书了，但访问网站时还是会出现不安全的提示？这里就和大家一起来看看可能导致这个问题的原因和解决方法。



 <!-- more -->



# 为什么部署SSL证书后，还是提示不安全

很多朋友可能都会遇到这个让人抓狂的问题：为什么按要求部署SSL证书了，但访问网站时还是会出现不安全的提示？这里[必盛云](https://www.bisend.cn/)就和大家一起来看看可能导致这个问题的原因和解决方法。

首先我们来看看为什么会出现不安全提示：

当我们访问一个已安装[SSL证书](https://www.bisend.cn/ssl-certificate)的网站页面时，如果页面出现”您的连接不是私密链接“或”此网页含有不安全的内容“等类似的错误提示，这说明浏览器检查到你当前所访问的HTTPS网站的SSL证书出现问题了，存在访问风险，所以提示用户“此站点不安全”。

那么这个问题是怎么出现的呢？要怎样才能解决呢？请看下面的分析：

### 域名不匹配

HTTPS网站其中一个非常重要的作用，就是确认网站的身份。这样就能非常有效地预防DNS劫持。一张SSL证书必须对应一个网站域名，当你访问的网站域名和SSL证书中设置的域名不一致时，浏览器就会提示用户网站不安全。可能是由于配置错误导致SSL证书与网站域名不配置，也可能因为多个域名都能使用同一张SSL证书，导致域名与SSL证书不匹配。

此种情况下，需要站长确认域名与所部署的SSL证书是否相匹配，如不匹配需及时替换为正确的证书。

### 证书到期

这个原因非常好理解，即您的SSL证书到期需要续费重新申请了； SSL证书过期也是导致出现错误的原因之一。一般SSL证书的有效期为1~2年，当证书过期后，就必须要更新证书，HTTPS网站才能继续正常工作。否则过了有效期就会提示错误。可能有的站长管理很多网站，在证书到期之后重新安装会非常耗时，希望申请证书的有效期能长一点，但从CA（Certificate Authority证书颁发机构）的角度来看，设置证书有效期是非常有必要的。

首先是为了安全考虑，CA机构不能保证一个网站永远是合法的，因此它需要定期检查网站。其次，涉及到证书吊销。当网站的私钥丢失时，网站应该向CA申请将他们的证书加入到证书吊销列表（CRL）里。当用户访问https站点时，浏览器会自动向CA请求吊销列表，如果用户访问的站点提供的证书在CRL里，浏览器就不信任这个证书，因为攻击者可能拥有同样的证书。所以如果证书永久有效，随着越来越多的私钥丢失，吊销列表也越来越大，因为只有加进去的，没有剔出去的，这既给CA增加流量压力，也会增加浏览器的流量。而一旦有效期只有几年，那么CA就可以将那些已经过期了的证书从CRL里剔除，因为反正浏览器也不信任过期证书。

### 系统时间错误

另外一个常见的原因，是客户端的系统时间错误。浏览器会判断SSL证书是否过期，而浏览器的时间判断是依照你的系统时间。假如你的系统时间不正确，那么很有可能浏览器就会出现误判的情况，导致一张还没过期的SSL证书被认为是过期了。最终页面显示错误提示。解决办法也非常简单，修改您的系统时间为正确的就可以了。

### 不受信任的SSL证书

一些站长或者个人站长由于经费上的预算有限，又恰好懂得一些代码，自己制作出一张自签名的SSL证书或向一些SSL证书服务商申请不受信任的SSL证书。但现在浏览器对SSL证书的管理也更加严格，这些使用自签名SSL证书或不受信任的SSL证书部署的网站是不被浏览器信任的，所以访问时仍然会出现错误提示。因为自签名证书或小型SSL证书并不在操作系统的可信任根证书之中。只有是由受信任根证书所签发出来的SSL证书，浏览器才会认为是安全的，其他的SSL证书浏览器一律都会提示错误。

### 站内调用非HTTPS素材

站内调用非HTTPS素材包括图文、CSS、js等素材。出现这种情形时，一般在浏览器安全锁处会看到锁出现了一个三角形图形，点击查看SSL证书详细信息能看到有感叹号说明。下面我们就来看看如何解决这一问题：

首先您可以通过按住键盘上的F12键进入开发者模式，或者鼠标放在页面空白处点击右键后选择“查看网页源代码”，之后就可以在右侧边框找是哪些非安全链接导至整个网站不被信任。

![img](https://www.bisend.cn/blog/wp-content/uploads/2019/09/1.jpeg)

查看上图中找到的两个链接，的确是http的素材路径。

![img](https://www.bisend.cn/blog/wp-content/uploads/2019/09/2.jpeg)

那么现在我们需要做的就是检查这两个链接是否为有用链接；如果这个链接没有什么作用，删掉也不会对网站有任何影响，那么直接删除就行了。然后继续检查直至清除所有无用的非安全链接。完成清理后，再清除浏览器缓存，则可以看到网站已经重新被浏览器信任了。

![img](https://www.bisend.cn/blog/wp-content/uploads/2019/09/u20jw3wl6p.jpeg)

现在再来看看第二种情况，如果这个元素或图片/CSS/JS是对网站极为重要的内容，绝对不可以删除。此时，则只需把源站的非安全素材的路径由绝对路径放置在本站文件夹变成相对路径。

# 什么是混合内容？

**混合内容**在以下情况下出现：初始 HTML 内容通过安全的 HTTPS 连接加载，但其他资源（例如，图像、视频、样式表、脚本）则通过不安全的 HTTP 连接加载。之所以称为混合内容，是因为同时加载了 HTTP 和 HTTPS 内容以显示同一个页面，且通过 HTTPS 加载的初始请求是安全的。现代浏览器会针对此类型的内容显示警告，以向用户表明此页面包含不安全的资源。

### TL;DR

- HTTPS 对于保护您的网站和用户免受攻击非常重要。
- 混合内容会降低您的 HTTPS 网站的安全性和用户体验。

## 资源请求和网络浏览器

当浏览器访问网站的页面时，它将请求 HTML 资源。然后，网络服务器返回 HTML 内容，浏览器进行解析并显示给用户。通常，一个 HTML 文件不足以显示一个完整页面，因此，HTML 文件包含浏览器需要请求的其他资源的引用。这些子资源可以是图像、视频、额外 HTML、CSS 或 JavaScript 之类的资源；每个资源均使用单独的请求获取。

## HTTPS 的优势

当浏览器通过 HTTPS（HTTP Secure 的缩写形式）请求资源时，它使用一个已加密连接与网络服务器进行通信。

使用 HTTPS 有三个主要优势：

- 身份验证
- 数据完整性
- 保密性

### 身份验证

我正在访问的网站是正确的吗？

HTTPS 让浏览器检查并确保其已打开正确的网站，并且没有被重定向到恶意的网站。 当导航到您的银行网站时，您的浏览器对该网站进行身份验证，从而防止攻击者冒充您的银行窃取您的登录凭据。

### 数据完整性

是否有人篡改我正在发送或接收的内容？

HTTPS 让浏览器检测是否有攻击者更改了浏览器接收的任何数据。 使用您的银行网站转账时，这样做可防止当您的请求在传输中时攻击者更改目标帐号。

### 保密性

是否有人能看到我正在发送或接收的内容？

HTTPS 可防止攻击者窃取浏览器的请求，跟踪访问的网站或窃取已发送或接收的信息。

### HTTPS、传输层安全协议 (TLS) 和 SSL

HTTPS 是 HTTP Secure 的缩写，即超文本传输安全协议。此处的 **secure** 部分来自于添加到浏览器发送和接收的请求的加密。目前大多数浏览器都使用传输层安全协议 (TLS) 提供加密；**TLS** 有时称为 SSL。

- [Wikipedia HTTPS](https://en.wikipedia.org/wiki/HTTPS)
- [Wikipedia TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security)
- [可汗学院 (Khan Academy) 的加密课程](https://www.khanacademy.org/computing/computer-science/cryptography)
- [高性能浏览器网络](https://hpbn.co/)（作者：Ilya Grigorik）中的[传输层安全协议 (TLS) 章节](https://hpbn.co/transport-layer-security-tls/)

## 混合内容会降低 HTTPS 的安全性

使用不安全的 HTTP 协议请求子资源会降低整个页面的安全性，因为这些请求容易受到**中间人攻击**，攻击者窃听网络连接，查看或修改双方的通信。通过使用这些资源，攻击者通常可以完全控制页面，而不只是泄露的资源。

尽管许多浏览器向用户报告混合内容警告，但出现警告时为时已晚：不安全的请求已被执行，且页面的安全性被破坏。遗憾的是，这种情况在网络中很普遍，正因如此，浏览器不能简单地阻止所有混合请求，否则将会限制许多网站的功能。

![混合内容：页面已通过 HTTPS 加载，但请求了不安全的图像。此内容也应通过 HTTPS 提供。](https://developers.google.com/web/fundamentals/security/prevent-mixed-content/imgs/image-gallery-warning.png?hl=zh-cn)修正应用中的混合内容问题是开发者的责任。

### 一个简单的示例

从 HTTPS 页面加载不安全的脚本。

查看通过 **HTTPS**—[**https**://googlesamples.github.io/web-fundamentals/.../simple-example.html](https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/simple-example.html)加载的此示例页面 — 添加一个 **HTTP** 脚本标记，其尝试加载混合内容。

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons">
    <link rel="stylesheet" href="https://code.getmdl.io/1.2.1/material.indigo-pink.min.css">
    <script defer src="https://code.getmdl.io/1.2.1/material.min.js"></script>
    <style>
      body {
        margin: 2em;
      }
    </style>
    <title>Simple mixed content example</title>
  </head>
  <body>
    <div role="main">
      <h1>
        Simple mixed content example!
      </h1>
      <p>
        View page over: <a href="http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/simple-example.html">HTTP</a> - <a href="https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/simple-example.html">HTTPS</a>
      </p>
      <p>
        This page loads the script simple-example.js using HTTP. This is the simplest case of mixed content. When the simple-example.js file is requested by the browser, an attacker can inject code into the returned content and take control of the entire page. Thankfully, most modern browsers block this type of dangerous content by default and display an error in the JavaScript console. This can be seen when the page is viewed over HTTPS.
      </p>
      <div id="output">Waiting for insecure script to run...</div>
      <script src="http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/simple-example.js"></script>
    </div>
    <script>
  (function(b,o,i,l,e,r){b.GoogleAnalyticsObject=l;b[l]||(b[l]=
  function(){(b[l].q=b[l].q||[]).push(arguments)});b[l].l=+new Date;
  e=o.createElement(i);r=o.getElementsByTagName(i)[0];
  e.src='https://www.google-analytics.com/analytics.js';
  r.parentNode.insertBefore(e,r)}(window,document,'script','ga'));
  ga('create','UA-52746336-1');ga('send','pageview');
  var isCompleted = {};
  function sampleCompleted(sampleName){
    if (ga && !isCompleted.hasOwnProperty(sampleName)) {
      ga('send', 'event', 'WebCentralSample', sampleName, 'completed'); 
      isCompleted[sampleName] = true;
    }
  }
</script>
  </body>
</html>
```

[试一下](https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/simple-example.html)

在此示例中，使用一个 **HTTP** 网址加载脚本 `simple-example.js`。这是最简单的混合内容案例。浏览器请求 `simple-example.js` 文件时，攻击者可以将代码注入返回的内容，并控制整个页面。

幸运的是，大多数现代浏览器均默认阻止此类危险的内容。 请参阅[具有混合内容的浏览器行为](https://developers.google.com/web/fundamentals/security/prevent-mixed-content/what-is-mixed-content?hl=zh-cn#browser-behavior-with-mixed-content)。

![混合内容：页面已通过 HTTPS 加载，但请求了不安全的脚本。此请求已被阻止，内容必须通过 HTTPS 提供。](https://developers.google.com/web/fundamentals/security/prevent-mixed-content/imgs/simple-mixed-content-error.png?hl=zh-cn)Chrome 可阻止不安全的脚本。

### 一个 XMLHttpRequest 示例

通过 XMLHttpRequest 加载不安全的数据。

查看通过 **HTTPS**—[**https**://googlesamples.github.io/web-fundamentals/.../xmlhttprequest-example.html](https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/xmlhttprequest-example.html) 加载的此示例页面 — 添加一个通过 **HTTP** 加载的`XMLHttpRequest`，以获取混合内容 `JSON` 数据。

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons">
    <link rel="stylesheet" href="https://code.getmdl.io/1.2.1/material.indigo-pink.min.css">
    <script defer src="https://code.getmdl.io/1.2.1/material.min.js"></script>
    <style>
      body {
        margin: 2em;
      }
    </style>
    <title>XMLHttpRequest mixed content example</title>
  </head>
  <body>
    <div role="main">
      <h1>
        XMLHttpRequest mixed content example!
      </h1>
      <p>
        View page over: <a href="http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/xmlhttprequest-example.html">HTTP</a> - <a href="https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/xmlhttprequest-example.html">HTTPS</a>
      </p>
      <p>
        This page constructs an HTTP URL dynamically in JavaScript, the URL is eventually used to load an insecure resource by XMLHttpRequest. When the xmlhttprequest-data.js file is requested by the browser, an attacker can inject code into the returned content and take control of the entire page. Thankfully, most modern browsers block this type of dangerous content by default and display an error in the JavaScript console. This can be seen when the page is viewed over HTTPS.
      </p>
      <div id="output">Waiting for data...</div>
      <script>
        var rootUrl = 'http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content';
        var resources = {
          jsonData: '/xmlhttprequest-data.js'
        };
        var request = new XMLHttpRequest();
        request.addEventListener('load', function() {
          var jsonData = JSON.parse(request.responseText);
          document.getElementById('output').innerHTML += '<br>' + jsonData.data;
        });
        request.open('GET', rootUrl + resources.jsonData, true);
        request.send();
      </script>
    </div>
    <script>
  (function(b,o,i,l,e,r){b.GoogleAnalyticsObject=l;b[l]||(b[l]=
  function(){(b[l].q=b[l].q||[]).push(arguments)});b[l].l=+new Date;
  e=o.createElement(i);r=o.getElementsByTagName(i)[0];
  e.src='//www.google-analytics.com/analytics.js';
  r.parentNode.insertBefore(e,r)}(window,document,'script','ga'));
  ga('create','UA-52746336-1');ga('send','pageview');
  var isCompleted = {};
  function sampleCompleted(sampleName){
    if (ga && !isCompleted.hasOwnProperty(sampleName)) {
      ga('send', 'event', 'WebCentralSample', sampleName, 'completed'); 
      isCompleted[sampleName] = true;
    }
  }
</script>
  </body>
</html>
```

[试一下](https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/xmlhttprequest-example.html)

下面的 **HTTP** 网址是在 JavaScript 中动态构建的，并且最终被 `XMLHttpRequest` 用于加载不安全的资源。 与上面简单的示例相似，当浏览器请求 `xmlhttprequest-data.js` 文件时，攻击者可以将代码注入返回的内容中，并控制整个页面。

大多数现代浏览器也会阻止这些危险的请求。

![混合内容：页面已通过 HTTPS 加载，但请求了不安全的 XMLHttpRequest 端点。此请求已被阻止，内容必须通过 HTTPS 提供。](https://developers.google.com/web/fundamentals/security/prevent-mixed-content/imgs/xmlhttprequest-mixed-content-error.png?hl=zh-cn)Chrome 可阻止不安全的 XMLHttpRequest。

### 图像库示例

使用 jQuery 灯箱加载不安全的图像。

查看通过 **HTTPS**—[**https**://googlesamples.github.io/web-fundamentals/.../image-gallery-example.html](https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/image-gallery-example.html) 加载的此示例页面时 — 最初没有任何混合内容问题；但是当点击缩略图时，将通过 **HTTP** 加载完整尺寸的混合内容图像。

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons">
    <link rel="stylesheet" href="https://code.getmdl.io/1.2.1/material.indigo-pink.min.css">
    <script defer src="https://code.getmdl.io/1.2.1/material.min.js"></script>
    <style>
      body {
        margin: 2em;
      }
    </style>
    <title>Image gallery mixed content example</title>
    <script src="https://www.gstatic.com/external_hosted/jquery2.min.js"></script>
    <script>
      $(function() {
        $('.gallery').click(function(e) {
          e.preventDefault();
          $('.overlay-foreground').css('background-image', 'url(' + $(this).attr('href') + ')');
          $('.overlay').fadeIn('slow');
        })
        $('.overlay').click(function() {
          $('.overlay').fadeOut('slow');
        })
      });
    </script>
    <style>
      .overlay {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
      }
      .overlay-background {
        background-color: #000;
        filter:alpha(opacity=80);
        -moz-opacity: 0.8;
        -khtml-opacity: 0.8;
        opacity: 0.8;
        z-index: 10000;
      }
      .overlay-foreground {
        background-position: center center;
        background-repeat: no-repeat;
        z-index: 10001;
      }
    </style>
  </head>
  <body>
    <div role="main">
      <h1>
        Image gallery mixed content!
      </h1>
      <p>
        View page over: <a href="http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/image-gallery-example.html">HTTP</a> - <a href="https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/image-gallery-example.html">HTTPS</a>
      </p>
      <p>
        Image galleries often rely on the &lt;img&gt; tag src attribute to display thumbnail images on the page, the anchor ( &lt;a&gt; ) tag href attribute is then used to load the full sized image for the gallery overlay. Normally &lt;a&gt; tags do not cause mixed content, but in this case the jQuery code overrides the default link behavior &mdash; to navigate to a new page &mdash; and instead loads the HTTP image on this page. While this content isn't blocked, modern browsers display a warning in the JavaScript console. This can be seen when the page is viewed over HTTPS and the thumbnail is clicked.
      </p>
      CLICK ME! -->
      <a class="gallery" href="http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/puppy.jpg">
        <img src="https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/puppy-thumb.jpg">
      </a>
      <div class="overlay overlay-background" style="display: none;"></div>
      <div class="overlay overlay-foreground" style="display: none;"></div>
    </div>
    <script>
  (function(b,o,i,l,e,r){b.GoogleAnalyticsObject=l;b[l]||(b[l]=
  function(){(b[l].q=b[l].q||[]).push(arguments)});b[l].l=+new Date;
  e=o.createElement(i);r=o.getElementsByTagName(i)[0];
  e.src='//www.google-analytics.com/analytics.js';
  r.parentNode.insertBefore(e,r)}(window,document,'script','ga'));
  ga('create','UA-52746336-1');ga('send','pageview');
  var isCompleted = {};
  function sampleCompleted(sampleName){
    if (ga && !isCompleted.hasOwnProperty(sampleName)) {
      ga('send', 'event', 'WebCentralSample', sampleName, 'completed');
      isCompleted[sampleName] = true;
    }
  }
</script>
  </body>
</html>
```

[试一下](https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/image-gallery-example.html)

图像库通常依靠 `<img>` 标记 `src` 属性在页面上显示缩略图，然后，使用定位 (`<a>`) 标记 `href` 属性为图像库叠加层加载完整尺寸的图像。正常情况下，`<a>` 标记不会产生混合内容，但在此例中，jQuery 代码替换默认链接行为（导航到新页面），改为在此页面上加载 **HTTP** 图像。

![混合内容：页面已通过 HTTPS 加载，但请求了不安全的图像。此内容也应通过 HTTPS 提供。](https://developers.google.com/web/fundamentals/security/prevent-mixed-content/imgs/image-gallery-warning.png?hl=zh-cn)

不安全的图像会降低网站的安全性，但是它们的危险性与其他类型的混合内容不一样。 现代浏览器仍会加载混合内容图像，但也会向用户显示警告。

## 混合内容类型与相关安全威胁

混合内容有两种：主动混合内容和被动混合内容

**被动混合内容**指的是不与页面其余部分进行交互的内容，从而使中间人攻击在拦截或更改该内容时能够执行的操作受限。被动混合内容包括图像、视频和音频内容，以及无法与页面其余部分进行交互的其他资源。

**主动混合内容**作为整体与页面进行交互，并且几乎允许攻击者对页面进行任何操作。 主动混合内容包括浏览器可下载和执行的脚本、样式表、iframe、flash 资源及其他代码。

### 被动混合内容

被动混合内容仍会给您的网站和用户带来安全威胁。 例如，攻击者可以拦截针对网站上的图像的 HTTP 请求，调换或更换这些图像；此攻击者可以调换“save and delete”按钮图像，导致您的用户无意间删除内容；将您的产品图表更换为下流或淫秽内容，从而损害您的网站；或将您的产品图像更换为不同网站或产品的广告。

即使攻击者不改变您的网站内容，您仍面临严重的隐私问题，攻击者可以使用混合内容请求跟踪用户。攻击者可以基于浏览器加载的图像或其他资源了解用户访问哪些页面，以及查看了哪些产品。

以下是被动混合内容的示例：

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons">
    <link rel="stylesheet" href="https://code.getmdl.io/1.2.1/material.indigo-pink.min.css">
    <script defer src="https://code.getmdl.io/1.2.1/material.min.js"></script>
    <style>
      body {
        margin: 2em;
      }
    </style>
    <title>Passive mixed content example</title>
    <style>
      audio, img, video {
        display: block;
        margin: 10px;
      }
    </style>
  </head>
  <body>
    <div role="main">
      <h1>
        Passive mixed content!
      </h1>
      <p>
        View page over: <a href="http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/passive-mixed-content.html">HTTP</a> - <a href="https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/passive-mixed-content.html">HTTPS</a>
      </p>
      <p>
        Several examples of passive mixed content. When viewed over HTTPS most browsers do <b>not</b> block this content but instead display warnings in the JavaScript console.
      </p>

      <!-- An insecure audio file loaded on a secure page -->
      <audio src="http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/sleep.mp3" type="audio/mp3" controls></audio>

      <!-- An insecure image loaded on a secure page -->
      <img src="http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/puppy.jpg">

      <!-- An insecure video file loaded on a secure page -->
      <video src="http://storage.googleapis.com/webfundamentals-assets/videos/chrome.webm" type="video/webm" controls></video>
    </div>
    <script>
  (function(b,o,i,l,e,r){b.GoogleAnalyticsObject=l;b[l]||(b[l]=
  function(){(b[l].q=b[l].q||[]).push(arguments)});b[l].l=+new Date;
  e=o.createElement(i);r=o.getElementsByTagName(i)[0];
  e.src='//www.google-analytics.com/analytics.js';
  r.parentNode.insertBefore(e,r)}(window,document,'script','ga'));
  ga('create','UA-52746336-1');ga('send','pageview');
  var isCompleted = {};
  function sampleCompleted(sampleName){
    if (ga && !isCompleted.hasOwnProperty(sampleName)) {
      ga('send', 'event', 'WebCentralSample', sampleName, 'completed');
      isCompleted[sampleName] = true;
    }
  }
</script>
  </body>
</html>
```

[试一下](https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/passive-mixed-content.html)

大多数浏览器仍向用户渲染此类型的混合内容，但是也会显示警告，因为这些内容会给您的网站和用户带来安全风险和隐私风险。

![混合内容：页面已通过 HTTPS 加载，但请求了不安全的视频。此内容也应通过 HTTPS 提供。](https://developers.google.com/web/fundamentals/security/prevent-mixed-content/imgs/passive-mixed-content-warnings.png?hl=zh-cn)来自 Chrome JavaScript 控制台的混合内容警告。

### 主动混合内容

与被动混合内容相比，主动混合内容造成的威胁更大。攻击者可以拦截和重写主动内容，从而完全控制页面，甚至整个网站。这让攻击者可以更改有关页面的任何内容，包括显示完全不同的内容、窃取用户密码或其他登录凭据、窃取用户会话 Cookie，或将用户重定向到一个完全不同的网站。

鉴于这种威胁的严重性，许多浏览器都会默认阻止此类型的内容以保护用户，但是其作用因浏览器供应商和版本而有所差异。

以下包含主动混合内容的示例：

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons">
    <link rel="stylesheet" href="https://code.getmdl.io/1.2.1/material.indigo-pink.min.css">
    <script defer src="https://code.getmdl.io/1.2.1/material.min.js"></script>
    <style>
      body {
        margin: 2em;
      }
    </style>
    <title>Active mixed content example</title>
    <!-- An insecure script file loaded on a secure page -->
    <script src="http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/simple-example.js"></script>

    <!-- An insecure stylesheet loaded on a secure page -->
    <link href="http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/style.css" rel="stylesheet">

    <style>
      .insecure-background {
        /* An insecure resources loaded from a style property on a secure page, this can
           happen in many places including, @font-face, cursor, background-image, and so on. */
        background: url('http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/puppy-thumb.jpg') no-repeat;
      }
    </style>

    <style>
      .insecure-style-holder span {
        color: #fff;
      }
      .insecure-background {
        color: #000;
        font-weight: bold;
        background-position: left center;
        background-repeat: no-repeat;
        width: 300px;
        height: 140px;
      }
      iframe {
        width: 400px;
        height: 300px;
      }
    </style>

  </head>
  <body>
    <div role="main">
      <h1>
        Active mixed content!
      </h1>
      <p>
        View page over: <a href="http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/active-mixed-content.html">HTTP</a> - <a href="https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/active-mixed-content.html">HTTPS</a>
      </p>
      <p>
        Several examples of active mixed content. When viewed over HTTPS most browsers block this content and display errors in the JavaScript console.
      </p>
      <div class="insecure-style-holder">
        <span style="ba">Insecure style loaded</span>
      </div>
      <div class="insecure-background">
        Loading insecure background here...
      </div>

      <p>Loading insecure iframe...</p>
      <!-- An insecure iframed page loaded on a secure page -->
      <iframe src="http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/image-gallery-example.html"></iframe>

      <!-- Flash resources also qualify as active mixed content and pose a
      serious security risk. Be sure to look for <object> tags with type set
      to "application/x-shockwave-flash", and an http:// data attribute. -->
      <!-- <object type="application/x-shockwave-flash" data="http://..."></object> -->

      <script>
        // An insecure resource loaded using XMLHttpRequest
        var request = new XMLHttpRequest();
        request.addEventListener('load', function() {
          var jsonData = JSON.parse(request.responseText);
          document.getElementById('output').innerHTML += '<br>' + jsonData.data;
        });
        request.open("GET", "http://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/xmlhttprequest-data.js", true);
        request.send();
      </script>
      <div id="output">Waiting for insecure script to run...</div>
    </div>
    <script>
  (function(b,o,i,l,e,r){b.GoogleAnalyticsObject=l;b[l]||(b[l]=
  function(){(b[l].q=b[l].q||[]).push(arguments)});b[l].l=+new Date;
  e=o.createElement(i);r=o.getElementsByTagName(i)[0];
  e.src='//www.google-analytics.com/analytics.js';
  r.parentNode.insertBefore(e,r)}(window,document,'script','ga'));
  ga('create','UA-52746336-1');ga('send','pageview');
  var isCompleted = {};
  function sampleCompleted(sampleName){
    if (ga && !isCompleted.hasOwnProperty(sampleName)) {
      ga('send', 'event', 'WebCentralSample', sampleName, 'completed');
      isCompleted[sampleName] = true;
    }
  }
</script>
  </body>
</html>
```

[试一下](https://googlesamples.github.io/web-fundamentals/fundamentals/security/prevent-mixed-content/active-mixed-content.html)

![混合内容：页面已通过 HTTPS 加载，但请求了不安全的资源。此请求已被阻止，内容必须通过 HTTPS 提供。](https://developers.google.com/web/fundamentals/security/prevent-mixed-content/imgs/active-mixed-content-errors.png?hl=zh-cn)来自 Chrome JavaScript 控制台的混合内容错误。

## 具有混合内容的浏览器行为

鉴于上述威胁，浏览器最好是阻止所有混合内容。 但是，这将破坏大量网站，而数百万用户每天都要访问这些网站。 当前的折衷做法是阻止最危险的混合内容类型，同时仍允许请求不太危险的混合内容类型。

现代浏览器遵循[混合内容规范](https://w3c.github.io/webappsec/specs/mixedcontent/)，其定义了[**可选择性地阻止的内容**](https://w3c.github.io/webappsec/specs/mixedcontent/#category-optionally-blockable)和[**可阻止的内容**](https://w3c.github.io/webappsec/specs/mixedcontent/#category-blockable)类别。

根据此规范，“当破坏网络重要部分的风险超过允许此资源作为混合内容使用的风险时”，该资源有资格成为可选择性阻止的内容；这是上述[被动混合内容](https://developers.google.com/web/fundamentals/security/prevent-mixed-content/what-is-mixed-content?hl=zh-cn#passive-mixed-content)类别的子集。在撰写本文时，可选择性阻止的内容中仅包括图像、视频和音频资源以及预获取的链接这些资源类型。随着时间的推移，此类别可能会缩小。

**可选择性阻止的内容**以外的所有内容被视为**可阻止的内容**，将被浏览器阻止。

### 浏览器版本

切记，并不是网站的每个访问者都使用最新的浏览器。 不同浏览器供应商的不同版本的浏览器处理混合内容的方式不尽相同。 最糟糕的情况是，有些浏览器和版本根本不会阻止任何混合内容，这对于用户而言非常不安全。

每个浏览器的确切行为不断变化，因此，我们在这里不做具体介绍。 如果您对特定浏览器的行为方式感兴趣，请直接查看供应商发布的信息。

### 缺少中间根证书

还有一种情况是由于证书链不完整缺少中间根证书导致的。这个情况下[找到SSL证书的中间根证书](https://www.bisend.cn/blog/ruhezhaodaosslzhengshudezhongjiangenzhengshu)、补全证书链再重新安装即可解决问题。

以上即为几种常见的导致SSL证书部署后仍出现不安全提示的原因。如果一一排查之后还是不能解决问题，或者处理过程中不知该如何操作，可提交工单或在线联系必盛云客服为您处理。

### 相关文章：

- [Comodo证书合并及转换教程](https://www.bisend.cn/blog/comodozhengshuhebingjizhuanhuanjiaocheng)
- [Digicert、Symantec和Geotrust证书合并及转换教程](https://www.bisend.cn/blog/digicertzhengshuhebingjizhuanhuanjiaocheng)
- [申请SSL证书怎样做域名验证](https://www.bisend.cn/blog/shenqingsslzhengshuzenyangzuoyumingyanzheng)
- [如何申请OV/EV SSL 证书](https://www.bisend.cn/blog/ru-he-shen-qing-ov-ev-ssl-zheng-shu)
- [如何部署SSL证书](https://www.bisend.cn/blog/ruhebushusslzhengshu)
- [Apache服务器安装SSL证书](https://www.bisend.cn/blog/apachefuwuqianzhuangsslzhengshu)
- [如何备份或导出SSL证书](https://www.bisend.cn/blog/ruhebeifenhuodaochusslzhengshu)
- [IP地址可以安装SSL证书吗？](https://www.bisend.cn/blog/ip-ssl-certificates)
- [混合内容](https://developers.google.com/web/fundamentals/security/prevent-mixed-content/what-is-mixed-content?hl=zh-cn)