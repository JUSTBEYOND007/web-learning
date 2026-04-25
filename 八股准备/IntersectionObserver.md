好的，我给你详细介绍一下 **`IntersectionObserver`**，这是一个现代浏览器提供的 API，用于监听一个元素与其父容器或视口（viewport）的交叉情况。相比传统的滚动事件监听，它更加高效，性能开销小。下面我分点说明：

---

## 1️⃣ 基本概念

**IntersectionObserver** 用来观察某个目标元素（target）是否进入另一个元素（root，通常是视口）内，以及交叉的比例。

- **目标元素（target）**：你想要观察的 DOM 元素。
    
- **根元素（root）**：判断 target 是否可见的参考元素，默认是 `viewport`。
    
- **交叉比例（threshold）**：目标元素进入 root 的可见比例，可以是 0~1 的数字或数组。