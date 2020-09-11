# 虚拟 DOM 以及 diff 算法

在 React 中，我们使用编写 JSX 代码。但是，JavaScript 引擎是没法直接执行 JSX 的，需要使用 Babel 将 JSX 转化为 JavaScript 代码。

在实际执行时，JSX 代码会生成虚拟 DOM。本质上，虚拟 DOM 就是一个 JavaScript 对象。通过虚拟 DOM，使用某种机制，使它与真实 DOM 发生关联，从而渲染出页面。

当页面发生更改时，通过 diff 算法算出真实 DOM 需要进行的更改，然后更改真实 DOM。本文主要讲 diff 算法。diff 指的是比较新的虚拟 DOM 和老的虚拟 DOM 之间的差别。

标准的的 Diff 算法复杂度需要 O(n^3)，但是 React 的 diff 算法将算法复杂度降到了 O(n)。能做到这点，主要基于两个假设：

- 两个相同组件产生类似的 DOM 结构，不同的组件产生不同的 DOM 结构；
- 对于同一层次的一组子节点，它们可以通过唯一的 id 进行区分。

## 不同节点类型

如果是不同节点类型的两个节点比较，直接删除老的节点，创建并插入新的节点。

## 相同节点类型

### 同一种原生元素

比较两个节点的属性。

### 同一种 Component

- 组件实例还是同一个，依次执行 `componentWillReceiveProps()` 、 `componentWillUpdate()`、`render()`
- 递归 diff render 出来的各个节点

## Children 的递归 - 列表节点的比较

列表节点的操作通常包括添加、删除和排序。为了提高性能，可以给 Children 设置 key。

## 参考

- [reconciliation](https://reactjs.org/docs/reconciliation.html)
