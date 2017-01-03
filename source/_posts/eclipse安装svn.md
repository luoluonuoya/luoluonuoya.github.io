---
title: eclipse安装svn
date: 2016-08-21 12:59:58
categories: [Java]
tags: [Java,eclipse]
---
#### eclipse在线安装svn：
　　打开Eclipse
　　Help->Software Updates->find and install(如果没有这个就用help->Software Updates->Add/Remove Software即可)

　　选择search for new features to install, Next
　　
　　点击new remote site（有以下版本可供选择）
　　输入
　　Name: Subclipse 1.6.x (Eclipse 3.2+)
　　URL:  http://subclipse.tigris.org/update_1.6.x
　　或
　　Name: Subclipse 1.4.x (Eclipse 3.2+)
　　URL:http://subclipse.tigris.org/update_1.4.x
　　或
　　Name: Subclipse 1.2.x (Eclipse 3.2+)
　　URL:  http://subclipse.tigris.org/update_1.2.x
　　或
　　Name: Subclipse 1.0.x (Eclipse 3.0/3.1)

　　选中subclipse，点击finish，一路NEXT。
　　
　　**注意：**有些时候会出现 Subclipse Integration for Mylyn 3.x (Optional) (3.0.0) requires plug-in "org.eclipse.mylyn.tasks.core (3.0.0)", or compatible.错误，这个不要紧，在弹出框中选择subclipse，把Subclipse Integration for Mylyn 3.x选项去掉即可Next一路安装完成！