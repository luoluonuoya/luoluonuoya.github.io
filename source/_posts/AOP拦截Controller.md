---
title: AOP拦截Controller处理@RequestBody注入参数问题
date: 2017-05-03 17:24:26
categories: [Java]
tags: [Java]
---
**问题：** 使用springMVC，一些请求前的校验，都可以在拦截器或者过滤器里做处理，处理完后到Controller层再根据喜好做相关操作，但是遇到content-type是application/json等放在body体里的就尴尬了，读完body，在Controller就无法再获取一次了，或许你可以把request的inputStream读出来缓存两份。但这要动根基了，不划算。另外，如果需要改变某个参数再由@RequestBody注入，又是一个尴尬的问题，可以把<mvc:message-converters>配置的Json处理写成自己的逻辑。但所有请求都经过这个也显得累赘。

**键值形式的传参处理**
```Java
package com.sq580.mall.order.web;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class XxxInterceptor implements Filter {

    @Override
    public void destroy() {}

    @Override
    public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2) throws IOException,
            ServletException {
        HttpServletRequest req = (HttpServletRequest) arg0;
        ServletRequest requestWrapper = new ParameterRequestWrapper(req);
        String body = HttpHelper.getBodyString(requestWrapper);
        System.out.println(body);
        // 自己拦截的逻辑
        arg2.doFilter(requestWrapper, (HttpServletResponse) arg1);
    }

    @Override
    public void init(FilterConfig arg0) throws ServletException {}
}
```
```Java
package com.xxx;

import java.io.BufferedReader;
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.nio.charset.Charset;
import java.util.Enumeration;

import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;

public class ParameterRequestWrapper extends HttpServletRequestWrapper {
	private final byte[] body;

    public ParameterRequestWrapper(HttpServletRequest request) throws IOException {
        super(request);
        body = HttpHelper.getBodyString(request).getBytes(Charset.forName("UTF-8"));
    }

    @Override
    public BufferedReader getReader() throws IOException {
        return new BufferedReader(new InputStreamReader(getInputStream()));
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        final ByteArrayInputStream bais = new ByteArrayInputStream(body);
        return new ServletInputStream() {
            @Override
            public int read() throws IOException {
                return bais.read();
            }
        };
    }

    @Override
    public String getHeader(String name) {
        return super.getHeader(name);
    }

    @Override
    public Enumeration<String> getHeaderNames() {
        return super.getHeaderNames();
    }

    @Override
    public Enumeration<String> getHeaders(String name) {
        return super.getHeaders(name);
    }
}
```
HttpHelper工具类
```Java
package com.xxx;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.nio.charset.Charset;

import javax.servlet.ServletRequest;

public class HttpHelper {
    /**
     * 获取请求Body
     * @param request
     * @return
     */
    public static String getBodyString(ServletRequest request) {
        StringBuilder sb = new StringBuilder();
        InputStream inputStream = null;
        BufferedReader reader = null;
        try {
            inputStream = request.getInputStream();
            reader = new BufferedReader(new InputStreamReader(inputStream, Charset.forName("UTF-8")));
            String line = "";
            while ((line = reader.readLine()) != null) {
                sb.append(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return sb.toString();
    }

}
```
web.xml
```XML
<filter>
	<filter-name>requestFilter</filter-name>
	<filter-class>com.xxx.XxxInterceptor</filter-class>
</filter>
<filter-mapping>
	<filter-name>requestFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

**body传参处理** 使用aop来切Controller中的方法，这时参数已经注入了，不存在重复读request流的问题
**注意：** 这里开启aop的配置需要写在springmvc中，不能写在spring的applicationContext.xml中，两者有所区别
```XML
<aop:aspectj-autoproxy proxy-target-class="true" />
<context:component-scan base-package="com.xxx" />
```
切面的使用，注意要扫包
```Java
@Aspect
@Component
public class PariseAspect {

	// 第一个*表示所有返回类型，第二段是com.xxx包下任何包的所有后缀为Controller的类的所有方法，(..)表示方法所带任何参数
	@Pointcut("execution(* com.xxx.*.*Controller.*(..))")    
    public  void controllerAspect() {}

	// @Before是指在方法执行前执行，@After指在方法执行后执行，这两个都好理解，@Around并不是在方法执行过程循环执行，它表示的是
	/* {
		// before
		// 你可以让它继续执行，也可以返回，是这么一种环绕意义，有点类似过滤器
		// after
	}*/
	@Around("controllerAspect()")
	public Object checkToken(ProceedingJoinPoint point) {
		// 获取所有参数，自己断点可以理解清楚点，一般第一个为@RequestBody所注入的对象，第二个为BindingResult，假设对象中有个userId，此处改变了对象的值后，通知继续执行
		Object[] args = point.getArgs();
		Class clazz = args[0].getClass();
		Method m = clazz.getDeclaredMethod("setUserId", Long.class);
		if (null != m) {
			m.invoke(arg, 1L);
			args[1] = arg;
		} else {
			// 返回错误
			return "error.jsp";
		}
		return point.proceed(args);
	}

}
```