---
layout: post
title:  "Explore a more complete front-end testing strategy"
date:   2018-12-18 12:30:00 +0800
categories: blog
author: Michael Lin
---


![test-strategy](/integration-blog/assets/2018-12-18-blog-a-more-complete-front-end-testing-strategy/test-strategy.jpg)

> The assumption in this article is that we are continuously developing a relatively large front-end project, and we have adopted a domain-driven design, as well as an object-oriented programming model. The front-end business logic is split into **`domain modules`**/**`UI components`**. So we may need to have a more complete testing strategy to assure such a front-end project.

## General front-end testing

Whether it's a traditional test model or a continuous delivery model, we typically define the following three types of tests:

![generic-test](/integration-blog/assets/2018-12-18-blog-a-more-complete-front-end-testing-strategy/generic-test.png)

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

![new-test-strategy](/integration-blog/assets/2018-12-18-blog-a-more-complete-front-end-testing-strategy/new-test-strategy.png)

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


![it-cover](/integration-blog/assets/2018-12-18-blog-a-more-complete-front-end-testing-strategy/it-cover.png)

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


----

中文版

----


![test-strategy](/integration-blog/assets/2018-12-18-blog-a-more-complete-front-end-testing-strategy/test-strategy.jpg)

> 本文假设我们在持续开发一个相对较大的前端项目, 我们采用了域驱动设计以及面向对象的编程模型。前端业务逻辑分为 **`domain modules`**/**`UI components`**。那么, 我们可能需要有一个更完整的测试策略来确保这样的前端项目。

## 通常的前端测试

无论是传统测试模型还是连续交付模型, 通常定义了以下三种类型的测试:

![generic-test](/integration-blog/assets/2018-12-18-blog-a-more-complete-front-end-testing-strategy/generic-test.png)

- E2E

E2E测试包括确保应用程序的集成组件按预期运行, 以及整个应用程序都在实际场景中进行测试, 例如与数据库、网络、硬件和其他应用程序进行通信。

- IT

集成测试是软件开发生命周期策略的关键。通常的, 小型软件系统在一个阶段进行集成和测试, 而较大的系统涉及多个集成阶段, 以构建完整的系统, 例如将模块集成到低级子系统中, 以便与较大的子系统集成。集成测试涵盖了软件系统性能、功能和可靠性的所有方面等。

- UT

单元测试是一种软件测试方法, 通过该方法对源代码、一个或多个计算机程序模块集以及相关的控制数据、使用过程和操作过程的各个单元进行测试, 以确定它们是否适合使用。

## 通常的前端测试有什么问题？

在评估测试策略的完整性时, 我们尝试使用以下一些条件或因素对其进行验证:

* 通过验收标准
* 尽早捕获Bugs
* 运行速度更快, 尤其是在越上层的测试
* 高效排查Bugs
* 测试编写和维护成本
* 持续重构的风险

我认为更好的测试策略应该尽可能与上述项目保持一致。

在上文提到的通常的策略中，E2E能覆盖更多AC条件，但通常它运行频率不高；IT是运行频率会比E2E高，但IT常常包括集成了app几乎整体App本身，通常这样情况下也就会比较臃肿的；而UT在这样的策略下通常占比重较多，这样虽然逻辑覆盖率不错，但在较大程度重构中通常UT也会随着更改，当然了，大多这样的UT也没有什么太大问题。

从尽早抓到bugs角度说，是不是意味着通常测试策略能有更好的改进. 因为E2E测试极有可能是一个比较长的时间内才会运行一次，而不是每次PR甚至是每次代码commit，因为E2E运行通常是缓慢而且不稳定的，它是运行成本最高的自动化测试。

有时候我们会有一些集成测试会启动一个臃肿的集成系统，可能它里面已经包括很多mock，也能不断的重复运行测试，但是当集成测试的case越来越多，我们甚至都无法保证在每个pr内比较快速完成，在资源有限的CI环境可能是半小时，甚至更久。

随着系统变得越来越复杂，我们需要一个完整的测试策略来告诉我们就是这些测试中有什么cases运行失败了，我们通过测试报告能比较高效地抓到bugs。无论是网络不稳定、后端服务器APIs异常、前端的domain modules异常 或者 UI components 异常等等因素，我们都能快速定位到这个bugs。显然，通常的测试策略能提供的查找bugs帮助是有限的。例如 UT 成功了，IT 失败了，E2E 也失败了。我们却很难从这样的测试报告中分析到更明确的信息。

编写测试代码的成本应该要和可持续交付的开发模式进行一些平衡。在AC定义清楚的情况下，只是我们测试代码应该能覆盖AC所描述的信息。理论上，如果所有AC都由E2E来完全实现，这样也能实现对AC的验证。但显然，这样带来是一种极不平衡的测试不稳定性以及运行低效率。

如果单元测试足够大，这样是不是能保证我们的AC能被验收而变得可行呢？这应该取决于单元测试的维护成本，我们的每次重构都将修改代码，以及涉及代码的单元测试，那么单元测试的代码易变性，是不是意味着我们应该需要比UT更上层的测试来保证我们重构的代码逻辑。

## 解决这个问题的关键是什么？

在我们提到一些测试策略需要考虑的问题中，基于我们ATDD的可持续交付开发模式，AC的保证显然是最重要的，好的测试策略应该能保证每次的重构是有充分的信心；同时，在运行速度、查找bugs以及维护测试代码成本上有很好的平衡。在这些额外要素中，我们非常不建议走极端的方式，而是一种类似木桶原理的方式，让我们的测试策略有更完整。

## 提出更完整的测试策略

![new-test-strategy](/integration-blog/assets/2018-12-18-blog-a-more-complete-front-end-testing-strategy/new-test-strategy.png)

- E2E

这里的E2E应该实现最重要的AC部分，同时最好能支持冒烟测试／UI测试／多浏览器兼容测试。

- IT3

IT3是一种基于mock 服务的整体系统的集成测试，它能运行E2E的代码，但是它不会真实的启动一个浏览器来测试，所有的测试都在node.js中运行。因为它mock了服务器资源也mock了浏览器的真实的dom，所以它比E2E更快，它能不断的重复运行。这里需要强调的是，IT3是完全复用运行E2E的代码， 而上文提到的通常测试策略中的IT是通常没办法复用E2E的代码。

- IT2

IT2是最小集合的UI和最小集合的modules的集成测试，它同样也基于mock服务（包括服务器资源/DOM/BOM），由于它都是启动最小集合的模块，所以它的测试运行速度和最小集合的可运行性都能被保证，它比IT3更快。同时它IT3又有很好的分治，IT2负责最小集合，而IT3是负责整体的集合。除了最小集合，编写IT2和IT3差别并不大，它能满足通过AC的映射关系。当然了，除非是底层无依赖的模块出问题，不然我们其实更容易通过IT2/IT3的测试报告得中bugs的定位。

- IT1

IT1是只是最小集合的modules的集成测试，它只需要mock服务器资源，由于它只启动最小集合modules，所以它是比IT2更快的运行。在AC中的一个或多个步骤是能被转化成IT2测试。通过IT1/IT2/IT3的测试报告，我们也更容易推断出bugs的位置或原因。

- UT

在IT2谈到的底层模块(较少依赖的模块或者无依赖模块)的测试问题，我们建议是这样的模块适合更完整的UT，尤其是核心函数，另外就是其他模块的核心模块，或者helper function都可以考虑进行单元测试，它在很多场合下，能帮忙AC覆盖更多的examples。它是IT1/IT2/IT3的重要补充。


![it-cover](/integration-blog/assets/2018-12-18-blog-a-more-complete-front-end-testing-strategy/it-cover.png)

在不同的测试类型中，涵盖的测试因素也不同。那么我们希望以这样的测试策略能更完整和高效的满足测试各种因素：E2E测试涵盖了几乎全部的测试因素，IT3比E2E少了真实的服务器APIs和真实的浏览器，IT2比T3少了一些非必须的模块和UI组件，IT1比IT2少了UI组件，而UT只是少量覆盖核心逻辑部分。

在这样的测试策略下，我们能制定更好的运行测试代码的策略，我们在提交commit的时候可以运行UT/IT1，我们在提交PR的时候，我们可以运行UT/IT1/IT2甚至是IT3，而在我们回归的时候或者我们定期（一周或几天）运行一次E2E。在这样的运行体系下，我们可以在保证一定验证AC的情况下，同时也能保证一定的运行效率的平衡，同时不同类型的测试报告也会有助于bugs的定位。

## 如何实现这个更完整的测试

演示伪业务代码：

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

演示AC：

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

演示伪测试代码：

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


## 结论

制定测试策略的时候，我们需要考虑的因素很多，从基于AC的正确验证的出发点，我们应该对运行策略，运行效率，编写和维护测试的成本， bugs的易查找和重构保证性等重要的因素也进行综合平衡考虑，而不应该走极端。在保证一定AC的情况，我们希望这样E2E/IT3/IT2/IT1/UT能从多方面都对代码的质量和项目工程质量上有保证，同时又足够敏捷，可进行持续交付。
