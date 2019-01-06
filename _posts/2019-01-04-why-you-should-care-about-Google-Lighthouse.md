 ---
 layout: post
 title:  "why you should care about Google Lighthouse"
 date:   2019-01-04 10:00:00 +0800
 categories: blog
 author: Harry Hou
 ---
 
  ![Lighthouse image](https://cdn-images-1.medium.com/max/1600/1*sFizeNrXYeuDXq209KyIJg.png)
 
## Foreword
	
Last month before team released the new version of the project, we found a very serious issue about performance, we took much time to discuss product design and tried many ways to optimize the performance. Although we fixed it finally, the issue like this will let our team very passive before release. So how to avoid that situation?  mabey we need more tool to test the project so that we can find out issues earlier. 


## Lighthouse

Here to discuss a tool Lighthouse which I investigative recently.   

### What is Lighthouse

[Lighthouse](https://developers.google.com/web/tools/lighthouse/) is an Open Source **auditing tool helping increase the quality of web pages and applications**. I think it's a great tool to test our web page, it can give us some actionable advice to improve our web application.

### How to use Lighthouse

The list below is some ways to use Lighthouse

- **Chrome extension** Install Chrome extension of Lighthouse is a easy way to use it.
- **Chrome DevTools** If you have a recent versions Chrome browser, you can easily use Lighthouse by open browser's dev tools.
- **command line interface** If you have NodeJS in your computer, you can install the Lighthouse package then use it via command line interface.

### Who should use it

Lighthouse can be used in various ways by developers and non-technical audience. you are always can get corresponding information by using it.

### What we can get from Lighthouse

At present, Audits are conducted in a few key areas — `performance`, `accessibility`, `best practices`, `progressive web applications` and `seo`. For every area, the report will give you the test 
 result and some actionable advice.
 
#### Related to our project

- performance

In fact, The reason why I notice this tool is that it can provide some performance test information, and I am focusing on it now. But seems that the Lighthouse give more info about page's load and initialization, but what I more interesting is the page's runtime performance. 

- accessibility

![accessibility](/integration-blog/assets/2019-01-04-why-you-should-care-about-Google-Lighthouse/accessibility.jpeg)

After using it I found that it also may be a good tool to help us improve the `accessibility` of our app. For every page, the report will give you a list of accessibility advice, we can do stuff things according to this list to improve the accessibility.

### Reference

- [lighthouse](https://developers.google.com/web/tools/lighthouse/)

- [Three reasons why you should care about Google Lighthouse](https://building.calibreapp.com/three-reasons-why-you-should-care-about-google-lighthouse-ccaaa72ed9a1)
