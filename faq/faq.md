## FAQ

##### 支持哪些浏览器？

MobX在ES5环境下运行，那意味着在Node.js、Rhino和IE9+浏览器都能支持。请查阅[caniuse.com](http://caniuse.com/#feat=es5)

##### MobX能否和RxJS一起使用?
是的，你可以使用来自于MobX-utils包的[toStream 和 fromStream ](https://github.com/MobXjs/MobX-utils#tostream) 这两个方法，来使用 RXJS（and other TC 39 compatible observables）。


##### 什么时候改用RxJS代替MobX？（后续了解更多再翻译这个对比）
For anything that involves explictly working with the concept of time,
or when you need to reason about the historical values / events of an observable (and not just the latest), RxJs is recommended as it provides more low-level primitives.
Whenever you want to react to _state_ instead of _events_, MobX offers an easier and more high-level approach.
In practice, combining RxJS and MobX might result in really powerful constructions.
Use for example RxJS to process and throttle user events and as a result of that update the state.
If the state has been made observable by MobX, it will then take care of updating the UI and other derivations accordingly.

##### 是否支持React Native?
是的，`MobX` 和 `mobx-react`能够在React Native上工作，后者需要引入`"mobx-react/native"`。
但是开发工具不支持React Native。注意如果你希望存储组件的状态，并且该状态可以热更新，组件中不要使用装饰器，而是用函数去处理（例如使用 `action(fn)` 代替 `@action`）。


##### MobX与其他响应式框架的对比?

请查阅[issue](https://github.com/MobXjs/MobX/issues/18) .

##### MobX是否是一个框架?

MobX不是一个框架，它不会告诉你如何整理你的代码结构、在何处保存状态或处理事件。它会将你从各种框架带来的限制中解放出来。


##### 我能否将MobX和Flux一起使用?

Flux implementations that do not work on the assumption that the data in their stores is immutable should work well with MobX.
However, the need for Flux is reduced when using MobX.
MobX already optimizes rendering, and it works with most kinds of data, including cycles and classes.
So other programming paradigms like classic MVC can now be easily applied in applications that combine ReactJS with MobX.

##### 我能否将MobX和其他框架组合使用?

很可能可以。
MobX反对框架，并且可以在任何现代JS环境中运行。
为了方便，它只是用一些小函数封装，将React组件转换为可响应组件。
MobX在服务端渲染也能够执行，并且和jQuery一起使用。（请查阅[Fiddle](http://jsfiddle.net/mweststrate/vxn7qgdw)) 和 [Deku](https://gist.github.com/mattmccray/d8740ea97013c7505a9b)）。


##### 我能否记录状态？
是的，请查阅一些[createTransformer](http://MobXjs.github.io/MobX/refguide/create-transformer.html) 的例子。
