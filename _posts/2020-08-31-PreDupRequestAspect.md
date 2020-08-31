---
layout: post
title: 最简单的接口重复请求处理方法
date: 2020-08-31
Author: minweikai
tags: [工具]
comments: true
---

### 前言

常见的业务处理中，我们会遇到用户提交数据时出现重复的数据，可能出现：

1. 用户重复点击提交按钮
2. 接口被别有用心之人恶意请求
3. 其它可能出现的问题网络或程序崩溃

### 解决

**接口**一定要保持对调用方的不信任

在重复请求处理中，我们的想法

1. 用户在较短时间内，可能几秒内重复提交，可以给用户提示“重复请求”
2. 某些接口需要处理
3. 在执行业务方法前就知道是否是重复请求，减缓服务器压力
4. 知道当前用户和用户请求的接口，这样才能针对用户做重复判断

结合以上想法，我们应该能想到spring aop

> ##### 什么是aop
>
> 面向切面编程，在程序开发中主要用来解决一些系统层面上的问题，比如日志，事务，权限等。可以在不改变原有的逻辑的基础上，增加一些额外的功能。
>
> 它利用一种称为"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。
>
> ##### 通知方法
>
> 1. 前置通知:在我们执行目标方法之前运行**@Before**
> 2. 后置通知:在我们目标方法运行结束之后 ,不管有没有异常**(@After)**
> 3. 返回通知:在我们的目标方法正常返回值后运行**(@AfterReturning)**
> 4. 异常通知:在我们的目标方法出现异常后运行**(@AfterThrowing)**
> 5. 环绕通知:动态代理, 需要手动执行joinPoint.procced()(其实就是执行我们的目标方法执行之前相当于前置通知, 执行之后就相当于我们后置通知**(@Around)**

这里很自然的就可以使用前置通知**@Before**，然后结合自定义注解**Annotation**，添加注解到我们想要控制的接口上就可以了。

首先定义一个预重复请求处理注解

```java

/**
 * 预重复请求注解
 *
 * @author MinWeikai
 * @date 2020-08-31 14:21:37
 */
//声明方法
@Target(ElementType.METHOD)
//运行时执行
@Retention(RetentionPolicy.RUNTIME)
public @interface PreDupRequest {

}
```

然后定义一个切面类，包含对使用该注解的处理方法

```java
/**
 * 预重复请求切面方法
 *
 * @author MinWeikai
 * @date 2020-08-31 15:21:35
 */
@Aspect
@Component
public class PreDupRequestAspect {

	@Autowired
	private RedisSpringProxy redisSpringProxy;

	private static final Logger log = LoggerFactory.getLogger(PreDupRequestAspect.class);

	/**
	 * 业务开始处理前执行
	 *
	 * @param preDupRequest
	 * @return
	 * @throws BusinessException 抛出自定义异常，由全局异常处理捕获
	 *                           {@link GlobalExceptionHandler#handleBusinessException(BusinessException)}
	 */
    //定义切入点规则并且是使用了PreDupRequest注解的接口
	@Before("execution(public * com.mwk.controller.*Controller.*(..)) && @annotation(preDupRequest)")
	public void before(PreDupRequest preDupRequest) throws BusinessException {
		try {
			ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
             // 用户请求唯一标识，获取当前sessionId
			String sessionId = attributes.getSessionId();
             // 用户请求的uri
			String uri = attributes.getRequest().getServletPath();
             // 针对用户对应接口的唯一key
			String key = sessionId + "-" + uri;
             // 将生成的key存储在redis中，便于集群处理
			Object object = redisSpringProxy.read(key);
			if (ObjectUtil.isNull(object)) {
				redisSpringProxy.saveEx(key, 2, key);
				return;
			}
		} catch (Throwable e) {
			log.error("预防重复请求处理异常", e);
		}
		throw new BusinessException(ErrorCode.REPEATED_SUBMIT);
	}

}
```

*说明：用户唯一请求判断可以是sessionId、token或其它用户唯一标识。因为我们要处理的是：某用户请求某接口在某时间内。*

自己定义的全局处理异常

```java
/**
 * @author MinWeikai
 * @date 2019/9/11 15:10
 */
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {
	
	private static final Logger log = LoggerFactory.getLogger(ExceptionHandler.class);

	/**
	 * 处理所有业务异常
	 *
	 * @param e
	 * @return
	 */
	@org.springframework.web.bind.annotation.ExceptionHandler(BusinessException.class)
	ResponseResult handleBusinessException(BusinessException e) {
		return new ResponseResult(e.getCode(), e.getMsg());
	}
	
}

```

### 使用

在需要处理重复请求的接口所对应切入点controller并添加该注解**@PreDupRequest**

### 问题

在微信小程序请求接口中会出现sessionId变化的问题，因为这个请求经过微信服务器中转，对于我们的服务器来说每次请求都是新的请求，会生成新的sessionId。

我们可以做用户登陆后返回sessionId给小程序，然后小程序存储到本地，下次请求时将sessionId放在header中传过来，或使用其它用户唯一标识。
