---
title: springmvc实现restful api版本控制并兼容swagger
date: 2017-11-10 14:00:26
categories: [Java]
tags: [Java,springboot,springMvc]
---
开发的时候经常会出现接口的变更，而接口的变更又常需兼容老的客户端版本，此时便需要做接口的版本控制，比较好的方案是在header带上version参数，但是不排除有些在url上做控制的，类似/v1/xxx/xxx，/v2_0/xxx/xxx，此文为第二种，使用如下：
```Java
@RestController
@Api(value = "测试", tags = {"测试"})
@RequestMapping("/{version}/test/")
public class TestController {

    @Autowired
    private TestService testService;
    
    @ApiOperation(value = "测试接口", notes = "测试")
    @PostMapping(value = "test")
    @ApiVersion("1_0")
    public Object region(@RequestBody TestReq model) {
        return testService.test(model);
    }
}
```
首先是@ApiVersion的使用，定义如下
```Java
import org.springframework.web.bind.annotation.Mapping;

import java.lang.annotation.*;

/**
 * ClassName: ApiVersion<br/>
 * Function: url版本注解<br/>
 * Reason: value书写格式 n_n，匹配规则为小于等于n_n最近的一个版本，示例如下：
 * 现有接口 v1_0,v1_1,v2_0,v2_3
 * 访问 v1_1/xxx   执行： v1_1
 * 访问 v1_2/xxx   执行： v1_1
 * 访问 v2_1/xxx   执行： v2_0
 * 访问 v4_0/xxx   执行： v2_3
 * <br/>
 * Date: 2017-10-31 14:33<br/>
 *
 * @author zoro
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface ApiVersion {
    // 版本号
    String value();
}
```
接下来是自定义条件选择器，当请求进来可以正确匹对需要调用的方法
```Java
import org.springframework.web.servlet.mvc.condition.RequestCondition;

import javax.servlet.http.HttpServletRequest;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * ClassName: ApiVesrsionCondition<br/>
 * Function: API版本条件筛选器<br/>
 * Reason: TODO ADD REASON(可选)<br/>
 * Date: 2017-10-31 14:35<br/>
 *
 * @author zoro
 */
public class ApiVesrsionCondition implements RequestCondition<ApiVesrsionCondition> {

    private String apiVersion;

    public ApiVesrsionCondition(String apiVersion){
        this.apiVersion = apiVersion;
    }

    public ApiVesrsionCondition combine(ApiVesrsionCondition other) {
        // 采用最后定义优先原则，则方法上的定义覆盖类上面的定义
        return new ApiVesrsionCondition(other.getApiVersion());
    }

    public ApiVesrsionCondition getMatchingCondition(HttpServletRequest request) {
        if (compare(getPathInfo(request)) >= 0) {
            return this;
        }
        return null;
    }

    public int compareTo(ApiVesrsionCondition other, HttpServletRequest request) {
        // 优先匹配最新的版本号
        return compare("v" + other.getApiVersion() + "/");
    }

    public String getApiVersion() {
        return apiVersion;
    }

    private String getPathInfo(HttpServletRequest request) {
        String uri = request.getRequestURI();
        String contextPath = request.getContextPath();
        if (contextPath != null && contextPath.length() > 0) {
            uri = uri.substring(contextPath.length());
        }
        return uri;
    }

	/**
     * Description: 这里是版本的控制规则，不是很优雅，时间有限，写死了只识别vn_n的格式.<br/>
     * Date: 2017/11/10.<br/>
     * @param version
     * @return 
     * @throws 
     */
    private int compare(String version) {
        // 路径中版本的前缀， 这里用 /v[0-9]_[0-9]/的形式
        Pattern req_patten = Pattern.compile("v(\\d+)_(\\d+)/");
        // api版本的前缀， 这里用 [0-9]_[0-9]的形式
        Pattern api_patten = Pattern.compile("(\\d+)_(\\d+)");
        Matcher req_m = req_patten.matcher(version);
        Matcher api_m = api_patten.matcher(this.apiVersion);
        if (req_m.find() && api_m.find()) {
            Integer first_version = Integer.valueOf(req_m.group(1));
            Integer first_apiVersion = Integer.valueOf(api_m.group(1));
            // 如果请求的版本号大于配置版本号， 则满足
            if(first_version > first_apiVersion) {
                return 1;
            } else if (first_version == first_apiVersion) {
                Integer last_version = Integer.valueOf(req_m.group(2));
                Integer last_apiVersion = Integer.valueOf(api_m.group(2));
                if(last_version > last_apiVersion) {
                    return 1;
                } else if (last_version == last_apiVersion) {
                    return 0;
                }
            }
        }
        return -1;
    }

}
```
我们知道springmvc默认是用RequestMappingInfoHandlerMapping根据RequestMappingInfo来匹配条件的，所以我们自定义一个Handler来修改RequestMappingInfoHandlerMapping原有的匹配规则
```Java
import org.springframework.core.annotation.AnnotationUtils;
import org.springframework.web.servlet.mvc.condition.RequestCondition;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

import java.lang.reflect.Method;

/**
 * ClassName: CustomRequestMappingHandlerMapping<br/>
 * Function: TODO ADD FUNCTION(可选)<br/>
 * Reason: TODO ADD REASON(可选)<br/>
 * Date: 2017-10-31 15:13<br/>
 *
 * @author zoro
 */
public class CustomRequestMappingHandlerMapping extends RequestMappingHandlerMapping {

    @Override
    protected RequestCondition<ApiVesrsionCondition> getCustomTypeCondition(Class<?> handlerType) {
        ApiVersion apiVersion = AnnotationUtils.findAnnotation(handlerType, ApiVersion.class);
        return createCondition(apiVersion);
    }

    @Override
    protected RequestCondition<ApiVesrsionCondition> getCustomMethodCondition(Method method) {
        ApiVersion apiVersion = AnnotationUtils.findAnnotation(method, ApiVersion.class);
        return createCondition(apiVersion);
    }

    private RequestCondition<ApiVesrsionCondition> createCondition(ApiVersion apiVersion) {
        return apiVersion == null ? null : new ApiVesrsionCondition(apiVersion.value());
    }

}
```
最后，只需要重新装配一下自定义的HandlerMapping即可
```Java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

/**
 * ClassName: WebConfig<br/>
 * Function: TODO ADD FUNCTION(可选)<br/>
 * Reason: TODO ADD REASON(可选)<br/>
 * Date: 2017-10-31 15:15<br/>
 *
 * @author zoro
 */
@Configuration
public class WebConfig extends WebMvcConfigurationSupport {

    @Override
    @Bean
    public RequestMappingHandlerMapping requestMappingHandlerMapping() {
        RequestMappingHandlerMapping handlerMapping = new CustomRequestMappingHandlerMapping();
        handlerMapping.setOrder(0);
        handlerMapping.setInterceptors(getInterceptors());
        return handlerMapping;
    }

}
```
至此，便可以正常的使用了，但是如果项目引入了swagger的，会发现一个问题，项目启动后，swagger页面无法打开，这是因为继承WebMvcConfigurationSupport后，静态文件的映射出现了问题，需要重新指定一下，WebConfig修改如下：
```Java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

/**
 * ClassName: WebConfig<br/>
 * Function: TODO ADD FUNCTION(可选)<br/>
 * Reason: TODO ADD REASON(可选)<br/>
 * Date: 2017-10-31 15:15<br/>
 *
 * @author zoro
 */
@Configuration
public class WebConfig extends WebMvcConfigurationSupport {

    @Override
    @Bean
    public RequestMappingHandlerMapping requestMappingHandlerMapping() {
        RequestMappingHandlerMapping handlerMapping = new CustomRequestMappingHandlerMapping();
        handlerMapping.setOrder(0);
        handlerMapping.setInterceptors(getInterceptors());
        return handlerMapping;
    }

    /**
     * 发现如果继承了WebMvcConfigurationSupport，则在yml中配置的相关内容会失效。
     * 需要重新指定静态资源
     * @param registry
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations("classpath:/static/");
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
        super.addResourceHandlers(registry);
    }

    /**
     * 配置servlet处理
     */
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

}
```
此时，swagger也能正常打开了，却发现swagger生成的api文档是{version}/xxx/xxx的，这就非常恶心了，我们知道，springmvc的映射信息是放在RequestMappingInfo里的，先来看一段RequestMappingHandlerMapping的源码
```Java
public class RequestMappingHandlerMapping extends RequestMappingInfoHandlerMapping implements MatchableHandlerMapping, EmbeddedValueResolverAware {

    .......上面的代码省略......

    protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
        RequestMappingInfo info = this.createRequestMappingInfo(method);
        if (info != null) {
            RequestMappingInfo typeInfo = this.createRequestMappingInfo(handlerType);
            if (typeInfo != null) {
                info = typeInfo.combine(info);
            }
        }

        return info;
    }

    private RequestMappingInfo createRequestMappingInfo(AnnotatedElement element) {
        RequestMapping requestMapping = (RequestMapping)AnnotatedElementUtils.findMergedAnnotation(element, RequestMapping.class);
        RequestCondition<?> condition = element instanceof Class ? this.getCustomTypeCondition((Class)element) : this.getCustomMethodCondition((Method)element);
        return requestMapping != null ? this.createRequestMappingInfo(requestMapping, condition) : null;
    }

    protected RequestCondition<?> getCustomTypeCondition(Class<?> handlerType) {
        return null;
    }

    protected RequestCondition<?> getCustomMethodCondition(Method method) {
        return null;
    }

    protected RequestMappingInfo createRequestMappingInfo(RequestMapping requestMapping, RequestCondition<?> customCondition) {
        return RequestMappingInfo.paths(this.resolveEmbeddedValuesInPatterns(requestMapping.path())).methods(requestMapping.method()).params(requestMapping.params()).headers(requestMapping.headers()).consumes(requestMapping.consumes()).produces(requestMapping.produces()).mappingName(requestMapping.name()).customCondition(customCondition).options(this.config).build();
    }

    .......下面的代码省略......

}
```
从getMappingForMethod方法可以知道，springmvc是先读取方法上的@RequestMapping上的value，再读类上@RequestMapping上的value，然后两个值拼接在一起，理论上是可以在生成RequestMappingInfo后通过反射修改RequestMappingInfo里的值来达到目的的，这里想折腾一下，从createRequestMappingInfo的时候入手，通过改变注解上的value来达到效果，修改CustomRequestMappingHandlerMapping类如下
```
import org.springframework.core.annotation.AnnotatedElementUtils;
import org.springframework.core.annotation.AnnotationUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.mvc.condition.RequestCondition;
import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

import java.lang.reflect.*;
import java.util.Map;

/**
 * ClassName: CustomRequestMappingHandlerMapping<br/>
 * Function: TODO ADD FUNCTION(可选)<br/>
 * Reason: TODO ADD REASON(可选)<br/>
 * Date: 2017-10-31 15:13<br/>
 *
 * @author zoro
 */
public class CustomRequestMappingHandlerMapping extends RequestMappingHandlerMapping {

    @Override
    protected RequestCondition<ApiVesrsionCondition> getCustomTypeCondition(Class<?> handlerType) {
        ApiVersion apiVersion = AnnotationUtils.findAnnotation(handlerType, ApiVersion.class);
        return createCondition(apiVersion);
    }

    @Override
    protected RequestCondition<ApiVesrsionCondition> getCustomMethodCondition(Method method) {
        ApiVersion apiVersion = AnnotationUtils.findAnnotation(method, ApiVersion.class);
        return createCondition(apiVersion);
    }

    private RequestCondition<ApiVesrsionCondition> createCondition(ApiVersion apiVersion) {
        return apiVersion == null ? null : new ApiVesrsionCondition(apiVersion.value());
    }

    @Override
    protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
        RequestMappingInfo info = this.createRequestMappingInfo(method, null);
        if (info != null) {
            ApiVersion apiVersion = (ApiVersion) AnnotatedElementUtils.findMergedAnnotation(method, ApiVersion.class);
            RequestMappingInfo typeInfo = this.createRequestMappingInfo(handlerType, apiVersion);
            if (typeInfo != null) {
                info = typeInfo.combine(info);
            }
        }
        return info;
    }

    private RequestMappingInfo createRequestMappingInfo(AnnotatedElement element, ApiVersion apiVersion) {
        RequestMapping requestMapping = (RequestMapping) AnnotatedElementUtils.findMergedAnnotation(element, RequestMapping.class);
        RequestCondition<?> condition = element instanceof Class ? this.getCustomTypeCondition((Class)element) : this.getCustomMethodCondition((Method)element);
        if (element instanceof Class && null != apiVersion) {
            try {
                // 动态修改RequestMapping注解的属性
                InvocationHandler invocationHandler = Proxy.getInvocationHandler(requestMapping);
                Field field = invocationHandler.getClass().getDeclaredField("valueCache");
                // SynthesizedAnnotationInvocationHandler的valueCache是私有变量，需要打开权限
                field.setAccessible(true);
                Map map = (Map) field.get(invocationHandler);
                String[] paths = new String[requestMapping.path().length];
                for (int i = 0; i< requestMapping.path().length; i++) {
                    paths[i] = requestMapping.path()[i].replace("{version}", "v".concat(apiVersion.value()));
                }
                map.put("path", paths);
                String[] values = new String[requestMapping.value().length];
                for (int i = 0; i< requestMapping.value().length; i++) {
                    values[i] = requestMapping.value()[i].replace("{version}", "v".concat(apiVersion.value()));
                }
                map.put("value", values);
                // 上面改了value和path是因为注解里@AliasFor，两者互为，不晓得其它地方有没有用到，所以都改了，以免其它问题
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return requestMapping != null ? this.createRequestMappingInfo(requestMapping, condition) : null;
    }

}
```
PS：不太推荐使用swagger
1.文档在代码里维护，需要改一些描述的时候还需要改代码
2.swagger只在测试阶段使用，上线了还得禁用掉，虽然可以根据配置切换环境从而切换启停，但难免不优雅
3.swagger使用变量不方便，不能真正达到测试的效果，如接口需要签名，防报文等验证的时候
4.swagger无法像postman之类的一样可以动态修改请求参数，动态处理响应参数，真正联调的时候其实是很不方便的