---
title: "RFC 6749 - OAuth 2.0"
date: 2021-12-05T11:30:03+00:00
tags: ["RFC","OAUTH"]
series: ["关于RFC规约那档事"]
categories: ["用户认证"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS:  false
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---

# 1.RFC 6749

RFC 6749 主要对 OAuth2 做出了一些规范规定，OAuth2 主要分为下列几种模式

- 授权码
- 隐藏式
- 密码式
- 客户端凭证

第三方应用申请令牌前，必须先到系统备案，拿到两个身份识别码

- client_id 客户端 ID
- client_secret 客户端密钥

# 2.授权码

- 1.A 网站跳转 B 网站
  
  ```
  https://b.com/oauth/authorize?
    response_type=code&
    client_id=CLIENT_ID&
    redirect_uri=CALLBACK_URL&
    scope=read
  ```
  
  - response_type 返回类型
  - client_id 客户端 ID
  - redirect_uri 重定向地址
  - scope 权限范围
- 2.跳转 B 网站后用户点击授权，B 网站携带授权码跳转至 `redirect_uri`
  
  - ```
    https://a.com/callback?code=AUTHORIZATION_CODE
    ```
- 3.A 网站拿到授权码，在后端向 B 请求令牌
  
  ```
  https://b.com/oauth/token?
   client_id=CLIENT_ID&
   client_secret=CLIENT_SECRET&
   grant_type=authorization_code&
   code=AUTHORIZATION_CODE&
   redirect_uri=CALLBACK_URL
  ```
  
  - client_id 客户端 ID
  - client_secret 客户端密钥
  - grant_type 授权模式
  - code  授权码 "2.跳转 B 网站后用户点击授权，B 网站携带授权码跳转至 redirect_uri"
    
    > 2.跳转 B 网站后用户点击授权，B 网站携带授权码跳转至 `redirect_uri`
  - redirect_uri 令牌颁发后的回调地址
- 4. B 网站拿到几个参数后，向 redirect_uri 接口发送一串数据
     
     ```
     {
       "access_token":"ACCESS_TOKEN",
       "token_type":"bearer",
       "expires_in":2592000,
       "refresh_token":"REFRESH_TOKEN",
       "scope":"read",
       "uid":100101,
       "info":{...}
     }
     ```

# 3.隐藏式

- 1.无后端模式，必须将令牌储存在前端
- 2.A 网站跳转到 B 网站授权用户数据给 A 网站使用。
  
  ```
  https://b.com/oauth/authorize?
    response_type=token&
    client_id=CLIENT_ID&
    redirect_uri=CALLBACK_URL&
    scope=read
  ```
  
  - response_type 为 token，要求直接返回令牌
  - client_id 客户端标识
  - redirect_uri 重定向地址
  - scope 授权范围
- 3.B 网站跳回 A 网站，直接给出 token
  
  ```
  https://a.com/callback#token=ACCESS_TOKEN
  ```
  
  令牌位置是锚点，并不是 query_param 等，这里保证了中间传递过程中的安全性问题

# 4.密码式

直接使用用户名密码请求 JSON 接口

```
https://oauth.b.com/token?
  grant_type=password&
  username=USERNAME&
  password=PASSWORD&
  client_id=CLIENT_ID
```

注意 `grant_type` 为 password 模式，最后拿到令牌相关的 JSON 数据

# 5.客户端凭证

没有凭证模式适用于没有前端的命令行

```
https://oauth.b.com/token?
  grant_type=client_credentials&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET
```

- grant_type 为 client_credential
- client_id 客户端 id
- client_secret 客户端凭证

网站经过校验后直接给出令牌，这种令牌是应用级别的，而不是用户级别的。

# 6.更新令牌

```
https://b.com/oauth/token?
  grant_type=refresh_token&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET&
  refresh_token=REFRESH_TOKEN
```

- grant_type 为更新令牌模式
- refresh_token 为前一次获取 Token 时给出的 refresh_token

