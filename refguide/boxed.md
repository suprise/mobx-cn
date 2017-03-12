## 原始类型和引用


JavaScript中的所有原始数据都是不可变的，因此根据此定义这些是不可被观察的。通常来说这没问题，因为MobX通常只是把包含值的 _属性_ 变成可被观察的，详见 [observable objects](object.md)。在特殊情况下，它可以方便的使一个不被对象拥有的"原始"数据可被观察。对于这些情况，我们可以创建一个可被观察的box去管理。


### `observable.box(value)`

所以 `observable.box(value)` 接受任何值，并将其存储在box内，这个值可以使用 `.get()` 获取，使用 `.set(newValue)` 更新。
甚至你可以注册一个回调，使用它的 `.observe` 方法以监听存储的数据变化。
但自从MobX自动追踪box的任何变化，在大多数情况下最好的方法是使用 [`MobX.autorun`](autorun.md) 来代替上述方法。


所以 `observable(scalar)` 返回的对象签名是：
* `.get()` 返回当前值
* `.set(value)` 替换当前存储的值，并通知所有的观察者。
* `intercept(interceptor)` 可以在应用之前拦截更改，参阅 [observe & intercept](observe.md)
* `.observe(callback: (newValue, previousValue) => void, fireImmediately = false): disposerFunction` 注册一个存储的值每次替换时都会触发的观察者函数，其返回一个函数以取消该观察。请参阅 [observe & intercept](observe.md)


### `observable.shallowBox(value)`

`shallowBox` 创建一个基于 [`ref`](modifiers.md) 修改器的box。 这意味着任何box内的值不会被自动转换为可观察的。 


### `observable(primitiveValue)`

当使用普通的 `observable(value)` 方法，MobX会自动创建box。


### 例如

```javascript
import {observable} from "MobX";

const cityName = observable("Vienna");

console.log(cityName.get());
// prints 'Vienna'

cityName.observe(function(change) {
	console.log(change.oldValue, "->", change.newValue);
});

cityName.set("Amsterdam");
// prints 'Vienna -> Amsterdam'
```

Array Example:

```javascript
import {observable} from "MobX";

const myArray = ["Vienna"];
const cityName = observable(myArray);

console.log(cityName[0]);
// prints 'Vienna'

cityName.observe(function(observedArray) {
	if (observedArray.type === "update") {
		console.log(observedArray.oldValue + "->" + observedArray.newValue);
	} else if (observedArray.type === "splice") {
		if (observedArray.addedCount > 0) {
			console.log(observedArray.added + " added");
		}
		if (observedArray.removedCount > 0) {
			console.log(observedArray.removed + " removed");
		}
	}
});

cityName[0] = "Amsterdam";
// prints 'Vienna -> Amsterdam'

cityName[1] = "Cleveland";
// prints 'Cleveland added'

cityName.splice(0, 1);
// prints 'Amsterdam removed'
```

## Name 参数

`observable.box` and `observable.shallowBox` 都获取第二个参数作为debug name，给开发工具使用。

