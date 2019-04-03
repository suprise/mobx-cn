# observable

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
* 如果使用 `@observable` 装饰器，要确保在你的编译器（babel 或 typescript）中[启用装饰器语法](http://MobXjs.github.io/MobX/refguide/observable-decorator.html)
* 创建一个可观察的数据结构是具有*传染性*的。那意味着 `observable` 会自动将数据结构中所有的值一同转变为 `observable` 。这个行为可以通过 *修饰符（modifiers）* 或者 *浅观察（shallow）* 改变。


一些例子：

```javascript
const map = observable(asMap({ key: "value"}));
map.set("key", "new value");

const list = observable([1, 2, 4]);
list[2] = 3;

const person = observable({
    firstName: "Clive Staples",
    lastName: "Lewis"
});
person.firstName = "C.S.";

const temperature = observable(20);
temperature.set(25);
```
