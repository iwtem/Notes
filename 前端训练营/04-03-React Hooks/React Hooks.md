## React Hooks

### 1. React Hooks 的作用

对函数型组件进行增强，让函数式组件可以存储状态，可以拥有处理副作用的能力。让开发者在不使用类组件的情况下，实现相同的功能。

> **组件副作用**：在组件中，只要是不把数据转换为视图的操作，都属于副作用代码。 比如获取 DOM 元素并添加事件监听、设置定时器、 发送 Ajax 请求等。在类组件中，会使用生命周期去处理这些副作用；而在函数组件中， 则使用 hooks 处理这些副作用。

### 2. 类组件的缺点

1. **缺少逻辑副用机制**

   为了复用逻辑增加无实际渲染效果的组件（比如高阶组件，在原有组件上包裹一层组件），增加了组件的嵌套层级，显示十分臃肿，增加了调试的难度以及运行效率的降低。

2. **类组件经常会变得很复杂难以维护**

   将一组相干的业务逻辑拆分到了多个生命周期函数中，在一个生命周期函数内存在多个不相干的业务逻辑。

3. **类成员方法不能保证 this 指向的正确性**

### 3. React Hooks 的使用

详见 Hooks API：https://zh-hans.reactjs.org/docs/hooks-reference.html

新版 Hooks API 链接：https://beta.reactjs.org/reference

### 4. useState 的原理实现

#### 4.1 实现原理分析

**参数**：`initalState` 

`useState` 钩子函数接收一个可选的参数，作为状态的初始值。

**返回值**：`[state, setState]`

`useState` 返回一个数组，该数组存在两个值 `state` 和 `setState`。

- `state` 组件的状态；

- `setState` 是设置状态的函数，该函数支持两种类型的参数，如果是一个值，则表示将该值更新为最新的 `state`，如果是一个函数，则会将该函数的返回值作为最新的 `state`，并且该函数会传入上一次的 `state`，用于计算下一个 `state`。

**其他特性**:

1. 同组件的多个 `state`

   React 组件内部可以维护多个状态，即 `useState` 函数可以在同一个组件中被多次调用，每一次的调用都会返回不同的 `state`，并且每个 `state` 都是独立的，互补影响。

   其原理在于 `useState` 内部会维护一个状态的数组和一个设置状态的函数的数组，在组件第一次加载初始化状态时，会将每个不同的状态和更新状态的函数按照调用的先后顺序维护在两个数组中；当再次更新的时候，再依次按照调用的顺序从数组中获取对应的状态和函数。

   当组件在调用 `setState` 更新状态的时候，是如何得知更新状态数组的哪个值呢？见2介绍。

2. `setState` 的调用原理

   `setState` 之所以知道更新状态数组的哪个下标值，是因为在创建时候使用了闭包的特性，这样在其内部就维护了一个下标。

3. 组件的更新

   当调用 `setState` 时，如果传入的是一个值，那么会直接将 `state` 更新为最新的值；如果是一个函数，那么会将当前的 `state` 传入，然后调用该函数获取最新的`state` 进行更新。接着需要进行更新组件，通过 `ReactDOM.render` 方法重新渲染组件，组件内部再次依次调用 `useState` ，获取最新的状态更新到视图中。

#### 4.2 代码实现

基础代码，`useState` 待实现：

```react
import ReactDOM from 'react-dom';

function render() {
  ReactDOM.render(
    <App />,
    document.getElementById('root'),
  );
}

const App = () => {
  // TODO useState 待实现
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <h3>{count}</h3>
      <button onClick={() => setCount(count + 1)}>点击+1</button>
    </div>
  )
}
```

简版的 `useState` 实现：

```react
let state;
function useState(initalState) {
  // state 存在值表示非首次加载，否则使用初始值
  state = state ? state : initalState;
  
  // state 的更新函数
  const setState = (nextState) => {
    // nextState 是一个函数
    if(typeof nextState === 'function') {
      // 调用该函数，传入当前的 state，并获取最新的 state
      state = nextState(state);
    } else {
      state = nextState;
    }
    
    // setState 调用完成之后，要更新组件
    render();
  }
}
```

支持多个状态的管理：

```react
// 获取状态的下标
let stateIndex = 0;
// 将状态和函数维护在数组中，按照调用 useState 的顺序存入和取值
const states = [];
const setStates = [];

// 使用闭包特性，让 setState 记住需要更新的是哪个 state
function createSetState(index) {
  // 返回的函数是 setState
  return function setState(nextState) {
    // nextState 是一个函数
    if(typeof nextState === 'function') {
      // 调用该函数，传入当前的 state，并获取最新的 state
      states[index] = nextState(states[index]);
    } else {
      states[index] = nextState;
    }
    
    // setState 调用完成之后，要更新组件
    render();
  }
}

function useState(initalState) {
  // 根据下标更新数组中对应的 state
  const nextState = states[stateIndex] ? states[stateIndex] : initalState;
  states[stateIndex] = nextState;
  // 根据下标获取 setState，并维护在数组中
  const setState = createSetState(stateIndex);
  setStates.push(setState);
  
  // 更新下标，留给下次调用 useState 使用
  stateIndex++;
	
  return [nextState, setState];
}
```

最后，更新基础代码，在组件每次重新渲染的时候，将 index 重置：

```react
import ReactDOM from 'react-dom';

function render() {
  // 重置下标，保证重新依次调用 useState 的顺序
  stateIndex = 0;

  ReactDOM.render(
    <App />,
    document.getElementById('root'),
  );
}

const App = () => {
  const [count, setCount] = useState(0);
  const [title, setTitle] = useState('');
  
  return (
    <div>
      <h3>{title}{count}</h3>
      <button onClick={() => setCount(count + 1)}>点击+1</button>
      <button onClick={() => setCount((count) => count + 2)}>点击+2</button>
			<button onClick={() => setTitle('标题')}>点击+1</button>
    </div>
  )
}
```

### 5. useEffect 的原理实现

#### 5.1 实现原理分析

`useEffect` 的实现原理比 `useState` 简单一些，简单点在于只需要在组件的更新的时候判断是否需要执行回调函数，没有返回的函数需要去执行，其他基本与 `useState` 相同。

**参数**：`callback, depts`

接收两个参数，第一个参数 `callback` 必填，`callback` 必须是一个函数，无入参，返回值可选，如果返回一个值则该值必须是函数。第二个可选参数 `depts` 是依赖的值，如果传入必须是一个数组，当 `depts` 数组的某个值发生改变，`callback` 则会再次执行。

**返回值**：无

基础代码：

```react
import React from 'react';
import ReactDOM from 'react-dom';

function render() {
  ReactDOM.render(
    <App />,
    document.getElementById('root')
  )
}

const App = () => {
  const [count, setCount] = React.useState(0);
  const [title, setTitle] = React.useState('标题');
  
  // 不依赖任何值，任何状态发生改变，callback 都会重新执行
  useEffect(() => {
		console.log(count, title);
  });
  
  // 依赖列表是空数组，callback 只会在组件挂载后执行一次，后续不会执行
  useEffect(() => {
		console.log(count, title);
  }, []);
  
  // 依赖 title 和 count，callback 会在 title 或 count 发生改变后重新执行
  useEffect(() => {
		console.log(count, title);
  }, [title]);
  
  return (
    <div>
      <h3>{title}-{count}</h3>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <button onClick={() => setTitle(`标题${Math.random()}`)}>更新标题</button>
    </div>
  )
}
```

`useEffect` 功能实现

```react
// useEffect 可在一个组件中多次调用，所以需要记录是哪次调用的参数
// 当前执行的 useEffect 的下标
let effectIndex = 0;
// 当前执行的 useEffect 的依赖数组，用于判断 callback 是否需要重新执行
const effectDepts = [];

// 判断
function needExecute(depts) {
  const preDepts = effectDepts[effectIndex];

  if (!depts || !preDepts) {
    return true;
  }

  return depts.some((dept, index) => dept !== preDepts[index]);
}

const useEffect = (callback, dept) => {
  // callback 必须是一个函数
  if (typeof callback !== "function") {
    throw new Error("callback not a function");
  }

  // dept 存在则必须是一个数组
  if (dept && Object.prototype.toString.apply(dept) !== "[object Array]") {
    throw new Error("depts must be a array");
  }

  // 获取前一次记录的依赖数组
  const preDepts = effectDepts[effectIndex];
  
  // 如果 depts 不存在，则直接执行 callback
  // 如果 depts 某项发生改变，也执行 callback
  if (!depts || !preDepts || depts.some((dept, index) => dept !== preDepts[index])) {
    callback();
  }

  // 将当前的 depts 进行更新
  effectDepts[effectIndex] = dept;
  effectIndex++;
};
```



















