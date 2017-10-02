
# HSTS
* [wiki](https://zh.wikipedia.org/wiki/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8)

HTTP严格传输安全（英语：HTTP Strict Transport Security，缩写：HSTS）是一套由互联网工程任务组发布的互联网安全策略机制。网站可以选择使用HSTS策略，来让浏览器强制使用HTTPS与网站进行通信，以减少会话劫持风险.

HSTS的作用是强制客户端（如浏览器）使用HTTPS与服务器创建连接。服务器开启HSTS的方法是，当客户端通过HTTPS发出请求时，在服务器返回的超文本传输协议响应头中包含 `Strict-Transport-Security` 字段。非加密传输时设置的HSTS字段无效。

比如，`https://example.com/` 的响应头含有 `Strict-Transport-Security: max-age=31536000; includeSubDomains`。这意味着两点：
1. 在接下来的一年（即31536000秒）中，浏览器只要向example.com或其子域名发送HTTP请求时，必须采用HTTPS来发起连接。比如，用户点击超链接或在地址栏输入 http://www.example.com/ ，浏览器应当自动将 http 转写成 https，然后直接向 https://www.example.com/ 发送请求。
2. 在接下来的一年中，如果 example.com 服务器发送的TLS证书无效，用户不能忽略浏览器警告继续访问网站。

# range

## Range

```http
GET /test.rar HTTP/1.1 
Connection: close 
Host: 127.0.0.1
Range: bytes=0-801 
```

`Range: bytes=first-byte-pos "=" [last-byte-pos]`
* `first-byte-pos` 从0开始.
* `last-byte-pos` 可以省略.
* 闭区间, 包含 `first`, 也包含 `last`.

示例:
* 表示头500个字节: `bytes=0-499`
* 表示第二个500字节: `bytes=500-999`
* 表示最后500个字节: `bytes=-500`
* 表示500字节以后的范围: `bytes=500-`
* 第一个和最后一个字节: `bytes=0-0,-1`
* 同时指定几个范围: `bytes=500-600,601-999`

## Content-Range

响应体中的 Content-Range: `Content-Range: bytes 0-500/1000`
* `0-500` 表示当前下载的501个字节.
* `1000` 表示文件的总大小.