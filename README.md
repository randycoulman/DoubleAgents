# DoubleAgents

Create and use "test doubles" in unit tests.

DoubleAgents is licensed under the MIT license.  See the copyright tab
in the RB, the 'notice' property of this package, or the License.txt
file on GitHub.

DoubleAgents' primary home is the
[Cincom Public Store Repository](http://www.cincomsmalltalk.com/CincomSmalltalkWiki/Public+Store+Repository).
Check there for the latest version.  It is also on
[GitHub](https://github.com/randycoulman/DoubleAgents).

DoubleAgents was developed in VW 7.10.1, but is compatible with VW 7.7
and later.  If you find any incompatibilities with VW 7.7 or later,
let me know (see below for contact information) or file an issue on
GitHub.

# Introduction

DoubleAgents is a library for creating and working with "test doubles"
(think "stunt doubles" in acting) when writing unit tests.  It is
optimized for keeping test code as readable as possible.

DoubleAgents supports two kinds of test doubles. See
[Mocks Aren't Stubs](http://martinfowler.com/articles/mocksArentStubs.html)
for more information.

* Stubs are simple objects that don't have any real behavior.  Each
  stubbed method does no work and returns a given value.

* Mocks are objects that are pre-programmed with a set of
  "expectations".  After running the code under test, the mock is
  checked to see if all of the expectations have been met.

In this library, all test doubles are instances of `DoubleAgent` and
can contain a mixture of stubbed methods and expectations.
`DoubleAgent`s can be created directly and used in place of instances
of other classes.  In addition, it is possible to stub or mock methods
on existing instances (including class-side methods of existing
classes).  It is also possible to stub or mock methods on all
instances of a class.  In these latter cases, all non-doubled
methods will continue to function like normal.

DoubleAgents was designed to support unit testing in the style
recommended by [Sandi Metz](https://twitter.com/sandimetz) in her
excellent
[Practical Object-Oriented Design in Ruby: An Agile Primer](http://www.poodr.info/book/).
See her
[slides from a talk at Ancient City Ruby](https://speakerdeck.com/skmetz/magic-tricks-of-testing)
for a good summary of her advice.

# Creating a DoubleAgent

There are four kinds of `DoubleAgent`:

* A standalone `DoubleAgent` acts as an instance of its target class.
This is the normal usage.  A standalone `DoubleAgent` is created by
sending one of the `#expect:*` or `#stub:*` API methods directly to a
class, by sending `#doubleAgent` to the class, or by explicitly
creating the `DoubleAgent`.

* `AClass expect: #aMessage [...]`
* `AClass stub: #aMessage [...]`
* `AClass doubleAgent`
* `DoubleAgent of: AClass`

* An in-place instance-side `DoubleAgent` is used for objects that are
part of a more complex object structure where it is difficult to
inject a standalone `DoubleAgent` as a dependency.  It allows
selective mocking and stubbing, where only a few methods are doubled
and all other methods have their normal behavior.  An in-place
instance-side `DoubleAgent` is created by sending one of the
`#expect:*` or `#stub:*` API methods directly to the instance, by
sending `#doubleAgent` to the instance, or by explicitly creating the
`DoubleAgent`.

* `anInstance expect: #aMessage [...]`
* `anInstance stub: #aMessage [..]`
* `anInstance doubleAgent`
* `DoubleAgent around: anInstance`

* An in-place class-side `DoubleAgent` is similar to an in-place
instance-side double, but is used for mocking or stubbing class-side
methods of a class.  Note that doubled methods on classes are global
throughout the system, so use this facility with care.  For example,
attempting to stub something like `Timer class>>after:do:` will hang the
debugger.  An in-place class-side `DoubleAgent` is created by sending
`#classSideDouble` to the class, or by explicitly creating the
`DoubleAgent`.

* `aClass classSideDouble`
* `DoubleAgent around: AClass`

* An any-instance `DoubleAgent` is used when it is necessary to mock
or stub methods on instances of a class that are not directly visible
to the test.  Any-instance doubles should be used very rarely, as they
generally suggest that there is a more fundamental problem with the
design of the code under test.  However, there are times when they are
needed.  An any-instance `DoubleAgent` is created by sending
`#anyInstanceDouble` to the class, or by explicitly creating the
`DoubleAgent`.

* `aClass anyInstanceDouble`
* `DoubleAgent forAnyInstanceOf: AClass`

# Stubbing and Mocking Methods

There is a family of methods for creating method stubs and defining
method expectations.  These methods can be sent to a `DoubleAgent`, to
a class (which automatically creates and returns a `DoubleAgent`), or
to an object (which stubs or mocks the method "in place").

A method may only be mocked or stubbed if it is understood by the
class or instance being doubled.  This is to provide extra assurance
that the test double uses the same API as the real class.

## Return Values

All stubbed and mocked methods return a value when they are called.

* If the API call doesn't specify a return value, then `self` itself is returned.
* If the API call ends with `return: anObject`, then `anObject` is returned from every call to the method.
* If the API call ends with `do: aBlock`, then `aBlock` is evaluated
  with the arguments passed to the method, and the result is returned.
  Note that the arguments are "culled" to the block, so the block
  doesn't need to specify the arguments if it doesn't need them.

## Stubbing a Method

To stub a method, use one of the following:

* `stub: aMessage`
* `stub: aMessage return: anObject`
* `stub: aMessage do: aBlock`

These methods do nothing but return a value as outlined above.

If the arguments are important, use one of the following:

* `stub: aMessage with: anObject`
* `stub: aMessage with: anObject with: anotherObject`
* `stub: aMessage with: anObject with: anotherObject with: aThirdObject`
* `stub: aMessage withArguments: aCollection`
* `stub: aMessage with: anObject return: anObject`
* `stub: aMessage with: anObject with: anotherObject return: anObject`
* `stub: aMessage with: anObject with: anotherObject with: aThirdObject return: anObject`
* `stub: aMessage withArguments: aCollection return: anObject`
* `stub: aMessage with: anObject do: aBlock`
* `stub: aMessage with: anObject with: anotherObject do: aBlock`
* `stub: aMessage with: anObject with: anotherObject with: aThirdObject do: aBlock`
* `stub: aMessage withArguments: aCollection do: aBlock`

The general forms are the `#stub:withArguments:*` methods; the others
are provided as convenient shortcuts.  These stubs are used when
`aMessage` is sent with arguments that are "congruent" (see below) to
those specified.  They do nothing else except return a value as outlined above.

For even more flexible argument checking, use one of the following:

* `stub: aMessage where: aBlock`
* `stub: aMessage where: aBlock return: anObject`
* `stub: aMessage where: aBlock do: returnBlock`

These stubs are used when `aMessage` is sent with arguments that
satisfy `aBlock`.  `aBlock` must take some or all of the arguments and
return a Boolean indicating whether the arguments are satisfactory.
They do nothing else except return a value as outlined above.

The argument-matching forms of the `#stub:*` methods are only
useful when you need to respond differently to a stubbed message
depending on one or more of the arguments.  This is needed
occasionally, but only after careful consideration of other options.
In general, you should prefer non-argument-matching forms above.

## Disallowing a Message

Sometimes, it is desirable to explicitly state that a particular
message will not be sent to an object.  To disallow a message send,
use the following:

* `disallow: aMessage`

If a disallowed message is sent, a `BurnNotice` is raised.

## Defining Expectations for a Method

In order to verify that a message is sent to an object, use one of the
following:

* `expect: aMessage`
* `expect: aMessage return: anObject`
* `expect: aMessage do: aBlock`

These methods check that `aMessage` was sent, but do not perform any
checks on the arguments.  They do nothing else except return a value
as outlined above.

If the arguments are important, use one of the following:

* `expect: aMessage with: anObject`
* `expect: aMessage with: anObject with: anotherObject`
* `expect: aMessage with: anObject with: anotherObject with: aThirdObject`
* `expect: aMessage withArguments: aCollection`
* `expect: aMessage with: anObject return: anObject`
* `expect: aMessage with: anObject with: anotherObject return: anObject`
* `expect: aMessage with: anObject with: anotherObject with: aThirdObject return: anObject`
* `expect: aMessage withArguments: aCollection return: anObject`
* `expect: aMessage with: anObject do: aBlock`
* `expect: aMessage with: anObject with: anotherObject do: aBlock`
* `expect: aMessage with: anObject with: anotherObject with: aThirdObject do: aBlock`
* `expect: aMessage withArguments: aCollection do: aBlock`

The general forms are the `#expect:withArguments:*` methods; the
others are provided as convenient shortcuts.  These methods check that
`aMessage` was sent with arguments that are "congruent" (see below) to
those specified.  They do nothing else except return a value as
outlined above.

## Argument Congruency

Argument congruency is implemented using the `#===` method provided by
the [Threequals](https://github.com/randycoulman/Threequals) package.
See that package for more details, but as a summary:

* Objects that are `#=` are also `#===`.
* A class is `#===` to an object that `#isKindOf:` the class.
* A block is `#===` to an object if it evaluates to true when passed
  the object.
* An interval is `#===` to a number that is between the endpoints of
  the interval, including the endpoints.
* If Threequals-Regex is loaded, a regular expression is `#===` to a
  string that matches it.

Using `#===` allows for expectations like the following:

```
myDouble expect: #with:and:do:
         with: (40 to: 42)
         with: [:x | x even]
         with: BlockClosure
```

This expectation will be satisfied if the arguments are some number
between 40 and 42 inclusive, an even number, and a block.

For even more flexible argument checking, use one of the following:

* `expect: aMessage where: aBlock`
* `expect: aMessage where: aBlock return: anObject`
* `expect: aMessage where: aBlock do: returnBlock`

These methods check that `aMessage` was sent with arguments that
satisfy `aBlock`.  `aBlock` must take some or all of the arguments and
return a Boolean indicating whether the arguments are satisfactory.
They do nothing else except return a value as outlined above.

## Verifying that Expectations Are Met

If an unexpected message is sent to a `DoubleAgent`, it will
immediately raise a `BurnNotice` exception.  It is necessary to check
that all expectations are met at the end of a test.  All DoubleAgents
register with a singleton instance of `Agency`.  `Agency` is
responsible for verifying all of the `DoubleAgent`s and for ensuring
that they clean up after themselves.  If an expectation is not met, a
`BurnNotice` exception will be raised.

`Agency` provides several options for verification and cleanup.

* `Agency class>>tearDown` verifies all registered `DoubleAgent`s and
  then ensures that any cleanup actions they need to perform are done.
  Only the first `BurnNotice` will be reported.  All cleanup actions
  will be performed even if a `BurnNotice` is raised, and even if a
  cleanup action raises an exception.  This method should be called
  from the `tearDown` of your test class.  If your `tearDown` method
  performs other actions that might fail, it is recommended that you
  use an `#ensure:` block to guarantee that `Agency class>>tearDown`
  is sent in all cases.

* `Agency class>>setUp` verifies that the `Agency` was torn down
  correctly by the last test that used it.  If not, a `BurnNotice`
  will be raised.  This may not help figure out which test failed to
  tear down the `Agency`, but will alert you to a problem and ensure
  that each test starts out in a clean state.  This method should be
  called from the `setUp` of your test class.

* If you use SUnitToo, `DoubleAgentTestCase` (in
  DoubleAgents-SUnitToo) implements `setUp` and `tearDown` methods
  that forward to the `Agency`.  You can have your test class inherit
  from `DoubleAgentTestCase` to ensure that the `Agency` is managed
  properly.  Make sure that your `setUp` and `tearDown` also send to
  `super` in addition to their own actions.  If your `tearDown` method
  performs other actions that might fail, it is recommended that you
  use an `#ensure:` block to guarantee that the superclass `tearDown`
  is sent in all cases.

* `Agency class>>verifyAfter: aBlock` wraps `aBlock` with `setUp` and
  `tearDown` calls.  This method is handy for a single test that uses
  `DoubleAgent`s.  For multiple tests in a class, though, it is better
  to use one of the earlier options.

* `aBlock verifyAgents` is a handy shortcut for `verifyAfter:`.  You
  can wrap the body of your test in a block and send `verifyAgents` to
  the block.

* `Agency class>>forceReset` ensures that the `Agency` is cleaned up
  correctly, but does not perform any verification.  This method
  should not be used in normal circumstances, but can be used in a
  pinch if your image gets left in a bad state somehow.

## Flexible vs Strict

By default, standalone `DoubleAgent`s are "strict".  That is, they
only allow messages to be sent that have been stubbed or mocked.  All
other message sends raise a `BurnNotice`.  For in-place
`DoubleAgent`s, messages that have not been stubbed or mocked
implement their normal behavior.  A "flexible" `DoubleAgent` will
allow other messages to be sent; they will simply return `self`.

`DoubleAgent` implements `#flexible` and `#strict` to convert between
the two.

## Ordered Sends

By default, mock expectations are "unordered".  That is, the messages
can be sent to the `DoubleAgent` in any order.  An "ordered"
`DoubleAgent` requires the messages to be sent in the specified order.
It will raise a `BurnNotice` if any messages are sent out of order.

`DoubleAgent` implements `#ordered` and `#unordered` to convert
between the two.

## Conflicts

The same method can be mocked or stubbed repeatedly.  The rule is
"last one wins".  That is, if you first stub a method, and then later
set a mock expectation on it, then the method will be a mock that is
verified.  Similarly, if you set a mock expectation on a method and
then later stub the same method, then it will be a stub.  Setting a
mock expectation on a method that is already a mock simply adds the
new expectation to the existing expectations; it means that the method
must be sent more than once.

A common pattern is to stub a method in a test's `setUp`, and then in
one or more tests, set a mock expectation to verify that the message
is sent to the object.

# Understanding the Code

`DoubleAgent` is the central class in this library.  It has subclasses
that implement the four main types of agent: `StandaloneDouble`,
`InPlaceInstanceDouble`, `InPlaceClassDouble`, and
`AnyInstanceDouble`.

All `DoubleAgent`s register with the singleton `Agency`, which is
responsible for verifying all mock expectations and cleaning up.

When verifying expectations, method arguments are verified by an
`ArgumentPolicy` such as `IgnoreArguments`, `ArgumentsEqual`, or
`ArgumentsMatch`.

Doubled methods are represented by a `MethodDouble`, either
`MockMethod` or a `StubMethod`.

# Acknowledgements

I stood on the shoulders of several giants when implementing this
library.

As already mentioned, I was inspired to write this library by trying
to follow [Sandi Metz](https://twitter.com/sandimetz)'s testing advice
in
[Practical Object-Oriented Design in Ruby: An Agile Primer](http://www.poodr.info/book/).

I looked at several other test double libraries for API and
implementation ideas, including:

* [Smallmock](http://www.cincomsmalltalk.com/publicRepository/SmallMock.html)
* [Flexmock](https://github.com/jimweirich/flexmock)
* [JMock](http://jmock.org/)
* [RSpec mocks](https://github.com/rspec/rspec-mocks)

The in-place double implementations use some clever tricks that were
inspired by the
[MethodWrappers](http://www.refactory.com/tools/method-wrappers)
project and a couple of blog posts by
[Travis Griggs](http://objology.blogspot.com/):

* [Superpower Adventures in Lightweight Classing](http://www.cincomsmalltalk.com/userblogs/travis/blogView?showComments=true&entry=3440856756)
* [Mutating your Process, for Instance](http://www.cincomsmalltalk.com/userblogs/travis/blogView?showComments=true&entry=3421076502)

# Contributing

I'm happy to receive bug fixes and improvements to this package.  If
you'd like to contribute, please publish your changes as a "branch"
(non-integer) version in the Public Store Repository and contact me as
outlined below to let me know.  I will merge your changes back into
the "trunk" as soon as I can review them.

# Contact Information

If you have any questions about DoubleAgents and how to use it, feel free to contact me.

* Web site: http://randycoulman.com
* Blog: Courageous Software (http://randycoulman.com/blog)
* E-mail: randy _at_ randycoulman _dot_ com
* Twitter: @randycoulman
* GitHub: randycoulman
