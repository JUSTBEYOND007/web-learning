**关键词    非持久连接/持久连接  更多的缓存控制策略（etag）  加了大量新状态码（206  断点续传）  新增请求方式更多  **       


连接方面，http1.0 默认使用非持久连接，而 http1.1 默认使用持久连接。http1.1 通过使用持久连接来使多个 http 请求复用同一个 TCP 连接，以此来避免使用非持久连接时每次需要建立连接的时延。

缓存方面，在 http1.0 中主要使用 header 里的 If-Modified-Since、Expires 来做为缓存判断的标准，http1.1 则引入了更多的缓存控制策略，例如 Etag、If-Unmodified-Since、If-Match、If-None-Match 等更多可供选择的缓存头来控制缓存策略。

## **4. 状态码更丰富：1.1 增加了大量新状态码**

HTTP/1.1 比 1.0 多出了如下重要状态码：

- **206 Partial Content**（断点续传）
    
- **409 Conflict**
    
- **410 Gone**
    
- **100 Continue**（大文件上传优化）

请求方式更多：1.1 新增 PUT / PATCH / DELETE / OPTIONS...



## **6. Host 头：1.1 要求必须携带 Host**

### 为什么？

因为 1 台服务器上可以部署多个网站（虚拟主机），需要区分域名。