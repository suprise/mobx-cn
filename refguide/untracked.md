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
		// this untracked block will return the person's firstName without establishing a dependency
		untracked(() => person.firstName)
	);
});
// prints: Weststrate, Michel

person.firstName = "G.K.";
// doesn't print!

person.lastName = "Chesterton";
// prints: Chesterton, G.K.
```
