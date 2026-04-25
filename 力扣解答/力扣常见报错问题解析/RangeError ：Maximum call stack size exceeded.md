是说 我们递归函数无限递归了，导致调用栈爆了（死递归）

报错中的：

Line 9: Char 16 in dfs  
Line 9: Char 16 in dfs  
Line 9: Char 16 in dfs

重复出现说明在同一个位置重复递归 → **死循环**。