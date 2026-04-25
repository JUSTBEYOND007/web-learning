Babel 的作用是解决 JavaScript 的兼容性问题，让我们可以放心使用最新的 JS 语法、TS、JSX 等语言特性。

它的核心做法是把代码解析成 AST，再经过插件对 AST 做语法转换，最后生成兼容旧环境的 JS。  
同时 Babel 还能按需注入 polyfill，让像 Promise、Map 这类 API 在旧浏览器也能使用。