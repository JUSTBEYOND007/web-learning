## "基于 TypeScript 对 Axios 二次封装" 面试回答模板
就说我们使用了401  403  404  500 这几个状态码 
该部分主要实现Axios实例的初始化与类型系统建设。⾸先通过 axios.create() 创建带有基础 URL、超时时间、默认请求头的基础实例。通过TypeScript类型扩展定义增强型配置参数： 
• retry字段⽤于⾃定义单个请求的重试次数
• kipErrorHandler标记⽤于特殊场景跳过全局错误处理

### 📋 标准回答框架（建议分3个层次）

#### 第1层：设计思路（快速定位）

> "我采用**基于类的封装设计**，将 Axios 实例和相关逻辑封装在 `Request` 类中，通过 **TypeScript 泛型**实现类型安全的请求方法。主要做了三个方面的工作：
> 1. **请求拦截器**：自动注入 access_token，处理统一的请求配置
> 2. **响应拦截器**：处理 401 Token 刷新、500 错误重试、全局错误提示
> 3. **扩展方法**：封装 get/post/put/delete 等便捷方法

#### 第2层：核心实现（展示技术深度）

如果面试官追问细节，继续展开：

**1. 类的设计与 TypeScript 泛型**

> "我定义了一个 `Request` 类，内部维护一个 Axios 实例。使用 TypeScript 泛型让返回值类型与后端接口保持一致：
> 
> ```typescript
> public async request<T = any>(config: BizAxiosRequestConfig): Promise<T> {
>   const response = await this.instance.request<T>(config);
>   return response.data;
> }
> 
> public get<T = any>(url: string, config?: BizAxiosRequestConfig): Promise<T> {
>   return this.request<T>({ ...config, method: 'GET', url });
> }
> ```
> 
> 这样调用时就能获得类型提示：`request.get<UserInfo>('/api/user')`"

**2. 请求拦截器 - Token 自动注入**

> "在请求拦截器中，我自动从 localStorage 读取 access_token 并添加到请求头：
> 
```typescript
> this.instance.interceptors.request.use(
>   (config: InternalAxiosRequestConfig) => {
>     const accessToken = this.getAccessToken();
>     if (accessToken && config.headers) {
>       config.headers.Authorization = `Bearer ${accessToken}`;
>     }
>     return config;
>   }
> );
> ```
```

> 
> 这样所有业务代码都不需要手动处理 Token，实现**无感注入**。"

**3. 响应拦截器 - 401 自动刷新**

> "当接口返回 401 时，我会自动触发 Token 刷新流程：
> 
> 
> 
> 
```ts
> if (error.response?.status === 401 && !originalRequest._retry) {
>   if (this.isRefreshing) {
>     // 如果正在刷新，将请求加入队列
>     return new Promise((resolve, reject) => {
>       this.retryQueue.push({ resolve, reject });
>     }).then(token => {
>       originalRequest.headers.Authorization = `Bearer ${token}`;
>       return this.instance(originalRequest);
>     });
>   }
>   
>   originalRequest._retry = true;
>   this.isRefreshing = true;
>   
>   const newAccessToken = await this.refreshAccessToken();
>   // 处理队列中的请求
>   this.retryQueue.forEach(({ resolve }) => resolve(newAccessToken));
>   this.retryQueue = [];
>   
>   // 重放当前请求
>   return this.instance(originalRequest);
> }

```



> `
> 
> 这里使用**请求队列**避免并发刷新，比如同时发送 10 个请求，只会触发一次刷新，其他请求都等待新 Token。"

**4. 指数退避重试机制**

*对于500错误，我们不会立即显示'Internal Server Error'，而是先自动重试，避免服务端临时抖动影响用户体验。只有重试全部失败后，才显示一次'请求失败'提示。



> "对于 500 等服务器错误，我实现了指数退避重试：
> 
> ```
> typescript
> private async handleNetworkError(config: InternalAxiosRequestConfig, error: AxiosError) {
>   const retryCount = config._retryCount || 0;
>   if (retryCount >= this.retryLimit) return Promise.reject(error);
>   
>   config._retryCount = retryCount + 1;
>   const delay = Math.pow(2, retryCount) * 1000; // 1s → 2s → 4s
>   
>   await new Promise(resolve => setTimeout(resolve, delay));
>   return await this.instance(config);
> }
> ```
> 
> 这样可以**避免服务器雪崩**，失败后延迟时间递增重试，最多重试 3 次。"

**5. 全局错误处理**

> "对于业务层面的错误，我实现了统一错误提示：
> 
> ```typescript
> private handleError(error: AxiosError) {
>   if (error.response) {
>     const status = error.response.status;
>     switch (status) {
>       case 400: message.error('Bad Request'); break;
>       case 403: message.error('Access Forbidden'); break;
>       case 404: message.error('Resource Not Found'); break;
>       case 500: message.error('Internal Server Error'); break;
>     }
>   }
> }
> ```
> 
> 通过 `skipErrorHandler` 标志可以跳过全局处理，用于自定义错误逻辑的场景。"


应该是    我们按照这个背
```ts
// 1. 统一拦截
axios.interceptors.response.use(
  response => response,
  async error => {
    const status = error.response?.status;
    
    // 2. 智能分类
    if (status === 401) {
      // 自动恢复：Token刷新
      return await refreshToken();
    }
    
    if (status === 500) {
      // 自动恢复：重试机制
      return await retryWithBackoff(error);
    }
    
    // 3. 用户通知
    if (status === 404) {
      message.error('资源不存在');  // 状态码提示
    }
    
    // 4. 兜底处理
    message.error('请求失败，请稍后重试');
    return Promise.reject(error);
  }
);
```

#### 第3层：技术亮点（展示思考深度）

如果面试官问"为什么要这样设计"：

> "1. **基于类而非函数**：类可以更好地维护状态（如 `isRefreshing`、`retryQueue`），也方便后续扩展
> 2. **TypeScript 泛型**：提供类型安全，避免 `any` 带来的维护问题
> 3. **请求队列**：解决了并发刷新导致的重复请求问题，避免 Token 冲突
> 4. **指数退避**：相比固定延迟重试，更合理，减少服务器压力
> 5. **配置扩展**：通过 `BizAxiosRequestConfig` 扩展类型，添加 `skipErrorHandler` 等自定义配置"

---

### 🎯 面试官可能追问的问题

#### Q1：为什么不用简单的函数封装？

> "函数封装难以维护状态，比如刷新 Token 时的 `isRefreshing` 标志和请求队列。类可以更清晰地组织这些状态，并且符合面向对象的设计思想，后续也容易扩展功能（比如添加请求日志、性能监控等）。"

#### Q2：401 刷新时用户会感知到吗？

> "不会。我们使用请求队列机制，所有并发请求都会被挂起等待新 Token，刷新成功后自动重放。整个过程对业务代码是透明的，用户无感知。只有当刷新失败时才会跳转登录页。"

#### Q3：如何避免刷新接口无限循环？

> "我通过 `_retry` 标志位防止无限重试，每个请求只尝试刷新一次。如果刷新接口本身也返回 401，说明 refresh_token 也过期了，这时会直接清除 Token 并跳转登录页。"

#### Q4：指数退避的延迟时间是怎么计算的？

> "使用 `Math.pow(2, retryCount) * 1000`，第1次重试等1秒，第2次等2秒，第3次等4秒。这种几何级数增长既能给服务器恢复时间，又不会让用户等太久。"

#### Q5：Mock 数据是怎么实现的？

> "在 Demo 中我内置了一个 mock adapter，拦截 `/api/me` 请求。通过检查 `access_token_exp` 判断 Token 是否过期，过期则抛出 401 错误，成功则返回用户信息。真实项目中会替换为后端接口。"

---

### 📊 代码结构总结

```
Request 类
├── constructor
│   ├── 创建 Axios 实例
│   ├── 配置基础参数（baseURL、timeout）
│   └── 设置拦截器
├── setupInterceptors
│   ├── 请求拦截器（注入 Token）
│   └── 响应拦截器（401刷新、500重试、错误处理）
├── handleNetworkError
│   └── 指数退避重试逻辑
├── refreshAccessToken
│   └── 调用刷新接口
├── handleAuthError
│   └── 清除 Token，跳转登录
├── handleError
│   └── 全局错误提示
└── 便捷方法
    ├── request<T>()
    ├── get<T>()
    ├── post<T>()
    ├── put<T>()
    └── delete<T>()
```

---

### 💡 记忆口诀

**类封装 + 泛型，拦截器注入Token**
**401 自动刷新，请求队列不并发**
**500 指数退避，全局错误统一看**
**TypeScript 保类型，业务调用更简单**

---

### 🔗 相关代码位置

- 核心实现：[request.ts](src/utils/request.ts#L1-L327)
- 请求拦截器：[request.ts#L48-L58](src/utils/request.ts#L48-L58)
- 响应拦截器：[request.ts#L60-L118](src/utils/request.ts#L60-L118)
- 重试机制：[request.ts#L120-L148](src/utils/request.ts#L120-L148)
- 类型定义：[request.ts#L7-L8](src/utils/request.ts#L7-L8)

---

### 🚀 下一步学习

建议结合以下文档深入理解：

[Axios 类封装](9-axios-class-based-encapsulation)
[双Token认证](10-dual-token-authentication-access-refresh-token)
[请求队列管理](12-request-queue-management-for-token-refresh)
[指数退避重试](11-exponential-backoff-retry-mechanism)




我们的 Request 类封装了四个核心功能（见 request.ts#L12-L19L12-L19）：

1. **请求拦截器**：自动注入 Token，处理请求头配置
2. **响应拦截器**：统一错误处理、Token 刷新
3. **指数退避重试**：对 500 错误自动重试，延迟为 `2^retryCount * 1000ms`（request.ts#L108-L117L108-L117）
4. .
5. **全局错误提示**：通过 Antd message 统一展示错误信息