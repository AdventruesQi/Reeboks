# 七、组件设计模式（3）：render props 模式

### 1.render props 模式:

- `让 React 组件的 props 支持函数这种模式`;因为作为 props 传入的函数往往被用来渲染一部分界面，所以这种模式被称为 render props。

🍪最简单的 render props 组件 `RenderAll`

```react
const RenderAll = (props) => {
  return(
     <React.Fragment>
     	{props.children(props)}
     </React.Fragment>
  );
};
```

🍰上述代码完成的功能：

- RenderAll预想子组件是一个函数；

  - 把子组件当做函数进行调用，调用的参数就是传入的props

  - 然后把返回结果渲染出来

    

🌵上述代码的使用方法：

```react
    <RenderAll>
        {() => <h1>hello world</h1>}
     </RenderAll>
```

可以看到，RenderAll 的子组件，也就是夹在 RenderAll 标签之间的部分，其实是一个函数。这个函数渲染出 `hello world`，这就是上面使用 RenderAll 渲染出来的结果。



接下来就是来认识下render props的真正强大之处吧🐳：

### 2.传递props：

以根据是否登录状态来显示一些界面元素为例，来实现一个 render props：`Login`组件

```react

const Login = (props) => {
  
  const userName = getUserName();   //判断用户是否登录的信息，以便下面来进行判断使用
  
  if (userName) {
    
    const allProps = {userName, ...props};   //添加新的属性
    
    return (
      <React.Fragment>
        {props.children(allProps)}                 //对被Login包裹的组件进行属性添加
      </React.Fragment>
    );
  } else {
    return null;
  }
};
```

当然，render props 完全可以决定哪些 props 可以传递给 props.children，在 Login 中，我们把 `userName` 作为增加的 props 传递给下去，这样就是 Login 的增强功能。

一个使用上面 Login 的 JSX 代码示例如下：

```react
 <Login>
    {({userName}) => <h1>Hello {userName}</h1>}
  </Login>
```

- 首先要清楚的一个事情就是props.children指的是被当前这个组件包裹的内部所有的东东；
- 所以呢，这里面指的props.children指的就是花括号"{}"里面的函数，`userName实际上就是传递过来的allProps中的userName，这里是通过es6解构赋值`



### 3.不限于children：

在上面的例子中，就是利用 `children` 这个 props 来作为函数传递。

实际上，render props 这个模式不必局限于 children 这一个 props，任何一个 props 都可以作为函数，也可以利用多个 props 来作为函数。

来扩展 Login，拓展后的组件叫做Auth：

```react
const Auth= (props) => {
  
  const userName = getUserName();        //确认用户是否登录
  
  if (userName) {
    const allProps = {userName, ...props};
    return (
      <React.Fragment>
        {props.login(allProps)}
      </React.Fragment>
    );
  } else {
    <React.Fragment>
      {props.nologin(props)}
    </React.Fragment>
  }
};
```

然后是Auth组件的使用过程：

```react
  <Auth
    login={({userName}) => <h1>Hello {userName}</h1>}
    nologin={() => <h1>Please login</h1>}
  />
```

- 分别通过 `login` 和 `nologin` 两个 props 来指定用户登录或者没登录时显示什么



### 4.依赖注入：

`render props 其实就是 React 世界中的“依赖注入”`

所谓依赖注入，指的是解决这样一个问题：

- 逻辑 A 依赖于逻辑 B，如果让 A 直接依赖于 B，当然可行，但是 A 就没法做到通用了。

- 依赖注入就是`把 B 的逻辑以函数形式传递给 A`，`A 和 B 之间只需要对这个函数接口达成一致就行`，如此一来，再来一个逻辑 C，也可以用一样的方法重用逻辑 A。



### 5.render props 和高阶组件的比较：

比对一下这两种重用 React 组件逻辑的模式：

- 1.组件和函数：
  - render props 模式的应用，就是做一个 React 组件
  - 高阶组件，虽然名为“组件”，其实只是一个产生 React 组件的函数
- 2.“老铁没毛病”：
  - render props 不像上一小节中介绍的高阶组件有那么多毛病，如果说 render props 有什么缺点，那就是 render props 不能像高阶组件那样链式调用，当然，这并不是一个致命缺点。



🌵🌻🌵在需要React组件重用逻辑时的思考：

- 当需要重用 React 组件的逻辑时，建议首先看这个功能是否可以抽象为一个简单的组件
  - 如果行不通的话，考虑是否可以应用 render props 模式
    - 如果行不通的话，考虑是否可以应用 render props 模式

