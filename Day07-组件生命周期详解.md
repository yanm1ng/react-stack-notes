# Day07-组件生命周期详解

React 组件的生命周期主要通过3个阶段进行管理——mounting、receiveProps、unmounting，他们负责通知组件当前所处的阶段，应该执行生命周期中的哪个步骤。

这3个阶段对应3个方法，分别为：mountComponent、updateComponent 和 unmountComponent，每个方法都提供了几种处理方法，其中带 will 前缀的方法在进入状态前调用，带 did 前缀的方法在进入状态后调用。三个阶段包含5种处理方法以及2中特殊状态处理方法。

##  1.mountComponent会触发5个钩子函数

**getDefaultProps()**

设置默认的props，当使用 ES6 classes 构建 React 组件时也可以用 defaultProps 设置组件的默认属性。该方法在整个生命周期中只执行一次，这样所有实例初始化的 props 将会被共享。

**getInitialState()**

在使用es6的class语法时是没有这个钩子函数的，可以直接在constructor中定义this.state。此时可以访问this.props。

**componentWillMount()**

组件初始化时只调用，以后组件更新不调用，整个生命周期只调用一次。若在此时调用 setState 方法是不会触发 re-render 的，而是会进行 state 的合并，并且 componentWillMount 中的 state 不是最新的，在 render 之后在可以获取最新的 state。

**render()**

React 最重要的步骤，创建虚拟DOM，进行diff算法，更新DOM树都在此进行。此时就不能更改state了。

**componentDidMount()**

组件渲染之后调用，可以通过this.getDOMNode()获取和操作DOM节点。

其实 mountComponent 本质是通过**递归**渲染内容的，由于递归的特性，父组件的 componentWillMount 在其子组件的 componentWillMount 之前调用，而父组件的 componentDidMount 在其子组件的 componentDidMount 之后调用。

![](https://pic3.zhimg.com/ec65c26c1123f588c2a57e40423cf6fa_b.png)

## 2.updateComponent会触发5个钩子函数

**componentWillReceivePorps(nextProps)**

组件初始化时不调用，组件接受新的props时调用。

**shouldComponentUpdate(nextProps, nextState)**

React 性能优化非常重要的一环。组件接受新的 state 或者 props 时调用，我们可以设置在此对比前后两个 props 和 state 是否相同，如果相同则返回 false 阻止 re-render，因为相同的属性状态一定会生成相同的 DOM 树，这样就不需要进行diff算法对比，节省大量性能，尤其是在DOM结构复杂的时候。不过调用this.forceUpdate会跳过此步骤。

**componentWillUpdate(nextProps, nextState)**

组件初始化时不调用，只有在组件将要更新时才调用，禁止在此时调用 setState，这会造成循环调用，直至耗光浏览器内存。

**render()**

不多说

**componentDidUpdate()**

组件初始化时不调用，组件更新完成后调用，此时可以获取DOM节点。

updateComponent 本质上也是通过递归完成的，由于递归的特性，父组件的 componentWillUpdate 在其子组件的 componentWillUpdate 之前调用，而父组件的 componentDidUpdate 在其子组件的 componentDidUpdate 之后调用。

![](https://pic1.zhimg.com/34357c2a84e53be3619667ffa4ebbc90_b.png)

## 3.unmountComponent触发1个钩子函数

**componentWillUnmount()**

组件将要卸载时调用，重置所有相关参数、更新队列、以及更新状态，如果此时调用 setState 是不会触发 re-render 的，因为所有更新队列和更新状态都被重置为 null，并且清除了公共类。一般我们可以在这里清除一些事件监听和定时器。

## 4.生命周期图



![img](https://github.com/bailicangdu/pxq/raw/master/src/images/react-lifecycle.png)](https://github.com/bailicangdu/pxq/blob/master/src/images/react-lifecycle.png)