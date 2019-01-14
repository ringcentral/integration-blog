---
layout: post
title:  "Build Chrome extension for CRMs with RingCentral Embeddable widgets"
date:   2018-11-29 13:31:07 +0800
categories: blog
author: Drake Zhao
---

![ ](https://github.com/ringcentral/ringcentral-embeddable-extension-factory/raw/master/screenshots/bb.jpg)

## About RingCentral Embeddable widgets

RingCentral Embeddable widgets is a powerful tool for CRMs, its core power is add click-to-call function, around this core function, it can extend CRM's communication workflow. And it can be easily integrated into CRM sites, even without official support, developer can still do the integration through building chrome extension.

## Advanced features of these extensions could provide

[Embbnux Ji](https://github.com/embbnux) has a tuturial: [Building Chrome Extension Integrations with RingCentral Embeddable](https://medium.com/ringcentral-developers/build-a-chrome-extension-with-ringcentral-embeddable-bb6faee808a3), with this tutorial, we could create Chrome extension for any site with RingCentral Embeddable.

For CRM sites, we may want to add more advanced features, like click-to-call buttons/links and all the third party features of our ringcentral-embeddable supported:

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

以下是中文翻译版本

# 使用RingCentral Embeddable widgets为CRM网站构建谷歌浏览器插件

![ ](https://github.com/ringcentral/ringcentral-embeddable-extension-factory/raw/master/screenshots/bb.jpg)

## 关于 RingCentral Embeddable widgets

RingCentral Embeddable widgets 可以为CRM网站提供独特而且强大的通讯集成，它的核心功能是点击即刻拨打电话功能，围绕这个核心功能，它可以扩展CRM网站的通讯工作流程。它可以被轻松的集成到CRM网站，即便没有官方支持，开发者也可以通过谷歌浏览器插件的形式提供集成。

## 谷歌浏览器插件可以提供的高级特性

参考[Embbnux Ji](https://github.com/embbnux) 的通用教程: [Building Chrome Extension Integrations with RingCentral Embeddable](https://medium.com/ringcentral-developers/build-a-chrome-extension-with-ringcentral-embeddable-bb6faee808a3)，我们可以创建针对任意网站的RingCentral Embeddable widgets集成。

针对CRM网站, 我们可以更进一步，提供许多高级特性, 例如点击即刻拨打电话的按钮和链接，以及许多的专门针对第三方网站的特性:

- 针对CRM网站的联系人列表，插件可以提供tooltip，气泡显示点击即刻拨打按钮。

![ ](https://github.com/zxdong262/ringcentral-embeddable-extension-factory/raw/master/screenshots/fac-2.png)

- 针对CRM网站联系人信息页面，插件可以插入点击即刻拨打电话的按钮。

![ ](https://github.com/zxdong262/ringcentral-embeddable-extension-factory/raw/master/screenshots/fac-3.png)

- 可以把CRM网站上的电话号码文字转换为点击即刻拨打电话的链接。

![ ](https://github.com/zxdong262/ringcentral-embeddable-extension-factory/raw/master/screenshots/fac-1.png)

- 支持把CRM网站的联系人列表同步到RingCentral Embeddable widgets。

![ ](https://github.com/zxdong262/insightly-embeddable-ringcentral-phone/raw/master/screenshots/insightly-4.png)

- 支持自动手动同步电话记录到CRM网站.

![ ](https://github.com/zxdong262/hubspot-embeddable-ringcentral-phone/raw/master/screenshots/hs6.png)

- 在RingCentral Embeddable widgets显示CRM网站联系人的活动信息。

![ ](https://github.com/zxdong262/hubspot-embeddable-ringcentral-phone/raw/master/screenshots/hs7.png)

- 接听、拨打电话时候，显示CRM网站联系人详细信息。

![ ](https://github.com/zxdong262/hubspot-embeddable-ringcentral-phone/raw/master/screenshots/hubspot1.png)

## 使用命令行工具创建插件项目

为了方便开发者创建类似的扩展程序，我们提供了命令行工具: [ringcentral-embeddable-extension-factory](https://github.com/ringcentral/ringcentral-embeddable-extension-factory)

使用命令行工具创建一个插件工程:

```bash
# 请确保已经安装npm版本5.2+
npx ringcentral-embeddable-extension-factory my-app
```

![ ](https://github.com/zxdong262/ringcentral-embeddable-extension-factory/raw/master/screenshots/cli.png)

然后按照my-app/README.md的说明初始化，测试和构建插件即可。

我们制作整个流程的视频演示:

[https://youtu.be/2njQSk8x2K4](https://youtu.be/2njQSk8x2K4)

无需按任何设置和代码编辑，即可使Embeddable widgets集成到网站，如果你对此已经满意，那么插件可以宣告完工。如果你想要实现以上列出的高级特性，那就需要实际的编码工作了。你可以从我们提供的教程和案例入手。

## 案例

我们的教程将详细讲解创建插件的全过程。

[https://ringcentral-tutorials.github.io/build-chrome-ringcentral-widgets-extension-for-crm/](https://ringcentral-tutorials.github.io/build-chrome-ringcentral-widgets-extension-for-crm/)

## 实际的案例

从这些案例中你也可以得到一些如何实现的启示。

- [hubspot-embeddable-ringcentral-phone (spa)](https://github.com/zxdong262/hubspot-embeddable-ringcentral-phone)
- [insightly-embeddable-ringcentral-phone (spa)](https://github.com/zxdong262/insightly-embeddable-ringcentral-phone)
- [redtail-embeddable-ringcentral-phone (non spa)](https://github.com/zxdong262/redtail-embeddable-ringcentral-phone)
