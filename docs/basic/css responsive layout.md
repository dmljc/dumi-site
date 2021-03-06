---
toc: content
order: 13
---

# CSS 响应式布局方案

## 媒体查询

CSS3 `媒体查询`可以让我们针对不同的`媒体类型`定义不同的`样式`，当重置浏览器窗口大小的过程中，页面也会根据浏览器的宽度和高度重新渲染页面。

```css
@media screen and (min-width: 1100px) {
    body {
        background-color: black;
    }
}
```

缺点:

-   需要为不同分辨率的机型都维护一套代码 成本较大

## 百分比布局

通过`百分比单位`，可以使 `组件的宽和高` 随着 `浏览器的宽高` 的变化而变化，从而实现响应式的效果。

```css
@media screen and (max-width: 1100px) {
    aside {
        width: 8%;
        background-color: yellow;
    }
}
```

缺点

-   我们必须要弄清楚 css 中子元素的百分比到底是 `相对谁的百分比`。子元素的 `height` 或 `width` 中使用百分比，是相对于 `子元素` 的 `直接父元素`。

两个缺点：

-   `计算困难`，如果我们要定义一个元素的宽度和高度，按照设计稿，必须换算成百分比单位。
-   可以看出，各个属性中如果使用百分比，`相对父元素的属性并不是唯一的`。

## rem 布局

`rem` 是 `CSS3` 新增的单位。rem 单位都是相对于 `根元素 html` 的 `font-size` 来决定大小的。

`rem` 布局也是目前多屏幕适配的 `使用率相对高点的方式`。默认情况下我们 `html` 标签的 `font-size` 为`16px`，我们利用媒体查询，设置在不同设备下的字体大小。

rem 布局的缺点：

-   必须通过 `js` 来 `动态控制 根元素 font-size` 的大小，也就是说 `css` 样式和 `js` 代码有一定的`耦合性`，且必须将改变 `font-size` 的代码放在 `css` 样式`之前`。

## 视口单位

`css3` 中引入了一个新的单位 `vw/vh`，与`视图窗口`有关，`vw` 表示相对于视图窗口的`宽度`，`vh` 表示相对于视图窗口 `高度`，除了 vw 和 vh 外，还有 `vmin` 和 `vmax` 两个相关的单位。

缺点

-   ie 不支持 `vmin` 和 `vmax` 两个属性

## tailwind css

`Tailwind CSS` 是一个`功能类优先的 CSS 框架`，它集成了诸如 flex, pt-4, text-center 和 rotate-90 这样的类，它们能直接在脚本标记语言中组合起来，构建出任何设计。

优势如下

-   `构建您想要的任何东西`

-   生产环境的`体积非常小` (在 构建生产文件时 会 自动删除所有未使用的 CSS)

-   `一切皆是响应式的`

-   `支持 hover 和 focus 状态`

-   担心复用性问题 ？( `@apply 指令` 可以把重复的功能类抽取到一个自定义的 CSS 类中)

-   `支持深色模式` (开箱即用，无需配置)

-   `易于扩展`、调整和改变 (使用 tailwind.config.js 配置自定义 css)

-   世界一流的 `IDE 集成` (VS Code 在编辑器内既可得到智能的自动完成建议、提示及类定义等功能，而且无需配置)

-   使用 `Tailwind UI` 加快开发速度 (支持 任何模板语言 或 JavaScript 框架 如 React、Vue))

缺点:

-   从 `v2.0` 开始，`Tailwind CSS` 专为 `Chrome`，`Firefox`，`Edge` 和 `Safari` 的`最新稳定版本`设计并在其上经过了测试。 `Tailwind CSS v2.0 不支持任何版本的 IE，包括 IE 11。`

-   如果您需要`支持 IE11`，我们建议您使用 `Tailwind CSS v1.9`，它仍然是一个优秀的、非常有生产力的框架。

浏览器引擎前缀

-   如果您正在使用 `tailwindcss CLI` 工具，`浏览器引擎前缀将自动添加`。
-   如果没有，我们建议您使用 `Autoprefixer`。
