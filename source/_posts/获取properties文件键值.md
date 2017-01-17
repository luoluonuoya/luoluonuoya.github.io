---
title: spring4.0读取properties文件
date: 2016-09-24 17:02:22
categories: [Java]
tags: [Java]
---
##### 若我们的项目配置了多环境
##### 注入properties属性，spring4.0配置，要引入
##### xmlns:util="http://www.springframework.org/schema/util"和
##### http://www.springframework.org/schema/util/spring-util-4.0.xsd

我的配置是
```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:util="http://www.springframework.org/schema/util" 
	xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
       http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd
       ">
	<description>spring总配置</description>

	<!-- 打开Spring的Annotation支持 -->
	<context:annotation-config />

	<!-- 设定Spring 去哪些包中找Annotation -->
	<context:component-scan base-package="com.test" />

	<!-- 使用aop -->
	<aop:aspectj-autoproxy proxy-target-class="true" />

	<!-- 读入配置属性文件 -->
	<context:property-placeholder location="classpath:${profiles.active}/system.properties" />
	
	<util:properties id="pro" location="classpath:${profiles.active}/system.properties"/>

	<!--引入其他spring配置文件  -->
	<import resource="classpath:spring/spring-datasource.xml" />
	<import resource="classpath:spring/spring-mybatits.xml" />
	<import resource="classpath:spring/spring-transaction.xml" />
	<import resource="classpath:spring/spring-redis.xml" />

</beans>

```
在类中的属性上@Value("#{pro.xxx}")就可以自动注入了
```Java
package com.sq580.mall.common.util.proper;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class PropsUtils {

	@Value("#{pro.protocal}")
	private String protocal;

	@Value("#{pro.endpoint}")
	private String endpoint;

	public String getProtocal() {
		return protocal;
	}

	public void setProtocal(String protocal) {
		this.protocal = protocal;
	}

	public String getEndpoint() {
		return endpoint;
	}

	public void setEndpoint(String endpoint) {
		this.endpoint = endpoint;
	}
	
}
```
直接在需要使用的地方注入PropsUtils，然后get需要的属性即可，需要注意的是，配置中的属性名不能使用“.”