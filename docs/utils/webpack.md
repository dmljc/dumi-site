---
toc: content
order: 4
---

# Webpack

webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

![](https://t1.picb.cc/uploads/2019/09/09/gXSVgg.png)

## 为什么要进行构建和打包？

-   体积更小 (Tree-Shaking、压缩、合并)，加载更快
-   编译高级语言 (TS、Less) 或 语法 (ES6+、模块化)
-   兼容性和错误检查(Pollyfill、postcss、eslint)
-   统一、高效的开发环境
-   统一的构建流程和产出标准
-   集成公司的构建规范(提测、上线等)

## module chunk bundle

-   module 各个源代码文件，webpack 中一切皆模块
-   chunk 多模块合成的，如 entry、import()、splitChunk
-   bundle 最终的输出文件

## loader 和 plugin 的区别

`Loader` 本质就是一个`函数`，在该函数中对接收到的内容进行`转换`，返回转换后的结果。
因为 `Webpack` 只认识 `JavaScript`，所以 `Loader` 就成了翻译官，对其他类型的资源进行转译的预处理工作。

`Plugin` 就是插件，基于事件流框架 `Tapable`，插件可以扩展 Webpack 的功能，在 `Webpack` 运行的`生命周期中`会广播出许多`事件`，`Plugin` 可以监听这些事件，在合适的时机通过 Webpack 提供的 API `改变输出结果`。

## 懒加载

-   import() 函数
-   结合 Vue React 异步组件
-   结合 Vue-router React-router 异步加载路由

## 自动刷新

```js
"scripts": {
    "watch": "webpack --watch",
},
```

唯一的缺点是，为了看到修改后的实际效果，你需要`刷新浏览器`。

如果能够`自动刷新`浏览器就更好了，因此接下来我们会尝试通过 `webpack-dev-server` 实现此功能。

## 热更新

`webpack-dev-server` 为你提供了一个简单的 `web server`，并且具有 live reloading(`实时重新加载`)。

```js
devServer: {
    contentBase: './dist',
},
```

以上配置告知 `webpack-dev-server`，将 `dist` 目录下的`文件`，`serve` 到 localhost:8080 下。

如果你更改任何源文件并保存它们，web server 将在`编译代码后`自动重新加载。

`webpack-dev-server` 在`编译之后`不会写入到任何输出文件。而是将 `bundle` 文件保留在`内存`中，然后将它们 serve 到 server 中，就好像它们是挂载在 server 根路径上的真实文件一样。

如果希望在其他`不同路径`中找到 `bundle` 文件，则可以通过 `devServer` 配置中的 `publicPath` 选项进行修改。

## 处理 HTML

`HtmlWebpackPlugin`

该插件将为你生成一个 `HTML5 文件`， 在 `body` 中使用 `script` 标签引入你所有 webpack 生成的 `bundle`。

## 处理 CSS

`postcss-loader` + `autoprefixer` 为 `css3` 添加浏览器前缀。

```js
rules: [
    {
        test: /\.less$/,
        use: [
            'style-loader',
            'css-loader',
            {
                loader: 'postcss-loader',
                options: {
                    plugins: [require('autoprefixer')],
                },
            },
            'less-loader',
        ], // 从右向左解析原则
    },
];
```

这时候我们发现 CSS 通过 Style 标签的方式添加到了 Html 文件中，但是如果样式文件很多，全部添加到 Html 中，难免显得混乱。这时候我们想用把 CSS 拆分出来用外链的形式引入 CSS 文件怎么做呢？这时候我们就需要借助插件来帮助我们。

### MiniCssExtractPlugin

`将 CSS 提取到单独的文件中`，为每个包含 CSS 的 JS 文件创建一个 CSS 文件，并且支持 `CSS` 和 `SourceMaps` 的 `按需加载`。

### ExtractTextWebpackPlugin

`MiniCssExtractPlugin` 会将所有的 CSS 样式合并为 `一个 CSS 文件`。如果你想拆分为一一对应的`多个 CSS 文件`，我们需要使用到 `ExtractTextWebpackPlugin`。

### CssMinimizerWebpackPlugin

`优化` 和 `压缩 CSS`。这将`仅在生产环境开启` CSS 优化。 如果还想在开发环境下启用 CSS 优化，请将 optimization.minimize 设置为 true。

```js
module.exports = {
    optimization: {
        minimize: true,
        minimizer: [
            new CssMinimizerPlugin({
                test: /\.foo\.css$/i, // 用来匹配文件
                include: /\/includes/, // 要包含的文件
                parallel: true, // Boolean|Number  多进程并发执行
            }),
        ],
    },
};
```

## 处理 JS

### 优化 babel-loader

把 ES6+ 语法转为 ES5 语法。

```js
rules: [
    {
        test: /\.m?js$/,
        include: path.resolve(__dirname, 'src')  // 明确范围,二选一
        // exclude: /node_modules/,
        use: ['babel-loader?cacheDirectory']   // 开启缓存
    }
]
```

### babel-pollyfill

上面 `babel-loader` 只会将 ES6/7/8 语法转换为 ES5 语法，但是对新 api 并不会转换 例如(`promise`、`Generator`、`Set`、`Maps`、`Proxy`等) 此时我们需要借助 `babel-polyfill` 来帮助我们转换。

`babel-polyfill` 是 `core-js` 和 `regenerator` 两者的集合。

`Babel` 从 `7.4.0` 开始，不赞成使用此软件包，而推荐使用 `core-js` 和 `regenerator`。

### babel-runtime

`Babel` 在每个文件都插入了`辅助代码`，使代码`体积过大`, 你可以引入 `babel-runtime` 作为一个独立模块，来 `避免重复引入`。

### UglifyjsWebpackPlugin

压缩 JavaScript。但是，uglifyjs 不支持 es6 语法。所以，用 TerserWebpackPlugin 替代此插件。

### TerserWebpackPlugin

压缩 JavaScript。

如果使用 webpack v4，则必须安装 terser-webpack-plugin v4 的版本。

如果你使用的是 webpack v5 或以上版本，你不需要安装这个插件。webpack v5 自带最新的 terser-webpack-plugin。

```js
new TerserWebpackPlugin({
    test: /\.js(\?.*)?$/i   // 用来匹配文件。
    include: /\/includes/,  // 要包含的文件。
    parallel: Boolean || Number, // 多进程并发执行，提升构建速度, Number 启用多进程并发执行且设置并发数。
}),
```

### happyPack

JS 是单线程，使用 HappyPack 开启 `多进程打包`，提高构建速度。

作者已经放弃维护，推荐使用 `thread-loader`

## 处理文件、图片、字体等

`file-loader` 就是将`文件`在进行一些处理后（主要是处理文件名和路径、解析文件 url），`并将文件移动到输出的目录中`；

`url-loader` 一般与 `file-loader` 搭配使用，功能与 `file-loader` 类似，如果文件`小于限制的大小`，则会返回 `base64` 编码，否则使用 `file-loader` 将文件移动到输出的目录中；

```js
rules: {
    {
        test: /\.(jpe?g|png|gif)$/i, //图片文件
        //  test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/, //媒体文件
        //  test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/i, // 字体
        use: [
            {
                loader: 'url-loader',
                options: {
                    limit: 10240,
                    fallback: {
                        loader: 'file-loader',
                        options: {
                            name: 'img/[name].[hash:8].[ext]'
                            //  name: 'media/[name].[hash:8].[ext]'
                            //  name: 'fonts/[name].[hash:8].[ext]'
                        }
                    }
                }
            }
        ]
    }
}
```

### 图片压缩

`image-webpack-loader` PNG, JPEG, GIF, SVG and WEBP....

```js
rules: [
    {
        test: /\.(gif|png|jpe?g|svg)$/i,
        use: [
            'file-loader',
            {
                loader: 'image-webpack-loader',
                options: {
                    mozjpeg: {
                        progressive: true,
                    },
                    optipng: {
                        enabled: false,
                    },
                    pngquant: {
                        quality: [0.65, 0.9],
                        speed: 4,
                    },
                    gifsicle: {
                        interlaced: false,
                    },
                    webp: {
                        quality: 75,
                    },
                },
            },
        ],
    },
];
```

## 处理第三方包

### IgnorePlugin 避免引入无用模块

这是 webpack `内置插件`，它的作用是：`忽略第三方包指定目录，让这些指定目录不要被打包进去`。

比如我们要使用 `moment` 这个第三方依赖库，该库主要是对时间进行格式化，并且支持多个国家语言。虽然我设置了语言为中文，但是在打包的时候，是会将所有语言都打包进去的。这样就导致包很大，打包速度又慢。对此，我们可以用 `IgnorePlugin` 使得指定目录被忽略，从而使得打包变快，文件变小。

```js
const Webpack = require('webpack')
......
plugins: [
    // moment这个库中，如果引用了 ./locale/目录的内容，就忽略掉，不会打包进去
    new Webpack.IgnorePlugin(/\.\/locale/, /moment/),
]
```

我们虽然按照上面的方法忽略了包含’./locale/'该字段路径的文件目录，但是也使得我们使用的时候不能显示`中文语言`了，所以这个时候可以手动引入中文语言的目录。

```js
import moment from 'moment';

// 手动引入所需要的语言包
import 'moment/locale/zh-cn';

moment.locale('zh-cn');
```

**IgnorePlugin 直接不引入，代码中没有**

### noParse

`noParse` 对完全`不需要解析`的库进行忽略，可以`提高构建性能`。（忽略的文件中 不应该含有 import, require, define 的调用，或任何其他导入机制。）

```js
module.exports = {
    //...
    module: {
        noParse: /jquery|lodash/,
    },
};
```

### DllPlugin

对于开发项目中不经常会变更的`静态依赖文件`。类似于我们的 `elementUi`、`vue` 全家桶等等。因为很少会变更，所以我们不希望这些依赖要被集成到每一次的`构建`逻辑中去。

这样做的好处是每次更改我本地代码时，`webpack` 只需要`打包`我项目本身的文件代码，而`不会再去编译第三方库`。以后只要我们不升级第三方包，那么 webpack 就不会对这些库去打包，这样可以快速的提高`打包`的速度。

webpack.dll.config.js

```js
const path = require('path');
const webpack = require('webpack');
module.exports = {
    entry: {
        vendor: ['vue', 'element-ui'], // 你想要打包的模块的数组
    },
    output: {
        path: path.resolve(__dirname, 'static/js'), // 打包后文件输出的位置
        filename: '[name].dll.js',
        library: '[name]_library',
        // 这里需要和webpack.DllPlugin中的`name: '[name]_library',`保持一致。
    },
    plugins: [
        new webpack.DllPlugin({
            path: path.resolve(__dirname, '[name]-manifest.json'),
            name: '[name]_library',
            context: __dirname,
        }),
    ],
};
```

在 `package.json` 中配置如下命令

```js
"dll": "webpack --config build/webpack.dll.config.js"
```

接下来在我们的 `webpack.config.js` 中增加以下代码

```js
module.exports = {
    plugins: [
        new webpack.DllReferencePlugin({
            context: __dirname,
            manifest: require('./vendor-manifest.json'),
        }),
        new CopyWebpackPlugin([
            // 拷贝生成的文件到dist目录 这样每次不必手动去cv
            {
                from: 'static',
                to: 'static',
            },
        ]),
    ],
};
```

执行

```js
npm run dll
```

会发现生成了我们需要的集合第三地方 代码的 `vendor.dll.js` 我们需要在 `html`文件中`手动引入`这个 js 文件。

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title></title>
    <script src="static/js/vendor.dll.js"></script>
</head>
<body>
    <div id="app"></div>
</body>
</html>
```

如果我们没有`更新第三方`依赖包，就不必 `npm run dll`。直接执行 `npm run dev` 和 `npm run build` 的时候会发现我们的`打包速度`明显有所提升。因为我们已经通过`dllPlugin`将第三方依赖包`抽离`出来了。

## CleanWebpackPlugin

每次执行 `npm run build` 会发现 `dist` 文件夹里会残留上次打包的文件，这里我们推荐一个 plugin 来帮我们在打包输出前`清空文件夹` clean-webpack-plugin。

## Tree Shaking

这里单独提一下 `tree-shaking` 是因为这里有个坑。`tree-shaking` 的主要作用是`移除 JavaScript 上下文中的未引用代码(dead-code)`。目前在 `webpack4` 我们设置 mode 为 `production` 的时候已经`自动开启`了 tree-shaking。

`Tree-shaking` 只对 `ES Module` 起作用，对于 `Commonjs` 无效。

因为 `tree-shaking` 是针对 `静态结构` 进行分析，只有 `import` 和 `export` 是静态的导入和导出。而 `Commonjs` 有`动态`导入和导出的功能，无法进行`静态分析`。

如果使用 `Babel` 的话，这里有一个小问题，因为 `Babel的预案（preset）`默认会将任何模块类型都转译成 `CommonJS` 类型，这样会导致 `tree-shaking` 失效。修正这个问题也很简单，在 `.babelrc` 文件或在`webpack.config.js`文件中设置 `modules：false` 即可。

```js
// .babelrc
{
    "presets": [
        ["@babel/preset-env",
            {
                "modules": false
            }
        ]
    ]
}
```

或者

```js
// webpack.config.js

module: {
    rules: [
        {
            test: /\.js$/,
            use: {
                loader: 'babel-loader',
                options: {
                    presets: [
                        '@babel/preset-env',
                        {
                            modules: false
                        }
                    ]
                }
            }，
            exclude: /(node_modules)/
        }
    ]
}
```

## bundle 分析

`webpack-bundle-analyzer` 一个 plugin 和 CLI 工具，它将 `bundle` 内容展示为一个便捷的、交互式、可缩放的树状图形式。

![analyzer](/images/webpack/analyzer.png)

## 核心工具

`Tapable` 是 webpack 的一个`核心工具`。在 webpack 中的许多对象都扩展自 Tapable 类。 它对外暴露了 `tap`，`tapAsync` 和 `tapPromise` 等方法，插件可以使用这些方法向 webpack 中注入`自定义构建的步骤`，这些步骤将在`构建过程中触发`。

根据使用不同的`钩子`(hooks)和 `tap` 方法， 插件可以以多种不同的方式`运行`。

### 编译阶段

`compiler hooks` 钩子 在 `编译阶段` 时(compile)，只有 `同步的 tap 方法`可以使用。

```js
compiler.hooks.compile.tap('MyPlugin', (params) => {
    console.log('以同步方式触及 compile 钩子。');
});
```

### 运行阶段

在 `run 阶段` 则需使用 `tapAsync` 或 `tapPromise`（以及 `tap`）方法。

```js
compiler.hooks.run.tapAsync(
    'MyPlugin',
    (source, target, routesList, callback) => {
        console.log('以异步方式触及运行钩子。');
        callback();
    },
);

compiler.hooks.run.tapPromise('MyPlugin', (source, target, routesList) => {
    return new Promise((resolve) => setTimeout(resolve, 1000)).then(() => {
        console.log('以异步的方式触发具有延迟操作的钩子。');
    });
});

compiler.hooks.run.tapPromise(
    'MyPlugin',
    async (source, target, routesList) => {
        await new Promise((resolve) => setTimeout(resolve, 1000));
        console.log('以异步的方式触发具有延迟操作的钩子。');
    },
);
```

各种插件都能以合适的方式去运行。

## webpack 5 的改动

-   主要是内部效率的优化
-   对比 webpack 4，没有太多使用上的改动
-   webpack 5.0 旨在减少配置的复杂度，使其更容易上手
