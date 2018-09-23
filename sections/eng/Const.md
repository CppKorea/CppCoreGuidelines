
# <a name="S-const"></a>Con: Constants and immutability

You can't have a race condition on a constant.
It is easier to reason about a program when many of the objects cannot change their values.
Interfaces that promises "no change" of objects passed as arguments greatly increase readability.

Constant rule summary:

* [Con.1: By default, make objects immutable](#Rconst-immutable)
* [Con.2: By default, make member functions `const`](#Rconst-fct)
* [Con.3: By default, pass pointers and references to `const`s](#Rconst-ref)
* [Con.4: Use `const` to define objects with values that do not change after construction](#Rconst-const)
* [Con.5: Use `constexpr` for values that can be computed at compile time](#Rconst-constexpr)

### <a name="Rconst-immutable"></a>Con.1: By default, make objects immutable

##### Reason

Immutable objects are easier to reason about, so make objects non-`const` only when there is a need to change their value.
Prevents accidental or hard-to-notice change of value.

##### Example
```c++
    for (const int i : c) cout << i << '\n';    // just reading: const

    for (int i : c) cout << i << '\n';          // BAD: just reading
```
##### Exception

Function arguments are rarely mutated, but also rarely declared const.
To avoid confusion and lots of false positives, don't enforce this rule for function arguments.
```c++
    void f(const char* const p); // pedantic
    void g(const int i);        // pedantic
```
Note that function parameter is a local variable so changes to it are local.

##### Enforcement

* Flag non-`const` variables that are not modified (except for parameters to avoid many false positives)

### <a name="Rconst-fct"></a>Con.2: By default, make member functions `const`

##### Reason

A member function should be marked `const` unless it changes the object's observable state.
This gives a more precise statement of design intent, better readability, more errors caught by the compiler, and sometimes more optimization opportunities.

##### Example; bad
```c++
    class Point {
        int x, y;
    public:
        int getx() { return x; }    // BAD, should be const as it doesn't modify the object's state
        // ...
    };

    void f(const Point& pt) {
        int x = pt.getx();          // ERROR, doesn't compile because getx was not marked const
    }
```
##### Note

It is not inherently bad to pass a pointer or reference to non-`const`,
but that should be done only when the called function is supposed to modify the object.
A reader of code must assume that a function that takes a "plain" `T*` or `T&` will modify the object referred to.
If it doesn't now, it might do so later without forcing recompilation.

##### Note

There are code/libraries that are offer functions that declare a`T*` even though
those function do not modify that `T`.
This is a problem for people modernizing code.
You can

* update the library to be `const`-correct; preferred long-term solution
* "cast away `const`"; [best avoided](#Res-casts-const)
* provide a wrapper function

Example:
```c++
    void f(int* p);   // old code: f() does not modify `*p`
    void f(const int* p) { f(const_cast<int*>(p)); } // wrapper
```
Note that this wrapper solution is a patch that should be used only when the declaration of `f()` cannot be modified,
e.g. because it is in a library that you cannot modify.

##### Note

A `const` member function can modify the value of an object that is `mutable` or accessed through a pointer member.
A common use is to maintain a cache rather than repeatedly do a complicated computation.
For example, here is a `Date` that caches (mnemonizes) its string representation to simplify repeated uses:
```c++
    class Date {
    public:
        // ...
        const string& string_ref() const
        {
            if (string_val == "") compute_string_rep();
            return string_val;
        }
        // ...
    private:
        void compute_string_rep() const;    // compute string representation and place it in string_val
        mutable string string_val;
        // ...
    };
```
Another way of saying this is that `const`ness is not transitive.
It is possible for a `const` member function to change the value of `mutable` members and the value of objects accessed
through non-`const` pointers.
It is the job of the class to ensure such mutation is done only when it makes sense according to the semantics (invariants)
it offers to its users.

**See also**: [Pimpl](#Ri-pimpl)

##### Enforcement

* Flag a member function that is not marked `const`, but that does not perform a non-`const` operation on any member variable.

### <a name="Rconst-ref"></a>Con.3: By default, pass pointers and references to `const`s

##### Reason

 To avoid a called function unexpectedly changing the value.
 It's far easier to reason about programs when called functions don't modify state.

##### Example
```c++
    void f(char* p);        // does f modify *p? (assume it does)
    void g(const char* p);  // g does not modify *p
```
##### Note

It is not inherently bad to pass a pointer or reference to non-`const`,
but that should be done only when the called function is supposed to modify the object.

##### Note

[Do not cast away `const`](#Res-casts-const).

##### Enforcement

* Flag function that does not modify an object passed by  pointer or reference to non-`const`
* Flag a function that (using a cast) modifies an object passed by pointer or reference to `const`

### <a name="Rconst-const"></a>Con.4: Use `const` to define objects with values that do not change after construction

##### Reason

 Prevent surprises from unexpectedly changed object values.

##### Example
```c++
    void f()
    {
        int x = 7;
        const int y = 9;

        for (;;) {
            // ...
        }
        // ...
    }
```
As `x` is not `const`, we must assume that it is modified somewhere in the loop.

##### Enforcement

* Flag unmodified non-`const` variables.

### <a name="Rconst-constexpr"></a>Con.5: Use `constexpr` for values that can be computed at compile time

##### Reason

Better performance, better compile-time checking, guaranteed compile-time evaluation, no possibility of race conditions.

##### Example
```c++
    double x = f(2);            // possible run-time evaluation
    const double y = f(2);      // possible run-time evaluation
    constexpr double z = f(2);  // error unless f(2) can be evaluated at compile time
```
##### Note

See F.4.

##### Enforcement

* Flag `const` definitions with constant expression initializers.
