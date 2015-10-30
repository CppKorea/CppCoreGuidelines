# 함수
># F: Functions

함수는 동작이나 계산을 명세하는데 이것들은 하나의 상태에서 다른 상태로 일관성 있게 넘어가는 시스템입니다. 이것은 프로그램의 기초적인 설계 단위입니다.
>A function specifies an action or a computation that takes the system from one consistent state to the next. It is the fundamental building block of programs.

함수의 이름은 의미있게 작성이 되어야 합니다. 그래야 인자의 요구사항, 인자간의 관계와 결과를 명확하게 기술할 수 있습니다. 함수의 구현은 사양서가 아닙니다. 함수가 무엇을 해야하는지, 어떻게 동작하는지를 생각해 보세요.
함수는 모든 인터페이스에서 가장 중요한 부분입니다. 인터페이스 규칙을 참고 하세요.
>It should be possible to name a function meaningfully, to specify the requirements of its argument, and clearly state the relationship between the arguments and the result. An implementation is not a specification. Try to think about what a function does as well as about how it does it.
Functions are the most critical part in most interfaces, so see the interface rules.

함수 규칙 정리:
>Function rule summary:

함수 정의 규칙:
>Function definition rules:

* [F.1: "패키지" meaningful operations as carefully named functions](#Rf-package)
* [F.2: 함수는 하나의 동작만 수행해야 한다](#Rf-logical)
* [F.3: 함수를 간결하고 단순하게 유지시켜야 한다](#Rf-single)
* [F.4: 만약 함수가 컴파일 시간에 평가되어야 한다면 `constexpr`로 선언하라](#Rf-constexpr)
* [F.5: 만약 함수가 매우 작고 성능이 중요하다면 인라인으로 선언하라](#Rf-inline)
* [F.6: 만약 함수가 예외를 발생시키지 않는다면 `noexcept`로 선언하라](#Rf-noexcept)
* [F.7: 일반적으로 사용되는 함수라면 스마트 포인터 보다는 `T*`를 인자로 받아라](#Rf-smart)
* [F.8: Prefer pure functions](#Rf-pure)

>* [F.1: "Package" meaningful operations as carefully named functions](#Rf-package)
>* [F.2: A function should perform a single logical operation](#Rf-logical)
>* [F.2: A function should perform a single logical operation](#Rf-logical)
>* [F.3: Keep functions short and simple](#Rf-single)
>* [F.4: If a function may have to be evaluated at compile time, declare it `constexpr`](#Rf-constexpr)
>* [F.5: If a function is very small and time critical, declare it inline](#Rf-inline)
>* [F.6: If your function may not throw, declare it `noexcept`](#Rf-noexcept)
>* [F.7: For general use, take `T*` arguments rather than a smart pointers](#Rf-smart)
>* [F.8: Prefer pure functions](#Rf-pure)

인자 전달 규칙:
>Argument passing rules:

* [F.15: 정보를 전달 할 때 단순하고 전통적인 방식을 선호하라](#Rf-conventional)
* [F.16: 객체 하나를 지정 할 때는 `T*` 나 `owner<T*>` 또는 스마트 포인터를 사용하라](#Rf-ptr)
* [F.17: "null"이 유효하지 않는 값을 의미할 때는 `not_null<T>`를 사용하라](#Rf-nullptr)
* [F.18: 반 개방 범위를 나타날 때는 `array_view<T>` 또는 `array_view_p<T>`를 사용하라](#Rf-range)
* [F.19: C언어 형 문자열을 지정 할 때는 `zstring` 또는 `not_null<zstring>`을 사용하라](#Rf-string)
* [F.20: 크기가 큰 객체를 매개변수로 사용 할 때는 `const T&`를 사용하라](#Rf-const-T-ref)
* [F.21: 크기가 작은 객체를 매개변수로 사용 할 때는 `T`를 사용하라](#Rf-T)
* [F.22: 입출력 매개변수는 `T&`를 사용하라](#Rf-T-re)
* [F.23: 값을 이동하는데 비용이 많이 드는 출력 매개변수는 `T&`를 사용하라 (only)](#Rf-T-return-out)
* [F.24: Use a `TP&&` parameter when forwarding (only)](#Rf-pass-ref-ref)
* [F.25: Use a `T&&` parameter together with `move` for rare optimization opportunities](#Rf-pass-ref-move)
* [F.26: 포인터가 필요한 곳에서 소유권을 이동 시킬 때는 `unique_ptr<T>`를 사용하라](#Rf-unique_ptr)
* [F.27: 소유권을 공유 할 때는 `shared_ptr<T>`을 사용하라](#Rf-shared_ptr)

>* [F.15: Prefer simple and conventional ways of passing information](#Rf-conventional)
>* [F.16: Use `T*` or `owner<T*>` or a smart pointer to designate a single object](#Rf-ptr)
>* [F.17: Use a `not_null<T>` to indicate "null" is not a valid value](#Rf-nullptr)
>* [F.18: Use an `array_view<T>` or an `array_view_p<T>` to designate a half-open sequence](#Rf-range)
>* [F.19: Use a `zstring` or a `not_null<zstring>` to designate a C-style string](#Rf-string)
>* [F.20: Use a `const T&` parameter for a large object](#Rf-const-T-ref)
>* [F.21: Use a `T` parameter for a small object](#Rf-T)
>* [F.22: Use `T&` for an in-out-parameter](#Rf-T-re)
>* [F.23: Use `T&` for an out-parameter that is expensive to move (only)](#Rf-T-return-out)
>* [F.24: Use a `TP&&` parameter when forwarding (only)](#Rf-pass-ref-ref)
>* [F.25: Use a `T&&` parameter together with `move` for rare optimization opportunities](#Rf-pass-ref-move)
>* [F.26: Use a `unique_ptr<T>` to transfer ownership where a pointer is needed](#Rf-unique_ptr)
>* [F.27: Use a `shared_ptr<T>` to share ownership](#Rf-shared_ptr)

값 전달 규칙:
>Value return rules:

* [F.40: Prefer return values to out-parameters](#Rf-T-return)
* [F.41: Prefer to return tuples to multiple out-parameters](#Rf-T-multi)
* [F.42: Return a `T*` to indicate a position (only)](#Rf-return-ptr)
* [F.43: Never (directly or indirectly) return a pointer to a local object](#Rf-dangle)
* [F.44: Return a `T&` when "returning no object" isn't an option](#Rf-return-ref)
* [F.45: Don't return a `T&&`](#Rf-return-ref-ref)

기타 규칙:
>Other function rules:

* [F.50: Use a lambda when a function won't do (to capture local variables, or to write a local function)](#Rf-capture-vs-overload)
* [F.51: Prefer overloading over default arguments for virtual functions](#Rf-default-arg)
* [F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms](#Rf-reference-capture)
* [F.53: Avoid capturing by reference in lambdas that will be used nonlocally, including returned, stored on the heap, or passed to another thread](#Rf-value-capture)

함수는 람다와 함수 객체와 매우 비슷합니다. 다음 섹션을 참조 하세요.
>Functions have strong similarities to lambdas and function objects so see also Section ???.


<a name="SS-fct-def"></a>
## 함수 정의
>## F.def: Function definitions

함수 정의란 함수의 선언과 함수의 몸체인 구현을 말합니다.
>A function definition is a function declaration that also specifies the function's implementation, the function body.


<a name="Rf-package"></a>
### F.1: "패키지" 
>### F.1: "Package" meaningful operations as carefully named functions

**근거**: 공용코드를 분해해 보면 코드를 더 읽기 쉽게 만들고 재사용률을 높이고 복잡한 코드에서 에러를 줄일 수 있습니다.
만약 어떤 동작이 잘 정의되어 있다면 그것을 코드로 분리하고 이름을 지어주세요.
>**Reason**: Factoring out common code makes code more readable, more likely to be reused, and limit errors from complex code.
If something is a well-specified action, separate it out from its  surrounding code and give it a name.

**예, 하지 말 것**:
>**Example, don't**:

	void read_and_print(istream& is)	// read and print and int
	{
		int x;
		if (is>>x)
			cout << "the int is " << x << '\n';
		else
			cerr << "no int on input\n";
	}


`read_and_print`는 거의 모든것이 잘못 되어 있습니다.
이 함수는 읽고, (고정된 `ostream`에) 씁니다. 이 함수는 에러 메시지를 (고정된 `ostream`에) 쓰고 `int`형만을 다룹니다.
재사용도 없고, 논리적으로 구분 될 수 있는 동작들은 뒤섞여 있고 지역변수는 사용이 끝난 후에도 논리적 범위에 남아 있습니다.
작은 예를 들어보면, 이것은 문제가 없어 보입니다. 하지만 입력을 처리하고 출력을 처리하고, 에러를 처리해야 한다면 더 복잡해 집니다.
>Almost everything is wrong with `read_and_print`.
>It reads, it writes (to a fixed `ostream`), it write error messages (to a fixed `ostream`), it handles only `int`s.
>There is nothing to reuse, logically separate operations are intermingled and local variables are in scope after the end of their logical use.
>For a tiny example, this looks OK, but if the input opeartion, the output operation, and the error handling had been more complicated the tangled 
mess could become hard to understand.

**주의**: 한군데 이상에서 사용되는 사소하지 않은 람다를 작성한다면 함수에 이름을 짓고 (대부분 로컬이 아닌)변수에 할당하세요.
>**Note**: If you write a non-trivial lambda that potentially can be used in more than one place, give it a name by assigning it to a (usually non-local) variable.

**예**:
>**Example**:

	sort(a, b, [](T x, T y) { return x.valid() && y.valid() && x.value()<y.value(); });

람다에 이름을 짓게되면 표현식을 여러개의 논리적 부분으로 나눌 수 있고, 람다가 어떤 일을 하는지 가늠하게 해줍니다.
>Naming that lambda breaks up the expression into its logical parts and provides a strong hint to the meaning of the lambda.

	auto lessT = [](T x, T y) { return x.valid() && y.valid() && x.value()<y.value(); };
	
	sort(a, b, lessT);
	find_if(a,b, lessT);

유지보수나 성능을 고려하다면 짧은 코드가 항상 좋은것은 아닙니다.
>The shortest code is not always the best for performance or maintainability.

**예외**: 반복문, 람다를 반복문으로 사용하는 경우는 거의 이름을 지어줄 필요가 없습니다.
그러나 수십줄에서 수십페이지가 되는 큰 반복문에는 문제가 있습니다.
[함수를 간결하게 유지하라](#Rf-single)는 규칙은 "반복문을 간결하게 유지하라"를 의미합니다.
비슷한 경우로, 콜백함수 인자로 사용되는 람다가 중요한 경우도 있습니다. 물론 재사용 될지는 알 수 없습니다.
>**Exception**: Loop bodies, including lambdas used as loop bodies, rarely needs to be named.
>However, large loop bodies (e.g., dozens of lines or dozens of pages) can be a problem.
>The rule [Keep functions short](#Rf-single) implies "Keep loop bodies short."
>Similarly, lambdas used as callback arguments are sometimes non-trivial, yet unlikely to be re-usable.

**시행하기**:
>**Enforcement**:

[Keep functions short](#Rf-single)를 참조하세요.
여러곳에서 사용되는 동일하거나 매우 비슷한 람다는 표시해 두세요.
>* See [Keep functions short](#Rf-single)
>* Flag identical and very similar lambdas used in different places.


<a name="Rf-logical"></a>
### F.2: A function should perform a single logical operation

**Reason**: A function that performs a single operation is simpler to understand, test, and reuse.

**Example**: Consider

	void read_and_print()	// bad
	{
		int x;
		cin >> x;
		// check for errors
		cout << x << "\n";
	}

This is a monolith that is tied to a specific input and will never find a another (different) use. Instead, break functions up into suitable logical parts and parameterize:

	int read(istream& is)	// better
	{
		int x;
		is >> x;
		// check for errors
		return x;
	}

	void print(ostream& os, int x)
	{
		os << x << "\n";
	}

These can now be combined where needed:

	void read_and_print()
	{
		auto x = read(cin);
		print(cout, x);
	}

If there was a need, we could further templatize `read()` and `print()` on the data type, the I/O mechanism, etc. Example:

	auto read = [](auto& input, auto& value)	// better
	{
		input >> value;
		// check for errors
	}

	auto print(auto& output, const auto& value)
	{
		output << value << "\n";
	}

**Enforcement**:

* Consider functions with more than one "out" parameter suspicious. Use return values instead, including `tuple` for multiple return values.
* Consider "large" functions that don't fit on one editor screen suspicious. Consider factoring such a function into smaller well-named suboperations.
* Consider functions with 7 or more parameters suspicious.


<a name="Rf-single"></a>
### F.3: Keep functions short and simple

**Reason**: Large functions are hard to read, more likely to contain complex code, and more likely to have variables in larger than minimal scopes.
Functions with complex control stryuctures are more likely to be long and more likely to hide logical errors

**Example**: Consider

	double simpleFunc(double val, int flag1, int flag2)
		// simpleFunc: takes a value and calculates the expected ASIC output, given the two mode flags.
	{
  		double intermediate;
  		if (flag1 > 0) {
    		intermediate = func1(val);
    		if (flag2 % 2)
      			intermediate = sqrt(intermediate);
  		}
  		else if (flag1 == -1) {
    		intermediate = func1(-val);
    		if (flag2 % 2)
      			intermediate = sqrt(-intermediate);
    		flag1 = -flag1;
  		}
  		if (abs(flag2) > 10) {
   			intermediate = func2(intermediate);
  		}
  		switch (flag2 / 10) {
    	case 1: if (flag1 == -1) return finalize(intermediate, 1.171); break;
    	case 2: return finalize(intermediate, 13.1);
    	default: ;
  		}
  		return finalize(intermediate, 0.);
	}

This is too complex (and also pretty long).
How would you know if all possible alternatives have been correctly handled?
Yes, it break other rules also.

We can refactor:

	double func1_muon(double val, int flag)
	{
		// ???
	}
	
	double funct1_tau(double val, int flag1, int flag2)
	{
		// ???
	}
	
	double simpleFunc(double val, int flag1, int flag2)
		// simpleFunc: takes a value and calculates the expected ASIC output, given the two mode flags.
	{
  		if (flag1 > 0)
 			return func1_muon(val, flag2);
		if (flag1 == -1)
    		return func1_tau(-val, flag2);	// handled by func1_tau: flag1 = -flag1;
  		return 0.;
	}

**Note**: "It doesn't fit on a screen" is often a good practical definition of "far too large."
One-to-five-line functions should be considered normal.

**Note**: Break large functions up into smaller cohesive and named functions.
Small simple functions are easily inlined where the cost of a function call is significant.

**Enforcement**:

* Flag functions that do not "fit on a screen."
How big is a screen? Try 60 lines by 140 characters; that's roughly the maximum that's comfortable for a book page.
* Flag functions that are too complex. How complex is too complex?
You could use cyclomatic complexity. Try "more that 10 logical path through." Count a simple switch as one path.


<a name="Rf-constexpr"></a>
### F.4: If a function may have to be evaluated at compile time, declare it `constexpr`

**Reason**: `constexpr` is needed to tell the compiler to allow compile-time evaluation.

**Example**: The (in)famous factorial:

	constexpr int fac(int n)
	{
		constexpr int max_exp = 17;     // constexpr enables  this to be used in Expects
		Expects(0<=x && x<max_exp);		// prevent silliness and overflow
		int x = 1;
		for (int i=2; i<=n; ++i) x*= n;
		return x;
	}
	
This is C++14. For C++11, use a functional formulation of `fac()`.

**Note**: `constexpr` does not guarantee compile-time evaluation;
it just guarantees that the function can be evaluated at compile time for constant expression arguments if the programmer requires it or the compiler decides to do so to optimize.

	constexpr int min(int x, int y) { return x<y?x:y;}
	
	void test(int v)
	{
		int m1 = min(-1,2);				// probably compile-time evaluation
		constexpr int m2 = min(-1,2);	// compile-time evaluation
		int m3 = min(-1,v);				// run-time evaluation
		constexpr int m4 = min(-1,2);	// error: connot evaluate at compile-time
	}
	
**Note**: `constexpr` functions are pure: they can have no sideefects.

	int dcount = 0;
	constexpr int double(int v)
	{
		++dcount;		// error: attempted side effect from constexpr function
		return v+v;
	}
	
This is usually a very good thing.

**Note**: Don't try to make all functions `constexpr`. Most computation is best done at run time.

**Enforcement**: Imposible and unnecessary.
The compiler gives an error if a non-`constexpr` function is called where a constant is required.


<a name="Rf-inline"></a>
### F.5: If a function is very small and time critical, declare it `inline`

**Reason**: Some optimizers are good an inlining without hints from the programmer, but don't rely on it.
Measure! Over the last 40 years or so, we have been promised compilers that can inline better than humans without hints from humans.
We are still waiting.
Specifying `inline` encourages the compiler to do a better job.

**Exception**: Do not put an `inline` function in what is meant to be a stable interface unless you are really sure that it will not change.
An inline function is part of the ABI.

**Note**: `constexpr` implies `inline`.

**Note**: Member functions defined in-class are `inline` by default.

**Exception**: Template functions (incl. template member functions) must be in headers and therefore inline.

**Enforcement**: Flag `inline` functions that are more than three statements and could have been declared out of line (such as class member functions).
To fix: Declare the function out of line. [[NM: Certainly possible, but size-based metrics can be very annoying.]]


<a name="Rf-noexcept"></a>
### F.6: If your function may not throw, declare it `noexcept`

**Reason**: If an exception is not supposed to be thrown, the program cannot be assumed to cope with the error and should be terminated as soon as possible. Declaring a function `noexcept` helps optimizers by reducing the number of alternative execution paths. It also speeds up the exit after failure.

**Example**: Put `noexcept` on every function written completely in C or in any other language without exceptions.
The C++ standard library does that implicitly for all functions in the C standard library.

**Note**: `constexpr` functions cannot throw, so you don't need to use `noexcept` for those.

**Example**: You can use `noexcept` even on functions that can throw:

	vector<string> collect(istream& is) noexcept
	{
		vector<string> res;
		for(string s; is>>s; )
			res.push_back(s);
		return res;
	}

If `collect()` runs out of memory, the program crashes.
Unless the program is crafted to survive memory exhaustion, that may be just the right thing to do;
`terminate()` may generate suitable error log information (but after memory runs out it is hard to do anything clever).

**Note**: In most programs, most functions can throw
(e.g., because they use `new`, call functions that do, or use library functions that reports failure by throwing),
so don't just springle `noexcept` all over the place.
`noexcept` is most useful for frequently used, low-level functions.

**Note**: Destructors, `swap` functions, move operations, and default constructors should never throw.


**Enforcement**:

* Flag functions that are not `noexcept`, yet cannot thow
* Flag throwing `swap`, `move`, destructors, and default constructors.


<a name="Rf-smart"></a>
### F.7: For general use, take `T*` arguments rather than a smart pointers

**Reason**: Passing a smart pointer transfers or shares ownership.
Passing by smart pointer restricts the use of a function to callers that use smart pointers.
Passing a shared smart pointer (e.g., `std::shared_ptr`) implies a run-time cost.

**Example**:

	void f(int*);		// accepts any int*
	void g(unique_ptr<int>);	// can only accept ints for which you want to transfer ownership
	void g(shared_ptr<int>);	// can only accept ints for which you are willing to share ownership

**Note**: We can catch dangling pointers statically, so we don't need to rely on resource management to avoid violations from dangling pointers.

**See also**: Discussion of [smart pointer use](#Rr-summary-smartptrs).

**Enforcement**: Flag smart pointer arguments.


<a name="Rf-pure"></a>
### F.8: Prefer pure functions


**Reason**: Pure functions are easier to reason about, sometimes easier to optimize (and even parallelize), and sometimes can be memoized.

**Example**:

    template<class T>
	auto square(T t) { return t*t; }
	
**Note**: `constexpr` functions are pure.
	
**Enforcement**: not possible.


<a name="SS-call"></a>
## F.call: Argument passing

There are a variety of ways to pass arguments to a function and to return values.


<a name="Rf-conventional"></a>
### Rule F.15: Prefer simple and conventional ways of passing information

**Reason**: Using "unusual and clever" techniques causes surprises, slows understanding by other programmers, and encourages bugs.
If you really feel the need for an optimization beyond the common techniques, measure to ensure that it really is an improvement,
and document/comment because the improvement may not be portable.

![Normal parameter passing table](./param-passing-normal.png "Normal parameter passing")

**For an "output-only" value:** Prefer return values to output parameters.
This includes large objects like standard containers that use implicit move operations for performance and to avoid explicit memory management.
If you have multiple values to return, [use a tuple](#Rf-T-multi) or similar multi-member type.

**Example**:

	vector<const int*> find_all(const vector<int>&, int x);	// return pointers to elements with the value x
	
**Example, bad**:

	void find_all(const vector<int>&, vector<const int*>& out, int x);	// place pointers to elements with value x in out

**Exceptions**:

* For non-value types, such as types in an inheritance hierarchy, return the object by `unique_ptr` or `shared_ptr`.
* If a type is expensive to move (e.g., `array<BigPOD>`), consider allocating it on the free store and return a handle (e.g., `unique_ptr`), or passing it in a non-`const` reference to a target object to fill (to be used as an out-parameter).
* In the special case of allowing a caller to reuse an object that carries capacity (e.g., `std::string`, `std::vector`) across multiple calls to the function in an inner loop, treat it as an in/out parameter instead and pass by `&`. This one use of the more generally named "caller-allocated out" pattern.

**For an "in-out" parameter:** Pass by non-`const` reference. This makes it clear to callers that the object is assumed to be modified.

**For an "input-only" value:** If the object is cheap to copy, pass by value.
Otherwise, pass by `const&`. It is useful to know that a function does not mutate an argument, and both allow initialization by rvalues.
What is "cheap to copy" depends on the machine architecture, but two or three words (doubles, pointers, references) are usually best passed by value.
In particular, an object passed by value does not require an extra reference to access from the function.

![Advanced parameter passing table](./param-passing-advanced.png "Advanced parameter passing")

For advanced uses (only), where you really need to optimize for rvalues passed to "input-only" parameters:

* If the function is going to unconditionally move from the argument, take it by `&&`.
* If the function is going to keep a copy of the argument, in addition to passing by `const&` add an overload that passes the parameter by `&&` and in the body `std::move`s it to its destination. (See [F.25](#Rf-pass-ref-move).)
* In special cases, such as multiple "input + copy" parameters, consider using perfect forwarding. (See [F.24](#Rf-pass-ref-ref).)

**Example**:

		int multiply(int, int); // just input ints, pass by value
 
		string& concatenate(string&, const string& suffix); // suffix is input-only but not as cheap as an int, pass by const&
		
		void sink(unique_ptr<widget>);	// input only, and consumes the widget
 
Avoid "esoteric techniques" such as:

* Passing arguments as `T&&` "for efficiency". Most rumors about performance advantages from passing by `&&` are false or brittle (but see [F.25](#Rf-pass-ref-move).)
* Returning `const T&` from assignments and similar operations.

**Example**: Assuming that `Matrix` has move operations (possibly by keeping its elements in a `std::vector`.

	Matrix operator+(const Matrix& a, const Matrix& b)
	{
		Matrix res;
		// ... fill res with the sum ...
		return res;
	}

	Matrix x = m1+m2;	// move constructor

	y = m3+m3;			// move assignment

**Note**: The (optional) return value optimization doesn't handle the assignment case.

**See also**: [implicit arguments](#Ri-explicit).

**Enforcement**: This is a philosophical guideline that is infeasible to check directly and completely.
However, many of the the detailed rules (F.16-F.45) can be checked,
such as passing a `const int&`, returning an `array<BigPOD>` by value, and returning a pointer to fre store alloced by the function.


<a name="Rf-ptr"></a>
### F.16: Use `T*` or `owner<T*>` to designate a single object

**Reason**: In traditional C and C++ code, "Plain `T*` is used for many weakly-related purposes, such as

* Identify a (single) object (not to be deleted by this function)
* Point to an object allocated on the free store (and delete it later)
* Hold the `nullptr`
* Identify a C-style string (zero-terminated array of characters)
* Identify an array with a length specified separately
* Identify a location in an array

Confusion about what meaning a `T*` is the source of many serious errors, so using separate names for pointers of these separate uses makes code clearer.
For debugging, `owner<T*>` and `not_null<T>` can be instrumented to check.
For example, `not_null<T*>` makes it obvious to a reader (human or machine) that a test for `nullptr` is not necessary before dereference.

**Example**: Consider

	int length(Record* p);

When I call `length(r)` should I test for `r==nullptr` first? Should the implementation of `length()` test for `p==nullptr`?

	int length(not_null<Record*> p);	// it is the caller's job to make sure p!=nullptr

	int length(Record* p);	// the implementor of length() must assume that p==nullptr is possible

**Note**: A `not_null<T>` is assumed not to be the `nullptr`; a `T*` may be the `nullptr`; both can be represented in memory as a `T*` (so no run-time overhead is implied).

**Note**: `owner<T*>` represents ownership.

**Also**: Assume that a `T*` obtained from a smart pointer to `T` (e.g., unique_ptr<`T`>) pointes to a single element.

**See also**: [Support library](#S-support).

**Enforcement**:

* (Simple) ((Bounds)) Warn for any arithmetic operation on an expression of pointer type that results in a value of pointer type.


<a name="Rf-nullptr"></a>
### F.17: Use a `not_null<T>` to indicate that "null" is not a valid value

**Reason**: Clarity. Making it clear that a test for null isn't needed.

**Example**:

	not_null<T*> check(T* p) { if (p) return not_null<T*>{p}; throw Unexpected_nullptr{}; }

	void computer(not_null<array_view<int>> p)
	{
		if (0<p.size()) {	// bad: redundant test
			// ...
		}
	}

**Note**: `not_null` is not just for built-in pointers. It works for `array_view`, `string_view`, `unique_ptr`, `shared_ptr`, and other pointer-like types.

**Enforcement**:

* (Simple) Warn if a raw pointer is dereferenced without being tested against `nullptr` (or equivalent) within a function, suggest it is declared `not_null` instead.
* (Simple) Error if a raw pointer is sometimes dereferenced after first being tested against `nullptr` (or equivalent) within the function and sometimes is not.
* (Simple) Warn if a `not_null` pointer is tested against `nullptr` within a function.


<a name="Rf-range"></a>
### F.18: Use an `array_view<T>` or an `array_view_p<T>` to designate a half-open sequence

**Reason**: Informal/non-explicit ranges are a source of errors

**Example**:

	X* find(array_view<X> r, const X& v)	// find v in r

	vector<X> vec;
	// ...
	auto p = find({vec.begin(),vec.end()},X{});	// find X{} in vec

**Note**: Ranges are extremely common in C++ code. Typically, they are implicit and their correct use is very hard to ensure. In particular, given a pair of arguments `(p,n)` designating an array [`p`:`p+n`), it is in general impossible to know if there really are n elements to access following `*p`. `array_view<T>` and `array_view_p<T>` are simple helper classes designating a [p:q) range and a range starting with p and ending with the first element for which a predicate is true, respectively.

**Note**: an `array_view<T>` object does not own its elements and is so small that it can be passed by value.

**Note**: Passing an `array_view` object as an argument is exactly as efficient as passing a pair of pointer arguments or passing a pointer and an integer count.

**See also**: [Support library](#S-support).

**Enforcement**: (Complex) Warn where accesses to pointer parameters are bounded by other parameters that are integral types and suggest they could use `array_view` instead.


<a name="Rf-string"></a>
### F.19: Use a `zstring` or a `not_null<zstring>` to designate a C-style string

**Reason**: C-style strings are ubiquitous.
They are defined by convention: zero-terminated arrays of characters.
Functions are inconsistent in their use of `nullptr` and we must be more explicit.

**Example**: Consider

	int length(const char* p);

When I call `length(s)` should I test for `s==nullptr` first? Should the implementation of `length()` test for `p==nullptr`?

	int length(zstring p);	// it is the caller's job to make sure p!=nullptr

	int length(not_null<Zstring> p);	// the implementor of length() must assume that p==nullptr is possible

**Note**: `zstring` do not represent ownership.

**See also**: [Support library](#S-support).


<a name="Rf-const-T-ref"></a>
### F.20: Use a `const T&` parameter for a large object

**Reason**: Copying large objects can be expensive. A `const T&` is always cheap and protects the caller from unintended modification.

**Example**:

	void fct(const string& s);	// OK: pass by const reference; always checp

	void fct2(string s);		// bad: potentially expensive

**Exception**: Sinks (that is, a function that eventually destroys an object or passes it along to another sink), may benefit ???

**Note**: A reference may be assumed to refer to a valid object (language rule).
There in no (legitimate) "null reference."
If you need the notion of an optional value, use a pointer, `std::optional`, or a special value used to denote "no value."

**Enforcement**:

* (Simple) ((Foundation)) Warn when a parameter being passed by value has a size greater than `4*sizeof(int)`.
Suggest using a `const` reference instead.


<a name="Rf-T"></a>
### F.21: Use a `T` parameter for a small object

**Reason**: Nothing beats the simplicity and safety of copying.
For small objects (up to two or three words) is is also faster than alternatives.

**Example**:

	void fct(int x);		// OK: Unbeatable

	void fct(const int& x);	// bad: overhead on access in fct2()

	void fct(int& x);		// OK, but means something else; use only for an "out parameter"

**Enforcement**:

* (Simple) ((Foundation)) Warn when a `const` parameter being passed by reference has a size less than `3*sizeof(int)`. Suggest passing by value instead.


<a name="Rf-T-ref"></a>
### F.22: Use a `T&` for an in-out-parameter

**Reason**: A called function can write to a non-`const` reference argument, so assume that it does.

**Example**:

	void update(Record& r);	// assume that update writes to r
	
**Note**: A `T&` argument can pass information into a function as well as well as out of it.
Thus `T&` could be and in-out-parameter. That can in itself be a problem and a source of errors:

	void f(string& s)
	{
		s = "New York";	// non-obvious error
	}
	
	string g()
	{
		string buffer = ".................................";
		f(buffer);
		// ...
	}
	
Here, the writer of `g()` is supplying a buffer for `f()` to fill,
but `f()` simply replaces it (at a somewhat higher cost than a simple copy of the characters).
If the writer of `g()` makes an assumption about the size of `buffer` a bad logic error can happen.

**Enforcement**:

* (Moderate) ((Foundation)) Warn about functions with non-`const` reference arguments that do *not* write to them.
* Flag functions that take a `T&` and replace the `T` referred to, rather what the contents of that `T`


<a name="Rf-T-return-out"></a>
### F.23: Use `T&` for an out-parameter that is expensive to move (only)

**Reason**: A return value is harder to miss and harder to miuse than a `T&` (an in-out parameter); [see also](#Rf-return); [see also](#Rf-T-multi).

**Example**:

	struct Package {
		char header[16];
		char load[2024-16];
	};
	
	Package fill();			// Bad: large return value
	void fill(Package&);	// OK
	
	int val();				// OK
	val(int&);				// Bad: Is val reading its argument

**Enforcement**: Hard to choose a cutover value for the size of the value returned.


<a name="Rf-pass-ref-ref"></a>
### F.24: Use a `TP&&` parameter when forwarding (only)

**Reason**: When `TP` is a template type parameter, `TP&&` is a forwarding reference -- it both *ignores* and *preserves* const-ness and rvalue-ness. Therefore any code that uses a `T&&` is implicitly declaring that it itself doesn't care about the variable's const-ness and rvalue-ness (because it is ignored), but that intends to pass the value onward to other code that does care about const-ness and rvalue-ness (because it is preserved). When used as a parameter `TP&&` is safe because any temporary objects passed from the caller will live for the duration of the function call. A parameter of type `TP&&` should essentially always be passed onward via `std::forward` in the body of the function.

**Example**:

	template <class F, class... Args>
    inline auto invoke(F&& f, Args&&... args) {
        return forward<F>(f)(forward<Args>(args)...);
    }

**Enforcement**: Flag a function that takes a `TP&&` parameter (where `TP` is a template type parameter name) and uses it without `std::forward`.


<a name ="Rf-pass-ref-move"></a>
### F.25: Use a `T&&` parameter together with `move` for rare optimization opportunities

**Reason**: Moving from an object leaves an object in its moved-from state behind.
In general, moved-from objects are dangerous. The only guaranteed operation is destruction (more generally, member functions without preconditions).
The standard library additionally requires that a moved-from object can be assigned to.
If you have performance justification to optimize for rvalues, overload on `&&` and then `move` from the parameter ([example of such overloading](#)).

**Example**:

	void somefct(string&&);
	
	void user()
	{
		string s = "this is going to be fun!";
		// ...
		somefct(std::move(s));		// we don't need s any more, give it to somefct()
		//
		cout << s << '\n';			// Oops! What happens here?
	}

**Enforcement**:

* Flag all `X&&` parameters (where `X` is not a template type parameter name) and uses it without `std::move`.
* Flag access to moved-from objects


<a name="Rf-unique_ptr"></a>
### F.26: Use a `unique_ptr<T>` to transfer ownership where a pointer is needed

**Reason**: Using `unique_ptr` is the cheapest way to pass a pointer safely.

**Example**:

	unique_ptr<Shape> get_shape(istream& is)	// assemble shape from input stream
	{
		auto kind = read_header(is); // read header and identify the next shape on input
		switch (kind) {
		case kCicle:
			return make_unique<Circle>(is);
		case kTriangle:
			return make_unique<Triangle>(is);
		// ...
	}

**Note**: You need to pass a pointer rather than an object if what you are transferring is an object from a class hierarchy that is to be used through an interface (base class).

**Enforcement**: (Simple) Warn if a function returns a locally-allocated raw pointer. Suggest using either `unique_ptr` or `shared_ptr` instead.


<a name="Rf-shared_ptr"></a>
### F.27: Use a `shared_ptr<T>` to share ownership

**Reason**: Using `std::shared_ptr` is the standard way to represent shared ownership. That is, the last owner deletes the object.

**Example**:

	shared_ptr<Image> im { read_image(somewhere); };
	
	std::thread t0 {shade,args0,top_left,im};
	std::thread t1 {shade,args1,top_right,im};
	std::thread t2 {shade,args2,bottom_left,im};
	std::thread t3 {shade,args3,bottom_right,im};
	
	// detach treads
	// last thread to finish deletes the image


**Note**: Prefer a `unique_ptr` over a `shared_ptr` if there is never more than one owner at a time.
`shared_ptr` is for shared ownership.

**Alternative**: Have a single object own the shared object (e.g. a scoped object) and destroy that (preferably implicitly) when all users have completd.

**Enforcement**: (Not enforceable) This is a too complex pattern to reliably detect.


<a name="Rf-T-return"></a>
### F.40: Prefer return values to out-parameters

**Reason**: It's self-documenting. A `&` parameter could be either in/out or out-only.

**Example**:

	void incr(int&);
	int incr();
	
	int i = 0;
	incr(i);
	i = incr(i);

**Enforcement**: Flag non-const reference parameters that are not read before being written to and are a type that could be cheaply returned.


<a name="Rf-T-multi"></a>
### F.41: Prefer to return tuples to multiple out-parameters

**Reason**: A return value is self-documenting as an "output-only" value.
And yes, C++ does have multiple return values, by convention of using a `tuple`, with the extra convenience of `tie` at the call site.

**Example**:

    int f( const string& input, /*output only*/ string& output_data ) { // BAD: output-only parameter documented in a comment
	    // ...
		output_data = something();
		return status;
	}

    tuple<int,string> f( const string& input ) { // GOOD: self-documenting
	    // ...
		return make_tuple(something(), status);
    }

In fact, C++98's standard library already used this convenient feature, because a `pair` is like a two-element `tuple`.
For example, given a `set<string> myset`, consider:

    // C++98
	result = myset.insert( “Hello” );
	if (result.second) do_something_with( result.first );    // workaround
	
With C++11 we can write this, putting the results directly in existing local variables:

	Sometype iter;                                          // default initialize if we haven't already
	Someothertype success;                                  // used these variables for some other purpose

	tie( iter, success ) = myset.insert( “Hello” );         // normal return value
	if (success) do_something_with( iter );

**Exception**: For types like `string` and `vector` that carry additional capacity, it can sometimes be useful to treat it as in/out instead by using the "caller-allocated out" pattern, which is to pass an output-only object by reference to non-`const` so that when the callee writes to it the object can reuse any capacity or other resources that it already contains. This technique can dramatically reduce the number of allocations in a loop that repeatedly calls other functions to get string values, by using a single string object for the entire loop.

**Note**: In some cases it may be useful to return a specific, user-defined `Value_or_error` type along the lines of `variant<T,error_code>`,
rather than using the generic `tuple`.

**Enforcement**:

    * Output parameters should be replaced by return values.
	An output parameter is one that the function writes to, invokes a non-`const` member function, or passes on as a non-`const`.


<a name="Rf-return-ptr"></a>
### F.42: Return a `T*` to indicate a position (only)

**Reason**: That's what pointers are good for.
Returning a `T*` to transfer ownership is a misuse.

**Note**: Do not return a pointer to something that is not in the caller's scope.

**Example**:

		Node* find(Node* t, const string& s)	// find s in a binary tree of Nodes
		{
			if (t->name==s) return t;
			if (auto p = find(t->left,s)) return p;
			if (auto p = find(t->right,s)) return p;
			return nullptr;
		}

If it isn't the `nullptr`, the pointer returned by `find` indicates a `Node` holding `s`.
Importantly, that does not imply a transfer of ownership of the pointed-to object to the caller.

**Note**: Positions can also be transferred by iterators, indices, and references.

**Example, bad**:

	int* f()
	{
		int x = 7;
		// ...
		return &x;		// Bad: returns pointer to object that is about to be destroyed
	}

This applies to references as well:

	int& f()
	{
		int x = 7;
		// ...
		return x;	// Bad: returns refence to object that is about to be destroyed
	}

**See also**: [discussion of dangling pointer prevention](#???).

**Enforcement**: A slightly diffent variant of the problem is placing pointers in a container that outlives the objects pointed to.

* Compilers tend to catch return of reference to locals and could in many cases catch return of pointers to locals.
* Static analysis can catch most (all?) common patterns of the use of pointers indicating positions (thus eliminating dangling pointers)


<a name="Rf-dangle"></a>
### F.43: Never (directly or indirectly) return a pointer to a local object

**Reason**: To avoid the crashes and data corruption that can result from the use of such a dangling pointer.

**Example**, bad: After the return from a function its local objects no longer exist:

	int* f()
	{
		int fx = 9;
		return &fx;	// BAD
	}
	
	void g(int* p)	// looks innocent enough
	{
		int gx;
		cout << "*p == " << *p << '\n';
		*p = 999;
		cout << "gx == " << gx << '\n';
	}
		
	void h()
	{
		int* p = f();
		int z = *p;		// read from abandoned stack frame (bad)
		g(p);			// pass pointer to abandoned stack frame to function (bad)
		
	}
	
Here on one popular implementation I got the output

	*p == 9
	cx == 999
	
I expected that because the call of `g()` reuses the stack space abandoned by the call of `f()` so `*p` refers to the space now occupied by `gx`.

Imagine what would happen if `fx` and `gx` were of different types.
Imagine what would happen if `fx` or `gx` was a type with an invariant.
Imagine what would happen if more that dangling pointer was passed around among a larger set of functions.
Imagine what a cracker could do with that dangling pointer.

Fortunately, most (all?) modern compilers catch and warn against this simple case.

**Note**: you can construct similar examples using references.

**Note**: This applies only to non-`static` local variables.
All `static` variables are (as their name indicates) statically allocated, so that pointers to them cannot dangle.

**Example**, bad: Not all examples of leaking a pointer to a local variable are that obvious:

	int* glob;		// global variables are bad in so many ways

	template<class T>
	void steal(T x)
	{
		glob = x();	// BAD
	}

	void f()
	{
		int i = 99;
		steal([&] { return &i; });
	}

	int main()
	{
		f();
		cout << *glob << '\n';
	}
	
Here I managed to read the location abandoned by the call of `f`.
The pointer stored in `glob` could be used much later and cause trouble in unpredictable ways.

**Note**: The address of a local variable can be "returned"/leaked by a return statement,
by a `T&` out-parameter, as a member of a returned object, as an element of a returned array, and more.

**Note**: Similar examples can be constructed "leaking" a pointer from an inner scope to an outer one;
such examples are handled equivalently to leaks of pointers out of a function.

**See also**: Another way of getting dangling pointers is [pointer invalidation](#???).
It can be detected/prevented with similar techniques.

**Enforcement**: Preventable through static analysis.


<a name="Rf-return-ref"></a>
### F.44: Return a `T&` when "returning no object" isn't an option

**Reason**: The language guarantees that a `T&` refers to an object, so that testing for `nullptr` isn't necessary.

**See also**: The return of a reference must not imply transfer of ownership:
[discussion of dangling pointer prevention](#???) and [discussion of ownership](#???).

**Example**:

	???

**Enforcement**: ???


<a name="Rf-return-ref-ref"></a>
### F.45: Don't return a `T&&`

**Reason**: It's asking to return a reference to a destroyed temporary object. A `&&` is a magnet for temporary objects. This is fine when the reference to the temporary is being passed "downward" to a callee, because the temporary is guaranteed to outlive the function call. (See [F.24](#RF-pass-ref-ref) and [F.25](#Rf-pass-ref-move).) However, it's not fine when passing such a reference "upward" to a larger caller scope. See also [F54](#Rf-local-ref-ref).

For passthrough functions that pass in parameters (by ordinary reference or by perfect forwarding) and want to return values, use simple `auto` return type deduction (not `auto&&`).

**Example; bad**: If `F` returns by value, this function returns a reference to a temporary.

	template<class F>
	auto&& wrapper(F f) {
		log_call(typeid(f)); // or whatever instrumentation
	    return f();
	}

**Example; good**: Better:
	
	template<class F>
	auto wrapper(F f) {
		log_call(typeid(f)); // or whatever instrumentation
	    return f();
	}

**Exception**: `std::move` and `std::forward` do return `&&`, but they are just casts -- used by convention only in expression contexts where a reference to a temporary object is passed along within the same expression before the temporary is destroyed. We don't know of any other good examples of returning `&&`.

**Enforcement**: Flag any use of `&&` as a return type, except in `std::move` and `std::forward`.


<a name="Rf-capture-vs-overload"></a>
### F.50: Use a lambda when a function won't do (to capture local variables, or to write a local function)

**Reason**: Functions can't capture local variables or be declared at local scope; if you need those things, prefer a lambda where possible, and a handwritten function object where not. On the other hand, lambdas and function objects don't overload; if you need to overload, prefer a function (the workarounds to make lambdas overload are ornate). If either will work, prefer writing a function; use the simplest tool necessary.

**Example**:

	// writing a function that should only take an int or a string -- overloading is natural
	void f(int);
	void f(const string&);
	
	// writing a function object that needs to capture local state and appear
	// at statement or expression scope -- a lambda is natural
	vector<work> v = lots_of_work();
	for(int tasknum = 0; tasknum < max; ++tasknum) {
	    pool.run([=, &v]{
			/*
			...
			... process 1/max-th of v, the tasknum-th chunk
			...
			*/
		});
	}
	pool.join();

**Exception**: Generic lambdas offer a concise way to write function templates and so can be useful even when a normal function template would do equally well with a little more syntax. This advantage will probably disappear in the future once all functions gain the ability to have Concept parameters.

**Enforcement**:

    * Warn on use of a named non-generic lambda (e.g., `auto x = [](int i){ /*...*/; };`) that captures nothing and appears at global scope. Write an ordinary function instead.



<a name="Rf-default-args"></a>
### F.51: Prefer overloading over default arguments for virtual functions
??? possibly other situations?

**Reason**: Virtual function overrides do not inherit default arguments, leading to surprises.

**Example; bad**:

	class base {
	public:
		virtual int multiply(int value, int factor = 2) = 0;
	};

	class derived : public base {
	public:
		override int multiply(int value, int factor = 10);
	};
	
	derived d;
	base& b = d;
	
	b.multiply(10);	// these two calls will call the same function but
	d.multiply(10); // with different arguments and so different results

**Enforcement**: Flag all uses of default arguments in virtual functions.


<a name="Rf-reference-capture"></a>
### F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms

**Reason**: For efficiency and correctness, you nearly always want to capture by reference when using the lambda locally. This includes when writing or calling parallel algorithms that are local because they join before returning.

**Example**: This is a simple three-stage parallel pipeline. Each `stage` object encapsulates a worker thread and a queue, has a `process` function to enqueue work, and in its destructor automatically blocks waiting for the queue to empty before ending the thread.

	void send_packets( buffers& bufs ) {
	    stage encryptor  ([] (buffer& b){ encrypt(b); });
	    stage compressor ([&](buffer& b){ compress(b); encryptor.process(b); });
	    stage decorator  ([&](buffer& b){ decorate(b); compressor.process(b); });
	    for (auto& b : bufs) { decorator.process(b); }
	} // automatically blocks waiting for pipeline to finish

**Enforcement**: ???


<a name="Rf-value-capture"></a>
### F.53: Avoid capturing by reference in lambdas that will be used nonlocally, including returned, stored on the heap, or passed to another thread

**Reason**: Pointers and references to locals shouldn't outlive their scope. Lambdas that capture by reference are just another place to store a reference to a local object, and shouldn't do so if they (or a copy) outlive the scope.

**Example**:

	{
		// ...
		
		// a, b, c are local variables
		background_thread.queue_work([=]{ process(a,b,c); });	// want copies of a, b, and c
	}

**Enforcement**: ???

