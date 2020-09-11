# useEffect

useEffect 给予了组件产生副作用的能力，部分代替了 class 组件中 lifecycle 函数的功能。

## useEffect 初识

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

第一次的渲染过程：

1. React: 给我状态为 0 时候的 UI。
2. 你的组件:
   1. 给你需要渲染的内容: `<p>You clicked 0 times</p>`。
   2. 记得在渲染完了之后调用这个 effect: `() => { document.title = 'You clicked 0 times' }`。
3. React: 没问题。开始更新 UI，喂浏览器，我要给 DOM 添加一些东西。
4. 浏览器: 酷，我已经把它绘制到屏幕上了。
5. React: 好的， 我现在开始运行给我的 effect。运行 `() => { document.title = 'You clicked 0 times' }`。

我们点击之后发生的事情：

1. 你的组件: 喂 React, 把我的状态设置为 1。
2. React: 给我状态为 1 时候的 UI。
3. 你的组件:
   1. 给你需要渲染的内容: `<p>You clicked 1 times</p>`。
   2. 记得在渲染完了之后调用这个 effect： `() => { document.title = 'You clicked 1 times' }`。
4. React: 没问题。开始更新 UI，喂浏览器，我修改了 DOM。
5. Browser: 酷，我已经将更改绘制到屏幕上了。
6. React: 好的， 我现在开始运行属于这次渲染的 effect。运行 `() => { document.title = 'You clicked 1 times' }`。

## 语法

```javascript
useEffect(() => {
  something();
  return cleanup;
}, [deps]);
```

## 函数组件的特点 -- 每一次渲染都有它自己的 Mr Right

- 每一次渲染都有它自己的 Props 和 State。我们的组件函数每次渲染都会被调用，但是每一次调用中 state 值都是常量，并且它被赋予了当前渲染中的状态值。也就是说，在某次渲染内，state 和 props 是不变的。
- 每一次渲染都有它自己的事件处理函数。也就是说，每出一次都是一个新的事件处理函数。，每一个版本的事件处理函数都记住了当前版本的 state。
- 每一次渲染都有它自己的 Effects。原理、效果与上面的第二点类似。每一个 effect 版本“看到”的 state 值都来自于它属于的那次渲染。

可以推导出结论：

> 每一个组件内的函数（包括事件处理函数，effects，定时器或者 API 调用等等）会捕获某次渲染中定义的 props 和 state。

原因也很简单，那就是闭包。所以，学好闭包很重要啊。

## Effect 的清理

React 只会在浏览器绘制后运行 effects。Effect 的一个很有意思的点在于，它能返回一个清理函数，负责打扫战场。有了这个清理函数，useEffect 回调函数中的代码执行次序就很有意思了。

```javascript
useEffect(() => {
  // 副作用代码
  ChatAPI.subscribeToFriendStatus(props.id, handleStatusChange);

  // 清理函数
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.id, handleStatusChange);
  };
});
```

假设第一次渲染的时候 props 是 `{id: 10}`，第二次渲染的时候是 `{id: 20}`，会发生：

1. React 渲染{id: 20}的 UI。
2. 浏览器绘制。我们在屏幕上看到{id: 20}的 UI。
3. React 清除{id: 10}的 effect。 // surprise!!!!
4. React 运行{id: 20}的 effect。

那为什么第三步还能记得之前的 props 呢？又是什么前端黑魔法吗？其实答案你刚才看到过：

> 每一个组件内的函数（包括事件处理函数，effects，定时器或者 API 调用等等）会捕获某次渲染中定义的 props 和 state。

因为你执行的是上一个版本中定义的清理函数嘛！

## 根据依赖决定是否执行 Effect

当你需要根据条件来确定是否执行 Effect 了，你可以选择使用的武器是：可选参数 deps 依赖项。React 会根据这个依赖项，来确定是否执行 Effect。如果当前渲染中的这些依赖项和上一次运行这个 effect 的时候值一样，因为没有什么需要同步 React 会自动跳过这次 effect。

由上面你也可以知道，React 会根据依赖来判断是否要执行 effect，所以你不要当渣男欺骗 React。

## 依赖的解决方案

- 如果依赖是 state，可以使用 useReducer

  ```javascript
  const [state, dispatch] = useReducer(reducer, initialState);
  const { count, step } = state;

  useEffect(() => {
    const id = setInterval(() => {
      dispatch({ type: "tick" }); // Instead of setCount(c => c + step);
    }, 1000);
    return () => clearInterval(id);
  }, [dispatch]);
  ```

- 如果依赖是 props，可以使用 useReducer

  ```javascript
  function Counter({ step }) {
    const [count, dispatch] = useReducer(reducer, 0);

    // 把 reducer 放到组件内，这样就能用了
    function reducer(state, action) {
      if (action.type === "tick") {
        return state + step;
      } else {
        throw new Error();
      }
    }

    useEffect(() => {
      const id = setInterval(() => {
        dispatch({ type: "tick" });
      }, 1000);
      return () => clearInterval(id);
    }, [dispatch]);

    return <h1>{count}</h1>;
  }
  ```

所以，难怪说 useReducer 是一种作弊器啊。

## 参考

- [useEffect 完整指南](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)
