# C++ Core Guidelines

September 9, 2015

Editors:

* [Bjarne Stroustrup](http://www.stroustrup.com)
* [Herb Sutter](http://herbsutter.com/)

This document is a very early draft. It is inkorrekt, incompleat, and pÂµÃ¸oorly formatted.
Had it been an open source (code) project, this would have been release 0.6.
Copying, use, modification, and creation of derivative works from this project is licensed under an MIT-style license.
Contributing to this project requires agreeing to a Contributor License. See the accompanying LICENSE file for details.
We make this project available to "friendly users" to use, copy, modify, and derive from, hoping for constructive input.

Comments and suggestions for improvements are most welcome.
We plan to modify and extend this document as our understanding improves and the language and the set of available libraries improve.
When commenting, please note [the introduction](#S-introduction) that outlines our aims and general approach.
The list of contributors is [here](#SS-ack).

Problems:

* The sets of rules have not been thoroughly checked for completeness, consistency, or enforceability.
* Triple question marks (???) mark known missing information
* Update reference sections; many pre-C++11 sources are too old.
* For a more-or-less up-to-date to-do list see: [To-do: Unclassified proto-rules](#S-unclassified)

You can [Read an explanation of the scope and structure of this Guide](#S-abstract) or just jump straight in:

* [P: Philosophy](#S-philosophy)
* [I: Interfaces](#S-interfaces)
* [F: Functions](#S-functions)
* [C: Classes and class hierarchies](#S-class)
* [Enum: Enumerations](#S-enum)
* [ES: Expressions and statements](#S-expr)
* [E: Error handling](#S-errors)
* [R: Resource management](#S-resource)
* [T: Templates and generic programming](#S-templates)
* [CP: Concurrency](#S-concurrency)
* [STL: The Standard library](#S-stdlib)
* [SF: Source files](#S-source)
* [CPL: C-style programming](#S-cpl)
* [PRO: Profiles](#S-profile)
* [GSL: Guideline support library](#S-support)

Supporting sections:

* [NL: Naming and layout](#S-naming)
* [PER: Performance](#S-performance)
* [N: Non-Rules and myths](#S-not)
* [RF: References](#S-references)
* [Appendix A: Libraries](#S-libraries)
* [Appendix B: Modernizing code](#S-modernizing)
* [Appendix C: Discussion](#S-discussion)
* [To-do: Unclassified proto-rules](#S-unclassified)

or look at a specific language feature

* [assignment](#S-???)
* [`class`](#S-class)
* [constructor](#SS-ctor)
* [derived `class`](#SS-hier)
* [destructor](#SS-ctor)
* [exception](#S-errors)
* [`for`](#S-???)
* [`inline`](#S-class)
* [initialization](#S-???)
* [lambda expression](#SS-lambda)
* [operator](#S-???)
* [`public`, `private`, and `protected`](#S-???)
* [`static_assert`](#S-???)
* [`struct`](#S-class)
* [`template`](#S-???)
* [`unsigned`](#S-???)
* [`virtual`](#S-hier)

Definitions of terms used to express and discuss the rules, that are not language-technical, but refer to design and programming techniques

* error
* exception
* failure
* invariant
* leak
* precondition
* postcondition
* resource
* exception guarantee