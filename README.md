**以下`UNSAFE_`开头的周期都是官方将要废弃的周期**

**保留`UNSAFE_`前缀还可以继续使用 在最新的16.11.0中将抛出warning 在17.0的大版本中彻底移除componentWillMount、componentWillUpdate、componentWillReceiveProps这三个生命周期 帮助理解 但不建议继续使用和学习**


## 组件挂载时

> constructor()
- `class`的构造器 继承父类`Component`可以做一些基本的绑定操作
- 可以绑定`ref`(这里指React.createdRef()这种创建方式)、给事件绑定`this`或是`state`根据`props`获取的默认数据等
- 只会触发一次！！

> static getDerivedStateFromProps(props, state)

- 新出的静态周期 为了替代 `componentWillReceiveProps`
- 只有在父组件重新渲染 `props`发生变化才会触发
- `this.setState()`不会触发`getDerivedStateFromProps`
- 如果想要触发 需要`this.forceUpdate(callback)`=> {小常识：有时一些变量我们无需在视图层上展示 就会偷懒直接存到组件内部的`this`上 当我们某些条件下修改这个值的时候并不会触发重新渲染 这个时候我们就可以`this.forceUpdate()`手动触发告诉`react`我要渲染 给我渲染！！ 也请记住 这样做会直接跳过本组件的`shouldComponentUpdate()`直接进行渲染 所以请尽量避免这么干}
- 在此周期内部无法访问到`this`
很多刚看到这个周期的人可能都不太理解 我们在`componentWillReceiveProps`里可以根据当前`props`和下一次`props`的变更去进行`this.setState`或是派生数据之类的做法 可在这个周期里 我们无法进行比较 那我们该怎么办

在我个人看来这个周期是主要为了能更好的派生数据 很多时候我们当前需要用到的数据来自于`props` 而我们也无法直接使用`props`需要进行二次派生 这个时候我们就可以用这个周期将派生的数据存在`state`里以供使用 而不是频繁的触发`componentWillReceiveProps`不停的`render()`
```javascript
static getDerivedStateFromProps(props, state) {
    if (state.data.length === 0 && props.data.length) {
        return { data: props.data }
    }
    return null
}
```
此方法的用法很好理解 是否需要更新`state` 不需要就`return` `null` 如果需要 可以根据`props`和上一次的`state`做一些简单的比较 `return`的对象就会更新`state`

**如果我们只想要更新`state`里的`data` 但不更新其他的 我们不需要像`redux`里一样`return {...state, data}` 我们只需要更新自己想要更新的那一条就可以了**


对于上面讲到的很多时候需要根据当前`props`和下一次进行比较而做一些业务或是更新 这个时候我们就可以考虑使用`componentDidUpdate` 甚至是 `shouldComponentUpdate`(当然 考虑到周期的取名和合理性 我们还是尽量避免这种姿势)

> UNSAFE_componentWillMount()

- 此周期在`render`前触发 有的人喜欢在这里做一些订阅或是监听的操作 建议移到`constructor`或是`componentDidMount`里

> render()

- 更新组件的`props`和`state`返回一个React元素/节点
- 但`render`是一个纯函数 没有副作用 也不应该有副作用 所以我们不要在`render`里做`this.setState`类似这种操作
- 不负责最后的渲染工作
- `null`和`boolean`不会被渲染

> componentDidMount()
- 组件挂载后 官方的话说就是已经插入`dom`节点 我们需要做一些绑定的操作可以在这里
- 方便做一些订阅或是监听在这里
- 绑定的`ref`在这是拿不到的
- 我们也可以在这里做一些修改`state`的操作 那将会触发再次`render`(并非浏览器页面的再次渲染) 但尽量避免这么干 至于为什么 这还用说吗
- 和`constructor`一样 只会触发一次

## update更新时
既然是更新时 那就代表 下面这些会按顺序全部都触发！

当然 不完全是这样 组件内部`state`的更新不会触发`UNSAFE_componentWillReceiveProps`和`getDerivedStateFromProps`

但如果是组件内部的`state`更新了 子组件用到了这里的`state`(也就是子组件的`props` 父组件的`state`) 则会遵循规则 子组件会触发这些周期 组件之间都会遵循这些机制

> UNSAFE_componentWillReceiveProps(nextProps, nextState)

```javascript
    UNSAFE_componentWillReceiveProps(nextProps, nextState) {
        if (this.state.visible !== nextProps.visible) {
            this.setState({ visible: nextProps.visible })
        }
    }
```

- 接收两个参数 分别是下一次的`props`和下一次的`state` 我们可以在里面比较两次的变化然后to do something
- 但也因为如此 很容易出现一些我们无法预估的bug出现
- 之前这里的周期名写错了 也没人提醒一下我 非常抱歉~
> static getDerivedStateFromProps(props, state)
- 同上述一样
- 注意组件内部的`state`直接变化不会触发这里 若有需要 可以`this.forceUpdate(callback)` 同上
- 如果是`props`的数据变化触发此周期 再次派生`state` 这里的触发是因为`props` 而不是单纯的组件内部`state`变化

> shouldComponentUpdate(nextProps, nextState)

官方认为`state`每次更新了 就应该重新`render` 大部分情况下我们应该遵循渲染的规则

大多出现在需要优化的地方`props`或`state`不可控 频繁触发`render`的时候 如长时间的倒计时等

只作为优化而存在 也可参考使用[PureComponent](https://zh-hans.reactjs.org/docs/react-api.html#reactpurecomponent)浅比较组件

若你使用的是纯函数式组件也想达到此效果或是目的可以使用[React.memo()](https://zh-hans.reactjs.org/docs/react-api.html#reactmemo)

如果你也在同时学习或使用`hook` 你也可以使用[useMemo](https://zh-hans.reactjs.org/docs/hooks-reference.html#usememo)
```javascript
    shouldComponentUpdate(nextProps, nextState){
        if (this.props.value !== nextProps.value) {
            return true
        }
        return false
    }
```

- 接收两个参数 下一次拿到的`props`和下一次拿到的`state`。
- 这个周期是`render`渲染前的最后一个周期 代表是否`render`(除开`componentWillUpdate`将要废弃)。
- 在最后一定要返回一个`false`代表不渲染 否则会出现意想不到的影响。
- 在`return false`前可以根据当前的`this.props || this.state`等等里面的某些值进行判断 需要渲染就 `return true`。
- 如果判断是否渲染的条件里要用到循环或是`JSON.parse JSON.stringify`等序列化 反序列化的操作 性能可能更差 不如不用

> UNSAFE_componentWillUpdate(nextProps, nextState)
- 在更新发生前被立即调用 你不能在此调用`this.setState()`或`dispatch`等操作触发组件再次更新
- 在这个时候做任何操作其实都是不合理的 不论是导致`props`还是`state`的改变再次导致重复性的工作 副作用不可想象
- 确实有需要 请移步`componentDidUpdate`
- 官方将其废除这个做法非常的nice！！

> getSnapshotBeforeUpdate(prevProps, prevState)
- 这个方法会在把渲染结果提交到`DOM`之前被调用 它可以返回一个参数
- 这个参数被`componentDidUpdate(prevProps, prevState, snapshot)`方法的第三个参数接收
- 官方的说法是组件能在发生更改之前从`DOM`中捕获一些信息（例如滚动位置）
- 我是还没有遇到过使用场景太细节也不清楚

> render()
- 各种将数据过滤转译成字符串 diff算法 更新部分更改部分等等操作 然后生成节点 同上

> componentDidUpdate(prevProps, prevState, snapshot)
```javascript
    componentDidUpdate(prevProps, prevState, snapshot) {
        if (this.props.value !== prevProps.value) {
          this.changeValue(this.props.value)
        }
    }
```
- 在更新发生后被立即调用。可以在DOM更新完之后做一些收尾的工作 也就是说 首次挂载一定不会触发 别理解错
- 我们可以在这里做一些原来使用`UNSAFE_componentWillUpdate`所做的一些根据上一次`props state`和下一次的不同而触发的逻辑或是业务需要
- 但和`UNSAFE_componentWillUpdate`不同 这里接收到的是上一次的`props`和`state`

## 卸载时
> componentWillUnmount()
- 我们可以在这边做收尾工作 清除计时器、订阅、监听等等
- 大多数时候会出现一些`this.setState`不能对不存在的组件进行修改的警告甚至是报错
- 其实就是组件已经卸载(销毁)了 异步的`this.setState`还在修改已销毁的`state` 当然拿不到
- 所以有时我们需要以下种种姿势
```javascript
componentWillUnmount() {
    this.setState = (state, callback) => { return }
    window.removeEventListener()
    clearInterval()
}
```



**这是本人曾经学习到使用后来一点点积累的笔记整理出来的 仅供参考 如果有写的不对的有是遗漏 欢迎及时指出**
