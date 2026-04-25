*浏览器绘制  同步/异步*

useEffect 和 useLayoutEffect 的核心区别在于执行时机：useEffect 在浏览器绘制后异步执行，useLayoutEffect 在绘制前同步执行，useLayoutEffect会阻塞渲染。