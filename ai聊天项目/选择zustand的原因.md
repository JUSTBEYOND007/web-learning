#### 精简版本（1 分钟版本）：[](https://zread.ai/JUSTBEYOND007/ai-chat#%E7%B2%BE%E7%AE%80%E7%89%88%E6%9C%AC1-%E5%88%86%E9%92%9F%E7%89%88%E6%9C%AC)

> "主要考虑了三点：
> 
> 1. **开发效率**：Zustand 的样板代码比 Redux 少 80%，创建 Store 只需要一步，而 Redux 需要定义 Action、Reducer、Selector。
>     
> 2. **性能优化**：我们的 AI 流式回复场景每秒有 100+ 次状态更新，Context API 会导致所有消费者（消费该 Context 的组件）重渲染，而 Zustand 提供了细粒度订阅，只有订阅的部分变化才重渲染。
>     
> 3. **业务场景匹配**：项目有 5 个独立 Store，且需要跨 store 通信（聊天 store 读取会话 store 的 ID），Zustand 天然支持多 Store 和跨 store 调用，比 Redux 的复杂配置简单得多。
>     
> 
> 此外，Zustand 的打包体积只有 1KB（Redux 需要 20KB+），并且内置了 DevTools 和持久化功能，开箱即用。"


```js
const StoreContext = createContext()

function App(){
  const store = {
    count:0,
    user:"tom"
  }

  return (
    <StoreContext.Provider value={store}>
        <ChildA/>
        <ChildB/>
    </StoreContext.Provider>
  )
}
React Context 


/*
本质只是一个跨组件传值的机制，它默认是对整个 value 进行订阅，当 Provider 的 value 发生变化时，所有使用 useContext 的组件都会重新渲染，无法做到对 state 的细粒度订阅。而像 Zustand 这样的状态管理库支持 selector 订阅，可以只监听 store 中的某一部分状态，从而避免不必要的组件重新渲染，在大型应用中性能更好，所以很多项目更倾向使用 Zustand。
*/
```