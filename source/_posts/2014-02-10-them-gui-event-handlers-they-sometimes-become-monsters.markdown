---
layout: post
title: "Them GUI event handlers, they sometimes become monsters"
date: 2014-02-10 12:00:00 +0100
comments: true
categories: programming
---

I've recently started building games for [Gambit.com](https://gambit.com). We build turn-based games and you can gamble for Bitcoins, that's the general idea.

When writing turn-based browser games, most of the client-side programming consists of handling events - player clicked on something, placed a figure on the board, dropped a tile somewhere.

So you need to implement the game logic in the GUI. Here's a real world requirement from the Dominoes game I'm building:

1. A player clicks on a tile from his hand
2. Next he clicks on a open end of the board in order to place the tile
3. If it's a valid move, the tile gets placed, the server gets notified etc.

So you say *"okay, simple enough"* and start writing code. The first would be to catch the `click` event and update `model.selectedTile`. You write a handler:

```javascript
this.on('click .player-tile', function(ev) {
  var tile = $(ev.currentTarget).attr('data-tile-value');
  this.model.selectedTile = tile;
})
```

This is a pseudo-Backbone code, but you get the idea.

Now you want to put a border around the selected tile. Where do you put that code? Well, in the handlers of course. Let's just add:

```javascript
this.on('click .player-tile', function(ev) {
  var tile = $(ev.currentTarget).attr('data-tile-value');
  this.model.selectedTile = tile;

  $(tile).css('border', '1px solid silver'); // added this
})
```

Wait. Now, what's this handler doing? It's catching the 'click' event **and** it's updating the view. You're in trouble.

Why should a click handler be even aware of the looks of the game? It's an innocent line of code - you might say. Okay.

Now let's tell the server the human player clicked an element, just for analytics sake.

Where do we put that code? In the click handler of course.

```javascript
this.on('click .player-tile', function(ev) {
  var tile = $(ev.currentTarget).attr('data-tile-value');
  this.model.selectedTile = tile;
  $(tile).css('border', '1px solid silver');

  // added this too
  $.post('/api/stats', { event: 'click-on-tile', tile: tile });
})
```

And where do we handle the error of the call, if any?

**Well in the click handler of course.** The system is quickly and surely turning into chaos.

**What should an event handler do, what's it's job?** Well:

1. Catch the event
2. Do some data transformation possibly (like taking the 'data-tile-value' out of the currentTarget)
3. Perhaps update the model - and finally
4. Notify the rest of the system that there's been an update

```javascript
this.on('click .player-tile', function(ev) {
  var tile = $(ev.currentTarget).attr('data-tile-value');
  this.model.selectedTile = tile;

  // at the end of the call, just notify about a change
  this.trigger('tile-selected', tile);
});
```

Now, want to highlight the tile? Want to notify the server. Sure:

```javascript
this.on('tile-selected', this.hightlightTile);
this.on('tile-selected', this.logStats);
```

Now that the `this.highlight` function is separated and does exactly one thing, you can call it in case of, lets say, you want to mark only a valid tile in the players hand.

## Watch out for

* Event handlers that do more then the necessary model updates and data transformations
* Functions (event handlers included) that do two things or more things at once, like highlighting a tile and sending the server an update
* Event handlers that know too much about the GUI or the model

## Try to do

* Emit another event at the end of the current events' handler; chain logic in this way
