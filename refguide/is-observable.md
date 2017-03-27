# isObservable

如果给定的值是被MobX转化为可观察的，则返回true。

可以接受第二个字符串参数来查看特定属性是否可观察。


```javascript
var person = observable({
	firstName: "Sherlock",
	lastName: "Holmes"
});

person.age = 3;

isObservable(person); // true
isObservable(person, "firstName"); // true
isObservable(person.firstName); // false (just a string)
isObservable(person, "age"); // false
```

# isObservableMap

如果给定的对象是由`MobX.map`建立的，则返回 true。

# isObservableArray

如果给定的对象是使用`MobX.observable(array)`建立的可观察数组，则返回true。

# isObservableObject

如果给定对象是使用`MobX.observable(object)`建立的可观察对象，则返回true。
