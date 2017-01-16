---
title: SpringMvc接收参数
date: 2016-07-17 14:27:48
categories: [Java]
tags: [Java,SpringMvc]
---
#### SpringMvc接收对象的方式
##### 1、最普通的我们可以用HttpServletRequest对象来接收，但下面的方式我们只能用来接收键值对，对application/json之类的类型就没有用了，需要读取request的流再解析出参数
request.getParameter(name);
　
##### 2、但既然用了SpringMvc我们可以用注解的方式来获取各种参数
**1）@PathVariable **
当使用@RequestMapping URI template 样式映射时， 即 someUrl/{paramId}, 这时的paramId可通过 @Pathvariable注解绑定它传过来的值到方法的参数上。
示例代码：
```Java
@Controller
@RequestMapping("/user/{operation}")
public class UserController {
  
	@RequestMapping("/{userId}")
	public void findPet(@PathVariable String operation, @PathVariable String userId, Model model) {
	  // TODO 
	}
}
```
**2）@RequestHeader **
@RequestHeader 注解，可以把Request请求header部分的值绑定到方法的参数上。
常见的header：
```Plain
Accept:					image/webp,image/*,*/*;q=0.8
Accept-Encoding:		gzip, deflate, sdch
Accept-Language:		zh-CN,zh;q=0.8
Host:					localhost:8080
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
```
示例代码：
```Java
@Controller
@RequestMapping("/user")
public class UserController {
  
	@RequestMapping("/getlist")  
	public void findPet(
		@RequestHeader("Accept-Encoding") String encoding, 
		@RequestHeader("Keep-Alive") long keepAlive) {
	  // TODO 
	}
}
```
**3）@CookieValue **
@CookieValue 可以把Request header中关于cookie的值绑定到方法的参数上。
例如有如下Cookie值：
```Plain
JSESSIONID=070c17a0028d1dcc66eba40e7781d177
```
示例代码：
```Java
@Controller
@RequestMapping("/user")
public class UserController {
  
	@RequestMapping("/login")  
	public void findPet(@CookieValue("JSESSIONID") String cookie) {
	  // TODO 
	}
}
```
**4）@RequestParam**
@RequestParam等同于request.getParameter();
@RequestParam用来处理Content-Type: 为 application/x-www-form-urlencoded编码的内容，提交方式为GET、POST；
@RequestParam有两个属性：
　value：用来指定要传入值的name
　required：用来指定参数是否必须
示例代码：
```Java
@Controller
@RequestMapping("/user")
public class UserController {
  
	@RequestMapping("/login", method = RequestMethod.GET)
	public void findPet(@RequestParam("userId") String userId) {
	  // TODO 
	}
}
```
**5）@RequestBody**
像ajax原生的请求就是application/json类型，此时用@RequestBody就十分合适
@RequestBody常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等；
**但是注意springmvc4.2版本才能直接转对象，4.2之前只能接收String**
示例代码：
```Java
@Controller
@RequestMapping("/user")
public class UserController {
  
	// 4.2之前接收String
	@RequestMapping("/login", method = RequestMethod.POST)
	public void findPet(@RequestBody String body) {
		User user = (User) JSON.parse(body);
	}
	
	// 4.2之后接收bean
		@RequestMapping("/login", method = RequestMethod.POST)
		public void findPet(@RequestBody User user) {
			return user.getName(); 
		}
	}
```
**6）@SessionAttributes**
@SessionAttribute用来绑定HttpSession中的attribute对象的值，便于在方法中的参数里使用。
示例代码：
```Java
@Controller  
@RequestMapping("/user")
@SessionAttributes("userId")
public class UserController {
  
	@RequestMapping("/login", method = RequestMethod.POST)
	public void findPet() {
	  // TODO 
	}
}
```
**7）@ModelAttribute**
@ModelAttribute可以用在方法上也可以用在参数上，@ModelAttribute只能获取键值对，即接收的Content-Type是from-data或者application/x-www-form等类型
首先是用在方法上时，相当于为request对象的model里put("account", Account);
如此在使用@RequestParam时就不需要带上参数名了
示例代码：
```Java
@Controller  
@RequestMapping("/user")
public class UserController {
  
	@RequestMapping("/login", method = RequestMethod.POST)
	@ModelAttribute
	public void findPet(@RequestParam String userId) {
	  return accountManager.findAccount(userId);
	}
}
```
如果是在参数上时，一般来说，我们需要的是request中的参数，所以这里说明一下，@ModelAttribute绑定参数的来源
　A） 首先是 @SessionAttributes 上绑定的attribute 对象
　B） 没有就查询@ModelAttribute 用于方法上时指定的model对象
　C） 当两种情况都没有时，new一个需要绑定的bean对象，然后把request中按名称对应的方式把值绑定到bean中。
示例代码：
```Java
@Controller  
@RequestMapping("/user")
public class UserController {
  
	@RequestMapping("/login", method = RequestMethod.POST)
	public void findPet(@ModelAttribute User user) {
	  // TODO 
	}
}
```