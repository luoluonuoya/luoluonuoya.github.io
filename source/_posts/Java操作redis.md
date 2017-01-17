---
title: Java操作redis
date: 2016-07-17 11:53:17
categories: [Java]
tags: [Java]
---
Jedis的简单使用
```Java
package test;

import java.util.Iterator;
import java.util.List;
import java.util.Set;

import redis.clients.jedis.Jedis;

public class Test {

	public static void main(String[] args) {
		// 连接本地的 redis 服务
		Jedis jedis = new Jedis("localhost");
		System.out.println("Connection to server sucessfully");
		// 存储数据到列表中
		jedis.lpush("tutorial-list", "Redis");
		jedis.lpush("tutorial-list", "Mongodb");
		jedis.lpush("tutorial-list", "Mysql");
		// 获取存储的数据并输出
		List<String> list = jedis.lrange("tutorial-list", 0, 5);
		for (int i = 0; i < list.size(); i++) {
			System.out.println("Stored string in redis: " + list.get(i));
		}
		// 获取数据并输出
		Set<String> list1 = jedis.keys("*");
		Iterator<String> it = list1.iterator();
		while (it.hasNext()) {
			System.out.println("List of stored keys: " + it.next());
		}
	}
	
}
```
在实际的开发中操作redis总是需要用到连接池，同时redis也需要加密
先说一下windows上加密的坑
```
设置密码：
redis 127.0.0.1:6379> config set requirepass test123
查询密码：
redis 127.0.0.1:6379> config get requirepass
(error) ERR operation not permitted
密码验证：
redis 127.0.0.1:6379> auth test123
OK
再次查询：
redis 127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "test123"
```
PS：如果配置文件中没添加密码 那么redis重启后，密码失效；所以我们需要在配置文件中配置requirepass的密码（当redis重启时密码依然有效）。
打开redis.windows.conf配置文件，找到requirepass，然后修改如下:
```
requirepass yourpassword
```

然而发现在windows下，启动redis-server.exe后用redis-cli.exe登录后并不需要密码验证，原因是redis-server.exe的启动并不依赖redis.windows.conf，我的解决方法是为redis-server.exe创建一个快捷方式，然后右键属性，将目标后加上：（空格）+ redis.windows.conf，如果有更好的方法，欢迎留言。

当然，如果想把redis注册成windows服务也可以
```
注册服务
Redis-server.exe –service-install redis.windows.conf
删除服务
redis-server –service-uninstall
开启服务
redis-server –service-start
停止服务
redis-server –service-stop
```
下面提供jedis操作的工具类
```Java
package test.util;

import java.util.List;
import java.util.Map;
import java.util.Set;

import org.apache.commons.lang.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.google.common.collect.Maps;
import test.util.ObjectUtils;
import test.util.StringUtil;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.exceptions.JedisException;

public class JedisUtils {
	private final static Logger logger = LoggerFactory.getLogger(JedisUtils.class);
	
	public final static int DEFAULT_CACHE_SECONDS = 30 * 60; //缓存30分钟
	
	// 这里如果用spring的话可以使用配置注入，配置见文章末尾
	public JedisPool jedisPool;

	public JedisPool getJedisPool() {
		return jedisPool;
	}

	public void setJedisPool(JedisPool jedisPool) {
		this.jedisPool = jedisPool;
	}

	// 如果不使用注入，这里初始化一下连接池
	public void init() {
		if (jedisPool== null) {
			JedisPoolConfig config = new JedisPoolConfig();
			// 控制一个pool可分配多少个jedis实例，通过pool.getResource()来获取
			config.setMaxActive(50);
			// 控制一个pool最多有多少个状态为idle(空闲的)的jedis实例
            config.setMaxIdle(5);
            // 表示当borrow(引入)一个jedis实例时，最大的等待时间，如果超过等待时间，则直接抛出JedisConnectionException
            config.setMaxWait(1000 * 100);
            // 在borrow一个jedis实例时，是否提前进行validate操作；如果为true，则得到的jedis实例均是可用的
            config.setTestOnBorrow(true);  
            // 这里获取配置不多说，后面的3600表示超时时间，database表示数据库号，都可以不填
            pool = new JedisPool(config, "host:port", password, 3600, database);
		}
	}

	/**
	 * 获取缓存
	 * @param key 键
	 * @return 值
	 */
	public String get(String key)throws Exception {
		String value = null;
		Jedis jedis = null;
		try {
			jedis = getResource();
			if (jedis.exists(key)) {
				value = jedis.get(key);
				value = StringUtils.isNotBlank(value) && !"nil".equalsIgnoreCase(value) ? value : null;
				logger.debug("get {} = {}", key, value);
			}
		} catch (Exception e) {
			logger.warn("get {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return value;
	}
	
	/**
	 * 获取缓存
	 * @param key 键
	 * @param  cacheSeconds 缓存时间
	 * @return 值
	 */
	public String get(String key, int cacheSeconds)throws Exception {
		String value = null;
		Jedis jedis = null;
		try {
			jedis = getResource();
			if (jedis.exists(key)) {
				value = jedis.get(key);
				if (cacheSeconds != 0) {
					jedis.expire(key, cacheSeconds);
				}
				value = StringUtils.isNotBlank(value) && !"nil".equalsIgnoreCase(value) ? value : null;
				logger.debug("get {} = {}", key, value);
			}
		} catch (Exception e) {
			logger.warn("get {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return value;
	}
	
	
	/**
	 * 设置缓存
	 * @param key 键
	 * @param value 值
	 * @param cacheSeconds 超时时间，0为不超时
	 * @return
	 */
	public String set(String key, String value, int cacheSeconds) throws Exception{
		String result = null;
		Jedis jedis = null;
		try {
			jedis = getResource();
			result = jedis.set(key, value);
			if (cacheSeconds != 0) {
				jedis.expire(key, cacheSeconds);
			}
			logger.debug("set {} = {}", key, value);
		} catch (Exception e) {
			logger.warn("set {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}
	
	/**
	 * 获取单个hash缓存
	 * @param @param key
	 * @param @param field 
	 * @return String
	 */
	public String hget(String key, String field) throws Exception {
		String value = null;
		Jedis jedis = null;
		try {
			jedis = getResource();
			value = jedis.hget(key, field);
			value = StringUtils.isNotBlank(value) && !"nil".equalsIgnoreCase(value) ? value : null;
			logger.debug("hget {} = {}", field, value);
		} catch (Exception e) {
			logger.warn("hget {} = {}", field, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return value;
	}

	/**
	 * 设置单个hash缓存
	 * @param @param key
	 * @param @param field 
	 * @return String
	 */
	public String hset(String key, String field, String value) throws Exception {
		String result = null;
		Jedis jedis = null;
		try {
			jedis = getResource();
			jedis.hset(key, field, value);
			logger.debug("hset {} : {} = {}", key, field, value);
		} catch (Exception e) {
			logger.warn("hset {} : {} = {}", key, field, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}
	
	/**
	 * 获取List缓存
	 * @param key 键
	 * @return 值
	 */
	public List<String> getList(String key) throws Exception{
		List<String> value = null;
		Jedis jedis = null;
		try {
			jedis = getResource();
			if (jedis.exists(key)) {
				value = jedis.lrange(key, 0, -1);
				logger.debug("getList {} = {}", key, value);
			}
		} catch (Exception e) {
			logger.warn("getList {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return value;
	}
	
	
	/**
	 * 设置List缓存
	 * @param key 键
	 * @param value 值
	 * @param cacheSeconds 超时时间，0为不超时
	 * @return
	 */
	public long setList(String key, List<String> value, int cacheSeconds)throws Exception {
		long result = 0;
		Jedis jedis = null;
		try {
			jedis = getResource();
			if (jedis.exists(key)) {
				jedis.del(key);
			}
			result = jedis.rpush(key, (String[])value.toArray());
			if (cacheSeconds != 0) {
				jedis.expire(key, cacheSeconds);
			}
			logger.debug("setList {} = {}", key, value);
		} catch (Exception e) {
			logger.warn("setList {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}
	
	
	/**
	 * 向List缓存中添加值
	 * @param key 键
	 * @param value 值
	 * @return
	 */
	public long listAdd(String key, String... value)throws Exception  {
		long result = 0;
		Jedis jedis = null;
		try {
			jedis = getResource();
			result = jedis.rpush(key, value);
			logger.debug("listAdd {} = {}", key, value);
		} catch (Exception e) {
			logger.warn("listAdd {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}

	/**
	 * 获取有序zet缓存 （按分值由低到高排序输出）
	 * @param key 键
	 * @return 值
	 */
	public Set<String> getZsetZrange(String key) throws Exception {
		return getZsetZrange(key,0 , -1);
	}
	
	/**
	 * 获取有序zet缓存 （按分值由高到低排序输出）
	 * @param key 键
	 * @return 值
	 */
	public Set<String> getZsetZrevrange(String key)throws Exception{
		return  getZsetZrevrange(key,0 , -1);
	}
	

	/**
	 * 
	* @Title: getZsetZrangePage
	* @Description: 分页获取 获取有序zet缓存 （按分值由低到高排序输出）
	* @param @param key
	* @param @param end
	* @param @return    
	* @return Set<String>    
	* @throws
	 */
	public Set<String> getZsetZrange(String key,long start , long end) throws Exception{
		Set<String> value = null;
		Jedis jedis = null;
		try {
			jedis = getResource();
			if (jedis.exists(key)) {
				value = jedis.zrange(key, start, end);
				logger.debug("getZset {} = {}", key, value);
			}
		} catch (Exception e) {
			logger.warn("getZset {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return value;
	}
	
	/**
	 * 
	* @Title: getZsetZrevrange
	* @Description: 获取有序zet缓存 （按分值由高到低排序输出）
	* @param @param key
	* @param @param start
	* @param @param end
	* @param @return    
	* @return Set<String>    
	* @throws
	 */
	public Set<String> getZsetZrevrange(String key,long start , long end) throws Exception{
		Set<String> value = null;
		Jedis jedis = null;
		try {
			jedis = getResource();
			if (jedis.exists(key)) {
				value = jedis.zrevrange (key, start,end);
				logger.debug("getZset {} = {}", key, value);
			}
		} catch (Exception e) {
			logger.warn("getZset {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return value;
	}
	

	
	/**
	 * 设置Set缓存
	 * @param key 键
	 * @param value 值
	 * @param cacheSeconds 超时时间，0为不超时
	 * @return
	 */
	public long setZset(String key, Map<String, Double> scoreMembers, int cacheSeconds) throws Exception{
		long result = 0;
		Jedis jedis = null;
		boolean isExists = false;
		try {
			jedis = getResource();
			if (jedis.exists(key)) {
				isExists = true;
			}
			jedis.zadd(key, scoreMembers);
			if (!isExists && cacheSeconds != 0) {
				jedis.expire(key, cacheSeconds);
			}
			logger.debug("setZset {} = {}", key, scoreMembers);
		} catch (Exception e) {
			logger.warn("setZset {} = {}", key, scoreMembers, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}


	
	/**
	 * 获取Map缓存
	 * @param key 键
	 * @return 值
	 */
	public Map<String, String> getMap(String key) throws Exception{
		Map<String, String> value = null;
		Jedis jedis = null;
		try {
			jedis = getResource();
			if (jedis.exists(key)) {
				value = jedis.hgetAll(key);
				logger.debug("getMap {} = {}", key, value);
			}
		} catch (Exception e) {
			logger.warn("getMap {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return value;
	}
	
	/**
	 * 获取Map缓存里某个key的值
	* @Title: getMapKeyVal
	* @Description: TODO
	* @param @param key
	* @param @param mapKey
	* @param @return    
	* @return String    
	* @throws
	 */
	public String  getMapKeyVal(String key,String mapKey) throws Exception{
		String  value = null;
		Jedis jedis = null;
		try {
			jedis = getResource();
			if (jedis.exists(key)) {
				if(jedis.hexists(key, mapKey)){
					value = jedis.hget(key, mapKey);
					logger.debug("getMap {}  {} = {}", key, mapKey,value);
				}
				
			}
		} catch (Exception e) {
			logger.warn("getMap {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return value;
	}
	
	/**
	 * 获取Map缓存
	 * @param key 键
	 * @return 值
	 */
	public Map<String, Object> getObjectMap(String key) throws Exception{
		Map<String, Object> value = null;
		Jedis jedis = null;
		try {
			jedis = getResource();
			if (jedis.exists(getBytesKey(key))) {
				value = Maps.newHashMap();
				Map<byte[], byte[]> map = jedis.hgetAll(getBytesKey(key));
				for (Map.Entry<byte[], byte[]> e : map.entrySet()){
					value.put(StringUtil.toString(e.getKey()), toObject(e.getValue()));
				}
				logger.debug("getObjectMap {} = {}", key, value);
			}
		} catch (Exception e) {
			logger.warn("getObjectMap {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return value;
	}
	
	/**
	 * 设置Map缓存 
	 * @param key 键
	 * @param value 值
	 * @param cacheSeconds 超时时间，0为不超时
	 * @return
	 */
	public String setMap(String key, Map<String, String> value, int cacheSeconds)throws Exception {
		String result = null;
		Jedis jedis = null;
		boolean isExists = false;
		try {
			jedis = getResource();
			if (jedis.exists(key)) {
				isExists = true;
			}
			result = jedis.hmset(key, value);
			if (!isExists && cacheSeconds != 0) {
				jedis.expire(key, cacheSeconds);
			}
			logger.debug("setMap {} = {}", key, value);
		} catch (Exception e) {
			logger.warn("setMap {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}
	
	/**
	 * 设置Map缓存 
	 * @param key 键
	 * @param value 值
	 * @param cacheSeconds 超时时间，0为不超时
	 * @return
	 */
	public String setObjectMap(String key, Map<String, Object> value, int cacheSeconds) throws Exception{
		String result = null;
		Jedis jedis = null;
		boolean isExists = false;
		try {
			jedis = getResource();
			if (jedis.exists(getBytesKey(key))) {
				isExists = true;
			}
			Map<byte[], byte[]> map = Maps.newHashMap();
			for (Map.Entry<String, Object> e : value.entrySet()){
				map.put(getBytesKey(e.getKey()), toBytes(e.getValue()));
			}
			result = jedis.hmset(getBytesKey(key), (Map<byte[], byte[]>)map);
			if (!isExists && cacheSeconds != 0) {
				jedis.expire(key, cacheSeconds);
			}
			logger.debug("setObjectMap {} = {}", key, value);
		} catch (Exception e) {
			logger.warn("setObjectMap {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}
	
	/**
	 * 向Map缓存中添加值
	 * @param key 键
	 * @param value 值
	 * @return
	 */
	public String mapPut(String key, Map<String, String> value) throws Exception{
		String result = null;
		Jedis jedis = null;
		try {
			jedis = getResource();
			result = jedis.hmset(key, value);
			logger.debug("mapPut {} = {}", key, value);
		} catch (Exception e) {
			logger.warn("mapPut {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}
	
	/**
	 * 向Map缓存中添加值
	 * @param key 键
	 * @param value 值
	 * @return
	 */
	public String mapObjectPut(String key, Map<String, Object> value)throws Exception {
		String result = null;
		Jedis jedis = null;
		try {
			jedis = getResource();
			Map<byte[], byte[]> map = Maps.newHashMap();
			for (Map.Entry<String, Object> e : value.entrySet()){
				map.put(getBytesKey(e.getKey()), toBytes(e.getValue()));
			}
			result = jedis.hmset(getBytesKey(key), (Map<byte[], byte[]>)map);
			logger.debug("mapObjectPut {} = {}", key, value);
		} catch (Exception e) {
			logger.warn("mapObjectPut {} = {}", key, value, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}
	
	/**
	 * 移除Map缓存中的值
	 * @param key 键
	 * @param value 值
	 * @return
	 */
	public long mapRemove(String key, String mapKey)throws Exception {
		long result = 0;
		Jedis jedis = null;
		try {
			jedis = getResource();
			result = jedis.hdel(key, mapKey);
			logger.debug("mapRemove {}  {}", key, mapKey);
		} catch (Exception e) {
			logger.warn("mapRemove {}  {}", key, mapKey, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}
	
	/**
	 * 移除Map缓存中的值
	 * @param key 键
	 * @param value 值
	 * @return
	 */
	public long mapObjectRemove(String key, String mapKey) throws Exception {
		long result = 0;
		Jedis jedis = null;
		try {
			jedis = getResource();
			result = jedis.hdel(getBytesKey(key), getBytesKey(mapKey));
			logger.debug("mapObjectRemove {}  {}", key, mapKey);
		} catch (Exception e) {
			logger.warn("mapObjectRemove {}  {}", key, mapKey, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}
	
	/**
	 * 判断Map缓存中的Key是否存在
	 * @param key 键
	 * @param value 值
	 * @return
	 */
	public boolean mapExists(String key, String mapKey)throws Exception  {
		boolean result = false;
		Jedis jedis = null;
		try {
			jedis = getResource();
			result = jedis.hexists(key, mapKey);
			logger.debug("mapExists {}  {}", key, mapKey);
		} catch (Exception e) {
			logger.warn("mapExists {}  {}", key, mapKey, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}
	
	/**
	 * 判断Map缓存中的Key是否存在
	 * @param key 键
	 * @param value 值
	 * @return
	 */
	public boolean mapObjectExists(String key, String mapKey)throws Exception {
		boolean result = false;
		Jedis jedis = null;
		try {
			jedis = getResource();
			result = jedis.hexists(getBytesKey(key), getBytesKey(mapKey));
			logger.debug("mapObjectExists {}  {}", key, mapKey);
		} catch (Exception e) {
			logger.warn("mapObjectExists {}  {}", key, mapKey, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}
	
	/**
	 * 删除缓存
	 * @param key 键
	 * @return
	 */
	public long del(String key) throws Exception{
		long result = 0;
		Jedis jedis = null;
		try {
			jedis = getResource();
			if (jedis.exists(key)){
				result = jedis.del(key);
				logger.debug("del {}", key);
			}else{
				logger.debug("del {} not exists", key);
			}
		} catch (Exception e) {
			logger.warn("del {}", key, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}

	/**
	 * 删除缓存
	 * @param key 键
	 * @return
	 */
	public long delObject(String key)throws Exception {
		long result = 0;
		Jedis jedis = null;
		try {
			jedis = getResource();
			if (jedis.exists(getBytesKey(key))){
				result = jedis.del(getBytesKey(key));
				logger.debug("delObject {}", key);
			}else{
				logger.debug("delObject {} not exists", key);
			}
		} catch (Exception e) {
			logger.warn("delObject {}", key, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}
	
	/**
	 * 缓存是否存在
	 * @param key 键
	 * @return
	 */
	public boolean exists(String key)throws Exception {
		boolean result = false;
		Jedis jedis = null;
		try {
			jedis = getResource();
			result = jedis.exists(key);
			logger.debug("exists {}", key);
		} catch (Exception e) {
			logger.warn("exists {}", key, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}
	
	/**
	 * 缓存是否存在
	 * @param key 键
	 * @return
	 */
	public boolean existsObject(String key)throws Exception  {
		boolean result = false;
		Jedis jedis = null;
		try {
			jedis = getResource();
			result = jedis.exists(getBytesKey(key));
			logger.debug("existsObject {}", key);
		} catch (Exception e) {
			logger.warn("existsObject {}", key, e);
			throw new Exception(e);
		} finally {
			returnResource(jedis);
		}
		return result;
	}

	/**
	 * 获取资源
	 * @return
	 * @throws JedisException
	 */
	public Jedis getResource()  throws Exception {
		Jedis jedis = null;
		try {
			jedis = jedisPool.getResource();
		} catch (JedisException e) {
			logger.warn("getResource.", e);
			returnBrokenResource(jedis);
			throw new Exception(e);
		}
		return jedis;
	}

	/**
	 * 归还资源
	 * @param jedis
	 * @param isBroken
	 */
	public void returnBrokenResource(Jedis jedis) {
		if (jedis != null) {
			/*jedisPool.returnBrokenResource(jedis);*/
			jedis.close();
		}
	}
	
	/**
	 * 释放资源
	 * @param jedis
	 * @param isBroken
	 */
	public void returnResource(Jedis jedis) {
		if (jedis != null) {
			/*jedisPool.returnResource(jedis);*/
			jedis.close();
		}
	}

	/**
	 * 获取byte[]类型Key
	 * @param key
	 * @return
	 */
	public byte[] getBytesKey(Object object){
		if(object instanceof String){
    		return StringUtil.getBytes((String)object);
    	}else{
    		return ObjectUtils.serialize(object);
    	}
	}
	
	/**
	 * Object转换byte[]类型
	 * @param key
	 * @return
	 */
	public byte[] toBytes(Object object){
    	return ObjectUtils.serialize(object);
	}

	/**
	 * byte[]型转换Object
	 * @param key
	 * @return
	 */
	public Object toObject(byte[] bytes){
		return ObjectUtils.unserialize(bytes);
	}

}
```
Object序列化和反序列化工具类
```Java
package test.util;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

public class ObjectUtils {

	
	/**
	 * 序列化对象
	 * @param object
	 * @return
	 */
	public static byte[] serialize(Object object) {
		 byte[]  bytes = null;
		ObjectOutputStream oos = null;
		ByteArrayOutputStream baos = null;
		try {
			if (object != null){
				baos = new ByteArrayOutputStream();
				oos = new ObjectOutputStream(baos);
				oos.writeObject(object);
				bytes = baos.toByteArray();
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally{
			try {
				if(null != oos) {
					oos.close();
				}
				if(null != baos) {
					baos.close();
				}
				
			} catch (Exception e2) {
				e2.printStackTrace();
			}
			
		}
		return bytes;
	}

	/**
	 * 反序列化对象
	 * @param bytes
	 * @return
	 */
	public static Object unserialize(byte[] bytes) {
		Object object = null;
		ByteArrayInputStream bais = null;
		ObjectInputStream ois = null;
		try {
			if (bytes != null && bytes.length > 0){
				bais = new ByteArrayInputStream(bytes);
				ois = new ObjectInputStream(bais);
				object = ois.readObject();
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally{
			try {
				if(null != ois) {
					ois.close();
				}
				
				if(null != bais) {
					bais.close();
				}
			} catch (Exception e2) {
				e2.printStackTrace();
			}
		}
		return  object ;
	}
}
```
String工具类
```Java
package test.util;


public class StringUtil {
	
	private static final String CHARSET_NAME = "UTF-8";

	/**
 	 * 
 	* @Title: toString
 	* @Description: 把字节流转为字符串
 	* @param @param content
 	* @param @param charset
 	* @param @return    
 	* @return String    
 	* @throws
 	 */
    public static String toString(byte[]content, String charset){
    	String result = "";
    	try {
        	if(null != content && content.length > 0){
        		result =  new String(content, charset);
        	}
		} catch (Exception e) {
			e.printStackTrace();
		}
    	

    	return result;
    }
 
	/**
	 * 转换为字节数组
	 * @param str
	 * @return
	 */
	public static String toString(byte[] bytes){
		return  toString(bytes, "UTF-8");
	}
	
	/**
	 * 转换为字节数组
	 * @param str
	 * @return
	 */
	public static byte[] getBytes(String str){
		if (str != null){
			try {
				return str.getBytes(CHARSET_NAME);
			} catch (Exception e) {
				return null;
			}
		}else{
			return null;
		}
	}
	
}
```
spring配置的片段
```XML
<!-- 连接池配置 -->
<bean id="jedisPool" class="redis.clients.jedis.JedisPool" destroy-method="destroy">
	<constructor-arg index="0" ref="jedisPoolConfig" />
	<constructor-arg index="1" value="${redis_host}" type="java.lang.String"  />
	<constructor-arg index="2" value="${redis_port}" type="int" />
	<constructor-arg index="3" value="360000" type="int" /> <!-- 超时时间 单位ms -->
	<constructor-arg index="4" value="${redis_password}" type="java.lang.String" />
	<constructor-arg index="5" value="${redis_database}" type="int" />
</bean>
<!-- 注入 -->
<bean id="jedisUtils" class="test.util.JedisUtils">
	<property name="jedisPool" ref="jedisPool"/>
</bean>
```