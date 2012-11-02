Back from Callback Hell with Bacon.js
=====================================

I've heard the words Callback Hell quite a many times lately. I'll take
it that you know what I'm talking about. Just gonna briefly show how to
get Back from Hell with [bacon.js](https://github.com/raimohanska/bacon.js)...

So, what about if you want to perform two independent AJAX calls and do something with the
values of both? You might

    $.ajax("/cats").done(function(cats) {
      $.ajax("/dogs").done(function(dogs) {
        doStuffWithCatsAndDogs(cats, dogs)
      })
    })

That's a minor callback hell allready. How about

    var cats = Bacon.fromPromise($.ajax("/cats")
    var dogs = Bacon.fromPromise($.ajax("/dogs")
    Bacon.combineAsArray(cats, dogs).onValues(doStuffWithCatsAndDogs)

Know what? Your AJAX just got parallel, too. Maybe we should refactor
this too:

    function ajax(url) { return Bacon.fromPromise($.ajax(url)) }
    Bacon.combineAsArray(ajax("/cats"), ajax("/dogs")).onValues(doStuffWithCatsAndDogs)

That's it.

Now if you wanted both cats and dogs and then do something with both
*and* something based on an AJAX query triggered by user input... You
would do
    
    $.ajax("/cats").done(function(cats) {
      $.ajax("/dogs").done(function(dogs) {
        $("input.name").onKeyUp(function() {
          $.ajax("/stuff/" + $("input.name").val()).done(function(stuff) {
            var allTheStuff = {
              cats: cats,
              dogs: dogs,
              stuff: stuff
            }
            doStuff(allTheStuff)
          })
        })
      })
    })

That's some serious callback hell. See any problems? For one, the user
input won't be captured until both AJAX calls (cats, dogs) are
completed, and they are not done in parallel, either. Further, the AJAX
calls triggered by user input do not necessarily return in the same
order as they are issued, so the last call to "doStuff" might be based
on stale data. That might prove hard to fix?

With Bacon, you could do

    function ajax(url) { return Bacon.fromPromise($.ajax(url)) }
    function urlForStuff(name) { return "/stuff/" + name }

    var name = Bacon.UI.textFieldValue($("input.name"))

    Bacon.combineTemplate({
      cats: ajax("/cats"),
      dogs: ajax("/dogs",
      stuff: name.map(urlForStuff).ajax()
    }).onValue(doStuff)

Now you'll start capturing user input immediately. Your initial AJAX
calls are executed in parallel and combined with the latest value
returned by the "/stuff/*" AJAX when all values are available.

And the ".ajax()" call actually ensures that the results are based on
latest user input.
