---
toc: content
order: 10
---

# Vue 和 React 的区别

## 设计思想

`React` 是 `facebook` 团队 在 2013 年 开源的前端框架。推荐使用 `JSX语法` 构建项目 主张 `函数式编程`，所以推崇`纯组件`，`数据不可变`，`单向数据流`，当然需要双向的地方也可以手动实现，比如借助 `setState` 来实现一个双向的数据流。

`Vue` 是 `尤雨溪` 团队在 2015 年 开源的一个 `渐进式` JavaScript 框架。 所谓渐进式 在声明式渲染的基础上，我们可以选择性的通过添加 客户端`路由`（vue-router）、大规模`状态管理`（vuex）来构建一个完整的框架。主要是使用简单的 `模版语法` 构建项目 以 较低的入门门槛 灵活和高效的特点 吸引了一大批粉丝 。

Vue 是基于`可变数据`的，通过 `v-model` 指令来实现`数据流双向绑定`。

## diff 算法区别

### React 递增法

React 的思路是`递增法`。通过对比新的列表中的节点，在原本的列表中的位置是否是递增，来判断当前节点是否需要移动。

这里是有可优化的空间的，接下来我们介绍 vue2.x 中的 diff 算法 —— 双端比较，该算法解决了上述的问题。

### Vue2 双端比较

`所谓双端`比较就是`新列表`和`旧列表`两个列表的`头与尾`互相对比，在对比的过程中指针会逐渐向内靠拢，直到某一个列表的节点全部遍历过，对比停止。

### Vue3 最长递增子序列

`Vue3` 的 diff 借鉴于 inferno，该算法其中有两个理念。第一个是相同的前置与后置元素的预处理；第二个则是最长递增子序列，此思想与 React 的 diff 类似又不尽相同。

##