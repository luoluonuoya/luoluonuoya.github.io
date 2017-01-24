---
title: Spring boot的入门使用
date: 2017-01-24 16:25:49
categories: [Java]
tags: [Java,Srping boot]
---
pom.xml的配置
```XML
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>test</groupId>
  <artifactId>test</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>test</name>
  <url>http://maven.apache.org</url>
  
  	<!-- 若已经有父项目依赖，则在父项目中添加如下代码 -->
  	<!-- <dependencyManagement>
	<dependencies>
	   <dependency>
	       Import dependency management from Spring Boot
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-dependencies</artifactId>
	            <version>1.2.3.RELEASE</version>
	            <type>pom</type>
	            <scope>import</scope>
	        </dependency>
	    </dependencies>
	</dependencyManagement> -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.3.0.RELEASE</version>
	</parent>

	<properties>
	    <!-- 项目编码配置 start -->
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<!-- 项目编码配置 end -->
		
		<!-- 依赖的jar的版本号控制 start -->
		
		<!-- 依赖的jar的版本号控制 end -->
	</properties>

	<dependencies>
  
	    <!-- 核心模块，包括自动配置支持、日志和YAML -->
	    <dependency>
		    <groupId>org.springframework.boot</groupId>
		    <artifactId>spring-boot-starter</artifactId>
		</dependency>
		
	    <!-- 测试模块，包括JUnit、Hamcrest、Mockito -->
		<dependency>
		    <groupId>org.springframework.boot</groupId>
		    <artifactId>spring-boot-starter-test</artifactId>
		    <scope>test</scope>
		</dependency>
		
		<!-- Web模块 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
    
	</dependencies>
  
	<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <dependencies>
                    <dependency>
                        <groupId>org.springframework</groupId>
                        <artifactId>springloaded</artifactId>
                        <version>1.2.5.RELEASE</version>
                    </dependency>
                </dependencies>
                <configuration>
                	<!-- 热部署配置 -->
		            <jvmArguments>
		            	-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005
		            </jvmArguments>
					<!-- spring-boot 默认使用1.6，若使用其它版本，需要指定好相应版本 -->
	                <source>1.7</source>
	                <target>1.7</target>
	            </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```
model类
```Java
package per.test.mdoel;

/**
 * @Title: User.java
 * @Package per.test.mdoel
 * @Description: 用户实体类
 * @author zoro
 * @date 2017年1月22日 下午5:09:58
 */
public class User {

	private Long id;

	private String name;

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	@Override
	public int hashCode() {
		final int prime = 31;
		int result = 1;
		result = prime * result + ((id == null) ? 0 : id.hashCode());
		result = prime * result + ((name == null) ? 0 : name.hashCode());
		return result;
	}

	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (getClass() != obj.getClass())
			return false;
		User other = (User) obj;
		if (id == null) {
			if (other.id != null)
				return false;
		} else if (!id.equals(other.id))
			return false;
		if (name == null) {
			if (other.name != null)
				return false;
		} else if (!name.equals(other.name))
			return false;
		return true;
	}

	@Override
	public String toString() {
		return "User [id=" + id + ", name=" + name + "]";
	}

}
```
Controller类
```Java
package per.test.controller;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import per.test.mdoel.User;

/**  
 * @Title: UserController.java
 * @Package per.test.controller
 * @Description: 用户请求控制类
 * @author zoro
 * @date 2017年1月22日 下午5:31:52
 * pre: 
 * 	@Controller + @EnableAutoConfiguration = @RestController
 */
@RestController
@RequestMapping("/user")
public class UserController {
	
	@RequestMapping("/{id}")
	public User view(@PathVariable("id") Long id) {
		User user = new User();
        user.setId(id);
        user.setName("zhang");
        return user;
	}
	
}
```
运行
```Java
package per.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**  
 * @Title: UserController.java
 * @Package per.test.controller
 * @Description: 用户请求控制类
 * @author zoro
 * @date 2017年1月22日 下午5:33:01
 * pre: 
 * 	@SpringBootApplication继承了 @Configuration、 @EnableAutoConfiguration、 @ComponentScan
 */
@SpringBootApplication
public class AppTest {
	
    public static void main(String[] args) {
    	SpringApplication.run(AppTest.class);
	}
    
}
```