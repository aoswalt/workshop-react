# Todo List

Every bit of sample code eventually includes a todo list and this one is no different. Let's try using render props to separate the logic of the list management from the rest of the application.

In all, we will need our todo management component, a method of entering new todo items, and a component for a single item. For a streamlined walkthrough, we will go through these piecemeal instead of iteratively.

## List Logic

Our todo list should accept simple text for the items and provide 3 functions: add, delete, and toggle completion. The list itself should also include the completion status of an item and an id to track it by. For simplicity, we will stick to incrementing numeric ids and allow the consumer to pass in an initial id to start at if they pass in a list.

A single todo item will look like this:

```javascript
{
  id: 0,
  text: 'Do the thing',
  complete: false,
}
```

It would probably be best to store the todo list as an object for quick accessing of the items, but we will stick to an array for overall simplicity.

With these details in mind, we can start building our component to maintain the todo list. We need to maintain the list itself and the next item's id. If we set the starting state values from props, we can provide the option to initialize the component to an existing list. Because this component should only be responsible for the logic, we can make use of render props by only calling the children prop inside the render method.

```jsx
class Todos extends React.Component {
  state = {
    list: this.props.initialList,
    nextId: this.props.nextId,
  }

  render() {
    this.props.children({ list: this.state.list })
  }
}

Todos.defaultProps = {
  initialList: [],
  nextId: 0,
}
```

With the foundation in place, we can add a simple function to add new todo items and include it in the call to the render prop. The add function will take the text of the todo item and insert a new fully-formed item into state while incrementing the value for the next id.

```javascript
addItem = text =>
  this.setState(({ list, nextId }) => {
    const newItem = { text, id: nextId, complete: false }

    return {
      list: [...list, newItem],
      nextId: nextId + 1,
    }
  })

render() {
  return this.props.children({
    list: this.state.list,
    addItem: this.addItem,
  })
}
```

The delete function is quite straightforward as well. Sticking with an array and immutability, the easiest method to remove an item is to filter out based on its id.

```javascript
deleteItem = id =>
  this.setState(({ list }) => ({
    list: list.filter(t => t.id !== id),
  }))

render() {
  return this.props.children({
    list: this.state.list,
    addItem: this.addItem,
    deleteItem: this.deleteItem,
  })
}
```

The last function to add is a function to toggle the completion status. To keep with immutability in javascript, this becomes a bit cumbersome, but it is good practice, especially within React. We need to find the item, toggle the completion status, remove the item from the list, and create a new list with the updated item.

```javascript
toggleComplete = id =>
  this.setState(({ list }) => {
    const item = list.find(t => t.id === id)
    const toggledItem = { ...item, complete: !item.complete }
    const listWithoutItem = list.filter(t => t.id !== id)

    return { list: [...listWithoutItem, toggledItem] }
  })

render() {
  return this.props.children({
    list: this.state.list,
    addItem: this.addItem,
    deleteItem: this.deleteItem,
    toggleComplete: this.toggleComplete,
  })
}
```

This wraps up all of the pieces of data manipulation around the todo list itself. By providing the functions along with the data to the render prop, all of the details stay contained inside the `Todos` component.

## Todo Entry

A new todo item is nothing more than a bit of text. We can make a component that encapsulates the details of maintaining the field's value. All the component should need is a function for what to do with the field's value on submission.

```javascript
class EntryField extends React.Component {
  state = { value: this.props.value || '' }

  onChange = ({ target: { value } }) => this.setState({ value })

  onOk = e => {
    e.preventDefault()
    this.props.onOk(this.state.value)
    this.setState({ value: '' })
  }

  render() {
    return (
      <form onSubmit={this.onOk}>
        <input type="text" value={this.state.value} onChange={this.onChange} />
        <button type="submit" disabled={!this.state.value}>
          OK
        </button>
      </form>
    )
  }
}
```

## Single Todo Item

For a single todo item, aside from the item's text, we want to show its completed status and a button to delete it. We can represent the completed status by a simple checkbox. We can include a bit of styling to make the items a consistent width.

```javascript
const todoItemStyles = {
  container: {
    display: 'flex',
    alignItems: 'center',
  },
  text: {
    flex: 1,
  },
}

class TodoItem extends React.Component {
  onToggle = () => this.props.onToggle(this.props.todo.id)
  onDelete = () => this.props.onDelete(this.props.todo.id)

  render() {
    const {
      todo: { complete, id, text },
    } = this.props

    return (
      <div style={todoItemStyles.container}>
        <input type="checkbox" checked={complete} onChange={this.onToggle} />
        <span style={todoItemStyles.text} onClick={this.onToggle}>
          {text}
        </span>
        <button onClick={this.onDelete}>X</button>
      </div>
    )
  }
}
```

## All Put Together

Now that we have our foundational pieces, we can build our complete todo list application.

First, we can set up the rendering of each item by iterating of the `list` variable given by the render function call and providing each `TodoItem` component with the `deleteItem` and `toggleComplete` functions. We will add some styling for presentation's sake.

```javascript
const itemsContainerStyles = {
  maxWidth: '25%',
  display: 'flex',
  flexDirection: 'column',
  justifyContent: 'stretch',
}

const TodoList = () => (
  <Todos>
    {({ list, deleteItem, toggleComplete }) => (
      <div style={itemsContainerStyles}>
        {list.map(t => (
          <TodoItem
            key={t.id}
            todo={t}
            onDelete={deleteItem}
            onToggle={toggleComplete}
          />
        ))}
      </div>
    )}
  </Todos>
)
```

Because we do not control the ordering at this point, we should sort the list as it is rendered.

```javascript
const itemSort = (a, b) => a.id - b.id

const TodoList = () => (
  <Todos>
    {({ list, addItem, deleteItem, toggleComplete }) => (
      <div style={itemsContainerStyles}>
        {list.sort(itemSort).map(t => (
          <TodoItem
            key={t.id}
            todo={t}
            onDelete={deleteItem}
            onToggle={toggleComplete}
          />
        ))}
      </div>
    )}
  </Todos>
)
```

The last thing we need to do is to add the `EntryField` component and pass it the `addItem` function for adding new todo items. We can wrap the `EntryField` and list rendering in a `Fragment` because we are not concerned with how they are laid out together.

```javascript
const TodoList = () => (
  <Todos>
    {({ list, addItem, deleteItem, toggleComplete }) => (
      <React.Fragment>
        <EntryField onOk={addItem} />
        <div style={itemsContainerStyles}>
          {list.sort(itemSort).map(t => (
            <TodoItem
              key={t.id}
              todo={t}
              onDelete={deleteItem}
              onToggle={toggleComplete}
            />
          ))}
        </div>
      </React.Fragment>
    )}
  </Todos>
)
```

With everything put together, we have our complete todo list application. We can add, delete, and toggle items. The logic is contained within the `Todos` component but spread apart as needed through the children as a render prop.
