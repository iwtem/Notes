## webpack 性能优化

### 问题分析

现在的系统主要存在以下几点比较明显的问题：

- 页面跳转逻辑有误
- 首次页面加载缓慢
- 构建时间较长



- 路径的跳转，当打开根路径后，首先会被重定向到 /index.html，再跳转到登录页，存在大量的资源请求的浪费
- 单个 chunkjs 文件比较大，Gzip 未压缩之前最大的有 21.6 MB，压缩后仍有 5.2 MB，首次加载速度很慢
- 无效的文件引入，存在很多未使用的静态文件的引入，也会增加首屏的加载时长





chunk js 可以减少非必要的请求



### 1. 优化前后对比

优化前构建时长：280123 ms		优化后构建时长：167610 ms

优化前 chunk 总大小：39M				 优化后 chunk 总大小：

优化前最大 chunk 大小：19.2M				 优化后 chunk 总大小：

### 2. 优化思路

- 减少非必要的入口文件
- 减少非必要的 babel 插件配置
- 减少 webpck 插件，针对开发与生产不同的环境配置对应的插件
- 使用最新版本的 webpack 与 node
- 优化其他 webpack 配置
- 开启缓存（开发环境）

### 3.1 减少入口文件 entry

由之前的 9 个入口减少到 6 个入口

#### 3.1 public 文件夹下的静态资源打包

优化之前，public 下所以的静态资源统一作为一个单独的入口进行打包

优化之后，使用 copy-webpack-plugin 拷贝到输出目录下，不作为一个独立的打包入口

#### 3.2 i18n.js

优化之前，i18n.js 作为一个单独的入口文件进行打包

优化之后，将 i18n.js 拷贝到 public 下，不作为一个打包入口

#### 3.3 删除/合并 compileMap.js 中不再使用的入口文件

1. WeChat/transfer 不再使用，删除
2. 合并首页 -> spa 同一个路由下（待定）
3. 移动 login.js 和 login.html 到 public 下，不作为打包入口（待定）
4. 合并 invitation、invitationV2 到 spa 路由下，不作为打包入口（待定）
5. 删除 dependencies 入口，都通过 webpack 打包和 tree shaking



### 3.2 减少入口文件 entry



### 3.3 减少 webpack 的 plugin 和 loader



### 3.4 更新 webpack 及 node 到最新版本

避免使用生产/开发环境下才需要用的工具，用到了其他环境

### 3.5 优化 webpack 配置项

#### 3.5.1 outpu.pathinfo 设置 false



#### 3.5.2 resolve.extensions 配置



当模块导入语句未携带文件后缀时，如 `import './a'` ，Webpack 会遍历 `resolve.extensions` 项定义的后缀名列表，尝试在 `'./a'` 路径追加后缀名，搜索对应物理文件。

在 Webpack 5 中，`resolve.extensions` 默认值为 `['.js', '.json', '.wasm']` ，这意味着 Webpack 在针对不带后缀名的引入语句时可能需要执行三次判断逻辑才能完成文件搜索，针对这种情况，可行的优化措施包括：

- 调整顺序，频率出现最高的文件后缀要优先放在最前面，以做到尽快的退出寻找过程
- 后缀尝试列表要尽可能的小，不要把项目中不可能存在的情况写到后缀尝试列表中

- 修改 `resolve.extensions` 配置项，减少匹配次数

- 代码中尽量补齐文件后缀名，比如 data.json 类似的文件，都补齐 .json 后缀名
- 设置 `resolve.enforceExtension = true` ，强制要求开发者提供明确的模块后缀名，这种做法侵入性太强，不太推荐



#### 3.5.2 resolve.modules 配置

当 Webpack 遇到 `import 'lodash'` 这样的 npm 包导入语句时，会尝试先当前项目的 `node_modules` 搜索资源，如果找不到则按目录层级尝试逐级向上查找 `node_modules` 目录，如果依然找不到则最终尝试在全局 `node_modules` 中搜索。

在一个依赖管理执行的比较良好的业务系统中，我们通常会尽量保持 `node_modules` 资源的高度内聚，控制在有限的一两个层级上，因此 Webpack 这一逐层查找的逻辑大多数情况下实用性并不高，开发者可以通过修改 `resolve.modules` 配置项，主动关闭逐层搜索功能。



#### 3.5.3 resolve.mainFiles 配置

与 `resolve.extensions` 类似，`resolve.mainFiles` 配置项用于定义文件夹默认文件名，例如对于 `import './dir'` 请求，假设 `resolve.mainFiles = ['index', 'home']` ，Webpack 会按依次测试 `./dir/index` 与 `./dir/home` 文件是否存在。

因此，实际项目中应控制 `resolve.mainFiles` 数组数量，减少匹配次数。



#### 3.5.4 noParse 跳过文件编译

有不少 npm 包默认提供了提前打包好，不需要做二次编译的资源版本，例如：

- Vue 包的 `node_modules/vue/dist/vue.runtime.esm.js` 文件
- React 包的 `node_modules/react/umd/react.production.min.js` 文件

对使用方来说，这些资源版本都是高度独立、内聚的代码片段，没必要重复做依赖解析、代码转译操作，此时可以使用 `module.noParse` 配置项跳过这些 npm 包，例如：

```js
// webpack.config.js
module.exports = {
  //...
  module: {
    noParse: /vue|lodash|react/,
  },
};
```

配置该属性后，任何匹配该选项的包都会跳过耗时的分析过程，直接打包进 chunk，提升编译速度。



#### 3.5.5 最小化 Loader 作用范围

Loader 组件用于将各式文件资源转换为可被 JavaScript 理解、运行的代码片段，正是这一特性支撑起 Webpack 强大的资源处理能力。不过，Loader 在执行内容转换的过程可能需要做大量的 CPU 运算操作，例如 babel-loader、eslint-loader、vue-loader 等，因此开发者有必要根据实际需求，通过 `module.rules.include`、`module.rules.exclude` 等配置项限定 Loader 的执行范围，例如：

```js
// webpack.config.js
module.exports = {
    // ...
    module: {
        rules: [{
            test: /\.js$/,
            exclude: /node_modules/,
            // include: path.join(__dirname, './src'),
            use: ['babel-loader', 'eslint-loader']
        }]
    }
};
```

示例配置 `exclude: /node_modules/` 属性后，Webpack 在处理 `node_modules` 中的 js 文件时会直接跳过这个 `rule` 项，不会为这些文件执行后续的 Loader。



#### 3.5.6 最小化 watch 监控范围

在 watch 模式下(通过 `npx webpack --watch` 命令启动)，Webpack 会持续监听项目所有代码文件，发生变化时重新构建最新产物。不过，通常情况下前端项目中某些资源并不会频繁更新，例如 `node_modules` ，此时可以设置 `watchOptions.ignored` 属性忽略这些文件，例如：

```js
// webpack.config.js
module.exports = {
  //...
  watchOptions: {
    ignored: /node_modules/
  },
};
```



#### 3.5.7 跳过 TS 类型检查

JavaScript 本身是一门弱类型语言，这在多人协作项目中经常会引起一些不必要的类型错误，影响开发效率。随前端能力与职能范围的不断扩展，前端项目的复杂性与协作难度也在不断上升，TypeScript 所提供的静态类型检查能力也就被越来越多人所采纳。

不过，类型检查涉及 AST 解析、遍历以及其它非常消耗 CPU 的操作，会给工程化流程引入性能负担，必要时开发者可选择关闭编译主进程中的类型检查功能，同步用 `fork-ts-checker-webpack-plugin` 插件将其剥离到单独进程执行，例如对于 `ts-loader`：

```js
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin');

module.exports = {
  // ...
  module: {
    rules: [{
      test: /\.ts$/,
      use: [
        {
          loader: 'ts-loader',
          options: {
            transpileOnly: true
          }
        }
      ],
    }, ],
  },
  plugins:[
    new ForkTsCheckerWebpackPlugin()
  ]
};
```



#### 3.5.8 慎用 source-map

`source-map` 是一种将经过编译、压缩、混淆的代码代码映射回源码的技术，它能够帮助开发者迅速定位到更有意义、更结构化的源码中，方便调试。不过，同样的 `source-map` 操作本身也有很大性能开销，建议读者根据实际场景慎重选择最合适的 `source-map` 方案。

针对 `source-map` 功能，Webpack 提供了 `devtool` 选项，可以配置 `eval`、`source-map`、`cheap-source-map` 等值，不考虑其它因素的情况下，最佳实践：

- 开发环境使用 `eval` ，确保最佳编译速度
- 生产环境使用 `source-map`，获取最高质量



#### 3.5.9 module.rules.oneof 优化 loader 匹配



### 3.6 删除非必要的 babel 插件

- @jacklu/i18n，项目中暂时未找到改插件🚗
- @babel/plugin-proposal-nullish-coalescing-operator，@babel/preset-env 已经包含，需要配置该插件🚗
- react-dev-inspector/plugins/babel，暂时先给删除🚗

### 3.7 删除无用的 package 包

- finally-pllyfill，浏览器已经支持 Promise.prototype.finally，所以无需安装此包

