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

Consider you want to implement a simple algorism that show the balance in foreign currency given a list of incomes and outcomes. I will do this in `Typescript`. For the sake of simplicity, I will **ignore** the `float point issue` of ECMAScript and `type/value validation` for the arguments.

So the algorism needs 3 arguments: a balance, list of numbers as income in positive number and outcome in negative number, and an exchange rate. And the result is a number.

### A Naive Solution

```Typescript
function getBalanceInFC (
    balance: number = 0, costs:number[], exchangeRate:number
) {
    return costs.reduce((acc, flow) => acc + flow, balance) / costs;
}
```

### Compositionality

The code above actually is composited of two arithmetic operations: accumulating and dividing. Both of them has their corresponding meaning in the domain of banking and accounting.

So, in terms of single responsibility principle, reusability and readability, it would be nice if we decompose the function:

```Typescript
const getBalance = (balance: number, costs:number[]) => costs.reduce((acc, flow) => acc + flow, balance);

const getFC = (accounting: number, exchangeRate: number) => accounting / exchangeRate;

const getBalanceInFC = (
    balance: number = 0, costs:number[], exchangeRate:number
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

I would use the curried version of `compose` in `Ramda` as my `after`.

## Conclusion

Coding practice is a kind of social practice because it involves cooperations and communications. Communications require certain education background and intelligence between the participants. Here is an example when it comes to communications: Whenever it comes to a situation that people ask me to explain the idea of `monad`, the shortest and the best version is always the version comes from `James Iry`, from his highly entertaining [Brief, Incomplete and Mostly Wrong History of Programming Languages](http://james-iry.blogspot.com/2009/05/brief-incomplete-and-mostly-wrong.html):

> A monad is just a monoid in the category of endofunctors

I believe whenever after a person hearing this, the conversation ends.

Why? Because in order to understand the meaning of the sentence, the listener needs to understand a bunch of mathematical ideas like: `poset` and `monoid`, `category` connection between `category` and `poset`, `galois connection` and generalize it to `adjoint situation`, `functor` and `natrual transformations`, and `Kleisli Categories`, which if he does he won't even ask in the first place. But if I start with a simple commutative diagram, which I thought is the best part of the `category theory` that you can prove things by drawing strings and dots, and then proceed with the idea of `Kleisli Categories`, they might gain some understand about the intimidating idea.

Abstraction has advantages over enforcing certain functionalities and robustness for certain problem domain. But coders should choose their level of abstraction based on the knowledge background of their cooperators, especially when practicing in an enterprise scale.

So a good smell of a function is the function that always readable, understandable and maintainable to each and every team members, the length of a function doesn't matter that much.
