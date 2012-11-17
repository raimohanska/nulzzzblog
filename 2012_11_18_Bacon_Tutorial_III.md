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

Not so complicated after all.
