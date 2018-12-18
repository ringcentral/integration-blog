---
layout: post
title:  "Reading Code Review Best Practices"
date:   2018-12-10 13:31:07 +0800
categories: blog
---

[Article Link](https://medium.com/palantir/code-review-best-practices-19e02780015f)

#### Summary:

Preparing code for review(for reviewees)
- Scope and size. Shorter changes are preferred over longer ones.
- Only submit complete, self-reviewed (by diff), and self-tested CRs(code reviews)
- Refactoring changes should not alter behavior. Separate refactoring and feature development.
- **Expensive human review time should be spent on the program logic rather than style, syntax, or formatting debates.** As this part, we can use tools to deal with it. But if we find that there exists style, syntax, or formatting problem, we need to point it out.

Performing code reviews(for reviewers)
- Review Implementation
  - Think about how you would have solved the problem
  - Are there any potential for useful abstractions
  - Try to “catch” authors taking shortcuts or missing cases by coming up with problematic configurations/input data that breaks their code.
  - Think about libraries or existing product code
  - Does the change follow standard patterns
  - Does the change add compile-time or run-time dependencies (especially between sub-projects)
- Review Legibility and Style
  - Think about your reading experience
  - Does the code adhere to coding guidelines and code style
  - Does this code have TODOs
- Review Maintainability
  - Leave feedback on code-level documentation, comments, and commit messages
  - Was the external documentation updated

#### Reflection:

Same with coding dojo, code review is a good way to share knowledge and improve code quality as well. Although our project benifits a lot from code review, sometimes code review adds pressure to reviewers and reviewees, what should we do when we want to create a new PR or how to perform code reviews always confuse me.

**Preparing code for review** is a good way to make the code easier to review which could save reviewers’ time and reduce the reviewing pressure.

Furthermore, this article gives a list of things a reviewer should pay attention to in a code review, this is very useful and practicable for teams to perform. However, I only list out the ones that are suitable for us.
