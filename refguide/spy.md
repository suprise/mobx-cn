# Spy

用法: `spy(listener)`.
注册一个全局的 spy 监听器来监听 MobX 发生的所有事件。
这相当于将 `observe` 监听器一次性附加到 *所有* observables，也包括通知正在运行的 (trans/re)actions 和 computations。
使用的例子如：`mobx-react-devtools`。

使用 spy 监听所有的 action：

```javascript
spy((event) => {
    if (event.type === 'action') {
        console.log(`${event.name} with args: ${event.arguments}`)
    }
})
```

Spy 监听器总是接受一个对象，通常至少有 `type` 字段。默认状态下，spy 会发出下列事件：

| 事件 | 字段 | 嵌套 |
| --- | --- |--- |
| action | name, target (scope), arguments, fn (source function of the action | yes |
| transaction | name, target (scope) | yes |
| scheduled-reaction | object (Reaction instance) | no |
| reaction | object (Reaction instance), fn (source of the reaction) | yes
| compute | object (ComputedValue instance), target (scope), fn (source) | no
| error | message | no |
| update (array) | object (the array), index, newValue, oldValue | yes
| update (map) | object (observable map instance), name, newValue, oldValue | yes
| update (object) | object (instance), name, newValue, oldValue | yes
| splice (array) | object (the array), index, added, removed, addedCount, removedCount | yes
| add (map) | object, name, newValue | yes
| add (object) | object, name, newValue | yes
| delete (map) | object, name, oldValue | yes
| create (boxed observable) | object (ObservableValue instance), newValue | yes |

注意，这些事件都具有签名 `{ spyReportEnd: true, time? }`。
这些事件可能没有 `type` 字段，但他们是部分早期触发的事件具有 `spyReportStart: true`。
事件可能表示事件的结束，之后具有子事件的事件组被创建。
事件也可能报告总执行时间。

observable 值的 spy 事件与传递给 `observe` 的事件相同。详见 [intercept & observe](observe.md)

你也可以发出你自己的 spy 事件。详见 `extras.spyReport`, `extras.spyReportStart` 与 `extras.spyReportEnd`。
