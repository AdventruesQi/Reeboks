# 十一、React 状态管理（1）：组件状态

### 引子：

```react
UI = f(data)
```

f 的参数 data，除了 props，就是 state。

- props 是组件外传递进来的数据;
- state 代表的就是 React 组件的内部状态;



### 1.为什么要了解 React 组件自身状态管理：

🏖 React 开发社区中 Redux 和 Mobx 这样的状态管理工具，不过还是要先从了解React组件自身的管理开始：为什么呢？下面就是原因

- 因为 React 组件自身的状态管理是基础，其他第三方工具都是在这个基础上构筑的，连基础都不了解，无法真正理解第三方工具。
- 对于很多应用场景，React 组件自身的状态管理就足够解决问题，犯不上动用 Redux 和 MobX 这样的大杀器，简单问题简单处理，可以让代码更容易维护。



### 2.组件自身状态 state：

#### ①什么数据放到state中？

React组件的数据分为两种：

- state
- props

一个经常被问到的问题，就是为什么不把组件的数据直接存放在组件类的成员变量中？比如像下面这样：

```react
class Foo extends React.Component {
  
  foo = 'foo'      //以组件类的成员变量

  render() {
    return (
      <React.Fragment>{this.foo}</React.Fragment>
    );
  }
}
```

像上面，数据存在 `this.foo` 中，而不是存在 `this.state.foo` 中，当这个组件渲染的时候，当然 `this.foo` 的值也就被渲染出来了，问题是，更新 `this.foo` 并不会引发组件的重新渲染，这很可能不是想要的结果。

🌵判断一个数据应该放在哪里，用下面的原则：

- 如果数据由外部传入，放在 props 中；

- 如果是组件内部状态，是否这个状态更改应该立刻引发一次组件重新渲染？

  - 如果是，放在 state 中；

  - 不是，放在成员变量中；就像上面的foo=“foo”

    

### 3. state 的正确方式：

`this.state` 本身就是一个对象，但是修改状态不应该通过直接修改 `this.state` 对象来完成。

🏝`因为，我们修改 state，当然不只是想修改这个对象的值，而是想引发 React 组件的重新渲染。`

```react
this.state.foo = 'bar'; //错误的方式

this.setState({foo:'bar'}); //正确的方式
```

- 使用this.state直接修改，那也仅仅是修改了this.state这个对象👇;

- 我们所要期望的是，修改了对象后还要引发组件的重新渲染👇；

- 重新渲染后用的就是修改后的state状态👇

- 达到根据 `state` 改变公式左侧 UI 的目的

  ```react
  UI = f(state)
  ```

  

### 4.state 改变引发重新渲染的时机:

🎏`setState` 函数来修改组件 state，而且可以引发组件重新渲染,但是有趣的是，**并不是一次 `setState` 调用肯定会引发一次重新渲染。**

这是 React 的一种性能优化策略，如果 React 对每一次 `setState` 都立刻做一次组件重新渲染，那代价有点大，比如下面的代码：

```react
this.setState({count: 1});
this.setState({caption: 'foo'});
this.setState({count: 2});
```

连续的同步调用 `setState`，第三次还覆盖了第一次调用的效果，但是效果只相当于调用了下面这样一次：

```react
this.setState({count: 2, caption: 'foo'});
```

虽然不会故意连续写三个 setState 调用，但是代码一旦写得复杂，可能有多个 setState 分布在一次执行的不同代码片段中，还是会同步连续调用 setState，这时候，如果真的每个 setState 都引发一次重新渲染，实在太浪费了。



🍀React利用`任务队列`解决了这个问题，可以理解为：

- 每次 setState 函数调用都会往 React 的任务队列里放一个任务👇；
- 多次 setState 调用自然会往队列里放多个任务👇；
- React 会选择时机去批量处理队列里执行任务👇；
- 当批量处理开始时，React 会合并多个 setState 的操作；
  - 比如上面的三个 setState 就被合并为只更新 state 一次，也只引发一次重新渲染。

`因为这个任务队列的存在，React 并不会同步更新 state，所以，在 React 中，setState 也不保证同步更新 state 中的数据。`



### 5.setState 的第二个参数:

setState 的第二个参数:

- 可以是一个回调函数，当 state 真的被修改时，这个回调函数会被调用

- ```react
    console.log(this.state.count); // 0
    this.setState({count: 1}, () => {
      console.log(this.state.count); // 这里就是1了
    })
    console.log(this.state.count); // 依然为0
  ```

  🌼 **当 setState 的第二个参数被调用时，React 已经处理完了任务列表**，所以 this.state 就是更新后的数据。

  `如果需要在 state 更新之后做点什么，请利用第二个参数。`



### 6.函数式 setState：

setState 不能同步更新的确会带来一些麻烦，尤其是多个 setState 调用之间有依赖关系的时候，很容易写错代码。

一个很典型的例子🌰，当我们不断增加一个 state 的值时：

```react
  this.setState({count: this.state.count + 1});
  this.setState({count: this.state.count + 1});
  this.setState({count: this.state.count + 1});
```

上面的这个例子表面上看会让this.state.count的值增加3，实际上只增加了1；

**setState 没有同步更新 this.state 啊，所以给任务队列加的三个任务都是给 this.state.count 同一个值而已**

面对这种情况，我们很自然地想到，如果任务列表中的任务不只是给 state 一个固定数据，如果任务列表里的“任务”是一个函数，能够根据当前 state 计算新的状态，那该多好！



#### **`实际上，setState第一个参数常用的是对象，其实也可以是一个函数`**

🌵当setState的第一个参数是函数时：

- 任务列表上增加的就是一个可执行的任务函数了
- React 每处理完一个任务，都会更新 this.state，然后把**新的 state** 传递给这个任务函数

setState 第一个参数的形式如下：

```react
function increment(state, props) {
  return {count: state.count + 1};
}
```

**🐡这是一个纯函数，不光接受当前的 state，还接受组件的 props，在这个函数中可以根据 state 和 props 任意计算`返回的结果会用于修改 this.state。`**

如此一来，我们就可以这样连续调用 setState：

```react
	this.setState(increment);
  this.setState(increment);
  this.setState(increment);
```

这种函数式方式连续调用 setState，就真的能够让 this.state.count 增加 3，而不只是增加 1。



🌵 这里接续说下setState的第一个参数是函数和第二个参数是函数的区别：

- 第一个参数是函数：是react任务队列，每次处理完一个任务，都会更新this.state
- 第二个参数是函数：在react已经处理完任务队列列表了，this.state是最终更新后的数据。

