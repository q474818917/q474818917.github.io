---
layout: post
title: wrk搭配lua脚本，个性化压测
categories: Blog
description: wrk搭配lua脚本，个性化压测
keywords: wrk, jmeter, lua, apache benchmark
---

## 压测工具

+ wrk

```
wrk -t2 -c10 -d10s -s wrk.lua http://www.baidu.com
```

+ apache benchmark：性能基准测试时使用

+ Hey：go实现的压测工具

+ http_load：

```
http_load -p 100 -s 10 urls
```

+ siege

```
siege -c 200 -r 10 -f baidu.url
```

## wrk
对于一些动态构建的请求，比如：认证、校验、MD加密、http请求参数化， ab、http_load、siege都不能满足需求，倒是jmeter、wrk可以。
更多的lua示例可以参照[github](https://github.com/wg/wrk/tree/master/scripts)
wrk请求压测，调用lua分下面3个阶段：setup、running、done
![](http://type.so/usr/uploads/2016/08/970528889.png)

+ wrk的全局属性, 可以直接拿到lua中使用的

```
wrk = {
  scheme  = "http",
  host    = "localhost",
  port    = nil,
  method  = "GET",
  path    = "/",
  headers = {},
  body    = nil,
  thread  = <userdata>,
}
```

+ wrk的全局方法, 可以直接拿到lua中使用的

```
-- 生成整个request的string，例如：返回
-- GET / HTTP/1.1
-- Host: tool.lu
function wrk.format(method, path, headers, body)

-- 获取域名的IP和端口，返回table，例如：返回 `{127.0.0.1:80}`
function wrk.lookup(host, service)

-- 判断addr是否能连接，例如：`127.0.0.1:80`，返回 true 或 false
function wrk.connect(addr)
```

+ Setup阶段
setup是在线程创建之后，启动之前。

```
function setup(thread)

-- thread提供了1个属性，3个方法
-- thread.addr 设置请求需要打到的ip
-- thread:get(name) 获取线程全局变量
-- thread:set(name, value) 设置线程全局变量
-- thread:stop() 终止线程
```

+ Running阶段

```
function init(args)
-- 每个线程仅调用1次，args 用于获取命令行中传入的参数, 例如 --env=pre

function delay()
-- 每个线程调用多次，发送下一个请求之前的延迟, 单位为ms

function request()
-- 每个线程调用多次，返回http请求

function response(status, headers, body)
-- 每个线程调用多次，返回http响应
```

+ Done阶段

可以用于自定义结果报表，整个过程中只执行一次

```
function done(summary, latency, requests)


latency.min              -- minimum value seen
latency.max              -- maximum value seen
latency.mean             -- average value seen
latency.stdev            -- standard deviation
latency:percentile(99.0) -- 99th percentile value
latency(i)               -- raw value and count

summary = {
  duration = N,  -- run duration in microseconds
  requests = N,  -- total completed requests
  bytes    = N,  -- total bytes received
  errors   = {
    connect = N, -- total socket connection errors
    read    = N, -- total socket read errors
    write   = N, -- total socket write errors
    status  = N, -- total HTTP status codes > 399
    timeout = N  -- total request timeouts
  }
}
```

## 示例，秒杀系统，QPS压测：

+ 接口：

```
GET /miaosha/i/miaosha?goodsRandomName=0e67e331-c521-406a-b705-64e557c4c06c&mobile=15050033920 HTTP/1.1
Host: 127.0.0.1:8080
```

+ lua

```
-- example script that demonstrates use of setup() to pass
-- data to and from the threads

local counter = 1
local threads = {}

function setup(thread)
   thread:set("id", counter)
   table.insert(threads, thread)
   counter = counter + 1
end

function init(args)
   requests  = 0
   responses = 0

   local msg = "thread %d created"
   print(msg:format(id))
end

function request()
   requests = requests + 1
   local returnRequest = "/miaosha/i/miaosha?goodsRandomName=0e67e331-c521-406a-b705-64e557c4c06c"
                        .. "&mobile=" .. math.random(15000000000,19999999999)
   print(wrk.format(nil, returnRequest))
   return wrk.format(nil, returnRequest)
end

function response(status, headers, body)
   responses = responses + 1
end

function done(summary, latency, requests)
   for index, thread in ipairs(threads) do
      local id        = thread:get("id")
      local requests  = thread:get("requests")
      local responses = thread:get("responses")
      local msg = "thread %d made %d requests and got %d responses"
      print(msg:format(id, requests, responses))
   end
end

```

+ run

```
wrk -t400 -c400 -d60s -s wrk.lua http://127.0.0.1:8080
```



