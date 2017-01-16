---
title: StaleStateException
date: 2016-06-11 09:25:31
categories: [Java]
tags: [Java,hibernate]
---
**org.hibernate.StaleStateException**
org.hibernate.StaleStateException: Batch update returned unexpected row count from update: 0 actual row count: 0 expected: 1
这个错误的原因是当你更新的时候id不对应了，查看hibernate的sql输出，要么就是id原本是自增的，而在新增的时候自己给id赋值了。
我遇到的问题是新增的时候忘了调用save()方法，然后就去update()，这时update不到对象，结果就报错了