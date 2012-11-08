## Bacon.js Tutorial Part II: Get Started

In my [previous blog posting posting](http://nullzzz.blogspot.fi/2012/11/baconjs-tutorial-part-i-hacking-with.html), I introduced a Registration Form application case study, and then hacked it 
together with just jQuery, with appalling results. Now we shall try again, using [Bacon.js](https://github.com/raimohanska/bacon.js). 
Let's start from the basics.

This is how you implement an app with Bacon.js.

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

In this posting we'll capture the input signals, then define the `username` 
and `fullname` properties. In the end, we'll be able to print the values to the 
console. Not much, but you gotta start somewhere.

## Setup

You can just read the tutorial, or you can try things yourself too. In case you prefer the latter, 
here are the instructions.

First, you should get the code skeleton on your machine.

    git clone git@github.com:raimohanska/bacon-devday-code.git
    cd bacon-devday-code
    git co -t origin/clean-slate

So now you've cloned the source code and switched to the [clean-slate](https://github.com/raimohanska/bacon-devday-code/tree/clean-slate) branch. 
Alternatively you may consider forking the repo first and creating a new branch
if you will.

Anyway, you can now open the `index.html` in your browser to see the registration form.
You may also open the file in your favorite editor and have a brief look. You'll find some 
helper variables and functions for easy access to the DOM elements.

## Capturing input from DOM events

Bacon.js is not a jQuery plugin or dependent on jQuery in any way. However, if it finds
jQuery, it adds a method to the jQuery object prototype. This method is called `asEventStream`,
and it is used to capture events into an EventStream. It's quite easy to use.

To capture the `keyup` events on the username field, you can do

    $("#username input").asEventStream("keyup")

And you'll get an `EventStream` of the jQuery keyup events. Try this in your browser Javascript console:

    $("#username input").asEventStream("keyup").log()

Now the events will be logged into the console, whenever you type something to the username field (try!).
To define the `username` property, we'll transform this stream into a stream of textfield values (strings)
and then convert it into a Property:

    $("#username input").asEventStream("keyup").map(function(event) { return $(event.target).val() }).toProperty("")

To see how this works in practise, just add the `.log()` call to the end and you'll see the results in your console.

What did we just do?

We used the `map` method to transform each event into the current value of the username field. The `map` method
returns another stream that contains mapped values. Actually it's just like the map function 
in [underscore.js](http://underscorejs.org/), but for EventStreams and Properties.

After mapping stream values, we converted the stream into a Property by calling `toProperty("")`. The empty string is the initial value
for the Property, that will be the current value until the first event in the stream. Again, the toProperty method returns a
new Property, and doesn't change the source stream at all. In fact, all methods in Bacon.js
return something, and most have no side-effects. That's what you'd expect from a functional programming library, wouldn't you?

The `username` property is in fact ready for use. Just name it and copy it to the source code:

    username = $("#username input").asEventStream("keyup").map(function(event) { return $(event.target).val() }).toProperty("")

I intentionally omitted "var" at this point to make it easier to play with the property in the browser developer console.

Next we could define `fullname` similarly just by copying and pasting. Shall we?

Nope. We'll refactor to avoid duplication:

    function textFieldValue(textField) {
        function value() { return textField.val() }
        return textField.asEventStream("keyup").map(value).toProperty(value())
    }

    username = textFieldValue($("#username input"))
    fullname = textFieldValue($("#fullname input"))

Better! In fact, there's already a `textFieldValue` function available in [Bacon.UI](https://github.com/raimohanska/Bacon.UI.js/blob/master/Bacon.UI.js),
and it happens to be incluced in the code already so you can just go with

    username = Bacon.UI.textFieldValue($("#username input"))
    fullname = Bacon.UI.textFieldValue($("#fullname input"))

So, there's a helper library out there where I've shoveled some of the things that seems to repeat in different projects.
Feel free to contribute!

Anyway, if you put the code above into your source code file, reload the page in the browser and type

    username.log()

to the developer console, you'll see username changes in the console log.

