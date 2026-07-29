#### React生命周期

**挂载**（`constructor`→`render`→`componentDidMount`）、**更新**（`render`→`componentDidUpdate`）、**卸载**（`componentWillUnmount`）。Hooks 时代用 `useEffect` 统一处理副作用，返回清理函数模拟**卸载**，依赖数组控制更新时机，类组件生命周期逐渐被函数组件取代

#### react的filber

Fiber 是 React 用链表重构的组件树结构，让渲染过程可中断、可恢复、可优先级调度，从而解决大数据量更新时的页面卡顿问题，双缓冲机制（两棵filber树）

1. **JSX 编译**：Babel 把 `<div>` 编译成 `React.createElement('div', ...)`
2. **创建虚拟 DOM**：`createElement` 返回 `{ type, props, children }` 对象树
3. **Render Phase（渲染阶段）**：
   - 从根节点开始，构建 **Fiber 树**（链表结构：child → sibling → parent）
   - 每个 Fiber 节点对应一个工作单元
   - 利用 `requestIdleCallback` 在浏览器空闲时执行，可中断
   - **Reconciliation（协调/Diff）**：对比新旧 Fiber 树
     - 同类型节点 → `UPDATE`（复用 DOM，更新 props）
     - 新节点 → `PLACEMENT`（创建新 DOM）
     - 旧节点多余 → `DELETION`（删除 DOM）
4. **Commit Phase（提交阶段）**：
   - 遍历 Fiber 树，根据 `effectTag` 执行实际 DOM 操作
   - 这个阶段**同步执行**，不可中断，确保 UI 一致性
5. **Hooks 执行**：函数组件在 Render Phase 执行，Hook 状态保存在 Fiber 节点的 `memoizedState` 链表中

#### jsx的本质

**`React.createElement()` 的语法糖**

```js
// 嵌套 JSX
const app = (
  <div>
    <h1>Title</h1>
    <p>Content</p>
  </div>
);
// 编译后
const app = React.createElement(
  'div',
  null,
  React.createElement('h1', null, 'Title'),
  React.createElement('p', null, 'Content')
);
```

const [, forceUpdate] = useState({});//执行useState用于强制刷新
