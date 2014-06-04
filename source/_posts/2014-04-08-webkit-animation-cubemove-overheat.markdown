---
layout: post
title: '"-webkit-animation: cubemove" in Chrome 29 can overheat your CPU'
date: 2014-04-08 12:00:00 +0100
comments: true
categories: programming
---

Open this page and carefully monitor the temperature of your CPU.

http://whoeverest.github.io/spinner/

It is steadily going up? If it is, it's time to update Chrome. Today the sensors on my CPU measured **97°C** and it was all because I had this tab open in the background. Yep, just those two orange boxes rotating around each other.

I was running Chrome v29 under Ubuntu 13.10.

The problem was how `webkit-animation: cubemove` was being rendered in this older version of Chrome. Running `apt-get update` followed by `apt-get upgrade` solved the problem for me.

Now I'm running Chrome v33 and everything's cool.

I recorded a video while I was debugging:

<iframe width="560" height="315" src="//www.youtube.com/embed/-g-X8xKYnxw" frameborder="0" allowfullscreen></iframe>

I'm running `watch sensors` in the top-right terminal. When I load the page, the temperature raises to +70°C. When I disable the CSS statement, it drops back to ~50°C.

Weird stuff.