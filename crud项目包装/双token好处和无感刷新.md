关键词:  *请求的鉴权   对用户透明   请求队列机制防止并发刷新 刷新完成后自动重试  refresh_token也失效时重新登录

```
检测到 401 时，用 refresh_token 换取新 access_token，并用新 token 重放失败请求。同时用 isRefreshing 锁和请求队列处理`retryQueue`并发场景：刷新期间的新请求进入队列，刷新成功后统一用新 token 重新执行。整个过程对用户透明，无需重新登录或重新操作。

我们采用了 **access_token + refresh_token 双Token机制** 来平衡安全性和用户体验：access_token用于每次请求的鉴权，有效期短（15分钟）；refresh_token用于无感刷新。当access_token过期返回401时，前端通过拦截器自动使用refresh_token获取新的access_token，并利用请求队列机制防止并发刷新，期间挂起的所有请求会在刷新完成后自动重试，用户完全无感知；仅当refresh_token也失效时才强制登出，从而既减少了频繁登录对用户体验的影响，又通过短期access_token降低了token泄露的安全风险。


```



这是合理的有效期  不是我们项目中的设置
access_token: 15 min
refresh_token: 7 days

1. **保证活跃用户不频繁登录**
    
- 如果你每天频繁操作应用，Access Token 可能过期
    
- 使用 Refresh Token 自动刷新 Access Token → 用户不需要每次都输入账号密码
    
- ✅ 用户体验好，登录“无感刷新”
    

2. **提升非活跃用户安全性**
    

- 非活跃用户长时间没访问，Access Token 过期
    
- Refresh Token 也可能有过期时间
    
- 一旦超期，用户必须重新登录 → 避免旧 Token 被盗用造成安全风险
    
- ✅ 增强安全性

“双 Token 机制保证了用户在 Access Token 过期时仍能通过 Refresh Token 无感刷新，提升体验和安全性。而 `isRefreshing` + `retryQueue` 是前端实现自动刷新机制的方案：多个请求同时遇到过期时，只发起一次刷新请求，其他请求排队等待，从而保证刷新操作高效、无冲突。”

### 如何实现无感刷新的？
检测到 401 时，用 refresh_token 换取新 access_token，并用新 token 重放失败请求。同时用 isRefreshing 锁和请求队列处理`retryQueue`并发场景：刷新期间的新请求进入队列，刷新成功后统一用新 token 重新执行。整个过程对用户透明，无需重新登录或重新操作。



- 在 request.tsL44-L56 的请求拦截器中自动注入 `Authorization: Bearer ${accessToken}`
- 当遇到 401 响应时，在响应拦截器中触发 token 刷新逻辑
- 使用 `isRefreshing` 标志防止并发刷新