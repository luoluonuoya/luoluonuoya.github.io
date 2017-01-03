---
title: 使用fatjar打包项目
date: 2016-07-28 11:29:07
categories: [Java]
tags: [Java,eclipse]
---
> 直接用eclipse打包项目，如果项目有依赖的jar包，则运行时需要将生成的jar文件和存放引用包的lib文件夹放在同一个目录下，但是这种方式十分不美观，在非maven项目中，推荐使用fatjar打包项目，fatjar工具可以把引用包也引入到生成的jar文件中，使用方法不做解释，下面为使用失败的记录。
> 

##### 因为fatjar只有0.0.31版本，而0.0.31版本在eclipse 3.4之后就再没有更新过了，导致新版本的eclipse无法使用，有以下几种解决方案
**一、下载eclipse兼容版本插件**
　　使用eclipse的Help>Install new software>Work with后面的Add按钮，分别填入
　　The Eclipse Project Updates
　　和
　　http://download.eclipse.org/eclipse/updates/x.x（x.x是你目前eclipse的版本，我是4.4）

　　选中Eclipse Tests,Examples,and Extras下的Eclipse 2.0 Style Plugin Support，安装，安装完会提示重启，重启。
　　
　　这个时候eclipse就兼容了2.0插件，这时可使用以下两种方式安装fatjar插件:
　　
　　　**1.直接下载**
　　　fat-jar它是sourceforge.net下的一个开源工具，从
　　　[http://sourceforge.net/projects/fjep/files/](http://sourceforge.net/projects/fjep/files/)地址可以下载该工具，下载完成后是一个zip压缩包，将包中的net.sf.fjep.fatjar_0.0.31.jar放到eclipse安装目录的plugins下，重启eclipse即可（虽简单粗暴，但不推荐）
　　　如果下载不了，可以到我的csdn下载：[下载地址](http://download.csdn.net/detail/zhangfs_sl/9727387)
　　　**2.在线安装**
　　　Help>Install new software>Add-Name:Fat Jar;Location:http://kurucz-grafika.de/fatjar>OK>一直next，不管安装什么，Contact all update sites during install to find required softaware一般都不勾选，不然等更新要等很久

**二、下载更高版本的fastjar工具--net.sf.fjep.fatjar_0.0.32.jar(大神改过的)放到eclipse安装目录的plugins下，重启eclipse即可**
　　附fatjar0.0.32: [下载地址](http://download.csdn.net/detail/zhangfs_sl/9727391 "http://download.csdn.net/detail/zhangfs_sl/9727391")

**实在不行的话，就把第一种方法中的fatjar版本改成小一点的试试，如果还不行，就用maven打包吧**