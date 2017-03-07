# 如何使用装饰器

使用ES.next的语法是可选的。这一部分解释了如何去使用它们，或者如果避免使用它们。

使用装饰器的优势:
* 声明式语法，最小代码量。
* 便于使用和阅读，大多数Mobx用户都这么用。


使用装饰器的缺点：
* 这是Stage-2 ES.next的特性，兼容性上可能存在问题。
* 需要一些安装和编译过程，目前只支持Babel/Typescript两种预编译器

## 使用装饰器

如果你想使用装饰器可以按照以下步骤

**TypeScript**

使`tsconfig.json` 文件中的 `experimentalDecorators` 选项变为启用，或者以flag`--experimentalDecorators`的形式传给编译器

**Babel:**

安装：`npm i --save-dev babel-plugin-transform-decorators-legacy`,
并且在你的`.babelrc` 配置文件中开启。

```
{
  "presets": [
    "es2015",
    "stage-1"
  ],
  "plugins": ["transform-decorators-legacy"]
}
```

注意插件的顺序非常重要：`transform-decorators-legacy` 应该放在插件的第一个。关于Babel还有什么问题的话，请先查看这个[issue](https://github.com/mobxjs/mobx/issues/105)


当使用react native的时候，下面这个预设可以代替`transform-decorators-legacy`
```
{
  "presets": ["stage-2", "react-native-stage-0/decorator-support"]
}
```

## 装饰器的限制

* 元数据反射（reflect-metadata） https://github.com/mobxjs/mobx/issues/534
* 装饰器还没有被Next.JS支持[issue](https://github.com/zeit/next.js/issues/26)
* decorators are not supported out of the box in `create-react-app`. To fix this, you can either eject, or use [custom-react-scripts](https://www.npmjs.com/package/custom-react-scripts) for `create-react-app` ([blog](https://medium.com/@kitze/configure-create-react-app-without-ejecting-d8450e96196a#.n6xx12p5c))


## 不使用装饰器也可以创建一个可观察的对象。
如果不使用装饰器，`extendObservable` 可以用于将一个可观察的属性引入对象，一个典型的例子就是构造器所做的。
下面这个例子介绍了如何在构造器函数中定义可观察属性、计算属性和行为

```javascript
function Timer() {
	extendObservable(this, {
		start: Date.now(),
		current: Date.now(),
		get elapsedTime() {
			return (this.current - this.start) + "seconds"
		},
        tick: action(function() {
          	this.current = Date.now()
        })
	})
}
```

Or, when using classes:

```javascript
class Timer {
	constructor() {
		extendObservable(this, {
			/* See previous listing */
		})
	}
}
```

## 使用装饰器创建可观察属性。

装饰器和Class一起使用是非常爽的。
当使用装饰器时，可观察变量、计算值、行为都能很简单地定义。

```javascript
class Timer {
	@observable start = Date.now();
	@observable current = Date.now();

	@computed get elapsedTime() {
		return (this.current - this.start) + "seconds"
	}

	@action tick() {
		this.current = Date.now()
	}
}
```

## 创建观察者组件

来自于mobx-package这个包的`observer` 函数或装饰器可以将React组件转换为观察者组件。
你所需要记住的是：`@observer class ComponentName {}`只是 `const ComponentName = observer(class { })` 的语法糖。
下面这些形式创建的观察者组件都是可行的：

无状态组件，ES5：

```javascript
const Timer = observer(function(props) {
	return React.createElement("div", {}, props.timer.elapsedTime)
})
```

无状态组件, ES6:

```javascript
const Timer = observer(({ timer }) =>
	<div>{ timer.elapsedTime }</div>
)
```

React 组件, ES5:

```javascript
const Timer = observer(React.createClass({
	/* ... */
}))
```

React 组件 class, ES6:

```javascript
const Timer = observer(class Timer extends React.Component {
	/* ... */
})
```

React 组件类和装饰器一起使用, ES.next:

```javascript
@observer
class Timer extends React.Component {
	/* ... */
}
```
