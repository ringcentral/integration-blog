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

We want the Redux-based model to be more intuitive and concise, just like the OO example of ES6+ 'class' counter mentioned above, if such an OO paradigm is also a generic state model, a better unified state library encapsulation, This will undoubtedly lead to a more flexible and friendly development experience for developers (of course, include easy to read/maintain, etc.).

USM has four sub-packages, which are `usm`, `usm-redux`, `usm-mobx` and `usm-vuex`. In this example, it mainly uses `usm-redux`. `usm-redux` is mainly based on Redux and [Immer](https://github.com/mweststrate/immer). Immer is mainly used in mutable modifying created to get immutable state.

The following is the connector section using `usm-redux`:

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
  state => ({ ount: state.count })
)( props => 
  <div>
    <button onClick={() => counter.increase()}>+</button>
    {props.count}
    <button onClick={() => counter.decrease()}>-</button>
  </div>
);
```

And here is the same counter `View` connector using `usm-mobx`:

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

In the `usm-redux` and `usm-mobx` examples, in addition to some differences in the View connector, the state module part is in accord, and this is the universal state module that we had proposed.

** USM currently supports Redux, MobX, Vuex and Angular.**

### Features

- Universal State Management
- Standardized Module Lifecycle
- Optional Event System
- Support Stateless Model
- Support React/Vue/Angular

### Decorators

`usm` provides decorator `@state` to wrap a variable with a state, and decorator `@action` is used to wrap a function that changes state (the last parameter passed in by the function is always the current state object).

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
      `pendding: ${this.pending}`, `ready: ${this.ready}`
    );
  }

  async moduleWillInitializeSuccess() {
    console.log(
      'TodoList -> moduleWillInitializeSuccess',
      `pendding: ${this.pending}`, `ready: ${this.ready}`
      );
  }

  async moduleDidInitialize() {
    console.log(
      'TodoList -> moduleDidInitialize',
      `pendding: ${this.pending}`, `ready: ${this.ready}`
      );
  }
}


class App extends Module{
  async moduleWillInitialize() {
    console.log(
      'App -> moduleWillInitialize',
      `pendding: ${this.pending}`, `ready: ${this.ready}`
    );
  }

  async moduleWillInitializeSuccess() {
    console.log(
      'App -> moduleWillInitializeSuccess',
      `pendding: ${this.pending}`, `ready: ${this.ready}`
    );
  }

  async moduleDidInitialize() {
    console.log(
      'App -> moduleDidInitialize',
      `pendding: ${this.pending}`, `ready: ${this.ready}`
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
console.log reulst:

```
App -> moduleWillInitialize pendding: false ready: false
TodoList -> moduleWillInitialize pendding: false ready: false
TodoList -> moduleWillInitializeSuccess pendding: true ready: false
App -> moduleWillInitializeSuccess pendding: true ready: false
TodoList -> moduleDidInitialize pendding: false ready: true
App -> moduleDidInitialize pendding: false ready: true
```

### An ideal architecture

![flow chart](/integration-blog/assets/2019-02-25-practice-oop-to-front-end-universal-state-module/flow_chart.png)

In a complex front-end application, this may be a more typical modular architecture design that contains the following sections:

- Lifecycle
- Store Subscriber
- Event System
- State
- Dependency Modules
- Domain Models

## Conclusion

When you develop using libraries or framework combinations such as React+Redux/React+MobX/Vue+Vuex and so on, it is hoped that `usm` is a nice option for modularization in your application, which may be the important modular jigsaw puzzle that you lack when building libraries such as React/Vue.

In other words, if you use `usm` for OOP architecture design, your system can not only reduce the boilerplate of different state libraries, especially boilerplate more libraries like Redux, which should be very helpful. Most importantly, `usm` can make the modularization of your OOP architecture simple and intuitive, and even `usm` can make your business code compatible with various state libraries, whether Redux/MobX/Vuex or Angular, and if you're using a UI components library that's just as compatible React/Vue/Angular, then your app will quickly and seamlessly use React/Vue/Angular.

Finally, we can ask a question worth thinking about:

> Is it really important to choose a front-end state library from OOP's way?

USM's repo: [https://github.com/unadlib/usm](https://github.com/unadlib/usm)
