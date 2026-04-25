# 1 默认盒子模型（content-box）

默认值：

box-sizing: content-box;

计算方式：

元素真实宽度 =  
width + padding + border

例如：

div {  
  width: 200px;  
  padding: 20px;  
  border: 10px solid black;  
}

真实宽度：

200 + 20*2 + 10*2 = 260px

---

# 2️⃣ border-box（常用）

box-sizing: border-box;

计算方式：

width = content + padding + border

也就是：

真实宽度 = width

例如：

div {  
  width: 200px;  
  padding: 20px;  
  border: 10px solid black;  
  box-sizing: border-box;  
}

content 实际宽度变成：

200 - 40 - 20 = 140px

但 **元素整体宽度仍然是 200px**。