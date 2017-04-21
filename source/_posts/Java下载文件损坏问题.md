---
title: Java下载文件损坏问题
date: 2017-04-21 14:30:19
categories: [Java]
tags: [Java]
---
#### 开发的过程中经常需要导出各种报表，生成表的过程此处不表。说说下载遇到的问题：
##### 下载可以分为2种，一种是将文件保存到某个位置，然后返回url让client进行访问下载，但大多数时候我们并不需要这些文件，这个时候就浪费了磁盘空间，即便后面做了清理，也是比较不友好的一种情况。所以我们一般都是直接以流的形式将文件传给client
```Java
@RequestMapping("/export")
public ResponseEntity<byte[]> export() {
	
	byte[] body = null;
	String url = "你的资源文件url";
	HttpURLConnection conn = (HttpURLConnection) (new URL(url)).openConnection();
	// 不管你是读本地文件，网络文件还是Workbook等等，这里以网络文件为例
	InputStream in = conn.getInputStream();

	body = new byte[in.available()];
	in.read(body);
        
	HttpHeaders headers = new HttpHeaders();
	// 响应头的名字和响应头的值
	headers.add("Content-Disposition", "attachment;filename=" + "文件名，要加后缀");
        
	HttpStatus statusCode = HttpStatus.OK;
        
	ResponseEntity<byte[]> response = new ResponseEntity<byte[]>(body, headers, statusCode);
	return response;
	
}
```
##### 以上的方法可以基本满足下载的功能，但是我在实际开发中导出excel文件，发现下载的Excel文件打开的时候已经损坏了，本来是读取HSSFWorkbook直接返回的，为了方便排查，把生成的excel保存在了本地再执行响应，发现本地的excel打开是正常的，而下载后的却是损坏的，后来查阅了资料发现，如果一次性将流转为byte可能会有丢失问题，于是改用如下方式，将流先缓存包装起来，再返回，通过。
#### 修改代码如下
```Java
	HttpURLConnection conn = (HttpURLConnection) (new URL(url)).openConnection();
	InputStream in = conn.getInputStream();
		
	ByteArrayOutputStream swapStream = new ByteArrayOutputStream();  
	byte[] buff = new byte[1024];  
	int len = 0;  
	while ((len = in.read(buff, 0, 100)) > 0) {  
		swapStream.write(buff, 0, len);  
	}  
	byte[] body = swapStream.toByteArray();
        
	HttpHeaders headers = new HttpHeaders();
	// 响应头的名字和响应头的值
	headers.add("Content-Disposition", "attachment;filename=" + fileName);
        
	HttpStatus statusCode = HttpStatus.OK;
        
	ResponseEntity<byte[]> response = new ResponseEntity<byte[]>(body, headers, statusCode);
	return response;
```