---
title: springboot使用ResponseBodyAdvice统一包装结果
date: 2020-01-16 10:22:38
toc: true
categories: 后端
tags:
    - springboot
---



在我们使用SpringBoot编写接口的时候，最好是返回一个统一格式的JSON，该格式包含错误码，附带信息，以及携带的数据。这样前端在解析的时候就能统一解析，同时携带错误码可以更加容易的排查错误。

<!-- more -->

## 添加ResponseBodyAdvice

**`ResponseBodyAdvice`接口里一共包含了两个方法**

- `supports`:该组件是否支持给定的控制器方法返回类型和选择的{@code HttpMessageConverter}类型。**用于判断是否需要做处理。**
- `beforeBodyWrite`:在选择{@code HttpMessageConverter}之后调用，在调用其写方法之前调用。**用于做返回处理。**

```java com.xiaotang.springaccount.dto.ResponseBodyDTO
// 返回的结果类
@Getter
@Setter
public class ResponseBodyDTO {
    private String msg;
    private Integer code;
    private Object result;
    private Boolean success;

    public ResponseBodyDTO(Object result) {
        this.code = 200;
        this.success = true;
        this.msg = "操作成功";
        this.result = result;
    }

    public ResponseBodyDTO(){}
}
```



```java com.xiaotang.springaccount.config.ResponseDataHandler
@ControllerAdvice
public class ResponseDataHandler implements ResponseBodyAdvice {

    @Override
    public boolean supports(MethodParameter methodParameter, Class aClass) {
        return true;
    }

    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType mediaType, Class aClass, ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse) {
        if(! (body instanceof ResponseBodyDTO)){
            ResponseBodyDTO responseBodyDTO = new ResponseBodyDTO();
            responseBodyDTO.setCode(200);
            responseBodyDTO.setMsg("操作成功");
            responseBodyDTO.setSuccess(true);
            responseBodyDTO.setResult(body);
            body = responseBodyDTO;
        }
        return body;
    }
}
```

### 效果

```java
@PostMapping("getMap")
public Map<String, Object> getMap(){
    Map<String, Object> map = new HashMap<>();
    map.put("username", "xiaotang");
    map.put("age", 1);
    map.put("sex", true);
    return map;
}
```



![](3.png)

返回结果已经成功包装了。

---

### 遇到的问题

```java com.xiaotang.springaccount.controller.LoginController
@RestController
@RequestMapping("login")
public class LoginController {

    @PostMapping("test")
    public String test(){
        return null;
    }
}
```

如果如上所示，最后返回值是null，则会报错，因此需要再添加一个 **HttpMessageConverter**

![](2.png)

---

## 添加 HttpMessageConverter

简单来说只要在添加 WebMvcConfigurer中添加 `MappingJackson2HttpMessageConverter`

```java com.xiaotang.springaccount.config.WebConfigurer
@Configuration
public class WebConfigurer implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
		converters.add(0, new MappingJackson2HttpMessageConverter());
    }
}
```

{% note danger %}
converters 列表里的转换器位置越靠前，优先级越高，因此在后面的转换器可能会被覆盖，因此我们自己添加的converter 最好添加在前面，以免失效。
{% endnote %}

### 使用 FastJsonHttpMessageConverter

> 如果你使用 Spring MVC 来构建 Web 应用并对性能有较高的要求的话，可以使用 Fastjson 提供的`FastJsonHttpMessageConverter` 来替换 Spring MVC 默认的 `HttpMessageConverter` 以提高 `@RestController @ResponseBody @RequestBody` 注解的 JSON序列化速度。而且可选择的配置也很多。

{% note info %}FastJsonHttpMessageConverter 可以将返回对象中的 `list: null` 转换成 []，还可以给 int 、 boolean、string 附上初始值 0、false、""。
{% endnote %}

FastJsonHttpMessageConverter是基于fastjson的一种HttpMessageConverter，spring系统默认使用的是MappingJackson2HttpMessageConverter，因此添加的 fastJsonHttpMessageConverter 一定要放在MappingJackson2HttpMessageConverter 之前。

![Converters](1.png)

```java com.xiaotang.springaccount.config.WebConfigurer
@Configuration
public class WebConfigurer implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
//        converters.add(0, new MappingJackson2HttpMessageConverter());
        FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();

        //2、添加fastjson的配置信息
        FastJsonConfig fastJsonConfig = new FastJsonConfig();

        SerializerFeature[] serializerFeatures = new SerializerFeature[]{
                //    输出key是包含双引号
//                SerializerFeature.QuoteFieldNames,
                //    是否输出为null的字段,若为null 则显示该字段
                SerializerFeature.WriteMapNullValue,
                //    数值字段如果为null，则输出为0
                SerializerFeature.WriteNullNumberAsZero,
                //     List字段如果为null,输出为[],而非null
//                SerializerFeature.WriteNullListAsEmpty,
                //    字符类型字段如果为null,输出为"",而非null
                SerializerFeature.WriteNullStringAsEmpty,
                //    Boolean字段如果为null,输出为false,而非null
                SerializerFeature.WriteNullBooleanAsFalse,
                //    Date的日期转换器
                SerializerFeature.WriteDateUseDateFormat,
                //    循环引用
                SerializerFeature.DisableCircularReferenceDetect,
        };
        fastJsonConfig.setSerializerFeatures(serializerFeatures);
        fastJsonConfig.setCharset(Charset.forName("UTF-8"));

        //3、在convert中添加配置信息
        fastConverter.setFastJsonConfig(fastJsonConfig);
        converters.add(0, fastConverter);
    }
}
```



## 更多的配置接口 WebMvcConfigurer

参考 [Spring Boot配置接口 WebMvcConfigurer](https://blog.csdn.net/fmwind/article/details/81235401)


## 参考链接

- [springboot 使用fastjson替代默认jackson（踩坑路）](https://blog.csdn.net/BBsatan/article/details/98748229)