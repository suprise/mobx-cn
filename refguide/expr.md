# Expr

`expr` 可用于在 computed 值中创建临时的 computed 值。
嵌套的 computed 值对简便的计算是有用的，能避免昂贵的计算。

在下列示例中，如果 selection 在别处修改（译者注：即不造成 isSelected 改变），`expr` 会阻止 `TodoView` 组件重渲染。
而组件只会在相关 todo 被（取消）选择时发生重渲染。

```javascript
const TodoView = observer(({todo, editorState}) => {
    const isSelected = MobX.expr(() => editorState.selection === todo);
    return <div className={isSelected ? "todo todo-selected" : "todo"}>{todo.title}</div>;
});
```

`expr(func)` 相当于 `computed(func).get()`.
