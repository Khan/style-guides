# Go

Our style guide is based on the advice in [Effective
Go](https://golang.org/doc/effective_go.html) and the [Code Review
Comments wiki](https://github.com/golang/go/wiki/CodeReviewComments),
with the following additions and modifications.

## Naming

### Module names with multiple words should use snake_case

According to "Effective Go," we should use short package names, that
are a single word.  For our library code under pkg/, we follow that
recommendation.

But in our services/ tree, where our packages are all internal, we
have a little bit more freedom (and a lot more packages to name). In
some cases multi-word package names are clearer/less ambiguous, and in
those cases, separate the words with underscores.

### Use a leading underscore for file-private symbols

In addition to the standard Go convention that capitalized names are
public and lowercased names are visible to all files in the same
package, we add an additional rule: if you want to declare that only
_this_ file should access the symbol, name it with a leading
underscore.
```
// style/visibility.go
package style
func AnyoneCanCall()                {}
func anyFileInPackageStyleCanCall() {}
func _visibilityDotGoCanCall()      {}
```

The same convention applies to type names and global
variables/constants.  For struct fields and method names, an
underscore is a hint that the name is somehow private even within the
package, but this is not enforced.

Note that we allow tests in the same package to access file-private
symbols, but tests in different packages must follow the rules and
only access exported symbols.

The reason for this rule is to help code readers understand the scope
of code as they're reading it.  It's very common to write helper
functions that are only used once -- maybe twice -- and by another
function in the same file.  Marking those helper functions tells
people reading the code not to try to make sense of an awkward API, or
generically named function, or lacking documentation: looking at the
nearby caller would be more productive instead.

### Initialisms

Go
[chooses](https://github.com/golang/go/wiki/CodeReviewComments#initialisms)
naming like `RequestURL`, rather than `RequestUrl`, for initialisms
like `URL`.  (Prefixes like `urlPath` are not capitalized at all.)  We
follow this but not religiously; if multiple initialisms appear
consecutively (`NWEAMAPID`), do what seems clearest in context.


## Formatting

Most issues of formatting are covered by gofmt -- we actually use
[gofumpt](https://github.com/mvdan/gofumpt) -- but we add a few
additional rules.

### Line length

Neither Go nor gofmt specify a maximum line length.  We do: most lines
should be at most 79 characters, and all lines must be at most 100
characters.

Specifically, code should be wrapped around 79 characters, but a few
over is okay if there's not a great way to wrap the line.  Comments
must always be wrapped at 79, unless they have non-breaking words like
urls, that are longer.

Go is liberal in allowing line breaks without explicit continuation.
Any binary operator that ends a line signals that the next line
continues the same expression.  For example:
```
// bad
if myLongCondition("isToo", long) && yourEvenLongerCondition("has", "so", "many", "arguments") {
    …
}

// good
if myLongCondition("isToo", long) &&
    yourEvenLongerCondition("has", "so", "many", "arguments") {
    …
}
```

In general, this style is preferable to breaking up a single element
of the condition.  When possible, break in a way that matches the
precedence of the operators:
```
// bad
if aa && ab && ac || ba &&
    bb && bc {
    …
}

// good
if aa && ab && ac ||
    ba && bb && bc {
...
}

// ok
if aa &&
    ab &&
    ac ||
    ba &&
       bb &&
       bc {
    ...
}
```
As you can see, gofmt's indentation will help you out in some of the
awkward cases.  Similar style applies to other operators like `+`, and
even the field/method-lookup operator `.`. (Of course in reality, this
code would benefit from some parentheses, which will also help you get
the wrapping right.)

For function/method definitions, if the signature is too long for a
single line, put each argument on its own line (note you will need a
trailing comma):
```
// bad
func (t *MyType) MyBadMethod(ctx context.Context, argumentOne string, argumentTwo string) (string, error) {
    ...
}

func (t *MyType) MyBadMethod(
    ctx context.Context, argumentOne string, argumentTwo string,
) (string, error) {
    ...
}

// good
func (t *MyType) MyGoodMethod(
    ctx context.Context,
    argumentOne string,
    argumentTwo string,
) (string, error) {
    ...
}

func (t *MyType) MyOtherGoodMethod(
    ctx context.Context,
    argumentOne, argumentTwo string,
) (string, error) {
    ...
}
```
It is acceptable to put multiple arguments on the same line if they
use the `nameOne, nameTwo type` style, but otherwise either the
signature should be a single line, or each argument should have its
own line.  The same wrapping is possible, albeit rarely necessary, for
receivers and returns:
```
func (
    t *MyVeryLongReceiverTypeName,
) MyVeryLongMethodName(
    ctx context.Context,
) (
    MyVeryLongReturnTypeName,
    MyOtherVeryLongReturnTypeName,
) {
    ...
}
```

Sometimes, the best way is a temporary variable:
```
// bad
donationAsk.DefaultDonationFrequency = donationFrequencyGraphQLtoModel[*input.DefaultDonationFrequency]

// good
freq := *input.DefaultDonationFrequency
donationAsk.DefaultDonationFrequency = donationFrequencyGraphQLtoModel[freq]
```

A few lines get exceptions to even the 100 character rule, because
there's no good way to wrap them.  These include machine-readable
comments (like //go:generate), lines ending with a URL, or lines
containing struct tags.
```
// ok
var serviceURL = "https://www.my.long.service.domain.name/some/path/to/api/v1/my/call/yikes/"

type myStruct struct {
    MyLongFieldName map[MyLongFieldKeyType]MyLongFieldValueType `json:"myLongFieldName" datastore:"my_long_field_name"`
}

//go:generate go run github.com/Khan/mypackage/path/to/codegen arguments --long-path=/path/to/my/directory
```

### Leave a blank line between toplevel functions

This is one detail gofmt doesn't specify.  We choose to leave a blank
line before and after each toplevel function.  Other toplevel
declarations, like `var`, `const`, and `type`, may go on consecutive
lines to each other, especially if they are related and fit on a
single line.

```
// good
type fooString string
type barString string

type bazString string
var baz bazString

var host = "www.khanacademy.org"
var hostOverride string

func DoSomething() {
    ...
}

func somethingElse() {
    ...
}

// ok
type fooString string
var unrelated = 2

type fooInnerStruct {
    bar string
    baz int
}
type fooOuterStruct {
    fooInnerStruct
    qux map[string]string
}

// bad
type notLikeThis string
func Yuck() {
    ...
}
func yuckier() {
    ...
}
var dontDoThis string
```


## Documentation

The advice in [Effective
Go](https://golang.org/doc/effective_go.html#commentary) applies here
too.  For all godoc-based documentation, it can be useful to check how
it renders in godoc on the commandline (`go doc -all ./my/pkg`) or in
a browser (`godoc -http=localhost:6060`).

### Documenting packages and files

Packages may be documented with a README.md, or a godoc package
comment.  Prefer godoc if the package's contents and consumers are all
Go code (such as `pkg/mypackage` or `services/my-service/testutil`)
and README if not (such as `services/my-service` or
`services/my-service/my-data-files`).  The formatting in godoc is not
as expressive as markdown, so packages where complex formatting is
particularly useful to the documentation may wish to have a godoc and
a README, which reference each other.

Package comments may be at the top of any Go file, before the `package
mypackage`.  Typically, they should go in the main entrypoint (often
called `mypackage.go`, or sometimes `api.go` or `client.go`).
Particularly long package comments can go in their own file,
canonically `doc.go`.  Only one file should have a package comment (Go
doesn't require this, but it's a bit confusing otherwise.)

Package comments or READMEs should include a general description of
the package for someone unfamiliar with it, and general pointers as to
how to use it.  For example, they may point to the most common
entrypoint functions or types, or give a usage example.  They may also
include any other useful documentation that doesn't pertain to a
specific type or function.  Package comments may include
section-headings, which are simply text on a line by itself with no
punctuation.

Go has no convention for file-level documentation.  We choose to put
such documentation after the package comment, before the imports.
This documentation should contextualize this file within the package,
and is intended for other implementers (or people trying to understand
the code) moreso than for the package's consumers.  All non-test files
in a package should have a file comment, unless the package is only a
single (non-test) file.

### Documenting types, functions, methods, and fields

As Effective Go says, all exported names (types, functions, methods,
fields, etc.) should have godoc comments.  Such comments start with a
summary, which begins "FunctionName gets/does/returns/etc. …" and uses
full sentences to give a short (1-2 line) summary of the function.  If
more information is useful, leave a blank comment-line, then further
comments.  Doc-comments need not repeat the signature of the function
(or type of the field), except as to document arguments or returns not
obvious from the name and type.

Use [named
returns](https://golang.org/doc/effective_go.html#named-results) if
the types are not sufficient to tell what the return values mean; this
is especially useful if multiple of them have the same type:
```
// bad
func GetEmailBody(...) (string, string, error) { … }

// good
func GetEmailBody(...) (text, html string, err error) { … }
```

Methods which are exclusively used via an interface (such as exported
methods of unexported types) should be documented, but the
documentation may simply refer to the interface, e.g. "MyMethod
implements the interface MyInterface".  If the method may also be used
on its own, or there is more to add about this type's particular
implementation of the interface, that can be included as well.

Unexported names should often be documented, too, for other
implementers of the package.  You can see the documentation with `go
doc -u`, or by reading the file.


## Imports

### Always qualify imports, with their package name if possible

Go has three styles of import: you can import a package with its
default name (the last component of its path), with your own name, or
unqualified (by using the special name ".").  Whenever possible, use
the default name; this makes it easier for readers to know what code
is referenced without reading your import block.  If this name is not
acceptable (e.g. it's very long, not meaningful, not a valid
identifier, or collides with another name), it's okay to use an
abbreviated name, but don't abbreviate so far that the name is
nonspecific.  (If the package is first-party, a better name may be in
order!)  Do not import unqualified.  For example:
```
import (
    // good
    "net/http"
    "cloud.google.com/go/datastore"
    "github.com/Khan/webapp/pkg/lib"
    cloudkms "cloud.google.com/go/kms/apiv1"
    userlib "github.com/Khan/webapp/pkg/user"
    mobileData "github.com/Khan/webapp/services/mobile-data"

    // ok only if there are naming collisions or
    // other problems with "lib"
    khanPkgLib "github.com/Khan/webapp/pkg/lib"

    // bad
    l "github.com/Khan/webapp/pkg/lib" // nonspecific
    . "github.com/Khan/webapp/pkg/lib" // unqualified
)
```

### Sort imports into stdlib, first-party, and third-party

Standard go style requires that all imports use a single import
statement, but allows blank lines to separate groups of imports within
that.  We follow a style similar to python: standard library imports,
third-party imports, and first-party imports go in separate blocks.
We consider "pkg" imports from webapp to be first-party imports.  Each
block is sorted (goimports enforces this).  For example:
```
// good
import (
    "log"
    "net/http"
    "os"

    "github.com/99designs/gqlgen/handler"

    "github.com/Khan/webapp/pkg/lib"
    "github.com/Khan/webapp/pkg/web"
    "github.com/Khan/webapp/services/myservice"
    "github.com/Khan/webapp/services/myservice/mypkg"
)

// bad
import (
    "github.com/99designs/gqlgen/handler"
    "github.com/Khan/webapp/pkg/lib"

    "github.com/Khan/webapp/pkg/web"
    "github.com/Khan/webapp/services/myservice"
    "github.com/Khan/webapp/services/myservice/mypkg"
    "log"

    "net/http"
)
```


## Best practices

### Be sparing in using assignments in if statements

Go lets you do -- perhaps encourages you to do -- code like this:
```
// bad
if value, err := myfunc(); err != nil { ... }
```
This hides the important part -- calling `myfunc()` and assigning its
return-type to a new local variable -- in the middle of the line, with
lots of other stuff going on in that line as well.

Clearer is to do the assignment separately, even though it's an extra
line of code:
```
// good
value, err := myfunc()
if err != nil { ... }
```

### Do not use unadorned returns

If you use [named
returns](https://golang.org/doc/effective_go.html#named-results) for a
function, Go allows you to just say `return` and it will automatically
return the named variables.  Opinions differ about whether this helps
or hurts readability.  We follow the advice on the
[wiki](https://github.com/golang/go/wiki/CodeReviewComments#named-result-parameters):
say `return <var1>, <var2>` as if the return values were not named,
unless the function is very short.

### Construct URLs using net/url, not string methods

In general, avoid using string methods to manipulate URLs, such as by
making a relative URL absolute: it's a bug-magnet as an extra or
missing slash can totally break the URL.  Instead, parse the URLs
using `net/url.Parse`, and manipulate the `net/url.URL` objects.  This
can be a bit more verbose but it's much safer.

The same applies for file paths: use `path/filepath`, not string
manipulation.

For example:
```
// bad
return "https://" + req.Host + "/about"

// good
aboutPageURL := url.URL{
    Scheme: "https",
    Host: req.Host,
    Path: "/about",
}
return url.String()
```

### Use reflection sparingly, especially in application code

Go's reflect library is useful and powerful, but it makes for much
harder-to-read and less type-safe code.  We do sometimes need it,
especially in infrastructure code, but it's best to avoid if it's not
really necessary.  Definitely don't use reflect if all you need is a
type switch!

### Do not be afraid to use codegen

It is a great way to get type-safety, at least until generics come along.


## Concurrency

Go makes writing good concurrent code easier than some languages, but
there are still some rough edges and style rules to be aware of.

### Use goroutines when *you* have concurrent work to do

In some languages, it's common to write functions which return a
Promise (or similar), to allow the caller to wait on the result.  In
Go, there is no need: there is no "blocking the event loop" to worry
about, and if the caller wants to do something else while they wait,
they can simply use their own goroutine.  So all functions should
typically be written to simply return their result, not a
promise-style structure.

For more on this topic, see [this
talk](https://drive.google.com/file/d/1nPdvhB0PutEJzdCq5ms6UI58dp50fcAN/view).
```
// good: doesn't use a goroutine
func GetUserData(ctx ...) (*UserData, error) {
    err := ctx.Datastore().Get(...)
    return userData, err
}

// good: uses goroutines to do two things in parallel
func GetUserSettingsAndUserData(ctx ...) (*UserData, *UserSettings, error) {
    g, ctx := errgroup.WithContext(ctx)
    g.Go(func() error {
        userData, err = GetUserData(ctx)
        return err
    })
    g.Go(func() error {
        userSettings, err = GetUserSettings(ctx)
        return err
    })
    err := g.Wait()
    return userData, userSettings, err
}

// bad: forces concurrency onto its caller
func GetUserData(ctx ...) (<-chan *UserData, error) {
    ...
}
```

### Arrange for all your goroutines to exit

If you start a goroutine, it's your responsibility to arrange for it
to exit, when this request ends or otherwise, unless it's really
intended to go forever.

The easiest, and recommended, way to do this for most users is with
`x/sync/errgroup`.  This package makes it easy to start a bunch of
pieces of work, wait for them all to complete, and early-exit if any
fail.  Make sure to use `errgroup.WithContext` to set up the errgroup,
and pass the returned context into each goroutine.

Generally, make sure to always pass a context through to your callees.
This helps ensure that if a request is cancelled, all its work will
quickly exit.  If you're doing something that might take a while, make
sure you check for context-cancellation.  Typically the low-level APIs
will do that for you, so you just need to pass them a context, but if
you are doing something slow that doesn't handle context, you'll need
to do so yourself, typically with a `select` statement.

To explicitly set a timeout on some work (in a goroutine or not), use
`context.WithTimeout` or `context.WithDeadline` to set a deadline on
the context which will be used for that work; then follow the
instructions in the previous paragraph to ensure that deadline is
observed.  See their documentation for more.

> [Khan-specific: For work that may outlive the request (such as
> "fire-and-forget" RPCs), use `pkg/kacontext.Detach` along with a
> goroutine; see its documentation for details.  Note that such work is
> not guaranteed to happen, for example if the instance is no longer
> needed; use tasks for larger or more important blocks of asynchronous
> work.]

### Defer log.PanicHandler

When starting a goroutine, make sure you call a defer that handles any
panics, before doing anything else.  This ensures that a a panic in
your goroutine will fail just this request, and not the whole
webserver.

> [Khan-specific: use `defer log.PanicHandler(ctx.Log())` for this.]

### If in doubt, use a lock

If you enjoy thinking about concurrency, you'll find the Go
specification as when it's safe to read and write to shared memory
very interesting.  In general, avoid using this knowledge: if you have
to think carefully to figure out whether you need a lock to share
memory safely, you probably need a lock.

### You might not need channels

Channels are a great tool, and are often useful for returning results
back from work happening in a goroutine.  However, using channels
means paying attention to when they might block, and making sure to
close them if needed.  Not all concurrent code needs channels.
```
// good: needs channels to do a select
c := make(chan int)
go func{
    result := ... // some work that can't be interrupted
    c <- result
    close(c)
}()
select {
    case <-ctx.Done():
        return "", ctx.Err() // cancelled or timed out
    case val := <-c:
        return val // operation completed
}

// good: doesn't need channels
results := make([]int, len(inputs))
g, ctx := errgroup.WithContext(ctx)
for i, input := range inputs {
    g.Go(func() error {
        results[i] = f(input)
        return nil
    })
}
err := g.Wait()
return results, err

// bad: uses channels unnecessarily, which leads to several bugs!
c := make(chan int) // unbuffered channel
g, ctx := errgroup.WithContext(ctx)
for i, input := range inputs {
    g.Go(func() error {
        c <- f(input) // waits until receive (range c below)
        return nil
    })
}
err := g.Wait() // waits until goroutines finish -- DEADLOCK
var results []int
for val := range c { // waits until channel is closed (it never is)
    results = append(results, val)
}
return results, err
```

## Contexts

> [Khan-specific: Note that most Khan Academy code uses the enriched
> KAContext system described by ADR-229 rather than ordinary Go
> context; see pkg/kacontext for further documentation.  The below
> rules apply to both Go-style context and KA-style context.]

### Context should be named `ctx` and the first argument

If a function needs to be passed a `context.Context`, that context
should always be the first argument and should be named `ctx`.
Variables referring to context should generally be named `ctx`.

> [Khan-specific: if the code needs to distinguish between Go-context
> and KA-context, such as the caller of `kacontext.Upgrade`, it may use
> `ktx` for the KA-context.]

### Only use `context.Background` in special circumstances

It's easy to use `context.Background` because you don't have to pass
anything around.  But unless you're spawning something that you don't
want canceled if the current request is canceled, like a long-lived
thread, it's the wrong thing to use.  Pass around an actual context
object instead.

This rule does not apply to tests, nor to `main` or `init` functions,
where you aren't necessarily within a request or any other meaningful
context.

### Do not store context in a struct

The package documentation says:

> Do not store Contexts inside a struct type; instead, pass a Context
> explicitly to each function that needs it.

Other than a few special cases in library code where there is no good
alternative, it's inadvisable to store context in a struct: the caller
may need to modify that context, which gets to be a mess as the struct
gets nested.  Instead, pass the context explicitly.

## Google libs

### Low-level functions that modify datastore should take a Transaction object or Client object

If a low-level function -- one not immediately called by an API
handler or graphql resolver -- needs to `put()` to the datastore, it
should explicitly take in a `datastore.Transaction` object, if the
`put()` needs to be transactional, or a `ctx` that includes
`datastore.KAContext`, if it does not.  (A `put()` needs to be
transactional if two things are being put in the same function, and we
must have both-or-neither semantics; or if the helper function does
get()-modify-put().)  This is how the function documents whether it
needs to take place in a transaction or not.

### Do not use get-or-insert semantics for read-only functions

The Python datastore library has a `get_or_insert()` function, which
given a key either returns an existing item from the datastore, or
creates a new one if the key is not found.  Do not use that idiom for
code that is semantically read-only -- that is, for code that doesn't
promise that the key exists in the datastore after it exits.
(Accessors, such as `getFoo()`, fall into this category.)  Instead
just return a default entity on key-miss without actually inserting it
into the datastore.

### Return a NotFound error on datastore lookup failures, rather than nil

Treat not-found as an error, not as an "expected" case that returns a
nil pointer.  This way, downstream clients don't have to check for
nil-ness in addition to checking for errors (but can still check
separately for not-found vs other errors if they care to).

Good:
```
func GetMyModel(key *datastore.Key) (*MyModel, error) {
    var mymodel MyModel
    err := ctx.Datastore().Get(ctx, key, &mymodel)
    return &mymodel, err
}
```

Bad:
```
func GetMyModel(key *datastore.Key) (*MyModel, error) {
    var mymodel MyModel
    err := ctx.Datastore().Get(ctx, key, &mymodel)
    if errors.Is(err, datastore.EntityNotFound) {
        return nil, nil
    }
    return &mymodel, err
}
```

## Testing

> [Khan-specific: this entire section is Khan-specific]

### Use the Khan Suite instead of Testify's

Our tests should always use `servicetest.Suite` (or, for low-level
tests, `khantest.Suite`) rather than Testify's version.  We use suites
to support tear-down functionality, extra methods, and for
consistency; see ADR-222 for more..  The API is more or less the same;
see their respective godocs for details.  The receiver variable should
always be named `suite`, e.g.
```
func (suite *mySuite) TestFoo() { … }
```

A file may contain a single suite, or multiple suites if additional
grouping is useful.  The suite type should come first, followed by the
methods, then the runner-test.  If there are multiple suites, they
should come one after the other, and not be interleaved; this makes it
easy to see what code goes with what.
```
// good
type mySuite struct{ servicetest.Suite }
// methods of mySuite
// runner-test for mySuite

type mySuiteWithField struct {
    servicetest.Suite
    myVar int
}
// methods of mySuiteWithField
// runner-test for mySuiteWithField
```

Suites may override testify lifecycle methods; if they do so those
methods must call the "super", i.e. suite.Suite.MethodName().

### Generally, use require over assert

The default style in Testify is for assertions to allow the test to
continue running:
```
suite.Assert().Equal(3, 1 + 1)    // or assert.Equal
suite.Assert().Equal(4, 2 + 3)
```
will report two errors in the same test.  To exit the test if the
assertion fails, one can use require:
```
suite.Require().Equal(3, 1 + 1)   // or require.Equal
suite.Require().Equal(4, 2 + 3)   // will not run
```
We use the latter style: it tends to give clearer errors.  However, we
allow one exception: within a call to `dev/khantest`'s `suite.All`,
use `Assert()` -- this allows reporting multiple error messages which
can aid in debugging.  (See its documentation for details.)  Do not
use `suite.Equal`: always use the equivalent and more explicit
`suite.Assert().Equal`.  (One exception: `suite.FailNow(message)` is
acceptable.)
```
// good
suite.Require().Equal(3, 1 + 2)
suite.All(
    suite.Assert().Equal(http.StatusOK, responseCode),
    suite.Assert().Equal("pong\n", responseText))
if badStuffHappened {
    suite.FailNow("yikes, bad stuff happened!")
}

// bad
suite.Assert().Equal(3, 1 + 2)
suite.Equal(3, 1 + 2)
suite.All(
    suite.Equal(http.StatusOK, responseCode),
    suite.Equal("pong\n", responseText))
suite.All(
    suite.Require().Equal(http.StatusOK, responseCode),
    suite.Require().Equal("pong\n", responseText))
```

### Table-driven tests

In cases where we have many test cases that all look very similar --
maybe we want to test a simple pure function and we have many
different inputs to test it on -- a table-driven test is often a good
option.  Testify's `suite.Run` is useful for this: it makes each
sub-case execute as a separate Go test, so failures will be reported
individually.  Make sure to give each one a useful name.  For example:
```
// good
func (suite *mySuite) TestF() {
    tests := []struct{
        name string
        a, b, c int
    }{
        {"AddPositiveNumbers", 1, 2, 3},
        {"AddZeroToPositiveNumber", 0, 1, 1},
        …
    }
    for _, test := range tests {
        test := test
        suite.Run(test.name, func() {
            suite.Require().Equal(test.c, test.a+test.b)
        })
    }
}
```

Note that `test := test` is necessary to ensure that the variable
captured by the closure is not modified as the for loop proceeds.

In side-effectful application code, table-driven tests are often more
confusing than helpful.  If the test body needs more than one or two
conditionals, it may be clearer to just write out every test case
separately.

Note that ordinary tests should not use `suite.Run`; the test
method-name should suffice to describe the test.  It's only necessary
for table-driven tests where each sub-case needs a name.

## Error handling

> [Khan-specific: this entire section is Khan-specific]

We have a [special page all about error
handling](https://khanacademy.atlassian.net/wiki/spaces/ENG/pages/150208513/Goliath+Errors+Best+Practices)!
In summary:
* use `pkg/lib/errors` to create errors
* do not include PII in errors
* do not include `Sprintf`ed strings in errors
* choose the error-kind that best describes your error
* always wrap non-Khan errors before returning them
* log at error level if the request failed, and at warn level if we can
  do fallback
* when creating an error, either return it or log it; do not do both
