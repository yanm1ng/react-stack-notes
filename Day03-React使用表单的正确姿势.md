# React使用表单的正确姿势

> 在 WEB 应用开发中，表单的作用尤为重要，正是由于表单的存在才使得用户能够与 WEB 应用进行交互。其中表单在实际应用中还涉及到表单数据校验等复杂工作。

## 1.应用表单组件

### 文本框

文本框包含单行文本框 input 和多行文本框 textarea。

```javascript
import React from 'react'

export default class App extends React.Component {
  constructor(props) {
  	super(props)
    this.state = {
      inputValue: '',
      textAreaValue: ''
    }
  }
  
  handleInputChange = (e) => {
    this.setState({
      inputValue: e.target.value
    })
  }
  
  handleTextAreaChange = (e) => {
    this.setState({
      textAreaValue: e.target.value
    })
  }
  
  render() {
    const { inputValue, textAreaValue } = this.state
    return (
      <div>
        <p>Input:</p>
      	<input type="text" value={inputValue} onChange={(e) => this.handleInputChange(e)} />
      	<p>TextArea:</p>
      	<textarea value={textAreaValue} onChange={(e) => this.handleTextAreaChange(e)} />
      </div>
    )
  }
}
```

这里我们可以发现在 JSX 中 textarea 和 input 一样利用 value props 来表示表单的值，而在 HTML 中 textarea 的值则是用 children 来表示。

### 单选框和复选框

在 HTML 中，用类型为 radio 的 input 标签来表示单选按钮。类似的，用类型为 checkbox 的 input 来表示复选框。这两种表单的 value 值一般是不会改变的，而是通过一个布尔值 checked 来表示是否为选中状态，但在 JSX 中用法有一些小区别。

**Radio示例**

```javascript
import React from 'react'

export default class App extends React.Component {
  constructor(props) {
  	super(props)
    this.state = {
      radioValue: ''
    }
  }
  
  handleRadioChange = (e) => {
    this.setState({
      radioValue: e.target.value
    })
  }
  
  render() {
    const { radioValue } = this.state
    return (
      <div>
        <p>Radio:</p>
      	<lable>
          male:
      	  <input 
            type="radio"
            value="male"
            checked={radioValue === 'male'}
            onChange={(e) => this.handleRadioChange(e)}
           />
        </lable>
        <lable>
          female:
      	  <input 
            type="radio"
            value="female"
            checked={radioValue === 'female'}
            onChange={(e) => this.handleRadioChange(e)}
           />
        </lable>
      </div>
    )
  }
}
```

**复选框示例**

```javascript
import React from 'react'

export default class App extends React.Component {
  constructor(props) {
  	super(props)
    this.state = {
      checkedValues: []
    }
  }
  
  handleCheckBoxChange = (e) => {
    const { checked, value } = e.target
    const { checkedValues } = this.state
    if (checked && checkedValues.indexOf(value) === -1) {
      checkedValues.push(value)
    } else {
      checkedValues.filter(item => item !== value)
    }
    this.setState({
      checkedValues
    })
  }
  
  render() {
    const { checkedValues } = this.state
    return (
      <div>
        <p>Checkbox:</p>
      	<lable>
          React:
      	  <input 
            type="checkbox"
            value="react"
            checked={checkedValues.indexOf('react') !== -1}
            onChange={(e) => this.handleCheckBoxChange(e)}
           />
        </lable>
        <lable>
          Vue:
      	  <input 
            type="checkbox"
            value="vue"
            checked={checkedValues.indexOf('vue') !== -1}
            onChange={(e) => this.handleCheckBoxChange(e)}
           />
        </lable>
        <lable>
          jQuery:
      	  <input 
            type="checkbox"
            value="jquery"
            checked={checkedValues.indexOf('jquery') !== -1}
            onChange={(e) => this.handleCheckBoxChange(e)}
           />
        </lable>
      </div>
    )
  }
}
```

看到这里一定会产生疑问了，命名很简单的单选框和复选框使用 JSX 后好像变得复杂了。的确，为了使 React 能够控制到表单的状态，我们使用了 onChange 方法。

### 下拉选择

在 HTML 的 select 元素中，存在单选和多选两种。在 JSX 语法中，同样可以设置 select 标签的 multiple={true} 来实现一个多选下拉列表。这里我们使用多选情况下作为例子：

```javascript
import React from 'react'

export default App extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      selectedValues: []
    }
  }
  
  hanleSelectChange = (e) => {
    const { options } = e.target
    const selectedValues = Object.keys(options)
      .filter(item => options[item].selected)
      .map(item => options[item].value)
    
    this.setState({
      selectedValues
    })
  }
  
  render() {
    const { selectedValues } = this.state
    return (
      <select multiple={true} value={selectedValues} onChange={(e) => this.handleSelectChange(e)}>
        <option value="react">React</option>
        <option value="vue">Vue</option>
        <option value="jquery">jQuery</option>
      </div>
    )
  }
}
```

当然我们也可以像使用 checkbox 一样将每个 option 选项都利用 selectedValue.indexOf 来判断 selected 的值，只不过这样子不太优雅，而且 React 也会报出警告。

> Warning: Use the defaultValue or value props on <select> instead of setting selected on <option>

## 2.受控组件

在上面的例子中，每当表单组件的状态变化时，都会被写入到组件的 state 中，这样的组件在 React 中被称为受控组件。在受控组件中，组件渲染出的状态与它 value 绑定的 state 相对应。React 通过这种方式消除了组件的内部状态，是的应用内的整个状态可控。总结一下 React 受控组件更新 state 的流程：

1. 通过初始 state 设置表单默认值
2. 每当表单发生变化时，调用 onChange 事件处理器
3. 事件处理器通过合成事件对象 event 得到最新的状态，更新 state
4. setState 触发视图的重新渲染，完成表单值的更新

在 React 中数据是单向流动的。表单的数据源于组件的 state，并通过 props 的形式传入，这称之为单向的数据绑定。然后我们又通过 onChange 时间处理器将新的表单数据写回到 state 中，完成了整个双向绑定的过程。

受控组件虽然复杂了很多但是方便我们对内部状态进行维护，通过这样的方式我们可以很轻松的完成表单的校验，使得表单的状态更可靠。

```javascript
handleInputChange = (e) => {
  this.setState({
    inputValue: e.target.value.substring(0, 120)
  })
}
```

## 3.非受控组件

React 同样允许我们使用非受控组件。那么什么是非受控组件呢？

简单的说如果一个表单组件没有 value props，就可以成为非受控组件。

```javascript
import React from 'react'

export default class App extends React.Component {
  constructor(props) {
    super(props)
  }
  
  handleSubmit = () => {
    const { value } = this.refs.name
    console.log(value)
  }
  
  render() {
    <div>
      <input type="text" ref="name" defaultValue="yanm1ng" />
      <button onClick={() => this.handleSubmit()}>Submit</button>
    </div>
  }
}
```

## 4.对比受控组件与非受控组件

### 性能

在受控组件中，每次表单的值发生变化都会调用一次 onChange 事件处理器，这确实会有一些性能上的损耗。这个问题可以通过 Flux/Redux 架构的方式来达到统一的组件状态。

### 事件绑定

受控组件需要额外添加 onChange 的事件绑定，并且可以对表单的值进行校验等操作。 非受控组件由于组件状态不会受到应用状态的控制，因此多了局部的内部状态，这是 React 不希望看到的。