### Methodology

All the following benchmarks have been executed on an Intel Core i7-5820K @ 3.30GHz (12 virtual cores). The number of cores is not particularly important since Fruit is not multithreaded (even though it's reentrant and thread-safe).

The following compiler options were used: `-std=c++11 -O2 -DNDEBUG`

All values have been rounded to 2 significant digits.

### Benchmarks of runtime performance

The following tables summarize the benchmark results for a medium-sized system with 100 injected classes and a large system with 1000 injected classes.

In both cases, 10% of the classes had no dependencies, and 90% had 10 dependencies each (i.e. each of them needed 10 instances of other objects from the injector to be constructed). So the number of edges in the dependency graph was respectively 900 and 9000.


Below we have separate tables for:

*   the startup time: the time for the creation of the `NormalizedComponent`, usually done once at startup
*   the per-request time: the time for the creation of a per-request injector from the `NormalizedComponent`, for the injection itself (creation of instances of all classes in the injector) and for the destruction of those objects.

We also have a third table for the corresponding time when managing dependencies explicitly without Fruit, using new/delete on a class C derived from a class I, both with a virtual destructor (as for the classes used in the benchmarks using Fruit). Compare this table with the table for the per-request time. In the second table, the difference with the corresponding cell of the third table is also shown.

| Startup time | 100 classes      | 1000 classes     |
|-------------------------|------------------|------------------|
| GCC 4.8.5   | 340 us (0.34 ms) | 4100 us (4.1 ms) |
| GCC 5.3.1   | 340 us (0.34 ms) | 4100 us (4.1 ms) |
| Clang 3.7.0 | 300 us (0.30 ms) | 3200 us (3.2 ms) |

The above table of startup time is shown for completeness, but even the ~6 ms of startup overhead when using 1000 classes are easily dwarfed by other initializations (or even just the loader).

| Per-request time  | 100 classes      | 1000 classes     |
|-----------------------------|------------------|------------------|
| GCC 4.8.5   | 2.7 us (+1 us)   | 100 us (+53 us)  |
| GCC 5.3.1   | 2.6 us (+0 us)   | 100 us (+66 us)  |
| Clang 3.7.0 | 3.6 us (+1.2 us) | 150 us (+103 us) |

| new/delete time | 100 classes | 1000 classes |
|-------------|--------|-------|
| GCC 4.8.5   | 1.7 us | 47 us |
| GCC 5.3.1   | 2.6 us | 34 us |
| Clang 3.7.0 | 2.4 us | 47 us |


So for a system with 100 classes the per-request performance difference is negligible (compared to using just new/delete). Fruit can compete with new/delete because it uses its own allocation strategy.

For a system with 1000 classes the per-request performance difference is about 50-60us on GCC and 100us on Clang. Note that this is when all the 1000 classes need to be constructed to process the request; in that case, the overhead should be a small percentage of the total processing, and otherwise the overhead will be much lower. In the latter case, it's possible to perform lazy injection using Provider objects.

So in both cases, creating an injector per request seems reasonable (as long as the `NormalizedComponent` is created at startup, to avoid paying its ms-range overhead for each request).

The above per-request results are impressively low considering that Fruit has to traverse the graph of dependencies while new/delete "just" have to find some empty spots in memory.

Side note: the super-linear increase observed in the per-request time for both new/delete and Fruit injection is due to CPU caches. Profiling the injection benchmark has shown a ~5% L1 instruction miss rate, due to the 1K constructors being called at each request, that don't all fit in the L1 cache.
Each edge of the dependency graph adds an O(1) cost.

### Benchmarks of compile-time performance

Since Fruit does most injection checks at compile-time using template metaprogramming, in some sense a part of it "runs" at compile time too.

It's hard to measure exactly how much time is spent "inside Fruit during compile time". To achieve an approximation (overestimating) we have created a component with N bindings (for N=20,80,320) with an equal mix of `bind()`, `bindInstance()`, `install()`, auto-registered constructors and `registerProvider()`. Then we timed the compilation with various compilers (to be precise: the full compilation, not just type-checking, but without linking).

| Compile time |  20 bindings | 80 bindings | 320 bindings |
|-------------|--------------|-------------|--------------|
| GCC 4.8.5   | 0.72 s       | 1.6 s       | 44 s         |
| GCC 5.3.1   | 0.83 s       | 1.9 s       | 46 s         |
| Clang 3.7.0 | 0.95 s       | 2.3 s       | 14 s         |

Note that the last two examples are huge components, a real system would be more likely to many small components instead.

Installing a component A in a component B is equivalent to a single binding as far as compile-time performance is concerned, it adds an O(1) overhead to the compile-time of B, no matter how many bindings are in A; it does _not_ add an O(num_bindings_in(A)) overhead. So even a system with 1000 types might only have up to (say) 20 bindings per component.
