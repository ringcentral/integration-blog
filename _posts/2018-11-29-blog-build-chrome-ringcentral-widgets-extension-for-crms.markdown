---
layout: post
title:  "Build Chrome extension for CRMs with RingCentral Embeddable widgets"
date:   2018-11-29 13:31:07 +0800
categories: blog
---

## About RingCentral Embeddable widgets

RingCentral Embeddable widgets is a powerful tool for CRMs, its core power is add click-to-call function, around this core function, it can extend CRM's communication workflow. And it can be easily integrated into CRM sites, even without official support, developer can still do the integration through building chrome extension.

## Features of these extensions could provide

With chrome extension, we could add click-to-call buttons/links etc and all the third party features of our ringcentral-embeddable supported:

- For CRM contact list, extension could add hover-to-show tooltip to show click-to-call button.

![ ](https://github.com/zxdong262/ringcentral-embeddable-extension-factory/raw/master/screenshots/fac-2.png)

- For CRM contact info page, extension could add click-to-call button in proper positions.

![ ](https://github.com/zxdong262/ringcentral-embeddable-extension-factory/raw/master/screenshots/fac-3.png)

- For CRM contact phone number text, extension could convert them to click-to-call link.

![ ](https://github.com/zxdong262/ringcentral-embeddable-extension-factory/raw/master/screenshots/fac-1.png)

- Sync CRM contacts to our widgets after user authorization.

![ ](https://github.com/zxdong262/insightly-embeddable-ringcentral-phone/raw/master/screenshots/insightly-4.png)

- Sync call log to CRM automatically or manually.

![ ](https://github.com/zxdong262/hubspot-embeddable-ringcentral-phone/raw/master/screenshots/hs6.png)

- Check CRM contact activities from our widgets.

![ ](https://github.com/zxdong262/hubspot-embeddable-ringcentral-phone/raw/master/screenshots/hs7.png)

- Show CRM contact info panel when inbound/outbound call with CRM contact.

![ ](https://github.com/zxdong262/hubspot-embeddable-ringcentral-phone/raw/master/screenshots/hubspot1.png)

## Create extension with CLI tool

To minimize the effort to create these extensions, I created the CLI tool: [ringcentral-embeddable-extension-factory](https://github.com/ringcentral/ringcentral-embeddable-extension-factory)

Use CLI tool to init a extension project will be like this:

```bash
# make sure you have npm@5.2+ installed
npx ringcentral-embeddable-extension-factory my-app
```

![ ](https://github.com/zxdong262/ringcentral-embeddable-extension-factory/raw/master/screenshots/cli.png)

Then just follow the instruction inmy-app/README.md to build, test and develop the extension.

I have a demo video to show this process.

[https://youtu.be/2njQSk8x2K4](https://youtu.be/2njQSk8x2K4)

As you can see, before setting proper config/functions, it will work as a **Embeddable widgets** only, if you are satisfied with this, it is already done! If you want the extension to do the listed features above, you need more work to get there. The tutorial and examples would be a good place to start.

## The Tutorial

This detailed tutorial will walk you through the process of creating Chrome RingCentral widgets extension for CRM.

[https://ringcentral-tutorials.github.io/build-chrome-ringcentral-widgets-extension-for-crm/](https://ringcentral-tutorials.github.io/build-chrome-ringcentral-widgets-extension-for-crm/)

## Realworld examples

You may get some ideas from these codes.

- [hubspot-embeddable-ringcentral-phone (spa)](https://github.com/zxdong262/hubspot-embeddable-ringcentral-phone)
- [insightly-embeddable-ringcentral-phone (spa)](https://github.com/zxdong262/insightly-embeddable-ringcentral-phone)
- [redtail-embeddable-ringcentral-phone (non spa)](https://github.com/zxdong262/redtail-embeddable-ringcentral-phone)
