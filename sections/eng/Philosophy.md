
# <a name="S-philosophy"></a>P: Philosophy

The rules in this section are very general.

Philosophy rules summary:

* [P.1: Express ideas directly in code](#Rp-direct)
* [P.2: Write in ISO Standard C++](#Rp-Cplusplus)
* [P.3: Express intent](#Rp-what)
* [P.4: Ideally, a program should be statically type safe](#Rp-typesafe)
* [P.5: Prefer compile-time checking to run-time checking](#Rp-compile-time)
* [P.6: What cannot be checked at compile time should be checkable at run time](#Rp-run-time)
* [P.7: Catch run-time errors early](#Rp-early)
* [P.8: Don't leak any resources](#Rp-leak)
* [P.9: Don't waste time or space](#Rp-waste)
* [P.10: Prefer immutable data to mutable data](#Rp-mutable)
* [P.11: Encapsulate messy constructs, rather than spreading through the code](#Rp-library)
* [P.12: Use supporting tools as appropriate](#Rp-tools)
* [P.13: Use support libraries as appropriate](#Rp-lib)

Philosophical rules are generally not mechanically checkable.
However, individual rules reflecting these philosophical themes are.
Without a philosophical basis, the more concrete/specific/checkable rules lack rationale.

### <a name="Rp-direct"></a>P.1: Express ideas directly in code

##### Reason

Compilers don't read comments (or design documents) and neither do many programmers (consistently).
What is expressed in code has defined semantics and can (in principle) be checked by compilers and other tools.

##### Example
```c++
    class Date {
        // ...
    public:
        Month month() const;  // do
        int month();          // don't
        // ...
    };
```
The first declaration of `month` is explicit about returning a `Month` and about not modifying the state of the `Date` object.
The second version leaves the reader guessing and opens more possibilities for uncaught bugs.

##### Example; bad

This loop is a restricted form of `std::find`:
```c++
    void f(vector<string>& v)
    {
        string val;
        cin >> val;
        // ...
        int index = -1;                    // bad, plus should use gsl::index
        for (int i = 0; i < v.size(); ++i) {
            if (v[i] == val) {
                index = i;
                break;
            }
        }
        // ...
    }
```
##### Example; good

A much clearer expression of intent would be:
```c++
    void f(vector<string>& v)
    {
        string val;
        cin >> val;
        // ...
        auto p = find(begin(v), end(v), val);  // better
        // ...
    }
```
A well-designed library expresses intent (what is to be done, rather than just how something is being done) far better than direct use of language features.

A C++ programmer should know the basics of the standard library, and use it where appropriate.
Any programmer should know the basics of the foundation libraries of the project being worked on, and use them appropriately.
Any programmer using these guidelines should know the [guidelines support library](#S-gsl), and use it appropriately.

##### Example
```c++
    change_speed(double s);   // bad: what does s signify?
    // ...
    change_speed(2.3);
```
A better approach is to be explicit about the meaning of the double (new speed or delta on old speed?) and the unit used:
```c++
    change_speed(Speed s);    // better: the meaning of s is specified
    // ...
    change_speed(2.3);        // error: no unit
    change_speed(23m / 10s);  // meters per second
```
We could have accepted a plain (unit-less) `double` as a delta, but that would have been error-prone.
If we wanted both absolute speed and deltas, we would have defined a `Delta` type.

##### Enforcement

Very hard in general.

* use `const` consistently (check if member functions modify their object; check if functions modify arguments passed by pointer or reference)
* flag uses of casts (casts neuter the type system)
* detect code that mimics the standard library (hard)

### <a name="Rp-Cplusplus"></a>P.2: Write in ISO Standard C++

##### Reason

This is a set of guidelines for writing ISO Standard C++.

##### Note

There are environments where extensions are necessary, e.g., to access system resources.
In such cases, localize the use of necessary extensions and control their use with non-core Coding Guidelines.  If possible, build interfaces that encapsulate the extensions so they can be turned off or compiled away on systems that do not support those extensions.

Extensions often do not have rigorously defined semantics.  Even extensions that
are common and implemented by multiple compilers may have slightly different
behaviors and edge case behavior as a direct result of *not* having a rigorous
standard definition.  With sufficient use of any such extension, expected
portability will be impacted.

##### Note

Using valid ISO C++ does not guarantee portability (let alone correctness).
Avoid dependence on undefined behavior (e.g., [undefined order of evaluation](#Res-order))
and be aware of constructs with implementation defined meaning (e.g., `sizeof(int)`).

##### Note

There are environments where restrictions on use of standard C++ language or library features are necessary, e.g., to avoid dynamic memory allocation as required by aircraft control software standards.
In such cases, control their (dis)use with an extension of these Coding Guidelines customized to the specific environment.

##### Enforcement

Use an up-to-date C++ compiler (currently C++17, C++14, or C++11) with a set of options that do not accept extensions.

### <a name="Rp-what"></a>P.3: Express intent

##### Reason

Unless the intent of some code is stated (e.g., in names or comments), it is impossible to tell whether the code does what it is supposed to do.

##### Example
```c++
    gsl::index i = 0;
    while (i < v.size()) {
        // ... do something with v[i] ...
    }
```
The intent of "just" looping over the elements of `v` is not expressed here. The implementation detail of an index is exposed (so that it might be misused), and `i` outlives the scope of the loop, which may or may not be intended. The reader cannot know from just this section of code.

Better:
```c++
    for (const auto& x : v) { /* do something with the value of x */ }
```
Now, there is no explicit mention of the iteration mechanism, and the loop operates on a reference to `const` elements so that accidental modification cannot happen. If modification is desired, say so:
```c++
    for (auto& x : v) { /* modify x */ }
```
For more details about for-statements, see [ES.71](#Res-for-range).
Sometimes better still, use a named algorithm. This example uses the `for_each` from the Ranges TS because it directly expresses the intent:
```c++
    for_each(v, [](int x) { /* do something with the value of x */ });
    for_each(par, v, [](int x) { /* do something with the value of x */ });
```
The last variant makes it clear that we are not interested in the order in which the elements of `v` are handled.

A programmer should be familiar with

* [The guidelines support library](#S-gsl)
* [The ISO C++ Standard Library](#S-stdlib)
* Whatever foundation libraries are used for the current project(s)

##### Note

Alternative formulation: Say what should be done, rather than just how it should be done.

##### Note

Some language constructs express intent better than others.

##### Example

If two `int`s are meant to be the coordinates of a 2D point, say so:
```c++
    draw_line(int, int, int, int);  // obscure
    draw_line(Point, Point);        // clearer
```
##### Enforcement

Look for common patterns for which there are better alternatives

* simple `for` loops vs. range-`for` loops
* `f(T*, int)` interfaces vs. `f(span<T>)` interfaces
* loop variables in too large a scope
* naked `new` and `delete`
* functions with many parameters of built-in types

There is a huge scope for cleverness and semi-automated program transformation.

### <a name="Rp-typesafe"></a>P.4: Ideally, a program should be statically type safe

##### Reason

Ideally, a program would be completely statically (compile-time) type safe.
Unfortunately, that is not possible. Problem areas:

* unions
* casts
* array decay
* range errors
* narrowing conversions

##### Note

These areas are sources of serious problems (e.g., crashes and security violations).
We try to provide alternative techniques.

##### Enforcement

We can ban, restrain, or detect the individual problem categories separately, as required and feasible for individual programs.
Always suggest an alternative.
For example:

* unions -- use `variant` (in C++17)
* casts -- minimize their use; templates can help
* array decay -- use `span` (from the GSL)
* range errors -- use `span`
* narrowing conversions -- minimize their use and use `narrow` or `narrow_cast` (from the GSL) where they are necessary

### <a name="Rp-compile-time"></a>P.5: Prefer compile-time checking to run-time checking

##### Reason

Code clarity and performance.
You don't need to write error handlers for errors caught at compile time.

##### Example
```c++
    // Int is an alias used for integers
    int bits = 0;         // don't: avoidable code
    for (Int i = 1; i; i <<= 1)
        ++bits;
    if (bits < 32)
        cerr << "Int too small\n";
```
This example fails to achieve what it is trying to achieve (because overflow is undefined) and should be replaced with a simple `static_assert`:
```c++
    // Int is an alias used for integers
    static_assert(sizeof(Int) >= 4);    // do: compile-time check
```
Or better still just use the type system and replace `Int` with `int32_t`.

##### Example
```c++
    void read(int* p, int n);   // read max n integers into *p

    int a[100];
    read(a, 1000);    // bad, off the end
```
better
```c++
    void read(span<int> r); // read into the range of integers r

    int a[100];
    read(a);        // better: let the compiler figure out the number of elements
```
**Alternative formulation**: Don't postpone to run time what can be done well at compile time.

##### Enforcement

* Look for pointer arguments.
* Look for run-time checks for range violations.

### <a name="Rp-run-time"></a>P.6: What cannot be checked at compile time should be checkable at run time

##### Reason

Leaving hard-to-detect errors in a program is asking for crashes and bad results.

##### Note

Ideally, we catch all errors (that are not errors in the programmer's logic) at either compile time or run time. It is impossible to catch all errors at compile time and often not affordable to catch all remaining errors at run time. However, we should endeavor to write programs that in principle can be checked, given sufficient resources (analysis programs, run-time checks, machine resources, time).

##### Example, bad
```c++
    // separately compiled, possibly dynamically loaded
    extern void f(int* p);

    void g(int n)
    {
        // bad: the number of elements is not passed to f()
        f(new int[n]);
    }
```
Here, a crucial bit of information (the number of elements) has been so thoroughly "obscured" that static analysis is probably rendered infeasible and dynamic checking can be very difficult when `f()` is part of an ABI so that we cannot "instrument" that pointer. We could embed helpful information into the free store, but that requires global changes to a system and maybe to the compiler. What we have here is a design that makes error detection very hard.

##### Example, bad

We can of course pass the number of elements along with the pointer:
```c++
    // separately compiled, possibly dynamically loaded
    extern void f2(int* p, int n);

    void g2(int n)
    {
        f2(new int[n], m);  // bad: a wrong number of elements can be passed to f()
    }
```
Passing the number of elements as an argument is better (and far more common) than just passing the pointer and relying on some (unstated) convention for knowing or discovering the number of elements. However (as shown), a simple typo can introduce a serious error. The connection between the two arguments of `f2()` is conventional, rather than explicit.

Also, it is implicit that `f2()` is supposed to `delete` its argument (or did the caller make a second mistake?).

##### Example, bad

The standard library resource management pointers fail to pass the size when they point to an object:
```c++
    // separately compiled, possibly dynamically loaded
    // NB: this assumes the calling code is ABI-compatible, using a
    // compatible C++ compiler and the same stdlib implementation
    extern void f3(unique_ptr<int[]>, int n);

    void g3(int n)
    {
        f3(make_unique<int[]>(n), m);    // bad: pass ownership and size separately
    }
```
##### Example

We need to pass the pointer and the number of elements as an integral object:
```c++
    extern void f4(vector<int>&);   // separately compiled, possibly dynamically loaded
    extern void f4(span<int>);      // separately compiled, possibly dynamically loaded
                                    // NB: this assumes the calling code is ABI-compatible, using a
                                    // compatible C++ compiler and the same stdlib implementation

    void g3(int n)
    {
        vector<int> v(n);
        f4(v);                     // pass a reference, retain ownership
        f4(span<int>{v});          // pass a view, retain ownership
    }
```
This design carries the number of elements along as an integral part of an object, so that errors are unlikely and dynamic (run-time) checking is always feasible, if not always affordable.

##### Example

How do we transfer both ownership and all information needed for validating use?
```c++
    vector<int> f5(int n)    // OK: move
    {
        vector<int> v(n);
        // ... initialize v ...
        return v;
    }

    unique_ptr<int[]> f6(int n)    // bad: loses n
    {
        auto p = make_unique<int[]>(n);
        // ... initialize *p ...
        return p;
    }

    owner<int*> f7(int n)    // bad: loses n and we might forget to delete
    {
        owner<int*> p = new int[n];
        // ... initialize *p ...
        return p;
    }
```
##### Example

* ???
* show how possible checks are avoided by interfaces that pass polymorphic base classes around, when they actually know what they need?
  Or strings as "free-style" options

##### Enforcement

* Flag (pointer, count)-style interfaces (this will flag a lot of examples that can't be fixed for compatibility reasons)
* ???

### <a name="Rp-early"></a>P.7: Catch run-time errors early

##### Reason

Avoid "mysterious" crashes.
Avoid errors leading to (possibly unrecognized) wrong results.

##### Example
```c++
    void increment1(int* p, int n)    // bad: error-prone
    {
        for (int i = 0; i < n; ++i) ++p[i];
    }

    void use1(int m)
    {
        const int n = 10;
        int a[n] = {};
        // ...
        increment1(a, m);   // maybe typo, maybe m <= n is supposed
                            // but assume that m == 20
        // ...
    }
```
Here we made a small error in `use1` that will lead to corrupted data or a crash.
The (pointer, count)-style interface leaves `increment1()` with no realistic way of defending itself against out-of-range errors.
If we could check subscripts for out of range access, then the error would not be discovered until `p[10]` was accessed.
We could check earlier and improve the code:
```c++
    void increment2(span<int> p)
    {
        for (int& x : p) ++x;
    }

    void use2(int m)
    {
        const int n = 10;
        int a[n] = {};
        // ...
        increment2({a, m});    // maybe typo, maybe m <= n is supposed
        // ...
    }
```
Now, `m <= n` can be checked at the point of call (early) rather than later.
If all we had was a typo so that we meant to use `n` as the bound, the code could be further simplified (eliminating the possibility of an error):
```c++
    void use3(int m)
    {
        const int n = 10;
        int a[n] = {};
        // ...
        increment2(a);   // the number of elements of a need not be repeated
        // ...
    }
```
##### Example, bad

Don't repeatedly check the same value. Don't pass structured data as strings:
```c++
    Date read_date(istream& is);    // read date from istream

    Date extract_date(const string& s);    // extract date from string

    void user1(const string& date)    // manipulate date
    {
        auto d = extract_date(date);
        // ...
    }

    void user2()
    {
        Date d = read_date(cin);
        // ...
        user1(d.to_string());
        // ...
    }
```
The date is validated twice (by the `Date` constructor) and passed as a character string (unstructured data).

##### Example

Excess checking can be costly.
There are cases where checking early is dumb because you may not ever need the value, or may only need part of the value that is more easily checked than the whole.  Similarly, don't add validity checks that change the asymptotic behavior of your interface (e.g., don't add a `O(n)` check to an interface with an average complexity of `O(1)`).
```c++
    class Jet {    // Physics says: e * e < x * x + y * y + z * z
        float x;
        float y;
        float z;
        float e;
    public:
        Jet(float x, float y, float z, float e)
            :x(x), y(y), z(z), e(e)
        {
            // Should I check here that the values are physically meaningful?
        }

        float m() const
        {
            // Should I handle the degenerate case here?
            return sqrt(x * x + y * y + z * z - e * e);
        }

        ???
    };
```
The physical law for a jet (`e * e < x * x + y * y + z * z`) is not an invariant because of the possibility for measurement errors.

???

##### Enforcement

* Look at pointers and arrays: Do range-checking early and not repeatedly
* Look at conversions: Eliminate or mark narrowing conversions
* Look for unchecked values coming from input
* Look for structured data (objects of classes with invariants) being converted into strings
* ???

### <a name="Rp-leak"></a>P.8: Don't leak any resources

##### Reason

Even a slow growth in resources will, over time, exhaust the availability of those resources.
This is particularly important for long-running programs, but is an essential piece of responsible programming behavior.

##### Example, bad
```c++
    void f(char* name)
    {
        FILE* input = fopen(name, "r");
        // ...
        if (something) return;   // bad: if something == true, a file handle is leaked
        // ...
        fclose(input);
    }
```
Prefer [RAII](#Rr-raii):
```c++
    void f(char* name)
    {
        ifstream input {name};
        // ...
        if (something) return;   // OK: no leak
        // ...
    }
```
**See also**: [The resource management section](#S-resource)

##### Note

A leak is colloquially "anything that isn't cleaned up."
The more important classification is "anything that can no longer be cleaned up."
For example, allocating an object on the heap and then losing the last pointer that points to that allocation.
This rule should not be taken as requiring that allocations within long-lived objects must be returned during program shutdown.
For example, relying on system guaranteed cleanup such as file closing and memory deallocation upon process shutdown can simplify code.
However, relying on abstractions that implicitly clean up can be as simple, and often safer.

##### Note

Enforcing [the lifetime safety profile](#SS-lifetime) eliminates leaks.
When combined with resource safety provided by [RAII](#Rr-raii), it eliminates the need for "garbage collection" (by generating no garbage).
Combine this with enforcement of [the type and bounds profiles](#SS-force) and you get complete type- and resource-safety, guaranteed by tools.

##### Enforcement

* Look at pointers: Classify them into non-owners (the default) and owners.
  Where feasible, replace owners with standard-library resource handles (as in the example above).
  Alternatively, mark an owner as such using `owner` from [the GSL](#S-gsl).
* Look for naked `new` and `delete`
* Look for known resource allocating functions returning raw pointers (such as `fopen`, `malloc`, and `strdup`)

### <a name="Rp-waste"></a>P.9: Don't waste time or space

##### Reason

This is C++.

##### Note

Time and space that you spend well to achieve a goal (e.g., speed of development, resource safety, or simplification of testing) is not wasted.
"Another benefit of striving for efficiency is that the process forces you to understand the problem in more depth." - Alex Stepanov

##### Example, bad
```c++
    struct X {
        char ch;
        int i;
        string s;
        char ch2;

        X& operator=(const X& a);
        X(const X&);
    };

    X waste(const char* p)
    {
        if (!p) throw Nullptr_error{};
        int n = strlen(p);
        auto buf = new char[n];
        if (!buf) throw Allocation_error{};
        for (int i = 0; i < n; ++i) buf[i] = p[i];
        // ... manipulate buffer ...
        X x;
        x.ch = 'a';
        x.s = string(n);    // give x.s space for *p
        for (gsl::index i = 0; i < x.s.size(); ++i) x.s[i] = buf[i];  // copy buf into x.s
        delete[] buf;
        return x;
    }

    void driver()
    {
        X x = waste("Typical argument");
        // ...
    }
```
Yes, this is a caricature, but we have seen every individual mistake in production code, and worse.
Note that the layout of `X` guarantees that at least 6 bytes (and most likely more) are wasted.
The spurious definition of copy operations disables move semantics so that the return operation is slow
(please note that the Return Value Optimization, RVO, is not guaranteed here).
The use of `new` and `delete` for `buf` is redundant; if we really needed a local string, we should use a local `string`.
There are several more performance bugs and gratuitous complication.

##### Example, bad
```c++
    void lower(zstring s)
    {
        for (int i = 0; i < strlen(s); ++i) s[i] = tolower(s[i]);
    }
```
Yes, this is an example from production code.
We leave it to the reader to figure out what's wasted.

##### Note

An individual example of waste is rarely significant, and where it is significant, it is typically easily eliminated by an expert.
However, waste spread liberally across a code base can easily be significant and experts are not always as available as we would like.
The aim of this rule (and the more specific rules that support it) is to eliminate most waste related to the use of C++ before it happens.
After that, we can look at waste related to algorithms and requirements, but that is beyond the scope of these guidelines.

##### Enforcement

Many more specific rules aim at the overall goals of simplicity and elimination of gratuitous waste.

### <a name="Rp-mutable"></a>P.10: Prefer immutable data to mutable data

##### Reason

It is easier to reason about constants than about variables.
Something immutable cannot change unexpectedly.
Sometimes immutability enables better optimization.
You can't have a data race on a constant.

See [Con: Constants and immutability](#S-const)

### <a name="Rp-library"></a>P.11: Encapsulate messy constructs, rather than spreading through the code

##### Reason

Messy code is more likely to hide bugs and harder to write.
A good interface is easier and safer to use.
Messy, low-level code breeds more such code.

##### Example
```c++
    int sz = 100;
    int* p = (int*) malloc(sizeof(int) * sz);
    int count = 0;
    // ...
    for (;;) {
        // ... read an int into x, exit loop if end of file is reached ...
        // ... check that x is valid ...
        if (count == sz)
            p = (int*) realloc(p, sizeof(int) * sz * 2);
        p[count++] = x;
        // ...
    }
```
This is low-level, verbose, and error-prone.
For example, we "forgot" to test for memory exhaustion.
Instead, we could use `vector`:
```c++
    vector<int> v;
    v.reserve(100);
    // ...
    for (int x; cin >> x; ) {
        // ... check that x is valid ...
        v.push_back(x);
    }
```
##### Note

The standards library and the GSL are examples of this philosophy.
For example, instead of messing with the arrays, unions, cast, tricky lifetime issues, `gsl::owner`, etc.,
that are needed to implement key abstractions, such as `vector`, `span`, `lock_guard`, and `future`, we use the libraries
designed and implemented by people with more time and expertise than we usually have.
Similarly, we can and should design and implement more specialized libraries, rather than leaving the users (often ourselves)
with the challenge of repeatedly getting low-level code well.
This is a variant of the [subset of superset principle](#R0) that underlies these guidelines.

##### Enforcement

* Look for "messy code" such as complex pointer manipulation and casting outside the implementation of abstractions.


### <a name="Rp-tools"></a>P.12: Use supporting tools as appropriate

##### Reason

There are many things that are done better "by machine".
Computers don't tire or get bored by repetitive tasks.
We typically have better things to do than repeatedly do routine tasks.

##### Example

Run a static analyzer to verify that your code follows the guidelines you want it to follow.

##### Note

See

* [Static analysis tools](???)
* [Concurrency tools](#Rconc-tools)
* [Testing tools](???)

There are many other kinds of tools, such as source code repositories, build tools, etc.,
but those are beyond the scope of these guidelines.

##### Note

Be careful not to become dependent on over-elaborate or over-specialized tool chains.
Those can make your otherwise portable code non-portable.


### <a name="Rp-lib"></a>P.13: Use support libraries as appropriate

##### Reason

Using a well-designed, well-documented, and well-supported library saves time and effort;
its quality and documentation are likely to be greater than what you could do
if the majority of your time must be spent on an implementation.
The cost (time, effort, money, etc.) of a library can be shared over many users.
A widely used library is more likely to be kept up-to-date and ported to new systems than an individual application.
Knowledge of a widely-used library can save time on other/future projects.
So, if a suitable library exists for your application domain, use it.

##### Example
```c++
    std::sort(begin(v), end(v), std::greater<>());
```
Unless you are an expert in sorting algorithms and have plenty of time,
this is more likely to be correct and to run faster than anything you write for a specific application.
You need a reason not to use the standard library (or whatever foundational libraries your application uses) rather than a reason to use it.

##### Note

By default use

* The [ISO C++ Standard Library](#S-stdlib)
* The [Guidelines Support Library](#S-gsl)

##### Note

If no well-designed, well-documented, and well-supported library exists for an important domain,
maybe you should design and implement it, and then use it.
