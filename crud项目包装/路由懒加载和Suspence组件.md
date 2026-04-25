在项目中我使用 React.lazy 做路由级代码分割，并在路由层统一使用 Suspense 进行包裹，通过 fallback 提供统一的 Loading 组件。

Suspence组件组件作用:这样可以对懒加载组件的加载状态进行统一管理，避免在每个页面单独处理 loading，同时保证页面加载体验一致，也减少了重复代码。
