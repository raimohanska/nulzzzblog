1. Capture input into EventStreams and Properties
2. Transform and compose signals into ones that describe your domain.
3. Assign side-effects to signals

In practise, you'll probably pick a single feature and do steps 1..3 for
that. Then pick the next feature and so on until you're done. Hopefully
you'll do some refactoring on the way to keep your code clean.

Sometimes it may help to draw the thing on paper. For our case study,
I've done that for you:

![signals](https://raw.github.com/raimohanska/bacon-devday-slides/master/images/registration-form-bacon.png)

In this diagram, the greenish boxes are EventStreams and the gray boxes
are Properties. The top three boxes represent the raw input signals:

* Key-up events on the two text fields
* Clicks on the register button

From these signals we make the `username` and `fullname` properties.
