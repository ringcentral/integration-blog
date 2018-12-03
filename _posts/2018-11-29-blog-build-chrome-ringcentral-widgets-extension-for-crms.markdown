---
layout: post
title:  "Build Chrome RingCentral widgets extension for CRMs"
date:   2018-11-29 13:31:07 +0800
categories: blog
---

## Why is it need?

- First, and the important reason, I guess, is Our clients need it, so for big CRM, like salesforce, we even build special app for it. For not that big CRMs, still have clients need it too, we may not build a special app for it. But we have the powerful widgets, so we can build a chrome extension easily.
- Second, our RingCentral widgets is userful and powerful for these CRMs, we could greatly improve their work proccess.

## What will we do in these extensions?

In general, All the third party features of our [ringcentral-embeddable](https://github.com/ringcentral/ringcentral-embeddable) supported and some content insert:

- For CRM contact list, we will add a hover-to-show tooltip to show click-to-call button.

![ ](https://github.com/zxdong262/ringcentral-embeddable-extension-factory/raw/master/screenshots/fac-2.png)

- For CRM contact info page, we will add a click-to-call button in proper position.

![ ](https://github.com/zxdong262/ringcentral-embeddable-extension-factory/raw/master/screenshots/fac-3.png)

- For CRM contact phone number text, we make it click-to-call link.

![ ](https://github.com/zxdong262/ringcentral-embeddable-extension-factory/raw/master/screenshots/fac-1.png)

- Sync CRM contacts to our widgets after user authorize.

![ ](https://github.com/zxdong262/insightly-embeddable-ringcentral-phone/raw/master/screenshots/insightly-4.png)

- Sync call log to CRM automatticly or mannually.

![ ](https://github.com/zxdong262/hubspot-embeddable-ringcentral-phone/raw/master/screenshots/hs6.png)

- Check CRM conatct activities from our widgets.

![ ](https://github.com/zxdong262/hubspot-embeddable-ringcentral-phone/raw/master/screenshots/hs7.png)

- Show CRM contact info panel when inbound/outbpund call with CRM contact.

![ ](https://github.com/zxdong262/hubspot-embeddable-ringcentral-phone/raw/master/screenshots/hubspot1.png)

## How do we do it

Ok, two steps, build a chrome extension, and add all the above features, with [Embbnux Ji](https://github.com/embbnux)'s tuturial:
 [Building Chrome Extension Integrations with RingCentral Embeddable](https://medium.com/ringcentral-developers/build-a-chrome-extension-with-ringcentral-embeddable-bb6faee808a3)
and [third-party-service-in-widget document](https://github.com/ringcentral/ringcentral-embeddable/blob/master/docs/third-party-service-in-widget.md)

Not hard, but still need more effort, so I created the [ringcentral-embeddable-extension-factory](https://github.com/zxdong262/ringcentral-embeddable-extension-factory) to minimize the work.

We could simplify it like this.

```bash
# make sure you have npm@5.2+ installed
npx ringcentral-embeddable-extension-factory my-app
# or install it first
# npm i -g ringcentral-embeddable-extension-factory && reef my-app
# then carefully answer all questions, then the my-app folder will be create
cd my-app
npm i
npm start
# Then just follow my-app/README.md's instruction
```

![ ](https://github.com/zxdong262/ringcentral-embeddable-extension-factory/raw/master/screenshots/cli.png)

For more detail, read [https://github.com/zxdong262/ringcentral-embeddable-extension-factory](https://github.com/zxdong262/ringcentral-embeddable-extension-factory)

Still, this is just a skeleton, need developer efforts to make it a work, CLI tool can not know what css selector to use yet, do the crm support oauth or not, how to get the csrf token etc, not that smart. I do add some common module to do common task, developer could just set proper config to make it work.

## Realworld examples

- [hubspot-embeddable-ringcentral-phone (spa)](https://github.com/zxdong262/hubspot-embeddable-ringcentral-phone)
- [insightly-embeddable-ringcentral-phone (spa)](https://github.com/zxdong262/insightly-embeddable-ringcentral-phone)
- [redtail-embeddable-ringcentral-phone (non spa)](https://github.com/zxdong262/redtail-embeddable-ringcentral-phone)
