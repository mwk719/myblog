---
layout: post
title: spring cloud gateway中修改响应体，保证返回体是完整的
date: 2019-12-23
Author: minweikai
tags: [工具,spring cloud gateway]
comments: true
---

## 1. 我的版本：

- spring-cloud：Hoxton.RELEASE
- spring-boot：2.2.2.RELEASE
- spring-cloud-starter-gateway

### 2. 问题：

之前试过一些方法拦截到了返回体，但是第一次的请求返回体参数输出是断开的，后来找到方法使其完整输出。

```java
import io.netty.util.ReferenceCountUtil;
import lombok.extern.slf4j.Slf4j;
import org.reactivestreams.Publisher;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.core.io.buffer.DataBufferFactory;
import org.springframework.core.io.buffer.DataBufferUtils;
import org.springframework.core.io.buffer.DefaultDataBufferFactory;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.http.server.reactive.ServerHttpResponseDecorator;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.nio.charset.Charset;

/**
 * 修改响应体过滤器
 *
 * @author MinWeikai
 * @date 2019-12-21 15:01:15
 */
@Slf4j
@Component
public class ModifyResponseBodyFilter implements GlobalFilter, Ordered {

	@Override
	public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
		ServerHttpResponse originalResponse = exchange.getResponse();
		DataBufferFactory bufferFactory = originalResponse.bufferFactory();
		ServerHttpResponseDecorator decoratedResponse = new ServerHttpResponseDecorator(originalResponse) {
			@Override
			public Mono<Void> writeWith(Publisher<? extends DataBuffer> body) {
				if (body instanceof Flux) {
					Flux<? extends DataBuffer> fluxBody = (Flux<? extends DataBuffer>) body;

					return super.writeWith(fluxBody.buffer().map(dataBuffers -> {

						DataBufferFactory dataBufferFactory = new DefaultDataBufferFactory();
						DataBuffer join = dataBufferFactory.join(dataBuffers);

						byte[] content = new byte[join.readableByteCount()];

						join.read(content);
						// 释放掉内存
						DataBufferUtils.release(join);
						String str = new String(content, Charset.forName("UTF-8"));
						//todo 拦截到的返回体内容，可以随意去操作了
						log.info("返回体：{}", str);
						originalResponse.getHeaders().setContentLength(str.getBytes().length);
						return bufferFactory.wrap(str.getBytes());
					}));

				}
				// if body is not a flux. never got there.
				return super.writeWith(body);
			}
		};
		// replace response with decorator
		return chain.filter(exchange.mutate().response(decoratedResponse).build());

	}

	@Override
	public int getOrder() {
		return -1;
	}
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

参考博客：<https://blog.csdn.net/a807719447/article/details/102823340>