# Workshop: React

A quick workshop to learn the basics of [React](https://reactjs.org).

---

* [Introduction](#introduction) - What we are talking about
* [Creating Elements](#creating-elements) - Making basic elements
* [Getting Stylish](#getting-stylish) - Adding inline styles to elements
* [Elementary Composition](#elementary-composition) - Putting elements together
* [Props and Components](#props-and-components) - Custom elements and their arguments
* [Babeling about JSX](#babeling-about-jsx) - Using Babel to allow writing JSX
* [Getting Stateful](#getting-stateful) - Storing state in components
* [Sharing with Others](#sharing-with-others) - Sharing state between components
  * [Taking Full Control](#taking-full-control) - Overring state and state changes
* [Composition Patterns](#composition-patterns) - Methods of flexibly composing components
  * [Children](#children) - Accepting elements as children
  * [Slots](#slots) - Providing slots for inserting elements
  * [Render Props](#render-props) - Using a function to allow for flexible rendering
* [Examples](#examples) - Some example designs

## Introduction

What are we talking about

---

Generally, React is a javascript library for building reactive user interfaces that follow the unidirectional [Flux](https://facebook.github.io/flux/) architecture to efficiently handle changing UI based on changing data.

More specifically, the library known as "React" is now 2 separate packages: React and ReactDOM. The React package provides the logic and machinery to generate the change diffs as data is changed. ReactDOM is the set of html bindings fork working with the virtual DOM to efficiently update elements of the DOM.

Any application that can make use of a diffing engine can implement the React architecture. Some notable examples include [React Native](https://facebook.github.io/react-native/) for mobile development, [react-blessed](https://github.com/Yomguithereal/react-blessed) for cli applications, and [React Hardware](http://iamdustan.com/react-hardware/) for hardware development.

For the remainder of this write up, "React" will describe the entire implemention for the web platform unless specifically noted.

## Creating Elements

Making basic elements

---

React elements are the building blocks of a React application. Conceptually, they are the same as html elements.

To create your first React elements, first, add a couple of script tags to your `index.html` file, one for react and the other for react-dom.

```html
<script src="https://unpkg.com/react@16/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>
```

Next, add a div with an id that you can reference.

```html
<div id="app"></div>
```

Finally, add some javascript to a script tag that creates an element and renders it to the div.

```html
<script>
  // an h1 tag with the content of the text "Hellow, World!"
  const title = React.createElement('h1', null, 'Hello, World!')

  // get a reference to the app div
  const appContainer = document.querySelector('#app')

  // render the title inside the app div
  ReactDOM.render(title, appContainer)
</script>
```

You have just written your first React code!

Frequently, the `createElement` and `render` functions will be aliased to make the code feel less cumbersome.

```html
<script>
  const e = React.createElement
  const r = ReactDOM.render

  const appContainer = document.querySelector('#app')

  r(
    e('h1', null, 'Hello, World!'),
    appContainer
  )
</script>
```

## Getting Stylish

Adding inline styles to elements

---

A big `h1` element is fun, but we can do more.

The `createElement` function takes 3 arguments: the html tag name, an object of html attributes (props in react terms), and the elements child(ren).

```javascript
React.createElement(
  'h1',  // this is an `h1` tag
  null,  // we will come back to the attributes
  'Hello, World!'  // the h1 only has a string for a child
)
```

Let's add a bit of color to our title. Add a `style` section with some css styling to the page.

```html
<style>
  .title {
    color: red;
  }
</style>
```

With that in place, we can add the class to our component. In standard html, we would add an attribute to the element.

```html
<h1 class="title">Hello, World!</h1>
```

To add the class to the element, we pass in an any attributes inn the shape of an object to the second parameter. One caveat to React is that we use `className` as the class attribute instead of `class` to avoid conflicting with the javascript keyword.

```javascript
React.createElement(
  'h1',
  { className: 'title' }, // equivalent to `class="title"`
  'Hello, World!'
)
```

We have **color**!

We could have added it as an inline style, but all of the usual html best practices still apply with React. If we still wanted to do so, inline styles can be passed in as an object.

```javascript
React.createElement(
  'h1',
  { style: { color: 'red' } }, // equivalent to `style="color: red"`
  'Hello, World!'
)
```

## Elementary Composition

Putting elements together

---

Having some text as an element's children is nice for the simple case, but we can do better. HTML elements can contain other elements, and React can too.

A general approach to creating a nav menu is a set of list items in an unordered list.

```html
<ul>
  <li>Home</li>
  <li>Contact</li>
  <li>About</li>
</ul>
```

We can easily do the same in React by using an array of elements as the children.

```javascript
e('ul', null, [
  e('li', null, 'Home')
  e('li', null, 'Contact')
  e('li', null, 'About')
])
```

With the aliased `createElement` method, the React version looks very similar to the raw html version. Alternately, since it is all javascript, we could always store the list items as variables first.

```javascript
const home = e('li', null, 'Home')
const contact = e('li', null, 'Contact')
const about = e('li', null, 'About')

const menu = e('ul', null, [home, contact, about])
```

Just as with plain html, elements can be nested indefintely. Let's turn the items into links with `a` tags.

```javascript
const home =
  e('li', null,
    e('a', null, 'Home')
  )
const contact =
  e('li', null,
    e('a', null, 'Contact')
  )
const about =
  e('li', null,
    e('a', null, 'About')
  )

const menu = e('ul', null, [home, contact, about])
```

To make them proper links, we need to give the `a` tags an `href` attribute in the second argument of the function.

```javascript
const home =
  e('li', null,
    e('a', { href: './home' }, 'Home')
  )
const contact =
  e('li', null,
    e('a', { href: './contact' }, 'Contact')
  )
const about =
  e('li', null,
    e('a', { href: './about' }, 'About')
  )

const menu = e('ul', null, [home, contact, about])
```

Just like regular HTML, we have the makings of a simple nav menu.

## Props and Components

Custom elements and their arguments

---

A terminology note: The object of properties given to an element translates to the `attributes` on an HTML element. Within React, these are known as `props`.

So far, we have a standard looking set of HTML elements to build a simple nav menu. Because this is javascript, we can extract the common parts into a function to simplify the repetition.

Let's make a `NavItem` function that wraps the `li` and `a` tags together. If we provide the arguments as an object, we can keep them explicitly labeled. To make the anchor tag, a NavItem needs to take 2 props: an href and a label.

```javascript
// destructuring the props in the function parameter
const NavItem = ({ href, label }) =>
  e('li', null,
    e('a', { href }, label)
  )

const home = NavItem({ href: './home', label: 'Home' })
const contact = NavItem({ href: './contact', label: 'Contact' })
const about = NavItem({ href: './about', label: 'About' })

// the menu doesn't care that the elements were created with components
const menu = e('ul', null, [home, contact, about])
```

`NavItem` is our first `component`. Fundamentally, a component takes an object of arguments (known as props) and returns a number of elements or other components. In this simplified form, known as a functional component, that is all that is involved with a component.

Components are one of the core parts of react. They allow you to group related logic and design into reusable and composable pieces. As we will see later, components can be expanded to have more behaviour if needed, but the simple functional form remains at the core.


## Babeling about JSX

Using Babel to allow writing JSX

---

Creating elements with an aliased createElement function is straightforward, but wouldn't it be easier if we could write them as HTML? HTML is not valid inside of JavaScript, but we can use babel to expand JavaScript's capabilities, including making use of JSX.

In a project, you would set up a build pipeline to compile our JSX-filled, non-standard JavaScript into standard JavaScript; however, we can use a script tag to do the compilation on page load.

Add another script tag with the React ones.

```html
<script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
```

Change the opening script tag around our JavaScript to include a type of `text/babel`.

```html
<script type="text/babel">
```

Now, our javascript will not be run by the browser directly; instead, Babel will take over its execution. React includes configuration to translate JSX into calls to createElement.

Working from top down, we can rewrite our nav menu to make use of JSX.

First, the NavItem component can return HTML as JSX tags. Note: within JSX, variables and direct JavaScript code must be wrapped in curly braces.

```javascript
const NavItem = ({ href, label }) =>
  <li>
    <a href={href}>{label}</a>
  </li>
```

If we continue storing our components as simple variables, we must use curly braces to insert them into the list.

```javascript
const home = <NavItem href="./home" label="Home" />
const contact = <NavItem href="./contact" label="Contact" />
const about = <NavItem href="./about" label="About" />
```

```javascript
const menu =
  <ul>
    {home}
    {contact}
    {about}
  </ul>
```

Instead, if we instead write them as components, we use them as their own JSX tags, just like native elements. Note: any custom component should begin with a capital letter to distinguish it from the native elements.

```javascript
const Home = () => <NavItem href="./home" label="Home" />
const Contact = () => <NavItem href="./contact" label="Contact" />
const About = () => <NavItem href="./about" label="About" />
```

Now, we can build the menu with our custom components but write them as JSX tags.

```javascript
const menu =
  <ul>
    <Home />
    <Contact />
    <About />
  </ul>
```

## Getting Stateful

Storing state in components

---

So far, we have been creating stateless functional components: they are functions that take in data as props and return React elements. One thing we cannot do is keep track of data changes. For that, we need class components.

Class components extend `React.Component` and define a `render` method that returns the resulting elements. Instead of being given props in the method's arguments as with a functional component, props are accessed directly as `this.props`.

---

Let's make a toggle button.

First, we should create a simple component that wraps a button.

```javascript
class ToggleButton extends React.Component {
  render() {
    return (
      <button>OK</button>
    )
  }
}
```

State must be a plain JavaScript object. With newer JavaScript syntax support, we can directly set state with a class property. Originally, it needed to be done in the class's constructor.

Let's track the status with a boolean `value` attribute and default it to `false`.

```javascript
class ToggleButton extends React.Component {
  state = {
    value: false,
  }

  render() {
    return (
      <button>OK</button>
    )
  }
}
```

We can use a simple ternary to change the button's text to reflect the value of the toggle.

```javascript
class ToggleButton extends React.Component {
  state = {
    value: false,
  }

  render() {
    return (
      <button>{this.state.value ? 'Yes' : 'No'}</button>
    )
  }
}
```

We can pass a function to the button's `onClick` prop to update our state. Because another element will be calling the function, it must be bound to the current instance of the component; the easiest way to do so is to use an arrow function.

To update state in React, we must use the `this.setState` function (not update state directly) and provide it the desired changes to state. If the changes do not depend on the current state, you can directly pass an object of the changes you want. If the changes depend on current state, you should **ALWAYS** use the functional version of setState that takes the previous state as its first argument. State changes may be asynchronous and batched, so referencing `this.state` for changes may result in unexpected and hard to detect errors. The React documentation has more [detailed information](https://reactjs.org/docs/state-and-lifecycle.html#state-updates-may-be-asynchronous).

For the ToggleButton, we want to toggle state based on previous state. We can create a simple function on the component and pass it to the button.

```javascript
class ToggleButton extends React.Component {
  state = {
    value: false,
  }

  toggle = () =>
    this.setState(({ value }) => ({ value: !value }))

  render() {
    return (
      <button onClick={this.toggle}>
        {this.state.value ? 'Yes' : 'No'}
      </button>
    )
  }
}
```

We now have a simple button that maintains state and can toggle between its states, but we do not yet have external access to that state.

## Sharing with Others

Sharing state between components

---

State inside of a React component cannot be accessed from outside of it unless the component accepts related props or passes its state to its children.

Let's make use of the button by adding a label that reflects the status of the button and sits beside it. We can call this a `LabeledToggle`.

```javascript
class ToggleButton extends React.Component {
  state = {
    value: false,
  }

  toggle = () =>
    this.setState(({ value }) => ({ value: !value }))

  render() {
    return (
      <button onClick={this.toggle}>
        {this.state.value ? 'Yes' : 'No'}
      </button>
    )
  }
}

class LabeledToggle extends React.Component {
  render() {
    const value = false

    return (
      <div>
        <span>{value ? 'Yes' : 'No'}</span>
        <ToggleButton />
      </div>
    )
  }
}
```

Now, we have a label that should be reflecting the state of the button, but we cannot react into the button to read its state. We need to track the state within the `LabeledToggle` so that we can pass it to the label. We also need a function similar to the Button's `toggle` that can update the state. Let's call it `updateValue` to be explicit.

```javascript
class ToggleButton extends React.Component {
  state = {
    value: false,
  }

  toggle = () =>
    this.setState(({ value }) => ({ value: !value }))

  render() {
    return (
      <button onClick={this.toggle}>
        {this.state.value ? 'Yes' : 'No'}
      </button>
    )
  }
}

class LabeledToggle extends React.Component {
  state = {
    value: false,
  }

  updateValue = value =>
    this.setState({ value })

  render() {
    return (
      <div>
        <span>{this.state.value ? 'Yes' : 'No'}</span>
        <ToggleButton />
      </div>
    )
  }
}
```

To get the data from the button, we need to pass our `updateValue` to the button and have the button call it when its state changes.

Because setting state is asynchronous, we should call the button's passed in function in the callback to `this.setState` if it exists. The second argument to `this.setState` is an optional function to be called once state is fully updated.

```javascript
class ToggleButton extends React.Component {
  state = {
    value: false,
  }

  toggle = () =>
    this.setState(
      ({ value }) => ({ value: !value }),
      () => this.props.onChange && this.props.onChange(this.state.value)
    )

  render() {
    return (
      <button onClick={this.toggle}>
        {this.state.value ? 'Yes' : 'No'}
      </button>
    )
  }
}

class LabeledToggle extends React.Component {
  state = {
    value: false,
  }

  updateValue = value =>
    this.setState({ value })

  render() {
    return (
      <div>
        <span>{this.state.value ? 'Yes' : 'No'}</span>
        <ToggleButton onChange={this.updateValue} />
      </div>
    )
  }
}
```

### Taking Full Control

Overring state and state changes

---

This approach is fine if we want the button to be the source of truth; however, if we take control of the button's value, we can move the source of truth into a place where we have more control. In general, you want to override both the `onChange` and `value` props together for a component.

There are various approaches to changing the control of a component, but one of the most direct is to add some checks about which data to respond to and which functions to call.

```javascript
class ToggleButton extends React.Component {
  state = {
    value: false,
  }

  toggle = () => {
    this.props.onChange === undefined
      ? this.setState(({ value }) => ({ value: !value }))
      : this.props.onChange(!this.props.value)
  }

  render() {
    const value = this.props.value === undefined
      ? this.state.value
      : this.props.value

    return (
      <button onClick={this.toggle}>
        {value ? 'Yes' : 'No'}
      </button>
    )
  }
}

class LabeledToggle extends React.Component {
  state = {
    value: false,
  }

  updateValue = value =>
    this.setState({ value })

  render() {
    const { value } = this.state

    return (
      <div>
        <span>{value ? 'Yes' : 'No'}</span>
        <ToggleButton value={value} onChange={this.updateValue} />
      </div>
    )
  }
}
```

Now, our `LabeledToggle` maintains the source of truth for the value of the label as well as the button; therefore, we can make changes in different ways while keeping everything in sync.

As a proof of concept, we can test it by adding a rough `onClick` event handler to our label's `span` tag. Note: you generally [should not use a raw arrow function for an event handler](https://reactjs.org/docs/handling-events.html).

```javascript
render() {
  const { value } = this.state

  return (
    <div>
      <span onClick={() => this.updateValue(!value)}>{value ? 'Yes' : 'No'}</span>
      <ToggleButton value={value} onChange={this.updateValue} />
    </div>
  )
}
```

We can now change the same state with the same funciton but by 2 separate sources while keeping one source of truth for the value.

## Composition Patterns

Methods of flexibly composing components

---

So far, our custom components have bundled up their presentation and logic with only allowing minimal control to consumers of the component. This is fine for most cases when you know the uses of your component. A component can be designed for more general cases by separating the logic and presentation.

There are numerous methods to exposing control of a component to its user. 3 of the most common are making use of children, adding slots, and render props.

### Children

Accepting elements as children

---

We have been making use of childen with the native elements, such as the list items of a list.

```html
<ul>
  <li>First</li>
  <li>Second</li>
  <li>Third</li>
</ul>
```

When creating components, we can access the children through the children prop.

As a simple example, if we have a design for a card to be used for items on a page, we can make use of the children prop to allow the user to pass in any elements they want. Let's also add a prop for a card title.

```javascript
const Card = ({ children, title }) => (
  <div
    style={{
      border: '2px dashed black',
      flex: '0 0 30%',
      minHeight: '10rem',
    }}
  >
    <h3 style={{ textAlign: 'center' }}>{title}</h3>
    <div style={{ margin: '.5rem' }}>
      {children}
    </div>
  </div>
)
```

To use it, we simply put any elements we desire between opening and closing `Card` tags.

From a simple `p` tag:

```html
<Card title='First'>
  <p>A body of text</p>
</Card>
```

To an `img` tag:

```html
<Card title='Second'>
  <img
    src='https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png'
    style={{ maxWidth: '100%', maxHeight: '100%' }}
  />
</Card>
```

To an assortment of elements:

```html
<Card title='Third'>
  <p>A random assortment</p>
  <h4>of elements</h4>
  <p>with varying</p>
  <h5>types</h5>
</Card>
```

Putting it all together:

```html
<div style={{ display: 'flex', flexWrap: 'wrap' }}>
  <Card title='First'>
    <p>A body of text</p>
  </Card>
  <Card title='Second'>
    <img
      src='https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png'
      style={{ maxWidth: '100%', maxHeight: '100%' }}
    />
  </Card>
  <Card title='Third'>
    <p>A random assortment</p>
    <h4>of elements</h4>
    <p>with varying</p>
    <h5>types</h5>
  </Card>
</div>
```

Going further, because the layout of a `Card` and its container are related, we could make the container its own component to treat them as a pair intended to be used together.

```javascript
const CardContainer = ({ children }) => (
  <div style={{ display: 'flex', flexWrap: 'wrap' }}>
    {children}
  </div>
)
```

Now, the relationship is clear, and the user does not have to be aware of the intended layout of the `Card`s.

```html
<CardContainer>
  <Card title='First'>
    <p>A body of text</p>
  </Card>
  <Card title='Second'>
    <img
      src='https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png'
      style={{ maxWidth: '100%', maxHeight: '100%' }}
    />
  </Card>
  <Card title='Third'>
    <p>A random assortment</p>
    <h4>of elements</h4>
    <p>with varying</p>
    <h5>types</h5>
  </Card>
</CardContainer>
```

Making use of children allows for flexible container elements that do not prescribe their usage and can prevent the need of threading props through layers of specialized components.

### Slots

Providing slots for inserting elements

---

Using the children prop works well when there is one section of a component with varying content. To expose multiple places where the user can insert elements, such as with a layout component, the slots pattern is a straightforward option, simply provide some props that will be placed directly into the DOM.

We can change our `Card` to insert a header instead of only a text title. To keep backwards compatibility, we should keep the option to provide a simple title. If a full header is provided, we will use it; otherwise, we will fall back to using the title. This can be done with a simple short circuit expression.

```javascript
const Card = ({ children, title, header }) => (
  <div
    style={{
      border: '2px dashed black',
      flex: '0 0 30%',
      minHeight: '10rem',
    }}
  >
    {header || <h3 style={{ textAlign: 'center' }}>{title}</h3>}
    <div style={{ margin: '.5rem' }}>
      {children}
    </div>
  </div>
)
```

With that simple addition, the user can override the entire header and provide any elements they want. Let's change the header of the first card to be a centered `a` tag.

```javascript
<Card
  header={
    <div style={{ textAlign: 'center' }}>
      <a href='#'>Link Title</a>
    </div>
  }
>
  <p>A body of text</p>
</Card>
```

Providing slots for a component allows for separating a different set of concerns to the user.

### Render Props

Using a function to allow for flexible rendering

---

At times, there is data whose management could be separated from any display aspect.

As a simple example, we could create a simple `Hover` component to encapsulate hover logic.

```javascript
class Hover extends React.Component {
  state = { hovering: false }

  onEnter = () => this.setState({ hovering: true })
  onLeave = () => this.setState({ hovering: false })

  render() {
    return (
      <span
        onMouseEnter={this.onEnter}
        onMouseLeave={this.onLeave}
      >
        {this.props.render(this.state.hovering)}
      </span>
    )
  }
}
```

Now, we give a function to the `Hover`'s render prop and return elements to render based on whether it is being hovered or not.

```javascript
<Hover
  render={hovering =>
    hovering
      ? <h1>Hovering!</h1>
      : <h2>Not Hovering...</h2>
  }
/>
```

The logic of tracking the hover state is contained within the `Hover` component, and the user of the component only has to be concerned with what to do with the hovered states.

Optionally, instead of defining a specific render prop, the component's children can be used in its place. If following this approach, it should be well documented since it deviates from the standard React pattern.

```javascript
class Hover extends React.Component {
  state = { hovering: false }

  onEnter = () => this.setState({ hovering: true })
  onLeave = () => this.setState({ hovering: false })

  render() {
    return (
      <span
        onMouseEnter={this.onEnter}
        onMouseLeave={this.onLeave}
      >
        {this.props.children(this.state.hovering)}
      </span>
    )
  }
}
```

With this change, we simply provide the function as the `Hover` component's children instead of to the render prop.

```javascript
<Hover>
  {hovering =>
    hovering
      ? <h1>Hovering!</h1>
      : <h2>Not Hovering...</h2>
  }
</Hover>
```

Functionally, these 2 approaches lead to the same result, but the differing semantics may be clearer for different situations.

## Examples

Some example designs

---

* [Multiple Use Cards](./examples/multiple_use_cards.md)
* [Reusable Form](./examples/reusable_form.md)
* [Todo List](./examples/todo_list.md)

## Supplements

Extra bits after getting the basics down

---

* [API Requests](./supplements/api_requests.md)
