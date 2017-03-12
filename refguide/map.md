# Observable Maps

`observable(asMap(values?, modifier?))` （和 `map(values?, modifier?)` ）创建一个具有动态键值的可观察Map。如果你不想要对特定的条目更改做出响应，而只是在添加和删除条目时使得这一行为可观察，maps是非常有用的。ES6 map可以使用对象、 数组或者字符串作为初始值。
而MobX Map 和 ES6 的maps不同，其只接受字符串作为键。

以下的方式是根据 [ES6 Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)规范提供的：

* `has(key)` 返回这个map是否含有提供的key。要注意key本身实际上是可被观察的。
* `set(key, value)` 设置提供的 `key` 的值。即使提供的key不存在，其在map中会被添加。
* `delete(key)` 在map中删除提供的key值
* `get(key)` 返回提供的key的值(可能为 `undefined` )，可以通过 `has` 确认你是否可以调用 `get`。
* `keys()` 返回map中的所有key，插入的顺序也是被保留的。
* `entries()` 返回一个包含对于map中每个键/值对的映射数组 `[key, value]` (有插入顺序)的数组。
* `forEach(callback: (value, key, map) => void, thisArg?)` 。对map中的任何一个键/值对进行调用回调。
* `clear()` 移除map中所有条目。
* `size` 返回map中的条目数量。

下面的方法不是 ES6 支持的，但是在MobX中是非常有用的：

* `toJS()` 返回该map的一个浅拷贝简单对象(对于深拷贝请使用`MobX.toJS(map))。
* `intercept(interceptor)` 注册一个在应用于map中任何改变之前会被触发的拦截器。请参阅 [observe & intercept](observe.md)。
* `observe(listener, fireImmdidately)` 注册一个当map中任何改变时被触发的监听者，类似 [Object.observe](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/observe)的触发事件。详情请参阅 [observe & intercept](observe.md)。
* `merge(object | map)` 复制提供的对象中的所有条目到这个map中。
* `replace(values)`。 使用 `values` 替换当前map的整个内容. 只是 `.clear().merge(values)` 的缩略方式。


## `observable.shallowMap(values)`


所有传给 [`observable`](observable.md) 的map中的值（主要是对象）都会被转变为可观察的。创建一个浅对象以禁止这种行为，请查阅 [modifiers](modifiers.md) 获取关于这一机制的更多细节。

## Name 参数

`observable.map` and `observable.shallowMap` 都获取第二个参数作为debug name，给开发工具使用。

