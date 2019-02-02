# API Requests

Don't overthink your API requests or overwork your API.

---

## The Setup

Let's start with the classic todo list.

```javascript
class TodoList extends React.Component {
  state = {
    newTodoText: '',
    todos: [],
    nextId: 1,
  }

  addTodo = () => {
    this.setState(({ newTodoText, todos, nextId }) => ({
      newTodoText: '',
      todos: [...todos, { id: nextId, text: newTodoText }],
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

  updateNewTodo = e => this.setState({ newTodoText: e.target.value })

  render() {
    const { newTodoText, todos } = this.state

    return (
      <div>
        <div>
          <input type="text" onChange={this.updateNewTodo} value={newTodoText} />
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

## Using the Functions

Within our component, only minimal changes are needed to use the api functions.

### Initial Data Loading

Because we only have one component and want to immediately load our data, we can put our get request inside `componentDidMount`.

```javascript
componentDidMount() {
  API.getAllTodos()
    .then(todos => this.setState({ todos }))
}
```

**Care should be taken when deciding when to automatically load data.**

Since we are at the top of the application, we can load data when the component mounts and move on. If we had multiple pages, we may consider loading data when different pages load and even storing the data at the top of the app but loading it further down.

If a child component is loading data on mount, it will load data _every time every instance of the component is mounted_. This could be excessive for items in a list, _especially_ if the data is already loaded, such as with the todos on our todo list.

### Deleting

Starting with probably the simplest change, let's take care of deleting todos.

Our logic for upating state can stay the same: filter out the todo with the matching id. The only change needed is to wait until we get a successful response from the api.

```javascript
removeTodo = removeId =>
  API.deleteTodo(removeId)
    .then(() => this.setState(({ todos }) => ({
      todos: todos.filter(({ id }) => id !== removeId)
    })))
```

### Adding and Updating

Both adding and updating a todo require the same type of minimal change: using the response from the api to update state.

Since our upate function does read from state, it is a bit more trivial. Instead of constructing the todo, we need to use the updated one returned from the api.

```javascript
updateTodo = (id, newText) =>
  API.updateTodo(id, newText)
    .then(updatedTodo =>
      this.setState(({ todos }) => ({
        todos: todos.map(todo =>
          todo.id === updatedTodo.id ? updatedTodo : todo,
        ),
      })),
    )
```

Adding a new todo is nearly the same change as updating except we need to pull the new text out of state first. We do not need to embed the api call inside of the setState function because the state update is happening asynchronously.

```javascript
addTodo = () => {
  const { newTodoText } = this.state

  API.createTodo(newTodoText)
    .then(newTodoText =>
      this.setState(({ todos, nextId }) => ({
        newTodoText: '',
        todos: [...todos, newTodo],
      })))
}
```

## Showing Errors

Usually, you will want to let the user know when something goes wrong. We can easily create a temporary message with a state property.

```javascript
state = {
  newTodoText: '',
  todos: [],
  errorMessage: '',
}
```

And a timeout.

```javascript
showErrorMessage = (error) => {
  this.setState({ errorMessage: error.message })
  setTimeout(() => this.setState({ errorMessage: '' }), 500)
}
```

Using the `showErrorMessage` function in the `.catch()` method takes care of the error handling within the component.

```javascript
getAllTodos()
  .then(todos => this.setState({ todos }))
  .catch(this.showErrorMessage)
```

The error message can be displayed by dropping it into the render method.

```javascript
render() {
  const { errorMessage, newTodoText, todos } = this.state

  return (
    <div>
      <div>
        <input type="text" onChange={this.updateNewTodo} value={newTodoText}/>
        <button onClick={this.addTodo}>Add</button>
        <div>{errorMessage}</div>
      </div>
      <div>
        {todos.map(todo =>
          <div key={todo.id}>
            <button onClick={() => this.removeTodo(todo.id)}>X</button>
            {todo.text}
          </div>
        )}
      </div>
    </div>
  )
}
```

Even with such rudimentary error handling, the user can be informed that something went wrong.

## Summarizing

Changing the todo functionality to work with the api required no change to the function signatures; the changes were all internal to the logic of the functions. This means that we could have given these functions to other components and had no need to change those components when the switch to an api was made.

By containing all of the logic in one place, the implmentation can be changed without requiring changes in other parts of the application. Since we started with React state, we were forced to already have all of that logic together. When starting with the api, it can be tempting to scatter the api calls, distributing the logic across the application.

Starting with React state also encourages us to make more use of the tools built into React, primarily state updates. Relying on refetching data after making changes loses that benefit and requires more round-trips to the api.

Working with apis should fit neatly into your application. If the api logic seems to be making things difficult, it may be time to take a step back and rethink what you are trying to accomplish and how you are going about it.
