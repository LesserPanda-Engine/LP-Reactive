# Panda Reactive

Panda Reactive is a reactive programming plugin for Panda.js based on [Kefir](http://pozadi.github.io/kefir/).


# Examples

## Advanced Timer

```javascript
// Speed factor is also supported
game.Timer.speedFactor = 0.5;

// Emit a number every second
var srcEvents = game.R.sequentially(1000, [
    1, 2, 3, 4, 5, 6
]);
// Listen to timed events (like a normal timer)
srcEvents.onValue(function(t) {
    console.log('- %is passed -', t);
});

// Create a new stream without events emitted at 3x seconds.
var logStream = srcEvents.filter(function(x) {
    return (x % 3 !== 0);
});
logStream.onValue(function(t) {
    console.log('[Event]: spawned at %is', t);
});

// Create more streams.
// Note: If a stream is not yet subscribed
// it will not get updated. It means 
// creating streams causes zero overhead.
var s = srcEvents.map(function(t) {
    return t * 2;
});
var s1 = srcEvents.reduce(function(prev, next) {
    return prev + next;
});
var s2 = srcEvents.throttle(1500);
```

## Computed Property

```javascript
var myNameText = new game.Text('').addTo(game.scene.stage);

var firstPart = game.R.sequentially(200, ['P', 'Pa', 'Pan', 'Pand', 'Panda']).toProperty('');
var lastPart = game.R.sequentially(200, ['j', 'js']).delay(1000).toProperty('');

var fullName = game.R.combine([firstPart, lastPart], function(first, last) {
    return first + '.' + last;
}).toProperty('');

fullName.onValue(function(name) {
    myNameText.setText(name);
});
```

## Subscribe to Property Changes

```javascript
game.createClass('Player', {
    health: 2,
    init: function() {
        // Make sure this is called before creating properties
        this.enableProperties();

        // This will fire an event when `health` is less than 0
        var getKilled = this.createProperty('health').filter(function(c) {
            return c < 0;
        });
        // Call `remove` when received the killed event
        getKilled.onValue(this.remove.bind(this));
    },
    receiveDamage: function(damage) {
        // Call `set`, `incrementProperty` or `decrementProperty`
        // which will fire a property change event.
        this.decrementProperty('health', damage);
    },
    remove: function() {
        // Remove sprite, body, do whatever to clean up
        console.log('remove from world');
    }
});

// Remember to inject the mixin to enable property change events for 
game.MyBox.inject(game.R.PropertyMixin);
```


## Input Stream

```javascript
// Player will move 32px(up/down/left/right) each step
var STEP_DISTANCE = 32;

game.createScene('MyScene', {
    init: function() {
        // Create your player
        var player = /*...*/;

        // Scene has a property named "events" which is a 
        // EventEmitter instance. It'll emit all Panda.js
        // supported input events(keydown, mousedown...)
        var keydown = game.R.fromEvent(this.events, 'keydown');

        // Logic below ONLY happens when any key is pressed
        var whenToMoveLeft = keydown
            // Create a LEFT down event stream
            .filter(function(key) {
                return key === 'LEFT';
            })
            // "Translate" LEFT event to "move ONE step left"
            .map(function() {
                return { 
                    x: -1,
                    y: 0 
                };
            });
        
        // Also define logic for other 3 directions
        // - whenToMoveRight
        // - whenToMoveUp
        // - whenToMoveDown

        // Now lets combine them all
        var whenToMove = game.R.combine([
            whenToMoveLeft, 
            whenToMoveRight, 
            whenToMoveUp, 
            whenToMoveDown 
        ]);
        // And move our player
        whenToMove.onValue(function(direction) {
            player.position.x += direction.x * STEP_DISTANCE;
            player.position.y += direction.y * STEP_DISTANCE;
        });

        // That's it! 
        // Without any useless `update` code and it works well.
    }
});
```


# API Document

You can find docs of most APIs in the official site of [kefir](http://pozadi.github.io/kefir/). For other APIs please see the **Examples** section.

---

The MIT License (MIT)

Copyright (c) 2015 Sean Bohan

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
