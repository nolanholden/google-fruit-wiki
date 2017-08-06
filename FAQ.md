#### How mature is this project?

This project was started in May 2014, Fruit 1.0.0 was released in November 2014 and Fruit 3.0.0 was released in
August 2017. Fruit now has extensive unit and integration tests.

At the time of writing, Fruit is the most popular dependency injection library for C++ (at least, it's the first result
in a [Google search](https://www.google.co.uk/search?q=C%2B%2B+dependency+injection) for "C++ dependency injection" and
it's
[the top C++ dependency injection library](https://github.com/search?l=&o=desc&q=dependency+injection+language%3AC%2B%2B&ref=advsearch&s=stars&type=Repositories)
by number of stars on Github).

#### Why the name "Fruit"?

Fruit is loosely inspired by the [Guice](https://github.com/google/guice) library for Java (pronounced as "juice"),
which uses run-time checks (unlike Fruit where most checks occur at compile-time).

_If you'd like some Guice but don't want to wait (for run-time errors), have some Fruit._

It also hints to the fact that Fruit components have clearly-defined boundaries while Guice modules don't (all Guice
modules have the same type) and to the fact that both Fruit and Guice are 100% natural (no need to run any code
generation tools).

#### Does Fruit use Run-Time Type Identification (RTTI)?

Fruit **optionally** uses compile-time RTTI (i.e., it calls `typeid(T)` at compile time) to print type names in error
messages.
However, Fruit never calls `typeid` on an object, and never calls it at all at runtime.

For the `typeid(T)` calls to work RTTI has to be enabled for the build, but you should not expect any performance
degradation due to RTTI, since it's only used at compile time.

If RTTI is disabled, Fruit will still compile and work, but if an injection error occurs Fruit won't be able to print
the affected types as it otherwise would. It's therefore reasonable to have RTTI disabled in release builds, but
enabled in debug builds. If you disable RTTI when building code that uses Fruit, you'll need to link against a Fruit
build that also has RTTI disabled (note that the pre-packaged Fruit binaries have RTTI enabled).

#### Fruit uses templates heavily. Will this have an impact on the executable size?

Fruit uses templates heavily for metaprogramming, but the storage that backs injectors and components is *not*
templated.

Most of the templated methods are just wrappers and are written in such a way that they can be inlined by the compiler;
so you should not expect a significant increase in the executable size due to the use of templates.

See also the next question.

#### Does Fruit have an impact on the executable size?

If before using Fruit you were already using virtual classes, the impact on the executable size will be negligible.

Otherwise, there will be a (likely small, but noticeable) increase due to the RTTI information for classes with virtual
methods (if RTTI is enabled). Note that [RTTI can be disabled](faq#does-fruit-use-run-time-type-identification-rtti).

In addition, for each binding there will be up to 4 additional symbols in the executable (one function to create the
object, one function to destroy it and two static constants). The two functions are small forwarding functions, so the
constructor/destructor of the injected type might get inlined into them; in this case 1 or 2 symbols can be saved.

An example codebase with 100 interfaces, 100 (empty) classes, 100 component functions and 900 dependency edges between
classes takes around `400 KB` (when compiled with GCC, with RTTI enabled).

See also the [benchmarks page](benchmarks) and the next question.

#### Does Fruit have an impact on performance?

If before using Fruit you were already using virtual classes allocated with `new`, injection shouldn't have a noticeable
impact on performance.

Otherwise, there will be the cost of the memory allocations for implementation classes (if they were allocated on the
stack before), and some speedup that was gained by inlining functions into their callers might also be lost.

There are benchmarks results for various compilers in the [benchmarks page](benchmarks). The summary is that with 100
injected interfaces, 100 implementation classes and 900 dependency edges between them, the injection time (creation of
the injector + injection of all the 100 classes) is around 70 us (when using GCC).

When using `NormalizedComponent` to pre-collect the bindings (while still having a separate injector for each request),
you can pay the 70 us cost once and then pay only 2 us at injection time (creation of the injector + injection of all
100 classes). 

So creating an injector for each request (in addition to the single injector created during the server startup) can be
reasonable, depending of course on your performance requirements.
