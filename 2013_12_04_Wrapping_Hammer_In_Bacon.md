Wrap Hammer in Bacon
====================

I've got a lot of question along the lines of "can I integrate Bacon with X". And the answer is Yes You Can. Assuming X is something with a Javascript API. For many values of X, there are ready-made solutions. TODO links/examples

- JQuery events is supported out-of-the box
- With Bacon.JQuery, you get more, including AJAX, two-way bindings
- Node.js style callbacks, with (err, result) parameters, are supported with Bacon.fromNodeCallback
- General unary callbacks are supported with Bacon.fromCallback
- TODO
- There's probably a plenty of wrappers I haven't listed here. Let's put them on the Wiki, shall we? TODO

In case X doesn't fall into any of these categories, you may have to roll your own. And that's not hard either. Using Bacon.fromBinder, you should be able to plug into any data source quite easily.

Example: Timer

Let's start with a simple example. Suppose you want to create a stream that produces timestamp events each second. There are options here.

1) Using Bacon.interval, you'll get a stream that constantly produces a value. Then you can use `map` to convert the values into timestamps.

    Bacon.interval(1000).map(function() { return new Date().getTime() })

2) Using Bacon.fromPoll, you can have Bacon call your function each 1000 milliseconds, and produce the current timestamp on each call.

    Bacon.fromPoll(1000, function() { return new Date().getTime() })

3) Using Bacon.fromBinder is an overkill here, but if you want to learn to roll your own streams, this might be a nice example: http://jsfiddle.net/PG4c4/

So, 

- you call `Bacon.fromBinder` and you provide your own "subscribe" function
- there there you register to your underlying data source. In the example, `setTimeout`. 
- when you get data from your data source, you push it to the provided "sink" function. In the example, you push the current timestamp
- from your "subscribe" function you return another function that cleans up. In this example, you'll call `setTimeout`

Example 2: hammer.js

Hammer.js is a library for handling touch events. Just to prove my point, I created a fiddle where I introduce "hammerStream" function that wraps any Hammer.js event into an EventStream.

http://jsfiddle.net/axDJy/1/

It's exactly the same thing as with the above example. In my "subscribe" function, I register an event handler to Hammer.js. In this event handler I push a value "hammer time!" to the stream. I return a function that will de-register the hammer event handler.

More examples

You're not probably surprised at the fact that all the included wrappers and generators ($.asEventStream, Bacon.fromNodeCallback, Bacon.interval, Bacon.fromPoll) are implemented on top of Bacon.fromBinder. So, for more examples, just dive into the Bacon.js codebase itself.
