# JavaScript Style Guide

----------

## Linting, Formatting, and TypeChecking

We use [Prettier](https://prettier.io/) for formatting our code. You shouldn't need to stress about how your code is formatted, just let Prettier do its job.

We also use [ESLint](https://eslint.org/) to enforce a number of rules (some of which are mentioned here, some of which are clarified by other documentation).

We highly recommend that you don't ignore ESLint errors, nor ignore TypeScript errors (as both of
those could be preventing serious issues from appearing).

## Syntax

### Naming

```js
ClassNamesLikeThis
ComponentNamesLikeThis
methodNamesLikeThis
variableNamesLikeThis
parameterNamesLikeThis
propertyNamesLikeThis
SYMBOLIC_CONSTANTS_LIKE_THIS
```

### File names

We exclusively use `.ts` and `.tsx` file extensions, as we use TypeScript.

Files should be formatted like so:

```sh
file-names-like-this.tsx
test-files-like-this.test.tsx
test-files-like-this.stories.tsx
```

### Imports

**Module System**
We exclusively use ES2015 imports (`import foo from 'foo'` or `import('foo')`).

Don't worry about the import order, we prefer using a tool to handle this for us, like:

* [Prettier Sort Imports Plugin](https://github.com/IanVS/prettier-plugin-sort-imports)
* [ESLint Sort Imports](https://eslint.org/docs/latest/rules/sort-imports)

----------

## Comments and documentation

### Inline Comments

Inline comments should be of the `//` variety, not the `/* */`
variety.

### Top level file, class, and component comments

All files, classes, and components should have a comment (if a file only has a single class or component, then there's no need for an extra comment for the whole file).

Ideally, we should use a JSDoc-style comment, so that IDEs can pick them up for display. You
don't, necessarily, have to worry about documenting each argument (or prop), you can rely upon
TypeScript for that, instead.

Syntax:

```js
/**
 * A comment should begin with a slash and 2 asterisks.
 */
```

Top-level (top-of-file) comments are designed to orient readers
unfamiliar with the code to what is in this file and any other
disclaimers clients of the code should be given.  It should provide a
description of the file's contents and any dependencies or
compatibility information.  As an example:

```js
/**
 * Various components to handle management of lists of coaches for
 * the profile page.
 *
 * These utilities were not written to be a general purpose utility
 * for the entire code base, but has been optimized with the
 * assumption that the Profile namespace is fully loaded.
 */
```

### Methods and properties comments

All non-trivial methods and properties should also have a comment.

----------

## Core language rules

### Equality

Prefer `===` (strict equality) to `==` due to the [numerous oddities
related to JavaScript's type coercion](https://javascriptweblog.wordpress.com/2011/02/07/truth-equality-and-javascript/).

The only valid use of `==` is for comparing against null and undefined
at the same time:

```js
// Check null and undefined, but distinguish between other falsey values
if (someVariable == null) {
```

Though you will often want to just check against falsey values, and
can just say `if (!someVariable) {...}`.

### Array and Object literals

Always use `[]` and `{}` style literals to initialize arrays and
objects, not the `Array` and `Object` constructors.

Array constructors are error-prone due to their arguments: `new
Array(3)` yields `[undefined, undefined, undefined]`, not `[3]`.

To avoid these kinds of weird cases, always use the more readable
array literal.

Object constructors don't have the same problems, but follow the same
rule for consistency with arrays.  Plus, `{}` is more readable.

### Use a new statement for each declaration

Yes:

```js
const a = "foo";
const b = a + "bar";
const c = fn(a, b);
```

No:

```js
const a = "foo",
    b = a + "bar",
    c = fn(a, b);
```

A single `const` statement is bad because:

* If you forget a comma, you just made a global
* It originated when people wanted to save bytes, but we have a minifier
* It makes line-based diffs/editing messier
* It encourages C89-style declarations at the top of scope, preventing
  you from only declaring vars before first use, the latter preferable
  as it conveys intended scope to the reader

### Use modules, not global variables

In all of our major JavaScript repositories, we use some form of module system.

In all of these cases, there are mechanisms for an explicit
import/export mechanism rather than using global variables to export
functionality.

Yes:

```js
export default class Cat {
    pet() {
        // ...
    }
    meow() {
        // ...
    }
}
```

No:

```js
window.Cat = {
    welcome() {
        // ...
    },
    haveFever() {
        // ...
    }
};
```

You can export multiple objects in one file, but consider if it
wouldn't be better to split up the file to maintain one export per file.

----------

## Library rules

### Use `khanFetch()` instead of `fetch()`

We now provide a polyfill for the [`fetch()` method](https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch/fetch). We should use this for all network requests. We wrote a wrapper around `fetch()` which adds in some Khan Academy-specific logic (such as adding cachebusting parameters and handling auth correctly). The interface to `khanFetch()` is exactly the same as the normal `fetch()` function. You can use it like:

```js
import {khanFetch} from "~/data-access/khan-fetch.js";
```

### Prefer to not use Underscore or Lodash

We use modern ECMAScript which includes many of the features of Underscore or Lodash! Using these libraries should be avoided in favor of these native language features.

There are a couple of methods that are sufficiently complicated and don't have a direct equivalent so instead we import a couple `lodash` methods directly, you can see which ones exist in the `package.json` file.
