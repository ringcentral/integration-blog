---
layout: post
title:  "Explore a more complete front-end testing strategy"
date:   2018-12-18 12:30:00 +0800
categories: blog
author: Michael Lin
---


![test-strategy](https://raw.githubusercontent.com/unadlib/integration-blog/testing_strategy/_site/assets/test-strategy.jpg)

> The assumption in this article is that we are continuously developing a relatively large front-end project, and we have adopted a domain-driven design, as well as an object-oriented programming model. The front-end business logic is split into **`domain modules`**/**`UI components`**. So we may need to have a more complete testing strategy to assure such a front-end project.

## General front-end testing

Whether it's a traditional test model or a continuous delivery model, we typically define the following three types of tests:

![generic-test](https://raw.githubusercontent.com/unadlib/integration-blog/testing_strategy/_site/assets/generic-test.png)

- E2E

End-to-end testing involves ensuring that the integrated components of an application function as expected. The entire application is tested in a real-world scenario such as communicating with the database, network, hardware and other applications.

- IT

Integration testing is a key software development life cycle strategy. Generally, small software systems are integrated and tested in a single phase, whereas larger systems involve several integration phases to build a complete system, such as integrating modules into low-level subsystems for integration with larger subsystems. Integration testing encompasses all aspects of a software system's performance, functionality and reliability.


- UT

Unit testing is a software testing method by which individual units of source code, sets of one or more computer program modules together with associated control data, usage procedures, and operating procedures, are tested to determine whether they are fit for use.



## What's wrong with the general front-end testing?

When we are evaluating the integrity of a test strategy, we try to validate it with some of the following criteria:

* Pass acceptance criteria
* Catch bugs as early as possible
* Faster running speed, especially at the top level of testing
* Find bugs efficiently
* Test writing costs and maintenance costs
* Continuous refactoring risks

I think a better test strategy should be as consistent as possible with the above items.

In the general testing strategy mentioned above, E2E can override more AC conditions, but usually it runs less frequently; IT is run more frequently than E2E, but IT often includes an integrated app in almost the entire system, which is often more bloated in this case, and UT typically accounts for a larger proportion of such this strategy, although the logical coverage is good, but in a large refactoring usually UT will also change, of course, most of these UT is not too much of a problem.

From the point of view of catching bugs as early as possible, does that mean that the general testing strategy can be better improved? Because the E2E test is most likely to run once in a relatively long period of time, rather than every PR or even every code commit, because the e2e run is usually slow and unstable, it is the most expensive automated test to run.

Sometimes we have some integration tests that start a bloated integrated system, maybe it already includes a lot of mock, and you can keep running tests over and over again, but when there are more and more cases of integration testing, we can't even guarantee faster completion within each PR, and in a resource-constrained CI environment, it could be half an hour, or even longer.

As a system becomes more complex, we need a complete testing strategy to tell us what cases failed in these tests, and to enable us to catch bugs more efficiently through test reports. Whether it is network instability, back-end server APIs exceptions, front-end domain modules exceptions or UI components exceptions, and so on, we can quickly catch these bugs. Obviously, the general testing strategy can provide limited help in locating bugs. For example, UT has succeeded, IT has failed, and E2E has failed too. It is difficult for us to analyze clearer information from such test reports.

The cost of writing test code should be balanced with a continuous delivery development model. When the AC definition is clear, it's just that our test code should be able to align the information described by AC. In theory, if all AC is fully implemented by E2E, this will also enable the verification of AC. But obviously, this brings with it a highly unbalanced test instability and inefficient operation.

If the unit tests are adequate enough, will this ensure that our AC can be accepted and become viable? This should depend on the maintenance cost of the unit test, each time we refactor our code, we will have to modify the unit tests involved in these code as well, which means we need a higher level of test to ensure the quality and correctness of these code, despite their fast changing nature.

## What's the key to solving the problem?

Among the issues we mentioned for some test strategies, based on our ATDD sustainable delivery development model, AC's assurance is clearly the most important, and a good test strategy should ensure that every refactoring is fully confident, while at the same time, there is a good balance between running speed, finding bugs, and maintaining the cost of test code. Among these additional elements, we very much do not recommend going in the extreme way, but rather a way of resembling the [Liebig's law][1] to make our test strategy more complete.

## Propose a more complete testing strategy

![new-test-strategy](https://raw.githubusercontent.com/unadlib/integration-blog/testing_strategy/_site/assets/new-test-strategy.png)

- E2E

The E2E here should implement the most important AC parts, and it is best to support smoke testing/UI testing/multi-browser compatibility testing.

- IT3

IT3 is an integrated test of the overall system based on the mock service, which can run E2E code, but it does not actually start a browser to test, and all tests run in Node.js. Because it's a mock for back-end server APIs and the browser's real DOM, so it's faster than E2E, and it keeps running over and over again. In particular, it is important to note that IT3 is a fully reusable E2E code, and IT is the general testing strategy mentioned above that is often not able to reuse E2E code.

- IT2

IT2 is an integrated test of the minimum set of UI and the minimum set of domain modules, and it is also based on the mock service (including server APIs/DOM/BOM), because it is the module that starts the minimum set, so its test run speed and the operational performance of the minimum set can be guaranteed, it is faster than IT3. At the same time, it IT3 and has a good partition, IT2 is responsible for the minimum set, and IT3 is responsible for the overall collection. In addition to the minimum set, writing IT2 and IT3 is not very different, it can satisfy the mapping relationship through AC. Of course, unless there is a problem with the non-dependent module at the bottom, it is actually easier for us to pass the IT2/IT3 test report to bugs's positioning.

- IT1

IT1 is just the minimum set of modules integration tests, it only requires the mock back-end server APIs, because it only starts the minimum set domain modules, so it is faster than IT2 run. One or more steps in AC are capable of being converted into IT2 tests. Through IT1/IT2/IT3 's test report, it is also easier to infer the location or cause of bugs.

- UT

In IT2 talking about testing problems with the underlying modules (less dependent modules or non-dependent modules), we recommend that such modules be suitable for more complete UT, especially core functions, and that the core modules of other modules, or helper functions, can be considered for unit testing, and that in many cases, Can help AC cover more examples. It is an important addition to IT1/IT2/IT3.


![it-cover](https://raw.githubusercontent.com/unadlib/integration-blog/testing_strategy/_site/assets/it-cover.png)

Among the different test types, the test factors covered are also different. Then we hope that with such a test strategy can be more complete and efficient to meet the various factors of testing: The E2E test covers almost all of the test factors, IT3 is less than E2E real server APIs and real browsers, IT2 is less than T3 a number of non-essential modules and UI components, IT1 has fewer UI components than IT2, and UT only covers a small number of core logical parts.

In such a test strategy, we can develop a better strategy to run the test code, we can run ut/it1 when submitting commit, we can run PR when we submit the UT/IT1/IT2 or even IT3, And at the time of our return or when we run the E2E regularly (a week or a few days). In such a operating system, we can ensure that the AC is guaranteed to be verified, but also to ensure a certain degree of operational efficiency balance, while different types of test reports will also contribute to the positioning of the bugs.

## How to implement building this more complete test

Example for business code:

```javascript

@Module()
class Foo {
    a() {}
    _x() {
        //No dependent module core logic
    }
}

@Module({ dependences:['Foo'] })
class Bar() {
    b() {}
     _y() {
        //No dependent module core logic
    }
    get name(){}
}

@Module({ dependences:['Foo', 'Bar'] })
class Foobar() {
    c() {}
    get name(){}
}

const store = createStore(
    //...factory module
);

const FoobarContainer = (props) => (
    <div onClick={props.foobar.c}>
        {props.foobar.name}
    </div>
);

const BarContainer = (props) => (
    <div onClick={props.bar.b}>
        {props.bar.name}
    </div>
);

class App extends Component {
    render() {
        return (
        <div>
            {this.props.foobar ? (
                <FoobarContainer {...this.props}>
            ): null }
            {this.props.bar ? (
                <BarContainer {...this.props}>
            ): null }
        </div>
        );
    }
}

render(
    <App store={store} />,
    mountNode
);
```

Example for acceptance criteria

```cucumber
Feature: AC

  Scenario Outline:
    Given User saw 'b' node
    When User click 'b'
    Then User should see 'b' changed
    When User click 'f'
    Then User should see 'f' changed

  Scenario Outline:
    Given User saw 'c' node
    When User click 'c'
    Then User should see 'c' changed
    When User click 'e'
    Then User should see 'e' changed

```

Example for testing

```javascript

// E2E & IT3
test(() => {
    const app = getApp();
    app.find(nodeSelectorB).click();
    expect(result).toBe(expectedValue1);
    app.find(nodeSelectorF).click();
    expect(result).toBe(expectedValue2);
});

test(() => {
    const app = getApp();
    app.find(nodeSelectorC).click();
    expect(result).toBe(expectedValue1);
    app.find(nodeSelectorE).click();
    expect(result).toBe(expectedValue2);
});

// IT2
test(() => {
    const barContainer = getMinimalSet(BarContainer);
    barContainer.find(nodeSelector).click();
    expect(result).toBe(expectedValue2);
});

test(() => {
    const foobarContainer = getMinimalSet(FoobarContainer);
    foobarContainer.find(nodeSelector).click();
    expect(result).toBe(expectedValue1);
});


// IT1
test(() => {
    const app = getMinimalSet(Bar);
    app.b();
    expect(result).toBe(expectedValue);
});

test(() => {
    const app = getMinimalSet(Foobar);
    app.c();
    expect(result).toBe(expectedValue);
});

// UT
test(() => {
    const result = Foo.prototype._x();
    expect(result).toBe(expectedValue);
});

test(() => {
    const result = Bar.prototype._y();
    expect(result).toBe(expectedValue); 
});
```

## Conclusion

There are many factors we need to consider when developing a test strategy. From the point of view of correct verification based on AC, we should also consider the operation strategy, operating efficiency, writing and maintaining the cost of testing, bugs easy to find and refactoring assurance and other important factors, and should not go to extremes. In the case of ensuring a certain AC, we hope that this E2E/IT3/IT2/IT1/UT can be guaranteed in many ways to the quality of the code and the quality of the project engineering, while being agile enough for continuous delivery.



[1]: https://en.wikipedia.org/wiki/Liebig%27s_law_of_the_minimum