---
title: maven打包可执行项目
date: 2017-01-17 15:25:49
categories: [Java]
tags: [Java,maven]
---
pom.xml配置
```XML
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-assembly-plugin</artifactId>
			<version>2.3</version>
			<configuration>
				<archive>
					<manifest>
						<!-- 项目主清单（main）入口类 -->
						<mainClass>com.test.TestMain</mainClass>
					</manifest>
				</archive>
				<descriptors>
					<!-- 因为不是web程序，所以需要另外增加assmbly.xml文件 -->
					<descriptor>assembly.xml</descriptor>
				</descriptors>
			</configuration>
			<executions>
				<execution>
					<id>make-assembly</id>
					<phase>package</phase>
					<goals>
						<goal>single</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```
assmbly.xml（放在pom.xml同目录下）文件配置如下：
```XML
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd">
	<id>uberjar</id>
	<formats>
		<format>jar</format>
	</formats>
	<includeBaseDirectory>false</includeBaseDirectory>
	<dependencySets>
		<dependencySet>
			<!-- 引用的第三方jar包是否解压，如果为true，则解压第三方jar包为单独的文件目录；如果为false，则是将第三方jar包打进工程jar包下 -->
			<unpack>true</unpack>
			<scope>runtime</scope>
		</dependencySet>
	</dependencySets>
	<fileSets>
		<fileSet>
			<directory>${project.build.outputDirectory}</directory>
			<outputDirectory>/</outputDirectory>
		</fileSet>
	</fileSets>
</assembly>
```