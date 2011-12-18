Shake Your Rump
---------------

We've been working on a mobile (Android/iOS) app called Karma for some
time. The app will answer the ever-crucial question "who's turn is it to
buy the next round?". To make it fun and easy to use, it needs a cool
way of telling the system that two guys, say Bob and Mike, are having
drinks together. Thanks to location and acceleration info, it's possible
to make this pairing happen just by shaking phones together. So, in
Karma it goes like this:

- Bob and Mike meet in a bar
- Bob and Mike fire up the Karma app in their phones
- They shake their phone simulaneously
- Karma tells it's Bobs turn to buy!
- Bob buys a round and presses the "round bought" button
- Karma tells it's Mike's turn to buy
- Bob and Mike keep on drinking and having fun
- Jack joins the table and shakes his phone with Mike
- Karma tells all that it's now Jack's turn
- Serious drinking takes place

On the background Karma records all rounds bought and calculates the
next buyer based on this info and maybe some other variables.

But hey, the topic of this post is really the "shake to pair" technology
included. We started by trying the [Bump](http://bu.mp/) API. However,
we found it quite unreliable, even though the actual Bump application
seemed solid. I guess they don't give their best performance for API
users?

Now, what would a proper nerd do? What the heck, let's write it from
scratch. And POW, here it is: 

- [RUMP Server](https://github.com/raimohanska/rump) 
- [RUMP Android Client](https://github.com/raimohanska/rump-android)

As you see, it's open-source. And, it's actually very simple:

1. When user shakes the phone, the RUMP Client sends a message to the
   server. Like this:

      { "userId" : "john", "displayName" : "John Kennedy", "location": {
"latitude": 51.0, "longitude": -0.1}}

2. Server matches incoming requests by time (3 second time window to
   match) and location (1 kilometer max distance).

3. After 3 seconds from the first received request, the server sends
   match info to all clients that were matched with the first one. The
   response is just an array of all the incoming requests.

4. Rump Client delivers the match info to the client application.

5. Client application does something cool. For instance, the Karma
   application joins the matched users into the same table and
   calculates the next buyer.


