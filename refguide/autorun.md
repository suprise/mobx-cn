# Autorun

`mobx.autorun` 用于你希望创建一个响应函数，而这个响应函数本身并不是观察者。 
这也是一个桥接命令式编程和响应式编程的常见场景。例如记录日志、持久化和UI更新。
一个简短的规则：如果你想使用一个自动运行函数，且并不想产生一个新值时，使用 `autorun`
如果`autorun`的第一个参数是一个字符串，则它会被当做一个debug name。


与 [`@observer` 装饰器/函数](./observer-component.md)类似, `autorun` 只会观察依赖的相关数据

```javascript
var numbers = observable([1,2,3]);
var sum = computed(() => numbers.reduce((a, b) => a + b, 0));

var disposer = autorun(() => console.log(sum.get()));
// prints '6'
numbers.push(4);
// prints '10'

disposer();
numbers.push(5);
// won't print anything, nor is `sum` re-evaluated
```

## 错误处理

任何autorun 或者其他类型的响应行为所抛出的错误，都会被捕捉并在控制台打印出来。但不会追踪到原本的触发代码，这是为了确保一个响应行为的异常，不会阻止其他很可能不相关的响应行为的继续运行。

可以重写原本的记录行为，通过调用`onError`传入一个错误处理。

例如:

```javascript
const age = observable(10)
const dispose = autorun(() => {
    if (age.get() < 0)
        throw new Error("Age should not be negative")
    console.log("Age", age.get())
})

age.set(18)  // Logs: Age 18
age.set(-10) // Logs "Error in reaction .... Age should not be negative
age.set(5)   // Recovered, logs Age 5

dispose.onError(e => {
    window.alert("Please enter a valid age")
})

age.set(-5)  // Shows alert box
```

一个全局的错误处理方法可以通过`extras.onReactionError(handler)`进行设置。这对于测试和监控非常有用。
A global onError handler can be set as well through `extras.onReactionError(handler)`. This can be useful in tests or for monitoring.
