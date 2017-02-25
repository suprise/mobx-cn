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
Mobx 是一个让状态管理变得简单、具有高扩展性的库，以对开发者透明的函数响应式编程方式，并且这个库经过了严格的测试。

Mobx的思想非常简单：

_任何事情都源于应用的状态,并且这一过程应该是自动的_
_Anything that can be derived from the application state, should be derived. Automatically._

包括UI、数据变更、与服务器通信等等

![MobX unidirectional flow](docs/flow.png)

React和Mobx 一起使用具有强大的联动效果。
React渲染应用的状态，通过一套将状态传递到已渲染的组件树中的机制。
Mobx则提供了存储和更新应用状态的机制。

React和Mobx都提供了非常优秀、独特的方式以解决应用开发中遇到的共同问题。
React提供了优异的Virtual DOM的处理机制以减少DOM操作成本。
Mobx则提供了优化将应用状态同步到React组件内的机制，通过使用了一种响应式的state依赖图，该依赖图严格只在需要的时候更新，并且不会出现代码腐化。
