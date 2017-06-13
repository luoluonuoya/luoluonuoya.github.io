---
title: 使用kong管理api
date: 2017-06-05 14:42:16
categories: [kong]
tags: [kong]
---
*摘要：Kong，Mashape开源的API层。是基于Nginx_Lua模块写的。是一个基于openresty的api代理层，数据采用了 Apache Cassandra/PostgreSQL存储，并且提供了一些优秀的插件，比如验证，日志，调用频次限制等。*
详细资料见官网：https://getkong.org
环境：ubuntu 16.04
1、下载kong：https://getkong.org/install/ 在此页面选择你的系统，之后再下载相应的系统版本，因为我是 unbuntu 16.04，则下载https://github.com/Mashape/kong/releases/download/0.10.3/kong-0.10.3.xenial_all.deb

2、安装kong，根据官网提示，执行下面三条命令
```
$ sudo apt-get update
$ sudo apt-get install openssl libpcre3 procps perl
$ sudo dpkg -i kong-0.10.3.*.deb
```
3、kong的使用肯定需要存储一些自己的数据，目前支持PostgreSQL和Cassandra两种数据库，这里以postgresql为例进行安装

3.1 添加postgresql源：
```
$ sudo touch /etc/apt/sources.list.d/pgdb.list
$ sudo vim /etc/apt/sources.list.d/pgdb.list
```
3.2 将下文添加到pgdb.list中
```
deb https://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main
```
3.3 执行如下命令添加postgresql安装包的秘钥：
```
$ sudo wget --quiet -O - https://postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```
3.4 安装
```
sudo apt-get update
sudo apt-get install postgresql-9.4
```
3.5 验证是否成功
```
$ dpkg -l |grep postgresql
```
3.6 看到如下信息则为成功
```
ii  pgdg-keyring                                2017.1                                     all          keyring for apt.postgresql.org
ii  postgresql-9.4                              9.4.12-1.pgdg14.04+1                       amd64        object-relational SQL database, version 9.4 server
ii  postgresql-client-9.4                       9.4.12-1.pgdg14.04+1                       amd64        front-end programs for PostgreSQL 9.4
ii  postgresql-client-common                    182.pgdg14.04+1                            all          manager for multiple PostgreSQL client versions
ii  postgresql-common                           182.pgdg14.04+1                            all          PostgreSQL database-cluster manager
ii  postgresql-contrib-9.4                      9.4.12-1.pgdg14.04+1                       amd64        additional facilities for PostgreSQL
```
3.7 postgres是超级管理员，类似mysql的root和sqlserver的sa，一开始是没有密码的，用超级管理员登录
```
$ sudo -i -u postgres
```
3.8 创建一个数据库
```
createdb kongdb
```
3.8.1 如果出现类似createdb: 无法联接到数据库 template1: 无法联接到服务器: 没有那个文件或目录
	服务器是否在本地运行并且在 Unix 域套接字
	"/var/run/postgresql/.s.PGSQL.5432"上准备接受联接?的错误则切回ubuntu的登录用户，执行下列操作
```
$ sudo service postgresql start
$ sudo systemctl unmask postgresql
$ sudo systemctl restart postgresql
$ sudo service postgresql start
```
3.9 创建角色（这里我在psql工具中创建一直不成功不知为啥，所以用sql来创建）
```
create user kong
```
3.10 通过postgresql的客户端来实现
```
psql
```
3.11 为"kong"用户设置密码
```
\password 123456
```
3.12 把数据库 kongdb的所有者设置为 kong
```
alter database kongdb owner to kong
```
3.13 介绍一下一些基本的命令
```
\help 查看psql工具的命令
\q 退出
\du 查看所有用户（刚才创建后用这个查一下）
\l 查看所有数据库（同上）
```
3.14 用 \q 命令退出并用kong用户登录kongdb数据库
```
psql -d kongdb -U kong -h 127.0.0.1 -W
```
3.15 这时会提示输入密码，输入刚才设置的123456即可

4、为kong配置数据源，找到/usr/local/share/lua/5.1/kong/templates/中的kong_defaults.lua，将里面的pg\_pssword = NONE改为刚才设置的123456，pg\_database = kong改为刚才创建的kongdb，如果无意外的话，这里可以启动kong了
```
$ sudo kong start
```
5、查阅官网kong的api操作，不同版本可能参数不同，这里以0.10.x版本为例，详见： https://getkong.org/docs/0.10.x/getting-started/adding-your-api/ 
6、安装curl，如果已经装过git可能会有冲突，我这里就遇到了，记录一下
```
$ sudo apt-get install curl
```
6.1 报错
```
curl : 依赖: libcurl3-gnutls (= 7.47.0-1ubuntu2) 但是 7.47.0-1ubuntu2.2 正要被安装
```
6.2 一开始不清楚原因，执行了$ sudo apt-get update，出现了如下错误
```
E: 无法获得锁 /var/lib/apt/lists/lock - open (11: 资源暂时不可用) 
E: 无法对目录 /var/lib/apt/lists/ 加锁
```
6.3 解决方法，强制删除
```
$ sudo rm /var/cache/apt/archives/lock

$ sudo rm /var/lib/dpkg/lock
```
6.4 **6.3、6.4两个操作其实是没必要的，只是作为记录**这里说说6.1的问题，重新安装 sudo apt-get install libcurl3-gnutls是没用的，这里是包冲突，需要指定版本来安装
```
$ sudo apt-get install libcurl3-gnutls=7.47.0-1ubuntu2
```
6.5 安装完后再执行一下$ sudo apt-get install curl即可

7、将业务侧的api加到kong中
```
curl -i -X POST --url http://localhost:8001/apis/ --data 'name=test' --data 'upstream_url=http://192.168.0.1:8080/' --data 'uris=/'  
```
7.1 这里需要说明，根据官网/apis/接口request body参数说明中，hosts、uris、methods需要至少指定一个，不赘述
7.2 name指的是在kong api中的一个名字标识，具有唯一性，这里创建了test后，不允许再创建同名，若上方命令执行两次会报错
7.3 可以到/usr/local/kong/logs中查看日志

8、然后就可以用kong来转发请求了，在https://getkong.org/docs/0.10.x/getting-started/adding-your-api/中有这么一句：
```
Issue the following cURL request to verify that Kong is properly forwarding requests to your API. Note that by default Kong handles proxy requests on port :8000
```
所以这里可以用 http://localhost:8000/ + 资源的方法访问http://192.168.0.1:8080/ 下所有的资源

9、Kong的一个非常诱人的地方就是提供了大量的插件来扩展应用，通过设置不同的插件可以为服务提供各种增强的功能。执行下面命令可以查看kong的所有插件
```
curl -i -X GET --url http://localhost:8001/plugins/enabled
```
9.1 用Rate Limiting来控制访问访问次数
```
curl -i -X POST --url http://localhost:8001/apis/test/plugins/ --data "name=rate-limiting" --data "config.minute=5"  
```
接着随访找一个接口连续访问6次，如 http://localhost:8000/test 这个接口，第6次的时候就会提示 {"message": "API rate limit exceeded"}
9.2 另外kong集成了Galileo和Datadog，只要有账号就可以监控一些数据，具体到各自官网了解详情，另外Runscope是收费的，没有试过。