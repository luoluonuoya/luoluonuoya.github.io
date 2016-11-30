---
title: Java去除Html标签
date: 2016-11-24 17:07:43
categories: [Java]
tags: [java]
---
#### 普通的正则去除标签
```Java
// 剔出<html>的标签
String txtcontent = htmlcontent.replaceAll("</?[^>]+>", "");
// 去除字符串中的空格,回车,换行符,制表符
txtcontent = txtcontent.replaceAll("<a>\\s*|\t|\r|\n</a>", ""); 
```
#### 但是有时候html标签比较复杂，容易忽略掉很多，这时就可以使用Jsoup工具来完成这种复杂的解析，如果使用了maven，引入jar包
```
 <dependency>
	<groupId>org.jsoup</groupId>
	<artifactId>jsoup</artifactId>
	<version>1.9.2</version>
</dependency>
```
#### 工具类代码如下：
```Java
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;

public class HtmlUtil {
	/**
	* @Title: getText
	* @Description: 解析html标签
	* @param @param html串
	* @return 文本内容
	*/
	public static String getText(String html) {
		Document doc = Jsoup.parse(html);
		String txtcontent = doc.text();
		StringBuilder builder = new StringBuilder(txtcontent);
		int index = 0;
		while (builder.length() > index) {
			char tmp = builder.charAt(index);
			if (Character.isSpaceChar(tmp) || Character.isWhitespace(tmp)) {
				builder.setCharAt(index, ' ');
			}
			index++;
		}
		txtcontent = builder.toString().replaceAll(" +", " ").trim();
		return txtcontent;
	}
}
```