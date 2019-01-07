---
  layout: post
  title:  "What is a PWA and why should you care?"
  date:   2018-12-24 13:31:07 +0800
  categories: blog
  author: Aermin Huang
  ---
  
   [Article Link](https://blog.bitsrc.io/what-is-a-pwa-and-why-should-you-care-388afb6c0bad)
  
  ### Summary:
  
  There are some core attributes which progressive web applications support. In other words, they are the reasons why we should care about PWA and apply it in our applications.

  1.Reliable

  The app should be lightning fast when loading, it should be close to instantaneous and should also open when there is no network or fairly low-speed network like 2G.

  2.Fast

  The scrolls and page transitions should be buttery smooth when the user is interacting with the web app. 

  3.Responsive

  The app should fit in all the different sizes of devices. 

  4.Installable

  If we want to make web apps closer to the native apps, they have to be installable and should reside in the home screen along with other native apps, so that the user can access the PWA in one click.

  5.Splash Screen

  PWA adds a splash screen during the startup of the app. This makes the PWA feel more like a native app.

  6.Highly engage-able
  
  The app should keep the users engaged. A PWA provides features like push notification, home screen icon, full-screen and offline first app to glorify user engagement.

  And no matter what framework you use, PWA only needs the required components.

  There are three components required to develop a PWA as shown below:

  ![image](https://user-images.githubusercontent.com/24861316/50399596-8600f800-07bb-11e9-9087-b697a95376b1.png)
  
  The first component is the Service worker. Service worker works as a proxy between the browser and the network. It manages notifications pushing and helps to use web application in offline with the browser's cache API. 
  
  The second component is The manifest file. It is a config JSON file which contains the information of your application, such as the icon which is displayed on the home screen when installed, the short name of the application, background color, and theme.

  The third component is HTTPS. PWA need secure protocol HTTPS because that the service worker has the ability to intercept the network requests and modify the responses.

  ### Reflection:
  
  Progressive web application (PWA) is created to make web app provide a native-like experience with some components for users. Such as enabling to load when there is no network or network is poor, installing as an app and show in the home screen, opening the app in full screen, having notification pushing and so on.

  I think PWA can play a more and more important part in the unified application development. On the one hand, web is open and the mobile native application is closed. It causes users to search app with low efficiency in fixed app stores. On the other hand, Progressive web applications developed by a set of web technologies can work on different platforms. It can reduce lots of development cost. Of cause, the premise is that Progressive web applications should provide as good experience as native applications providing. And It almost does now in most scenarios.

  PWA deserves our attention because of the benefits.