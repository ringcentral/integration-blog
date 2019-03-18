---
layout: post
title:  "TypeScript intro and migrating"
date:   2018-12-25 13:00:00 +0800
categories: blog
author: Jeff Wu
---

![](/integration-blog/assets/2018-12-25-blog-typescript-intro-and-migrating/babel-ts.jpg)

### Background

It's not easy to build a great project that is easy to maintain. Especially when you are in a large company, you have to work with other members. There are many different opinions and ideas even for the simplest of things. Fortunately we have a lot of talented members in RingCentral. We maintain a high quality in our libraries, widgets and products which we build. But, as the project or requirements increase, more and more issue appear. It's time to prepare our project for facing huge projects in the future with innovations.


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

   Apart from the efforts in learning TypeScript, we also have to consider how we can maintain our product development velocity.


### TypeScript

"Static types can make it easier to maintain your code by catching bugs early on, making it easier to navigate your projects, giving accurate code completion, and providing handy fixes for when you do make mistakes", said Microsoft's Daniel Rosenwasser in an introductory [blog post](https://blogs.msdn.microsoft.com/typescript/2017/08/31/announcing-typescript-2-5/).

Microsoft designed TypeScript with specific architectural parameters in mind to allow TypeScript to integrate fully and easily with existing JavaScript code while providing robust features external to JavaScript. Any valid JavaScript code is valid TypeScript code with only a few exceptions: handling option function parameters and assigning a value to an object literal.

Let's take a look at how to install TypeScript:

```bash
npm install typescript
yarn add typescript
```

If you want to install at global instead of local repository, just add `global` and `-g` argument.

After Installed, we can refer the [tutorial](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html) from TypeScript document and practice.

By the tutorial, we can know the advantages of using TypeScript. Static typing is a feature that detects bugs as developers write the scripts. This allows us to write more robust code and maintain it, resulting in better and clean code. Static language helps you implementing SOLID design patterns into a language that doesn't really support it. Promoting innovations and changes while providing safety measures to ensure that it doesn't go completely in the wrong direction. Types make the code more readable. It helps us remember faster what each piece of code is supposed to do. We can add and change the current code faster. With these benefits of using TypeScript, there will be more confidence in working with large projects, better experience in co-working with each other, and ultimately lead to more efficiency and productivity.


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

------

------

![](/integration-blog/assets/2018-12-25-blog-typescript-intro-and-migrating/babel-ts.jpg)

### 挑戰不曾間斷

打造一個易維護的大型項目往往是個挑戰，尤其是身在大型的跨國企業，團隊開發更是必然的工作場景，每個人對於一件事情的看法，儘管是件微小的事物，基於不同的經歷或思考角度都會有所不同。而在開發的過程中，每個成員的靈感或想法都相當珍貴，即使是簡單的議題探討，我們都能激發出各式各樣的創意。很幸運地，在鈴盛我們擁有一群對編程開發充滿熱情與天份的夥伴，以高質量的方式，共同維護著我們所打造的組件庫及面對各式各樣的平台集成產品。隨著項目因需求快速變化及代碼量成長，在持續地迭代過程中，越來越多的挑戰逐漸浮出，面對必然成為大型項目所帶來的挑戰，是時候創新以克服迎面而來的挑戰了。

### 逐漸浮現的問題與挑戰

1. 項目擴大

   如上所述，隨著各項目的源碼持續推入，總有一天會變得相當龐大，如何使我們所推送的源碼可以保持易讀與易維護，這是我們必須要克服的挑戰。

2. 重構是編程開發週期的其中一個階段，常見的重構方式有:

   1. 批次將函數或屬性重命名為具有意義的名稱。
   2. 依據使用情況，判斷未使用的函數或屬性是否移除。
   3. 從函數中提取通用的處理邏輯。

3. 構建兼具靈活與可維護的庫

   基於我們所開發的庫，除了運用在我們的持續投入開發的集成產品項目外，同時也作為開源項目提供第三方的開發者所使用，因此，項目擴展與維護的重要性比起以往更是我們不可忽視的環節之一，TypeScript 可讓我們更有效率的方式來克服這些挑戰。

4. 如何減少從 JavaScript 到 TypeScript 轉移成本?

   投入學習 TypeScript 的同時，如何維持產品開發的速度與品質，也是要考慮到的。畢竟在快速迭代的場景下，若影響到開發的效率，對於整體項目的生產力會大打折扣。若要在短時間內將目前所有項目的 JavaScript 代碼轉換成 TypeScript，這是不太現實的，因此我們必須在不影響開發效率為前提下，以優雅的方式轉移，轉移的過程需要有過渡期，使項目可以同時兼容 JavaScript 與 TypeScript 的開發。

### 關於 TypeScript

微軟的 Daniel Rosenwasser 在一篇 TypeScript 介紹的[博客文章](https://blogs.msdn.microsoft.com/typescript/2017/08/31/announcing-typescript-2-5/)中提到："靜態類型可藉由提早捕捉到 Bug 使代碼更易於維護，亦使項目的導航更加容易，並透過準確的代碼提示快速完成，進而在我們犯錯時能帶來便利的修正方式"

微軟設計了具有明確參數架構的 TypeScript，可讓 TypeScript 很容易的完全集成了現有的 JavaScript 代碼，同時也為 JavaScript 拓展強大功能。任何有效的 JavaScript 代碼都是有效的 TypeScript 代碼，只有少數例外：處理函數可選的參數以及對象賦值等。

首先，如何安裝 TypeScript ? 我們可以透過 NPM 或 Yarn 等依賴管理工具進行安裝：

```bash
npm install typescript
yarn add typescript
```

若要安裝在全域，要在 NPM 加上參數 -g 或是 yarn 的 add 前面補上 global：

```bash
npm install typescript -g
yarn global add typescript
```

安裝完成後，輸入以下指令確認目前的 TypeScript 版本，若有顯示則代表安裝成功：

```bash
tsc -v
```

我們可以參考 TypeScript 官方文檔的[五分鐘快速體驗 TypeScript](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html) 文章進行練習。

透過五分鐘快速體驗，我們可以體驗到 TypeScript 所帶來的好處。靜態類型是讓開發者在編程時可以檢測 Bug 的功能，這使我們能撰寫強大的代碼並維護著，進而產出更高質量與乾淨的代碼。靜態語言幫助我們實現到一個原本不支持 SOLID 設計模式的語言，進而促進創新與改變，同時帶來安全措施以確保方向的正確性。類型可使代碼更有可讀性，有助於我們更快地記住每段代碼應該做的事情，更快的新增和修改當前的代碼。TypeScript 所帶來的這些優勢，將使我們更有信心在大型項目上的開發，並擁有更好地合作體驗，最終提高效率和生產力。了解到 TypeScript 所帶來的好處後，我們接著要解決的問題就是，如何優雅的在項目中進行優雅的轉移。

### 從 JavaScript 到 TypeScript 優雅的轉移

要轉移到目前的 repository，從 ES6 Babel 的轉移我們需要的是一個平滑的過程。為此，筆者找了以下兩種方式來實現：

1. 使用 [react-app-script-ts](https://github.com/wmonk/create-react-app)

   `react-app-script-ts` 是由一位 [Will Monk](https://github.com/wmonk) 基於 Facebook 開發的 `create-react-app` 進行擴展，若想要避開應用建置的設定相關問題，利用這個 CLI 工具是一個不錯的選擇。只需要透過依賴管理工具(例如：NPM)安裝，即可輕鬆地以輸入指令的方式來進行操作。這個工具主要是透過 Webpack 配置 `babel-loader` 與 `ts-loader`，藉此實現 `JavaScript` 與 `TypeScript` 同時兼容的場景，關於配置的內容想了解的話，可以在 `node_modules` 目錄下找到。而大部分項目下的 Webpack 都是由我們自行配置的，因此我們可以參考其 `loader` 配置來進行個別項目下的調整。

2. 使用 [babel-preset-typescript](https://babeljs.io/docs/en/babel-preset-typescript)

   正如我們目前項目的自動化處理流程使用到 `webpack` 及 `gulp`，使用 `babel-preset-typescript` 可以使 JavaScript 與 TypeScript 並存，且在編譯過程是同一階段進行的。透過 Babel preset 配置，可以更容易的實現我們所需的平滑轉移過程。

雖然這些方案可以完整的符合我們的需求，但還有一個挑戰需要克服，就是我們最近遷移的 monorepo。在進行這些配置之前，我們必須先升級到 Babel 7，然而 Babel 7 對於我們的 monorepo 存在相容性問題，特別是要從 subrepo 引用 module 時，會導致 Babel 的配置無法正確取得。我們持續嘗試找出問題點並解決，若順利解決這個問題，我們就能順利的進行後續的配置，也就是引入 typescript preset 使 TypeScript 與 JavaScript 兼容，有利於克服未來邁向大型項目開發的挑戰，增進團隊開發的生產效率。

### 小結

1. TypeScript & JavaScript 是可同時運行的。
2. TypeScript 目前已有許多中大型知名企業廣泛用於大型項目開發上。
3. TypeScript 擁有豐富且可靠的開發者社區與支援。
4. 升級 Babel 7 是眼前的第一個議題是我們需要解決的。

### 參考來源

1. [TypeScript in 5 minutes](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)
2. [Top TypeScript Advantages](https://apiumhub.com/tech-blog-barcelona/top-typescript-advantages/)
3. [Migrating a Babel project to TypeScript](https://medium.com/pleo/migrating-a-babel-project-to-typescript-af6cd0b451f4)
4. [Introduction to TypeScript](https://toddmotto.com/typescript-introduction)
5. [TypeScript vs JavaScript deep comparison](https://juejin.im/entry/5a52ed336fb9a01cbd586f9f)
