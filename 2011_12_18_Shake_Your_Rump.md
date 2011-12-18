Shake Your Rump
---------------

We've been working on a mobile (Android/iOS) app called Karma for some
time. The app will answer the ever-crucial question "who's turn is it to
buy the next round?". To make it fun and easy to use, it needed an easy
way of telling the system that two guys, say Bob and Mike, have just met 
for some beers. Thanks to location and acceleration info, it's possible
to make this pairing happen just by shaking phones together.

The topic of this post is the "shake to pair" technology
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

Well, that's it. On the Github pages, you'll find instructions for building, 
running and integrating the Android Client into your app.

I think this would provide a nice basis for matching users in a multiplayer
game, for instance. Shake your phones to start multi-player, anyone?

What else? We have an iPhone version of Karma in the works. Mayhaps we'll also
extract the Rump client part from there and open-source that too? Would it make
sense?

The server-side piece is written in Haskell. I'm on a horse.