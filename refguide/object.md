## Observable Objects

如果一个纯 JavaScript 对象传递给`observable`，其包含的所有的属性都将被观察（纯对象就是不通过构造函数创造的对象）。默认情况下，`observable`是递归调用的，所以如果遇到一个值是对象或数组的情况，它的值也会传递给`observable`。

```javascript
import {observable, autorun, action} from "MobX";

var person = observable({
    // observable properties:
	name: "John",
	age: 42,
	showAge: false,

    // computed property:
	get labelText() {
		return this.showAge ? `${this.name} (age: ${this.age})` : this.name;
	},

    // action:
    setAge: action(function() {
        this.age = 21;
    })
});

// object properties don't expose an 'observe' method,
// but don't worry, 'MobX.autorun' is even more powerful
autorun(() => console.log(person.labelText));

person.name = "Dave";
// prints: 'Dave'

person.setAge(21);
// etc
```

在使一个对象可观察时需要注意一些事情：

* 当通过使用`observable`传递对象时，在使对象变得可观察时，只有当时存在的属性才会被观察。后续在添加到对象中的属性是不会被观察的，除非使用[`extendObservable`](extend-observable.md)
* 只有纯对象才能被观察。对于非纯对象，其构造函数负责初始化其可被观察的属性。使用[`@observable`](observable.md)装饰或 [`extendObservable`](extend-observable.md)函数。
* getters 属性将会自动转换为衍生属性，就像[`@computed`](computed-decorator)做的事是一样的。
* `observable`自动的被递归应用于整个对象结构，在实例化和任何将来会被分配给可观察属性的新值。Observable 不会递归到非纯对象里。
* 默认情况下，95%的情况都可以运行良好，但是对于哪些属性应该细粒度地被观察，请参阅 [modifiers](modifiers.md) 章节。

# `observable.object(props)` & `observable.shallowObject(props)`

`observable(object)` 只是`observable.object(props)`的一个缩略写法。所有的属性都会被深度转换为可观察的。
[modifiers](modifiers.md) 可以重写这一行为。
`shallowObject(props)` 用于确保属性只是浅观察的。意思是引用属性是可观察的，但引用属性对应的值的内容并不是可观察的。

## Name 参数

`observable.object` and `observable.shallowObject` 都获取第二个参数作为debug name，给开发工具使用。
