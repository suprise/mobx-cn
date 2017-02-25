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

```
_任何事情都源于应用的状态,并且这一过程应该是自动的_

_Anything that can be derived from the application state, should be derived. Automatically._
```

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
```
_Mobx中，在已被标记的函数执行过程中，如果任一已被观察的属性被引用时，这个被观察的属性会触发响应_

_MobX reacts to any existing observable property that is read during the execution of a tracked function._

```

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

Mobx是上手成本最小的状态管理库之一，以下几点措施保证 `MobX` 不仅仅简单，还具有很好的可扩展性。

### 支持类（Class）和引用 {#using-classes-and-real-references}

使用Mobx，你不需要重构你的数据结构，这使得这个库适用于拥有非常复杂领域模型的情况。在Mendix这一个应用中，有大概500个不同的类。

### 参照完整性是有保障的（Referential integrity is guaranteed）

因为数据不需要重构，Mobx会自动追踪状态和衍生状态之间的关系，可以获得很好的参照完整性。你有没有遇到过这样一种情况，为了渲染一些东西，需要经过3个层级的、方向不明的复杂状态传递？

没有关系，Mobx会追踪他们并且重渲染，当其中的任何一个引用发生变化的时候。也许在某个边际情况中，你改变某个数据会影响一个看上去不相关的组件，但Mobx不会出现。


### 更加容易维护的简单行为 

就像上面的示例，使用Mobx时改变状态是非常直观的，你只需要写下你的意图，Mobx会处理剩下的。

### 合适地切分观察能力会变得非常高效（Fine grained observability is efficient）

Mobx构建了一个关于你的应用的衍生关系图，并以成本最低的方式重新计算。响应任何事情，听起来非常昂贵，但Mobx构建了一个虚拟的响应关系图以最小化计算成本，并且保持衍生关系与状态同步。

实际上，当在Mendix这个应用中进行测试时，我们发现当改变发生时，使用这个库区追踪关系，使得我们的代码非常高效。
最简单的原因是，Mobx构建了一种非常高效的数据监听机制，比你这个程序员做得更好。

因为Mobx可以非常清晰地了解各个变化之间的关系，所以可以控制其不会重复变化或引入故障。

它是怎么做到的？看这里 [in-depth explanation of MobX](https://medium.com/@mweststrate/becoming-fully-reactive-an-in-depth-explanation-of-mobservable-55995262a254).

### 高度可集成性

Mobx 基于普通的javascript结构，使得非常容易和其他库进行集成。
你可以简单地保留你的路由库、数据获取库、工具库，例如 `react-router`, `director`, `superagent`, `lodash` 等等。
基于同样的理由，你可以用于服务端或客户端当中。
所以上手成本很低，你完全不需要像其他状态管理库一样学习很多新概念。

![](https://www.mendix.com/styleguide/img/logo-mendix.png) 
__MobX is proudly used in mission critical systems at [Mendix](https://www.mendix.com)__

## 致谢 

Mobx 的响应式编程思想受到MVVM框架的启发，例如MeteorJS tracker, knockout 和 Vue.js等，但Mobx使得透明函数响应式编程（TFRP）的发展更进一步，并提供了一种独立的实现。它实现的TFRP具有鲁棒性、同步、可预见性、高效等特性。

对[Mendix](https://github.com/mendix)致以成吨的感谢，为了支持Mobx的维护，尤其是在一个真实的、复杂的、高性能要求的应用中证明了Mobx的思想。

最后，荣誉属于那些相信、尝试、验证甚至[贡献](https://github.com/mobxjs/mobx/blob/master/sponsors.md)Mobx的人们。

## 更多资源和文档 

*   [MobX 主页](http://mobxjs.github.io/mobx/faq/blogs.html)
*   [API 一览](http://mobxjs.github.io/mobx/refguide/api.html)
*   [Tutorials, Blogs &amp; Videos](http://mobxjs.github.io/mobx/faq/blogs.html)
*   [Boilerplates](http://mobxjs.github.io/mobx/faq/boilerplates.html)
*   [相关项目](http://mobxjs.github.io/mobx/faq/related.html)


## 如何贡献

* 放轻松，先实现个小目标，提一个小pull request。如果新增一些特性或者大的改动，请先在Github的issue中大家一起来讨论。
* 使用 `npm test` 测试基本用例，使用`npm run coverage` 测试用例的代码覆盖率，使用 `npm run perf` 进行性能测试。


## Bower 支持 {#bower-support}

通过以下方式安装 `bower install https://unpkg.com/mobx/bower.zip`
然后使用 `lib/mobx.umd.js` 或者 `lib/mobx.umd.min.js`

## 捐赠

Mobx是否是你项目成功的关键因素？欢迎点击[捐赠按钮](https://mobxjs.github.io/mobx/donate.html)分享你的成功！
Mobx在很长一段发展的过程中保持了免费，对于任何回报我都万分感谢 :-）。如果留下你的名字，你会被加入[贡献者](https://github.com/mobxjs/mobx/blob/master/sponsors.md)的列表当中~:-）

也欢迎给翻译者捐赠，用于抚慰因为翻译被冷落的女朋友。
支付宝（Alipay）：
![](//img.alicdn.com/tps/TB1Eo2DPFXXXXccaXXXXXXXXXXX-600-900.jpg_400x400.jpg)