---
layout: post
title:  "Practice OOP to front-end universal state module"
date:   2019-02-25 12:30:00 +0800
categories: blog
author: Michael Lin
---


![architecture](/integration-blog/assets/2019-02-25-practice-oop-to-front-end-universal-state-module/architecture.jpg)

> If you are interested in how to better oop design such as Redux/MobX/Vuex, etc. And then this article will propose a complete universal solution for the front-end state library's OOP design.

## Motivation


As front-end single-page application development becomes increasingly complex, when we use React/Vue, we have to use some state management or state containers (collectively referred to as state library) in order to develop complex apps, and we  need a model design that is easier to modularize. And the front-end state library is very prosperous, whether it is Redux/MobX/Vuex and Angular self-contained state management, the modularization of the state library has been a new requirement in the field of front-end development in complex systems in recent years. Of course, Angular this requirement has been implemented by the angular framework itself already, but it is a very important problem for other libraries, so this article tries to explore a set of OOP modular designs that are universal to popular state libraries.

## Universal state module

Normally, object-oriented programming (OOP) is commonly used in the architecture design of large projects in the front-end, and questions such as which front-end state libraries are often turned into controversial focal points:

  * Is it Redux or MobX more suitable for React?
  * Is Redux suitable for OOP?
  * How does MobX's observable weigh the pros and cons in React?
  * How do Vuex OOPin Vue?
  * ..., etc.

In addition, Universal JavaScript is more for JavaScript's perform environment in most cases, and once a architecture design selects a state library, it will mean that this architecture is difficult to disengage from the use of this state library, and any system based on this architecture will be based on such a state library. But a better front-end architecture would include more flexible options and scalability, especially the front-end parts of generic requirements such as typical integration business, which can be reflected in the choice of view render libraries and even the availability of state libraries, such as in mainstream scenarios React+Redux/React+MobX/Vue+Vuex/Angular and so on have a choice, then it brings the problem is how to solve universal state module.

## Specific issues that need to be solved


* The design of OOP based on state libraries such as Redux/MobX/Vuex, which is also the most important, especially for React/Vue.
* Whether the encapsulated OOP design is simple enough and easy to use, while they are quite flexible.
* From DDD's way, the dependencies between complex domain modules require IoC, and the startup logic between them is dependent, so there must be a similar event mechanism or the introduction of the module lifecycle.

> to solve these issues, universal of the OOP encapsulation and module standardization lifecycle or event mechanism becomes indispensable.

## Propose the solution

Based on the concept of universalization, we propose a new universal state module of the library —— **[usm](https://github.com/unadlib/usm)**。

Firstly, it should be able to solve the OOP design based on the Redux/MobX/Vuex and other state libraries.


This is a typical example of Redux's counter:

```js
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

store.subscribe(() => console.log(store.getState()))

store.dispatch({ type: 'INCREMENT' })
store.dispatch({ type: 'DECREMENT' })
```

And this is an example of counter based on `usm`:

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
```

I have to admit that Redux is definitely one of the best libraries in the immutable type of state libraries, where I have no intention of discussing some of Redux's defects, and we want to explore how to use Redux for better OOP design. We want the Redux-based model to be more intuitive and concise, just like the OO example of ES6+ 'class' counter mentioned above, if such an OO paradigm is also a generic state model, a better unified state library encapsulation, This will undoubtedly lead to a more flexible and friendly development experience for developers (of course, include easy to read/maintain, etc.).

**`usm` just solved these problems, and `usm` currently supports Redux/MobX/Vuex/Angular.**


## USM‘s Features

- Universal State Management
- Standardized module lifecycle
- Optional Event System
- Support stateless minimization model
- Support React/Vue/Angular

### USM‘s decorators

`usm`提供`@state`用于包装一个带状态的变量，`@action`用于包装一个改变状态的函数(函数传入的最后一个参数均为当前state对象)，除此以外和一个普通的`class`封装的OO模块没有区别。

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

## Simple example about Todo

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

> Is it really important to choose the front-end State Library from OOP's way?

