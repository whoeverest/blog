---
layout: post
title: 'The "Read" bookmarklet'
date: 2014-05-19 12:00:00 +0100
comments: true
categories: programming
---

Every once in a while, I open a page that I want to read, and it looks like this:

[![wide-page.png](https://d23f6h5jpj26xu.cloudfront.net/8tyceu3plktg_small.png)](http://img.svbtle.com/8tyceu3plktg.png)

No `max-width` and small `font-size`, which makes it unreadable.

To fix this, I've created a really tiny bookmarklet which narrows the page and proportionally enlarges the font size, so it looks like this:

[![narrow-page.png](https://d23f6h5jpj26xu.cloudfront.net/xf1l2lnofkty1g_small.png)](http://img.svbtle.com/xf1l2lnofkty1g.png)

The code:

```javascript
javascript:(function() {
  var b = document.getElementsByTagName('body')[0];
  b.style.width = '40em';
  b.style.background = 'white';
  b.style.fontSize = '1.2em';
  b.style.marginLeft = 'auto';
  b.style.marginRight = 'auto';
})()
```

If you want to try it out, just create a new Bookmark, then paste the code above in the URL field.