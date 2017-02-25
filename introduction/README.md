# MobX {#mobx}

简单、高扩展的状态管理层

[![Build Status](https://travis-ci.org/mobxjs/mobx.svg?branch=master)](https://travis-ci.org/mobxjs/mobx) [![Coverage Status](https://coveralls.io/repos/mobxjs/mobx/badge.svg?branch=master&service=github)](https://coveralls.io/github/mobxjs/mobx?branch=master) [![Join the chat at https://gitter.im/mobxjs/mobx](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/mobxjs/mobx?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

![npm install mobx](https://nodei.co/npm/mobx.png?downloadRank=true&downloads=true)

* 安装: `npm install mobx --save`。React绑定打包: `npm install mobx-react --save`。如果要使用EmcaScript下一代语法装饰器，请见下文
* CDN: [https://unpkg.com/mobx/lib/mobx.umd.js](https://unpkg.com/mobx/lib/mobx.umd.js)


## 相关教程

*   [十分钟上手Mobx和React（Ten minute, interactive MobX + React tutorial）](https://mobxjs.github.io/mobx/getting-started.html)
*   [Official documentation and API overview](https://mobxjs.github.io/mobx/refguide/api.html)
*   Videos:
    *   [Egghead.io course: Manage Complex State in React Apps with MobX](https://egghead.io/courses/manage-complex-state-in-react-apps-with-mobx) - 30m.
    *   [ReactNext 2016: Real World MobX](https://www.youtube.com/watch?v=Aws40KOx90U) - 40m [slides](https://docs.google.com/presentation/d/1DrI6Hc2xIPTLBkfNH8YczOcPXQTOaCIcDESdyVfG_bE/edit?usp=sharing)
    *   [Mobx与React实战](https://www.youtube.com/watch?v=XGwuM_u7UeQ). In depth introduction and explanation to MobX and React by Matt Ruby on OpenSourceNorth (ES5 only) - 42m.
    *   LearnCode.academy MobX tutorial [Part I: MobX + React is AWESOME (7m)](https://www.youtube.com/watch?v=_q50BXqkAfI) [Part II: Computed Values and Nested/Referenced Observables (12m.)](https://www.youtube.com/watch?v=nYvNqKrl69s)
    *   [Screencast: intro to MobX](https://www.youtube.com/watch?v=K8dr8BMU7-8) - 8m
    *   [Talk: State Management Is Easy, React Amsterdam 2016 conf](https://www.youtube.com/watch?v=ApmSsu3qnf0&feature=youtu.be) ([slides](https://speakerdeck.com/mweststrate/state-management-is-easy-introduction-to-mobx))
*   [Boilerplates and related projects](http://mobxjs.github.io/mobx/faq/boilerplates.html)
*   More tutorials, blogs and videos can be found on the [MobX homepage](http://mobxjs.github.io/mobx/faq/blogs.html)


## 介绍 
Mobx 是一个让状态管理（state management）变得简单、具有高扩展性的库，以对开发者透明的函数响应式编程方式，并且这个库经过了严格的测试。

Mobx的思想非常简单：

_任何事情都源于应用的状态,并且这一过程应该是自动的_
_Anything that can be derived from the application state, should be derived. Automatically._

包括UI、数据变更、与服务器通信等等

![MobX unidirectional flow](https://mobx.js.org/docs/flow.png)

React和Mobx 一起使用具有强大的联动效果。
React渲染应用的状态，通过一套将状态传递到已渲染的组件树中的机制。
Mobx则提供了存储和更新应用状态的机制。

React和Mobx都提供了非常优秀、独特的方式以解决应用开发中遇到的共同问题。
React提供了优异的Virtual DOM的处理机制以减少DOM操作成本。
Mobx则提供了优化将应用状态同步到React组件内的机制，通过使用了一种响应式的state依赖图，该依赖图严格只在需要的时候更新，并且不会出现代码腐化。


## 核心概念 

Mobx只有很少的核心概念，可以通过下面这些在线demo进行尝试。
[JSFiddle](https://jsfiddle.net/mweststrate/wv3yopo0/) 
(or [without ES6 and JSX](https://jsfiddle.net/rubyred/55oc981v/)).

### 可观察的状态（observable state）
Mobx 给已有的数据结构增加了可观察的能力（如对象、数组、类实例等）。你只需要很简单地使用[@observable](http://mobxjs.github.io/mobx/refguide/observable-decorator.html) 装饰你的类属性即可。

```
class Todo {
    id = Math.random();
    @observable title = "";
    @observable finished = false;
}

```

使用 `observable` 就像将对象中的属性转变成Excel中的单元格，但不同的是，这些值并不仅限于初级类型，也可作用于引用、对象、数组等类型。你甚至可以[定义你自己的](http://mobxjs.github.io/mobx/refguide/extending.html) 可观察数据结构。

### 幕间: 在 ES5, ES6 and ES.next 环境中使用Mobx

如果 `@` 对你来说像一个外星生物，不要害怕，这是ES下一代的装饰器预发。对于它的使用完全是可选的。看这个[文档](http://mobxjs.github.io/mobx/best/decorators.html) 来获取更多的信息，来决定使用或者完全不使用它们。对于ES下一代的特性，如装饰器，仅仅是锦上添花的事情。

例如，上述例子用ES5的语法可以写成这样

```
function Todo() {
    this.id = Math.random()
    extendObservable(this, {
        title: "",
        finished: false
    })
}

```

### 计算属性（Computed values）

使用Mobx，你可以很容易的定义计算属性，当相关数据变化时，计算属性的值会自动发生变化。
通过使用 [`@computed`](http://mobxjs.github.io/mobx/refguide/computed-decorator.html) 装饰器声明，也可以通过 `(extend)Observable` 这个方法进行声明。

```
class TodoList {
    @observable todos = [];
    @computed get unfinishedTodoCount() {
        return this.todos.filter(todo => !todo.finished).length;
    }
}

```

Mobx 会确保当一个todo增加或者当`finished`改变时， `unfinishedTodoCount` 这个方法返回的值是自动更新的。
这个更新永远是自动的，并且只在需要的时候。

### 响应行为（Reactions）

响应行为和计算属性类似，但是与产生一个新的值不同，一个响应行为产生一系列副作用（side effect），例如输出console、发出网络请求、更新React组件等等。简而言之，响应行为是[响应式编程](https://en.wikipedia.org/wiki/Reactive_programming)和[命令式编程](https://en.wikipedia.org/wiki/Imperative_programming)的一座桥梁

#### React components {#react-components}

如果你使用React，你只需增加 [`observer`](http://mobxjs.github.io/mobx/refguide/observer-component.html)装饰器，即可将你的无状态组件转换为可响应组件。

注意，这个函数/装饰器来源于 `mobx-react` 这个包

```
import React, {Component} from 'react';
import ReactDOM from 'react-dom';
import {observer} from "mobx-react";

@observer
class TodoListView extends Component {
    render() {
        return <div>
            <ul>
                {this.props.todoList.todos.map(todo =>
                    <TodoView todo={todo} key={todo.id} />
                )}
            </ul>
            Tasks left: {this.props.todoList.unfinishedTodoCount}
        </div>
    }
}

const TodoView = observer(({todo}) =>
    <li>
        <input
            type="checkbox"
            checked={todo.finished}
            onClick={() => todo.finished = !todo.finished}
        />{todo.title}
    </li>
)

const store = new TodoList();
ReactDOM.render(<TodoListView todoList={store} />, document.getElementById('mount'));

```

`observer` 将React组件与他们用于渲染的数据关联上。当使用Mobx时，没有自动更新组件（smart）或者不更新组件（dumb）的概念。所有的组件都自动更新，但是却有一些『惰性』 的行为。Mobx只在该组件确实需要时，触发其重渲染。所以上面的例子中，`onClick` 方法会触发 `TodoView` 的重渲染。但如果你移除 `Tasks left` 这一行（或者将其放到一个独立的组件中），则`TodoListView` 不会再重渲染。你可以通过以下例子尝试[JSFiddle](https://jsfiddle.net/mweststrate/wv3yopo0/).


#### 自定义响应行为 {#custom-reactions}

自定义响应行为可以非常简单地创建，请根据你的实际情况适用以下3个函数：
[`autorun`](http://mobxjs.github.io/mobx/refguide/autorun.html), 
[`autorunAsync`](http://mobxjs.github.io/mobx/refguide/autorun-async.html) 或者
[`when`](http://mobxjs.github.io/mobx/refguide/when.html) 

下面这个例子，当`unfinishedTodoCount` 发生改变时，autorun会自动执行

```
autorun(() => {
    console.log("Tasks left: " + todos.unfinishedTodoCount)
})

```

### 什么情况下Mobx会触发响应？ {#what-will-mobx-react-to}

为什么当`unfinishedTodoCount`每次发生改变时，都会打印出一条新的记录？
答案是：
_Mobx中，在已被标记的函数执行过程中，如果任一已被观察的属性被引用时，这个被观察的属性会触发响应_
_MobX reacts to any existing observable property that is read during the execution of a tracked function._

为了深入地解释Mobx如何检测被观察对象，及什么时候会触发响应，请看[understanding what MobX reacts to](https://github.com/mobxjs/mobx/blob/gh-pages/docs/best/react.md)

### 行为（action）

与其他flux框架不同，Mobx 并不强制如何处理用户的行为
* 可以用类似于Flux的行为处理
* 也可以用RxJS处理事件
* 也可以用onClick等直接的方式

总的来说，上述方式都是为了处理这个问题：以某种方式更新状态。

在更新状态之后，Mobx会小心、高效地处理剩下的问题。所以像下面这种一个简单的状态，完全可以自动更新。
不需要通过dispatcher等来触发事件。一个React组件就是你状态的最好描述。

```
store.todos.push(
    new Todo("Get Coffee"),
    new Todo("Write simpler code")
);
store.todos[0].finished = true;

```
尽管如此，Mobx也还是有一个可选的概念，[`actions`](https://mobxjs.github.io/mobx/refguide/action.html)。
使用它们的好处：它们帮助你更好地明确你的代码结构，以明确在合适的时间与地方来修改应用状态。


## 再聊聊简单与可扩展性（Simple and scalable） 

MobX is one of the least obtrusive libraries you can use for state management. That makes the `MobX` approach not just simple, but very scalable as well:

### Using classes and real references {#using-classes-and-real-references}

With MobX you don&#039;t need to normalize your data. This makes the library very suitable for very complex domain models (At Mendix for example ~500 different domain classes in a single application).

### Referential integrity is guaranteed {#referential-integrity-is-guaranteed}

Since data doesn&#039;t need to be normalized, and MobX automatically tracks the relations between state and derivations, you get referential integrity for free. Rendering something that is accessed through three levels of indirection?

No problem, MobX will track them and re-render whenever one of the references changes. As a result staleness bugs are a thing of the past. As a programmer you might forget that changing some data might influence a seemingly unrelated component in a corner case. MobX won&#039;t forget.

### Simpler actions are easier to maintain {#simpler-actions-are-easier-to-maintain}

As demonstrated above, modifying state when using MobX is very straightforward. You simply write down your intentions. MobX will take care of the rest.

### Fine grained observability is efficient {#fine-grained-observability-is-efficient}

MobX builds a graph of all the derivations in your application to find the least number of re-computations that is needed to prevent staleness. &quot;Derive everything&quot; might sound expensive, MobX builds a virtual derivation graph to minimize the number of recomputations needed to keep derivations in sync with the state.

In fact, when testing MobX at Mendix we found out that using this library to track the relations in our code is often a lot more efficient than pushing changes through our application by using handwritten events or &quot;smart&quot; selector based container components.

The simple reason is that MobX will establish far more fine grained &#039;listeners&#039; on your data than you would do as a programmer.

Secondly MobX sees the causality between derivations so it can order them in such a way that no derivation has to run twice or introduces a glitch.

How that works? See this [in-depth explanation of MobX](https://medium.com/@mweststrate/becoming-fully-reactive-an-in-depth-explanation-of-mobservable-55995262a254).

### Easy interoperability {#easy-interoperability}

MobX works with plain javascript structures. Due to its unobtrusiveness it works with most javascript libraries out of the box, without needing MobX specific library flavors.

So you can simply keep using your existing router, data fetching and utility libraries like `react-router`, `director`, `superagent`, `lodash` etc.

For the same reason you can use it out of the box both server- and client side, in isomorphic applications and with react-native.

The result of this is that you often need to learn fewer new concepts when using MobX in comparison to other state management solutions.

<center>![](https://www.mendix.com/styleguide/img/logo-mendix.png) __MobX is proudly used in mission critical systems at [Mendix](https://www.mendix.com)__</center>