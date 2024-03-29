---
toc: content
order: 1
---

# JavaScript

## 数据类型

JavaScript 是一种弱类型脚本语言，在定义变量时不需要指定类型，在程序运行过程中会自动判断类型。

> 涉及面试题：原始类型有哪几种？null 是对象嘛？

JavaScript 内置类型分为两大类型：**基本类型** 和 **引用类型**。

基本类型有七种： `Number`，`String`，`Boolean`，`Undefined`，`Null`，[Symbol](/basic/es6#symbol)，[BigInt](/basic/es6#bigint)

引用类型包括： `Object` ， `Array` ， `Date` ， `function` ，对象在使用过程中会遇到浅拷贝和深拷贝的问题。

```js
// 下列代码会出现：对象 a 的 name 属性被误改的 bug

let a = {
    name: 'zfc',
};
let b = a;
b.name = '张芳朝';

console.log(a.name); // 张芳朝
console.log(b.name); // 张芳朝

// 解决方式：先对 a 拷贝一份数据，再赋值给 b，即可

let a = {
    name: 'zfc-zfc',
};
let b = { ...a };
b.name = '张芳朝-张芳朝';

console.log(a.name); // 'zfc-zfc'
console.log(a.name); // '张芳朝-张芳朝'
```

### typeof

> typeof 是否能正确判断类型？instanceof 能正确判断对象的原理是什么？

`typeof` 对于基本类型，除了 `null` 都可以显示正确的类型

```js
typeof 1; // 'number'
typeof '1'; // 'string'
typeof undefined; // 'undefined'
typeof true; // 'boolean'
typeof Symbol(); // 'symbol'
typeof b; //  b 没有声明，但是还会显示 'undefined'
```

对于 `null` 来说，虽然它是基本类型，但是会显示 `object` ，这是一个存在很久了的 `Bug`

```js
typeof null; // 'object'
```

**为什么会出现这种 `Bug`呢？**

因为在 JS 的最初版本中，使用的是 32 位系统，为了性能考虑使用低位存储了变量的类型信息， `000` 开头表是对象，然而 `null` 表示为全零，所以将它错误的判断为 `object` 。虽然现在的内部类型判断代码已经变了，但是对于这个 Bug 却是一直流传下来。

`typeof` 对于引用类型，除了函数都会显示 `object`

```js
typeof []; // 'object'
typeof {}; // 'object'
typeof new Date(); // 'object'
typeof console.log; // 'function'
```

`typeof NaN` ????

`NaN` Not a Number 的缩写，表示`非数字`。常见于字符串和数字运算的结果。

```js
5 - 'a'; // NaN
0 / 0; // NaN
```

需要注意的是: `NaN` 不是独立的 `数据类型`，而是一个`特殊数值`，它的数据类型依然属于 `Number`。

```js
typeof NaN; // "number"
```

NaN 不等于任何值，包括它本身。所以 NaN 也是唯一一个和自身不严格相等的值。

```js
NaN === NaN; // false
```

数组的 indexOf 方法内部使用的是`严格相等`运算符，所以该方法对 NaN 不成立。

```js
[NaN].indexOf(NaN); // -1
```

NaN 在布尔运算时被当作 false。

```js
Boolean(NaN); // false
```

NaN 与任何数（包括它自己）的运算，得到的都是 NaN。

```js
NaN + 32; // NaN
NaN - 32; // NaN
NaN * 32; // NaN
NaN / 32; // NaN
```

### constructor

Object 的每个实例都有构造函数 constructor，用来保存创建当前对象的函数。

```js
let arr = [];
console.log(arr.constructor === Array); // true
```

需要注意的是，constructor 有被修改的风险，判断结果不一定准确，比如：

```js
let arr = [1, 2, 3];
arr.constructor = function () {};
console.log(arr.constructor === Array); // false
```

一般不推荐使用 constructor 来判断是否为数组，我们只需要知道有这么一个方法就行。

### instanceof

`instanceof` 可以正确的判断引用类型的类型，因为内部机制是 **通过判断对象的原型链中能否找到 构造函数的 prototype 属性**。

```js
// object instanceof constructor    左边是要测试的对象，右边是构造函数

{} instanceof Object                // true
[] instanceof Array                 // true
[] instanceof Object                // true
function() {} instanceof Function   // true
function() {} instanceof Object     // true
```

**instanceof 实现原理：**

```js
function instanceof(left, right) {
    // left 表示 instanceof 左边的 object，right 表示右边 constructor
    left = left.__proto__; // 获得对象的原型

    let prototype = right.prototype; // 获得类型的原型

    // 判断对象的类型是否等于类型的原型
    while (true) {
        // while(true)作为无限循环，经常在不知道循环次数的时候使用
        if (left === null) {
            return false; //已经找到顶层
        }
        if (left === prototype) {
            return true; //当 left 严格等于 prototype 时，返回 true
        }
        left = left.__proto__; //继续向上一层原型链查找
    }
}
```

instanceof 的弊端：

-   构造函数的 prototype 和 实例的原型链都有可能会改变，所以判断出的结果不一定一成不变。
-   在有 iframe 的页面脚本中使用 instanceof，可能会得到错误的结果，因为 iframe 拥有独立的全局环境，不同的全局环境拥有不同的全局对象，从而拥有不同的内置类型构造函数。

### Array.isArray

Array.isArray() 是 ES5 新增的方法，用于确定传递的值是否是一个数组，如果是数组，则返回 true，否则返回 false。

```js
let arr = [];
console.log(Array.isArray(arr)); // true
```

需要注意的一点是：其实 Array.prototype 也是一个数组。

```js
Array.isArray(Array.prototype); // true
```

### Object.prototype.toString.call(xx)

使用 `Object.prototype.toString.call(xx)` 可以获得变量的正确类型。

痛过调用原型链顶端的 toString 方法实现；这里用 call 是为了改变 toString 函数内部的 this 指向，其实也可以用 apply。如果不把 this 指向为我们的目标变量，this 将永远指向调用 toString 的 prototype。

为什么一定要调用原型链顶端的 toString 方法呢？????
因为，对象和数组本身是可以改写 toString 的，所以才调用原型链顶端的 toString 方法，代码如下；

```js
// 改写 toString 前

var obj = {};

console.log(obj.toString() === '[object Object]'); // true

// 改写后

var obj = {
    toString: function () {
        return 'xxx'; // 改写 toString
    },
};

console.log(obj.toString() === '[object Object]'); // false
```

```js
Object.prototype.toString.call(12); // "[object Number]"
Object.prototype.toString.call('12'); // "[object String]"
Object.prototype.toString.call(true); // "[object Boolean]"
Object.prototype.toString.call(undefined); // "[object Undefined]"
Object.prototype.toString.call(Null); // "[object Null]"
Object.prototype.toString.call(12n); // "[object BigInt]"
Object.prototype.toString.call(Symbol()); // "[object Symbol]"

Object.prototype.toString.call({}); // "[object Object]"
Object.prototype.toString.call([]); // "[object Array]"
Object.prototype.toString.call(new Date()); // "[object Date]"
Object.prototype.toString.call(console.log); // "[object Function]"
```

> 涉及面试题：基本类型和引用类型的不同之处？函数参数是对象会发生什么问题？

基本类型和引用类型不同的是: 基本类型存储的是值，引用类型存储的是地址（指针）。当你创建了一个对象类型的时候，计算机会在内存中帮我们开辟一个空间来存放值，但是我们需要找到这个空间，这个空间会拥有一个地址（指针）。

## var let const

var

-   ES5 命令
-   没有块级作用域的概念
-   会变量提升
-   未声明之前可以调用，值为 undefined
-   能重复声明
-   声明的全局变量会挂在顶层对象(window)下面

let / const

-   ES6 命令
-   有块级作用域概念
-   不会变量提升，因为暂时性死区而报错 (在声明之前调用就处于暂时性死区)
-   不能重复声明
-   声明的全局变量不会挂在顶层对象(window)下面

const

-   声明之后必须马上赋值，否则会报错
-   基本类型一旦声明不允许修改
-   引用类型 指针指向的地址不能修改，内部数据可以修改

```js
const person = {
    name: 'zfc',
    age: 18,
};
person.age = 19; // 没问题
person = {}; // 报错 不能修改对象指针
```

## get 和 post

-   get 用来获取数据，post 用来提交数据。
-   get 请求在 url 中传送的参数是有长度限制的(最长 2048 字节)，而 post 么有。
-   get 参数通过 url 传递，所以不能用来传递敏感信息，post 放在 Request body 中。

get 和 post 还有一个重大区别，简单的说：

`get 产生一个 TCP 数据包；post 产生两个 TCP 数据包。`

因为 post 需要两步，时间上消耗的要多一点，看起来 get 比 post 更有效。

1、get 与 post 都有自己的语义，不能随便混用。

2、在网络环境好的情况下，发一次包的时间和发两次包的时间差别基本可以无视。
而在网络环境差的情况下，两次包的 TCP 在验证数据包完整性上，有非常大的优点。

3、并不是所有浏览器都会在 post 中发送两次包，Firefox 就只发送一次。

## for in 和 for of

> for in 循环是用于遍历对象的，它可以用来遍历数组吗？

答案是 可以的，因为数组在某种意义上也是对象，但是如果用其遍历数组会存在一些隐患：`其会遍历数组原型链上的属性`。

-   for in 遍历对象场景：

```js
let obj = {
    name: 'zfc',
    age: 18,
};

for (let key in obj) {
    console.log(key); // name age
    console.log(obj[key]); // zfc 18
}

// 但是如果在 Object 原型链上添加一个方法，会遍历到原型链上的方法
Object.prototype.test = function () {};

for (let key in obj) {
    console.log(key); // name age test
    console.log(obj[key]); // zfc 18 ƒ () {}
}

// hasOwnProperty 方法可以判断某属性是否是该对象的实例属性
for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
        console.log(key); // name age
        console.log(obj[key]); // zfc 18
    }
}
```

-   for in 遍历数组场景：

```js
let arr = [1, 2];

for (let key in arr) {
    console.log(key); // 会打印数组的 下标 0, 1
    console.log(arr[key]); // 会打印数组的 元素 1, 2
}

// 但是如果在 Array 原型链上添加一个方法，
Array.prototype.test = function () {};

for (let key in arr) {
    console.log(arr[key]); // 此时会打印 1, 2, ƒ () {}
}
```

因为我们不能保证项目代码中不会对`数组原型链`进行操作，也不能保证引入的第三方库不对其进行操作，所以不要使用 for in 循环来遍历数组。

-   for of 只能遍历数组（不包括数组原型链上的属性和方法）

```js
let arr = [1, 2];

for (let value of arr) {
    console.log(value); // 会打印数组的 元素 1, 2
}
```

因为能够被 for...of 正常遍历的，都需要实现一个遍历器 `Iterator`。而数组、字符串、Set、Map 结构，早就内置好了 Iterator（迭代器），它们的原型中都有一个 Symbol.iterator 方法，而 Object 对象并没有实现这个接口，使得它无法被 for...of 遍历。

如何让对象可以被 for of 遍历，当然是给它添加遍历器，代码如下：

```js
Object.prototype[Symbol.iterator] = function () {
    let _this = this;
    let index = 0;
    let length = Object.keys(_this).length;
    return {
        next: () => {
            let value = _this[index];
            let done = index >= length;
            index++;
            return { value, done };
        },
    };
};
```

<Alert type="warning">
整体来说直接用来遍历对象的目前只有 for in，其他的都是遍历数组用的。
</Alert>

## 数组去重

**1、双重 for 循环 (如果前一个值与后一个值相等，那么就去掉后一个值)**

```js
const arr = [1, 'a', 'a', 'b', 'd', 'e', 'e', 1, 0];

let test = (arr) => {
    for (let i = 0, len = arr.length; i < len; i++) {
        for (let j = i + 1, len = arr.length; j < len; j++) {
            if (arr[i] === arr[j]) {
                arr.splice(j, 1);
            }
        }
    }
    return arr;
};
test(arr); // [1, "a", "b", "d", "e", 0]
```

2、for...of + includes()

双重 for 循环的升级版，外层用 for...of 语句替换 for 循环，把内层循环改为 includes()

先创建一个空数组，当 includes() 返回 false 的时候，就将该元素 push 到空数组中

类似的，还可以用 indexOf() 来替代 includes()

```js
let arr = [1, 'a', 'a', 'b', 'd', 'e', 'e', 1, 0];

let test = (arr) => {
    let result = [];
    for (let i of arr) {
        !result.includes(i) && result.push(i);
    }
    return result;
};

test(arr); // [1, "a", "b", "d", "e", 0]
```

3、Array.filter() + indexOf

```js
let arr = [1, 1, 2, 2, 2, 6];

let test = () => {
    return arr.filter((item, index, array) => array.indexOf(item) === index);
};

test(arr); // [1, 2, 6]
```

4、for...of + Object

首先创建一个空对象，然后用 for 循环遍历。利用对象的属性不会重复这一特性，校验数组元素是否重复。

```js
let arr = [1, 'a', 'a', 'b', 'd', 'e', 'e', 1, 0];

let test = () => {
    let result = [];
    let obj = {};
    for (let i of arr) {
        if (!obj[i]) {
            result.push(i);
            obj[i] = 1;
        }
    }
    return result;
};

test(); // [1, "a", "b", "d", "e", 0]
```

5、ES6 Set

```js
let arr = [1, 'a', 'a', 'b', 'd', 'e', 'e', 1, 0];

let test = (arr) => Array.from(new Set(arr));

test(arr); // [1, "a", "b", "d", "e", 0]
```

### reduce 实现数组去重

```js
const uniq = (arr) => {
    let result = [];
    result = arr.reduce((pre, cur) => {
        if (!pre.includes(cur)) {
            console.log('pre--cur', pre, cur);
            return pre.concat(cur);
        } else {
            return pre;
        }
    }, []);

    return result;
};

console.log('-----', uniq([1, 1, 1, 2, 2, 2, 3, 3, 4, 4, 5, 6])); // [1, 2, 3, 4, 5, 6]
```

## 数组排序

数组排序比较常用的：`冒泡排序`、`快速排序`

-   冒泡排序：

从数组中随便拿一个数与后一位比较，如果前者比后者大，那么两者交换位置，从而遍历数组可以得到排序的效果。

```js
let arr = [1, 9, 4, 50, 49, 6, 3, 2];
let test = (arr) => {
    for (let i = 0, len = arr.length; i < len - 1; i++) {
        for (let j = i + 1, len = arr.length; j < len; j++) {
            let tempi = arr[i]; // 获取第一个值，并与后一个值比较
            let tempj = arr[j];
            if (tempi > tempj) {
                arr[i] = tempj;
                arr[j] = tempi; // 如果前一个值比后一个值大，那么相互交换
            }
        }
    }
    console.log(arr); // [1, 2, 3, 4, 6, 9, 49, 50]
};
test(arr);
```

-   快速排序：

找出数组中间那一个值，然后用这个值跟数组里面的值相比较，大于此值的放在一边，小于的也放在一边， 然后用 concat()合并，再进行比较，如此反复。

```js
let arr = [1, 9, 4, 50, 49, 6, 3, 2];
let test = (arr) => {
    if (arr.length <= 1) return arr; // 如果数组只有一位，就没有必要比较了
    let index = Math.floor(arr.length / 2); // 获取中间值的索引
    let cur = arr.splice(index, 1); // 截取中间值，如果此处使用 cur=arr[index]; 那么将会出现无限递归的错误
    let left = [];
    let right = []; // 小于中间值的放在left数组里，大于的放在right数组
    for (let i = 0, len = arr.length; i < len; i++) {
        if (cur > arr[i]) {
            left.push(arr[i]);
        } else {
            right.push(arr[i]);
        }
    }
    return test(left).concat(cur, test(right)); // 通过递归，上一轮比较好的数组合并，并且再次进行比较
};
test(arr);
```

## 函数式编程

简单说，"函数式编程" 是一种 "编程范式"，也就是如何编写程序的方法论。

属于 "结构化编程" 的一种，主要思想是`把运算过程`尽量写成一系列`嵌套的函数调用`。举例来说，现在有这样一个数学表达式：

```js
(1 + 2) * 3 - 4;

// 传统的过程式编程，可能这样写：
var a = 1 + 2;
var b = a * 3;
var c = b - 4;

// 函数式编程要求使用函数，我们可以把运算过程定义为不同的函数，然后写成下面这样：
var result = subtract(multiply(add(1, 2), 3), 4);
```

这就是函数式编程。

函数式编程优点

-   代码简洁，开发快速

```js
函数式编程大量使用函数，减少了代码的重复，因此程序比较短，开发速度较快。
```

-   接近自然语言，易于理解

```js
函数式编程的自由度很高，可以写出很接近自然语言的代码。

上文 subtract(multiply(add(1,2), 3), 4)

// 可以变形为

add(1,2).multiply(3).subtract(4)

// 这基本就是自然语言的表达了。
```

-   更方便的代码管理

```js
函数式编程不依赖、也不会改变外界的状态，只要给定输入参数，返回的结果必定相同。
因此，每一个函数都可以被看做独立单元，很有利于进行单元测试和除错，以及模块化组合。
```

## 函数柯里化

所谓 "柯里化"，就是把一个`多参数的函数`，`转化为单参数函数`。

```js
// 柯里化之前
function add(x, y) {
    return x + y;
}

add(1, 2); // 3

// 柯里化之后
function addX(y) {
    return function (x) {
        return x + y;
    };
}

addX(2)(1); // 3
```

被`柯里化`的函数 addX 每次的返回值都为一个函数，并使用下一个参数作为形参，直到 2 个参数都被传入后，在返回的最后一个函数内部`执行`求和操作，其实是充分的利用了`闭包`的特性来实现的。

柯里化的一个很大的好处是可以帮助我们基于一个被转换函数，通过对参数的拆分实现不同功能的函数，

## 原型和原型链

- 每个实例对象都有一个 proto 属性，该属性指向自己的原型对象
- 每个构造函数都有一个 prototype 属性，该属性指向实例对象的原型对象
- 原型对象里的 constructor 指向构造函数本身。


当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾null，这样一个查找过程叫原型链。
## 执行上下文

当执行 JS 代码时，会产生三种执行上下文

-   全局执行上下文 (浏览器中，全局上下文是 window 对象，node 环境中 是 global)
-   函数执行上下文
-   eval 执行上下文（耗性能）

每个执行上下文中都有三个重要的属性

-   变量对象（VO），包含变量、函数声明和函数的形参，该属性只能在全局上下文中访问
-   作用域链（JS 采用词法作用域，也就是说变量的作用域是在定义时就决定了）
-   this 指向

```js
console.log(a) // undefined
var a = 100

fn('zhangsan') // 'zhangsan' 20
function fn(name) {
    age = 20;
    console.log(name, age);
    var age;
}

console.log(b); // 这里报错
// Uncaught ReferenceError: b is not defined
b = 100...
```

对于上述代码，执行栈中有两个上下文：全局上下文和函数 `fn` 上下文。

我们来看下上面的面试小题目，为什么 a 是 undefined，而 b 却报错了，实际 JS 在代码执行之前，要「全文解析」，发现 var a，知道有个 a 的变量，存入了执行上下文，而 b 没有找到 var 关键字，这时候没有在执行上下文提前「占位」，所以代码执行的时候，提前报到的 a 是有记录的，只不过值暂时还没有赋值，即为 undefined，而 b 在执行上下文没有找到，自然会报错（没有找到 b 的引用）。

另外，一个函数在执行之前，也会创建一个 函数执行上下文 环境，跟 全局上下文 差不多，不过 函数执行上下文 中会多出 this arguments 和函数的参数。

## 防抖

你是否在日常开发中遇到一个问题，在滚动事件中需要做个复杂计算或者实现一个按钮的防二次点击操作。

这些需求都可以通过函数防抖动来实现。尤其是第一个需求，如果在频繁的事件回调中做复杂计算，很有可能导致页面卡顿，不如将多次计算合并为一次计算，只在一个精确点做操作。

PS：防抖和节流的作用都是防止函数多次调用。区别在于，假设一个用户一直触发这个函数，且每次触发函数的间隔小于 delay 情况会每隔一定时间（参数 delay）调用函数。

```js
// fn是我们需要包装的事件回调, delay是每次推迟执行的等待时间

function debounce(fn, delay) {
    let timer = null; // 定时器

    // 将debounce处理结果当作函数返回
    return () => {
        let self = this; // 保留调用时的this上下文
        let args = arguments; // 保留调用时传入的参数

        if (timer) clearTimeout(timer); // 每次事件被触发时，都去清除之前的旧定时器

        // 设立新定时器
        timer = setTimeout(() => {
            fn.apply(self, args);
        }, delay);
    };
}

// 用debounce来包装scroll的回调
document.addEventListener(
    'scroll',
    debounce(() => console.log('触发了滚动事件'), 1000),
);
```

## 节流

节流和防抖动本质是不一样的。防抖动是将多次执行变为最后一次执行，节流是将多次执行变成每隔一段时间执行。

```js
// fn是我们需要包装的事件回调, time是时间间隔的阈值

function throttle(fn, time) {
    let last = 0; // last为上一次触发回调的时间

    // 将throttle处理结果当作函数返回
    return () => {
        let self = this; // 保留调用时的this上下文
        let args = arguments; // 保留调用时传入的参数
        let now = +new Date(); // 记录本次触发回调的时间

        // 判断本次触发的时间和上次触发的时间差是否大于时间间隔的阈值
        // 如果时间间隔大于我们设定的时间间隔阈值，则执行回调
        if (now - last > time) {
            fn.apply(self, args);
            last = now;
        }
    };
}

// 用throttle来包装scroll的回调
document.addEventListener(
    'scroll',
    throttle(() => console.log('触发了滚动事件'), 1000),
);
```

## 控制台打印当前时间，间隔一秒，格式 YYYY-MM-DD hh:mm:ss

```js
const toStr = (num = '--') => {
    return num.toString().padStart(2, '0');
};

const onTimer = () => {
    const date = new Date();

    const year = date.getFullYear();
    const month = toStr(date.getMonth() + 1);
    const day = toStr(date.getDate());

    const hour = toStr(date.getHours());
    const min = toStr(date.getMinutes());
    const second = toStr(date.getSeconds());

    return `${year}-${month}-${day} ${hour}:${min}:${second}`;
};

setInterval(() => {
    console.log('当前时间是:', onTimer());
}, 1000);
```

## 找出字符串中出现最多的字母，以及出现的次数

```js
const str = 'abbcccc';

const getFun = (str) => {
    const arr = str.split('');

    return arr.reduce((pre, cur) => {
        // console.log('pre--cur', pre, cur)
        // cur：当前项，是个字符串：a，或者b，c
        // pre：上次计算出来的结果，是一个对象 类似：{a: 3, b: 6, c: 9}
        if (cur in pre) {
            pre[cur] += 1;
        } else {
            pre[cur] = 1;
        }
        return pre;
    }, {});
};

const maxObj = getFun(str);

let maxStr = null; // 出现次数最多的字符串
let max = 0; // 出现次数最多的字符串，出现的次数

for (let key in maxObj) {
    // 冒泡排序
    // 假设 max 小于 maxObj[key]
    // 说明假设错误，max是最大的，则给max重新赋值
    if (max < maxObj[key]) {
        max = maxObj[key];
        maxStr = key;
    }
}

console.log(`出现次数最多的是：${maxStr}，出现的次数是：${max}`);
// 出现次数最多的是：c，出现的次数是：4
```

## 获取指定范围的随机数

```js
const random = (min, max) => {
    const range = max - min;
    const random = Math.random();
    return min + Math.round(random * range);
};
console.log('---', random(1, 10));
```

### 二维数组扁平化

```js
let arr = [
    [1, 2],
    [3, 4],
    [5, 6],
];

const flat = (arr) => {
    return arr.reduce((pre, cur) => {
        return pre.concat(cur);
    }, []);
};

console.log(flat(arr));
```

### 多维数组扁平化

```js
let arr = [
    [1, 2],
    [3, 4],
    [5, 6, [7, 8, 9]],
];

const flat = (arr) => {
    return arr.reduce((pre, cur) => {
        if (Array.isArray(cur)) {
            return pre.concat(flat(cur));
        } else {
            return pre.concat(cur);
        }
    }, []);
};

console.log('flat(arr)---', flat(arr)); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### 求和：参数不固定

```js
function sum() {
    let total = 0;
    let len = arguments.length;

    for (let i = 0; i < len; i++) {
        total += arguments[i];
    }

    return total;
}

console.log('求和：参数不固定：', sum(1, 2, 3, 4)); // 10
```

### 对象格式数据求和

```js
let arr = [
    {
        subject: 'chinese',
        score: 100,
    },
    {
        subject: 'math',
        score: 50,
    },
    {
        subject: 'english',
        score: 30,
    },
];

const sum = (arr) => {
    return arr.reduce((pre, cur) => {
        return pre + cur.score;
    }, 0);
};

console.log('求和=', sum(arr)); // 180
```

## 设计模式

### 单例模式

单例模式：`保证一个类仅有一个实例，并提供一个访问它的全局访问点`。

实现的方法为：先判断实例存在与否，如果存在则直接返回，如果不存在就创建了再返回，这就确保了一个类只有一个实例对象。

适用场景：一个单一对象。比如：弹窗，无论点击多少次，弹窗只应该被创建一次。

```js
// 1、class 实现单例模式
class CreateUser {
    constructor(name) {
        this.name = name;
        this.getName();
    }
    getName() {
        return this.name;
    }
}
// 代理实现单例模式
var ProxyMode = (function () {
    var instance = null;
    return function (name) {
        if (!instance) {
            instance = new CreateUser(name);
        }
        return instance;
    };
})();
// 测试单体模式的实例
var a = new ProxyMode('aaa');
var b = new ProxyMode('bbb');
// 因为单体模式是只实例化一次，所以下面的实例是相等的
console.log(a === b); //true

// 2、闭包实现单例模式
<button id="login">登录</button>;

var createLoginLayer = (function () {
    var div = null;

    return function () {
        if (!div) {
            div = document.createElement('div');
            div.innerHTML = '我是登录弹框';
            div.style.display = 'none';
            document.body.append(div);
        }
        return div;
    };
})();

document.getElementById('login').onclick = function () {
    var loginLayer = createLoginLayer();
    loginLayer.style.display = 'block';
};
```

### 代理模式

> 是为一个对象提供一个代用品或占位符，以便控制对它的访问。

1、HTML 事件代理

如果一个节点中的子节点是动态生成的，那么子节点需要注册事件的话应该注册在父节点上。

```js
<ul id="ul">
	<li>1</li>
    <li>2</li>
	<li>3</li>
</ul>
<script>
	let ul = document.querySelector('#ul')
	ul.addEventListener('click', (event) => {
		console.log(event.target);
	})
</script>
```

事件代理的方式相较于直接给目标注册事件来说，有以下优点：

-   节省内存
-   不需要给子节点注销事件

2、图片懒加载的方式：先通过一张 loading 图占位，然后通过异步的方式加载图片，等图片加载好了再把完成的图片加载到 img 标签里面。

### 工厂模式

> 工厂模式定义一个用于创建对象的接口，这个接口由子类决定实例化哪一个类。该模式使一个类的实例化延迟到了子类。而子类可以重写接口方法以便创建的时候指定自己的对象类型。

![](/images/basic/gongchangmoshi.png)

### 观察者模式

> 观察者模式，它定义了一种一对多的关系，让多个观察者对象同时监听某一个主题对象，这个主题对象的状态发生变化时就会通知所有的观察者对象，使得它们能够自动更新自己。

在观察者模式中有两个主要角色：Subject（主题）和 Observer（观察者）。

![](/images/basic/guanchazhe.png)

在上图中，Subject（主题）就是阿宝哥的 TS 专题文章，而观察者就是小秦和小王。由于观察者模式支持简单的广播通信，当消息更新时，会自动通知所有的观察者。

### 发布订阅模式

在软件架构中，发布/订阅是一种消息范式，消息的发送者（称为发布者）不会将消息直接发送给特定的接收者（称为订阅者）。而是将发布的消息分为不同的类别，然后分别发送给不同的订阅者。 同样的，订阅者可以表达对一个或多个类别的兴趣，只接收感兴趣的消息，无需了解哪些发布者存在。
在发布订阅模式中有三个主要角色：Publisher（发布者）、 Channels（通道）和 Subscriber（订阅者）。

![](/images/basic/fabudingyue.png)
