Zipping With Arrays in RxJs
===========================
Have you ever tried to zip a hot observable with a cold one? That's what happens below.

~~~ {.javascript}
   var former = function(a, b) { return a }
   var latter = function(a, b) { return b }
   var logger = function(a) { console.log(a) }
   var numbers = Rx.Observable.FromArray([1, 2, 3])
   var timer = Rx.Observable.Interval(100)
   var zipped = numbers.Zip(timer, former)
   zipped.Subscribe(logger)
~~~

The idea is to print out the contents of an array to the console one-by-one, each 100 milliseconds. It doesn't work though. What I've heard is that it fails because of a bug in Rx. Instead of printing out numbers 1,2,3 to the console, the code does ... nothing.

A simple workaround (Thanks [Jussi](http://twitter.com/#!/jvesala)) seems to work, though: Just concat your array observable with `Rx.Observable.Never`!

~~~ {.javascript}
   // ...
   numbers.Concat(Rx.Observable.Never).Zip(timer, former).Subscribe(logger)

TypeError
1
2
3

~~~

So, now it prints the stuff to the console, but also produces some TypeError. What is that?
