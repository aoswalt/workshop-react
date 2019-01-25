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
