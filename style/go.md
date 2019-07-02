# Go

| **NOTE**: This style guide is still very much a work in progress. |
| ----------------------------------------------------------------- |

Our style guide is based on the advice in [Effective
Go](https://golang.org/doc/effective_go.html), with the following
additions and modifications.

## Naming

### Use a leading underscore for file-private symbols

If a symbol is declared at a global scope, it's visible to other files
in the same package.  If you want to declare that only _this_ file
should access the symbol, name it with a leading underscore.  We will
lint to ensure it is not used outside the file.


## Best practices

### Use `errgroup` for all goroutines

TODO -- maybe we shouldn't require this?  Maybe we should use a
different lib?

### Only use `context.Background` in special circumstances

It's easy to use `context.Background` because you don't have to pass
anything around.  But unless you're spwaning something that you don't
want canceled if the current request is canceled, like a long-lived
thread, it's the wrong thing to use.  Pass around an actual `context`
object instead.

### Do not be afraid to use codegen

It is a great way to get type-safety, at least until generics come
along.

### Embrace the REPL

[Gore](https://github.com/motemen/gore) can help you debug Go code as
you write it.


## Error handling

### Write error-handling code so that `tryhard` can convert it

Robert Griesemer has written a
[tool](https://github.com/griesemer/tryhard) to convert old-style
error handlers to the new Go 2.0 `try` keyword.  Write your
error-handling code so that `tryhard` can convert it to `try` when the
functionality is released.

In particular:

* Name your error variable `err`
* If you want to modify the error before returning it, do so in a
  `defer` rather than inline in the code.
* (When using defer, you will need to use a named return value)

No:
```
func (req *Request) Decode(r Reader) error {
        var err error
	req.Body, err = readString(r)
	if err != nil {
		return unexpected(err)
	}
}
```

Yes:
```
func (req *Request) Decode(r Reader) (err error) {
	defer func() { err = unexpected(err) }()
	req.Body, err = readString(r)
	if err != nil {
		return err
	}
}
```
