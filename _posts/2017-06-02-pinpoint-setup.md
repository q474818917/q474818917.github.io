---
layout: post
title: 基于字节码注入的分布式追踪系统：Pinoint 安装及使用
categories: Blog
description: 基于字节码注入的分布式追踪系统：Pinoint 安装及使用
keywords: pinpoint、dagger、zipkin、分布式、追踪
---

## Pinpoint：java写的APM
+ callstack：分布式事物追踪，基于google的dagger。
+ Inspector：监控cpu、垃圾回收、tps、jvm图表
+ serverMap：服务分布架构图
+ Realtime active Thread Count ：实时活跃线程数
+ Request、response Scatter Chart：请求应答分布图
+ 监控报警：alarm


## [安装](https://github.com/naver/pinpoint)
+ hbase：版本1.2.4
    + 集群或者单机都可以
    + 创建schemas,最新的schemas脚本在github上

+ pinpoint-collector：收集agent发送过去的数据
    + 丢入tomcat中
    + 配置hbase.properties:zk_host,zk_port
    + 配置pinpoint-collector.properties:tcpListenPort,udpStatListenPort,udpSpanListenPort
    + ps:三个port和agent中配置的port一致，agent连接到对应的port
    
+ pinpoint-web：
    + 修改ROOT.war，丢入tomcat
    + 配置hbase.properties:zk_host,zk_port
    + 配置pinpoint-web.properties:cluster.enable=false,也可以集群
    + 记得配置下index.html，这里面有访问google的js，拿掉
    
+ pinpoint-agent：整合java Application
    + 该服务和被监控的java Application同一服务器
    + 配置agent目录下的pinpoint.config
    + 启动应用:agentId,和applicationName不同于别的应用
    ```
     -javaagent:/Users/wangzx/Downloads/pinpoint-agent-1.6.0/pinpoint-bootstrap-1.6.0.jar -Dpinpoint.agentId=1001 -Dpinpoint.applicationName=tijiantong-web
    ```

## 使用
+ server-map:架构图
    + 用户三个request打入到前置应用服务器tomcat
    + tomcat请求后端的服务、组件
    + 其中redis调用次数101次
    + 另外一个是dubbo服务，20881调用的服务2次，20388调用的服务1次
![](/images/blog/4728.png)

+ 请求分布图
    + 其中三个点，代表tomcat的三次请求
    + Response summary：1s产生的2个请求，3s产生的1个请求
![](/images/blog/4741.png)

+ callstack图
    + 拉取上图的三个点产生的
    + 这是一个请求api，下面是总响应时间及调用链
![](/images/blog/4748.png)

## 上生产
+ 设置采样率，可以获取请求数