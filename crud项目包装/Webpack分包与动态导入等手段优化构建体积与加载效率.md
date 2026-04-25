
在Webpack 中使用splitChunks进行了三块vendor分包 分别是react-vendor \ antd-vendor 和其他第三方库分包 包括（axios和zustand）等 对于核心业务 我们根据路由懒加载 来分包 来回答我们使用webpack'分包 实行性能优化


在这个 Demo 里（虽然实际用的是 Vite，但分包策略和 Webpack splitChunks + import() 是等价的），我们主要对这几类东西做了分包：


*- 按第三方库 / 业务代码 / UI 库分包*

我们按照依赖类型分为三个vendor包：

- **vendor-react**：React核心库（react、react-dom、react-router-dom）
- **vendor-antd**：UI组件库（antd、@ant-design/icons）
- **vendor-others**：其他第三方库（axios、zustand等）

### 1. 第三方库（vendor 分包）
Webpack 中使用splitChunks进行了三块vendor分包 

在 vite.config.ts 里通过 manualChunks 做了三块 vendor 分包（Webpack 里就是 optimization.splitChunks.cacheGroups 的思路）：

- vendor-react

- 包含：react、react-dom、react-router-dom 等 React 栈核心依赖

- 特点：变化频率低 → 适合长期缓存

- vendor-antd

- 包含：antd、@ant-design/icons

- 特点：体积最大，单独抽出来，浏览器可以单独缓存 UI 库

- vendor-others

- 包含：axios、zustand 等其余三方依赖

- 特点：中等体积，和业务无关的工具库都放在这里

> 在 Webpack 里对应的是：

> react-core、antd、vendors 这类 cacheGroups。

### 2. 业务代码（按路由懒加载分包）

在 src/App.tsx 中使用了 React.lazy(() => import('./pages/XXX'))：

- 拆成独立 chunk 的页面：

- Login 页面

- Dashboard 仪表盘

- WorkflowDesigner 工作流设计器

- ApprovalList 审批列表

- AdminPanel 管理后台

每个页面在构建后都会生成一个独立的业务 chunk（Webpack 下会看到类似 page-dashboard.xxx.js 这种），只有在访问对应路由时才按需加载，降低首屏 JS 体积。

根据配置自动把**公共模块、第三方库、UI 库**拆分为独立 chunk



我们是怎么判断包体积过大，需要分包???

1. **通过打包分析工具**
    

- **Webpack Bundle Analyzer**（最常用）
    
    - 可视化显示每个 chunk 的大小和占比
        
    - 轻松看到哪些模块体积最大
        
    
    `npm install --save-dev webpack-bundle-analyzer`
    
    `const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');  plugins: [   new BundleAnalyzerPlugin() ]`


### 1️⃣ SplitChunksPlugin（最核心）

- Webpack 默认开启 `optimization.splitChunks`
    
- 根据配置自动把**公共模块、第三方库、UI 库**拆分为独立 chunk
    

`optimization: {   splitChunks: {     chunks: 'all', // async + sync 都考虑     cacheGroups: {       react: {         test: /[\\/]node_modules[\\/](react|react-dom)/,         name: 'react-core',         priority: 100,       },       vendors: {         test: /[\\/]node_modules[\\/]/,         name: 'vendors',         priority: 10,       },     },   }, }`

- **逻辑**：
    
    1. 检查多个入口是否引用同一个模块
        
    2. 如果超过 `minSize` 或 `minChunks` → 拆分
        
    3. 给 chunk 一个独立文件名 → 浏览器缓存
        

---

### 2️⃣ 动态导入（按需分包）

- 使用 `import()` 动态加载模块
    

`// 路由懒加载示例 const FeatureA = React.lazy(() => import('./FeatureA'));`

- Webpack 会把 `FeatureA` 单独打包成一个 chunk
    
- 首屏不下载这个 chunk，用户触发时再加载






你的回答 **方向是对的，但还不够“面试官满意”**。  
它解决了“分什么包”，但没有讲到 **“怎么分”、为什么这样分、Webpack 的原理配置”等关键点**。

下面我来告诉你：  
✔ 哪些地方需要补强  
✔ 如何说到面试官“哇，这孩子懂 webpack” 的程度

---

# ✅ 你现在的回答（不错但不完整）

你说：

- 按第三方库 / 业务代码 / UI 库分包
    
- 第三方库：react、zustand、antd 拆包
    
- 业务代码按路由懒加载拆包
    

这是 **方向正确** 的，但有两个问题：


# ✅ 最优回答（面试直接拿分）

**Webpack 分包主要有两个方向：手动分包和自动分包。**

---

## **💡 1. 第三方库分包（自动）**

我会通过 `splitChunks.cacheGroups` 把框架层的依赖拆成独立 chunk，例如：

- React、React DOM 拆成 `react-vendor`
    
- 状态管理库（zustand）也拆出去
    
- UI 库（比如 Ant Design）由于体积更大，单独拆成 `antd-vendor`
    

**好处：**

- 浏览器可以对大库做 **长期缓存**
    
- 第三方库变化频率低，不会跟业务代码一起重复下载
    

---

## **💡 2. 公共模块分包（自动）**

业务里多个页面用到的工具函数/组件，会被 Webpack 自动拆到：

```
common.chunk.js
```

减少重复打包体积。

---

## **💡 3. 业务代码按路由懒加载（手动）**

业务代码会按路由进行拆分：

```js
const Home = () => import('./pages/Home')
```

这样 Webpack 会为每个路由生成一个 chunk。

**好处：**

- 首屏 bundle 更小
    
- 不访问的页面不会加载
    
- 加快首屏渲染速度
    

---

## **💡 4. UI 库单独拆分（自动）**

UI 组件库（例如 Ant Design）体积大、且与 React 不完全同步更新，所以我会把它拆成独立 chunk：

```
antd.chunk.js
```

减少与主包的耦合，提高缓存命中率。

---

## 🔧（可加分）Webpack 实现方式

```js
optimization: {
  splitChunks: {
    chunks: "all",
    cacheGroups: {
      reactVendor: {
        test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
        name: "react-vendor",
        chunks: "all",
      },
      uiVendor: {
        test: /[\\/]node_modules[\\/](@ant-design|antd)[\\/]/,
        name: "ui-vendor",
        chunks: "all",
      },
      common: {
        minChunks: 2,
        name: "common",
        chunks: "all",
      },
    },
  },
}
```

