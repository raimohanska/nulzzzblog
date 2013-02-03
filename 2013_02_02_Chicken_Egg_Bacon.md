# Chicken, Egg and Bacon.js

FRP is easy when you can define your "event network" first and then just
watch it make profit. For instance, you may define EventStreams and
Properties based on DOM events, compose them, do some AJAX and show the
result in the UI, just like we did in the Bacon.js Tutorial parts II & III.

Sooner or later, though, you'll run into the "chicken and egg" problem:
you need to base your streams on something that will change on the fly.
So, you cannot just define your streams based on existing DOM objects,
because the DOM will change.

For example, if you want to implement the quite well-known [TodoMVC](http://addyosmani.github.com/todomvc/)
application with Bacon.js, you'll have to solve this problem. I
suggest you take a look at TodoMVC yourself, but here's a brief
description of the problem. Let's start with a screenshot.

![TodoMVC](https://raw.github.com/addyosmani/todomvc/gh-pages/screenshot.png)

The TodoMVC application is a simple task manager; you add new tasks,
mark them complete, edit them and delete them. You have the All, Active
and Completed views. Stuff like that.

Assuming you've read the previous log posts and done some Bacon.js
coding, you'd hack most of the app together in a couple of hours, save
for the tough part. The tough part is the dynamic view,
say `TodoListView`, which lists the `Todos` that match the current filter
(All/Active/Completed). In fact, the only tough part of the view is that
the view also allows you to edit and delete the tasks.

Now let's start with the assumption that we want to define a Property,
say `todosProperty` that captures the whole data model of the
application. The value of this Property will be an Array of Todos, i.e.
something like this:

    [ 
      { id: 1, title: "Buy Bacon", completed: false },
      { id: 2, title: "Buy Eggs", completed: false }
    ]

This model would make it very easy to re-render the list whenever the
property changes. Also it would be easy to show only completed or active
task by applying a filter:

```javascript
var activeTodos = todosProperty
  .map(function(todos) { return _.where({ completed: false })})
```

The number of active tasks could be derived
like this:

    var activeTaskCount = activeTodos.map(".length")

To simplify a bit, the `todosProperty` itself would now depend on item 
additions only, so we could define the property in a very FRP'ish way:

```javascript
var newTodos; // stream of new Todo items
var todosProperty = addedItems.scan([], function(todos, todo) {
  return todos.concat([todo])
})
```

In real life, there are more factors involved, but let's stick to this
assumption for now.

## If the list was not editable

If the list items were not editable, it would be easy to render the list
based on the `todosProperty`:

```javascript
var todosProperty; // a Property containing an Array of Todos
var listElement; // jQuery element
todosProperty.onValue(function(todos) {
  listElement.children().remove()
  _.each(todos, function(todo) {
    var todoElement = renderTodoView(todo)
    listElement.append(todoElement)
  })
})
```

We just redraw the list each time *anything* changes. The
list items are not editable so we don't have to capture any data from
them. Just render and forget.

## If the view was static

On the other hand, if view was static, i.e. the set of
displayed items was not changing, things would be quite easy too. 
Now you could capture the edits for the Todo items with something like this:

```javascript
var todos; // list of todos
var listElement; // jQuery element
var properties = _.map(todos, function(todo) {
  // call renderer function, returns a jQuery element
  var todoElement = renderTodoView(todo) 
  listElement.append(todoElement)
  // Use Bacon.UI helpers to create Properties for the "title" and "completed" fields
  return Bacon.combineTemplate({
    title: Bacon.UI.textFieldValue(todoElement.find(".edit"), todo.title)
    completed: Bacon.UI.checkBoxValue(todoElement.find(".toggle", todo.completed)
  })
})
// now we convert this list of Properties to a single property
// each value of which is an Array of Properties
var todosProperty = Bacon.combineTemplate(properties)
```

Nice and easy. Render Todos once, collect editor values into a single
Property using a couple of `combineTemplate` calls and we've re-defined
the `todosProperty`.

## But it's dynamic and editable

So it happens to be the case that the value of `todosProperty` depends
on user's interaction with the DOM elements corresponding each Todo item
on the list. And these DOM elements will be added and removed based on
changes to the `todosProperty`. Classic chicken-egg.

The point of this blog post is to present some of the known solutions to
this problem.

## Event delegation (a.k.a live bindings)

One way around that chicken-egg problem with dynamically-created views is to use event delegation; 
bind event handlers to a containing element, and use a specific selector as the second argument to asEventStream.
In TodoMVC, we could try something like

```javascript
var itemEdits = listElement.asEventStream("keyup", ".edit")
```

This would give us all keyup events from child elements of
`listElement`, matching the `.edit` selector. Practically we'd get
events when the user edits any of the Todo titles on the list.

Now we could re-define `todosProperty` to take these events into
account. But before that we'd have to take care of something: we need to
include some identifier into the events in order to know which Todo was
actually edited. One way to do this would be to include a `data-todo-id`
attribute to the text input element. This wouldn't be a big deal
especially if we used a templating lib like Handlebars.

Assuming the text input element had this data attribute, we could
continue like

```javascript
var itemEdits = listElement.asEventStream("keyup", ".edit")
  .map(function(event) {
    var textField = $(event.target)
    return {
      id: textField.attr("data-todo-id"),
      newTitle: textField.val()
    }
  })
```

Which would give us a stream of all item edits. For example

    { id: 1, newTitle: "Buy a lot of bacon" }

Re-implementing the `todosProperty` could now be done in a number of
different ways. For some reason, I like to model this by constructing a
number of streams containing *modifications* to the list, where the
modifications are actually *functions* that take the previous list of
Todos and return the modified list. So we might say

```javascript
var itemModifications = itemEdits.map(function(edit) {
  return function(todos) {
    return _.map(todos, function(todo) {
      if (todo.id == edit.id) {
        return _.extend(_.clone(todo), { title: edit.newTitle})
      } else {
        return todo
      }
    })
  }
})
```

This converts the stream of edits into a stream of functions that will
transform the list of todos by changing the title of a single todo. We'd 
also have to convert the stream of new Todos into this form, but
I'll leave that as an excercise for you. The `todosProperty` would now
look like

```javascript
var todosProperty = itemModifications.merge(itemAdditions)
  .scan([], function(todos, modification) {
    return modification(todos)
  })
```

So, for some cases, we can come by the chicken-egg problem by using
jQuery smartly. But at least I find this a bit inelegant and not very
scalable; speaking in MVC terms, you have to create your Views before
you create your Model because the latter depends on the former.

## Apply some MVC

At this point I'm struggling not to bake in some smart-ass analogy involving chickens,
eggs and bacon.
And speaking of MVC, there's another way to get by the chicken-egg problem.

So far we've kept trying to
define `todoProperty` in the traditional FRP way, which is, by composing
from a fixed set of source streams. This is elegant to some point. But
sometimes it may be more practical and arguably more elegant to make
a switch from pull to push. And use `Bacon.Bus`.

Instead of defining all of the event sources at once, we can introduce a
Bus, which allows us to explicitly `push()` events to the stream. It
also allows us to `plug()` in streams after the creation of `todosProperty`.

So let's take a step towards a more traditional MVC architecture and use a 
stateful model object to represent the list of Todos. 
The UI can tell this model to change using methods like `addTodo`, 
`removeTodo` and `modifyTodo`. 

We'll use a variable, `todos` in the list model to hold the current list of Todos. 

```javascript
function TodoListModel() {
  var todos = []
  var changes = new Bacon.Bus()
  this.todoProperty = changes.toProperty(todos)

  function update(modification) {
    todos = modification(todos)
    this.changes.push(todos)
  }

  this.addTodo = function(newTodo) {
    update(function(todos) {
      return todos.concat([newTodo])
    })
  }
  this.modifyTodo = function(updatedTodo) {
    update(function(todos) { 
      return _.map(todos, function(todo) {
        return todo.id === updatedTodo.id ? updatedTodo : todo
      })
    })
  }
}
```

This model object now exposes the field `todoProperty` which can be used
like in the earlier examples. You might notice that I'm using the exact
same functions for the actual list modifications too. The idea is to
expose the required set of imperative-style mutator functions that will
modify the model's state and push the new list of Todos to the Bus
object `changes`. The exposed `todoProperty` now reflects the current
state of the model, which is the latest list pushed to `changes`.

Now it's easy to derive Properties like `activeTodos`, `completedTodos`,
`activeTodoCount` etc in the FRP way.

To summarize, we use imperative style for changing state, but expose the
state as FRP EventStreams and Properties. One might say that we gain the
best of both worlds.

This approach may get easier by using, for instance, an actual Backbone
model and expose its state as EventStreams and Properties, as in the
[Backbone-Bacon TodoMVC list model](https://github.com/pyykkis/todomvc/blob/bacon-backbone-require/labs/dependency-examples/bacon_backbone_require/src/models/todo_list.coffee) by Jarno Keskikangas. He's even put a [Backbone.EventStreams](https://github.com/pyykkis/Backbone.EventStreams)
library on Github. In his own words,

> Bacon gives superpowers to Backbone controllers.

Amen.

## One step back towards FRP

If you feel uncomfortable with variables (I do), you might want to
revise the MVC solution above. Instead of using a variable, we'll use
`scan`:

```javascript
function TodoListModel() {
  // A Bus of modification functions
  var modifications = new Bacon.Bus()
  this.todoProperty = modifications.scan([], function(todos, modification) {
    return modification(todos)
  })

  function update(modification) {
    this.modifications.push(modification)
  }

  // no changes to the rest...
}
```

So that's the same `scan` approach as in the pure FRP solution.

## Yet another step towards FRP

This is the way I actually implemented TodoMVC. And submitted a Pull
Request too.

Instead of exposing imperative mutators in the model, you can expose
corresponding Bus objects. This allows you to plug-in source streams to
the model. The model would look like this:

```javascript
function TodoListModel() {
  function mapTodos(f) { return function (todos) { return _.map(todos, f) } }

  function modifyTodo(updatedTodo) {
    return mapTodos(function (todo) {
      return todo.id === updatedTodo.id ? updatedTodo : todo
    })
  }
  function addTodo(newTodo) {
    return function (todos) {
      return todos.concat([newTodo])
    }
  }

  this.todoAdded = new Bacon.Bus()
  this.todoModified = new Bacon.Bus()

  var modifications = this.todoAdded.map(addTodo)
                      .merge(this.todoModified.map(modifyTodo))

  this.todoProperty = modifications.scan([], function (todos, modification) {
    return modification(todos)
  })
}
```

It the View code you'll plug-in streams to the exposed Buses `todoAdded`
and `todoModified`. For instance, this is how you might implement a
simplified view responsible of rendering a single Todo row.

```javascript
function TodoView(todo, listModel) {
  this.element = render(todo) // use Handlebars or whatnot
  var titleChanges = this.element.find(".edit").asEventStream("keyup")
    .map(".target").map($).map(".val")
  var modifications = titleChanges.map(function(title) {
    return { id: todo.id, title: title, completed: todo.completed }
  })
  listModel.todoModified.plug(modifications)
}
```

One advantage of this approach, compared to the previous one with
the imperative mutators is that when you expose the buses `todoAdded`
and `todoModified` you're also exposing the same things as streams. So
now your View components can subscribe to these streams to react to Todo
additions and modifications!

There's a little catch here too: you should make sure you apply an "end
condition" to the input streams you plug into the model Buses. If you don't do
this, the Bus will have references to the streams even after the
corresponding UI components have been removed. In practise, you'll
probably remove the UI components based on an event in some EventStream,
so you can use this same stream for ending the input streams. For
example, in `TodoView` you might have a `deleted` stream that signals
the deletion of this particular Todo. Just add `.takeUntil(deleted)` to
your modifications stream and it will end on deletion:

```javascript
function TodoView(todo, listModel) {
  // .. begins as before ..
  var deleted = listModel.todoProperty.changes.filter(function(todos) {
    return _.where({ id: todo.id}).length == 0
  })
  deleted.onValue(function() { this.element.remove() })
  listModel.todoModified.plug(modifications.takeUntil(deleted))
}
```

You may have a look at the full [Bacon.js TodoMVC implementation](https://github.com/raimohanska/todomvc/blob/bacon-jquery/labs/architecture-examples/baconjs/js/app.js)
for reference. I'm sorry for the tab indentation. That's the standard for TodoMVC.
Also, don't expect the implementation to be *exactly* the same as in any
of the above examples. But it definitely is based on a TodoListModel
that exposes Buses and streams to the Views.

# Conclusion

We've actually covered two related problems in the pure FRP approach,
both of which stem from having to introduce some Views before the Model
because the Model depends on the Views.

1. The Chicken-Egg Problem - your Model depends on your Views which
   depend on the Model
2. The Coupling Problem - you have to Introduce some View components
   before the Model, so you cannot properly modularize your application

The first problem in isolation can be in some cases solved by using
techniques like jQuery event delegation. But to address the problem in
the bigger picture, you need to apply architectural solutions, such as
introducing a Model object with either mutator functions or pluggable
Bus objects.
