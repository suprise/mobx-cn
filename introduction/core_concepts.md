## Core concepts {#core-concepts}

MobX has only a few core concepts. The following snippets can be tried online using [JSFiddle](https://jsfiddle.net/mweststrate/wv3yopo0/) (or [without ES6 and JSX](https://jsfiddle.net/rubyred/55oc981v/)).

### Observable state {#observable-state}

MobX adds observable capabilities to existing data structures like objects, arrays and class instances. This can simply be done by annotating your class properties with the [@observable](http://mobxjs.github.io/mobx/refguide/observable-decorator.html) decorator (ES.Next).

```
class Todo {
    id = Math.random();
    @observable title = "";
    @observable finished = false;
}

```

Using `observable` is like turning the properties of an object into a spreadsheet cells. But unlike spreadsheets, these values cannot just be primitive values, but also references, objects and arrays. You can even [define your own](http://mobxjs.github.io/mobx/refguide/extending.html) observable data sources.

### Intermezzo: Using MobX in ES5, ES6 and ES.next environments {#intermezzo-using-mobx-in-es5-es6-and-es-next-environments}

If these `@` thingies look alien to you, these are ES.next decorators. Using them is entirely optional in MobX. See the [documentation](http://mobxjs.github.io/mobx/best/decorators.html) for details how to either use or avoid them. MobX runs on any ES5 environment, but leveraging ES.next features like decorators are the cherry on the pie when using MobX. The remainder of this readme uses decorators, but remember, _they are optional_.

For example, in good ol&#039; ES5 the above snippet would look like:

```
function Todo() {
    this.id = Math.random()
    extendObservable(this, {
        title: "",
        finished: false
    })
}

```

### Computed values {#computed-values}

With MobX you can define values that will be derived automatically when relevant data is modified. By using the [`@computed`](http://mobxjs.github.io/mobx/refguide/computed-decorator.html) decorator or by using getter / setter functions when using `(extend)Observable`.

```
class TodoList {
    @observable todos = [];
    @computed get unfinishedTodoCount() {
        return this.todos.filter(todo => !todo.finished).length;
    }
}

```

MobX will ensure that `unfinishedTodoCount` is updated automatically when a todo is added or when one of the `finished` properties is modified. Computations like these can very well be compared with formulas in spreadsheet programs like MS Excel. They update automatically whenever, and only when, needed.

### Reactions {#reactions}

Reactions are similar to a computed value, but instead of producing a new value, a reaction produces a side effect for things like printing to the console, making network requests, incrementally updating the React component tree to patch the DOM, etc. In short, reactions bridge [reactive](https://en.wikipedia.org/wiki/Reactive_programming) and [imperative](https://en.wikipedia.org/wiki/Imperative_programming) programming.

#### React components {#react-components}

If you are using React, you can turn your (stateless function) components into reactive components by simply adding the [`observer`](http://mobxjs.github.io/mobx/refguide/observer-component.html) function / decorator from the `mobx-react` package onto them.

```
import React, {Component} from 'react';
import ReactDOM from 'react-dom';
import {observer} from "mobx-react";

@observer
class TodoListView extends Component {
    render() {
        return <div>
            <ul>
                {this.props.todoList.todos.map(todo =>
                    <TodoView todo={todo} key={todo.id} />
                )}
            </ul>
            Tasks left: {this.props.todoList.unfinishedTodoCount}
        </div>
    }
}

const TodoView = observer(({todo}) =>
    <li>
        <input
            type="checkbox"
            checked={todo.finished}
            onClick={() => todo.finished = !todo.finished}
        />{todo.title}
    </li>
)

const store = new TodoList();
ReactDOM.render(<TodoListView todoList={store} />, document.getElementById('mount'));

```

`observer` turns React (function) components into derivations of the data they render. When using MobX there are no smart or dumb components. All components render smartly but are defined in a dumb manner. MobX will simply make sure the components are always re-rendered whenever needed, but also no more than that. So the `onClick` handler in the above example will force the proper `TodoView` to render, and it will cause the `TodoListView` to render if the number of unfinished tasks has changed. However, if you would remove the `Tasks left` line (or put it into a separate component), the `TodoListView` will no longer re-render when ticking a box. You can verify this yourself by changing the [JSFiddle](https://jsfiddle.net/mweststrate/wv3yopo0/).

#### Custom reactions {#custom-reactions}

Custom reactions can simply be created using the [`autorun`](http://mobxjs.github.io/mobx/refguide/autorun.html), [`autorunAsync`](http://mobxjs.github.io/mobx/refguide/autorun-async.html) or [`when`](http://mobxjs.github.io/mobx/refguide/when.html) functions to fit your specific situations.

For example the following `autorun` prints a log message each time the amount of `unfinishedTodoCount` changes:

```
autorun(() => {
    console.log("Tasks left: " + todos.unfinishedTodoCount)
})

```

### What will MobX react to? {#what-will-mobx-react-to}

Why does a new message get printed each time the `unfinishedTodoCount` is changed? The answer is this rule of thumb: _MobX reacts to any existing observable property that is read during the execution of a tracked function._

For an in-depth explanation about how MobX determines to which observables needs to be reacted, check [understanding what MobX reacts to](https://github.com/mobxjs/mobx/blob/gh-pages/docs/best/react.md)

### Actions {#actions}

Unlike many flux frameworks, MobX is unopinionated about how user events should be handled.

*   This can be done in a Flux like manner.
*   Or by processing events using RxJS.
*   Or by simply handling events in the most straightforward way possible, as demonstrated in the above `onClick` handler.

In the end it all boils down to: Somehow the state should be updated.

After updating the state `MobX` will take care of the rest in an efficient, glitch-free manner. So simple statements, like below, are enough to automatically update the user interface.

There is no technical need for firing events, calling dispatcher or what more. A React component is in the end nothing more than a fancy representation of your state. A derivation that will be managed by MobX.

```
store.todos.push(
    new Todo("Get Coffee"),
    new Todo("Write simpler code")
);
store.todos[0].finished = true;

```

Nonetheless, MobX has an optional built-in concept of [`actions`](https://mobxjs.github.io/mobx/refguide/action.html). Use them to your advantage; they will help you to structure your code better and make wise decisions about when and where state should be modified.