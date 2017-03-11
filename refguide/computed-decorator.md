# (@)computed


计算值是通过当前状态或者其他计算值衍生出来的。理论上，它们和Excel中的公式非常类似。
请重视计算值，它们会帮助你减少需要修改的状态。因为它是经过高度优化的，所以请尽可能地使用他们

无需为 `computed` 和 `autorun`的使用感到困惑，虽然他们都是一种状态改变的响应。
`computed` 是你希望生成一个新的*值*时所使用的。
`autorun` 则是希望产生一些副作用，例如记录日志，发送请求等。

计算值是通过所有会影响它的状态所自动衍生的。计算值在大部分情况下都是可以优化的，所以应尽可能地使用纯函数（同样输入永远产生同样输出）。例如一个计算值不会重新执行，只要它所依赖的状态没有变化。如果一个计算值没有被其他计算值所使用或者被观察，则也不会重新执行。

这一套机制非常方便，甚至当一个计算值不再使用时，MobX可以自动进行垃圾回收。这和你必须手动处理的`autorun`不同

有些时候，这会使刚开始使用MobX的人们感到疑惑，如果你创建了一个计算值，但在任何地方都没有使用，则计算值不会缓存它的值或重计算。如果你希望强制计算值更新，你可以使用[`observe`](observe.md) 或者 [`keepAlive`](https://github.com/MobXjs/MobX-utils#keepalive).

Note that `computed` properties are not enumerable. Nor can they be overwritten in an inheritance chain.

注意 `computed` 是不可遍历的，也不能被继承链覆盖

## `@computed`

如果你 [能使用装饰器](../best/decorators.md) 你可以在任何类属性getter使用 `@computed` decorator来声明这是一个计算值。

```javascript
import {observable, computed} from "MobX";

class OrderLine {
    @observable price = 0;
    @observable amount = 1;

    constructor(price) {
        this.price = price;
    }

    @computed get total() {
        return this.price * this.amount;
    }
}
```

## `computed` 修改方法

如果你的环境不支持装饰器。请和`extendObservable` / `observable` 一起，使用`computed(expression)` 修改方法给可观察对象增加一个新的计算值。

`@computed get propertyName() { }` 只是类创建时调用 [`extendObservable(this, { propertyName: get func() { } })`](extend-observable.md) 的一个基本语法糖.

```javascript
import {extendObservable, computed} from "MobX";

class OrderLine {
    constructor(price) {
        extendObservable(this, {
            price: price,
            amount: 1,
            // valid:
            get total() {
                return this.price * this.amount
            },
            // also valid:
            total: computed(function() {
                return this.price * this.amount
            })
        })
    }
}
```

## Setters 计算值

如果需要创建一个setter类型的计算值，注意这些setter不能直接改变计算值的值。但他们可用于衍生的相反用途，例如：

```javascript
const box = observable({
    length: 2,
    get squared() {
        return this.length * this.length;
    },
    set squared(value) {
        this.length = Math.sqrt(value);
    }
});
```

或者类似的

```javascript
class Foo {
    @observable length: 2,
    @computed get squared() {
        return this.length * this.length;
    }
    set squared(value) { //this is automatically an action, no annotation necessary
        this.length = Math.sqrt(value);
    }
}
```

_注意：永远在getter之后使用setter，否则一些TypeScript的版本会将同样的命名声明为两个属性_

_注意: setters 需要 MobX 2.5.1 以上版本_

## `computed(expression)` 作为函数


`computed` 可以像函数一样被调用。就像 `observable.box(primitive value)` 创建了一个独立的被观察者。
使用 `.get()` 获取当前的计算值，或者使用`.observe(callback)` 观察它的变化。
这个`computed` 的形式并不常使用，但是在一些你需要传递一个"boxed"的计算值时，可能会非常有用。

例如:

```javascript
import {observable, computed} from "MobX";
var name = observable("John");

var upperCaseName = computed(() =>
	name.get().toUpperCase()
);

var disposer = upperCaseName.observe(change => console.log(change.newValue));

name.set("Dave");
// prints: 'DAVE'
```

## `computed`的选项
当将`computed`作为一个修改器或者box使用时，它接受第二个配置选项，你可以传入下面这些选项参数。

* `name`: 字符型, debug时devtools或spy使用的debug name。
* `context`: 上下文
* `setter`: 这个参数是需要传入的。如果没有setter函数，则不能赋予计算值一个新的值。如果第二个配置选项传入一个函数，则被认为是setter函数。
* `compareStructural`: 默认值为 `false`. When true, the output of the expression is structurally compared with the previous value before any observer is notified about a change. This makes sure that observers of the computation don't re-evaluate if new structures are returned that are structurally equal to the original ones. This is very useful when working with point, vector or color structures for example.

当设置为true时，`computed`函数在输出之前会比较当前值与之前值结构上是否有变化。例如当将a.sender替换为a.cloneSender时，依旧会触发`computed`变化，当在一些点、向量或者颜色结构的场景下非常有用。

## `@computed.struct` 用于结构比较

`@computed` 不带入任何变量，如果你想创建一个结构比较的计算值，请使用`@computed.struct`


## 注意错误处理
如果一个计算值在计算的过程中抛出一个错误，这个错误会被捕捉并且在被查阅时抛出。抛出错误并不中断追踪，所以可以从报错中恢复运行。

例如:

```javascript
const x = observable(3)
const y = observable(1)
const divided = computed(() => {
    if (y.get() === 0)
        throw new Error("Division by zero")
    return x.get() / y.get()
})

divided.get() // returns 3

y.set(0) // OK
divided.get() // Throws: Division by zero
divided.get() // Throws: Division by zero

y.set(2)
divided.get() // Recovered; Returns 1.5
```
