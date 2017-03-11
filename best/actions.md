# 编写行为（actions）

使用MobX来编写action是一件非常直观的事情。只需要创建、改变或者删除数据，MobX会确保相应的一切都改变，因为Store和组件都依赖于你的数据。
以之前创建的这个Store为例，action可以想这样简单：

```javascript
var todo = todoStore.createTodo();
todo.task = "make coffee";
```
这样就能简单地创建todo，然后就可以提交给服务器和更新UI


## 什么时候使用actions？
Actions 应该只用于_改变_状态。
提供查阅、过滤的函数不应该被标记为action以允许MobX追踪它们的调用。


## 异步action
编写一个异步action也很简单。直接用就可以了

```javascript
// ...
	this.isLoading = true;
	this.transportLayer.fetchTodos().then(fetchedTodos => {
		fetchedTodos.forEach(json => this.updateTodoFromServer(json));
		this.isLoading = false;
	});
// ...
```
在异步action完成之后，更新数据后View就会更新。

```javascript
import {observer} from "mobx-react";

var TodoOverview = observer(function(props) {
	var todoStore = props.todoStore;
	if (todoStore.isLoading) {
		return <div>Loading...</div>;
	} else {
		return <div>{
			todoStore.todos.map(todo => <TodoItem key={todo.id} todo={todo} />)
		}</div>
	}
});
```

上面这个组件当`isLoading`改变的时候会自动更新，或者`isLoading`为true且`todos`改变也会自动更新。
