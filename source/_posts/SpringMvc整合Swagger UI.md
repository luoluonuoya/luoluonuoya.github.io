---
title: SpringMvc整合Swagger UI
date: 2017-01-14 09:48:05
categories: [Java]
tags: [Java,SpringMvc,api]
---
首先，导入主要的jar包
```XML
<!-- swagger-springmvc start -->
<dependency>
	<groupId>com.mangofactory</groupId>
	<artifactId>swagger-springmvc</artifactId>
	<version>1.0.2</version>
</dependency>
<dependency>
	<groupId>com.mangofactory</groupId>
	<artifactId>swagger-models</artifactId>
	<version>1.0.2</version>
</dependency>
<dependency>
	<groupId>com.wordnik</groupId>
	<artifactId>swagger-annotations</artifactId>
	<version>1.3.11</version>
</dependency>
<!-- swagger-springmvc end -->
```
定义一个config类来实现swagger的配置
```Java
package test.util;

import java.util.List;

import javax.inject.Inject;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

import com.mangofactory.swagger.configuration.SpringSwaggerConfig;
import com.mangofactory.swagger.models.dto.ApiInfo;
import com.mangofactory.swagger.plugin.EnableSwagger;
import com.mangofactory.swagger.plugin.SwaggerSpringMvcPlugin;

@Configuration
@EnableWebMvc
@EnableSwagger
public class SwaggerConfig {
	
	private SpringSwaggerConfig springSwaggerConfig;
	
	public SpringSwaggerConfig getSpringSwaggerConfig() {
		return springSwaggerConfig;
	}
	
	@Inject
	public void setSpringSwaggerConfig(SpringSwaggerConfig springSwaggerConfig) {
		this.springSwaggerConfig = springSwaggerConfig;
	}
	
	@Bean
	public SwaggerSpringMvcPlugin customImplementation() {
		return new SwaggerSpringMvcPlugin
				(this.springSwaggerConfig)
				.apiInfo(apiInfo())
				.includePatterns(".*?");
	}
	
	private ApiInfo apiInfo() {
		ApiInfo apiInfo = new ApiInfo(
                "系统管理平台API",
                "",
                "", 
                "",
                "",
                "");
        return apiInfo;
	}

}
```
在网上可能会看到@WebAppConfiguration配置，那是测试时候用的，发布的时候需要用@Configuration代替，@Configuration等价于xml里的beans标签，@EnableWebMvc注解驱动，将@RequestMapping的传入请求映射到一定的方法@Controller -annotated类的支持，@EnableSwagger自动注入SpringSwaggerConfig

上面的代码写完后springSwaggerConfig可能是null，因为还没有注入，有可能是@EnableSwagger没被扫到，也有可能是无效，加上以下配置
```XML
<bean class="com.mangofactory.swagger.configuration.SpringSwaggerConfig" />
```
也有可能没有报错，就是启动的时候并没有把Controller中的url管理起来，原因是需要在启动时注入我们自己写的配置，加入以下代码
```XML
<!-- swaggerUI 开始 -->
<mvc:annotation-driven/>
<bean class="test.SwaggerConfig"></bean>
<!-- swaggerUI 结束 -->
```
有如下的几种使用方式
```Java
@Api(value = "/user", description = "用户操作接口", position = 1) 
@ApiOperation(value = "/user/getdetail", httpMethod = "POST", notes = "获取用户详细信息", response = DetailResponseVo.class)
@ApiResponses(value = {  
	@ApiResponse(code = 200, message = "操作成功！", response = DetailResponseVo.class),  
	@ApiResponse(code = 404, message = "找不到页面"),  
	@ApiResponse(code = 500, message = "内部报错")}
)
@ApiModel(value = "用户信息")
@ApiModelProperty(value = "用户id")
```
说明
```
@Api表示一个开放的API，简要描述该API的功能。
在一个@API下，可有多个@ApiOperation，表示针对该API的CRUD操作。在ApiOperation Annotation中可以通过value，notes描述该操作的作用，response描述正常情况下该请求的返回对象类型。
在一个ApiOperation下，可以通过ApiResponses描述该API操作可能出现的情况。
@ApiParam用于描述该API操作接受的参数类型
@ApiModel用在类上
@ApiModelProperty用在类中的属性上
```
展示　[Swagger-UI下载地址](https://github.com/swagger-api/swagger-ui "https://github.com/swagger-api/swagger-ui")
```
从github下载Swagger-UI, 解压后把该项目dist目录下的内容拷贝到自己的项目中，
最好在webapp下建一个swagger目录来存放，编辑swagger/index.html文件，
找到url = "http://petstore.swagger.io/v2/swagger.json";这一句，
修改为url = "/projectName/api-docs";
projectName是指自己的项目名字；
也可以写成http://{ip}:{port}/{projectName}/api-docs这个格式
```
因为springMvc会拦截静态资源，所以还需加上下面的过滤配置
```XML
<mvc:resources mapping="/swagger/**" location="/swagger/"/>
```
打开浏览器直接访问swagger目录下的index.html文件即可，如
```
http://localhost:8080/projectName/swagger/index.html
```