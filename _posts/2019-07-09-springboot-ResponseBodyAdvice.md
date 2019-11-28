---
layout: post
title: springboot中实现ResponseBodyAdvice接口在项目中统一处理修饰返回体
date: 2019-07-09
Author: minweikai
tags: [技巧,工具]
comments: true
---

如果你有遇到此问题，相信我这篇博客可以帮你减少一部分工作量，让你专心于业务代码的实现。

在项目中接口里总是会遇到这样一个问题，每次接口返回json数据都要return自己建立的公共返回体。

如我的

![img](https://img-blog.csdnimg.cn/20190709165341574.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE4Nzg3Ng==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

每个接口都要写这样一句话，虽然就一句，但是我很懒，尤其是重复的代码，我真的不想再写第二遍。然而当时也没有什么更好的方法，只能这样一遍一遍的去写。

```java
return new ResponseResult<>(menuInfoListVo);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

.................很久之后，我无意间了解到ResponseBodyAdvice接口，简单的说这个接口可以处理返回body数据，经过在这个方法beforeBodyWrite的二次加工再返回到前端，详细的知识点可以在网上搜索。

实现源码

```java
import com.haj.tool.util.ResponseResult;
import org.springframework.core.MethodParameter;
import org.springframework.http.MediaType;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice;

import java.util.Collection;

/**
 * 修饰返回体
 *
 * @author MinWeikai
 * @date 2019/6/24 10:56
 */
@RestControllerAdvice(basePackages = "com.mwk")//要扫描的包
public class MyResponseBodyAdvice implements ResponseBodyAdvice {

	@Override
	public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType,
	                              Class selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {

                //如果body为空，返回默认信息
		if (body == null) {
			return new ResponseResult<>();
		}
		//匹配ResponseResult
		if (body instanceof ResponseResult) {
			return body;
		}
 
                /**
		 * 其他情况直接将返回的信息直接塞在data中
		 */
		return new ResponseResult<>().setData(body);
	}

	@Override
	public boolean supports(MethodParameter returnType, Class converterType) {
		return true;
	}
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

配置完成后，就可以在controller中直接返回vo了。

```java
return menuInfoListVo;
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

剩下的工作就交给MyResponseBodyAdvice ，它会帮你自动装载。