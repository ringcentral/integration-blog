---
layout: post
title: "Shorter functions are't always better"
date: 2019-09-05 10:00:00 +0800
categories: blog
author: Lex Huang
---

# Shorter functions aren't always better

This is just a thought that came to me during day to day code reviews and I want to share the idea with you.

## A trivial example

Consider you want to implement a simple algorism that show the balance in foreign currency whenever you spend it. We will do this in `Typescript` and ignore the float point issue of ECMAScript for the sake of simplicity.

So the algorism just needs 3 arguments: a balance, how much you spent, and an exchange rate. And the out is a number.

### A naive Solution

### Compositionality

### Point-free Style

## Conclusion

Coding practice is a kind of social practice because it involves cooperations and communications. Communications require certain education background and intelligence between the participants. Here is an example when it comes to communications: Whenever it comes to a situation that people ask me to explain the idea of `monad`, the shortest and the best version is always the version comes from `James Iry`, from his highly entertaining [Brief, Incomplete and Mostly Wrong History of Programming Languages](http://james-iry.blogspot.com/2009/05/brief-incomplete-and-mostly-wrong.html):

> A monad is just a monoid in the category of endofunctors

I believe whenever after a person hearing this, the conversation ends.

Why? Because in order to understand the meaning of the sentence, the listener needs to understand a bunch of mathematical ideas like: `poset` and `monoid`, `category` connection between `category` and `poset`, `galois connection` and generalize it to `adjoint situation`, `functor` and `natrual transformations`, and `Kleisli Categories`, which if he does he won't even ask in the first place. But if I start with a simple commutative diagram, which I thought is the best part of the `category theory` that you can prove things by drawing strings and dots, and then proceed with the idea of `Kleisli Categories`, they might gain some understand about the intimidating idea.

Abstraction has advantages over enforcing certain functionalities and robustness for certain problem domain. But coders should choose their level of abstraction based on the knowledge background of their cooperators, especially when practicing in an enterprise scale.

So a good smell of a function is the function that always readable, understandable and maintainable to each and every team members, the length of a function doesn't matter that much.
