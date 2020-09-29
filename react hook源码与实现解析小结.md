#### useState
- useState 会返回一对值：当前状态和一个让你更新它的函数
- 初始 state 参数只有在第一次渲染时会被用到
- React 假设当你多次调用 useState 的时候，你**能保证**每次渲染时它们的调用顺序是不变的
- Effect Hook，给函数组件增加了操作副作用的能力
---

#### Hook 使用规则
- 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用。
- 只能在 React 的函数组件中调用 Hook。
---

#### useEffect
- React只会在浏览器绘制后运行effects
- Effect的清除同样被延迟了。上一次的effect会在重新渲染后被清除
- React也保证dispatch在每次渲染中都是一样的
- 可以从依赖中去除dispatch, setState, 和useRef包裹的值因为React会确保它们是静态的。不过你设置了它们作为依赖也没什么问题。

- useMemo 和 useCallback 只能作为优化的一种手段而不是程序逻辑的一部分。比如**如果**因为 useCallback 和 useMemo **失效**导致性能变差是符合预期的，但是**如果因为失效导致出现 bug，则是不正确的用法**

### 为什么不能在分支中使用hook
- 因为Hook在设计上是一个链结构
- 返回给组件的是state和对应的setter，re-render时框架并不知道这个setter对应哪个Hooks实例（除非用HashMap来存储Hooks，但这就要求调用的时候把相应的key传给React，会增加Hooks使用的复杂度）

### 小结
hooks设计的初衷是为了**抽象组件中的逻辑以达到复用的目的**，为了组件hook化而使用hook写法是本末倒置。这是因为当组件中存在多个state时，hook写法中针对state的零碎化管理会加重阅读理解代码的负担。