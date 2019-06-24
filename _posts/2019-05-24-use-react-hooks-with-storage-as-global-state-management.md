---
layout: post
title:  "Use React Hooks with storage as global state management"
date:   2019-05-24 10:00:00 +0800
categories: blog
author: Embbnux Ji
---

![react hooks with storage](https://cdn-images-1.medium.com/max/800/1*b9_KoH-ShP-JQROsrQANjw.png)

React Hooks give us a new way to manage state in React. But how to manage global state as redux and how to persist state? This article will show how to use hook as global state management by storage.

Reference the following GitHub repos for additional code:

* Hook library: [use-global-storage](https://github.com/embbnux/use-global-storage)
* Storage library: [rt-storage](https://github.com/embbnux/rt-storage)
* Demo: [online](https://embbnux.github.io/use-global-storage-demo/), project: [use-global-storage-demo](https://github.com/embbnux/use-global-storage-demo)

## React Hooks Introduce

[Hooks](https://reactjs.org/docs/hooks-intro.html) was added in React 16.8. With hooks, we can use state and other React features without writing a class.

With `useState`, we can read and setState in function component:

```js
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}

```

With `useEffect`, we can run some function as you do in React class lifecycle methods. We can think it as `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount` combined.

```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

## Use global storage library

In this article, I will persist React state into storage, and use storage to broadcast and sync state between components and browser tabs. So we need a storage solution that support those features. localStorage is a good choice. But in some browsers, `localStorage` only supports 5m data. For indexedDB, it supports unlimited storage space, but doesn’t have event between tabs.

In [rt-storage](https://github.com/embbnux/rt-storage), it uses localStorage to broadcast event in different tabs, uses indexedDB to persist data. It is based on localforage and rxjs:

```js
import RTStorage from 'rt-storage';

const storage = new RTStorage({ name: 'test-db' });
const subscription = storage.subscribe((event) => {
  console.dir(event)
});
const storageKey = 'storageKey';
storage.getItem(storageKey).then(a => console.log(a));
storage.setItem(storageKey, data).then(() => console.log('set successfully'));
subscription.unsubscribe();
```

For more details, please check the project [repo](https://github.com/embbnux/rt-storage).

## Use storage hook

So let’s start to create a customized react hook with global storage. Following is the core source code of [use-global-storage](https://github.com/embbnux/use-global-storage):

```js
import { useState, useEffect } from 'react';
import * as RTStorage from 'rt-storage';
export default function useGlobalStorage({ storageOptions } : { storageOptions: any }) {
  const storage = new RTStorage(storageOptions);
  const useStorage = (key: string, initialData: any) => {
    const [data, setState] = useState(initialData);
    useEffect(() => {
      function handleStorageChange(data) {
        setState(data);
      }
      storage.getItem(key).then(lastData => {
        if (lastData) {
          setState(lastData);
        }
      });
      const subscription = storage.subscribe(key, handleStorageChange);
      return () => {
        subscription.unsubscribe();
      };
    }, []);
    const setData = async(newData: any) => {
      let newValue;
      if (typeof newData === 'function') {
        newValue = newData(data);
      } else {
        newValue = newData
      }
      setState(newValue);
      await storage.setItem(key, newValue);
    }
  
    return [data, setData];
  }
  return useStorage;
};
```

It will return a hook function which allow user to set state into storage and sync for other components and tabs. setData function will set state into react state and save data into storage. And we use `useEffect` to subscribe storage changed event.

Example to use `use-global-storage` hook in project:

```js
import useGlobalStorage from 'use-global-storage';

const useStorage = useGlobalStorage({
  storageOptions: { name: 'test-db' }
});

const Counter = () => {
  const [state, setState] = useStorage('counter');
  return (
    <div className="Counter">
      <p>
        Counter:
        {state || 0}
      </p>
      <button type="button" onClick={() => setState(state + 1)}>
        +1 to global
      </button>
    </div>
  );
};
```

It is very simple to use it. Just get `state` and `setState` from `useStorage` hook, and your component is connected with storage data. You can get the online demo page [here](https://embbnux.github.io/use-global-storage-demo/). To get full code in this Github [repo](https://github.com/embbnux/use-global-storage-demo).

## Conclusion

This article provides a new way to manage global state with react hook and storage. Hope you will like this library: [use-global-storage](https://github.com/embbnux/use-global-storage).
