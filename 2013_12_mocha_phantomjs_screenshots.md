Today I wondered whether you can take screenshots in your [mocha-phantomjs](https://github.com/metaskills/mocha-phantomjs) tests.
Within 15 minutes I found out that my colleagues at Reaktor had already solved the problem in their own fork. So I took a look and
with some help got screenshots working in the [open-source application](https://github.com/Opetushallitus/omatsivut) I'm working on 
for the Board of Education in Finland. Then I made a [pull request](https://github.com/metaskills/mocha-phantomjs/pull/165) and now I'm
telling you how to make screenshots work.

A couple of days later, version 3.5.2 with screenshot support was released. So, first make sure you're using 3.5.2 or later:

````
  ...
    "mocha-phantomjs": "^3.5.2",
    "phantomjs": "^1.9",
  ...
```

Then, in your test code, write a helper for generating a screenshot on demand:

```javascript
function takeScreenshot() {
  if (window.callPhantom) {
    var date = new Date()
    var filename = "target/screenshots/" + date.getTime()
    console.log("Taking screenshot " + filename)
    callPhantom({'screenshot': filename})
  }
}
```

In this function you decide where to put the screenshots and how to name them. In my case, I'm putting them under `target/screenshots`.

If you want to generate a screenshot for each test failure you just add the following into your test code.

```javascript
  afterEach(function () {
    if (this.currentTest.state == 'failed') {
      takeScreenshot()
    }
  })
```

And next time you run your `mocha-phantomjs` tests, KABOOM, you have screenshots. Good times!
