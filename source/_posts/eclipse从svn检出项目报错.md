---
title: eclipse从svn检出项目报错
date: 2016-03-13 13:21:23
categories: [Java]
tags: [Java,svn,eclipse]
---
##### 从svn把项目导入到eclipse后出现错误，运行又没有问题，提示如下：
```Java
Description	Resource	Path	Location	Type
Target runtime com.genuitec.embedded.tomcat.runtime.v70 is not defined.	XXX		
Unknown	Faceted Project Problem
```
##### 解决方法：找到项目目录下的.settings/org.eclipse.wst.common.project.facet.core.xml文件
```XML
<?xml version="1.0" encoding="UTF-8"?>
<faceted-project>
  <!-- 将这一句删除后刷新一下项目即可 -->
  <runtime name="com.genuitec.embedded.tomcat.runtime.v70"/>
  <fixed facet="jst.web"/>
  <fixed facet="wst.jsdt.web"/>
  <fixed facet="java"/>
  <installed facet="java" version="1.7"/>
  <installed facet="jst.web" version="3.0"/>
  <installed facet="wst.jsdt.web" version="1.0"/>
</faceted-project>
```
原因：新导入的工程出错的可能性非常高，但大多数都是缺少jar包导致的。还有一种缺少或者是错误的类库（比如JDK、Tomcat等等）。这里是因为跟项目原先导出时所使用的Tomcat冲突了。