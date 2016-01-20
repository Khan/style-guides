# iOS Style Guide

We have guidelines both for [code style](#code-style-guide) and for [architectural style](#architectural-style-guide).

## Code Style Guide

Hosted [externally](https://github.com/Khan/objective-c-style-guide) to this document.

TODO(andy): Make a Swift code style guide.
TODO(andy): Incorporate both style guides here.

## Architectural Style Guide

Think of these tenets as "forces" which may push your designs incrementally in various directions. Other forces (like readability, expediency, performance, etc.) may push harder!

TODO(andy): Flesh out sections with more examples and rationale.

### Don't repeat yourself

Starting with a softball here: we should not repeat ourselves.

We most often think of this in terms of code, where we should be careful of repeating even small idioms like `if (something) obj.doAThing()`. But it also applies to *knowledge* and *high-level structure*.

We should not encode *semantic knowledge* (like "the image's size must be set before you can start the renderer"), even indirectly, in multiple places. This is particularly true when that knowledge relates to business logic.

We should be on the lookout for repeated *high-level structures*: sets of statements or even subgraphs of instances which are *shaped* the same way, though they might resist abstraction via a simple parameter, often represent a liability or impediment. Sometimes straightforward patterns or abstractions can resolve these cases (e.g. `map`, the observer pattern); at other times, the cost of abstraction is perhaps too high for us to pay (e.g. applicative functors).

### Separate concerns

Strive to make a given component responsible for only one thing. This makes components more composable, easier to evolve, and more testable. This is often called the [single responsibility principle](http://en.wikipedia.org/wiki/Single_responsibility_principle)

### Prefer composition to implementation inheritance

Implementation inheritance creates coupling between child classes' implementations and the parent classes. When possible, make a new type with a *has-a* relationship to another type, rather than an *is-a* relationship to another type.

*Interface* inheritance (i.e. via protocols) does not suffer from this problem.

### Prefer values to references

Values are [inherently inert, isolated, and interchangeable](http://www.objc.io/issue-16/swift-classes-vs-structs.html). When possible, build abstractions using value types instead of using reference types. When a reference type is required (because what you're building is intentionally not inert), consider breaking the type's logic out into value types.

When you can't use a value type, try to minimize mutability in your reference types to achieve a closer approximation to isolation and interchangeability.

### Prefer tree-like object graphs

Avoid constructing object graphs in which objects have multiple long-lived owners. The API contract of these objects becomes much more complicated because it must specify how responsibility is divided between the owners.

### Isolate mutation: data flows downwards; events flow upwards

It should always be clear where the "true" source of any piece of data is, relative to any piece of your application. Strive to thread that data linearly down to dependent components.

Isolate mutation by passing events or requested changes back up the object graph from leaves to "the source of truth."

### Minimize hard coupling

Consider minimizing the number of concrete types referenced and instantiated in a given implementation. This is often called the [dependency inversion principle](http://en.wikipedia.org/wiki/Dependency_inversion_principle).

Minimizing the number of concrete types *referenced* is about working in terms of interfaces, not implementations. Push to replace references to concrete types with generics or protocols, especially with the standard library, where you might define interfaces in terms of `SequenceType` instead of `Array`.

Dependency injection can help minimize the number of concrete types *instantiated* within an implementation. You can also pass around factory interfaces instead of instances.

This tenet helps make our components more composable, makes the object graph easier to change over time, and allows for simpler, higher-quality tests.

### Isolate UIKit and Core Data

Interactions with these frameworks embody a complex responsibility; their vended types often create inter-component dependencies. Implementations which interact with UIKit or Core Data should strive to have *only* that responsibility.

#### Decomposing view controllers

`UIViewController`s often entangle many responsibilities:

1. Receiving lifecycle and system events from UIKit and instantiating `UIView`s; handling user-interaction gestures.
2. Transforming model data for presentation.
3. Performing side effects (e.g. network requests, I/O) in response to user actions.

Try to restrict subclasses to responsibility #1. Not sure how to go about decomposing your view controller's responsibility? One coarse split to start from: split #2 out into a (easily testable) value-typed "presenter" component to transform model data, and #3 into a reference-typed "interactor" component to interact with side effects like network requests or I/O.

### Minimize secret handshakes between types

Try to avoid calling methods on another type which are less visible than that type itself. This increases testability and forces us to think clearly about our interfaces. We're not going to do [design by contract](http://en.wikipedia.org/wiki/Design_by_contract), but this helps us get marginally closer.

```swift
public struct GuideDemo {
  public func demo() {
    PublicType().public() // A-OK
    PublicType().internal() // Secret handshake alarm!

    InternalType().internal() // A-OK
    InternalType().private() // Secret handshake alarm!

    internal() // A-OK, since this is called on self.
    private() // A-OK, since this is called on self.
  }

  internal func internal() {}
  private func private() {}
}

public struct PublicType {
  public func public() {}
  internal func internal() {}
}

internal struct InternalType {
  internal func internal() {}
  private func private() {}
}
```

### Expose the simplest interfaces

Don't expose more than you have to in a type's interface. If you're passing a complex type to a consumer which uses only a small subset of its functionality, consider using a [facade](http://en.wikipedia.org/wiki/Facade_pattern). This minimizes coupling and makes components easier to evolve. Taken to an extreme, this is the [interface segregation principle](http://en.wikipedia.org/wiki/Interface_segregation_principle), but we don't need to be quite so authoritarian.

#### Minimize use of `Optional`s

Only use `Optional`s when the semantic you really intend is: there's a value here sometimes; and there isn't a value here sometimes. When possible, isolate the optionality so that the highest scope is non-optional.

When the semantic you intend is more specific (i.e. "nil" has a special interpretation or is a sentinel), use a custom `enum` instead.

### Use simple higher-order transformations when possible

Prefer `map`, `filter`, `reduce`, `sum`, `max`, etc. to ad-hoc implementations of those transformations.

Avoid more esoteric higher-order transformations (e.g. `scan`, `span`, etc).

### A broad heuristic: think "how would I test this?"

Prefer architectural approaches which allow you to test a component in isolation, by passing values to some interface and looking at its return values.

Avoid architectural approaches which will require you to test a component in conjunction with several other components, or by configuring and observing state external to the component.

A few strategies which help components be useful in isolation:

 * moving more functionality to the value layer (aka playing "the value layer game")
 * using dependency injection
 * making methods `static` or free
