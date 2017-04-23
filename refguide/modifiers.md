# observable 的修饰符

修饰符可以使用 decorator 或者对于特殊属性结合 `extendObservable` 与 `observable.object` 来改变自动转换规则。

* `observable.deep`: 默认修饰符，被使用于任何 observable，在尚未转化的情况下，它会将任何已分配，非原始的值转化为 observable。
* `observable.ref`: 无法自动进行 observable 的转化，用创建一个 observable 的引用来代替。
* `observable.shallow`:

 Can only used in combination with collections. Turns any assigned collection into an collection, which is shallowly observable (instead of deep)
* `computed`: 创建一个派生的属性, 详见 [`computed`](computed-decorator.md)
* `action`: 创建一个 action, 详见 [`action`](action.md)

## 深度可观察的

当 MobX 使用 `observable`, `observable.object`, 或 `extendObservable` 创建 observable 的对象时，它添加 observable 属性默认使用 `deep` 的修饰符。对于任何新分配的值，深度修饰符深度递归调用 `observable(newValue)`。
当不停得使用 `deep` 修饰符... 你懂的。

默认是十分方便的，不需要任何额外的开销，所有被分配给 observable 值的值将是 observable 的，所以不需要额外开销来使对象深度 observable。

## 引用可观察的

然而，在一些情况下，对象不需要转化为可观察的。
典型的例子是不可变的对象，或者是由第三方库来管理的对象。
例如：JSX 元素，DOM 元素，History、window等等原生对象。
对于这些对象，你就想存储一个引用，而不是将他们转化为 observable。

对于这些情况，`ref` 修饰符创建的 observable 属性只追踪引用，不会尝试转换值。
例如：

```javascript
class Message {
    @observable message = "Hello world"

    // 示例，如果 author 是不可变的，我们就只需要存储引用，不需要转化为一个可变的，observable 的对象
    // fictional example, if author is immutable, we just need to store a reference and shouldn't turn it into a mutable, observable object
    @observable.ref author = null
}
```

或者使用 ES5 的语法：

```javascript
function Message() {
    extendObservable({
        message: "Hello world",
        author: observable.ref(null)
    })
}
```

注意：一个 observable、box 的引用，使用 `const box = observable.shallowBox(value)` 创建。

## 浅可观察的

`observable.shallow` 修饰符应用于可观察的 '一级深度' 。如果你需要创建 observable 引用的 _集合_。
如果一个新的集合被分配为一个属性使用这个修饰符，它会是 observable，但它的值与 `deep` 不同，将保持原样，不会递归。
例如：

```javascript
class AuthorStore {
    @observable.shallow authors = []
}
```
在上述例子中，存储 authors 信息的普通数组分配给了属性 `authors`，更新 authors 将使用 observable 数组，这其中包括原始的，不可观察的 authors。

注意：可以使用以下方法来手动创建浅集合：`observable.shallowObject`, `observable.shallowArray`, `observable.shallowMap` 和 `extendShallowObservable`.

## Action & Computed

`action`, `action.bound`, `computed` 和 `computed.struct` 也可以使用修饰符。分别查阅 [`computed`](computed-decorator.md) 与 [`action`](action.md).

```javascript
const taskStore = observable({
    tasks: observable.shallow([]),
    taskCount: computed(function() {
        return this.tasks.length
    }),
    clearTasks: action.bound(function() {
        this.tasks.clear()
    })
})
```

## asStructure

MobX 2 具有 `asStructure` 修饰符，在实践中很少使用，或者仅在使用 `reference` / `shallow` 的情况下使用会比较合适（例如使用不可变数据）。
对于计算属性（computed）与反应（reaction）的结构比较仍是可能的。

## 修饰符的作用

```javascript
class Store {
    @observable/*.deep*/ collection1 = []

    @observable.ref collection2 = []

    @observable.shallow collection3 = []
}

const todos = [{ test: "value" }]
const store = new Store()

store.collection1 = todos;
store.collection2 = todos;
store.collection3 = todos;
```

在这些操作后：

1. `collection1 === todos` 是 false；todos 的内容将被克隆进入新的 observable 数组
2. `collection1[0] === todos[0]` 是 false；第一个 todo 是普通对象，它被克隆进入 observable 对象，存储于数组中
3. `collection2 === todos` 是 true；`todos` 保持原样，不是 observable 的。仅仅 `collection2` 属性本身是 observable 的
4. `collection2[0] === todos[0]` 是 true；因为第 3 点
5. `collection3 === todos` 是 false；collection 3 是一个新的 observable 数组
6. `collection3[0] === todos[0]` 是 true；`collection3` 的值仅仅浅转换为 observable，但数组的内容维持原样
