---
title: Mybatis执行insert后获取主键方式
date: 2016-08-21 16:21:16
categories: [Java]
tags: [Java,mybatis]
---
```
在hibernate中，hibernate的一级缓存会把insert后的主键绑定到对象中，我们可以直接在session.save(user);
后直接使用user.getId();的方式来取得插入后的主键，而在mybatis中，我们一般使用mapper.xml来编辑sql语句，
当我们执行insert后，返回的model中并没有帮我们把主键返回
```
使用selectKey来实现类似JDBC的getGeneratedKeys();获取主键的功能
```XML
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="test.dao.UserDao" >
  <resultMap id="BaseResultMap" type="test.model.User" >
    <id column="store_id" property="storeId" jdbcType="BIGINT" />
    <result column="name" property="name" jdbcType="VARCHAR" />
    <result column="role" property="role" jdbcType="BIGINT" />
    <result column="mobile" property="mobile" jdbcType="VARCHAR" />
  </resultMap>
  
<insert id="insert" parameterType="test.model.User">
  	insert into u_user
    <trim prefix="(" suffix=")" suffixOverrides="," >
      <if test="userId != null" >
        user_id,
      </if>
      <if test="name != null" >
        name,
      </if>
      <if test="role != null" >
        role,
      </if>
      <if test="mobile != null" >
        mobile,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides="," >
      <if test="userId != null" >
        #{userId,jdbcType=BIGINT},
      </if>
      <if test="name != null" >
        #{name,jdbcType=VARCHAR},
      </if>
      <if test="role != null" >
        #{role,jdbcType=BIGINT},
      </if>
      <if test="mobile != null" >
        #{mobile,jdbcType=VARCHAR},
      </if>
    </trim>
    <selectKey resultType="java.lang.Long" keyProperty="userId">  
      <![CDATA[SELECT LAST_INSERT_ID() AS user_id ]]>
    </selectKey>
</insert>
```
这里需要说明一下不同的数据库主键的生成，对各自的数据库有不同的方式：
```plain
mysql:	SELECT LAST_INSERT_ID() AS VALUE
mssql:	SELECT @@IDENTITY AS VALUE
oracle:	SELECT STOCKIDSEQUENCE.NEXTVAL AS VALUE FROM DUAL
```
还有一点需要注意的是不同的数据库生产商生成主键的方式不一样，有些是预先生成 (pre-generate)主键的，如Oracle和PostgreSQL。有些是事后生成(post-generate)主键的，如MySQL和SQL Server 所以如果是Oracle数据库，则需要将selectKey写在insert语句之前