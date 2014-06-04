---
layout: post
title: "Small changes that make code more debuggable"
date: 2014-03-27 12:00:00 +0100
comments: true
categories: programming
---

### Name the return expression

If you change:

```javascript
function add(a, b) {
    return a + b;
}
```

to:

```javascript
function add(a, b) {
    var res = a + b;
    return res;
}
```

you can `console.log(res)` to see if it has the correct value. In the first case, you have to copy-paste the `a + b` expression. It's more DRY if you don't copy-paste.

### Wrap the one-line body in brackets

This obviously matters only in a language with optional brackets around one-liners like:

```javascript
if (something)
    do_something();
```

If you to see if the code enters the branch, by inserting `console.log` , you have to add the brackets afterwards. If you always put brackets, no matter the length of the body, you'll always be able to debug. Plus, it works for situations where the body gets bigger later.

So instead write:

```javascript
if (something) {
    do_something();
}
```