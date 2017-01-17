---
title: maven打包可执行项目
date: 2017-01-17 15:20:15
categories: [Java]
tags: [Java,maven]
---
从路径读取jar的方式，比较好的做法是在项目中建一个lib目录来存放jar包
```XML
<dependency>
	<groupId>postmsg-ump</groupId>
	<artifactId>postmsg-ump</artifactId>
	<scope>system</scope>
	<!-- 版本随便，为了好管理从1.0开始写，我这里已经迭代很多了 -->
	<version>2.4</version>
	<!-- basedir表示根目录，如果不在项目中则写绝对路径，如d:\\postmsg-ump-2.4.jar -->
	<systemPath>${basedir}\src\lib\postmsg-ump-2.4.jar</systemPath>
</dependency>
```
也可以用命令的方式将jar包导入本地仓库
```
mvn install:install-file -DgroupId=postmsg-ump -DartifactId=postmsg-ump -Dversion=2.4 -Dpackaging=jar -Dfile=d:\postmsg-ump-2.4.jar 
```
-Dfile表示要加入仓库的jar包路径，执行命令后，在pom.xml中则不需要system和systemPath这两项了，但是不推荐使用这种方式，因为实际开发的过程中常需要协同开发，当把项目提交到代码库，从另一个地方把项目check out的之后，发现少了包

**最好的方法也是推荐的方法是建立一个共享私库，比如nexus**