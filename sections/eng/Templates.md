
# <a name="S-templates"></a>T: Templates and generic programming

Generic programming is programming using types and algorithms parameterized by types, values, and algorithms.
In C++, generic programming is supported by the `template` language mechanisms.

Arguments to generic functions are characterized by sets of requirements on the argument types and values involved.
In C++, these requirements are expressed by compile-time predicates called concepts.

Templates can also be used for meta-programming; that is, programs that compose code at compile time.

A central notion in generic programming is "concepts"; that is, requirements on template arguments presented as compile-time predicates.
"Concepts" are defined in an ISO Technical specification: [concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
A draft of a set of standard-library concepts can be found in another ISO TS: [ranges](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf)
Concepts are supported in GCC 6.1 and later.
Consequently, we comment out uses of concepts in examples; that is, we use them as formalized comments only.
If you use GCC 6.1 or later, you can uncomment them.

Template use rule summary:

* [T.1: Use templates to raise the level of abstraction of code](#Rt-raise)
* [T.2: Use templates to express algorithms that apply to many argument types](#Rt-algo)
* [T.3: Use templates to express containers and ranges](#Rt-cont)
* [T.4: Use templates to express syntax tree manipulation](#Rt-expr)
* [T.5: Combine generic and OO techniques to amplify their strengths, not their costs](#Rt-generic-oo)

Concept use rule summary:

* [T.10: Specify concepts for all template arguments](#Rt-concepts)
* [T.11: Whenever possible use standard concepts](#Rt-std-concepts)
* [T.12: Prefer concept names over `auto` for local variables](#Rt-auto)
* [T.13: Prefer the shorthand notation for simple, single-type argument concepts](#Rt-shorthand)
* ???

Concept definition rule summary:

* [T.20: Avoid "concepts" without meaningful semantics](#Rt-low)
* [T.21: Require a complete set of operations for a concept](#Rt-complete)
* [T.22: Specify axioms for concepts](#Rt-axiom)
* [T.23: Differentiate a refined concept from its more general case by adding new use patterns](#Rt-refine)
* [T.24: Use tag classes or traits to differentiate concepts that differ only in semantics](#Rt-tag)
* [T.25: Avoid complementary constraints](#Rt-not)
* [T.26: Prefer to define concepts in terms of use-patterns rather than simple syntax](#Rt-use)
* [T.30: Use concept negation (`!C<T>`) sparingly to express a minor difference](#Rt-not)
* [T.31: Use concept disjunction (`C1<T> || C2<T>`) sparingly to express alternatives](#Rt-or)
* ???

Template interface rule summary:

* [T.40: Use function objects to pass operations to algorithms](#Rt-fo)
* [T.41: Require only essential properties in a template's concepts](#Rt-essential)
* [T.42: Use template aliases to simplify notation and hide implementation details](#Rt-alias)
* [T.43: Prefer `using` over `typedef` for defining aliases](#Rt-using)
* [T.44: Use function templates to deduce class template argument types (where feasible)](#Rt-deduce)
* [T.46: Require template arguments to be at least `Regular` or `SemiRegular`](#Rt-regular)
* [T.47: Avoid highly visible unconstrained templates with common names](#Rt-visible)
* [T.48: If your compiler does not support concepts, fake them with `enable_if`](#Rt-concept-def)
* [T.49: Where possible, avoid type-erasure](#Rt-erasure)

Template definition rule summary:

* [T.60: Minimize a template's context dependencies](#Rt-depend)
* [T.61: Do not over-parameterize members (SCARY)](#Rt-scary)
* [T.62: Place non-dependent class template members in a non-templated base class](#Rt-nondependent)
* [T.64: Use specialization to provide alternative implementations of class templates](#Rt-specialization)
* [T.65: Use tag dispatch to provide alternative implementations of functions](#Rt-tag-dispatch)
* [T.67: Use specialization to provide alternative implementations for irregular types](#Rt-specialization2)
* [T.68: Use `{}` rather than `()` within templates to avoid ambiguities](#Rt-cast)
* [T.69: Inside a template, don't make an unqualified nonmember function call unless you intend it to be a customization point](#Rt-customization)

Template and hierarchy rule summary:

* [T.80: Do not naively templatize a class hierarchy](#Rt-hier)
* [T.81: Do not mix hierarchies and arrays](#Rt-array) // ??? somewhere in "hierarchies"
* [T.82: Linearize a hierarchy when virtual functions are undesirable](#Rt-linear)
* [T.83: Do not declare a member function template virtual](#Rt-virtual)
* [T.84: Use a non-template core implementation to provide an ABI-stable interface](#Rt-abi)
* [T.??: ????](#Rt-???)

Variadic template rule summary:

* [T.100: Use variadic templates when you need a function that takes a variable number of arguments of a variety of types](#Rt-variadic)
* [T.101: ??? How to pass arguments to a variadic template ???](#Rt-variadic-pass)
* [T.102: ??? How to process arguments to a variadic template ???](#Rt-variadic-process)
* [T.103: Don't use variadic templates for homogeneous argument lists](#Rt-variadic-not)
* [T.??: ????](#Rt-???)

Metaprogramming rule summary:

* [T.120: Use template metaprogramming only when you really need to](#Rt-metameta)
* [T.121: Use template metaprogramming primarily to emulate concepts](#Rt-emulate)
* [T.122: Use templates (usually template aliases) to compute types at compile time](#Rt-tmp)
* [T.123: Use `constexpr` functions to compute values at compile time](#Rt-fct)
* [T.124: Prefer to use standard-library TMP facilities](#Rt-std-tmp)
* [T.125: If you need to go beyond the standard-library TMP facilities, use an existing library](#Rt-lib)
* [T.??: ????](#Rt-???)

Other template rules summary:

* [T.140: Name all operations with potential for reuse](#Rt-name)
* [T.141: Use an unnamed lambda if you need a simple function object in one place only](#Rt-lambda)
* [T.142: Use template variables to simplify notation](#Rt-var)
* [T.143: Don't write unintentionally nongeneric code](#Rt-nongeneric)
* [T.144: Don't specialize function templates](#Rt-specialize-function)
* [T.150: Check that a class matches a concept using `static_assert`](#Rt-check-class)
* [T.??: ????](#Rt-???)

## <a name="SS-GP"></a>T.gp: Generic programming

Generic programming is programming using types and algorithms parameterized by types, values, and algorithms.

### <a name="Rt-raise"></a>T.1: Use templates to raise the level of abstraction of code

##### Reason

Generality. Reuse. Efficiency. Encourages consistent definition of user types.

##### Example, bad

Conceptually, the following requirements are wrong because what we want of `T` is more than just the very low-level concepts of "can be incremented" or "can be added":
```c++
    template<typename T>
        // requires Incrementable<T>
    T sum1(vector<T>& v, T s)
    {
        for (auto x : v) s += x;
        return s;
    }

    template<typename T>
        // requires Simple_number<T>
    T sum2(vector<T>& v, T s)
    {
        for (auto x : v) s = s + x;
        return s;
    }
```
Assuming that `Incrementable` does not support `+` and `Simple_number` does not support `+=`, we have overconstrained implementers of `sum1` and `sum2`.
And, in this case, missed an opportunity for a generalization.

##### Example
```c++
    template<typename T>
        // requires Arithmetic<T>
    T sum(vector<T>& v, T s)
    {
        for (auto x : v) s += x;
        return s;
    }
```
Assuming that `Arithmetic` requires both `+` and `+=`, we have constrained the user of `sum` to provide a complete arithmetic type.
That is not a minimal requirement, but it gives the implementer of algorithms much needed freedom and ensures that any `Arithmetic` type
can be used for a wide variety of algorithms.

For additional generality and reusability, we could also use a more general `Container` or `Range` concept instead of committing to only one container, `vector`.

##### Note

If we define a template to require exactly the operations required for a single implementation of a single algorithm
(e.g., requiring just `+=` rather than also `=` and `+`) and only those, we have overconstrained maintainers.
We aim to minimize requirements on template arguments, but the absolutely minimal requirements of an implementation is rarely a meaningful concept.

##### Note

Templates can be used to express essentially everything (they are Turing complete), but the aim of generic programming (as expressed using templates)
is to efficiently generalize operations/algorithms over a set of types with similar semantic properties.

##### Note

The `requires` in the comments are uses of `concepts`.
"Concepts" are defined in an ISO Technical specification: [concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
Concepts are supported in GCC 6.1 and later.
Consequently, we comment out uses of concepts in examples; that is, we use them as formalized comments only.
If you use GCC 6.1 or later, you can uncomment them.

##### Enforcement

* Flag algorithms with "overly simple" requirements, such as direct use of specific operators without a concept.
* Do not flag the definition of the "overly simple" concepts themselves; they may simply be building blocks for more useful concepts.

### <a name="Rt-algo"></a>T.2: Use templates to express algorithms that apply to many argument types

##### Reason

Generality. Minimizing the amount of source code. Interoperability. Reuse.

##### Example

That's the foundation of the STL. A single `find` algorithm easily works with any kind of input range:
```c++
    template<typename Iter, typename Val>
        // requires Input_iterator<Iter>
        //       && Equality_comparable<Value_type<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }
```
##### Note

Don't use a template unless you have a realistic need for more than one template argument type.
Don't overabstract.

##### Enforcement

??? tough, probably needs a human

### <a name="Rt-cont"></a>T.3: Use templates to express containers and ranges

##### Reason

Containers need an element type, and expressing that as a template argument is general, reusable, and type safe.
It also avoids brittle or inefficient workarounds. Convention: That's the way the STL does it.

##### Example
```c++
    template<typename T>
        // requires Regular<T>
    class Vector {
        // ...
        T* elem;   // points to sz Ts
        int sz;
    };

    Vector<double> v(10);
    v[7] = 9.9;
```
##### Example, bad
```c++
    class Container {
        // ...
        void* elem;   // points to size elements of some type
        int sz;
    };

    Container c(10, sizeof(double));
    ((double*) c.elem)[7] = 9.9;
```
This doesn't directly express the intent of the programmer and hides the structure of the program from the type system and optimizer.

Hiding the `void*` behind macros simply obscures the problems and introduces new opportunities for confusion.

**Exceptions**: If you need an ABI-stable interface, you might have to provide a base implementation and express the (type-safe) template in terms of that.
See [Stable base](#Rt-abi).

##### Enforcement

* Flag uses of `void*`s and casts outside low-level implementation code

### <a name="Rt-expr"></a>T.4: Use templates to express syntax tree manipulation

##### Reason

 ???

##### Example

    ???

**Exceptions**: ???

### <a name="Rt-generic-oo"></a>T.5: Combine generic and OO techniques to amplify their strengths, not their costs

##### Reason

Generic and OO techniques are complementary.

##### Example

Static helps dynamic: Use static polymorphism to implement dynamically polymorphic interfaces.
```c++
    class Command {
        // pure virtual functions
    };

    // implementations
    template</*...*/>
    class ConcreteCommand : public Command {
        // implement virtuals
    };
```
##### Example

Dynamic helps static: Offer a generic, comfortable, statically bound interface, but internally dispatch dynamically, so you offer a uniform object layout.
Examples include type erasure as with `std::shared_ptr`'s deleter (but [don't overuse type erasure](#Rt-erasure)).

##### Note

In a class template, nonvirtual functions are only instantiated if they're used -- but virtual functions are instantiated every time.
This can bloat code size, and may overconstrain a generic type by instantiating functionality that is never needed.
Avoid this, even though the standard-library facets made this mistake.

##### See also

* ref ???
* ref ???
* ref ???

##### Enforcement

See the reference to more specific rules.

## <a name="SS-concepts"></a>T.concepts: Concept rules

Concepts is a facility for specifying requirements for template arguments.
It is an [ISO technical specification](#Ref-conceptsTS), but currently supported only by GCC.
Concepts are, however, crucial in the thinking about generic programming and the basis of much work on future C++ libraries
(standard and other).

This section assumes concept support

Concept use rule summary:

* [T.10: Specify concepts for all template arguments](#Rt-concepts)
* [T.11: Whenever possible use standard concepts](#Rt-std-concepts)
* [T.12: Prefer concept names over `auto`](#Rt-auto)
* [T.13: Prefer the shorthand notation for simple, single-type argument concepts](#Rt-shorthand)
* ???

Concept definition rule summary:

* [T.20: Avoid "concepts" without meaningful semantics](#Rt-low)
* [T.21: Require a complete set of operations for a concept](#Rt-complete)
* [T.22: Specify axioms for concepts](#Rt-axiom)
* [T.23: Differentiate a refined concept from its more general case by adding new use patterns](#Rt-refine)
* [T.24: Use tag classes or traits to differentiate concepts that differ only in semantics](#Rt-tag)
* [T.25: Avoid complimentary constraints](#Rt-not)
* [T.26: Prefer to define concepts in terms of use-patterns rather than simple syntax](#Rt-use)
* ???

## <a name="SS-concept-use"></a>T.con-use: Concept use

### <a name="Rt-concepts"></a>T.10: Specify concepts for all template arguments

##### Reason

Correctness and readability.
The assumed meaning (syntax and semantics) of a template argument is fundamental to the interface of a template.
A concept dramatically improves documentation and error handling for the template.
Specifying concepts for template arguments is a powerful design tool.

##### Example
```c++
    template<typename Iter, typename Val>
    //    requires Input_iterator<Iter>
    //             && Equality_comparable<Value_type<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }
```
or equivalently and more succinctly:
```c++
    template<Input_iterator Iter, typename Val>
    //    requires Equality_comparable<Value_type<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }
```
##### Note

"Concepts" are defined in an ISO Technical specification: [concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
A draft of a set of standard-library concepts can be found in another ISO TS: [ranges](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf)
Concepts are supported in GCC 6.1 and later.
Consequently, we comment out uses of concepts in examples; that is, we use them as formalized comments only.
If you use GCC 6.1 or later, you can uncomment them:
```c++
    template<typename Iter, typename Val>
        requires Input_iterator<Iter>
               && Equality_comparable<Value_type<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }
```
##### Note

Plain `typename` (or `auto`) is the least constraining concept.
It should be used only rarely when nothing more than "it's a type" can be assumed.
This is typically only needed when (as part of template metaprogramming code) we manipulate pure expression trees, postponing type checking.

**References**: TC++PL4, Palo Alto TR, Sutton

##### Enforcement

Flag template type arguments without concepts

### <a name="Rt-std-concepts"></a>T.11: Whenever possible use standard concepts

##### Reason

 "Standard" concepts (as provided by the [GSL](#S-GSL) and the [Ranges TS](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf), and hopefully soon the ISO standard itself)
saves us the work of thinking up our own concepts, are better thought out than we can manage to do in a hurry, and improves interoperability.

##### Note

Unless you are creating a new generic library, most of the concepts you need will already be defined by the standard library.

##### Example (using TS concepts)
```c++
    template<typename T>
        // don't define this: Sortable is in the GSL
    concept Ordered_container = Sequence<T> && Random_access<Iterator<T>> && Ordered<Value_type<T>>;

    void sort(Ordered_container& s);
```
This `Ordered_container` is quite plausible, but it is very similar to the `Sortable` concept in the GSL (and the Range TS).
Is it better? Is it right? Does it accurately reflect the standard's requirements for `sort`?
It is better and simpler just to use `Sortable`:
```c++
    void sort(Sortable& s);   // better
```
##### Note

The set of "standard" concepts is evolving as we approach an ISO standard including concepts.

##### Note

Designing a useful concept is challenging.

##### Enforcement

Hard.

* Look for unconstrained arguments, templates that use "unusual"/non-standard concepts, templates that use "homebrew" concepts without axioms.
* Develop a concept-discovery tool (e.g., see [an early experiment](http://www.stroustrup.com/sle2010_webversion.pdf)).

### <a name="Rt-auto"></a>T.12: Prefer concept names over `auto` for local variables

##### Reason

 `auto` is the weakest concept. Concept names convey more meaning than just `auto`.

##### Example (using TS concepts)
```c++
    vector<string> v{ "abc", "xyz" };
    auto& x = v.front();     // bad
    String& s = v.front();   // good (String is a GSL concept)
```
##### Enforcement

* ???

### <a name="Rt-shorthand"></a>T.13: Prefer the shorthand notation for simple, single-type argument concepts

##### Reason

Readability. Direct expression of an idea.

##### Example (using TS concepts)

To say "`T` is `Sortable`":
```c++
    template<typename T>       // Correct but verbose: "The parameter is
    //    requires Sortable<T>   // of type T which is the name of a type
    void sort(T&);             // that is Sortable"

    template<Sortable T>       // Better (assuming support for concepts): "The parameter is of type T
    void sort(T&);             // which is Sortable"

    void sort(Sortable&);      // Best (assuming support for concepts): "The parameter is Sortable"
```
The shorter versions better match the way we speak. Note that many templates don't need to use the `template` keyword.

##### Note

"Concepts" are defined in an ISO Technical specification: [concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
A draft of a set of standard-library concepts can be found in another ISO TS: [ranges](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf)
Concepts are supported in GCC 6.1 and later.
Consequently, we comment out uses of concepts in examples; that is, we use them as formalized comments only.
If you use a compiler that supports concepts (e.g., GCC 6.1 or later), you can remove the `//`.

##### Enforcement

* Not feasible in the short term when people convert from the `<typename T>` and `<class T`> notation.
* Later, flag declarations that first introduces a typename and then constrains it with a simple, single-type-argument concept.

## <a name="SS-concepts-def"></a>T.concepts.def: Concept definition rules

Defining good concepts is non-trivial.
Concepts are meant to represent fundamental concepts in an application domain (hence the name "concepts").
Similarly throwing together a set of syntactic constraints to be used for a the arguments for a single class or algorithm is not what concepts were designed for
and will not give the full benefits of the mechanism.

Obviously, defining concepts will be most useful for code that can use an implementation (e.g., GCC 6.1 or later),
but defining concepts is in itself a useful design technique and help catch conceptual errors and clean up the concepts (sic!) of an implementation.

### <a name="Rt-low"></a>T.20: Avoid "concepts" without meaningful semantics

##### Reason

Concepts are meant to express semantic notions, such as "a number", "a range" of elements, and "totally ordered."
Simple constraints, such as "has a `+` operator" and "has a `>` operator" cannot be meaningfully specified in isolation
and should be used only as building blocks for meaningful concepts, rather than in user code.

##### Example, bad (using TS concepts)
```c++
    template<typename T>
    concept Addable = has_plus<T>;    // bad; insufficient

    template<Addable N> auto algo(const N& a, const N& b) // use two numbers
    {
        // ...
        return a + b;
    }

    int x = 7;
    int y = 9;
    auto z = algo(x, y);   // z = 16

    string xx = "7";
    string yy = "9";
    auto zz = algo(xx, yy);   // zz = "79"
```
Maybe the concatenation was expected. More likely, it was an accident. Defining minus equivalently would give dramatically different sets of accepted types.
This `Addable` violates the mathematical rule that addition is supposed to be commutative: `a+b == b+a`.

##### Note

The ability to specify a meaningful semantics is a defining characteristic of a true concept, as opposed to a syntactic constraint.

##### Example (using TS concepts)
```c++
    template<typename T>
    // The operators +, -, *, and / for a number are assumed to follow the usual mathematical rules
    concept Number = has_plus<T>
                     && has_minus<T>
                     && has_multiply<T>
                     && has_divide<T>;

    template<Number N> auto algo(const N& a, const N& b)
    {
        // ...
        return a + b;
    }

    int x = 7;
    int y = 9;
    auto z = algo(x, y);   // z = 16

    string xx = "7";
    string yy = "9";
    auto zz = algo(xx, yy);   // error: string is not a Number
```
##### Note

Concepts with multiple operations have far lower chance of accidentally matching a type than a single-operation concept.

##### Enforcement

* Flag single-operation `concepts` when used outside the definition of other `concepts`.
* Flag uses of `enable_if` that appears to simulate single-operation `concepts`.


### <a name="Rt-complete"></a>T.21: Require a complete set of operations for a concept

##### Reason

Ease of comprehension.
Improved interoperability.
Helps implementers and maintainers.

##### Note

This is a specific variant of the general rule that [a concept must make semantic sense](#Rt-low).

##### Example, bad (using TS concepts)

    template<typename T> concept Subtractable = requires(T a, T, b) { a-b; };

This makes no semantic sense.
You need at least `+` to make `-` meaningful and useful.

Examples of complete sets are

* `Arithmetic`: `+`, `-`, `*`, `/`, `+=`, `-=`, `*=`, `/=`
* `Comparable`: `<`, `>`, `<=`, `>=`, `==`, `!=`

##### Note

This rule applies whether we use direct language support for concepts or not.
It is a general design rule that even applies to non-templates:
```c++
    class Minimal {
        // ...
    };

    bool operator==(const Minimal&, const Minimal&);
    bool operator<(const Minimal&, const Minimal&);

    Minimal operator+(const Minimal&, const Minimal&);
    // no other operators

    void f(const Minimal& x, const Minimal& y)
    {
        if (!(x == y)) { /* ... */ }    // OK
        if (x != y) { /* ... */ }       // surprise! error

        while (!(x < y)) { /* ... */ }  // OK
        while (x >= y) { /* ... */ }    // surprise! error

        x = x + y;          // OK
        x += y;             // surprise! error
    }
```
This is minimal, but surprising and constraining for users.
It could even be less efficient.

The rule supports the view that a concept should reflect a (mathematically) coherent set of operations.

##### Example
```c++
    class Convenient {
        // ...
    };

    bool operator==(const Convenient&, const Convenient&);
    bool operator<(const Convenient&, const Convenient&);
    // ... and the other comparison operators ...

    Minimal operator+(const Convenient&, const Convenient&);
    // .. and the other arithmetic operators ...

    void f(const Convenient& x, const Convenient& y)
    {
        if (!(x == y)) { /* ... */ }    // OK
        if (x != y) { /* ... */ }       // OK

        while (!(x < y)) { /* ... */ }  // OK
        while (x >= y) { /* ... */ }    // OK

        x = x + y;     // OK
        x += y;        // OK
    }
```
It can be a nuisance to define all operators, but not hard.
Ideally, that rule should be language supported by giving you comparison operators by default.

##### Enforcement

* Flag classes that support "odd" subsets of a set of operators, e.g., `==` but not `!=` or `+` but not `-`.
  Yes, `std::string` is "odd", but it's too late to change that.


### <a name="Rt-axiom"></a>T.22: Specify axioms for concepts

##### Reason

A meaningful/useful concept has a semantic meaning.
Expressing these semantics in an informal, semi-formal, or formal way makes the concept comprehensible to readers and the effort to express it can catch conceptual errors.
Specifying semantics is a powerful design tool.

##### Example (using TS concepts)
```c++
    template<typename T>
        // The operators +, -, *, and / for a number are assumed to follow the usual mathematical rules
        // axiom(T a, T b) { a + b == b + a; a - a == 0; a * (b + c) == a * b + a * c; /*...*/ }
        concept Number = requires(T a, T b) {
            {a + b} -> T;   // the result of a + b is convertible to T
            {a - b} -> T;
            {a * b} -> T;
            {a / b} -> T;
        }
```
##### Note

This is an axiom in the mathematical sense: something that may be assumed without proof.
In general, axioms are not provable, and when they are the proof is often beyond the capability of a compiler.
An axiom may not be general, but the template writer may assume that it holds for all inputs actually used (similar to a precondition).

##### Note

In this context axioms are Boolean expressions.
See the [Palo Alto TR](#S-references) for examples.
Currently, C++ does not support axioms (even the ISO Concepts TS), so we have to make do with comments for a longish while.
Once language support is available, the `//` in front of the axiom can be removed

##### Note

The GSL concepts have well-defined semantics; see the Palo Alto TR and the Ranges TS.

##### Exception (using TS concepts)

Early versions of a new "concept" still under development will often just define simple sets of constraints without a well-specified semantics.
Finding good semantics can take effort and time.
An incomplete set of constraints can still be very useful:
```c++
    // balancer for a generic binary tree
    template<typename Node> concept bool Balancer = requires(Node* p) {
        add_fixup(p);
        touch(p);
        detach(p);
    }
```
So a `Balancer` must supply at least thee operations on a tree `Node`,
but we are not yet ready to specify detailed semantics because a new kind of balanced tree might require more operations
and the precise general semantics for all nodes is hard to pin down in the early stages of design.

A "concept" that is incomplete or without a well-specified semantics can still be useful.
For example, it allows for some checking during initial experimentation.
However, it should not be assumed to be stable.
Each new use case may require such an incomplete concepts to be improved.

##### Enforcement

* Look for the word "axiom" in concept definition comments

### <a name="Rt-refine"></a>T.23: Differentiate a refined concept from its more general case by adding new use patterns.

##### Reason

Otherwise they cannot be distinguished automatically by the compiler.

##### Example (using TS concepts)
```c++
    template<typename I>
    concept bool Input_iter = requires(I iter) { ++iter; };

    template<typename I>
    concept bool Fwd_iter = Input_iter<I> && requires(I iter) { iter++; }
```
The compiler can determine refinement based on the sets of required operations (here, suffix `++`).
This decreases the burden on implementers of these types since
they do not need any special declarations to "hook into the concept".
If two concepts have exactly the same requirements, they are logically equivalent (there is no refinement).

##### Enforcement

* Flag a concept that has exactly the same requirements as another already-seen concept (neither is more refined).
To disambiguate them, see [T.24](#Rt-tag).

### <a name="Rt-tag"></a>T.24: Use tag classes or traits to differentiate concepts that differ only in semantics.

##### Reason

Two concepts requiring the same syntax but having different semantics leads to ambiguity unless the programmer differentiates them.

##### Example (using TS concepts)
```c++
    template<typename I>    // iterator providing random access
    concept bool RA_iter = ...;

    template<typename I>    // iterator providing random access to contiguous data
    concept bool Contiguous_iter =
        RA_iter<I> && is_contiguous<I>::value;  // using is_contiguous trait
```
The programmer (in a library) must define `is_contiguous` (a trait) appropriately.

Wrapping a tag class into a concept leads to a simpler expression of this idea:
```c++
    template<typename I> concept Contiguous = is_contiguous<I>::value;

    template<typename I>
    concept bool Contiguous_iter = RA_iter<I> && Contiguous<I>;
```
The programmer (in a library) must define `is_contiguous` (a trait) appropriately.

##### Note

Traits can be trait classes or type traits.
These can be user-defined or standard-library ones.
Prefer the standard-library ones.

##### Enforcement

* The compiler flags ambiguous use of identical concepts.
* Flag the definition of identical concepts.

### <a name="Rt-not"></a>T.25: Avoid complementary constraints

##### Reason

Clarity. Maintainability.
Functions with complementary requirements expressed using negation are brittle.

##### Example (using TS concepts)

Initially, people will try to define functions with complementary requirements:
```c++
    template<typename T>
        requires !C<T>    // bad
    void f();

    template<typename T>
        requires C<T>
    void f();
```
This is better:
```c++
    template<typename T>   // general template
        void f();

    template<typename T>   // specialization by concept
        requires C<T>
    void f();
```
The compiler will choose the unconstrained template only when `C<T>` is
unsatisfied. If you do not want to (or cannot) define an unconstrained
version of `f()`, then delete it.
```c++
    template<typename T>
    void f() = delete;
```
The compiler will select the overload and emit an appropriate error.

##### Note

Complementary constraints are unfortunately common in `enable_if` code:
```c++
    template<typename T>
    enable_if<!C<T>, void>   // bad
    f();

    template<typename T>
    enable_if<C<T>, void>
    f();
```

##### Note

Complementary requirements on one requirements is sometimes (wrongly) considered manageable.
However, for two or more requirements the number of definitions needs can go up exponentially (2,4,9,16,...):
```
    C1<T> && C2<T>
    !C1<T> && C2<T>
    C1<T> && !C2<T>
    !C1<T> && !C2<T>
```
Now the opportunities for errors multiply.

##### Enforcement

* Flag pairs of functions with `C<T>` and `!C<T>` constraints

### <a name="Rt-use"></a>T.26: Prefer to define concepts in terms of use-patterns rather than simple syntax

##### Reason

The definition is more readable and corresponds directly to what a user has to write.
Conversions are taken into account. You don't have to remember the names of all the type traits.

##### Example (using TS concepts)

You might be tempted to define a concept `Equality` like this:
```c++
    template<typename T> concept Equality = has_equal<T> && has_not_equal<T>;
```
Obviously, it would be better and easier just to use the standard `EqualityComparable`,
but - just as an example - if you had to define such a concept, prefer:
```c++
    template<typename T> concept Equality = requires(T a, T b) {
        bool == { a == b }
        bool == { a != b }
        // axiom { !(a == b) == (a != b) }
        // axiom { a = b; => a == b }  // => means "implies"
    }
```
as opposed to defining two meaningless concepts `has_equal` and `has_not_equal` just as helpers in the definition of `Equality`.
By "meaningless" we mean that we cannot specify the semantics of `has_equal` in isolation.

##### Enforcement

???

## <a name="SS-temp-interface"></a>Template interfaces

Over the years, programming with templates have suffered from a weak distinction between the interface of a template
and its implementation.
Before concepts, that distinction had no direct language support.
However, the interface to a template is a critical concept - a contract between a user and an implementer - and should be carefully designed.

### <a name="Rt-fo"></a>T.40: Use function objects to pass operations to algorithms

##### Reason

Function objects can carry more information through an interface than a "plain" pointer to function.
In general, passing function objects gives better performance than passing pointers to functions.

##### Example (using TS concepts)
```c++
    bool greater(double x, double y) { return x > y; }
    sort(v, greater);                                    // pointer to function: potentially slow
    sort(v, [](double x, double y) { return x > y; });   // function object
    sort(v, std::greater<>);                             // function object

    bool greater_than_7(double x) { return x > 7; }
    auto x = find_if(v, greater_than_7);                 // pointer to function: inflexible
    auto y = find_if(v, [](double x) { return x > 7; }); // function object: carries the needed data
    auto z = find_if(v, Greater_than<double>(7));        // function object: carries the needed data
```
You can, of course, generalize those functions using `auto` or (when and where available) concepts. For example:
```c++
    auto y1 = find_if(v, [](Ordered x) { return x > 7; }); // require an ordered type
    auto z1 = find_if(v, [](auto x) { return x > 7; });    // hope that the type has a >
```
##### Note

Lambdas generate function objects.

##### Note

The performance argument depends on compiler and optimizer technology.

##### Enforcement

* Flag pointer to function template arguments.
* Flag pointers to functions passed as arguments to a template (risk of false positives).


### <a name="Rt-essential"></a>T.41: Require only essential properties in a template's concepts

##### Reason

Keep interfaces simple and stable.

##### Example (using TS concepts)

Consider, a `sort` instrumented with (oversimplified) simple debug support:
```c++
    void sort(Sortable& s)  // sort sequence s
    {
        if (debug) cerr << "enter sort( " << s <<  ")\n";
        // ...
        if (debug) cerr << "exit sort( " << s <<  ")\n";
    }
```
Should this be rewritten to:
```c++
    template<Sortable S>
        requires Streamable<S>
    void sort(S& s)  // sort sequence s
    {
        if (debug) cerr << "enter sort( " << s <<  ")\n";
        // ...
        if (debug) cerr << "exit sort( " << s <<  ")\n";
    }
```
After all, there is nothing in `Sortable` that requires `iostream` support.
On the other hand, there is nothing in the fundamental idea of sorting that says anything about debugging.

##### Note

If we require every operation used to be listed among the requirements, the interface becomes unstable:
Every time we change the debug facilities, the usage data gathering, testing support, error reporting, etc.
The definition of the template would need change and every use of the template would have to be recompiled.
This is cumbersome, and in some environments infeasible.

Conversely, if we use an operation in the implementation that is not guaranteed by concept checking,
we may get a late compile-time error.

By not using concept checking for properties of a template argument that is not considered essential,
we delay checking until instantiation time.
We consider this a worthwhile tradeoff.

Note that using non-local, non-dependent names (such as `debug` and `cerr`) also introduces context dependencies that may lead to "mysterious" errors.

##### Note

It can be hard to decide which properties of a type is essential and which are not.

##### Enforcement

???

### <a name="Rt-alias"></a>T.42: Use template aliases to simplify notation and hide implementation details

##### Reason

Improved readability.
Implementation hiding.
Note that template aliases replace many uses of traits to compute a type.
They can also be used to wrap a trait.

##### Example
```c++
    template<typename T, size_t N>
    class Matrix {
        // ...
        using Iterator = typename std::vector<T>::iterator;
        // ...
    };
```
This saves the user of `Matrix` from having to know that its elements are stored in a `vector` and also saves the user from repeatedly typing `typename std::vector<T>::`.

##### Example
```c++
    template<typename T>
    void user(T& c)
    {
        // ...
        typename container_traits<T>::value_type x; // bad, verbose
        // ...
    }

    template<typename T>
    using Value_type = typename container_traits<T>::value_type;
```

This saves the user of `Value_type` from having to know the technique used to implement `value_type`s.
```c++
    template<typename T>
    void user2(T& c)
    {
        // ...
        Value_type<T> x;
        // ...
    }
```
##### Note

A simple, common use could be expressed: "Wrap traits!"

##### Enforcement

* Flag use of `typename` as a disambiguator outside `using` declarations.
* ???

### <a name="Rt-using"></a>T.43: Prefer `using` over `typedef` for defining aliases

##### Reason

Improved readability: With `using`, the new name comes first rather than being embedded somewhere in a declaration.
Generality: `using` can be used for template aliases, whereas `typedef`s can't easily be templates.
Uniformity: `using` is syntactically similar to `auto`.

##### Example
```c++
    typedef int (*PFI)(int);   // OK, but convoluted

    using PFI2 = int (*)(int);   // OK, preferred

    template<typename T>
    typedef int (*PFT)(T);      // error

    template<typename T>
    using PFT2 = int (*)(T);   // OK
```
##### Enforcement

* Flag uses of `typedef`. This will give a lot of "hits" :-(

### <a name="Rt-deduce"></a>T.44: Use function templates to deduce class template argument types (where feasible)

##### Reason

Writing the template argument types explicitly can be tedious and unnecessarily verbose.

##### Example
```c++
    tuple<int, string, double> t1 = {1, "Hamlet", 3.14};   // explicit type
    auto t2 = make_tuple(1, "Ophelia"s, 3.14);         // better; deduced type
```
Note the use of the `s` suffix to ensure that the string is a `std::string`, rather than a C-style string.

##### Note

Since you can trivially write a `make_T` function, so could the compiler. Thus, `make_T` functions may become redundant in the future.

##### Exception

Sometimes there isn't a good way of getting the template arguments deduced and sometimes, you want to specify the arguments explicitly:
```c++
    vector<double> v = { 1, 2, 3, 7.9, 15.99 };
    list<Record*> lst;
```
##### Note

Note that C++17 will make this rule redundant by allowing the template arguments to be deduced directly from constructor arguments:
[Template parameter deduction for constructors (Rev. 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0091r1.html).
For example:
```c++
    tuple t1 = {1, "Hamlet"s, 3.14}; // deduced: tuple<int, string, double>
```
##### Enforcement

Flag uses where an explicitly specialized type exactly matches the types of the arguments used.

### <a name="Rt-regular"></a>T.46: Require template arguments to be at least `Regular` or `SemiRegular`

##### Reason

 Readability.
 Preventing surprises and errors.
 Most uses support that anyway.

##### Example
```c++
    class X {
            // ...
    public:
        explicit X(int);
        X(const X&);            // copy
        X operator=(const X&);
        X(X&&) noexcept;                 // move
        X& operator=(X&&) noexcept;
        ~X();
        // ... no more constructors ...
    };

    X x {1};    // fine
    X y = x;      // fine
    std::vector<X> v(10); // error: no default constructor
```
##### Note

Semiregular requires default constructible.

##### Enforcement

* Flag types that are not at least `SemiRegular`.

### <a name="Rt-visible"></a>T.47: Avoid highly visible unconstrained templates with common names

##### Reason

 An unconstrained template argument is a perfect match for anything so such a template can be preferred over more specific types that require minor conversions.
 This is particularly annoying/dangerous when ADL is used.
 Common names make this problem more likely.

##### Example
```c++
    namespace Bad {
        struct S { int m; };
        template<typename T1, typename T2>
        bool operator==(T1, T2) { cout << "Bad\n"; return true; }
    }

    namespace T0 {
        bool operator==(int, Bad::S) { cout << "T0\n"; return true; }  // compare to int

        void test()
        {
            Bad::S bad{ 1 };
            vector<int> v(10);
            bool b = 1 == bad;
            bool b2 = v.size() == bad;
        }
    }
```
This prints `T0` and `Bad`.

Now the `==` in `Bad` was designed to cause trouble, but would you have spotted the problem in real code?
The problem is that `v.size()` returns an `unsigned` integer so that a conversion is needed to call the local `==`;
the `==` in `Bad` requires no conversions.
Realistic types, such as the standard-library iterators can be made to exhibit similar anti-social tendencies.

##### Note

If an unconstrained template is defined in the same namespace as a type,
that unconstrained template can be found by ADL (as happened in the example).
That is, it is highly visible.

##### Note

This rule should not be necessary, but the committee cannot agree to exclude unconstrained templated from ADL.

Unfortunately this will get many false positives; the standard library violates this widely, by putting many unconstrained templates and types into the single namespace `std`.


##### Enforcement

Flag templates defined in a namespace where concrete types are also defined (maybe not feasible until we have concepts).


### <a name="Rt-concept-def"></a>T.48: If your compiler does not support concepts, fake them with `enable_if`

##### Reason

Because that's the best we can do without direct concept support.
`enable_if` can be used to conditionally define functions and to select among a set of functions.

##### Example

    enable_if<???>

##### Note

Beware of [complementary constraints](# T.25).
Faking concept overloading using `enable_if` sometimes forces us to use that error-prone design technique.

##### Enforcement

???

### <a name="Rt-erasure"></a>T.49: Where possible, avoid type-erasure

##### Reason

Type erasure incurs an extra level of indirection by hiding type information behind a separate compilation boundary.

##### Example

    ???

**Exceptions**: Type erasure is sometimes appropriate, such as for `std::function`.

##### Enforcement

???


##### Note


## <a name="SS-temp-def"></a>T.def: Template definitions

A template definition (class or function) can contain arbitrary code, so only a comprehensive review of C++ programming techniques would cover this topic.
However, this section focuses on what is specific to template implementation.
In particular, it focuses on a template definition's dependence on its context.

### <a name="Rt-depend"></a>T.60: Minimize a template's context dependencies

##### Reason

Eases understanding.
Minimizes errors from unexpected dependencies.
Eases tool creation.

##### Example
```c++
    template<typename C>
    void sort(C& c)
    {
        std::sort(begin(c), end(c)); // necessary and useful dependency
    }

    template<typename Iter>
    Iter algo(Iter first, Iter last) {
        for (; first != last; ++first) {
            auto x = sqrt(*first); // potentially surprising dependency: which sqrt()?
            helper(first, x);      // potentially surprising dependency:
                                   // helper is chosen based on first and x
            TT var = 7;            // potentially surprising dependency: which TT?
        }
    }
```
##### Note

Templates typically appear in header files so their context dependencies are more vulnerable to `#include` order dependencies than functions in `.cpp` files.

##### Note

Having a template operate only on its arguments would be one way of reducing the number of dependencies to a minimum, but that would generally be unmanageable.
For example, an algorithm usually uses other algorithms and invoke operations that does not exclusively operate on arguments.
And don't get us started on macros!

**See also**: [T.69](#Rt-customization)

##### Enforcement

??? Tricky

### <a name="Rt-scary"></a>T.61: Do not over-parameterize members (SCARY)

##### Reason

A member that does not depend on a template parameter cannot be used except for a specific template argument.
This limits use and typically increases code size.

##### Example, bad
```c++
    template<typename T, typename A = std::allocator{}>
        // requires Regular<T> && Allocator<A>
    class List {
    public:
        struct Link {   // does not depend on A
            T elem;
            T* pre;
            T* suc;
        };

        using iterator = Link*;

        iterator first() const { return head; }

        // ...
    private:
        Link* head;
    };

    List<int> lst1;
    List<int, My_allocator> lst2;
```
This looks innocent enough, but now `Link` formally depends on the allocator (even though it doesn't use the allocator). This forces redundant instantiations that can be surprisingly costly in some real-world scenarios.
Typically, the solution is to make what would have been a nested class non-local, with its own minimal set of template parameters.
```c++
    template<typename T>
    struct Link {
        T elem;
        T* pre;
        T* suc;
    };

    template<typename T, typename A = std::allocator{}>
        // requires Regular<T> && Allocator<A>
    class List2 {
    public:
        using iterator = Link<T>*;

        iterator first() const { return head; }

        // ...
    private:
        Link* head;
    };

    List<int> lst1;
    List<int, My_allocator> lst2;
```
Some people found the idea that the `Link` no longer was hidden inside the list scary, so we named the technique
[SCARY](http://www.open-std.org/jtc1/sc22/WG21/docs/papers/2009/n2911.pdf).From that academic paper: 
"The acronym SCARY describes assignments and initializations that are Seemingly erroneous (appearing Constrained by conflicting generic parameters), but Actually work with the Right implementation (unconstrained bY the conflict due to minimized dependencies."

##### Enforcement

* Flag member types that do not depend on every template argument
* Flag member functions that do not depend on every template argument

### <a name="Rt-nondependent"></a>T.62: Place non-dependent class template members in a non-templated base class

##### Reason

 Allow the base class members to be used without specifying template arguments and without template instantiation.

##### Example
```c++
    template<typename T>
    class Foo {
    public:
        enum { v1, v2 };
        // ...
    };
```
???
```c++
    struct Foo_base {
        enum { v1, v2 };
        // ...
    };

    template<typename T>
    class Foo : public Foo_base {
    public:
        // ...
    };
```
##### Note

A more general version of this rule would be
"If a template class member depends on only N template parameters out of M, place it in a base class with only N parameters."
For N == 1, we have a choice of a base class of a class in the surrounding scope as in [T.61](#Rt-scary).

??? What about constants? class statics?

##### Enforcement

* Flag ???

### <a name="Rt-specialization"></a>T.64: Use specialization to provide alternative implementations of class templates

##### Reason

A template defines a general interface.
Specialization offers a powerful mechanism for providing alternative implementations of that interface.

##### Example

    ??? string specialization (==)

    ??? representation specialization ?

##### Note

???

##### Enforcement

???

### <a name="Rt-tag-dispatch"></a>T.65: Use tag dispatch to provide alternative implementations of a function

##### Reason

* A template defines a general interface.
* Tag dispatch allows us to select implementations based on specific properties of an argument type.
* Performance.

##### Example

This is a simplified version of `std::copy` (ignoring the possibility of non-contiguous sequences)
```c++
    struct pod_tag {};
    struct non_pod_tag {};

    template<class T> struct copy_trait { using tag = non_pod_tag; };   // T is not "plain old data"

    template<> struct copy_trait<int> { using tag = pod_tag; };         // int is "plain old data"

    template<class Iter>
    Out copy_helper(Iter first, Iter last, Iter out, pod_tag)
    {
        // use memmove
    }

    template<class Iter>
    Out copy_helper(Iter first, Iter last, Iter out, non_pod_tag)
    {
        // use loop calling copy constructors
    }

    template<class Itert>
    Out copy(Iter first, Iter last, Iter out)
    {
        return copy_helper(first, last, out, typename copy_trait<Iter>::tag{})
    }

    void use(vector<int>& vi, vector<int>& vi2, vector<string>& vs, vector<string>& vs2)
    {
        copy(vi.begin(), vi.end(), vi2.begin()); // uses memmove
        copy(vs.begin(), vs.end(), vs2.begin()); // uses a loop calling copy constructors
    }
```
This is a general and powerful technique for compile-time algorithm selection.

##### Note

When `concept`s become widely available such alternatives can be distinguished directly:
```c++
    template<class Iter>
        requires Pod<Value_type<iter>>
    Out copy_helper(In, first, In last, Out out)
    {
        // use memmove
    }

    template<class Iter>
    Out copy_helper(In, first, In last, Out out)
    {
        // use loop calling copy constructors
    }
```
##### Enforcement

???


### <a name="Rt-specialization2"></a>T.67: Use specialization to provide alternative implementations for irregular types

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Rt-cast"></a>T.68: Use `{}` rather than `()` within templates to avoid ambiguities

##### Reason

 `()` is vulnerable to grammar ambiguities.

##### Example
```c++
    template<typename T, typename U>
    void f(T t, U u)
    {
        T v1(x);    // is v1 a function of a variable?
        T v2 {x};   // variable
        auto x = T(u);  // construction or cast?
    }

    f(1, "asdf"); // bad: cast from const char* to int
```
##### Enforcement

* flag `()` initializers
* flag function-style casts


### <a name="Rt-customization"></a>T.69: Inside a template, don't make an unqualified nonmember function call unless you intend it to be a customization point

##### Reason

* Provide only intended flexibility.
* Avoid vulnerability to accidental environmental changes.

##### Example

There are three major ways to let calling code customize a template.
```c++
    template<class T>
        // Call a member function
    void test1(T t)
    {
        t.f();    // require T to provide f()
    }

    template<class T>
    void test2(T t)
        // Call a nonmember function without qualification
    {
        f(t);  // require f(/*T*/) be available in caller's scope or in T's namespace
    }

    template<class T>
    void test3(T t)
        // Invoke a "trait"
    {
        test_traits<T>::f(t); // require customizing test_traits<>
                              // to get non-default functions/types
    }
```
A trait is usually a type alias to compute a type,
a `constexpr` function to compute a value,
or a traditional traits template to be specialized on the user's type.

##### Note

If you intend to call your own helper function `helper(t)` with a value `t` that depends on a template type parameter,
put it in a `::detail` namespace and qualify the call as `detail::helper(t);`.
An unqualified call becomes a customization point where any function `helper` in the namespace of `t`'s type can be invoked;
this can cause problems like [unintentionally invoking unconstrained function templates](#Rt-unconstrained-adl).


##### Enforcement

* In a template, flag an unqualified call to a nonmember function that passes a variable of dependent type when there is a nonmember function of the same name in the template's namespace.


## <a name="SS-temp-hier"></a>T.temp-hier: Template and hierarchy rules:

Templates are the backbone of C++'s support for generic programming and class hierarchies the backbone of its support
for object-oriented programming.
The two language mechanisms can be used effectively in combination, but a few design pitfalls must be avoided.

### <a name="Rt-hier"></a>T.80: Do not naively templatize a class hierarchy

##### Reason

Templating a class hierarchy that has many functions, especially many virtual functions, can lead to code bloat.

##### Example, bad
```c++
    template<typename T>
    struct Container {         // an interface
        virtual T* get(int i);
        virtual T* first();
        virtual T* next();
        virtual void sort();
    };

    template<typename T>
    class Vector : public Container<T> {
    public:
        // ...
    };

    Vector<int> vi;
    Vector<string> vs;
```
It is probably a dumb idea to define a `sort` as a member function of a container, but it is not unheard of and it makes a good example of what not to do.

Given this, the compiler cannot know if `vector<int>::sort()` is called, so it must generate code for it.
Similar for `vector<string>::sort()`.
Unless those two functions are called that's code bloat.
Imagine what this would do to a class hierarchy with dozens of member functions and dozens of derived classes with many instantiations.

##### Note

In many cases you can provide a stable interface by not parameterizing a base;
see ["stable base"](#Rt-abi) and [OO and GP](#Rt-generic-oo)

##### Enforcement

* Flag virtual functions that depend on a template argument. ??? False positives

### <a name="Rt-array"></a>T.81: Do not mix hierarchies and arrays

##### Reason

An array of derived classes can implicitly "decay" to a pointer to a base class with potential disastrous results.

##### Example

Assume that `Apple` and `Pear` are two kinds of `Fruit`s.
```c++
    void maul(Fruit* p)
    {
        *p = Pear{};     // put a Pear into *p
        p[1] = Pear{};   // put a Pear into p[1]
    }

    Apple aa [] = { an_apple, another_apple };   // aa contains Apples (obviously!)

    maul(aa);
    Apple& a0 = &aa[0];   // a Pear?
    Apple& a1 = &aa[1];   // a Pear?
```
Probably, `aa[0]` will be a `Pear` (without the use of a cast!).
If `sizeof(Apple) != sizeof(Pear)` the access to `aa[1]` will not be aligned to the proper start of an object in the array.
We have a type violation and possibly (probably) a memory corruption.
Never write such code.

Note that `maul()` violates the a [`T*` points to an individual object rule](#Rf-ptr).

**Alternative**: Use a proper (templatized) container:
```c++
    void maul2(Fruit* p)
    {
        *p = Pear{};   // put a Pear into *p
    }

    vector<Apple> va = { an_apple, another_apple };   // va contains Apples (obviously!)

    maul2(va);       // error: cannot convert a vector<Apple> to a Fruit*
    maul2(&va[0]);   // you asked for it

    Apple& a0 = &va[0];   // a Pear?
```
Note that the assignment in `maul2()` violated the [no-slicing rule](#Res-slice).

##### Enforcement

* Detect this horror!

### <a name="Rt-linear"></a>T.82: Linearize a hierarchy when virtual functions are undesirable

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Rt-virtual"></a>T.83: Do not declare a member function template virtual

##### Reason

C++ does not support that.
If it did, vtbls could not be generated until link time.
And in general, implementations must deal with dynamic linking.

##### Example, don't
```c++
    class Shape {
        // ...
        template<class T>
        virtual bool intersect(T* p);   // error: template cannot be virtual
    };
```
##### Note

We need a rule because people keep asking about this

##### Alternative

Double dispatch, visitors, calculate which function to call

##### Enforcement

The compiler handles that.

### <a name="Rt-abi"></a>T.84: Use a non-template core implementation to provide an ABI-stable interface

##### Reason

Improve stability of code.
Avoid code bloat.

##### Example

It could be a base class:
```c++
    struct Link_base {   // stable
        Link_base* suc;
        Link_base* pre;
    };

    template<typename T>   // templated wrapper to add type safety
    struct Link : Link_base {
        T val;
    };

    struct List_base {
        Link_base* first;   // first element (if any)
        int sz;             // number of elements
        void add_front(Link_base* p);
        // ...
    };

    template<typename T>
    class List : List_base {
    public:
        void put_front(const T& e) { add_front(new Link<T>{e}); }   // implicit cast to Link_base
        T& front() { static_cast<Link<T>*>(first).val; }   // explicit cast back to Link<T>
        // ...
    };

    List<int> li;
    List<string> ls;
```
Now there is only one copy of the operations linking and unlinking elements of a `List`.
The `Link` and `List` classes do nothing but type manipulation.

Instead of using a separate "base" type, another common technique is to specialize for `void` or `void*` and have the general template for `T` be just the safely-encapsulated casts to and from the core `void` implementation.

**Alternative**: Use a [Pimpl](#Ri-pimpl) implementation.

##### Enforcement

???

## <a name="SS-variadic"></a>T.var: Variadic template rules

???

### <a name="Rt-variadic"></a>T.100: Use variadic templates when you need a function that takes a variable number of arguments of a variety of types

##### Reason

Variadic templates is the most general mechanism for that, and is both efficient and type-safe. Don't use C varargs.

##### Example

    ??? printf

##### Enforcement

* Flag uses of `va_arg` in user code.

### <a name="Rt-variadic-pass"></a>T.101: ??? How to pass arguments to a variadic template ???

##### Reason

 ???

##### Example

    ??? beware of move-only and reference arguments

##### Enforcement

???

### <a name="Rt-variadic-process"></a>T.102: How to process arguments to a variadic template

##### Reason

 ???

##### Example

    ??? forwarding, type checking, references

##### Enforcement

???

### <a name="Rt-variadic-not"></a>T.103: Don't use variadic templates for homogeneous argument lists

##### Reason

There are more precise ways of specifying a homogeneous sequence, such as an `initializer_list`.

##### Example

    ???

##### Enforcement

???


## <a name="SS-meta"></a>T.meta: Template metaprogramming (TMP)

Templates provide a general mechanism for compile-time programming.

Metaprogramming is programming where at least one input or one result is a type.
Templates offer Turing-complete (modulo memory capacity) duck typing at compile time.
The syntax and techniques needed are pretty horrendous.

### <a name="Rt-metameta"></a>T.120: Use template metaprogramming only when you really need to

##### Reason

Template metaprogramming is hard to get right, slows down compilation, and is often very hard to maintain.
However, there are real-world examples where template metaprogramming provides better performance than any alternative short of expert-level assembly code.
Also, there are real-world examples where template metaprogramming expresses the fundamental ideas better than run-time code.
For example, if you really need AST manipulation at compile time (e.g., for optional matrix operation folding) there may be no other way in C++.

##### Example, bad

    ???

##### Example, bad

    enable_if

Instead, use concepts. But see [How to emulate concepts if you don't have language support](#Rt-emulate).

##### Example

    ??? good

**Alternative**: If the result is a value, rather than a type, use a [`constexpr` function](#Rt-fct).

##### Note

If you feel the need to hide your template metaprogramming in macros, you have probably gone too far.

### <a name="Rt-emulate"></a>T.121: Use template metaprogramming primarily to emulate concepts

##### Reason

Until concepts become generally available, we need to emulate them using TMP.
Use cases that require concepts (e.g. overloading based on concepts) are among the most common (and simple) uses of TMP.

##### Example
```c++
    template<typename Iter>
        /*requires*/ enable_if<random_access_iterator<Iter>, void>
    advance(Iter p, int n) { p += n; }

    template<typename Iter>
        /*requires*/ enable_if<forward_iterator<Iter>, void>
    advance(Iter p, int n) { assert(n >= 0); while (n--) ++p;}
```
##### Note

Such code is much simpler using concepts:
```c++
    void advance(RandomAccessIterator p, int n) { p += n; }

    void advance(ForwardIterator p, int n) { assert(n >= 0); while (n--) ++p;}
```
##### Enforcement

???

### <a name="Rt-tmp"></a>T.122: Use templates (usually template aliases) to compute types at compile time

##### Reason

Template metaprogramming is the only directly supported and half-way principled way of generating types at compile time.

##### Note

"Traits" techniques are mostly replaced by template aliases to compute types and `constexpr` functions to compute values.

##### Example

    ??? big object / small object optimization

##### Enforcement

???

### <a name="Rt-fct"></a>T.123: Use `constexpr` functions to compute values at compile time

##### Reason

A function is the most obvious and conventional way of expressing the computation of a value.
Often a `constexpr` function implies less compile-time overhead than alternatives.

##### Note

"Traits" techniques are mostly replaced by template aliases to compute types and `constexpr` functions to compute values.

##### Example
```c++
    template<typename T>
        // requires Number<T>
    constexpr T pow(T v, int n)   // power/exponential
    {
        T res = 1;
        while (n--) res *= v;
        return res;
    }

    constexpr auto f7 = pow(pi, 7);
```
##### Enforcement

* Flag template metaprograms yielding a value. These should be replaced with `constexpr` functions.

### <a name="Rt-std-tmp"></a>T.124: Prefer to use standard-library TMP facilities

##### Reason

Facilities defined in the standard, such as `conditional`, `enable_if`, and `tuple`, are portable and can be assumed to be known.

##### Example

    ???

##### Enforcement

???

### <a name="Rt-lib"></a>T.125: If you need to go beyond the standard-library TMP facilities, use an existing library

##### Reason

Getting advanced TMP facilities is not easy and using a library makes you part of a (hopefully supportive) community.
Write your own "advanced TMP support" only if you really have to.

##### Example

    ???

##### Enforcement

???

## <a name="SS-temp-other"></a>Other template rules

### <a name="Rt-name"></a>T.140: Name all operations with potential for reuse

##### Reason

Documentation, readability, opportunity for reuse.

##### Example
```c++
    struct Rec {
        string name;
        string addr;
        int id;         // unique identifier
    };

    bool same(const Rec& a, const Rec& b)
    {
        return a.id == b.id;
    }

    vector<Rec*> find_id(const string& name);    // find all records for "name"

    auto x = find_if(vr.begin(), vr.end(),
        [&](Rec& r) {
            if (r.name.size() != n.size()) return false; // name to compare to is in n
            for (int i = 0; i < r.name.size(); ++i)
                if (tolower(r.name[i]) != tolower(n[i])) return false;
            return true;
        }
    );
```
There is a useful function lurking here (case insensitive string comparison), as there often is when lambda arguments get large.
```c++
    bool compare_insensitive(const string& a, const string& b)
    {
        if (a.size() != b.size()) return false;
        for (int i = 0; i < a.size(); ++i) if (tolower(a[i]) != tolower(b[i])) return false;
        return true;
    }

    auto x = find_if(vr.begin(), vr.end(),
        [&](Rec& r) { compare_insensitive(r.name, n); }
    );
```
Or maybe (if you prefer to avoid the implicit name binding to n):
```c++
    auto cmp_to_n = [&n](const string& a) { return compare_insensitive(a, n); };

    auto x = find_if(vr.begin(), vr.end(),
        [](const Rec& r) { return cmp_to_n(r.name); }
    );
```
##### Note

whether functions, lambdas, or operators.

##### Exception

* Lambdas logically used only locally, such as an argument to `for_each` and similar control flow algorithms.
* Lambdas as [initializers](#???)

##### Enforcement

* (hard) flag similar lambdas
* ???

### <a name="Rt-lambda"></a>T.141: Use an unnamed lambda if you need a simple function object in one place only

##### Reason

That makes the code concise and gives better locality than alternatives.

##### Example
```c++
    auto earlyUsersEnd = std::remove_if(users.begin(), users.end(),
                                        [](const User &a) { return a.id > 100; });
```

##### Exception

Naming a lambda can be useful for clarity even if it is used only once.

##### Enforcement

* Look for identical and near identical lambdas (to be replaced with named functions or named lambdas).

### <a name="Rt-var"></a>T.142?: Use template variables to simplify notation

##### Reason

Improved readability.

##### Example

    ???

##### Enforcement

???

### <a name="Rt-nongeneric"></a>T.143: Don't write unintentionally nongeneric code

##### Reason

Generality. Reusability. Don't gratuitously commit to details; use the most general facilities available.

##### Example

Use `!=` instead of `<` to compare iterators; `!=` works for more objects because it doesn't rely on ordering.
```c++
    for (auto i = first; i < last; ++i) {   // less generic
        // ...
    }

    for (auto i = first; i != last; ++i) {   // good; more generic
        // ...
    }
```
Of course, range-`for` is better still where it does what you want.

##### Example

Use the least-derived class that has the functionality you need.
```c++
    class Base {
    public:
        Bar f();
        Bar g();
    };

    class Derived1 : public Base {
    public:
        Bar h();
    };

    class Derived2 : public Base {
    public:
        Bar j();
    };

    // bad, unless there is a specific reason for limiting to Derived1 objects only
    void my_func(Derived1& param)
    {
        use(param.f());
        use(param.g());
    }

    // good, uses only Base interface so only commit to that
    void my_func(Base& param)
    {
        use(param.f());
        use(param.g());
    }
```
##### Enforcement

* Flag comparison of iterators using `<` instead of `!=`.
* Flag `x.size() == 0` when `x.empty()` or `x.is_empty()` is available. Emptiness works for more containers than size(), because some containers don't know their size or are conceptually of unbounded size.
* Flag functions that take a pointer or reference to a more-derived type but only use functions declared in a base type.

### <a name="Rt-specialize-function"></a>T.144: Don't specialize function templates

##### Reason

You can't partially specialize a function template per language rules. You can fully specialize a function template but you almost certainly want to overload instead -- because function template specializations don't participate in overloading, they don't act as you probably wanted. Rarely, you should actually specialize by delegating to a class template that you can specialize properly.

##### Example

    ???

**Exceptions**: If you do have a valid reason to specialize a function template, just write a single function template that delegates to a class template, then specialize the class template (including the ability to write partial specializations).

##### Enforcement

* Flag all specializations of a function template. Overload instead.


### <a name="Rt-check-class"></a>T.150: Check that a class matches a concept using `static_assert`

##### Reason

If you intend for a class to match a concept, verifying that early saves users pain.

##### Example
```c++
    class X {
    public:
        X() = delete;
        X(const X&) = default;
        X(X&&) = default;
        X& operator=(const X&) = default;
        // ...
    };
```
Somewhere, possibly in an implementation file, let the compiler check the desired properties of `X`:
```c++
    static_assert(Default_constructible<X>);    // error: X has no default constructor
    static_assert(Copyable<X>);                 // error: we forgot to define X's move constructor
```

##### Enforcement

Not feasible.
