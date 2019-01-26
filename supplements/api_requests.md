# API Requests

Don't overthink your API requests or overwork your API.

---

## The Setup

Let's start with the classic todo list.

```javascript
class TodoList extends React.Component {
  state = {
    newTodo: '',
    todos: [],
    nextId: 1,
  }

  addTodo = () => {
    this.setState(({ newTodo, todos, nextId }) => ({
      newTodo: '',
      todos: [...todos, { id: nextId, text: newTodo }],
      nextId: nextId + 1,
    }))
  }

  removeTodo = removeId =>
    this.setState(({ todos }) => ({
      todos: todos.filter(({ id }) => id !== removeId),
    }))

  // left out of ui for example simplicity
  updateTodo = (id, newText) =>
    this.setState(({ todos }) => ({
      todos: todos.map(todo => (todo.id === id ? { ...todo, newText } : todo)),
    }))

  updateNewTodo = e => this.setState({ newTodo: e.target.value })

  render() {
    const { newTodo, todos } = this.state

    return (
      <div>
        <div>
          <input type="text" onChange={this.updateNewTodo} value={newTodo} />
          <button onClick={this.addTodo}>Add</button>
        </div>
        <div>
          {todos.map(todo => (
            <div key={todo.id}>
              <button onClick={() => this.removeTodo(todo.id)}>X</button>
              {todo.text}
            </div>
          ))}
        </div>
      </div>
    )
  }
}
```

Each todo item has `text` and an `id`. The id is simply an incrementing number also tracked in state for now. The list itself is an array of todo items.

We have functions for the typical CRUD actions on our local data.

## API Functions

A simple setup for JSON server would do nicely for our testing purposes.

```json
{
  "todos": [
    { "id": 1, "text": "The first thing to do" }
  ]
}
```

Depending on the app complexity, how much testability is desired, personal style, etc., api requests could be entirely within components, or the logic could be extracted into separate functions. Let's separate them for now.

Outside of any components, we can create the functions that take care of the api requests so that our component doesn't have to handle all of the specifics.

```javascript
const getAllTodos = () =>
  fetch('http://localhost:3000/todos')
    .then(response => response.json())

const createTodo = text =>
  fetch('http://localhost:3000/todos', {
    method: 'POST',
    body: JSON.stringify({ text }),
    headers:{
    'Content-Type': 'application/json'
    },
  }).then(response => response.json())

const updateTodo = (id, text) =>
  fetch(`http://localhost:3000/todos/${id}`, {
    method: 'PUT',
    body: JSON.stringify({ text }),
    headers:{
    'Content-Type': 'application/json'
    },
  }).then(response => response.json())

const deleteTodo = id =>
  fetch(`http://localhost:3000/todos/${id}`, { method: 'DELETE' })
    .then(response => response.json())
```

### Error Reporting

When dealing with promises, care must be taken to handle errors in the appropriate places. Overall, the error should be handled where the function is called because that consumer knows what does or does not need to be done with the error. However, we still may want to log the errors or take some other action.

One seemingly straightforward but (usually) subtley problematic approach is to catch errors in the api functions.

```javascript
const getAllTodos = () =>
  fetch('http://localhost:3000/todos')
    .then(response => response.json())
    .catch(console.error)
```

This will work just fine and log any errors encountered. The issue is that **a promise will will exectue any `.then()`s after a `.catch()` if they are caught**, but _the value will be undefined_ (or whatever is returned from the catch handler).

One way to get around this is to _re-throw an error_. We can tatke whatever actiosn we want and let the consumer of the function function as expected. A custom error allows us to provide an application-specific message to be consumed farther down the promise chain, but we could also simply re-throw the same error with `throw error` and let the consumer implement specific error handling.

```javascript
const getAllTodos = () =>
  fetch('http://localhost:3000/todos')
    .then(response => response.json())
    .catch(error => {
      console.error(error)
      throw new Error("Could not get todos")
    })
```

In a full production application, this could record the error, notify the server, etc., but at least putting a console message prevents errors from silently occurring during development.

Implementing this for all api functions, we end up with this:

```javascript
const getAllTodos = () =>
  fetch('http://localhost:3000/todos')
    .then(response => response.json())
    .catch(error => {
      console.error(error)
      throw new Error("Could not get todos")
    })

const createTodo = text =>
  fetch('http://localhost:3000/todos', {
    method: 'POST',
    body: JSON.stringify({ text }),
    headers:{
    'Content-Type': 'application/json'
    },
  })
    .then(response => response.json())
    .catch(error => {
      console.error(error)
      throw new Error("Could not get todos")
    })

const updateTodo = (id, text) =>
  fetch(`http://localhost:3000/todos/${id}`, {
    method: 'PUT',
    body: JSON.stringify({ text }),
    headers:{
    'Content-Type': 'application/json'
    },
  })
    .then(response => response.json())
    .catch(error => {
      console.error(error)
      throw new Error("Could not get todos")
    })

const deleteTodo = id =>
  fetch(`http://localhost:3000/todos/${id}`, { method: 'DELETE' })
    .then(response => response.json())
    .catch(error => {
      console.error(error)
      throw new Error("Could not get todos")
    })
```

Note: Axios has a [more complex errors](https://github.com/axios/axios#handling-errors).