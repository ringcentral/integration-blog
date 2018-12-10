---
layout: post
title:  "Reading: Enabling Agility with Branch By Abstraction by Mark J. Balbes, Ph.D."
date:   2018-11-13 23:31:07 +0800
categories: readingnote
---

[Article Link](https://adtmag.com/articles/2017/04/21/agile-branch-by-abstraction.aspx)

#### Summary:
Traditional workflow with version control often creates “islands of stability”, or feature/release specific branches that are tested amongst themselves. The longer these branches are in development, the harder it is to merge back into the master branch. In some cases, the divergence can be too great and the branch will stay forever to avoid the refactoring needed for the merge to happen, which hinders the concept of agile parallel development.  
Branch by abstraction is a practice designed to solve these issue. It aims to allow parallel development of feature, release of different features in different times, refactors and features to be shared quickly, and can be beneficial to code design. The idea is simple: first you introduce the abstraction to encapsulate the new feature or the old feature to be refactored with a toggle to switch between old and new implementations, then you build or refactor behind the abstraction. This way, these new implementation can be quickly merged into master and shared among others because they can be turned off. After the new code is fully tested and hits production, then the old code and feature toggle can be removed if necessary.
The concept may appear to be big-design-up-front methodology that had failed before, but in practice, only just enough design has to be done, and the abstraction will have to evolve as more features are to be introduced into the code. As for deployment, since the new code is abstracted away and controlled by a toggle, merging in these new code should not be a problem to production releases. The new code can be quickly shared and tested, while the production builds will simply toggle them off.


#### Reflection:
This article is very relatable to our team. Having a common code base that is shared among multiple production apps is not easy. Branching by abstraction provides a way for us to introduce features into the common library quickly and to be able to manage the releases of each app separately. I think in some ways we have been doing this already, where each module can be viewed as a layer of abstraction. I think through more layers of abstraction, we can have  more granular control over refactoring and new feature introductions. As we are moving towards a monorepo approach, I think this will be even more important to us, as we will be sharing new and refactored code even faster.
