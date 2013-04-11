## Functor

The `map` method of EventStream makes it a Functor.

    map :: EventStream a -> (a -> b) -> EventStream b

The `map` method of Property makes it a Functor, too.

    map :: Property a -> (a -> b) -> Property b

## Monad

The `flatMap` method of EventStream, in combination with `Bacon.once` makes it a Monad.

    flatMap :: EventStream a -> (a -> EventStream b) -> EventStream b
    once :: a -> EventStream a

More precisely, `flatMap` turns any `Observable` into an `EventStream`:

    flatMap :: Observable a -> (a -> Observable b) -> EventStream b

Which implies that Property isn't strictly a Monad. This doesn't
practically limit it's use as if it was a Monad. It quacks like a Monad,
you know. The "return" for Properties would be `Bacon.constant`.

## Applicative Functor

The `combine` method in combination with `Bacon.constant` makes it an
Applicative Functor.

    combine :: Property a -> Property b -> (a -> b -> c) -> Property c
    constant :: a -> Property a

I was tempted to say that the `combine` method in combination with `Bacon.once` makes it an
Applicative Functor, but that would be inaccurate, because `combine`
returns a Property. Again, that won't limit the practical use of
EventStreams as Applicative Functors, will it?

## More

Applicative Functors can be used to lift an n-ary function on values of
type `a` into a function of Properties of type `a`. There's a shorthand
for that in Bacon, namely `Bacon.combineWith`. Here's an example:

    function sum3(x,y,z) { return x + y + z }
    Bacon.combineWith(sum3, p1, p2, p3)

In fact, it's not limited to Properties. You can pass in any combination
of Properties, EventStreams and constant values.


