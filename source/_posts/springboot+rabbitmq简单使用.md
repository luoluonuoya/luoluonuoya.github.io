---
title: springboot+rabbitmq简单使用
date: 2017-05-27 11:40:31
categories: [Java]
tags: [Java,springboot,mq]
---
搭建springboot的时候比较喜欢用多模块管理，所以先新建一个父maven程序，select archetype时选择maven-archetype-site-simple，创建完后，pom.xml如下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>per.example.mq</groupId>
	<artifactId>example-mq-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>pom</packaging>
	
	<name>example-mq-parent</name>
	<url>http://maven.apache.org</url>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>

	<dependencyManagement>
		<dependencies>

			<!-- Import dependency management from Spring Boot -->
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-dependencies</artifactId>
				<version>1.3.5.RELEASE</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>

			<!--如果要把springboot工程打包成war执行，需要该jar -->
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-legacy</artifactId>
				<version>1.0.2.RELEASE</version>
			</dependency>

		</dependencies>
	</dependencyManagement>

</project>
```
到此父maven就完成，把项目下的src文件夹删了，具体看实际需求
在父maven项目上右键新建maven module，选择maven-archetype-webapp创建一个web项目，完成，这里都是以eclipse为例，子maven的pom.xml如下
```xml
<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
	xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>per.example.mq</groupId>
		<artifactId>example-mq-parent</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>
	<artifactId>example-mq-server</artifactId>
	<packaging>war</packaging>
	<name>example-mq-server Maven Webapp</name>
	<url>http://maven.apache.org</url>

	<properties>
		<!-- 项目编码配置 start -->
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<!-- 项目编码配置 end -->
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
			<!-- 要使用LOG4J，去掉该依赖 -->
			<exclusions>
				<exclusion>
					<groupId>org.slf4j</groupId>
					<artifactId>log4j-over-slf4j</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>

		<!-- 使用log4j，不使用默认的logback -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-log4j</artifactId>
		</dependency>

		<!-- 要打成war包执行，需要该依赖 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-legacy</artifactId>
		</dependency>

		<!-- 要打成war包执行，去掉内嵌的tomcat -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>provided</scope>
		</dependency>
		
		<!-- 引入mq包 -->
		<dependency>  
			<groupId>org.springframework.boot</groupId>  
			<artifactId>spring-boot-starter-amqp</artifactId>  
		</dependency>

	</dependencies>

	<build>
		<finalName>sq580-mq-server</finalName>
	</build>
</project>
```
先创一个简单的app来测试springboot启动有没有问题，在最外层的包目录下创建一个Application.java类，必须在外层，如pre.example.mq路径下创建
```Java
package pre.example.mq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.context.web.SpringBootServletInitializer;

@SpringBootApplication
public class Application extends SpringBootServletInitializer {

	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
		return application.sources(Application.class);
	}

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```
在pre.example.mq路径下创建一个controller包，然后创建一个TestController.Java
```Java
package pre.example.mq.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import pre.example.mq.mq.data.service.MqSendService;

@Controller
@EnableAutoConfiguration
public class TestController {

	@Autowired
	private MqSendService mqSendService;
	
	@RequestMapping("/test")
    @ResponseBody
    String test() {
	    // 这个再这里暂时还没有用，可以先注释
		mqSendService.sendBindMsg();
        return "success";
    }
	
}
```
完成，点击项目执行Run As - Java Application也可以，直接像普通的web项目在tomcat下启动也可以，如果8080端口被占，前者要在配置里修改访问端口，后者跟平常使用无异，springboot还可以配置彩色日志，为了快速开发，此处都不赘述，启动后会看到类似下方的界面
```

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.3.5.RELEASE)

[2017-05-27 10:57:23.677] boot - 9464  INFO [localhost-startStop-1] --- Application: Starting Application on Yangshaokun-PC with PID 9464 (E:\apache-tomcat-7.0.67\webapps\wtpwebapps5\sq580-mq-server\WEB-INF\classes started by Yangshaokun in E:\eclipse)
[2017-05-27 10:57:23.679] boot - 9464  INFO [localhost-startStop-1] --- Application: The following profiles are active: develop
[2017-05-27 10:57:23.744] boot - 9464  INFO [localhost-startStop-1] --- AnnotationConfigEmbeddedWebApplicationContext: Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@294536ae: startup date [Sat May 27 10:57:23 CST 2017]; root of context hierarchy
[2017-05-27 10:57:25.200] boot - 9464  INFO [localhost-startStop-1] --- PostProcessorRegistrationDelegate$BeanPostProcessorChecker: Bean 'org.springframework.amqp.rabbit.annotation.RabbitBootstrapConfiguration' of type [class org.springframework.amqp.rabbit.annotation.RabbitBootstrapConfiguration$$EnhancerBySpringCGLIB$$4d0a742a] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
[2017-05-27 10:57:25.239] boot - 9464  INFO [localhost-startStop-1] --- [/sq580-mq-server]: Initializing Spring embedded WebApplicationContext
[2017-05-27 10:57:25.239] boot - 9464  INFO [localhost-startStop-1] --- ContextLoader: Root WebApplicationContext: initialization completed in 1496 ms
[2017-05-27 10:57:25.710] boot - 9464  INFO [localhost-startStop-1] --- TomcatWebSocketContainerCustomizer: NonEmbeddedServletContainerFactory detected. Websockets support should be native so this normally is not a problem.
[2017-05-27 10:57:26.017] boot - 9464  INFO [localhost-startStop-1] --- ServletRegistrationBean: Mapping servlet: 'dispatcherServlet' to [/]
[2017-05-27 10:57:26.018] boot - 9464  INFO [localhost-startStop-1] --- FilterRegistrationBean: Mapping filter: 'errorPageFilter' to: [/*]
[2017-05-27 10:57:26.018] boot - 9464  INFO [localhost-startStop-1] --- FilterRegistrationBean: Mapping filter: 'characterEncodingFilter' to: [/*]
[2017-05-27 10:57:26.019] boot - 9464  INFO [localhost-startStop-1] --- FilterRegistrationBean: Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
[2017-05-27 10:57:26.019] boot - 9464  INFO [localhost-startStop-1] --- FilterRegistrationBean: Mapping filter: 'httpPutFormContentFilter' to: [/*]
[2017-05-27 10:57:26.019] boot - 9464  INFO [localhost-startStop-1] --- FilterRegistrationBean: Mapping filter: 'requestContextFilter' to: [/*]
[2017-05-27 10:57:26.122] boot - 9464  INFO [localhost-startStop-1] --- AmqpConfig: recieveQueue:dataToCare_defaultrabbitmqIp:112.74.209.184:5672
[2017-05-27 10:57:26.361] boot - 9464  INFO [localhost-startStop-1] --- RequestMappingHandlerAdapter: Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@294536ae: startup date [Sat May 27 10:57:23 CST 2017]; root of context hierarchy
[2017-05-27 10:57:26.473] boot - 9464  INFO [localhost-startStop-1] --- RequestMappingHandlerMapping: Mapped "{[/test]}" onto java.lang.String com.sq580.mq.controller.TestController.one()
[2017-05-27 10:57:26.481] boot - 9464  INFO [localhost-startStop-1] --- RequestMappingHandlerMapping: Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
[2017-05-27 10:57:26.481] boot - 9464  INFO [localhost-startStop-1] --- RequestMappingHandlerMapping: Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
[2017-05-27 10:57:26.514] boot - 9464  INFO [localhost-startStop-1] --- SimpleUrlHandlerMapping: Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
[2017-05-27 10:57:26.514] boot - 9464  INFO [localhost-startStop-1] --- SimpleUrlHandlerMapping: Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
[2017-05-27 10:57:26.559] boot - 9464  INFO [localhost-startStop-1] --- SimpleUrlHandlerMapping: Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
[2017-05-27 10:57:26.988] boot - 9464  INFO [localhost-startStop-1] --- AnnotationMBeanExporter: Registering beans for JMX exposure on startup
[2017-05-27 10:57:26.998] boot - 9464  INFO [localhost-startStop-1] --- DefaultLifecycleProcessor: Starting beans in phase -2147482648
[2017-05-27 10:57:26.998] boot - 9464  INFO [localhost-startStop-1] --- DefaultLifecycleProcessor: Starting beans in phase 2147483647
[2017-05-27 10:57:27.156] boot - 9464  INFO [SimpleAsyncTaskExecutor-1] --- CachingConnectionFactory: Created new connection: SimpleConnection@35b4193e [delegate=amqp://openapi@112.74.209.184:5672/openapi]
[2017-05-27 10:57:27.346] boot - 9464  INFO [localhost-startStop-1] --- Application: Started Application in 4.331 seconds (JVM running for 7.63)
[2017-05-27 10:57:27.359] boot - 9464  INFO [main] --- Http11AprProtocol: Starting ProtocolHandler ["http-apr-9094"]
[2017-05-27 10:57:27.374] boot - 9464  INFO [main] --- AjpAprProtocol: Starting ProtocolHandler ["ajp-apr-8049"]
[2017-05-27 10:57:27.375] boot - 9464  INFO [main] --- Catalina: Server startup in 6336 ms
```
看到这里没有报错就说明成功了，下面开始来配置mq，mq的选择可以自己查询资料做对比，此处以rabbitmq为例，在子maven中要引入包，见pom.xml，在src/main/resources目录下创建一个application.yml或者properties文件，这里以yml为例
```yml
spring:
  profiles.active: develop

---
spring:
  profiles: develop
  output: 
    ansi: 
      enabled: detect
      
rabbitmq:
  ip: 你的mq服务器ip:5672
  user: guest
  password: guest
  virtualHost: myhost
  app:
    reciveQueue: xxx(队列名)
---
```
这样准备工作就做完了，mq的工作流程、原理、搭建此处不赘述，上官网下一个mq包，解压，运行即可，跟redis类似，上述配置文件中的output.ansi.enabled是开启彩色日志的。
首先，在pre.example.mq路径下创建config包，再创建一个AmqpConfig.java，如下
```Java
package pre.example.mq.config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.AcknowledgeMode;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.ChannelAwareMessageListener;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

import com.rabbitmq.client.Channel;

@Configuration
public class AmqpConfig {

	private final static Logger LOG = LoggerFactory.getLogger(AmqpConfig.class);

	@Value("${rabbitmq.app.reciveQueue}")
	private String recieveQueue;
	
	@Value("${rabbitmq.ip}")
	private String rabbitmqIp;
	
	@Value("${rabbitmq.user}")
	private String rabbitmqUser;
	
	@Value("${rabbitmq.password}")
	private String rabbitmqPassword;
	
	@Value("${rabbitmq.virtualHost}")
	private String virtualHost;

	public static final String EXCHANGE = "spring-boot-exchange";

	/**
	 * 不管是发送端还是接收端，都需要创建这个
	 */
	@Bean
	public ConnectionFactory connectionFactory() {
		CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
		connectionFactory.setAddresses(rabbitmqIp);
		connectionFactory.setUsername(rabbitmqUser);
		connectionFactory.setPassword(rabbitmqPassword);
		connectionFactory.setVirtualHost(virtualHost); // 如果不设置这个，默认在根目录
		connectionFactory.setPublisherConfirms(true); // 显示调用，才能进行消息的回调。
		return connectionFactory;
	}

	/**
	 * 发送者发送消息需要RabbitTemplate提供api，此处消费者不需要
	 */
	@Bean
	@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
	// 最好为prototype类型
	public RabbitTemplate rabbitTemplate() {
		RabbitTemplate template = new RabbitTemplate(connectionFactory());
		return template;
	}

	/**
	 * 针对消费者配置 1. 设置交换机类型 2. 将队列绑定到交换机 FanoutExchange:
	 * 将消息分发到所有的绑定队列，无routingkey的概念 HeadersExchange ：通过添加属性key-value匹配
	 * DirectExchange:按照routingkey分发到指定队列 TopicExchange:多关键字匹配
	 */
	@Bean
	public TopicExchange defaultExchange() {
		return new TopicExchange(EXCHANGE);
	}

	@Bean
	public Queue queue() {
		return new Queue(recieveQueue, true); // 队列持久
	}

	/**
	 * 这里有必要记录一下，这里queue跟exchange绑定了，
	 * 也就是生产者发消息到该exchange后，
	 * exchange会根据routingKey关键字发送到符合规则的queue中去，
	 * 如果不符合则丢弃，如果上rabbitmq的管理后台看的话可以看到即便项目关闭，
	 * exchang和queue还是存在绑定关系，看到defaultExchange()方法中，
	 * 创建的topicExchange使用了spring-boot-exchange这个名字，
	 * 如果临时想换个类型的交换机，则需要使用不同的exchang名字，
	 * 不然的话会报错，或者直接到管理后台把exchang或删了
	 */
	@Bean
	public Binding binding() {
		// 绑定队列到交换机中，并使用关键字匹配，后面会讲到关键字的匹配
		return BindingBuilder.bind(queue()).to(defaultExchange())
				.with("info.red.#");
	}

	/**
	 * 消费者接收消息并作出应答，确保消息不会因为消费者的奔溃而被丢弃
	 */
	@Bean
	public SimpleMessageListenerContainer messageContainer() {
		LOG.info("recieveQueue:" + recieveQueue + "rabbitmqIp:" + rabbitmqIp);
		SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(
				connectionFactory());
		container.setQueues(queue());
		container.setExposeListenerChannel(true);
		container.setMaxConcurrentConsumers(1);
		container.setConcurrentConsumers(1);
		container.setAcknowledgeMode(AcknowledgeMode.MANUAL); // 设置确认模式手工确认
		container.setMessageListener(new ChannelAwareMessageListener() {

			@Override
			public void onMessage(Message message, Channel channel)
					throws Exception {
				byte[] body = message.getBody();
				LOG.info("receive msg : " + new String(body));
				System.out.println("receive msg : " + new String(body));
				try {
					channel.basicAck(message.getMessageProperties()
							.getDeliveryTag(), false); // 确认消息成功消费
				} catch (Exception e) {
					LOG.error("消费队列失败:" + "", e);
					// 重试一次
					channel.basicNack(message.getMessageProperties()
							.getDeliveryTag(), false, false);
				}

			}
		});
		return container;
	}
}
```
打开TestController中的service调用，创建一个service，如
```Java
package pre.example.mq.mq.data.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import pre.example.mq.mq.MessageProducer;


@Service
public class MqSendService {
    
    @Autowired
    private MessageProducer messageProducer;
    
    /**
    * @Title: sendMsg
    * @Description: 发送消息
     */
    public void sendMsg(){
        messageProducer.sendMessage("info.red.a");
        messageProducer.sendMessage("info.red.b");
        messageProducer.sendMessage("info.black.a");
        messageProducer.sendMessage("err.red.a");
        messageProducer.sendMessage("info.red.a.a");
    }
}
```
MessageProducer.java
```Java
package pre.example.mq.mq;

import javax.annotation.Resource;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.stereotype.Service;

import pre.example.mq.config.AmqpConfig;

@Service
public class MessageProducer {
	
	private final static Logger LOG = LoggerFactory.getLogger(MessageProducer.class);

	@Resource
	private AmqpTemplate rabbitTemplate;
	/*public void setRabbitTemplate(RabbitTemplate rabbitTemplate) { // amqp 1.3.5之前版本需要显示设置消息确认，无法直接使用AmqpTemplate，需要使用RabbitTemplate
		this.rabbitTemplate = rabbitTemplate;
		this.rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback(){
		     @Override
		     public void confirm(CorrelationData correlationData, boolean ack, String cause) {
		         if (ack) {
		             System.out.println("消息确认成功");
		         } else {
		             //处理丢失的消息（nack）
		            System.out.println("消息确认失败");
		         }
		     }
		});
	}*/

	public void sendMessage(Object message) {
		LOG.debug("to send message:{}", message);
		try {
			rabbitTemplate.convertAndSend(AmqpConfig.EXCHANGE, message.toString(), message);
		} catch (Exception e) {
			LOG.error("发送队列消息错：" + "", e);
		}
	}
}
```
发送者很简单，这里我们通过接口访问的形式，调用了MessageProducer的sendMessage方法，rabbitTemplate.convertAndSend是发送消息的调用，第一个参数表示交换机，使用mq的订阅功能并不需要理会queue，只需要指定交换机，这里的交换机需要跟消费者一样；第二个参数是关键字匹配，如这里发送了5条消息，关键字如下
```
messageProducer.sendMessage("info.red.a");
messageProducer.sendMessage("info.red.b");
messageProducer.sendMessage("info.black.a");
messageProducer.sendMessage("err.red.a");
messageProducer.sendMessage("info.red.a.a");
```
看到AmqpConfig.java类的binding方法，那里消费者匹配了info.red.#方法，\*表示匹配一个，#表示匹配多个，所以这里发送的5条消息，只有1、2、4会被接收到，如果是info.red.\* 则只有1、2会被接收。