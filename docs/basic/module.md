---
toc: content
order: 10
---

# 前端模块化

> 为什么要使用模块化？都有哪几种方式可以实现模块化，各有什么特点？

使用一个技术肯定是有原因的，那么使用模块化可以给我们带来以下好处

-   解决命名冲突
-   提供复用性
-   提高代码可维护性

## 立即执行函数

在早期，使用`立即执行函数`实现模块化是常见的手段，通过函数作用域解决了`命名冲突`、`污染全局作用域`的问题。

```js
(function (globalVariable) {
    globalVariable.test = function () {};
    // ... 声明各种变量、函数都不会污染全局作用域
})(globalVariable);
```

## AMD 和 CMD

鉴于目前这两种实现方式已经很少见到，所以不再对具体特性细聊，只需要了解这两者是如何使用的。

```js
// AMD (require.js)
define(['./a', './b'], function (a, b) {
    // 加载模块完毕可以使用
    a.do();
    b.do();
});
// CMD (sea.js)
define(function (require, exports, module) {
    // 加载模块
    // 可以把 require 写在函数体的任意地方实现延迟加载
    var a = require('./a');
    a.doSomething();
});
```

## CommonJS

CommonJS 最早是 Node 在使用，目前也仍然广泛使用，比如在 Webpack 中你就能见到它，当然目前在 Node 中的模块管理已经和 CommonJS 有一些区别了。

```js
// a.js
module.exports = {
    a: 1,
};
// or
exports.a = 1;

// b.js
var module = require('./a.js');
module.a; // -> log 1
```

因为 CommonJS 还是会使用到的，所以这里会对一些疑难点进行解析

先说 require 吧

```js
var module = require('./a.js');
module.a;
// 这里其实就是包装了一层立即执行函数，这样就不会污染全局变量了，
// 重要的是 module 这里，module 是 Node 独有的一个变量
module.exports = {
    a: 1,
};
// module 基本实现
var module = {
    id: 'xxxx', // 我总得知道怎么去找到他吧
    exports: {}, // exports 就是个空对象
};
// 这个是为什么 exports 和 module.exports 用法相似的原因
var exports = module.exports;
var load = function (module) {
    // 导出的东西
    var a = 1;
    module.exports = a;
    return module.exports;
};
// 然后当我 require 的时候去找到独特的
// id，然后将要使用的东西用立即执行函数包装下，over
```

另外虽然 exports 和 module.exports 用法相似，但是不能对 exports 直接赋值。因为 var exports = module.exports 这句代码表明了 exports 和 module.exports 享有相同地址，通过改变对象的属性值会对两者都起效 (module.exports)，但是如果直接对 exports 赋值就会导致两者不再指向同一个内存地址，修改并不会对 module.exports 起效。

## ES Module

在 Es Module 中导出分为两种，单个导出(export)、默认导出(export default)，允许导出任意类型的值，通过 import 关键字导入。

```js
// index.js
export const name = '蛙人';
export const age = 24;

import { name, age } from './index.js';
console.log(name, age); // "蛙人" 24

// 如果里面全是单个导出，我们就想全部直接导入则可以这样写
import * as all from './index.js';
console.log(all); // {name: "蛙人", age: 24}
```

可以使用 export 和 export default 同时使用并且互不影响，只需要在导入时地方注意，如果文件里有混合导入，则必须先导入默认导出的，再导入单个导入的值。

```js
export const name = "蛙人"
export const age = 24

export default {
    fn() {}，
    msg: "hello 蛙人"
}
```

ES Module 是 ES6 提出的模块化方案，与 CommonJS 有以下几个区别:

-   CommonJS 支持`动态导入`，也就是 `require`(${path}/xx.js)，前者目前不支持，但是已有提案。
-   CommonJS 是`同步导入`，因为用于服务端，文件都在本地，同步导入即使卡住主线程影响也不大。而前者是异步导入，因为用于浏览器，需要下载文件，如果也采用同步导入会对渲染有很大影响。
-   CommonJS 在导出时都是`值拷贝`，就算导出的值变了，导入的值也不会改变，所以如果想更新值，必须重新导入一次。但是 ES Module 采用实时绑定的方式，导入导出的值都指向同一个`内存地址`，所以导入值会跟随导出值变化。
-   CommonJS 模块是`运行时加载`，ES6 模块是`编译时输出接口`。因为 CommonJS 加载的是一个对象（即module.exports属性），该对象只有在脚本运行完才会生成。而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。
