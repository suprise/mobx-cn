# MobX Api 参考


这篇文档适用于MobX3或者更高版本。如果你还使用MobX的话，老版本的[文档](https://github.com/MobXjs/MobX/blob/7c9e7c86e0c6ead141bb0539d33143d0e1f576dd/docs/refguide/api.md)仍然是可用的。

# 核心 API

_这是最重要的MobX API。仅仅理解`observable`, `computed`, `reactions`和`actions`就足够让你掌握MobX并且在应用中使用它!_


## 创建 observables


### `observable(value)`
Usage:
* `observable(value)`
* `@observable classProperty = value`


Observable的值可以是JS元数据，引用，纯对象，类实例，数组和maps。
`observable(value)` 是一个方便而又强大的方法，他会尽可能地用最合适的可观察类型来创建Observable。

有如下转换规则，但是它们可以使用装饰符微调，我们往下看。

1. 如果`value`是[ES6 Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) 的一个实例。将会返回一个新的[Observable Map](map.md)。当你想指定一个特定条目改变时不触发响应,而是在增加或者删除时响应，Observable maps是非常有用的一种方式。
2. 如果`value`是一个数组，将会返回一个新的[Observable Array](array.md)。
3. 如果`value`是一个_无_原型的对象，这个对象会被复制，并且当前含有的所有属性都会被观察。详见[Observable Object](object.md)。
4. 如果`value`是_包含_原型，JS初始类型或者函数，将会返回一个[Boxed Observable](boxed.md)。MobX不会自动的观察一个包含原型的对象，因为这是它的构造函数的责任。在构造函数中使用`extendObservable`，或者试用`@observable`修饰符在其类定义的时候取代。



这些规则第一眼看上去很复杂，但是在实际使用中，你会发现它们运作起来是非常直观的。

一些注意事项：

* 为了创建一个包含动态属性的对象，请永远使用maps！对象中只有初始化时存在的属性是可观察的，虽然可以使用`extendObservable` 来新增属性。
* 如果使用`@overvable`装饰器，要确保在你的编译器（babel 或 typescript）中[启用装饰器语法](http://MobXjs.github.io/MobX/refguide/observable-decorator.html)
* 创建一个可观察的数据结构是具有*传染性*的。那意味着`observable`会自动将数据结构中所有的值一同转变为`observable`。这个行为可以通过 *modifiers* 或者 *shallow* 改变。


[&laquo;`observable`&raquo;](observable.md)  &mdash;  [&laquo;`@observable`&raquo;](observable-decorator.md)




### 用法：`@observable property =  value`

`observable` 可以用作一个属性的装饰器. 这需要使[装饰器可用](../best/decorators.md) 这也是一个`extendObservable(this, { property: value })`的语法糖.

[&laquo;`详情`&raquo;](observable-decorator.md)

### 用法：`observable.box(value)` & `observable.shallowBox(value)`

这个方法创建一个存储了可观察的_box_，这个box是一个可观察引用。请使用`get()`获取当前这个box的值，使用`set()`去更新这个值。
这是其他可观察对象构建的基础。但在实际使用中，你只有很少的情况会用到。
普通box会自动将任何新的值转换为可观察的，可以使用`shallowBox`来禁止这种行为

[&laquo;`详情`&raquo;](boxed.md)

### 用法：`observable.object(value)` & `observable.shallowObject(value)`

克隆一个对象，并且将其所有属性都转为可观察的。
默认所有属性的值都会转化为可观察的，但是如果使用了`shallowObject`，这只有这个属性自身会变为可观察的，但属性的值不会受到干扰。

[&laquo;`详情`&raquo;](object.md)

### 用法：`observable.array(value)` & `observable.shallowArray(value)`

创建一个新的可观察数组。如果数组中的值不想变为可观察的，请使用`shallowArray`。

[&laquo;`详情`&raquo;](array.md)

### 用法：`observable.map(value)` & `observable.shallowMap(value)`

创建一个新的可观察map。如果map中的值不想变为可观察的，请使用`shallowMap`。
注意map只支持字符串类型的key。

[&laquo;`详情`&raquo;](map.md)

### 用法：`extendObservable` & `extendShallowObservable`

### `extendObservable`

用法: `extendObservable(target, propertyMap)`
对于`propertyMap`中的任何一个键/值对，目标对象上将会被引入一个新的`observable`属性。我们可以使用它在`constructor`构造函数引入`observable`属性，取代装饰器。
如果`propertyMap`中的值为无参函数时，会被当做是一个计算值。

如果新的属性不需要被传染（新加入的属性的值也会被转化成可观察的）
注意：`extendObservable`改变已有的对象，而不是像`observable.object`创建一个新对象。

[&laquo;详情&raquo;](extend-observable.md)

### 改变规则（Modifiers）

Modifiers可以使用装饰器的用法，也可以组合`extendObservable` 和 `observable.object`，以改变特定属性的自动转化规则。

以下modifiers 可用：

* `observable.deep`：默认的改变规则。将任何非原始类型的值转变为可观察的。
* `observable.ref`：禁用自动的可观察转换，只创建一个可观察的引用。
* `observable.shallow`：只可用于集合（collections）。集合中每个元素不会自动转变为可观察的。
* `computed`：创建一个计算值。看[`computed`](computed-decorator.md)
* `action`：创建一个行为。看[`action`](action.md)

Modifiers可以使用装饰器的用法：
```javascript
class TaskStore {
    @observable.shallow tasks = []
}
```

或者使用ES5的语法。

注意：modifiers的作用会一直和属性绑定，即使属性的值改变了。

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

创建一个计算值。`expression` 不应该有任何副作用，而仅仅是返回一个值。当这个`expression`依赖的可观察属性变化时，这个表达式会重新计算。

[&laquo;详情&raquo;](computed-decorator.md)

## 行为（Actions）

任何应用都有行为。任何改变状态的代码都称为行为。
使用MobX可以使你的代码更加清晰，Action会使你的代码结构更优。
建议在任何改变状态或具有副作用的函数上使用。
当结合调试工具使用时，`action` 也会提供有用的debug信息。
注意：当严格模式开启时。使用`action`是强制的。

[&laquo;详情&raquo;](action.md)

用法:
* `action(fn)`
* `action(name, fn)`
* `@action classMethod`
* `@action(name) classMethod`
* `@action boundClassMethod = (args) => { body }`
* `@action(name) boundClassMethod = (args) => { body }`

对于一次性行为，可以使用 `runInAction(name?, fn, scope?)`，这是 `action(name, fn, scope)()`的语法糖。

## 响应 & 衍生

*计算值（Computed values）*是当状态变化时会自动响应。
*Reactions*是当状态变化时会自动执行的副作用。
`Reaction`可用于确保在相关状态改变（如状态改变，日志记录，网络请求等）时自动的执行某些副作用(主要是`I/O`操作)。最常用的响应是`React`组件的`observer`装饰器（见下文）。


### `observer`

可以作用在React组件周围的高阶组件。然后在组件的`render`函数中使用的任何被observable的变量变化时，组件就自动的重新渲染。**注意`observer`由`mobx-react`包提供，而不是由`MobX`本身提供的**。

[&laquo;详情&raquo;](observer-component.md)

用法:
* `observer(React.createClass({ ... }))`
* `observer((props, context) => ReactElement)`
* `observer(class MyComponent extends React.Component { ... })`
* `@observer class MyComponent extends React.Component { ... })`


### `autorun`
用法: `autorun(debugname?, () => { sideEffect })`. 
用法：`autorun(debugname?, () => { sideEffect })`，`Autorun`会运行提供的`sideEffect`并且会跟踪副作用运行时使用的被观察的状态。任何一个使用的被观察的变量变化时，`sideEffect`都会被重新运行。其返回一个处理器函数以取消副作用。
。[&laquo;详情&raquo;](autorun.md)

### `when`
用法：`when(debugname?, () => condition, () => { sideEffect })`。条件表达式在其使用的任何可观察的变量变化时会自动执行。一旦表达式返回true，`sideEffect`函数将被调用，但只调用一次。`when`会返回一个处理器函数以取消整个过程。 [&laquo;详情&raquo;](when.md)

### `autorunAsync`
用法：`autorunAsync(debugname?, () => { sideEffect }, delay)`。和`autorun`相似，但是`sideEffect`将被延迟执行以达到去抖目的。
[&laquo;details&raquo;](autorun-async.md)

### `reaction`
用法: `reaction(debugname?, () => data, data => { sideEffect }, fireImmediately = false, delay = 0)`.
`autorun`的一个变种，给予了更多的可控制性。
需要传递两个函数，第一个函数追踪状态变化，并返回第二个函数（副作用函数）所使用的数据。
与`autorun`不同，副作用不会立刻执行，并且在执行副作用函数过程中，任何被改变的可观察变量都不会被追踪和响应。
副作用可以去抖，就像 `autorunAsync` 一样。[&laquo;详情&raquo;](reaction.md)

### `expr`
用法: `expr(() => someExpression)`. 只是 `computed(() => someExpression).get()`的快捷方式.
在一些特殊的情况下，为了充分优化计算值和`reaction`，`expr` 非常有用。

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
与`Provider` 结合使用的部分。用于将store中的部分状体，通过上下文的形式注入给子组件，用法如下：
* `inject("store1", "store2")(observer(MyComponent))`
* `@inject("store1", "store2") @observer MyComponent`
* `@inject((stores, props, context) => props) @observer MyComponent`
* `@observer(["store1", "store2"]) MyComponent` is a shorthand for the the `@inject() @observer` combo.

### `toJS`
用法: `toJS(observableDataStructure)`. 将数据结构转成简单的JS形式。[&laquo;详情&raquo;](tojson.md)

### `isObservable`
Usage: `isObservable(thing, property?)`. Returns true if the given thing, or the `property` of the given thing is observable.
Works for all observables, computed values and disposer functions of reactions. [&laquo;details&raquo;](is-observable)

### `isObservableObject|Array|Map`
Usage: `isObservableObject(thing)`, `isObservableArray(thing)`, `isObservableMap(thing)`. Returns `true` if.., well, do the math.

### `isArrayLike`
Usage: `isArrayLike(thing)`. Returns `true` if the given thing is either a true JS-array or an observable (MobX-)array.
This is intended as convenience/shorthand.
Note that observable arrays can be `.slice()`d to turn them into true JS-arrays.

### `isAction`
Usage: `isAction(func)`. Returns true if the given function is wrapped / decorated with `action`.

### `isComputed`
Usage: `isComputed(thing, property?)`. Returns true if the giving thing is a boxed computed value, or if the designated property is a computed value.

### `createTransformer`
用法: `createTransformer(transformation: A => B, onCleanup?): A = B`.
可以用于创建一个函数，将一个值转换为另一个可以响应和缓存的值。它和计算值类似，但是可以用于跟进一步的模式，更加高效地处理数组或是不是对象一部分的计算值。
[&laquo;详情&raquo;](create-transformer.md)

### `intercept`
用法: `intercept(object, property?, interceptor)`.
用于拦截可观察变量的变化。用于验证、格式化和取消。
[&laquo;details&raquo;](observe.md)

### `observe`
Usage: `observe(object, property?, listener, fireImmediately = false)`
观察对象的底层接口
[&laquo;details&raquo;](observe.md)

### `useStrict`
Usage: `useStrict(boolean)`.
Enables / disables strict mode *globally*.
In strict mode, it is not allowed to change any state outside of an [`action`](action.md).
See also `extras.allowStateChanges`.




# 开发工具（Development utilities）

_属于高阶用法，暂不翻译，各位大牛自取_

_The following api's might come in handy if you want to build cool tools on top of MobX or if you want to inspect the internal state of MobX_

### `"mobx-react-devtools"` package
The mobx-react-devtools is a powerful package that helps you to investigate the performance and dependencies of your react components.
Also has a powerful logger utility based on `spy`. [&laquo;details&raquo;](../best/devtools.md)

### `spy`
Usage: `spy(listener)`.
Registers a global spy listener that listens to all events that happen in MobX.
It is similar to attaching an `observe` listener to *all* observables at once, but also notifies about running (trans/re)actions and computations.
Used for example by the `mobx-react-devtools`.
[&laquo;details&raquo;](spy.md)

### `whyRun`
Usage:
* `whyRun()`
* `whyRun(Reaction object / ComputedValue object / disposer function)`
* `whyRun(object, "computed property name")`

`whyRun` is a small utility that can be used inside computed value or reaction (`autorun`, `reaction` or the `render` method of an `observer` React component)
and prints why the derivation is currently running, and under which circumstances it will run again.
This should help to get a deeper understanding when and why MobX runs stuff, and prevent some beginner mistakes.


### `extras.getAtom`
Usage: `getAtom(thing, property?)`.
Returns the backing *Atom* of a given observable object, property, reaction etc.

### `extras.getDebugName`
Usage: `getDebugName(thing, property?)`
Returns a (generated) friendly debug name of an observable object, property, reaction etc. Used by for example the `mobx-react-devtools`.

### `extras.getDependencyTree`
Usage: `getDependencyTree(thing, property?)`.
Returns a tree structure with all observables the given reaction / computation currently depends upon.

### `extras.getObserverTree`
Usage: `getObserverTree(thing, property?)`.
Returns a tree structure with all reactions / computations that are observing the given observable.

### `extras.isSpyEnabled`
Usage: `isSpyEnabled()`. Returns true if at least one spy is active

### `extras.spyReport`
Usage: `spyReport({ type: "your type", &laquo;details&raquo; data})`. Emit your own custom spy event.

### `extras.spyReportStart`
Usage: `spyReportStart({ type: "your type", &laquo;details&raquo; data})`. Emit your own custom spy event. Will start a new nested spy event group which should be closed using `spyReportEnd()`

### `extras.spyReportEnd`
Usage: `spyReportEnd()`. Ends the current spy group that was started with `extras.spyReportStart`.

### `"mobx-react"` development hooks
The `mobx-react` package exposes the following additional api's that are used by the `mobx-react-devtools`:
* `trackComponents()`: enables the tracking of `observer` based React components
* `renderReporter.on(callback)`: callback will be invoked on each rendering of an `observer` enabled React component, with timing information etc
* `componentByNodeRegistery`: ES6 WeakMap that maps from DOMNode to a `observer` based React component instance

# 内部函数（Internal functions）

_下面这些方法都是MobX内部使用的，并且可能可以用于处理一些特殊情况。但通常MoMobX供更加语义化的方式去解决这些问题_

### `transaction`
用法: `transaction(() => { block })`.
Deprecated。

### `untracked`
Usage: `untracked(() => { block })`.
底层函数，用于包裹不希望触发响应的代码。
[&laquo;详情&raquo;](untracked.md)

### `Atom`
工具类，可用于创建你自己的可观察对象，并挂载在MobX上。内部被所有可观察类型使用。
[&laquo;详情&raquo;](extending.md)

### `Reaction`
工具类，用于创建你独特的响应方法并将其挂载在MobX上。内部被`autorun`, `reaction`所使用
Utility class that can be used to create your own reactions and hook them up to MobX.
Used internally by `autorun`, `reaction` (function) etc.
[&laquo;详情&raquo;](extending.md)