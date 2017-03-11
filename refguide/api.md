# MobX Api 参考


这篇文档适用于 MobX 3 或者更高版本。如果你还使用 MobX 2的话，老版本的[文档](https://github.com/MobXjs/MobX/blob/7c9e7c86e0c6ead141bb0539d33143d0e1f576dd/docs/refguide/api.md)仍然是可用的。

# 核心 API

_这是最重要的 MobX API。仅仅理解 `observable`, `computed`, `reactions` 和 `actions` 就足够让你掌握 MobX 并且在应用中使用它!_


## 创建可观察变量（observables）


### `observable(value)`
用法:
* `observable(value)`
* `@observable classProperty = value`


observable 函数的参数可以是JS原始类型、引用、纯对象、类实例、数组和maps。
`observable(value)` 是一个方便而又强大的方法，他会尽可能地用最合适的类型来创建可观察变量。

有如下转换规则，但是它们可以使用装饰符调整行为，我们往下看。

1. 如果 `value` 是 [ES6 Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)  的一个实例。将会返回一个新的 [Observable Map](map.md) 。当你想指定一个特定条目改变时不触发响应,而是在增加或者删除时响应，Observable maps是非常有用的一种方式。
2. 如果 `value` 是一个数组，将会返回一个新的 [Observable Array](array.md)。
3. 如果 `value` 是一个_无_原型的对象，这个对象会被复制，并且当前含有的所有属性都会被观察。详见 [Observable Object](object.md)。
4. 如果 `value` 是一个_有原型_的对象，JS原始类型或者函数，将会返回一个 [Boxed Observable](boxed.md) 。MobX不会自动观察一个包含原型的对象，因为这是它的构造函数的责任。在构造函数中使用 `extendObservable` ，或者使用 `@observable` 修饰符在其类定义的时候代替之前的用法。


这些规则第一眼看上去很复杂，但是在实际使用中，你会发现它们运作起来是非常直观的。

一些注意事项：

* 为了创建一个包含动态属性的对象，请永远使用maps！（译者按：不用就有坑）对象中只有初始化时存在的属性是可观察的，虽然可以使用 `extendObservable` 来新增属性。
* 如果使用 `@overvable` 装饰器，要确保在你的编译器（babel 或 typescript）中[启用装饰器语法](http://MobXjs.github.io/MobX/refguide/observable-decorator.html)
* 创建一个可观察的数据结构是具有*传染性*的。那意味着 `observable` 会自动将数据结构中所有的值一同转变为 `observable` 。这个行为可以通过 *修饰符（modifiers）* 或者 *浅观察（shallow）* 改变。


[&laquo;`observable`&raquo;](observable.md)  &mdash;  [&laquo;`@observable`&raquo;](observable-decorator.md)




### 用法：`@observable property =  value`

`observable` 可以用作一个属性的装饰器. 这需要使[装饰器可用](../best/decorators.md) ，这也是一个 `extendObservable(this, { property: value })` 的语法糖.

[&laquo;`详情`&raquo;](observable-decorator.md)

### 用法：`observable.box(value)` & `observable.shallowBox(value)`

这个方法创建一个存储了可观察的 _box_ ，这个 box 是一个可观察引用。请使用 `get()` 获取当前这个 box 的值，使用 `set()` 更新这个box的值。
这是其他可观察对象构建的基础。但在实际使用中，你只有很少的情况会用到。
普通 box 会自动将任何新的值转换为可观察的，可以使用 `shallowBox` 来禁止这种行为

[&laquo;`详情`&raquo;](boxed.md)

### 用法：`observable.object(value)` & `observable.shallowObject(value)`

这个方法克隆一个对象，并且将其所有属性都转为可观察的。
默认所有属性的值都会转化为可观察的，但是如果使用 `shallowObject` ，则只有这个属性自身会变为可观察的，属性的值不会受到干扰。

[&laquo;`详情`&raquo;](object.md)

### 用法：`observable.array(value)` & `observable.shallowArray(value)`

创建一个新的可观察数组。如果数组中的值不想变为可观察的，请使用 `shallowArray` 。

[&laquo;`详情`&raquo;](array.md)

### 用法：`observable.map(value)` & `observable.shallowMap(value)`

创建一个新的可观察map。如果map中的值不想变为可观察的，请使用 `shallowMap` 。
注意map只支持字符串类型的key。

[&laquo;`详情`&raquo;](map.md)

### 用法：`extendObservable` & `extendShallowObservable`

### `extendObservable`

用法: `extendObservable(target, propertyMap)`
对于 `propertyMap` 中的任何一个键/值对，目标对象上将会对应生成一个新的 `observable` 属性。我们可以使用它在 `constructor` 构造函数引入 `observable` 属性，以取代装饰器。
如果 `propertyMap` 中的值为无参函数时，会被当做是一个计算值（computed value）。

如果新的属性不需要被传染（新加入的属性的值也会被转化成可观察的）
注意： `extendObservable` 改变已有的对象，而不是像 `observable.object` 创建一个新对象。

[&laquo;详情&raquo;](extend-observable.md)

### 修饰符（Modifiers）



修饰符可以使用装饰器的用法，也可以组合 `extendObservable` 和 `observable.object` ，以改变特定属性的自动转化行为。

可以使用以下修饰符：

* `observable.deep`：默认的修饰符。将任何非原始类型的值转变为可观察的。
* `observable.ref`：禁用自动的可观察转换，只创建一个可观察的引用。
* `observable.shallow`：只可用于集合（collections）。集合中每个元素不会自动转变为可观察的。
* `computed`：创建一个计算值。看[`computed`](computed-decorator.md)
* `action`：创建一个行为。看[`action`](action.md)

修饰符可以使用装饰器的用法：

```javascript
class TaskStore {
    @observable.shallow tasks = []
}
```

或者使用ES5的语法。

注意：修饰符的作用会一直和属性绑定，即使属性的值改变了。

```javascript
const taskStore = observable({
    tasks: observable.shallow([])
})
```

[&laquo;详情&raquo;](modifiers.md)


## 计算值（Computed values）

用法:
* `computed(() => expression)`
* `computed(() => expression, (newValue) => void)`
* `computed(() => expression, options)`
* `@computed get classProperty() { return expression; }`
* `@computed.struct get classProperty() { return expression; }`

创建一个计算值。 `expression` 不应该有任何副作用，而仅仅是返回一个值。当这个 `expression` 依赖的可观察属性变化时，这个表达式会重新计算。

[&laquo;详情&raquo;](computed-decorator.md)

## 行为（Actions）

任何应用都有行为。任何改变状态的代码都称为行为。
使用MobX可以使你的代码更加清晰，Action会使你的代码结构更优。
建议在任何改变状态或具有副作用的函数上使用。
当结合调试工具使用时， `action` 也会提供有用的debug信息。
注意：当严格模式开启时。使用 `action` 是强制的。

[&laquo;详情&raquo;](action.md)

用法:
* `action(fn)`
* `action(name, fn)`
* `@action classMethod`
* `@action(name) classMethod`
* `@action boundClassMethod = (args) => { body }`
* `@action(name) boundClassMethod = (args) => { body }`

对于只执行一次的行为，可以使用 `runInAction(name?, fn, scope?)`，这是 `action(name, fn, scope)()` 的语法糖。

## 响应 & 衍生

*计算值（Computed values）* 是当状态变化时会自动生成的新值。
*Reactions* 是当状态变化时会自动执行的副作用（side effect)。
`Reaction` 可用于确保在相关状态改变（如状态改变，日志记录，网络请求等）时自动的执行某些副作用(主要是`I/O`操作)。最常用的响应是 `React` 组件的 `observer` 装饰器（见下文）。


### `observer`

可以作为包裹React组件的高阶组件。然后在组件的 `render` 函数中使用的任何可观察变量变化时，组件就自动的重新渲染。**注意`observer`由`mobx-react`包提供，而不是由`MobX`本身提供的**。

[&laquo;详情&raquo;](observer-component.md)

用法:
* `observer(React.createClass({ ... }))`
* `observer((props, context) => ReactElement)`
* `observer(class MyComponent extends React.Component { ... })`
* `@observer class MyComponent extends React.Component { ... })`


### `autorun`
用法: `autorun(debugname?, () => { sideEffect })`. 
用法：`autorun(debugname?, () => { sideEffect })`，`Autorun` 会运行提供的 `sideEffect` 并且会追踪副作用运行时使用的可观察变量。任何一个使用的可观察变量变化时， `sideEffect` 都会被重新运行。其返回一个销毁函数以取消副作用。
。[&laquo;详情&raquo;](autorun.md)

### `when`
用法：`when(debugname?, () => condition, () => { sideEffect })`。条件表达式在其使用的任何可观察变量变化时会自动执行。一旦表达式返回true，`sideEffect` 函数将被调用，但只调用一次。 `when` 会返回一个销毁函数以取消整个过程。 [&laquo;详情&raquo;](when.md)

### `autorunAsync`
用法：`autorunAsync(debugname?, () => { sideEffect }, delay)`。和 `autorun` 相似，但是 `sideEffect` 将被延迟执行以达到去抖目的。
[&laquo;details&raquo;](autorun-async.md)

### `reaction`
用法: `reaction(debugname?, () => data, data => { sideEffect }, fireImmediately = false, delay = 0)`.
`autorun` 的一个变种，给予了更多的可控制性。
需要传递两个函数，第一个函数追踪状态变化，并返回第二个函数（副作用函数）所使用的数据。
与 `autorun` 不同，副作用函数不会立刻执行，并且在执行过程中，任何被改变的可观察变量都不会被追踪和响应。
副作用可以去抖，就像 `autorunAsync` 一样。[&laquo;详情&raquo;](reaction.md)

### `expr`
用法: `expr(() => someExpression)`. 只是 `computed(() => someExpression).get()` 的快捷方式.
在一些特殊的情况下，为了充分优化计算值和 `reaction`，`expr` 非常有用。

[&laquo;详情&raquo;](expr.md)

### `onReactionError`

用法: `extras.onReactionError(handler: (error: any, derivation) => void)`

这个方法绑定了一个全局错误监听器。当任何_reaction_中的错误触发时，都会抛出错误。这个特性主要用于监控和测试。

------

# 工具方法（Utilities）

这里可能有一些工具，使得使用被观察对象和计算值更加方便。参见[MobX-utils](https://github.com/MobXjs/MobX-utils)

### `Provider` (`mobx-react` package)

可以通过React的上下文机制，将store传给子组件。[`mobx-react` docs](https://github.com/MobXjs/mobx-react#provider-experimental).

### `inject` (`mobx-react` package)
与 `Provider` 结合使用的部分。用于将store中的部分状态，通过上下文的形式注入给子组件，用法如下：
* `inject("store1", "store2")(observer(MyComponent))`
* `@inject("store1", "store2") @observer MyComponent`
* `@inject((stores, props, context) => props) @observer MyComponent`
* `@observer(["store1", "store2"]) MyComponent` 是 `@inject() @observer` 的快捷用法.

### `toJS`
用法: `toJS(observableDataStructure)`. 将数据结构转成简单的JS形式。[&laquo;详情&raquo;](tojson.md)

### `isObservable`
用法: `isObservable(thing, property?)`. 如果传入的 `thing` 或者 `property` 是可观察的，则返回true。
可用于所有的可观察变量，计算值或相应行为的销毁函数。 [&laquo;details&raquo;](is-observable)

### `isObservableObject|Array|Map`
用法: `isObservableObject(thing)`, `isObservableArray(thing)`, `isObservableMap(thing)`. 如果匹配就返回 `true` 。

### `isArrayLike`
用法: `isArrayLike(thing)`. 如果传入的 `thing` 是一个JS数组或可观察的Mobx数组，则返回 `true` 。
这通常作为一个快捷用法。注意可观察的数组可以通过 `.slice()` 转变成真正的JS数组。

### `isAction`
用法: `isAction(func)`. 如果传入的函数被 `action` 包裹或装饰，则返回true。

### `isComputed`
用法: `isComputed(thing, property?)`. 如果传入的函数被 `computed` 包裹或装饰，则返回true。

### `createTransformer`
用法: `createTransformer(transformation: A => B, onCleanup?): A = B`.
可以用于创建一个函数，将一个值转换为另一个可以响应和缓存的值。它和计算值类似，但是可以用于更进一步的模式，更加高效地处理数组或者不是对象一部分的计算值。
[&laquo;详情&raquo;](create-transformer.md)

### `intercept`
用法: `intercept(object, property?, interceptor)`.
用于拦截可观察变量的变化。用于验证、格式化和取消。
[&laquo;details&raquo;](observe.md)

### `observe`
用法: `observe(object, property?, listener, fireImmediately = false)`
观察对象的底层接口
[&laquo;details&raquo;](observe.md)

### `useStrict`
用法: `useStrict(boolean)`.
全局开启或关闭严格模式。
在严格模式中，不允许除 [`action`](action.md) 以外的任何改变状态的代码。
也可查阅 `extras.allowStateChanges`.

# 开发工具（Development utilities）

_如果你想在 MobX 顶层构建酷炫的工具或是检查 MobX 的内部状态，下列 API 可能会被使用_

### `"mobx-react-devtools"` 包
mobx-react-devtools 是一个强大的包，帮你查看 react 组件的性能和依赖。
也是一个基于 `spy` 强大的日志实用程序。[&laquo;details&raquo;](../best/devtools.md)

### `spy`
用法: `spy(listener)`.
注册一个全局的 spy 监听器来监听 MobX 发生的所有事件。
这相当于将 `observe` 监听器一次性附加到 *所有* observables，也包括通知正在运行的 (trans/re)actions 和 computations。
使用的例子如：`mobx-react-devtools`。
[&laquo;details&raquo;](spy.md)

### `whyRun`
用法:
* `whyRun()`
* `whyRun(Reaction object / ComputedValue object / disposer function)`
* `whyRun(object, "computed property name")`

`whyRun` 是用于在计算值（computed value）或 reaction（`autorun`, `reaction` 或是 `observer` React 组件的 `render` 方法）
并打印出为何 derivation 可以正确运行，在什么情况下将再次运行
这将有助于更深入理解 MobX 何时以及为何运行，避免许多初学者会犯的错误。

### `extras.getAtom`
用法: `getAtom(thing, property?)`.
返回给定 observable 对象，属性，reaction等等 的 *Atom*。

### `extras.getDebugName`
用法: `getDebugName(thing, property?)`
返回 observable 对象，属性，reaction等等的（生成的）对开发者友好的 debug 名称。例如 `mobx-react-devtools` 的使用。

### `extras.getDependencyTree`
用法: `getDependencyTree(thing, property?)`.
返回给定 reaction / computation 当前依赖 observables 的树结构。

### `extras.getObserverTree`
用法: `getObserverTree(thing, property?)`.
返回一个所有 reactions / computations 的树结构，这些 reactions / computations 均正在观察给定 observable。

### `extras.isSpyEnabled`
用法: `isSpyEnabled()`. 如果当前至少有一个 spy，返回 true。

### `extras.spyReport`
用法: `spyReport({ type: "your type", &laquo;details&raquo; data})`. 触发你自定义的 spy 事件。

### `extras.spyReportStart`
用法: `spyReportStart({ type: "your type", &laquo;details&raquo; data})`. 触发你自定义 spy 事件。将开始一个新的嵌套 spy 事件组，可以用 `spyReportEnd()` 关闭。

### `extras.spyReportEnd`
用法: `spyReportEnd()`. 结束当前用 `extras.spyReportStart` 开始的 spy 事件组。

### `"mobx-react"` 开发钩子
`mobx-react` 包为 `mobx-react-devtools` 暴露额外的 API：
* `trackComponents()`: 启用对基于 React 组件的 `observer` 跟踪。
* `renderReporter.on(callback)`: 每当作用于 `observer` 的 React 组件渲染时，callback 将会被调用，伴随着时间信息等等。
* `componentByNodeRegistery`: ES6 WeakMap 从 DOM 节点映射到基于 React 组件实例的 `observer`。

# 内部函数（Internal functions）

_下面这些方法都是 MobX 内部使用的，并且可能可以用于处理一些特殊情况。但通常 MobX 会提供更加语义化的方式去解决这些问题。_

### `transaction`
用法: `transaction(() => { block })`.
已废弃。

### `untracked`
用法: `untracked(() => { block })`.
底层函数，用于包裹不希望触发响应的代码。
[&laquo;详情&raquo;](untracked.md)

### `Atom`
工具类，可用于创建你自己的可观察对象，并挂载在MobX上。内部被所有可观察类型使用。
[&laquo;详情&raquo;](extending.md)

### `Reaction`
工具类，用于创建你独特的响应方法并将其挂载在MobX上。内部被 `autorun`, `reaction` 所使用。
[&laquo;详情&raquo;](extending.md)

