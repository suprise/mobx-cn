# @observable

装饰器可以用于ES7环境或者TypeScript中，以使类属性转变为可观察的。
@observable 可用于实例字段和属性getters。
这为你的普通对象转变为可观察对象提供了细粒度的方法

```javascript
import {observable} from "MobX";

class OrderLine {
    @observable price = 0;
    @observable amount = 1;

    constructor(price) {
        this.price = price;
    }

    @computed get total() {
        return this.price * this.amount;
    }
}
```

如果你的环境不支持装饰器或者字段初始化，`@observable key = value;` 只是 [`extendObservable(this, { key: value })`](extend-observable.md) 的一个语法糖，你可以使用 `extendObservable` 方法。

可枚举性：`@observable` 装饰过的属性是可枚举的，但是这个属性是定义在类声明上的，而非实例上。
换句话说：

```javascript
const line = new OrderLine();
console.log("price" in line); // true
console.log(line.hasOwnProperty("price")); // false,  price _属性_ 是定义在类声明上的, 即使每个实例都存储有自己的值.
```

`@observable` 装饰器可以和 `asStructure` 之类的修饰符一起使用

```javascript
@observable position = asStructure({ x: 0, y: 0})
```


### 使你的构建工具支持装饰器

装饰器语法默认是不被支持的。

* 如果你使用的是 _typescript_ ，请开启 `--experimentalDecorators` 这一构建工具标记，并在 `tsconfig.json` 中将 `experimentalDecorators` 设置为 `true` (推荐)。
* 如果你使用的是 _babel5_ ， 请确保 `--stage 0` 传给了Babel CLI
* 如果你使用的是 _babel6_ ，见下面这个配置[例子]((https://github.com/MobXjs/MobX/issues/105))
