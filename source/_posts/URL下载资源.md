---
title: URL下载资源
date: 2016-03-16 22:53:18
categories: [Java]
tags: [Java]
---
#### url资源下载，有一个很实用的工具类FileUtils
```Java
try {  
	URL httpUrl = new URL("http://xxx");
	File f = new File("存储路径+文件名+后缀");  
	FileUtils.copyURLToFile(httpUrl, f);  
} catch (Exception e) {  
	e.printStackTrace();
}
```