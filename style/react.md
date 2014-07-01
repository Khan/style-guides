# React style guide

**Follow the normal Javascript style guide - including the 80 character limit. In addition there are several React-specific rules.**

## props vs state

You almost always want to use [props, not state](http://facebook.github.io/react/docs/interactivity-and-dynamic-uis.html).

Copying data from props to state can cause the UI to get out of sync form the underlying data models because what's displayed no longer directly tracks what's stored. Instead, strive to have data in one place instead of copying it, which ensures that everything is consistent.

## When and how do you use Backbone models?

Use Backbone models often. Always use a model for data that's both readable and writable. This is optional for data thats read-only or write-only. Backbone models work really well with React - see for instance this [example](http://jsfiddle.net/WV3U7/8/) (note BackboneMixin is provided by `react-package`). Same goes for collections - they should just work.

We're principled software engineers.

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

## Waiting for jsx compilation in dev

On prod everything executes in the order it’s included on the page.

In dev, due to jsx compilation, jsx files execute last (though Craig’s planned packaging work will change this so they execute at the same time on prod and dev). Say you have a setup like this:

pkg1.js:
    pkg1file1.jsx
    pkg1file2.js

pkg2.js:
    pkg2file.js
Assuming they’re included in the page in that order, execution will go:

pkg1file2.js
pkg2file.js
pkg1file1.jsx
So if pkg2file.js relies on pkg1file1.jsx, it could find that pkg1file1.jsx hasn’t been evaluated yet.

The fix is simple - just use PackageManager to define and require packages. The code in pkg2file.js that relies on pkg1file1.jsx will wait until that file has executed and called setLoaded.

There’s one catch - PackageManager by default calls setLoaded for you after including the file. So the files are executed in this order:

pkg1file2.js
PackageManager.setLoaded('pkg1.js');
pkg2file.js
PackageManager.setLoaded('pkg2.js');
pkg1file1.jsx
The trick is to tell PackageManager to not setLoaded, by using {{ js_css_packages.package('pkg1.js', register=False) }}, or something similar.

## Standard components

react-package provides several standard components, and there are more to come.

- BackboneMixin - automatically updates your component whenever the model or collection it represents changes
- SetIntervalMixin - provides a setInterval method so something can be done every x milliseconds
- Animation - still a work in progress. demo

More KA-specific

- TimeAgo - “five minutes ago”, etc - this replaces $.timeago
- UserHoverable - the thing where you can hover over a username
- blink, rumble, shudder, sparkle - for some reason React doesn’t come with these built in
- etc - a lot of components have been built and they’re all supremely reusable

For now (until the rise of the New Package Manager) drop any reusable components (anything you could see someone else using) you've made in react-package. Also, let everyone know what you've built. Email the team and blog it.

Open elements on the same line

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
