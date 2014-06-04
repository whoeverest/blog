---
layout: post
title: Thoughts on building a PureScript app
date: 2014-06-01 12:00:00 +0100
comments: true
categories: programming
---

For some time now I've been interested in learning functional programming more deeply. I'm already familiar with some concepts: pure vs. impure functions, what does it mean for some code to cause side-effects, what are higher-order functions etc. I try to refactor my imperative code in such a way that most of it remains pure, while I pull out the impurities.

However, I've never wrote anything substantial in a purely functional language. By substantial I mean more then a recursive factorial function. I started reading "Learn You a Haskell for Great Good!" one year ago but I didn't get past the "Oh, cool!" stage.

Now I want to try something different. I've set myself a task: **build an HTTP server on which I can play Dominoes.**

The reason why I chose Dominoes is because I'm familiar with the domain; I already built the game in Node.js for [Gambit.com](https://www.gambit.com) so I have the API and model figured out. Now I want to try and rebuild a simplified version - in PureScript.

[PureScript](http://docs.purescript.org) is a language that resembles Haskell a lot. The main difference is that it compiles to JavaScript, which makes it possible to also use it on the client. It has two-way bindings with JavaScript so you can: 1) call PureScript functions from JS and 2) call JS functions from PureScript.

As I'm fighting my way through, I'll do a series of posts which describe how I feel and what I've learned.

Here's the project on GitHub: https://github.com/whoeverest/dominoes-purescript

List of posts in the series:

* [Connecting PureScript and JavaScript](/purescript-connecting-it-with-javascript)