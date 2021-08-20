## 一.概念

webpack 是一个模块打包器(module bundler)。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。

### 1.1.入口(entry)

指示webpack应该使用哪个模块来作为构建其内部依赖图的入口起点。(默认值为**./src**)

### 1.2.出口(output)

这个属性告诉webpack在哪里输出它所创建的 bundles，以及如何命名这些文件。(默认值为**./dist**) 

1. 通过output.filename: 告诉webpack bundle的名称；
2. 通过output.path:告诉webpack bundle生成(emit)到哪里。

### 1.3.加载器(loader)

让webpack能够去处理那些非js文件(webpack自身只能处理js)，loader可以将所有类型的文件转换为webpack能够处理的模块(modules)。

1. test属性：用于标识出应该被对应的loader进行转换的某个或者某些文件；
2. use属性：用于表示进行转换时，应该使用哪个loader。

### 1.4.插件(plugins)

loader被用于转换某些类型的模块，而插件可以用于执行范围更广的任务，从打包优化到压缩，插件目的在于解决loader无法实现的其他事情。使用步骤为：

1. 先require它，然后把它添加到plugins数组中；
2. 可以在配置文件中因为不同目的而多次使用同一个插件，通过 new 来创建一个它的实例。

### 1.5.模式(mode)

通过选择development和production之中的一个，来设置mode参数，你可以启用相应模式下的webpack内置的优化。

## 二.项目实战

### 2.1.初始化项目

```js
npm init -y
```

执行上述命令后会生成一个package.json文件，接着安装webpack4

### 2.2.安装webpack4

npm i -D webpack webpack-cli

如果你使用的是webpack4+版本，你还需要安装CLI。-D是指开发环境需要，上线不需要。

### 2.3.创建目录

1. 创建src 

   这个里面放的是源代码，“源”代码是用于书写和编辑的代码。

2. 创建dist，并且在该文件夹下创建index.html

   这个里面放的是分发代码，“分发”代码是构建过程产生的压缩后的“输出”目录，最终将在浏览器中加载；

   index.html页面script引入的标签由原始的./src文件改为加载最终打包后的bundle，main.js；

   执行 webpack 命令，会将我们的脚步作为入口起点，然后输出 main.js。

3. 配置webpack.config.js

   添加配置文件比在终端中手动输入大量命令要高效的多，所以让我们创建一个取代之前使用CLI选项方式的配置文件。

   ```diff
   webpack-demo
     |- package.json
   + |- webpack.config.js
     |- /dist
       |- index.html
     |- /src
       |- index.js
   ```

​	考虑到用CLI这种方式来运行本地的webpack很不方便，我们可以通过创建一个npm脚本的方式来设置快捷方式。

​	

```diff
"scripts": {
      "test": "echo \"Error: no test specified\" && exit 1",
+     "build": "webpack"
    },
```

​	通过运行 npm run build 命令就可以代替我们之前使用的webpack命令。并且通过向该命令和你的参数见添加两个中横线，可以将自定义参数传递给webpack，npm run build -- --colors。

### 2.4.管理资源

​	1.加载css

```js
cnpm i -D style-loader css-loader
```

安装上述两个模块，style-loader依赖于css-loader，loader的书写规则为从右到左形成一种依赖关系，最先处理的放在最右边。

​	2.加载图片

```js
cnpm i -D file-loader
```

可以看到实际的文件名已更改为诸如hash模式那样前缀的类型，这意味着webpack在src文件中找到我们的图片文件，并成功处理过它，下一步紧接着的就是压缩和优化你的图像。

```js
cnpm i -D url-loader
```

提高文件的限制大小，之前是224kib，配置如何展示性能提示。例如，如果一个资源超过250kb，webpack会对此输出一个警告来通知你，performance属性。

​	3.加载字体

file-loader和url-loader可以接收并加载任何文件，然后将其输出到构建目录，也就是说，我们可以将它们用于任何类型的文件，包括字体。

### 2.5.管理输出

到目前为止，我们所做的一切都是在index.html文件中手动引入所有资源，然而随着应用程序的增长，使用了多个bundle的时候，需要手动地对index.html文件进行管理，一切就会变得困难起来。可以通过下载一些插件，让我们不用手动更改index.html文件。

​	1.为你自动生成一个HTML文件，其中包括使用script标签的body中的所有webpack包。

```js
cnpm i -D html-webpack-plugin
```

由于过去的代码示例遗留下来，导致/dist文件夹相当杂乱，包含了之前的示例代码。webpack会生成文件，然后将这些文件放置在、dist文件夹中，但是webpack无法追踪到哪些文件是实际在项目中所需要使用到的。

​	2.通常，在每次构建前清理/dist文件夹，是比较推荐的做法，因此只会生成用到的文件。

```js
cnpm i -D clean-webpack-plugin
```

现在，你不会看到旧的文件，只有构建后生成的文件。

​	3.Manifest，webpack及其插件似乎“知道”应该哪些文件生成，答案是，通过manifest，webpack能够对【你的模块映射到输出bundle的过程】保持追踪。

### 2.6.正式开发

​	1.追踪错误和警告在源代码中的原始位置。

For development, use `cheap-module-eval-source-map`. For production, use `cheap-module-source-map`.

​	devtool: 'cheap-module-eval-source-map', // 这是 "cheap(低开销)" 的 source map。

​	2.每次要编译代码时，手动运行 npm run build 就会变得很麻烦。

```js
cnpm i -D webpack-dev-server // 提供了一个简单的web服务器，并且能够实时重新加载(live reloading) 在代码发生变化后自动编译代码
```

​	修改配置文件，告诉开发服务器(dev server)，在哪里查找文件：告知webpack-dev-server，在localhost:8080下建立服务，将dist目录下的文件，作为可访问文件。

```diff
+   devServer: {
+     contentBase: './dist'
+   },
```

​	Please update `webpack-cli` to v4 and use `webpack serve` to run webpack-dev-server

### 2.7.模块热替换

模块热替换(Hot Module Replacement 或 HMR)是 webpack 提供的最有用的功能之一。它允许在运行时更新各种模块，而无需进行完全刷新。

### 2.8.tree shaking

Tree Shaking指的就是当我引入一个模块的时候，我不引入这个模块的所有代码，我只引入我需要的代码，这就需要借助webpack里面自带的Tree Shaking这个功能，帮助我们实现。

压缩输出：从 webpack 4 开始，也可以通过 `"mode"` 配置选项轻松切换到压缩输出，只需设置为 `"production"`。

```diff
+ mode: "production"
```

## 三、生产环境

### 3.1.生产环境构建

开发环境(development)：需要强大的、具有实时重新加载(live reloading)或热模块(hot module replacement)能力的source map和localhost server。

生产环境(production)：我们的目标倾向于关注更小的bundle，更轻量的source map，以及更优化的资源，以改善加载时间。

由于这两个环境的构建目标差异很大，并且要遵循逻辑分离，我们通常建议为每个环境编写彼此独立的webpack配置。

​	1.我们还是会遵循不重复原则(Don't repeat yourself - DRY)，保留一个“通用”配置。为了将这些配置合并在一起，我们使用“通用”配置。

```js
cnpm i -D webpack-merge
```

​	2.使用tree shaking

```js
cnpm i -D uglifyjs-webpack-plugin
```

```diff
+ const UglifyJSPlugin = require('uglifyjs-webpack-plugin');
```

`UglifyJs` does not support ES6。可以使用 UglifyEs

​	3.配置scripts

- npm start 定义为开发环境脚本

- npm run build 定义为生产环境脚本

  4.指定环境

  许多库library将通过与环境变量关联，以决定库中应该应用哪些内容。例如，当处于开发环境中，某些库为了使调试变得容易，可能会添加额外的日志记录和测试。

  ```diff
  +     new webpack.DefinePlugin({
  +       'process.env.NODE_ENV': JSON.stringify('production')
  +     })
  ```

  *技术上讲，*`NODE_ENV` *是一个由 Node.js 暴露给执行脚本的系统环境变量。通常用于决定在开发环境与生产环境(dev-vs-prod)下，服务器工具、构建脚本和客户端 library 的行为。*

### 3.2.代码分离

bundle分析输出结果。一款分析 bundle 内容的插件及 CLI 工具，以便捷的、交互式、可缩放的树状图形式展现给用户：

```js
cnpm i -D webpack-bundle-analyzer
```

通过使用 `output.filename` 进行[文件名替换](https://www.webpackjs.com/configuration/output#output-filename)，可以确保浏览器获取到修改后的文件。`[hash]` 替换可以用于在文件名中包含一个构建相关(build-specific)的 hash，但是更好的方式是使用 `[chunkhash]` 替换，在文件名中包含一个 chunk 相关(chunk-specific)的哈希。输出文件的文件名：

```diff
+     filename: '[name].[chunkhash].js',
```

### 3.3.shimming

webpack编译器能够识别遵循ES2015模块的语法、CommonJS或AMD规范编写的模块，然而，一些第三方的库(library)可能会引用一些全局依赖(例如jQuery中的$)。这些库也可能创建一些需要被导出的全局变量。这些“不合符规范的模块”就是shimming发挥作用的地方。

​	1.使用 ProvidePlugin 后，能够在通过webpack编译的每个模块中，通过访问一个变量来获取package包。(如果webpack知道这个变量在某个模块中被使用了，那么webpack将在最终bundle中引入我们给定的package) 自动加载模块，而不必到处 import或require

```diff
+   plugins: [
+     new webpack.ProvidePlugin({
+       _: 'lodash'
+     })
+   ]
```

shim是一个库，它将一个新的API引入到旧的环境中，并且仅依靠旧环境中已有的手段实现。polyfil就是一个用在浏览器API上的shim。我们通常的做法是先检查当前浏览器是否支持某个API，如果不支持的话就加载对应的polyfill。然后新旧浏览器就都可以使用这个API了。

3.2.渐进式网络应用程序(PWA)

Progressive Web Application 是一种可以提供类似于原生应用程序体验的网络应用程序。PWA可以用来在离线时能够继续运行功能。

​	1.搭建一个简易服务器，真正的用户是通过网络访问网络应用程序；用户的浏览器会与一个提供所需资源(例如 html、js和css文件)的服务器通讯。

```js
cnpm i -D http-server
```

### 3.4.使用环境变量

要在开发和生产构建之间，消除webpack.config.js的差异，你可能需要环境变量。

直接在命令行环境配置中，通过设置--env并传入尽可能多的环境变量。

```bash
webpack --env.NODE_ENV=local --env.production --progress
```

### 3.5.配置process.env

通常情况下，我们需要针对不同环境(开发环境、集成环境、生产环境等)，进行相应策略的配置(比如是否替换接口地址，代码是否进行压缩等)。webpack就是通过process.env这个属性进行区别的。它是Node.js提供的一个API，它返回的是一个包含用户环境信息的独一性。如果我们给Node.js设置一个环境变量·，并把它挂载到process.env返回的对象上，便可以在代码中进行相应环境的判断。

通常的做法是，新建一个环境变量NODE_ENV，用它确定当前所处的开发阶段，生产阶段设为production，开发阶段设为development或staging，然后在脚本中读取process.env.NODE_ENV即可。要说明的是，NODE_ENV 这个名称只是开发社区的一种共识，名称内容是可以修改的。

由于Windows和Mac系统有区别，这就会到值我在Windows上开发部署的项目，到了Mac系统下就无法使用了，反之依然。为了解决这个问题，引入cross-env。它是一个跨平台设置环境变量的第三方包，它可以让你一行命令，就能轻松地在多个平台设置环境变量。

```js
cnpm i -D cross-env
```

然后package.json文件中配置就可以了。在项目中可以根据这个值来区分当前环境。

## 四、项目报错处理

1. Error: Cannot find module 'webpack-cli/bin/config-yargs'

报错前安装的版本为：

```js
"webpack": "4.4.1",
"webpack-cli": "^4.1.0",
"webpack-dev-server": "^3.11.0",
```

原因为webpack新版本并不与现有版本兼容(webpack-cli几天前发布的最新版4.0.0版移除了yargs包，而紧挨4.0.0版本的上一正式版本3.3.12还在：)去寻找适合webpack4的webpack-cli和webpack-dev-server

```js
"webpack": "4.4.1",
"webpack-cli": "^3.3.2",
"webpack-dev-server": "^3.11.0",
```

新版本尤其是大的版本更新都改变了很多，很多都从架构上进行了更改，老版本的功能包比如webpack-dev-server就不能兼容新版本的webpack了。那么我们是不是就不能使用最新版本的webpack了？其实不是，想用就要么等兼容版本，要么有精力自己搞一个兼容版本，再或者可以搜索相关功能适合最新webpack的解决方案。

## 五、插件详解

### 5.1.预处理器postcss-loader

用postcss处理css的loader，利用js代码来处理css，负责把css代码解析成抽象语法树结构，再交由插件来进行处理，插件基于css代码的AST所能进行的操作是多种多样的，比如增加浏览器相关的声明前缀。

### 5.2.后处理程序autoPrefixer

是一个流行的PostCss插件，其作用是为css中的属性添加浏览器特定的前缀，开发人员在编写css时只需要使用css规范中的标准属性名即可。

```
#content {
	display: flex;
}
```

在经过 Autoprefixer 处理之后得到的css：

```
#content {
    display: -webkit-box;
    display: -webkit-flex;
    display: -ms-flexbox;
    display: flex;
}
```

### 5.3.拆分css

webpack4.0以前，我们通过extract-text-webpack-plugin插件，把css样式从js文件中提取到单独的css文件中。webpack4.0以后，官方推荐使用mini-css-extract-plugin插件来打包css文件，会将所有的css样式合并为一个css文件。

配置文件如下：

```js
module: {
    rules: [
        // 这个插件应该只在生产环境构建中使用，并且在loader链中不应该有style-loader，特别是我们在开发模式中使用HMR时
        {
            test: /\.css$/,
            use: [ MiniCssExtractPlugin.loader,'css-loader','postcss-loader' ]
        }
    ],
},
```

### 5.4.用babel转译js文件

为了使我们的js代码能够兼容更多的环境我们需要安装依赖

1. babel-loader：加载ES2015+代码，然后使用Babel(一个js编译器，让你立即使用下一代的js)转译为ES5，是一个npm包，它使得webpack可以通过babel转译js代码。

   ```json
   注释：（在 babel 7 中 `babel-core` 和 `babel-preset` 被建议使用 `@babel` 开头声明作用域，因此应该分别下载 `@babel/core` 和`@babel/presets`。就类似于 vue-cli 升级后 使用@vue/cli一样的道理 ）Babel的功能在于【代码转译】，具体一点，就是将目标代码转译为能够符合期望语法规范的代码。在转译的过程中，babel内部经历了【解析-转换-生成】三个步骤。而`@babel/core`这个库则负责【解析】，具体的【转换】和【生成】步骤则交给各种插件(plugin)和预设(preset)来完成。
   ```

2. @babel/core：如果你需要以编程的方式来使用Babel，可以使用这个包。

   ```json
   注释：（`@babel/core`的作用是把js代码分析成ast树，方便各个插件分析语法进行相应的处理。有些新语法在低版本js中是不存在的，比如箭头函数，reset函数，函数默认值等，这种语言层面的不兼容只能通过将代码转为ast，分析其语法后再转为低版本的js）
   ```

3. @babel/preset-env：智能预设，是一系列插件的集合，可以根据配置的目标浏览器或者运行环境来按需加载插件。

   ```json
   注释：（实际上就是各种插件的打包组合，包含了我们在babel7中常用的es2015,es2016, es2017等最新的语法转化插件，允许我们使用最新的js语法，比如let、const、箭头函数等等，但不包括stage-x阶段的插件。也就是说各种转译规则的统一设定，目的是告诉loader要以什么规则来转化成对应的js版本）
   ```

   `babel-loader`与`babel-core`的版本对应关系

   1. `babel-loader` 8.x 对应`babel-core` 7.x
   2. `babel-loader` 7.x 对应`babel-core` 6.x

   配置文件如下：

   ```js
   module: {
       // 创建模块时，匹配请求的规则数组。这些规则能够修改模块的创建方式，这些规则能够对模块(module)应用loader，或者修改解析器(parser)。
       rules: [
           {
               test: /\.js$/,
               exclude: /node_modules/, // 确保转译尽可能少的文件
               use: ['babel-loader?cacheDirectory'] // 将babel-loader提速至少两倍，将转译的结果缓存到文件系统中 将使用默认的缓存目录(node_modules/.cache/babel-loader)
           },
           { test: /\.(png|svg|jpg|gif)$/, use: ['file-loader'] }, // 解析图片
           { test: /.(woff|woff2|eot|ttf|otf)$/, use: ['url-loader'] }, // 解析字体
       ]
   },
   ```

   你需要创建一个.babelrc文件，让它实际执行操作：

   ```json
   {
       "presets": ["@babel/preset-env"]
   }
   ```

   上面的babel-loader只会将ES6/7/8语法转换为ES5语法，但是对新的api并不会转换，例如(Promise、Generator、Set、Maps、Proxy等)，此时我们需要借助babel-polyfill来帮助我们转换：

   ```js
   cnpm i -S @babel/polyfill
   ```

   因为polyfill要在编译你的源代码之前执行，所以要安装为dependency而不是devDependency，解决低版本浏览器(比如IE)不兼容问题。

## 六、搭建vue环境

### 6.1.加载和转译Vue组件

1.vue-loader 是webpack的一个预处理器，它允许你以一种成为单文件组件single-file-components(SFCs)的格式来编写Vue组件。

```js
cnpm i -D vue-loader
```

前提得先安装vue这个框架：

```js
cnpm i -S vue
```

```json
-D：--save-dev 生产环境的包 
-S：--save     开发环境的包
```

安装编译.vue组件里的template部分的编译器

```js
cnpm i -D vue-template-compiler
```

2.需要注意的是vue-template-compiler编译器的版本必须和基本的 `vue` 包保持同步，这样 `vue-loader` 就会生成兼容运行时的代码。

需要注意的是，vue有不同的构建版本。基于构建工具时使用 **ES** **Module**，为打包工具提供的ESM，ESM格式被设计为可以被静态分析，所以打包工具可以利用这一点来进行“tree-shaking”并将用不到的代码排除出最终的包，为这些打包工具提供的默认文件是只有运行时的ES Module构建【vue.runtime.esm.js】

在webpack中的配置：

```json
// webpack.config.js
const VueLoaderPlugin = require('vue-loader/lib/plugin')

module.exports = {
  module: {
    rules: [
      // ... 其它规则
      {
        test: /\.vue$/,
        loader: 'vue-loader'
      }
    ]
  },
  plugins: [
    // 请确保引入这个插件！
    new VueLoaderPlugin()
  ]
}
```

**这个插件是必须的！** 它的职责是将你定义过的其它规则复制并应用到 `.vue` 文件里相应语言的块。例如，如果你有一条匹配 `/\.js$/` 的规则，那么它会应用到 `.vue` 文件里的 `<script>` 块。

3.vue-loader的作用就是：解析和转换.vue文件，提取出其中的逻辑代码 script、样式代码 style以及HTML模板template，再分别把他们交给对应的loader去处理。

css-loader的作用就是：加载由vue-loader提取出的css代码。

vue-template-compiler的作用就是：把vue-loader提取出的HTML模板编译成对应的可执行的js代码，这和react中的jsx语法被编译成js代码类似。预先编译好HTML模板相对于在浏览器中再去编译HTML模板的好处在于性能更好。

总结：vue-loader的作用就是提取。

### 6.2.path.join()和path.resolve()

1.首先先说下文件路径的区别：

- `/`代表的是根目录；
- `./`代表的是当前目录；
- `../`代表的是父级目录；

这两个方法均属于path模块，先引入path模块:

```js
const path = require("path")
```

node.js中，__dirname表示的是当前被执行js文件的绝对路径，

2.path.join()方法 使用平台特定的分隔符将所有给定的path片段连接到一起，并规范化生成的路径。

```js
const SRC_PATH = path.join(__dirname, '../src') // 定义src的路径
```

3.path.resolve()方法 把传入的路径或者路径片段解析为绝对路径

## 七、安装elemnetUI

```json
cnpm i -S element-ui // 安装elementUi框架
cnpm i -D babel-plugin-component // 安装该插件 支持按需引入 减小压缩包体积
```

## 八、安装vue-router

Vue Router 是 [Vue.js](http://cn.vuejs.org/) 官方的路由管理器。它和 Vue.js 的核心深度集成，让构建单页面应用变得易如反掌。

```json
cnpm i -S vue-router
```

### 8.1.用路由能够干什么

使用Vue.js，我们已经可以通过组合组件来组成应用程序，当你要把VueRouter添加进来，我们需要做的是，将组件(components)映射到路由(routes)，然后告诉VueRouter在哪里渲染它们。