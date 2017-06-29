# React事件系统

> Virtual DOM 在内存中是以对象的形式存在的，所以在这些对象上添加事件将会非常简单。React 基于 Virtual DOM 实现了一个 SyntheticEvent 合成事件层，我们所定义的事件处理器都会接收到一个 SyntheticEvent 对象的实例，符合 W3C 的标准，不会存在 IE 的兼容性问题，与原生的浏览器事件一样拥有同样的接口，同样支持事件的冒泡机制，我们可以使用 stopPropagation() 和 preventDefault() 来中断它。

## 1.React事件的绑定方式

React 事件的绑定方式在写法上与原生的 HTML 事件监听器属性很相似，并且含义和触发的场景也很相似。比如为按钮添加点击事件：

```javascript
render() {
  return (
  	<button onClick={this.handleClick}>Click Me!</button>
  )
}
```

与 HTML 不同的是，在 JSX 中我们必须使用驼峰的形式来书写，而 HTML 事件中我们使用全部小写的属性名。同时 React 并不会真正把事件绑定在元素上，只是借鉴了这种写法。

## 2.合成事件的实现机制

在 React 中主要对合成事件做了两件事情：事件委派和自动绑定。

### 事件委派

前面已经说了 React 并不会把事件处理函数直接绑定到真实的 DOM 节点上，而是把所有的事件绑定到结构的最外层，使用一个统一的事件监听器，这个事件监听器维护了一个映射来保存所有组件内部的事件处理函数。 当一个组件挂载和卸载时会在这个统一的事件监听器中插入/删除一些对象。当事件发生时，首先被这个统一的事件监听器处理，然后在映射中找到真正的事件处理函数并调用。这样做简化了事件的处理和回收机制。

### 自动绑定

在 React 组件中，每一个方法的上下文都会指向组件实例，即自动绑定 this 为当前的组件。在使用 ES6 classes 或纯函数时，这种自动绑定就不存在了，需要我们手动绑定。

绑定的三种方法：

* bind

  `Function.prototype.bind(thisArg [, arg1 [, arg2, …]])` 是 ES5 新增的函数扩展方法， bind() 返回一个新的函数对象，该函数的 this 被绑定到 thisArg 上，并向事件处理器中传入参数。

  ```javascript
  import React, { Component } from 'react'

  class Test extends React.Component {
    constructor (props) {
      super(props)
      this.state = {
        message: 'Hello!'
      }
    }
    handleClick (name, e) {
      console.log(this.state.message + name)
    }
    render () {
      return (
        <div>
          <button onClick={this.handleClick.bind(this, 'yanm1ng')}>Say Hello</button>
        </div>
      )
    }
  }
  ```

* 构造函数

  在构造函数  constructor 内绑定 this，好处是仅需要绑定一次，避免每次渲染时都要重新绑定，函数在别处复用时也无需再次绑定。

  ```javascript
  import React, { Component } from 'react'

  class Test extends React.Component {
    constructor (props) {
      super(props)
      this.state = {
        message: 'Hello!'
      }
      this.handleClick = this.handleClick.bind(this)
    }
    handleClick (e) {
      console.log(this.state.message)
    }
    render () {
      return (
        <div>
          <button onClick={this.handleClick}>Say Hello</button>
        </div>
      )
    }
  }
  ```

* 箭头函数

  箭头函数则会捕获其所在上下文的 this 值，作为自己的 this 值，使用箭头函数就不用担心函数内的 this 不是指向组件内部了。

  ```javascript
  class Test extends React.Component {
    constructor (props) {
      super(props)
      this.state = {
        message: 'Hello!'
      }
    }
    handleClick (e) {
      console.log(this.state.message)
    }
    render () {
      return (
          <div>
            <button onClick={() => this.handleClick()}>Say Hello</button>
          </div>
      )
    }
  }
  ```


## 3.使用原生事件

React 提供了很好的合成事件系统，但不意味着 React 中无法使用原生的时间，在 React 生命周期的 componentDidMount 中组件以及挂载完成并且已经存在真实的 DOM 后，便可以进行原生事件的绑定。比如：

```javascript
function handleClick(e) {
  console.log(e)
}

class Test extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      message: 'Hello!'
    }
  }
  componentDidMount () {
    this.refs.button.addEventListener('click', e => {
      handleClick(e)
    })
  }
  componentWillUnmount () {
    this.refs.button.removeEventListener('click')
  }
  render () {
    return (
        <div>
          <button ref="button">Say Hello</button>
        </div>
    )
  }
}
```

不过千万别忘记一定要在组件卸载时手动移除哦，否则可能出现内存泄漏的问题。而合成事件则不需要，因为 React 已经帮我们处理了。