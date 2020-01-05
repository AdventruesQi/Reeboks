# 二十、React 的未来（3）：函数化的 Hooks

**Hooks 的目的，简而言之就是让开发者不需要再用 class 来实现组件**。



### 一、useState:

Hooks 会提供一个叫 `useState` 的方法，它开启了一扇新的定义 state 的门，对应 Counter 的代码可以这么写：

```react
import { useState } from 'react';

const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
       <div>{count}</div>
       <button onClick={() => setCount(count + 1)}>+</button>
       <button onClick={() => setCount(count - 1)}>-</button>
    </div>
  );
};
```

注意看，Counter 拥有自己的“状态”，但它只是一个函数，不是 class。

`useState` 只接受一个参数，也就是 state 的初始值，它返回一个只有两个元素的数组，第一个元素就是 state 的值，第二个元素是更新 state 的函数。

我们利用解构赋值（destructuring assignment）把两个元素分别赋值给 count 和 setCount，相当于这样的代码：

```react
🌻// 下面代码等同于： const [count, setCount] = useState(0);
  const result = useState(0);
  const count = result[0];
  const setCount = result[1];
```

利用 `count` 可以读取到这个 state，利用 `setCount` 可以更新这个 state，而且我们完全可以控制这两个变量的命名，只要高兴，你完全可以这么写:

```react
 const [theCount, updateCount] = useState(0);
```

🌵 因为 `useState` 在 `Counter` 这个函数体中，每次 Counter 被渲染的时候，这个 `useState` 调用都会被执行，`useState` 自己肯定不是一个纯函数，因为它要区分第一次调用（组件被 mount 时）和后续调用（重复渲染时），只有第一次才用得上参数的初始值，而后续的调用就返回“记住”的 state 值。



🍊Q:看到这里，心里可能会有这样的疑问：如果组件中多次使用 `useState` 怎么办？React 如何“记住”哪个状态对应哪个变量？

React 是完全根据 `useState` 的调用顺序来“记住”状态归属的，假设组件代码如下：

```react
const Counter = () => {
  const [count, setCount] = useState(0);
  const [foo, updateFoo] = useState('foo'); 
  ...
}
```

🌵 每一次 Counter 被渲染都是:

- 第一次 `useState` 调用获得 `count` 和 `setCount`
- 第二次 `useState` 调用获得 `foo` 和 `updateFoo`

React 是渲染过程中的“上帝”，每一次渲染 Counter 都要由 React 发起，所以它有机会准备好一个内存记录，当开始执行的时候，每一次 useState 调用对应内存记录上一个位置，而且是按照顺序来记录的。

React 不知道你把 `useState` 等 Hooks API 返回的结果赋值给什么变量，但是它也不需要知道，它只需要按照 `useState` 调用顺序记录就好了。



**React 是渲染过程中的“上帝”，每一次渲染 Counter 都要由 React 发起，所以它有机会准备好一个内存记录，当开始执行的时候，每一次 useState 调用对应内存记录上一个位置，而且是按照顺序来记录的。**

React 不知道你把 `useState` 等 Hooks API 返回的结果赋值给什么变量，但是它也不需要知道，它只需要按照 `useState` 调用顺序记录就好了。

<font color=blue>正因为这个原因，**Hooks，千万不要在 if 语句或者 for 循环语句中使用！**</font>

🌵像下面的代码，肯定会出乱子的：

```react
const Counter = () => {
    const [count, setCount] = useState(0);
    if (count % 2 === 0) {
        const [foo, updateFoo] = useState('foo');
    }
    const [bar, updateBar] = useState('bar');
  ...
}
```

**因为条件判断，让每次渲染中 `useState` 的调用次序不一致了，于是 React 就错乱了。**



### 二、useEffect

在 React 组件生命周期中如果要做有副作用的操作，代码放在哪里？

如果是想放在 componentDidMount 或者 componentDidUpdate 里，但是这意味着组件必须是一个 class。

React 还提供 `useEffect`，用于支持组件中增加副作用的支持。

有了 `useEffect`，我们就不用写一个 class 了，对应代码如下：

```react
import { useState, useEffect } from 'react';

const Counter = () => {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    document.title = `Count: ${count}`;
  });

  return (
    <div>
       <div>{count}</div>
       <button onClick={() => setCount(count + 1)}>+</button>
       <button onClick={() => setCount(count - 1)}>-</button>
    </div>
  );
};
```

`🍀 useEffect` 的参数是一个函数，组件每次渲染之后，都会调用这个函数参数，这样就达到了 componentDidMount 和 componentDidUpdate 一样的效果。



🍑 Q：可能会有疑问，现在把 componentDidMount 和 componentDidUpdate 混在了一起，那假如某个场景下我只在 mount 时做事但 update 不做事，用 useEffect 不就不行了吗？

A:其实，用一点小技巧就可以解决。`useEffect` 还支持第二个可选参数，只有同一 `useEffect` 的两次调用第二个参数不同时，第一个函数参数才会被调用，所以，如果想模拟 `componentDidMount`，只需要这样写：

```react
  useEffect(() => {
    // 这里只有mount时才被调用，相当于componentDidMount
  }, [123]);
```

在上面的代码中，`useEffect` 的第二个参数是 `[123]`，其实也可以是任何一个常数，因为它永远不变，所以 `useEffect` 只在 mount 时调用第一个函数参数一次，达到了 componentDidMount 一样的效果。



### 三、useContext

比如，一段 JSX 如果既依赖于 ThemeContext 又依赖于 LanguageContext，那么按照 React Context API 应该这么写：

```react
<ThemeContext.Consumer>
    {
        theme => (
            <LanguageContext.Cosumer>
                language => {
                    //可以使用theme和lanugage了
                }
            </LanguageContext.Cosumer>
        )
    }
</ThemeContext.Consumer>
```

因为 Context API 要用 render props，所以用两个 Context 就要用两次 render props，也就用了两个函数嵌套，这样的缩格看起来也的确过分了一点点。

使用 Hooks 的 `useContext`，上面的代码可以缩略为下面这样：

```react
const theme = useContext(ThemeContext);
const language = useContext(LanguageContext);
// 这里就可以用theme和language了
```

这个`useContext`把一个需要很费劲才能理解的 Context API 使用大大简化，不需要理解render props，直接一个函数调用就搞定。

但是，`useContext`也并不是完美的，它会造成意想不到的重新渲染，我们看一个完整的使用`useContext`的组件。

```react
const ThemedPage = () => {
    const theme = useContext(ThemeContext);
    
    return (
       <div>
            <Header color={theme.color} />
            <Content color={theme.color}/>
            <Footer color={theme.color}/>
       </div>
    );
};
```

因为这个组件`ThemedPage`使用了`useContext`，它很自然成为了Context的一个消费者，所以，只要Context的值发生了变化，`ThemedPage`就会被重新渲染，这很自然，因为不重新渲染也就没办法重新获得`theme`值，但现在有一个大问题，对于ThemedPage来说，实际上只依赖于`theme`中的`color`属性，如果只是`theme`中的`size`发生了变化但是`color`属性没有变化，`ThemedPage`依然会被重新渲染，当然，我们通过给`Header`、`Content`和`Footer`这些组件添加`shouldComponentUpdate`实现可以减少没有必要的重新渲染，但是上一层的`ThemedPage`中的JSX重新渲染是躲不过去了。

说到底，`useContext`需要一种表达方式告诉React：“我没有改变，重用上次内容好了。”



