## 简介
利用 HTML 里的信息，浏览器将会创建一个叫做 DOM 的家伙。[[html]]
使用 JavaScript 书写代码来向 DOM API 下达命令，网页内容便随着 DOM 的更改而发生变化。
React 由 JavaScript 开发而成，它的设计目标之一是让我们在大多数情况下不再需要直接操作 DOM API。
## 新增的特点：

 组件（Component）
- React 组件有两条重要的属性： 
1. 组件是可组合的（composable）。组件的目的在于重用，我们可以使用现有组件创建新组件。
2. 一个组件独立于其它组件（independent）。当修改一个组件时，不相干的组件不会受到干扰。

声明式界面编程（Declarative UI） 
- 声明式编程软件，将问题的定义自动翻译成为具体的指令，便于更多地关注问题本身。
- React提供模板（抽象层级）供用户使用
响应式 DOM 更新机制（Reactive DOM updates）
- 只需要将该数据与相应界面元素关联好，就不需要再做任何后续干涉。当数据发生变化时，React 将自动对相关 DOM 元素做相应的调整。

**注意**：创建React项目不需要Node / NPM。但它可以让你使用一些有用的工具，比如Create React App和Next.js，让你 更容易管理React应用。
- 在没有Node和NPM的情况下，如果你想要在React项目中添加一个新的JavaScript库(比如day.js，一个用于格式化日期 的库)，你需要手动添加一组 script 标记，并自己管理它们。
- 使用Node和NPM(或者像Yarn这样的包管理器)，简单运行一个命令来安装任何JavaScript库。

### 创建项目
```
# for Create React App 
npx create-react-app my-react-app 

# for Vite 
npm init vite@latest my-react-app --template react 

# for Next.js 
npx create
```
## 核心思想
![[Pasted image 20251129181947.png]]
### 前端架构
![[Pasted image 20251129182145.png]]
### Why SSR+BFF
![[Pasted image 20251129182246.png]]
### 性能体验优化
![[Pasted image 20251129182419.png]]
## 小程序开放
目录结构：
![[Pasted image 20251129182715.png]]
页面组成：
![[Pasted image 20251129182754.png]]
生命周期：
![[Pasted image 20251129183004.png]]
## 组件通信与插槽
同一组件复用：
- 单层
```JSX
function Article({ title, content, active }) {
  return (
    <div>
      <h2>{title}</h2>
      <p>{content}</p>
      <p>状态：{active ? '显示中' : '已隐藏'}</p>
    </div>
  );
}

export default function App() {
  return (
    <div>
      <Article
        title="标题1"
        content="内容1"
        active={true}
      />
      <Article
        title="标题2"
        content="内容2"
      />
      <Article
        title="标题3"
        content="内容3"
        active={true}
      />
    </div>
  );
}
```
- 父子双层对象
```jsx
function Detail({ content, active }) {
  return (
    <>
      <p>{content}</p>
      <p>状态：{active ? '显示中' : '已隐藏'}</p>
    </>
  );
}


function Article({ title, detailData }) {
  return (
    <div>
      <h2>{title}</h2>
      <Detail {...detailData} />
    </div>
  );
}

export default function App() {
  const articleData = {
    title: '标题1',
    detailData: {
      content: '内容1',
      active: true,
    },
  };

  return <Article {...articleData} />;
}
```
- 将JSX作为Props转递（组件插槽）
```jsx
function List({ children, title, footer = <div>默认底部</div> }) {
  return (
    <div>
      <h2>{title}</h2>
      <ul>{children}</ul>
      {footer}
    </div>
  );
}


export default function App() {
  return (
    <List
      title="列表1"
      footer={<p>这是底部内容1</p>}
    >
      <li>内容1</li>
      <li>内容2</li>
      <li>内容3</li>
    </List>

    <List
      title="列表1"
    >
      <li>内容X</li>
      <li>内容Y</li>
      <li>内容Z</li>
    </List>
  );
}
```
没有给值得时候 `footer = <div>默认底部</div>` footer会默认使用div内得内容当作值。
- 子组件向父组件传值
父组件给子组件进行自定义事件的设置，再通过事件触发后向父组件传递参数的方式实现。
```jsx
function Detail({ onActive }) {
  const [status, setStatus] = useState(false);

  function handleClick() {
    const newStatus = !status;
    setStatus(newStatus);
    onActive(newStatus); // 把最新状态回传给父组件
  }
  return (
    <div>
      <button onClick={handleClick}>按钮</button>
      <p style={{ display: status ? 'block' : 'none' }}>
        Detail的内容
      </p>
    </div>
  );
}

export default function App() {
  function handleActive(status) {
    console.log(status);
  }

  return (
    <Detail onActive={handleActive} />
  );
}
```
- 同级组件也可以传值
常用方式：借助父组件来进行中转。
- 使用Context进行多级组件传值
```js
import { createContext, useContext } from 'react';

export function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}

export function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error(`未知的 level: `+level);
  }
}

export default function App() {
  return (
    <div>
      <Section>
        <Heading>主标题</Heading>

        <Section>
          <Heading>副标题</Heading>
          <Heading>副标题</Heading>
          <Heading>副标题</Heading>

          <Section>
            <Heading>子标题</Heading>
            <Heading>子标题</Heading>
            <Heading>子标题</Heading>

            <Section>
              <Heading>子子标题</Heading>
              <Heading>子子标题</Heading>
              <Heading>子子标题</Heading>
            </Section>
          </Section>
        </Section>
      </Section>
    </div>
  );
}
```
## React Hooks
- Reducer用于同一管理状态的操作方式
就是，你点击了一个事件，其余的事件需要知道这个操作。
```JSX
import React, { useReducer } from 'react';

function countReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return state + 1;
    case 'decrement':
      return state - 1;
    default:
      throw new Error('未知 action: ' + action.type);
  }
}


export default function App() {
  const [state, dispatch] = useReducer(countReducer, 0);
  const handleIncrement = () => dispatch({ type: 'increment' });
  const handleDecrement = () => dispatch({ type: 'decrement' });

  return (
    <div style={{ padding: 10 }}>
      <button onClick={handleDecrement}>-</button>
      <span style={{ margin: '0 10px' }}>{state}</span>
      <button onClick={handleIncrement}>+</button>
    </div>
  );
}
```
- useRef，希望记住上一个值
```JSX
import React, { useState, useRef } from 'react';

export default function App() {
  const [count, setCount] = useState(0);
  const prevCount = useRef();

  function handleClick() {
    prevCount.current = count;   // 先保存旧值
    setCount(count + 1);         // 再更新状态
  }

  return (
    <div>
      <p>最新的 count：{count}</p>
      <p>上次的 count：{prevCount.current}</p>
      <button onClick={handleClick}>增大 count</button>
    </div>
  );
}
```
Ref可以实现子组件访问父组件的目的。
- useEffect，副作用函数
```jsx
import React, { useEffect, useState } from 'react';

export default function App() {
  const [count, setCount] = useState(0);

  const handleIncrement = () => setCount(count + 1);
  const handleDecrement = () => setCount(count - 1);

  useEffect(() => {
    console.log('useEffect');
  }, [count]);

  return (
    <div style={{ padding: 10 }}>
      <button onClick={handleDecrement}>-</button>
      <span style={{ margin: '0 10px' }}>{count}</span>
      <button onClick={handleIncrement}>+</button>
    </div>
  );
}
```
- useMemo,进行数据缓存的钩子
```jsx
import React, { useState, useMemo } from 'react';

export default function App() {
  const [a, setA] = useState(1);
  const [b, setB] = useState(2);
  const [c, setC] = useState(0); // 与计算无关的干扰状态

  // 仅当 a 或 b 变化时才重新计算，避免在 c 变化时重复执行耗时运算
  const sum = useMemo(() => {
    console.log('重新计算 sum');
    return a + b; // 假设这里是代价较高的计算
  }, [a, b]);

  return (
    <div style={{ padding: 20 }}>
      <p>a: {a} <button onClick={() => setA(a + 1)}>+1</button></p>
      <p>b: {b} <button onClick={() => setB(b + 1)}>+1</button></p>
      <p>c: {c} <button onClick={() => setC(c + 1)}>+1（不会触发 sum 重算）</button></p>
      <hr />
      <p>sum = {sum}</p>
    </div>
  );
}
```
## `React+TypeScript`
