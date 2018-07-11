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
<script crossorigin src="https://unpkg.com/react@16/umd/react.development.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>
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

Just like regular html, we have the makings of a simple nav menu.
