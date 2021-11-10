### Promise源码剖析

**Promise **对象用于表示一个异步操作的最终完成 (或失败)及其结果值，可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。此外，Promise 对象提供统一的接口，使得控制异步操作更加容易

一个 `Promise` 必然处于以下几种状态之一：

- *待定（pending）*: 初始状态，既没有被兑现，也没有被拒绝
- *已兑现（fulfilled）*: 意味着操作成功完成
- *已拒绝（rejected）*: 意味着操作失败

> Promise内部还有一个状态`adopted`，用来表示返回值`_value`也是一个Promise的情况

简单介绍就到这了

下面从源码角度深度剖析Promise的调用流程

#### 内部数据结构

```js
function Promise(fn) {
  ...
  this._deferredState = 0;
  this._state = 0;
  this._value = null;
  this._deferreds = null;
  ...
}
```

* `_deferredState` 它的*下一个需要处理的Promise*（下文简称p）的状态，而Handler的实例对象h包含了p的引用。有3种状态

  * 0 初始状态
  * 1 等待状态，此时`_deferreds`指向h
  * 2 等待状态，此时`_deferreds`是一个队列，h是队列中的一员

* `_deferreds` 根据`_deferredState `变化而变化，对p进行2次及以上`.then`调用时，会转化为数组（队列），p1，p2就存储在队列中。这种用法比较少

  ```js
  const p = new Promise(fn)
  const p1 = p.then(callback)
  const p2 = p.then(callback)
  ...
  ```

* `_state` 即上文介绍的Promise的4种状态

* `_value` Promise处理完成后的结果

```js
function Handler(onFulfilled, onRejected, promise) {
  this.onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : null;
  this.onRejected = typeof onRejected === 'function' ? onRejected : null;
  this.promise = promise;
}
```

内部函数Handler，简单来说就是用来缓存`.then`调用产生的一系列属性，包括回调函数`onFulfilled`，`onRejected`以及新建的Promise对象，也就是上文提到的p

> **总结** Promise是一个链表+队列的数据结构。`_deferreds`是Handler对象时，则形成一个Promise链；是数组时，链式结构到此为止，但数组中的Handler的引用p又可形成新的链表+队列。在实际使用中，形成队列的情况比较少，大多数都是一个链表结构

最后，通过一段代码，一张图，来表示它的链式结构

```js
new Promise(() => { 
 // async operations... 
}).then(res => {
  // some operations
}).then(end => {
  // end
})
```

![](images/c/c-6-promise-chain.png)

如果忽略Handler这一层包装，就可以把它看成一个真正的Promise链了

#### 调用流程

为了简化模型，不考虑队列的情况，将调用流程分为2个阶段

##### 宏任务阶段

即从new promise到.then链式调用完成，此阶段生成了完整的promise链

* new promise干了什么

  1. 验证：必须通过new操作符实例化Promise；必须传入回调函数fn
  2. 参数初始化
  3. 调用回调函数fn，且会给fn传入两个参数（也是函数）`resolve`和`reject`，fn执行完成后，必须调用其中一个函数，否则后续`.then`的回调函数都不会执行（`.then`函数是会执行的）

* `.then`链式调用

  每次`.then`调用都会初始化一个新的Handler实例h和Promise实例p，当前Promise实例的`_deferreds`属性指向h，h的promise属性指向p，并且h中保存了`.then`传入的回调函数的引用

