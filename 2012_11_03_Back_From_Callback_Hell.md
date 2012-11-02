Back from Callback Hell with Bacon.js
=====================================

I've heard the words Callback Hell quite a many times lately. I'll take
it that you know what I'm talking about. Just gonna briefly show how to
get Back from Hell with [bacon.js](https://github.com/raimohanska/bacon.js)...

So, suppose you need to get some things with AJAX and then do
some stuff. Now, you might do

    $("input.name").onKeyUp(function() {
      $.ajax("/cats/" + $("input.name").val()).done(function(cats) {
        $.ajax("/dogs/" + cats).done(function(dogs) {
          $.ajax("/stuff/" + dogs).done(function(stuff) {
            doStuff(stuff)
          })
        })
      })
    })

Nice? Well, you could do it also like

    Bacon.UI.textFieldValue($("input.name"))
      .map(function(name) { return "/cats/" + name }).ajax()
      .map(function(cats) { return "/dogs/" + cats }).ajax()
      .map(function(dogs) { return "/stuff" + dogs }).ajax()
      .onValue(doStuff)

Any better? At least we got rid of the arrowhead. You could easily
refactor that into

    function url(thing, id) { return "/" + thing + "/" + id }
      
    Bacon.UI.textFieldValue($("input.name"))
      .map(url, "cats").ajax()
      .map(url, "dogs").ajax()
      .map(url, "stuff").ajax()
      .onValue(doStuff)

Now, what about if you
want to perform two independent AJAX calls and do something with the
values of both?

    $.ajax("/cats").done(function(cats) {
      $.ajax("/dogs").done(function(dogs) {
        doStuffWithCatsAndDogs(cats, dogs)
      })
    })

A minor hell there. How about

    var cats = Bacon.fromPromise($.ajax("/cats")
    var dogs = Bacon.fromPromise($.ajax("/dogs")
    Bacon.combineAsArray(cats, dogs).onValues(doStuffWithCatsAndDogs)

Know what? Your AJAX just got parallel, too. Maybe we should refactor
this too:

    function ajax(url) { return Bacon.fromPromise($.ajax(url)) }
    Bacon.combineAsArray(ajax("/cats"),
ajax("/dogs")).onValues(doStuffWithCatsAndDogs)

That's it.
