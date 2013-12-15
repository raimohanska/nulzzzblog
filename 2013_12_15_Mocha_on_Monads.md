## Mocha on Monads

** Disclaimer: This article can be classified as a Monad Tutorial and therefor considered harmful. Proceed at own risk **

The challenge in testing a browser application with [Mocha](http://visionmedia.github.io/mocha/) is that the application behaves asynchronously because of things like page transitions and AJAX. This means that the test code often has to wait for some condition before continuing. And, as we all know, Javascript doesn't have threads and thus we cannot block when we wait. This means we need to use callbacks.

And because writing asynchronous code is tough, the test writer (me) avoids it and favors writing tests synchronously except when absolutely necessary. And sometimes they write a test synchronously even if the application actually behaves asynchronously. And often they get away with it, because the test passes. Until it doesn't. It may, for instance, pass on Chrome in 99% of the cases. But when you have a hundred test cases, that's not going to be enough.

And when something that used to be synchronous becomes asynchronous, shit will really hit the fan if the tests were sloppy. In my experience this happens quite often. For example, you add an AJAX call somewhere. Bang!

Mocha supports asynchronous testing for sure. You can use callback-style functions in your `before`s and `it`s. When you need to perform a long sequence of (possibly asynchrous) operations, you may use a library such as [zhain](https://github.com/mtkopone/zhain) and you're a bit better off.

But you still can't beat the convenience of normal synchronous Javascript blocks, when it comes to passing any state from one of your async operations to another. You will end up with code looking like this.

```js
  findElement("#loginForm button") (function(button) {
    button.click();
    findElement("#customerId") (function(customerIdElement) {
       var customerId = customerIdElement.text()
       assertEquals(666, customerId)
    })
  })

  function findElement(selector) {
     return function f(done) {
       var element = $(selector)
       if (element.length) {
         done(element)
       } else {
         setTimeout(function() { f(done) }, 100)
       }
     }
  }
```

Not a very complex case yet, yet a Callback Hell is starting to form here.

What if, just what if, you could write it like this instead:

```js
  button <- findElement("#loginForm button")
  button.click()
  customerIdElement <- findElement("#customerId")
  let customerId = customerIdElement.text()
  assertEquals(666, customerId)
```

In this form, the asynchronous calls do not break the flow. You just indicate an asynchronous call with the arrow symbo `<-`. Nice?

Shouldn't be too hard to implement a precompiler that would automatically convert this to the former callbacky form.

## A New Abstraction

Quizz: What is the abstraction that allows the fancy syntax above to be applied not only to callbacks but to arbitrary constructs sharing a couple of common traits? Which programming language supports this syntax and automatically "desugars" it?

In this abstraction, you need a method for chaining things together. Using Haskell syntax, it would look like

```
chain :: M a -> (a -> M b) -> M b
```

But let's call it the `chain` method. And stick to Javascript, with a little sugar. By sugar I mean support for this fancy syntax extension. Not actual sugar.

Anyway, the actual code might look like this:

```js
do {
  button <- findElement("#loginForm button")
  button.click()
  customerIdElement <- findElement("#customerId")
  let customerId = customerIdElement.text()
  assertEquals(666, customerId)
}
```

And this would be automatically preprocessed into

```js
findElement("#loginForm button").chain(function(button) { 
    button.click() 
    return findElement("#customerId").chain(function(customerIdElement) {
      var customerId = customerIdElement.text()
      assertEquals(666, customerId)
    })
  })
```

Which is vanilla Javascript again.

To make this work, the asynchronous operations, like the object returned by `findElement`, need a method named `chain`.

We might implement it like here.

```js
  Async = function(f) {
    if (!(this instanceof Async)) return new Async(f)
    this.run = f
  }
  Async.prototype.chain = function(f) {
    var self = this
    return function(callback) {
      self.run(function(value) {
        var next = f(value)
        next.run(callback)
      })
    }    
  }
```

And the `findElement` method would change to

```js
  function findElement(selector) {
     return Async(function f(done) {
       var element = $(selector)
       if (element.length) {
         done(element)
       } else {
         setTimeout(function() { f(done) }, 100)
       }
     })
  }
```

The only change here is that we wrap the returned function using Async.

Now, given we had this preprocessor, we should be able to use our fancy new syntax with asynchronous operations as long as they are wrapped using `Async`.

## Some composition

Now what if we wanted to write a function `getText` that simply calls `findElement` and returns the element text by calling `element.text()`?

It would look like this.

```js
function getText(selector) {
  return Async(function(done) {
    findElement(selector)(function(element) {
      done(element.text())
    })
  })
}
```

Ugly huh? Fancy syntax to the resque:

```js
function getText(selector) {
  return do Async {
    element <- findElement(selector)
    return element.getText()
  }
}
```

The `do Async` part here is to tell the preprocessor to use namely the `Async` abstraction. And `return` is not your typical Javascript return. In this syntax it is used to wrap a simple expression into an Async operation. This function would be converted to the following piece of Javascript.

```js
function getText(selector) {
  return findElement(selector).chain(function(element) {
    return Async.of(element.getText())
  })
}
```

So the preprocessor would convert `return x` into `Async.of(x)`. This means we have to implement an `of` method into `Async`. Like here.

```js
Async.of = Async.prototype.of = function(value) {
  return new Async(function(callback) {
    callback(value)
  })
}
```

## Conclusion

As you probably know, what I described above is a Javascripty version of Haskell's [do-notation](http://en.wikibooks.org/wiki/Haskell/do_Notation) for Monads. The process of conververting do-notation into calls to the Monad methods is called *desugaring*.

But simply put, Monads are just things that support `of` and `chain`. These functions have different names in different environments, but here I've chosen to use the names used in the [fantasy-land](https://github.com/fantasyland/fantasy-land) specification. TODO: links to more info on monads

Had we this notation in Javascript, we could use is for surprisingly many things, including [Promises](https://github.com/fantasyland/fantasy-promises) and [Options](https://github.com/fantasyland/fantasy-options). And the notation is not the only benefit of the Monads; there's a lot more you can build on top of this common interface.

For the record, the [Roy](http://roy.brianmckenna.org/) language supports a very similar do-notation as described above. The thing with Roy is though that it is very remote to base Javascript and is not an easy replacement as-is. But on the Roy website, you can play with the do-notation (Monad) examples and see how it desugars the code.

The `Async` Monad here is perhaps more widely known as the [Continuation Monad](http://hackage.haskell.org/package/mtl-1.1.0.2/docs/Control-Monad-Cont.html). For the intended purpose (asynchronous testing in Javascript) a bit better suited Monad would be one that allows error propagation too, possibly using [Node-style callbacks](http://howtonode.org/control-flow-part-ii).
