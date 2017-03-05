#action

用法:
* `action(fn)`
* `action(name, fn)`
* `@action classMethod() {}`
* `@action(name) classMethod () {}`
* `@action boundClassMethod = (args) => { body }`
* `@action(name) boundClassMethod = (args) => { body }`
* `@action.bound classMethod() {}`
* `@action.bound(function() {})`

任何应用都有行为。任何改变状态的代码都称为行为。
使用Mobx可以使你的代码更加清晰，Action会使你的代码结构更优。
它获得一个函数，并将这个函数使用`untracked`, `transaction` and `allowStateChanges` 包裹后返回。
建议在任何改变状态或具有副作用的函数上使用。也提供了有效的调试信息。

不支持在 [ES 5.1 setters](http://www.ecma-international.org/ecma-262/5.1/#sec-11.1.5) (i.e. `@action set propertyName`) 使用 `@action`, 即使[计算属性自动触发action](https://github.com/mobxjs/mobx/blob/gh-pages/docs/refguide/computed-decorator.md#setters-for-computed-values).


注意：当严格模式开启时。使用`action`是强制的。请查阅 [`useStrict`](https://github.com/mobxjs/mobx/blob/gh-pages/docs/refguide/api.md#usestrict)

 可以查看关于`action` 的进一步介绍 [MobX 2.2 release notes](https://medium.com/p/45cdc73c7c8d/).

两个来自于『通讯录』项目的例子:

```javascript
	@action	createRandomContact() {
		this.pendingRequestCount++;
		superagent
			.get('https://randomuser.me/api/')
			.set('Accept', 'application/json')
			.end(action("createRandomContact-callback", (error, results) => {
				if (error)
					console.error(error);
				else {
					const data = JSON.parse(results.text).results[0];
					const contact = new Contact(this, data.dob, data.name, data.login.username, data.picture)
					contact.addTag('random-user');
					this.contacts.push(contact);
					this.pendingRequestCount--;
				}
			}));
	}
```


## `async`actions 和`runInAction`

`action`只影响当前运行的函数，不是当前调度（不是调用）的函数。也就是说，如果你有一个`setTimeOut`，promise`.then`或`async`构造，在回调中有更多的状态被该改变，这些回调也应该被包含在`action`中。这可以在上面的`"createRandomContact-callback"`操作来演示。

如果你使用`async/await`，这就非常棘手了，因为你不能只是在`action`中单纯的包装异步函数体了。在这种情况下，`runInAction`就可以派上用场了，在你打算更新状态的地方使用它就可以了。(但是不要在`await`中调用这些区块)

例如:
```javascript
@action /*optional*/ updateDocument = async () => {
    const data = await fetchDataFromUrl();
    /* required in strict mode to be allowed to update state: */
    runInAction("update state after fetching data", () => {
        this.data.replace(data);
        this.isSaving = true;
    })
}
```

`runInAction` 的用法是: `runInAction(name?, fn, scope?)`.

如果你使用babel，这个插件可以帮助你处理你的异步行为：[mobx-deep-action](https://github.com/mobxjs/babel-plugin-mobx-deep-action).

## 绑定行为（Bound actions）

 `action` 装饰器 / 函数使用javascript通常的绑定（binding）规则。
 而Mobx3 引入了`action.bound`以自动地将action与目标对象绑定。
 注意`(@)action.bound` 与 `action` 不同，不需要一个name参数，这个名称与action绑定的属性相同。

例如:

```javascript
class Ticker {
	@observable this.tick = 0

	@action.bound
	increment() {
		this.tick++ // 'this' will always be correct
	}
}

const ticker = new Ticker()
setInterval(ticker.increment, 1000)
```

Or

```javascript
const ticker = observable({
	tick: 1,
	increment: action.bound(function() {
		this.tick++ // bound 'this'
	})
})

setInterval(ticker.increment, 1000)
```

_注意：不要将*action.bind*与箭头函数组合使用，因为箭头函数已经绑定上下文，且不能重新绑定_
