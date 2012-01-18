Things You Should Know About RX
===============================

As I've said before, RX for Javascript is a great library for event-rich Javascript apps. 
But as with many tools of great power, it allows you to do stupid things and get hurt.
In this posting I'm focusing on those features that will probably provide you with the most WTFs.

Some examples
-------------

A counter that increases or decreases its value based on clicks to `+` and `-` buttons can be implemented
like this:

~~~ .javascript  
  var incr = $('#incr').toObservable('click').Select(always(1))
  var decr = $('#decr').toObservable('click').Select(always(-1))
  var series = incr.Merge(decr)
    .Scan(0, function(total, x) { return total + x })
~~~

Is good? Well, as long as you subscribe at the right time. If you subscribe later, the later subscriber will 
get different results from the first one, because it's counter will start from 0 when it subscribes.

You could derive a `position` stream from a `movements` stream as follows:

~~~
  var position = movements
    .Scan(startPos, function(pos, move) { return pos.add(move) })
    .StartWith(startPos)
~~~

Is good? Well, the exact same problem. Position stream delivers different results depending when you subscribe.

Both examples seem perfectly ok, unless you know the details of RX. Let's take a walk.

Misconception #1 : Observable Is a Stream of Events
---------------------------------------------------

Well, I have said the above sentence countless times, regardless of how wrong it is. Actually Rx
would make a lot more sense (with less WTFs) if that were true. A real stream, in my opinion, would
provide all of its Subscribers with the same series of events after any moment of time `t`. Unfortunately with RX, 
the events that you receive after `t` depends on when you subscribed.

Why?

Hot and Cold Observables
------------------------

In RX, there are "hot" and "cold" observables. As Bnaya Eshet described in his [blog posting]
(http://blogs.microsoft.co.il/blogs/bnaya/archive/2010/03/13/rx-for-beginners-part-9-hot-vs-cold-observable.aspx):

>If a tree falls in a forest and no one is around to hear it,
>does it make a sound?
>if it do make a sound when nobody observed it, we should mark it as hot,
>otherwise it should be marked as cold.

So what? Well, simply put, hot observables are consistent among subscribers and cold ones are not. A 
hot observable can be seen as a list of `(time, value)` pairs. A hot observable won't
produce the an event to subscribers that subscribed after the event occurred. The cold ones are, well, 
something that depends on when you subscribe.

All Observables that are directly derived from some real-world events are hot. For instance, 
mouse and keyboard events will be consistent among all subscribers. But, if you combine them with a cold
observable, or apply a stateful combinator, such as `Scan`, you're not so safe anymore.

Observables from Arrays
-----------------------

`Observable.FromArray` serves you a cold Observable. It always spits out the same list of objects when
you subscribe. Doesn't provide a consistent mapping of time to events. Period.

Mixing Hot and Cold
-------------------

If you mix hot with cold, what do you get? Medium? So, if you

~~~
var hotness = $(document).toObservable("keyup")
var temperature = Observable.FromArray("coldness").Concat(hotness.Select(always("hotness")))
~~~

You might expect to get an observable starting with "coldness" and producing "hotness" at each keyup.
However, `StartWith` made your new observable a bit colder in the sense that any new subscriber will
always get "coldness" first.

No matter how you combine coldness with hotness, you won't get a hot Observable back. It won't be cold
as in "tree falling in the woods", but inconsistent anyway. Lukewarm? No, more like Groundhog Day.

Stateful Streams Using Scan
---------------------------

As I showed in the counter-example (pun intended) above, Scan will give you a Groundhog Day Observable.

StartWith
---------

Using `StartWith` won't save you. It's the same thing as concatenating with a cold stream of one event.

Workaround : Publish/Connect
-------------------------------

This is a workaround I already presented in my hot-tempered [Failing with RX-JS](http://nullzzz.blogspot.com/2011/02/failing-with-rx-js.html)
posting. You can make your Observable hot again by using `Publish` and `Connect`. For instance, you can 
fix the broken counter like this:

~~~ .javascript
  var series = incr.Merge(decr)
    .Scan(0, function(total, x) { return total + x })
    .Publish()
  var dispose = series.Connect()
~~~

So first you call `Publish` on your Observable. That'll give you an Observable, that will have a single connection to the
underlying Observable, no matter how many subscribers you add. Then you call `Connect`, which will start it, by calling
the `Subscribe` method of the underlying stream. It will also give you a back a "dispose" function for disconnecting.

The obvious drawback of this method is that there's more clutter. Also, the connection to the underlying stream won't be
automatically disconnected. You have to call `dispose` yourself.