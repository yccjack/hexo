---
title: ngrok内网穿透
categories: 服务器
tags: [ngrok,java,内网穿透] 
date: 2020-08-28
cover: https://gschaos.club/ico/img/bora-kim-1-thumbnail2.jpg

---

ngrok是一个反向代理，它能够让你本地的web服务或tcp服务通过公共的端口和外部建立一个安全的通道，使得外网可以访问本地的计算机服务。ngrok1.x开源，ngrok2.x不开源



<!-- more -->



- - ## 1. ngrok简介

  [ngrok](https://ngrok.com/)是一个反向代理，它能够让你本地的web服务或tcp服务通过公共的端口和外部建立一个安全的通道，使得外网可以访问本地的计算机服务。ngrok1.x开源，ngrok2.x不开源

  ![ngrok](https://gitee.com/MysticalYu/pic/raw/master/hexo/ngrok.png)

  ngrok的主要用途有以下几种：

  - 内网穿透，可代替vpn
  - 将无外网IP的desktop映射到公网
  - 临时搭建网络并分配二级域名
  - 微信二次开发的本地调试

  ## 2. 准备工作

  自己搭建ngrok服务需要一台外网服务器，一个域名。本文中使用的服务器系统为Ubuntu 16.04。

  ### 2.1 域名

  有域名之后，需要配置DNS的Host Records，将准备分配给ngrok服务器的域名解析到公网服务器IP地址，如下图所示：

  ![image-20200824143715666](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200824143715666.png)

  **使用泛域名，使其能解析子域名**

  ### 2.2 配置环境

  ngrok是基于go语言开发的，因此需要先安装go：

  ```
  sudo apt-get install golang
  ```

  输入`go version`来验证安装：

  ```
  go versiongo version go1.6.2 linux/amd64
  ```

  设置go环境变量(好像可以不用？)：

  此外还要使用git，一般ubuntu系统都会自带。

  ## 3. 搭建ngrok服务器

  ## 3.1 clone ngrok

  ```
  git clone https://github.com/inconshreveable/ngrok.gitcd ngrok
  ```

  ## 3.2 生成证书

  使用[ngrok.com](http://ngrok.com/)官方服务时，我们使用的是官方的SSL证书。自己建立ngrok服务，需要我们生成自己的证书，并提供携带该证书的ngrok客户端。首先指定域名：

  ```
  export NGROK_DOMAIN="ngrok.test.website"
  ```

  生成证书：

  ```
  openssl genrsa -out rootCA.key 2048openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pemopenssl genrsa -out device.key 2048openssl req -new -key device.key -subj "/CN=$NGROK_DOMAIN" -out device.csropenssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000
  ```

  我们在编译可执行文件之前，需要把生成的证书分别替换到 assets/client/tls和assets/server/tls中，这两个目录分别存放着ngrok和ngrokd的默认证书。

  ```
  cp rootCA.pem assets/client/tls/ngrokroot.crtcp device.crt assets/server/tls/snakeoil.crtcp device.key assets/server/tls/snakeoil.key
  ```

  ## 3.3 编译ngrok

  有没有release的区别是，包含release的编译结果会把assets目录下的内容包括进去，从而可以独立执行。如果你今后还要更换证书，建议编译不包含release的版本。。首先编译ngrok服务端（ngrokd），默认为Linux版本：

  ```bash
  make clean
  make release-server
  ```

  如果make clean报错，忽略make clean直接执行make release-server。

  编译过程需要等待一会，因为需要通过git安装相关依赖包。如果提示没有权限，使用`sudo`命令来安装。

  在编译客户端的时候需要指明对应的操作系统和构架：

  - Linux 平台 32 位系统：GOOS=linux GOARCH=386
  - Linux 平台 64 位系统：GOOS=linux GOARCH=amd64
  - Windows 平台 32 位系统：GOOS=windows GOARCH=386
  - Windows 平台 64 位系统：GOOS=windows GOARCH=amd64
  - MAC 平台 32 位系统：GOOS=darwin GOARCH=386
  - MAC 平台 64 位系统：GOOS=darwin GOARCH=amd64
  - ARM 平台：GOOS=linux GOARCH=arm

  例如编译Linux64位的客户端：

  ```
  GOOS=linux GOARCH=amd64 make release-client
  ```

  生成的文件放在`/bin`对应的文件夹中，如windows 64位的为：windows_amd64，默认版本的文件就在根目录下。

  ## 3.4 启动ngrokd服务器

  编译后生成两个文件分别为服务端（ngrokd）和客户端(ngrok)。切换到对应的文件夹，运行服务端：

  ```
  ./ngrokd -domain="$NGROK_DOMAIN" -httpAddr=":801" -httpsAddr=":802"
  ```

  参数`-domain`表示服务器域名，请改成你自己的域名；`-httpAddr`表示默认监听的HTTP端口，`-httpsAddr`表示默认监听的HTTPS端口，因为我用不到所以都设置成空字符串”“来关闭监听，如果需要打开的话记得格式是`:12345`（冒号+端口号）这样的；`-tunnelAddr`表示服务器监听客户端连接的隧道端口号，格式和前面一样；`-log`表示日志文件位置；还有个`-log-level`用来控制日志记录的事件级别，选项有DEBUG、INFO、WARNING、ERROR。

  如果编译的是不带release的版本，还可以通过`-tlsCrt`和`-tlsKey`选项来指定证书文件的位置。

  出现类似以下内容，则说明我们的服务器端ngrokd正常运行了:

  ```bash
  [16:41:56 CST 2017/04/20] [INFO] (ngrok/log.(*PrefixLogger).Info:83) [registry] [tun] No affinity cache specified
  [16:41:56 CST 2017/04/20] [INFO] (ngrok/log.(*PrefixLogger).Info:83) [metrics] Reporting every 30 seconds
  [16:41:57 CST 2017/04/20] [INFO] (ngrok/log.Info:112) Listening for public http connections on [::]:80
  [16:41:57 CST 2017/04/20] [INFO] (ngrok/log.Info:112) Listening for public https connections on [::]:443
  [16:41:57 CST 2017/04/20] [INFO] (ngrok/log.Info:112) Listening for control and proxy connections on [::]:4443
  [16:41:57 CST 2017/04/20] [INFO] (ngrok/log.(*PrefixLogger).Info:83) [tun:627acc92] New connection from 42.53.196.242:9386
  [16:41:57 CST 2017/04/20] [DEBG] (ngrok/log.(*PrefixLogger).Debug:79) [tun:627acc92] Waiting to read message
  [16:41:57 CST 2017/04/20] [DEBG] (ngrok/log.(*PrefixLogger).Debug:79) [tun:627acc92] Reading message with length: 159
  ```

  如果需要后台运行可以使用`screen`或`nohup`，详情自行搜索。

  ## 4. 配置ngrok客户端

  将之前编译好的客户端文件拷贝到需要使用服务的设备上。

  ### 4.1 建立配置文件

  在ngrok同路径下建立配置文件`ngrok.yml`：

  ```yaml
  server_addr: “ngrok.test.website:4443"
  trust_host_root_certs: false
  tunnels:
    ssh:
      remote_port: 6666
      proto:
        tcp: 22
  ```

  server_addr端口默认4443，可通过ngrokd服务端启动修改端口。在`tunnels`里配置隧道信息，具体可见[「翻译」ngrok 1.X 配置文档](https://imlonghao.com/28.html)。注意`http`和`https`隧道可设置`subdomain`和`auth`，而`tcp`里只能设置`remote_port`。

  还可以**转发其他IP的端口**，方法就是在proto下的tcp（或http、https）后的端口号写成IP地址:端口号的格式（中间是英文冒号）。如：`tcp: 192.168.11.1:80`

  ### 4.2 运行客户端

  现在运行客户端：

  ```
  ./ngrok -config=ngrok.yml start ssh
  ```

  回车后，看到这样一个界面，说明启动成功：

  ![Online](https://gitee.com/MysticalYu/pic/raw/master/hexo/online.png)

  如果显示`reconnecting`说明连接有错，在运行时加入`-log=stdout`来进行debug。可能有以下几方面原因：

  1. 可能是服务器端口未开放，在服务器上使用`sudo iptables --list`查看当前规则

  2. 查看是否网络问题，`ping`到对应的地址检查

  3. 可能是编译的时候证书没有覆盖或者版本不对，重新编译试试

  4. 这里有几个注意的点：

     - 在使用ECS时，注意是否在安全组中配置了相关端口。例如上边服务端和客户端使用的4443。
     - 这里有些一键安装脚本，大家可参考，有些配置和版本已经过时：
       - https://gist.github.com/popucui/18c342baefefed2ba66f87a9420efae5
       - https://github.com/sunnyos/ngrok/blob/master/ngrok.sh

     ### Nginx 代理共享80出口

     微信公众号开发时，要求后端服务没有端口。那么我们ngrok服务的http端口就需要设置为80。问题来了，我们服务器上还可能跑着其他应用，比如我的ECS上还跑了我的博客实例。这怎么办呢？解决方案是使用nginx的反向代理。

     nginx 的安装配置，大家可自行百度，这里不做过多描述。

     大家只需在nginx的配置中增加一段server配置，如下：

     ```xml
     server {
             server_name *.ngrok.pylixm.top 
             listen 80;
             keepalive_timeout 70;
             proxy_set_header "Host" $host:8081;  # 必须, 8081 为ngrok http转发端口
             location / {
                     proxy_pass_header Server;
                     proxy_redirect off;
                     proxy_pass http://127.0.0.1:8081;  # 必须, 8081 为ngrok http转发端口
             }
             access_log off;
             log_not_found off;
     }
     ```

     

     这样我们可以直接使用`*.ngrok.pylixm.top` 这个子域名访问ngrok代理的我们本地的服务了，同时还又不影响其他的80端口服务。

  ## 5. 详细资料

  以下是我搭建服务器时参考的一些资料：

  - 基础知识：[一分钟实现内网穿透（ngrok服务器搭建）](http://blog.csdn.net/zhangguo5/article/details/77848658)
  - 配置文档：[「翻译」ngrok 1.X 配置文档](https://imlonghao.com/28.html)
  - 源码分析：[ngrok原理浅析](https://tonybai.com/2015/05/14/ngrok-source-intro/)
  - 后续定制优化：[CentOS7配置ngrok实现内网穿透](http://blog.leanote.com/post/jesse/045ba03e0da6)
  - [给ngrok添加身份验证](https://prikevs.github.io/2016/12/26/add-authentication-to-ngrok/)
  - [关于 ngrok 使用上的注意事项](https://toontong.github.io/blog/about-ngrok.html)
  - [从零教你搭建ngrok服务，解决外网调试本地站点](https://morongs.github.io/2016/12/28/dajian-ngrok/)
  - [搭建并配置优雅的 ngrok 服务实现内网穿透](https://yii.im/posts/pretty-self-hosted-ngrokd/)