# React 封装模式

Higher Order Components 和 Render Prop Components 是 hooks 出现以前，被广泛采用的 React 封装模式。 React 16.8 引入了 hooks 的概念，是另一种抽象、封装组件方法。

## Higher Order Components

HOCs 可以理解为一个函数，这个函数接受一个 Component 作为参数，返回一个新的 Component。设计上来说，就是不改变原 Component，但是会在原 Component 基础上额外封装一层 Wrapper。主要用于抽象很多组件都拥有的类似功能。

```javascript
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

关于 HOCs，可以多看看 FP 相关知识，加深理解。至于学习 FP，要么去学习实现一个 Toy Lodash 看看？

可以看一个例子：

```javascript
function withLogProps(WrappedComponent) {
  return class extends React.Component {
    componentDidUpdate(prevProps) {
      console.log("Current props: ", this.props);
      console.log("Previous props: ", prevProps);
    }
    render() {
      // Wraps the input component in a container, without mutating it. Good!
      return <WrappedComponent {...this.props} />;
    }
  };
}

const WrappedComponentWithLog = withLogProps(WrappedComponent);
function App(props) {
  return <WrappedComponentWithLog />;
}
```

## Render Prop Components

Render Prop Components 表示组件的 props 对象有一个 render 属性，该属性是一个函数，根据传入值返回 React Element。

```javascript
<DataProvider render={(data) => <h1>Hello {data.target}</h1>} />
```

注意，不一定非要是 render 属性，你可以取任何名字，只是 render 属性更符合语义而已。可以看一个例子：

```javascript
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img
        src="/cat.jpg"
        style={{ position: "absolute", left: mouse.x, top: mouse.y }}
      />
    );
  }
}

// Mouse 与 Cat 解耦，调用的时候才关联
class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY,
    });
  }

  render() {
    return (
      <div style={{ height: "100vh" }} onMouseMove={this.handleMouseMove}>
        {/*
          Instead of providing a static representation of what <Mouse> renders,
          use the `render` prop to dynamically determine what to render.
          执行 render 方法，渲染传进去的内容
        */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={(mouse) => <Cat mouse={mouse} />} />
      </div>
    );
  }
}
```

## Hooks

Hooks 会有一个文档单独介绍。
