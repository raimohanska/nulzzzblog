## Context

Me and my friends been doing functional testing on our rather large single-page app,
using Mocha and JQuery. JQuery is used to drive the system under test (SUT), usually started in an IFrame. JQuery is also used to verify that the application behaves as expected. For instance, in a todo-application, you might

    S("#addTodo").click()
    expect(S("#tasks").length).to.equal(1)

Here S is a helper function that delegates to the JQuery instance in the SUT window. So the basic pattern is that you perform certain operations on the SUT and then verify that the application ends up in the expected end-state. In mocha, you might do this like

    describe("Add todo button", function() {
        before(function() {
          S("#addTodo").click()      
        })
        
        it("Adds todo", function() {
          expect(S("#todos").length).to.equal(1)
        })
    })

If the application is synchronous this works just fine.

## Problem

You might have guessed this. The minute you add your first asynchronous thing in the application, you enter the world of uncertain test results. That is, if your tests assume that after doing X, the application goes from state A to B. To get past this, you may use asynchronous functions in your `before` blocks. Like

    before(function(done) {
      wait.until(function() { return S("#todos").length == 2}, done)
    })

This will fix a single test case. Everytime you find that some test is unreliable, you add some kind of an ad-hoc asynchoronous wait. Now step to a situation where you have a hundred tests and you change your application so that something that used to be synchronous is now asynchronous. You'll get some failing tests. You may get tests that fail on some browsers and only sometimes. Horror!

Adding async waits to individual test setup steps doesn't scale. Been
there, done that.

## Solution

I strongly believe that we need a more systematic solution here. The
developer shouldn't need to consider asyncronicity at each test step.

Why not go for a solution where after each `before` block and before
each `it`, there should be an implicit async wait that would ensure that
before the test proceeds, the framework ensures that all AJAX calls,
animations, transitions, page loads and whatnot have finished.

Is this hard? Shouldn't be. Just monkey-patch `before` and `it` to
include these steps. Gonna try this and write more afterwards. What do
you think?
