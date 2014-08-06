IntegratedStyle
===============

The motivation for IntegratedStyle is that CSS is problematic to maintain and
React components as an abstraction give all the borders you actually need.

IntegratedStyle proposes that styles for a component should be defined in the
JavaScript code as a part of the component definition itself.

```
var React = require('react')
var IntegratedStyle = require('integratedstyle')

var Button = React.createClass({

  normalStyle: IntegratedStyle(function() {
    return {
      backgroundColor: 'orange'
    }
  }),

  activeStyle: IntegratedStyle(function() {
    if (this.state.active) {
      return {
        color: 'yellow',
        padding: '10px'
      }
    }
  }),

  getInitialState: function() {
    return {
      active: false
    }
  },

  render: function() {
    return (
      <button styles={[this.normalStyle(), this.activeStyle()]} onClick={this.onClick}>
        Hello, I'm styled
      </button>
    )
  },

  onClick: function() {
    this.setState({active: !this.state.active})
  }

})
```

The example above defines two styles for the `Button` component, `normalStyle`
and `activeStyle`.

Styles are defined as regular methods wrapped into `IntegratedStyle` method
decorator. This fact makes it possible to use the full power of JavaScript: you
can require modules with variables to be used in styles or use predefined styles
as mixins and so on.

Styles are applied to the component via `styles` property which is provide by
IntegratedStyle for all React.DOM components.

By default all styles are applied in the DOM inline, through `style` attribute.
This is OK for during development or small experiments but in production you
probably would want to use separate CSS bundle with styles.

IntegratedStyle provides [Webpack][] loader which can generate CSS classes for
component styles during bundling.

With the help of webpack loader the example above will generate the following
CSS:

```
.a {
  background-color: 'orange';
}
```

While the JavaScript code itself gets transformed into:

```
var React           = require('react/addons')
var IntegratedStyle = require('integratedstyle')
var vars            = require('./vars')

var Button = React.createClass({

  normalStyle: function() {
    return " a"
  },

  activeStyle: IntegratedStyle(function() {
    if (this.state.active) {
      return {
        color: 'yellow',
        padding: '10px'
      }
    }
  }),

  getInitialState: function() {
    return {
      active: false
    }
  },

  render: function() {
    var styles = [
      this.normalStyle(),
      this.activeStyle()
    ]
    return (
      <div styles={styles} onClick={this.onClick}>
        Hello, I'm styled
      </div>
    )
  },

  onClick: function() {
    this.setState({active: !this.state.active})
  }

})
```

Note how `normalStyle` style declaration is extracted into CSS class because it
is completely "static" (doesn't depend on the component's props and state) while
`activeStyle` is left as-is due to its dependency on `this.state.active`.

What does it actually do?
-------------------------

At runtime:

1. It adds style declaration to React components. Style declarations are regular
   methods which are decorated with `IntegratedStyle` decorator and return
   regular style rules.
2. It adds `styles` prop to all React.DOM components which allows to add styles
   to a component from a style declaration.

At code transformation:

1. It finds all component methods wrapped into `IntegratedStyle()` decorator.
2. It checks if such methods have references to `this`.

  2.1. If method has no reference to `this` it is executed and result is used to
       generated CSS class with a corresponding ruleset.

  2.2. If method has reference to `this` it is left as is, as it will be used to
       generate inline styles.

Usage
-----

TODO describe how to integrate IntegratedStyle with [webpack][].

Other options
-------------

- [RCSS](https://github.com/chenglou/rcss)
- [ReactStyles](https://github.com/hedgerwang/react-styles)
- [react-css](https://github.com/elierotenberg/react-css)

Biggest difference here is that IntegratedStyle is a CSS + JS preprocessor
solution instead of a runtime solution.

Also there is (from the React.js team):

- [Inline Style Extension](https://github.com/reactjs/react-future/blob/master/04 - Layout/Inline Style Extension.md)

LICENSE
-------

MIT

[React.js]: http://github.com/facebook/react
[recast]: http://github.com/benjamn/recast
[Webpack]: https://webpack.github.io
