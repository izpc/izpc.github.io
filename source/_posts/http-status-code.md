---
title: HTTP 状态码
date: 2024-01-03 18:13:47
tags: http
---

# 介绍

HTTP状态码用于表示一个HTTP请求是否成功，以及在不成功的情况下出现了什么类型的错误。状态码分为几个范围，每个范围代表了一类响应。

1xx（100-199）：信息性状态码，表示接收的请求正在处理。
2xx（200-299）：成功状态码，表示请求正常处理完毕。
3xx（300-399）：重定向状态码，要完成请求需要进一步操作。
4xx（400-499）：客户端错误状态码，表示请求有语法错误或请求无法实现。
5xx（500-599）：服务器错误状态码，表示服务器在处理请求时发生了错误。

## 不常见的状态码

101：这是一个信息性（1xx）状态码，表示服务器理解了客户端的请求，并且通过这个状态码告知客户端它将切换到请求的协议，这通常用在WebSockets协议切换中。

499：这个状态码实际上不是一个官方的HTTP状态码，它是由Nginx定义的，用来表示客户端在服务器有机会发送回响应之前关闭了连接。也就是说，当服务器准备发送回响应，但是发现客户端已经关闭了连接时，Nginx会记录这个状态码。这通常发生在客户端取消了请求的情况下。