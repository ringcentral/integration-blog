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
  * How to do object-oriented programming with Vuex?

Typically front-end architectures are tightly coupled with state management. Once a state management library is selected, it is difficult to switch to another without major refactoring. So any system that uses such architecture will also have to use the same state library. 

Better front-end architecture design should be flexible and scalable. Especially for designs that aim to fulfill integration purposes, where adapting to the target environments and sdk architectures is very important. In order to create modules that work with popular frameworks like React+Redux, React+MobX, Vue+Vuex, and Angular, we need an universal state module design.

## Design Goals

* The design of OOP based on state libraries such as Redux/MobX/Vuex, which is also the most important.
* Whether the encapsulated OOP design is simple and flexible.
* Dependency detection and module lifecycle.

## Proposal

**Based on the concept of universalization, we propose a new universal state module of the library —— [usm](https://github.com/unadlib/usm).**

Firstly, it should be able to solve the OOP design based on the Redux/MobX/Vuex and other state libraries.

This is a typical example of Redux's counter:

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

And Compare another example of counter based on `usm`:

```js
import Module, { state, action } from 'usm';

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

And then I have to admit that Redux is definitely one of the best libraries in the immutable type of state libraries, where I have no intention of discussing some of Redux's defects, and we want to explore how to use Redux for better OOP design. We want the Redux-based model to be more intuitive and concise, just like the OO example of ES6+ 'class' counter mentioned above, if such an OO paradigm is also a generic state model, a better unified state library encapsulation, This will undoubtedly lead to a more flexible and friendly development experience for developers (of course, include easy to read/maintain, etc.).

**Finally, `usm` just solved these problems, and `usm` currently supports Redux/MobX/Vuex/Angular.**


### USM‘s Features

- Universal State Management
- Standardized module lifecycle
- Optional Event System
- Support stateless minimization model
- Support React/Vue/Angular

### USM‘s decorators

`usm` provides decorator `@state` to wrap a variable with a state, and decorator `@action` is used to wrap a function that changes state (the last parameter passed in by the function is always the current state object), which is no different from the OO module of a normal `class`.

### USM‘s module lifecycle

If necessary, `usm` provides five lifecycles that support asynchronous:

- `moduleWillInitialize`
- `moduleWillInitializeSuccess`
- `moduleDidInitialize`
- `moduleWillReset`
- `moduleDidReset`

The order in which they are run is shown in the following flow chart:

![lifecycle](/integration-blog/assets/2019-02-25-practice-oop-to-front-end-universal-state-module/usm_lifecycle.png)


In particular, `usm` provides a lifecycle because in most complex domain module scenarios, startup dependencies between modules are often necessary, but when they are not necessary, their settings can be omitted without having to use them.

### An ideal architecture

![flow chart](/integration-blog/assets/2019-02-25-practice-oop-to-front-end-universal-state-module/flow_chart.png)

In a complex front-end module system, this may be a more typical modular architecture design that contains the following sections:

- Lifecycle
- Store Subscriber
- Event System
- State
- Dependency Modules
- Domain Models

Of course, here is just a scenario, perhaps some architecture using the scenario may be the design of the model's scaling.

## An example of Todo based on `usm`

```js
// if necessary, you can use `usm-redux`/`usm-mobx`/`usm-vuex` with states.
import Module, { state, action } from 'usm'; 

class Todos extends Module {
  @state list = [];

  @action
  addTodo(text, state){
    state.list.push({text});
  }

  @action
  toggle(id, state){
    const todo = state.list.find(item => item.id === id);
    todo.completed = !todo.completed;
  }
}
```

## Conclusion

When you develop using libraries or framework combinations such as React+Redux/React+MobX/Vue+Vuex and so on, it is hoped that `usm` is a nice option for modularization in your application, which may be the important modular jigsaw puzzle that you lack when building libraries such as React/Vue.

In other words, if you use `usm` for OOP architecture design, your system can not only reduce the boilerplate of different state libraries, especially boilerplate more libraries like Redux, which should be very helpful. Most importantly, `usm` can make the modularization of your OOP architecture simple and intuitive, and even `usm` can make your business code compatible with various state libraries, whether Redux/MobX/Vuex or Angular, and if you're using a UI components library that's just as compatible React/Vue/Angular, then your app will quickly and seamlessly use React/Vue/Angular.

Finally, we can ask a question worth thinking about:

> Is it really important to choose a front-end state library from OOP's way?

