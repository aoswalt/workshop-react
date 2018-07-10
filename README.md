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
