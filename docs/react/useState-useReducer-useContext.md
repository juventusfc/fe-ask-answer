# 状态管理相关的 Hooks

## useState -- 简单的状态管理

useState 是 Hooks 自带的一个 Hook，用于在函数组件中引入 state 概念。

### 基本用法

```javascript
const App = () => {
  // 1. 使用 useState
  const [count, setCount] = useState(0);

  // 2. 调用 setCount，更改 count 值， App函数重新执行
  const handleIncrease = () => setCount((preCount) => preCount + 1);
  const handleDecrease = () => setCount((preCount) => preCount - 1);

  return (
    <div>
      Count: {count}
      <hr />
      <button onClick={handleIncrease}>Increase</button>
      <button onClick={handleDecrease}>Decrease</button>
    </div>
  );
};
```

### 更改 state 时有两种方案

setState(arg)的 arg 写法有两种：

- arg 就是新的 state
- arg 是一个函数，根据来的 state 返回新的 state

我们可以来看两个例子。

```javascript
const App = () => {
  const [count, setCount] = React.useState(0);

  const handleIncrease = () => {
    // 新的 state 就是 count + 1
    setTimeout(() => setCount(count + 1), 1000);
  };

  const handleDecrease = () => {
    setTimeout(() => setCount(count - 1), 1000);
  };

  return (
    <div>
      Count: {count}
      <hr />
      <div>
        <button type="button" onClick={handleIncrease}>
          Increase
        </button>
        <button type="button" onClick={handleDecrease}>
          Decrease
        </button>
      </div>
    </div>
  );
};
```

在上面的例子中，arg 就是新的 state。 当运行起来，我们快速点击时，发现页面上呈现了一个类似于函数防抖的功能。也就是说，页面上的 Count 数量并不是我们实际点击的数量。

我们可以看一下实际发生了什么：

1. 当我们连续点击 Increase 按钮，会连续触发 handleIncrease 函数。但是，当前函数组件的 count 值还是 0，执行了多个 `setTimeout(() => setCount(0 + 1), 1000);`。
2. 当第一次点击后的 1000ms 到期后，由于 count 值变成了 1，所以函数组件重新渲染。但是要记住，由于步骤 1 中有多个 setTimeout 执行了，他们到期也会重新执行 App 方法，但是因为 count 值还是 1，所以不会重新渲染。
3. 其他时间线上的步骤与上面类似。

那么，我们有没有办法得到准确的点击次数呢？还真有，而且改动不大。

```javascript
const App = () => {
  const [count, setCount] = React.useState(0);

  const handleIncrease = () => {
    // 每次依据老的 state 得到新的state
    setTimeout(() => setCount((state) => state + 1), 1000);
  };

  const handleDecrease = () => {
    setTimeout(() => setCount((state) => state - 1), 1000);
  };

  return (
    <div>
      Count: {count}
      <hr />
      <div>
        <button type="button" onClick={handleIncrease}>
          Increase
        </button>
        <button type="button" onClick={handleDecrease}>
          Decrease
        </button>
      </div>
    </div>
  );
};
```

上面的代码就改动了 `setTimeout(() => setCount((state) => state - 1), 1000);` 这一行。解释起来也很简单， `(state) => state - 1` 根据老的 state 得到新的 state，这就保证了每次执行的时候，count 值都是最新的。至于这是怎么实现的，应该是 React 帮你记住了所有的 state，真正去执行 callback 时，将你需要的给了你。

## useReducer -- 复杂的状态管理

useReducer 部分借鉴了 redux，实现了组件中的复杂状态管理。基本语法为：

`const [state, dispatch] = useReducer(reducer, initialArg, init);`

```javascript
// 初始值
const initialState = { count: 0 };

// 定义 reducer，含义与 redux 一致
function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}

function Counter() {
  // 当用户触发 dispatch， 会得到新的state，使组件重新渲染
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
    </>
  );
}
```

## useContext -- 全局的状态管理

如果组件树很深，子组件为了获取祖先组件的某些信息，需要层层传递。为了解决这个问题，引入了 Context 的概念。有了 Context，子组件就能直接得到里面的状态，而不用一层层通过 props 传下来了。

useContext 的使用步骤有：

```javascript
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee",
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222",
  },
};

// 1. 创建 Context
const ThemeContext = React.createContext(themes.light);

function App() {
  // 2.  提供 Context 的值
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  // 3. 子组件获得 Context
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```
