## Garden Variety IoT with Big Data and FRP

Lately, I've been turning my home into an Internet of Things (IoT) lab.

My livingroom white leds enhance natural light based on an algorithm that depends on how light it is outside. Other lights turn on automatically when the night gets dark if I'm home and forgot to turn them on myself (this happens often). The air humidifier in the bedroom is regulated with an external humidity sensor and an algorithm that keeps the humidity between healthy limits. 

I've been telling myself – and my spouse – that I'm doing this to make living easier. However, I guess I could admit that I do it mostly just because it's possible and the tweaking itself is fun. What is cool and worth writing about is how nicely FRP (functional reactive programming) is suited for home automation.

For instance, yesterday I put the fountain in my garden under automatic control. It will only run when someone’s home, it is daytime and, obviously, as we live in Finland, when it isn't freezing outside.

![fountain](images/fountain.jpg)

I integrated the fountain pump into my home automation system this with this piece of code (I'll explain it later):

```coffeescript
outdoorTempP = sensors.sensorP({type:"temperature", location: "outdoor"})
dayTimeP = time.hourOfDayP.map((hours) -> hours >= 7 && hours <= 22)
freezingP = outdoorTempP.map((t) -> t < 0)
someoneHomeP = motion.occupiedP("livingroom", time.oneHour * 8)
fountainP = dayTimeP.and(freezingP.not()).and(someoneHomeP)
houm.controlLight "fountain", fountainP
````

This is possible because I’ve already installed some sensors here and there around my house, measuring things like temperature, 
humidity and lightness. Some sensors came straight from the store, many have I've soldered together from stuff like ESP-8266 WiFi microcontrollers, Raspberry Pis and numerous sensor modules. I’ve even designed a printed circuit board that has motion detection and a bunch of measurements and ordered the manufacturing of 10 units from China. It looks like this:

![raimo-unit](images/raimo-unit.jpg)

Also, I have the [Huom.io](http://houm.io/en/) lighting control system set up. It allows turning 
lights and, in fact, any electric appliances on and off using a simple [API](https://github.com/houmio/houmio-docs/blob/master/apidoc.md).

Because I’m an FRP nerd and happen to have built an [FRP library](https://github.com/baconjs/bacon.js/) of my own a few years ago,
I want to do my automation by combining streams of data using FRP operators like `map`, `flatMap` and `combine`. For this, I wrote a simple [server platform](https://github.com/raimohanska/sensor-server) that allows gathering the data from the sensors and lighting system and then piping and combining it to control the house lighting. And, of course, the fountain.

### Introduction to FRP IoT: Combining Properties

For me, home automation and IoT are about collecting streams of measurement values, storing them for later use and visualization as well as transforming and combining data into control streams that can then be fed to actuators, such as lighting, pumps and valves. FRP with a library like Bacon.js seems like the perfect fit.

In Bacon.js, we use `EventStreams` to represent distinct events and `Properties` to represent values that change over time. For instance, in my home automation platform, there's an API called `sensors` that will give any measured value as a Property. So, when I write

```coffeescript
outdoorTempP = sensors.sensorP({type:"temperature", location: "outdoor"})
outdoorTempP.forEach((t) -> console.log("temperature is " + t)
````

...my function on line 2 will be called when the outside temperature property `outdoorTempP` changes and the temperature will be written to standard output. Not very useful yet, but let's add more stuff.

```coffeescript
freezingP = outdoorTempP.map((t) -> t < 0)
dayTimeP = time.hourOfDayP.map((hours) -> hours >= 7 && hours <= 22)
````

Here I've used the `map` method of `outdoorTempP` to transform the temperature values into boolean values so that the new property `freezingP` will hold `true` when temperature outside is freezing – I'm obviously using Celsius degrees here. Then I added a new property `dayTimeP` using a similar `map` call on the `hourOfDayP` property of the `time` API.

Finally, I add one more property from the `motion` API and combine all of the data using boolean logic:

```coffeescript
someoneHomeP = motion.occupiedP("livingroom", time.oneHour * 8)
fountainP = dayTimeP.and(freezingP.not()).and(someoneHomeP)
```

So the final `fountainP` property will hold `true` when it's daytime, not freezing and someone's home. The last line that uses the `and` and `not` operators could also be written with the more generic `combineWith` operator that let's you combine the latest values of a bunch of Properties using a function. Like this:

```coffeescript
fountainP = Bacon.combineWith(dayTimeP, freezingP, someoneHomeP, (daytime, freezing, atHome) ->
  daytime && atHome && !freezing
```

Admittedly, the "someone home" property is not very accurate, as it's based on whether there's been motion in the livingroom in the last 8 hours. I'll add an outdoor motion sensor later for more accuracy, but this will do for now. It's not fatal to have a fountain running, even when no one is home. But at least it won't be running when I'm on a two-week vacation in Africa. 

During which the home automation system will, by the way, give the impression of an occupied house by turning lights on and off every now and then.

Anyways, now that I've defined when the fountain should be running, it can be made to obey my will by using my reactive HOUM.IO API wrapper:

```coffeescript
houm.controlLight "fountain", fountainP
````

### The Time Axis

FRP is not just great for combining the current values of a bunch of Properties. It also really shines when you have to deal with time, like

  *"Turn off lights if there's no activity in 2 hours"*
  
You can express this with this code

```coffeescript
  # motionP holds 1 when there's motion and 0 when there's no motion
  motionP = sensors.sensorP({type: "motion", location: "livingroom"})
  # motionE emits an event when motionP changes from 0 to 1
  motionE = motionP.changes().filter((m) -> m == 1)
  # inactiveE emits an event after 2 hours have passed since last detected motion
  inactiveE = motionE.debounce(time.oneHour * 2)
  # sets livingroom brightness to zero when inactiveE emits an event
  inactiveE.forEach(-> houm.setLight("livingroom")(0))
```

In fact, libraries like Bacon.js and RxJs are packed full of operators that you can do to achieve any kind of effect on the time axis with declarative, functional code. And that's the kind of code I like.

### My IoT Platform

My [Homegrown Reactive IoT Platform](https://github.com/raimohanska/sensor-server) currently has these API's:

- `houm` for controlling lighting and electric appliances
- `time` for time-of-day related EventStreams and Properties
- `sun` for sunrise/sunset information and sun brightness in my (or your) location
- `sensors` for sensor input data (temperature, humidity, etc). Data is fed through TCP and HTTP endpoints using JSON
- `motion` motion sensor data, room occupied indication with throttling. Built on the `sensors` API

The platform is not quite documented yet, but I can write some docs if there's interest. Please add a Star and create an Issue if you're interested in more information!
