# T: Templates and generic programming

Generic programming is programming using types and algorithms parameterized by types, values, and algorithms.
In C++, generic programming is supported by the `template` language mechanisms.

Arguments to generic functions are characterized by sets of requirements on the argument types and values involved.
In C++, these requirements are expressed by compile-time predicates called concepts.

Templates can also be used for meta-programming; that is, programs that compose code at compile time.

Template use rule summary:

* [T.1: Use templates to raise the level of abstraction of code](#Rt-raise)
* [T.2: Use templates to express algorithms that apply to many argument types](#Rt-algo)
* [T.3: Use templates to express containers and ranges](#Rt-cont)
* [T.4: Use templates to express syntax tree manipulation](#Rt-expr)
* [T.5: Combine generic and OO techniques to amplify their strengths, not their costs](#Rt-generic-oo)

Concept use rule summary:

* [T.10: Specify concepts for all template arguments](#Rt-concepts)
* [T.11: Whenever possible use standard concepts](#Rt-std)
* [T.12: Prefer concept names over `auto` for local variables](#Rt-auto)
* [T.13: Prefer the shorhand notation for simple, single-type agument concepts](#Rt-shorthand)
* ???

Concept definition rule summary:

* [T.20: Avoid "concepts" without meaningful semantics](#Rt-low)
* [T.21: Define concepts to define complete sets of operations](#Rt-complete)
* [T.22: Specify axioms for concepts](#Rt-axiom)
* [T.23: Differentiate a refined concept from its more general case by adding new use patterns](#Rt-refine)
* [T.24: Use tag classes or traits to differentiate concepts that differ only in semantics](#Rt-tag)
* [T.25: Avoid negating constraints](#Rt-not)
* [T.26: Prefer to define concepts in terms of use-patterns rather than simple syntax](#Rt-use)
* ???

Template interface rule summary:

* [T.40: Use function objects to pass operations to algorithms](#Rt-fo)
* [T.41: Require complete sets of operations for a concept](#Rt-operations)
* [T.42: Use template aliases to simplify notation and hide implementation details](#Rt-alias)
* [T.43: Prefer `using` over `typedef` for defining aliases](#Rt-using)
* [T.44: Use function templates to deduce class template argument types (where feasible)](#Rt-deduce)
* [T.46: Require template arguments to be at least `Regular` or `SemiRegular`](#Rt-regular)
* [T.47: Avoid highly visible unconstrained templates with common names](#Rt-visible)
* [T.48: If your compiler does not support concepts, fake them with `enable_if`](#Rt-concept-def)
* [T.49: Where possible, avoid type-erasure](#Rt-erasure)
* [T.50: Avoid writing an unconstrained template in the same namespace as a type](#Rt-unconstrained-adl)

Template definition rule summary:

* [T.60: Minimize a template's context dependencies](#Rt-depend)
* [T.61: Do not over-parameterize members (SCARY)](#Rt-scary)
* [T.62: Place non-dependent template members in a non-templated base class](#Rt-nondependent)
* [T.64: Use specialization to provide alternative implementations of class templates](#Rt-specialization)
* [T.65: Use tag dispatch to provide alternative implementations of functions](#Rt-tag-dispatch)
* [T.66: Use selection using `enable_if` to optionally define a function](#Rt-enable_if)
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
* [T.124: Prefer to use standard-library TMP facilities](#Rt-std)
* [T.125: If you need to go beyond the standard-library TMP facilities, use an existing library](#Rt-lib)
* [T.??: ????](#Rt-???)

Other template rules summary:

* [T.140: Name all nontrivial operations](#Rt-name)
* [T.141: Use an unnamed lambda if you need a simple function object in one place only](#Rt-lambda)
* [T.142: Use template variables to simplify notation](#Rt-var)
* [T.143: Don't write unintentionally nongeneric code](#Rt-nongeneric)
* [T.144: Don't specialize function templates](#Rt-specialize-function)
* [T.??: ????](#Rt-???)


<a name="SS-GP"></a>
## T.gp: Generic programming

Generic programming is programming using types and algorithms parameterized by types, values, and algorithms.


<a name="Rt-raise"></a>
### T.1: Use templates to raise the level of abstraction of code

**Reason**: Generality. Re-use. Efficiency. Encourages consistent definition of user types.

**Example, bad**: Conceptually, the following requirements are wrong because what we want of `T` is more than just the very low-level concepts of "can be incremented" or "can be added":

	template<typename T, typename A>
		// requires Incrementable<T>
	A sum1(vector<T>& v, A s)
	{
		for (auto x : v) s+=x;
		return s;
	}
		
	template<typename T, typename A>
		// requires Simple_number<T>
	A sum2(vector<T>& v, A s)
	{
		for (auto x : v) s = s+x;
		return s;
	}

Assuming that `Incrementable` does not support `+` and `Simple_number` does not support `+=`, we have overconstrained implementers of `sum1` and `sum2`.
And, in this case, missed an opportunity for a generalization.

**Example**:

	template<typename T, typename A>
		// requires Arithmetic<T>
	A sum(vector<T>& v, A s)
	{
		for (auto x : v) s+=x;
		return s;
	}

Assuming that `Arithmetic` requires both `+` and `+=`, we have constrained the user of `sum` to provide a complete arithmetic type.
That is not a minimal requirement, but it gives the implementer of algorithms much needed freedom and ensures that any `Arithmetic` type
can be user for a wide variety of algorithms.

For additional generality and reusability, we could also use a more general `Container` or `Range` concept instead of committing to only one container, `vector`.

**Note**: If we define a template to require exactly the operations required for a single implementation of a single algorithm
(e.g., requiring just `+=` rather than also `=` and `+`) and only those,
we have overconstrained maintainers.
We aim to minimize requirements on template arguments, but the absolutely minimal requirements of an implementation is rarely a meaningful concept.

**Note**: Templates can be used to express essentially everything (they are Turing complete), but the aim of generic programming (as expressed using templates)
is to efficiently generalize operations/algorithms over a set of types with similar semantic properties.

**Enforcement**:

* Flag algorithms with "overly simple" requirements, such as direct use of specific operators without a concept.
* Do not flag the definition of the "overly simple" concepts themselves; they may simply be building blocks for more useful concepts.


<a name="Rt-algo"></a>
### T.2: Use templates to express algorithms that apply to many argument types

**Reason**: Generality. Minimizing the amount of source code. Interoperability. Re-use.

**Example**: That's the foundation of the STL. A single `find` algorithm easily works with any kind of input range:

	template<typename Iter, typename Val>
		// requires Input_iterator<Iter>
		//       && Equality_comparable<Value_type<Iter>,Val>
	Iter find(Iter b, Iter e, Val v)
	{
		// ...
	}

**Note**: Don't use a template unless you have a realistic need for more than one template argument type.
Don't overabstract.

**Enforcement**: ??? tough, probably needs a human


<a name="Rt-cont"></a>
### T.3: Use templates to express containers and ranges

**Reason**: Containers need an element type, and expressing that as a template argument is general, reusable, and type safe.
It also avoids brittle or inefficient workarounds. Convention: That's the way the STL does it.

**Example**:

	template<typename T>
		// requires Regular<T>
	class Vector {
		// ...
		T* elem;	// points to sz Ts
		int sz;
	};
	
	Vector<double> v(10);
	v[7] = 9.9;

**Example, bad**:

	class Container {
		// ...
		void* elem;	// points to size elements of some type
		int sz;
	};
	
	Container c(10,sizeof(double));
	((double*)c.elem)[] = 9.9;
	
This doesn't directly express the intent of the programmer and hides the structure of the program from the type system and optimizer.

Hiding the `void*` behind macros simply obscures the problems and introduces new opportunities for confusion.

**Exceptions**: If you need an ABI-stable interface,
you might have to provide a base implementation and express the (type-safe) template in terms of that.
See [Stable base](#Rt-abi).

**Enforcement**:

* Flag uses of `void*`s and casts outside low-level implementation code


<a name="Rt-expr"></a>
### T.4: Use templates to express syntax tree manipulation

**Reason**: ???

**Example**:

	???

**Exceptions**: ???



<a name="Rt-generic-oo"></a>
### T.5: Combine generic and OO techniques to amplify their strengths, not their costs

**Reason**: Generic and OO techniques are complementary.

**Example**: Static helps dynamic: Use static polymorphism to implement dynamically polymorphic interfaces.

	class Command {
		// pure virtual functions
	};
	
	// implementations
	template</*...*/>
	class ConcreteCommand : public Command {
		// implement virtuals
	};

**Example**: Dynamic helps static: Offer a generic, comfortable, statically bound interface, but internally dispatch dynamically, so you offer a uniform object layout. Examples include type erasure as with `std::shared_ptr`’s deleter. (But [don't overuse type erasure](#Rt-erasure).)

**Note**: In a class template, nonvirtual functions are only instantiated if they're used -- but virtual functions are instantiated every time. This can bloat code size, and may overconstrain a generic type by instantiating functionality that is never needed. Avoid this, even though the standard facets made this mistake.

**Enforcement**:
* Flag a class template that declares new (non-inherited) virtual functions.



<a name="SS-concepts"></a>
## TPG.concepts: Concept rules

Concepts is a facility for specifying requirements for template arguments.
It is an [ISO technical specification](#Ref-conceptsTS), but not yet supported by currently shipping compilers.
Concepts are, however, crucial in the thinking about generic programming and the basis of much work on future C++ libraries
(standard and other).

Concept use rule summary:

* [T.10: Specify concepts for all template arguments](#Rt-concepts)
* [T.11: Whenever possible use standard concepts](#Rt-std)
* [T.14: Prefer concept names over `auto`](#Rt-auto)
* [T.15: Prefer the shorhand notation for simple, single-type agument concepts](Rt-shorthand)
* ???

Concept definition rule summary:

* [T.20: Avoid "concepts" without meaningful semantics](#Rt-low)
* [T.21: Define concepts to define complete sets of operations](#Rt-complete)
* [T.22: Specify axioms for concepts](#Rt-axiom)
* [T.23: Differentiate a refined concept from its more general case by adding new use patterns](#Rt-refine)
* [T.24: Use tag classes or traits to differentiate concepts that differ only in semantics](#Rt-tag)
* [T.25: Avoid negating constraints](#Rt-not)
* [T.26: Prefer to define concepts in terms of use-patterns rather than simple syntax](#Rt-use)
* ???


<a name="SS-concept-use"></a>
## T.con-use: Concept use

<a name="Rt-concepts"></a>
### T.10: Specify concepts for all template arguments

**Reason**: Correctness and readability.
The assumed meaning (syntax and semantics) of a template argument is fundamental to the interface of a template.
A concept dramatically improves documentation and error handling for the template.
Specifying concepts for template arguments is a powerful design tool.

**Example**:

	template<typename Iter, typename Val>
		requires Input_iterator<Iter>
		         && Equality_comparable<Value_type<Iter>,Val>
	Iter find(Iter b, Iter e, Val v)
	{
		// ...
	}

or equivalently and more succinctly

	template<Input_iterator Iter, typename Val>
		requires Equality_comparable<Value_type<Iter>,Val>
	Iter find(Iter b, Iter e, Val v)
	{
		// ...
	}

**Note**: Until your compilers support the concepts language feature, leave the concepts in comments:

	template<typename Iter, typename Val>
		// requires Input_iterator<Iter>
		//       && Equality_comparable<Value_type<Iter>,Val>
	Iter find(Iter b, Iter e, Val v)
	{
		// ...
	}
	
**Note**: Plain `typename` (or `auto`) is the least constraining concept.
It should be used only rarely when nothing more than "it's a type" can be assumed.
This is typically only needed when (as part of template metaprogramming code) we manipulate pure expression trees, postponing type checking.

**References**: TC++PL4, Palo Alto TR, Sutton

**Enforcement**: Flag template type arguments without concepts


<a name="Rt-std"></a>
### T.11: Whenever possible use standard concepts

**Reason**: "Standard" concepts (as provideded by the GSL, the ISO concepts TS, and hopefully soon the ISO standard itself)
saves us the work of thinking up our own concepts, are better thought out than we can manage to do in a hurry, and improves interoperability.

**Note**: Unless you are creating a new generic library, most of the concepts you need will already be defined by the standard library.

**Example**:

	concept<typename T>
	Ordered_container = Sequence<T> && Random_access<Iterator<T>> && Ordered<Value_type<T>>;	// don't define this: Sortable is in the GSL

	void sort(Ordered_container& s);

This `Ordered_container` is quite plausible, but it is very similar to the `Sortable` concept in the GSL (and the Range TS).
Is it better? Is it right? Does it accurately reflect the standard's requirements for `sort`?
It is better and simpler just to use `Sortable`:

	void sort(Sortable& s);		// better

**Note**: The set of "standard" concepts is evolving as we approaches real (ISO) standardization.

**Note**: Designing a useful concept is challenging.

**Enforcement**: Hard.

* Look for unconstrained arguments, templates that use "unusual"/non-standard concepts, templates that use "homebrew" concepts without axioms.
* Develop a concept-discovery tool (e.g., see [an early experiment](http://www.stroustrup.com/sle2010_webversion.pdf).

	
<a name="Rt-auto"></a>
### T.12: Prefer concept names over `auto` for local variables

**Reason**: `auto` is the weakest concept. Concept names convey more meaning than just `auto`.

**Example**:

	vector<string> v;
	auto& x = v.front();	// bad
	String& s = v.begin();	// good

**Enforcement**:

* ???


<a name="Rt-shorthand"></a>
### T.13: Prefer the shorhand notation for simple, single-type agument concepts

**Reason**: Readability. Direct expression of an idea.

**Example**: To say "`T` is `Sortable`":

	template<typename T>		// Correct but verbose: "The parameter is
		requires Sortable<T>	// of type T which is the name of a type
	void sort(T&);				// that is Sortable"

	template<Sortable T>		// Better: "The parameter is of type T
	void sort(T&);				// which is Sortable"

	void sort(Sortable&);		// Best: "The parameter is Sortable"

The shorter versions better match the way we speak. Note that many templates don't need to use the `template` keyword.

**Enforcement**:

* Not feasible in the short term when people convert from the `<typename T>` and `<class T`> notation.
* Later, flag declarations that first introduces a typename and then constrains it with a simple, single-type-argument concept.


<a name="SS=concept-def"></a>
## T.con-def: Concept definition rules

???


<a name="Rt-low"></a>
### T.20: Avoid "concepts" without meaningful semantics

**Reason**: Concepts are meant to express semantic notions, such as "a number", "a range" of elements, and "totally ordered."
Simple constraints, such as "has a `+` operator" and "has a `>` operator" cannot be meaningfully specified in isolation
and should be used only as building blocks for meaningful concepts, rather than in user code.

**Example, bad**:

	template<typename T>
	concept Addable = has_plus<T>;    // bad; insufficient
			
	template<Addable N> auto algo(const N& a, const N& b) // use two numbers
	{
		// ...
		return a+b;
	}
	
	int x = 7;
	int y = 9;
	auto z = plus(x,y);	// z = 18
	
	string xx = "7";
	string yy = "9";
	auto zz = plus(xx,yy);	// zz = "79"

Maybe the concatenation was expected. More likely, it was an accident. Defining minus equivalently would give dramatically different sets of accepted types.
This `Addable` violates the mathematical rule that addition is supposed to be commutative: `a+b == b+a`,

**Note**: The ability to specify a meaningful semantics is a defining characteristic of a true concept, as opposed to a syntactic constraint.

**Example (using TS concepts)**:

	template<typename T>
	// The operators +, -, *, and / for a number are assumed to follow the usual mathematical rules
	concept Number = has_plus<T>
				  && has_minus<T>
				  && has_multiply<T>
				  && has_divide<T>;

	template<Number N> auto algo(const N& a, const N& b) // use two numbers
	{
		// ...
		return a+b;
	}

	int x = 7;
	int y = 9;
	auto z = plus(x,y);	// z = 18
	
	string xx = "7";
	string yy = "9";
	auto zz = plus(xx,yy);	// error: string is not a Number
	
**Note**: Concepts with multiple operations have far lower chance of accidentally matching a type than a single-operation concept.
	
**Enforcement**:

* Flag single-operation `concepts` when used outside the definition of other `concepts`.
* Flag uses of `enable_if` that appears to simulate single-operation `concepts`.


<a name="Rt-complete"></a>
### T.21: Define concepts to define complete sets of operations

**Reason**: Improves interoperability. Helps implementers and maintainers.

**Example, bad**:

	template<typename T> Subtractable = requires(T a, T,b) { a-b; }	// correct syntax?

This makes no semantic sense. You need at least `+` to make `-` meaningful and useful.

Examples of complete sets are

* `Arithmetic`: `+`, `-`, `*`, `/`, `+=`, `-=`, `*=`, `/=`
* `Comparable`: `<`, `>`, `<=`, `>=`, `==`, `!=`

**Enforcement**: ???


<a name="Rt-axiom"></a>
### T.22: Specify axioms for concepts

**Reason**: A meaningful/useful concept has a semantic meaning.
Expressing this semantics in a informal, semi-formal, or informal way makes the concept comprehensible to readers and the effort to express it can catch conceptual errors.
Specifying semantics is a powerful design tool.

**Example**:

	template<typename T>
		// The operators +, -, *, and / for a number are assumed to follow the usual mathematical rules
		// axiom(T a, T b) { a+b == b+a; a-a == 0; a*(b+c)==a*b+a*c; /*...*/ }
		concept Number = requires(T a, T b) {
			{a+b} -> T;	// the result of a+b is convertible to T
			{a-b} -> T;
			{a*b} -> T;
			{a/b} -> T;
		};
	
**Note** This is an axiom in the mathematical sense: something that may be assumed without proof.
In general, axioms are not provable, and when they are the proof is often beyond the capability of a compiler.
An axiom may not be general, but the template writer may assume that it holds for all inputs actually used (similar to a precondition).

**Note**: In this context axioms are Boolean expressions.
See the [Palo Alto TR](#S-references) for examples.
Currently, C++ does not support axioms (even the ISO Concepts TS), so we have to make do with comments for a longish while.
Once language support is available, the `//` in front of the axiom can be removed

**Note**: The GSL concepts have well defined semantics; see the Palo Alto TR and the Ranges TS.

**Exception**: Early versions of a new "concept" still under development will often just define simple sets of contraints without a well-specified semantics.
Finding good semantics can take effort and time.
An incomplete set of constraints can still be very useful:

	??? binary tree: rotate(), ...
	
A "concept" that is incomplete or without a well-specified semantics can still be useful.
However, it should not be assumed to be stable. Each new use case may require such an incomplete concepts to be improved.

**Enforcement**:

* Look for the word "axiom" in concept definition comments


<a name="Rt-refine"></a>
### T.23: Differentiate a refined concept from its more general case by adding new use patterns.

**Reason**: Otherwise they cannot be distinguished automatically by the compiler.

**Example**:

	template<typename I>
	concept bool Input_iterator = requires (I iter) { ++iter; };

	template<typename I>
	concept bool Fwd_iter = Input_iter<I> && requires (I iter) { iter++; }

The compiler can determine refinement based on the sets of required operations.
If two concepts have exactly the same requirements, they are logically equivalent (there is no refinement).

This also decreases the burden on implementers of these types since
they do not need any special declarations to "hook into the concept".

**Enforcement**:
* Flag a concept that has exactly the same requirements as another already-seen concept (neither is more refined). To disambiguate them, see [T.24](#Rt-tag).


<a name="Rt-tag"></a>
### T.24: Use tag classes or traits to differentiate concepts that differ only in semantics.

**Reason**: Two concepts requiring the same syntax but having different semantics leads to ambiguity unless the programmer differentiates them.

**Example**:

	template<typename I>		// iterator providing random access
	concept bool RA_iter = ...;

	template<typename I>		// iterator providing random access to contiguous data
	concept bool Contiguous_iter =
  		RA_iter<I> && is_contiguous<I>::value;  // ??? why not is_contiguous<I>() or is_contiguous_v<I>?

The programmer (in a library) must define `is_contiguous` (a trait) appropriately.

**Note**: Traits can be trains classes or type traits.
These can be user-defined or standard-libray ones.
Prefer the standard-libray ones.

**Enforcement**:

* The compiler flags ambiguous use of identical concepts.
* Flag the definition of identical concepts.


<a name="Rt-not"></a>
### T.25: Avoid negating constraints.

**Reason**: Clarity. Maintainability.
Functions with complementary requirements expressed using negation are brittle.

**Example**: Initially, people will try to define functions with complementary requirements:

	template<typename T>
  		requires !C<T> 		// bad
	void f();

	template<typename T>
  		requires C<T>
	void f();

This is better:

	template<typename T>	// general template
	void f();

	template<typename T>	// specialization by concept
  		requires C<T>
	void f();

The compiler will choose the unconstrained template only when `C<T>` is
unsatisfied. If you do not want to (or cannot) define an unconstrained
version of `f()`, then delete it.

	template<typename T>
	void f() = delete;

The compiler will select the overload and emit an appropriate error.

**Enforcement**:
* Flag pairs of functions with `C<T>` and `!C<T>` constraints
* Flag all constraint negation


<a name="Rt-use"></a>
### T.27: Prefer to define concepts in terms of use-patterns rather than simple syntax

**Reason**: The definition is more readable and corresponds directly to what a user has to write.
Conversions are taken into account. You don't have to remember the names of all the type traits.

**Example**:

	???

**Enforcement**: ???


<a name="SS-temp-interface"></a>
## Template interfaces

???

<a name="Rt-fo"></a>
### T.40: Use function objects to pass operations to algorithms

**Reason**: Function objects can carry more information through an interface than a "plain" pointer to function.
In general, passing function objects give better performance than passing pointers to functions.

**Example**:

	bool greater(double x, double y) { return x>y; }
	sort(v,greater);								// pointer to function: potentially slow
	sort(v,[](double x, double y) { return x>y; });	// function object
	sort(v,greater<>);								// function object
	
	bool greater_than_7(double x) { return x>7; }
	auto x = find(v,greater_than_7);				// pointer to function: inflexible
	auto y = find(v,[](double x) { return x>7; });	// function object: carries the needed data
	auto y = find(v,Greater_than<double>(7));		// function object: carries the needed data
	
	??? these lambdas are crying out for auto parameters -- any objection to making the change?
	
**Note**: Lambdas generate function objects.

**Note**: The performance argument depends on compiler and optimizer technology.

**Enforcement**:

* Flag pointer to function template arguments.
* Flag pointers to functions passed as arguments to a template (risk of false positives).


<a name="Rt-operations"></a>
### T.41: Require complete sets of operations for a concept

**Reason**: Ease of comprehension.
Improved interoperability.
Flexibility for template implementers.

**Note**: The issue here is whether to require the minimal set of operations for a template argument
(e.g., `==` but not `!=` or `+` but not `+=`).
The rule supports the view that a concept should reflect a (mathematically) coherent set of operations.

**Example**:

	???

**Enforcement**: ???


<a name="Rt-alias"></a>
### T.42: Use template aliases to simplify notation and hide implementation details

**Reason**: Improved readability. Implementation hiding. Note that template aliases replace many uses of traits to compute a type. They can also be used to wrap a trait.

**Example**:

	template<typename T, size_t N>
	class matrix {
		// ...
		using Iterator = typename std::vector<T>::iterator;
		// ...
	};

This saves the user of `Matrix` from having to know that its elements are stored in a `vector` and also saves the user from repeatedly typing `typename std::vector<T>::`.

**Example**:

	template<typename T>
	using Value_type<T> = container_traits<T>::value_type;

This saves the user of `Value_type` from having to know the technique used to implement `value_type`s.

**Enforcement**:

* Flag use of `typename` as a disambiguator outside `using` declarations.
* ???


<a name="Rt-alias"></a>
### T.43: Prefer `using` over `typedef` for defining aliases

**Reason**: Improved readability: With `using`, the new name comes first rather than being embedded somewhere in a declaration.
Generality: `using` can be used for template aliases, whereas `typedef`s can't easily be templates.
Uniformity: `using` is syntactically similar to `auto`.

**Example**:

	typedef int (*PFI)(int);	// OK, but convoluted
	
	using PFI2 = int (*)(int);	// OK, preferred
	
	template<typename T>
	typedef int (*PFT)(T);		// error
		
	template<typename T>
	using PFT2 = int (*)(T);	// OK

**Enforcement**:

* Flag uses of `typedef`. This will give a lot of "hits" :-(


<a name="Rt-deduce"></a>
### T.44: Use function templates to deduce class template argument types (where feasible)

**Reason**: Writing the template argument types explicitly can be tedious and unnecessarily verbose.

**Example**:

	tuple<int,string,double> t1 = {1,"Hamlet",3.14};	// explicit type
	auto t2 = make_tuple(1,"Ophelia"s,3.14);			// better; deduced type

Note the use of the `s` suffix to ensire that the string is a `std::string`, rather than a C-style string.

**Note**: Since you can trivially write a `make_T` function, so could the compiler. Thus, `make_T` functions may become redundant in the future.

**Exception**: Sometimes there isn't a good way of getting the template arguments deduced and sometimes, you want to specify the arguments explicitly:

	vector<double> v = { 1, 2, 3, 7.9, 15.99 };
	list<Record*> lst;

**Enforcement**: Flag uses where an explicitly specialized type exactly matches the types of the arguments used.



<a name="Rt-regular"></a>
### T.46: Require template arguments to be at least `Regular` or `SemiRegular`

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rt-visible"></a>
### T.47: Avoid highly visible unconstrained templates with common names

**Reason**: ???

**Example**:

	???

**Enforcement**: ???

<a name="Rt-concept-def"></a>
### T.48: If your compiler does not support concepts, fake them with `enable_if`

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rt-erasure"></a>
### T.49: Where possible, avoid type-erasure

**Reason**: Type erasure incurs an extra level of indirection by hiding type information behind a separate compilation boundary.

**Example**:

	???

**Exceptions**: Type erasure is sometimes appropriate, such as for `std::function`.

**Enforcement**: ???


<a name="Rt-unconstrained-adl"></a>
### T.50: Avoid writing an unconstrained template in the same namespace as a type

**Reason**: ADL will find the template even when you think it shouldn't.

**Example**:

	???

**Note**: This rule should not be necessary; the committee cannot agree on how to fix ADL, but at least making it not consider unconstrained templates would solve many of the actual problems and remove the need for this rule.

**Enforcement**: ??? unfortunately this will get many false positives; the standard library violates this widely, by putting many unconstrained templates and types into the single namespace `std`



<a name="SS=temp-def"></a>
## TCP.def: Template definitions

???


<a name="Rt-depend"></a>
### T.60: Minimize a template's context dependencies

**Reason**: Eases understanding. Minimizes errors from unexpected dependencies. Eases tool creation.

**Example**:

	???
	
**Note**: Having a template operate only on its arguments would be one way of reducing the number of dependencies to a minimum,
but that would generally be unmaneageable. For example, an algorithm usually uses other algorithms.

**Enforcement**: ??? Tricky


<a name="Rt-scary"></a>
### T.61: Do not over-parameterize members (SCARY)

**Reason**: A member that does not depend on a template parameter cannot be used except for a specific template argument.
This limits use and typically increases code size.

**Example, bad**:

	template<typename T, typename A = std::allocator{}>
		// requires Regular<T> && Allocator<A>
	class List {
	public:
		struct Link {	// does not depend on A
			T elem;
			T* pre;
			T* suc;
		};
		
		using iterator = Link*;
		
		iterator first() const { return head; }
		
		// ...
	private:
		Node* head;
	};
	
	
	List<int> lst1;
	List<int,my_allocator> lst2;
	
	???

This looks innocent enough, but ???

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
		Node* head;
	};
	
	List<int> lst1;
	List<int,my_allocator> lst2;
	
	???

**Enforcement**:

* Flag member types that do not depend on every template argument
* Flag member functions that do not depend on every template argument


<a name="Rt-nondependent"></a>
### T.62: Place non-dependent template members in a non-templated base class

**Reason**: ???

**Example**:

	template<typename T>
	class Foo {
	public:
		enum { v1, v2 };
		// ...
	};
	
???

	struct Foo_base {
		enum { v1, v2 };
		// ...
	};
	
	template<typename T>
	class Foo : public Foo_base {
	public:
		// ...
	};
	
**Note**: A more general version of this rule would be
"If a template class member depends on only N template parameters out of M, place it in a base class with only N parameters."
For N==1, we have a choice of a base class of a class in the surrounding scope as in [T.41](#Rt-scary).

??? What about constants? class statics?

**Enforcement**:

* Flag ???


<a name="Rt-specialization"></a>
### T.64: Use specialization to provide alternative implementations of class templates

**Reason**: A template defines a general interface.
Specialization offers a powerful mechanism for providing alternative implementations of that interface.

**Example**:

	??? string specialization (==)
	
	??? representation specialization ?
	
**Note**: ???

**Enforcement**: ???


<a name="Rt-tag-dispatch"></a>
### T.65: Use tag dispatch to provide alternative implementations of a function

**Reason**: A template defines a general interface. ???

**Example**:

	??? that's how we get algorithms like `std::copy` which compiles into a `memmove` call if appropriate for the arguments.

**Note**: When `concept`s become available such alternatives can be distinguished directly.

**Enforcement**: ???


<a name="Rt-specialization"></a>
### T.66: Use selection using `enable_if` to optionally define a function

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rt-customization"></a>
### T.69: Inside a template, don't make an unqualified nonmember function call unless you intend it to be a customization point

**Reason**: To provide only intended flexibility, and avoid accidental environmental changes.

If you intend to call your own helper function `helper(t)` with a value `t` that depends on a template type parameter, put it in a `::detail` namespace and qualify the call as `detail::helper(t);`. Otherwise the call becomes a customization point where any function `helper` in the namespace of `t`'s type can be invoked instead -- falling into the second option below, and resulting in problems like [unintentionally invoking unconstrained function templates of that name that happen to be in the same namespace as `t`'s type](#Rt-unconstrained-adl).

There are three major ways to let calling code customize a template.

* Call a member function. Callers can provide any type with such a named member function.

	template<class T>
	void test(T t) {
		t.f();			// require T to provide f()
	}

* Call a nonmember function without qualification. Callers can provide any type for which there is such a function available in the caller's context or in the namespace of the type.

	template<class T>
	void test(T t) {
		f(t);			// require f(/*T*/) be available in caller's cope or in T's namespace
	}
	
* Invoke a "trait" -- usually a type alias to compute a type, or a `constexpr` function to compute a value, or in rarer cases a traditioal traits template to be specialized on the user's type.

	template<class T>
	void test(T t) {
		test_traits<T>::f(t);	// require customizing test_traits<> to get non-default functions/types
		test_traits<T>::value_type x;
	}

**Enforcement**:
* In a template, flag an unqualified call to a nonmember function that passes a variable of dependent type when there is a nonmember function of the same name in the template's namespace.


<a name="SS-temp-hier"></a>
## T.temp-hier: Template and hierarchy rules:

Templates are the backbone of C++'s support for generic programming and class hierarchies the backbone of its support
for object-oriented programming.
The two language mechanisms can be use effectively in combination, but a few design pitfalls must be avoided.


<a name="Rt-hier"></a>
### T.80: Do not naively templatize a class hierarchy

**Reason**: Templatizing a class hierarchy that has many functions, especially many virtual functions, can lead to code bloat.

**Example, bad**:

	template<typename T>
	struct Container {			// an interface
		virtual T* get(int i);
		virtual T* first();
		virtial T* next();
		virtual void sort();
	};
	
	template<typename T>
	class Vector : public Container {
	public:
		// ...
	};

	Vector<int> vi;
	Vector<string> vs;

It is probably a dumb idea to define a `sort` as a member function of a container,
but it is not unheard of and it makes a good example of what not to do.
 
Given this, the compiler cannot know if `vector<int>::sort()` is called, so it must generate code for it.
Similar for `vector<string>::sort()`.
Unless those two functions are called that's code bloat.
Imagine what this would do to a class hierarchy with dozens of member functions and dozens of derived classes with many instantiations.

**Note**: In many cases you can provide a stable interface by not parameterize a base; see [???](#Rt-abi).

**Enforcement**:

* Flag virtual functions that depend on a template argument. ??? False positives


<a name="Rt-array"></a>
### T.81: Do not mix hierarchies and arrays

**Reason**: An array of derived classes can implicitly "decay" to a pointer to a base class with potential disastrous results.

**Example**: Assume that `Apple` and `Pear` are two kinds of `Fruit`s.

	void maul(Fruit* p)
	{
		*p = Pear{};	// put a Pear into *p
		p[1] = Pear{};	// put a Pear into p[2]
	}
	
	Apple aa [] = { an_apple, another_apple };	// aa contains Apples (obviously!)
	
	maul(aa);
	Apple& a0 = &aa[0];	// a Pear?
	Apple& a1 = &aa[1];	// a Pear?

Probably, `aa[0]` will be a `Pear` (without the use af a cast!).
If `sizeof(Apple)!=sizeof(Pear)` the access to `aa[1]` will not be aligned to the proper start of an object in the array.
We have a type violation and possibly (probably) a memory corruption.
Never write such code.

Note that `maul()` violates the a `T*` points to an individual object [Rule](#???).

**Alternative**: Use a proper container:

	void maul2(Fruit* p)
	{
		*p = Pear{};	// put a Pear into *p
	}
	
	vector<Apple> va = { an_apple, another_apple };	// aa contains Apples (obviously!)
	
	maul2(aa);		// error: cannot convert a vector<Apple> to a Fruit*
	maul2(&aa[0]);	// you asked for it
	
	Apple& a0 = &aa[0];	// a Pear?

Note that the assignment in `maul2()` violated the no-slicing [Rule](#???).

**Enforcement**:

* Detect this horror!


<a name="Rt-linear"></a>
### T.82: Linearize a hierarchy when virtual functions are undesirable

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rt-virtual"></a>
### T.83: Do not declare a member function template virtual

**Reason** C++ does not support that.
If it did, vtbls could not be generated until link time.
And in general, implementations must deal with dynamic linking.

**Example; don't**:

	class Shape {
		// ...
		template<class T>
		virtual bool intersect(T* p);	// error: template cannot be virtual
	};

**Alternative**: ??? double dispatch, visitor, calculate which function to call

**Enforcement**: The compiler handles that.



<a name="Rt-abi"></a>
### T.84: Use a non-template core implementation to provide an ABI-stable interface

**Reason**: Improve stability of code. Avoids code bloat.

**Example**: It could be a base class:

	struct Link_base {		// stable
		Link* suc;
		Link* pre;
	};
	
	template<typename T>	// templated wrapper to add type safety
	struct Link : Link_base {
		T val;
	};
	
	struct List_base {
		Link_base* first;	// first element (if any)
		int sz;				// number of elements
		void add_front(Link_base* p);
		// ...
	};
	
	template<typename T>
	class List : List_base {
	public:
		void put_front(const T& e) { add_front(new Link<T>{e}); }	// implicit cast to Link_base
		T& front() { static_cast<Link<T>*>(first).val; }			// explicit cast back to Link<T>
		// ...
	};
	
	List<int> li;
	List<string> ls;

Now there is only one copy of the operations linking and unlinking elements of a `List`.
The `Link` and `List` classes does nothing but type manipulation.

Instead of using a separate "base" type, another common technique is to specialize for `void` or `void*` and have the general template for `T` be just the safely-encapsulated casts to and from the core `void` implementation.
	
**Alternative**: Use a [PIMPL](#???) implementation.

**Enforcement**: ???


<a name="SS-variadic"></a>
## T.var: Variadic template rules

???


<a name="Rt-variadic"></a>
### T.100: Use variadic templates when you need a function that takes a variable number of arguments of a variety of types

**Reason**: Variadic templates is the most general mechanism for that, and is both efficient and type-safe. Don't use C varargs.

**Example**:

	??? printf

**Enforcement**:

    * Flag uses of `va_arg` in user code.


<a name="Rt-variadic-pass"></a>
### T.101: ??? How to pass arguments to a variadic template ???

**Reason**: ???

**Example**:

	??? beware of move-only and reference arguments

**Enforcement**: ???


<a name="Rt-variadic-process"></a>
### T.102: How to process arguments to a variadic template

**Reason**: ???

**Example**:

	??? forwarding, type checking, references

**Enforcement**: ???


<a name="Rt-variadic-not"></a>
### T.103: Don't use variadic templates for homogeneous argument lists

**Reason** There are more precise ways of specifying a homogeneous sequence, such as an `initializer_list`.

**Example**:

	???

**Enforcement**: ???


<a name="SS-meta"></a>
## T.meta: Template metaprogramming (TMP)

Templates provide a general mechanism for compile-time programming.

Metaprogramming is programming where at least one input or one result is a type.
Templates offer Turing-complete (modulo memory capacity) duck typing at compile time.
The syntax and techniques needed are pretty horrendous.


<a name="Rt-metameta"></a>
### T.120: Use template metaprogramming only when you really need to

**Reason**: Template metaprogramming is hard to get right, slows down compilation, and is often very hard to maintain.
However, there are real-world examples where template metaprogramming provides better performance that any alternative short of expert-level assembly code.
Also, there are real-world examples where template metaprogramming expresses the fundamental ideas better than run-time code.
For example, if you really need AST manipulation at compile time (e.g., for optional matrix operation folding) there may be no other way in C++.

**Example, bad**:

	???
	
**Example, bad**:

	enable_if

Instead, use concepts. But see [How to emulate concepts if you don't have language support]("#Rt-emulate").

**Example**:

	??? good

**Alternative**: If the result is a value, rather than a type, use a [`constexpr` function](#Rt-fct).

**Note**: If you feel the need to hide your template metaprogramming in macros, you have probably gone too far.


<a name="Rt-emulate"></a>
### T.121: Use template metaprogramming primarily to emulate concepts

**Reason**: Until concepts become generally available, we need to emulate them using TMP.
Use cases that require concepts (e.g. overloading based on concepts) are among the most common (and simple) uses of TMP.

**Example**:

	template<typename Iter>
	    /*requires*/ enable_if<random_access_iterator<Iter>,void>
	advance(Iter p, int n) { p += n; }

	template<typename Iter>
	    /*requires*/ enable_if<forward_iterator<Iter>,void>
	advance(Iter p, int n) { assert(n>=0); while (n--) ++p;}
	
**Note**: Such code is much simpler using concepts:

	void advance(RandomAccessIterator p, int n) { p += n; }

	void advance(ForwardIterator p, int n) { assert(n>=0); while (n--) ++p;}

**Enforcement**: ???


<a name="Rt-tmp"></a>
### T.122: Use templates (usually template aliases) to compute types at compile time

**Reason**: Template metaprogramming is the only directly supported and half-way principled way of generating types at compile time.

**Note**: "Traits" techniques are mostly replaced by template aliases to compute types and `constexpr` functions to compute values.

**Example**:

	??? big object / small object optimization

**Enforcement**: ???


<a name="Rt-fct"></a>
### T.123: Use `constexpr` functions to compute values at compile time

**Reason**: A function is the most obvious and conventional way of expressing the computation of a value.
Often a `constexpr` function implies less compile-time overhead than alternatives.

**Note**: "Traits" techniques are mostly replaced by template aliases to compute types and `constexpr` functions to compute values.

**Example**:

	template<typename T>
		// requires Number<T>
	constexpr T pow(T v, int n)	// power/exponential
	{
		T res = 1;
		while (n--) res *= v;
		return res;
	}
	
	constexpr auto f7 = fac(pi,7);

**Enforcement**:

    * Flag template metaprograms yielding a value. These should be replaced with `constexpr` functions.


<a name="Rt-std"></a>
### T.124: Prefer to use standard-library TMP facilities

**Reason**: Facilities defined in the standard, such as `conditional`, `enable_if`, and `tuple`, are portable and can be assumed to be known.

**Example**:

	???

**Enforcement**: ???


<a name="Rt-lib"></a>
### T.125: If you need to go beyond the standard-library TMP facilities, use an existing library

**Reason**: Getting advanced TMP facilities is not easy and using a library makes you part of a (hopefully supportive) community.
Write your own "advanced TMP support" only if you really have to.

**Example**:

	???

**Enforcement**: ???


<a name="SS-temp-other"></a>
## Other template rules


<a name="Rt-name"></a>
### T.140: Name all nontrivial operations

**Reason**: Documentation, readability, opportunity for reuse.

**Example**:

	???
	
**Example; good**:

	???

**Note**: whether functions, lambdas, or operators.

**Exceptions**:

* Lambdas logically used only locally, such as an argument to `for_each` and similar control flow algorithms.
* Lambdas as [initializers](#???)

**Enforcement**: ???


<a name="Rt-lambda"></a>
### T.141: Use an unnamed lambda if you need a simple function object in one place only

**Reason**: That makes the code concise and gives better locality than alternatives.

**Example**:

	??? for-loop equivalent

**Exception**: Naming a lambda can be useful for clarity even if it is used only once

**Enforcement**:

* Look for identical and near identical lambdas (to be replaced with named functions or named lambdas).


<a name="Rt-var"></a>
### T.142?: Use template variables to simplify notation

**Reason**: Improved readability.

**Example**:
	
	???
	
**Enforcement**: ???


<a name="Rt-nongeneric"></a>
### T.143: Don't write unintentionally nongeneric code

**Reason**: Generality. Reusability. Don't gratuitously commit to details; use the most general facilities available.

**Example**: Use `!=` instead of `<` to compare iterators; `!=` works for more objects because it doesn't rely on ordering.
	
	for(auto i = first; i < last; ++i) {	// less generic
		// ...
	}
	
	for(auto i = first; i != last; ++i) {	// good; more generic
		// ...
	}
	
Of course, range-for is better still where it does what you want.

**Example**: Use the least-derived class that has the functionality you need.

	class base {
	public:
		void f();
		void g();
	};
	
	class derived1 : public base {
	public:
		void h();
	};
	
	class derived2 : public base {
	public:
		void j();
	};
	
	void myfunc(derived& param) {	// bad, unless there is a specific reason for limiting to derived1 objects only
		use(param.f());
		use(param.g());
	}
	
	void myfunc(base& param) {		// good, uses only base interface so only commit to that
		use(param.f());
		use(param.g());
	}
	
**Enforcement**:
* Flag comparison of iterators using `<` instead of `!=`.
* Flag `x.size() == 0` when `x.empty()` or `x.is_empty()` is available. Emptiness works for more containers than size(), because some containers don't know their size or are conceptually of unbounded size.
* Flag functions that take a pointer or reference to a more-derived type but only use functions declared in a base type.


<a name="Rt-specialize-function"></a>
### T.144: Don't specialize function templates

**Reason**: You can't partially specialize a function template per language rules. You can fully specialize a function tempalte but you almost certainly want to overload instead -- because function template specializations don't participate in overloading, they don't act as you probably wanted. Rarely, you should actualy specialize by delegating to a class template that you can specialize properly.

**Example**:
	
	???

**Exceptions**: If you do have a valid reason to specialize a function template, just write a single function template that delegates to a class template, then specialize the class template (including the ability to write partial specializations).
	
**Enforcement**:
* Flag all specializations of a funciton template. Overload instead.