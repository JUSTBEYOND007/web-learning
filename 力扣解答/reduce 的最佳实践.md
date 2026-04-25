> **只要做的是“累积类”操作（求和/求积/求最大/求最小），一定要写初始值。**
previousValue的初始化值就是initialValue
接受两个参数 回调函数，和初始化值
例如：

```js
nums.reduce((pre, cur) => pre + cur, 0);  
nums.reduce((pre, cur) => pre * cur, 1);  
nums.reduce((pre, cur) => Math.max(pre, cur), -Infinity);  
nums.reduce((pre, cur) => Math.min(pre, cur), Infinity);
```

例如求和的写法：
```js
let sum = nums.reduce((pre,value)=> pre+value, 0)
```



