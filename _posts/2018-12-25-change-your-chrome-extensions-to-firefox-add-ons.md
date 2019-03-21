---
layout: post
title:  "Change your Chrome Extensions to Firefox Add-ons"
date:   2018-12-25 10:00:00 +0800
categories: blog
author: Embbnux Ji
---

![ ](https://user-images.githubusercontent.com/7036536/50415951-e5f9ac00-0858-11e9-991a-60c3f93f2380.png)

Chrome extension is very popular, there are a lot of great extension for Chrome browser. But what do you do if you also want to build Firefox Add-ons. WebExtension API is what you need. It is the extension API that has been used to build Firefox Add-ons. And it can also help you to build a cross-browser extension.

## What is WebExtension

[WebExtension](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions) is a cross-browser extension API for modern browser like Firefox, Google Chrome, Opera. To following this API, you can just build extension once, and run it everywhere.

## Differences between Firefox and Chrome

Firefox extension is fully compatible with WebExtension API. But there are some differences in Google Chrome Browser.

### 1. Global variables

In Chrome extension, you can get a namespace `chrome` which can be used to use Chrome Extension API. With WebExtension, you should use `browser` to get browser API. But in Firefox, it still provides `chrome` as you used in Chrome.

### 2. Iframe

In Chrome, if you host html file in local extension and inject this page into website using iframe, you can still use full Extension API in it. But in Firefox, this iframe will be same as common  web page, you can not use full extension API in it.

### 3. Extension id

In Chrome, every extension has a unique extension id that is the same when installed on different computers. But for Firefox, the extension id will be different for every installation. So remember to use Extension API to get current extension id. Don't hard code the extension id.

### 4. Extension API with Promise

In chrome extension, most API is used with callbacks. But in Firefox, there are built with promises.

### 5. Some different keys in `manifest.json`

Now you also need to build different manifest file for different browser.

You can get more detailed differences in [here](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Porting_a_Google_Chrome_extension).

## Library that help to build WebExtension

You can use `webextension-polyfill` [library](https://github.com/mozilla/webextension-polyfill) to help you avoid some differences between Chrome and Firefox.

With `webextension-polyfill` you can also use browser namespace in chrome extension with promise.

```js
var browser = require("webextension-polyfill");
browser.tabs.sendMessage(tabId, "get-ids").then(results => {
  processResults(results);
});
```

## Conclusion

If you follow WebExtension API when you are building chrome extension, your chrome extension would be able to be migrated to Firefox Add-ons with just a little work. With WebExtension API, you can also use promise API to make your code clean. But remember to handle the differences.

------

------

Chinese Version:

![ ](https://user-images.githubusercontent.com/7036536/50415951-e5f9ac00-0858-11e9-991a-60c3f93f2380.png)

Chrome 扩展插件开发非常流行，现在也存在很多受欢迎的 Chrome 浏览器插件。但是如果想开发 Firefox 浏览器插件，又该如何？WebExtension API 正是你所需要的。这是用来构建 Firefox 插件的标准 API，同样也能够帮助构建跨浏览器插件。

## 一、什么是 WebExtension？

WebExtension 是现代浏览器（比如 Chrome, Firefox 和 Opera）用于构建跨浏览器插件的 API。依据这个 API，可以实现只开发一次插件，到处安装。


## 二、在 Firefox 和 Chrome 上的区别

Firefox 插件对 WebExtension API 是完全兼容的。对于 Chrome 浏览器还是有一些区别。

### 1. 全局变量

在 Chrome 插件中，我们可以拿到 `chrome` 这个全局命名空间，用来调用浏览器的一些高端 API。在 WebExtension，我们应该使用 `browser` 去请求浏览器 API。不过在 Firefox 中，同样支持 `chrome`. 不过还是推荐用 `browser`.

### 2. Iframe

在 Chrome extension 中，如果使用 extension 本地的 HTML 文件利用 Iframe 去插入到网页中，在 Iframe 里同样可以使用插件独有的一些 API. 但在 Firefox 里, iframe 内的网页就只能使用和普通网页相同的 API，拿不到插件独有的一些API。

### 3. Extension ID

在 Chrome 浏览器中，每个 extension 都有一个独一无二的 ID，这个 ID 基于 extension, 在不同的用户和电脑中安装都能得到相同的 ID. 但在 Firefox 中，这个 ID 并不是固定的，不同用户或者重新安装都会得到一个新的。所以只能 Extension 提供的 API 去拿 ID。

### 4. Extension API with Promise

在 Chrome extension 中，基于 chrome 的 extension API 都是使用 Callback 来进行异步处理。而在 Firefox 中，这个情况得到改进，这个异步 API 返回的是 Promise。

### 5. `manifest.json` 中不同的属性

Chrome extension 和 Firefox add-on 都用 `manifest.json` 来定义 extension 的行为，不过还是有一些存在差别的属性。

可以在这个[页面](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Porting_a_Google_Chrome_extension)找到详细的差别。


## 三、可以用来助力构建 Extension 的库

Mozilla 提供了 `webextension-polyfill` 这个[库](https://github.com/mozilla/webextension-polyfill) 可以用来帮助我们构建标准的 Web Extension. 使用这个库在 Chrome 和 Firefox 中都可以使用 `browser` 关键字来调用浏览器 API。

```js
var browser = require("webextension-polyfill");
browser.tabs.sendMessage(tabId, "get-ids").then(results => {
  processResults(results);
});
```

## 总结

如果你正在进行构建 Chrome extension, 那么推荐基于 Webextension API 来进行开发，这样当你想也做 Firefox add-on 时会变得非常简单，水到渠成。同样适用 Webextension API 可以避免用 Callback 来调用异步接口，而可以使用 Promise 来调用 异步接口。 不过在 Chrome 转 Firefox 中还是有一些差异要处理。
