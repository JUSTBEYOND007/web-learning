在你的代码中，有这一行：

codeTypeScript

```
const newToken = await refreshToken(); // 调⽤刷新Token接⼝
```

只要有这个动作（Access Token 失效了，去调接口换一个新的），**这就已经是双 Token 机制了**。如果是单 Token，过期了就只能跳转登录页，无法在后台静默刷新。