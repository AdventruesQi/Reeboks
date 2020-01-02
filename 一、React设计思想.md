# 一、React设计思想：

### 1.react基础原则：

- React界面完全由数据驱动
- React中一切都是组件
- props 是 React 组件之间通讯的基本方式

React 的哲学，简单说来可以用下面这条公式来表示：

-   UI = f(data)
  - 左边的 UI 代表最终画出来的界面
  - 等号右边的 f 是一个函数，也就是我们写的 React 相关代码；
  - 等号右边的 f 是一个函数，也就是我们写的 React 相关代码；

这个公式的含义就是，如果要渲染界面，不要直接去操纵 DOM 元素，而是修改数据，由数据去驱动 React 来修改界面。

这样一种程序结构，是声明式编程（Declarative Programming）的方式，代码结构会更加容易理解和维护。



### 2.组件：React 世界的一等公民：

关于React中组件：

- 1.用户界面就是组件；
- 2.组件可以嵌套包装组成复杂功能；
- 3.组件可以用来实现副作用。



这里就详细的说明下第三点🏖：

<u>组件可以用来实现副作用。并不是说组件必须要在界面画一些东西，一个组件可以什么都不画，或者把画界面的事情交给其他组件去做，自己做一些和界面无关的事情，比如获取数据。</u>

```javascript

class Beacon extends React.Component {
  render() {
    return null;
  }
  
  componentDidMount() {
    const beacon = new Image();
    beacon.src = 'https://domain.name/beacon.gif';
  }
}

```

不过，Beacon 的 componentDidMount 函数中创造了一个 `Image` 对象，访问了一个特定的图片资源，这样就可以对应服务器上留下日志记录，用于记录这一次网页访问。



### 3.组件之间的语言：props:

来自“父亲的叮咛”🤺：

如果一个父组件有话要对子组件说，也就是，想要传递数据给子组件，则应该通过 props。



来自“儿子的回馈”⛹🏿‍♂️：

如果子组件有话要同父组件说，那应该支持函数类型的 props。身为 JavaScript 里一等公民的函数可以作为参数传递，当然也可以作为 props 传递。让父组件传递一个函数类型的 props 进来，当子组件要传递数据给父组件时，调用这个函数类型 props，就把信息传递给了父组件。

举个🌰：

```javascript
    
 //父组件的代码🧵：
     render() {
        var name = "<span style='color: #FF0000;'>张三</span>";
        return (
            <div>
                {/*父组件接收子组件的值*/}
                <EventComponet sendFatherValue={this.getChildValue.bind(this)}/>
                <div>从父组件接收过来的值:{this.state.childValue}</div>
            </div>
        )
    }

 //获取从子组件传递过来的值
    getChildValue(val) {
        console.log("getChildValue", val);
        this.setState({
            childValue:val
        })
    }


//子组件的代码🧩：

render() {
        return (
            <div>
                {/*当input的值改变时将子组件的值传给父组件，toFatherValue是父组件一个属性，用来接口子组件的值*/}
                <input onChange={(e) => {
                    this.props.sendFatherValue( e.target.value);
                }}/>
                {/*点击按钮是给父组件传值*/}
                <button type="button" onClick={this.props.sendFatherValue.bind(this, "我是子组件EventComponet的值")}>向父组件传值</button>
            </div>
        )
    }

```







