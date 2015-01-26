# React style guide

**Follow the normal Javascript style guide - including the 80 character limit. In addition there are several React-specific rules.**

## props vs state

You almost always want to use [props, not state](http://facebook.github.io/react/docs/interactivity-and-dynamic-uis.html).

Copying data from props to state can cause the UI to get out of sync form the underlying data models because what's displayed no longer directly tracks what's stored. Instead, strive to have data in one place instead of copying it, which ensures that everything is consistent.

## When and how do you use Backbone models?

You probably shouldn't. **TODO: expand**

## Use [propTypes](http://facebook.github.io/react/docs/reusable-components.html)

Use it for every prop. This is the place to go to figure out which props need to be passed to a model. Great [example](http://jsfiddle.net/spicyj/DEpwb/).

Make sure you understand required, instance, enum, and custom PropTypes.

## Minimize use of jQuery

With React, jQuery should

- *never* be used for DOM manipulation
- rarely be used for ajax - Backbone can usually take care of this for you
- rarely be used for plugins - I recommend wrapping the jQuery plugin with a React component so you only have to touch the jQuery once

In a similar vein, *never* store information in the DOM (using data- attributes or classes). All information should be stored in Javascript. Note: this rule probably applied before but is worth emphasizing with React. Ask Joel if you want to hear a rant on this subject.

## Name handlers `handleEventName`

Example:

```jsx
<Component onClick={this.handleClick} onLaunchMissiles={this.handleLaunchMissiles} />
```

As you may have noticed there is also a convention to name a handler that your component takes as a parameter `onEventName`. This is consistent with React's event naming - `onClick`, `onDrag`, `onChange`, etc.

## Standard components

**TODO: update this list**

react-package provides several standard components, and there are more to come.

- SetIntervalMixin - provides a setInterval method so something can be done every x milliseconds
- Animation - still a work in progress. demo

More KA-specific

- TimeAgo - “five minutes ago”, etc - this replaces $.timeago
- UserHoverable - the thing where you can hover over a username
- blink, rumble, shudder, sparkle - for some reason React doesn’t come with these built in
- etc - a lot of components have been built and they’re all supremely reusable

For now (until the rise of the New Package Manager) drop any reusable components (anything you could see someone else using) you've made in react-package. Also, let everyone know what you've built. Email the team and blog it.

## Open elements on the same line

```jsx
return <div>
   ...
</div>
```

not

```jsx
return (
    <div>
        ...
    </div>
);
```

80 chars is a bit tight so we opt to conserve the extra 4.

Align properties

Fit them all on the same line if you can, but put them all in the same column if you have to break.

Good:

`<div class="highlight" key="highlight-div">`

Good

```jsx
<div
    class="highlight"
    key="highlight-div">
```

Bad

```jsx
<div class="highlight"
    key="highlight-div"
```

This makes it easy to see the props at a glance.
