```
Cookie 是浏览器存储的一小段数据，服务器可以通过 Set-Cookie 写入浏览器，之后浏览器请求会自动携带 Cookie。

Session 是服务器端存储的用户会话信息，服务器会生成一个 sessionId，并把 sessionId 存到 Cookie 中，浏览器请求时带上 sessionId，服务器通过 sessionId 找到对应的用户会话。

Token 是服务器生成的身份令牌，一般存储在客户端，每次请求通过 Authorization 头主动携带。Token 通常是无状态的，比如 JWT，服务器只需要验证 Token，不需要保存会话信息，更适合分布式系统。
```