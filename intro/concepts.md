# 概念 & 原则

## 概念

MobX 将会区分以下概念，你可以在之前的例子中已经见过，不过让我们钻得更深一些以了解更多的细节。

### 1. 状态（State）

_状态（State）_ 是驱动你应用的数据。
通常每个应用都会有自己独有的状态结构，就像一系列todo元素中，有一个视图状态来表示被选中的元素。
记住，状态就像Excel的单元格一样持有一份数据。

### 2. 衍生物（Derivation）

可以从状态中衍生出衍生物。
衍生物通常存在以下几种形式：

* UI视图
* 衍生数据，例如没完成的todo列表
* 后台交互，例如发送变动给服务器

MobX区分两种衍生物

* *计算属性（Computed values）* 这是一种可以将当前被观察的状态，通过纯函数衍生出的新值。
* *响应行为（Reactions）* 响应行为是当状态发生变化时会自动执行的副作用（side effects），它们是响应式和命令式两种编程方式的桥梁。通常用于获取I/O

当开始使用MobX时，人们喜欢大量使用响应行为。
黄金准则是：当你想要基于当前的状态创建一个新值时，使用计算属性 `computed`

类似Excel，公式计算是一种衍生物，用于*计算*某个值，而将这个公式显示到显示器上，则是一种GUI的响应行为。


### 3. 行为（action）

任何导致状态改变的代码称为行为。例如用户事件、后端数据推送等等
行为可以帮助你的代码结构变得更清晰。
如果使用严格模式，则MobX会强制只有在行为中才能改变状态。

## 原则

MobX支持单向数据流，但行为改变状态的时候，会更新所有被影响到的视图

![Action, State, View](../images/action-state-view.png)

* 所有的衍生物，在状态改变时都会自动地、最小粒度地更新。作为结果不会依赖任何中间形式的值。
* 所有的衍生物，默认更新都是同步的。举个例子，这意味着当状态改变后，行为可以直接检查（inspect）一个计算属性
* 计算属性的值是懒（lazily）更新的。任何计算属性都不会激活更新，直到它被一些副作用所使用。当一个视图不再使用它时，它会被自动回收
* 所有的计算属性需要是一个纯函数，不要用于改变状态。

## 举例

下面这个例子体现了上面的概念和原则

```javascript
import {observable, autorun} from 'MobX';

var todoStore = observable({
	/* some observable state */
	todos: [],

	/* a derived value */
	get completedCount() {
		return this.todos.filter(todo => todo.completed).length;
	}
});

/* a function that observes the state */
autorun(function() {
	console.log("Completed %d of %d items",
		todoStore.completedCount,
		todoStore.todos.length
	);
});

/* ..and some actions that modify the state */
todoStore.todos[0] = {
	title: "Take a walk",
	completed: false
};
// -> synchronously prints 'Completed 0 of 1 items'

todoStore.todos[0].completed = true;
// -> synchronously prints 'Completed 1 of 1 items'

```

在这个视频中 [10 minute introduction to MobX and React](https://MobXjs.github.io/MobX/getting-started.html) 你可以了解更多，并且用[React](https://facebook.github.io/react/) 建立一个UI视图。