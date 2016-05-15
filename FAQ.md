#### How mature is this project?

This project was started in May 2014, and Fruit 1.0.0 was released in November 2014. Fruit now has extensive unit and integration tests.

I'm not aware of any use of Fruit in production code (but it may be used and I just don't know about it).

If you're starting to use Fruit in production code, if you want I can add a link to your project here, just let me know.

#### Why the name "Fruit"?

Fruit is inspired by Guice (pronounced as "juice"), which uses run-time checks.

_If you'd like some Guice but don't want to wait (for run-time errors), have some Fruit._

It also hints to the fact that Fruit components have clearly-defined boundaries while Guice modules don't (all Guice modules have the same type) and to the fact that both Fruit and Guice are 100% natural (no code generation).

#### Does Fruit use Run-Time Type Identification (RTTI)?

Fruit **optionally** uses compile-time RTTI (calls `typeid(T)` at compile time) to print type names in error messages.
However, Fruit never calls `typeid` on an object, and never calls it at all at runtime.
For this to work, RTTI has to be enabled for the build, but you won't have any performance degradation due to RTTI, since it's only used at compile time.

If RTTI is disabled, Fruit will still compile and work, but if an injection error occurs Fruit won't be able to print the affected type(s) as it otherwise would. It's therefore reasonable to have RTTI disabled in release builds, but enabled in debug builds. If you disable RTTI when building code that uses Fruit, you'll need to link against a Fruit build that also had RTTI disabled (note that the pre-packaged binaries have RTTI enabled).

#### Fruit uses templates heavily. Will this have an impact on the executable size?

Fruit uses templates heavily for metaprogramming, but the storage that backs injectors and components is *not* templated.
Most of the templated methods are just wrappers and are written in such a way that they can be inlined by the compiler; so you should not expect an increase in the executable size due to the use of templates.
See also the next question.

#### Does Fruit have an impact on the executable size?

If before using Fruit you were already using virtual classes, the impact on the executable size will be negligible.
Otherwise, there will be a (likely small, but noticeable) increase due to the RTTI information for classes with virtual methods (if RTTI is enabled). Note that [RTTI can be disabled](faq#does-fruit-use-run-time-type-identification-rtti).

In addition, for each binding there will be up to 4 additional symbols in the executable (one function to create the object, one function to destroy it and two static constants). The two functions are small forwarding functions, so the constructor/destructor of the injected type might get inlined into them; in this case 1 or 2 symbols can be saved.

#### Does Fruit have an impact on performance?

If before using Fruit you were already using virtual classes allocated with new, injection shouldn't have a noticeable impact on performance.

Otherwise, there will be the cost of the memory allocations for implementation classes (if they were allocated on the stack before).
Some speedup that was gained by inlining functions into their callers might also be lost.

There are benchmarks results for various compilers in the [benchmarks page](benchmarks). The summary is that when using `NormalizedComponent` to precompute the bindings (while still having a separate injector for each request), in a system with 100 classes the startup overhead is ~0.3ms and the per-request overhead is ~1us, in a system with 1000 classes the startup overhead is ~4ms and the per-request overhead is ~50-100us).

So creating an injector for each request is reasonable (in addition to the single injector created during the server startup).
