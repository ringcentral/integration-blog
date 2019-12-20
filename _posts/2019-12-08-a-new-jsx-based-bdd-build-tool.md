---
layout: post
title:  "A new Step-based BDD build tool - Crius"
date:   2019-12-08 23:30:00 +0800
categories: blog
author: Michael Lin
---

BDD (behavior-driven development) is an extension of TDD (test-driven development). It emphasizes on writing tests specifically to test the behaviors of the application before implementation process begins. DSLs are often used to fascilitate the expression of behaviors and expected results in a format that resembles natural languages. BDD is an affective development process in dealing with complex business requirements.

## Motivation

![BDD](/integration-blog/assets/2019-12-08-a-new-jsx-based-bdd-build-tool/bdd.png)

A typical BDD development process begins with Epics, these Epics are then broken down into User Stories to decribe the various features of the application. From User Stories, AC (Acceptance Criteria) will be listed to clearly define the behaviors and specifications of the feature. Because AC precisesly defined the behaviors and test conditions for all the features of the application, they are perfect for writing into automated tests.

The emergence of Cucumber in 2008 popularized the use of BDD in software development, which had a lot of influence over other BDD tools the came after Cucumber. The core value of Cucumber is the DSL that provided a way to express behaviors and appliction features similar to natural languages, including the popular clauses of Given, When, and Then. However, the actual test code are often kept in "steps" files and are language agnostic. The implementation of the test driver relies heavily on string pattern matching to link the feature definitions implicitly to the steps, often leading to performance issues difficulties in managing the steps. As the complexity of the application grows, it is often increasingly difficult to define unique step names due to many steps sharing similar words.

From our years of BDD practice, we had found several shortcomings of Cucumber that we need to address:

1. String Pattern Matching
2. Implicit link between features and steps
3. Lacks simple ways to re-use scenarios or steps

These shortcomings eventually led to the advent of Crius.

## What's Crius

Crius is our answer to Cucumber. Let's see some example code:

```jsx
@autorun(test)
@title('User add ${todo} item in todo list')
class AddTodoItem extends Step {

  // cucumber-like example table, with JavaScript literals
  @examples`
    | todo       | object             |
    | 'learning' | { obj: 'literal' } |
    | 'crius'    | { obj: 'hello' }   |
  `

  //which is equivalent to
  @examples([{
      todo: 'learning',
      object: { obj: 'literal' }
  }, {
      todo: 'crius',
      object: { obj: 'hello' }
  }])

  run() {

    // JSX-like declarative feature description
    return (
      // explicit reference to actions
      <Scenario desc='User login website' action={Login}>
        <Given desc='User navigate to todo list' action={Navigate} />

        // Allow Step class AddTodo to be used as an action to promote re-use
        <When desc='User type ${todo} in input field and click "add" button' action={AddTodo} />
        <Then desc='User should see ${todo} item in todo list' action={CheckTodo} />

        // action is optional
        <Then desc='Just an description' />
      </Scenario>
    )
  }
}

// Functional Step definition similar to Functional Components in React
const Login = async () => {
  // actual code to perform login
};

const Navigate = async () => {
  // navigate to todo list
}

// Step is similar to React Component class
class AddTodo extends Step {
  async run() {
    return (
      <>
        <TypeTodoText text={this.context.example.todo} />
        <SubmitTodo />
      </>
    );
  };
}

const TypeTodoText = async (props) => {
  // `props.text` is the todo text.
}

const SubmitTodo = async () => {
  // click the "add" button
};

const CheckTodo = async (_, { example }) => {
  // check todo item
};
```

### Features of Crius
* **Declarative and Expressive DSL** - By combining DSL charateristics of Cucumber and React
* **Re-usable Step definitions** - Provide better Step and Scenario composition
* **Step lifecycle** - Provide lifecycle hooks to have more control over tests
* **Plugin Support** - Allow more custom features to be easily added
* **Test Runner Agnostic** - Compatible to Jest, Mocha and Jasmine out of the box
* **Use JavaScript literals for examples** - Easily define complex object in examples
* **Lightweight** - Core source code is less than 17k

Compared to Cucumber, Crius offers the follow benefits:

* **No string pattern matching** - An explicit relationship between feature definition and test steps exist via direct reference
* **Syntax highlighting in IDE and Diff tools** - Since the feature definitions are written in TSX or JSX
* **Extensible through lifecycle hooks and plugins** - So you can customize the test solution to fit the project needs
* **Support static type checking (if using typescript)** - Helps you avoid typos and/or type errors with IDE support


## Conclusion

Crius's declarative Step design enriches the combined reuse of BDD in behavioral logic, while JSX-based featrue specification documentation is both good readability and test execution code. It will help us significantly improve building efficiency and sustainability when making BDD, and it will be object-oriented analysis and design, providing development and management teams with shared TDD tools for better collaboration.

Crius Repository:

[https://github.com/unadlib/crius](https://github.com/unadlib/crius)
