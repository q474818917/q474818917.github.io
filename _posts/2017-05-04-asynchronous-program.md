---
layout: post
title: 关于异步编程看法
categories: Blog
description: 关于异步编程看法
keywords: CompletableFuture、Vertx
---

### 异步编程几种方式
+ 类似于Golang、lua语言中的协程
+ 类似于NodeJs中的异步无阻塞的事件驱动
+ java中并发处理方案：
    + 多线程
    + 协程Quasar
    + 异步回调Vert.x、响应式编程RxJava
    
### Callback Hell
    涉及到异步编程必然会带来callback hell，以Java中Vert.x为例，是怎么处理回调问题的。

+ CompletableFuture
+ RxJava
+ Quasar

### 案例：JDBC的执行请求
+ CompletableFuture

```
CompletableFuture<SQLConnection> connectionCompletableFuture = new CompletableFuture<>();
future.setHandler(conn -> {
    if(conn.failed()){
        logger.error("数据库连接异常", conn.cause().getMessage());
        return;
    }

    connectionCompletableFuture.complete(conn.result());
});

CompletableFuture<ResultSet> futureA = connectionCompletableFuture.thenCompose(result -> {
    CompletableFuture<ResultSet> resultSetCompletableFuture = new CompletableFuture<>();
    result.query("select id from t_user where mobilephone = 15110119364", rs -> {
        if(rs.failed()) {
            logger.info("查询用户异常", rs.cause().getMessage());
            return;
        }
        resultSetCompletableFuture.complete(rs.result());
    });

    return resultSetCompletableFuture;
});

CompletableFuture<ResultSet> futureB = connectionCompletableFuture.thenCompose(result -> {
    CompletableFuture<ResultSet> resultSetCompletableFuture = new CompletableFuture<>();
    result.query("select id from t_user where mobilephone = 17000107708", rs -> {
        if(rs.failed()) {
            logger.info("查询用户异常", rs.cause().getMessage());
            return;
        }
        resultSetCompletableFuture.complete(rs.result());
    });

    return resultSetCompletableFuture;
});

futureA.thenCombine(futureB, (x, y) -> {
    CompletableFuture<List<JsonArray>> tempFuture = new CompletableFuture<>();

    List<JsonArray> j1 = x.getResults();
    List<JsonArray> j2 = y.getResults();
    j1.addAll(j2);

    tempFuture.complete(j1);
    return j1;
}).whenComplete((rs, ex) -> {
    rs.stream().forEach(s -> {
        System.out.println(s.getString(0));
    });
});
```

+ RxJava

```
JDBCClient client = JDBCClient.createShared(vertx, config);
client.getConnectionObservable().subscribe(conn -> {
    Observable<ResultSet> rs1 =  conn.queryObservable("select id from t_user where mobilephone = 17000107708");
    Observable<ResultSet> rs2 = conn.queryObservable("select id from t_user where mobilephone = 15110119364");

    Observable
            .merge(rs1, rs2)                    
            .map(ResultSet::getRows)            //数据转换
            .subscribe(System.out::println);    //数据消费
    /*rs.subscribe(resultSet -> {
        System.out.println("Results : " + resultSet.getRows());
    }, err -> {
        System.out.println("Database problem");
        err.printStackTrace();
    }, conn::close);*/

});
```
