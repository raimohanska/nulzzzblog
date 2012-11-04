This is the first part of a hopefully upcoming series of postings
intended as a [Bacon.js](https://github.com/raimohanska/bacon.js)
tutorial. I'll be building a fully functional, however simplified, AJAX
registration form for an imaginary web site. Something like this.

![ui-sketch](https://raw.github.com/raimohanska/bacon-devday-slides/master/images/registration-form-ui.png)

This seems ridiculously simple, right? As in

          registerButton.click(function(event) {
            event.preventDefault()
            var data = { username: usernameField.val(), fullname: fullnameField.val()}
            $.ajax({
              type: "post",
              url: "/register",
              data: JSON.stringify(data)
            })
          })

At first it might seem so, 
but if you're planning on implementing a top-notch form, you'll want 
to consider including

1. Username availability checking while the user is still typing the username
2. Showing feedback on unavailable username
3. Showing an AJAX indicator while this check is being performed
4. Disabling the Register button until both username and fullname have been entered
5. Disabling the Register button in case the username is unavailable
6. Disabling the Register button while the check is being performed
7. Disabling the Register button immediately when pressed to prevent double-submit
8. Showing an AJAX indicator while registration is being processed
9. Showing feedback after registration

Some requirements, huh? Still, all of these sound quite reasonable, at least to me.
I'd even say that this is quite standard stuff nowadays. You might now model the UI like this:

![dependencies](https://raw.github.com/raimohanska/bacon-devday-slides/master/images/registration-form-thorough.png)

Now you see that, for instance, enabling/disabling the Register button depends on quite a many different things, some
of them asynchronous. But hey, fuck the shit. Let's just hack it together now, right? Some jQuery and we're done in a while.