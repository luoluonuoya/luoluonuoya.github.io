---
title: JDK1.7新特性
date: 2016-04-15 13:36:52
categories: [Java]
tags: [java]
---
小记jdk7新增的一些好用的小特性
```Java
// 1.7新增数字字面量下划线支持：int 和 long 可使用下划线分割数字
int oneMillion = 1_000_000;
System.out.println((oneMillion * 2) + "---" + oneMillion);

// 1.7新增二进制字面量支持
byte aByte = (byte) 0b1000;
short aShort = (short) 0b010;
System.out.println(aByte + "-" + aShort);

// 1.7新增自动关闭流支持，不必在finally中写关闭操作
try(InputStream is = new InputStream(file);) {
} catch (Exception e) {}

// 反射
try {
	Class cls = Class.forName("chb.test.reflect.Student");
	Method m = cls.getDeclaredMethod("方法名", new Class[]{int.class, String.class});  
	m.invoke(cls.newInstance(), 20, "chb");
} catch (Exception e) {
	e.printStackTrace();
}
```