# Java

At Khan Academy, we only use Java for Android and, as such, follow the [Android Code Style Guidelines for Contributors](https://source.android.com/source/code-style.html). Note that those rules themselves extend the [standard Java language style guide as of 1999](http://www.oracle.com/technetwork/java/javase/documentation/codeconventions-141855.html). The rest of this document describes additions and clarifications to those rules that we follow at Khan Academy.

To import these settings for Android Studio, copy [the config file](/configs/KhanAcademyAndroid.xml) into `$HOME/Library/Preferences/AndroidStudio/codestyles/`.

## Imports

Import statements are divided into the following groups, in this order, with each group separated by a blank line:

1. All static imports in a single group
2. org.khanacademy imports
3. All other non-Android third-party imports, in ASCII sort order (for example: com, junit, org, sun)
4. android imports
5. java imports
6. javax imports

Imports are not subject to the 100 column limit rule; they should never be wrapped.

We prefer using static imports when using the following:

- `com.google.common.base.Preconditions.*` (e.g. `checkNotNull`)
- `org.junit.Assert.*` (e.g. `assertEquals`)


## No per-file boilerplate

We do not include copyright or `@author` or `Created by` boilerplate at the top of files. The first line of the file is typically the `package` statement.

Package statements are not subject to the 100 column limit rule; they should never be wrapped.


## Use your best judgement for Javadoc

We don't strictly require Javadoc for _all_ public methods, though it's _strongly_ encouraged for non-trivial ones.


## Use `@Override` where possible

In addition to requiring the `@Override` tag when a class overrides the declration or implementation from a superclass, we also require it to be used when classes implement a method from an implemented interface.


## No single-line if statements.

Bad:
```java

if (foo) {bar();}   // No single-line if statements

// ...and braces are always required
if (foo)
    bar();
```

Good:
```java
if (foo) {
    bar();
}
```


## No C-style array declarations

The square brackets form part of the _type_, not the variable.

Good: `String[] args`
Bad: `String args[]`

