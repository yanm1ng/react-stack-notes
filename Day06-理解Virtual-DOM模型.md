# 理解Virtual DOM模型

> Virtual DOM 之于 React，就好比一个虚拟空间，React 几乎所有的工作都是基于 Virtual DOM 完成的。其中，Virtual DOM 模型负责 Virtual DOM 底层框架的构建工作，它拥有一整套的 Virtual DOM 标签，并负责虚拟节点及其属性的构建、更新、删除等工作。

Virtual DOM 只需具备一个 DOM 标签该有的基本元素即可：

* 标签名 tagname
* 节点属性 property
* 子节点 childnode
* 标识 id

在 React 中，Virtual DOM 中的节点被称之为 ReactNode，它分为3种类型：ReactElement、ReactFragment 和 ReactText。其中 ReactElement 又分为 ReactComponentElement 和 ReactDOMElement。

Virtual DOM 模型几乎覆盖了所有的原生 DOM 标签，如 `<div>` 、`<p>` 、`span` 等。当开发者使用 JSX 时，此时的 `<div>` 标签实际上是 React 生成的 Virtual DOM 对象，只不过看上去长得像 `<div>` 罢了。由于使用了 Virtual DOM，React 的处理并不是直接操作和污染原生 DOM，这样不仅保持了性能上的高效和稳定，而且降低了直接操作 DOM 而导致的错误风险。

ReactDOMComponent 针对 Virtual DOM 标签的处理只要分为以下两个部分：

* 属性的更新，包括更新样式、更新属性、处理事件等。
* 子节点的更新，包括更新内容、更新子节点。

## 属性更新

当执行 mountComponent 方法时，ReactDOMComponent 首先会生成标记和标签，通过 this.createOpenTagMarkupAndPutListeners 来处理 DOM 节点的属性和事件。

* 如果存在事件，则针对当前的节点添加事件代理，即调用 enqueuePutListener
* 如果存在样式，首先会对样式进行合并操作 `Object.assign({}, props.style)` ，然后通过 CSSPropertyOperations.createMarkupForStyles 创建样式
* 通过 DOMPropertyOperations.createMarkupForProperty 创建属性
* 通过 DOMPropertyOperations.createMarkupForID 创建唯一标识 `react-id`

当执行 receiveComponent 方法时，ReactDOMComponent 会通过 this.updateComponent 来更新 DOM 节点属性。

**先是删除不需要的旧属性，再是更新新属性。**

## 子节点更新

当执行 mountComponent 方法时，ReactDOMComponent 会通过 this._createContentMarkup 来处理 DOM 节点的内容。

首先，获取节点内容 props.dangerouslySetInnerHTML。如果子节点存在，则通过 this.mountChildren 对子节点进行初始化渲染。

当执行 receiveComponent 方法时，ReactDOMComponent 会通过 this._updateDOMChildren 来更新 DOM 内容和子节点。

**先是删除不需要的子节点和内容，再是更新子节点和内容。**

