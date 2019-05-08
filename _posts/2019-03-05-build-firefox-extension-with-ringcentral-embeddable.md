---
layout: post
title:  "Building a Firefox add-on with RingCentral Embeddable"
date:   2019-03-05 10:00:00 +0800
categories: blog
author: Embbnux Ji
---

![ringcentral-embeddable-with-firefox](https://user-images.githubusercontent.com/7036536/55455660-0e8f0000-5617-11e9-89be-17c008c7d3f4.png)

In our previous [article](https://medium.com/ringcentral-developers/build-a-chrome-extension-with-ringcentral-embeddable-bb6faee808a3), we introduced how to use RingCentral Embeddable to build a Chrome extension to integrate RingCentral voices and text features into any CRM websites. Now RingCentral Embeddable also has support for Firefox. So now we can also use Firefox add-on to integrate RingCentral into any CRM websites.

In this article, we will build RingCentral Embeddable with Google API integration in Firefox add-on.

**Related Link:**

* Core widget: [RingCentral Embeddable](https://github.com/ringcentral/ringcentral-embeddable)
* Full code: [RingCentral Embeddable for Google in Firefox add-on](https://github.com/ringcentral/ringcentral-embeddable)
* Base Firefox add-on: [RingCentral Embeddable Firefox extension template](https://github.com/embbnux/ringcentral-embeddable-firefox-extension)

## Build Firefox add-on manifest.json

Before we build a Firefox add-on, we need to define `manifest.json`

```json
{
  "name": "RingCentral Embeddable for Google",
  "description": "RingCentral Embeddable For Google with Firefox extension",
  "version": "0.0.3",
  "permissions": [
    "http://*/",
    "https://*/",
    "storage",
    "activeTab",
    "tabs",
    "identity",
    "notifications",
    "unlimitedStorage",
    "https://*.google.com/*"
  ],
  "content_scripts": [
    {
      "matches": ["https://www.google.com/contacts/*", "https://contacts.google.com/*", "https://calendar.google.com/*", "https://mail.google.com/*"],
      "js": [
        "vendors/libphonenumber-js.min.js",
        "clickToDial.js",
        "content.js"
      ],
      "css": [
        "clickToDial.css"
      ]
    }
  ],
  "icons": {
    "16": "rc16.png",
    "32": "rc32.png",
    "48": "rc48.png",
    "128": "rc128.png"
  },
  "background": {
    "scripts": ["googleClient.js", "background.js"]
  },
  "browser_action": {
    "default_icon": {
      "16": "rc16.png",
      "32": "rc32.png"
    }
  },
  "content_security_policy": "script-src 'self' https://ringcentral.github.io/ringcentral-embeddable; object-src 'self'",
  "manifest_version": 2,
  "applications": {
    "gecko": {
      "id": "integration-embeddable-firefox@ringcentral.com",
      "strict_min_version": "60.0"
    }
  }
}
```

The `manifest.json` file of Firefox addons is very similar to chrome extensions. We need to add permissions, content, background and content_security_policy configurations. We will be injecting JS and CSS files into google related websites, so we need to add urls of these website into content_scripts.matches list.

With `manifest.json`, we can go to add-ons debug page `about:debugging#addons`, and install it into Firefox.

![firefox extension install](https://user-images.githubusercontent.com/7036536/55455933-2b780300-5618-11e9-8db1-2abc664933dc.png)

Remember to reload add-on every time you update code.

## Inject RingCentral Embeddable into web page

We will use content_scripts to inject RingCentral Embeddable into web pages.

### Inject RingCentral Embeddable with adapter.js

Add following code into `content.js`

```js
(function() {
  var rcs = document.createElement("script");
  rcs.src = "https://ringcentral.github.io/ringcentral-embeddable/adapter.js";
  var rcs0 = document.getElementsByTagName("script")[0];
  rcs0.parentNode.insertBefore(rcs, rcs0);
})();
```

### Init widget message listener

Add following code into `content.js`

```js
// Listen message from RingCentral Embeddable widget and response:
window.addEventListener('message', (e) => {
  const request = e.data;
  if (!request || !request.type) {
    return;
  }
  if (request.type === 'rc-adapter-pushAdapterState' && !this._registered) {
    this._registered = true;
    // To get third party service info from background and register service into widget
    chrome.runtime.sendMessage({ type: 'rc-register-service' }, (response) => {
      this.postMessageToWidget({
        type: 'rc-adapter-register-third-party-service',
        service: response.service,
      })
    });
    return;
  }
  // send widget message to background
  chrome.runtime.sendMessage(request, (response) => {
    console.log('response:', response);
    if (request.type === 'rc-post-message-request') {
      // send background response message to widget
      this.responseMessageToWidget(request, response)
    }
  });
});
```

We will pass request messages from widget to extension background, and background will handle and respond.

### Init message response service in background.js

Add following code into `background.js`

```js
// Listen message from content and standalong to response
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.type === 'rc-register-service') {
    this.registerService(sendResponse)
    return true;
  }
  if (request.type === 'rc-post-message-request') {
    if (request.path === '/authorize') {
      this.onAuthorize(request.body.authorized);
      sendResponse({ data: 'ok' });
    }
    if (request.path === '/contacts') {
      this.onGetContacts(request, sendResponse);
    }
    if (request.path === '/conference/invite') {
      this.createCalendarEvent(request, sendResponse);
    }
    if (request.path === '/activities') {
      this.getContactGmails(request, sendResponse);
    }
    if (request.path === '/activity') {
      this.openGmailPage(request, sendResponse);
    }
    if (request.path === '/contacts/search') {
      this.onContactSearch(request, sendResponse);
    }
    if (request.path === '/contacts/match') {
      this.onContactMatch(request, sendResponse);
    }
    return true;
  }
});
```

### Integrate Google related feature

After injecting RingCentral Embeddable widget into webpage, we also want to integrate third party services into RingCentral Embeddable. In this section, we will integrate Google Authorization, Google Contacts, Google Calendar, and Gmail into widget.

![ringcentral embeddable integrate with gsuit](https://user-images.githubusercontent.com/7036536/55455660-0e8f0000-5617-11e9-89be-17c008c7d3f4.png)

### Integrate Google authorization into widget:

Add following code into `background.js`

```js
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  // Register serivce into widget
  if (request.type === 'rc-register-service') {
    registerService(sendResponse)
    return true;
  }
  if (request.type === 'rc-post-message-request') {
    // Response to authorize request
    if (request.path === '/authorize') {
      this.onAuthorize(request.body.authorized);
      sendResponse({ data: 'ok' });
    }
  }
});

async function registerService(sendResponse) {
  const authorized = await googleClient.checkAuthorize();
  // Register Google service
  sendResponse({
    action: 'registerService',
    service: {
      name: 'Google',
      authorizationPath: '/authorize',
      authorizedTitle: 'Unauthorize',
      unauthorizedTitle: 'Authorize',
      authorized,
    }
  });
}

async function onAuthorize(authorized) {
  if (!authorized) {
    await googleClient.authorize();
  } else {
    await googleClient.unAuthorize();
  }
  const newAuthorized = await googleClient.checkAuthorize();
  if (newAuthorized) {
    googleClient.setUserInfo();
  }
  await sendMessageToContentAndStandalong(
    { action: 'authorizeStatusChanged', authorized: newAuthorized }
  );
}
```

You can get full google client code in here. Firstly, we register a authorize path when we register the service. Then we build a message listener to respond the authorize request.

### Integrate Google contacts into widget

Add following code into background.js

```js
chrome.runtime.onMessage.addListener(function(request, sender, sendResponse) {
  if (request.type === 'rc-register-service') {
    registerService(sendResponse)
    return true;
  }
  if (request.type === 'rc-post-message-request') {
    if (request.path === '/authorize') {
      onAuthorize(request.body.authorized);
      sendResponse({ data: 'ok' });
    }
    if (request.path === '/contacts') {
      onGetContacts(request, sendResponse);
    }
    return true;
  }
});

async function registerService(sendResponse) {
  const authorized = googleClient.checkAuthorize();
  sendResponse({
    action: 'registerService',
    service: {
      name: 'Google',
      authorizationPath: '/authorize',
      authorizedTitle: 'Unauthorize',
      unauthorizedTitle: 'Authorize',
      authorized,
      contactsPath: '/contacts',
    }
  });
}

async function onGetContacts(request, sendResponse) {
  const pageToken = request.body.page === 1 ? null : request.body.page;
  const syncToken = request.body.syncTimestamp;
  const response = await googleClient.queryContacts({ pageToken, syncToken });
  const contacts = response.connections || [];
  sendResponse({
    data: contacts.map((c) => ({
      id: c.resourceName.replace('people/', ''),
      name: c.names[0] && c.names[0].displayName,
      type: 'Google', // need to same as service name
      phoneNumbers:
        (c.phoneNumbers && c.phoneNumbers.map(p => ({ phoneNumber: p.value, phoneType: p.type }))) ||
        [],
      emails: (c.emailAddresses && c.emailAddresses.map(c => c.value)) || [],
    })),
    nextPage: response.nextPageToken,
    syncTimestamp: response.nextSyncToken,
  })
}
```

Firstly, we register a contact path when we register the service. Then we respond to contacts request.

For more feature integrations and details, you can get the full code in [here](https://github.com/ringcentral/ringcentral-embeddable-for-google-firefox-addon).

We also provide the CRM Firefox extension factory CLI tool to help developers build Firefox extensions. You can get it [here](https://github.com/ringcentral/ringcentral-embeddable-extension-factory). Just run it and select Firefox extension, and you can get it to work.

## Try it out

To get the release packages [here](https://github.com/ringcentral/ringcentral-embeddable-for-google-firefox-addon/releases) and try it out by installing them into Firefox. Reach out to us on [GitHub issues](https://github.com/ringcentral/ringcentral-embeddable-for-google-firefox-addon/issues), [Developer Community](https://devcommunity.ringcentral.com/ringcentraldev), or [Stack Overflow](https://stackoverflow.com/questions/tagged/ringcentral) if you have any questions.
