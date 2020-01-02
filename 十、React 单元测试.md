# 十、React 单元测试

### 1.测试的目的：

简单来说，测试就是尽力发现软件中的缺陷（俗称 bug），当我们发现不了更多的 bug 时，说明这个软件质量可以接受了。



### 2.单元测试：

简单的介绍三个React组件的单元测试，三个要点：

- #### 🌵用Jest

  - 不使用Jest出现的问题：

    - **单元测试用例庞大，执行时间过长**。

    - **单元测试用例之间相互影响**。

      - 所以Jest完美的解决了上面的问题， Jest 最重要的一个特性，就是支持并行执行。

        `使用 create-react-app 产生的项目自带 Jest 作为测试框架，不奇怪，因为 Jest 和 React 一样都是出自 Facebook。`

        运行下面的命令，就可以进入交互式的”测试驱动开发“模式：

        ```react
        npm test
        ```

        **测试 React 或者 JavaScript 代码，用 Jest！**

        

- #### 🌵用Enzyme:最受欢迎的 React 测试工具库却出自 Airbnb，这个工具库叫做 Enzyme。

  在 create-react-app 产生的应用中并不包含 Enzyme，需要我们自己来添加。

  ```react
  npm i --save-dev enzyme enzyme-adapter-react-16
  ```

  我们不光要安装 enzyme，还要安装 `enzyme-adapter-react-16`

  - 因为不同 React 版本有各自特点，所用的适配器也会不同：
    - 我们的项目中使用的是 16.4 之后的版本，所以用 `enzyme-adapter-react-16`
    - 如果用 16.3 版本，需要用 `enzyme-adapter-react-16.3`
    - 如果用 16.2 版本，需要用 `enzyme-adapter-react-16.2`；

  实在是不太能往下写后续的

- 保持100%代码覆盖率

  在 create-react-app 创造的应用中，已经自带了代码覆盖率的支持，运行下面的命令，不光会运行所有单元测试，也会得到覆盖率汇报。

  ```react
  npm test -- --coverage
  ```

  🌵代码覆盖率包含的四个方面：

  - 语句覆盖率
  - 逻辑分支覆盖率
  - 函数覆盖率
  - 代码行覆盖率

