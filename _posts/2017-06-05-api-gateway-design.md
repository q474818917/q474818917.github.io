---
layout: post
title: 基于Kong的Auth2.0的设计
categories: Ops
description: 基于Kong的Auth2.0的设计
keywords: openresty、lua、kong、nginx
---

## 关于短信防刷机制

+ 通过图形验证码解决，每次发送短信验证码都需要请求一次图形验证码接口，使用完成后，将本次请求的图形验证码删除
+ redis 对api请求限制，通过key：sessionID+requestIP， value：访问次数。超过直接返回
+ nginx_lua:access_by_lua,通过UA设置过滤黑名单

## kong在Docker环境安装

## kong相关术语：

+ 一个consumer对应一个客户，consumer ID就是authorization_userid。一个consumer有多个证书，其中包括：key Auth、Basic Auth、OAuth2、HMAC Auth、jwt、Acls


## 关于Auth2.0

	关于Kong的认证，还包括几种插件：basic auth、jwt、和该内容中的Auth2.0，其中basic auth，是针对用户名、密码进行base64编码， jwt存在的主要目的是服务器不对session进行存储。

+ Authorization Code:先跳转服务端认证，认证完服务端跳转客户端并附带code，使用Code值请求Access Token
+ Implicit:简化的Authorization Code，不需要code这个环节
+ Password:用户填写信息给客户端，客户端提交给认证服务器，获取token，不安全
+ Client Credentials:客户端以自己的省份提交给认证服务器，获取token

![](/images/blog/Auth2.0.png)

## Auth2.0在kong上的应用

### 通用化插件请求创建步骤

+ 创建API

+ 为该API创建Auth2.0 plugin

+ 创建Consumer

+ 为Consumer创建Auth2.0 应用

### Authorization Code

+ 调用接口：/oauth2/authorize

```
curl --insecure https://120.26.116.188:8443/oauth2/authorize \
-H "Host:360guanai.com" \
-d "client_id=d21f86f642eb49b2940948754cdd9487" \
-d "provision_key=041ed99597e3467a871c53f26a33e9ac" \
-d "authenticated_userid=ab29588f-6668-4981-a9b2-19ee93b9ebef" \
-d "response_type=code" \
-d "scope=read"
```

```
{"redirect_uri":"http:\/\/www.jd.com?code=6038edd0fec84b2283892057f5f1f7a7"}
```

```
curl --insecure https://120.26.116.188:8443/oauth2/token \
-H "Host:360guanai.com" \
-d "client_id=d21f86f642eb49b2940948754cdd9487" \
-d "client_secret=e6cefea2d824457e868a1aea4fede2dd" \
-d "code=6038edd0fec84b2283892057f5f1f7a7" \
-d "grant_type=authorization_code"
```

### Implicit

+ 调用接口：/oauth2/authorize

```
curl --insecure https://120.26.116.188:8443/oauth2/authorize \
-H "Host:360guanai.com" \
-d "client_id=d21f86f642eb49b2940948754cdd9487" \
-d "response_type=token" \
-d "provision_key=041ed99597e3467a871c53f26a33e9ac" \
-d "scope=read" \
-d "authenticated_userid=ab29588f-6668-4981-a9b2-19ee93b9ebef"

```

### Password

+ 调用接口：/oauth2/token

```
curl --insecure https://120.26.116.188:8443/oauth2/token \
-H "Host:360guanai.com" \
-d "client_id=d21f86f642eb49b2940948754cdd9487" \
-d "client_secret=e6cefea2d824457e868a1aea4fede2dd" \
-d "provision_key=041ed99597e3467a871c53f26a33e9ac" \
-d "authenticated_userid=ab29588f-6668-4981-a9b2-19ee93b9ebef" \
-d "grant_type=password" \
-d "scope=read"

```

### Client Credentials

+ 调用接口：/oauth2/token

```
curl --insecure https://120.26.116.188:8443/oauth2/token \
-H "Host:360guanai.com" \
-d "client_id=d21f86f642eb49b2940948754cdd9487" \
-d "client_secret=e6cefea2d824457e868a1aea4fede2dd" \
-d "grant_type=client_credentials" \
-d "scope=read"
```
