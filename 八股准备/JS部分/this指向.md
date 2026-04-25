1 全局上下文中：window或者global对象
2
function上下文中:
●
独立函数调用，this 指向全局对象
●
对象方法调用：调用该方法的对象
●
显式绑定：通过 call、apply、bind 指定 this
●
构造函数调用：新创建的对象实例
3
箭头函数：没有自己的 this，继承外层作用域的 this