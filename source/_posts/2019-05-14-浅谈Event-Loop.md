---
title: 浅谈Event Loop
date: 2019-05-14 18:11:30
tags:
---

本文是对于 [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/) 的感悟，记录下来以深入理解。

## 背景
众所周知 JavaScript 是单线程脚本语言，那么在当 JavaScript 接收到多个任务的时候，这些任务就需要一个一个地在执行栈（stack）中排队等待处理，只有当前一个任务处理完成，才会执行下一个任务，这种任务队列被称为同步任务（Synchronous）。

但是如果有一个任务等待的时间很长，后面的任务就不得不等待下去，直到前一个任务结束。那么能不能让时间较长的任务先挂起执行，让后面的任务先处理，等到耗时较长的任务返回了结果才处理呢？异步任务（Asynchronous）就是做这件事的。

## 异步任务
异步任务又被分为 Tasks（Macrotask，宏任务） 和 Microtasks（微任务）。

* **Tasks**： script 标签中的代码、`setTimeout`、`setInterval`、I/O、UI render；
* **Microtasks**： `Promise`、`Object.observe`、`MutationObserver`

## 事件循环的执行过程
1. 处理完执行栈中的任务；
2. 依照先进先出原则依次将 Microtasks 推入执行栈并执行，直到 Microtask 为空；
3. 依照先进先出原则取出一个 Task 执行；
4. 重复步骤三四。

## 简单的例子
``` Javascript
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');
```
在这个例子中,的执行流程入下：
1. script 中的代码被推入 Task，此时 MicroTask 为空，因此 script 被推入 Stack 主线程执行，打印 `script start`；
2. 接着执行遇到 `setTimeout`，它属于 Task 队列，因此被推入 Task，此时 Tasks 中存在 `[script, setTimeout]`；
3. 继续执行遇到 `Promise` 的 then, 它属于 Microtask 队列，因此被推入 Microtasks 中，此时 Microtasks 中存在 `[Promise.then]`；
4. 执行到最后一行输出 `script end`, 到目前为止 script 执行完毕，script 从 Stack 中出栈；
5. 当执行完一个 Task 后，会检查 Microtasks 队列, 发现 Microtasks 中存在 `Promise.then` 的回调，将该回调推入 Stack 中执行，并打印 `promise1`；
6. 接着返回另一个 `Promise` 的回调，将其推入 Microtask 队列，并将第一个 `Promise` 回调从 Stack 中弹出, 该回调也从 Microtask 中弹出；
7. 再次检查 Microtasks 队列, 发现队列不为空，继续执行 `Promise` 的回调，推入 Stack，打印 `promise2` 后该回调从 Stack 弹出，从 Microtask 弹出；
8. 此时再检查 Microtasks 已经为空， script 从 Tasks 队列中弹出，此时 Task 队列为 `[setTimeout]`；
9. 最后将 `setTimeout` 推入 Stack, 打印 `setTimout`，Stack 弹出，Task 弹出，全部任务到此结束。

最后打印结果为：
```
script start
script end
promise1
promise2
setTimeout
```

## 小结
本文通过一个最简单的例子解析了，浏览器里事件循环中 Tasks 和 Microtasks 的关系。
在 [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/) 中会有更多的例子展示出不同浏览器中的差异。主要体现在对 Promise 的执行顺序上。
 
（完）