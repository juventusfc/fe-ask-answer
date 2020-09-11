# React.memo 和 useCallback 和 useMemo

这两个 hooks 主要用于性能优化。

## React.memo

```javascript
const MyComponent = React.memo(function MyComponent(props) {
  /* render using props */
});
```

React.memo 会判断新的 props 和老的 props，只有 props 变化了，MyComponent 函数组件才会重新执行。

## useCallback

useCallback 用于 memorize function。由于每次函数组件重新执行，都会重新定义函数，所以需要 memorize 函数。

```javascript
import React from "react";
import { v4 as uuidv4 } from "uuid";

const App = () => {
  console.log("Render: App");
  const [users, setUsers] = React.useState([
    { id: "a", name: "Robin" },
    { id: "b", name: "Dennis" },
  ]);

  const [text, setText] = React.useState("");

  const handleText = (event) => {
    setText(event.target.value);
  };

  const handleAddUser = () => {
    setUsers(users.concat({ id: uuidv4(), name: text }));
  };

  const handleRemove = (id) => {
    setUsers(users.filter((user) => user.id !== id));
  };

  return (
    <div>
      <input type="text" value={text} onChange={handleText} />
      <button type="button" onClick={handleAddUser}>
        Add User
      </button>

      <List list={users} onRemove={handleRemove} />
    </div>
  );
};

const List = ({ list, onRemove }) => {
  console.log("Render: List");
  return (
    <ul>
      {list.map((item) => (
        <ListItem key={item.id} item={item} onRemove={onRemove} />
      ))}
    </ul>
  );
};

const ListItem = ({ item, onRemove }) => {
  console.log("Render: ListItem");
  return (
    <li>
      {item.name}
      <button type="button" onClick={() => onRemove(item.id)}>
        Remove
      </button>
    </li>
  );
};

export default App;
```

上面的例子中，只要在输入框中打字，App -> List -> ListItem 都会重新执行。我们很快就能想到使用 `React.memo()` 提高性能。也就是说，改成：

```javascript
// 以 List 为例子
const List = React.memo(({ list, onRemove }) => {
  console.log("Render: List");
  return (
    <ul>
      {list.map((item) => (
        <ListItem key={item.id} item={item} onRemove={onRemove} />
      ))}
    </ul>
  );
});
```

但是你会发现，只要在输入框中打字，App -> List -> ListItem 还是重新执行。那么问题来了，为什么会这样呢？

原因其实上面说了，由于每次函数组件重新执行，都会重新定义函数。既然 onRemove 变了，当然要重新渲染啦。

React 团队也算为开发者操碎了心，提供了 useCallback 来解决这个问题。

```javascript
const App = () => {
  /// ...

  const handleRemove = React.useCallback(
    (id) => setUsers(users.filter((user) => user.id !== id)),
    [users]
  );

  // ...
};
```

useCallback 接受一个回调和可选的 deps，当 deps 中的值进行了更改，handleRemove 才会新建一个新的。这样，就实现了之只更新 App，不更新 List 和 ListItem 的目的。

## useMemo

```javascript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

useMemo 返回一个 memoized 的值，也就是说，它会执行 `(() => computeExpensiveValue(a, b))()`。

`useCallback(fn, deps)` 等价于 `useMemo(() => fn, deps)`。
