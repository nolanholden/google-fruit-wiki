
In some situations several implementation types might have the same "natural interface" they implement, except that only
one of these types can be bound to it within a single injector. For example, a car might have a main brake and an
emergency brake, which both implement the same `Brake` interface.

Intuitively, we'd want a component like:

    Component<???> getBrakesComponent() {
        return fruit::createComponent()
            .bind<Brake, MainBrakeImpl>()
            // This doesn't actually work, Brake is already bound
            .bind<Brake, EmergencyBrakeImpl>();
    }

And a way to inject `MainBrakeImpl` and `EmergencyBrakeImpl` instead of just `Brake`. While it's of course possible to
inject the two concrete classes, doing so requires including their definitions (that would otherwise be in the `.cpp`
files that define the components) into all places that need to inject one or the other.

Assisted injection is a feature of Fruit that helps with these situations (if you're familiar with Guice, this should
sound familiar). Let's see how we can use it to solve this example. The full sources are available at
[examples/annotated_injection](https://github.com/google/fruit/tree/master/examples/annotated_injection).

Let's start with the `Brake` interface, which is straightforward (no sign of annotated injection yet):

    // brake.h
    class Brake {
    public:
        // Activates the brake. Throws an exception if braking failed.
        virtual void activate() = 0;
    };

Then we can write an implementation of `Brake` marked as the main brake as follows:

    // main_brake.h
    struct MainBrake {};
    
    fruit::Component<fruit::Annotated<MainBrake, Brake>> getMainBrakeComponent();

<div/>

    // main_brake.cpp
    class MainBrakeImpl : public Brake {
    public:
        INJECT(MainBrakeImpl()) = default;
      
        void activate() override {
            // ...
        }
    };
    
    fruit::Component<fruit::Annotated<MainBrake, Brake>> getMainBrakeComponent() {
        return fruit::createComponent()
            .bind<fruit::Annotated<MainBrake, Brake>, MainBrakeImpl>();
    }

`fruit::Annotated<MainBrake, Brake>` is a way to mark the type `Brake` with the "annotation" `MainBrake`. The annotation
type can be any type, but to avoid confusion it's preferable to define a type just for this purpose, for example as an
empty struct as we do here.

Fruit accepts annotated types in many places. For example here we see it passed as a parameter to `Component` and
`bind()`. Note that it's not necessary to annotate `MainBrakeImpl` too: there's only one binding of that type in the
injector, and when binding types with `bind()` the two types don't need to have the same annotation. It would be
possible though (it's just unnecessary in this case).

We can then define an emergency brake class on the exact same way:

    // emergency_brake.h
    struct EmergencyBrake {};
    
    fruit::Component<fruit::Annotated<EmergencyBrake, Brake>>
        getEmergencyBrakeComponent();

<div />

    // emergency_brake.cpp
    class EmergencyBrakeImpl : public Brake {
    public:
        INJECT(EmergencyBrakeImpl()) = default;
      
        void activate() override {
            // ...
        }
    };
    
    fruit::Component<fruit::Annotated<EmergencyBrake, Brake>>
        getEmergencyBrakeComponent() {
        
        return fruit::createComponent()
            .bind<fruit::Annotated<EmergencyBrake, Brake>, EmergencyBrakeImpl>();
    }

We can then inject the two brakes in our `Car` class, as follows:

    // car.cpp
    class CarImpl : public Car {
    private:
        Brake* mainBrake;
        Brake* emergencyBrake;
      
    public:
        INJECT(CarImpl(ANNOTATED(MainBrake, Brake*) mainBrake,
                       ANNOTATED(EmergencyBrake, Brake*) emergencyBrake))
            : mainBrake(mainBrake), emergencyBrake(emergencyBrake) {
        }
        // Or:
        // using Inject = CarImpl(fruit::Annotated<MainBrake, Brake*>,
        //                        fruit::Annotated<EmergencyBrake, Brake*>);
        // CarImpl(Brake* mainBrake, Brake* emergencyBrake)
        //     : mainBrake(mainBrake), emergencyBrake(emergencyBrake) {
        // }
      
        void brake() override {
            try {
                mainBrake->activate();
            } catch (...) {
                // The main brake failed!
                emergencyBrake->activate();
            }
        }
    };
    
    fruit::Component<Car> getCarComponent() {
        return fruit::createComponent()
            .bind<Car, CarImpl>()
            .install(getMainBrakeComponent)
            .install(getEmergencyBrakeComponent);
    }

Note the use of the `ANNOTATED` macro inside `INJECT`, the equivalent `Inject` typedef, and the fact that the two
parameters of the constructor are of type `Brake*` (i.e., `ANNOTATED` only adds the annotation to the parameter type in
the `Inject` typedef, but not to the constructor parameter).

In this case both parameters were marked with `ANNOTATED`, but it's of course possible to mix annotated parameters with
non-annotated ones in the same class.

This diagram models the Car component above:

<p align="center">
    <img src="car_component.png" />
</p>

In the [next part of the tutorial](https://github.com/google/fruit/wiki/tutorial:-errors) we'll see how Fruit reports
injection errors.
