# <a name="S-concurrency"></a> CP: Concurrency and Parallelism

???

Concurrency and parallelism rule summary:

* [CP.1: Assume that your code will run as part of a multi-threaded program](#Rconc-multi)
* [CP.2: Avoid data races](#Rconc-races)

See also:

* [CP.con: Concurrency](#SScp-con)
* [CP.par: Parallelism](#SScp-par)
* [CP.simd: SIMD](#SScp-simd)
* [CP.free: Lock-free programming](#SScp-free)

### <a name="Rconc-multi"></a> CP.1: Assume that your code will run as part of a multi-threaded program

##### Reason

It is hard to be certain that concurrency isn't used now or sometime in the future.
Code gets re-used.
Libraries using threads my be used from some other part of the program.

##### Example

    ???

**Exception**: There are examples where code will never be run in a multi-threaded environment.
However, there are also many examples where code that was "known" to never run in a multi-threaded program
was run as part of a multi-threaded program. Often years later.
Typically, such programs lead to a painful effort to remove data races.

### <a name="Rconc-races"></a> CP.2: Avoid data races

##### Reason

Unless you do, nothing is guaranteed to work and subtle errors will persist.

##### Note

If you have any doubts about what this means, go read a book.

##### Enforcement

Some is possible, do at least something.

## <a name="SScp-con"></a> CP.con: Concurrency

???

Concurrency rule summary:

* ???
* ???

???? should there be a "use X rather than `std::async`" where X is something that would use a better specified thread pool?

Speaking of concurrency, should there be a note about the dangers of `std::atomic` (weapons)?
A lot of people, myself included, like to experiment with `std::memory_order`, but it is perhaps best to keep a close watch on those things in production code.
Even vendors mess this up: Microsoft had to fix their `shared_ptr` (weak refcount decrement wasn't synchronized-with the destructor, if I recall correctly, although it was only a problem on ARM, not Intel)
and everyone (gcc, clang, Microsoft, and Intel) had to fix their `compare_exchange_*` this year, after an implementation bug caused losses to some finance company and they were kind enough to let the community know.

It should definitely be mentioned that `volatile` does not provide atomicity, does not synchronize between threads, and does not prevent instruction reordering (neither compiler nor hardware), and simply has nothing to do with concurrency.

    if (source->pool != YARROW_FAST_POOL && source->pool != YARROW_SLOW_POOL) {
        THROW(YARROW_BAD_SOURCE);
    }

??? Is `std::async` worth using in light of future (and even existing, as libraries) parallelism facilities? What should the guidelines recommend if someone wants to parallelize, e.g., `std::accumulate` (with the additional precondition of commutativity), or merge sort?

???UNIX signal handling???. May be worth reminding how little is async-signal-safe, and how to communicate with a signal handler (best is probably "not at all")

## <a name="SScp-par"></a> CP.par: Parallelism

???

Parallelism rule summary:

* ???
* ???

## <a name="SScp-simd"></a> CP.simd: SIMD

???

SIMD rule summary:

* ???
* ???

## <a name="SScp-free"></a> CP.free: Lock-free programming

???

Lock-free programming rule summary:

* ???
* ???

### <a name="Rconc"></a> Don't use lock-free programming unless you absolutely have to

##### Reason

It's error-prone and requires expert level knowledge of language features, machine architecture, and data structures.

**Alternative**: Use lock-free data structures implemented by others as part of some library.
