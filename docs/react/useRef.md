# useRef

useRef 最大的特点是：更改它的值，不会触发 re-render。所以，如果你的应用中需要一些常量，但是又不想在更改这些常量时触发 re-render，可以选择使用 userRef。

useRef 另一个使用场景是跟 DOM 一起使用。

```javascript
function App() {
  return <ComponentWithDomApi label="Label" value="Value" isFocus />;
}

function ComponentWithDomApi({ label, value, isFocus }) {
  const ref = React.useRef(); // (1) 创建 ref

  React.useEffect(() => {
    if (isFocus) {
      ref.current.focus(); // (3) 操作 DOM 节点
    }
  }, [isFocus]);

  return (
    <label>
      {/* (2) React 会将对应的 DOM 节点传递给 ref */}
      {label}: <input type="text" value={value} ref={ref} />
    </label>
  );
}
```
