---
layout: post
title:  "Change your Chrome Extensions to Firefox Add-ons"
date:   2018-12-25 10:00:00 +0800
categories: blog
author: Embbnux Ji
---

![ ](https://user-images.githubusercontent.com/7036536/50415951-e5f9ac00-0858-11e9-991a-60c3f93f2380.png)

Chrome extension is very popular, there are a lot of great extension for Chrome browser. But how to do if you also want to build Firefox Add-ons. WebExtension API is what you need. It is extension API that been used to build Firefox Add-ons. And it can also help you to build a cross-browser extension.

## What is WebExtension

[WebExtension](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions) is a cross-browser extension API for modern browser like Firefox, Google Chrome, Opera. To following this API, you can just build extension once, and run it everywhere.

## Differences between Firefox and Chrome

Firefox extension is fully compatible with WebExtension API. But there are some differences in Google Chrome Browser.

### 1. Global variables

In Chrome extension, you can get a namespace `chrome` which can be used to use Chrome Extension API. With WebExtension, you should use `browser` to get browser API. But in Firefox, it still provides `chrome` as you used in Chrome.

### 2. Iframe

In Chrome, if you host html file in local extension and inject this page into website using iframe, you can still use full Extension API in it. But in Firefox, this iframe will be same as common  web page, you can not use full extension API in it.

### 3. Extension id

In Chrome, every extension have a unique extension event install in different computer. But in Firefox, extension id would be different for every installing. So remember to use Extension API to get current extension id. Don't use it with hard code.

### 4. Extension API with Promise

In chrome extension, most API is used with callback. But in Firefox, there are built with promise.

### 5. Some different keys in `manifest.json`

Now you also need to build different manifest file for different browser.

You can get more detail differences in [here](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Porting_a_Google_Chrome_extension).

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

If you follow WebExtension API when you are building chrome extension, your chrome extension would be migrated to Firefox Add-ons with just little work. With WebExtension API, you can also use promise API to make your codes clean. But remember to handle differences.
