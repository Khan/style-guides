# React style guide

----

> Follow the normal [JavaScript style guide](javascript.md). We generally try to follow the official [React tutorials and guides](https://react.dev/reference/react). Please default to them as a source-of-truth.

----

## Syntax

### Prefer to export a single React component

Every .tsx file should ideally export a single React component.
This is for testability and composition; it's easier to write Storybook stories and
it's clearer where the breaks in functionality are.

Note that the file can still define multiple components, it just shouldn't export
more than one.
