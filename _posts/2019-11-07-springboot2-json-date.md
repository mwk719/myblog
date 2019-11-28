---
layout: post
title: 踩坑-spring boot2.2.0返回json日期格式问题
date: 2019-11-07
Author: minweikai
tags: [技巧,踩坑]
comments: true
---

### 问题

  请求接口返回的日期参数总是毫秒值，但是我需要的是这种格式：2019-11-07 15:35:48

### 项目概况

- spring boot2.2.0
- 使用了实现 WebMvcConfigurer接口的拦截器

试了好几种方法

1. ```java
   @JsonFormat(pattern = "yyyy-MM-dd HH-mm-ss", timezone = "GMT+8")，//没有效果
   ```

2. ```java
   spring.jackson.date-format=yyyy-MM-dd
   spring.jackson.time-zone=GMT+8
   spring.jackson.serialization.write-dates-as-timestamps=false //没有效果
   ```

因为我使用了WebMvcConfigurer拦截器影响到了序列化日期格式

原来的代码WebMvcConfigurer配置代码

```java
import com.alibaba.fastjson.serializer.SerializerFeature;
import com.alibaba.fastjson.support.config.FastJsonConfig;
import com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter;
import com.cxd.tool.filter.TokenHandlerInterceptor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.MediaType;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.StringHttpMessageConverter;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
 
import java.nio.charset.Charset;
import java.util.ArrayList;
import java.util.List;
 
/**
 * 请求与返回参数处理
 *
 * @author MinWeikai
 * @date 2019/6/24 10:57
 */
@Configuration
@Slf4j
public class WebConfig implements WebMvcConfigurer {
 
	public HttpMessageConverter<String> stringConverter() {
		StringHttpMessageConverter converter = new StringHttpMessageConverter(
				Charset.forName("UTF-8"));
		return converter;
	}
 
	/**
	 * 格式化返回体，去除null为空字符串
	 *
	 * @return
	 */
	public HttpMessageConverter fastConverter() {
		//1、定义一个convert转换消息的对象
		FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
		//2、添加fastjson的配置信息
		FastJsonConfig fastJsonConfig = new FastJsonConfig();
		fastJsonConfig.setSerializerFeatures(
				//字符串null返回空字符串
				SerializerFeature.WriteNullStringAsEmpty,
				//数值类型为0
				SerializerFeature.WriteNullNumberAsZero,
 
				//空字段保留
				SerializerFeature.WriteNullListAsEmpty);
 
		fastJsonConfig.setCharset(Charset.forName("UTF-8"));
		//2-1 处理中文乱码问题
		List<MediaType> fastMediaTypes = new ArrayList<>();
		fastMediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
		fastConverter.setSupportedMediaTypes(fastMediaTypes);
		//3、在convert中添加配置信息
		fastConverter.setFastJsonConfig(fastJsonConfig);
		return fastConverter;
	}
 
	@Override
	public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
		converters.clear();
		converters.add(fastConverter());
		converters.add(stringConverter());
	}
 
	/**
	 * 请求拦截处理
	 * addPathPatterns 用于添加拦截规则
	 * excludePathPatterns 用于排除拦截
	 *
	 * @param registry
	 */
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		/**
		 * 跨域拦截器需放在最上面
		 * 解決H5页面OPTIONS问题
		 */
		registry.addInterceptor(new CorsInterceptor()).addPathPatterns("/**");
		//业务拦截处理
		registry.addInterceptor(new TokenHandlerInterceptor()).addPathPatterns("/common/**")
				.excludePathPatterns("/swagger-ui.html/**");
	}
}
```

### 现在在这里加上这一行就可以了

```java
SerializerFeature.WriteDateUseDateFormat,
```

![](https://img-blog.csdnimg.cn/20191107172509933.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE4Nzg3Ng==,size_16,color_FFFFFF,t_70)
