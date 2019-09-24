---
layout: post
title: "Shorter Functions Aren't Always Better"
date: 2019-09-05 10:00:00 +0800
categories: blog
author: Lex Huang
---

# Shorter Functions Aren't Always Better

This is an idea that came to me during day to day code reviews that I want to share with you. A function's readability, understandability, and maintainability are more important than the length and abstraction level of the function.

## A Trivial Example

Consider that you want to implement a simple algorism that show the balance in foreign currency, given a list of incomes and expenditures. For the sake of simplicity, I will do this in `Javascript` and **ignore** the `float point issue` of ECMAScript and `type/value validation` for the arguments.

The algorithm needs 3 arguments: the balance, the list of numbers containing incomes with positive values or expenditures with negative values, and the exchange rate, and return a number value.

### A Naive Solution

```Javascript
function getBalanceInFC (
    balance = 0, costs = [], exchangeRate = 1
) {
    return costs.reduce((acc, flow) => acc + flow, balance) / exchangeRate;
}
```

### Compositionality

The code above actually is composited of two arithmetic operations: accumulating and dividing. Both of them has their corresponding meaning in the domain of banking and accounting.

So, in terms of single responsibility principle, reusability and readability, it would be nice if we decompose the function:

```Javascript
const getBalance = (balance = 0, costs = []) => costs.reduce((acc, flow) => acc + flow, balance);

const getFC = (balance = 0, exchangeRate = 1) => balance / exchangeRate;

const getBalanceInFC = (
    balance: number = 0, costs = [], exchangeRate = 1
) => getFC(getBalance(balance, costs), exchangeRate);
```

### Point-free Style

The code above is readable, simple and maintainable enough since each of functions has only one responsibility and pure. But the abstraction can go higher: we can eliminate the arguments for the `getBalanceInFC`, this coding style is called `point-free style`.

The `point-free style` comes from math, like in geometry it emphasis the surface/space over individual points. In programming, it simply means functions that never mentioning the actual arguments they will be applied to.

One way to achieve this is by using [eta-conversion](https://en.wikipedia.org/wiki/Lambda_calculus#%CE%B7-conversion):

> Given `z` is a single parameter function, then:
>
>> x => z(x)
>
> is equal to:
>
>> z
>
> itself.

But we need one more ingredients since javascript lack of a build-in `combinator` we call `after` in `Haskell`:
> Given any functions: `f` and `g`, the expression
>
>> x => f(g(x))
>
> can be written as:
>
>> f.g
>
> in haskell, read as `f after g`
> which can be translated into a prefix operator:
>
>> after(f)(g)

Noticed that the `after` is `curried`.

Here I will give a version of `curry` and `after`:

```Javascript
const curry = f => {
    // Need to remove the default value for `f`
    // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/length#Description
    const subCurry = (args) =>
        args.length === f.length
            ? f.apply(null, args)
            : (x) => subCurry([...args, x]);
    return x => subCurry([x]);
}

const after = f => g => x => f(g(x));
```

Then in order to do the eta-conversion, we also need curried version of `getBalance` and `getFC`:

```Javascript
// the prefix `c` stands for `curry`
const cGetBalance = curry(getBalance);
const cGetFC = curry(getFC);
```
Now we are ready to go point-free, I will do this by equation reasoning:
1. Writing a curried version of `getBalanceInFC`:

    ```Javascript
    // the prefix `pf` stands for `point-free`
    const pfGetBalanceInFC = (balance = 0) =>
                                (costs = []) =>
                                    (exchangeRate = 1) =>
                                        getFC(getBalance(balance, costs), exchangeRate)
    ```

2. Replacing the `getFC` using its curried version:

    ```Javascript
    const pfGetBalanceInFC = (balance = 0) =>
                                (costs = []) =>
                                    (exchangeRate = 1) =>
                                        cGetFC(getBalance(balance, costs))(exchangeRate);
    ```

3. Cancelling the `exchangeRate` parameter using eta-conversion:

    ```Javascript
    const pfGetBalanceInFC = (balance = 0) => (costs = []) => cGetFC(getBalance(balance, costs));
    ```
4. Replacing the `getBalance` using its curried version:

    ```Javascript
    const pfGetBalanceInFC = (balance = 0) => (costs = []) => cGetFC(cGetBalance(balance)(costs));
    ```

5. Here we are taking the advantage of curring that `cGetBalance(balance)` is a function! Let's say `cGetBalance(balance)` is equal to some function `f`, then the `cGetFC(cGetBalance(balance)(costs))` can be read as **`cGetFC` *after* `f` then *applied* with `costs`**, also by taking the advantage of the declarative side from functional programming paradigm:

    ```Javascript
    const pfGetBalanceInFC
        // = (balance = 0) => (costs = []) => after(cGetFC)(cGetBalance(balance))(costs);
        = (balance = 0) => after(cGetFC)(cGetBalance(balance));
    ```
6. For the expression above, we find a rule that `f.g ≅ \x->(f.g)(x) ≅ \x->f(g(x))` (for simplicity I'm using `Haskell` notations here). Then let's apply this rule again by taking `after(cGetFC)` as some function `f` and `cGetBalance`  as some function `g` then we can see the expression matches the rule perfectly:

    ```Javascript
    const pfGetBalanceInFC
        // = (balance = 0) => after(after(cGetFC))(cGetBalance)(balance);
        = after(after(cGetFC))(cGetBalance);
    ```

So the point-free style `getBalanceInFC` is just `after(after(cGetFC))(cGetBalance)`. It's short and abstract but it's totally confusing! Even though the point free style does has its advatages, if we extract the pattern above into a combinator:

```Javascript
const owl
    // = f => g => after(after(f))(g);
    = after(after)(after); // applying same rules above
```

and its Haskell version looks much more terrifying:

```Haskell
-- because it looks like an owl
owl = (.).(.)
```

Then the `owl` works for any versions of function `f` with any number of parameters. Given any 2 parameter function like `getBalance` that has the same amount of parameters:

```Javascript
const f = (x, y, z) => x + y + z;
const g = (x, y) => x / y;
const p = (w, x, y, z) => f((g(w, x)), y, z);
const owlP = owl(curry(f))(curry(g));

console.log(p(1,2,3,4) === owlP(1)(2)(3)(4)) //true
```

Which means given the same parameters, the execution result of `p` and `owlP` is always the same (one can prove the feature by the type signature of our `after` combinator). This does provide maintanability but still, hard to read and understand.

## Conclusion

Coding practices are social practices because because it involves cooperations and communications. Communications require certain common knowledge between the participants. For example, whenever people ask me to explain the concept of `monad`, the shortest and the best version is always the version comes from `James Iry`, from his highly entertaining [Brief, Incomplete and Mostly Wrong History of Programming Languages](http://james-iry.blogspot.com/2009/05/brief-incomplete-and-mostly-wrong.html):

> A monad is just a monoid in the category of endofunctors

I believe the conversation would end when most people hear this, because to understand the sentence, they need to understand a bunch of mathematical ideas like `poset`, `monoid`, category, and `endofunctors`, which would require knowledge of `Galois connection`, `adjoint situation`, `functor`, `natural transformations`. But if I started with simple a `commutative diagram`, which in my opinion is the best section of the category theory, I can walk them through the proof by just drawing lines and dots, then proceed to explain the concept of `Kleisli Categories`, then perhaps they can gain some understanding of these complex concepts.

Abstraction has its advantages over enforcing certain functionalities and robustness for certain problem domain. But coders should choose their level of abstraction based on the knowledge background of their cooperators, especially when practicing in an enterprise scale. So a good smell of a function is the function that always readable, understandable and maintainable to each and every team members, the length of a function doesn't matter that much.

## References

1. More info about `owl` can be found in the logic puzzle book [To Mock a Mockingbird](https://www.amazon.com/Mock-Mockingbird-Raymond-Smullyan/dp/0192801422)

2. Here is a Haskell version:

    ```Haskell
    -- because it looks like an owl
    owl = (.).(.)

    getBalance:: Float -> [Float] -> Float
    getBalance original costs = foldl (+) original costs

    getFC:: Float -> Float -> Float
    getFC balance exchangeRate = balance / exchangeRate

    -- main = print $ ((getFC.).getBalance) 1 [1, 2] 2
    main = print $ (getFC `owl` getBalance) 1 [1, 2] 2
    ```
3. Simple type proof for the our owl combinator:

    Given:

        (.):: (b → C) → (a → b) → (a → c)

    we want to figure out the result of:

        (.).(.)

    ∵ Let's marking the 3 combinators as #1, #2 for the two `(.)`, and #3 for the `.`, then:

        #1:: (b→c)→(a→b)→(a→c)

        #2:: (b1→c1)→(a1→b1)→(a1→c1)

        #3:: (b2→c2)→(a2→b2)→(a2→c2)

    ∴ We have:
        #1:: b2→c2

        #2:: a2→b2

    ∴ We have:

        b2:: b→c

        c2:: (a→b)→(a→c)

      And:

        a2:: b1→c1

        b2:: (a1→b1)→(a1→c1)

      Which means:

        b2:: b→c
          :: (a1→b1)→(a1→c1)

    ∴ We have:

        b:: c1→b1

        c:: a1→c1

    So the final reasult is:

        (.).(.):: a2→c2
               :: (b1 → c1)→[a→(a1→b1)]->[a→(a1→c1)]

    Which is exactly what you would get in GHC with `:type (.).(.)`.
    So once you feed the `owl` with 2 functions and 2 parameters for the second function, the result is `c1` which is the type of (partial) evaluation result of your first input function. That's the reason why the number of parameters does not matter for the function.
