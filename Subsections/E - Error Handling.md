# E: Error handling

Error handling involves

* Detecting an error
* Transmitting information about an error to some handler code
* Preserve the state of a program in a valid state
* Avoid resource leaks
*
It is not possible to recover from all errors. If recovery from an error is not possible, it is important to quickly "get out" in a well-defined way. A strategy for error handling must be simple, or it becomes a source of even worse errors.

The rules are designed to help avoid several kinds of errors:

* Type violations (e.g., misuse of `union`s and casts)
* Resource leaks (including memory leaks)
* Bounds errors
* Lifetime errors (e.g., accessing an object after is has been `delete`d)
* Complexity errors (logical errors make likely by overly complex expression of ideas)
* Interface errors (e.g., an unexpected value is passed through an interface)

Error-handling rule summary:

* [E.1: Develop an error-handling strategy early in a design](#Re-design)
* [E.2: Throw an exception to signal that a function can't perform its assigned task](#Re-throw)
* [E.3: Use exceptions for error handling only](#Re-errors)
* [E.4: Design your error-handling strategy around invariants](#Re-design-invariant)
* [E.5: Let a constructor establish an invariant, and throw if it cannot](#Re-invariant)
* [E.6: Use RAII to prevent leaks](#Re-raii)
* [E.7: State your preconditions](#Re-precondition)
* [E.8: State your postconditions](#Re-postcondition)

* [E.12: Use `noexcept` when exiting a function because of a `throw` is impossible or unacceptable](#Re-noexcept)
* [E.13: Never throw while being the direct owner of an object](#Re-never-throw)
* [E.14: Use purpose-designed user-defined types as exceptions (not built-in types)](#Re-exception-types)
* [E.15: Catch exceptions from a hierarchy by reference](#Re-exception-ref)
* [E.16: Destructors, deallocation, and `swap` must never fail](#Re-never-fail)
* [E.17: Don't try to catch every exception in every function](#Re-not-always)
* [E.18: Minimize the use of explicit `try`/`catch`](#Re-catch)
* [E.19: Use a `Final_action` object to express cleanup if no suitable resource handle is available](#Re-finally)

* [E.25: ??? What to do in programs where exceptions cannot be thrown](#Re-no-throw)
* ???


<a name="Re-design"></a>
### E.1: Develop an error-handling strategy early in a design

**Reason**: a consistent and complete strategy for handling errors and resource leaks is hard to retrofit into a system.

<a name="Re-throw"></a>
### E.2: Throw an exception to signal that a function can't perform its assigned task

**Reason**: To make error handling systematic, robust, and non-repetitive.

**Example**:

	struct Foo {
		vector<Thing> v;
		File_handle f;
		string s;
	};

	void use()
	{
		Foo bar { {Thing{1}, Thing{2}, Thing{monkey}}, {"my_file","r"}, "Here we go!"};
		// ...
	}

Here, `vector` and `string`s constructors may not be able to allocate sufficient memory for their elements,
`vector`s constructor may not be able copy the `Thing`s in its initializer list, and `File_handle` may not be able to open the required file.
In each case, they throw an exception for `use()`'s caller to handle.
If `use()` could handle the failure to construct `bar` it can take control using `try`/`catch`.
In either case, `Foo`'s constructor correctly destroys constructed members before passing control to whatever tried to create a `Foo`.
Note that there is no return value that could contain an error code.

The `File_handle` constructor might defined like this

		File_handle::File_handle(const string& name, const string& mode)
			:f{fopen(name.c_str(),mode.c_str()}
		{
			if (!f)
				throw runtime_error{"File_handle: could not open "S-+ name + " as " + mode"}
		}

**Note**: It is often said that exceptions are meant to signal exceptional events and failures.
However, that's a bit circular because "what is exceptional?"
Examples:

* A precondition that cannot be met
* A constructor that cannot construct an object (failure to establish its class's [invariant](#Rc-struct))
* An out-of-range error (e.g., `v[v.size()]=7`)
* Inability to acquire a resource (e.g., the network is down)

In contrast, termination of an ordinary loop is not exceptional.
Unless the loop was meant to be infinite, termination is normal and expected.

**Note**: Don't use a `throw` as simply an alternative way of returning a value from a function.

**Exception**: Some systems, such as hard-real time systems require a guarantee that an action is taken in a (typically short) constant maximum time known before execution starts. Such systems can use exceptions only if there is tool support for accurately predicting the maximum time to recover from a `throw`.

**See also**: [RAII](#Rc-raii)

**See also**: [discussion](#Sd-noexcept)


<a name="Re-errors"></a>
### E.3: Use exceptions for error handling only

**Reason**. To keep error handling separated from "ordinary code."
C++ implementations tend to be optimized based on the assumption that exceptions are rare.

**Example; don't**:

	int find_index(vector<string>& vec, const string& x)	// don't: exception not used for error handling
	{
		try {
			for (int i =0; i<vec.size(); ++i)
				if (vec[i]==x) throw i;	// found x
		} catch (int i) {
			return i;
		}
		return -1;	// not found
	}

This is more complicated and most likely runs much slower than the obvious alternative.
There is nothing exceptional about finding a value in a `vector`.


<a name="Re-design-invariants"></a>
### E.4: Design your error-handling strategy around invariants

**Reason**: To use an objects it must be in a valid state (defined formally or informally by an invariant)
and to recover from an error every object not destroyed must be in a valid state.

**Note**: An [invariant](#Rc-struct) is logical condition for the members of an object that a constructor must establish for the public member functions to assume.


<a name="Re-invariant"></a>
### E.5: Let a constructor establish an invariant, and throw if it cannot

**Reason**: Leaving an object without its invariant aestablished is asking for trouble.
Not all member function can be called.

**Example**:

	???

**See also**: [If a constructor cannot construct a valid object, throw an exception](#Rc-throw)

**Enforcement**: ???


<a name="Re-raii"></a>
### E.6: Use RAII to prevent leaks

**Reason**: Leaks are typically unacceptable. RAII ("Resource Acquisition Is Initialization") is the simplest, most systematic way of preventing leaks.

**Example**:

	void f1(int i)	// Bad: possibly leak
	{
		int* p = new int[12];
		// ...
		if (i<17) throw Bad {"in f()",i};
		// ...
	}

We could carefully release the resource before the throw

	void f2(int i)	// Clumsy: explicit release
	{
		int* p = new int[12];
		// ...
		if (i<17) {
			delete p;
			throw Bad {"in f()",i};
		}
		// ...
	}

This is verbose. In larger code with multiple possible `throw`s explicit releases become repetitive and error-prone.

	void f3(int i)	// OK: resource management done by a handle
	{
		auto p = make_unique<int[12]>();
		// ...
		if (i<17) throw Bad {"in f()",i};
		// ...
	}

Note that this works even when the `throw` is implicit because it happened in a called function:

	void f4(int i)	// OK: resource management done by a handle
	{
		auto p = make_unique<int[12]>();
		// ...
		helper(i);	// may throw
		// ...
	}

Unless you really need pointer semantics, use a local resource object:

	void f5(int i)	// OK: resource management done by local object
	{
		vector<int> v(12);
		// ...
		helper(i);	// may throw
		// ...
	}

**Note**: If there is no obvious resource handle, cleanup actions can be represented by a [`Finally` object](#Re-finally)

**Note**: But what do we do if we are writing a program where exceptions cannot be used?
First challenge that assumption; there are many anti-exceptions myths around.
We know of only a few good reasons:

* We are on a system so small that the exception support would eat up most of our 2K or memory.
* We are in a hard-real-time system and we don't have tools that allows us that an exception is handled withon the required time.
* We are in a system with tons of legacy code using lots of pointers in difficult-to-understand ways
(in particular without a recognizable ownership strategy) so that exceptions could cause leaks.
* We get fired if we challenge our manager's ancient wisdom.

Only the first of these reasons is fundamental,
so whenever possible, use exception to implement RAII.
When exceptions cannot be used, simulate RAII.
That is, systematically check that objects are valid after construction and still release all resources in the destructor.
One strategy is to add a `valid()` operation to every resource handle:

	void f()
	{
		Vector<string> vs(100);	// not std::vector: valid() added
		if (!vs.valid()) {
			// handle error or exit
		}
		
		Ifstream fs("foo");		// not std::ifstream: valid() added
		if (!fs.valid()) {
			// handle error or exit
		}

		// ...
	} // destructors clean up as usual
	
Obviously, this increases the size of the code,
doesn't allow for implicit propagation of "exceptions" (`valid()` checks),
and `valid()` checks can be forgotten.
Prefer to use exceptions.

**See also**: [discussion](##Sd-noexcept).

**Enforcement**: ???


<a name="Re-precondition"></a>
### E.7: State your preconditions

**Reason**: To avoid interface errors.

**See also**: [precondition rule](#Ri-???).


<a name="Re-postcondition"></a>
### E.8: State your postconditions

**Reason**: To avoid interface errors.

**See also**: [postcondition rule](#Ri-???).


<a name="Re-noexcept"></a>
### E.12: Use `noexcept` when exiting a function because of a `throw` is impossible or unacceptable

**Reason**: To make error handling systematic, robust, and efficient.

**Example**:

	double compute(double d) noexcept
	{
		return log(sqrt(d<=0? 1 : d));
	}

Here, I know that `compute` will not throw because it is composed out of operations that don't throw. By declaring `compute` to be `noexcept` I give the compiler and human readers information that can make it easier for them to understand and manipulate 'compute`.

**Note**: Many standard library functions are `noexcept` including all the standard library functions "inherited" from the C standard library.

**Example**:

	vector<double> munge(const vector<double>& v) noexcept
	{
		vector<double> v2(v.size());
		// ... do something ...
	}

The `noexcept` here states that I am not willing or able to handle the situation where I cannot construct the local `vector`. That is, I consider memory exhaustion a serious design error (on line with hardware failures) so that I'm willing to crash the program if it happens.

**See also**: [discussion](#Sd-noexcept).


<a name="Re-never-throw"></a>
### E.13: Never throw while being the direct owner of an object

**Reason**: That would be a leak.

**Example**:

	void leak(int x)	// don't: may leak
	{
		auto p = new int{7};
		int (x<0) throw Get_me_out_of_here{};	// may leak *p
		// ...
		delete p;	// we may never get here
	}

One way of avoiding such problems is to use resource handles consistently:

	void no_leak(int x)
	{
		auto p = make_unique<int>(7);
		int (x<0) throw Get_me_out_of_here{};	// will delete *p if necessary
		// ...
		// no need for delete p
	}

**See also**: ???resource rule ???


<a name="Re-exception-types"></a>
### E.14: Use purpose-designed user-defined types as exceptions (not built-in types)

**Reason**: A user-defined type is unlikely to clash with other people's exceptions.

**Example**:

	void my_code()
	{
		// ...
		throw Moonphase_error{};
		// ...
	}

	void your_code()
	{
		try {
			// ...
			my_code();
			// ...
		}
		catch(Bufferpool_exhausted) {
			// ...
		}
	}

**Example; don't**:

	void my_code()	// Don't
	{
		// ...
		throw 7;	// 7 means "moon in the 4th quarter"
		// ...
	}

	void your_code()	// Don't
	{
		try {
			// ...
			my_code();
			// ...
		}
		catch(int i) {	// i==7 means "input buffer too small"
			// ...
		}
	}

**Note**: The standard-library classes derived from `exception` should be used only as base classes or for exceptions that require only "generic" handling. Like built-in types, their use could class with other people's use of them.

**Example; don't**:

	void my_code()	// Don't
	{
		// ...
		throw runtime_error{"moon in the 4th quarter"};
		// ...
	}

	void your_code()	// Don't
	{
		try {
			// ...
			my_code();
			// ...
		}
		catch(runtime_error) {	// runtime_error means "input buffer too small"
			// ...
		}
	}


**See also**: [Discussion](#Sd-???)

**Enforcement**: Catch `throw` and `catch` of a built-in type. Maybe warn about `throw` and `catch` using an standard-library `exception` type. Obviously, exceptions derived from the `std::exception` hierarchy is fine.


<a name="Re-exception-ref"></a>
### E.15: Catch exceptions from a hierarchy by reference

**Reason**: To prevent slicing.

**Example**:

	void f()
	try {
		// ...
	}
	catch (exception e) {	// don't: may slice
		// ...
	}

Instead, use

	catch (exception& e) { /* ... */ }

**Enforcement**: Flag by-value exceptions if their type are part of a hierarchy (could require whole-program analysis to be perfect).


<a name="Re-never-fail"></a>
### E.16: Destructors, deallocation, and `swap` must never fail

**Reason**: We don't know how to write reliable programs if a destructor, a swap, or a memory deallocation fails; that is, if it exits by an exception or simply doesn't perform its required action.

**Example; don't**:

	class Connection {
		// ...
	public:
		~Connection()	// Don't: very bad destructor
		{
			if (cannot_disconnect()) throw I_give_up{information};
			// ...
		}
	};

**Note**: Many have tried to write reliable code violating this rule for examples such as a network connection that "refuses to close". To the best of our knowledge nobody has found a general way of doing this though occasionally, for very specific examples, you can get away with  setting some state for future cleanup. Every example, we have seen of this is error-prone, specialized, and usually buggy.

**Note**: The standard library assumes that destructors, deallocation functions (e.g., `operator delete`), and `swap` do not throw. If they do, basic standard library invariants are broken.

**Note**: Deallocation functions, including `operator delete`, must be `noexcept`. `swap` functions must be `noexcept`. Most destructors are implicitly `noexcept` by default. destructors, make them `noexcept`.

**Enforcement**: Catch destructors, deallocation operations, and `swap`s that `throw`. Catch such operations that are not `noexcept`.

**See also**: [discussion](#Sd-never-fail)


<a name="Re-not-always"></a>
### E.17: Don't try to catch every exception in every function

**Reason**: Catching an exception in a function that cannot take a meaningful recovery action leads to complexity and waste.
Let an exception propagate until it reaches a function that can handle it.
Let cleanup actions on the unwinding path be handles by [RAII](#Re-raii).

**Example; don't**:

	void f()	// bad
	{
		try {
			// ...
		}
		catch (...) {
			throw;	// propagate excption
		}
	}

**Enforcement**:

* Flag nested try-blocks.
* Flag source code files with a too high ratio of try-blocks to functions. (??? Problem: define "too high")


<a name="Re-catch"></a>
### E.18: Minimize the use of explicit `try`/`catch`

**Reason**: `try`/`catch` is verbose and non-trivial uses error-prone.

**Example**:

	???

**Enforcement**: ???


<a name="Re-finally"></a>
### E.19: Use a `Final_action` object to express cleanup if no suitable resource handle is available

**Reason**: `finally` is less verbose and harder to get wrong than `try`/`catch`.

**Example**:

	void f(int n)
	{
		void* p = malloc(1,n);
		auto __ = finally([] { free(p); });
		// ...
	}

**See also** ????
 

<a name="Re-no-throw"></a>
### E.25: ??? What to do in programs where exceptions cannot be thrown

**Note**: ??? mostly, you can afford exceptions and code gets simpler with exceptions ???


**See also**: [Discussion](#Sd-???).
