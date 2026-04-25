## typeof：判断基本类型的“主力军”

`typeof` 是专门用来检测**原始数据类型**（基本类型）的。

- **适用范围**：`number`, `string`, `boolean`, `undefined`, `symbol`, `bigint` 以及 `function`（虽然函数是对象，但 `typeof` 会返回 "function"）。
    
- **局限性**：它无法区分 `null`（会返回 `"object"`，这是 JS 的一个历史遗留 Bug）和普通的 `Object`。


## 2. instanceof：基于原型链的“身份鉴定”

`instanceof` 运算符用于检测构造函数的 `prototype` 属性是否出现在某个实例对象的原型链上。

- **适用范围**：判断一个变量是否属于某个**类（Class）或构造函数**的实例。
    
- **对基本类型的表现**：**无法直接判断字面量形式的基本类型**。