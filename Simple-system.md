In this second part of the tutorial we'll implement a simple command line utility that reads a number and outputs the number incremented by 1. In a real-world system such a simple functionality would be in a single class (or even all in `main()`). Here we instead (over)use dependency injection to split functionality as much as possible, showing how Fruit can help building a complex system from several components, while limiting the dependencies between components.

In the following, we omit the parts of the code that are not relevant (e.g. include guards, system includes). The full source is available in [examples/simple_injection](https://github.com/google/fruit/tree/master/examples/simple_injection).

We start by writing the interface for the increment:

    // incrementer.h
    class Incrementer {
    public:
        // Returns x + 1.
        virtual int increment(int x) = 0;
    };

An increment is a special case of addition, so let's also write an interface for a class that does addition:

    // adder.h
    class Adder {
    public:
        // Returns the sum of x and y.
        virtual int add(int x, int y) = 0;
    };

Now we want a component that, given an implementation of Adder, provides an implementation of Incrementer.

    // incrementer_impl.h
    #include "incrementer.h"
    #include "adder.h"
    
    fruit::Component<fruit::Required<Adder>, Incrementer> getIncrementerImplComponent();

    // incrementer_impl.cpp
    #include "incrementer_impl.h"
    class IncrementerImpl : public Incrementer {
    private:
        Adder* adder;
    
    public:
        INJECT(IncrementerImpl(Adder* adder))
            : adder(adder) {
        }
    
        virtual int increment(int x) override {
            return adder->add(x, 1);
        }
    };
    
    fruit::Component<fruit::Required<Adder>, Incrementer> getIncrementerImplComponent() {
        return fruit::createComponent()
            .bind<Incrementer, IncrementerImpl>();
    }

![Incrementer](https://sites.google.com/site/fruitlib/_/rsrc/1424518335448/tutorial/simple-system/incrementer.png)

This is our first encounter with a component that has requirements. A Fruit component can have required types, and these types are specified using `fruit::Required<T1, ..., Tn>` as the first type argument of a component. If the signature of `getIncrementerImplComponent()` didn't specify the requirement, Fruit would have looked for a binding for Adder and, not finding it, would have aborted the compilation with an error. All required types that a component doesn't bind must be exposed.

Note that the only information exposed in the header file is what the module requires and provides. The `IncrementerImpl` class is defined in the .cpp file only. This is similar to what happens using the [Pimpl idiom](http://en.wikipedia.org/wiki/Pimpl).

Side note: readers used to C++ might have expected an anonymous namespace wrapping the class. Adding it is reasonable, as the class is only used within the .cpp file. We have omitted the anonymous namespace here to make the code easy to understand.

The lack of virtual destructors is also intended, as Fruit handles the destruction of the objects in the injector, and it destroys the concrete classes directly (not through a pointer to the base class). The compiler will report no warnings, not even with "-W -Wall".

Now, let's implement Adder.

    // simple_adder.h
    #include "adder.h"
    
    fruit::Component<Adder> getSimpleAdderComponent();

    // simple_adder.cpp
    #include "simple_adder.h"
    
    class SimpleAdder : public Adder {
    public:
        INJECT(SimpleAdder()) = default;
    
        virtual int add(int x, int y) override {
            return x + y;
        }
    };
    
    fruit::Component<Adder> getSimpleAdderComponent() {
        return fruit::createComponent()
            .bind<Adder, SimpleAdder>();
    }

![Simple adder](https://sites.google.com/site/fruitlib/_/rsrc/1424518379296/tutorial/simple-system/simple_adder.png)

This component is very simple, it should be self-explanatory at this point.

So, we have a component that provides `Adder` and one that requires `Adder` and provides `Incrementer`. Now we want to compose the two.

    // simple_incrementer.h
    #include "incrementer.h"
    
    fruit::Component<Incrementer> getSimpleIncrementerComponent();

    // simple_incrementer.cpp
    #include "simple_incrementer.h"
    
    #include "incrementer_impl.h"
    #include "simple_adder.h"
    
    fruit::Component<Incrementer> getSimpleIncrementerComponent() {
        return fruit::createComponent()
            .install(getIncrementerImplComponent())
            .install(getSimpleAdderComponent());
    }

![Simple incrementer](https://sites.google.com/site/fruitlib/_/rsrc/1424518443045/tutorial/simple-system/simple_incrementer.png)

`install()` is an operation that is used to "install" a sub-component inside the current component. As we've already seen in the previous page of the tutorial, note that here we are not exposing all the interfaces that we could. The Adder interface is considered an implementation detail, so we don't want to expose it. Note that it's not even included in the header file, only the .cpp file depends on it (indirectly, through `simple_adder.h`). This is an example of how Fruit helps reduce the number of includes (and therefore also the compilation time) of large projects. Without dependency injection, in order to expose an implementation of Incrementer we would have included IncrementerImpl, and the IncrementerImpl implementation would have included SimpleAdder. With Dependency injection but without Fruit, the IncrementerImpl implementation would no longer include SimpleAdder, but the client code would have to include both IncrementerImpl and SimpleAdder.

Phew! So much for incrementing a number.

Now that the implementation part is complete, we just need to write the `main()` function.

    #include "simple_incrementer.h"
    
    using fruit::Component;
    using fruit::Injector;
    
    int main() {
        Injector<Incrementer> injector(getSimpleIncrementerComponent());
        Incrementer* incrementer = injector.get<Incrementer*>();

        int x;
        std::cin >> x;
        std::cout << incrementer->increment(x) << std::endl;

        return 0;
    }

We construct an Injector from the component, then get an instance of Incrementer from the injector. No need to include any class definition here.

After releasing the above program with a hefty price tag (or open-source, depending on your taste) we get some customer feedback. Some customers are happy but some noticed that incrementing a big number sometimes yields a negative number. They would like us to add a `--checked` option that enables overflow checking.

Our implementation code is modular (thanks to dependency injection), so we don't need to modify any of the above components. Also, the Incrementer component delegates the work to Adder, so the only thing that we need is a checked implementation of Adder.

    // checked_adder.h
    #include "adder.h"
    
    fruit::Component<Adder> getCheckedAdderComponent();

    // checked_adder.cpp
    #include "checked_adder.h"
    
    class CheckedAdder : public Adder {
    private:
        bool add_overflows(int x, int y) {
            ... // Implementation here
        }

    public:
        INJECT(CheckedAdder()) = default;
    
        virtual int add(int x, int y) override {
            if (add_overflows(x, y)) {
                std::cerr << "CheckedAdder: detected overflow during addition of " << x << " and " << y << std::endl;
                abort();
            }
            return x + y;
        }
    };
    
    fruit::Component<Adder> getCheckedAdderComponent() {
        return fruit::createComponent()
            .bind<Adder, CheckedAdder>();
    }


![Checked adder](https://sites.google.com/site/fruitlib/_/rsrc/1424518475701/tutorial/simple-system/checked_adder.png)

Nothing new here. Now we assemble the new component with the IncrementComponent that we've written before.

    // checked_incrementer.h
    #include "incrementer.h"
    
    fruit::Component<Incrementer> getCheckedIncrementerComponent();

    // checked_incrementer.cpp
    #include "checked_incrementer.h"
    
    #include "incrementer_impl.h"
    #include "checked_adder.h"
    
    fruit::Component<Incrementer> getCheckedIncrementerComponent() {
        return fruit::createComponent()
            .install(getIncrementerImplComponent())
            .install(getCheckedAdderComponent());
    }

![Checked incrementer](https://sites.google.com/site/fruitlib/_/rsrc/1424518627367/tutorial/simple-system/checked_incrementer.png)

Except the reuse of the same `IncrementerComponent`, there's nothing interesting to note here.

    // incrementer_component.h
    #include "incrementer.h"
    
    fruit::Component<Incrementer> getIncrementerComponent(bool checked);

    // incrementer_component.cpp
    #include "incrementer_component.h"
    
    #include "simple_incrementer.h"
    #include "checked_incrementer.h"
    
    fruit::Component<Incrementer> getIncrementerComponent(bool checked) {
        if (checked)
            return getCheckedIncrementerComponent();
        else
            return getSimpleIncrementerComponent();
    }

![Incrementer component](https://sites.google.com/site/fruitlib/_/rsrc/1424518679355/tutorial/simple-system/incrementer_component.png)

A `get*Component()` is just a function after all, so we can have parameterized components just by adding parameters and using `if` inside the function with multiple returns.

Let's now rewrite the `main()` to accept the new argument, and to use the new parametrized module.

    // main.cpp
    #include "incrementer_component.h"
    
    using fruit::Component;
    using fruit::Injector;
    
    // Try e.g.:
    // echo 5 | ./incrementer
    // echo 2147483647 | ./incrementer
    // echo 2147483647 | ./incrementer --checked
    int main(int argc, const char* argv[]) {
        bool checked = false;
        if (argc == 2 && std::string(argv[1]) == "--checked")
            checked = true;
        
        Injector<Incrementer> injector(getIncrementerComponent(checked));
        Incrementer* incrementer(injector);
        
        int x;
        std::cin >> x;
        std::cout << incrementer->increment(x) << std::endl;
        
        return 0;
    }

Here we see an alternative syntax for obtaining a class instance from an injector. Instead of calling `get` on the injector, we convert the injector to the type that we want. This is convenient to avoid repeating the type twice (compare with the main function above).

In the [next part of the tutorial](tutorial-annotated-injection) we'll learn how to use annotated injection.
