# Day08-解密setState机制

> state 是 React 中的重要概念。React 是通过管理状态来实现对组件的管理，通过 this.state 来访问 state，通过 this.setState 来更新 state。

setState 通过一个队列机制来实现 state 的更新。调用 setState 函数之后，会将需要更新的 state 合并放入状态队列，而不会立即更新 state，队列机制可以高效地批量更新 state。如果不通过 setState 而直接修改 state 的值，那么该 state 不会被放入到状态队列中，当下次调用 setState 并对状态队列进行合并是，将会忽略之前直接修改的 state，从而造成无法预知的错误。

因此，应该使用 setState 方法来更新 state，同时 React 也正是利用状态队列机制实现了 setState 的异步更新，避免频繁地重复更新 state。

```javascript
var nextState = this._processPendingState(nextProps, nextState);

var shouldUpdate = this._pendingForceUpdate || !inst.shouldComponentUpdate || inst.shouldComponentUpdate(nextProps, nextState, nextContext)
```

当调用 setState 时，实际上会执行 enqueueSetState 方法，并对 partialState 以及 _pendingStateQueue 更新队列进行合并操作，最终通过 enqueueUpdate 执行 state 更新。

既然 setState 最终是通过 enqueueUpdate 执行 state 的更新，那么 enqueueUpdate 到底是如何更新 state 的呢？

![](https://pic1.zhimg.com/4fd1a155faedff00910dfabe5de143fc_b.png)

上图是一个简化的 setState 调用栈。

```javascript
function enqueueUpdate(component) {
  // ...

  if (!batchingStrategy.isBatchingUpdates) {
    batchingStrategy.batchedUpdates(enqueueUpdate, component);
    return;
  }

  dirtyComponents.push(component);
}
```

若 isBatchingUpdates 为 true，则把当前组件（即调用了 setState 的组件）放入 dirtyComponents 数组中；否则 batchUpdate 所有队列中的更新。

那么 batchingStrategy 究竟是何方神圣呢？其实它只是一个简单的对象，定义了一个 isBatchingUpdates 的布尔值，和一个 batchedUpdates 方法。下面是一段简化的定义代码：

```javascript
var batchingStrategy = {
  isBatchingUpdates: false,
  // ...
  batchedUpdates: function(callback, a, b, c, d, e) {
    // ...
    var alreadyBatchingUpdates = ReactDefaultBatchingStrategy.isBatchingUpdates;
    batchingStrategy.isBatchingUpdates = true;
    if (alreadyBatchingUpdates) {
      callback(a, b, c, d, e);
    } else {
      transaction.perform(callback, null, a, b, c, d, e); 
    }
  }
};
```

注意 batchingStrategy 中的 **batchedUpdates** 方法中，有一个 transaction.perform 调用。这就引出了本文要介绍的核心概念 —— Transaction（事务）。

熟悉 MySQL 的同学看到 Transaction 是否会心一笑？然而在 React 中 Transaction 的原理和行为和 MySQL 中并不完全相同，让我们从源码开始一步步开始了解。

在 Transaction 的源码中有一幅特别的 ASCII 图，形象的解释了 Transaction 的作用。

```
/*
 * 
 *                       wrappers (injected at creation time)
 *                                      +        +
 *                                      |        |
 *                    +-----------------|--------|--------------+
 *                    |                 v        |              |
 *                    |      +---------------+   |              |
 *                    |   +--|    wrapper1   |---|----+         |
 *                    |   |  +---------------+   v    |         |
 *                    |   |          +-------------+  |         |
 *                    |   |     +----|   wrapper2  |--------+   |
 *                    |   |     |    +-------------+  |     |   |
 *                    |   |     |                     |     |   |
 *                    |   v     v                     v     v   | wrapper
 *                    | +---+ +---+   +---------+   +---+ +---+ | invariants
 * perform(anyMethod) | |   | |   |   |         |   |   | |   | | maintained
 * +----------------->|-|---|-|---|-->|anyMethod|---|---|-|---|-|-------->
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | +---+ +---+   +---------+   +---+ +---+ |
 *                    |  initialize                    close    |
 *                    +-----------------------------------------+
 * 
 */
```

简单地说，一个所谓的 Transaction 就是将需要执行的 method 使用 wrapper 封装起来，再通过 Transaction 提供的 perform 方法执行。而在 perform 之前，先执行所有 wrapper 中的 initialize 方法；perform 完成之后（即 method 执行后）再执行所有的 close 方法。一组 initialize 及 close 方法称为一个 wrapper，从上面的示例图中可以看出 Transaction 支持多个 wrapper 叠加。

具体到实现上，React 中的 Transaction 提供了一个 mixin 方便其它模块实现自己需要的事务。而要使用 Transaction 的模块，除了需要把 Transaction 的 mixin 混入自己的事务实现中外，还需要额外实现一个抽象的 getTransactionWrappers 接口。这个接口是 Transaction 用来获取所有需要封装的前置方法（initialize）和收尾方法（close）的，因此它需要返回一个数组的对象，每个对象分别有 key 为 initialize 和 close 的方法。

一个简单的使用事物的例子：

```javascript
var Transaction = require('./Transation');

var MyTransaction = function() {
  
};

Object.assign(MyTransaction.protoType, Transaction.Mixin, {
  getTransactionWrappers: function() {
    return [{
      initialize: function() {
        console.log('before perform')
      },
      close: function() {
        console.log('after perform')
      }
    }]
  }
});

var testTransaction = new MyTransaction();
var testMethod = function() {
  console.log('method called')
};
testTransaction.perform(testMethod);

// console
// before perform
// method called
// after perform
```

