---
layout: post
title:  "Reading: Best practices for using Typescript with React"
date:   2018-12-22 10:00:00 +0800
categories: readingnote
author: Rukai Yu
---

[Article Link](https://medium.freecodecamp.org/effective-use-of-typescript-with-react-3a1389b6072a)

![ ](https://cdn-images-1.medium.com/max/2000/1*qLgLDFCPLeZZlJ4v00jIOw.jpeg)

# Summary

* The best practices for using TypeScript in a larger React application are less clear before practicing with actual project.
* They tried migrated a React front-end for one of our applications from JavaScript to TypeScript. The consensus is that their codebase is now easier to understand and maintain.
* It helped deepen our understanding of React and the architecture of our own application.

# Conclusion

IMO, I also agree to use TypeScript with React development in decent sized project. It benefits more than just:

* Instant type error detection
* Better IDE experience
* More self-documenting code
* More readable & easier to understand
* Easy for refactoring, especially in tens of JS files

But I also agreed that in a big project, it is worth discussing that we cannot do the migration immediately. Things we need to consider:

* Will it impact the project roadmap? In my last company, we spent more than half year on migrating the AngularJs front-end project (with about >= 120K lines of code) to Angular 4 (TypeScript), with two teams side-by-side running, one for migration, one for Maintenance. It will be more easier for a new created project.
* Is the DEV team ready for it? Not everyone is suitable with TypeScript. It means you have to change your thought, coding style, usual practice from a dynamic language to a static (or static-like) one, and many other constraints that you don't have in pure JS development. As someone who has backend development experience, it could be more smooth to fit this.
* If we adopt Typescript, we should aim for long term use! It is not usable nor valuable for project to change main framework and architecture frequently. That means if we moved to TypeScript, we keep the future in mind. We also could obtain more experiences.

To summarize, I welcome/enjoy to TS development, but we need time to make the transition smoother.
