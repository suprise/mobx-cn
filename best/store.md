# Best Practices for building large scale maintainable projects
# 构建一个大型的、可扩展的、可持续维护项目的最佳实践。

这部分包括了一些我们在使用 MobX 开发 Mendix 时发现的最佳实践
这部分是是可选的，你并不强制需要这么做。有很多方式来玩 MobX 和 React ，这只是其中的一种方式。

# Stores

让我们从 _stores_ 开始。之后我们会讨论 _actions_ 和 React _组件_。
Store 可以从任何类 Flux 架构中找到，并且与 MVC 模式中的 controller 对比。
Store 的主要职责是，从组件内移除 _逻辑_ 和 _状态_ ，抽取到一个独立的可单元测试的部分，这部分可以同时应用于前端和后端两部分。

## 存储UI状态


大多数应用都需要至少两个 Store，一个用于存储 UI 状态，另一个用于存储数据。
分成两个的意义是你可以全局重用和测试，你也可以用于在其他系统中重用。
UI状态Store 通常是否为你的应用定制的，但通常会比较简单。这个Store通常没有太多的逻辑，这对于开发是理想的，因为在开发过程中改变UI状态是很经常的事情。

UI Store中常见存储的信息有：
* Session 信息
* 不会再后端存储的信息
* 会全局影响UI的信息：
  * Window尺寸
  * 提示消息
  * 当前语言
  * 当前主题
* 更多可能存储的组件信息：
  * 当前选择
  * 工具条显示隐藏状态

也许一开始你会尝试用组件内部状态存储这些信息。但后面你会发现，别的地方也需要这些信息。这种情况下，你可以将状态信息控制移到UI State Store。确保这个状态是单例的。
对于同构应用，你可能也想使用同样的默认 Store，这样所有组件都可以按照预期运行。
你可以传递这个 Store，也可以作为模块全局引用。
出于测试的目的，我建议你通过组件树传递。

Store 的例子:

```javascript
import {observable, computed, asStructure} from 'mobx';
import jquery from 'jquery';

class UiState {
    @observable language = "en_US";
    @observable pendingRequestCount = 0;

    // asStructure 确保 dimensions 对象进行深度比较而改变时，才会发送信号给observer
    @observable windowDimensions = asStructure({
        width: jquery(window).width(),
        height: jquery(window).height()
    });

	constructor() {
        jquery.resize(() => {
            this.windowDimensions = getWindowDimensions();
        });
    }

    @computed get appIsInSync() {
        return this.pendingRequestCount === 0
    }
}

singleton = new UiState();
export default singleton;
```

## 领域 Stores（domain Store）

你的应用会包含一个或者多个领域 Store。这些会存储你应用的所有数据。
例如todo items, users, books, movies, orders, 或者你所命名的其他领域Store。
你的应用很可能至少有一个领域 Store。

一个领域 Store 应该在你的应用中负责单一的职能。然而一个单一的职能很可能包含多种不同的子类型，经常会是一个（循环的）树形结构
例如：一个领域 Store 用于负责 products，另一个用于负责 orders 和 orderlines。
一个简短的规则是：如果两个元素有天然的关系，则通常他们应该在同一个 Store 中。
所以是一个 Store 管理多个_领域对象_。

一个领域 Store 的职责有：
* 标明领域对象，确保领域对象知道他们属于哪个Store。
* 确保每一个领域对象只有一个实例。例如同一个用户、订单或者todo，不应该在你的内存中出现两次。通过这种方式，你可以安全的使用引用，并且确保你永远使用的是实例的最新状态，而不用处理各种引用，调试时这是一种高效、直观、方便的方式。
* 提供后端集成。在需要的时候存储数据。
* 当从后端获取到更新时，更新已有的实例。
* 提供一个你应用的独立的、通用的、可测试组件。
* 确保你的 Store 是可测试的，并且可以运行在服务端。你可能会将处理websocket / http请求的相关代码抽出到一个独立的对象中，所以你可以抽象你的服务端通信层。
* 每个 Store 都应该只有一个实例。

### 领域对象（domain object）

每个领域对象都应该是它对应的类（或者构造函数）的表达。
建议将你的数据以_非格式化_的形式存储。
没有必要将浏览器端的应用状态当做一个数据库来使用。
引用、循环数据结构和实例方法是 JavaScript 中强大的概念。
领域对象允许直接根据来自于其他 Store 的领域对象来推导生成。
记住：我们希望保持我们的行为、视图尽可能地简单，并且管理引用和手动处理垃圾回收是一种退步的行为。
与大多数的 Flux架构不同，Mobx没有必要将你的数据标准化，而是通过更简单的方法去构建你应用中复杂的部分：你的业务规则、行为和 UI。

领域对象可以通过Store来代理它们的逻辑，如果对你的应用而言更方便的话。
你可以使用普通对象来表示一个领域对象，但是通过类的方式，有更多显著的优点：
* 它们有方法（method）。这使得你的领域概念更加方便地减少上下文依赖。只需要传递object。你不需要传递stores，或者指出哪个行为可以用于某个对象，如果它们是通过实例方法提供的话。这在大型项目中尤其重要。
* 它们提供细粒度视觉属性和方法的细粒度控制。
* 通过构造函数创建的对象可以自由地组合可观察属性、函数、和不可观察的属性和方法。
* 它们是非常容易识别和做严格类型校验的。


### 领域 store 示例

```javascript
import {observable, autorun} from 'mobx';
import uuid from 'node-uuid';

export class TodoStore {
    authorStore;
    transportLayer;
    @observable todos = [];
    @observable isLoading = true;

    constructor(transportLayer, authorStore) {
        this.authorStore = authorStore; // 可以处理author的信息的Store
        this.transportLayer = transportLayer; // 可以帮助我们发送请求
        this.transportLayer.onReceiveTodoUpdate(updatedTodo => this.updateTodoFromServer(updatedTodo));
        this.loadTodos();
    }

    /**
     * 从服务端获取所有的 todo's 数据
     */
    loadTodos() {
        this.isLoading = true;
        this.transportLayer.fetchTodos().then(fetchedTodos => {
            fetchedTodos.forEach(json => this.updateTodoFromServer(json));
            this.isLoading = false;
        });
    }

    /**
     * 根据服务端的信息更新一个todo。确保一个todo只会存在一次。
     * 构建一个新的todo或者更新一个已存在的todo、或者同步服务端的已被删除的todo的状态。
     */
    updateTodoFromServer(json) {
        var todo = this.todos.find(todo => todo.id === json.id);
        if (!todo) {
            todo = new Todo(this, json.id);
            this.todos.push(todo);
        }
        if (json.isDeleted) {
            this.removeTodo(todo);
        } else {
            todo.updateFromJson(json);
        }
    }

    /**
     * 在浏览器端和服务端创建一个todo
     */
    createTodo() {
        var todo = new Todo(this);
        this.todos.push(todo);
        return todo;
    }

    /**
     * 一个todo被清除了，在浏览器端删除它
     */
    removeTodo(todo) {
        this.todos.splice(this.todos.indexOf(todo), 1);
        todo.dispose();
    }
}

export class Todo {

    /**
     * 确保每个todo都有唯一的不可变id。
     */
    id = null;

    @observable completed = false;
    @observable task = "";

    /**
     * 从 authorStore 引用一个author对象
     */
    @observable author = null;

    store = null;

    /**
     * 指示这个对象的改变是否被自动提交给服务端。
     */
    autoSave = true;

    /**
     * 自动存储todo的响应行为的销毁函数。
     */
    saveHandler = null;

    constructor(store, id=uuid.v4()) {
        this.store = store;
        this.id = id;

        this.saveHandler = reaction(
            // 观察这个json中每个被使用的属性
            () => this.asJson,
            // 如果自动保存是开启的，则自动存储数据到服务器
            (json) => {
                if (this.autoSave) {
                    this.store.transportLayer.saveTodo(json);
                }
            }
        );
    }

    /**
     * Remove this todo from the client and server
     */
    delete() {
        this.store.transportLayer.deleteTodo(this.id);
        this.store.removeTodo(this);
    }

    @computed get asJson() {
        return {
            id: this.id,
            completed: this.completed,
            task: this.task,
            authorId: this.author ? this.author.id : null
        };
    }

    /**
     * Update this todo with information from the server
     */
    updateFromJson(json) {
        // make sure our changes aren't send back to the server
        this.autoSave = false;
        this.completed = json.completed;
        this.task = json.task;
        this.author = this.store.authorStore.resolveAuthor(json.authorId);
        this.autoSave = true;
    }

    dispose() {
        // clean up the observer
        this.saveHandler();
    }
}
```
