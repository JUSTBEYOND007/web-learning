function curry(fn, ...args)
这里args是一个数组

但是function curry(fn, args)  `args就只表示第二个参数`
例如  function curry(fn, args)

调用：

curry(add,1,2)

得到：

fn = add  
args = 1

第二个参数只会接收：

1

`2` 会被丢掉