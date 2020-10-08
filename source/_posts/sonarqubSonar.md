---
title: sonarQube 
categories: 工具 
tags: [sonarQube]
date: 2019-11-14 
cover: https://mysticalyu.gitee.io/pic/img/daehwan-jang-pose2.jpg

---

SonarQube 是一款用于代码质量管理的开源工具，它主要用于管理源代码的质量。 通过插件形式，可以支持众多计算机语言，比如 java, C#, go，C/C++, PL/SQL, Cobol, JavaScrip, Groovy 等。sonar可以通过PMD,CheckStyle,Findbugs等等代码规则检测工具来检测你的代码，帮助你发现代码的漏洞，Bug，异味等信息。以下转自自己的CSDN博客：（关于截图背景颜色请无https://blog.csdn.net/qq_17238449/article/details/97392513



<!-- more -->



## Sonarqube搭建


### 1、安装

这里准备的是***sonarqube7.7.zip***，我的安装路径是/u02/ycc

使用unzip解压压缩包；

**预置条件**

1).已安装JAVA环境

2).已安装有MySQL数据库

3).sonarQube压缩包

### 2、数据库配置：



```sql
# 创建数据库sonar
create database sonar character set utf8 collate utf8_general_ci;

# 创建数据库用户sonar可用地址为192.168.6.226密码sonar
CREATE USER sonar@'192.168.6.226' identified by 'sonar';

# 赋权给用户sonar对数据库sonar有所有权限
grant all on sonar.* to 'sonar'@'%' identified by 'sonar';

# 授权sonar用户可以在本地连接数据库
grant all on sonar.* to 'sonar'@'localhost' identified by 'sonar';

# 授权sonar用户可以在226连接数据库
grant all on sonar.* to 'sonar'@'192.168.6.226' identified by 'sonar';

# 刷新权限
flush privileges;
```

![1564051508551](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_1.png)



**对权限的所有操作最后需要刷新下权限，即flush privileges;使之更改立马生效。**



### 3、修改sonar配置文件：sonar.properties



**我的数据库在17，使用时更改这个地址到自己的数据库地址即可。**

```prop
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:mysql://192.168.6.17:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL
```

**修改wrapper配置文件的java路径**

wrapper配置文件额sonar.properties在同一个目录里，这里需要注意一点，路径后面需要额外加上/java。不加会报

Unable to start JVM: Permission denied (13)的错误。

```properties
wrapper.java.command=/u02/ycc/jdk1.8.0_161/bin/java
```

**启动**

```pro
sh /u02/ycc/sonar.../bin/linux.../sonar.sh start
```

如果使用root的话会出现如下错误：

![1564098693803](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_2.png)

![1564098711687](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_3.png)

换个用户，并赋予这个用户sonar目录的权限即可。

### 4、启动sonarqube



  将中文插件sonar-l10n-zh-plugin-1.28.jar复制到extensions/plugins

![1564122590906](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_4.png)



### 5、安装SonarQube Scanner和配置



```html
//解压文件
//进入文件
//编辑文件
[root@localhost local]#unzip  sonar-scanner-cli-3.0.3.778-linux.zip
[root@localhost local]#mv sonar-scanner-cli-3.0.3.778-linux  sonar-scanner
[root@localhost local]# cd sonar-scanner
[root@localhost sonar-scanner]# vim conf/sonar-scanner.properties 
```

```pr
#Configure here general information about the environment, such as SonarQube DB details for example
#No information about specific project should appear here
#----- Default source code encoding
sonar.sourceEncoding=UTF-8
 
sonar.host.url=http://192.168.6.226:9000
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:mysql://192.168.6.17:3306/sonar?useUnicode=true&characterEncoding=utf8
sonar.login=sonar
sonar.password=sonar
```

在项目的根目录创建sonar-project.properties 

```proper
#sonar登陆用户
sonar.login=admin
#sonar登陆密码
sonar.password=admin
#需要扫描的项目对应的key自定义即可
sonar.projectKey=content-receive
#需要扫描的项目对应的显示项目名自定义即可
sonar.projectName=content-receive
sonar.projectVersion=1.0-SNAPSHOT
sonar.sourceEncoding=UTF-8
sonar.language=java
#扫描的源码位置
sonar.sources=src/main/java/com/jsc/content
#扫描的test位置
sonar.tests=src/test/java/com/jsc/content
#扫描java的源码位置
sonar.java.binaries=target/classes/com/jsc/content
```

在项目当前目录执行scanner ：

```
sh /sonarscannerdir/bin/sonar-scanner -X
```

运行结束在sonarQube页面即可看到刚才扫描的项目。



![1564109007362](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_5.png)

### 6、maven-sonar-plugin



对于Maven项目，除了使用SonarQube Scanner进行分析之外，还可以使用maven-sonar-plugin插件进行分析。使用maven-sonar-plugin插件的步骤如下：(setting.xml)

![1564110217662](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_6.png)

![1564110205988](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_7.png)

```xml
	<profile>
		<id>sonar</id>
		<activation>
		<activeByDefault>true</activeByDefault>
		</activation>
		<properties>
		<sonar.jdbc.url>jdbc:mysql://192.168.6.226:3306/sonar</sonar.jdbc.url>
		<sonar.jdbc.username>sonar</sonar.jdbc.username>
		<sonar.jdbc.password>sonar</sonar.jdbc.password>
			<sonar.host.url>http://192.168.6.226:9000</sonar.host.url>
		</properties>
		
		</profile> 
  </profiles>
   <activeProfiles>
        <activeProfile>UFindNexus</activeProfile>
		<activeProfile>sonar</activeProfile>
    </activeProfiles>
```



**pom.xml**的**build**中增加如下配置：



```xml
 <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.sonarsource.scanner.maven</groupId>
                    <artifactId>sonar-maven-plugin</artifactId>
                    <version>3.1.1</version>
                </plugin>
            </plugins>
        </pluginManagement>
```



在对应项目的控制台输入mvn clean verify sonar:sonar
或mvn clean install org.sonarsource.scanner.maven:sonar-maven-plugin:3.1.1:sonar执行扫描；这里注意，如果有多个maven的setting.xml会使用环境变量配置的setting.xml。执行完即可在sonarqube页面查看。

如果这里执行报错的话可以使用IDEA的run maven运行：

![1564121819350](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_8.png)



也可以在pom.xml中增加profile,此时选中sonar-project,执行 clean install sonar:sonar即可。

```xml
    <profiles>
        <profile>
            <id>sonar-project</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <sonar.jdbc.url>jdbc:mysql://192.168.6.213:3306/sonar</sonar.jdbc.url>
                <sonar.jdbc.username>root</sonar.jdbc.username>
                <sonar.jdbc.password>passok</sonar.jdbc.password>
                <sonar.host.url>http://192.168.6.213:9000</sonar.host.url>
                <!-- 需要忽略的-->
                <sonar.exclusions>src/main/java/com/jsc/codec/**</sonar.exclusions>
            </properties>
        </profile>
    </profiles>
```

![1564121973140](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_9.png)

---



## Sonarqube使用



SonarQube 是一个开源的代码分析平台, 用来持续分析和评测项目源代码的质量。 通过SonarQube我们可以检测出项目中重复代码， 潜在bug， 代码规范，安全性漏洞等问题， 并通过SonarQube web UI展示出来。

![1564124669244](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_10.png)



### 1.SonarQube扫描方法

Jenkins中调用 
通过jenkins插件调用sonarScanner或使用Maven、Gradle等内置扫描器 
依据项目需要，对代码持续扫描，并将结果推送到sonarqube 进行页面展示

SonarQube Scanner 
使用scanner，通过配置文件，修改项目信息，在命令行中调用scanner工具，进行扫描，并推送给sonarqube

Maven、Gradle等内置扫描器 

以maven为例，需要修改maven和sonarqube配置文件，在mvn编译后，使用mvn命令，进行代码扫描，并推送给sonarqube（需要编译源代码） ,参见上文。



### 2.SonarQube web UI 

显示用户所有的项目概况，各项目质量评级，并提供条件筛选 

![1564125155644](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_11.png)

### 3.SonarQube web UI –项目页面 
通过在主页面选择单个项目，进入项目详情，该页面提供了当前项目最近一次扫描的结果评级，历史累计和新增问题数量，代码行数等信息 。

![1564125184892](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_12.png)



### 4.SonarQube web UI –问题页面 

提供当前用户名下所有问题的列表，并提供条件筛选，包括问题类型，严重程度等 
在当个项目中，问题页面显示单项目信息 。

![1564125329743](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_13.png)



选中单个问题，查看问题代码详情，sonarqube给出问题描述和修改意见 。

![1564125632530](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_14.png)

### 5.SonarQube web UI –评估页面 

给出当前项目的评估概况信息，大小，可靠性，重复率，覆盖率等 。

![1564125742365](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_15.png)



### 6.SonarQube web UI –代码页面 



以.java文件为依据，给出各个.java文件统计信息 。

![1564126037590](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_16.png)



### 7.SonarQube web UI –活动页面 

页面展示了每次代码扫描的基本信息和代码情况的折线图，折线图可以根据需要调整显示bugs数量，代码行数，覆盖率等信息 。

![1564126123324](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_167.png)

---



## SonarQube Jekins集成



### 1、安装jenkins sonar插件。

略

### 2、配置sonarServer



进入系统管理-->系统配置界面。(这里选择测试环境的sonarQube地址)

![1564123349587](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_18.png)



进入系统管理-->全局工具配置

![1564123462680](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_19.png)



### 3、构建项目



回到主页找到需要配置的项目，如果没有则需要新建项目，这里不赘述如何创建。选中项目配置sonar(这里使用

sonar-scanner)。

![1564124297347](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_20.png)

![1564123835385](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_21.png)



在构建历史中可以看到运行中的构建，点进去查看信息：

![1564124138444](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_22.png)



另外一种方式是使用maven命令打包，此时需要配置setting.xml，配置见前文。

![1564128857055](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_23.png)



### 4、查看结果



![1564124244878](https://gitee.com/MysticalYu/pic/raw/master/hexo/sonarqube_24.png)

