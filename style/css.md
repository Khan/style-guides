# CSS

Welcome to the CSS style guide. This guide borrows heavily from the [Github style guide](https://github.com/styleguide/css), so thanks to them!

 

## Coding Style

- Use a four space indent.
- Put spaces after : in property declarations.
- Spaces before braces: put spaces before { in rule declarations. 
- Separate selector and property declarations with a new line
- Separate rules with a new line
- End properties declarations with a ;
- Alphabetize properties within rule declarations
- Properties with values of 0 should *not* have units
- Use lowercase hex color codes #fff unless using rgba.
- Use /* */ for comment blocks (instead of //).
- Use a maximum line length of 80 characters (rationale: looking at files side-by-side)
- Use dashes in selectors instead of underscores (.my-class, not .my_class)

Here is good example syntax:

```css
/* This is a good example! */
.styleguide-format,
.other-format,
.third-format {
    background: rgba(0,0,0,0.5);
    border: 1px solid #0f0;
    color: #000;
    margin: 0;
}

.next-rule {
    color: #000;
}
```

## LESS Style

If you aren't familiar with LESS, [check out the documentation](http://lesscss.org/). 

- Any `$variable` or `@mixin` that is used in more than one file should be put in put in the appropriate package or, if used across packages, in variables.less or mixins.less in shared-package. Others should be put at the top of the file where they're used.
- As a rule of thumb, don't nest further than 3 levels deep. If you find yourself going further, think about reorganizing your rules (either the specificity needed, or the layout of the nesting).
- If you are creating mixins that don't take parameters in a file that is going to be imported elsewhere (e.g. `shared-package/mixins.less`), include an empty parameter list to guard against the class being output each time the file is imported.

```less
/* This is a good example of rule nesting with LESS */
.third-format {
    background: rgba(0,0,0,0.5);
    border: 1px solid #0f0;
    color: #000;
    margin: 0;

    .next-rule {
    /* This outputs: .third-format .next-rule {...} */
        color: #fff;
    }
    
    &.next-rule {
    /* This outputs: .third-format.next-rule {...} */
        color: #aaa;
    }

    > .next-rule {
    /* This outputs: .third-format > .next-rule {...} */
        color: red;
    }
}

.parameterless-mixin-in-an-imported-less-file() {
/* The empty parameter list allows the mixin to be used without
 * params and prevents the LESS compiler from spitting it out
 * each time the file is imported
 */
    background: rgba(0,0,0,0.5);}
```

## Pixels vs. Ems

Use `px` for `font-size`, because it offers absolute control over text. **You should almost never have to define a font-size for anything**. If you feel the need to do it, stop yourself and look through existing shared styles for an applicable class.

## Specificity (classes vs. ids)

Elements that occur **exactly once** inside a page should use IDs, otherwise, use classes. When in doubt, use a class name.

- **Good** candidates for ids: header, footer, modal popups.
- **Bad** candidates for ids: navigation, item listings, item view pages (ex: issue view).

When styling a component, start with an element + class namespace (prefer class names over ids), prefer direct descendant selectors by default, and use as little specificity as possible. Here is a good example:

```html
<ul class="category-list">
  <li class="item">Category 1</li>
  <li class="item">Category 2</li>
  <li class="item">Category 3</li>
</ul>
```

```less
ul.category-list { // element + class namespace

  > .item { // direct descendant selector > for list items
    list-style-type: disc;
  }
}
```

## CSS Specificity guidelines

- If you must use an id selector (`#selector`) make sure that you have no more than one in your rule declaration. A rule like `#header .search #quicksearch { ... }` is considered harmful. Curious why? Check out this primer on [CSS specificity](http://css-tricks.com/specifics-on-css-specificity/).
- When modifying an existing element for a specific use, try to use specific class names. Instead of `.listings-layout.bigger` use rules like `.listings-layout.listings-bigger`. Think about ack/greping your code in the future.
- Key (rightmost) Selectors should be as specific as possible. For example `a.navigation-link` instead of `#navigation-links a`. This has [performance implications](http://www.stevesouders.com/blog/2009/06/18/simplifying-css-selectors/).

## File Structure

If you are adding LESS files to a package you are working on, add only a single LESS file to the list in packages.py that imports all of the required files like this:

```less
@import "../shared-package/variables.less";
@import "../shared-package/mixins.less";
@import "my-new-package-file.less";
@import "my-second-package-file.less";
```

With this approach, all imports are done in this file. This prevents duplications in the compiled CSS that are caused by cascading imports of the same file.
