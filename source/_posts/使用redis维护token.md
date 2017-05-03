---
title: 使用redis维护token
date: 2017-05-03 17:23:40
categories: [Java]
tags: [Java,redis]
---
### 用redis维护token的思路：
#### 存入
```Java
// 生成token
String token = UUID.randomUUID().toString();
// 将对象转为Json串
String json = JacksonUtils.newInstance().obj2json(user);
// 以token为key，对象为value缓存到redis
jedisUtils.set(token, json, 120 * 60);
// 将userId - token以key - value的形式放入到一个hash表中，这里最好遵循标准的命名规则：（项目：模块：形式）
jedisUtils.hset("project:sso:uid_token", "project:sso:".concat(user.getUserId()), token);
```
#### 校验
```Java
String token = request.getHeader("token");
if(null != token && StringUtils.isNotBlank(token)) {
	// 获取key为token的对象，并续租token
	String objJson = jedisUtils.get(token, 120 * 60);
	if (objJson != null && StringUtils.isNotBlank(objJson)) {
		// 获取对象
		User user = JSON.parseObject(objJson, User.class);
		if (user != null) {
			// 通过userid获取hash表中的token，并续租
			String val = jedisUtils.get(key, 120 * 60);
			if (val.equals(token)) {
				// 通过验证
			} else {
				// 不通过验证
			}
		} else {
			// 不通过验证
		}
	} else {
		// 不通过验证
	}
} else {
	// 提示需传token
}
```
**如果只以键值的形式存储，会有以下几种问题：** 
1、用户登录的时候重新分配了token，同一用户可重复登录
2、也可以以uid-token的形式存储，每次登录判断token是否一致，但这种就无法简略到只传token了
**用hash配合的逻辑** 
1、存一对token-user
2、hash存uid-token
3、每次接收token后拿到user，用uid去获取token
4、比对两个token
**好处：** 
B用户登录后，两个键值都改变了，A用户操作的时候用旧的token可以获取到user，但是hash中的token已经变了，所以验证不通过，因为设置了操作时效性，所以旧的token-user会在规定时间后失效，释放内存

**附：** jedisUtils工具类见【Java操作redis】文章