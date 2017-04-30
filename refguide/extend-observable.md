# extendObservable

与 `Object.assign` 类似，`extendObservable` 可以接受两个甚至更多参数，一个 `目标` 对象和一个或多个 `属性` map。
它将所有属性中的键值对添加到 `目标` 对象中作为 observable 属性。

```javascript
var Person = function(firstName, lastName) {
	// 在一个新的实例，初始化 observable 属性
	extendObservable(this, {
		firstName: firstName,
		lastName: lastName
	});
}

var matthew = new Person("Matthew", "Henry");

// 向已存在的 observable 对象，增加一个 observable 属性
extendObservable(matthew, {
	age: 353
});
```

注:  `observable.object(object)` 实际上是 `extendObservable({}, object)`.

注意，属性 map 不总是逐一复制到目标对象上的，而被视为是属性描述符。
大多数值是维持原样的，但值会包含修饰符的会被特别处理。所有属性都具有 getter。

## 修饰符

[修饰符](modifiers.md) 被用来定义某些属性的特殊行为。

例如 `observable.ref` 建立 observable 引用，它不会自动将值转化为 observable，`computed` 引入了派生属性：

```javascript
var Person = function(firstName, lastName) {
	// 在一个新的实例，初始化 observable 属性
	extendObservable(this, {
		firstName: observable.ref(firstName),
		lastName: observable.ref(lastName),
		fullName: computed(function() {
			return this.firstName + " " + this.lastName
		})
	});
}
```

所有有效修饰符的介绍可以在 [修饰符](modifiers.md) 中寻找。

## Computed 属性

Computed 属性也可以通过使用 *getter* 函数编写。同时也可以设定 setter：

```javascript
var Person = function(firstName, lastName) {
	// 在一个新的实例，初始化 observable 属性
	extendObservable(this, {
		firstName: firstName,
		lastName: lastName,
		get fullName() {
			return this.firstName + " " + this.lastName
		},
		set fullName(newValue) {
			var parts = newValue.split(" ")
			this.firstName = parts[0]
			this.lastName = parts[1]
		}
	});
}
```

_注意: getter / setter 是有效的 ES5 语法，不需要转换器!_

## `extendShallowObservable`

`extendShallowObservable` 就像 `extendObservable`, 除了默认情况，属性将默认 *不* 自动将值转化为 observable。
因此就相当于 `extendObservable` 中每个属性调用了 `observable.ref` 修饰符。
注意：`observable.deep` 能使特定的属性自动回退（即转换到默认情况）
