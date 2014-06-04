---
layout: post
title: "Exiting early; nesting is a code smell"
date: 2014-01-28 12:00:00 +0100
comments: true
categories: programming
---

Nesting code more than 2 or 3 levels deep is a code smell and should be avoided. Here's a trick I learned and continue to use everyday.

Let's say you have:

```python
def some_function(name):
    if name in DB:
        # long block of code here
    else:
        return "Name not found"
```

You can transform this into:

```python
def some_function(name):
    if name not in DB:
        return "Name not found"
    # long block of code is not nested anymore
```

I call this pattern "returning early with the inverse condition". The cool thing is: **it stacks**. So instead of:

```python
def some_function(name):
    if name in DB:
        if some_other_condition:
            if not another_condition:
                # 4-levels deep nesting
```

you now have:

```python
def some_function(name):
    if name not in DB:
        return "Name not found"
    if not some_other_condition:
        return "Nope, can't do"
    if another_condition:
        return "Sorry mate"
    # long block of code here, not nested anymore
```

You can also do this in loops:

```python
for item in array:
    if not condition:
        break
    # code goes here
```

or even files:

```python
if some_condition_while_testing == 0:
    exit(0)
# the rest of the code
```

Perhaps it's obvious, but I still see a lot of code that could use this change. Indentation matters. First post written.

Cheers.