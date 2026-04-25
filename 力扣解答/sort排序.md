```js
candidates.sort((a,b)=>a-b)
```
- `sort()` 默认是 **原地排序**，会改变原数组。
    
- 返回值是原数组本身，可以链式调用。
    
- 如果需要保留原数组，先用 `slice()` 或 `[...arr]` 复制一份


注意： candidates.sort((a,b)=> return a-b)会报错
- 箭头函数里，如果用了 `return`，就必须用 `{}` 包裹起来。
    
- 单行箭头函数 **不能直接写 `return`**。