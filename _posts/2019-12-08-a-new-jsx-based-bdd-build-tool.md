---
layout: post
title:  "A new Step-based BDD build tool - Crius"
date:   2019-12-08 23:30:00 +0800
categories: blog
author: Michael Lin
---

> BDD(behavior-driven development) is an extension of TDD(test-driven development). BDD emphasizes the use of DSLs to express behavior and expected results to address the "problem space" complexity of the business.

## Motivation and Goal

Since 2008, Cucumber has brought some innovations and practices on BDD. Most other BDD tools draw on some of Cucumber's strengths and models. String matching patterns through DSLs (Scenario, Given, When, and Then) are an important feature. In addition to some of the performance losses and matching uniqueness that result from a large number of regular matching, it is more important to re-engineer and encapsulate them when it emphasizes the more in-depth structure of testing. As the test complexity of developing application systems increases, both integration test and E2E test often require clearer and more concise test model designs to make it easier to practice more maintainable BDD.

![BDD](/integration-blog/assets/2019-12-08-a-new-jsx-based-bdd-build-tool/bdd.png)

From a typical BDD development process, most of them go from basic product requirements to Epic, then to User Story, and finally into Acceptance Criteria. The Feature file formed by Acceptance Criteria often defines the close relationship between testing and use functional. At the Acceptance Criteria layer, we're trying to create a better model to fit the more complex functional requirements and test validation.

Through continuous trial and exploration, we find that behavior and validation descriptions in specific scenarios are appropriate for common concepts such as `Step`, and that Step's standardization and universal become very important. The standardized `Step` design will make it easy to build `Step` with its internal logic and state, then combine it to form a complex `Action`, and use `Action` for defined code of conduct descriptions (Scenario, Given, Then, and Then) with good readability.

So the new BDD framework we're trying to build should meet the following goals:

* **Proper declarative Step design**
* **Quickly build test behavior and validation logic**
* **Define readable specification documents**
* **Easy to cooperate with all parties involved**
* **Increase collaboration and maintenance efficiency**

## What's Crius

Crius is a Step-based BDD build tool. It is the primary expression of the DSLs with JSX, all behavior and expected results can be defined as Step, and the declarative Step design makes it easy to quickly build test logic.


### Using Example

```jsx
@autorun(test)
@title('User add ${todo} item in todo list')
class AddTodoItem extends Step {
  @examples`
    | todo       |
    | 'learning' |
  `
  run() {
    return (
      <Scenario desc='User login website' action={Login}>
        <Given desc='User navigate to todo list' action={Navigate} />
        <When desc='User type ${todo} in input field and click "add" button' aciton={AddTodo} />
        <Then desc='User should see ${todo} item in todo list' action={CheckTodo} />
      </Scenario>
    )
  }
}

const Login = async () => {
  // user entry and login
};

const Navigate = async () => {
  // navigate to todo list
}

const AddTodo = async (_, { example }) => {
  return (
    <>
      <TypeTodoText text={example.todo} />
      <SubmitTodo />
    </>
  );
};

const TypeTodoText = async (props) => {
  // `props.text` is the todo text.
}

const SubmitTodo = async () => {
  // click the "add" button
};

const CheckTodo = async () => {
  // check todo item
};
```

### Features

* **Specifications readable** - Feature file as JS/TS code
* **Step** - Support Class Step and Function Step
* **JSX** - Build and composition Step based on JSX
* **Declarative** - Predictable and easy to debug
* **Hooks** - Easy to logically decouple and reuse
* **Test Runners** - Adapt to various test frameworks

### Pros

* **Lightweight Library** - Core source code is less than 17k.
* **Easy to Understand** - Function Step and Class Step are concise for quick build.
* **Not Regular/String Matching Pattern** - `Scenario`, `Given`, `When` and `Then` are based on JSX.
* **Readable Feature File** - It facilitates collaboration in software development(Stakeholders, PM, QA, Dev, etc.).
* **Feature File as JS/TS Code** - No additional tool checks are required, and IDEs that support JS/TS by default are with no configuration.
* **Reusable Step** - JSX-based Step has composed multiplexing expressiveness (passed parameters by porps and context).
* **Step Plug-in** - Encapsulating and using plug-ins can be used to track Step and analyze its logs, making it easy to debug bugs, performance analysis, and more.
* **Adapted to Various Test Frameworks Runners** - Crius can be adapted to a variety of test frameworks, Jest, Mocha and Jasmine, etc.
* **Flexible Example Parameter Form** - Examples API(`@examples`) supports Array and template literals。
* **Good TypeScript Support** - Crius is based on TypeScript, which supports both JavaScript and TypeScript.

## Conclusion

Crius's declarative Step design enriches the combined reuse of BDD in behavioral logic, while JSX-based featrue specification documentation is both good readability and test execution code. It will help us significantly improve building efficiency and sustainability when making BDD, and it will be object-oriented analysis and design, providing development and management teams with shared TDD tools for better collaboration.

Crius Repository:

[https://github.com/unadlib/crius](https://github.com/unadlib/crius)
