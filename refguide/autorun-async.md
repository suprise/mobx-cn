# autorunAsync

用法：`autorunAsync(action: () => void, minimumDelay?: number, scope?)`

与`autorun`类似，`autorunAsync`希望在最短的毫秒后`action`不是被同步而是被异步地调用，这时候`action`将会被执行与观测。但是，在它观察到的值已更改时，`action`不是立即运行该操作，而是在再次执行该操作之前等待 minimumDelay 毫秒。

如果被观测的值在等待时被多次修改，则`action`仍然只会被触发一次，因此在某种意义上，`autorunAsync`实现了与`transaction`相似的效果。这可能对于那些消耗十分昂贵但又没必要同步触发的操作十分有用，比如与服务器通信时的防抖动。

如果作用域已经被给定了，那么`action`将会绑定在这个作用域对象上。

`autorunAsync(debugName: string, action: () => void, minimumDelay?: number, scope?)`

如果`autorunAsync`的第一个参数是字符串，那么他将被用作调试名称 (debug name) 。

`autorunAsync` 返回一个取消`autorun`的处理器.

```javascript
autorunAsync(() => {
	//假设 profile.asJson 返回一个可观察的 Json 格式的数据，
	//在每次更改时将其发送到服务器，但在发送之前等待至少300毫秒，
	//发送时，将使用profile.asJson的最新值。
	sendProfileToServer(profile.asJson);
}, 300);
```
