因为 JS 是单线程的，而且 setInterval / setTimeout 是宏任务，需要等待同步代码和微任务执行完之后，才会进入事件循环被执行。

setTimeout(fn, 1000)

意思是：
1秒后执行
其实真正含义是：
至少 1 秒后才可能执行