# Bacon.update for TodoMVC

In the [previous
post](http://baconjs.blogspot.fi/2013/02/chicken-egg-and-baconjs.html) I described how a TodoListModel can be implemented in Bacon.js by creating an EventStream of changes to the list of Todos. Each event in this stream is a function that changes the state of the list model. For instance, when a new Todo is added, a function like this would appear in the modification stream:

```js
addTodo = function(newTodo) {
  update(function(todos) {
    return todos.concat([newTodo])
  })
}

This still all fine. Except that with `Bacon.update` you can do the same
thing with much less code that's easier to read. So, here's the original
`TodoListModel` from my first TodoMVC implementation:

```js
function TodoListModel() {
  function mapTodos(f) { return function (todos) { return _.map(todos, f); }; }
  function filterTodos(f) { return function (todos) { return _.filter(todos, f); }; }

  function toggleAll(toggle) { return mapTodos(function (todo) { return _.extend(_.clone(todo), { completed: toggle }); }); }
  function modifyTodo(updatedTodo) { return mapTodos(function (todo) { return todo.id === updatedTodo.id ? updatedTodo : todo; }); }
  function removeTodo(deletedTodo) { return filterTodos(function (todo) { return todo.id !== deletedTodo.id; }); }
  function addTodo(newTodo) { return function (todos) { return todos.concat([newTodo]); }; }
  function clearCompleted() { return function (todos) { return _.where(todos, {completed : false}); }; }

  var storage = new LocalStorage();

  this.todoAdded = new Bacon.Bus();
  this.todoModified = new Bacon.Bus();
  this.todoDeleted = new Bacon.Bus();
  this.clearCompleted = new Bacon.Bus();
  this.toggleAll = new Bacon.Bus();

  var todoChanges = this.todoAdded.map(addTodo)
                  .merge(this.todoDeleted.map(removeTodo))
                  .merge(this.clearCompleted.map(clearCompleted))
                  .merge(this.todoModified.map(modifyTodo))
                  .merge(this.toggleAll.map(toggleAll));

  this.allTodos = todoChanges.scan(storage.readTodos(), function (todos, f) { return f(todos); });
  this.activeTodos = this.allTodos.map(function (todos) { return _.where(todos, { completed: false}); });
  this.completedTodos = this.allTodos.map(function (todos) { return _.where(todos, { completed: true}); });

  this.allTodos.changes().onValue(storage.writeTodos);
}
```

Quite a piece, isn't it? That's mainly because lambdas in Javascript are quite verbose. But still.

Using `Bacon.update` this can be shrunk to just

```js
  function TodoListModel() {
    var storage = LocalStorage()

    this.todoAdded = new Bacon.Bus()
    this.todoModified = new Bacon.Bus()
    this.todoDeleted = new Bacon.Bus()
    this.clearCompleted = new Bacon.Bus()
    this.toggleAll = new Bacon.Bus()

    this.allTodos = Bacon.update(
      storage.readTodos(),
      [this.todoAdded], function(todos, todo) { return todos.concat([todo])},
      [this.todoDeleted], function(todos, deletedTodo) { return _.reject(todos, function(todo) { return todo.id === deletedTodo.id})},
      [this.clearCompleted], function(todos) { return _.where(todos, {completed : false})},
      [this.todoModified], function(todos, updatedTodo) { return _.map(todos, function(todo) { return todo.id === updatedTodo.id ? updatedTodo : todo }) },
      [this.toggleAll], function(todos, toggle) { return _.map(todos, function(todo) { return _.extend(_.clone(todo), { completed: toggle })})}
    )

    this.activeTodos = this.allTodos.map(function(todos) { return _.where(todos, { completed: false})})
    this.completedTodos = this.allTodos.map(function(todos) { return _.where(todos, { completed: true})})

    this.allTodos.changes().onValue(storage.writeTodos)
  }
```
