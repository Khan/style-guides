## React style guide

> Follow the normal Javascript style guide - including the 80 character limit. In addition there are several React-specific rules.

----------
### Syntax

#### Name handlers `handleEventName`

Example:

```jsx
<Component onClick={this.handleClick} onLaunchMissiles={this.handleLaunchMissiles} />
```

As you may have noticed there is also a convention to name a handler
that your component takes as a parameter `onEventName`. This is
consistent with React's event naming - `onClick`, `onDrag`,
`onChange`, etc.


#### Open elements on the same line.

80 chars is a bit tight so we opt to conserve the extra 4.

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

#### Prefer [props to state](http://facebook.github.io/react/docs/interactivity-and-dynamic-uis.html).

You almost always want to use props.  Copying data from props to state
can cause the UI to get out of sync from the underlying data models
because what's displayed no longer directly tracks what's stored.  By
having the data in one place instead of copying it, you ensure
everything is consistent.

#### Use [propTypes](http://facebook.github.io/react/docs/reusable-components.html).

React Components should always have complete propTypes.  Every
attribute of this.props should have a corresponding entry in
propTypes.  This documents that props need to be passed to a model.
([example](http://jsfiddle.net/spicyj/DEpwb/))

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

Do not use data- attributes or classes).  All information should be
stored in Javascript, either in the React component itself, or in a
React store if using a framework such as Redux.


----------------------------------
### React libraries and components

#### Do not use Backbone models.

TODO

#### Minimize use of jQuery.

*Never* use jQuery for DOM manipulation.

Use native promises instead of jQuery promises.

Try to avoid using jQuery plugins.  When necessary, wrap the jQuery
plugin with a React component so you only have to touch the jQuery
once.

You can use `$.ajax` (but no other function, such as $.post) for
network communication.

#### Reuse standard components.

If possible, re-use existing "dumb" components (pure components that
emit html directly).  If you write a new one, as soon as it finds one
other client put it in a shared location.

The standard shared location for useful components provided *by the
React team* is the `react-components` package in
`javascript-packages.json`.  This includes components such as these:

* `SetIntervalMixin` - provides a setInterval method so something can be
  done every x milliseconds
* `$_` - the i18n wrapper to allow for translating text in React.
* `TimeAgo` - “five minutes ago”, etc - this replaces $.timeago

Reusable components *written internally* are in the (poorly named)
`react.js` package.  This include components such as these:

* `KUIButton` - render a Khan Academy styled button.
* `Modal` - create a modal dialog.

This list is far from complete.

### 
