In this part of the tutorial, we'll build a component that scales double values by a constant factor. The full source is available in [examples/scaling_doubles](https://github.com/google/fruit/tree/master/examples/scaling_doubles).
To warm up, let's write a Multiplier component.

    // multiplier.h
    class Multiplier {
    public:
        // Returns the product of x and y.
        virtual double multiply(double x, double y) = 0;
    };
    
    fruit::Component<Multiplier> getMultiplierComponent();

    // multiplier.cpp
    #include "multiplier.h"
    class MultiplierImpl : public Multiplier {
    public:
        double multiply(double x, double y) override {
            return x * y;
        }
    };
    
    fruit::Component<Multiplier> getMultiplierComponent() {
        return fruit::createComponent()
            .bind<Multiplier, MultiplierImpl>()
            .registerConstructor<MultiplierImpl()>();
    }

![Multiplier](https://sites.google.com/site/fruitlib/_/rsrc/1463297953470/tutorial/assisted-injection/multiplier.png)

When there is a canonical implementation of an interface, as in this case, we can put the definition of the interface and the get*Component() function declaration in the same header file, saving some boilerplate.

Note that the MultiplierImpl constructor was _not_ wrapped with `INJECT()`. It's a convenient way to tell Fruit what injector to use, but it can't be used in some cases, for example if MultiplierImpl was provided by another project that doesn't use Fruit. Here we could use it, but we don't just to show the (equivalent) explicit registerConstructor call with the constructor signature.

    // scaler.h
    class Scaler {
    public:
        virtual double scale(double x) = 0;
    };
    
    using ScalerFactory = std::function<std::unique_ptr<Scaler>(double)>;
    
    fruit::Component<ScalerFactory> getScalerComponent();

At first, one might be tempted to write `Component<Scaler>`. However, there is no single Scaler implementation, there is a Scaler for each double value (scaling factor). So we expose this in the component signature. When returning a value type we can just return it by value, but for interfaces it's better to return a unique_ptr so that we're sure that the object will be destroyed. Fruit can inject any class, but there is special support for types of the form `std::function<std::unique_ptr<T>(...)>` as we'll see shortly.
Let's write the implementation now.

    // scaler.cpp
    #include "scaler.h"
    #include "multiplier.h"
    
    using fruit::Component;
    using fruit::Injector;
    using fruit::createComponent;
    
    class ScalerImpl : public Scaler {
    private:
        Multiplier* multiplier;
        double factor;

    public:
        INJECT(ScalerImpl(ASSISTED(double) factor, Multiplier* multiplier))
            : multiplier(multiplier), factor(factor) {
        }
    
        double scale(double x) override {
            return multiplier->multiply(x, factor);
        }
    };
    
    Component<ScalerFactory> getScalerComponent() {
        return createComponent()
            .bind<Scaler, ScalerImpl>()
            .install(getMultiplierComponent());
    }

![Scaler](https://sites.google.com/site/fruitlib/_/rsrc/1463297953470/tutorial/assisted-injection/scaler.png)

Here we see for the first time the use of the `ASSISTED()` macro. It's used to mark types in a constructor wrapped with `INJECT()` that don't have to be injected, but will become the parameters of an injected factory function.

Note that here we installed the multiplier component directly, instead of declaring it as a requirement and then writing a component that composes ScalerImplComponent and MultiplierComponent. It's a tradeoff: this slightly reduces modularity, but makes the code more concise (avoids 2 files). There is no definite answer on what's better; this tradeoff must be evaluated on a case-by-case basis.

Note also that the bind operation now is doing something different: instead of binding Scaler to ScalerImpl, it's binding an `std::function<std::unique_ptr<Scaler>(double)>` to a `std::function<std::unique_ptr<ScalerImpl>(double)>`.

    // main.cpp
    #include "scaler.h"
    
    using fruit::Injector;
    
    int main() {
        Injector<ScalerFactory> injector(getScalerComponent());
        ScalerFactory scalerFactory(injector);
        
        std::unique_ptr<Scaler> scaler = scalerFactory(12.1);
        std::cout << scaler->scale(3) << std::endl;
    
        return 0;
    }

Here is the `main()` function, which should be easy to understand at this point. We get an instance of the factory from the injector, and then we call the factory ourselves to get an instance of Scaler.

Now you know the basics of how to use Fruit. To get more familiar with it, try rewriting the above system by yourself or any other program that you like, and experiment a bit changing the code.

In the [next part of the tutorial](server) we'll see how to use Fruit to write a server.
