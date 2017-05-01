# 创建 observable 数据结构与反应

## Atoms

在某些时候，你或许想要更多的数据结构或是可以被使用于反应计算的东西（例如流）。
通过 `Atom` 类来实现是十分简单的。
Atoms 可以向 MobX 发出信号，当一些 observable 数据源被观察或是改变。
并且当它们被使用或不再使用时 MobX 会向 atom 发出信号。

以下例子展示了你要怎样创建一个 observable 的 `Clock`，可用于反应函数，返回当前日期时间。
这个 clock 当被观察时才会真正地运行。

`Atom` 类的全部 API 将通过这个示例来示范。

```javascript
import {Atom, autorun} from "mobx";

class Clock {
	atom;
	intervalHandler = null;
	currentDateTime;

	constructor() {
	  // 创建一个 atom 与 MobX 的核心算法进行交互
		this.atom =	new Atom(
			// 第一个参数：这个 atom 的名称，为了 debug
			"Clock",
			// 第二（可选）参数：当 atom 从 unobserved 到 observed 转换时触发的回调
			() => this.startTicking(),
			// 第三（可选）参数：当 atom 从 observed 到 unobserved 转换时触发的回调
			// 注意：同一个 atom 可在这两个 state 中转换多次
			() => this.stopTicking()
		);
	}

	getTime() {
		// 让 MobX 知道 observable 数据源被使用
		// 如果 atom 当前正在被一些 reaction 观察，reportObserved 将会返回 true
		// 如果需要，reportObserved 将会触发 onBecomeObserved 事件（startTicking）
		if (this.atom.reportObserved()) {
			return this.currentDateTime;
    } else {
			// 显然 getTime 被调用时没有 reaction 正在运行
			// 所以，没有东西依赖这个值，因此 onBecomeObserved （startTicking）没有被触发
			// 根据你的 atom，在这种情况下可能有不同的表现（就像抛出错误，返回一个默认值）
			return new Date();
		}
	}

	tick() {
		this.currentDateTime = new Date();
		// 让 MobX 知道数据源改变
		this.atom.reportChanged();
	}

	startTicking() {
		this.tick(); // 初始化 tick
    this.intervalHandler = setInterval(
			() => this.tick(),
			1000
		);
	}

	stopTicking() {
		clearInterval(this.intervalHandler);
		this.intervalHandler = null;
	}
}

const clock = new Clock();

const disposer = autorun(() => console.log(clock.getTime()));

// ... 每秒输出时间

disposer();

// 输出停止，如果没有其他人使用相同的 `clock`，clock 也将会停止。
```

## 反应

`Reaction` 允许你创建你自己的 'auto runner'。
当函数由于一个或多个依赖改变需要再次执行时，Reaction 会追踪函数和信号。

这是 `autorun` 如何使用 `Reaction` 来定义的：

```typescript
export function autorun(view: Lambda, scope?: any) {
	if (scope)
		view = view.bind(scope);
		
	const reaction = new Reaction(view.name || "Autorun", function () {
		this.track(view);
	});
	
	// 开始运行刚创建的 reaction 或将其纳入计划中
	if (isComputingDerivation() || globalState.inTransaction > 0)
		globalState.pendingReactions.push(reaction);
	else
		reaction.runReaction();

	return reaction.getDisposer();
}
```
