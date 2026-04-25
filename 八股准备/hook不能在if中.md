关键词 *Hook 是以链表形式挂在对应的 Fiber 节点上   React 在每次渲染固定顺序读取   会导致某些渲染分支不执行 Hook  破坏调用顺序   状态错乱或崩溃。 

因为 React 在每次渲染时，会按照固定顺序去读取和匹配 Hook。  
Hook 是以链表形式挂在对应的 Fiber 节点上，React 并不会根据 Hook 的名字或类型来识别，而是靠“调用顺序”来逐个取用。如果把 Hook 写在 if/for 或条件中，会导致某些渲染分支不执行 Hook，破坏调用顺序，React 将取错 Hook 节点，并导致状态错乱或崩溃。


理解
**同一个 Fiber 节点的 Hook 之间是用 `next` 指针连接起来，形成链表**

useState 的状态就存在 Hook 节点（memoizedState）里。  
React 每次渲染都是按 Hook 调用顺序去依次读取 Hook 节点的状态。  
如果条件渲染导致 Hook 调用次数或顺序变化，React 就会读取错节点，导致状态错乱。