Have you ever found out that there's a broken image on your web application? 
Maybe on a page that is seldom visited by developers. You've got great mocha
tests for your UI features, but none of these can catch broken images? I was
in this situation a while ago. No more!

Now I can write tests like this:

```js
  describe("The front page", function() {
    it ('Shows a cool image', function() {
      expect(page.find("img.cool")).to.be.visible()
    })

    checkAllImages()
  })
```

The `checkAllImages` call there generates a test case that check that all `<img>` tags are successfully loaded. 
It also verifies my CSS backgrounds!

TODO
