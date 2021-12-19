## MobX5

### 1. MobX 介绍

简单，可扩展的状态管理库。

MobX 是由 Mendix，Coinbase，Facebook 开源和众多个人赞助商所赞助。

React 和 MobX 是一对强力组合，React 负责渲染应用的状态，MobX 负责管理应用状态供 React 使

### 2. MobX 浏览器支持

MobX 5 版本运行在任何支持 ES6 proxy 的浏览器，不支持 IE11，Node.js 6。

MobX 4 可以运行在任何支持 ES5 的浏览器上。

MobX 4 和 5 的 API 是相同的。

### 3. 启用装饰器语法

首先需要安装装饰器语法 babel 插件：

`npm install @babel/plugin-proposal-decorators `

然后进行如下的 babel 配置：

```json
{
  "babel": {
    "plugins": [
      [
        "@babel/plugin-proposal-decorators",
        {
          "legacy": true
        }
      ]
    ]
  }
}
```

解决 vscode 编辑器关于装饰器语法的警告：

修改配置：`"javascript.implicitProjectConfig.experimentalDecorators": true`

### 4. 在 React 中使用 MobX

#### 4.1 安装相关的依赖包

首先下载如下依赖包：

`npm install mobx mobx-react` 或者 `yarn add mobx mobx-react`

其中 `mobx-react` 可以让 mobx 与 React 完美的结合使用。

#### 4.2 Mobx 的工作流程

MobX 中所有的状态 State 都存储在 Store 中；当 Store 中的状态发生改变时，会将 State 自动同步到 View 中。View 如果想更新状态，那么就需要触发一个 Action 函数，由 Action 去更改 Store 中的 State，然后再将最新的 State 同步到 View 中。

![mobx5-flow](./images/mobx5-flow.png)

#### 4.3 Mobx 的基本使用

