---
title: 读取jar包中的文件
date: 2016-11-27 11:47:23
categories: [Java]
tags: [Java]
---
在jar包中读取资源文件，有时，我们的项目需要打成jar包，比如做maven父子依赖的时候，公共的项目被打成jar包放在子项目下，有一些公共的配置也被打入jar包中，读取相应文件的时候，如果按照常规web运行时的方式读取，读取到的路径是xxx.jar!xxx的形式，虽然路径是没有错，但是这种路径并不是一个目录路径，是不可读的，这时就需要用读流的方式来读取了

对于在classpath下的文件，一般web启动时我们都是通过ClassLoader来读取资源文件
```Java
// 必须和class同级目录或者子目录下才能读到
System.out.println(Test.class.getResource(""));
// 和调用classloader读取是同一个效果
System.out.println(Test.class.getResource("/"));
// 效果同上
System.out.println(Test.class.getClassLoader().getResource(""));
// 这是一种错误的读取方式，使用classloader的时候，不可以指定/来读取
System.out.println(Test.class.getClassLoader().getResource("/"));
```
正确的路径读取如下三种
```Java
System.out.println(Test.class.getResource("test.txt"));  
System.out.println(Test.class.getResource("/test.txt"));  
System.out.println(Test.class.getClassLoader().getResource("test.txt"));
```
假设要读取comm项目下的profile/test/test.txt文件，之后comm被打成jar包引入某web项目。通常，我们先获取路径再读取资源
```
File f = new File(Test.class.getClassLoader().getResource("/profile/test/test.txt").getPath());
try(FileInputStream fis = new FileInputStream(f);  InputStreamReader read = new InputStreamReader(fis, "UTF-8"); BufferedReader bufferedReader = new BufferedReader(read);) {
	String line = null;
	StringBuffer txtcontent = new StringBuffer();
	while((lineTxt = bufferedReader.readLine()) != null){
		txtcontent.append(line);
	}
	System.out.println(txtcontent.toString());
} catch (Exception e) {
	// TODO Exception
}
```
却发现，无论在被打成jar包的类里（comm项目中）读取还是在引用了该jar包的项目中的类（web项目中）读取，都会报找不到路径的错误，改用getResourceAsStream，这时读到的流是可用的
```Java
this.getClass().getResourceAsStream("/path"); 
```
代码示例：
```Java
InputStream is = Test.class.getResourceAsStream("/profile/test/test.txt");
InputStreamReader read = new InputStreamReader(is, "UTF-8");
BufferedReader bufferedReader = new BufferedReader(read);
String line = null;
StringBuffer txtcontent = new StringBuffer();
while ((line = bufferedReader.readLine()) != null) {
	txtcontent.append(line);
}
System.out.println(txtcontent.toString());
```
