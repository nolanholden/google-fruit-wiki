One of the key features of Fruit is that errors are reported at compile time. However, these error messages might be confusing at first. In this part of the tutorial we'll learn how Fruit error messages look like and what are the important parts to look for.

In this tutorial we'll see the error messages emitted by GCC (specifically, GCC 4.8.5). Clang error messages are similar.

Let's consider this example:

    struct X {
    };
    
    fruit::Component<X> getComponent() {
        return fruit::createComponent();
    }

Here we expect an error, because `X` is not explicitly bound and can't be automatically bound because it has no constructor wrapped in `INJECT` nor an `Inject` typedef.

    In file included from /usr/include/fruit/fruit.h:25:0,
                     from main.cpp:2:
    /usr/include/fruit/impl/injection_errors.h: In instantiation of ‘struct fruit::impl::NoBindingFoundError<X>’:
    /usr/include/fruit/impl/component.defn.h:58:3:   required from ‘fruit::Component<Types>::Component(fruit::PartialComponent<Bindings ...>) [with Bindings = {}; Params = {X}]’
    main.cpp:8:33:   required from here
    /usr/include/fruit/impl/injection_errors.h:33:3: error: static assertion failed: No explicit binding nor C::Inject definition was found for T.
       static_assert(
       ^
    In file included from /usr/include/fruit/component.h:498:0,
                     from /usr/include/fruit/fruit.h:28,
                     from main.cpp:2:
    /usr/include/fruit/impl/component.defn.h: In instantiation of ‘fruit::Component<Types>::Component(fruit::PartialComponent<Bindings ...>) [with Bindings = {}; Params = {X}]’:
    main.cpp:8:33:   required from here
    /usr/include/fruit/impl/component.defn.h:61:3: error: no type named ‘Result’ in ‘fruit::impl::meta::OpForComponent<>::ConvertTo<fruit::impl::meta::Comp<fruit::impl::meta::Vector<>, fruit::impl::meta::Vector<fruit::impl::meta::Type<X> >, fruit::impl::meta::Vector<fruit::impl::meta::Pair<fruit::impl::meta::Type<X>, fruit::impl::meta::Vector<> > >, fruit::impl::meta::Vector<>, fruit::impl::meta::EmptyList> > {aka struct fruit::impl::meta::Error<fruit::impl::NoBindingFoundErrorTag, X>}’
   (void)typename fruit::impl::meta::CheckIfError<fruit::impl::meta::Eval<fruit::impl::meta::CheckNoLoopInDeps(typename Op::Result)>>::type();
       ^
    In file included from /usr/include/fruit/component.h:498:0,
                     from /usr/include/fruit/fruit.h:28,
                     from main.cpp:2:
    /usr/include/fruit/impl/component.defn.h:64:15: error: no match for call to ‘(Op {aka fruit::impl::meta::Error<fruit::impl::NoBindingFoundErrorTag, X>}) (fruit::impl::ComponentStorage&)’
       Op()(storage);
                   ^`

When you receive Fruit error messages, you should only look at the first error. Different compilers have varying amount of success in reporting useful errors after the first one. So let's focus on the first error:

    In file included from /usr/include/fruit/fruit.h:25:0,
                     from main.cpp:2:
    /usr/include/fruit/impl/injection_errors.h: In instantiation of ‘struct fruit::impl::NoBindingFoundError<X>’:
    /usr/include/fruit/impl/component.defn.h:58:3:   required from ‘fruit::Component<Types>::Component(fruit::PartialComponent<Bindings ...>) [with Bindings = {}; Params = {X}]’
    main.cpp:8:33:   required from here
    /usr/include/fruit/impl/injection_errors.h:33:3: error: static assertion failed: No explicit binding nor C::Inject definition was found for T.
       static_assert(
       ^

From here you can see that:

*   the error is a `NoBindingFoundError` for the type `X` (third line)
*   it was caused by `main.cpp:8`
*   The detailed error message is `No explicit binding nor C::Inject definition was found for T.`. Note that in this message, T refers to the type in the `*Error<>` instantiation (in this case, `T` is `X`).

If we now change the code above to:

    struct X {
        INJECT(X()) = default;
    };
    
    fruit::Component<X> getComponent() {
        return fruit::createComponent();
    }

Then the file will compile.

This is how Fruit reports missing bindings, one of the most common injection errors.

In the [next part of the tutorial](https://github.com/google/fruit/wiki/tutorial:-assisted-injection) we'll learn how to use assisted injection.
