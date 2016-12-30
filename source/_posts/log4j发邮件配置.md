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
```Java
log4j.rootLogger=INFO,stdout,R

#输出到控制台
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %5p [%F:%L] : %m%n

#输出到日志文件
log4j.appender.R=org.apache.log4j.DailyRollingFileAppender
#日志文件只输出WARN级别以上的日志
log4j.appender.R.Threshold=ERROR
#日志文件输出路径
log4j.appender.R.File=/usr/local/tomcat7/logs/web/xxx.log
#日志输出格式
log4j.appender.R.DatePattern='.'yyyy-MM-dd
log4j.appender.R.Append=true
log4j.appender.R.layout=org.apache.log4j.PatternLayout
log4j.appender.R.layout.ConversionPattern=[%-5p][%d{yyyyMMdd HH:mm:ss,SSS}][%C{1}:%L] %m%n
```
如果需要利用日志发送邮件，添加activation.jar、mail.jar两个包
```XML
<dependency>
	<groupId>javax.activation</groupId>
	<artifactId>activation</artifactId>
	<version>1.1.1</version>
</dependency>

<dependency>
	<groupId>javax.mail</groupId>
	<artifactId>mail</artifactId>
	<version>1.4.1</version>
</dependency>
```
然后配置文件中加入
```Java
#log4j的邮件发送appender
log4j.appender.MAIL=org.apache.log4j.net.SMTPAppender
#发送邮件的门槛，仅当等于或高于ERROR（比如FATAL）时，邮件才被发送
log4j.appender.MAIL.Threshold=ERROR
#缓存文件大小，日志达到1000k时发送Email，但如果是ERROR或FATAL则立即发送  
log4j.appender.MAIL.BufferSize=1024KB
#此处发送邮件的邮箱帐号
log4j.appender.MAIL.From=发送者的邮箱地址，比如xxxemail@163.com
#SMTP邮件发送服务器地址（这里以网易邮箱举例，比如谷歌就会是smtp.gmail.com）
log4j.appender.MAIL.SMTPHost=smtp.163.com
#SMTP发送认证的帐号名，邮箱的名称（不包含@163.com后面的信息）
log4j.appender.MAIL.SMTPUsername=xxxemail
#SMTP发送认证帐号的密码，邮箱的密码
log4j.appender.MAIL.SMTPPassword=发送者的邮箱密码，比如123456789
#是否打印调试信息，如果选true，则会输出和SMTP之间的握手等详细信息
log4j.appender.MAIL.SMTPDebug=true
#邮件主题
log4j.appender.MAIL.Subject=xxx项目错误日志
#发送到什么邮箱，如果要发送给多个邮箱，则用逗号分隔； 
log4j.appender.MAIL.To=xxx1@qq.com,xxx2@163.com,xxx3@gmail.com
#日志格式
log4j.appender.MAIL.layout=org.apache.log4j.PatternLayout
log4j.appender.MAIL.layout.ConversionPattern=[framework]%d - %c -%-4r[%t]%-5p %c %x -%m%n
```
如果顺利的话可以正常的发送，但可能会有特殊情况，比如标题乱码（也可能是版本低了），则可以重写 SMTPAppender 
> 看到配置中log4j.appender.MAIL=org.apache.log4j.net.SMTPAppender这一句，这里就是 log4j 输出的控制类，如果有其它个性化的定制，比如认证信息的处理等，都是通过重写SMTPAppender来实现。
```Java
// 邮件标题乱码问题
import java.io.UnsupportedEncodingException;
import org.apache.log4j.net.SMTPAppender;

public class EncodingSMTPAppender extends SMTPAppender {
	@Override
	public void setSubject(String subject) {
		try {
			subject = new String(subject.getBytes("iso8859-1"), "utf-8");
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		}
		super.setSubject(subject);
	}	
}
```
还有一个最常见的错误，配置完后发现若发送邮件失败 log4j: ERROR Error occured while sending e-mail notification.则是发送的邮箱没有开启stmp服务
> 像网易邮箱默认是没有开启smtp服务的，进入 邮箱-设置-POP3/SMTP/IMAP 中，把POP3/SMTP服务和IMAP/SMTP服务都勾选上