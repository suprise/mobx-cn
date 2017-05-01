# createTransformer

`createTransformer<A, B>(transformation: (value: A) => B, onCleanup?: (result: B, value?: A) => void): (value: A) => B`

`createTransformer` 将一个函数（应该将一个值转为另一个值）转换为一个响应的记忆函数。
换句话说，如果给予 `transformation` 函数一个明确的 A 值来计算 B，对于相同的 A，未来任何对于转换函数的调用将返回相同的 B 值。
相反，如果 A 改变了，转换函数将会被重新运行导致 B 的更新。
最后，如果没有人使用对于明确 A 的转换函数，它的记录将会从记忆表中删去。

通过 `createTransformer` 是可以十分容易地将一个完整的数据图转换为另一个数据图的。
转换函数可以被组合，所以你可以使用许多小的转换函数来创建一颗树。
生成的数据图会始终保持更新，它将通过对结果图应用小更新来保持与源的同步。
这使得十分容易去实现强大的模式类似于 `sideways data loading`（将数据直接推送给某些具体的组件，而非从父级层层传递），`map-reduce`，使用不可变数据追踪状态历史等等。

可选的 `onCleanup` 函数被用于获取通知当一个转换对象不再需要。
如果需要，这可以用于处理附加到结果对象的资源。

总是要在类似于 `@observer` 或 `autorun` 的 reaction 内使用转换函数。
如果没有观察任何值，转换函数会像其他 computed 值一样，回退到惰性求值，当然这是违背它们目的的。

这一切可能是有些模糊的，因此这里有两个例子通过小而响应的函数将一个数据结构转换为另一个来解释整个想法：

## 使用不变的、可分享的数据结构追踪可变的 state

这个例子来源于 [Reactive2015 conference demo](https://github.com/MobXjs/mobx-reactive2015-demo):

```javascript
/*
    这个 store 保存了我们的域: boxes and arrows
*/
const store = observable({
    boxes: [],
    arrows: [],
    selection: null
});

/**
    每次更改都会将 store 序列化为 json，将它 push 进入 states
*/
const states = [];

autorun(() => {
    states.push(serializeState(store));
});

const serializeState = createTransformer(store => ({
    boxes: store.boxes.map(serializeBox),
    arrows: store.arrows.map(serializeArrow),
    selection: store.selection ? store.selection.id : null
}));

const serializeBox = createTransformer(box => ({...box}));

const serializeArrow = createTransformer(arrow => ({
    id: arrow.id,
    to: arrow.to.id,
    from: arrow.from.id
}));
```

在这个示例中，state 通过三个不同的转换函数组合来序列化。
autorunner 触发 `store` 对象序列化，依此将所有的 boxes 和 arrows 序列化。
让我们仔细看看虚构示例 box#3 的生命周期。

1. 首先 box#3 是通过 `map` 传递给 `serializeBox` 的，
执行 serializeBox 的转换，包含 box#3 与它的序列化表达被添加到 `serializeBox` 内部记忆表中。
2. 假设另一个 box 被添加到 `store.boxes` 列表。
将会使 `serializeState` 函数重新计算，导致所有 box 完全重新映射。
然而，所有 `serializeBox` 的调用将会从记忆表中返回它们的旧值，因为转换函数不（需要）再次运行。
3. 第二步，如果有人改变了 box#3 属性将会造成 `serializeBox` 对于 box#3 重新计算，就像在 MobX 中的其他 reactive 函数。
由于转换函数会生成 box#3 的新 Json 对象，该转换函数对应的所有观察者将会被强制重新运行。
在上述示例中是 `serializeState` 的转换。
`serializeState` 依次产生新的值再次映射所有的 box。但是除了 box#3，其他 box 将均从记忆表中返回。
4. 最后，如果 box#3 从 `store.boxes` 中移除，`serializeState` 将会重新计算。
但是 `serializeBox` 不再应用于 box#3, 响应式函数将会回退到非响应模式。
然后通知记忆表，能被删除，以便准备好 GC。

所以我们有效地使用不可变的，可分享的数据结构来实现状态的追踪。
所有的 box 与 arrow 均被映射和简化到一颗 state 树中。
每次变化都会使 `states` 数组中新增一条记录，但是不同的记录将分享它们所有的 box 与 arrow 表现。

## 一个数据图转换为另一个响应数据图

转换函数不仅能返回普通值，也可能返回 observable 对象。
它可以用来把一个 observable 数据图转换为另一个 observable 数据图，又可以用来转换...你懂的。

这里有个小示例，对一个每次改变都会更新的响应式文件探测器进行编码。
这样构建数据图通常会反应更快，与你使用自己代码来驱动数据图相比，将包含更为直接的代码。
查看 [性能测试](https://github.com/mobxjs/mobx/blob/3ea1f4af20a51a1cb30be3e4a55ec8f964a8c495/test/perf/transform-perf.js#L4) 
的相关示例

不同与先前的例子，`transformFolder` 在文件夹保持可见时仅会运行一次；
`DisplayFolder` 对象追踪与 `Folder` 对象本身相关联的东西。

以下示例均是所有对于 `state` 图的变化都会被自动处理。
一些示例：
1. 改变一个文件夹的名称将会更新它自己的 `path` 属性和所有后代的 `path` 属性。
2. 折叠（collapse）一个文件夹将会从树上移除所有后代 `DisplayFolders`。
3. 展开一个文件夹将会再次恢复。
4. 设置搜索过滤（filter）将会移除所有与过滤不匹配的节点，除非它们的后代与过滤相匹配。
5. 等等

```javascript
function Folder(parent, name) {
	this.parent = parent;
	m.extendObservable(this, {
		name: name,
		children: m.asFlat([]),
	});
}

function DisplayFolder(folder, state) {
	this.state = state;
	this.folder = folder;
	m.extendObservable(this, {
		collapsed: false,
		name: function() {
			return this.folder.name;
		},
		isVisible: function() {
			return !this.state.filter || this.name.indexOf(this.state.filter) !== -1 || this.children.some(child => child.isVisible);
		},
		children: function() {
			if (this.collapsed)
				return [];
			return this.folder.children.map(transformFolder).filter(function(child) {
				return child.isVisible;
			})
		},
		path: function() {
			return this.folder.parent === null ? this.name : transformFolder(this.folder.parent).path + "/" + this.name;
		}
	});
}

var state = m.observable({
	root: new Folder(null, "root"),
	filter: null,
	displayRoot: null
});

var transformFolder = m.createTransformer(function (folder) {
	return new DisplayFolder(folder, state);
});

m.autorun(function() {
    state.displayRoot = transformFolder(state.root);
});
```
