# Python

We follow the [PEP8 style guide for Python](http://www.python.org/dev/peps/pep-0008/). Docstrings follow [PEP257](http://www.python.org/dev/peps/pep-0257/). The rest of the document describes additions and clarifications to the PEP documents that we follow at AltSchool.

You can use `make lint` from the top-level of the project source tree to run a PEP8 check (using flake8) over all Python source files. You can also lint an individual file with `flake8 path_to_file.py`.
You can use `make format` from the top-level of the project source tree to automatically format all Python source files that do not lint using autopep8. You can format an individual file with `autopep8 -i -a path_to_file.py`.
You should aim to not introduce any new violations, before checking in code.

## Indentation

- Use 4 spaces -- never tabs -- for indentation.

## \_\_init\_\_.py

- `__init__.py` files should be empty.

> *Rationale:* when you do `import foo.bar` python imports two files: `foo/bar.py`, and `foo/__init__.py`. If `foo/__init__.py` has imports of its own, those will be run as well -- even if you don’t plan to run any of the code defined in `__init__.py`. This slows down execution, and worse causes circular-import problems that could be entirely avoided.

If you have code that you think every user of every function inside this directory needs to run first, `__init__.py` may be appropriate, but you should also consider just creating a function that executes that code, and running the function at the top level (that is, not indented) inside each file in your directory. This makes it more obvious what's going on, and also makes it easier to special-case certain files if the need ever arises.

Using `__init__.py` to bring variables from sub-modules into the main module space totally defeats the point of having sub-modules in the first place; don’t do it.

## Imports

### from...import

- Import entire modules or individual symbols from a module.
- Never import all symbols from a module using `import *`

> *Rationale:* Using `import *` pollutes the global namespace with an undeteremined set of symbols. Previously defined symbols may be overwritten, linting the file becomes impossible because the symbols cannot be determined easily through static analysis, and worst of all, the set of symbols may change between library versions. All import dependencies should be declared explicitly.

```py
# GOOD
import django.db.models               # module import: importing the file django/db/models.py
from models import Foo                # symbol import: importing the class Foo from models.py

# BAD
from auth import *                    # wildcard import: what did I just import exactly?
```

## Import style

- All imports should be at the top of the file, separated (by blank lines) into three sections: system imports, third-party imports, and then project-local imports.
- Each section should be sorted alphabetically by the main module name (second word of the line) then symbol name (if any). Sorting should ignore case.
- There should be one module per line (for module imports) or one symbol per line (for symbol imports).
- Symbol imports from the same module should be grouped together and line-separated.

> *Rationale:* When I see autocomplete.foo() in the code, and I want to know what it does, it’s helpful to know if I should be looking on the web (because autocomplete is part of the python distribution), or in the local source tree (because autocomplete is written by us). It’s also helpful to know if it’s code we wrote (and the barrier to hacking on it is low) or code someone else wrote (which means it may be easier to just work around any problems I have with it). The three sections tell me that with just a quick glance at the top of the file. Plus, since each section is alphabetical, it’s easy for me to find the import within the section.

```py
# GOOD
import sys
import os

from django.db import models
from django.db.contrib.auth.models import User

from project.models import (
    Bar
    Foo
)

# BAD
from app import App, Zebra          # two symbol imports on the same line
import .models                      # relative import. Alternative: from project import models
import sys                          # system library import after local imports
```

## Docstrings

All non-trivial methods should have docstrings. Docstrings should follow guidelines here: [PEP257](http://www.python.org/dev/peps/pep-0257/). For more examples, see the [Google style guide](http://google-styleguide.googlecode.com/svn/trunk/pyguide.html?showone=Comments#Comments) around docstrings.

To summarize: There are two types of docstrings, long-form and short-form.

A short-form docstring fits entirely on one line, including the triple quotes. It is used for simple functions, especially (though by no means exclusively) ones that are not part of a public API:

```py
"""Return a user-readable form of a Frobnozz, html-escaped."""
```

Note that the text is specified as an action (“return”) rather than a description (“returns”). This has the added advantage of taking less space, so the comment is more likely to fit on a single line.

If the description spills past one line, you should move to the long-form docstring: a summary line (one physical line) starting with a triple-quote ("""), terminated by a period, question mark, or exclamation point, followed by a blank line, followed by the rest of the doc string starting at the same cursor position as the first quote of the first line, ending with triple-quotes on a line by themselves. (Unlike what the BDFL suggests in PEP 257, we do not put a blank line before the ending quotes.)

```py
"""This comment serves to demonstrate the format of a docstring.

Note that the summary line is always at most one line long, and
on the same line as the opening triple-quote, and with no spaces
between the triple-quote and the text.  Always use double-quotes
(") rather than single-quotes (') for your docstring.   (This
lets us reserve ''' (3 single-quotes) as a way of commenting out
big blocks of code.
"""
```

A function (including methods and generators) must have a docstring, unless it meets all of the following criteria:

- not externally visible
- very short
- obvious

The docstring should describe the function's calling syntax and its semantics, not its implementation.

The docstring should end with the following special sections (see [the Google style guide](http://google-styleguide.googlecode.com/svn/trunk/pyguide.html?showone=Comments#Comments) for more details).

- **Arguments:** List each parameter by name, and a description of it. The description can span several lines (use a hanging indent if so). Use instead of "Args". 
- **Returns:** (or **Yields:** for generators): Describe the type and semantics of the return value. If the function only returns None, this section is not required.
- **Raises:** List all exceptions that are relevant to the interface.

Classes should follow a similar format: a single line describing the class, plus more details, but instead of Arguments/Returns/Raises, it should have an Attributes: section that lists and describes the public attributes of the class (if any).

Modules (files) should have a docstring too, at the top of the file, starting with the usual one-line summary:  
```py
"""One line summary

Longer description.
"""
```

> Rationale: People will read a piece of code many more times than they will write it. Time spent documenting at write-time more than pays off at read time. What is obvious to you as the code-author, well versed in the module where this function lives, may not be at all obvious to a code reader, who is possibly jumping into this function from some unrelated part of the codebase.

The rules here may seem like overkill, especially the need to document every argument and return value. I can say from experience two things: it often does seem like overkill when writing it (especially when the docstring is longer than the function!) but I've almost never thought it was overkill when reading unfamiliar code. You may find, as you write the docstring, you're putting down something that wasn't as obvious as you thought it was:

```py
def WriteTimestamp(now):
    """Write the current time to stdout in an arbitrary user-readable format.

    Arguments:
        now: the current time as a time_t. Should be time.time() except for tests.
    """
    ...
```

Even though the meaning of now may seem obvious, it's not obvious that it's only being passed in so it can be mocked out for tests. Its type also isn't obvious (a time_t? a tuple? a datetime object?), so it's nice to have that documented as well.

## Top of the file

Start your file with a module docstring. Do not put a shebang line (`#!/usr/bin/python`) or copyright notice, or anything else.  Follow the docstring with your imports; don't put an `__author__` line.

> *Exception:* if the python file is meant to be executable, it should start with the following shebang line:

> ```py
#!/usr/bin/env python
```

> *Rationale:* a shebang line is useless for non-executable files. An `__author__` line just gets out of date, and is better determined by looking at source control history in any case. Code is automatically copyrighted; a copyright line doesn't help. No need to put this useless boilerplate at the top of the file!

## Unused variables

If you want to assign a value to a variable that will not be used again, name the variable either `_` (python convention) or `unused_<something>` (less-well-known python convention). This will keep our lint checkers from complaining.

## Splitting lines

Using PEP8 as a guideline for Python formatting runs us head-long into a great debate: the 79-character line limit.

> Rationale: short lines have benefits, including:

> - Broad tool support. Code is read more frequently than it is written, often by those whose tools don't match the original author. Short lines are always well-supported. Many tools only naively support long lines (`<pre>` tags and most terminal tools), and no editor wraps long lines intelligently enough in all cases.
> - Side-by-side code windows. Try it out, it's great.
> - A free gut check. It's easy to reach 100 or 200 columns when writing complex expressions or nesting deeply. If this indicates unclear code, future readers would appreciate a quick refactoring.

> Of course having a hard limit for line length is silly. Any reasonable limit runs into a case where breaking the rule produces better code. However, having unnecessarily long lines scattered about due to assumptions about a reader's tools is also silly.

We follow these guidelines to produce legible code that adheres to the 79-character line limit:

- Do not use backslashes to split lines.
- Use parentheses, brackets, and braces to split lines.
- All parentheses should follow the inline or block rule:
  Either the open paren and the close paren are on the same line (inline) OR
  the open paren is the last character of a line and the close paren 
  is the first (non-whitespace) character of a subsequent line with the same indentation as the first line (block).
  * Exception: for block parens, the open paren can be followed by other open parens
    iff. the close paren is preceded by corresponding close parens.
- The contents of block expressions should be indented with 4 spaces relative to the start of the
  first and last lines.
- Comma-separated arguments to a block parenthetical expression should all be on separate lines.

```py
# BAD

# > 79 characters
words = ['the', 'big', 'brown', 'dog', 'jumped', 'over', 'the', 'lazy', 'fox', 'and', 'broke', 'pep8']

# comma-separated arguments are not all on separate lines
words = [
'the', 'big', 'brown', 'dog', 'jumped', 'over', 'the', 'lazy',
'fox', 'and', 'broke', 'pep8'
]

# block is not indented
words = [
'the',
'big',
'brown',
'dog',
'jumped',
'over',
'the',
'lazy',
'fox'
]

# use of backslash
user = User.objects.filter(name__icontains='joe'). \
    values_list('user' 'username')

# block is not clear
[{ 'foo': 'bar',    # open paren is not the last character
'biz': 'bang',
'boo' : 'qux',
'dog' : 'cat'}]     # close paren is not the first character

# GOOD

words = [
    'the',
    'big',
    'brown',
    'dog',
    'jumped',
    'over',
    'the',
    'lazy',
    'fox'
]

user = (
    User.objects.filter(name__icontains='joe').
    values_list('user', 'username')
)

[{
    'foo': 'bar',
    'biz': 'bang',
    'boo': 'qux',
    'dog': 'cat'
}]
```

### More examples

Splitting string literals doesn't require use of the + operator. Adjacent literals are automatically combined:

```py
# BAD
twist = (
    'Peter Piper' +
    'split a set ' +
    'of simple strings '
)

# GOOD
twist = (
     'Peter Piper '
     'split a set '
     'of simple strings'
)
```

Dictionary comprehension:

```py
# BAD
kwargs = dict((str(key), value) for key, value in topic_json.iteritems() if key in ['id', 'title', 'standalone_title', 'description', 'tags', 'hide'])

# GOOD
kwargs = dict(
    (str(key), value)
    for key, value in topic_json.iteritems()
    if key in [
        'id',
        'title',
        'standalone_title',
        'description',
        'tags',
        'hide'
    ]
)
```

Nested properties:

```py
# BAD
badge_name = badges.topic_exercise_badges.TopicExerciseBadge.name_for_topic_key_name(self.key().name())

# GOOD
badge_name = (
    badges.topic_exercise_badges.TopicExerciseBadge.
    name_for_topic_key_name(self.key().name())
)
```

Long string formatting:

```py
# BAD
self.redirect("/class_profile?selected_graph_type=%s&coach_email=%s&graph_query_params=%s" %
        (self.GRAPH_TYPE, urllib.quote(coach.email), urllib.quote(urllib.quote(self.request.query_string))))

# GOOD
self.redirect(
    "/class_profile?selected_graph_type=%s&coach_email=%s"
    "&graph_query_params=%s" % (
        self.GRAPH_TYPE,
        urllib.quote(coach.email),
        urllib.quote(urllib.quote(self.request.query_string))
    )
)
```

Using temporary variables:

```py
# BAD
topics_list = [t for t in topics if not (
    (t.standalone_title == "California Standards Test: Algebra I" and t.id != "algebra-i") or
    (t.standalone_title == "California Standards Test: Geometry" and t.id != "geometry-2"))
    ]

# GOOD
bad_title_and_ids = [
    ("California Standards Test: Algebra I", "algebra-i"),
    ("California Standards Test: Geometry", "geometry-2"),
]
topics_list = [
    t for t in topics
    if not (t.standalone_title, t.id) in bad_title_and_ids
]
```

Class initializers and if statements:

```py
# BAD
class Rectangle(Blob):
    def __init__(self, width,
        height, color='black',
        emphasis=None, highlight=0):
        if (width == 0 and height == 0 and
                color == 'red' and emphasis == 'strong' or
                highlight > 100):
            raise ValueError("sorry, you lose")

# GOOD
class Rectangle(Blob):
    def __init__(
        self,
        width,
        height,
        color='black',
        emphasis=None,
        highlight=0
    ):
        if (
            width == 0 and height == 0 and
            color == 'red' and emphasis == 'strong' or
            highlight > 100
        ):
            raise ValueError("sorry, you lose")
```
