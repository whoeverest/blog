---
layout: post
title: "How would you start building a 8-Ball game with real physics?"
date: 2014-02-26 12:00:00 +0100
comments: true
categories: programming
---

Here's what I'm thinking: there are two approaches, as far as I can tell. The first is to have a fixed step in time, and every step of the simulation to calculate the velocities, rotations and positions for the balls. The default "simulation" way.

The other one I find more interesting and presume it's correct. It consist of defining the board and it's static balls as a mathematical model, which, when given a "shot" input (cue hitting the ball) can start a chain of events at which the simulation unfolds.

In this post I'll try to explain my idea. I'm not a physicist, nor a mathematician. I haven't read what other people have written on the subject, I just want to have a little fun thinking about the topic. Chances are, it's a solved problem and what I'm saying is mostly incorrect.

## The Nature of a Pool Simulation

Unlike most of the games that I can think of, simulating Pool consists of two types of periods in time: the *"uneventful while ball rolling on the table"* type and *"packed with events, cluster of balls gets broken"* type.

In the first case, it's easy to calculate where the ball will be 1 second from now. No collision will happen, so no need to calculate the position 30 times in that second. Also, there's no need to calculate anything about the other balls.

In the second case, when breaking the balls for the first time, for example, 50 milliseconds is when it all happens. Where you calculated that the ball will be might just be wrong.

If you iterate over the balls and ask the Simulator "where will this ball be in 50ms" you can get into a situation where balls overlap, which is obviously impossible in real life. If a falling Mario is about to get *into* a solid wall, you can just move the poor fellow 3px up on the next frame and you're done. But here, arbitrarily moving the ball like that will make the physics feel phony.

You can fine grain the time resolution. Instead of 50ms, you can set it to 2ms, iterating 500 times in one second. But this would be a) slow and b) discriminatory to the "uneventful while ball rolling" type of frames of which you'll have a lot.

## A Graph of Collisions

Here's how I think the simulation should look like:

* The ball gets hit with a force, at a certain place which gives it velocity and spin
* Over the course of the next second, at every moment, the state of the white ball is defined by a formula
`F(initial conditions, time) = (velocity, spin)`
* I predict the next collision will happen at `time = 1.263s`
* Calculate the new paths of the two balls
* Calculate when the next collision will happen
* Repeat

When I think about "predicting collisions" I think about "crossing lines" in a way. If you were to draw lines which represent positions of the balls in the future, a collision is an intersection of lines.

Here's a diagram:

[![intersection-of-time-lines.png](https://d23f6h5jpj26xu.cloudfront.net/tznei1s3ztg5jw_small.png)](http://img.svbtle.com/tznei1s3ztg5jw.png)

The simulation unfolds as a graph of events, collisions. It's a chain reaction that is not bound to stepping through time at fixed intervals.

A Renderer is an object which works like this:

```python
while True:
    frame = Renderer(Game.positions.at(t=2.1))
    sleep(0.033)
```

If you build the system this way, you can use the "uneventful" frames for calculating in advance the complicated situations.

There, idea spit out. Now I'm going to try and implement some part of it. If I succeed, I'll update the post.