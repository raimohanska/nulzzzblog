## Bacon.js Tutorial Part III : AJAX and Stuff

This is the next step in the Bacon.js tutorial series. This time we're
going to implement an "as you type" username availability check with
AJAX. The steps are

1. Create an `EventStream` of AJAX requests for use with
   `jQuery.ajax()`
2. Create a stream of responses by issuing an AJAX request for each
   request event and capturing AJAX responses into the new stream.
3. Define the `usernameAvailable` Property based on the results
4. Some side-effect: disable the Register button if username is
   unavailable. Also, show a message.

So, at this point we've got a Property called `username` which
represents the current value entered to the username text input field.
We want to query for username availability each time this property
changes. First, to get the stream of changes to the property we'll do
this:

    username.changes()

This will return an `EventStream`. The difference to the `username`
Property itself is that there's no initial value (the empty string).
Next, we'll transform this to a stream that provides jQuery compatible
AJAX requests:

    availabilityRequest = username.changes().map(function(user) { return { url: "/usernameavailable/" + user }})

The next step is extremely easy, using Bacon.UI.js:

    availabilityResponse = availabilityRequest.ajax()

This maps the requests into AJAX responses. Looks very simple, but
behind the scene it takes care of request/response ordering so that
you'll only ever get the response of the latest issued request in that
stream. This is where, with pure jQuery, we had to resort to keeping a
counter variable for making sure we don't get the wrong result because
of network delays.

So, what does the `ajax()` method in Bacon.UI look like? Does it do
stuff with variables? Lets see.

    Bacon.EventStream.prototype.ajax = function() {
      return this["switch"](function(params) { return Bacon.fromPromise($.ajax(params)) })
    }

Not so complicated after all. But let's talk about flatMap now, for a
while, so that you can build this kind of helpers yourself, too.

## AJAX as a Promise

AJAX is asynchronous, hence the name. This is why we can't use `map` to
convert requests into responses. Instead, each AJAX request/response
should be modeled as a separate stream. And it can be done too. So, if
you do

    $.ajax({ url : "/usernameavailable/jack"})

you'll get a jQuery
[Deferred](http://api.jquery.com/category/deferred-object/) object. This
object can be thought of as a
[Promise](http://wiki.commonjs.org/wiki/Promises/A) of a result. Using
the Promise API, you can handle the asynchronous results by assigning a
callback with the `done(callback)` function, as in

    $.ajax({ url : "/usernameavailable/jack"}).done(function(result) { console.log(result)})

If you try this in you browser, you should see `true` printed to the
console shortly. You can wrap any Promise into an EventStream using
`Bacon.fromPromise(promise)`. Hence, the following will have the same
result:

    Bacon.fromPromise($.ajax({ url : "/usernameavailable/jack"})).log()

This is how you make an EventStream of an AJAX request.

## AJAX with `flatMap`

So now we have a stream of AJAX requests and the knowhow to create
a new stream from a jQuery AJAX. Now we need to

1. Create a response stream for each request in the request stream
2. Collect the results of all the created streams into a single response
   stream

This is where `flatMap` comes in:

    function toResultStream(request) {
      return Bacon.fromPromise($.ajax(request))
    }
    availabilityResponse = availabilityRequest.flatMap(toResultStream)

Now you'll have a new EventStream called `availabilityResponse`. What
`flatMap` does is

1. It calls your function for each value in the source stream
2. It expects your function to return a new EventStream
3. It collects the values of all created streams into the result stream

Like in this diagram.

![flatMap](https://raw.github.com/wiki/raimohanska/bacon.js/baconjs-flatmap.png)

So here we go. The only issue left is that `flatMap` doesn't care about
response ordering. It spits out the results in the same order as they
arrive. So, it may (and will) happen that

1. Request A is sent
2. Request B is sent
3. Result of request B comes
4. Result of request A comes

.. and your `availabilityResponse` stream will end with the wrong
answer, because the latest response is not for the latest request.
Fortunately there's a method for fixing this exact problem: Just replace
`flatMap` with `switch` and you're done.

![switch](https://raw.github.com/wiki/raimohanska/bacon.js/baconjs-switch.png)

Now that you know how it works, you may as well use the `ajax()` method
that Bacon.UI provides:

    availabilityResponse = availabilityRequest.ajax()

POW!
