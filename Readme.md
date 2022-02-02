# Khan Academy Coding Style Guides

We implement a style guide for our code with the intention of keeping things readable and consistent. Please do your part on the team to help keep the spirit of this consistency in both your own code, as well as politely pointing out violations in other people's code when doing their code reviews. While code prettiness should never be valued over launching or any user-visible impacting changes to the code, the idea is that maintaining a readable codebase helps things be more maintainable, and in the long run will make it easier to do the real changes that do make user-visible changes.

There may be lots of legacy files that do not adhere to the current style guide; if you're editing an old file, be consistent with what's around you.

To help adhere to these rules, some tools are available to automatically catch, and in some cases fix, style violations. See the per-language guides below for more info.

## TODOs

If there is something that you want to deal with later, it’s appropriate to mark it in code. There’s an advantage to using a standard format, which is this:

```
# TODO(your_username): Fix this to work with frobnozzes too
# TODO(your_username): Remove this once we support quxxes (at least by Dec 2012)
```

The text TODO is followed by your username in parentheses. This does not mean that you are on the hook to follow through on the TODO. Rather, it means that you are the person most knowledgeable about it, so if others run across the TODO and have questions about it, they know who to talk to.

In code reviews, it is common to put in TODOs when a reviewer points out some thing in the code that could be improved, but is not necessary to do right away.

## Language Style Guides

- [JavaScript](/style/javascript.md)
- [React](/style/react.md)
- [CSS](/style/css.md)
- [Python](/style/python.md)
- [Go](/style/go.md)
- [Java](/style/java.md)
- [ObjC](https://github.com/Khan/objective-c-style-guide) (hosted separately)
- [iOS](/style/ios.md)
