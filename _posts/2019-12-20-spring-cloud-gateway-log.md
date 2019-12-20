---
layout: post
title: spring cloud gateway中读取请求参数
date: 2019-12-20
Author: minweikai
tags: [工具,spring cloud gateway]
comments: true
---

## 1. 我的版本：

- spring-cloud：Hoxton.RELEASE
- spring-boot：2.2.2.RELEASE
- spring-cloud-starter-gateway

### 2. 请求日志

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.core.io.buffer.DataBufferUtils;
import org.springframework.http.HttpMethod;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.http.server.reactive.ServerHttpRequestDecorator;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.nio.charset.StandardCharsets;
import java.util.Map;
import java.util.stream.Collectors;

/**
 * @author MinWeikai
 * @date 2019-12-20 18:09:39
 */
@Slf4j
@Component
public class LoggerFilter implements GlobalFilter {

	@Override
	public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
		ServerHttpRequest request = exchange.getRequest();
		String method = request.getMethodValue();

		if (HttpMethod.POST.matches(method)) {
			return DataBufferUtils.join(exchange.getRequest().getBody())
					.flatMap(dataBuffer -> {
						byte[] bytes = new byte[dataBuffer.readableByteCount()];
						dataBuffer.read(bytes);
						String bodyString = new String(bytes, StandardCharsets.UTF_8);
						logtrace(exchange, bodyString);
						exchange.getAttributes().put("POST_BODY", bodyString);
						DataBufferUtils.release(dataBuffer);
						Flux<DataBuffer> cachedFlux = Flux.defer(() -> {
							DataBuffer buffer = exchange.getResponse().bufferFactory()
									.wrap(bytes);
							return Mono.just(buffer);
						});

						ServerHttpRequest mutatedRequest = new ServerHttpRequestDecorator(
								exchange.getRequest()) {
							@Override
							public Flux<DataBuffer> getBody() {
								return cachedFlux;
							}
						};
						return chain.filter(exchange.mutate().request(mutatedRequest)
								.build());
					});
		} else if (HttpMethod.GET.matches(method)) {
			Map m = request.getQueryParams();
			logtrace(exchange, m.toString());
		}
		return chain.filter(exchange);
	}

	/**
	 * 日志信息
	 *
	 * @param exchange
	 * @param param    请求参数
	 */
	private void logtrace(ServerWebExchange exchange, String param) {
		ServerHttpRequest serverHttpRequest = exchange.getRequest();
		String path = serverHttpRequest.getURI().getPath();
		String method = serverHttpRequest.getMethodValue();
		String headers = serverHttpRequest.getHeaders().entrySet()
				.stream()
				.map(entry -> "            " + entry.getKey() + ": [" + String.join(";", entry.getValue()) + "]")
				.collect(Collectors.joining("\n"));
		log.info("\n" + "----------------             ----------------             ---------------->>\n" +
						"HttpMethod : {}\n" +
						"Uri        : {}\n" +
						"Param      : {}\n" +
						"Headers    : \n" +
						"{}\n" +
						"\"<<----------------             ----------------             ----------------"
				, method, path, param, headers);
	}

}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 3. 测试输出，我这边测试没有问题，日志正常输出

![img](https://img-blog.csdnimg.cn/20191220183447900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE4Nzg3Ng==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)