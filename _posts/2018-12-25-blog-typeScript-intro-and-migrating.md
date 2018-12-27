---
layout: post
title:  "TypeScript intro and migrating"
date:   2018-12-25 13:00:00 +0800
categories: blog
author: Jeff Wu
---

![](/integration-blog/assets/2018-12-25-blog-typescript-intro-and-migrating/babel-ts.jpg)

### Backgound

It's not easy to build a great project that is easy to maintain. Especially when you are in a large company, you have to work with other members. There are many different opinions and ideas even for the simplest of things. Fortunately we have a lot of talent members in RingCentral. We maintain a high quality in our libraries, widgets and products which we build. But, as the project or requirements increase, more and more issue appear. It's time to prepare our project for facing huge projects in the future with innovations.


### Current problems

1. Project scaling

   As described above, more and more source code have been pushed into the projects. There will be a day when our projects grow to enormous sizes. How to make our source code easy to maintain and read, that is the challenge we would have to overcome.

2. Refactoring

   What are the common procedures of refactoring?

   1. Bulk rename functions or property with meaningful names.
   2. Check the reference counts of functions and properties, and remove if they are unused.
   3. Extract common logic from functions.

3. Build a great library that is flexible and maintainable.

   Since we also provide our library as open source project for 3rd-party use, the importance of maintaining and scaling the project is much higher than before. TypeScript can make this easier.

4. How to reduce the work of moving from Javascript to TypeScript?

   Apart from the efforts in learning TypeScirpt, we also have to consider how we can maintain our product development velocity.


### TypeScript

"Static types can make it easier to maintain your code by catching bugs early on, making it easier to navigate your projects, giving accurate code completion, and providing handy fixes for when you do make mistakes", said Microsoft's Daniel Rosenwasser in an introductory [blog post](https://blogs.msdn.microsoft.com/typescript/2017/08/31/announcing-typescript-2-5/).

Microsoft designed TypeScript with specific architectural parameters in mind allow TypeScript to integrate fully and easily with existing JavaScript code while providing robust feature external to JavaScript. Any valid JavaScript code is valid TypeScript code with only a few exceptions: handing option function parameters and assigning a value to an object literal.

Let's take a look at how to install TypeScript:

```bash
npm install typescript
yarn add typescript
```

If you want to install at global instead of local repository, just add `global` and `-g` argument.

After Installed, we can refer the [tutorial](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html) from TypeScript document and practice.

By the tutorial, we can know that advantage of using TypeScript. Static typing is a feature that detects bugs as developers write the scripts. This allows us to write more robust code and maintain it, resulting in better and clean code. Static language helps you implementing SOLID design patterns into a language that doesn't really support it. Innovation and change, also, with safety measures to ensure that it doesn't go completely in the wrong direction. Types make the code more readable. It helps us remember faster what each piece of code is supposed to do. We can add and change the current code faster. With these benefits which using TypeScript, it will more confident for large scaling projects and have a better experience of co-working, then increase the ability of powerful production efficiently.


### Migrating

To migrating with current repositories, we need a smooth progess of migration from ES6 babel. Here are 2 ways I found for smooth migration:

1. Using [react-app-script-ts](https://github.com/wmonk/create-react-app)

   If you want to start without configurating the build setup, this CLI tool is a good choice for you. Just install the CLI tool with NPM, then you can use the command provided by the CLI tool easily. The tool will setup webpack, babel-loader, and ts-loader behind the scenes, and the actual configuration can be found in the node_modules folders. In our scenario, we need to set the configuration by ourselves. We can refer the configuration to know how to config the loaders for our webpack projects.

2. Using [babel-preset-typescript](https://babeljs.io/docs/en/babel-preset-typescript)

   As our current projects use webpack and gulp for automatic processes, using babel-preset-typescript make it possible for JS files and TS files to co-exists and compile in one step.

Although these solutions can fully meet our requirements, we still have an issue to resolve - the monorepo that we transferred to recently. Before we can use this setup, we need to upgrade babel to babel v7. However, babel v7 is incompatible with our monorepo setup, in particular, referencing modules from subrepos will fail to use the proper babel configuration from the monorepo. We are still trying to figure out the solution to this problem, and we believe that solving this problem will enable us to use typescript, which is beneficial for our future development with large projects.


### Conclusion

1. TS and JS can work together.
2. TS is widely used for large projects in the industry.
3. TS is solid community and developer support.
4. Updating to Babel v7 is the first issue we need to solve.


### References

1. [TypeScript in 5 minutes](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)
2. [Top TypeScript Advantages](https://apiumhub.com/tech-blog-barcelona/top-typescript-advantages/)
3. [Migrating a Babel project to TypeScript](https://medium.com/pleo/migrating-a-babel-project-to-typescript-af6cd0b451f4)
4. [Introduction to TypeScript](https://toddmotto.com/typescript-introduction)
5. [TypeScript vs JavaScript deep comparison](https://juejin.im/entry/5a52ed336fb9a01cbd586f9f)

