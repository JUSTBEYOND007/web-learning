使用方法
```js
Array.from(arrayLike, mapFunc)
```

arrayLike：只需要提供一个 length
mapFunc：对每个位置调用一次函数

不是必须是数组！！
例如  我们来创建和board一样的true/false数组方法
这里Array(board[0].length).fill(false)
等于new  Array(board[0].length).fill(false)  board[0].length是大小 fill填值
```js
const used=Array.from({length:board.length},()=>Array(board[0].length).fill(false))
```