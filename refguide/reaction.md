# Reaction

用法: `reaction(() => data, data => { sideEffect }, options?)`.

`autorun` 的变体，对于 observables 的跟踪给予了更加细粒度的控制。
它需要两个函数，第一个（data 方法）被跟踪，返回的 data 会被作为 effect 方法的输入值。
不同于 `autorun`，副作用不会在创建时就出现，必须等到 data 表达式返回一个新的值。
副作用执行时，任何 observables 不会被跟踪。

副作用能去抖，就像 `autorunAsync`。
`reaction` 返回一个处理器函数。
这个函数传递给 `reaction` 一个参数，在 `reaction` 调用时接受，当前的 reaction，能在执行期间 dispose。

重要的是要注意副作用通过 data 的表达式对 data 做出反应，作用小于在副作用中 data 被实际使用。
当然，副作用仅仅能在 data 表达式返回值发生改变时触发。
换句话说，reaction 需要你在副作用中创建出你所需要的。

## 参数

Reaction 接受具有以下可选的 options 对象作为第三个参数：

* `context`: `this` 用于传递给 `reaction`. 默认为 undefined (使用箭头函数代替!)
* `fireImmediately`: 布尔类型的参数表明，在 data 方法第一次运行后立即触发 effect 方法。默认值是 `false`. 如果一个布尔值作为 `reaction` 作为第三个参数，将被当做 `fireImmediately`。
* `delay`: 可为 effect 方法去抖，时间是毫秒。如果是 0 (默认), 将不会发生去抖。
* `compareStructural`: 默认为 `false` . 如果 `true`, *data* 方法的返回值与之前的返回值会进行结构上的比较，*effect* 方法仅仅在结构发生改变时被调用。
* `name`: 为这个 reaction 设置一个名字为了 [`spy`](spy.md) 事件

## 示例

在下面例子中，`reaction1`, `reaction2` 与 `autorun1` 均会对 `todos` 数组的添加、删除、代替做出反应。
但是只有 `reaction2` 与 `autorun` 会对 todo 中的 `title` 变化做出反应，因为 `title` 在 reaction 2 的 data 方法中被使用，但在 reaction 1 的表达式中没有被使用。
`autorun` 跟踪完整的副作用，因此总是被正确地触发，但是更容易意外访问到不相关的数据。
详见 [what will MobX React to?](../best/react).

```javascript
const todos = observable([
    {
        title: "Make coffee",
        done: true,
    },
    {
        title: "Find biscuit",
        done: false
    }
]);

// 错误使用 reaction，反应 todos.length 变化，而不是 title
const reaction1 = reaction(
    () => todos.length,
    length => console.log("reaction 1:", todos.map(todo => todo.title).join(", "))
);

// 正确使用，反应 length 与 title 的变化
const reaction2 = reaction(
    () => todos.map(todo => todo.title),
    titles => console.log("reaction 2:", titles.join(", "))
);

// autorun 反应函数中访问到的一切
const autorun1 = autorun(
    () => console.log("autorun 1:", todos.map(todo => todo.title).join(", "))
);

todos.push({ title: "explain reactions", done: false });
// 输出:
// reaction 1: Make coffee, find biscuit, explain reactions
// reaction 2: Make coffee, find biscuit, explain reactions
// autorun 1: Make coffee, find biscuit, explain reactions

todos[0].title = "Make tea"
// 输出:
// reaction 2: Make tea, find biscuit, explain reactions
// autorun 1: Make tea, find biscuit, explain reactions
```

Reaction 的语法糖: `computed(expression).observe(action(sideEffect))` 或 `autorun(() => action(sideEffect)(expression)`
