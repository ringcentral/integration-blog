---
layout: post
title:  "Practice OOP to front-end universal state module"
date:   2019-02-25 12:30:00 +0800
categories: blog
author: Michael Lin
---


![architecture](/integration-blog/assets/2019-02-25-practice-oop-to-front-end-universal-state-module/architecture.jpg)

> This is a proposal of an universal state management module design rooted in the OOP paradigm.

## Motivation


As front-end single-page application development becomes increasingly complex, we have to use some state management or state containers (collectively referred to as state library) in order to develop complex apps, and we need a model design that is easier to modularize.

State management libraries are quite abundant in the front-end space, there are [Redux](https://github.com/reduxjs/redux), [MobX](https://github.com/mobxjs/mobx), [Vuex](https://github.com/vuejs/vuex), and the self-contained state management that comes with [Angular](https://github.com/angular/angular) to name a few. Redux is a predictable state container with immutable data structure. MobX is a observable state management. Vuex is centralized state management with observable pattern for Vue.js. As for modularization, Angular has its own implementations already, but the rest of the state management libraries only started to deal with this new requirement in complex systems in recent years.

In this article, let's explore an OOP modular design that has universal support for popular state management libraries.

## Universal state module

Object-oriented programming(OOP) is commonly used in the architecture design of large projects in the front-end. The following questions are often asked when deciding on a state management library:

  * Is it Redux or MobX more suitable for React?
  * Is Redux suitable for OOP?
  * What are the pros and cons of using MobX's observable in React?
  * How to do OOP with Vuex?

Typically front-end architectures are tightly coupled with state management. Once a state management library is selected, it is difficult to switch to another without major refactoring. So any system that uses such architecture will also have to use the same state library. 

Better front-end architecture design should be flexible and scalable. Especially for designs that aim to fulfill integration purposes, where adapting to the target environments and sdk architectures is very important. In order to create modules that work with popular frameworks like React+Redux, React+MobX, Vue+Vuex, and Angular, we need an universal state module design.

## Design Goals

* Object Oriented
* Simple & Flexible
* Dependency Detection & Module Lifecycle

## Proposal

**Based on the concept of universalization, we propose a new `Universal State Module` library —— [usm](https://github.com/unadlib/usm).**

Let's start with a typical Redux example of a counter:

```js
import { createStore } from 'redux';

function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    case 'DECREMENT':
      return state - 1;
    default:
      return state;
  }
}

const store = createStore(counter)

store.dispatch({ type: 'INCREMENT' })
store.dispatch({ type: 'DECREMENT' })
```

USM supports Redux, MobX, Vuex and Angular. It provides `usm`, `usm-redux`, `usm-mobx` and `usm-vuex` packages. Here is the same counter using `usm-redux`:

```js
import Module, { state, action } from 'usm-redux';

class Counter extends Module {
  @state count = 0;

  @action
  increase(state) {
    state.count += 1;
  }

  @action
  decrease(state) {
    state.count -= 1;
  }
}

const counter = Counter.create();

counter.increase();
counter.decrease();
```

The implementation of the same `counter` above is based on object-oriented paradigm. The use of ES6 class syntax is intuitive and concise. If this design can be universal to any state management library used, it will undoubtedly lead to more flexible and friendly development experience for developers, as well as better readability and maintainability.

USM has four sub-packages, namly `usm`, `usm-redux`, `usm-mobx` and `usm-vuex`. `usm-redux` is used in this example, which is based on the use of [Immer](https://github.com/mweststrate/immer) that enables modifying the immutable redux state in a mutable manner.

The following code demonstrate how the `usm-redux` module can be used with the connector from `react-redux`:

```js
// index.js
export const counter = Counter.create();

ReactDOM.render(
  <Provider store={counter.store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

```js
// app.js
import { connect } from 'react-redux';
import { counter } from './';

export default connect(
  state => ({ count: state.count })
)( props => 
  <div>
    <button onClick={() => counter.increase()}>+</button>
    {props.count}
    <button onClick={() => counter.decrease()}>-</button>
  </div>
);
```

And here is the same counter working with `mobx-react` using `usm-mobx`:


```js
// index.js

export const counter = Counter.create();

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```


```js
// app.js
import { observer } from 'mobx-react';
import { counter } from './';

export default observer(() =>
  <div>
    <button onClick={() => counter.increase()}>+</button>
    {counter.count}
    <button onClick={() => counter.decrease()}>-</button>
  </div>
);
```

The use of `usm-redux` and `usm-mobx` to connect with `react-redux` and `mobx-react` respectfully demonstrated that the core implementations of the state module is the same even when the connectors used is different. This is the core principle of the Universal State Module that we propose.

** USM currently supports Redux, MobX, Vuex and Angular.**

### Features

- Universal State Management
- Standardized Module Lifecycle
- Optional Event System
- Support Stateless Model
- Support React/Vue/Angular

### Decorators

`usm` provides decorator `@state` to wrap a variable with a state, and decorator `@action` is used to wrap a function that changes state.

```js
class Shop extends Module {
  @state goods = [];
  @state status = 'close';

  @action
  operate(item, status, state) {
    state.goods.push(item);
    state.status = status;
  }
  // call function -> this.operate({ name: 'fruits', amount: 10 }, 'open');
}
```

### Module lifecycle

`usm` provides these lifecycle events:

- `moduleWillInitialize`
- `moduleWillInitializeSuccess`
- `moduleDidInitialize`
- `moduleWillReset`
- `moduleDidReset`

The order in which they are run is shown in the following flow chart:

![lifecycle](/integration-blog/assets/2019-02-25-practice-oop-to-front-end-universal-state-module/usm_lifecycle.png)

These module lifecycles can be used to coordinate module initialization dependencies.

```js
class TodoList extends Module {  
  async moduleWillInitialize() {
    console.log(
      'TodoList -> moduleWillInitialize',
      `pending: ${this.pending}`, `ready: ${this.ready}`
    );
  }

  async moduleWillInitializeSuccess() {
    console.log(
      'TodoList -> moduleWillInitializeSuccess',
      `pending: ${this.pending}`, `ready: ${this.ready}`
      );
  }

  async moduleDidInitialize() {
    console.log(
      'TodoList -> moduleDidInitialize',
      `pending: ${this.pending}`, `ready: ${this.ready}`
      );
  }
}


class App extends Module{
  async moduleWillInitialize() {
    console.log(
      'App -> moduleWillInitialize',
      `pending: ${this.pending}`, `ready: ${this.ready}`
    );
  }

  async moduleWillInitializeSuccess() {
    console.log(
      'App -> moduleWillInitializeSuccess',
      `pending: ${this.pending}`, `ready: ${this.ready}`
    );
  }

  async moduleDidInitialize() {
    console.log(
      'App -> moduleDidInitialize',
      `pending: ${this.pending}`, `ready: ${this.ready}`
    );
  }
}

const todoList = new TodoList();

const app = App.create({
  modules: {
    todoList
  }
});
```
console.log results:

```
App -> moduleWillInitialize pending: false ready: false
TodoList -> moduleWillInitialize pending: false ready: false
TodoList -> moduleWillInitializeSuccess pending: true ready: false
App -> moduleWillInitializeSuccess pending: true ready: false
TodoList -> moduleDidInitialize pending: false ready: true
App -> moduleDidInitialize pending: false ready: true
```

### An ideal architecture

![flow chart](/integration-blog/assets/2019-02-25-practice-oop-to-front-end-universal-state-module/flow_chart.png)

In a complex front-end application, a typical modular architecture may contain the following:

- Lifecycle
- Store Subscriber
- Event System
- State
- Module dependency
- Domain Models

## Conclusion

`usm` is a module design that wants to bridge together the differences of using Redux, Mobx, and Vuex in conjuction with different view layers such as React, Vue and Angular. It is designed to help you build libraries that will work with any front-end architecture.

Modules built with `usm` should be free of boilerplates, especially the type that is introduced by state libraries like Redux. More importantly, the object-oriented nature of `usm` makes modules simple and intuitive. `usm` also makes your modules compatible with various state libraries and view layers, allowing you to share your business logic libraries across projects regardless of the frameworks they are using.

USM's repo: [https://github.com/unadlib/usm](https://github.com/unadlib/usm)
