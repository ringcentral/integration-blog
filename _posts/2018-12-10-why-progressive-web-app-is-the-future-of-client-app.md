---
layout: post
title:  "Why Progressive Web App is the future of Client app"
date:   2018-12-10 10:00:00 +0800
categories: blog
author: Embbnux Ji
---

![ ](https://cdn-images-1.medium.com/max/1600/1*wf0nzvLDFABh3X60lgOQdw.png)

As React.js and Vue.js become more and more popular, we have a lot single page web application now. They bring great user experience and technology advancements. But what's next. It is Progressive Web Apps. With Progressive Web Apps, you make your web app having great user experience as native app.

## Speed up web client

A big issue of web application is network. User need network to get data or redirect. With poor network or no network, your web app will be broken. Progressive Web Apps make our client work when offline. You can cache assets files or event API request in local caches, so app can still function when network is offline.

We can cache App shell that can be built with React.js in local cahce. So When we use refresh web page, app doesn't have to load JS file from network to reduce blank screen time.

In service worker, we can get fetch event when client creates a network request, so we can return cache data to request when the cache data is not expired. There are many ways to play with requests in service worker. You can even queue your request to avoid rate limit problems. But I think an awesome way is use service worker as a middle layer server.

## Use service worker as middle layer server

In server side, we may use Node.js server as middle layer to merge API requests and send formatted data to frontend. In server worker, we can also build a middle layer.

To create a new API in service worker:

```js
// In service worker
importScripts('https://storage.googleapis.com/workbox-cdn/releases/3.6.1/workbox-sw.js');

// will create a API `/newAPI`
workbox.routing.registerRoute(
  '/newAPI',
  ({url, event}) => {
    const responseData = {
      id: '123',
      name: 'test'
    };
    return new Response(
      JSON.stringify(responseData), {
        headers: { 'Content-Type': 'application/json' }
      }
    );
  }
);
```

Use server worker as middle layer server:

```js
// In service worker
importScripts('https://storage.googleapis.com/workbox-cdn/releases/3.6.1/workbox-sw.js');

// will create a API `/news.json`
workbox.routing.registerRoute(
  '/news.json',
  async ({url, event}) => {
    console.log(url);
    const idsResponse = await fetch('https://hacker-news.firebaseio.com/v0/topstories.json?orderBy=%22$key%22&startAt=%220%22&endAt=%225%22')
    const idMaps = await idsResponse.json()
    const ids = Object.keys(idMaps).map(key => idMaps[key]);
    const newsList = await Promise.all(
      ids.map(async (id) => {
        const response = await fetch(`https://hacker-news.firebaseio.com/v0/item/${id}.json`)
        const news = await response.json()
        return news
      })
    )
    return new Response(
      JSON.stringify(newsList), {
        headers: { 'Content-Type': 'application/json' }
      }
    );
  }
);
```

[Workbox](https://developers.google.com/web/tools/workbox/) is new tool from Google to help build progressive web apps.
For server-side notification, we can use [Push Notification](https://developers.google.com/web/ilt/pwa/introduction-to-push-notifications) api to get notification from server-side in service worker. And use [Client.postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Client/postMessage) to notify client app.

## Other framework like progressive web app

Now progressive web app works well in mobile browser, user can add app to their home screen to use it as a native app in Android and iOS. For some popular apps, they provide some framework to allow developer to embed their cross-platform app, such as WeChat's Mini-Programs, Facebook's instant games and Google's instant apps. They are very similar with progressive web app. 

With those framework, users don't need to install apps, just open it and use it. Those apps are becoming more and more popular, because they are easy to developer and cross-platform. If you are a client developer, you should pay more attention on them.

![ ](https://cdn-images-1.medium.com/max/1600/1*h_D5wqHdJjsDIVLkbDqLNQ.jpeg)

For WeChat's mini-program, it is built with JS and template like HTML. So it gets a lot of developers from Web developers, and grows very quickly. WeChat's mini-program makes it easy to integrate third party service with WeChat.

I think those frameworks are same idea  just with different technology. They are like web application with native enhancement. So it is same as Progressive Web Apps. But I think Progressive web apps are more common. It's a real web app and easier to learn and use.

## Conclusion

Progressive web apps can give our apps great user experience. With progressive web app, we can speed up our web application. We can also use service worker as a server. There are some framework like PWA such as WeChat's Mini-Programs, Facebook's instant games and Google's instant apps. They have been becoming more and more popular, we should also pay more attention at them.

I have run RingCentral JS SDK in service worker, it works well with restful API request, except pubnub. Pubnub's JS SDK doesn't work in service worker because of its http library. We can try to update pubnub sdk codes or use Push notification of service woker.
