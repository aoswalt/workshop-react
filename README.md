# Workshop: React

A quick workshop to learn the basics of [React](https://reactjs.org).

## Introduction

Generally, React is a javascript library for building reactive user interfaces that follow the unidirectional [Flux](https://facebook.github.io/flux/) architecture to efficiently handle changing UI based on changing data.

More specifically, the library known as "React" is now 2 separate packages: React and ReactDOM. The React package provides the logic and machinery to generate the change diffs as data is changed. ReactDOM is the set of html bindings fork working with the virtual DOM to efficiently update elements of the DOM.

Any application that can make use of a diffing engine can implement the React architecture. Some notable examples include [React Native](https://facebook.github.io/react-native/) for mobile development, [react-blessed](https://facebook.github.io/react-native/) for cli applications, and [React Hardware](http://iamdustan.com/react-hardware/) for hardware development.

For the remainder of this write up, "React" will describe the entire implemention for the web platform unless specifically noted.
