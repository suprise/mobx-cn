# Best Practices for building large scale maintainable projects
# 构建一个大型的、可扩展的、可持续维护项目的最佳实践。

这部分包括了一些我们在使用Mobx开发Mendix时发现的最佳实践
这部分是是可选的，你并不强制需要这么做。有很多方式来玩Mobx和React，这只是其中的一种方式。

# Stores

让我们从_stores_开始。之后我们会讨论_actions_和React _components_。
Store可以从任何类Flux架构中找到，并且与MVC模式中的controller 对比。
Store的主要职责是，从组件内移除_逻辑_和_状态_，抽取到一个独立的可单元测试的部分，这部分可以同时应用于前端和后端两部分。

## 存储UI状态


大多数应用朱旭至少两个Store，一个用于存储UI状态，另一个用于存储数据。
分成两个的意义是你可以全局重用和测试，你也可以用于在其他系统中重用。
UI状态Store通常是否为你的应用定制的，但通常会比较简单。这个store通常没有太多的逻辑，这对于开发是理想的，因为在开发过程中改变UI状态是很经常的事情。

UI Store中常见存储的信息有：
* Session 信息
* 不会再后端存储的信息
* 会全局影响UI的信息。
  * Window尺寸
  * 提示消息
  * 当前语言
  * 当前主题
* 更多可能存储的组件信息：
  * 当前选择
  * 工具条显示隐藏状态

也许一开始你会尝试用组件内部状态存储这些信息。但后面你会发现，别的地方也需要这些信息。这种情况下，你可以将状态信息控制移到UI State Store。确保这个状态是单例的。
对于同构应用，你可能也想使用同样的默认Store，这样所有组件都可以按照预期运行。
你可以传递这个store，也可以作为模块全局引用。
出于测试的目的，我建议你通过组件树传递。

Store的例子:

```javascript
import {observable, computed, asStructure} from 'mobx';
import jquery from 'jquery';

class UiState {
    @observable language = "en_US";
    @observable pendingRequestCount = 0;

    // asStructure makes sure observer won't be signaled only if the
    // dimensions object changed in a deepEqual manner
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

## 数据模型 Stores

稍后翻译。

Your application will contain one or multiple _domain_ stores.
These stores store the data your application is all about.
Todo items, users, books, movies, orders, you name it.
Your application will most probably have at least one domain store.

A single domain store should be responsible for a single concept in your application.
However a single concept might take the form of multiple subtypes and it is often a (cyclic) tree structure.
For example: one domain store for your products, and one for our orders and orderlines.
As a rule of thumb: if the nature of the relationship between two items is containment, they should typically be in the same store.
So a store just manages _domain objects_.

These are the responsibility of a store:
* Instantiate domain objects. Make sure domain objects know the store they belong to.
* Make sure there is only one instance of each of your domain objects.
The same user, order or todo should not be twice in your memory.
This way you can safely use references and also be sure you are looking at the latest instance, without ever having to resolve a reference.
This is fast, straightforward and convenient when debugging.
* Provide backend integration. Store data when needed.
* Update existing instances if updates are received from the backend.
* Provide a stand-alone, universal, testable component of your application.
* To make sure your store is testable and can be run server-side, you probably will move doing actual websocket / http requests to a separate object so that you can abstract over your communication layer.
* There should be only one instance of a store.

### Domain objects

Each domain object should be expressed using its own class (or constructor function).
It is recommended to store your data in _denormalized_ form.
There is no need to treat your client-side application state as some kind of database.
Real references, cyclic data structures and instance methods are powerful concepts in JavaScript.
Domain objects are allowed to refer directly to domain objects from other stores.
Remember: we want to keep our actions and views as simple as possible and needing to manage references and doing garbage collection yourself might be a step backward.
Unlike many Flux architectures, with MobX there is no need to normalize your data, and this makes it a lot simpler to build the _essentially_ complex parts of your application:
your business rules, actions and user interface.

Domain objects can delegate all their logic to the store they belong to if that suits your application well.
It is possible to express your domain objects as plain objects, but classes have some important advantages over plain objects:
* They can have methods.
This makes your domain concepts easier to use stand-alone and reduces the amount of contextual awareness that is needed in your application.
Just pass objects around.
You don't have to pass stores around, or have to figure out which actions can be applied to an object if they are just available as instance methods.
Especially in large applications this is important.
* They offer fine grained control over the visibility of attributes and methods.
* Objects created using a constructor function can freely mix observable properties and functions, and non-observable properties and methods.
* They are easily recognizable and can strictly be type-checked.


### Example domain store

```javascript
import {observable, autorun} from 'mobx';
import uuid from 'node-uuid';

export class TodoStore {
    authorStore;
    transportLayer;
    @observable todos = [];
    @observable isLoading = true;

    constructor(transportLayer, authorStore) {
        this.authorStore = authorStore; // Store that can resolve authors for us
        this.transportLayer = transportLayer; // Thing that can make server requests for us
        this.transportLayer.onReceiveTodoUpdate(updatedTodo => this.updateTodoFromServer(updatedTodo));
        this.loadTodos();
    }

    /**
     * Fetches all todo's from the server
     */
    loadTodos() {
        this.isLoading = true;
        this.transportLayer.fetchTodos().then(fetchedTodos => {
            fetchedTodos.forEach(json => this.updateTodoFromServer(json));
            this.isLoading = false;
        });
    }

    /**
     * Update a todo with information from the server. Guarantees a todo
     * only exists once. Might either construct a new todo, update an existing one,
     * or remove an todo if it has been deleted on the server.
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
     * Creates a fresh todo on the client and server
     */
    createTodo() {
        var todo = new Todo(this);
        this.todos.push(todo);
        return todo;
    }

    /**
     * A todo was somehow deleted, clean it from the client memory
     */
    removeTodo(todo) {
        this.todos.splice(this.todos.indexOf(todo), 1);
        todo.dispose();
    }
}

export class Todo {

    /**
     * unique id of this todo, immutable.
     */
    id = null;

    @observable completed = false;
    @observable task = "";

    /**
     * reference to an Author object (from the authorStore)
     */
    @observable author = null;

    store = null;

    /**
     * Indicates whether changes in this object
     * should be submitted to the server
     */
    autoSave = true;

    /**
     * Disposer for the side effect that automatically
     * stores this Todo, see @dispose.
     */
    saveHandler = null;

    constructor(store, id=uuid.v4()) {
        this.store = store;
        this.id = id;

        this.saveHandler = reaction(
            // observe everything that is used in the JSON:
            () => this.asJson,
            // if autoSave is on, send json to server
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
