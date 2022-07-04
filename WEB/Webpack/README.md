# Webpack

`webpack` 是一个现代 `JavaScript` 应用程序的静态模块打包器，当 `webpack` 处理应用程序时，会递归构建一个依赖关系图，其中包含应用程序需要的每个模块，然后将这些模块打包成一个或多个 `bundle`。

## 1. webpack 的核心概念

- entry: 入口
- output: 输出
- loader: 模块转换器，用于把模块原内容按照需求转换成新内容
- 插件(plugins): 扩展插件，在webpack构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要做的事情

## 2. 初始化项目

新建一个文件夹，如: `webpack-first` 

使用 `npm init -y` 进行初始化(也可以使用 `yarn`)

要使用 `webpack`，那么必然需要安装 `webpack`、`webpack-cli`：

```
npm install webpack webpack-cli -D
```

鉴于前端技术变更迅速，祭出本篇文章基于 `webpack` 的版本号:

```
├── webpack@4.41.5 
└── webpack-cli@3.3.10 
```

从 `wepack V4.0.0` 开始， `webpack` 是开箱即用的，在不引入任何配置文件的情况下就可以使用

新建 `src/index.js` 文件，我们在文件中随便写点什么:

```
//index.js
class Animal {
    constructor(name) {
        this.name = name;
    }
    getName() {
        return this.name;
    }
}

const dog = new Animal('dog');
```

使用 `npx webpack --mode=development` 进行构建，默认是 `production` 模式，我们为了更清楚得查看打包后的代码，使用 `development` 模式。

可以看到项目下多了个 `dist` 目录，里面有一个打包出来的文件 `main.js`。

`webpack` 有默认的配置，如默认的入口文件是 `./src`，默认打包到`dist/main.js`。更多的默认配置可以查看: `node_modules/webpack/lib/WebpackOptionsDefaulter.js`。

查看 `dist/main.js` 文件，可以看到，`src/index.js` 并没有被转义为低版本的代码，这显然不是我们想要的。

```js
{
    "./src/index.js":
        (function (module, exports) {

            eval("class Animal {\n    constructor(name) {\n        this.name = name;\n    }\n    getName() {\n        return this.name;\n    }\n}\n\nconst dog = new Animal('dog');\n\n//# sourceURL=webpack:///./src/index.js?");

        })
}
```

## 3. 将JS转义为低版本

前面我们说了 `webpack` 的四个核心概念，其中之一就是 `loader`，`loader` 用于对源代码进行转换，这正是我们现在所需要的。

将JS代码向低版本转换，我们需要使用 `babel-loader`

#### babel-loader

首先安装一下 `babel-loader`

```
npm install babel-loader -D
```

此外，我们还需要配置 `babel`，为此我们安装一下以下依赖:

```js
npm install @babel/core @babel/preset-env @babel/plugin-transform-runtime -D

npm install @babel/runtime @babel/runtime-corejs3
```

对babel7配置不熟悉的小伙伴，可以阅读一下这篇文章: 不可错过的 Babel7 知识

新建 `webpack.config.js`，如下:

```
// webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.jsx?$/,
                use: ['babel-loader'],
                exclude: /node_modules/          //排除 node_modules 目录
            }
        ]
    }
}
```

建议给 `loader` 指定 `include` 或是 `exclude`，指定其中一个即可，因为 `node_modules` 目录通常不需要我们去编译，排除后，有效提升编译效率。

这里，我们可以在 `.babelrc` 中编写 `babel` 的配置，也可以在 `webpack.config.js` 中进行配置。

#### 创建一个 .babelrc

配置如下：

```
{
    "presets": ["@babel/preset-env"],
    "plugins": [
        [
            "@babel/plugin-transform-runtime",
            {
                "corejs": 3
            }
        ]
    ]
}
```

现在，我们重新执行  `npx webpack --mode=development`，查看 `dist/main.js`，会发现已经被编译成了低版本的JS代码。

#### 在webpack中配置 babel

```
//webpack.config.js
module.exports = {
    // mode: 'development',
    module: {
        rules: [
            {
                test: /\.jsx?$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ["@babel/preset-env"],
                        plugins: [
                            [
                                "@babel/plugin-transform-runtime",
                                {
                                    "corejs": 3
                                }
                            ]
                        ]
                    }
                },
                exclude: /node_modules/
            }
        ]
    }
}
```

这里有几点需要说明：

- `loader` 需要配置在 `module.rules` 中，`rules` 是一个数组。
- `loader` 的格式为:

```
{
    test: /\.jsx?$/,//匹配规则
    use: 'babel-loader'
}
```

或者也可以像下面这样:

```
//适用于只有一个 loader 的情况
{
    test: /\.jsx?$/,
    loader: 'babel-loader',
    options: {
        //...
    }
}
```

`test` 字段是匹配规则，针对符合规则的文件进行处理。

`use` 字段有几种写法

- 可以是一个字符串，例如上面的 `use: 'babel-loader'`
- `use` 字段可以是一个数组，例如处理CSS文件时，`use: ['style-loader', 'css-loader']`
- `use` 数组的每一项既可以是字符串也可以是一个对象，当我们需要在`webpack` 的配置文件中对 `loader` 进行配置，就需要将其编写为一个对象，并且在此对象的 `options` 字段中进行配置，如：

```
rules: [
    {
        test: /\.jsx?$/,
        use: {
            loader: 'babel-loader',
            options: {
                presets: ["@babel/preset-env"]
            }
        },
        exclude: /node_modules/
    }
]
```

上面我们说了如何将`JS`的代码编译成向下兼容的代码，当然你可以还需要一些其它的 `babel` 的插件和预设，例如 `@babel/preset-react`，`@babel/plugin-proposal-optional-chaining` 等，不过，`babel` 的配置并非本文的重点，我们继续往下。

不要说细心的小伙伴了，即使是粗心的小伙伴肯定也发现了，我们在使用 `webpack` 进行打包的时候，一直运行的都是 `npx webpack --mode=development` 是否可以将 `mode` 配置在 `webpack.config.js` 中呢？显然是可以的。

## 4. mode

将 `mode` 增加到 `webpack.config.js` 中

```
module.exports = {
    //....
    mode: "development",
    module: {
        //...
    }
}
```

`mode` 配置项，告知 `webpack` 使用相应模式的内置优化。

`mode` 配置项，支持以下两个配置:

- `development`：将 `process.env.NODE_ENV` 的值设置为 `development`，启用 `NamedChunksPlugin` 和 `NamedModulesPlugin`
- `production`：将 `process.env.NODE_ENV` 的值设置为 `production`，启用 `FlagDependencyUsagePlugin`, `FlagIncludedChunksPlugin`, `ModuleConcatenationPlugin`, `NoEmitOnErrorsPlugin`, `OccurrenceOrderPlugin`, `SideEffectsFlagPlugin` 和 `UglifyJsPlugin`

现在，我们之间使用 `npx webpack` 进行编译即可！

## 5. 在浏览器中查看页面

搞了这么久，还不能在浏览器中查看页面，这显然不能忍！

查看页面，难免就需要 `html` 文件，有小伙伴可能知道，有时我们会指定打包文件中带有 `hash`，那么每次生成的 `js` 文件名会有所不同，总不能让我们每次都人工去修改 `html`，这样不是显得我们很蠢嘛 ~

我们可以使用 `html-webpack-plugin` 插件来帮助我们完成这些事情。

首先，安装一下插件:

```
npm install html-webpack-plugin -D 
```

新建 `public` 目录，并在其中新建一个 `index.html` 文件( 文件内容使用 `html:5` 快捷生成即可)

修改 `webpack.config.js` 文件。

```
//首先引入插件
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
    //...
    plugins: [
        new HtmlWebpackPlugin({
            template: './public/index.html',
            filename: 'index.html', //打包后的文件名
            minify: {
                removeAttributeQuotes: false, //是否删除属性的双引号
                collapseWhitespace: false, //是否折叠空白
            },
            // hash: true //是否加上hash，默认是 false
        })
    ]
}
```

此时执行 `npx webpack`，可以看到 `dist` 目录下新增了 `index.html` 文件，并且其中自动插入了 `<script>` 脚本，引入的是我们打包之后的 js 文件。

这里要多说一点点东西，`HtmlWebpackPlugin` 还为我们提供了一个 `config` 的配置，这个配置可以说是非常有用了。

#### html-webpack-plugin 的 config 的妙用

有时候，我们的脚手架不仅仅给自己使用，也许还提供给其它业务使用，`html` 文件的可配置性可能很重要，比如：你公司有专门的部门提供M页的公共头部/公共尾部，埋点`jssdk`以及分享的`jssdk`等等，但不是每个业务都需要这些内容。

一个功能可能对应多个 `js` 或者是 `css` 文件，如果每次都是业务自行修改 `public/index.html` 文件，也挺麻烦的。首先他们得搞清楚每个功能需要引入的文件，然后才能对 `index.html` 进行修改。

此时我们可以增加一个配置文件，业务通过设置 `true` 或 `false` 来选出自己需要的功能，我们再根据配置文件的内容，为每个业务生成相应的 `html` 文件。

首先，我们在 `public` 目录下新增一个 `config.js` ，将其内容设置为:

```js
// public/config.js 除了以下的配置之外，这里面还可以有许多其他配置，例如,pulicPath 的路径等等
module.exports = {
    dev: {
        template: {
            title: '你好',
            header: false,
            footer: false
        }
    },
    build: {
        template: {
            title: '你好才怪',
            header: true,
            footer: false
        }
    }
}
```

现在，我们修改下我们的 `webpack.config.js`:

```
//webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const isDev = process.env.NODE_ENV === 'development';
const config = require('./public/config')[isDev ? 'dev' : 'build'];

modue.exports = {
    //...
    mode: isDev ? 'development' : 'production'
    plugins: [
        new HtmlWebpackPlugin({
            template: './public/index.html',
            filename: 'index.html', //打包后的文件名
            config: config.template
        })
    ]
}
```

相应的，我们需要修改下我们的 `public/index.html` 文件(嵌入的`js`和`css`并不存在，仅作为示意)：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <% if(htmlWebpackPlugin.options.config.header) { %>
    <link rel="stylesheet" type="text/css" href="//common/css/header.css">
    <% } %>
    <title><%= (htmlWebpackPlugin.options.config.title) %></title>
</head>

<body>
</body> 
<% if(htmlWebpackPlugin.options.config.header) { %>
<script src="//common/header.min.js" type="text/javascript"></script> 
<% } %>
</html>
```

`process.env` 中默认并没有 `NODE_ENV`，这里配置下我们的 `package.json` 的 `scripts`.

```js
{
    "scripts": {
        "dev": "NODE_ENV=development webpack",
        "build": "NODE_ENV=production webpack"
    }
}
```

然后我们运行 `npm run dev` 和 运行 `npm run build` ，对比下 `dist/index.html` ，可以看到 `npm run build`，生成的 `index.html` 文件中引入了对应的 `css` 和 `js`。并且对应的 `title` 内容也不一样。

你说这里是不是非得是用 `NODE_ENV` 去判断？当然不是咯，你写 `aaa=1` ，`aaa=2` 都行（当然啦，`webpack.config.js` 和 `scripts` 都需要进行相应修改），但是可能会被后面接手的人打死。

#### 如何在浏览器中实时展示效果

装依赖:

```
npm install webpack-dev-server -D
```

修改下咱们的 `package.json` 文件的 `scripts`：

```
"scripts": {
    "dev": "NODE_ENV=development webpack-dev-server",
    "build": "NODE_ENV=production webpack"
},
```

在控制台执行 `npm run dev`，启动正常，页面上啥也没有，修改下我们的`JS`代码，往页面中增加点内容，正常刷新(也就是说不需要进行任何配置就可以使用了)。

在配置了 `html-webpack-plugin` 的情况下， `contentBase` 不会起任何作用

可以在 `webpack.config.js` 中进行 `webpack-dev-server` 的其它配置，例如指定端口号，设置浏览器控制台消息，是否压缩等等:

```
//webpack.config.js
module.exports = {
    //...
    devServer: {
        port: '3000', //默认是8080
        quiet: false, //默认不启用
        inline: true, //默认开启 inline 模式，如果设置为false,开启 iframe 模式
        stats: "errors-only", //终端仅打印 error
        overlay: false, //默认不启用
        clientLogLevel: "silent", //日志等级
        compress: true //是否启用 gzip 压缩
    }
}
```

- 启用 `quiet` 后，除了初始启动信息之外的任何内容都不会被打印到控制台。这也意味着来自 `webpack` 的错误或警告在控制台不可见 ———— 我是不会开启这个的，看不到错误日志，还敲个锤子！
- `stats`: "errors-only" ， 终端中仅打印出 `error`，注意当启用了 `quiet` 或者是 `noInfo` 时，此属性不起作用。 ————— 这个属性个人觉得很有用，尤其是我们启用了 `eslint` 或者使用 `TS`进行开发的时候，太多的编译信息在终端中，会干扰到我们。
- 启用 `overlay` 后，当编译出错时，会在浏览器窗口全屏输出错误，默认是关闭的。

![image-20220704151801010](images/image-20220704151801010.png)

- `clientLogLevel`: 当使用内联模式时，在浏览器的控制台将显示消息，如：在重新加载之前，在一个错误之前，或者模块热替换启用时。如果你不喜欢看这些信息，可以将其设置为 `silent` (`none` 即将被移除)。

![image-20220704151855036](images/image-20220704151855036.png)

本篇文章不是为了细说 `webpack-dev-server` 的配置，所以这里就不多说了。关于 `webpack-dev-server` 更多的配置可以查看官方文档。

细心的小伙伴可能发现了一个小问题，我们在`src/index.js`中增加一句 `console.log('aaa')`：

```
class Animal {
    constructor(name) {
        this.name = name;
    }
    getName() {
        return this.name;
    }
}

const dog = new Animal('dog');
console.log('aaa');
```

然后通过 `npm run dev` 查看效果，会发现：

![image-20220704152049080](images/image-20220704152042157.png)

这显然不是我们源码中对应的行号，点进去的话，会发现代码是被编译后的，我当前的代码非常简单，还能看出来，项目代码复杂后，“亲妈”看编译后都费劲，这不利于我们开发调试，不是我们想要的，我们肯定还是希望能够直接对应到源码的。

## 6. devtool

`devtool` 中的一些设置，可以帮助我们将编译后的代码映射回原始源代码。不同的值会明显影响到构建和重新构建的速度。

对我而言，能够定位到源码的行即可，因此，综合构建速度，在开发模式下，我设置的 `devtool` 的值是 `cheap-module-eval-source-map`。

```
//webpack.config.js
module.exports = {
    devtool: 'cheap-module-eval-source-map' //开发环境下使用
}
```

生产环境可以使用 `none` 或者是 `source-map`，使用 `source-map` 最终会单独打包出一个 `.map` 文件，我们可以根据报错信息和此 `map` 文件，进行错误解析，定位到源代码。

`source-map` 和 `hidden-source-map` 都会打包生成单独的 `.map` 文件，区别在于，`source-map` 会在打包出的js文件中增加一个引用注释，以便开发工具知道在哪里可以找到它。`hidden-source-map` 则不会在打包的js中增加引用注释。

但是我们一般不会直接将 `.map` 文件部署到CDN，因为会直接映射到源码，更希望将`.map` 文件传到错误解析系统，然后根据上报的错误信息，直接解析到出错的源码位置。

不过报错信息中只有行号，而没有列号。如果有行列号，那么可以通过`sourcemap` 来解析出错位置。只有行号，根本无法解析，不知道大家的生产环境是如何做的？怎么上报错误信息至错误解析系统进行解析。如有好的方案，请赐教。

还可以设置其他的devtool值，你可以使用不同的值，构建对比差异。

现在我们已经说了 `html`、`js` 了，并且也可以在浏览器中实时看到效果了，现在就不得不说页面开发三巨头之一的 `css` 

## 7. 如何处理样式文件呢

`webpack` 不能直接处理 `css`，需要借助 `loader`。如果是 `.css`，我们需要的 `loader` 通常有： `style-loader`、`css-loader`，考虑到兼容性问题，还需要 `postcss-loader`，而如果是 `less` 或者是 `sass` 的话，还需要 `less-loader` 和 `sass-loader`，这里配置一下 `less` 和 `css` 文件(`sass` 的话，使用 `sass-loader`即可):

先安装一下需要使用的依赖:

```js
npm install style-loader less-loader css-loader postcss-loader autoprefixer less -D

//webpack.config.js
module.exports = {
    //...
    module: {
        rules: [
            {
                test: /\.(le|c)ss$/,
                use: ['style-loader', 'css-loader', {
                    loader: 'postcss-loader',
                    options: {
                        plugins: function () {
                            return [
                                require('autoprefixer')({
                                    "overrideBrowserslist": [
                                        ">0.25%",
                                        "not dead"
                                    ]
                                })
                            ]
                        }
                    }
                }, 'less-loader'],
                exclude: /node_modules/
            }
        ]
    }
}
```

测试一下，新建一个 `less` 文件，`src/index.less`:

```
//src/index.less
@color: red;
body{
    background: @color;
    transition: all 2s;
}
```

再在入口文件中引入此 `less`:

```
//src/index.js
import './index.less';
```

我们修改了配置文件，重新启动一下服务: `npm run dev`。可以看到页面的背景色变成了红色。

简单说一下上面的配置：

- `style-loader` 动态创建 `style` 标签，将 `css` 插入到 `head` 中.
- `css-loader` 负责处理 `@import` 等语句。
- `postcss-loader` 和 `autoprefixer`，自动生成浏览器兼容性前缀 —— 2020了，应该没人去自己徒手去写浏览器前缀了吧
- `less-loader` 负责处理编译 `.less` 文件,将其转为 `css`

这里，我们之间在 `webpack.config.js` 写了 `autoprefixer` 需要兼容的浏览器，仅是为了方便展示。推荐大家在根目录下创建 `.browserslistrc`，将对应的规则写在此文件中，除了 `autoprefixer` 使用外，`@babel/preset-env`、`stylelint`、`eslint-plugin-conmpat` 等都可以共用。

**注意：** 

```
loader` 的执行顺序是从右向左执行的，也就是后面的 `loader` 先执行，上面 `loader` 的执行顺序为: `less-loader` ---> `postcss-loader` ---> `css-loader` ---> `style-loader
```

当然，`loader` 其实还有一个参数，可以修改优先级，`enforce` 参数，其值可以为: `pre`(优先执行) 或 `post` (滞后执行)。

现在，我们已经可以处理 `.less` 文件啦，`.css` 文件只需要修改匹配规则，删除 `less-loader` 即可。

现在的一切看起来都很完美，但是假设我们的文件中使用了本地的图片，例如:

```
body{
    backgroud: url('../images/thor.png');
}
```

你就会发现，报错啦，那么我们要怎么处理图片或是本地的一些其它资源文件呢。不用想，肯定又需要 `loader` 出马了。

## 8. 图片/字体文件处理

我们可以使用 `url-loader` 或者 `file-loader` 来处理本地的资源文件。`url-loader` 和 `file-loader` 的功能类似，但是 `url-loader` 可以指定在文件大小小于指定的限制时，返回 `DataURL`，因此，个人会优先选择使用 `url-loader`。

首先安装依赖:

```
npm install url-loader -D
```

![image-20220704152425619](images/image-20220704152425619.png)

安装 `url-loader` 的时候，控制台会提示你，还需要安装下 `file-loader`，听人家的话安装下就行(新版 `npm` 不会自动安装 `peerDependencies`)：

```
npm install file-loader -D
```

在 `webpack.config.js` 中进行配置：

```js
//webpack.config.js
module.exports = {
    //...
    modules: {
        rules: [
            {
                test: /\.(png|jpg|gif|jpeg|webp|svg|eot|ttf|woff|woff2)$/,
                use: [
                    {
                        loader: 'url-loader',
                        options: {
                            limit: 10240, //10K
                            esModule: false 
                        }
                    }
                ],
                exclude: /node_modules/
            }
        ]
    }
}
```

此处设置 `limit` 的值大小为 10240，即资源大小小于 `10K` 时，将资源转换为 `base64`，超过 10K，将图片拷贝到 `dist` 目录。`esModule` 设置为 `false`，否则，`<img src={require('XXX.jpg')} />` 会出现 `<img src=[Module Object] />`

将资源转换为 `base64` 可以减少网络请求次数，但是 `base64` 数据较大，如果太多的资源是 `base64`，会导致加载变慢，因此设置 `limit` 值时，需要二者兼顾。

默认情况下，生成的文件的文件名就是文件内容的 `MD5` 哈希值并会保留所引用资源的原始扩展名，例如我上面的图片(thor.jpeg)对应的文件名如下：

![image-20220704152602443](images/image-20220704152602443.png)

当然，你也可以通过 `options` 参数进行修改。

```
//....
use: [
    {
        loader: 'url-loader',
        options: {
            limit: 10240, //10K
            esModule: false,
            name: '[name]_[hash:6].[ext]'
        }
    }
]
```

重新编译，在浏览器中审查元素，可以看到图片名变成了: `thor_a5f7c0.jpeg`。

当本地资源较多时，我们有时会希望它们能打包在一个文件夹下，这也很简单，我们只需要在 `url-loader` 的 `options` 中指定 `outpath`，如: `outputPath: 'assets'`，构建出的目录如下:

![image-20220704152626399](images/image-20220704152626399.png)

更多的 `url-loader` 配置可以查看官方文档

到了这里，有点**岁月静好**的感觉了。

![图片](images/640.jpeg)

不过还没完，如果你在 `public/index.html` 文件中，使用本地的图片，例如，我们修改一下 `public/index.html`：

```
<img src="./a.jpg" />
```

重启本地服务，虽然，控制台不会报错，但是你会发现，浏览器中根本加载不出这张图片，Why？

因为构建之后，通过相对路径压根找不着这张图片呀。

## 9. 处理 html 中的本地图片

安装 `html-withimg-loader` 来解决！

```
npm install html-withimg-loader -D
```

修改 `webpack.config.js`：

```js
module.exports = {
    //...
    module: {
        rules: [
            {
                test: /.html$/,
                use: 'html-withimg-loader'
            }
        ]
    }
}
```

然后在我们的 `html` 中引入一张文件测试一下（图片地址自己写咯，这里只是示意）:

```
<!-- index.html -->
<img src="./thor.jpeg" />
```

重启本地服务，图片并没能加载，审查元素的话，会发现图片的地址显示的是 `{"default":"assets/thor_a5f7c0.jpeg"}`。

![image-20220704152724526](images/image-20220704152724526.png)

我当前 `file-loader` 的版本是 5.0.2，5版本之后，需要增加 `esModule` 属性：

```js
//webpack.config.js
module.exports = {
    //...
    modules: {
        rules: [
            {
                test: /\.(png|jpg|gif|jpeg|webp|svg|eot|ttf|woff|woff2)$/,
                use: [
                    {
                        loader: 'url-loader',
                        options: {
                            limit: 10240, //10K
                            esModule: false
                        }
                    }
                ]
            }
        ]
    }
}
```

再重启本地服务，就搞定啦。

话说使用 `html-withimg-loader` 处理图片之后，`html` 中就不能使用 `vm`, `ejs` 的模板了，如果想继续在 `html` 中使用 `<% if(htmlWebpackPlugin.options.config.header) { %>` 这样的语法，但是呢，又希望能使用本地图片，可不可以？

删除`html`的`loader`，像下面这样编写图片的地址就可以啦。

```
<!-- index.html -->
<img src="<%= require('./thor.jpeg') %>" />
```

图片加载OK啦，并且 `<% %>` 语法也可以正常使用，吼吼吼~~~

虽然，`webpack` 的默认配置很好用，但是有的时候，我们会有一些其它需要啦，例如，我们不止一个入口文件，这时候，该怎么办呢？

## 10. 入口配置

入口的字段为: `entry`

```
//webpack.config.js
module.exports = {
    entry: './src/index.js' //webpack的默认配置
}
```

`entry` 的值可以是一个字符串，一个数组或是一个对象。

字符串的情况无需多说，就是以对应的文件为入口。

为数组时，表示有“多个主入口”，想要多个依赖文件一起注入时，会这样配置。例如:

```
entry: [
    './src/polyfills.js',
    './src/index.js'
]
```

`polyfills.js` 文件中可能只是简单的引入了一些 `polyfill`，例如 `babel-polyfill`，`whatwg-fetch` 等，需要在最前面被引入（我在 webpack2 时这样配置过）。

## 11. 出口配置

配置 `output` 选项可以控制 `webpack` 如何输出编译文件

```js
const path = require('path');
module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'), //必须是绝对路径
        filename: 'bundle.js',
        publicPath: '/' //通常是CDN地址
    }
}
```

例如，你最终编译出来的代码部署在 CDN 上，资源的地址为: 'https://AAA/BBB/YourProject/XXX'，那么可以将生产的 `publicPath` 配置为: `//AAA/BBB/`。

编译时，可以不配置，或者配置为 `/`。可以在我们之前提及的 `config.js` 中指定 `publicPath`（`config.js` 中区分了 `dev` 和 `public`）， 当然还可以区分不同的环境指定配置文件来设置，或者是根据 `isDev` 字段来设置。

除此之外呢，考虑到CDN缓存的问题，我们一般会给文件名加上 `hash`.

```
//webpack.config.js
module.exports = {
    output: {
        path: path.resolve(__dirname, 'dist'), //必须是绝对路径
        filename: 'bundle.[hash].js',
        publicPath: '/' //通常是CDN地址
    }
}
```

如果你觉得 `hash` 串太长的话，还可以指定长度，例如 `bundle.[hash:6].js`。使用 `npm run build` 打包看看吧。

> 注意：
>
> 每次文件修改后，重新打包，导致 `dist` 目录下的文件越来越多。每次打包前，都先清空一下目录就好啦。

## 12.每次打包前清空dist目录

反正我是懒得手动去清理的，只要你足够懒，你总是会找到好办法的，懒人推动科技进步。

这里，我们需要插件: `clean-webpack-plugin`

安装依赖:

```
npm install clean-webpack-plugin -D
```

以前，`clean-webpack-plugin` 是默认导出的，现在不是，所以引用的时候，需要注意一下。另外，现在构造函数接受的参数是一个对象，可缺省。

```js
//webpack.config.js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    //...
    plugins: [
        //不需要传参数喔，它可以找到 outputPath
        new CleanWebpackPlugin() 
    ]
}
```

现在你再修改文件，重现构建，生成的`hash`值和之前`dist`中的不一样，但是因为每次 `clean-webpack-plugin` 都会帮我们先清空一波 `dist` 目录，所以不会出现太多文件。

#### 希望dist目录下某个文件夹不被清空

有些时候，我们并不希望整个 `dist` 目录都被清空，比如，我们不希望，每次打包的时候，都删除 `dll` 目录，以及 `dll` 目录下的文件或子目录，该怎么办呢？

`clean-webpack-plugin` 为我们提供了参数 `cleanOnceBeforeBuildPatterns`。

```js
//webpack.config.js
module.exports = {
    //...
    plugins: [
        new CleanWebpackPlugin({
            cleanOnceBeforeBuildPatterns:['**/*', '!dll', '!dll/**'] //不删除dll目录下的文件
        })
    ]
}
```

此外，`clean-webpack-plugin` 还有一些其它的配置，不过我使用的不多，大家可以查看`clean-webpack-plugin`

至此，我们算是完成了一个基础配置。但是这不够完美，或者说有些时候，我们还会有一些其它的需求。

## 13. 静态资源拷贝

有些时候，我们需要使用已有的JS文件、CSS文件（本地文件），但是不需要 webpack 编译。例如，我们在 public/index.html 中引入了 public 目录下的 js 或 css 文件。这个时候，如果直接打包，那么在构建出来之后，肯定是找不到对应的 js / css 了。

