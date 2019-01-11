 ---
 layout: post
 title:  "why you should care about Google Lighthouse"
 date:   2019-01-04 10:00:00 +0800
 categories: blog
 author: Harry Hou
 ---
 
  ![Lighthouse image](https://cdn-images-1.medium.com/max/1600/1*sFizeNrXYeuDXq209KyIJg.png)
 
## Foreword
	
Last month, before our team released a new version of our project, we found a very serious performance issue. We took much time to discuss the product design and tried many ways to optimize the performance. Although we fixed the issue eventually, issues like this will put our team in a bad position before releases. So how can we avoid these situations? Maybe we need more tools to test our projects so we can find these issues earlier.


## Lighthouse

I am here to discuss Lighthouse, a tool that I investigated recently.   

### What is Lighthouse

[Lighthouse](https://developers.google.com/web/tools/lighthouse/) is an Open Source **auditing tool helping to increase the quality of web pages and applications**. I think it's a great tool to test our web applications, it can give us some actionable advice to improve our web application.

### How to use Lighthouse

Here are some ways we can use Lighthouse:

- **Chrome extension** Installing the Lighthouse Chrome extension is an easy way to use it.
- **Chrome DevTools** If you have a recent version of Chrome browser, you can easily use Lighthouse by opening the browser's dev tools.
- **command line interface** If you have NodeJS in your computer, you can install the Lighthouse package and use it via command line interface.

### Who should use it

Lighthouse can be used in various ways by developers and non-technical audiences. You can always get some corresponding information by using it.

### What we can get from Lighthouse

At present, Audits are conducted in a few key areas — `performance`, `accessibility`, `best practices`, `progressive web applications` and `seo`. For every area, the report will give you the test 
 result and some actionable advice.
 
#### Related to our project

- performance

In fact, the reason why I noticed this tool is that it can provide some performance information, which is something I am focusing on right now. It seems that Lighthouse gives more information about the page load and initialization performance, but what I am more interested in is the runtime performance of the page.

- accessibility

![accessibility](/integration-blog/assets/2019-01-04-why-you-should-care-about-Google-Lighthouse/accessibility.jpeg)

After using it I found that it also may be a good tool to help us improve the `accessibility` of our app. For every page, the report will give you a list of accessibility advice, we can improve the accessibility according to this list.

### Reference

- [lighthouse](https://developers.google.com/web/tools/lighthouse/)

- [Three reasons why you should care about Google Lighthouse](https://building.calibreapp.com/three-reasons-why-you-should-care-about-google-lighthouse-ccaaa72ed9a1)
