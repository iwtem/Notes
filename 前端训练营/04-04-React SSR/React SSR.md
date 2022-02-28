## React SSR

### 1. 客户端渲染 CSR

CSR：Client Side Rendering

服务器端仅返回 JSON 数据，DATA 和 HTML 在客户端进行渲染。

#### 1.1 客户端渲染存在的问题

1. 首屏等待时间长，用户体验差
2. 页面结构为空，不利于 SEO

### 2. 服务端渲染 SSR

SSR：Server Side Rendering

服务器端返回 HTML，DATA 和HTML 在服务器端进行渲染。

#### 2.1 React SSR 同构

同构指的是代码复用，即实现客户端和服务器端最大程度的代码复用。