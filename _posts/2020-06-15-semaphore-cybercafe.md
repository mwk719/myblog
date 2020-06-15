---
layout: post
title: Semaphore信号量模拟去网吧上网吃鸡的过程
date: 2020-06-15
Author: minweikai
tags: [并发]
comments: true
---

### 简介

Semaphore（信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源。

这样说可能也比较晕，我就把它用去网吧上网来理解：比如现在有一个网吧有20台电脑，同时最多只能有20个人使用。但是因为这个网吧是王某聪开的，所以现在来了100多个人上网。没机子后，剩下的人也耐心的在这里等着。大家看到机子已经坐满了，都在旁边等着瞪着大眼看别人吃鸡，不时还在吐槽“这么有钱的富家公子，居然开这么小的网吧，鄙视他！！”；这句话也可以这样理解“这么有钱的公司，居然买这么小的服务器，看吧，这么大的并发你同时承受不了了吧，还要我来处理限流”。哈哈哈，这里的每一台电脑，就是一个线程了，现在这些等着上机的人就属于被阻塞了。每有一个人下机，他就可以立即去上机，执行了。

Semaphore是计数信号量。Semaphore管理一系列许可证。每个acquire方法阻塞，直到有一个许可证可以获得然后拿走一个许可证；每个release方法增加一个许可证，这可能会释放一个阻塞的acquire方法。然而，其实并没有实际的许可证这个对象，Semaphore只是维持了一个可获得许可证的数量。

acquire()方法 获取许可，即客户上机

release()方法 归还许可，即客户下机

### 应用场景

常用于限制可以访问某些资源的线程数量，例如通过 Semaphore 限流、限制数据库连接。

### 代码如下

```java
import lombok.extern.slf4j.Slf4j;

import java.util.Random;
import java.util.concurrent.Semaphore;

/**
 * 网吧
 *
 * @author MinWeikai
 * @date 2020/6/9 18:13
 */
@Slf4j
public class Cybercafe {

	/**
	 * 电脑数量-可容纳并发
	 */
	private static int computerCount = 5;

	/**
	 * 客户数量-最大并发
	 */
	private static int customerCount = 8;

	/**
	 * 电脑
	 * computerCount 表示有{@value customerCount}台电脑，即最大并发为{@value customerCount}
	 */
	private static Semaphore computer = new Semaphore(computerCount);


	public static void main(String[] args) {
		customerUseComputer();
	}

	/**
	 * 客户使用电脑，当电脑不够用时则客户阻塞等待有电脑空余，
	 * 电脑空余时则客户立即上机。当没有客户上机时则网吧下班
	 */
	private static void customerUseComputer() {

		for (int i = 0; i < customerCount; i++) {
			//客户上机
			Thread customer = new Thread(() -> {
				try {
					Thread thread = Thread.currentThread();
					String name = thread.getName();

					//客户使用，锁定电脑
					computer.acquire();
					log.info("客户[{}]使用电脑[吃鸡]----开始", name);
					Thread.sleep(new Random().nextInt(20) * 1000);
					log.info("客户[{}]使用电脑[吃鸡]----结束", name);
					//客户使用完，释放电脑
					computer.release();

					int free = computer.availablePermits();
					log.info("空余电脑[{}]", free);
					//空余电脑数=总电脑数，网吧下班
					if (free == computerCount) {
						log.info("无人使用电脑，空余电脑[{}]，网吧关门下班", free);
					}
				} catch (InterruptedException e) {
					log.error("客户使用电脑失败", e);
				}
			});
			customer.start();
		}
	}
}

```

### 执行结果

![image-20200615174508065](http://qn.minwk.top/img/image-20200615174508065.png)

先是可容纳最大并发执行5个线程，剩余3个线程阻塞，等待某个线程执行完成，被阻塞线程进入执行。

### 其他方法

- int availablePermits() ：返回此信号量中当前可用的许可证数。
- int getQueueLength()：返回正在等待获取许可证的线程数。
- boolean hasQueuedThreads() ：是否有线程正在等待获取许可证。
- void reducePermits(int reduction) ：减少reduction个许可证。是个protected方法。
- Collection getQueuedThreads() ：返回所有等待获取许可证的线程集合。是个protected方法。

### 资料

- [并发工具类（三）控制并发线程数的Semaphore](http://ifeve.com/concurrency-semaphore/#more-14753)

