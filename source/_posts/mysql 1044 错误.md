---
title: mysql 1044 错误
date: 2016-05-01 23:11:43
categories: [mysql]
tags: [mysql]
---
不小心把 mysql 的 'root@localhost' 权限都删了，处理过程记录
```xml
【报错】#1044 - Access denied for user 'root'@'localhost' to database 'mysql'
 根据报错的内容在网上搜索出处理方法，在my.ini中的[mysqld]中添加
 #skip-grant-tables --授权表，可以在忘记密码的时候使用
 skip-external-locking --是跳过外部锁定
 skip-name-resolve --禁止DNS解析
```
```xml
【报错】 #1130 - Host '127.0.0.1' is not allowed to connect to this MySQL server
 注释掉 skip-name-resolve 后不报错了，
 但是又是之前的问题，
 于是打开 skip-grant-tables 
 解决，可以打开数据库了，重新为用户加上权限，保存
```
```xml
【报错】#1290 -the mysql server is running with the --skip-grant-tables option so it cannot execute this statement
字面意思是说skip-grant-tables这个开着的时候就不可更改
网上查找答案，先执行flush privileges后再次授权，

flush privileges 命令本质上的作用是将当前user和privilige表中的用户信息/权限设置从mysql库(MySQL数据库的内置库)中提取到内存里。MySQL用户数据和权限有修改后，希望在"不重启MySQL服务"的情况下直接生效，那么就需要执行这个命令。通常是在修改ROOT帐号的设置后，怕重启后无法再登录进来，那么直接flush之后就可以看权限设置是否生效。而不必冒太大风险。

继续报原来1044的错误
```
```xml
【另寻思路】关掉mysql服务，到其他安装了Mysql的服务器（前提是要知道该服务器上Mysql的root用户密码），打开[Mysql的安装目录/var/mysql]，将其中的user.frm、user.MYD、user.MYI三个文件拷贝到出问题服务器的[Mysql的安装目录/var/mysql]目录中。然后重启服务。

仍然不行

直接将mysql数据库中的user表中的所有N改为Y，注释掉skip-grant-tables重启服务，搞定。
```
```xml
【总结】首先，所有更改配置文件的操作都需要关服务修改后重启服务才能保证配置生效。
关闭mysql服务，在my.ini的[mysqld]中添加skip-grant-tables，打开服务，然后保证能看到用户下所有数据库后，
根据#1044 - Access denied for user 'root'@'localhost' to database 'mysql' 这句信息的字面意思，修改'mysql'数据库中的user表，将其中为N权限的全部手动改为Y，关服务，把skip-grant-tables注释掉，重启服务。
```
遇到的其它错误情况
```xml
【报错】The user specified as a definer ('root'@'%') does not exist
 出现该问题的原因是('root'@'%')没有找到，查看数据库，发现当前登录的是('root'@'localhost')。
 
 报错原因：
 数据库中的视图跟存储过程，之前是在另一个数据库使用('root'@'%')登录的时候创建的，且安全性是definer，
 备份到新的数据库的时候，当前的('root'@'localhost')没有足够的权限去操作，所以只需把视图和存储过程高级设置中的的安全性选项改为invoker即可。

 DEFINER的作用是用于指明存储过程是由哪个用户定义的;
 INVOKER用于指定哪些用户有调用存储过程的权限;
```