# Inline styles: best practices

So you want to write some fancy schmancy React components, but you don't want
to deal with all of the pain of writing corresponding LESS/CSS and adding that
to a CSS bundle? Then this page is for you!

Note that the state of inline styles is being figured out! It isn't currently
clear what the absolute best way to do things is yet, but we've come up with a
library approach that solves most (if not all) of the problems we've found with
plain inline styles. However, this is still a work in progress, so if you are
trying to do something and it isn't working, please talk to me (Emily) and we
can figure out a good way to do it! (in the meantime, you can always fall back
to CSS if things are too hard, but I'd like to hear what went wrong to fuel
future decisions).


## How do I write inline styles in my React components?

We've written a library called [aphrodite](https://github.com/khan/aphrodite)
to help write inline styles. It solves a lot of problems with inline styles,
like pseudo-selectors, media queries, and precedence, and also (in the future)
will do things like vendor prefixing and rtl conversions for you automatically.
To use it, add this require line at the top of your file:

```jsx
const { StyleSheet, css } = require("aphrodite");
```

To add inline styles to your components, define a map of styles in the
top-level scope of your file, wrapped in a call to `StyleSheet.create`:

```jsx
const styles = StyleSheet.create({
    mainStyle: {
        border: "1px solid black",
        fontSize: 14
    },

    smallSize: {
        fontSize: 10
    }
});
```

Then, when you want to add these styles to a component, call `css()` on the
corresponding style, and set the result to `className`:

```jsx
render: function() {
    return <div className={css(styles.mainStyle)}>
        ...
    </div>;
}
```

We follow the React inline style guide in terms of properties. In particular,
number dimensions will be turned into pixels (when appropriate), and property
names should be `camelCased` instead of `kebab-cased`. (i.e. the `fontSize: 14`
above is equivalent to `font-size: 14px` in CSS). All other values should be in
strings. See more information at the
[React guide to Inline Styles](https://facebook.github.io/react/tips/inline-styles.html).

If you want to combine multiple styles, simply pass them all as arguments to
`css()`, where later ones will override earlier ones:

```jsx
<div className={css(styles.mainStyle, styles.smallSize)}>
    ...
</div>;
```

`css()` ignores falsey values, so you can dynamically turn classes on and off
using `&&`:

```jsx
<div className={css(styles.mainStyle, this.props.small && styles.smallSize)}>
    ...
</div>;
```

For dynamic styles (like styles resulting from an animation, or calculated
sizes), pass them straight to your React components:

```jsx
render: function() {
    const offset = (this.props.time / this.props.totalTime) * totalOffset;
    const style = {
        left: offset
    };
    return <div style={style}>
        ...
    </div>;
}
```

Note: if your dynamic styles need to override your `css()` styles, you need to
add `" !important"` to them, because aphrodite automatically adds `!important`
to all of its styles. This is subject to change.

### Caveats

The aphrodite library is a work in progress. If you encounter any problems,
please let myself (Emily) or Jamie Wong know!

There are a couple of important things to note when using this library. In
particular:

 - **The objects in your `styles` variable are not plain objects**: You cannot
   use plain merges or make modifications to the style objects once you run
   them through `StyleSheet.create`. aphrodite munges the objects in them on
   purpose, so we can do a better job when generating css.
 - **All of your styles should be combined with `css` at the same time**: The
   `css` function takes a list of styles and combines, producing a CSS class
   that it inserts into the DOM, as well as a class name associated with that.
   In particular, it handles later styles overriding earlier ones (so with
   `css(a, b)`, the `b` styles will override the `a` styles). _If you call
   `css()` multiple times and then combine the resulting class names together
   manually, this no longer works._

   In particular, if you want to pass styles down from a parent to a child, you
   should pass the styles directly, and then child should call `css` on them,
   instead of you calling `css` on them and passing down a `className`:
   
   YES:

    ```jsx
    // parent:
    
    <ChildComponent styles={[styles.a, styles.b]} />

    // child:
    
    <div className={css(styles.c, ...this.props.styles)}>...</div>
    ```

   NO:

    ```jsx
    // parent:
    
    <ChildComponent className={css(styles.a, styles.b)} />
    
    // child:
    
    <div className={this.props.className + “ ” + css(...)}>...</div>
    ```

## How do I nest my styles? What if I want to use different styles based on different environments?

Fortunately, since inline styles are added to the HTML itself and aren't global
in any way, you don't need to deal with the funky specificity rules of CSS,
just write your styles and go!

If you want your components to appear different in different environments, you
can simply add boolean props that toggle different styles on or off.

## What if I want to use other CSS features, like `:hover`, or media queries?

Using aphrodite makes these things easy! You can simply add extra keys to your
styles that tell aphrodite to make these things work.

Note that the syntax for these are just like plain CSS, and aren't subject to
the camel casing or other rules for other CSS properties.

For `:hover`, or any other pseudo-selector:

```jsx
const styles = StyleSheet.create({
    baseStyle: {
        backgroundColor: "grey",

        ":hover": {
            backgroundColor: "white"
        }
    }
});
```

For media queries:

```jsx
const styles = StyleSheet.create({
    baseStyle: {
        width: 500,

        "@media (max-width: 600px)": {
            width: 200
        }
    }
});
```

## What about pseudo-elements, like ::before?

(Side note: yes, CSS3 says you're supposed to use two colons before the word to
distinguish them from pseudo-selectors)

You can simulate `::before` and `::after` by just adding more DOM elements to
your components.

As you, a CSS wizard, have probably figured out, pseudo-elements don't actually
do anything special, they just add more DOM elements to your page and save you
from repeating the same HTML in certain places. Thankfully, React is good at
making reusable components that save you from the same tedium, and you don't
even have to deal with the annoying limitations that pseudo-elements have! It's
even better for i18n because we don't currently have a way to translate content
properties in our CSS.

For example, if you previously had something like this:

```jsx
return <div class="example"></div>;
```
```css
.example::after {
    content: "Hi!";
}
```

you can instead just add an extra element:

```jsx
return <div>
    <div>Hi!</div>
</div>;
```

## How do I use my favorite variables from our LESS code?

All of the variables from
[stylesheets/shared-package/variables.less](https://github.com/Khan/webapp/blob/master/stylesheets/shared-package/variables.less)
have been convered into JavaScript variables that you can use in your styles.
They live in
[javascript/react-package/style-constants.js](https://github.com/Khan/webapp/tree/master/javascript/react-package/style-constants.js).
Using them is as simple as requiring that file!
