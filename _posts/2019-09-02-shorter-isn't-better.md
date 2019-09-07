---
layout: post
title: "Shorter functions are't always better"
date: 2019-09-05 10:00:00 +0800
categories: blog
author: Lex Huang
---

# Shorter functions aren't always better

This is just a thought that came to me during day to day code reviews and I want to share the idea with you. My point is that for functions, readability, understandability and maintainability is much more important the length and the abstraction level.

## A Trivial Example

Consider you want to implement a simple algorism that show the balance in foreign currency given a list of incomes and outcomes. I will do this in `Javascript`. For the sake of simplicity, I will **ignore** the `float point issue` of ECMAScript and `type/value validation` for the arguments.

So the algorism needs 3 arguments: a balance, list of numbers as income in positive number and outcome in negative number, and an exchange rate. And the result is a number.

### A Naive Solution

```Javascript
function getBalanceInFC (
    balance = 0, costs = [], exchangeRate = 1
) {
    return costs.reduce((acc, flow) => acc + flow, balance) / costs;
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

The code above is readable, simple and maintainable enough since each of functions has only one responsibility and pure. But the abstraction can go higher: we can eliminate the arguments which called `point-free style`.

The `point-free style` comes from math, like in geometry it emphasis the surface/space over individual points. In programming, it simply means functions that never mentioning the actual arguments they will be applied to.

One way to achieve this is by using `eta conversion`:

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
> Expression
>
>> f(g(x))
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
	const subCurry = (args) => args.length === f.length ? f.apply(null, args) : (x) => subCurry([...args, x]);
	return x => subCurry([x]);
}

const after = f => g => x => f(g(x));
```

Then in order to do the eta-conversion, we also need curried version of `getBalance` and `getFC`:

```Javascript
const cGetBalance = curry(getBalance);
const cGetFC = curry(getFC);
```
Now we are ready to go point-free, I will do this by equation reasoning:
1. writing a curried version of `getBalanceInFC`:

    ```Javascript
    const pfGetBalanceInFC = (balance = 0) => (costs = []) => (exchangeRate = 1) => getFC(getBalance(balance, costs), exchangeRate)
        // = (balance = 0) => (costs = []) => (exchangeRate = 1) => cGetFC(getBalance(balance, costs))(exchangeRate);
    ```

2. replacing the `getFC` using its curried version:

    ```Javascript
    const pfGetBalanceInFC = (balance = 0) => (costs = []) => (exchangeRate = 1) => cGetFC(getBalance(balance, costs))(exchangeRate);
    ```

3. cancelling the `exchangeRate` parameter using eta-conversion:

    ```Javascript
    const pfGetBalanceInFC = (balance = 0) => (costs = []) => cGetFC(getBalance(balance, costs));
    ```
4. replacing the `getBalance` using its curried version:

    ```Javascript
    const pfGetBalanceInFC = (balance = 0) => (costs = []) => cGetFC(cGetBalance(balance)(costs));
    ```
    
5. Here we taking the advantage of curring that `cGetBalance(balance)` is a function! Let's say `cGetBalance(balance)` is equal to some function `f`, then the `cGetFC(cGetBalance(balance)(costs))` can be read as **`cGetFC` *after* `f` then *applied* with `costs`**, also by taking the advatage of declarativity of the functional programming:

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

So the point-free style `getBalanceInFC` is just `after(after(cGetFC))(cGetBalance)`. It's short and abstract but it's totally confusing! And for languages that has built-in `after` and `curry` like `Haskell` the issue unreadability gets even worse:

```Haskell
-- our black bird combinator
(...) = (.).(.)

getBalance:: Float -> [Float] -> Float
getBalance original costs = foldl (+) original costs

getFC:: Float -> Float -> Float
getFC balance exchangeRate = balance / exchangeRate

pfGetBalanceInFC = getFC...getBalance

main = print $ pfGetBalanceInFC 1 [1, 2] 2
```

Even though the point free style does has its advatages, if we extract this combinator:

```Javascript
const blackBird = f => g => after(after(f))(g); // (...) = (.).(.) in Haskell
```

Then the `blackBird` works for any number of parameters versions of function `f` given any 2 parameter function like `getBalance` that has the same amount of parameters:

```Javascript
const f = (x, y, z) => x + y + z;
const g = (x, y) => x / y;
const p = (w, x, y, z) => f((g(w, x)), y, z);
const blackBirdP = blackBird(curry(f))(curry(p)); 
```

Which means given same parameters, the execution result of `p` and `blackBirdP` is always the same. This does provide maintanability but still, hard to read and understand.

## Conclusion

Coding practice is a kind of social practice because it involves cooperations and communications. Communications require certain education background and intelligence between the participants. Here is an example when it comes to communications: Whenever it comes to a situation that people ask me to explain the idea of `monad`, the shortest and the best version is always the version comes from `James Iry`, from his highly entertaining [Brief, Incomplete and Mostly Wrong History of Programming Languages](http://james-iry.blogspot.com/2009/05/brief-incomplete-and-mostly-wrong.html):

> A monad is just a monoid in the category of endofunctors

I believe whenever after a person hearing this, the conversation ends.

Why? Because in order to understand the meaning of the sentence, the listener needs to understand a bunch of mathematical ideas like: `poset` and `monoid`, `category` connection between `category` and `poset`, `galois connection` and generalize it to `adjoint situation`, `functor` and `natrual transformations`, and `Kleisli Categories`, which if he does he won't even ask in the first place. But if I start with a simple commutative diagram, which I thought is the best part of the `category theory` that you can prove things by drawing strings and dots, and then proceed with the idea of `Kleisli Categories`, they might gain some understand about the intimidating idea.

Abstraction has advantages over enforcing certain functionalities and robustness for certain problem domain. But coders should choose their level of abstraction based on the knowledge background of their cooperators, especially when practicing in an enterprise scale.

So a good smell of a function is the function that always readable, understandable and maintainable to each and every team members, the length of a function doesn't matter that much.
