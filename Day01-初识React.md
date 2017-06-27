# 初识React

## 1. React特点
### Virtual DOM
我们知道，在传统的前端开发中，一个 HTML 页面对应一个 DOM 树。每次需要更新页面时，都是对响应的 DOM 进行操作。DOM 的操作成本很高，性能消耗最大，这也是为什么这几年前端涌现出各种 MVVM 框架的主要原因。而 React 把真实的 DOM 树转化为 JavaScript 对象，这就是 Virtual DOM。每次数据更新后，重新计算 Virtual DOM，并且和上次的 Virtual DOM 进行对比，对发生改变的部分进行批量更新（React 还提供 shouldComponentUpdate 生命周期函数来减少不必要的 Virtual DOM 对比过程，以保证性能）。
### JSX
JSX 像是在 Javascript 代码里直接写 XML 的语法，实质上这只是一个语法糖，每一个 XML 标签都会被 JSX 转换工具转换成纯 Javascript 代码。使用 JSX 可以使得组件的结构和组件之间的关系变得更加清晰，易于代码维护。React 官方在早期为 JSX 语法提供了一套解析工具 JSTransform，目前已经不再维护。现在主要使用的是 Babel 的 JSX 解析器。

```javascript
//使用JSX
React.render(
  <div>
    <div>content</div>
  </div>,
  document.getElementById('example')
);
 
//不使用JSX
React.render(
  React.createElement('div', null,
    React.createElement('div', null, 'content')
  )),
  document.getElementById('example')
);
```

可见我们也可以使用纯 JavaScript 代替 JSX，只不过体验上实在糟糕。

**JSX 语法注意点**
* 最外层需要被一个标签包裹
* 标签必须闭合
* 组件标签首字母必须大写
* 由于 `class` 和 `for` 在 JavaScript 中都是关键字，所以使用 `className` 代替 `class` ，使用 `htmlFor` 代替 `for`
* 缺省的 Boolean 属性值会导致 JSX 认为布尔值设为了 `true`
### 组件化
在 MV* 架构出现之前组件主要分为两种：
* UI组件  
  组件主要是交互动作上的抽象。
* 业务组件  
  包含业务和数据的集合，通常是各种UI组件的集合  

那么 React 的组件化是什么呢？

React 的组件基本上由三部分组成——属性（props）、状态（state）以及生命周期方法。

React 组件可以接受参数，同时自身也包含状态。一旦参数/状态改变就会触发相应的生命周期方法，重新渲染组件。

React 构建组件的方式通常有三种：
* React.createClass  
  使用 `React.createClass` 构建组件是 React 最传统同时也是兼容性最好的方法，在 0.14 版本发布之前一直都是官方推荐的组件写法。示例代码如下：

```javascript
const Button = React.createClass({
  getDefaultProps() {
    return {
      type: 'primary',
      text: 'submit'
    }
  },

  render() {
    const { type, text } = this.props;
    return (
      <div className={`btn btn-${type}`}>
        {text}
      </div>
    )
  }
})
```

* ES6 classes
  通过 ES6 标准的类语法来构建 React 组件

```javascript
class Button extends React.Component {
  constructor(props) {
    super(props)
  }

  static defaultProps = {
    type: 'primary',
    text: 'submit'
  }

  render() {
    const { type, text } = this.props;
    return (
      <div className={`btn btn-${type}`}>
        {text}
      </div>
    )
  }
}
```
* 无状态函数
  使用无状态函数构建的组件称为无状态组件，这种构建方式是 0.14 版本后新增的，并且官方推崇。

```javascript
function Button({ type: 'primary', text: 'submit' }) {
  return (
    <div className={`btn btn-${type}`}>
      {text}
    </div>
  )
}
```

无状态组件只传入 `props`，也就是说它不存在内部的 `state`，也没有生命周期方法。
在合适的时候，我们都应该使用无状态组件。无状态组件不像之前两种方法在调用时会创建新的实例，它在创建时始终保持了一个实例，避免了不必要的检查和内存分配，做到了内部的优化。

## 2. React数据流
在 React 中数据是自顶向下单向流动的，即从父组件 => 子组件。这条原则让组件之间的关系变得简单且可预测。

state 和 props 是 React 中最重要的概念。如果顶层组件初始化 props，那么 React 会向下遍历整个组件树，传递 props，而组件内部有各自的 state，这些状态只能在组件内部改变。

如果把组件看做一个函数，那么他接受了 props 作为参数，有 state 作为函数内部参数，函数返回一个 Virtual DOM。

React 提供 setState 方法来更新 state，值得注意的是 setState 是一个异步方法，一个生命周期内的所有 setState 会合并操作。

## 3.React生命周期
自然界的生命周期可以分为出生、成长、死亡。React 组件的生命周期同样可以分为三类：

### 组件挂载
这个过程主要进行组件状态的初始化。包含 componentWillMount 和 componentDidMount 两个生命周期方法，它们分别在 render 方法前后执行。

### 组件更新
更新过程指的是父组件向下传递 props 或 组件自身执行 setState 方法时发生的一系列更新动作。如果组件的 state 更新了，那么会依次执行 shouldComponentUpdate、componentWillUpdate、render 和 componentDidUpdate。

### 组件卸载
组件卸载非常简单，只有 componentWillUnmount 这一个卸载前的方法，在这个方法中我们通常会进行一些清理方法，如事件回收和清除定时器。