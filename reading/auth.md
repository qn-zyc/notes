
<!-- TOC -->

- [JWT](#jwt)
    - [优点](#优点)
    - [工作原理](#工作原理)
    - [JWT 组成](#jwt-组成)

<!-- /TOC -->

# JWT

* Json Web Token
* [原文](https://zhuanlan.zhihu.com/p/27370773?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

## 优点

* 体积小, 传输快.
* 传输方式多样, header, body, url...
* 严谨的结构化, 在 payload 中包含与用户相关的验证信息, 如可访问的路由, 过期时间等, 服务器端不需要查表验证.
* 跨域验证, 单点登录.
* 易于CDN, 任何节点都可以验证.

## 工作原理

![](pic/auth01.png)


## JWT 组成

```js
// Header
{
  "alg": "HS256",
  "typ": "JWT"
}

// Payload
{
  // reserved claims
  "iss": "a.com",
  "exp": "1d",
  // public claims
  "http://a.com": true,
  // private claims
  "company": "A",
  "awesome": true
}

// $Signature
HS256(Base64(Header) + "." + Base64(Payload), secretKey)

// JWT
JWT = Base64(Header) + "." + Base64(Payload) + "." + $Signature
```

**注意: JWT中 header 和 payload 是明文传输, 不要带有敏感信息.**

服务端收到 JWT 后会 decode 出 header 和 payload, 并用服务端保存的私钥验证 signature 是否合法.

私钥只有服务器端保存, 所以客户端无法构造 JWT.

