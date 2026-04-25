## 1️⃣ 使用 `toString()` 方法

`const arr = [1, 2, 3, 4]; const str = arr.toString(); console.log(str); // "1,2,3,4"`

- 默认使用 **逗号 `,`** 作为分隔符。
    
- 简单快速，但不能自定义分隔符。
    

---

## 2️⃣ 使用 `join()` 方法

`const arr = [1, 2, 3, 4]; const str1 = arr.join();       // 默认逗号分隔 console.log(str1); // "1,2,3,4"  const str2 = arr.join('-');    // 自定义分隔符 console.log(str2); // "1-2-3-4"  const str3 = arr.join('');     // 没有分隔符 console.log(str3); // "1234"`

- `join()` 更灵活，可自定义连接符。
    
- 常用于生成字符串输出或 CSV 格式。