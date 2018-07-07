---
title: http简易指南
date: 2018-07-07 10:24:03
tags: http
id: 1530930262
---
# HTTP 报文内容
HTTP 报文是由一行一行的简单纯文本字符串组成的，不是二进制代码。下面是一个简单例子：

请求：
.|内容
-|-
起始行(start line) | GET /test/hi-there.php HTTP/1.0
首部(header) | Accept: text/*<br>Accept-Language: en,zh

响应：
.|内容
-|-
起始行(start line) | HTTP/1.0 200 OK
首部(header) | Content-type: text/plain<br>Content-length: 19
主体(body)   | Hi! I'm a message!

起始行和首部，由一个回车符（ASCII 13）和一个换行符（ASCII 10）作为结束。

# 起始行
## 常用 HTTP 方法
请求报文起始行里的 *GET* 就是 HTTP 方法，其余方法参考下表：

方法 | 描述 | 是否包含主题
-|-|-
GET | 从服务器获取一份文档 | 否
HEAD | 只从服务器获取文档的首部 | 否
POST | 向服务器发送需要处理的数据 | 是
PUT | 将请求的主体部分存储在服务器上 | 是
TRACE | 对可能经过代理服务器传输到服务器上去的报文进行追踪 | 否
OPTIONS | 决定可以在服务器上执行哪些方法 | 否
DELETE | 从服务器上删除一份文档 | 否

## HTTP 状态码
响应报文起始行里的 *200* 就是 HTTP 状态吗，其他状态码参考下表：

整体范围 | 已定义范围 | 分类 | 描述
-|-|-|-
100 ~ 199 | 100 ~ 101 | 信息提示 | 这一类型的状态码，代表请求已被接受，需要继续处理。这类响应是临时响应，只包含状态行和某些可选的响应头信息，并以空行结束。
200 ~ 299 | 200 ~ 206 | 成功 | 这一类型的状态码，代表请求已成功被服务器接收、理解、并接受
300 ~ 399 | 300 ~ 305 | 重定向 | 这类状态码代表需要客户端采取进一步的操作才能完成请求。通常，这些状态码用来重定向，后续的请求地址（重定向目标）在本次响应的 Location 域中指明。其中301是永久重定向，302是临时重定向。
400 ~ 499 | 400 ~ 415 | 客户端错误 | 这类的状态码代表了客户端看起来可能发生了错误，妨碍了服务器的处理。其中401是未授权，404是找不到请求资源。
500 ~ 599 | 500 ~ 505 | 服务端错误 | 这类状态码代表了服务器在处理请求的过程中有错误或者异常状态发生，也有可能是服务器意识到以当前的软硬件资源无法完成对请求的处理。

# 首部
HTTP 首部字段有很多，而且可以添加自定义的字段，这里只列几个比较常见或重要的。

首先要说明很多字段值可以加一个质量值 **"q"**，定义各个值的权重，例如这个首部：
> Accept-Language: en;q=0.5, fr;q=0.0, nl;q=1.0, tr;q=0.0

q 值的范围从 0.0 ～ 1.0（0.0是优先级最低，而1.0是优先级最高的）。注意偏好的排序不重要，只有 q 值才是重要的。

## Accept
客户端用 Accept 首部来通知服务器可以接受哪些媒体类型。

- **类型**：请求首部
- **注释**：可以用"*"通配符。例如"*/*","image/*"
- **举例**：Accept: text/*, image/*

## Accept-Charset
客户端用 Accept-Charset 通知服务器可以接受哪些字符集。

- **类型**：请求首部
- **注释**：和 Accept 一样支持通配符
- **举例**：Accept-Charset: utf-8

## Accept-Encoding
客户端用 Accept-Encoding 首部来告诉服务器可以接受哪些编码方式。

- **类型**：请求首部
- **举例**：Accept-Encoding: gzip;q=1.0, compress;q=0.5

## Cache-Control

- **类型**：通用首部
- **举例**：Cache-Control: no-cache

## Connection
在 HTTP/1.0 中 默认关闭 Keep-Alive 模式，要通过 Connection: Keep-Alive 打开。在 HTTP/1.1 中默认开启了 Keep-alive 模式，通过加入 Connection: close 才关闭。

- **类型**：通用首部
- **举例**：Connection: close

## Content-Encoding
Content-Encoding 首部用于说明是否对某对象进行过编码。

- **类型**：实体首部
- **举例**：Content-Encoding: compress, gzip

## Content-Length
Content-Length 首部说明实体**主体**部分的长度。HEAD 请求响应中如果有这个首部，表示如果发送的话，实体主体的长度。

- **类型**：实体首部
- **举例**：Content-Length: 2417

## Content-Type
Content-Type 首部说明了报文中对象的媒体类型。

- **类型**：实体首部
- **举例**：Content-Type: text/html; charset=utf-8

## Cookie, Cookie2
Cookie 用来改变 Cookie，Cookie2 是对 Cookie 的扩展

- **类型**：扩展请求首部
- **举例**：Cookie: name="ljj"

## Date
Date 首部给出了报文创建的日期和时间。

- **类型**：通用首部
- **举例**：Date: Tue, 3 Oct 1997 02:15:31 GMT

## ETag
ETag 首部为保文中包含的实体提供了实体标记。实体标记实际上就是一种标识资源的方式。

- **类型**：实体首部
- **举例**：ETag: "11e92a-457b-31345aa"

## Location
服务器可以通过 Location 首部将客户端导向某个资源地址。浏览器收到这个首部将会跳转。

- **类型**：响应首部
- **举例**：Location: http://www.ljj.pub

## Set-Cookie, Set-Cookie2
Set-Cookie 首部是 Cookie 的搭档，规定了 Cookie 的作用域和其他一些信息。Set-Cookie2 是对 Set-Cookie 的扩展。

- **类型**：扩展响应首部
- **举例**：Set-Cookie: name=ljj; domain="ljj.pub"; path=/posts/

例子里 name=ljj 是强制的，其他都是可选的

## User-Agent
客户端用 User-Agent 首部来表示其类型

- **类型**：请求首部
- **举例**：User-Agent: Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 5.0)

另外说明一个，Age（秒）, Cache-Control, Expire（日期） 是设定为计算缓存生命周期的，但现在更好的做法是永久缓存，然后给 URL 加版本号的方法更新文件。

# HTTP 与 WebSocket
WebSocket 是在 HTTP 基础上的协议，需要浏览器支持。WebSocket 允许客户端和服务器通过长链接双工通信，即两边可以不断的向另一边发送信息而不用重新连接。没有 WebSocket 协议之前如果服务器要不断向浏览器发送信息，就要打开长链接（Connection: Keep-Alive，默认打开），而且只能服务器向浏览器发送。

-------------------------
参考资料：
《HTTP权威指南》
http状态码：https://baike.baidu.com/item/HTTP状态码/5053660?fr=aladdin