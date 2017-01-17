---
title: Controller异常处理
date: 2016-08-07 10:29:33
categories: [Java]
tags: [Java]
---
```
一般来说，项目中的错误最好手动处理，避免尴尬直接抛错给请求者，但是在一些特殊情况中，比如我们在用
SpringMvc的@RequestBody注入对象的时候可能很方便，但是在联调阶段也会出现很多尴尬的问题，
比如我们定义了一个注入的model中的price是一个decimal类型，那么按理说需要前端提供一个类似0.00的值上来，
但是像网页端，若是用input标签来实现且price非必填，一般就直接传了个""上来，这是注入的时候就会出现
类型转换失败的情况。
```
下面回到正题，使用@ExceptionHandler，当Controller中任何一个方法发生异常，定义一个默认的方法拦截，一般来说最好定义在父类中，然后所有Controller都继承该类
代码示例：
```Java
import com.alibaba.fastjson.JSONException;

@Controller
public class AccessController {

    /**
	 * 异常控制
	* @Title: handleOtherExceptions
	* @Description: RequestBody用fastjson解析出错的统一处理
	* @param @param ex
	* @param @return    
	* @return ResponseEntity<Object>    
	* @throws
	 */
	@ExceptionHandler(JSONException.class)
	public ResponseEntity<Object> handleOtherExceptions(JSONException e) {
		Result result = new Result();
		
		String eMsg = e.getMessage();
		if (eMsg.contains("not match : - ")) {
			result.setFailResultMsg("json数据格式不正确，没有匹配的 '}' 或 '\"'");
		} else if (eMsg.contains("unclosed string")) {
			result.setFailResultMsg("json数据格式不正确，没有闭合的 '\"'");
		} else if(eMsg.contains("syntax error, ")) {
			result.setFailResultMsg("json数据格式不正确： ");
		} else if (eMsg.contains("can not cast to")) {
			String[] msg = eMsg.split(" ");
			result.setFailResultMsg("转换类型失败：值 '" + msg[7] + "' 不能转成 "
									+ msg[4].replace(",", "") + " 类型");
		} else {
			result.setFailResultMsg("未知异常： " + eMsg);
		}
		
		return new ResponseEntity<Object>(result, HttpStatus.OK);
	}
}
```
还有一种方式，使用 @ControllerAdvice只要把这个类放在项目中，Spring能扫描到的地方，就可以实现全局异常的回调。
代码示例：
```Java
@ControllerAdvice
public class SpringExceptionHandler {
		/** 
		 * 全局处理Exception 
         * 错误的情况下返回500 
         * @param e
         * @param req
         * @return 
         */
        @ExceptionHandler(value = {Exception.class})
        public ResponseEntity<Object> handleOtherExceptions(final Exception e, final WebRequest req) {
            TResult tResult = new TResult();
            tResult.setStatus(CodeType.V_500);
            tResult.setErrorMessage(ex.getMessage());
            return new ResponseEntity<Object>(tResult,HttpStatus.OK);
        }
    }
```