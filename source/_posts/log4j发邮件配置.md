---
title: Log4j发邮件配置
date: 2016-04-08 09:12:09
categories: [Java]
tags: [java]
---
引入log4j.jar包，这里解释一下三个包的关系：
>**slf4j-api** 本质就是一个接口定义
**slf4j-log4j12** 是链接slf4j-api和log4j中间的适配器。它实现了slf4j-api中StaticLoggerBinder接口，从而使得在编译时绑定的是slf4j-log4j12的getSingleton()方法
**log4j** 是具体的日志系统。通过slf4j-log4j12初始化Log4j，达到最终日志的输出。

slf4j和log4j经常存在冲突的现在，需要在网上下载对应的版本，以下为无冲突的版本实例
```XML
<dependency>
	<groupId>log4j</groupId>
	<artifactId>log4j</artifactId>
	<version>1.2.16</version>
</dependency>

<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-api</artifactId>
	<version>1.7.5</version>
</dependency>

<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-log4j12</artifactId>
	<version>1.7.5</version>
</dependency>
```
log4j.properties文件配置
```J
log4j.rootLogger=WARN, stdout, R
	log4j.appender.stdout=org.apache.log4j.ConsoleAppender
	log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
	# Pattern to output the caller's file name and line number.
	#log4j.appender.stdout.layout.ConversionPattern=%5p [%t] (%F:%L) - %m%n
	# Print the date in ISO 8601 format
	log4j.appender.stdout.layout.ConversionPattern=%d [%t] %-5p %c - %m%n
	log4j.appender.R=org.apache.log4j.RollingFileAppender
	log4j.appender.R.File=example.log// 这里在eclipse的根路径下生成example.log
	log4j.appender.R.MaxFileSize=100KB
	# Keep one backup file
	log4j.appender.R.MaxBackupIndex=1
	log4j.appender.R.layout=org.apache.log4j.PatternLayout
	log4j.appender.R.layout.ConversionPattern=%p %t %c - %m%n
	# Print only messages of level WARN or above in the package com.foo.
	log4j.logger.com.foo=WARN
```