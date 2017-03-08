# 优化React 组件的渲染

Mobx性能非常好，[通常比 Redux 更好](https://twitter.com/mweststrate/status/718444275239882753)。这里有一些提示能应用到React和Mobx的大多数场景。注意这些提示对于React的应用也是通用的优化措施，并不仅限于Mobx。

## 使用小组件
`@observer` 组件会追踪所有需要重渲染的组件。你的组件越小，重渲染的范围越小。那意味着你的UI可以和其他部分独立渲染。

## 使用专门的组件渲染列表数据。
当渲染大量数据时，这一条尤其重要。
众所周知，React 在渲染大数据上很赵高，当数据集合改变时，每一个组件会重新协调（reconcile）。
所以推荐将组件映射到数据集合，以便改变对应的部分，而不影响其他。

坏的例子:

```javascript
@observer class MyComponent extends Component {
    render() {
        const {todos, user} = this.props;
        return (<div>
            {user.name}
            <ul>
                {todos.map(todo => <TodoView todo={todo} key={todo.id} />)}
            </ul>
        </div>)
    }
}
```

如果是像上面的列表，当`user.name`变化时，React会不必要地重新协调（reconcile）所有的`TodoView`组件，他不会重渲染，但重新协调这一过程代价昂贵。

好的例子:

```javascript
@observer class MyComponent extends Component {
    render() {
        const {todos, user} = this.props;
        return (<div>
            {user.name}
            <TodosView todos={todos} />
        </div>)
    }
}

@observer class TodosView extends Component {
    render() {
        const {todos} = this.props;
        return <ul>
            {todos.map(todo => <TodoView todo={todo} key={todo.id} />)}
        </ul>)
    }
}
```
## 不要使用数据索引作为keys

不要使用数组索引或者任何未来可能改变的值作为key。请为你的对象生成一个id。详细查阅这个[blog](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318).

## 尽可能晚的解引用

当使用 `mobx-react` 时，推荐的做法是尽可能晚地解引用。因为Mobx会自动重渲染那些解引用了可观察变量的组件。
所以如果这个重渲染出现在你组件树更底层，就会只有更少的组件需要重渲染。

快:

`<DisplayName person={person} />`

慢:

`<DisplayName name={person.name} />`.

用后面这种没毛病。但如果`name`属性变化，第一种情况下会触发`DisplayName`的重渲染，在第二种情况下会触发`DisplayName` 的_父组件_重渲染。
然而，对你的组件而言，比这个性能优化而言更重要的是可理解的API，所以不要过度优化哟。
如果需要鱼和熊掌兼得，考虑使用细粒度的组件。

`const PersonNameDisplayer = observer(({ props }) => <DisplayName name={props.person.name} />)`

## 尽早绑定函数

这个提示对于React使用而言是通用的，它会影响使用`PureRenderMixin`库。

请查看这些资源:
* [Autobinding with property initializers](https://facebook.github.io/react/blog/2015/01/27/react-v0.13.0-beta-1.html#autobinding)
* [ESLint rule for no-bind](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md)


坏例子:

```javascript
render() {
    return <MyWidget onClick={() => { alert('hi') }} />
}
```

好例子:

```javascript
render() {
    return <MyWidget onClick={this.handleClick} />
}

handleClick = () => {
    alert('hi')
}
```

坏例子会使`PureRenderMixin`所使用的 `shouldComponent` 一直返回false，因为当父组件重渲染时，你一直传了一个新的函数。