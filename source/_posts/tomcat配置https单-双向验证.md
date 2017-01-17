---
title: tomcat配置https单/双向验证
date: 2016-06-12 10:51:28
categories: [Java]
tags: [Java,Tomcat]
---
#### **tomcat配置https双向验证**

**以windows为例，进入cmd终端，用命令切换进入%JAVA_HOME%/bin目录**

**1.为服务器生成证书**
```
keytool -genkey -v -alias tomcat -keyalg RSA -keystore D:\home\tomcat.keystore -validity 36500
```
**参数简要说明：**
```
“D:\home\tomcat.keystore” 表示证书文件的保存路径
tomcat.keystore 表示证书文件名称
“-validity 36500” 表示证书有效期，36500表示100年，默认值是90天 “tomcat”为自定义证书名称
```
**输入keystore密码（必填项）**
　　此处需要输入大于6个字符的字符串
**您的名字与姓氏是什么？（必填项）**
　　必须是TOMCAT部署主机的域名或者IP[如：baidu.com 或者 119.75.217.109]（就是你将来要在浏览器中输入的访问地址）
**后面的那些都可以不填，直接敲回车，最后的主密码建议与之前keystore的密码一致）事例如下：**
```
C:\Windows\system32>keytool -genkey -v -alias tomcat -keyalg RSA -keystore D:\home\tomcat.keystore -validity 36500
输入keystore密码：123456
再次输入新密码：123456
您的名字与姓氏是什么？
  [Unknown]：  baidu.com
您的组织单位名称是什么？
  [Unknown]：  
您的组织名称是什么？
  [Unknown]：  
您所在的城市或区域名称是什么？
  [Unknown]：  
您所在的州或省份名称是什么？
  [Unknown]：  
该单位的两字母国家代码是什么
  [Unknown]：  CN
CN=baidu.com, OU=, O=, L=, ST=, C=CN 正确吗？  [否]：  y
正在为以下对象生成 1,024 位 RSA 密钥对和自签名证书 (SHA1withRSA)（有效期为 36,500 天）:
         CN=baidu.com, OU=, O=, L=, ST=, C=CN
输入<tomcat>的主密码
        （如果和 keystore 密码相同，按回车）：
[正在存储 D:\home\tomcat.keystore]
C:\Windows\system32>
```
 **2.为客户端生成证书（需要PKCS12格式）**
```
keytool -genkey -v -alias mykey -keyalg RSA -storetype PKCS12 -keystore D:\home\client.key.p12
```
```
C:\Windows\system32>keytool -genkey -v -alias mykey -keyalg RSA -storetype PKCS12 -keystore D:\home\client.key.p12
输入keystore密码：123456
再次输入新密码：123456
您的名字与姓氏是什么？
  [Unknown]：  baidu.com
您的组织单位名称是什么？
  [Unknown]：  
您的组织名称是什么？
  [Unknown]：  
您所在的城市或区域名称是什么？
  [Unknown]：  
您所在的州或省份名称是什么？
  [Unknown]：  
该单位的两字母国家代码是什么
  [Unknown]：  CN
CN=baidu.com, OU=, O=, L=, ST=, C=CN 正确吗？  [否]：  y
正在为以下对象生成 1,024 位 RSA 密钥对和自签名证书 (SHA1withRSA)（有效期为 90 天）:
         CN=baidu.com, OU=, O=, L=, ST=, C=CN
[正在存储 D:\home\client.key.p12]
C:\Windows\system32>
```
之后，在D:\home\路径下，便能看到两个文件
```
tomcat.keystore
client.key.p12
```
**3.双击client.key.p12安装**
　　安装需要密码，就是刚才设置的"123456"
　　安装过程选择“将所有的证书放入下列存储”--“个人”，下一步直至导入完成
　　
**4.让服务器信任客户端证书**
先导出客户端证书为cer格式，然后再添加到keystore中，此过程需要密码，步骤如下：
```
1）keytool -export -alias mykey -keystore D:\home\client.key.p12 -storetype PKCS12 -storepass password -rfc -file D:\home\client.cer
2）keytool -import -v -file D:\home\client.cer -keystore D:\home\tomcat.keystore
```
**5.查看服务器的证书库**
此时list命令查看服务器的证书库，可以看到两个证书，一个是服务器证书，一个是受信任的客户端证书
　keytool -list -keystore D:\home\tomcat.keystore
　
**6.导出服务端证书为cer格式**
　keytool -keystore D:\home\tomcat.keystore -export -alias tomcat -file D:\home\tomcat.cer

**7.安装服务端证书**
双击，安装到“将所有的证书放入下列存储”--“受信任的根证书颁发机构”

**8.配置Tomcat服务器**
打开Tomcat根目录下的/conf/server.xml，找到Connector port="8443"配置段，修改为如下：
```XML
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
	SSLEnabled="true" maxThreads="150" scheme="https"
	secure="true" clientAuth="true" sslProtocol="TLS"
	keystoreFile="D:\\home\\tomcat.keystore" keystorePass="123456"
	truststoreFile="D:\\home\\tomcat.keystore" truststorePass="123456" />
```
**属性说明：**
　clientAuth:设置是否双向验证，默认为false，设置为true代表双向验证
　keystoreFile:服务器证书文件路径
　keystorePass:服务器证书密码
　truststoreFile:用来验证客户端证书的根证书，此例中就是服务器证书
　truststorePass:根证书密码

**9.最后在项目的web.xml中 </web-app> 前加入，之后就可以用8443端口访问https请求了**
```XML
<security-constraint>
	<web-resource-collection>
		<web-resource-name>Protected Context</web-resource-name>
		<url-pattern>/*</url-pattern>
	</web-resource-collection>
	<user-data-constraint>
		<transport-guarantee>CONFIDENTIAL</transport-guarantee>
	</user-data-constraint>
</security-constraint>
```

#### **省去上方的2、3、4、5步骤即为单项配置**