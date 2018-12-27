# 【译】理解异步JavaScript-了解JavaScript工作原理(Event Loop)


> - 原文地址：[Understanding Execution Context and Execution Stack in Javascript](https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0)
> - 原文作者：[Sukhjinder Arora](https://sukhjinderarora.com/)


JavaScript 是一种单线程编程语言，这意味着同一时间只能完成一件事情。也就是说，JavaScript引擎只能在单一线程中处理一次语句。

单线程语言简化了代码编写。因为你不必担心并发问题，但这也意味着你无法在不阻塞主线程的情况下执行网络请求等长时间操作

想象一下从API中请求一些数据。根据情况，服务器可能需要一些时间来处理请求，同时堵塞主线程，让网页无法响应。

这也就是异步JavaScript的美妙之处了。使用异步JavaScript（例如回调，Promise或者await/async），你可以执行长时间网络请求同时不会堵塞主线程。

虽然您没有必要将所有这些概念都学会成为一名出色的JavaScript开发人员，但了解这些对你会很有帮助

所以不用多说了，让我们开始吧！

## 同步JavaScript如何工作？

在我们深入研究异步JavaScript之前，让我们首先了解同步JavaScript代码在JavaScript引擎中的执行情况。例如：

```js
const second = () => {
  console.log('Hello there!');
}
const first = () => {
  console.log('Hi there!');
  second();
  console.log('The End');
}
first();
```

要理解上述代码在JavaScript引擎中的执行方式，我们必须理解执行上下文和调用栈（也称为执行栈）的概念。

### 执行上下文

执行上下文是评估和执行JavaScript代码的环境的抽象概念。每当在JavaScript中运行任何代码时，它都在执行上下文中运行。

函数代码在函数执行上下文中执行，全局代码在全局执行上下文中执行。每个函数都有自己的执行上下文。

### 调用栈

顾名思义，调用栈是一个具有LIFO（后进先出）结构的栈，用于存储代码执行期间创建的所有执行上下文。

JavaScript有一个单独的调用堆栈，因为它是一种单线程编程语言。调用堆栈具有LIFO结构，这意味着只能从堆栈顶部添加或删除项目。

让我们回到上面的代码片段以便尝试理解代码在JavaScript引擎中的执行方式。

```js
const second = () => {
  console.log('Hello there!');
}
const first = () => {
  console.log('Hi there!');
  second();
  console.log('The End');
}
first();
```


![image](https://cdn-images-1.medium.com/max/1240/1*DkG1a8f7rdl0GxM0ly4P7w.png)
 <p align="center">上述代码的调用栈工作情况</p>

 