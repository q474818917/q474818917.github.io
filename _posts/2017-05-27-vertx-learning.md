---
layout: post
title: jvm响应式编程：vertx 异步费阻塞， 事件驱动编程
categories: Blog
description: jvm响应式编程：vertx 异步费阻塞， 事件驱动编程
keywords: vertx, nodejs, jvm, 并发
---


## vertx 



## vertx 顺序编程

+ 采用future.compose(handler, nextFuture);

	示例中，先设置第一个future执行标志，future.complete(), future会执行future.compose，
	这里会设置第二个stringFuture执行标志，而且会产生另外一个Future，这个future的setHandler接着执行。

```java
//限流器
Future<Long> future = Future.future();
redisClient.llen(Constants.MOON_QUEUE, result -> {
	if(result.succeeded()) {
		future.complete();
		System.out.println(1);
		if(result.result() > rateLimit) {
			System.out.println("对不起，请排队等待！！！");
		}
	}else{
		future.fail(result.cause());
	}
});

//检测是否在待处理中
Future<Long> stringFuture = Future.future();

future.compose(result -> {
	redisClient.hget(Constants.ACTION_MESSAGE + productId, userId, res -> {
		if(res.succeeded()) {
			System.out.println(2);
			stringFuture.complete();
		}
	});
}, stringFuture).setHandler(result -> {
	redisClient.rpush(Constants.MOON_QUEUE, jsonObject.toString(), res -> {
		if(res.succeeded()) {
			System.out.println(3);
			logger.info("加入到Moon消息队列成功，消息是{}", jsonObject.toString());
			//加入正在处理列表缓存
			redisClient.hset(Constants.ACTION_MESSAGE + productId, userId, userId, temp -> {});
		}
	});
});
```