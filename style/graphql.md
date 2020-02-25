# Graphql

## Docstrings

When writing a schema definition file (`graphql.schema`), every type
and field should be documented.  The schema file is the main source of
documentation for users who are crafting graphql queries and need to
know what fields to ask for.  We use triple-quotes for documentation,
since they are automatically picked up by GraphiQL and other graphql
UIs.

### Example

```graphql
"""
Summary of what this type is for; must fit on one line.

After the blank line, you can spend as much space talking
about this type as you'd like.
"""
interface Book {
   """Summary of this field; must fit on one line.
   If you need more space for discussing this field,
   you can use more lines, but don't use a blank line
   here.  If applicable, give examples of valid values.
   Examples: Sal Khan, Artemis Fowl
   """
   author: String!

   # Normal (`#`-prefixed) comments are not exposed via GraphiQL.
   # They are appropriate for text that is useful for code-authors but
   # not code-users.
   """This field needs only one line to describe it."""
   publishYear: Number

   # We don't need a docstring here; the documentation is inside @deprecated.
   published: Boolean @deprecated(
       reason: "Use publishYear instead.")

   """The company that published the book.
   Valid values: Random House, Macmillan, University of California Press
   """"
   publisher: String
}

type Hardcover implements Book {
   # These fields are defined on the Book interface.
   author: String!
   publishYear: Number
   published: Boolean

   """The number of pages the hardcover version of this book has."""
   numPages: Number!
}
```

### Commenting types

Top level datatypes -- `Type`, `Interface`, `Union`, etc -- have a
line of `"""` by itself, followed by a one-line summary of the
datatype, followed by a blank line, followed by other text describing
the type, ending with a `"""` on its own line.

This comment style helps point out, visually, where one type ends and
a new type begins.

### Commenting fields

Almost all fields within a `Type` should have a docstring.  This
docstring should start with a `"""` followed by a one-line summary of
the field, all on one line.  If more documentation is needed, text can
follow on the next line.  There should never be a blank line in a
field-docstring.

Unless it's entirely obvious, give an example of possible value or two
via the line: `Example: ..., ....`.  Do not put quotes around the
examples unless it's needed for clarity.

When appropriate -- especially when an object has type "String" but is
semantically an enum -- include a line `Valid values: ..., ...` that
lists all valid values that the field could have.

If the docstring is one line long, the trailing `"""` should go on
that one line.  Otherwise, the trailing `"""` should go on a line of
its own.

There should always be a blank line between a field and the docstring
that introduces the next field.

Always use triple-quotes, never a single quote, even for one-line
docstrings.

This comment style makes it easy to find field definitions within a
type.  In particular, blank lines are *only* used to separate fields.
There are never blank lines within the text pertaining to a field.

### Commenting deprecated fields

If *new code* should not query on a particular field, mark that fact
with a `@deprecated` tag.  The `reason` associated with that tag
should say what to use instead.  A docstring is not needed for this
field, since no new code should be using it.  (It's not forbidden to
have a docstring, though.)

You can mark a field `@deprecated` even if existing graphql queries
use it.  The `@deprecated` tag is meant only as a signal for people
writing *new* graphql queries, that they should not be using this
field anymore.

### Commenting interfaces

If you write an interface, you should document all the fields of that
interface, just like any other type.

When you define a type that implements that interface, you will need
to copy the fields over again, but you do *not* need to copy the
docstrings for those fields.

In an ideal world, Graphiql and other graphql browsers would know to
copy the docstrings from the interface to its various
implementations.  Right now they don't, but we'd rather fix them in
code than make people copy documentation all over the place.
