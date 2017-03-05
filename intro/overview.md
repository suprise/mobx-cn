# Mobx的要点

虽然听起来可能很魔幻，但使一个APP变成可响应的只需要3步：

## 1. 定义你的状态并且使其可观察

在任何你喜欢的数据结构中存储状态，对象、数组、类、循环引用的数据结构、引用，都可以。
只需要确保那些你希望随时改变的属性被标记，以使其可观察


```javascript
import {observable} from 'mobx';

var appState = observable({
    timer: 0
});
```

## 2. 创建一个View响应状态的变化

你现在可以创建一个视图（View），只要`appState` 中的数据发生变化，这个视图可以自动更新。Mobx会找到成本最小的方式更新你的视图。
这个简单的方法节省了你成吨的脚手架，并且[非常有效~](https://mendix.com/tech-blog/making-react-reactive-pursuit-high-performing-easily-maintainable-react-apps/).

通常来说，任何函数都可以变成一个观察关联数据的可响应的视图。Mobx可以用于ES5的环境，但这里也有一个ES6的例子。

```javascript
import {observer} from 'mobx-react';

@observer
class TimerView extends React.Component {
    render() {
        return (<button onClick={this.onReset.bind(this)}>
                Seconds passed: {this.props.appState.timer}
            </button>);
    }

    onReset () {
        this.props.appState.resetTimer();
    }
};

React.render(<TimerView appState={appState} />, document.body);
```

(`resetTimer` 方法的实现见下一部分)

## 3. 改变状态

第三件要做的事情就是改变状态。这是你的应用只要要做的全部事情。与其他框架不同，Mobx并不限制你的方法。
这里有很多最佳实践，但请记住这个关键点，
***Mobx帮助你用最简单直观的方式去做事***

下面这个代码会改变你的数据每秒钟，并且UI会在需要的时候自动更新
你不需要定义控制器（controller）函数去改变状态，或者让视图变得需要更新，Mobx会自动检测所有的关系。
这是改变状态的两个例子：

```javascript
appState.resetTimer = action(function reset() {
    appState.timer = 0;
});

setInterval(action(function tick() {
    appState.timer += 1;
}), 1000);
```

使用`action` 包裹函数，只有在严格模式下才需要（默认是关闭状态）
但推荐使用action，因为它会帮助你更好地构建你的应用、并且表达你的意图——你的这个行数是用于改变状态的。
并且会自动应用事务行为（transaction）以优化性能。

可以尝试这个例子[JSFiddle](http://jsfiddle.net/mweststrate/wgbe4guu/)，或者clone [MobX boilerplate project](https://github.com/mobxjs/mobx-react-boilerplate)