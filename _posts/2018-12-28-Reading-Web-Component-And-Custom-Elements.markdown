---
layout: post
title:  "Reading: Make a Native Web Component with Custom Elements v1 and Shadow DOM v1"
date: "2018-12-28 22:14:00 +0800"
categories: "reading note"
author: "Jackson"
tags: "web component"
---

[Article Link](https://bendyworks.com/blog/native-web-components)   
[中文版本](https://zhuanlan.zhihu.com/p/52802048)

#### Summary:
This article is mainly talking about the web component standards which includes the HTML Templates, Shadow Dom and Custom Elements.

The author made a form component base on the web component's standards, so this component just like native HTML elements, they can have properties, methods, and event listeners. Just the different with React component, web component didn't need any translation engine to translate the code to standard HTML, It just made by vanilla JavaScript and the native HTML. So it doesn't need any dependency. And it will work across modern browsers and it can be used with any JavaScript library or framework that works with HTML. And with the advantage of React component reuse and share easily. And the custom element has lifecycle hooks too, so you can control the component easily.

And with the Shadow Dom's help, we can build the component which defines its internal structure, scoped CSS, and encapsulates your implementation details. It does not interact with the parent DOM. So the parent's style and the script will not affect your component. And the component's style or behavior will not affect the context in which it is used too. And The component will be consistent across all instances of the component, wherever it is used.

#### Reflection:
[Omi](https://github.com/Tencent/omi) which the next generation web framework is powered by Tencent had started to use the web component. At our projects, we mostly use the react component. But in some area, We can't use the React Component because of some problem. And I think that we can try to use the web components in this situation, just like the meeting setting popup, add meeting button and some other string template at the extension of Google and Office. To make the template more reusable and easy control.

#### Practice:
In this demo, I try to refactor the meeting popup with web component.
<p data-height="365" data-theme-id="0" data-slug-hash="QzvRMN" data-default-tab="js,result" data-user="jackzong" data-pen-title="web components" class="codepen">See the Pen <a href="https://codepen.io/jackzong/pen/QzvRMN/">web components</a> by JinZhang (<a href="https://codepen.io/jackzong">@jackzong</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>
