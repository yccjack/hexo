



title: PicGo + Gitee(码云)实现markdown图床
categories: 图床
tags: [图床] 
date: 2020-09-07
cover: https://mysticalyu.gitee.io/pic/img/euginnx-_wu-09.jpg

---

![]( https://mysticalyu.gitee.io/pic/img/euginnx-_wu-09.jpg)



PicGo + Gitee(码云)实现markdown图床



<!-- more -->



# PicGo + Gitee(码云)实现markdown图床

## PicGo + Gitee(码云)实现markdown图床

## 1. 安装

- PicGo
- picgo-plugin-gitee-uploader插件

### 首先打开[picgo官网](https://link.zhihu.com/?target=https%3A//github.com/Molunerfinn/PicGo)，下载安装包



![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/v2-6a5d78ebb1910843ff4d2872580d21a5_720w.png)

如果速度慢，点击此地址下载：

mac:  https://gschaos.club/down/PicGo-2.3.0-beta.3-mac.zip

win:  https://gschaos.club/down/PicGo-Setup-2.3.0-beta.3.exe



### 安装之后打开主界面

![image-20200911093623043](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911093623043.png)



### 选择最底下的插件设置，搜索**gitee**



![image-20200911093648928](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911093648928.png)



### 点击右边的gitee-uploader 1.1.2开始安装

> 这里注意一下，必须要先安装[node.js](https://link.zhihu.com/?target=https%3A//nodejs.org/en/)才能安装插件，没装的自己装一下，然后重启就行。

------

## 2. 建立图床库

### 点击右上角的+号，新建仓库



![image-20200911093729491](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911093729491.png)



新建仓库的要点如下：

1.  输入一个仓库名称
2.  其次将仓库设为公开
3. 勾选使用Readme文件初始化这个仓库



![image-20200911135935950](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911135935950.png)

点击下一步完成创建

------

## 3. 配置PicGo

安装了**gitee-uploader 1.1.2**插件之后，我们开始配置插件

### 配置插件的要点如下：

![image-20200911093830243](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911093830243.png)

- repo：用户名/仓库名称，比如我自己的仓库MysticalYu/pic，找不到的可以直接复制仓库的url,<font color=red>复制浏览器的仓库地址，而不是页面左上角显示的，容易出现大小写问题</font>

![image-20200911093907929](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911093907929.png)

- branch：分支，这里写上master
- token：填入码云的私人令牌
- path：路径，一般写上img
- customPath：提交消息，这一项和下一项customURL都不用填。在提交到码云后，会显示提交消息，插件默认提交的是 `Upload 图片名 by picGo - 时间`

### 这个token怎么获取，下面登录进自己的码云

1. 点击头像，进入设置



![img](https://gitee.com/MysticalYu/pic/raw/master/hexo/v2-09207edcefff7852c91abcc3df3c5ba0_720w.png)

2. 找到右边安全设置里面的私人令牌



![image-20200911093943588](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911093943588.png)

3. 点击`生成新令牌`，把**projects**这一项勾上，其他的不用勾，然后提交

![image-20200911094009969](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911094009969.png)



这里需要验证一下密码，验证密码之后会出来一串数字，这一串数字就是你的token，将这串数字复制到刚才的配置里面去。

![image-20200911094119173](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911094119173.png)



> 注意：这个令牌只会明文显示一次，建议在配置插件的时候再来生成令牌，直接复制进去，搞丢了又要重新生成一个。



### 保存，完成即可。



### 4. 将仓库配置成giteePage页

我们需要通过链接来访问图片，这里将刚才建立的仓库设置成GiteePage页

1. 点击服务，选择Gitee Pages

![image-20200911095044868](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911095044868.png)



2. 如果自己想使用Https的图片，比如自己的博客网站是支持SSL认证的话，可以勾选强制使用Https;    这里参考：[为什么部署SSL证书后还是提示不安全](https://www.gschaos.club/%E4%B8%BA%E4%BB%80%E4%B9%88%E9%83%A8%E7%BD%B2SSL%E8%AF%81%E4%B9%A6%E5%90%8E%EF%BC%8C%E8%BF%98%E6%98%AF%E6%8F%90%E7%A4%BA%E4%B8%8D%E5%AE%89%E5%85%A8/)

![image-20200911095640287](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911095640287.png)



开启成功后再次访问就会变成下面的页面，这里的更新间隔是一分钟，需要手动更新，当然也可以配置WebHook触发服务器钩子来调用API自动更新，这里不涉及这方面，不展开。

![image-20200911095541779](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911095541779.png)

### 这样我们就获得了一个可以访问的网址

通过这个网址和上面PicGo配置的Path组合就可以访问我们需要的图片的。类似这种：

https://mysticalyu.gitee.io/pic/img/20200409141450-lee-gh-2.jpg



## Typora配置PicGo

**一个编写md文件的神器，官网地址：https://typora.io/ **  使用方法和基本配置见百度谷歌。

这里说明一下如何配置PicGo文件上传到服务器。

配置如下

![image-20200911094434501](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911094434501.png)



1. 配置好PigGo的执行文件
2. 验证一下图片是否能上传成功

![image-20200911094603050](https://gitee.com/MysticalYu/pic/raw/master/hexo/image-20200911094603050.png)



这里可以选择插入图片的操作，比如直接上传服务器。

以上配置后，基本就可以实现自己的图床了。



## 题外

GiteePage读取的是仓库的index.html页面，所以我们可以下载一些画廊的模板来放在index页面，至于画廊如何读取上传的图片就自己琢磨吧。

附上自己的画廊地址: https://mysticalyu.gitee.io/pic/