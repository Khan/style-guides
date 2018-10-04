## JavaScript Style Guide

----

* [Syntax](#syntax)
  * [Naming](#naming)
  * [Naming private methods and properties](#naming-private-methods-and-properties)
  * [File names](#file-names)
  * [Indentation](#indentation)
  * [Braces](#braces)
  * [Ternaries](#ternaries)
  * [Spaces](#spaces)
  * [Line length](#line-length)
  * [Imports](#imports)
* [Comments and documentation](#comments-and-documentation)
  * [Inline Comments](#inline-comments)
  * [Top level file and class comments](#top-level-file-and-class-comments)
  * [Methods and properties comments](#methods-and-properties-comments)
* [Core language rules](#core-language-rules)
  * [Equality](#equality)
  * [Array and Object literals](#array-and-object-literals)
  * [Use a new var statement for each declaration](#use-a-new-var-statement-for-each-declaration)
  * [Avoid href="#" for JavaScript triggers](#avoid-href-for-javascript-triggers)
  * [Use modules, not global variables](#use-modules-not-global-variables)
* [ES6/7 rules](#es67-rules)
  * [Use =&gt; instead of bind(this)](#use--instead-of-bindthis)
  * [Use backticks for string interpolation](#use-backticks-for-string-interpolation)
  * [Use ES6 classes for React classes](#use-es6-classes-for-react-classes)
  * [Do not use async/await or generators](#do-not-use-asyncawait-or-generators)
  * [Do not use Set or Map](#do-not-use-set-or-map)
  * [Use let and const for new files; do not use var](#use-let-and-const-for-new-files-do-not-use-var)
* [Flow rules](#flow-rules)
    * [Enabling Flow](#enabling-flow)
    * [Be specific](#be-specific)
    * [Use the long form for arrays](#use-the-long-form-for-arrays)
* [Library rules](#library-rules)
  * [Use $ for jQuery](#use--for-jquery)
  * [Use `Promise` instead of `$.when()` or `$.Deferred()`](#use-promise-instead-of-when-or-deferred)
  * [Don't use Underscore](#dont-use-underscore)

----

This guide is adapted from the jQuery style guide.

----------
### Syntax

#### Naming

```js
ClassNamesLikeThis
methodNamesLikeThis
variableNamesLikeThis
parameterNamesLikeThis
propertyNamesLikeThis
SYMBOLIC_CONSTANTS_LIKE_THIS
```

When naming variables and properties referring to jQuery element
objects, prefix the name with `$`:

```js
function doSomethingFancy(selector) {
  var $elements = $(selector);
  ...
}
```

#### Naming private methods and properties

Private methods and properties (in files, classes, and namespaces)
should be named with a leading underscore.

While we do not currently use any compilers to enforce this, clients
of an API or class are expected to respect these conventions.

```js
function _PrivateClass() {
    // should not be instantiated outside of this file
}

function PublicClass(param) {
    this.publicMember = param;
    this._privateMember = new _PrivateClass();
}

var x = new _PrivateClass();  // OK - we’re in the same file.
var y = new PublicClass();    // OK
var z = y._privateMember;     // NOT OK!
```

Rationale: leading underscores for private methods and properties is
consistent with the styles used in numerous JavaScript libraries, many
of which we include in our code base (e.g. Backbone). It is also
consistent with our Python style guide, lowering the mental effort for
developers to switch between the two.

#### File names

```
file-names-like-this.js
template-names-like-this.handlebars
```

#### Indentation

Use 4-space indenting for all code. Do not use tabs.

Extra indentation should be used to clearly distinguish multiline
conditionals from the following block of code (similar to the PEP8
rule for Python code).

Yes:
```js
if (someReallyLongBooleanVariableIMeanReallyLong &&
        someOtherBoolean) {
    return "monkeys";
}
```

No:
```js
if (someReallyLongBooleanVariableIMeanReallyLong &&
    someOtherBoolean) {
    return "monkeys";
}
```

#### Braces

Braces should always be used on blocks.

`if/else/for/while/try` should always have braces and always go on
multiple lines, with the opening brace on the same line.

Yes:
```js
if (true) {
    blah();
}
```

`else/else if/catch` should go on the same line as the brace:

```js
if (blah) {
    baz();
} else {
    baz2();
}
```

No:
```js
if (true)
    blah();
```

#### Ternaries

Ideally, ternaries are written on a single line:

```js
const color = selected ? 'green' : 'orange'
```

If the ternary is too long to fit on a single line, within the
79-character limit, each fork of the ternary should be on its own
line.

Yes:
```js
const result = reallyVeryLengthConditional
    ? superLongComputationOfPositiveResult()
    : superLongComputationOfNegativeResult();

const style = selected
    ? {
        color: 'green',
        fontWeight: 'bold',
    }
    : {
        color: 'orange',
    }
```

No:
```js
// Unnecessarily split:
const color = selected
    ? 'green'
    : 'orange';

// Too long
const result = reallyVeryLengthConditional ? superLongComputationOfPositiveResult() : superLongComputationOfNegativeResult();

// Incorrectly split
const result = reallyVeryLengthConditional ?
    superLongComputationOfPositiveResult() :
    superLongComputationOfNegativeResult();
```

#### Spaces

Don't insert extra spaces between parens, brackets, or braces.

Yes:
```js
// Literals:
const fancyPants = pants.map((pant) => ({...pant, isFancy: true}));
const toCartesian = (r, theta) => [r * cos(theta), r * sin(theta)];

// Destructuring:
const {StyleSheet, css} = require('aphrodite');
const [x, y] = coordinates;


// Template strings:
const mission = `A ${price}, ${quality} education for ${clientele}.`;

// Parens:
if ((a === b) || (b === c)) {...}
```

No:
```js
// Literals:
const fancyPants = pants.map((pant) => ({ ...pant, isFancy: true }));
const toCartesian = (r, theta) => [ r * cos(theta), r * sin(theta) ];

// Destructuring:
const { StyleSheet, css } = require('aphrodite');
const [ x, y ] = coordinates;

// Template strings:
const mission = `A ${ price }, ${ quality } education for ${ clientele }.`;

// Parens:
if ( ( a === b ) || ( b === c ) ) {...}
```

#### Line length

Lines should not exceed 79 characters.  (This is called the "80
character rule," leaving 1 character for the newline.)

This is consistent with our Python style guide, which adheres to PEP8.

#### Imports

**Module System**
Prefer ES2015 imports (`import foo from 'foo'`) to CommonJS requires
(`const foo = require('foo')`). [Learn more about ES2015 imports](https://docs.google.com/document/d/12kT37eK7VusH8OK4vU9b7AmXN2G0Sw8g0c1LZ83-l-c/edit#heading=h.2j42ozjzl6id).

*NOTE*: Khan Academy employees working on the webapp need to be aware of an exception to this rule. The `i18n` module should still be required using CommonJS:

```js
const i18n = require("../shared-package/i18n.js");
```

The reasoning for this is that our build process that updates translations
looks for very specific markup, and ES2015 modules get transpiled by Babel
to an incompatible syntax.

**Grouping**
There should be 3 clusters of imports: third-party (aka vendor) libraries,
first-party libraries, and Flow types.

This approximately mirrors our [Python import style](python.md#import-style),
though there are no "system" imports in JavaScript.

"First party" code is anything we wrote whose primary source lives in
the repository its being used in.  Underscore is third party because
we didn't write it.  KaTeX is third party in webapp because even
though we wrote it, its primary sources lives in a different
repository.

**Named Imports**
Named imports should go on the same line, when possible. When 3+ named
imports are imported, break each named import onto its own line.

Yes:
```js
import $ from "jquery";
import Kicksend from "../../third_party/javascript-khansrc/mailcheck/mailcheck.js";
import React, {Component} from "react";
import _ from "underscore";

import APIActionResults from "../shared-package/api-action-results.js";
import Cookies from "../shared-package/cookies.js";
import DashboardActions from './datastores/dashboard-actions.js';
import HappySurvey from "../missions-package/happy-survey.jsx";
import UserMission from "../missions-package/user-mission.js";
import cookieStoreRenderer from "../shared-package/cookie-store.handlebars";
import mainThing, {
    firstOtherThing,
    secondOtherThing,
    thirdOtherThing,
} from '../somewhere';

import type {Thing} from '../somewhere-else';
```

No:
```js
// Avoid require()
var _ = require("underscore");

// Avoid grouping first- and third-party libraries
import React from 'react';
import APIActionResults from "../shared-package/api-action-results.js";

// Avoid breaking named imports onto multiple lines when possible.
import React, {
    Component
} from 'react';

```

------------------------------
### Comments and documentation

#### Inline Comments

Inline style comments should be of the `//` variety, not the `/* */`
variety.

#### Top level file and class comments

All files and classes should have JSDoc comments.

JSDoc can be parsed by a number of open source tools, and must be well-formed.

Syntax:
```js
/**
 * A JSDoc comment should begin with a slash and 2 asterisks.
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

Class comments should be used for every class, and give a description
along with appropriate type tags (see "Methods and properties"
comments for more information on types on the constructor).

```js
/**
 * Class making something fun and easy.
 *
 * @param {string} arg1 An argument that makes this more interesting.
 * @param {Array.<number>} arg2 List of numbers to be processed.
 */
function SomeFunClass(arg1, arg2) {
  // ...
}
```

#### Methods and properties comments

All non-trivial methods and properties should also have JSDoc comments.

Type annotations are strongly encouraged; if there is even a slight
chance that the type will be ambiguous to future readers, put in a
type annotation.

Type annotations are based on the ES4/JS2 type system, and are
documented in the [Google JavaScript style
guide](https://google.github.io/styleguide/javascriptguide.xml).

`@param` and `@return` type annotations that have comments that do not
fit on one line wrap to the next line and indent 4 spaces.

Example:

```js
/**
 * A UI component allows users to select badges from their full list
 * of earned badges, displaying them in a container.
 * Expects a Badges.BadgeList as a model.
 */
Badges.DisplayCase = Backbone.View.extend({
    /**
     * Whether or not this is currently in edit mode and the full
     * badge list is visible.
     */
    editing: false,

    /**
     * The full user badge list available to pick from when in edit mode.
     * @type {Badges.UserBadgeList}
     */
    fullBadgeList: null,

    /**
     * Enters "edit mode" where badges can be added/removed.
     * @param {number=} index Optional index of the slot in the display-case
     *     to be edited. Defaults to the first available slot, or if none
     *     are available, the last used slot.
     * @return {Badges.DisplayCase} This same instance so calls can be
     *     chained.
     */
    edit: function(index) {
    …
    },
   ...
};
```

-----------------------
### Core language rules

#### Equality

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

#### Array and Object literals

Always use `[]` and `{}` style literals to initialize arrays and
objects, not the `Array` and `Object` constructors.

Array constructors are error-prone due to their arguments: `new
Array(3)` yields `[undefined, undefined, undefined]`, not `[3]`.

To avoid these kinds of weird cases, always use the more readable
array literal.

Object constructors don't have the same problems, but follow the same
rule for consistency with arrays.  Plus, `{}` is more readable.

#### Use a new var statement for each declaration

Yes:
```js
var a = "foo";
var b = a + "bar";
var c = fn(a, b);
```

No:
```js
var a = "foo",
    b = a + "bar",
    c = fn(a, b);
```

A single var statement is bad because:

* If you forget a comma, you just made a global
* It originated when people wanted to save bytes, but we have a minifier
* It makes line-based diffs/editing messier
* It encourages C89-style declarations at the top of scope, preventing
  you from only declaring vars before first use, the latter preferable
  as it conveys intended scope to the reader

#### Avoid `href="#"` for JavaScript triggers

When you want a link-like thing rather than a button to trigger a
JavaScript operation, rather than going to a new address.

Here's a discussion on Stack Overflow about options:
http://stackoverflow.com/questions/134845/href-tag-for-javascript-links-or-javascriptvoid0


Yes:
```js
<a href="javascript:void 0">Flag</a>
```

No:
```js
<a href="#">Flag</a>
```

#### Use modules, not global variables

In most of our major JavaScript repositories (webapp, perseus,
khan-exercises), we use some form of module system like
[RequireJS](http://requirejs.org/) or
[browserify](http://browserify.org/), or in the case of webapp our own
home built thing that works similarly to browserify.

In all of these cases, there are mechanisms for an explicit
import/export mechanism rather than using global variables to export
functionality.

Yes:
```js
var Jungle = {
    welcome: function() {
        // ...
    },
    haveFever: function() {
        // ...
    }
};

module.exports = Jungle;
```

No:
```js
window.Jungle = {
    welcome: function() {
        // ...
    },
    haveFever: function() {
        // ...
    }
};
```

**NO**:
```js
window.welcome = function() {
   // ...
};

window.haveFever = function() {
   // ...
};
```

You can export multiple objects in one file, but consider if it
wouldn't be better to split up the file to maintain one export per file.

---------------
### ES6/7 rules

Several of our supported browsers support only ES5 natively.  We use
polyfills to emulate [some -- but not all -- ES6 and ES7 language
features](https://docs.google.com/spreadsheets/d/12mF99oCpERzLKS07wPPV3GiISUa8bPkKveuHsESDYHU/edit#gid=0)
so they run on ES5-capable browsers.

In some cases, we do not yet allow a new language feature, if it's
expensive to polyfill.  In others, we require using the newer language
feature and avoiding the old:

| Construct | Use...                                | ...instead of |
| --------- | ------------------------------------- | ---------------------- |
| backticks | `` `http://${host}/${path}` `` | `"http://" + host + "/" + path` |
| destructuring | `var {x, y} = a;` | `var x = a.x; var y = a.y;` |
| fat arrow | `foo(() => {...})` | `foo(function() {...}.bind(this))` |
| let/const | `let a = 1; const b = "4EVAH"; a++;` | `var a = 1; var b = "4EVAH"; a++;` |
| includes | `array.includes(item)` | `array.indexOf(item) !== -1` |
| for/of | `for (const [key, value] of Object.entries(obj)) {...}` | `_.each(obj, function(value, key) {...})` |
| spread | `{...a, ...b, c: d}` | `_.extend({}, a, b, {c: d})` |
| rest params | `function(bar, ...args) {foo(...args);}` | `function(bar) {var args = Array.prototype.slice.call(arguments, 1); foo.apply(null, args);}` |

#### Use `=>` instead of `bind(this)`

Arrow functions are easier to read (and with Babel, more efficient)
than calling `bind` manually.

#### Use rest params instead of `arguments`

The magic `arguments` variable has some odd quirks. It's simpler to
use rest params like `(...args) => foo(args)`.

#### Use backticks for string interpolation

`+` is not forbidden, but backticks are encouraged!

#### Use ES6 classes for React classes

See [React Use ES6 classes](react.md#use-es6-classes) for details.

For classes outside of React -- which should actually be pretty rare -- you
should also use ES6 classes.  Some things to keep in mind when using ES6
classes:

- Use `static` properties instead of adding propertiers to the class object
  after defining the class.
- Use `extend` syntax for inheritance.


#### Do not use `async`/`await` or generators

This is because the polyfill for these constructs generates very large
code.

This rule may change once all our supported browsers support ES6
natively.

#### Do not use `Set` or `Map`

The polyfills for these, though not huge, are large enough it's not
worth the (small) benefit of using these classes for hashtables
instead of just using `object`.

This rule may change if strong enough support for these types is
evinced.

#### Use `let` and `const` for new files; do not use `var`

`let` is superior to `var`, so prefer it for new code.


-----------------
### Flow rules

Flow is a type-checker that runs at compile-time to catch issues and
prevent bugs. It should be enabled and used for all new JS.

[Read the full documentation](https://docs.google.com/document/d/1PQngtANm48R1d90WFJS3t1gixGeiShHgq_ef7j3hYYE/edit) for more information on setup, usage, and troubleshooting.

#### Use the long form for arrays.

The long form (`Array<X>`) should be preferred over the short-hand form (`X[]`).This is to avoid ambiguity when using [Maybe types](https://flow.org/en/docs/types/maybe/).

Good:
```
const arr: Array<number> = [1, 2, 3];
```

Bad:
```
const arr: number[] = [1, 2, 3];
```

-----------------
### Library rules

#### Use `$` for jQuery

We use `$` as the jQuery identifier, as opposed to typing out `jQuery`
in full.

Yes:
```js
$(".some-class span").hide();
```

No:
```js
jQuery(".some-class span").hide();
```

### Use `Promise` instead of `$.when()` or `$.Deferred()`

We use the [ES6 Promise polyfill](https://github.com/jakearchibald/es6-promise) on our site. In general the [MDN Polyfill Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) is extremely helpful in determining how to best use the `Promise` object.

Moving from `$.when()` to the `Promise` object is quite easy.

Instead of... | Use...
--------------|-------------------
`$.when()` (no arguments) | `Promise.resolve()`
`$.when.apply($, arrayOfPromises)` | `Promise.all(arrayOfPromises)`
`$.when(promise)` | `promise` (`$.when` just returns the promise)

Note that if you're calling `Promise.all()` on an array of promises that the result to be passed to the `.then()` callback will be an array of all the results from each of the promises in the array. More [details on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all).

Moving from `$.Deferred` to `Promise` can be a bit trickier, it all depends upon how you were originally using it. The biggest difference is that in order to mark a `Promise` as resolved you must execute the callback function that was passed in to the `new Promise()` constructor. This is easy if you're tracking progress on some asynchronous operation:

```
var promise = new Promise((resolve) => {
    $.ajax(...).then(resolve);
});
```

However it gets a bit trickier when you want to resolve the promise at some later time, outside the scope of the instantiation function. Since there is no `.resolve()` method, as was available on `$.Deferred()` objects, a common pattern may look like this:

```
var markResolved;
var promise = new Promise((resolve) => {
    markResolved = resolve;
});

// later in your code...
if (markResolved) {
    markResolved();
}
```

It's also important to note that Promises do not throw exceptions. If you wish to catch an exception you must explicitly attach a `.catch()` callback to it and listen for the error.

#### Use `khanFetch()` instead of `$.ajax`/`$.get`/`$.post`/`$.getJSON`

We now provide a polyfill for the [`fetch()` method](https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch/fetch). We should use this for all Ajax-style requests. We wrote a wrapper around `fetch()` which adds in some Khan Academy-specific logic (such as adding cachebusting parameters and handling API Action Results). The interface to `khanFetch()` is exactly the same as the normal `fetch()` function. You can use it like:

```
const {khanFetch} = require("./path-to-shared-package/khan-fetch.js");
```

##### `$.get()`

Get some textual data:

```
khanFetch("/some.json")
    .then((response) => response.text())
    .then((text) => {/* Use the textual data... */})
    .catch((err) => {/* Handle server error... */});
```

Get some JSON data (same use case as `$.getJSON`).

```
khanFetch("/some.json")
    .then((response) => response.json())
    .then((json) => {/* Use the JSON data... */})
    .catch((err) => {/* Handle server error... */});
```

##### `$.post()`

POSTing JSON to an API endpoint and getting JSON back.

```
khanFetch("/api/some/endpoint", {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
    },
    body: JSON.stringify(myJSONObject),
})
    .then((response) => response.json())
    .then((json) => {/* Use the JSON data... */})
    .catch((err) => {/* Handle server error... */});
```

POSTing form data to an API andpoint and getting JSON back. This is the default encoding that `$.post` used, so this should be used in place of `$.post(url, data)`. To make this use case easier, we wrote a function called `formUrlencode` which automatically encodes the form data and appends the `"Content-Type": "application/x-www-form-urlencoded;charset=UTF-8"` header.
```
const {khanFetch, formUrlencode} = require("./path-to-shared-package/khan-fetch.js");

khanFetch("/api/some/endpoint", {
    method: "POST",
    ...formUrlencode({
        key1: "value1",
        key2: 2,
    }),
})
    .then((response) => response.json())
    .then((json) => {/* Use the JSON data... */})
    .catch((err) => {/* Handle server error... */});
```

#### Don't use Underscore

We use ES6/7 which includes many of the features of Underscore.js! Using Underscore should be avoided in favor of these native language features.

There are a couple of methods that are sufficiently complicated and don't have a direct equivalent so instead we have a [custom-built](https://lodash.com/custom-builds) copy of [lodash](https://lodash.com/) containing only those specific methods. You can find this file at: `third_party/javascript-khansrc/lodash/lodash.js` along with instructions on how to build it and exactly what methods are included.

What follows is a method-by-method set of equivalents for what Underscore provides and what you could be using in ES6/7 instead:

Method | Use...                                | ...instead of
--------- | ------------------------------------- | ----------------------
bind | `fn.bind(someObj, args)` | `_.bind(fn, someObj, args)`
bind | `(a, b) => {...}` <sup>[1](#u1)</sup> | `_.bind(function(a, b) {...}, this)`
bindAll | `obj.method = obj.method.bind(someObj);` <sup>[2](#u2)</sup> | `_.bindAll(someObj, "method")`
clone | No alternative at the moment! <sup>[3](#u3)</sup> |
debounce | Our custom lodash build. |
defer | `setTimeout(fn, 0);` | `_.defer(fn);`
delay | `setTimeout(fn, 2000);` | `_.delay(fn, 2000);`
each (array) | `array.forEach((val, i) => {})` | `_.each(array, (val, i) => {})`
each (array) | `for (const val of array) {}` | `_.each(array, fn)`
each (object) | `for (const [key, val] of Object.entries(obj)) {}` | `_.each(obj, fn)`
extend (new) | `{...options, prop: 1}` | `_.extend({}, options, {prop: 1})`
extend (assign) | `Object.assign(json, this.model.toJSON())` | `_.extend(json, this.model.toJSON())`
filter | `array.filter(checkFn)` | `_.filter(array, checkFn)`
has (array) | `array.includes(value)` | `_.has(array, value)`
has (object) | `obj.hasOwnProperty(value)` <sup>[4](#u4)</sup> | `_.has(obj, value)`
isArray | `Array.isArray(someObj)` | `_.isArray(someObj)`
isFunction | `typeof fn === "function"` | `_.isFunction(fn)`
isNull | `obj === null` | `_.isNull(obj)`
isNumber | `typeof obj === "number"` | `_.isNumber(obj)`
isString | `typeof obj === "string"` | `_.isString(obj)`
keys | `Object.keys(obj)` | `_.keys(obj)`
last | `someArray[someArray.length - 1]` <sup>[5](#u5)</sup> | `_.last(someArray)`
map | `array.map(mapFn)` | `_.map(array, mapFn)`
map (object) | `Object.entries(obj).map(([key, val]) => mapFn(val, key))` | `_.map(obj, mapFn)`
max | `Math.max(...array)` | `_.max(array)`
object | <pre>Object.entries(obj).reduce(<br>(result, [key, val]) => {<br>&nbsp;&nbsp;&nbsp;&nbsp;result[key] = value;<br>&nbsp;&nbsp;&nbsp;&nbsp;return result;<br>})</pre> | <pre>\_.object(\_.map(obj, (val, key) => {<br>&nbsp;&nbsp;&nbsp;&nbsp;return [key, value];<br>})</pre>
omit (array) | `array.filter(prop => !props.includes(prop))` | `_.omit(array, props)`
omit (object) | <pre>Object.keys(obj).reduce((result, prop) => {<br>&nbsp;&nbsp;&nbsp;&nbsp;if (!props.includes(prop)) {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result[prop] = attrs[prop];<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>}, {})</pre> | `_.omit(obj, props)`
once | `$(...).one("click", ...)` | `$(...).on("click", _.once(...))`
once | <pre>{<br>&nbsp;&nbsp;&nbsp;&nbsp;method: () => {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if (this._initDone) {return;}<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;this._initDone = true;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>}</pre>| `{method: _.once(() => {...})}`</pre>
once | <pre>var getResult = () => {<br>&nbsp;&nbsp;&nbsp;&nbsp;let val = $.when(...).then(...);<br>&nbsp;&nbsp;&nbsp;&nbsp;getResult = () => val;<br>&nbsp;&nbsp;&nbsp;&nbsp;return val;<br>};</pre> | <pre>var getResult = _.once(() => {<br>&nbsp;&nbsp;&nbsp;&nbsp;return $.when(...).then(...);<br>});</pre>
range | `Array(n).fill().map((_, i) => i * i)` | `_.range(0, n).map(i => i * i)`
sortBy | `result = result.sort((a, b) => a.prop - b.prop)` | `_.sortBy(result, "prop")`
sortedIndex | Our custom lodash build. |
times | `Array(n).fill().map((_, i) => i * i)` | `_.times(n, i => i * i)`
throttle | Our custom lodash build. |
values | `Object.values(obj)` | `_.values(obj)`

1. To be used when you're creating a function and immediately binding its context to `this`. <b id="u1"></b>
2. Or use a loop if binding multiple methods. <b id="u2"></b>
3. No alternative at the moment! If you need it then you should add it to the compiled version of lodash and then update this guide to mention that it now exists! <b id="u3"></b>
4. While we recommend using `obj.hasOwnProperty(prop)` it is possible that the object could have a method named `hasOwnProperty` that does something else, causing this call to break. The likelihood of this happening is extremely slim - but if you're developing something that you wish to work absolutely everywhere you may want to do something like `Object.prototype.hasOwnProperty.call(obj, prop)`. <b id="u4"></b>
5. If you don't care about destructively modifying the array, you can also use `someArray.pop()``. <b id="u5"></b>
