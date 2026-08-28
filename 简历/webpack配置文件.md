> webpack.config.ts
> **Webpack 实际开发中，一般不会只用一个配置文件，而是拆成：**

在项目中我使用 webpack 做构建，主要配置包括：

- entry 和 output 定义入口和输出
    
- loader 处理 TS、JSX、CSS 和静态资源
    
- plugin 用于生成 HTML、注入环境变量等
    
- resolve 配置别名优化路径
    
- optimization 通过 splitChunks 和 Tree Shaking 做性能优化  
    同时结合 devServer 提升开发体验。
    

![Pasted image 20260120213052](../assets/images/Pasted%20image%2020260120213052.png)



```js webpack.config.ts
import path from 'path';
import webpack, { Configuration } from 'webpack';
import 'webpack-dev-server'; // 引入类型扩展
import HtmlWebpackPlugin from 'html-webpack-plugin';
import MiniCssExtractPlugin from 'mini-css-extract-plugin';
import CssMinimizerPlugin from 'css-minimizer-webpack-plugin';
import TerserPlugin from 'terser-webpack-plugin';
import { CleanWebpackPlugin } from 'clean-webpack-plugin';
import ForkTsCheckerWebpackPlugin from 'fork-ts-checker-webpack-plugin';
import ImageMinimizerPlugin from 'image-minimizer-webpack-plugin';
import Dotenv from 'dotenv-webpack';
// const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer'); // 可选：分析包体积时开启

// 环境变量判断
const isProduction = process.env.NODE_ENV === 'production';

// 文档中提到的 CDN 地址 (PDF Page 14)
const CDN_HOST = isProduction ? 'https://cdn.yourdomain.com' : '/';

const config: Configuration = {
  mode: isProduction ? 'production' : 'development',
  
  // 源码映射：生产环境不生成或生成 hidden-source-map 以提升安全性和构建速度
  devtool: isProduction ? false : 'eval-cheap-module-source-map',

  entry: './src/index.tsx', // 入口文件

  output: {
    path: path.resolve(__dirname, 'dist'),
    // 文件名带 hash，利用浏览器缓存 (PDF Page 13/14)
    filename: 'static/js/[name].[contenthash:8].js',
    chunkFilename: 'static/js/[name].[contenthash:8].chunk.js',
    // 资源引用的公共路径，这里配置 CDN
    publicPath: CDN_HOST,
    assetModuleFilename: 'static/media/[name].[hash:8][ext]',
  },

  resolve: {
    extensions: ['.tsx', '.ts', '.js', '.jsx', '.json'],
    // 路径别名，配合 TSConfig paths
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
  },

  cache: {
    type: 'filesystem', // 开启文件系统缓存，加快二次构建速度
  },

  module: {
    rules: [
      // 1. TypeScript & React 处理
      {
        test: /\.(ts|js)x?$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'babel-loader',
            options: {
              presets: [
                '@babel/preset-env',
                '@babel/preset-react',
                '@babel/preset-typescript',
              ],
              // 开启缓存
              cacheDirectory: true,
            },
          },
        ],
      },
      
      // 2. CSS/Less 处理 (Ant Design 常用)
      {
        test: /\.(css|less)$/,
        use: [
          isProduction ? MiniCssExtractPlugin.loader : 'style-loader',
          'css-loader',
          'postcss-loader', // 自动添加浏览器前缀
          {
            loader: 'less-loader',
            options: {
              lessOptions: {
                javascriptEnabled: true, // AntD 需要
              },
            },
          },
        ],
      },

      // 3. 图片/字体资源处理 (Webpack 5 Asset Modules)
      {
        test: /\.(png|jpg|jpeg|gif|svg|webp)$/i,
        type: 'asset',
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于 10kb 转 base64
          },
        },
      },
    ],
  },

  plugins: [
    new CleanWebpackPlugin(), // 每次构建清理 dist
    
    new HtmlWebpackPlugin({
      template: './public/index.html',
      inject: true,
      minify: isProduction ? {
        removeComments: true,
        collapseWhitespace: true,
        removeRedundantAttributes: true,
      } : false,
    }),

    // 环境变量注入
    new Dotenv(),
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
    }),

    // 独立的 TS 类型检查进程，不阻塞构建
    new ForkTsCheckerWebpackPlugin({
      async: !isProduction,
    }),
    
    // 生产环境提取 CSS
    ...(isProduction ? [
      new MiniCssExtractPlugin({
        filename: 'static/css/[name].[contenthash:8].css',
        chunkFilename: 'static/css/[name].[contenthash:8].chunk.css',
      }),
    ] : []),
    
    // 可选：构建分析
    // new BundleAnalyzerPlugin(),
  ],

  // -----------------------------------------------------------
  // 核心优化配置 (对应 PDF Page 12-15)
  // -----------------------------------------------------------
  optimization: {
    minimize: isProduction,
    minimizer: [
      // JS 压缩
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true, // 生产环境移除 console
          },
        },
      }),
      // CSS 压缩
      new CssMinimizerPlugin(),
      
      // 图片压缩 (对应 PDF Page 13: ImageminWebp + 压缩)
      // 注意：image-webpack-loader 是 loader，这里演示 Webpack 5 推荐的插件式写法
      new ImageMinimizerPlugin({
        minimizer: {
          implementation: ImageMinimizerPlugin.imageminMinify,
          options: {
            plugins: [
              ['imagemin-gifsicle', { interlaced: true }],
              ['imagemin-mozjpeg', { quality: 75 }], // 对应文档 method: 6 概念
              ['imagemin-pngquant', { quality: [0.6, 0.8] }],
              ['imagemin-svgo', {
                plugins: [{ name: 'preset-default', params: { overrides: { removeViewBox: false } } }],
              }],
            ],
          },
        },
        // 生成 WebP (对应文档：优先加载 WebP)
        generator: [
          {
            preset: 'webp',
            implementation: ImageMinimizerPlugin.imageminGenerate,
            options: {
              plugins: ['imagemin-webp'],
            },
          },
        ],
      }),
    ],

    // 代码分割策略 (严格对应 PDF Page 15 代码块)
    splitChunks: {
      chunks: 'all',
      minSize: 20000,
      cacheGroups: {
        // 拆分 React 全家桶
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom|react-router|zustand)[\\/]/,
          name: 'react-core',
          priority: 100, // 优先级最高
          chunks: 'all',
        },
        // 拆分工具库 Lodash / Axios
        libs: {
          test: /[\\/]node_modules[\\/](lodash|axios)[\\/]/,
          name: 'libs',
          priority: 90,
          chunks: 'all',
        },
        // Ant Design 单独拆分（防止 UI 库过大）
        antd: {
          test: /[\\/]node_modules[\\/](antd|@ant-design)[\\/]/,
          name: 'antd',
          priority: 80,
          chunks: 'all',
        },
        // 其他第三方库
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10,
          chunks: 'all',
        },
      },
    },
    runtimeChunk: 'single', // 将 manifest 提取出来，避免 hash 频繁变动
  },

  // 开发服务器配置
  devServer: {
    historyApiFallback: true, // React Router 刷新不 404
    port: 3000,
    hot: true,
    open: true,
    compress: true,
    // 代理接口，解决跨域
    proxy: [
      {
        context: ['/api'],
        target: 'http://localhost:8080',
        changeOrigin: true,
        pathRewrite: { '^/api': '' },
      },
    ],
  },
};

export default config;
```

<font color='purple'>解释上述代码</font>

好的，我来帮你**系统性地拆解这个 Webpack 配置文件**，让你从面试和实战角度都能理解。我们按模块和逻辑顺序讲。

---

## **一、环境判断**

```ts
const isProduction = process.env.NODE_ENV === 'production';
const CDN_HOST = isProduction ? 'https://cdn.yourdomain.com' : '/';
```

- `isProduction`：判断当前是开发还是生产环境
    
- `CDN_HOST`：生产环境资源通过 CDN 加载，开发环境本地加载
    

💡 面试讲法：

> 根据环境变量动态切换配置，实现开发与生产优化分离。

---

## **二、mode 与 devtool**

```ts
mode: isProduction ? 'production' : 'development',
devtool: isProduction ? false : 'eval-cheap-module-source-map',
```

- `mode`：
    
    - `production` → 自动开启压缩、Tree-shaking
        
    - `development` → 开启调试友好的配置
        
- `devtool`：
    
    - 开发环境 → 快速生成 SourceMap，方便调试
        
    - 生产环境 → 不生成 SourceMap 或用隐藏的 source map，提升安全性
        

---

## **三、入口和输出（entry / output）**

```ts
entry: './src/index.tsx',
output: {
  path: path.resolve(__dirname, 'dist'),
  filename: 'static/js/[name].[contenthash:8].js',
  chunkFilename: 'static/js/[name].[contenthash:8].chunk.js',
  publicPath: CDN_HOST,
  assetModuleFilename: 'static/media/[name].[hash:8][ext]',
},
```

- `entry`：项目主入口
    
- `filename` & `chunkFilename`：
    
    - 使用 `[contenthash]` 实现**缓存优化**
        
- `publicPath`：
    
    - 配置资源加载路径（CDN 或本地）
        
- `assetModuleFilename`：
    
    - 图片、字体等资源的输出规则
        

💡 面试讲法：

> 输出文件名使用 contenthash，实现缓存与更新分离，CDN 加速加载。

---

## **四、模块解析（resolve / module.rules）**

### 1️⃣ resolve

```ts
resolve: {
  extensions: ['.tsx', '.ts', '.js', '.jsx', '.json'],
  alias: { '@': path.resolve(__dirname, 'src') },
},
```

- 自动解析文件后缀，导入时可省略
    
- alias 设置路径别名，提升可维护性
    

### 2️⃣ module.rules

- **TS/JSX**：
    

```ts
{
  test: /\.(ts|js)x?$/,
  exclude: /node_modules/,
  use: [{ loader: 'babel-loader', options: { presets: ['@babel/preset-env','@babel/preset-react','@babel/preset-typescript'], cacheDirectory: true } }],
}
```

- Babel 处理 TS 和 React 代码
    
- `cacheDirectory` 提升二次构建速度
    
- **CSS / Less**：
    

```ts
{
  test: /\.(css|less)$/,
  use: [
    isProduction ? MiniCssExtractPlugin.loader : 'style-loader',
    'css-loader',
    'postcss-loader',
    { loader: 'less-loader', options: { lessOptions: { javascriptEnabled: true } } },
  ],
}
```

- 开发环境 → `style-loader` 内联样式
    
- 生产环境 → `MiniCssExtractPlugin` 提取 CSS
    
- `postcss-loader` → 自动添加浏览器前缀
    
- `less-loader` → 支持 Ant Design
    
- **图片 / 字体**：
    

```ts
{
  test: /\.(png|jpg|jpeg|gif|svg|webp)$/i,
  type: 'asset',
  parser: { dataUrlCondition: { maxSize: 10 * 1024 } },
}
```

- 小于 10KB 转 Base64
    
- 大文件独立输出，提高性能
    

---

## **五、插件（plugins）**

### 1️⃣ 构建优化相关

- `CleanWebpackPlugin` → 每次构建清理 dist
    
- `HtmlWebpackPlugin` → 自动生成 HTML
    
- `MiniCssExtractPlugin` → 提取 CSS（生产）
    
- `ForkTsCheckerWebpackPlugin` → 独立进程类型检查
    
- `Dotenv` + `DefinePlugin` → 注入环境变量
    

### 2️⃣ 可选插件

- `BundleAnalyzerPlugin` → 分析打包体积
    
- `ImageMinimizerPlugin` → 压缩图片 & 生成 WebP
    

💡 面试讲法：

> 插件主要用于生成 HTML、优化资源、类型检查、环境变量注入和压缩优化。

---

## **六、优化策略（optimization）**

```ts
optimization: {
  minimize: isProduction,
  minimizer: [
    new TerserPlugin({ terserOptions: { compress: { drop_console: true } } }),
    new CssMinimizerPlugin(),
    new ImageMinimizerPlugin({...}),
  ],
  splitChunks: {
    chunks: 'all',
    minSize: 20000,
    cacheGroups: {
      react: {...},
      libs: {...},
      antd: {...},
      vendors: {...},
    },
  },
  runtimeChunk: 'single',
}
```

### 1️⃣ 压缩

- `TerserPlugin` → JS 压缩 & 移除 console
    
- `CssMinimizerPlugin` → CSS 压缩
    
- `ImageMinimizerPlugin` → 图片压缩 & WebP
    

### 2️⃣ 代码分割（SplitChunks）

- 将第三方库、React、Ant Design 等拆成独立 chunk
    
- 避免重复打包 → **复用 + 并行加载 + 缓存优化**
    

### 3️⃣ runtimeChunk

- 单独提取 manifest → 防止 hash 频繁变化
    

💡 面试讲法：

> 通过压缩 + 代码拆分 + runtimeChunk，实现**首屏加载快、缓存利用率高、按需加载**。

---

## **七、开发服务器配置（devServer）**

```ts
devServer: {
  historyApiFallback: true,
  port: 3000,
  hot: true,
  open: true,
  compress: true,
  proxy: [
    { context: ['/api'], target: 'http://localhost:8080', changeOrigin: true, pathRewrite: { '^/api': '' } },
  ],
}
```

- SPA 路由刷新不 404 → `historyApiFallback`
    
- 热更新 → `hot: true`
    
- 自动打开浏览器 → `open: true`
    
- gzip 压缩 → `compress: true`
    
- 解决跨域 → `proxy`
    

💡 面试讲法：

> 开发环境配置主要关注快速开发、调试和跨域。

---

## **八、总结（面试可讲）**

1. **模块化**：TS/JSX、CSS、图片独立处理
    
2. **插件**：HTML 生成、CSS 提取、类型检查、环境变量注入
    
3. **优化**：JS/CSS/Image 压缩 + 代码拆分 + runtimeChunk
    
4. **缓存**：`contenthash` + 公共 chunk → 浏览器缓存
    
5. **开发体验**：热更新 + SPA 支持 + proxy 跨域
    
6. **CDN**：生产环境静态资源通过 CDN 加速
    

> 这个配置文件可以支持**大型 React/TS 项目生产环境构建**，同时兼顾开发环境调试和性能优化。

