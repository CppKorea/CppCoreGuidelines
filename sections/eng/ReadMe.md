# <a name="main"></a>C++ Core Guidelines

September 2, 2018


Editors:

* [Bjarne Stroustrup](http://www.stroustrup.com)
* [Herb Sutter](http://herbsutter.com/)

This is a living document under continuous improvement.
Had it been an open-source (code) project, this would have been release 0.8.
Copying, use, modification, and creation of derivative works from this project is licensed under an MIT-style license.
Contributing to this project requires agreeing to a Contributor License. See the accompanying [LICENSE](LICENSE) file for details.
We make this project available to "friendly users" to use, copy, modify, and derive from, hoping for constructive input.

Comments and suggestions for improvements are most welcome.
We plan to modify and extend this document as our understanding improves and the language and the set of available libraries improve.
When commenting, please note [the introduction](#S-introduction) that outlines our aims and general approach.
The list of contributors is [here](#SS-ack).

Problems:

* The sets of rules have not been completely checked for completeness, consistency, or enforceability.
* Triple question marks (???) mark known missing information
* Update reference sections; many pre-C++11 sources are too old.
* For a more-or-less up-to-date to-do list see: [To-do: Unclassified proto-rules](#S-unclassified)

You can [read an explanation of the scope and structure of this Guide](#S-abstract) or just jump straight in:

* [In: Introduction](#S-introduction)
* [P: Philosophy](#S-philosophy)
* [I: Interfaces](#S-interfaces)
* [F: Functions](#S-functions)
* [C: Classes and class hierarchies](#S-class)
* [Enum: Enumerations](#S-enum)
* [R: Resource management](#S-resource)
* [ES: Expressions and statements](#S-expr)
* [Per: Performance](#S-performance)
* [CP: Concurrency and parallelism](#S-concurrency)
* [E: Error handling](#S-errors)
* [Con: Constants and immutability](#S-const)
* [T: Templates and generic programming](#S-templates)
* [CPL: C-style programming](#S-cpl)
* [SF: Source files](#S-source)
* [SL: The Standard Library](#S-stdlib)

Supporting sections:

* [A: Architectural ideas](#S-A)
* [NR: Non-Rules and myths](#S-not)
* [RF: References](#S-references)
* [Pro: Profiles](#S-profile)
* [GSL: Guidelines support library](#S-gsl)
* [NL: Naming and layout rules](#S-naming)
* [FAQ: Answers to frequently asked questions](#S-faq)
* [Appendix A: Libraries](#S-libraries)
* [Appendix B: Modernizing code](#S-modernizing)
* [Appendix C: Discussion](#S-discussion)
* [Appendix D: Supporting tools](#S-tools)
* [Glossary](#S-glossary)
* [To-do: Unclassified proto-rules](#S-unclassified)

You can sample rules for specific language features:

* assignment:
[regular types](#Rc-regular) --
[prefer initialization](#Rc-initialize) --
[copy](#Rc-copy-semantic) --
[move](#Rc-move-semantic) --
[other operations](#Rc-matched) --
[default](#Rc-eqdefault)
* `class`:
[data](#Rc-org) --
[invariant](#Rc-struct) --
[members](#Rc-member) --
[helpers](#Rc-helper) --
[concrete types](#SS-concrete) --
[ctors, =, and dtors](#S-ctor) --
[hierarchy](#SS-hier) --
[operators](#SS-overload)
* `concept`:
[rules](#SS-concepts) --
[in generic programming](#Rt-raise) --
[template arguments](#Rt-concepts) --
[semantics](#Rt-low)
* constructor:
[invariant](#Rc-struct) --
[establish invariant](#Rc-ctor) --
[`throw`](#Rc-throw) --
[default](#Rc-default0) --
[not needed](#Rc-default) --
[`explicit`](#Rc-explicit) --
[delegating](#Rc-delegating) --
[`virtual`](#Rc-ctor-virtual)
* derived `class`:
[when to use](#Rh-domain) --
[as interface](#Rh-abstract) --
[destructors](#Rh-dtor) --
[copy](#Rh-copy) --
[getters and setters](#Rh-get) --
[multiple inheritance](#Rh-mi-interface) --
[overloading](#Rh-using) --
[slicing](#Rc-copy-virtual) --
[`dynamic_cast`](#Rh-dynamic_cast)
* destructor:
[and constructors](#Rc-matched) --
[when needed?](#Rc-dtor) --
[may not fail](#Rc-dtor-fail)
* exception:
[errors](#S-errors) --
[`throw`](#Re-throw) --
[for errors only](#Re-errors) --
[`noexcept`](#Re-noexcept) --
[minimize `try`](#Re-catch) --
[what if no exceptions?](#Re-no-throw-codes)
* `for`:
[range-for and for](#Res-for-range) --
[for and while](#Res-for-while) --
[for-initializer](#Res-for-init) --
[empty body](#Res-empty) --
[loop variable](#Res-loop-counter) --
[loop variable type ???](#Res-???)
* function:
[naming](#Rf-package) --
[single operation](#Rf-logical) --
[no throw](#Rf-noexcept) --
[arguments](#Rf-smart) --
[argument passing](#Rf-conventional) --
[multiple return values](#Rf-out-multi) --
[pointers](#Rf-return-ptr) --
[lambdas](#Rf-capture-vs-overload)
* `inline`:
[small functions](#Rf-inline) --
[in headers](#Rs-inline)
* initialization:
[always](#Res-always) --
[prefer `{}`](#Res-list) --
[lambdas](#Res-lambda-init) --
[in-class initializers](#Rc-in-class-initializer) --
[class members](#Rc-initialize) --
[factory functions](#Rc-factory)
* lambda expression:
[when to use](#SS-lambdas)
* operator:
[conventional](#Ro-conventional) --
[avoid conversion operators](#Ro-conversion) --
[and lambdas](#Ro-lambda)
* `public`, `private`, and `protected`:
[information hiding](#Rc-private) --
[consistency](#Rh-public) --
[`protected`](#Rh-protected)
* `static_assert`:
[compile-time checking](#Rp-compile-time) --
[and concepts](#Rt-check-class)
* `struct`:
[for organizing data](#Rc-org) --
[use if no invariant](#Rc-struct) --
[no private members](#Rc-class)
* `template`:
[abstraction](#Rt-raise) --
[containers](#Rt-cont) --
[concepts](#Rt-concepts)
* `unsigned`:
[and signed](#Res-mix) --
[bit manipulation](#Res-unsigned)
* `virtual`:
[interfaces](#Ri-abstract) --
[not `virtual`](#Rc-concrete) --
[destructor](#Rc-dtor-virtual) --
[never fail](#Rc-dtor-fail)

You can look at design concepts used to express the rules:

* assertion: ???
* error: ???
* exception: exception guarantee (???)
* failure: ???
* invariant: ???
* leak: ???
* library: ???
* precondition: ???
* postcondition: ???
* resource: ???
