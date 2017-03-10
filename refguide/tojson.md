# toJS

`toJS(value, supportCycles = true)`

将一个（可观察的）的对象递归转化为 javascript 结构。
支持可观察的数组，对象，map 以及原始值。
计算值（Computed）和其他不可枚举的属性无法成为结果的一部分。
默认情况下，循环引用可以被检测并且正确支持的，但出于性能的提升，可以将其禁止。

对于更加复杂的序列化场景，可以使用 [serializr](https://github.com/mobxjs/serializr)。

```javascript
var obj = MobX.observable({
    x: 1
});

var clone = MobX.toJS(obj);

console.log(MobX.isObservableObject(obj)); // true
console.log(MobX.isObservableObject(clone)); // false
```

注意：该方法在 MobX 2.2 前命名为 `toJSON`


