How to organize large application? That seems to be a topic not very well covered in the internets and rewards a blog post or two. Maybe my next post will be on this. Anyway, what I personally do is something like this.

As with any other tecnology, I split my application into independent components with a single responsibility. I use EventStreams and Properties to connect the components together. This comes quite naturally when they are also internally composed from the same. For example, a ShoppingCart model component might look like this.

```javascript
function ShoppingCart(initialContents) {
  var addBus = new Bacon.Bus()
  var removeBus = new Bacon.Bus()
  var contentsProperty = Bacon.update(initialContents,
    addBus, function(contents, newItem) { return contents.concat(newItem },
    removeBus, function(contents, removedItem { /* omitted */ }
  return {
    addBus: addBus,
    removeBus: removeBus,
    contentsProperty: contentsProperty
  }    
}
```

Now the external interface of this component exposes the `addBus` and `removeBus` Buses where you can plug external streams for adding and removing items. It also exposes the current contents of the cart as a `Property`.

Now you may define a view component that shows cart contents:

```javascript
function ShoppingCartView(contentsProperty) {
  function updateContentView(newContents) { /* omitted */ }
  contentsProperty.onValue(updateContentView)
}
```

And a component that can be used for adding stuff to your cart:

```javascript
function NewItemView() {
  var newItemProperty // property containing the item being added
  var newItemClick // clicks on the "add to cart" button
  var newItemStream = newItemProperty.sampledBy(newItemClick)
  return {
    newItemStream: newItemStream
  }
}
```

And you can plug these guys together as follows.

```javascript
var cart = ShoppingCart([])
var cartView = ShoppingCartView(cart.contentsProperty)
var newItemView = NewItemView()
cart.addBus.plug(newItemView.newItemStream)
```

I hope this helps!
