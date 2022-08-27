## CSS IN JS

CSS-IN-JS 是 WEB 项目中将 CSS 代码捆绑在 JavaScript 代码中的解决方案。这种方案旨在解决 CSS 的局限性，例如缺乏动态功能，作用域和可移植性。

### 1. css in js 的优点

1. 让 CSS 代码拥有独立的作用域, 阻止 CSS 代码泄露到组件外部, 防止样式冲突；
2. 让组件更具可移植性, 实现开箱即用, 轻松创建松耦合的应用程序；
3. 让组件更具可重用性, 只需编写一次即可, 可以在任何地方运行. 不仅可以在同一应用程序中重用组件, 而且可以在使用相同框架构建的其他应用程序中重用组件；
4. 让样式具有动态功能, 可以将复杂的逻辑应用于样式规则, 如果要创建需要动态功能的复杂UI, 它是理想的解决方案。

### 2. css in js 的缺点

1. 为项目增加了额外的复杂性
2. 自动生成的选择器大大降低了代码的可读性

### 3. Emotion 库的使用

Emotion 是一个旨在使用 JavaScript 编写 CSS 样式的库。

使用之前，需要安装如下 npm 包:

`npm install @emotion/core @emotion/styled`

#### 3.1 emotion 的两种配置方式

**方式一：：JSX Progma**

通知 babel，不再需要将 jsx 语法转换为 React.createElement 方法，而是需要转换为 jsx 方法。

|        | Input                      | Output                                               |
| ------ | -------------------------- | ---------------------------------------------------- |
| Before | `<img src="avatar.png" />` | `React.createElement('img', { src: 'avatar.png' });` |
| After  | `<img src="avatar.png" />` | `jsx('img', { src: 'avatar.png' });`                 |

然后在代码中引入 jsx，并添加注释，告诉 `babel` 如何进行转换

```react
/** @jsx jsx */
import { jsx } from "@emotion/core";

export default function App() {
  return (
    <div css={{ background: "yellow" }}>
      <h1>Hello CodeSandbox</h1>
      <h2>Start editing to see some magic happen!</h2>
    </div>
  );
}

```

此类方法不推荐使用，因为每次使用都需要引入额外的方法还有添加的注释。

**方式二：Babel Preset**

安装如下 npm 包：

`npm install --save-dev @emotion/babel-preset-css-prop`

添加 `babel` 的配置如下：

```json
"presets": [
  "react-app",
  "@emotion/babel-preset-css-prop"
]
```

#### 3.2 css 方法的使用

1. String Styles

   通过书写字符串的形式定义样式：

   ```react
   // 返回一个对象
   const style = css`
   	width: 100px;
   	height: 100px;
   	background: skyblue;
   `;
   
   function App() {
     return <div css={style}>App Works...</div>;
   }
   ```

2. Object Styles

   通过定义对象的方式定义样式：

   ```react
   // 返回一个对象
   const style = css({
   	width: 100,
   	height: 100,
   	background: 'skyblue',
   });
   
   function App() {
     return <div css={style}>App Works...</div>;
   }
   ```

#### 3.3 css 属性的优先级

props 对象中的 css 属性优先级高于组件内部的 css 属性. 在调用组件时可以在覆盖组件默认样式.

#### 3.4 Style Components

样式化组件就是用来构建用户界面，是 emotion 库提供的另一种为元素添加样式的方式。

**两种简单的使用方式**

```react
import styled from '@emotion/styled';

const Button = styled.button`
	color: red;
`;

// 或者
const Button = styled.button({
  color: 'yellow',
});
```

**根据 props 覆盖样式**

```react
import styled from '@emotion/styled';

const Button = styled.button`
	width: 100px;
	height: 30px;
	background: ${props => props.bgColor || 'skyblue'};
`;

// 或者
const Button = styled.button(props => ({
	width: 100,
	height: 30,
	background: props.bgColor || 'skyblue',
}));

// 或者
const Button = styled.button({
	width: 100,
	height: 30,
	background: 'skyblue',
}, props => ({
	background: props.bgColor, // 无需判断 props，内部会进行判断覆盖
}));

function App() {
  return <Button bgColor="yellow">按钮</Button>
}
```

**为任意组件添加样式**

```react
import styled from '@emotion/styled';

// 被 styled 调用的组件，会多出 className 属性
const Demo = ({ className }) => <div className={className}>Demo</div>

const Fancy= styled(Demo)`
  color: green;
`;

// 或者
const Fancy= styled(Demo)({
  color: 'green',
});
```

**通过父组件设置自组件的样式**

```react
import styled from '@emotion/styled';

const Child = styled.div`
	color: red;
`;

const Parent = styled.div`
	${Child} {
		color: yellow;
	}
`;

// 或者
const Parent = styled.div(
	[Child]: {
		color: 'yellow',
	}
);

function App() {
  // 当 Child 被 Parent 包裹，其 color 样式会被覆盖
  return (
    <div>
      <Child>Child</Child>
      <Parent>
        <Child>Child Parent</Child>
      </Parent>
    </div>
  )
}
```

**嵌套选择器 &**

& 表示的是组件本身

```react
const Container = styled.div`
	color: red;
	& > a: {
		color: pink;
	}
`;
```

**as 属性**

要使用组件中的样式, 但要更改呈现的元素, 可以使用 as 属性

```react
const Button = styled.button`
	color: red;
`;

function App() {
  // as 属性会把 button 元素更改为 a 元素
  return <Button as="a" href="#">按钮</Button>;
}
```

#### 3.5 样式的组合

在样式组合中, 后调用的样式优先级高于先调用的样式

```react
const base = css`
	color: yellow;
`;

const blue = css`
	color: blue;
`;

function App() {
  return <button css={[base, blue]}>按钮</button>;
}
```

#### 3.6 全局样式

```react
import { css, Global } from '@emotion/core';

const styles = css`
	body {
		margin: 0;
	}
`;

function App() {
  return (
    <>
    	<Global styles={styled} />
    	App works...
    </>
  )
}
```

#### 3.7 关键帧动画

```react
import { css, keyframes  } from '@emotion/core';

const move = keyframes`
	0% { left: 0; top: 0; background: pink; }
	100% { left: 300px; top: 300px; background: skyblue; }
`

const styles = css`
	position: absolute;
	width: 100px;
	height: 100px;
	animation: ${move} 2s ease infinite alternate;
`;

function App() {
  return (
    <div css={styles}>
    	App works...
    </>
  )
}
```

#### 3.8 主题

1. 首先需要下载主题模块

`npm install emotion-theming`

2. 然后引入 ThemeProvider 组件

```react
import ThemeProvider from 'emotion-theming';
```

3. 将 ThemeProvider 放置在视图在最外层

```react
import ThemeProvider from 'emotion-theming';

function App() {
  return (
    <ThemeProvider>
    app works...
  	</ThemeProvider>
  );
}
```

4. 添加主题内容

```react
import ThemeProvider from 'emotion-theming';

const theme = {
  colors:{
    primary: 'hotpink',
  }
};

function App() {
  return (
    <ThemeProvider theme={theme}>
    app works...
  	</ThemeProvider>
  );
}
```

5. 获取主题内容（方式一）

```react
const getPrimaryColor = props => css`
	color: ${props.colors.primary};
`;

<div css={getPrimaryColor}></div>
```

6. 获取主题内容（方式二）

```react
import { useTheme } from 'emotion-theming';

function Demo() {
  // 返回主题对象
  const theme = useTheme();
  
  const style = css`
		color: ${theme.colors.primary};
	`;

  <div css={style}></div>
}
```

