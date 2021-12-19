### EventLoop

#### 常见任务分类
* 宏任务
  * `script`
  * `setTimeout/setInterval`
  * `setImmediate` (Node环境)
  * `IO/UI事件`
  * `postMessage`
* 微任务
  * `Promise`
  * `nextTick` (Node环境)
  * `Object.observe`
  * `MutationObserver`

#### 浏览器EventLoop

1. 主线程由入口scrip开始（宏任务）

2. 遇到同步代码就继续执行，直到执行完

3. 遇到异步API，则交给对应异步线程，自己继续执行同步任务

4. 异步线程执行完后，将异步回调事件放入对应事件队列

5. 主线程的同步任务执行完成后，检查事件队列有没有任务

   > 1. 检查微任务队列，有的话全部执行完（此过程可加入新的微任务）
   > 2. 检查宏任务队列，有的话，执行第一个宏任务，回到第1步

6. 执行事件队列任务

#### Node EventLoop

> * Node在执行微任务时，会优先执行`nextTick task queue`中的任务，执行完毕之后，会接着执行`promise task queue`中的任务
> * 对于微任务的执行顺序，以下主要适用于`Node 11`之前。`Node 11`及其之后，针对事件循环的每一个阶段，微任务的执行顺序进行了统一：**在每次调用回调之后，就执行相应微任务，不会等到所有回调执行完毕后才执行**

Node事件循环分为`6`个阶段

1. **Timers（计时器阶段）**

   初次进入事件循环，从计时器阶段开始。此阶段会判断是否存在过期的计时器回调（包括`setTimeout`和`setInterval`）。如果存在则会执行所有过期的计时器回调，执行完毕后，如果回调中触发了相应的微任务，会接着执行所有微任务，执行完微任务后进入`Pending callbacks`阶段

2. **Pending callbacks**

   执行推迟到下一个循环迭代的I/O回调（系统调用相关）

3. **Idle/Prepare**

   仅供内部使用

4. **Poll**

   * 当回调队列不为空时：执行回调，若回调中触发了相应微任务，马上执行该微任务。执行完所有回调后，变为下面的情况
   * 当回调队列为空时：如果存在有计时器（`setTimeout/setInterval/setImmediate`）没有执行，会结束`轮询`阶段，进入`Check`阶段。否则会阻塞并等待任何正在执行的I/O操作完成，并马上执行相应回调，直到所有回调执行完毕

5. **Check**

   检查是否存在`setImmediate`相关的回调，如果存在，则执行所有回调。执行完毕后，如果回调中触发了相应的微任务，会接着执行所有微任务，完成后再进入`Close callbacks`阶段

6. **Close callbacks**

   执行一些关闭回调，如`socket.on('close', ...)`等

