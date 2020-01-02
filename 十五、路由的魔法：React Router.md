# 十五、路由的魔法：React Router

单页应用名为单页应用，其实在设计概念上依然是多页的界面，只不过从技术层面上页之间的切换是没有整体网页刷新的，只需要做局部更新。

要实现“单页应用”，一个最要紧的问题就是做好“路由”（Routing)，也就是处理好下面两件事：

- 把 URL 映射到对应的页面来处理
- 页面之间切换做到只需局部更新

### 一、react router v4 的动态路由：

动态路由：所谓动态路由，指的是`路由规则不是预先确定的，而是在渲染过程中确定的。`因为 react-router 的定位就是专供 React 应用服务，而 React 的世界中一切皆为组件，所以 react-router v4 就完全用 React 组件来实现路由功能。

通过一个很简单的例子来说明 react-router v4 如何工作的，然后在这个例子的基础上介绍“动态路由”。

### 二、React Router 实例：

#### 1.安装包 react-router-dom

 🌵react-router 为了支持 Web 和 React Native 出了两个包:

- Web : react-router-dom

- React Native: react-router-native

- 所以只需要安装 react-router-dom 。这个 react-router-dom 依赖于 react-router ，所以 react-router 也会被自动安装上。

  - ```react
    npm install react-router-dom
    ```

    

### 三、HashRouter 还是 BrowserRouter：

🏖react-router 的工作方式：

- 组件树顶层放一个 Router 组件；
- 在组件树中散落着很多 Route 组件（注意比 Router 少一个“r”）
- 顶层的 Router 组件负责分析监听 URL 的变化，在它保护伞之下的 Route 组件可以直接读取这些信息。

`很明显，Router 和 Route 的配合，就是之前我们介绍过的“提供者模式”，Router 是“提供者”，Route是“消费者”。`

更进一步，Router 其实也是一层抽象,让下面的 Route 无需各种不同 URL 设计的细节，不要以为 URL 就一种设计方法，至少可以分为两种。

🐳URL的两种设计方法：

-  BrowserRouter支持的URL：
  - 比如 `/` 对应 Home 页，`/about` 对应 About 页，但是这样的设计需要服务器端渲染，因为用户可能直接访问任何一个 URL，**服务器端必须能对 `/`的访问返回 HTML**，也要对 `/about` 的访问返回 HTML。
- HashRouter 支持的URL：
  - 看起来不自然，但是实现更简单。只有一个路径 `/`，通过 URL 后面的 `#` 部分来决定路由，`/#/` 对应 Home 页，`/#/about` 对应 About 页。因为 URL 中`#`之后的部分是不会发送给服务器的，所以，无论哪个 URL，最后都是访问服务器的 `/` 路径，服务器也只需要返回同样一份 HTML 就可以，然后由浏览器端解析 `#` 后的部分，完成浏览器端渲染。

**`因为 create-react-app 产生的应用默认不支持服务器端渲染`，为了简单起见，我们在下面的例子中使用 HashRouter，在实际产品中，其实最好还是用 BrowserRouter，这样用户体验更好。**

举个例子🌰：index.js文件代码如下：

```react
import {HashRouter} from 'react-router-dom';
ReactDOM.render(
  <HashRouter>
    <App />
  </HashRouter>,
  document.getElementById('root')
);
```

`把 Router 用在 React 组件树的最顶层，这是最佳实践。因为将来我们如果想把 HashRouter 换成 BrowserRouter，组件 App 以下几乎不用任何改变。`



### 四、使用Link

对于单页应用，需要在不同“页面”之间切换，往往需要一个“导航栏”，我们在这里也实现一个简单的导航栏。

在`App.js`中，我们让网页由两个组件 `Navigation` 和 `Content` 组成， `Navigation 就是导航栏`，而 `Content 是具体内容`。

```react
class App extends Component {
  render() {
    return (
      <div className="App">
        <Navigation />
        <Content />
      </div>
    );
  }
}
```

计划只增加两个页面，在 Navigation 中就应该有两个链接，但是，如果我们简单使用 HTML 的 `<a>` 标签那就错了,用户点击 `<a>` 标签是网页跳转,这违背了“单页应用”的原则。

虽然对于 HashRouter 使用的是没有网页跳转的 `#`，但是为了将来可以无缝切换为 BrowserRouter ，我们也不能使用 `[`](#about) 这样的标签。

**`正确的解法`**:使用react-router 提供的 Link 组件；

-  Link 最终还是渲染为 `<a>` 标签
- 但是这个`<a>`标签让react-router知晓这是一个`单页应用的链接`，不用网页跳转只做局部页面更新。

Navigation组件：

```react
const ulStyle = {
  'list-style-type': 'none',
  margin: 0,
  padding: 0,
};

const liStyle = {
  display: 'inline-block',
  width: '60px',
};

const Navigation = () => (
  <header>
    <nav>
      <ul style={ulStyle}>
        <li style={liStyle}><Link to='/'>Home</Link></li>
        <li style={liStyle}><Link to='/about'>About</Link></li>
      </ul>
    </nav>
  </header>
)
```



### 五、使用Route和switch

我们来看 Content 这个组件，这里会用到 react-router 最常用的两个组件 Route 和 Switch。

```react
const Content = () => (
  <main>
    <Switch>
      <Route exact path='/' component={Home}/>
      <Route path='/about' component={About}/>
    </Switch>
  </main>
)
```

`这里需要注意exact精确匹配的使用`;或者使用Switch组件

可以把 Switch 组件看做是 JavaScript 的 switch 语句，像这样：

```react
switch (条件) {
  case 1: 渲染1; break;
  case 2: 渲染2; break;
}
```

从上往下找第一个匹配的 Route，匹配中了之后，立刻就 break，不继续这个 Switch 下其他的 Route 匹配了。



### 六、动态路由

假设，我们增加一个新的页面叫 `Product`，对应路径为 `/product`，但是只有用户登录了之后才显示。

如果用静态路由，我们在渲染之前就确定这条路由规则，这样即使用户没有登录，也可以访问 `product`，我们还不得不在 Product 组件中做用户是否登录的检查。

如果用动态路由，则只需要在代码中的一处涉及这个逻辑：

```react
    <Switch>
      <Route exact path='/' component={Home}/>
      {
        isUserLogin() &&
        <Route exact path='/product' component={Product}/>,
      }  
      <Route path='/about' component={About}/>
    </Switch>
```

