# <a name="S-interfaces"></a> I: Interfaces

An interface is a contract between two parts of a program. Precisely stating what is expected of a supplier of a service and a user of that service is essential.
Having good (easy-to-understand, encouraging efficient use, not error-prone, supporting testing, etc.) interfaces is probably the most important single aspect of code organization.

Interface rule summary:

* [I.1: Make interfaces explicit](#Ri-explicit)
* [I.2: Avoid global variables](#Ri-global)
* [I.3: Avoid singletons](#Ri-singleton)
* [I.4: Make interfaces precisely and strongly typed](#Ri-typed)
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

### <a name="Ri-explicit"></a> I.1: Make interfaces explicit

##### Reason

Correctness. Assumptions not stated in an interface are easily overlooked and hard to test.

##### Example, bad

Controlling the behavior of a function through a global (namespace scope) variable (a call mode) is implicit and potentially confusing. For example:

    int rnd(double d)
    {
        return (rnd_up) ? ceil(d) : d;    // don't: "invisible" dependency
    }

It will not be obvious to a caller that the meaning of two calls of `rnd(7.2)` might give different results.

**Exception**: Sometimes we control the details of a set of operations by an environment variable, e.g., normal vs. verbose output or debug vs. optimized.
The use of a non-local control is potentially confusing, but controls only implementation details of otherwise fixed semantics.

##### Example, bad

Reporting through non-local variables (e.g., `errno`) is easily ignored. For example:

    fprintf(connection, "logging: %d %d %d\n", x, y, s); // don't: no test of printf's return value

What if the connection goes down so that no logging output is produced? See Rule I.??.

**Alternative**: Throw an exception. An exception cannot be ignored.

**Alternative formulation**: Avoid passing information across an interface through non-local state.
Note that non-`const` member functions pass information to other member functions through their object's state.

**Alternative formulation**: An interface should be a function or a set of functions.
Functions can be template functions and sets of functions can be classes or class templates.

##### Enforcement

* (Simple) A function should not make control-flow decisions based on the values of variables declared at namespace scope.
* (Simple) A function should not write to variables declared at namespace scope.

### <a name="Ri-global"></a> I.2 Avoid global variables

##### Reason

Non-`const` global variables hide dependencies and make the dependencies subject to unpredictable changes.

##### Example

    struct Data {
        // ... lots of stuff ...
    } data;            // non-const data

    void compute()     // don't
    {
        // ... use data ...
    }

    void output()     // don't
    {
        // ... use data ...
    }

Who else might modify `data`?

##### Note

Global constants are useful.

##### Note

The rule against global variables applies to namespace scope variables as well.

**Alternative**: If you use global (more generally namespace scope data) to avoid copying, consider passing the data as an object by const reference.
Another solution is to define the data as the state of some object and the operations as member functions.

**Warning**: Beware of data races: If one thread can access nonlocal data (or data passed by reference) while another thread executes the callee, we can have a data race.
Every pointer or reference to mutable data is a potential data race.

##### Note

You cannot have a race condition on immutable data.

**Reference**: See the [rules for calling functions](#SS-call).

##### Enforcement

(Simple) Report all non-`const` variables declared at namespace scope.

### <a name="Ri-singleton"></a> I.3: 싱글턴 패턴을 피하라.
>### <a name="Ri-singleton"></a> I.3: Avoid singletons

##### Reason

싱글턴은 위장하고 있지만 기본적으로는 복잡한 전역 객체이다.
>Singletons are basically complicated global objects in disguise.

##### Example

    class Singleton {
        // ... lots of stuff to ensure that only one Singleton object is created,
        // that it is initialized properly, etc.
    };

싱글턴에 대한 아이디어는 다양하다. 그것이 문제의 한 부분이다.
>There are many variants of the singleton idea.
That's part of the problem.

##### Note

전역객체를 변경시키고 싶지 않다면 `const`, `constexpr`로 선언하라.
>If you don't want a global object to change, declare it `const` or `constexpr`.

##### Exception

사용되자마자 최기화가 이뤄지는 제일 단순한 "싱글턴"을 사용할 수 있다:
>You can use the simplest "singleton" (so simple that it is often not considered a singleton) to get initialization on first use, if any:

    X& myX()
    {
        static X my_x {3};
        return my_x;
    }

이 방식이 초기화 순서와 관련한 문제를 처리하기에 가장 효과적인 해결책이다.
다중쓰레드 환경에서도 정적객체의 초기화는 경쟁상황을 일으키지는 않는다.
(부주의하게 생성자내에서 공유된 객체를 접근하지 않는다면 말이다.)
>This is one of the most effective solutions to problems related to initialization order.
In a multi-threaded environment the initialization of the static object does not introduce a race condition
(unless you carelessly access a shared object from within its constructor).

한 객체만 생성해야 하는 클래스로 싱글턴을 정의한다면 `myX`같은 함수는 싱글턴이 아니다.
그리고 이 유용한 기술은 비싱글턴 규칙에 예외적이지 않다.
>If you, as many do, define a singleton as a class for which only one object is created, functions like `myX` are not singletons, and this useful technique is not an exception to the no-singleton rule.

##### Enforcement

실제로 엄청 어렵다.
>Very hard in general.

* `singleton`을 포함하는 이름을 가진 클래스를 찾아본다.
* 객체를 헤아리거나 생성자를 조사해 보면서 한개의 객체가 생성되는 클래스를 찾아본다.
>* Look for classes with names that include `singleton`.
>* Look for classes for which only a single object is created (by counting objects or by examining constructors).

### <a name="Ri-typed"></a> I.4: 인터페이스를 정확하게 그리고 강하게 타입화해라.
>### <a name="Ri-typed"></a> I.4: Make interfaces precisely and strongly typed

##### Reason

타입은 가장 단순하지만 최고의 문서이다. 잘 정의된 의미를 가지고 있고, 컴파일 타임에 체크할 수 있도록 보장한다.
또한 정확하게 타입된 코드는 더 잘 최적화 된다.
>Types are the simplest and best documentation, have well-defined meaning, and are guaranteed to be checked at compile time.
Also, precisely typed code is often optimized better.

##### Example, don't

보자:
>Consider:

    void pass(void* data);    // void* is suspicious

지금 호출받은 함수는 올바른 타입으로 사용하기 위해서 데이터 포인터를 형변환해야 한다.
그건 에러가 발생하기 쉽고, 길어져서 구질구질하다. 인터페이스에서 `void*`를 피하라.
대신에 베이스클래스에 대한 포인터나 다형타입 사용을 고려해라. (?)
>Now the callee has to cast the data pointer (back) to a correct type to use it. That is error-prone and often verbose.
Avoid `void*` in interfaces.
Consider using a variant or a pointer to base instead. (Future note: Consider a pointer to concept.)

**Alternative**: 종종 템플릿 매개변수는 `void*`를 제거하고 `T*`나 비슷한 것으로 변환시켜 버린다.
>**Alternative**: Often, a template parameter can eliminate the `void*` turning it into a `T*` or something like that.

##### Example, bad

보자:
>Consider:

    void draw_rect(int, int, int, int);   // great opportunities for mistakes

    draw_rect(p.x, p.y, 10, 20);          // what does 10, 20 mean?

`int`는 정보를 임의대로 제공해서, 4개의 `int`가 어떤 의미인지를 유추해야만 한다.
아마도 첫 2개는 `x`, `y` 좌표일 것같지만, 나머지 두개는 머지?
주석이나 매개변수 이름이 도움을 주기도 하지만 명확히 한다면:
>An `int` can carry arbitrary forms of information, so we must guess about the meaning of the four `int`s.
Most likely, the first two are an `x`,`y` coordinate pair, but what are the last two?
Comments and parameter names can help, but we could be explicit:

    void draw_rectangle(Point top_left, Point bottom_right);
    void draw_rectangle(Point top_left, Size height_width);

    draw_rectangle(p, Point{10, 20});  // two corners
    draw_rectangle(p, Size{10, 20});   // one corner and a (height, width) pair

분명히 정적타입시스템 상에서도 모든 에러를 잡아낼 수는 없다.
(첫번째 인자가 좌-상단점이라는 사실은 이름이나 주석 등으로 컨벤션되었다.)
>Obviously, we cannot catch all errors through the static type system
(e.g., the fact that a first argument is supposed to be a top-left point is left to convention (naming and comments)).

##### Example, bad

다음 예제, `time_to_blink`이 무슨 의미일지 인터페이스를 보고는 모르겠다. 초, 밀리초?
>In the following example, it is not clear from the interface what `time_to_blink` means: Seconds? Milliseconds?

    void blink_led(int time_to_blink) // bad - the unit is ambiguous
    {
        // ...
        // do something with time_to_blink
        // ...
    }

    void use()
    {
        blink_led(2);
    }

##### Example, good

`std::chrono::duration` 타입은 C++11에서 도입되었는데 지속시간의 단위를 분명하게 만드는데 도움이 된다.
>`std::chrono::duration` types introduced in C++11 helps making the unit of time duration explicit.

    void blink_led(milliseconds time_to_blink) // good - the unit is explicit
    {
        // ...
        // do something with time_to_blink
        // ...
    }

    void use()
    {
        blink_led(1500ms);
    }

함수는 어떤 종류의 경과시간 단위도 허용하도록 아래와 같이 교체 쓸 수도 있다.
>The function can also be written in such a way that it will accept any time duration unit.

    template<class rep, class period>
    void blink_led(duration<rep, period> time_to_blink) // good - accepts any unit
    {
        // assuming that millisecond is the smallest relevant unit
        auto milliseconds_to_blink = duration_cast<milliseconds>(time_to_blink);
        // ...
        // do something with milliseconds_to_blink
        // ...
    }

    void use()
    {
        blink_led(2s);
        blink_led(1500ms);
    }

##### Enforcement

* (Simple) `void*`를 반환타입이나 매개변수로 사용한다면 보고한다.
* (Hard to do well) 다수의 내장 타입 인자를 가진 멤버함수를 찾는다.

>* (Simple) Report the use of `void*` as a parameter or return type.
>* (Hard to do well) Look for member functions with many built-in type arguments.

### <a name="Ri-pre"></a> I.5: 있다면 선행조건을 기술하라.
>### <a name="Ri-pre"></a> I.5: State preconditions (if any)

##### Reason

인자는 호출받는 함수 내에서 적절하게 사용하게 만드는 의도를 가진다.
>Arguments have meaning that may constrain their proper use in the callee.

##### Example

보자:
>Consider:

    double sqrt(double x);

여기서 `x`는 반드시 양수여야 한다. 타입 시스템으로는 표현할 수 없고, 그래서 다른 수단을 사용해야 한다.
예를 들면:
>Here `x` must be nonnegative. The type system cannot (easily and naturally) express that, so we must use other means. For example:

    double sqrt(double x); // x must be nonnegative

선행조건은 assertion(?)으로 표현하기도 한다. 예를 들면:
>Some preconditions can be expressed as assertions. For example:

    double sqrt(double x) { Expects(x >= 0); /* ... */ }

`Expects(x >= 0)` 조건이 `sqrt()`의 인터페이스에 포함되는 것이 가장 이상적이다. 그렇게 하기는 쉽지 않으니 당분간 함수 정의부 내에 위치시킨다.
>Ideally, that `Expects(x >= 0)` should be part of the interface of `sqrt()` but that's not easily done. For now, we place it in the definition (function body).

**Reference**: `Expects()`는 [GSL](#S-gsl)에 기술되어 있다.
>**Reference**: `Expects()` is described in [GSL](#S-gsl).

##### Note

요구사항의 공식적인 스팩을 선호하라. `Excepts(p != nullptr);`.
infeasible하다면, 주석문에 작성하라. 다음과 같이.
`// the sequence [p:q) is ordered using <`.
>Prefer a formal specification of requirements, such as `Expects(p != nullptr);`. If that is infeasible, use English text in comments, such as
`// the sequence [p:q) is ordered using <`.

##### Note

대부분의 멤버 함수는 클래스 불변조건(?)에 해당하는 선행조건을 갖고 있다.
그 불변조건은 생성자에서 구성되는데 클래스 외부로부터 호출되는 모든 멤버 함수에 의해 재구성되어야 한다.
그래서 함수마다 개별적으로 언급할 필요는 없다.
>Most member functions have as a precondition that some class invariant holds.
That invariant is established by a constructor and must be reestablished upon exit by every member function called from outside the class.
We don't need to mention it for each member function.

##### Enforcement

(Not enforceable)

**See also**: 포인터 인자전달 규칙을 참조해라. ???
>**See also**: The rules for passing pointers. ???

### <a name="Ri-expects"></a> I.6: 선행조건을 표현하고 싶다면 `Expects()`를 선호하라.
>### <a name="Ri-expects"></a> I.6: Prefer `Expects()` for expressing preconditions

##### Reason

선행조건임을 표시하고 툴 사용을 쉽고 명확하게 만들기 위해.
>To make it clear that the condition is a precondition and to enable tool use.

##### Example

    int area(int height, int width)
    {
        Expects(height > 0 && width > 0);            // good
        if (height <= 0 || width <= 0) my_error();   // obscure
        // ...
    }

##### Note

선행조건은 `if`문, `assert()`문, 주석문 등으로 기술할 수 있다.
이런 구문은 일반 코드와 구분하기 어렵고, 업데이트하기 어렵고, 툴로 조작하기 어렵고, 잘못된 의미를 가질 수도 있다.
(디버그 모드일 때는 중단시키고, 제품 모드일때는 체크하고 싶은가?)
>Preconditions can be stated in many ways, including comments, `if`-statements, and `assert()`. This can make them hard to distinguish from ordinary code, hard to update, hard to manipulate by tools, and may have the wrong semantics (do you always want to abort in debug mode and check nothing in productions runs?).

##### Note

선행조건은 구현부보다는 인터페이스부에 포함시켜야 한다. 아직은 그렇게 할 수 있는 언어기능이 없다.
>Preconditions should be part of the interface rather than part of the implementation, but we don't yet have the language facilities to do that.

##### Note

`Expects()`은 알고리즘 중간에 조건체크에도 사용할 수 있다.
>`Expects()` can also be used to check a condition in the middle of an algorithm.

##### Enforcement

(Not enforceable) 선행조건이 잘못될 다양한 조건을 찾는 것은 feasible하지 않다.
`assert()`로 쉽게 구별될 수 있을 때라도 경고한다면, 언어에 그 기능이 없다 하더라도 가치가 있을지 의심스럽다.
>(Not enforceable) Finding the variety of ways preconditions can be asserted is not feasible. Warning about those that can be easily identified (`assert()`) has questionable value in the absence of a language facility.

### <a name="Ri-post"></a> I.7: 후행조건을 기술하라.
>### <a name="Ri-post"></a> I.7: State postconditions

##### Reason

결과에 대한 잘못된 이해를 찾기 위해, 잘못된 구현을 찾아내기 위해.
>To detect misunderstandings about the result and possibly catch erroneous implementations.

##### Example, bad

보면
>Consider

	int area(int height, int width) { return height*width; }	// bad

여기보면 부주의하게 선행조건 명세를 뻬먹었는데 높이와 폭이 양수가 된다는 조건이 빠져있다.
역시 후행조건 명세를 뻬먹었는데 넓이를 구하는 알고리즘이 integer 범위를 벗어날 수 있다는 조건이 빠져 있다. 오버플로가 발생할 수 있다.
이렇게 해보자:
>Here, we (incautiously) left out the precondition specification, so it is not explicit that height and width must be positive.
We also left out the postcondition specification, so it is not obvious that the algorithm (`height*width`) is wrong for areas larger than the largest integer.
Overflow can happen.
Consider using:

	int area(int height, int width)
	{
		auto res = height*width;
		Ensures(res>0);
		return res;
	}

##### Example, bad

악명높은 보안 버그를 보자
>Consider a famous security bug

	void f()	// problematic
	{
		char buffer[MAX];
		// ...
		memset(buffer,0,MAX);
	}

버퍼가 초기화되고 중복 `memset()` 호출을 줄여서 최적화했다는 후행조건이 없다:
>There was no postcondition stating that the buffer should be cleared and the optimizer eliminated the apparently redundant `memset()` call:

	void f()	// better
	{
		char buffer[MAX];
		// ...
		memset(buffer,0,MAX);
		Ensures(buffer[0]==0);
	}

##### Note

후행조건은 함수의 목적을 기술하는 주석문에서 대충 언급된다. `Ensures()`를 사용함으로써 시스템적으로 체크할 수 있도록 보일 것이다.
>postconditions are often informally stated in a comment that states the purpose of a function; `Ensures()` can be used to make this more systematic, visible, and checkable.

##### Note

구조체 상태처럼 반환된 결과값 속에 간접적으로 영향을 미치는 것에 대한 후행조건이 특히 중요하다.
>Postconditions are especially important when they relate to something that is not directly reflected in a returned result, such as a state of a data structure used.

##### Example

경쟁상태을 피할 목적으로 `mutex`를 사용하는 `Record` 조작 함수를 생각해보자:
>Consider a function that manipulates a `Record`, using a `mutex` to avoid race conditions:

	mutex m;

	void manipulate(Record& r)	// don't
	{
		m.lock();
		// ... no m.unlock() ...
	}

여기서 `mutex`를 해제해야 한다는 언급을 잊어버렸는데, `mutex` 해제를 언급하지 않은 것이 단순 버그인지 의도인지 알 수가 없다.
후행조건을 기술함으로써 코드를 분명하게 만든다:
>Here, we "forgot" to state that the `mutex` should be released, so we don't know if the failure to ensure release of the `mutex` was a bug or a feature. Stating the postcondition would have made it clear:

	void manipulate(Record& r)	// better: hold the mutex m while and only while manipulating r
	{
		m.lock();
		// ... no m.unlock() ...
	}

이제 버그가 분명하게 보인다.
>The bug is now obvious.

락 해제에 대한 후행조건을 기술하기보다는 [RAII](#Rc-raii)를 사용해라:
>Better still, use [RAII](#Rc-raii) to ensure that the postcondition ("the lock must be released") is enforced in code:

	void manipulate(Record& r)	// best
	{
		lock_guard _ {m};
		// ...
	}

##### Note

이상적으로, 인터페이스/선언부에 사용자들이 쉽게 볼 수 있도록 후행조건을 기술해야 한다.
사용자들에게 언급되어야 하는 후행조건만 인터페이스에 언급하는데 그친다. 내부 상태에 대한 후행조건은 정의/구현부에만 포함된다.
>Ideally, postconditions are stated in the interface/declaration so that users can easily see them.
Only postconditions related to the users can be stated in the interface.
Postconditions related only to internal state belongs in the definition/implementation.

##### Enforcement

(Not enforceable) 체크하기 어려운 개념적인 가이드라인이다.
>(Not enforceable) This is a philosophical guideline that is infeasible to check directly.

### <a name="Ri-ensures"></a> I.8: 후행조건을 표현할때 `Ensures()`를 사용해라.
>### <a name="Ri-ensures"></a> I.8: Prefer `Ensures()` for expressing postconditions

##### Reason

후행조건이라는 것을 분명히 하기 위해, 또 분석툴을 사용하기 위해.
>To make it clear that the condition is a postcondition and to enable tool use.

##### Example

	void f()
	{
		char buffer[MAX];
		// ...
		memset(buffer,0,MAX);
		Ensures(buffer[0]==0);
	}

##### Note

선행조건은 주석문, `if`문, `assert()`문 등으로 다양하게 언급되었다. 이런 여러 방식때문에 일반적인 코드와 구분이 어렵고 업데이트하기 어렵고 툴로 조작하기 어렵고 틀린 의미를 가지고 있을 수도 있다.
>preconditions can be stated in many ways, including comments, `if`-statements, and `assert()`. This can make them hard to distinguish from ordinary code, hard to update, hard to manipulate by tools, and may have the wrong semantics.

**Alternative**: "이 자원은 반드시 해제되어야 한다" 형태의 후행조건은 [RAII](#Rc-raii)에 잘 정의되어 있다.
>**Alternative**: Postconditions of the form "this resource must be released" and best expressed by [RAII](#Rc-raii).

이상적으로 `Ensured`는 인터페이스의 일부가 되어야 한다. 지금으로서는 함수 정의에 위치시킨다.(함수 내부)
>Ideally, that `Ensured` should be part of the interface that's not easily done. For now, we place it in the definition (function body).

##### Enforcement

(Not enforceable) 다양한 방식의 후행조건은 가능하지 않다. 언어명세에 포함되지 않은 채로 대충 정의(assert())한 후행조건에 대해서 경고하기는 거의 불가능하다.
>(Not enforceable) Finding the variety of ways postconditions can be asserted is not feasible. Warning about those that can be easily identfied (assert()) has questionable value in the absence of a language facility.

### <a name="Ri-concepts"></a> I.9: 인터페이스가 템플릿이라면 concept을 사용해서 매개변수를 문서화해라.
>### <a name="Ri-concepts"></a> I.9: If an interface is a template, document its parameters using concepts

##### Reason
가까운 미래에 컴파일 타임 체크를 할 수 있도록 인터페이스를 정확하게 정의해라.
>Make the interface precisely specified and compile-time checkable in the (not so distant) future.

##### Example

TS 스타일 요구사항 명세에 ISO concept를 사용하라. 예제:
>Use the ISO Concepts TS style of requirements specification. For example:

	template<typename Iter, typename Val>
	//	requires InputIterator<Iter> && EqualityComparable<ValueType<Iter>>,Val>
	Iter find(Iter first, Iter last, Val v)
	{
		// ...
	}

##### Note

곧(2016년에) 대부분의 컴파일러가 `//`가 제거된 `requires` 구문을 체크할 수 있을 예정이다.
>Soon (maybe in 2016), most compilers will be able to check `requires` clauses once the `//` is removed.

**See also**: See [generic programming](???) and [???](???)

##### Enforcement

(Not enforceable yet) 언어명세를 작성중이다. 언어명세가 정의된다면, 비가변 템플릿 매개변수가 concept으로 제한되지 않는다면 경고하라. (선언이나 `requires` 구문에 언급되지 않은)
>(Not enforceable yet) A language facility is under specification. When the language facility is available, warn if any non-variadic template parameter is not constrained by a concept (in its declaration or mentioned in a `requires` clause.


### <a name="Ri-except"></a> I.10: 테스크 실행 실패를 알리려면 예외를 사용하라.
### <a name="Ri-except"></a> I.10: Use exceptions to signal a failure to perform a required task

##### Reason

시스템이나 계산결과를 예측불가능한 상태로 둘 수 없으므로 에러를 무시해서는 안된다. 이것이 에러의 주요 발생지가 된다.
>It should not be possible to ignore an error because that could leave the system or a computation in an undefined (or unexpected) state.
This is a major source of errors.

##### Example

	int printf(const char* ...);	// bad: return negative number if output fails

	template <class F, class ...Args>
	explicit thread(F&& f, Args&&... args);	// good: throw system_error if unable to start the new thread

##### Note: What is an error?

에러는 기능이 의도한 목적을 이룰 수 없는 것을 의미한다.(후행조건을 만족시키면서)
에러를 무시하는 코드를 호출하면 정의되지 않는 시스템 상태나 잘못된 결과를 야기할 수 있다.
예를 들어, 리모트 서버와 연결이 안된다면 에러는 아니다. 서버는 다양한 이유로 연결을 거부할 수 있다.
호출하는 쪽에서 결과를 체크할 수 있도록 반환해 주는 것이 자연스럽다. 그러나 연결에 실패하는 것을 에러로 간주할 생각이라면 예외를 던져주어야 한다.
>An error means that the function cannot achieve its advertised purpose (including establishing postconditions).
Calling code that ignores the error could lead to wrong results or undefined systems state.
For example, not being able to connect to a remote server is not by itself an error:
the server can refuse a connection for all kinds of reasons, so the natural thing is to return a result that the caller always has to check.
However, if failing to make a connection is considerd an error, then a failure should throw an exception.

**Exception**: 많은 예전 인터페이스 함수는 실제로는 상태값인 에러코드를 사용한다(예를 들면 `errno`). 별다른 대안이 없으므로 이 규칙에 위배되지는 않는다.
>**Exception**: Many traditional interface functions (e.g., UNIX signal handlers) use error codes (e.g., `errno`) to report what are really status codes, rather than errors. You don't have good alternative to using such, so calling these does not violate the rule.

**Alternative**: 예외를 사용할 수 없다면(코드가 예전 스타일의 포인터로 가득하던지, 실시간 제약이 심각하서), 벨류쌍를 반환하는 스타일을 사용하는 것을 생각해 보라.
>**Alternative**: If you can't use exceptions (e.g. because your code is full of old-style raw-pointer use or because there are hard-real-time constraints),
consider using a style that returns a pair of values:

	int val;
	int error_code;
	tie(val,error_code) do_something();
	if (error_code==0) {
		// ... handle the error or exit ...
	}
	// ... use val ...

##### Note

성능이슈가 예외를 사용하지 않을 타당한 이유라고는 생각지 않는다.
>We don't consider "performance" a valid reason not to use exceptions.

* 명시적인 에러 체크와 관리가 예외처리만큼 시간과 공간을 허비하지는 않는다.
* 간결한 코드는 예외처리를 포함해도 더 좋은 성능을 가진다. (프로그램 최적화를 통해 실행경로를 단순화시킨다).
* 성능향상를 위한 좋은 방법은 핵심코드는 핵심부분 밖에서 체크하도록 옮기는 것이다. ([checking](#Rper-checking)).
* 장기적으로 봤을 때 정규화된 코드가 최적화가 잘 된다.

>* Often, explicit error checking and handling consume as much time and space as exception handling.
* Often, cleaner code yield better performance with exceptions (simplifying the tracing of paths through the program and their optimization).
* A good rule for performance critical code is to move checking outside the critical part of the code ([checking](#Rper-checking)).
* In the longer term, more regular code gets better optimized.

**See also**:  선행조건, 후행조건 위반를 보고하기 위해 Rule I.??? and I.???.
>**See also**: Rule I.??? and I.??? for reporting precondition and postcondition violations.

##### Enforcement

* (Not enforceable) 체크하기 힘든 철학적인 가이드라인이다.
* `errno`를 살펴봐라.

>* (Not enforceable) This is a philosophical guideline that is infeasible to check directly.
* look for `errno`.

### <a name="Ri-raw"></a> I.11: 일반포인터로 소유권을 넘기지 마라.(`T*`)
>### <a name="Ri-raw"></a> I.11: Never transfer ownership by a raw pointer (`T*`)

##### Reason

호출하는 쪽이나 받는 쪽이 객체를 누가 소유할지 모른다면 메모리 누수나 불충분한 소멸이 일어나기 때문이다.
>If there is any doubt whether the caller or the callee owns an object, leaks or premature destruction will occur.

##### Example

Consider

	X* compute(args)	// don't
	{
		X* res = new X{};
		// ...
		return res;
	}


	vector<double> compute(args)	// good
	{
		vector<double> res(10000);
		// ...
		return res;
	}

##### Example

Consider:

    X* compute(args)    // don't
    {
        X* res = new X{};
        // ...
        return res;
    }

반환된 X를 누가 삭제해야 하나? 만약에 참조를 반환한다면 문제는 더 어려워질 것이다.
값으로 반환된 결과를 살펴보자.(결과 사이즈가 크다면 이동 개념을 사용하라.)
>Who deletes the returned `X`? The problem would be harder to spot if compute returned a reference.
Consider returning the result by value (use move semantics if the result is large):

    vector<double> compute(args)  // good
    {
        vector<double> res(10000);
        // ...
        return res;
    }

**Alternative**: 'smart pointer'를 사용해서 소유권을 넘겨라. `unique_ptr`(독점적인 소유를 위해) 또는 `shared_ptr`(소유권 공유를 위해)
그러나 참조 개념이 필요하지 않다면 효과는 제한적이고 우아히자는 않을 것이다.
>**Alternative**: Pass ownership using a "smart pointer", such as `unique_ptr` (for exclusive ownership) and `shared_ptr` (for shared ownership).
However that is less elegant and less efficient unless reference semantics are needed.

**Alternative**: ABI 호환 요구 또는 자원 부족때문에 예전 코드를 수정할 수 없는 상황이 있다.
그 경우에 `owner`를 사용해서 포인터의 소유권을 표시해둬라.
>**Alternative**: Sometimes older code can't be modified because of ABI compatibility requirements or lack of resources.
In that case, mark owning pointers using `owner` :

	owner<X*> compute(args)		// It is now clear that ownership is transferred
	{
		owner<X*> res = new X{};
		// ...
		return res;
	}

이것은 분석툴에게 `res`가 오너라고 알려준다.
즉, `return`하는 것처럼 그 값에 대해서 삭제하거나 다른 오너로 넘겨주어야 한다.
>This tells analysis tools that `res` is an owner.
That is, it's value must be `delete`d or transferred to another owner, as is done here by the `return`.

`owner`는 자원 관리하는 구현과 비슷하게 사용된다.
>`owner` is used similarly in the implementation of resource handles.

`owner`는 [Guideline Support Library](#S-GSL)에 정의했다.
> `owner` is defined in the [Guideline Support Library](#S-GSL).

##### Note

일반포인터(또는 순환자)로 넘긴 객체는 호출하는 쪽에서 소유해야 한다고 가정한다. 객체 생존시간은 호출하는 쪽에서 관리할 수 있도록 말이다.
>Every object passed as a raw pointer (or iterator) is assumed to be owned by the caller, so that its lifetime is handled by the caller.

**See also**: [Argument passing](#Rf-conventional) and [value return](#Rf-T-return).

##### Enforcement

* (Simple) `owner`가 아닌 일반포인터의 `delete`에 대해서는 경고하라.
* (Simple) 모든 실행경로 상에서 리셋이나 `owner` 포인터를 명시적으로 삭제할 때 실패하면 경고하라.
* (Simple) `new`의 결과값 또는 함수호출로 반환되는 포인터가 일반포인터라면 경고하라.

>* (Simple) Warn on `delete` of a raw pointer that is not an `owner`.
>* (Simple) Warn on failure to either `reset` or explicitly `delete` an `owner` pointer on every code path.
>* (Simple) Warn if the return value of `new` or a function call with return value of pointer type is assigned to a raw pointer.

### <a name="Ri-nullptr"></a> I.12: 포인터가 NULL값이 될 수 없다면 `not_null`로 선언하라.
>### <a name="Ri-nullptr"></a> I.12: Declare a pointer that must not be null as `not_null`

##### Reason  

`nullptr` 역참조(Null Pointer Dereference)에러를 피하기 위해, `nullptr` 반복 체크를 줄여서 성능을 높이기 위해.
>To help avoid dereferencing `nullptr` errors. To improve performance by avoiding redundant checks for `nullptr`.

##### Example

	int length(const char* p);		// it is not clear whether strlen(nullptr) is valid

	length(nullptr);				// OK?

	int length(not_null<const char*> p);		// better: we can assume that p cannot be nullptr

	int length(const char* p);				// we must assume that p can be nullptr

소스 안에 의도를 추가해 줌으로써 컴파일러와 툴의 분석 결과가 좋아진다. 정적(컴파일타임) 분석을 통해 클래스의 에러를 발견하는 것, 최적화 수행하기, 분기 줄이기, NULL 체크 말이다.
>By stating the intent in source, implementers and tools can provide better diagnostics, such as finding some classes of errors through static analysis, and perform optimizations, such as removing branches and null tests.

##### Note

c 스타일의 스트링(NULL로 끝나는 문자열)에 대한 포인터를 char형이라고 여전히 가정하고 있으며 잠재적인 에러 원인이 된다. `const char *` 대신에 `zstring`을 사용하라.
>The assumption that the pointer to `char` pointed to a C-style string (a zero-terminated string of characters) was still implicit, and a potential source of confusion and errors. Use `zstring` in preference to `const char*`.

    int length(not_null<zstring> p);   // we can assume that p cannot be nullptr
                                       // we can assume that p points to a zero-terminated array of characters

Note: `length()`는 당연히 `std::strlen()`이다.
>Note: `length()` is, of course, `std::strlen()` in disguise.

##### Enforcement

* (Simple) ((Foundation)) 함수내에서 값을 접근하기 전에 `nullptr` 체크를 한다면 `not_null`로 선언해야 한다고 경고하라.
* (Complex) 포인터를 반환하는 함수가 `nullptr`가 아닌 값을 반환하고 있다면 반환값을 `not_null`로 선언해야 한다고 경고하라.

>* (Simple) ((Foundation)) If a function checks a pointer parameter against `nullptr` before access, on all control-flow paths, then warn it should be declared `not_null`.
* (Complex) If a function with pointer return value ensures it is not `nullptr` on all return paths, then warn the return type should be declared `not_null`.

### <a name="Ri-array"></a> I.13: 포인터로 배열 인자를 넘기지 마라.
>### <a name="Ri-array"></a> I.13: Do not pass an array as a single pointer

##### Reason

(포인터, 사이즈) 형태의 함수 인터페이스는 에러날 확률이 높다. 배열에 대한 포인터는 반드시 사이즈를 판단할 수 있는 방식이 필요하다.  
>(pointer,size)-style interfaces are error-prone. Also, plain pointer (to array) must relies on some convention to allow the callee to determine the size.

##### Example

Consider:

	void copy_n(const T* p, T* q, int n); // copy from [p:p+n) to [q:q+n)

'q'의 사이즈가 n보다 적다면 어떻게 될까? 관계없는 메모리에 쓰기를 시도할 것이다.
'p'의 사이즈가 n보다 작다면 관계없는 메모리로부터 읽기를 시도할 것이다.
어느 쪽이나 미정의 동작을 하게 되고 잠재적인 고약한 버그가 생긴다.
>What if there are fewer than `n` elements in the array pointed to by `q`? Then, we overwrite some probably unrelated memory.
What if there are fewer than `n` elements in the array pointed to by `p`? Then, we read some probably unrelated memory.
Either is undefined behavior and a potentially very nasty bug.

##### Alternative

명시적 범위를 사용하는 걸 고려해봐라.
>Consider using explicit ranges,

	void copy(array_view<const T> r, array_view<T> r2); // copy r to r2

##### Example, bad

Consider:

	void draw(Shape* p, int n);	// poor interface; poor code
	Circle arr[10];
	// ...
	draw(arr,10);

10을 n에 인자로 넘기면 에러가 날 수 있다.: 보통 방법은 [0:n)로 가정하는 것이지만 어디에도 설명되지 않고 있다. 더 나쁜 부분은 'draw()' 호출이다. 배열을 포인터로 암시적으로 형변환했고 Circle을 Shape로 묵시적으로 형변환했다.
'draw()'함수가 배열을 안전하게 순환할 수 있을지 잘 모르겠다. 배열 원소 사이즈를 알 방법이 없기 때문이다.
>Passing `10` as the `n` argument may be a mistake: the most common convention is to assume [`0`:`n`) but that is nowhere stated. Worse is that the call of `draw()` compiled at all: there was an implicit conversion from array to pointer (array decay) and then another implicit conversion from `Circle` to `Shape`. There is no way that `draw()` can safely iterate through that array: it has no way of knowing the size of the elements.

**Alternative**: 배열 원소의 수가 정확하고 암시적인 형변환을 방어할 수 있는 지원 클래스를 사용하라. 예를 들면,
>**Alternative**: Use a support class that ensures that the number of elements is correct and prevents dangerous implicit conversions. For example:

	void draw2(array_view<Circle>);
	Circle arr[10];
	// ...
	draw2(array_view<Circle>(arr));	// deduce the number of elements
	draw2(arr);	// deduce the element type and array size

	void draw3(array_view<Shape>);
	draw3(arr);	// error: cannot convert Circle[10] to array_view<Shape>

'draw2()'는  'draw()'와 동일한 정보를 인자로 넘긴다. 그러나 사실은 명시적으로 'Circle'의 범위라는 의도이다.
>This `draw2()` passes the same amount of information to `draw()`, but makes the fact that it is supposed to be a range of `Circle`s explicit. See ???.

**Exception**: C 스타일 NULL로 끝나는 스트링을 표시할 때는 `zstring`, `czstring`를 사용해라. But see ???.
>**Exception**: Use `zstring` and `czstring` to represent a C-style, zero-terminated strings. But see ???.

##### Enforcement

* (Simple) ((Bounds)) 조용히 배열을 포인터로 형변환하려는 표현식에 대해서 경고하라. zstring/czstring은 제외.
* (Simple) ((Bounds)) 주소값을 계산하기 위해 포인터에 대해서 연산하는 계산식에 대해서 경고하라.  

>* (Simple) ((Bounds)) Warn for any expression that would rely on implicit conversion of an array type to a pointer type. Allow exception for zstring/czstring pointer types.
* (Simple) ((Bounds)) Warn for any arithmetic operation on an expression of pointer type that results in a value of pointer type. Allow exception for zstring/czstring pointer types.

### <a name="Ri-nargs"></a> I.14: 함수인자수를 최소로 유지하라.
>### <a name="Ri-nargs"></a> I.14: Keep the number of function arguments low

##### Reason

인자수가 많으면 혼란을 일으킨다. 많은 인자를 패싱하는 것은 종종 비용적으로 대안과 비교된다.
>Having many arguments opens opportunities for confusion. Passing lots of arguments is often costly compared to alternatives.

##### Example

표준 라이브러리 `merge()` 편하게 다룰 수 있는 한계점 정도이다.
>The standard-library `merge()` is at the limit of what we can comfortably handle

    template<class InputIterator1, class InputIterator2, class OutputIterator, class Compare>
    OutputIterator merge(InputIterator1 first1, InputIterator1 last1,
                         InputIterator2 first2, InputIterator2 last2,
                         OutputIterator result, Compare comp);

여기보면 3개의 템플릿 인자와 5개의 함수인자를 가진다.
많이 쓰는 방식으로 단순하게 생각해보면 비교 함수는 '보다 작다'가 기본값이므로 생략한다.
>Here, we have four template arguments and six function arguments.
To simplify the most frequent and simplest uses, the comparison argument can be defaulted to `<`:

    template<class InputIterator1, class InputIterator2, class OutputIterator>
    OutputIterator merge(InputIterator1 first1, InputIterator1 last1,
                         InputIterator2 first2, InputIterator2 last2,
                         OutputIterator result);

이렇게 한다고해서 전체 복잡성이 줄어 들지 않지만 사용자 입장에서 보기에는 복잡성이 줄어든 것처럼 보인다.
실제로 인자수를 줄이기 위해서는 인자를 좀더 추상적으로 묶을 필요가 있다.
>This doesn't reduce the total complexity, but it reduces the surface complexity presented to many users.
To really reduce the number of arguments, we need to bundle the arguments into higher-level abstractions:

    template<class InputRange1, class InputRange2, class OutputIterator>
    OutputIterator merge(InputRange1 r1, InputRange2 r2, OutputIterator result);

인자를 묶어서 그룹핑하는 것이 인자수를 줄이는 일반적인 방법이고 체크(?)할 기회를 늘려준다.
>Grouping arguments into "bundles" is a general technique to reduce the number of arguments and to increase the opportunities for checking.

##### Note

인자는 네개면 많다.
4개 인자를 가진 잘 정의된 함수들이 있지만 많지 않다.
>How many arguments are too many? Four arguments is a lot.
There are functions that are best expressed with four individual arguments, but not many.

**Alternative**: 인자를 의미있는 오브젝트로 그룹핑해서 패싱을 하라.(값으로 넘기던지, 참조로 넘기던지)
>**Alternative**: Group arguments into meaningful objects and pass the objects (by value or by reference).

**Alternative**: 인자의 기본값을 사용하여 적은 인자로 호출가능하도록 제일 공통적인 형태를 가지도록 함수를 재정의하라.(생략가능한 인자를 적극 활용하라는 의미임.)
>**Alternative**: Use default arguments or overloads to allow the most common forms of calls to be done with fewer arguments.

##### Enforcement

* 범위 또는 뷰 이외에 동일 타입의 반복자(iterator)를 2개이상 선언하는 함수가 있다면 경고한다.
* (Not enforceable) 체크하기 어려운 철학적인 가이드라인이다.

>* Warn when a functions declares two iterators (including pointers) of the same type instead of a range or a view.
>* (Not enforceable) This is a philosophical guideline that is infeasible to check directly.

### <a name="Ri-unrelated"></a> I.15: 동일 타입이지만 관련없는 인접한 매개변수는 피하라.
>### <a name="Ri-unrelated"></a> I.15: Avoid adjacent unrelated parameters of the same type

##### Reason

동일 타입의 인접한 매개변수는 실수로 쉽게 바뀐다.
>Adjacent arguments of the same type are easily swapped by mistake.

##### Example, bad

Consider:

   void copy_n(T* p, T* q, int n);  // copy from [p:p+n) to [q:q+n)

&R C 스타일 인터페이스인데 "to"와 "from" 매개변수가 쉽게 뒤바뀐다.
>This is a nasty variant of a K&R C-style interface. It is easy to reverse the "to" and "from" arguments.

"from" 인자에 `const`를 사용해라.
>Use `const` for the "from" argument:

   void copy_n(const T* p, T* q, int n);  // copy from [p:p+n) to [q:q+n)

##### Alternative

포인터로 배열을 인자로 넘기지 말고 오브젝트로 인자를 넘겨라.(e.g., `array_view`):
>Don't pass arrays as pointers, pass an object representing a range (e.g., an `array_view`):

   void copy_n(array_view<const T> p, array_view<T> q);  // copy from p to q

##### Enforcement

(Simple) 연속된 2개 매개변수가 동일한 타입을 가진다면 경고하라.
>(Simple) Warn if two consecutive parameters share the same type.

### <a name="Ri-abstract"></a> I.16: 클래스 상속보다 인터페이스로 추상 클래스(abstract class)를 선호하라.
>### <a name="Ri-abstract"></a> I.16: Prefer abstract classes as interfaces to class hierarchies

##### Reason

추상 클래스는 상태가 있는 부모클래스보다 더 안정적이다.
>Abstract classes are more likely to be stable than base classes with state.

##### Example, bad

You just knew that `Shape` would turn up somewhere :-)

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

이 클래스는 상속받는 모든 자식 클래스에 센터를 계속하도록 강요할 것이다. -- center를 사용하지도 않을 거고 non-trivial한데도 불구하고 말이다. 모든 'Shape'가 'Color'를 가지지 않는다. 게다가 많은 'Shape'는 포인트의 열로 정의된 외각선이 없이도 표현된다. 추상클래스는 그런류의 클래스를 사용할 수 있도록 개발되었다.
>This will force every derived class to compute a center -- even if that's non-trivial and the center is never used. Similarly, not every `Shape` has a `Color`, and many `Shape`s are best represented without an outline defined as a sequence of `Point`s. Abstract classes were invented to discourage users from writing such classes:

	class Shape {	// better: Shape is a pure interface
	public:
		virtual Point center() =0;	// pure virtual function
		virtual void draw() =0;
		virtual void rotate(int) =0;
		// ...
		// ... no data members ...
	};

##### Enforcement

(Simple) 'C' 클래스 포인터가 부모클래스의 포인터를 가지고 있고 그 부모클래스가 데이터 멤버를 가진다면 경고하라.
>(Simple) Warn if a pointer to a class `C` is assigned to a pointer to a base of `C` and the base class contains data members.

### <a name="Ri-abi"></a> I.16: 컴파일러간 호환을 위한 바이너리 인터페이스(ABI: Application Binary Interface)을 원한다면 C 스타일을 사용하라.
>### <a name="Ri-abi"></a> I.16: If you want a cross-compiler ABI, use a C-style subset

##### Reason

컴파일러는 제각각 클래스의 바이너리 레이아웃, 예외 처리, 함수명, 상세 실행방식이 다르게 구현되어 있다.
>Different compilers implement different binary layouts for classes, exception handling, function names, and other implementation details.

**Exception**: 상위레벨 C++ 타입만 사용해서 신경써서 인터페이스를 만들어야 한다. See ???.
>**Exception**: You can carefully craft an interface using a few carefully selected higher-level C++ types. See ???.

**Exception**: ~~ 몇몇 플랫폼 상에서 공통 ABI가 나타나고 있다.
>**Exception**: Common ABIs are emerging on some platforms freeing you from the more draconian restrictions.

##### Note

한종류만 사용한다면 C++ 기능을 풀로 사용할 수 있다는 것으로 새버전에 대해서도 재컴파일이 가능하는 의미이다.
>If you use a single compiler, you can use full C++ in interfaces. That may require recompilation after an upgrade to a new compiler version.

##### Enforcement

(Not enforceable) 어느 인터페이스가 ABI를 만족할지 신뢰할 정도로 구분해 낼 수 없다.
>(Not enforceable) It is difficult to reliably identify where an interface forms part of an ABI.
