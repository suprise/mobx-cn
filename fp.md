<img src="mobx.png" alt="logo" height="120" align="right" />

# MobX

_简单、高扩展的状态管理库_

[![Build Status](https://travis-ci.org/mobxjs/mobx.svg?branch=master)](https://travis-ci.org/mobxjs/mobx)
[![Coverage Status](https://coveralls.io/repos/mobxjs/mobx/badge.svg?branch=master&service=github)](https://coveralls.io/github/mobxjs/mobx?branch=master)
[![Join the chat at https://gitter.im/mobxjs/mobx](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/mobxjs/mobx?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Discuss MobX on Hashnode](https://hashnode.github.io/badges/mobx.svg)](https://hashnode.com/n/mobx)
[![Donate](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://mobxjs.github.io/mobx/donate.html)
[![OpenCollective](https://opencollective.com/mobx/backers/badge.svg)](#backers)
[![OpenCollective](https://opencollective.com/mobx/sponsors/badge.svg)](#sponsors)

![npm install mobx](https://nodei.co/npm/mobx.png?downloadRank=true&downloads=true)

* 安装: `npm install mobx --save`。配合React: `npm install mobx-react --save`。如果要使用 ECMAScript 下一代的语法：装饰器（可选），请见下文
* CDN:
- https://unpkg.com/mobx/lib/mobx.umd.js
- https://cdnjs.com/libraries/mobx

## 相关教程

*   [十分钟上手 Mobx 和 React（Ten minute, interactive MobX + React tutorial）](https://mobxjs.github.io/mobx/getting-started.html)
*   [官方文档与 API 概览](https://mobxjs.github.io/mobx/refguide/api.html)
*   视频:
    *   [Egghead.io course: Manage Complex State in React Apps with MobX](https://egghead.io/courses/manage-complex-state-in-react-apps-with-mobx) - 30m.
    *   [ReactNext 2016: Real World MobX](https://www.youtube.com/watch?v=Aws40KOx90U) - 40m [slides](https://docs.google.com/presentation/d/1DrI6Hc2xIPTLBkfNH8YczOcPXQTOaCIcDESdyVfG_bE/edit?usp=sharing)
    *   [MobX 与 React 实战](https://www.youtube.com/watch?v=XGwuM_u7UeQ). In depth introduction and explanation to MobX and React by Matt Ruby on OpenSourceNorth (ES5 only) - 42m.
    *   LearnCode.academy MobX tutorial [Part I: MobX + React is AWESOME (7m)](https://www.youtube.com/watch?v=_q50BXqkAfI) [Part II: Computed Values and Nested/Referenced Observables (12m.)](https://www.youtube.com/watch?v=nYvNqKrl69s)
    *   [Screencast: intro to MobX](https://www.youtube.com/watch?v=K8dr8BMU7-8) - 8m
    *   [Talk: State Management Is Easy, React Amsterdam 2016 conf](https://www.youtube.com/watch?v=ApmSsu3qnf0&feature=youtu.be) ([slides](https://speakerdeck.com/mweststrate/state-management-is-easy-introduction-to-mobx))
*   [项目模板与相关项目](http://mobxjs.github.io/mobx/faq/boilerplates.html)
*   更多的项目模板，博客与视频可在此获取 [MobX homepage](http://mobxjs.github.io/mobx/faq/blogs.html)

## 概述 

MobX 是一个通过对开发者透明的函数响应式编程（TFRP）方式，让状态管理（state management）变得简单、具有高扩展性的库，并且这个库经过了严格的测试。

MobX的思想非常简单：

_来源于应用的状态的任何事物，都能被自动获得_

_Anything that can be derived from the application state, should be derived. Automatically._

包括UI、数据变更、与服务器通信等等

<img alt="MobX unidirectional flow" src="flow.png" align="center" />

React 和 MobX 是一对强大的组合。
React 提供一个把应用程序的状态渲染成组件树的机制。
MobX 则提供了存储和更新应用状态的机制。

React 和 MobX 都提供了非常优秀、独特的方式以解决应用开发中遇到的共同问题。
React 利用虚拟 DOM 来优化 UI 的渲染，以减少 DOM 操作成本。
MobX 则提供了优化将应用状态同步到 React 组件内的机制，通过使用了一种响应式的状态依赖图，该依赖图严格只在需要的时候更新，并且不会出现代码腐化。

## 核心概念 

MobX 只有很少的核心概念，可以通过下面这些在线 demo 进行尝试。
[JSFiddle](https://jsfiddle.net/mweststrate/wv3yopo0/) 
(or [没有 ES6 与 JSX 的Demo](https://jsfiddle.net/rubyred/55oc981v/)).

### 可观察的状态（observable state）
MobX 给已有的数据结构增加了可观察的能力（如对象、数组、类实例等）。你只需要很简单地使用[@observable](http://mobxjs.github.io/mobx/refguide/observable-decorator.html) 装饰你的类属性（property）即可。

```javascript
class Todo {
    id = Math.random();
    @observable title = "";
    @observable finished = false;
}
```

使用 `observable` 就像将对象中的属性转变成 Excel 中的单元格，但不同的是，这些值并不仅限于初级类型，也可作用于引用、对象、数组等类型。你甚至可以 [定义你自己的](http://mobxjs.github.io/mobx/refguide/extending.html) 可观察数据结构。

### 幕间: 在 ES5/ES6/ES.next 环境中使用 MobX

如果 `@` 对你来说像一个外星生物，不要害怕，这是 ES 下一代的装饰器语法。对于它的使用完全是可选的。看这个 [文档](http://mobxjs.github.io/mobx/best/decorators.html) 获取更多的信息，来决定是否使用它们。MobX 可以在任何 ES5 的环境下运行，而利用ES下一代的特性，如装饰器，仅仅是锦上添花的事情。本文其余部分会使用装饰器，但请记住，他们是可选的。

例如，上述例子用ES5的语法可以写成这样:


```javascript
function Todo() {
    this.id = Math.random()
    extendObservable(this, {
        title: "",
        finished: false
    })
}
```

### 计算值（Computed values）

使用 MobX，你可以很容易的定义计算值，当相关数据变化时，计算值会自动发生变化。
计算值可以通过使用 [`@computed`](http://mobxjs.github.io/mobx/refguide/computed-decorator.html) 装饰器声明，也可以通过 `(extend)Observable` 配合 getter/setter 函数进行声明。


```javascript
class TodoList {
    @observable todos = [];
    @computed get unfinishedTodoCount() {
        return this.todos.filter(todo => !todo.finished).length;
    }
}
```

MobX 会确保当一个 todo 增加或者当 `finished` 改变时， `unfinishedTodoCount` 是自动更新的。
计算值类似于电子表格程序的公式。它们的更新永远是自动的，并且只在需要的时候更新。

### 响应行为（Reactions）

响应行为和计算值类似，但是与产生一个新的值不同，一个响应行为产生一系列副作用（side effect），例如输出 console、发出网络请求、增量更新 React 组件等等。简而言之，响应行为是 [响应式编程](https://en.wikipedia.org/wiki/Reactive_programming) 和 [命令式编程](https://en.wikipedia.org/wiki/Imperative_programming) 的一座桥梁

#### React 组件

如果你使用 React，你只需增加 [`observer`](http://mobxjs.github.io/mobx/refguide/observer-component.html) 装饰器，即可将你的无状态组件转换为可响应组件。

注意，这个函数/装饰器来源于 `mobx-react` 这个包。

```javascript
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

`observer` 将 React 组件与用于渲染的数据关联上。当使用 MobX 时，没有智能组件（smart）或者木偶组件（dumb）的概念。所有的组件都被智能地渲染，但是以木偶的方式进行定义。MobX 只在该组件确实需要时，触发其重渲染。所以上面的例子中，`onClick` 方法会触发 `TodoView` 的重渲染。如果没有完成的任务数量改变（`unfinishedTodoCount`），`TodoListView` 也将重渲染。但如果你移除 `Tasks left` 这一行（或者将其放到一个独立的组件中），当勾选 checkbox 时，`TodoListView` 不会再重渲染。你可以通过以下例子尝试 [JSFiddle](https://jsfiddle.net/mweststrate/wv3yopo0/).

#### 自定义响应行为 （custom-reactions）

自定义响应行为可以非常简单地创建，请根据你的实际情况适用以下3个函数：
[`autorun`](http://mobxjs.github.io/mobx/refguide/autorun.html), 
[`autorunAsync`](http://mobxjs.github.io/mobx/refguide/autorun-async.html) 或者
[`when`](http://mobxjs.github.io/mobx/refguide/when.html) 

下面这个例子，当 `unfinishedTodoCount` 发生改变时，autorun 会自动执行

```javascript
autorun(() => {
    console.log("Tasks left: " + todos.unfinishedTodoCount)
})
```

### 什么情况下MobX会触发响应（what-will-mobx-react-to）

为什么当 `unfinishedTodoCount` 每次发生改变时，都会打印出一条新的记录？
答案是：
_MobX中，在已被追踪的函数的执行过程中，如果任何已被观察的属性被使用时，这个被观察的属性会触发响应。_

_MobX reacts to any existing observable property that is read during the execution of a tracked function._

为了深入地解释Mobx如何检测被观察对象，及什么时候会触发响应，请看 [understanding what MobX reacts to](https://github.com/mobxjs/mobx/blob/gh-pages/docs/best/react.md)

### 行为（action）

与其他 flux 框架不同，Mobx 并不强制如何处理用户的行为：
* 可以用类似于 Flux 的行为处理
* 也可以用 RxJS 处理事件
* 也可以用 onClick 等直接的方式

总的来说，上述方式都是为了处理这个问题：以某种方式更新状态。

在更新状态之后，MobX 会高效、无干扰地处理剩下的问题。所以像下面这种一个简单的状态，完全可以自动更新用户界面，而不需要触发事件，调用 dispatcher 或者其他操作。一个 React 组件就是应用状态的最好描述。这个衍生由 MobX 来管理。

```javascript
store.todos.push(
    new Todo("Get Coffee"),
    new Todo("Write simpler code")
);
store.todos[0].finished = true;
```

尽管如此，MobX 也还是有一个可选的概念，[`actions`](https://mobxjs.github.io/mobx/refguide/action.html)。
使用它们的好处：它们帮助你更好地确定你的代码结构，以确保在合适的时间与地方来修改应用状态。

## 再聊聊简单与可扩展性（Simple and scalable） 

MobX 是上手成本最小的状态管理库之一，以下几点措施保证 `MobX` 不仅仅简单，还具有很好的可扩展性。

### 支持类（Class）和引用 （using-classes-and-real-references）

使用 MobX，你不需要重构你的数据结构，这使得这个库适用于拥有非常复杂领域模型的情况。在 Mendix 这一个应用中，有大概 500 个不同的类。

### 参照完整性是有保障的（Referential integrity is guaranteed）

因为数据不需要转换成标准格式，以及 MobX 会自动追踪状态和衍生状态之间的关系，可以获得很好的参照完整性。你有没有遇到过这样一种情况，为了渲染一些东西，需要经过 3 个层级的、方向不明的复杂状态传递？

没有问题，当其中的任何一个引用发生变化的时候，MobX 会追踪他们并且重渲染。因此原先的许多错误会成为历史。在某个边界情况中，作为程序员的你也许会忘记，改变某些数据可能会影响一个看上去不相关的组件，但 MobX 不会。

### 更加容易维护的简单行为 

就像上面的示例，使用 MobX 时改变状态是非常直观的，你只需要写下你的意图，Mobx会处理剩下的。

### 细粒度的观察能力会变得非常高效（Fine grained observability is efficient）

MobX 构建了一个关于你的应用的衍生关系图，并以成本最低的方式重新计算，避免了代码腐化。
“响应任何事情”，听起来非常昂贵，但 MobX 构建了一个虚拟的衍生关系图以最小化计算成本，并且保持衍生关系与状态同步。

实际上，当在 Mendix 这个应用中进行测试时，我们发现：当改变发生时，使用这个库追踪关系，比我们手写事件或是智能选择的容器组件来改变我们的应用，都更加高效。

最简单的原因是，MobX 构建了一种非常高效的数据监听机制，比你这个程序员做得更好。

其次 MobX 可以非常清晰地了解各个变化之间的关系，所以可以控制其不会重复响应或引入故障。

它是怎么做到的？看这里 [深入解读 MobX](https://medium.com/@mweststrate/becoming-fully-reactive-an-in-depth-explanation-of-mobservable-55995262a254).

### 高度可集成性（Easy interoperability）

MobX 基于普通的 javascript 结构，使得非常容易和其他库进行集成，开箱即用，MobX 没有特定代码风格。

你可以简单地保留你的路由库、数据获取库、工具库，例如 `react-router`, `director`, `superagent`, `lodash` 等等。

基于同样的理由，你可以在同构应用中用于服务端和客户端，也可用于 react-native。

所以上手成本很低，你完全不需要像其他状态管理库一样学习很多新概念。

![](https://www.mendix.com/styleguide/img/logo-mendix.png) 
__MobX is proudly used in mission critical systems at [Mendix](https://www.mendix.com)__

## 致谢 

MobX 的响应式编程思想来源于电子表格，同时受到 MVVM 框架的启发，例如MeteorJS tracker, knockout 和 Vue.js 等，但 MobX 使得透明函数响应式编程（TFRP）的发展更进一步，并提供了一种独立的实现。它实现的TFRP具有无干扰性、同步、可预见性、高效等特性。

对 [Mendix](https://github.com/mendix) 致以成吨的感谢，为支持 MobX 的维护，尤其是在一个真实的、复杂的、高性能要求的应用中证明了 MobX 的思想。

最后，荣誉属于那些相信、尝试、验证甚至 [贡献](https://github.com/mobxjs/mobx/blob/master/sponsors.md) MobX 的人们。

## 更多资源和文档 

*   [MobX 主页](http://mobxjs.github.io/mobx/faq/blogs.html)
*   [API 一览](http://mobxjs.github.io/mobx/refguide/api.html)
*   [教程、博客 & 视频](http://mobxjs.github.io/mobx/faq/blogs.html)
*   [模板](http://mobxjs.github.io/mobx/faq/boilerplates.html)
*   [相关项目](http://mobxjs.github.io/mobx/faq/related.html)

## 让我们听听元芳怎么说……

> After using #mobx for lone projects for a few weeks, it feels awesome to introduce it to the team. Time: 1/2, Fun: 2X

> Working with #mobx is basically a continuous loop of me going “this is way too simple, it definitely won’t work” only to be proven wrong

> Try react-mobx with es6 and you will love it so much that you will hug someone.

> I have built big apps with MobX already and comparing to the one before that which was using Redux, it is simpler to read and much easier to reason about.

> The #mobx is the way I always want things to be! It's really surprising simple and fast! Totally awesome! Don't miss it!

## 如何贡献

* 放轻松，先实现个小目标，提一个小 pull request。如果新增一些特性或者大的改动，请先在 Github 的 issue 中大家一起来讨论。
* 使用 `npm test` 测试基本用例，使用 `npm run coverage` 测试用例的代码覆盖率，使用 `npm run perf` 进行性能测试。

## Bower 支持

通过以下方式安装 `bower install https://unpkg.com/mobx/bower.zip`
然后使用 `lib/mobx.umd.js` 或者 `lib/mobx.umd.min.js`

## 捐赠

Mobx是否是你项目成功的关键因素？欢迎点击 [捐赠按钮](https://mobxjs.github.io/mobx/donate.html) 分享你的成功！
Mobx在很长一段发展的过程中保持了免费，对于任何回报我都万分感谢 :-）。如果留下你的名字，你会被加入 [贡献者](https://github.com/mobxjs/mobx/blob/master/sponsors.md) 的列表当中~:-）

## 翻译者捐赠

欢迎给翻译者捐赠，用于抚慰因为翻译被冷落的女朋友。
支付宝（Alipay）：

![](https://img.alicdn.com/tps/TB1Eo2DPFXXXXccaXXXXXXXXXXX-600-900.jpg_400x400.jpg)