# Common pitfalls & best practices
# 常见隐患 & 最佳实践。

感觉到Mobx很操蛋？这篇文章列举了刚开始接触Mobx时会遇到的常见问题。
Stuck with MobX? This section contains a list of common issues people new to MobX might run into.

#### 关于装饰器的讨论?

查阅这个[页面](decorators.md) 获取关于装饰器的安装提示和限制。


#### `Array.isArray(observable([1,2,3])) === false`

在ES5中，没有办法可靠地继承数组，因此可观察数组继承自对象。
这意味着那些常见的库不能将可观察数组识别为普通数组。（例如 lodash，或者内部操作如`Array.concat`）
在将数组传给其他库使用之前，可以通过`observable.toJS()` 或 `observable.slice()`来转换成普通数组。
如果其他库没有修改这个数组的时候，一切都能如预期般运转。
你可以使用`isObservableArray(observable)` 来检查一些变量是否是可观察数组。


#### `object.someNewProp = value` 是无效的

Mobx 可观察变量 _objects_ 不会检测和响应之前没有声明过的属性。
你可以使用`extendObservable(target, props)`来为一个对象引入新的可观察变量。

然而对象遍历器如`for .. in` 或者 `Object.keys()` 不会自动响应新增加的属性。
如果你需要动态的字段对象，请使用[`observable.map`](../refguide/map.md)
查阅[Mobx什么时候会响应?](react.md)以获取更多信息.

### Use `@observer` on all components that render `@observable`s.
`@observer` 只是挂载你装饰过组件，而不会关心组件内部使用的组件。所以通常最好所有组件都使用装饰器。不用担心，这不是一个低效的方案，正好相反，更多的`observer`组件会使得渲染更加由效率。

### 尽可能迟地解引用
Mobx可以做很多事情，但是不能让原始类型值也变成可观察的。
所以 一个对象中，对象的属性是可观察的，而属性对应的值不是。这意味着`@observer` 仅仅对你解引用的值进行响应。
如下面这个例子，通过这种方式初始化的话，，`Timer` 组件**不会**正常地响应。

```javascript
React.render(<Timer timerData={timerData.secondsPassed} />, document.body)
```

在这个片段中，只有`secondsPassed`的值被传递给了`Timer`，是一个不可改变的值0。这个值在未来不会有任何的改变，所以`Timer`不会更新。所以我们在传递store时，最好将可观察变量的父对象传给组件。

#### 计算值执行多次超过预期

如果一个计算属性没有被别的响应行为（`autorun`, `observer` 等等）所使用，计算表达式会被延后至当他们的值被获取的时候执行（所以它们表现得就像普通属性）。
计算属性只会追踪他们所依赖的观察。如果没有实际上使用时，这允许Mobx自动暂缓计算。

查阅这个 [blog](https://medium.com/@mweststrate/becoming-fully-reactive-an-in-depth-explanation-of-mobservable-55995262a254) 或者 [issue #356](https://github.com/mobxjs/mobx/issues/356) 来获得一些解释。
所以如果你用计算属性来做其他的事情，可能并不那么有效，但如果你和`observer`、`autorun`等结合起来用，就可以变的非常有效。
当执行`transactions`时，计算属性会保持激活状态。请查阅 PRs: [#452](https://github.com/mobxjs/mobx/pull/452) 和 [#489](https://github.com/mobxjs/mobx/pull/489)

#### Always dispose reactions
#### 一直需要终止响应行为

所有的响应行为形式如`autorun`、 `observe` 和 `intercept`只有当所有它们所观察的对象被回收后，它们自己才会回收。
所以推荐当你不再使用它们时，使用返回的处理函数来终止它们的行为。
通常对于`observe`和`intercept`而言，如果它们观察的内容是`this`，没有处理的严格必要。
但对于`autorun` 而言就有点复杂了他们可能观察大量不同的可观察变量。
只要有一个变量没被回收，这个响应行为就会持续存活，这意味着所有依赖于这个响应行为的可观察对象也需要保留以支持未来的计算。
所以确保终止你的响应行为，当你不再需要他们的时候！

例如:

```javascript
const VAT = observable(1.20)

class OrderLIne {
    @observable price = 10;
    @observable amount = 1;
    constructor() {
        // this autorun will be GC-ed together with the current orderline instance
        this.handler = autorun(() => {
            doSomethingWith(this.price * this.amount)
        })
        // this autorun won't be GC-ed together with the current orderline instance
        // since VAT keeps a reference to notify this autorun,
        // which in turn keeps 'this' in scope
        this.handler = autorun(() => {
            doSomethingWith(this.price * this.amount * VAT.get())
        })
        // So, to avoid subtle memory issues, always call..
        this.handler()
        // When the reaction is no longer needed!
    }
}

```
#### 当在React component 组件里使用`@observable`的时候，我有一个奇怪的异常
#### I have a weird exception when using `@observable` in a React component.

异常如下: `Uncaught TypeError: Cannot assign to read only property '__mobxLazyInitializers' of object` 
当使用`react-hot-loader` 的时候会出现，因为那不支持装饰器
你可以在 `componentWillMount` 中使用 `extendObservable`，或者将`react-hot-loader`升级到`"^3.0.0-beta.2"` 更高的版本。

#### react components组件的显示名是没被设置过的。
#### The display name of react components is not set

如果你使用 `export const MyComponent = observer((props => <div>hi</div>))`，则不会有任何显示名。
可以通过以下方式去修正。

```javascript
// 1 (set displayName explicitly)
export const MyComponent = observer((props => <div>hi</div>))
myComponent.displayName = "MyComponent"

// 2 (MobX infers component name from function name)
export const MyComponent = observer(function MyComponent(props) { return <div>hi</div> })

// 3 (transpiler will infer component name from variable name)
const _MyComponent = observer((props => <div>hi</div>)) //
export const MyComponent = observer(_MyComponent)

// 4 (with default export)
const MyComponent = observer((props => <div>hi</div>))
export default observer(MyComponent)
```

查阅这个: http://mobxjs.github.io/mobx/best/stateless-HMR.html or [#141](https://github.com/mobxjs/mobx/issues/141#issuecomment-228457886).


#### 可观察数组的propType是一个对象

可观察数组实际上是一个对象，所以它们属于`propTypes.object`而不是`array`，`mobx-react`提供了明确的`PropTypes` 对于所有可观察数据结构。


#### Rendering ListViews in React Native

`ListView.DataSource` in React Native expects real arrays. Observable arrays are actually objects, make sure to `.slice()` them first before passing to list views. Furthermore, `ListView.DataSource` itself can be moved to the store and have it automatically updated with a `@computed`, this step can also be done on the component level.

```javascript
class ListStore {
  @observable list = [
    'Hello World!',
    'Hello React Native!',
    'Hello MobX!'
  ];

  ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });

  @computed get dataSource() {
    return this.ds.cloneWithRows(this.list.slice());
  }
}

const listStore = new ListStore();

@observer class List extends Component {
  render() {
    return (
      <ListView
        dataSource={listStore.dataSource}
        renderRow={row => <Text>{row}</Text>}
        enableEmptySections={true}
      />
    );
  }
}
```

For more info see [#476](https://github.com/mobxjs/mobx/issues/476)

#### 开发模式下，声明propTypes可能会引起不必要的渲染

查看: https://github.com/mobxjs/mobx-react/issues/56


#### 当使用Babel时，`@observable` 属性会懒初始化

可观察属性不会实例化，直到首次读写。
这会导致如下细微的bug。

```javascript
class Todo {
    @observable done = true
    @observable title = "test"
}
const todo = new Todo()

"done" in todo // true
todo.hasOwnProperty("done") // false
Object.keys(todo) // []

console.log(todo.title)
"done" in todo // true
todo.hasOwnProperty("done") // true
Object.keys(todo) // ["done", "title"]
```

在实践中很少遇到，只有在读写前调用公共方法，如`Object.assign(target, todo)`或者 `assert.deepEquals` 会出现这个问题。
如果你希望完全避免这个问题，只需要在构造函数中初始化这个属性，而不是字段声明。或者使用`extendObservable` 方法。
