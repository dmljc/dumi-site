---
toc: content
order: 5
---

# Event Loop

在此之前我们了解了 JS 异步相关的知识。在实践的过程中，你是否遇到过以下场景，为什么 `setTimeout` 会比 `Promise` 后执行，明明代码写在 Promise 之前。这其实涉及到了 `Event Loop` 相关的知识，这一章节我们会来详细地了解 Event Loop 相关知识，知道 JS 异步运行代码的原理。

<!-- ## 进程与线程

> 涉及面试题：进程与线程区别？JS 单线程带来的好处？

相信大家经常会听到 JS 是`单线程`执行的，但是你是否疑惑过什么是线程？

讲到线程，那么肯定也得说一下进程。本质上来说，两个名词都是 `CPU 工作时间片的一个描述`。

`进程`描述了 CPU 在`运行指令、加载和保存上下文所需的时间`，放在应用上来说就代表了一个程序。`线程`是进程中的更小单位，描述了`执行一段指令所需的时间。`

把这些概念拿到浏览器中来说，当你打开一个 Tab 页时，其实就是创建了一个进程，一个进程中可以有多个线程，比如`渲染线程`、`JS 引擎线程`、`HTTP 请求线程`等等。当你发起一个请求时，其实就是创建了一个线程，当请求结束后，该线程可能就会被销毁。

上文说到了 `JS 引擎线程`和`渲染线程`，大家应该都知道，在 JS 运行的时候可能会阻止 UI 渲染，这说明了两个线程是互斥的。这其中的原因是因为 `JS 可以修改 DOM`，如果在 JS 执行的时候 UI 线程还在工作，就可能导致`不能安全的渲染 UI`。这其实也是一个单线程的好处，得益于 JS 是单线程运行的，可以达到`节省内存`，`节约上下文切换时间`，没有锁的问题的好处。

当然前面两点在服务端中更容易体现，对于锁的问题，形象的来说就是当我读取一个数字 15 的时候，同时有两个操作对数字进行了加减，这时候结果就出现了错误。解决这个问题也不难，只需要在读取的时候加锁，直到读取完毕之前都不能进行写入操作。 -->

## 执行栈

> 涉及面试题：什么是执行栈？

可以把`执行栈`认为是一个`存储函数调用的栈结构`，遵循`先进后出`的原则。

![](https://user-gold-cdn.xitu.io/2018/11/13/1670d2d20ead32ec?imageslim)

当开始执行 JS 代码时，首先会执行一个 main 函数，然后执行我们的代码。根据先进后出的原则，后执行的函数会先弹出栈，在图中我们也可以发现，foo 函数后执行，当执行完毕后就从栈中弹出了。

平时在开发中，大家也可以在报错中找到执行栈的痕迹

```js
function foo() {
    throw new Error('error');
}
function bar() {
    foo();
}
bar();
```

![](https://user-gold-cdn.xitu.io/2018/11/13/1670c0e21540090c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

大家可以在上图清晰的看到报错在 foo 函数，foo 函数又是在 bar 函数中调用的。

当我们使用递归的时候，因为栈可存放的函数是有**限制**的，一旦存放了过多的函数且没有得到释放的话，就会出现爆栈的问题

```js
function bar() {
    bar();
}
bar();
```

![](https://user-gold-cdn.xitu.io/2018/11/13/1670c128acce975f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 浏览器中的 Event Loop

> 涉及面试题：异步代码执行顺序？解释一下什么是 Event Loop ？

`主线程`从 `任务队列`中`读取事件`，这个过程是`循环不断`的，所以整个的这种`运行机制` 称为 Event Loop（事件循环）。

当执行 JS 代码的时候其实就是往`执行栈`中放入函数，当遇到`异步代码`时，会被`挂起`并在需要执行的时候加入到 `Task 队列`中。一旦`执行栈为空`，`Event Loop` 就会从 Task 队列中拿出需要执行的代码并放入执行栈中执行，所以`本质上来说 JS 中的异步还是同步行为`。

![](https://user-gold-cdn.xitu.io/2018/11/23/16740fa4cd9c6937?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

不同的`任务源`会被分配到不同的 Task 队列中，任务源分为 **微任务**（microtask） 和 **宏任务**（macrotask）(`微任务` 和 `宏任务` 表示 `异步任务` 的两种分类)。

在 ES6 规范中，microtask 称为 jobs，macrotask 称为 task。下面来看以下代码的执行顺序：

```js
console.log('script start')         1

async function async1() {
    await async2()
    console.log('async1 end')       7
}
async function async2() {
    console.log('async2 end')       2
}
async1()

setTimeout(function() {
    console.log('setTimeout')       8
}, 0)

new Promise(resolve => {
    console.log('Promise')          3
    resolve()
})
    .then(function() {
        console.log('promise1')     5
    })
    .then(function() {
        console.log('promise2')     6
    })

console.log('script end')           4
// script start => async2 end => Promise => script end => promise1 => promise2 => async1 end => setTimeout
```

<Alert type='info'>
注意：新的浏览器中不是如上打印的，因为 await 变快了，具体内容可以往下看
</Alert>

首先先来解释下上述代码的 async 和 await 的执行顺序。当我们调用 async1 函数时，会马上输出 async2 end，并且函数返回一个 Promise，接下来在遇到 await 的时候会就让出线程开始执行 async1 外的代码，所以我们完全可以把 await 看成是让出线程的标志。

然后当同步代码全部执行完毕以后，就会去执行所有的异步代码，那么又会回到 await 的位置执行返回的 Promise 的 resolve 函数，这又会把 resolve 丢到微任务队列中，接下来去执行 then 中的回调，当两个 then 中的回调全部执行完毕以后，又会回到 await 的位置处理返回值，这时候你可以看成是 Promise.resolve(返回值).then()，然后 await 后的代码全部被包裹进了 then 的回调中，所以 console.log('async1 end') 会优先执行于 setTimeout。

如果你觉得上面这段解释还是有点绕，那么我把 async 的这两个函数改造成你一定能理解的代码

```js
new Promise((resolve, reject) => {
    console.log('async2 end');
    // Promise.resolve() 将代码插入微任务队列尾部
    // resolve 再次插入微任务队列尾部
    resolve(Promise.resolve());
}).then(() => {
    console.log('async1 end');
});
```

所以 Event Loop 执行顺序如下所示：

-   首先执行同步代码，这属于宏任务
-   当执行完所有同步代码后，执行栈为空，查询是否有异步代码需要执行
-   执行所有微任务
-   当执行完所有微任务后，如有必要会渲染页面
-   然后开始下一轮 Event Loop，执行宏任务中的异步代码，也就是 setTimeout 中的回调函数
-   所以以上代码虽然 `setTimeout` 写在 `Promise` 之前，但是因为 `Promise` 属于微任务而 `setTimeout` 属于宏任务，所以会有以上的打印。

微任务包括 `promise.then()` ，`process.nextTick` ，`MutationObserver`，其中 `process.nextTick` 为 Node 独有。

宏任务包括 `script` ， `setTimeout` ，`setInterval` ，`setImmediate` ，I/O ，UI rendering。

这里很多人会有个误区，认为微任务快于宏任务，其实是错误的。因为宏任务中包括了 `script` ，浏览器会先执行一个宏任务，接下来有异步代码的话才会先执行微任务。

## 练习题

```js
setTimeout(function () {
    console.log('setTimeout');
}, 0);

new Promise(function (resolve) {
    console.log('promise');
    resolve();
}).then(function () {
    console.log('then');
});

console.log('console');

// "promise"、"console"、"then"、"setTimeout"
```

执行顺序解析如下：

-   这段代码作为`宏任务`，进入`主线程`。
-   先遇到 `setTimeout`，那么将其回调函数注册后分发到`宏任务`
-   接下来遇到了 `Promise`，`new Promise 立即执行`，`then` 函数分发到`微任务`
-   遇到 console.log()，立即执行。
-   好啦，整体代码 `script` 作为第一个宏任务执行结束，看看有哪些微任务？我们发现了 then 在微任务里面，执行。
-   ok，第一轮事件循环结束了，我们开始第二轮循环，当然要从宏任务开始。我们发现了宏任中 `setTimeout` 对应的回调函数，立即执行。
-   结束。

```js
const promise = new Promise((resolve, reject) => {
    console.log(1);

    setTimeout(() => {
        console.log('timeStart');

        resolve('success');

        console.log('timeEnd');
    }, 0);

    console.log(2);
});

promise.then((res) => {
    console.log(res);
});

console.log(4);

// 1、2、4、"timeStart"、"timeEnd"、"success"
```

```js
setTimeout(() => {
    console.log('timer1');

    setTimeout(() => {
        console.log('timer3');
    }, 0);
}, 0);

setTimeout(() => {
    console.log('timer2');
}, 0);

console.log('start');

// "start"、"timer1"、"timer2"、"timer3"
```

```js
setTimeout(() => {
    console.log('timer1');

    Promise.resolve().then(() => {
        console.log('promise');
    });
}, 0);

setTimeout(() => {
    console.log('timer2');
}, 0);

console.log('start');

// "start"、"timer1"、"promise"、"timer2"
```
