---
title: 网络
date: '2023-02-24 02:13:13'
quiz: true
categories:
  - Quetions
tags:
  - 面试
abbrlink: 50daec4
---

## 网络基础

0. 介绍一下 `七层网络模型` 和 `TCP/IP 四层概念模型` {.quiz .fill}

1. `HTTP` 是什么？ {.quiz .fill}

   > - `HTTP` 是为了让客户端能够理解和呈现服务端发送的超文本内容，基于 `TCP/IP` 协议的应用层协议`超文本传输协议`。
   > - `HTTP` 具有`无状态`、`明文传输`、`报文灵活可扩展`、`可靠传输`等特点。
   > - `HTTP` 有一些缺陷：
   >
   >   1. `明文传输`的特点导致`数据传输的安全性欠缺`；
   >   2. 当使用 `HTTP/1` 时，一个 **TCP 连接** 同时只能处理一条请求，当前请求耗时过长会导致`队头堵塞`的问题；
   >   3. 在需要保存上下文信息时，`无状态`的特性会让网络开销增加；

2. `HTTP` 的组成部分有哪些？它的报文结构是怎样的？ {.quiz .fill}

   > - `请求报文`的组成部分 -> `状态行` + `请求头` + `消息主体`
   > - `响应报文`的组成部分 -> `状态行` + `响应头` + `响应正文`
   > - `报文结构`
   >   > 起始行 + 头部 + 空行 + 实体
   >   > 请求报文的起始行： `Method` + `Path` + `Version`
   >   > 响应报文的起始行： `Version` + `Status` + `Status Content`

3. `HTTP` 各个版本（1.0、1.1、2.0、3.0）的区别 {.quiz .fill}

4. `HTTP` 有哪些请求方法？ {.quiz .fill}

   > `GET`、`POST`、`PUT`、`DELETE`、`OPTIONS`、`HEAD`、`CONNECT`、`PATCH`、`TRACE`

5. `GET` 和 `POST` 的区别 {.quiz .fill}

   > 1. `GET` 会被浏览器主动缓存下来，`POST` 不会。
   > 2. `GET` 的参数默认通过 URL 传参，并且会进行 `URL` 编码，`POST` 还可以在 `Body` 中进行传参，body 传参没有编码限制。
   > 3. `GET` 常用于获取数据，请求是`幂等`的，`POST` 常用于数据操作，默认没有`幂等性`
   > 4. `GET` 请求是一次性将报文都发送出去，`POST` 会先把 `header` 部分发出去，如果服务端响应了 100 状态才会把 `body` 部分发送出去。

6. `HTTP/2` 的`多路复用`和 `HTTP/1.1`的 `keep-alive` 的区别 {.quiz .fill}

7. `HTTPS` 是什么 {.quiz .fill}

   > - `HTTPS` 是基于 `HTTP` 协议上套的一层 `TLS` 协议，用于服务器的身份认证，让`HTTP`更加安全。
   > - `HTTPS` 主要解决了`服务器的身份认证`、`明文传输`等问题。
   > - `HTTPS` 具有以下特点：
   >   1. 可以让内容加密，避免中间服务器查看;
   >   2. 通过 `HTTPS` 证书，验证用户访问的是否正确的服务器，避免被第三方 `DNS` 劫持到其他服务器；
   > - `TLS` 采用的是混合加密方式，先使用`非对称加密`方式建立起通信连接，通过`对称加密`的方式进行消息加密

8. `HTTPS` 的握手过程 {.quiz .fill}

9. `HTTPS` 的握手过程中，客户端如何验证证书的合法性 {.quiz .fill}

10. 介绍一下 `HTTPS` 的中间人攻击 {.quiz .fill}

11. `TCP` 和 `UDP` 的区别？ {.quiz .fill}

12. `TCP` 的三次握手和四次挥手 {.quiz .fill}

13. A、B 机器正常连接后，B 机器突然重启，问 A 此时 处于 TCP 什么状态？ {.quiz}

    - SYN_SENT
    - SYN_RCVD
    - ESTABLISHED
    - FIN_WAIT1
    - CLOSE_WAIT
      {.options}

14. `HTTP/3` 为什么基于 UDP 实现？如何解决 UDP 的弊端 {.quiz .fill}

15. `HTTP 状态码` 有哪些以及对应的含义 {.quiz .fill}

16. `DNS` 的解析原理 {.quiz .fill}

17. `Cookie` 是什么？ {.quiz .fill}

18. `Cookie` 和 `Session`，以及和 `Storage` 的区别？ {.quiz .fill}

19. `跨域` 是什么？如何解决跨域 {.quiz .fill}

20. 什么是 `CSRF` 攻击，如何防范？ {.quiz .fill}

21. 什么是`预请求`？ {.quiz .fill}

## 浏览器

1. 描述从输入 URL 到页面显示的过程 {.quiz .fill}

2. 浏览器一帧内做了什么 {.quiz .fill}

3. 描述一下浏览器的事件机制 {.quiz .fill}

4. 浏览器存储数据的方式有哪些 {.quiz .fill}

5. service worker 是什么 {.quiz fill}

6. `Web Worker` 是什么？ {.quiz .fill}

7. `MessageChannel` 和 `window.postMessage` {.quiz .fill}

8. 如何进行跨页签通信？ {.quiz .fill}
