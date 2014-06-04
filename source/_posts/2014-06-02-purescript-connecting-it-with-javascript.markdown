---
layout: post
title: 'PureScript: Connecting it with JavaScript'
date: 2014-06-02 12:00:00 +0100
comments: true
categories: programming
---

Parent post: [Thoughts on building a PureScript app](/thoughts-on-building-a-purescript-app)

PureScript is a language that's compiled to JavaScript. When I first thought about this, I kinda understood it, but not quite. Okay, the output is valid JavaScript, but how do the two languages **relate?** If I write something in PureScript, **how do I use it** in JavaScript?

And vice versa: all those JavaScript libraries, how can I **import** them in PureScript?

Here's how I think about it: JavaScript is this messy language full of unsafe and bad and ugly and PureScript should be it's... well, pure cousin. But I can't use PureScript in isolation. At the end, it will take data from the real messy world, it will communicate with JavaScript, so I better learn how to separate the two worlds in a safe manner.


# Calling PureScript functions

Lets create a simple module which will contain an `add` function:

```haskell
module MyMath where

add :: Number -> Number -> Number
add a b = a + b
```

Saving this little script to `MyMath.purs` and compiling it with:

`psc MyMath.purs --browser-namespace PureScript > index.js`

will generate a file which we can import in our test `index.html` page:

```html
<!DOCTYPE html>
<html>
<head>
  <script src="index.js"/>
</head>
<body>
</body>
</html>
```

Upon loading the script, we can access our function like this:

```javascript
var res = PureScript.MyMath.add(3)(4);
console.log(res); // will print 7
```

A couple of notes here:

* The `--browser-namespace PureScript` flag I used told the compiler to export all modules in an object called `PureScript`. If you leave this flag out, the name of this object will default to `PS`
* The reason why I call `MyMath.add(3)(4)` in this rather strange way is because functions exported by PureScript always take **exactly zero or one** arguments. `MyMath.add(3)` in fact returns a new function which takes the second argument, which feels really strange if you've never seen it before. If you want to know more about *why* it is like that, you can check the Wiki article on [currying](http://en.wikipedia.org/wiki/Currying)

Co calling PureScript code is easy:

1. Write your modules
2. Compile them into an `index.js` script
3. Import the script in the `index.html` page
4. Call your functions


# Calling JavaScript functions

The first resource you want to check out is the PureScript documentation on [Foreign Function Interfaces](http://docs.purescript.org/en/latest/ffi.html#foreign-function-interface).

The next thing you need to realize is this: **at the end of the day, PureScript is just JavaScript that runs in some context.** There are global variables around. You just can't see them by default.

Let's upgrade the example above. We'll define a function in the global scope:

```html
<!-- index.html -->
<head>
  <script type="text/javascript">

    happySadParity = function(number) {
      return number % 2 == 0 ? "Happy!" : "Sad. :("; 
    }

  </script>
  <script src="index.js"/> <!-- our PureScript file -->
</head>
```

Now let's reference it from our compiled PureScript code:

```haskell
module MyMath where

foreign import happySadParity :: Number -> String

add :: Number -> Number -> Number
add a b = a + b

emotionalSum :: Number -> Number -> String
emotionalSum a b = happySadParity $ add a b
```

And finally, let's call our new function:

```javascript
var res = PureScript.MyMath.emotionalSum(3)(4);
console.log(res); // will print "Sad. :("
```

Let's revise. In order to call a JavaScript function from PureScript, you need to:

1. Have the function defined in the scope
2. Import it with `foreign import yourFunction :: InputType -> OutputType`
3. Use it like it's a PureScript function

Hopefully, life makes sense now.

# Important closing notes

### There are other ways of compiling PureScript

Instead of running `psc` which combines all modules in a single file, you can use `psc-make`. The difference is: `psc-make` exports CommonJS modules in separate folders in a directory called `output`.

This is particularly interesting if you're using PureScript functions from Node.js. If you copy the contents of `output` you should be able to do this:

```javascript
var myMath = require('MyMath');
console.log(myMath.add(10, 20));
```

### You can define JS functions inline

Instead of writing:

```haskell
foreign import happySadParity :: Number -> String
```

which relies on having `happySadParity` in scope, you can just define the function on the fly:

```haskell
foreign import happySadParity "\
\  function happySadParity(n) {\
\    return number % 2 == 0 ? 'Happy!' : 'Sad. :(';\
\  }" :: Number -> String
```

### Is it safe?

No, it's not. By binding code this way, you get **none** of the safety that PureScript offers.

Think about it. If you shove an exception in a PureScript function that doesn't know how to handle it, well, your script will crash, no matter how typechecked the language is.

Likewise, if you call a JavaScript function and you're not prepared for it to to return `null` or `undefined` -- you're screwed.

### How do I make it safe?

I'll cover that topic in another post which I'll link here.
