**关键词  带#不带# 来分割  SEO是否友好   URL是否美观 **
**URL 格式**：`https://example.com/#/login`、`https://example.com/#/user/123`

- **原理**：利用 URL 的 `#` 后面的部分（hash fragment）作为路由路径
- **特点**：`#` 后面的内容改变不会触发页面刷新
- **优点**：
    - 兼容性好（IE8+ 都支持）
    - 部署简单，不需要服务器配置
    - 避免 404 问题（服务器看不到 hash 后的内容）
- **缺点**：
    - URL 不美观（带 `#`）
    - 不利于 SEO（搜索引擎通常不索引 hash 后的内容）
    - 不能利用浏览器前进/后退的历史记录栈

### History 模式（HTML5 History API）

**URL 格式**：`https://example.com/login`、`https://example.com/user/123`

- **原理**：使用 `history.pushState()` 和 `history.replaceState()` 修改 URL
- **特点**：URL 看起来像正常的路径，不带 `#`
- **优点**：
    - URL 美观、语义化
    - SEO 友好
    - 可以更好地利用浏览器历史记录
- **缺点**：
    - **需要服务器配置**（关键！）
    - 用户刷新页面会向服务器请求完整路径，需要服务器返回同一个 index.html
    - 兼容性稍差（IE10+）