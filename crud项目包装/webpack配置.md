webpack 打包体积怎么优化
webpack 打包体积优化
●
首先使用 webpack-bundle-analyzer 可视化打包结果的每个文件大小及其依赖，并进行针对性优化，在实际操作过程中，我通常会去关注如下几点：
○
同种功能库的重复引用
○
组件懒加载
一些不是首屏渲染的组件可以使用懒加载，减小入口文件体积，以提高首屏的渲染速度。最常见的是路由页面及弹窗组件。webpack 提供的 import()可以将组件异步引入。该组件模块将被打包成一个单独的文件。
○
代码拆分（splitChunks）
webpack 默认是将所有代码都打包到入口文件中，很容易让入口文件的体积变得非常大，不利于加载。通过spiltChunks可以将代码拆分成多个文件，有效利用浏览器并发请求

**常用 Loader + Plugin 列表**

| Loader                                    | 用途                                   | 示例                                                                                   |
| ----------------------------------------- | ------------------------------------ | ------------------------------------------------------------------------------------ |
| **babel-loader**                          | 将 ES6+/JSX 转换为浏览器可识别 JS              | `test: /\.js$/, use: 'babel-loader'`                                                 |
| **ts-loader / awesome-typescript-loader** | TypeScript 编译                        | `test: /\.ts$/, use: 'ts-loader'`                                                    |
| **style-loader**                          | 将 CSS 注入到 DOM                        | `use: ['style-loader', 'css-loader']`                                                |
| **css-loader**                            | 解析 CSS `@import` 和 `url()`           | `use: 'css-loader'`                                                                  |
| **sass-loader / less-loader**             | 编译 SCSS/LESS 为 CSS                   | `use: ['style-loader', 'css-loader', 'sass-loader']`                                 |
| **file-loader / url-loader**              | 处理图片、字体资源                            | `test: /.(png                                                                        |
| **url-loader**                            | 小文件转 base64，大文件 fallback file-loader | `limit: 8 * 1024`                                                                    |
| **html-loader**                           | 解析 HTML 中的资源路径                       | `use: 'html-loader'`                                                                 |
| **source-map-loader**                     | 提取 JS 源码的 source map                 | `enforce: 'pre', use: 'source-map-loader'`                                           |
| Plugin                                    | 用途                                   | 示例                                                                                   |
| **HtmlWebpackPlugin**                     | 自动生成 HTML 并注入 JS/CSS                 | `new HtmlWebpackPlugin({ template: './index.html' })`                                |
| **MiniCssExtractPlugin**                  | 将 CSS 提取成单独文件（生产优化）                  | `new MiniCssExtractPlugin({ filename: '[name].[contenthash].css' })`                 |
| **CleanWebpackPlugin**                    | 构建前清理 dist 目录                        | `new CleanWebpackPlugin()`                                                           |
| **DefinePlugin**                          | 定义全局常量，区分开发/生产环境                     | `new webpack.DefinePlugin({ 'process.env.NODE_ENV': JSON.stringify('production') })` |
| **TerserPlugin**                          | JS 压缩                                | 配合 `optimization.minimizer`                                                          |
| **CopyWebpackPlugin**                     | 拷贝静态资源到输出目录                          | `new CopyWebpackPlugin({ patterns: [{ from: 'static', to: 'dist/static' }] })`       |
| **BundleAnalyzerPlugin**                  | 可视化分析打包体积                            | `new BundleAnalyzerPlugin()`                                                         |
| **ProvidePlugin**                         | 自动加载模块（如 jQuery）                     | `new webpack.ProvidePlugin({ $: 'jquery' })`                                         |
| **HotModuleReplacementPlugin**            | 开发环境热更新                              | `new webpack.HotModuleReplacementPlugin()`                                           |