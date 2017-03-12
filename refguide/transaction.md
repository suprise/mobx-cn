# Transaction

_Transaction 已经被废弃，建议使用 *action* 或者 *runInAction*_

`transaction(worker: () => void)` 被用于批量更新，不通知任何的观察者直到事务的结束。
`transaction` 将单一无参数的 `worker` 方法作为参数并运行。
在该方法完成之前没有观察者会被通知。
`transaction` 返回由 `worker` 方法返回的值。
注意 `transaction` 的运行是完全同步的.
事务是可以嵌套的。仅当最外层的 `transaction` 完成后 reactions 才会被执行。

```javascript
import {observable, transaction, autorun} from "MobX";

const numbers = observable([]);

autorun(() => console.log(numbers.length, "numbers!"));
// 输出: '0 numbers!'

transaction(() => {
	transaction(() => {
		numbers.push(1);
		numbers.push(2);
	});
	numbers.push(3);
});
// 输出: '3 numbers!'
```
