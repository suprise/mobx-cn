# Intercept & Observe

`observe` 和 `intercept` 能够监听单个 observable 的变化 (他们 ***无法*** 跟踪嵌套的 observable).
`intercept` 能够探测和修改突变，在它们作用于 observable 之前。
`observe` 允许你在它们改变后拦截改变。

## Intercept
用法: `intercept(target, propertyName?, interceptor)`

* `target`: 监听的 observable
* `propertyName`: 可选参数，为了指定拦截的特定属性。注意 `intercept(user.name, interceptor)` 与 `intercept(user, "name", interceptor)` 本质上是不同的。第一个试图向 `user.name` 中的 _当前_ 值添加拦截器（它可能完全不是一个 observable），后一个对 `user` 属性 `name` 的改变进行拦截。 
* `interceptor`: 对于监听的 observable 的 *每一个* 改变都会使 callback 被调用。接收描述突变的单个对象。

`intercept` 应该告诉 MobX 当前改变需要发生什么。
因此它应该执行以下事项：
1. 该方法中，接收的 `change` 被返回，在这种情况下改变将会被应用。
2. 修改 `change` 对象，比如规范化数据，并返回它。不是所有的字段都可改变，见下。
3. 返回 `null`，这表明这个突变可以被忽略且不该被应用。这是一个强大的概念去操作你的对象，例如临时不可变。
4. 抛出异常，例如某些不变量未被满足。

这个方法返回一个 `disposer` 方法用于将被调用的拦截器取消。
可能在多个 observable 上注册了多个拦截器。
它们将按照注册顺序进行排列。
当其中一个拦截器返回 `null` 或者抛出一个异常，其他拦截器将不会再运行。
可以在对象和其一个单独的属性均注册了拦截器。
在这种情况下，对象拦截器会在属性拦截器之前先运行。

```javascript
const theme = observable({
  backgroundColor: "#ffffff"
})

const disposer = intercept(theme, "backgroundColor", change => {
  if (!change.newValue) {
    // 忽略企图取消设置背景色
    return null;
  }
  if (change.newValue.length === 6) {
    // 修正丢失的前缀 “#”
    change.newValue = '#' + change.newValue;
    return change;
  }
  if (change.newValue.length === 7) {
      // 必须是正确格式的颜色代码
      return change;
  }
  if (change.newValue.length > 10) disposer(); // 停止未来的拦截变化
  throw new Error("This doesn't like a color at all: " + change.newValue);
})
```

## Observe
用法: `observe(target, propertyName?, listener, invokeImmediately?)`

* `target`: 观察 observable
* `propertyName`: 可选的参数，指定要观察的特定属性。注意 `observe(user.name, listener)` 与 `observe(user, "name", listener)` 本质上是不同的。第一个观察 `user.name` 中的 _当前_ 值（它可能完全不是一个 observable），后一个是观察 `user` 的属性 `name`。
* `listener`: 对于监听的 observable 的 *每一个* 改变都会使 callback 被调用。接收描述突变的单个对象，除了 boxed observable，将会调用 `listener` 的两个参数：`newValue, oldValue`。
* `invokeImmediately`：默认 false。设置为 true，如果你想要 `observe` 去立即调用 `listener` 中 observable 的状态（而不是等待第一次改变）。暂不支持所有类型的 observable。

该方法返回一个 `disposer` 方法用于取消观察者（observer）。
注意 `transaction` 不影响 `observe` 的工作方式。
这意味着在事务中，`observe` 对于每一次突变将会触发监听器。
因此 `autorun` 经常是更加强大和 `observe` 的有效替代。

_当发生突变时，`observe` 随改变做出响应，就像 `autorun` 或 `reaction` 对可用的新值做出响应，多数情况下后者是够用的_

Example:

```javascript
import {observable, observe} from 'mobx';

const person = observable({
	firstName: "Maarten",
	lastName: "Luther"
});

const disposer = observe(person, (change) => {
	console.log(change.type, change.name, "from", change.oldValue, "to", change.object[change.name]);
});

person.firstName =  "Martin";
// 输出: 'update firstName from Maarten to Martin'

disposer();
// 忽略之后任何的更新

// 观察单个字段
const disposer2 = observe(person, "lastName", (change) => {
	console.log("LastName changed to ", change.newValue);
});
```
相关博客: [Object.observe is dead. Long live MobX.observe](https://medium.com/@mweststrate/object-observe-is-dead-long-live-mobservable-observe-ad96930140c5)

## Event overview

`intercept` 与 `observe` 的回调会接收一个至少具有以下属性的事件对象：
* `object`: 触发事件的 observable
* `type`: (string) 当前事件的类型

这些是每种类型可用的其他字段：

| observable 类型 | event 类型 | 属性 | 描述 | 在 intercept 中可用 | 在 intercept 中可被修改 |
| -- | --- | ---| --| --| -- |
| Object | add | name | name of the property being added | √ | |
| | | newValue | the new value being assigned | √ | √ |
| | update\* | name | name of the property being updated | √ |  |
| | | newValue | the new value being assigned | √ | √ |
| | | oldValue | the value that is replaced |  |  |
| Array | splice | index | starting index of the splice. Splices are also fired by `push`, `unshift`, `replace` etc. | √ | |
| | | removedCount | amount of items being removed | √ | √ |
| | | added | array with items being added | √ | √ |
| | | removed | array with items that where removed | | |
| | | addCount | amount of items that where added | | |
| | update | index | index of the single entry that is being updated | √ | |
| | | newValue | the newValue that is / will be assigned | √ | √ |
| | | oldValue | the old value that was replaced | | |
| Map | add | name | the name of the entry that was added | √ | |
| | | newValue | the new value that is being assigned | √ | √ |
| | update | name | the name of the entry that is being updated | √ | |
| | | newValue | the new value that is being assigned | √ | √ |
| | | oldValue | the value that has been replaced | | |
| | delete | name | the name of the entry that is being removed | √ | |
| | | oldValue | the value of the entry that was removed | | |
| Boxed & computed observables | create | newValue | the value that was assigned during creation. Only available as `spy` event for boxed observables | | |
| | update | newValue | the new value being assigned | √ | √ |
| | | oldValue | the previous value of the observable | | |

_\* 对于已被更新的 computated values，`update` 事件不会触发（因为它们不是突变的）。但是可以通过使用 `observe(object, 'computedPropertyName', listener)` 显式订阅特定的属性来进行观察._