# when

`when(debugName?, predicate: () => boolean, effect: () => void, scope?)`

`when` 观察并运行 `predicate` 直到返回 `true`
, 则 `effect` 执行并且这个自动运行被处理。
该方法返回一个处理器用于提前取消自动运行。

该方法以一种响应式的方式处理或取消事物是非常有用的。
例如：

```javascript
class MyResource {
	constructor() {
		when(
			// once...
			() => !this.isVisible,
			// ... then
			() => this.dispose()
		);
	}

	@computed get isVisible() {
		// indicate whether this item is visible
	}

	dispose() {
		// dispose
	}
}

```
