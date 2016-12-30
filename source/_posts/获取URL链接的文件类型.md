---
title: 获取URL链接的文件类型
date: 2016-05-22 11:46:01
categories: [java]
tags: [java]
---
##### 事例中的URL原本是一个格式为wav的音频文件，file type: audio/x-wav，在保存文件的时候需要判断url是否还有效，这个时候就可以用到HttpURLConnection.guessContentTypeFromStream方法来做判断
```Java
package test;

import java.io.BufferedInputStream;
import java.io.IOException;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;

public class Test {

	public static void main(String[] args) {
		HttpURLConnection urlConn = null;
		URL url = null;
		BufferedInputStream bis = null;
     
        try {
        	url = new URL("http://www.ucpaas.com/fileserver/record/aee9011d2ee0f21f67310422ce62e71c_1464078387210374_20160524?sig=648DA4EC9E16704C44E5424D7A29CCD2"); 
			urlConn = (HttpURLConnection) url.openConnection();
			urlConn.connect();
			bis = new BufferedInputStream(urlConn.getInputStream());
			// HttpURLConnection.guessContentTypeFromStream 可以获取流的类型
			System.out.println("file type: " + HttpURLConnection.guessContentTypeFromStream(bis));
		} catch (MalformedURLException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			if(bis != null) {
				try {
					bis.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
	}
	
}
```
控制台打印结果：file type: text/html

---

##### 还可以使用下面这种方式获取文件类型

```Java
package test;

import java.io.IOException;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLConnection;

public class Test {

	public static void main(String[] args) {
		URL url = null;
		URLConnection conn = null;
     
        try {
        	url = new URL("http://www.ucpaas.com/fileserver/record/aee9011d2ee0f21f67310422ce62e71c_1464078387210374_20160524?sig=648DA4EC9E16704C44E5424D7A29CCD2"); 
        	conn = url.openConnection();
			System.out.println("file type: " + conn.getContentType());
		} catch (MalformedURLException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	
}
```
控制台打印结果：file type: text/html;charset=utf-8