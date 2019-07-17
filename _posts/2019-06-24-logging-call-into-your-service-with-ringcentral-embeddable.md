---
layout: post
title:  "Logging call into your service with RingCentral Embeddable"
date:   2019-06-24 10:00:00 +0800
categories: blog
author: Embbnux Ji
---

![hubspot call logging with Embeddable](https://github.com/ringcentral/hubspot-embeddable-ringcentral-phone/raw/master/docs/img/screenshots/hs6.png)

In previous articles, we showed how to use RingCentral Embeddable to integrate RingCentral services into your website. After integrating the web phone feature, you must be wanting to log calls into your services. This article will show you how to log call information with RingCentral Embeddable's third party service API.

Related Links:

* RingCentral Embeddable: [Github Repository](https://github.com/ringcentral/ringcentral-embeddable)
* Third Party service API: [Document](https://github.com/ringcentral/ringcentral-embeddable/blob/master/docs/third-party-service-in-widget.md#log-call-into-your-service)


## Integrate RingCentral Embeddable

We provide and `Adapter JS file` to help developers to integrate RingCentral Embeddable. You just need to add the following script into your website's header:

```html
<script>
  (function() {
    var rcs = document.createElement("script");
    rcs.src = "https://ringcentral.github.io/ringcentral-embeddable/adapter.js";
    var rcs0 = document.getElementsByTagName("script")[0];
    rcs0.parentNode.insertBefore(rcs, rcs0);
  })();
</script>
```

This script will inject a RingCentral widget into your website, so your customers can login with RingCentral account to access RingCentral services:

![RingCentral Embeddable widget](https://ringcentral-web-widget-demos.readthedocs.io/en/latest/static_crm/tutorial/static_crm_demo.png)

## Register call logging service into Embeddable

The `Adapter JS` will create an iframe in webpage to embed RingCentral Embeddable. And we can interact with RingCentral Embeddable by using the iframe message API.

Firstly, we need to register the call logging service into Embeddable:

```js
document.querySelector("#rc-widget-adapter-frame").contentWindow.postMessage({
  type: 'rc-adapter-register-third-party-service',
  service: {
    name: 'TestService',
    callLoggerPath: '/callLogger',
    callLoggerTitle: 'Log to TestService',
  }
}, '*');
```

After registration, we can get a `Log to TestService` button in calls page, and `Auto log calls` setting in settings page:

![RingCentral Embeddable log to service](https://user-images.githubusercontent.com/7036536/48827686-d1814a00-eda8-11e8-81e4-2b48b1df2bcc.png)

## Respond to RingCentral Embeddable's event message

When user clicks `Log` button in calls page, the widget will send a message including call info to its parent window. So we just need to add a event listener:

```js
window.addEventListener('message', function (e) {
  var data = e.data;
  if (data && data.type === 'rc-post-message-request') {
    if (data.path === '/callLogger') {
      // add your codes here to log call to your service
      console.log(data);
      // response to widget
      document.querySelector("#rc-widget-adapter-frame").contentWindow.postMessage({
        type: 'rc-post-message-response',
        responseId: data.requestId,
        response: { data: 'ok' },
      }, '*');
    }
  }
```

If user enables `Auto log calls` in settings, this event will be also fired when a call is started and updated. In this message event, you can get call information in `data.body.call`. When call is recorded and recording file is generated, you can get recording data in `data.body.call`:

```js
{
  contentUri: "https://media.devtest.ringcentral.com/restapi/v1.0/account/170848004/recording/6469338004/content"
  id: "6469338004"
  link: "http://apps.ringcentral.com/integrations/recording/sandbox/?id=Ab7937-59r6EzUA&recordingId=6469338004"
  type: "OnDemand"
  uri: "https://platform.devtest.ringcentral.com/restapi/v1.0/account/170848004/recording/6469338004"
}
```

The `link` property in `recording` is a link to get and play recording file from RingCentral server. The `contentUri` is a URI which can be used to get `recording` file with RingCentral access token. If you pass `recordingWithToken` when register service, you can get contentUri with access_token. The access_token will be expired in minutes, so you need to download immediately after getting it.

```js
{
  contentUri: "https://media.devtest.ringcentral.com/restapi/v1.0/account/170848004/recording/6469338004/content?access_token=ringcentral_access_token"
  id: "6469338004"
  link: "http://apps.ringcentral.com/integrations/recording/sandbox/?id=Ab7937-59r6EzUA&recordingId=6469338004"
  type: "OnDemand"
  uri: "https://platform.devtest.ringcentral.com/restapi/v1.0/account/170848004/recording/6469338004"
}
```

In this section, we can log the call into your service directly, or popup a window for user to confirm before saveing into your service.

## Add call log entity matcher

The widget needs to know if a call is logged, so we need to provide `matcher` response:

```js
document.querySelector("#rc-widget-adapter-frame").contentWindow.postMessage({
  type: 'rc-adapter-register-third-party-service',
  service: {
    name: 'TestService',
    callLoggerPath: '/callLogger',
    callLoggerTitle: 'Log to TestService',
    callLogEntityMatcherPath: '/callLogger/match',
  }
}, '*');
```

If we provide `callLogEntityMatcherPath` when we register third party service, the widget will send call matcher request when app is loaded or call is logged. 

```js
window.addEventListener('message', function (e) {
  var data = e.data;
  if (data && data.type === 'rc-post-message-request') {
    if (data.path === '/callLogger/match') {
      // add your codes here to reponse match result
      console.log(data); // get call session id list in here
      // response to widget
      document.querySelector("#rc-widget-adapter-frame").contentWindow.postMessage({
        type: 'rc-post-message-response',
        responseId: data.requestId,
        response: {
          data: {
            '214705503020': [{ // call session id from request
              id: '88888', // call log entity id from your platform
              note: 'Note', // Note of this call log entity
            }]
          }
        },
      }, '*');
    }
  }
});
```

If the widget gets call logger entity from the matcher service, it will show green logger button.

## Conclusion

At this point, we have finished the call logging feature. We just need to register the service and respond to the widget's messages to add your customization code. I hope this article will help you to integrate call logging feature with RingCentral Embeddable quickly.
