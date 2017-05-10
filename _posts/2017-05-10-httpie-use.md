---
layout: post
title: 比curl更好使的httpie
categories: Blog
description: 比curl更好使的httpie
keywords: curl, httpie, wget
---

![](https://httpie.org/static/img/httpie2.png?v=72661be530fde9d07e03be9df60312da)
## httpie

+ 直观的语法
+ 格式化和色彩化的终端输出
+ 内置 JSON 支持
+ 支持上传表单和文件
+ HTTPS、代理和认证
+ 任意请求数据
+ 自定义头部
+ 持久性会话
+ 类 Wget 下载
+ 支持 Python 2.6, 2.7 和 3.x
+ 支持 Linux, Mac OS X 和 Windows
+ 插件
+ 文档
+ 测试覆盖率

## httpie 使用说明

+ 安装

```
brew install httpie
``` 

+ get请求

```
http GET http://httpbin.org/ip
```

+ post请求
	+ json键值对: http POST http://httpbin.org/post name=123 city=shanghai
		```
			{
			    "args": {},
			    "data": "{\"name\": \"123\", \"city\": \"shanghai\"}",
			    "files": {},
			    "form": {},
			    "headers": {
			        "Accept": "application/json, */*",
			        "Accept-Encoding": "gzip, deflate",
			        "Connection": "close",
			        "Content-Length": "35",
			        "Content-Type": "application/json",
			        "Host": "httpbin.org",
			        "User-Agent": "HTTPie/0.9.9"
			    },
			    "json": {
			        "city": "shanghai",
			        "name": "123"
			    },
			    "origin": "116.231.142.172",
			    "url": "http://httpbin.org/post"
			}
		```
	+ json外部文件:http POST http://httpbin.org/post < post.json
		```
			{
			    "args": {},
			    "data": "{\n\"name\":\"王振兴\",\n\"city\":\"sh\"\n}\n",
			    "files": {},
			    "form": {},
			    "headers": {
			        "Accept": "application/json, */*",
			        "Accept-Encoding": "gzip, deflate",
			        "Connection": "close",
			        "Content-Length": "36",
			        "Content-Type": "application/json",
			        "Host": "httpbin.org",
			        "User-Agent": "HTTPie/0.9.9"
			    },
			    "json": {
			        "city": "sh",
			        "name": "王振兴"
			    },
			    "origin": "116.231.142.172",
			    "url": "http://httpbin.org/post"
			}

		```
+ form提交

```
http -f POST example.org hello=World
```
+ 文件下载

```
http example.org/file > file
```