---
layout: post
title:  "Reading: Make a Native Web Component with Custom Elements v1 and Shadow DOM v1"
date: "2018-12-28 22:14:00 +0800"
categories: "reading note"
author: "Jackson"
tags: "web component"
---
<a href="https://bendyworks.com/blog/native-web-components" target="_blank">Article Link</a>      
<a href="https://zhuanlan.zhihu.com/p/52802048" target="_blank">中文译版</a>


#### Summary:
This article is mainly talking about the web component standards which includes the HTML Templates, Shadow Dom and Custom Elements.

The author made a form component based on the web component's standards, so this component is just like native HTML elements: they can have properties, methods, and event listeners. The difference between web components and React component is that web components are just implemented by vanilla JavaScript and native HTML, so they don't need any transpilers to translate the code to standard HTML, so they didn't need any extra dependency and will work across modern browsers. They can also be used with any Javascript libraries or frameworks that works with standard HTML. Web components also have the same advantages of React components: reused and shared easily. And as the custom element have lifecycle hooks like React component, you can control the component easily.

And with the Shadow Dom's help, we can build the component that defines its internal structure, scoped CSS, and encapsulates your implementation details. It does not interact with the parent DOM. So the parent's style and the script will not affect your component. And the component's style or behavior will not affect the context in which it is used too. The component will be consistent across all instances of the component, wherever it is used.

#### Reflection:
[Omi](https://github.com/Tencent/omi), a next generation web framework is made by Tencent had started to use the web component. At our projects, we mostly use React components. But in some areas, We can't use React components because of some problems. I think we can use web components in these situation like meetings settings popup, add meeting button and some other string templates are used in Google and Office extensions. This can make the templates more reusable and easier to control.

#### Practice:
In this demo, I try to refactor the meeting popup with web components.
<p data-height="365" data-theme-id="0" data-slug-hash="QzvRMN" data-default-tab="js,result" data-user="jackzong" data-pen-title="web components" class="codepen">See the Pen <a href="https://codepen.io/jackzong/pen/QzvRMN/">web components</a> by JinZhang (<a href="https://codepen.io/jackzong">@jackzong</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>
