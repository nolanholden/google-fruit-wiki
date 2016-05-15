## What is Fruit?

Fruit is a [dependency injection](http://en.wikipedia.org/wiki/Dependency_injection) framework for C++, loosely inspired by the Guice framework for Java. It uses C++ metaprogramming together with some new C++11 features to detect most injection problems at compile-time. The code has also been optimized so that using injection adds very little overhead (more details here).

It allows to split the implementation code in "components" (aka modules) that can be assembled to form other components.

From a component with no requirements it's then possible to create an injector, that can create instances of the classes registered in the component.

The code is on github at [github.com/google/fruit](https://github.com/google/fruit) and [prebuilt packages](https://github.com/google/fruit/wiki/install#prebuilt-packages) are available.

## Features

*   Basic features:
    *   Binding of a type to an interface, [details here](quick-reference#bindings)
    *   Inject annotations for constructors, [details here](quick-reference#inject-macro)
    *   Binding to a provider (lambda), [details here](quick-reference#providers)
    *   Binding to an instance/value, [details here](quick-reference#binding-instances)
    *   Annotated bindings
    *   Assisted injection, [details here](quick-reference#factories-and-assisted-injection)
    *   Automatic registration of factories for a type I if there is a factory for C and I is bound to C, [details here](quick-reference#bindings)
*   Unlike most DI frameworks, most checks are done at **compile time**, so that errors can be caught early. Some examples of checks done at compile time:
    *   Checking that all required types are bound (implicitly or explicitly)
    *   Checking that there are no dependency loops in the bound types.
*   The only injection error that can't be detected at compile time is when a type has multiple inconsistent bindings in different components. This is checked at run-time.
*   No code generation. Just include `fruit/fruit.h` and link with the `fruit` library.
*   Not intrusive. You can bind interfaces and classes without modifying them (e.g. from a third party library that doesn't use Fruit).
*   Reduces the need of `#include`s. The header file of a component only includes the interfaces exposed by the component. The implementation classes and the interfaces that the component doesn't expose (for example, private interfaces that the client code doesn't need to know about) don't need to be included. So after changing the binding of a type in a component (as long as the interfaces exposed by the component remain the same) only the component itself needs to be re-compiled. Any components and injectors that use that component **don't** need to. This makes compilation of large projects much faster than an include-all-the-classes-I-need-to-inject approach.
*   Helps with binary compatibility: as a consequence of the previous point, since the client code doesn't include the implementation classes (not even the header files) if the interfaces exported by the component didn't change the compiled client code is binary compatible with the new implementation.
*   No internal static data. This allows the creation of separate injectors in different parts of a system (even concurrently), which might bind the same type in different ways.
*   Conditional injection based on runtime conditions. This allows to decide what to inject based on e.g. flags passed to the executable or an XML file loaded at runtime.
    *   Note that you don't need special support in Fruit for the way that you use to decide what to inject. For example, if you'd like to determine the classes to inject based on the result of an RPC to a server sent using a proprietary RPC framework, you can do this and you don't need to modify Fruit.
*   The combination of the previous two features means that at runtime you can decide to create a separate injector with a different configuration. E.g. think of a web server that receives a notification to reconfigure itself, creates a component with the new configuration for new requests and then deletes the old one when there are no more requests using it, never stopping serving requests.
*   Optional eager injection: after calling a specific method on the injector, multiple threads can use the same injector concurrently with no locking.
*   Multi-bindings: unlike the typical binding when in an injector there's a single binding for each type, multi-bindings allow components to specify several bindings and the collection of bound instances can be retrieved from the injector. This can be useful for e.g. plugin loading/hooks, or to register request handlers in a server.

Eager to get started? Jump to the [Getting started page](https://github.com/google/fruit/wiki/tutorial:-getting-started)

Look at the [examples/](https://github.com/google/fruit/tree/master/examples) directory in the source tree for example code, or see the [FAQ page](faq) for more information.

#### Rejected features

*   Compile-time detection of multiple inconsistent bindings. This feature has been rejected because it would interfere with some of the features above that are considered more important (conditional injection, binary compatibility, few includes).
*   Injection scopes, e.g. binding a type/value only for the duration of a request. This feature was implemented and then removed, replaced by the use of `NormalizedComponent`. If you need to create many injectors that have most of the bindings in common, `NormalizedComponent` allows to save most of work involved in the injector creation (but there will still be separate injectors). See [the server page in the tutorial](https://github.com/google/fruit/wiki/tutorial:-server) for an example use of `NormalizedComponent` with per-request injectors.

Do you have a feature in mind that's not in the above list? Drop me an email ([poletti.marco@gmail.com](mailto:poletti.marco@gmail.com)), I'm interested to hear your idea and I'll implement it if feasible.

### License

The code is released under the Apache 2.0 license. See the [COPYING](https://github.com/google/fruit/blob/master/COPYING) file for more details.

This project is not an official Google project. It is not supported by Google and Google specifically disclaims all warranties as to its quality, merchantability, or fitness for a particular purpose.

### Contact information

Currently [Marco Poletti](https://github.com/poletti-marco) ([poletti.marco@gmail.com](mailto:poletti.marco@gmail.com)) is the only regular developer of this project.

Occasional contributors: [Jason Mealler](https://github.com/jmealler), [Maxwell Koo](https://github.com/mjkoo)
