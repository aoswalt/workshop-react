# Multiple Use Cards

By making use of slots, we can make a card component that is reusable without having to dig into its internals to add more uses.

Our `Card` component should only need to accept a single item and provide a place to insert a button if needed. The `Card` should not be concerned with any logic; its only purpose is presentation.

Skipping the styling, our `Card` is very simple:

```javascript
const Card = ({ item: { title, text }, button }) => (
  <div>
    <h3>{title}</h3>
    <p>{text}</p>
    <div>{button}</div>
  </div>
)
```

We can map over our data, turning it into a set of `Card`s.

To display only the item information without a button, we simply do not pass in a button.

```javascript
<div>
  {data.map(i => <Card item={i} key={i.id} />)}
</div>
```

Including an `Add` button is as easy as passing in the button:

```javascript
<div>
  {data.map(i => (
    <Card button={<button>Add</button>} item={i} key={i.id} />
  ))}
</div>
```

For a different button, such as `Remove`, it is exactly the same:

```javascript
<div>
  {data.map(i => (
    <Card button={<button>Remove</button>} item={i} key={i.id} />
  ))}
</div>
```

In the cases of passing in a wide variety of props and having sets of conditional logic, frequently, the design can be simplified by rethinking a component's api and purpose.
