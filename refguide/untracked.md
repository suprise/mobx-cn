# Untracked

Untracked 允许你不建立观察地运行一段代码。
像 `transaction`, `untracked` 被 `(@)action` 自动应用, 所以通常使用 actions 比直接使用 `untracked` 更有意义。
例如:

```javascript

const person = observable({
	firstName: "Michel",
	lastName: "Weststrate"
});

autorun(() => {
	console.log(
		person.lastName,
		",",
		// untracked 区块将返回 person.firstName 但不建立依赖
		untracked(() => person.firstName)
	);
});
// 输出: Weststrate, Michel

person.firstName = "G.K.";
// 没有输出!

person.lastName = "Chesterton";
// 输出: Chesterton, G.K.
```
