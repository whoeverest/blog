---
layout: post
title: The cumulative fuckup of impure code
date: 2014-05-16 12:00:00 +0100
comments: true
categories: programming
---

You know why keeping a purely functional codebase written in a mostly imperative language is hard?

If you have stack of **five nested and impure** functions and you refactor one of them so it becomes pure, now you have **four** impure functions and **one** pure function.

If you have a stack of five nested **pure** functions and you introduce a side effect in the inner-most function, now you have **at least five impure functions.**

My question is: how does one bubble the side effects up?

</haiku>