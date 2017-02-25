## MobX: Simple and scalable {#mobx-simple-and-scalable}

MobX is one of the least obtrusive libraries you can use for state management. That makes the `MobX` approach not just simple, but very scalable as well:

### Using classes and real references {#using-classes-and-real-references}

With MobX you don&#039;t need to normalize your data. This makes the library very suitable for very complex domain models (At Mendix for example ~500 different domain classes in a single application).

### Referential integrity is guaranteed {#referential-integrity-is-guaranteed}

Since data doesn&#039;t need to be normalized, and MobX automatically tracks the relations between state and derivations, you get referential integrity for free. Rendering something that is accessed through three levels of indirection?

No problem, MobX will track them and re-render whenever one of the references changes. As a result staleness bugs are a thing of the past. As a programmer you might forget that changing some data might influence a seemingly unrelated component in a corner case. MobX won&#039;t forget.

### Simpler actions are easier to maintain {#simpler-actions-are-easier-to-maintain}

As demonstrated above, modifying state when using MobX is very straightforward. You simply write down your intentions. MobX will take care of the rest.

### Fine grained observability is efficient {#fine-grained-observability-is-efficient}

MobX builds a graph of all the derivations in your application to find the least number of re-computations that is needed to prevent staleness. &quot;Derive everything&quot; might sound expensive, MobX builds a virtual derivation graph to minimize the number of recomputations needed to keep derivations in sync with the state.

In fact, when testing MobX at Mendix we found out that using this library to track the relations in our code is often a lot more efficient than pushing changes through our application by using handwritten events or &quot;smart&quot; selector based container components.

The simple reason is that MobX will establish far more fine grained &#039;listeners&#039; on your data than you would do as a programmer.

Secondly MobX sees the causality between derivations so it can order them in such a way that no derivation has to run twice or introduces a glitch.

How that works? See this [in-depth explanation of MobX](https://medium.com/@mweststrate/becoming-fully-reactive-an-in-depth-explanation-of-mobservable-55995262a254).

### Easy interoperability {#easy-interoperability}

MobX works with plain javascript structures. Due to its unobtrusiveness it works with most javascript libraries out of the box, without needing MobX specific library flavors.

So you can simply keep using your existing router, data fetching and utility libraries like `react-router`, `director`, `superagent`, `lodash` etc.

For the same reason you can use it out of the box both server- and client side, in isomorphic applications and with react-native.

The result of this is that you often need to learn fewer new concepts when using MobX in comparison to other state management solutions.

<center>![](https://www.mendix.com/styleguide/img/logo-mendix.png) __MobX is proudly used in mission critical systems at [Mendix](https://www.mendix.com)__</center>