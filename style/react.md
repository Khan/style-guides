## React style guide

> Follow the normal [Javascript style guide](javascript.md) - including the 80 character line limit. In addition there are several React-specific rules.

In addition to these style rules, you may also be interested in
[React best practices](https://docs.google.com/document/d/1ChtFUao18IyNhaXZ5sE2W-CFuFcYnqlFTyi5gfe6XV0/edit).

----------
### Syntax

#### Order your methods with lifecycle first and render last.

Within your react component, you should order your methods like so:

1. lifecycle methods
2. everything else
3. `render()`

#### Name handlers `handleEventName`.

Example:

```jsx
<Component onClick={this.handleClick} onLaunchMissiles={this.handleLaunchMissiles} />
```

#### Name handlers in props `onEventName`.

This is consistent with React's event naming: `onClick`, `onDrag`,
`onChange`, etc.

Example:

```jsx
<Component onLaunchMissiles={this.handleLaunchMissiles} />
```


#### Open elements on the same line.

The 80-character line limit is a bit tight so we opt to conserve the extra 4.

Yes:
```jsx
return <div>
   ...
</div>;
```

No:
```jsx
return (
    <div>
        ...
    </div>
);
```

#### Align properties.

Fit them all on the same line if you can, but put them all in the same
column if you have to break.  This makes it easy to see the props at a
glance.

Yes:
```jsx
<div className="highlight" key="highlight-div">
<div
    className="highlight"
    key="highlight-div">
```

No:
```jsx
<div className="highlight"
     key="highlight-div"
```

#### Only export a single react class.

Every .jsx file should export a single react class, and nothing else.
This is for testability: the fixture framework requires it to
function.

Note the file can still define multiple classes, it just can't export
more than one.


---------------------
### Language features

#### Prefer [props to state](http://facebook.github.io/react/docs/interactivity-and-dynamic-uis.html#what-components-should-have-state).

You almost always want to use props.  By avoiding state when possible,
you minimize redundancy, making it easier to reason about your
application.

A common pattern is to create several stateless components that just
render data, and have a stateful component above them in the hierarchy
that passes its state to its children via props. The stateful
component encapsulates all of the interaction logic, while the
stateless components take care of rendering data in a declarative way.

Copying data from props to state ​can cause the UI to get out of sync
and is especially bad.

#### Use [propTypes](http://facebook.github.io/react/docs/reusable-components.html).

React Components should always have complete propTypes.  Every
attribute of this.props should have a corresponding entry in
propTypes.  This documents that props need to be passed to a model.
([example](https://github.com/Khan/webapp/blob/32aa862769d4e93c477dc0ee0388816056252c4a/javascript/search-package/search-results-list.jsx#L14))

Avoid these non-descriptive prop-types:
   * `React.PropTypes.any`
   * `React.PropTypes.array`
   * `React.PropTypes.object`

Instead, use 
   * `React.PropTypes.arrayOf`
   * `React.PropTypes.objectOf`
   * `React.PropTypes.instanceOf`
   * `React.PropTypes.shape`

An exception is if you are simply passing the data to a child component.

#### Make "dumb" components pure.

It's useful to think of the React world as divided into "smart"
components and "dumb" components.

"Smart" components have application logic, but do not emit HTML
themselves.

"Dumb" components are typically reusable, and do emit HTML.

Smart components can have internal state, but dumb components never
should.

#### *Never* store state in the DOM.

Do not use `data-` attributes or classes.  All information
should be stored in Javascript, either in the React component itself,
or in a React store if using a framework such as Redux.


----------------------------------
### React libraries and components

#### Do not use Backbone models.

Use flux actions, or `$.ajax` directly, instead.

We are trying to remove backbone from our codebase entirely.

#### Minimize use of jQuery.

*Never* use jQuery for DOM manipulation.

Try to avoid using jQuery plugins.  When necessary, wrap the jQuery
plugin with a React component so you only have to touch the jQuery
once.

You can use `$.ajax` (but no other function, such as $.post) for
network communication.

#### Reuse standard components.

If possible, re-use existing components, especially low-level, pure
components that emit html directly.  If you write a new such one, and
it finds a user in a different project, put it in a shared location
such as the react.js package.

The standard shared location for useful components that have been
opensourced is the `react-components.js` package in
`javascript-packages.json`.  This includes components such as these:

* `SetIntervalMixin` - provides a setInterval method so something can be
  done every x milliseconds
* `$_` - the i18n wrapper to allow for translating text in React.
* `TimeAgo` - “five minutes ago”, etc - this replaces $.timeago

Reusable components that have not (yet) been opensourced are in the
(poorly named) `react.js` package.  This include components such as
these:

* `KUIButton` - render a Khan Academy styled button.
* `Modal` - create a modal dialog.

This list is far from complete.

### 
