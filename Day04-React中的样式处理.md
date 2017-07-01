# React中的样式处理
> 在 React 中，样式处理是至关重要的一环，也是当前非常热门的话题  

## 1.基本样式处理

React 组件最终会被渲染成 HTML，所以我们可以使用原先 HTML 设置 CSS 一样的方法来设置样式。我们可以设置 className props 来添加类名，也可以通过 style props 来给组件设置行内样式，这里的 style props 是一个对象。

```javascript
render() {
  const style = {
    color: 'green',
    marginTop: 10
  }
  return (
  	<Component style={style} />
  )
}
```

注意点：

### 样式中的像素值

当设置 width 和 height 这类与大小有关的样式时，大部分会以像素为单位，为了提高效率，React 会自动对这样的属性添加 px。但对于某些支持 px 为单位同时还支持数字直接作为值的属性，React 并不添加 px，如 lineHeight。

### 使用 classnames 库

我们可以使用 classnames 库来动态操作类名。

如果不使用 classnames 库就需要这样处理类名：

```javascript
import React from 'react'

export default class Button extends React.Component {
  render() {
    const { disabled, loading, label } = this.state
    let btnCls = 'btn'
    if (disabled) {
      btnCls += ' btn-disabled'
    } else if (loading) {
      btnCls += ' btn-loading'
    }
    return (
      <button className={btnCls}>{label}</button>
    )
  }
}
```

使用 classnames 库之后代码的可读性会更高：

```javascript
import React from 'react'
import classnames from 'classnames'

export default class Button extends React.Component {
  render() {
    const { disabled, loading, label } = this.state
    const btnCls = classnames({
      'btn': true,
      'btn-disabled': disabled,
      'btn-loading': loading
    })
    return (
      <button className={btnCls}>{label}</button>
    )
  }
}
```

## 2.CSS Modules

CSS 模块化重要的是解决两个问题：CSS 样式的导入和导出。灵活按需导入以便复用代码，导出时要能够隐藏内部作用域，以免造成全局污染。Sass、Less、PostCss 等试图解决 CSS 编程能力弱的问题，但并没有解决模块化的问题。

* 全局污染

  CSS 使用全局选择器机制来设置样式，优点是书写方便。缺点是所有的样式都是全局生效，可能会被覆盖，由此产生了非常丑陋的 !important。

* 命名混乱

  由于全局污染的问题，在多人开发过程中为了避免样式冲突，选择器越来越复杂，容易形成不同风格的命名风格，尤其在样式多的情况下，命名更加混乱。

* 依赖管理不彻底

  组件应该相互独立，引入一个组件式应该只需要引入它所需要的样式

* 无法共享变量

  复杂组件使用 JavaScript 和 CSS 来处理样式就会造成有些变量在代码中冗余

* 代码压缩不彻底

CSS 模块化的解决方案有很多，但主要有以下两类：

### Inline Style

这种方案彻底抛弃了 CSS，使用 JavaScript 或 JSON 来书写样式，能使 CSS 和 JavaScript 一样拥有模块化能力，但缺点同样明显，Inline Style 几乎不能利用 CSS 本身强大的特性，比如级联、媒体查询等，:hover、:active 等伪类处理起来也比较复杂。与 React 有关的框架有 Radium、jsxstyle 和 react-style。

### CSS Modules

依旧使用 CSS，但使用 JavaScript 来管理样式依赖。CSS Modules 最大程度的结合了现有的 CSS 生态和 JavaScript 模块化的能力，其 API 非常简洁学习成本低。

### 使用CSS Modules

我们可以利用 webpack 的 css-loader 来实现用 JavaScript 管理样式。

启用 CSS Modules

```javascript
// webpack.config.js
css?modules&localIdentName=[name]_[local]_[hash:base64:5]
```

加上 modules 即为启用，其中 localIndentName 是设置生成类名的命名规则。

我们来看看 webpack 是如何进行转化的：

```css
// Button.css
.btn {}
.disabled {}
```

```javascript
// Button.js
import styles from './Button.css'
console.log(styles)

// =>
// Object {
//   btn: 'button_btn_abc12'
//   disabled: 'button_disabled_1ck1e'
// }
//
```

结果这样的混淆处理后，class 的名称就是唯一的了，大大降低了项目中样式覆盖的几率。

**注意点**

* 样式默认局部

  使用了 CSS Modules 后，就相当于给每个 class 名外加了 :local，以此来实现样式的局部化，如果我们想切换到全局样式可以使用 :global 包裹，如下：

  ```css
  .normal {
    color: green;
  }

  /* 编译后 */
  :local(.normal) {
    color: green;
  }

  /* 全局样式 */
  :global(.btn) {
    color: red;
  }

  /* 定义多个全局样式 */
  :global {
    .link {
      color: bule;
    }
    .active {
    	color: gray;
    }
  }
  ```

* 使用 composes 来组合样式

  ```css
  .base {
    color: green;
  }
  .normal {
    composes: base;
    text-ali
  }
  ```


  ```javascript
  // Button.js
  import styles from './Button.css'
  console.log(styles)

  <button className={styles.normal}>Normal</button>
  // =>
  // <button class="button-base-abcde btn-normal-12345">Normal</button>
  ```