固定模板
```js
const fs=require('fs')
let input=fs.readFileSync(0,"utf8").trim().split(/\s+/).map(Number)
//这里input 就是对应的包含所有数据的数组  注意如果不加.map(Number)这里是每一项是字符串
console.log(input[1])//用console.log来输出
```

我们在控制台
执行node  文件名.js  eg `node leetcode.js`
然后输入
输入完后我们enter执行  ctrl+z  
然后我们点击enter 
就会输出结果

![Pasted image 20260305221447](../assets/images/Pasted%20image%2020260305221447.png)


split(/\s+/) 一次性把所有数字拆成 **数组**