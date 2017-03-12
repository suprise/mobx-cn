## Observable Arrays

和对象相似，使用 `observable` 或 `observable.array(values?)` 也可以使数组变成可观察的，其还是通过递归工作实现的，所以所有（将来有）的数组值都是可被观察的。


```javascript
import {observable, autorun} from "MobX";

var todos = observable([
	{ title: "Spoil tea", completed: true },
	{ title: "Make coffee", completed: false }
]);

autorun(() => {
	console.log("Remaining:", todos
		.filter(todo => !todo.completed)
		.map(todo => todo.title)
		.join(", ")
	);
});
// 打印: 'Remaining: Make coffee'

todos[0].completed = false;
// 打印: 'Remaining: Spoil tea, Make coffee'

todos[2] = { title: 'Take a nap', completed: false };
// 打印: 'Remaining: Spoil tea, Make coffee, Take a nap'

todos.shift();
// 打印: 'Remaining: Make coffee, Take a nap'
```


由于 ES5 中原生数组的限制，`observable.array` 创建一个伪数组而非真实数组。在实践中，这些数组工作的和原生的数组一样好用，并且其支持所有原生方法，包括索引，甚至包括数组的长度。


请记住无论如何 `Array.isArray(observable([]))` 都将返回 `false` ，所以当你需要传递一个可观察的数组给外部库时，最好使用 `array.slice()` _创建一个其浅拷贝在传递给其它库或者内置函数_ 使用（这是最好的做法）。换句话说， `Array.isArray(observable([]).slice())` 将会返回 `true` 。

和函数内置实现的 `sort` 和 `reverse` 不同，observableArray.sort 和 reverse 不会改变数组的结构, 只会返回一个 sorted / reversed 的克隆。

除了所有的内置函数，下面的方法对可观察的数组来说也是非常有用的：

* `intercept(interceptor)` ，它可以作用于数组以拦截任何改变前的状态，具体请参阅 [observe & intercept](observe.md) 
* `observe(listener, fireImmediately? = false)` 监听数组的改变。其回调会在数组拼接或数组变化时接收参数 ：(。它将返回一个销毁函数以停止监听。
* `clear()` 从数组中删除 。
* `replace(newItems)` 用一个新的元素替换数组中所有存在的元素。
* `find(predicate: (item, index, array) => boolean, thisArg?), fromIndex?)` 基本上和ES7的 `Array.find` 提议相同，除了附加的 `fromIndex` 。
* `remove(value)` 按值移除数组中的单个元素。如果找到它们并移除时将会返回 `true` 。
* `peek()` 返回一个包含所有值的并可以安全的传递给其它库使用的数组，就像 `slice()` 一样。

和 `slice` 不同，`peek` 并不会创建一个新的副本。如果你确定要以只读方式使用数组，你可以在对对性能要求较高的应用中使用它。在性能关键部分，建议使用扁平的可观察数组。


## `observable.shallowArray(values)`

所有传给 [`observable`](observable.md) 的数组中的值（主要是对象）都会被转变为可观察的。创建一个浅数组以禁止这种行为，请查阅 [modifiers](modifiers.md) 获取关于这一机制的更多细节。


## Name 参数

`observable.array` and `observable.shallowArray` 都获取第二个参数作为debug name，给开发工具使用。

