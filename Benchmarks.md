### Methodology

All the following benchmarks have been executed on an Intel Core i7-5820K @ 3.30GHz (12 virtual cores). All runtime
benchmarks below are single-threaded (Fruit is not multithreaded, even though it's reentrant and thread-safe), so the
number of cores doesn't matter for the runtime performance benchmarks (while it does matter for the compile-time
benchmark).

The following compiler options were used: `-std=c++11 -O2 -DNDEBUG`

All values use 95% confidence intervals and have been rounded to 2 significant digits.

### Benchmarks of runtime performance

The following tables summarize the benchmark results for a medium-sized system with 100 injected interfaces and 100
injected implementation classes (called "100 classes" below) and a large system with 1000 injected interfaces and
1000 injected implementation classes (called "1000 classes" below).

In both cases, 10% of the classes had no dependencies, and 90% had 10 dependencies each (i.e. each of them needed 10
instances of other objects from the injector to be constructed). So the number of edges in the dependency graph is
respectively 900 and 9000 (when counting interface+implementation as one node) or 1000 and 10000 (when counting
interface and implementation as separate nodes).

#### Injection time

This benchmark measures the time to create an `Injector` from a component function (including collecting bindings by
calling the various `get*Component` functions), to inject the root class of the dependency graph (which involves
injecting all the other ones too, as dependencies) and finally the time to destroy the injector (which involves
destroying all injected objects).

| Full injection time | 100 classes | 1000 classes              |
|---------------------|-------------|---------------------------|
|         Clang 4.0.0 |    81-84 us | 2000-2100 us (2.0-2.1 ms) |
|           GCC 7.0.1 |    69-71 us |          1800 us (1.8 ms) |

#### Injection time when using NormalizedComponent

This is only relevant if you use `NormalizedComponent`, see the documentation on that class for more information on when
it should be used.

This is the time required to create a `NormalizedComponent` from a component function (including collecting bindings by
calling the various `get*Component` functions). 

| Fruit normalization time | 100 classes | 1000 classes     |
|--------------------------|-------------|------------------|
|              Clang 4.0.0 |    80-82 us | 2000 us (2 ms)   |
|                GCC 7.0.1 |    62-65 us | 1800 us (1.8 ms) |

And this is the time required to create an `Injector` from the `NormalizedComponent`, to inject the root class of the
dependency graph (which involves injecting all the other ones too, as dependencies) and finally the time to destroy the
injector (which involves destroying all injected objects).

| Per-injector time | 100 classes | 1000 classes |
|-------------------|-------------|--------------|
|       Clang 4.0.0 |  2.1-2.4 us |     94-95 us |
|         GCC 7.0.1 |    2-2.1 us |        96 us |

Here we assume that all classes need to be injected. However, in a real example we would use Fruit `Provider`s to only
inject the classes that are actually needed, so the time will be much lower. You can see an example of how to do that in
the [Server](https://github.com/google/fruit/wiki/tutorial:-server) page of the tutorial.

#### "Injection" time with new/delete

Here we measure the time required to allocate the classes manually using new/delete instead of using Fruit, as a
comparison.

Compare this table with the table of the "Full injection time" (if you're not using `NormalizedComponent`) or with the
table of the "Per-injector time" (if you're using `NormalizedComponent`).

| New/delete time | 100 classes | 1000 classes |
|-----------------|-------------|--------------|
|     Clang 4.0.0 |      2.6 us |        45 us |
|       GCC 7.0.1 |  2.7-2.8 us |        37 us |

Note that in the 100 classes case, Fruit manages to be faster than raw new/delete when using `NormalizedComponent`. This
is thanks to Fruit's custom allocation strategy.

The above per-request results are impressively low considering that Fruit has to traverse the graph of dependencies
while new/delete "just" have to find some empty spots in memory.

Side note: the super-linear increase observed in the per-request time for both new/delete and Fruit injection is due to
CPU caches. Profiling the benchmark has shown a ~5% L1 instruction miss rate, due to the 1K constructors being called at
each request, that don't all fit in the L1 cache. Each edge of the dependency graph adds an O(1) cost.

### Benchmarks of compile-time performance

Since Fruit does most injection checks at compile-time using template metaprogramming, in some sense a part of it "runs"
at compile time too.

This benchmark measures the time to compile and link the example codebase of 100/1000 classes described above, with each
pair of interface+class defined in a pair of `.h` and `.cpp` files. Unlike the run-time benchmarks, these compilations
run in parallel so the number of cores of the CPU in use (6 physical, 12 virtual) is relevant.

| Fruit compile time | 100 classes | 1000 classes |
|--------------------|-------------|--------------|
|        Clang 4.0.0 |        17 s |        160 s |
|          GCC 7.0.1 |        16 s |        150 s |

Note that this is the full compile time, not just the overhead of using Fruit; compiling and linking 1000 classes
that do not use Fruit takes a non-trivial amount of time too.

### Executable size

This table contains measurements of the (stripped) executable size for the 100 and 1000 classes example codebases
described above. If executable size is a concern, it can be reduced further by disabling RTTI.

| Executable size (stripped) | 100 classes | 1000 classes |
|----------------------------|-------------|--------------|
|                Clang 4.0.0 |    0.381 MB |      3.62 MB |
|                  GCC 7.0.1 |    0.401 MB |      3.91 MB |
