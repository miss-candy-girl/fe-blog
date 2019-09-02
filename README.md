### react16 新特性介绍 以及 react 实际项目应用和最佳实践

<ver />
### 目录
1. createElement 如何构建虚拟 dom
2. React.Component 如何实现组件化
3. setState 异步队列
4. dom 的 diff 算法
5. render 渲染逻辑
6. React16 fiber 架构
7. React Hooks

<ver />

### React 生命周期

<ver />
目前React 16.8 +的生命周期分为三个阶段,分别是挂载阶段、更新阶段、卸载阶段

```jsx
class ExampleComponent extends React.Component {
  // 构造函数，最先被执行,我们通常在构造函数里初始化state对象或者给自定义方法绑定this
  constructor() {}
  //getDerivedStateFromProps(nextProps, prevState)用于替换 `componentWillReceiveProps` ，该函数会在初始化和 `update` 时被调用
  // 这是个静态方法,当我们接收到新的属性想去修改我们state，可以使用getDerivedStateFromProps
  static getDerivedStateFromProps(nextProps, prevState) {
    // 新的钩子 getDerivedStateFromProps() 更加纯粹, 它做的事情是将新传进来的属性和当前的状态值进行对比, 若不一致则更新当前的状态。
    if (nextProps.riderId !== prevState.riderId) {
      return {
        riderId: nextProps.riderId
      }
    }
    // 返回 null 则表示 state 不用作更新
    return null
  }
  // shouldComponentUpdate(nextProps, nextState),有两个参数nextProps和nextState，表示新的属性和变化之后的state，返回一个布尔值，true表示会触发重新渲染，false表示不会触发重新渲染，默认返回true,我们通常利用此生命周期来优化React程序性能
  shouldComponentUpdate(nextProps, nextState) {
    return nextProps.id !== this.props.id
  }
  // 组件挂载后调用
  // 可以在该函数中进行请求或者订阅
  componentDidMount() {}
  // getSnapshotBeforeUpdate(prevProps, prevState):这个方法在render之后，componentDidUpdate之前调用，有两个参数prevProps和prevState，表示之前的属性和之前的state，这个函数有一个返回值，会作为第三个参数传给componentDidUpdate，如果你不想要返回值，可以返回null，此生命周期必须与componentDidUpdate搭配使用
  getSnapshotBeforeUpdate() {}
  // 组件即将销毁
  // 可以在此处移除订阅，定时器等等
  componentWillUnmount() {}
  // 组件销毁后调用
  componentDidUnMount() {}
  // componentDidUpdate(prevProps, prevState, snapshot):该方法在getSnapshotBeforeUpdate方法之后被调用，有三个参数prevProps，prevState，snapshot，表示之前的props，之前的state，和snapshot。第三个参数是getSnapshotBeforeUpdate返回的,如果触发某些回调函数时需要用到 DOM 元素的状态，则将对比或计算的过程迁移至 getSnapshotBeforeUpdate，然后在 componentDidUpdate 中统一触发回调或更新状态。
  componentDidUpdate() {}
  // 渲染组件函数
  render() {}
  // 以下函数不建议使用
  UNSAFE_componentWillMount() {}
  UNSAFE_componentWillUpdate(nextProps, nextState) {}
  UNSAFE_componentWillReceiveProps(nextProps) {}
}
```

<ver />

React 版本 17 将弃用几个类组件 API 生命周期：`componentWillMount`，`componentWillReceiveProps`和`componentWillUpdate`。

<hor />

### react 事件机制

<ver />
- react 里面绑定事件的方式和在 HTML 中绑定事件类似，使用驼峰式命名指定要绑定的 onClick 属性为组件定义的一个方法{this.handleClick.bind(this)}。
- 由于类的方法默认不会绑定 this，因此在调用的时候如果忘记绑定，this 的值将会是 undefined。 通常如果不是直接调用，应该为方法绑定 this，将事件函数上下文绑定要组件实例上。
<ver />

#### 绑定事件的四种方式

```js
class Button extends React.Component {
  constructor(props) {
    super(props)
    this.handleClick1 = this.handleClick1.bind(this)
  }
  //方式1：在构造函数中使用bind绑定this，官方推荐的绑定方式，也是性能最好的方式
  handleClick1() {
    console.log('this is:', this)
  }
  //方式2：在调用的时候使用bind绑定this
  handleClick2() {
    console.log('this is:', this)
  }
  //方式3：在调用的时候使用箭头函数绑定this
  // 方式2和方式3会有性能影响并且当方法作为属性传递给子组件的时候会引起重渲问题
  handleClick3() {
    console.log('this is:', this)
  }
  //方式4：使用属性初始化器语法绑定this，需要babel转义
  handleClick4 = () => {
    console.log('this is:', this)
  }
  render() {
    return (
      <div>
        <button onClick={this.handleClick1}>Click me</button>
        <button onClick={this.handleClick2.bind(this)}>Click me</button>
        <button onClick={() => this.handleClick3}>Click me</button>
        <button onClick={this.handleClick4}>Click me</button>
      </div>
    )
  }
}
```

<ver />

#### “合成事件”和“原生事件”

React 实现了一个“合成事件”层（`synthetic event system`），这抹平了各个浏览器的事件兼容性问题。所有事件均注册到了元素的最顶层-document 上，“合成事件”会以事件委托（`event delegation`）的方式绑定到组件最上层，并且在组件卸载（`unmount`）的时候自动销毁绑定的事件。

<hor />

### React 组件开发

<ver />

#### React 组件化思想

##### 一个 UI 组件的完整模板

```jsx
import classNames from 'classnames'
class Button extends React.Component {
  //参数传参与校验
  static propTypes = {
    type: PropTypes.oneOf(['success', 'normal']),
    onClick: PropTypes.func
  }
  static defaultProps = {
    type: 'normal'
  }
  handleClick() {}
  render() {
    let { className, type, children, ...other } = this.props
    const classes = classNames(
      className,
      'prefix-button',
      'prefix-button-' + type
    )
    return (
      <span className={classes} {...other} onClick={() => this.handleClick}>
        {children}
      </span>
    )
  }
}
```

<ver />
##### 函数定义组件（Function Component）

纯展示型的，不需要维护 state 和生命周期，则优先使用 `Function Component`

1. 代码更简洁，一看就知道是纯展示型的，没有复杂的业务逻辑
2. 更好的复用性。只要传入相同结构的 props，就能展示相同的界面，不需要考虑副作用。
3. 打包体积小，执行效率高

<ver />

```jsx
import React from 'react'
function MyComponent(props) {
  let { name } = props
  return <h1>Hello, {name}</h1>
  //会被babel转义成 return _react.default.createElement("h1", null, "Hello, ", name);
  // createElement函数对key和ref等特殊的props进行处理，并获取defaultProps对默认props进行赋值，并且对传入的孩子节点进行处理，最终构造成一个ReactElement对象（所谓的虚拟DOM）。
  // ReactDOM.render将生成好的虚拟DOM渲染到指定容器上，其中采用了批处理、事务等机制并且对特定浏览器进行了性能优化，最终转换为真实DOM。
}
```

<ver />
#### ES6 class 定义一个纯组件（PureComponent）

组件需要维护 state 或使用生命周期方法，则优先使用 `PureComponent`

<ver />

```jsx
class MyComponent extends React.Component {
  render() {
    let { name } = this.props
    return <h1>Hello, {name}</h1>
  }
}
```

### PureComponent

React15.3 中新加了一个类 PureComponent，前身是 PureRenderMixin ，和 Component 基本一样，只不过会在 render 之前帮组件自动执行一次 shallowEqual（浅比较），来决定是否更新组件，浅比较类似于浅复制，只会比较第一层。使用 PureComponent 相当于省去了写 shouldComponentUpdate 函数，当组件更新时，如果组件的 props 和 state：

引用和第一层数据都没发生改变， render 方法就不会触发，这是我们需要达到的效果。
虽然第一层数据没变，但引用变了，就会造成虚拟 DOM 计算的浪费。
第一层数据改变，但引用没变，会造成不渲染，所以需要很小心的操作数据。

### 使用不可变数据结构 Immutablejs

Immutable.js 是 Facebook 在 2014 年出的持久性数据结构的库，持久性指的是数据一旦创建，就不能再被更改，任何修改或添加删除操作都会返回一个新的 Immutable 对象。可以让我们更容易的去处理缓存、回退、数据变化检测等问题，简化开发。并且提供了大量的类似原生 JS 的方法，还有 Lazy Operation 的特性，完全的函数式编程。

```jsx
import { Map } from 'immutable'
const map1 = Map({ a: { aa: 1 }, b: 2, c: 3 })
const map2 = map1.set('b', 50)
map1 !== map2 // true
map1.get('b') // 2
map2.get('b') // 50
map1.get('a') === map2.get('a') // true
```

可以看到，修改 map1 的属性返回 map2，他们并不是指向同一存储空间，map1 声明了只有，所有的操作都不会改变它。

ImmutableJS 提供了大量的方法去更新、删除、添加数据，极大的方便了我们操纵数据。除此之外，还提供了原生类型与 ImmutableJS 类型判断与转换方法：

```jsx
import { fromJS, isImmutable } from 'immutable'
const obj = fromJS({
  a: 'test',
  b: [1, 2, 4]
}) // 支持混合类型
isImmutable(obj) // true
obj.size() // 2
const obj1 = obj.toJS() // 转换成原生 `js` 类型
```

`ImmutableJS` 最大的两个特性就是： `immutable data structures`（持久性数据结构）与 `structural sharing`（结构共享），持久性数据结构保证数据一旦创建就不能修改，使用旧数据创建新数据时，旧数据也不会改变，不会像原生 js 那样新数据的操作会影响旧数据。而结构共享是指没有改变的数据共用一个引用，这样既减少了深拷贝的性能消耗，也减少了内存。比如下图：
![react-tree](https://cdn.ru23.com/react_ppt/ppt_react_tree.png)
左边是旧值，右边是新值，我需要改变左边红色节点的值，生成的新值改变了红色节点到根节点路径之间的所有节点，也就是所有青色节点的值，旧值没有任何改变，其他使用它的地方并不会受影响，而超过一大半的蓝色节点还是和旧值共享的。在 ImmutableJS 内部，构造了一种特殊的数据结构，把原生的值结合一系列的私有属性，创建成 ImmutableJS 类型，每次改变值，先会通过私有属性的辅助检测，然后改变对应的需要改变的私有属性和真实值，最后生成一个新的值，中间会有很多的优化，所以性能会很高。
<ver />

#### 高阶组件(higher order component)

高阶组件是一个以组件为参数并返回一个新组件的函数。HOC 运行你重用代码、逻辑和引导抽象。
<ver />

```jsx
function visible(WrappedComponent) {
  return class extends Component {
    render() {
      const { visible, ...props } = this.props
      if (visible === false) return null
      return <WrappedComponent {...props} />
    }
  }
}
```

上面的代码就是一个 HOC 的简单应用，函数接收一个组件作为参数，并返回一个新组件，新组建可以接收一个 visible props，根据 visible 的值来判断是否渲染 Visible。
<ver />
最常见的还有 Redux 的 connect 函数。除了简单分享工具库和简单的组合，HOC 最好的方式是共享 React 组件之间的行为。如果你发现你在不同的地方写了大量代码来做同一件事时，就应该考虑将代码重构为可重用的 HOC。
下面就是一个简化版的 connect 实现：
<ver />

```jsx
export const connect = (
  mapStateToProps,
  mapDispatchToProps
) => WrappedComponent => {
  class Connect extends Component {
    static contextTypes = {
      store: PropTypes.object
    }

    constructor() {
      super()
      this.state = {
        allProps: {}
      }
    }

    componentWillMount() {
      const { store } = this.context
      this._updateProps()
      store.subscribe(() => this._updateProps())
    }

    _updateProps() {
      const { store } = this.context
      let stateProps = mapStateToProps
        ? mapStateToProps(store.getState(), this.props)
        : {}
      let dispatchProps = mapDispatchToProps
        ? mapDispatchToProps(store.dispatch, this.props)
        : {}
      this.setState({
        allProps: {
          ...stateProps,
          ...dispatchProps,
          ...this.props
        }
      })
    }

    render() {
      return <WrappedComponent {...this.state.allProps} />
    }
  }
  return Connect
}
```

代码非常清晰，connect 函数其实就做了一件事，将 `mapStateToProps` 和 `mapDispatchToProps` 分别解构后传给原组件，这样我们在原组件内就可以直接用 `props` 获取 `state` 以及 `dispatch` 函数了。
<ver />

#### 高阶组件的应用

某些页面需要记录用户行为，性能指标等等，通过高阶组件做这些事情可以省去很多重复代码。
<ver />

##### 日志打点

```jsx
function logHoc(WrappedComponent) {
  return class extends Component {
    componentWillMount() {
      this.start = Date.now()
    }
    componentDidMount() {
      this.end = Date.now()
      console.log(
        `${WrappedComponent.dispalyName} 渲染时间：${this.end - this.start} ms`
      )
      console.log(`${user}进入${WrappedComponent.dispalyName}`)
    }
    componentWillUnmount() {
      console.log(`${user}退出${WrappedComponent.dispalyName}`)
    }
    render() {
      return <WrappedComponent {...this.props} />
    }
  }
}
```

<ver />
##### 可用、权限控制

```jsx
function auth(WrappedComponent) {
  return class extends Component {
    render() {
      const { visible, auth, display = null, ...props } = this.props
      if (visible === false || (auth && authList.indexOf(auth) === -1)) {
        return display
      }
      return <WrappedComponent {...props} />
    }
  }
}
```

<ver /> 
##### 表单校验

基于上面的双向绑定的例子，我们再来一个表单验证器，表单验证器可以包含验证函数以及提示信息，当验证不通过时，展示错误信息：

```jsx
function validateHoc(WrappedComponent) {
  return class extends Component {
    constructor(props) {
      super(props)
      this.state = { error: '' }
    }
    onChange = event => {
      const { validator } = this.props
      if (validator && typeof validator.func === 'function') {
        if (!validator.func(event.target.value)) {
          this.setState({ error: validator.msg })
        } else {
          this.setState({ error: '' })
        }
      }
    }
    render() {
      return (
        <div>
          <WrappedComponent onChange={this.onChange} {...this.props} />
          <div>{this.state.error || ''}</div>
        </div>
      )
    }
  }
}
```

<ver />  
```jsx
const validatorName = {
  func: (val) => val && !isNaN(val),
  msg: '请输入数字'
}
const validatorPwd = {
  func: (val) => val && val.length > 6,
  msg: '密码必须大于6位'
}
<HOCInput validator={validatorName} v_model="name"></HOCInput>
<HOCInput validator={validatorPwd} v_model="pwd"></HOCInput>
```

<ver />

#### HOC 的缺陷

- HOC 需要在原组件上进行包裹或者嵌套，如果大量使用 HOC，将会产生非常多的嵌套，这让调试变得非常困难。
- HOC 可以劫持 props，在不遵守约定的情况下也可能造成冲突。

<hor />
### `setState` 数据管理
<ver />
**不要直接更新状态**

```js
// Wrong 此代码不会重新渲染组件,构造函数是唯一能够初始化 this.state 的地方。
this.state.comment = 'Hello'
// Correct 应当使用 setState():
this.setState({ comment: 'Hello' })
```

<ver />
组件生命周期中或者 React 事件绑定中，setState 是通过异步更新的，在延时的回调或者原生事件绑定的回调中调用 setState 不一定是异步的。

- 多个 setState() 调用合并成一个调用来提高性能。
- this.props 和 this.state 可能是异步更新的，不应该依靠它们的值来计算下一个状态。

```js
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment
})
// Correct
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}))
```

<ver />
原生事件绑定不会通过合成事件的方式处理，会进入更新事务的处理流程。`setTimeout` 也一样，在 `setTimeout` 回调执行时已经完成了原更新组件流程，不会放入 `dirtyComponent` 进行异步更新，其结果自然是同步的。
<ver />

#### `setState` 原理

待完善

### `React` 中的事务实现

待完善

<hor />

### ErrorBoundary、Suspense 和 Fragment

#### Error Boundaries

React 16 提供了一个新的错误捕获钩子 `componentDidCatch(error, errorInfo)`, 它能将子组件生命周期里所抛出的错误捕获, 防止页面全局崩溃。demo
componentDidCatch 并不会捕获以下几种错误

- 事件机制抛出的错误(事件里的错误并不会影响渲染)
- Error Boundaries 自身抛出的错误
- 异步产生的错误
- 服务端渲染

#### Suspense

Suspense 意思是能暂停当前组件的渲染, 当完成某件事以后再继续渲染。

`code splitting`(16.6, 已上线): 文件懒加载。在此之前的实现方式是 react-loadable;
`Concurrent mode`(2019 年 Q1 季度): 并发模式;
`data fetching`(2019 年中): 可以控制等所有数据都加载完再呈现出数据; `Suspense` 提供一个时间参数, 若小于这个值则不进行 loading 加载, 若超过这个值则进行 loading 加载;

```jsx
import React, { lazy, Suspense } from 'react'
const OtherComponent = lazy(() => import('./OtherComponent'))
function MyComponent() {
  return (
    <Suspense fallback={<div>loading...</div>}>
      <OtherComponent />
    </Suspense>
  )
}
```

一种简单的预加载思路, 可参考 preload

```jsx
const OtherComponentPromise = import('./OtherComponent')
const OtherComponent = React.lazy(() => OtherComponentPromise)
```

<hor />

### React hooks

在 React 16.7 之前, React 有两种形式的组件, 有状态组件(类)和无状态组件(函数)。Hook 是 React 16.8 的新增特性。它可以让你在不编写 `class` 的情况下使用 state 以及其他的 React 特性。Hooks 的意义就是赋能先前的无状态组件, 让之变为有状态。这样一来更加契合了 React 所推崇的函数式编程。React Hooks 如今可谓前端界"当红小生", 因其 API 简洁性、逻辑复用性等特性逐渐被开发者所应用，未来的 vue3.0 也是采用类似的 Function Based 的模式，因此学习 React Hooks 也是未来的大趋势，通过一个具体的项目来实践、应用 hooks 特性，我觉得比干啃文档要强太多，并且在实践的过程中会遇到一些坑，通过坑驱动来学习，可以加深我们对于 hooks 原理的理解。

接下来梳理 Hooks 中最核心的 2 个 api, `useState` 和 `useEffect`

#### useState

useState 是一个钩子，他可以为函数式组件增加一些状态，并且提供改变这些状态的函数，同时它接收一个参数，这个参数作为状态的默认值。

```jsx
const [count, setCount] = useState(initialState)
```

使用 Hooks 相比之前用 class 的写法最直观的感受是更为简洁

```jsx
function App() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  )
}
```

#### useEffect(fn)

在每次 render 后都会执行这个钩子。可以将它当成是 componentDidMount、componentDidUpdate、componentWillUnmount 的合集。因此使用 useEffect 比之前优越的地方在于:

可以避免在 componentDidMount、componentDidUpdate 书写重复的代码;
可以将关联逻辑写进一个 useEffect(在以前得写进不同生命周期里);

#### 怎么使用，会对之前的 React 开发造成什么冲击

<hor />

### React 中的高阶组件和 render props

<hor />

### React Fiber 架构分析

#### React Fiber 架构解决了什么问题

<hor />

### 深入理解 React 原理

#### 实现 React 核心 api

#### React 虚拟 dom 原理剖析

##### React 组件的渲染流程

使用 `React.createElement` 或 JSX 编写 React 组件，实际上所有的 JSX 代码最后都会转换成 `React.createElement(...)`，Babel 帮助我们完成了这个转换的过程。

createElement 函数对 key 和 ref 等特殊的 props 进行处理，并获取 defaultProps 对默认 props 进行赋值，并且对传入的孩子节点进行处理，最终构造成一个 ReactElement 对象（所谓的虚拟 DOM）。

ReactDOM.render 将生成好的虚拟 DOM 渲染到指定容器上，其中采用了批处理、事务等机制并且对特定浏览器进行了性能优化，最终转换为真实 DOM。

##### 虚拟 DOM 的组成

即 `ReactElementelement` 对象，我们的组件最终会被渲染成下面的结构：

`type`：元素的类型，可以是原生 html 类型（字符串），或者自定义组件（函数或 class）
`key`：组件的唯一标识，用于 Diff 算法，下面会详细介绍
`ref`：用于访问原生 dom 节点
`props`：传入组件的 props，chidren 是 props 中的一个属性，它存储了当前组件的孩子节点，可以是数组（多个孩子节点）或对象（只有一个孩子节点）
`owner`：当前正在构建的 Component 所属的 Component
`self`：（非生产环境）指定当前位于哪个组件实例
`_source`：（非生产环境）指定调试代码来自的文件(fileName)和代码行数(lineNumber)

当组件状态 state 有更改的时候，React 会自动调用组件的 render 方法重新渲染整个组件的 UI。
当然如果真的这样大面积的操作 DOM，性能会是一个很大的问题，所以 React 实现了一个 Virtual DOM，组件 DOM 结构就是映射到这个 Virtual DOM 上，React 在这个 Virtual DOM 上实现了一个 diff 算法，当要重新渲染组件的时候，会通过 diff 寻找到要变更的 DOM 节点，再把这个修改更新到浏览器实际的 DOM 节点上，所以实际上不是真的渲染整个 DOM 树。这个 Virtual DOM 是一个纯粹的 JS 数据结构，所以性能会比原生 DOM 快很多。

##### 防止 XSS

ReactElement 对象还有一个$$typeof属性，它是一个Symbol类型的变量Symbol.for('react.element')，当环境不支持Symbol时，$$typeof 被赋值为 0xeac7。
这个变量可以防止 XSS。如果你的服务器有一个漏洞，允许用户存储任意 JSON 对象， 而客户端代码需要一个字符串，这可能为你的应用程序带来风险。JSON 中不能存储 Symbol 类型的变量，而 React 渲染时会把没有\$\$typeof 标识的组件过滤掉。

<hor />

### React 和 Vue 特点比较

<hor />

### redux

#### redux 单向数据流架构如何设计

#### redux 中间件

<hor />

### React 性能分析与优化

#### 在 React-Router4 中使用懒加载性能优化

### 参考：

1. [深入分析虚拟 DOM 的渲染原理和特性](https://juejin.im/post/5cb66fdaf265da0384128445)
2. [React 事件机制](http://www.conardli.top/blog/article/React%E6%B7%B1%E5%85%A5%E7%B3%BB%E5%88%97/React%E4%BA%8B%E4%BB%B6%E6%9C%BA%E5%88%B6.html)
3. [从 Mixin 到 HOC 再到 Hook](https://juejin.im/post/5cad39b3f265da03502b1c0a)
