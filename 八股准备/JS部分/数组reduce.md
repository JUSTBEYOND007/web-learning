三种回调方式

第一种    第一个参数是数组第一个索引的值 返回是数字类型
```js
        const sum=arr.reduce((sum,value)=>{//

            sum+=value

            return sum

        })
```

第二种    第一个参数是传入第二个参数的值 这里为10 返回也是数字类型
```js
        const sum=arr.reduce((sum,value)=>{

            sum+=value

            return sum

        },10)
```

第三种    第一个参数是传入第二个参数的值 这里为空数组   返回也是数组类型
```js
const result = arr.reduce((result, value) => {
    if (!result.includes(value)) {
        result.push(value)
    }
    return result   // 这一句必须写在外面！
}, [])
```