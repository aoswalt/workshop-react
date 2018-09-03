# Workshop: React

A quick workshop to learn the basics of [React](https://reactjs.org).

## Introduction

Generally, React is a javascript library for building reactive user interfaces that follow the unidirectional [Flux](https://facebook.github.io/flux/) architecture to efficiently handle changing UI based on changing data.

More specifically, the library known as "React" is now 2 separate packages: React and ReactDOM. The React package provides the logic and machinery to generate the change diffs as data is changed. ReactDOM is the set of html bindings fork working with the virtual DOM to efficiently update elements of the DOM.

Any application that can make use of a diffing engine can implement the React architecture. Some notable examples include [React Native](https://facebook.github.io/react-native/) for mobile development, [react-blessed](https://facebook.github.io/react-native/) for cli applications, and [React Hardware](http://iamdustan.com/react-hardware/) for hardware development.

For the remainder of this write up, "React" will describe the entire implemention for the web platform unless specifically noted.

## Creating Elements

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

```jsx
const NavItem = ({ href, label }) =>
  <li>
    <a href={href}>{label}</a>
  </li>
```

If we continue storing our components as simple variables, we must use curly braces to insert them into the list.

```jsx
const home = <NavItem href="./home" label="Home" />
const contact = <NavItem href="./contact" label="Contact" />
const about = <NavItem href="./about" label="About" />
```

```jsx
const menu =
  <ul>
    {home}
    {contact}
    {about}
  </ul>
```

Instead, if we instead write them as components, we use them as their own JSX tags, just like native elements. Note: any custom component should begin with a capital letter to distinguish it from the native elements.

```jsx
const Home = () => <NavItem href="./home" label="Home" />
const Contact = () => <NavItem href="./contact" label="Contact" />
const About = () => <NavItem href="./about" label="About" />
```

Now, we can build the menu with our custom components but write them as JSX tags.

```jsx
const menu =
  <ul>
    <Home />
    <Contact />
    <About />
  </ul>
```

## Getting Stateful

So far, we have been creating stateless functional components: they are functions that take in data as props and return React elements. One thing we cannot do is keep track of data changes. For that, we need class components.

Class components extend `React.Component` and define a `render` method that returns the resulting elements. Instead of being given props in the method's arguments as with a functional component, props are accessed directly as `this.props`.

---

Let's make a toggle button.

First, we should create a simple component that wraps a button.

```jsx
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

```jsx
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

```jsx
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

```jsx
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
