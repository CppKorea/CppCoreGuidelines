> # I: Interfaces

An interface is a contract between two parts of a program. Precisely stating what is expected of a supplier of a service and a user of that service is essential. Having good (easy-to-understand, encouraging efficient use, not error-prone, supporting testing, etc.) interfaces is probably the most important single aspect of code organization.

# 인터페이스들 

인터페이스는

Interface rule summary:

* [I.1: Make interfaces explicit](#Ri-explicit)
* [I.2: Avoid global variables](#Ri-global)
* [I.3: Avoid singletons](#Ri-singleton)
* [I.4: Make interfaces precisely and strongly typed](#Ri-type)
* [I.5: State preconditions (if any)](#Ri-pre)
* [I.6: Prefer `Expects()` for expressing preconditions](#Ri-expects)
* [I.7: State postconditions](#Ri-post)
* [I.8: Prefer `Ensures()` for expressing postconditions](#Ri-ensures)
* [I.9: If an interface is a template, document its parameters using concepts](#Ri-concepts)
* [I.10: Use exceptions to signal a failure to perform a required tasks](#Ri-except)
* [I.11: Never transfer ownership by a raw pointer (`T*`)](#Ri-raw)
* [I.12: Declare a pointer that must not be null as `not_null`](#Ri-nullptr)
* [I.13: Do not pass an array as a single pointer](#Ri-array)
* [I.23: Keep the number of function arguments low](#Ri-nargs)
* [I.24: Avoid adjacent unrelated parameters of the same type](#Ri-unrelated)
* [I.25: Prefer abstract classes as interfaces to class hierarchies](#Ri-abstract)
* [I.26: If you want a cross-compiler ABI, use a C-style subset](#Ri-abi)

See also

* [F: Functions](#S-functions)
* [C.concrete: Concrete types](#SS-concrete)
* [C.hier: Class hierarchies](#SS-hier)
* [C.over: Overloading and overloaded operators](#SS-overload)
* [C.con: Containers and other resource handles](#SS-containers)
* [E: Error handling](#S-errors)
* [T: Templates and generic programming](#S-templates)

<a name="Ri-expicit"></a>
### I.1: Make interfaces explicit

**Reason**: Correctness. Assumptions not stated in an interface are easily overlooked and hard to test.

**Example, bad**:
Controlling the behavior of a function through a global (namespace scope) variable (a call mode) is implicit and potentially confusing. For example,

	int rnd(double d)
	{
		return (rnd_up) ? ceil(d) : d;	// don't: "invisible" dependency
	}

It will not be obvious to a caller that the meaning of two calls of `rnd(7.2)` might give different results.

**Exception**: Sometimes we control the details of a set of operations by an environment variable, e.g., normal vs. verbose output or debug vs. optimized.
The use of a non-local control is potentially confusing, but controls only implementation details of an otherwise fixed semantics.

**Example, bad**: Reporting through non-local variables (e.g., `errno`) is easily ignored. For example:

	fprintf(connection,"logging: %d %d %d\n",x,y,s); // don't: no test of printf's return value
	
What if the connection goes down so than no logging output is produced? See Rule I.??.

**Alternative**: Throw an exception. An exception cannot be ignored.
  
**Alternative formulation**: Avoid passing information across an interface through non-local state.
Note that non-`const` member functions pass information to other member functions thorough their object's state.

**Alternative formulation**: An interface should be a function or a set of functions.
Functions can be template functions and sets of functions can be classes or class  templates.

**Enforcement**:

* (Simple) A function should not make control-flow decisions based on the values of variables declared at namespace scope.
* (Simple) A function should not write to variables declared at namespace scope.

<a name="Ri-global"></a>
### I.2 Avoid global variables

**Reason**: Non-`const` global variables hide dependencies and make the dependencies subject to unpredictable changes.

**Example**:

	struct Data {
		// ... lots of stuff ...
	} data;			// non-const data

	void compute()	// don't
	{
		// ...use data ...
	}

	void output()	// don't
	{
		// ... use data ...
	}

Who else might modify `data`?

**Note**: global constants are useful.

**Note**: The rule against global variables applies to namespace scope variables as well.

**Alternative**: If you use global (more generally namespace scope data) to avoid copying, consider passing the data as an object by const reference.
Another solution is to define the data as the state of some objects and the operations as member functions.

**Warning**: Beware of data races: if one thread can access nonlocal data (or data passed by reference) while another thread execute the callee, we can have a data race.
Every pointer or reference to mutable data is a potential data race.

**Note**: You cannot have a race condition on immutable data.

**Reference**: See the [rules for calling functions](#SS-call).

**Enforcement**: (Simple) Report all non-`const` variables declared at namespace scope.


<a name="Ri-singleton"></a>
### I.3: Avoid singletons

**Reason**: Singletons are basically complicated global objects in disguise.

**Example**:
	
	class Singleton {
		// ... lots of stuff to ensure that only one Singleton object is created, that it is initialized properly, etc.
	};

There are many variants of the singleton idea.
That's part of the problem.
	
**Note**: If you don't want a global object to change, declare it `const` or `constexpr`.

**Exception**: You can use the simplest "singleton" (so simple that it is often not considered a singleton) to get initialization on first use, if any:

	X& myX()
	{
		static X my_x {3};
		return my_x;
	}

This one of the most effective solution to problem related to initialization order.
In a multi-threaded environment the initialization of the static object does not introduce a race conition
(unless you carelessly access a shared objects from within its consructor).

If you, as many do, define a singleton as a class for which only one object is created, functions like `myX` are not singletons,
and this useful technique is not an exception to the no-singleton rule.
	
**Enforcement**: Very hard in general

* Look for classes with name that includes `singleton`
* Look for clases for which only a single object is created (by counting objects or by examining constructors)


<a name="Ri-typed"></a>
### I.4: Make interfaces precisely and strongly typed

Reason: Types are the simplest and best documentation, have well-defined meaning, and are guaranteed to be checked at compile time.
Also, precisely typed code often optimize better.

**Example; don't**: Consider

	void pass(void* data);	// void* is suspicious

Now the callee has to cast the data pointer (back) to a correct type to use it. That is error-prone and often verbose.
Avoid `void*` in interfaces.
Consider using a variant or a pointer to base instead. (Future note: Consider a pointer to concept.)

**Alternative**: Often, a template parameter can eliminate the `void*` turning it into a `T*` or something like that.

**Example; bad**: Consider

	void draw_rect(int,int,int,int);	// great opportunities for mistakes
	
	draw_rect(p.x,p.y,10,20);			// what does 10,20 mean?

An `int` can carry arbitrary forms of information, so we must guess about the meaning of the four `int`s.
Most likely, the first two are an `x`,`y` coordinate pair, but what are the last two?
Comments and parameter names can help, but we could be explicit:

	void draw_rectange(Point top_left, Point bottom_right);
	void draw_rectange(Point top_left, Size height_width);
	
	draw_rectangle(p,Point{10,20});	// two corners
	draw_rectangle(p,Size{10,20});	// one corner and a (height,width) pair

Obviously, we cannot catch all errors through the static type system
(e.g., the fact that a first argument is supposed to be a top-left point is left to convention (naming and comments)).
	

**Example**: ??? units: time duration ???

**Enforcement**:

* (Simple) Report the use of void* as a parameter or return type.
* (Hard to do well) Look for member functions with many built-in type arguments


<a name="Ri-pre"></a>
### I.5: State preconditions (if any)

**Reason**: Arguments have meaning that may constrain their proper use in the callee.

**Example**: Consider

	double sqrt(double x);

Here `x` must be positive. The type system cannot (easily and naturally) express that, so we must use other means. For example:

	double sqrt(double x); // x must be positive

Some preconditions can be expressed as assertions. For example:

	double sqrt(double x) { Expects(x>=0); /* ... */ }

Ideally, that `Expects(x>=0)` should be part of the interface of `sqrt()` but that's not easily done. For now, we place it in the definition (function body).

**Reference**: `Expects()` is described in [GSL](S-Support).

**Note**: Prefer a formal specification of requirements, such as `Expects(p!=nullptr);` If that is infeasible, use English text in comments, such as
`// the sequence [p:q) is ordered using <`

**Note**: Most member functions have as a precondition that some class invariant holds.
That invariant is established by a constructor and must be reestablished upon exit by evenry member function called from outside the class.
We don't need to mentioning it for each member function.

**Enforcement**: (Not enforceable)

**See also**: the rules for passing pointers.


<a name="Ri-expects"></a>
### I.6: Prefer `Expects()` for expressing preconditions

**Reason**: To make it clear that the condition is a precondition and to enable tool use.

**Example**:

	int area(int height, int width)
	{
		Expects(height>0 && width>0);			// good
		if (height>0 && width>0) my_error();	// obscure
		// ...
	}

**Note**: Preconditions can be stated in many ways, including comments, `if`-statements, and `assert()`. This can make them hard to distinguish from ordinary code, hard to update, hard to manipulate by tools, and may have the wrong semantics (do you always want to abort in debug mode and check nothing in productions runs?).

**Note**: Preconditions should be part of the interface rather than part of the implementation, but we don't yet have the language facilities to do that.

**Note**: `Expects()` can also be used to check a condition in the middle of an algorithm.

**Enforcement**: (Not enforceable) Finding the variety of ways preconditions can be asserted is not feasible. Warning about those that can be easily identfied (assert()) has questionable value in the absence of a language facility.


<a name="Ri-post"></a>
### I.7: State postconditions

**Reason**: To detect misunderstandings about the result and possibly catch erroneous implementations.

**Example; bad**: Consider

	int area(int height, int width) { return height*width; }	// bad

Here, we (incautiously) left out the precondition specification, so it is not explicit that height and width must be positive.
We also left out the postcondition specification, so it is not obvious that the algorithm (`height*width`) is wrong for areas larger than the largest integer.
Overflow can happen.
Consider using:

	int area(int height, int width)
	{
		auto res = height*width;
		Ensures(res>0);
		return res;
	}

**Example, bad**: Consider a famous security bug

	void f()	// problematic
	{
		char buffer[MAX];
		// ...
		memset(buffer,0,MAX);
	}

There was no postcondition stating that the buffer should be cleared and the optimizer eliminated the apparently redundant `memset()` call:

	void f()	// better
	{
		char buffer[MAX];
		// ...
		memset(buffer,0,MAX);
		Ensures(buffer[0]==0);
	}

**Note** postconditions are often informally stated in a comment that states the purpose of a function; `Ensures()` can be used to make this more systematic, visible, and checkable.

**Note**: Postconditions are especially important when they relate to something that is not directly reflected in a returned result, such as a state of a data structure used.

**Example**: Consider a function that manipulates a `Record`, using a `mutex` to avoid race conditions:

	mutex m;

	void manipulate(Record& r)	// don't
	{
		m.lock();
		// ... no m.unlock() ...
	}

Here, we "forgot" to state that the `mutex` should be released, so we don't know if the failure to ensure release of the `mutex` was a bug or a feature. Stating the postcondition would have made it clear:

	void manipulate(Record& r)	// better: hold the mutex m while and only while manipulating r
	{
		m.lock();
		// ... no m.unlock() ...
	}

The bug is now obvious.

Better still, use [RAII](#Rc-raii) to ensure that the postcondition ("the lock must be released") is enforced in code:

	void manipulate(Record& r)	// best
	{
		lock_guard _ {m};
		// ...
	}
	
**Note**: Ideally, postconditions are stated in the interface/declaration so that users can easily see them.
Only postconditions related to the users can be stated in the interface.
Postconditions related only to internal state belongs in the definition/implementation.

**Enforcement**: (Not enforceable) This is a philosophical guideline that is infeasible to check directly.


<a name="Ri-ensures"></a>
### I.8: Prefer `Ensures()` for expressing postconditions

**Reason**: To make it clear that the condition is a postcondition and to enable tool use.

**Example**:

	void f()
	{
		char buffer[MAX];
		// ...
		memset(buffer,0,MAX);
		Ensures(buffer[0]==0);
	}

**Note**: preconditions can be stated in many ways, including comments, `if`-statements, and `assert()`. This can make them hard to distinguish from ordinary code, hard to update, hard to manipulate by tools, and may have the wrong semantics.

**Alternative**: Postconditions of the form "this resource must be released" and best expressed by [RAII](#Rc-raii).

Ideally, that `Ensured` should be part of the interface that's not easily done. For now, we place it in the definition (function body).

**Enforcement**: (Not enforceable) Finding the variety of ways postconditions can be asserted is not feasible. Warning about those that can be easily identfied (assert()) has questionable value in the absence of a language facility.


<a name="Ri-concepts"></a>
### I.9: If an interface is a template, document its parameters using concepts

**Reason**: Make the interface precisely specified and compile-time checkable in the (not so distant) future.

**Example**: Use the ISO Concepts TS style of requirements specification. For example:

	template<typename Iter, typename Val>
	//	requires InputIterator<Iter> && EqualityComparable<ValueType<Iter>>,Val>
	Iter find(Iter first, Iter last, Val v)
	{
		// ...
	}

**Note**: Soon (maybe in 2016), most compilers will be able to check `requires` clauses once the `//` is removed.

**See also**: See [generic programming](???) and [???](???)

**Enforcement**: (Not enforceable yet) A language facility is under specification. When the language facility is available, warn if any non-variadic template parameter is not constrained by a concept (in its declaration or mentioned in a `requires` clause.


<a name="Ri-except"></a>
### I.10: Use exceptions to signal a failure to perform an required task

**Reason**: It should not be possible to ignore an error because that could leave the system or a computation in an undefined (or unexpected) state.
This is a major source of errors.

**Example**:

	int printf(const char* ...);	// bad: return negative number if output fails

	template <class F, class ...Args>
	explicit thread(F&& f, Args&&... args);	// good: throw system_error if unable to start the new thread

	
**Note**: What is an error?
An error means that the function cannot achieve its advertised purpose (including establishing postconditions).
Calling code that ignores the error could lead to wrong results or undefined systems state.
For example, not being able to connect to a remote server is not by itself an error:
the server can refuse a connection for all kinds of reasons, so the natural thing is to return a result that the caller always has to check.
However, if failing to make a connection is considerd an error, then a failure should throw an exception.

**Exception**: Many traditional interface functions (e.g., UNIX signal handlers) use error codes (e.g., `errno`) to report what are really status codes, rather than errors. You don't have good alternative to using such, so calling these does not violate the rule.

**Alternative**: If you can't use exceptions (e.g. because your code is full of old-style raw-pointer use or because there are hard-real-time constraints),
consider using a style that returns a pair of values:

	int val;
	int error_code;
	tie(val,error_code) do_something();
	if (error_code==0) {
		// ... handle the error or exit ...
	}
	// ... use val ...

**Note**: We don't consider "performance" a valid reason not to use exceptions.

* Often, explicit error checking and handling consume as much time and space as exception handling.
* Often, cleaner code yield better performance with exceptions (simplifying the tracing of paths through the program and their optimization).
* A good rule for performance critical code is to move checking outside the critical part of the code ([checking](#Rper-checking)).
* In the longer term, more regular code gets better optimized.

**See also**: Rule I.??? and I.??? for reporting precondition and postcondition violations.

**Enforcement**:

* (Not enforceable) This is a philosophical guideline that is infeasible to check directly.
* look for `errno`.


<a name="Ri-raw"></a>
### I.11: Never transfer ownership by a raw pointer (`T*`)

**Reason**: if there is any doubt whether the caller or the callee owns an object, leaks or premature destruction will occur.

**Example**: Consider

	X* compute(args)	// don't
	{
		X* res = new X{};
		// ...
		return res;
	}

Who deletes the returned `X`? The problem would be harder to spot if compute returned a reference.
Consider returning the result by value (use move semantics if the result is large):

	vector<double> compute(args)	// good
	{
		vector<double> res(10000);
		// ...
		return res;
	}

**Alternative**: Pass ownership using a "smart pointer", such as `unique_ptr` (for exclusive ownership) and `shared_ptr` (for shared ownership).
However that is less elegant and less efficient unless reference semantics are needed.

**Alternative**: Sometimes older code can't be modified because of ABI compatibility requirements or lack of resources.
In that case, mark owning pointers using `owner` :

	owner<X*> compute(args)		// It is now clear that ownership is transferred
	{
		owner<X*> res = new X{};
		// ...
		return res;
	}

This tells analysis tools that `res` is an owner.
That is, it's value must be `delete`d or transferred to another owner, as is done here by the `return`.

`owner` is used similarly in the implementation of resource handles.

`owner` is defined in the [Guideline Support Library](#S-GSL).

**Note**: Every object passed as a raw pointer (or iterator) is assumed to be owned by the caller, so that its lifetime is handled by the caller.

**See also**: [Argument passing](#Rf-conventional) and [value return](#Rf-T-return).

**Enforcement**:

* (Simple) Warn on `delete` of a raw pointer that is not an `owner`.
* (Simple) Warn on failure to either `reset` or explicitly `delete` an `owner` pointer on every code path.
* (Simple) Warn if the return value of `new` or a function call with return value of pointer type is assigned to a raw pointer.


<a name="Ri-nullptr"></a>
### I.12: Declare a pointer that must not be null as `not_null`

**Reason**: To help avoid dereferencing `nullptr` errors. To improve performance by avoiding redundant checks for `nullptr`.

**Example**:

	int length(const char* p);		// it is not clear whether strlen(nullptr) is valid

	length(nullptr);				// OK?

	int length(not_null<const char*> p);		// better: we can assume that p cannot be nullptr

	int length(const char* p);				// we must assume that p can be nullptr

By stating the intent in source, implementers and tools can provide better diagnostics, such as finding some classes of errors through static analysis, and perform optimizations, such as removing branches and null tests.

**Note**: The assumption that the pointer to `char` pointed to a C-style string (a zero-terminated string of characters) was still implicit, and a potential source of confusion and errors. Use `zstring` in preference to `const char*`.

	int length(not_null<zstring> p);		// we can assume that p cannot be nullptr
									// we can assume that p points to a zero-terminated array of characters

Note: `length()` is, of course, `std::strlen()` in disguise.

**Enforcement**:

* (Simple) ((Foundation)) If a function checks a pointer parameter against `nullptr` before access, on all control-flow paths, then warn it should be declared `not_null`.
* (Complex) If a function with pointer return value ensures it is not `nullptr` on all return paths, then warn the return type should be declared `not_null`.


<a name="Ri-array"></a>
### I.13: Do not pass an array as a single pointer

**Reason**: (pointer,size)-style interfaces are error-prone. Also, plain pointer (to array) must relies on some convention to allow the callee to determine the size.

**Example**: Consider

	void copy_n(const T* p, T* q, int n); // copy from [p:p+n) to [q:q+n)

What if there are fewer than `n` elements in the array pointed to by `q`? Then, we overwrite some probably unrelated memory.
What if there are fewer than `n` elements in the array pointed to by `p`? Then, we read some probably unrelated memory.
Either is undefined behavior and a potentially very nasty bug.

**Alternative**: Consider using explicit ranges,

	void copy(array_view<const T> r, array_view<T> r2); // copy r to r2

**Example, bad**: Consider

	void draw(Shape* p, int n);	// poor interface; poor code
	Circle arr[10];
	// ...
	draw(arr,10);

Passing `10` as the `n` argument may be a mistake: the most common convention is to assume [`0`:`n`) but that is nowhere stated. Worse is that the call of `draw()` compiled at all: there was an implicit conversion from array to pointer (array decay) and then another implicit conversion from `Circle` to `Shape`. There is no way that `draw()` can safely iterate through that array: it has no way of knowing the size of the elements.

**Alternative**: Use a support class that ensures that the number of elements is correct and prevents dangerous implicit conversions. For example:

	void draw2(array_view<Circle>);
	Circle arr[10];
	// ...
	draw2(array_view<Circle>(arr));	// deduce the number of elements
	draw2(arr);	// deduce the element type and array size

	void draw3(array_view<Shape>);
	draw3(arr);	// error: cannot convert Circle[10] to array_view<Shape>

This `draw2()` passes the same amount of information to `draw()`, but makes the fact that it is supposed to be a range of `Circle`s explicit. See ???.

**Exception**: Use `zstring` and `czstring` to represent a C-style, zero-terminated strings. But see ???.

**Enforcement**:

* (Simple) ((Bounds)) Warn for any expression that would rely on implicit conversion of an array type to a pointer type. Allow exception for zstring/czstring pointer types.
* (Simple) ((Bounds)) Warn for any arithmetic operation on an expression of pointer type that results in a value of pointer type. Allow exception for zstring/czstring pointer types.


<a name="Ri-nargs"></a>
### I.14: Keep the number of function arguments low

**Reason**: Having many arguments opens opportunities for confusion. Passing lots of arguments is often costly compared to alternatives.

**Example**: The standard-library `merge()` is at the limit of what we can comfortably handle

	template<class InputIterator1, class InputIterator2, class OutputIterator, class Compare>
	OutputIterator merge(InputIterator1 first1, InputIterator1 last1,
						InputIterator2 first2, InputIterator2 last2,
						OutputIterator result, Compare comp);

Here, we have three template arguments and five function arguments.
To simplify the most frequent and simplest uses, the comparison argument can be defaulted to `<`:

	template<class InputIterator1, class InputIterator2, class OutputIterator>
	OutputIterator merge(InputIterator1 first1, InputIterator1 last1,
						InputIterator2 first2, InputIterator2 last2,
						OutputIterator result);

This doesn't reduce the total complexity, but it reduces the surface complexity presented to many users.
To really reduce the number of arguments, we need to bundle the arguments into higher-level abstractions:

	template<class InputRange2, class InputRange2, class OutputIterator>
	OutputIterator merge(InputRange1 r1, InputRange2 r2, OutputIterator result);

Grouping arguments into "bundles" is a general technique to reduce the number of arguments and to increase the opportunities for checking.

**Note**: How many arguments are too many? Four arguments is a lot.
There are functions that are best expressed with four individual arguments, but not many.

**Alternative**: Group arguments into meaningful objects and pass the objects (by value or by reference).

**Alternative**: Use default arguments or overloads to allow the most common forms of calls to be done with fewer arguments.

**Enforcement**:
   - Warn when a functions declares two iterators (including pointers) of the same type instead of a range or a view.
   - (Not enforceable) This is a philosophical guideline that is infeasible to check directly.


<a name="Ri-unrelated"></a>
### I.15: Avoid adjacent unrelated parameters of the same type

**Reason**: Adjacent arguments of the same type are easily swapped by mistake.

**Example; bad**: Consider

		void copy_n(T* p, T* q, int n); // copy from [p:p+n) to [q:q+n)

This is a nasty variant of a K&R C-style interface. It is easy to reverse the "to" and "from" arguments.

Use `const` for the "from" argument:

		void copy_n(const T* p, T* q, int n); // copy from [p:p+n) to [q:q+n)

**Alternative**: Don't pass arrays as pointers, pass an object representing a rage (e.g., an `array_view`):

		void copy_n(array_view<const T> p, array_view<T> q); // copy from b to q

**Enforcement**: (Simple) Warn if two consecutive parameters share the same type.


<a name="Ri-abstract"></a>
### I.16: Prefer abstract classes as interfaces to class hierarchies

**Reason**: Abstract classes are more likely to be stable than base classes with state.

**Example; bad**: You just knew that `Shape` would turn up somewhere :-)

	class Shape {	// bad: interface class loaded with data
	public:
		Point center() { return c; }
		virtual void draw();
		virtual void rotate(int);
		// ...
	private:
		Point c;
		vector<Point> outline;
		Color col;
	};

This will force every derived class to compute a center -- even if that's non-trivial and the center is never used. Similarly, not every `Shape` has a `Color`, and many `Shape`s are best represented without an outline defined as a sequence of `Point`s. Abstract classes were invented to discourage users from writing such classes:

	class Shape {	// better: Shape is a pure interface
	public:
		virtual Point center() =0;	// pure virtual function
		virtual void draw() =0;
		virtual void rotate(int) =0;
		// ...
		// ... no data members ...
	};

**Enforcement**: (Simple) Warn if a pointer to a class `C` is assigned to a pointer to a base of `C` and the base class contains data members.


<a name="Ri-abi"></a>
### I.16: If you want a cross-compiler ABI, use a C-style subset

**Reason**: Different compilers implement different binary layouts for classes, exception handling, function names, and other implementation details.

**Exception**: You can carefully craft an interface using a few carefully selected higher-level C++ types. See ???.

**Exception**: Common ABIs are emerging on some platfoms freeing you from the more Draconian restrictions.

**Note**: if you use a single compiler, you can use full C++ in interfaces. That may require recompilation after an upgrade to a new compiler version.

**Enforcement**: (Not enforceable) It is difficult to reliably identify where an interface forms part of an ABI.
