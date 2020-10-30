# Ref

### 引言

- 本篇从 React Refs 的:
  - 使用场景
  - 使用方式
  - 注意事项
- 到 createRef 与 Hook useRef 的对比使用;
- 最后以 React createRef 源码结束;

---

- 剖析整个 React Refs;
- 关于 React.forwardRef 会在 '另' 一篇文章深入探讨;

## 一、Refs

- React 的 '核心思想' 是:
  - 每次对于界面 state 的改动:
    - 都会重新渲染整个 Virtual DOM;
      - 然后'新老'的两个 Virtual DOM 树进行 diff（协调算法）;
        - 对比出变化的地方:
          - 然后通过 render 渲染到实际的 UI 界面;

---

- 使用 Refs:
  - 为我们提供了一种:
    - 绕过 '状态更新' 和 '重新渲染' 时:
      - 访问元素的方法;
  - 这在 '某些用例' 中很有用;
    - 但不应该作为 props 和 state 的替代方法;

---

- 在项目开发中:
  - 如果我们可以使用:
    - 声明式
    - 提升 state 所在的组件层级（状态提升）
  - 方法来更新组件;
    - 最好不要使用 refs;

### 使用场景

#### 管理焦点（如文本选择）或处理表单数据:

- Refs 将管理文本框当前焦点选中;
- 或文本框其它属性;

---

- 在大多数情况下,我们推荐使用:
  - '受控组件' 来处理 '表单数据';
    - 在一个受控组件中:
      - '表单数据' 是由 'React 组件' 来管理的;
        - 每个 '状态更新' 都编写 '数据处理' 函数;
- 另一种 '替代方案' 是:
  - 使用非受控组件:
    - 这时 '表单数据' 将交由:
      - DOM 节点来处理;
  - 要编写一个非受控组件:
    - 就需要使用 Refs :
      - 来从 DOM 节点中 '获取' 表单数据;

```js
import React from "react";

class NameForm extends React.Component {
  constructor(props) {
    super(props); // super()是必须，虽然用不到props，但是，还是放一下，完美一点;
    this.input = React.createRef();
  }

  handleSubmit = (e) => {
    console.log("A name was submitted:" + this.input.current.value);
    e.preventDefault();
  };

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={this.input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}

export default NameForm;
```

- 因为 '非受控组件' 将 '真实数据' 储存在 'DOM 节点中';
  - 所以在使用 '非受控组件' 时:
    - 有时候反而更容易 '同时集成' React 和非 React 代码;
- 如果你不介意代码美观性;
- 并且希望快速编写代码;
- 使用 '非受控组件' 往往可以减少你的代码量;
  - 否则，你应该使用受控组件;

#### 媒体播放：

- 基于 React 的:
  - 音乐或视频播放器;
    - 可以利用 Refs 来管理:
      - 当前状态（播放/暂停），或管理播放进度等;
        - 这些更新不需要进行状态管理;

#### 触发强制动画：

- 如果要在元素上触发强制动画时:
  - 可以使用 Refs 来执行此操作;

#### 集成第三方 DOM 库

### 使用方式

- Refs 有 三种实现：

#### 1、方法一：通过 createRef 实现

- createRef 是 **React v16.3 ** 新增的 API:
  - 允许我们访问 DOM 节点;
  - 或在 render 方法中创建的 React 元素;

---

- Refs 是使用 React.createRef() 创建的;
  - 并通过 ref 属性附加到 React 元素;

---

- Refs 通常在 React 组件的:
  - 构造函数中定义;
- 或者作为 '函数组件' 顶层的变量定义;
  - 然后附加到 render() 函数中的元素;

```js
export default class Hello extends React.Component {
  constructor(props) {
    super(props);
    // 创建 ref 存储 customRef DOM 元素
    this.customRef = React.createRef();
  }
  componentDidMount() {
    // 注意：通过 "current" 取得DOM节点
    // 直接使用原生 API 使 custom 输入框获得焦点
    this.customRef.current.focus();
  }
  render() {
    return <input ref={this.customRef} />;
  }
}
```

- 使用 React.createRef() 给组件创建了 Refs 对象;
- 在上面的示例中，ref 被命名 customRef;
  - 然后将其附加到 <input> DOM 元素;

---

- 其中， customRef 的属性 current 指的是:
  - 当前附加到 ref 的元素;
  - 并广泛用于:
    - 访问和修改
  - 我们的附加元素;
- 事实上，如果我们通过:
  - 登录 myRef 控制台:
    - 进一步扩展我们的示例;
      - 我们将看到该 current 属性:
        - 确实是唯一可用的属性:

```js
componentDidMount = () => {
  // myRef 仅仅有一个 current 属性
  console.log(this.textRef);
  // myRef.current
  console.log(this.textRef.current);
  // component 渲染完成后，使 text 输入框获得焦点
  this.textRef.current.focus();
};
```

- 在 componentDidMount 生命周期阶段:
  - myRef.current 将按预期:
    - 分配给我们的 <input> 元素;
- componentDidMount 通常是:
  - 使用 refs 处理一些:
    - '初始设置' 的 '安全位置';

---

- 我们不能在 componentWillMount 中更新 Refs:
  - 因为此时，组件还没渲染完成;
    - Refs 还为 null;(它要从组件中拿东西)

#### 2、方法二：回调 Refs

- 不同于传递 createRef() 创建的 ref 属性;
  - 你会传递一个函数;
- 这个函数中:
  - 接受 React 组件实例;
  - 或 HTML DOM 元素作为参数;
- 以使它们能:
  - 在 '其他地方' 被 '存储' 和 '访问';

```js
import React from "react";
export default class Hello extends React.Component {
  constructor(props) {
    super(props);
    this.customRef = null; // 创建ref为null
  }
  componentDidMount() {
    // 注意：这里没有使用 "current"
    // 直接使用原生 API 使 text 输入框获得焦点
    this.customRef.focus();
  }
  render() {
    // 把 <input> ref 关联到构造器里创建的 customRef 上
    return <input ref={(node) => (this.customRef = node)} />;
  }
}
```

- React 将在组件挂载时:
  - 将 DOM 元素传入 ref 回调函数并调用;
    - 当卸载时传入 null 并调用它;
- 在 componentDidMount 或 componentDidUpdate 触发前:
  - React 会保证 refs 一定是最新的;

---

- 像上例:
  - ref 回调函数：
    - 是以 '内联函数' 的方式定义的;
    - 在 '更新过程' 中它会被 '执行两次';
      - 第一次传入参数 null;
      - 然后第二次会传入参数 DOM 元素;

---

- 这是因为:
  - 在每次渲染时:
    - 会 '创建一个新的' 函数实例;
      - 所以 React 清空旧的 ref 并且设置新的;
- 我们可以通过:
  - 将 ref 的回调函数定义成:
    - class 的绑定函数的方式;
      - 可以避免上述问题;
- 但是大多数情况下它是无关紧要的;

#### 3、方法三：通过 stringRef 实现

```js
export default class Hello extends React.Component {
  constructor(props) {
    super(props);
  }
  componentDidMount() {
    // 通过 this.refs 调用
    // 直接使用原生 API 使 text 输入框获得焦点
    this.refs.textRef.focus();
  }
  render() {
    // 把 <input> ref 关联到构造器里创建的 textRef 上
    return <input ref="textRef" />;
  }
}
```

- 尽管字符串 stringRef 使用更方便:
  - 但是它有一些缺点;
    - 因此严格模式使用 stringRef 会报警告。
- 官方推荐采用回调 Refs;

### 注意

- 当 ref 属性被用于:
  - 一个普通的 HTML 元素时:
    - React.createRef() 将接收底层 DOM 元素:
      - 作为它的 current 属性以创建 ref;
  - 我们可以通过 Refs 访问 DOM 元素属性;
- 当 ref 属性被用于:
  - 一个自定义 class 组件时:
    - ref 对象将接收:
      - 该组件已挂载的实例:
        - 作为它的 current;
  - 与 ref 用于 HTML 元素不同的是:
    - 我们能够通过 ref 访问:
      - 该组件的 props, state, 方法以及它的整个原型;
- ref 是为了获取某个节点的 '实例':
  - 所以 你不能在函数式组件上使用 ref 属性:
    - 因为它们没有实例;
- 推荐使用 回调形式的 refs:
  - stringRef 将会废弃（严格模式下使用会报警告）;
  - React.createRef() API 是 React v16.3 引入的更新;
- 避免使用 refs 来做任何可以通过 '声明式' 实现来完成的事情;

## createRef 与 Hook useRef

### useRef

- useRef 返回一个:
  - 可变的 ref 对象;
    - 其 .current 属性:
      - 被初始化为传入的值;
- 返回的 ref 对象:
  - 在组件的 '整个生命周期内' 保持不变;

```js
import { useEffect, useRef } from "react";

function Hello() {
  const customRef = useRef(null);
  // 手动获取焦点的方式:
  const onButtonClick = () => {
    // 注意：通过 "current" 取得 DOM节点
    customRef.current.focus();
  };
  // 自动获取焦点的方式:
  useEffect(() => {
    customRef.current.focus();
  });
  return (
    <>
      <input ref={customRef} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}

export default Hello;
```

### 区别

- useRef() 比 ref 属性更有用:
  - useRef() Hook 不仅可以用于 DOM refs;
    - useRef() 创建的 ref 对象:
      - 是一个可变 current 属性;
      - 且可以 '容纳任意值' 的 '通用容器';
        - 类似于一个 class 的实例属性;

```js
function Timer() {
  const intervalRef = useRef();

  useEffect(() => {
    const id = setInterval(() => {
      // ...
    });
    intervalRef.current = id;
    return () => {
      clearInterval(intervalRef.current);
    };
  });

  // ...
}
```

- 这是因为它创建的是一个普通 Javascript 对象;
  - 而 useRef() 和自建一个 {current: ...}对象的唯一区别是:
    - useRef 会在每次渲染时返回 '同一个 ref 对象';

---

- 请记住:
  - 当 'ref 对象' 内容发生变化时:
    - useRef 并不会通知你;
  - 变更 .current 属性 '不会引发' 组件重新渲染;
  - 如果想要在:
    - React 绑定 或 解绑 DOM 节点的 ref 时:
      - 运行某些代码:
        - 则需要使用回调 ref 来实现;

## createRef 源码解析

```js
// ReactCreateRef.js 文件
import type { RefObject } from "shared/ReactTypes";

// an immutable object with a single mutable value
export function createRef(): RefObject {
  const refObject = {
    current: null,
  };
  if (__DEV__) {
    // 封闭对象:
    // 阻止添加新属性:
    // 并将所有现有属性:
    // 标记为不可配置;
    // 当前属性的值只要可写就可以改变;
    Object.seal(refObject);
  }
  return refObject;
}
```

- 其中 RefObject 为:

```js
export type RefObject = {|
  current: any,
|};
```

- 这就是的 createRef 源码，实现很简单，但具体的它如何使用，如何挂载，将在后面的 React 渲染中介绍，敬请期待;

---

- 作者：前端瓶子君
- 链接：https://juejin.im/post/6844903917021560839
- 来源：掘金
- 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
