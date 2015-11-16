# C: 클래스와 클래스 계층 구조

># C: Classes and Class Hierarchies

클래스는 사용자 정의 타입으로써, 프로그래머가 표현과 동작, 인터페이스를 정의할 수 있다.
클래스 계층 구조는 관련된 클래스들을 계층 구조로 구조화 할 때 사용된다.

> A class is a user-defined type, for which a programmer can define the representation, operations, and  interfaces.
> Class hierarchies are used to organize related classes into hierarchical structures.

클래스 규칙 요약:

> Class rule summary:

* [C.1: 관련된 데이터를 구조를 사용해서 조직화 하라 (`struct`s or `class`es)](#Rc-org)
* [C.2: 클래스가 invariant 하다면 `class` 를 사용하고; 데이터 멤버가 독립적으로 달라질 수 있으면 `struct` 를 사용하라](#Rc-struct)
* [C.3: 클래스를 사용할 때 인터페이스 인지 구현인지 분명하게 표현하라](#Rc-interface)
* [C.4: 클래스의 표현에 직접 접근 할 필요가 있는 경우만 함수를 멤버로 만들어라](#Rc-member)
* [C.5: 헬퍼 함수들은 그들이 도와주는 클래스와 같은 네임스페이스에 두어라](#Rc-member)
* [C.6: 객체의 상태를 수정하지 않는 멤버 함수는 `const` 로 선언하라](#Rc-const)

> * [C.1: Organize related data into structures (`struct`s or `class`es)](#Rc-org)
> * [C.2: Use `class` if the class has an invariant; use `struct` if the data members can vary independently](#Rc-struct)
> * [C.3: Represent the distinction between an interface and an implementation using a class](#Rc-interface)
> * [C.4: Make a function a member only if it needs direct access to the representation of a class](#Rc-member)
> * [C.5: Place helper functions in the same namespace as the class they support](#Rc-member)
> * [C.6: Declare a member function that does not modify the state of its object `const`](#Rc-const)

하위 영역:

> Subsections:

* [C.concrete: 구체적인 타입](#SS-concrete)
* [C.ctor: 생성자, 할당, 소멸자](#SS-ctor)
* [C.con: 컨테이너와 다른 리소스 핸들](#SS-containers)
* [C.lambdas: 함수 객체와 람다](#SS-lambdas)
* [C.hier: 클래스 계층구조 (OOP)](SS-hier)
* [C.over: 오버로딩과 오버로딩된 연산자](#SS-overload)
* [C.union: 유니온](#SS-union)

> * [C.concrete: Concrete types](#SS-concrete)
> * [C.ctor: Constructors, assignments, and destructors](#SS-ctor)
> * [C.con: Containers and other resource handles](#SS-containers)
> * [C.lambdas: Function objects and lambdas](#SS-lambdas)
> * [C.hier: Class hierarchies (OOP)](SS-hier)
> * [C.over: Overloading and overloaded operators](#SS-overload)
> * [C.union: Unions](#SS-union)


<a name="Rc-org"></a>

### C.1: 관련된 데이터를 구조를 사용해서 조직화 하라 (`struct`s or `class`es)

> ### C.1: Organize related data into structures (`struct`s or `class`es)

**근거**: 이해하기 쉽다. 근본적인 이유로 데이터가 관련이 있다면, 코드에 반영되어야 한다.

> **Reason**: Ease of comprehension. If data is related (for fundamental reasons), that fact should be reflected in code.

**예**:

	void draw(int x, int y, int x2, int y2);	// BAD: unnecessary implicit relationships
	void draw(Point from, Point to)				// better

**참고 사항**: 가상 함수가 없는 간단한 클래스는 공간, 시간적인 오버헤드가 없다는 것을 의미한다.

> **Note**: A simple class without virtual functions implies no space or time overhead.

**참고 사항**: 언어적인 관점에서 볼 때 `class` 와 `struct` 는 멤버의 기본적인 가시성만 다르다.

> **Note**: From a language perspective `class` and `struct` differ only in the default visibility of their members.

**시행하기**: 아마도 불가능하다. 데이터 항목들에 대한 경험적인 관점을 함께 사용하는 것은 가능할 것이다.

> **Enforcement**: Probably impossible. Maybe a heuristic looking for date items used together is possible.


<a name="Rc-struct"></a>

### C.2: 클래스가 invariant 하다면 `class` 를 사용하고; 데이터 멤버가 독립적으로 달라질 수 있으면 `struct` 를 사용하라

> ### C.2: Use `class` if the class has an invariant; use `struct` if the data members can vary independently

**근거**: 이해하기 쉽다. 프로그래머가 `class` 를 사용함으로써, invariant 가 필요하다는 것을 알게 된다.

> **Reason**: Ease of comprehension. The use of `class` alerts the programmer to the need for an invariant

**참고 사항**: invariant 는 객체 멤버들의 논리적인 상태로써, 공개 멤버 함수들이 가정할 수 있도록 생성자가 설정 해 주어야 한다. invariant 가 설정된 후에 (일반적으로 생성자에 의해) 모든 멤버 함수는 객체를 통해 호출될 수 있다. invariant 는 형식에 구애받지 않고 (예. 주석) 기술될 수 있으며, 더 형식을 갖춘다면 `Expects` 를 사용한다.

> **Note**: An invariant is logical condition for the members of an object that a constructor must establish for the public member functions to assume. After the invariant is established (typically by a constructor) every member function can be called for the object. An invariant can be stated informally (e.g., in a comment) or more formally using `Expects`.

**예**:

	struct Pair {	// the members can vary independently
		string name;
		int volume;
	};

but

	class Date {
	private:
		int y;
		Month m;
		char d;		// day
	public:
		Date(int yy, Month mm, char dd);		// validate that {yy,mm,dd} is a valid date and initialize
		// ...
	};

**시행하기**: 모든 데이터가 비공개인 `struct` 와 모든 멤버가 공개인 `class` 들을 찾아보라.

> **Enforcement**: Look for `struct`s with all data private and `class`es with public members.


<a name="Rc-interface"></a>

### C.3: 클래스를 사용할 때 인터페이스 인지 구현인지 분명하게 표현하라

> ### C.3: Represent the distinction between an interface and an implementation using a class

**근거**: 인터페이스와 구현에 대한 명시적인 구분은 가독성을 더 좋게 하고, 유지 보수를 단순하게 한다.

> **Reason**: an explicit distinction between interface and implementation improves readability and simplifies maintenance.

**예**:

	class Date {
		// ... some representation ...
	public:
		Date();
		Date(int yy, Month mm, char dd);		// validate that {yy,mm,dd} is a valid date and initialize

		int day() const;
		Month month() const;
		// ...
	};

예를 들면, 이제 사용자에게 영향을 주지 않고 `Date` 에 대한 표현을 변경할 수 있다. (비록 다시 컴파일 해야 할지라도)

> For example, we can now change the representation of a `Date` without affecting its users (recompilation is likely, though).

**참고 사항**: 인터페이스와 구현간의 구분을 표현하기 위해 클래스를 사용하는 것이 유일한 방법은 아니다.
예를 들면, 인터페이스를 표현하기 위한 개념으로 네임스페이스 안에 독립적인 함수들이나 추상 기본 클래스 혹은 템플릿 함수들을 선언해서 사용할 수 있다.
가장 중요한 이슈는 명시적으로 인터페이스와 그것들의 "상세한" 구현을 구분하는 것이다.
이상적으로, 그리고 전형적으로 인터페이스는 그 구현보다 훨씬 더 안정적이다.

> **Note**: Using a class in this way to represent the distinction between interface and implementation is of course not the only way.
> For example, we can use a set of declarations of freestandanding functions in a namespace,
> an abstract base class,
> or a template fuction with concepts to represent an interface.
> The most important issue is to explicitly distinguish between an interface and its implementation "details."
> Ideally, and typically, an interface is far more stable than its implementation(s).

**시행하기**: ???


<a name="Rc-member"></a>

### C.4: 클래스의 표현에 직접 접근 할 필요가 있는 경우만 함수를 멤버로 만들어라

> ### C.4: Make a function a member only if it needs direct access to the representation of a class

**근거**: 멤버 함수간 커플링을 작게하고, 객체 상태 변경에 의해 문제가 생기는 함수를 줄이고, 표현이 변경된 후에 수정될 필요가 있는 멤버 함수의 수를 줄인다.

> **Reason**: Less coupling than with member functions, fewer functions that can cause trouble by modifying object state, reduces the number of functions that needs to be modified after a change in representation.

**예**:

	class Date {
		// ... relatively small interface ...
	};

	// helper functions:
	Date next_weekday(Date);
	bool operator==(Date, Date);

"헬퍼 함수"는 `Date` 의 표현에 직접 접근 할 필요가 없다.

> The "helper functions" have no need for direct access to the representation of a `Date`.

**참고 사항**: 이 규칙은 C++17 에서 "uniform function call" 이 들어오면 더 좋아질 것이다. ???

> **Note**: This rule becomes even better if C++17 gets "uniform function call." ???

**시행하기**: 데이터 멤버를 직접 건드리지 않는 멤버 함수를 찾아보라. 문제는 데이터 멤버를 직접 건드릴 필요가 없는 많은 멤버 함수들이 실제로 그렇게 한다는 것이다.

> **Enforcement**: Look for member function that do not touch data members directly.
The snag is that many member functions that do not need to touch data members directly do.


<a name="Rc-member"></a>

### C.5: 헬퍼 함수들은 그들이 도와주는 클래스와 같은 네임스페이스에 두어라

> ### C.5: Place helper functions in the same namespace as the class they support

**근거**: 헬퍼 함수는 (보통 클래스 작성자가 제공하는) 클래스의 표현에 직접 접근할 필요가 없는 함수이며, 클래스에 대한 유용한 인터페이스 중에 하나로 볼 수 있다.
헬퍼 함수들을 같은 네임스페이스에 넣으면 클래스에 대한 관계가 명확해지고, 인자 종속적인 검색에서 발견 할 수 있게 된다.

> **Reason**: A helper function is a function (usually supplied by the writer of a class) that does not need direct access to the representation of the class,
> yet is seen as part of the useful interface to the class.
> Placing them in the same namespace as the class makes their relationship to the class obvious and allows them to be found by argument dependent lookup.

**예**:

	namespace Chrono { // here we keep time-related services

		class Time { /* ... */ };
		class Date { /* ... */ };

		// helper functions:
		bool operator==(Date,Date);
		Date next_weekday(Date);
		// ...
	}

**시행하기**:

* 하나의 네임스페이스에서 인자 타입을 취하는 전역함수들을 표시해 두어라.

> * Flag global functions taking argument types from a single namespace.


<a name="Rc-const"></a>

### C.6: 객체의 상태를 수정하지 않는 멤버 함수는 `const` 로 선언하라

> ### C.6: Declare a member function that does not modify the state of its object `const`

**근거**: 더 정확한 디자인 의도에 대한 기술, 더 좋은 가독성, 컴파일러에 의해 더 많은 에러가 찾아짐, 더 많은 최적화 기회.

> **Reason**: More precise statement of design intent, better readability, more errors caught by the compiler, more optimization opportunities.

**예**:

	int Date::day() const { return d; }

**참고 사항**: [`const` 를 멀리하지 말라](#Res-casts-const).

> **Note**: [Do not cast away `const`](#Res-casts-const).

**시행하기**: 겍체를 수정하지 않는 `const`가 아닌 멤버 함수들을 표시해 보라.

> **Enforcement**: Flag non-`const` member functions that do not write to their objects


<a name="SS-concrete"></a>

## C.concrete: 구체적인 타입

> ## C.concrete: Concrete types

이상적인 클래스는 기본 타입이 되는 것이다.
대략 "`int` 처럼 동작하는 것"을 의미한다. 구체적인 타입은 가장 간단한 종류의 클래스이다.
기본 타입의 값은 복사 될 수 있고, 복사이 결과는 원본과 같은 값을 갖는 독립적인 객체이다.
구체적인 타입이 `=` 와 `==` 를 둘다 갖는다면, `a=b` 는 `a==b` 일 때 참이 될 것이다.
할당과 동등연산이 없는 구체적인 타입을 정의할 수 있지만, 이렇게 하는 것은 드물다. (그래야 한다.)
C++ 의 내장 타입들은 규칙적이며, 표준 라이브러리의 클래스인 `string`, `vector`, `map` 도 그렇다.
구체적인 타입들은 계층구조의 일부로 사용되는 클래스와 구분하기 위해 종종 값 타입으로 언급되기도 한다.

> One ideal for a class is to be a regular type.
> That means roughly "behaves like an `int`." A concrete type is the simplest kind of class.
> A value of regular type can be copied and the result of a copy is an independent object with the same value as the original.
> If a concrete type has both `=` and `==`, `a=b` should result in `a==b` being `true`.
> Concrete classes without assignment and equality can be defined, but they are (and should be) rare.
> The C++ built-in types are regular, and so are standard-library classes, such as `string`, `vector`, and `map`.
> Concrete types are also often referred to as value types to distinguish them from types uses as part of a hierarchy.

구체적인 타입 규칙 요약:

> Concrete type rule summary:

* [C.10: 복잡한 클래스들 보다는 구체적인 타입을 선호하라](#Rc-concrete)
* [C.11: 구체적인 타입을 기본 타입처럼 동작하도록 하라](#Rc-regular)

> * [C.10: Prefer a concrete type over more complicated classes](#Rc-concrete)
> * [C.11: Make a concrete types regular](#Rc-regular)


<a name="Rc-concrete"></a>

### C.10 복잡한 클래스들 보다는 구체적인 타입을 선호하라

> ### C.10 Prefer a concrete type over more complicated classes

**근거**: 구체적인 타입은 근본적으로 계층구조 보다 단순하다:
디자인이 더 쉽고, 구현이 더 쉽고, 사용하기가 더 쉬우며, 추론하기 더 쉽다. 더 작고 더 빠르기도 하다.

> **Reason**: A concrete type is fundamentally simpler than a hierarchy:
> easier to design, easier to implement, easier to use, easier to reason about, smaller, and faster.
> You need a reason (use cases) for using a hierarchy.

**예**

	class Point1 {
		int x, y;
		//  ... operations ...
		// .. no virtual functions ...
	};

	class Point2 {
		int x, y;
		// ... operations, some virtual ...
		virtual ~Point2();
	};

	void use()
	{
		Point1 p11 { 1,2};	// make an object on the stack
		Point1 p12 {p11};	// a copy

		auto p21 = make_unique<Point2>(1,2);	// make an object on the free store
		auto p22 = p21.clone();					// make a copy

		// ...
	}

클래스가 계층구조의 일부가 될 수 있다면, (실제 코드에서 작은 예들에서는 필연적이지 않다면) 포인터나 레퍼런스로 객체를 다루어야 한다.
이것으로 인한 간접처리를 위해 더 많은 메모리를 사용하게 되고, 더 많은 할당과 해제, 실행시간 추가비용이 발생하게 된다.

> If a class can be part of a hierarchy, we (in real code if not necessarily in small examples) must manipulate it's objects through pointers or references.
> That implies more memory overhead, more allocations and deallocations, and more run-time overhead to perform the resulting indiretions.

**참고 사항**: 구체적인 타입은 스택에 할당 될 수 있고, 다른 클래스의 멤버가 될 수 있다.

> **Note**: Concrete types can be stack allocated and be members of other classes.

**참고 사항**: 실행시간 다형적 인터페이스를 위해 간접처리는 필수적이다.
할당과 해제의 추가비용은 그렇지 않다. (단지 가장 흔한 사례일 뿐이다)
패생 클래스의 scoped object 에 대한 인터페이스로 베이스 클래스를 사용할 수 있다.
동적 할당을 할 수 없으며, 플러그인과 같은 것들에게 안정적인 인터페이스를 제공하고자 할 때 이렇게 할 수 있다. (예, 하드 리얼타임)

> **Note**: The use of indirection is fundamental for run-time polymorphic interfaces.
> The allocation/deallocation overhead is not (that's just the most common case).
> We can use a base class as the interface of a scoped object of a derived class.
> This is done where dynamic allocation is prohibited (e.g. hard real-time) and to provide a stable interface to some kinds of plug-ins.

**시행하기**: ???


<a name="Rc-regular"></a>
### C.11: 구체적인 타입을 기본 타입처럼 동작하도록 하라]

> ### C.11: Make a concrete types regular

**근거**: 기본 타입은 그렇지 못한 타입보다 이해하거나 추론하기 쉽다 (변칙적인 것들은 이해하고 사용하는데 추가적인 노력을 필요로 한다)

> **Reason**: Regular types are easier to understand and reason about than types that are not regular (irregularities requires extra effort to understand and use).

**예**:

	struct Bundle {
		string name;
		vector<Record> vr;
	};

	bool operator==(const Bundle& a, const Bundle& b) { return a.name==b.name && a.vr==b.vr; }

	Bundle b1 { "my bundle", {r1,r2,r3}};
	Bundle b2 = b1;
	if (!(b1==b2)) error("impossible!");
	b2.name = "the other bundle";
	if (b1==b2) error("No!");

특히, 구체적인 타입이 할당 연산을 갖는 다면, 동등 연산자도 만들어서 `a=b` 할 경우 `a==b` 를 의미하도록 하라.

> In particular, if a concrete type has an assignment also give it an equals operator so that `a=b` implies `a==b`.

**시행하기**: ???


<a name="SS-ctor"></a>
## C.ctor: 생성자, 할당, 소멸자

> ## C.ctor: Constructors, assignments, and destructors

이 함수들은 객체의 생명주기를 제어 한다: 생성, 복사, 이동, 그리고 파괴
생성자를 정의해서 클래스의 초기화를 보장하고 단순화 하라.

> These functions control the lifecycle of objects: creation, copy, move, and destruction.
> Define constructors to guarantee and simplify initialization of classes.

*기본적으로 있는 연산들*:

> These are *default operations*:

* 기본 생성자: `X()`
* 복사 생성자: `X(const X&)`
* 복사 할당자: `operator=(const X&)`
* 이동 생성자: `X(X&&)`
* 이동 할당자: `operator=(X&&)`
* 소멸자: `~X()`

> * a default constructor: `X()`
> * a copy constructor: `X(const X&)`
> * a copy assignment: `operator=(const X&)`
> * a move constructor: `X(X&&)`
> * a a move assignment: `operator=(X&&)`
> * a destructor: `~X()`

각 연산들이 사용될 경우 컴파일러가 정의 해주는데, 컴파일러가 생성하지 않도록 할 수도 있다.

> By default, the compiler defines each of these operations if it is used, but the default can be suppressed.

컴파일러가 생성하는 기본 연산들은 객체의 생명주기를 의미하는 연산들의 집합이다.
기본적으로, C++은 클래스를 값과 같은 타입으로 다루지만 모든 타입이 값 타입은 아니다.

> The default operations are a set of related operations that together implement the lifecycle semantics of an object.
> By default, C++ treats classes as value-like types, but not all types are value-like.

기본 연산들의 규칙들:

> Set of default operations rules:

* [C.20: 기본 연산을 정의하지 않아도 되면 그렇게 하라](#Rc-zero)
* [C.21: 기본 연산을 정의 하거나 `=delete` 로 선언 한다면, 모두 정의하거나 모두 `=delete` 로 선언하라](#Rc-five)
* [C.22: 기본 연산들을 일관성 있도록 하라](#Rc-matched)

> * [C.20: If you can avoid defining any default operations, do](#Rc-zero)
> * [C.21: If you define or `=delete` any default operation, define or `=delete` them all](#Rc-five)
> * [C.22: Make default operations consistent](#Rc-matched)

소멸자 규칙들:

> Destructor rules:

* [C.30: 객체가 없어질 때, 명시적인 동작이 필요할 경우 소멸자를 정의하라](#Rc-dtor)
* [C.31: 클래스에 의해 얻어진 모든 리로스는 소멸자에서 해제되어야 한다](#Rc-dtor-release)
* [C.32: 클래스가 포인터(`T*`)나 참조(`T&`)를 갖고 있을 때, 소유하고 있는 것인지 고려해 보라](#Rc-dtor-ptr)
* [C.33: 클래스가 포인터 멤버를 소유하고 있다면, 파과자를 정의하거나 `=delete` 로 선언하라](#Rc-dtor-ptr)
* [C.34: 클래스가 참조 멤버를 소유하고 있다면, 파과자를 정의하거나 `=delete` 로 선언하라](#Rc-dtor-ref)
* [C.35: 가상 함수를 갖는 기본 클래스는 가상 소멸자가 필요하다](#Rc-dtor-virtual)
* [C.36: 소멸자는 실패하지 않을 것이다](#Rc-dtor-fail)
* [C.37: 소멸자를 `noexcept`로 하라](#Rc-dtor-noexcept)

> * [C.30: Define a destructor if a class needs an explicit action at object destruction](#Rc-dtor)
> * [C.31: All resources acquired by a class must be released by the class's destructor](#Rc-dtor-release)
> * [C.32: If a class has a raw pointer (`T*`) or reference (`T&`), consider whether it might be owning](#Rc-dtor-ptr)
> * [C.33: If a class has an owning pointer member, define or `=delete` a destructor](#Rc-dtor-ptr)
> * [C.34: If a class has an owning reference member, define or `=delete` a destructor](#Rc-dtor-ref)
> * [C.35: A base class with a virtual function needs a virtual destructor](#Rc-dtor-virtual)
> * [C.36: A destructor may not fail](#Rc-dtor-fail)
> * [C.37: Make destructors `noexcept`](#Rc-dtor-noexcept)

생성자 규칙들:

> Constructor rules:

* [C.40: 클래스가 불변조건이면 생성자를 정의하라](#Rc-ctor)
* [C.41: 생성자는 완전히 초기화된 객체를 생성하는 것이 좋다](#Rc-complete)
* [C.42: 생성자가 유효한 객체를 생성하지 못한다면, 예외를 던지도록 하라](#Rc-throw)
* [C.43: 클래스가 기본 생성자를 갖도록 하라](#Rc-default0)
* [C.44: 단순하고 예외를 던지지 않는 기본 생성자를 선호하라](#Rc-default00)
* [C.45: 데이터 멤버를 초기화 하기 위해 기본 생성자를 정의하지 말라; 대신 멤버 초기화자를 사용하라](#Rc-default)
* [C.46: 기본적으로 하나의 인자를 받는 생성자는 `explicit` 로 선언하라](#Rc-explicit)
* [C.47: 멤버 변수들은 선언된 순서대로 정의하고 초기화 하라](#Rc-order)
* [C.48: 생성자에서 초기화 할 때 항상 동일하게 초기화 되는 멤버는 in-class 초기화자를 선호하라](#Rc-in-class-initializer)
* [C.49: 생성자에서 할당 보다는 초기화를 선호하라](#Rc-initialize)
* [C.50: 초기화 하는 동안 "virtual behavior" 가 필요하다면 팩토리 함수를 사용하라](#Rc-factory)
* [C.51: Use delegating constructors to represent common actions for all constructors of a class](#Rc-delegating)
* [C.52: Use inheriting constructors to import constructors into a derived class that does not need further explicit initialization](#Rc-inheriting)

> * [C.40: Define a constructor if a class has an invariant](#Rc-ctor)
> * [C.41: A constructor should create a fully initialized object](#Rc-complete)
> * [C.42: If a constructor cannot construct a valid object, throw an exception](#Rc-throw)
> * [C.43: Give a class a default constructor](#Rc-default0)
> * [C.44: Prefer default constructors to be simple and non-throwing](#Rc-default00)
> * [C.45: Don't define a default constructor that only initializes data members; use member initializers instead](#Rc-default)
> * [C.46: By default, declare single-argument constructors `explicit`](#Rc-explicit)
> * [C.47: Define and initialize member variables in the order of member declaration](#Rc-order)
> * [C.48: Prefer in-class initializers to member initializers in constructors for constant initializers](#Rc-in-class-initializer)
> * [C.49: Prefer initialization to assignment in constructors](#Rc-initialize)
> * [C.50: Use a factory function if you need "virtual behavior" during initialization](#Rc-factory)
> * [C.51: Use delegating constructors to represent common actions for all constructors of a class](#Rc-delegating)
> * [C.52: Use inheriting constructors to import constructors into a derived class that does not need further explicit initialization](#Rc-inheriting)

Copy and move rules:

* [C.60: Make copy assignment non-`virtual`, take the parameter by `const&`, and return by non-`const&`](#Rc-copy-assignment)
* [C.61: A copy operation should copy](#Rc-copy-semantic)
* [C.62: Make copy assignment safe for self-assignment](#Rc-move-self)
* [C.63: Make move assignment non-`virtual`, take the parameter by `&&`, and return by non-`const&`](#Rc-move-assignment)
* [C.64: A move operation should move and leave its source in a valid state](#Rc-move-semantic)
* [C.65: Make move assignment safe for self-assignment](#Rc-copy-self)
* [C.66: Make move operations `noexcept`](#Rc-move-noexcept)
* [C.67: A base class should suppress copying, and provide a virtual `clone` instead if "copying" is desired](#Rc-copy-virtual)

Other default operations rules:

* [C.80: Use `=default` if you have to be explicit about using the default semantics](#Rc-=default)
* [C.81: Use `=delete` when you want to disable default behavior (without wanting an alternative)](#Rc-=delete)
* [C.82: Don't call virtual functions in constructors and destructors](#Rc-ctor-virtual)
* [C.83: For value-like types, consider providing a `noexcept` swap function](#Rc-swap)
* [C.84: A `swap` may not fail](#Rc-swap-fail)
* [C.85: Make `swap` `noexcept`](#Rc-swap-noexcept)
* [C.86: Make `==` symmetric with respect of operand types and `noexcept`](#Rc-eq)
* [C.87: Beware of `==` on base classes](#Rc-eq-base)
* [C.88: Make `<` symmetric with respect of operand types and `noexcept`](#Rc-lt)
* [C.89: Make a `hash` `noexcept`](#Rc-hash)


<a name="SS-defop"></a>

## C.defop: 기본 연산들

> ## C.defop: Default Operations

기본적으로, 언어에서 기본적인 의미를 담는 기본 연산들을 제공한다.
그러나, 프로그래머는 기본적으로 제공되는 것들을 막거나 바꿀 수 있다.

> By default, the language supply the default operations with their default semantics.
> However, a programmer can disalble or replace these defaults.

<a name="Rc-zero"></a>

### C.20: 기본 연산을 정의하지 않아도 되면 그렇게 하라

> ### C.20: If you can avoid defining default operations, do

**근거**: 가장 단순하고, 가장 명료한 의미를 준다.

> **Reason**: It's the simplest and gives the cleanest semantics.

**예**:

	struct Named_map {
	public:
		// ... no default operations declared ...
	private:
		string name;
		map<int,int> rep;
	};

	Named_map nm;		// default construct
	Named_map nm2 {nm};	// copy construct

`std::map` 과 `string` 은 모든 특수한 함수들을 갖고 있다, 추가적인 작업이 필요없다.

> Since `std::map` and `string` have all the special functions, not further work is needed.

**참고 사항**: "0의 규칙"으로 알려져 있다.

> **Note**: This is known as "the rule of zero".

**시행하기**: 시행할 수 없더라도, 좋은 정적 분석기는 이 규칙에 맞는 가능한 개선사항들을 가르키는 패턴들을 찾을 수 있다.
예를 들면, 포인터와 크기를 멤버로 갖는 클래스가 있고 소멸자에서 그 포인터를 `delete` 한다면 아마도 `vector` 로 바꿀 수 있을 것이다.

> **Enforcement**: (Not enforceable) While not enforceable, a good static analyzer can detect patterns that indicate a possible improvement to meet this rule.
> For example, a class with a (pointer,size) pair of member and a destructor that `delete`s the pointer could probably be converted to a `vector`.


<a name="Rc-five"></a>
### C.21: If you define or `=delete` any default operation, define or `=delete` them all

**Reason**: The semantics of the special functions are closely related, so it one needs to be non-default, the odds are that other need modification.

**Example, bad**:

	struct M2 {		// bad: incomplete set of default operations
	public:
		// ...
		// ... no copy or move operations ...
		~M2() { delete[] rep; }
	private:
		pair<int,int>* rep;  // zero-terminated set of pairs
	};

	void use()
	{
		M2 x;
		M2 y;
		// ...
		x = y;	// the default assignment
		// ...
	}

Given that "special attention" was needed for the destructor (here, to deallocate), the likelihood that copy and move assignment (both will implicitly destroy an object) are correct is low (here, we would get double deletion).

**Note**: This is known as "the rule of five" or "the rule of six", depending on whether you count the default constructor.

**Note**: If you want a default implementation of a default operation (while defining another), write `=default` to show you're doing so intentionally for that function.
If you don't want a default operation, suppress it with `=delete`.

**Note:** Compilers enforce much of this rule and ideally warn about any violation.

**Note**: Relying on an implicitly generated copy operation in a class with a destructor is deprecated.

**Enforcement**: (Simple) A class should have a declaration (even a `=delete` one) for either all or none of the special functions.


<a name="Rc-matched"></a>
### C.22: Make default operations consistent

**Reason**: The default operations are conceptually a matched set. Their semantics is interrelated.
Users will be surprised if copy/move construction and copy/move assignment do logically different things. Users will be surprised if constructors and destructors do not provide a consistent view of resource management. Users will be surprised if copy and move doesn't reflect the way constructors and destructors work.

**Example; bad**:

	class Silly { 		// BAD: Inconsistent copy operations
		class Impl {
			// ...
		};
        shared_ptr<Impl> p;
	public:
		Silly(const Silly& a) : p{a.p} { *p = *a.p; }   // deep copy
		Silly& operator=(const Silly& a) { p = a.p; }   // shallow copy
		// ...
	};

These operations disagree about copy semantics. This will lead to confusion and bugs.

**Enforcement**:

* (Complex) A copy/move constructor and the corresponding copy/move assignment operator should write to the same member variables at the same level of dereference.
* (Complex) Any member variables written in a copy/move constructor should also be initialized by all other constructors.
* (Complex) If a copy/move constructor performs a deep copy of a member variable, then the destructor should modify the member variable.
* (Complex) If a destructor is modifying a member variable, that member variable should be written in any copy/move constructors or assignment operators.



<a name="SS-dtor"></a>
## C.dtor: Destructors

Does this class need a destructor is a surprisingly powerful design question.
For most classes the answer is "no" either because the class holds no resources or because destruction is handled by [the rule of zero](#Rc-zero);
that is, it's members can take care of themselves as concerns destruction.
If the answer is "yes", much of the design of the class follows (see [the rule of five](#Rc-five).


<a name="Rc-dtor"></a>
### C.30: Define a destructor if a class needs an explicit action at object destruction

**Reason**: A destructor is implicitly invoked at the end of an objects lifetime.
If the default destructor is sufficient, use it.
Only if you need code that is not simply destructors of members executed, define a non-default destructor.

**Example**:

	template<typename A>
	struct Final_action {	// slightly simplified
		A act;
		Final_action(F a) :act{a} {}
		~Final_action() { act(); }
	};

	template<typename A>
	Final_action<A> finally(A act)	// deduce action type
	{
		return Final_action<A>{a};
	}

	void test()
	{
		auto act = finally([]{ cout<<"Exit test\n"; });	// establish exit action
		// ...
		if (something) return;	// act done here
		// ...
	} // act done here

The whole purpose of `Final_action` is to get a piece of code (usually a lambda) executed upon destruction.

**Note**: There are two general categories of classes that need a user-defined destructor:

* A class with a resource that is not already represented as a class with a destructor, e.g., a `vector` or a transaction class.
* A class that exists primarily to execute an action upon destruction, such as a tracer or `Final_action`.

**Example, bad**:

	class Foo {		// bad; use the default destructor
	public:
		// ...
		~Foo() { s=""; i=0; vi.clear(); }  // clean up
	private:
		string s;
		int i;
		vector<int> vi;
	}

The default destructor does it better, more efficiently, and can't get it wrong.

**Note**: If the default destructor is needed, but its generation has been suppressed (e.g., by defining a move constructor), use `=default`.

**Enforcement**: Look for likely "implicit resources", such as pointers and references. Look for classes with destructors even though all their data members have destructors.


<a name="Rc-dtor-release"></a>
### C.31: All resources acquired by a class must be released by the class's destructor

**Reason**: Prevention of resource leaks, especially in error cases.

**Note**: For resources represented as classes with a complete set of default operations, this happens automatically.

**Example**:

	class X {
		ifstream f;	// may own a file
		// ... no default operations defined or =deleted ...
	};

`X`'s `ifstream` implicitly closes any file it may have open upon destruction of its `X`.

**Example; bad**:

	class X2 {	// bad
		FILE* f;	// may own a file
		// ... no default operations defined or =deleted ...
	};

`X2` may leak a file handle.

**Note**: What about a sockets that won't close? A destructor, close, or cleanup operation [should never fail](#Rc-dtor-fail).
If it does nevertheless, we have a problem that has no really good solution.
For starters, the writer of a destructor does not know why the destructor is called and cannot "refuse to act" by throwing an exception.
See [discussion](#Sd-never-fail).
To make the problem worse, many "close/release" operations are not retryable.
Many have tried to solve this problem, but no general solution is known.
If at all possible, consider failure to close/cleanup a fundamental design error and terminate.

**Note**: A class can hold pointers and references to objects that it does not own.
Obviously, such objects should not be `delete`d by the class's destructor.
For example:

	Preprocessor pp { /* ... */ };
	Parser p { pp, /* ... */ };
	Type_checker tc { p, /* ... */ };

Here `p` refers to `pp` but does not own it.

**Enforcement**:

* (Simple) If a class has pointer or reference member variables that are owners
(e.g., deemed owners by using `GSL::owner`), then they should be referenced in its destructor.
* (Hard) Determine if pointer or reference member variables are owners when there is no explicit statement of ownership
(e.g., look into the constructors).


<a name="Rc-dtor-ptr"></a>
### C.32: If a class has a raw  pointer (`T*`) or reference (`T&`), consider whether it might be owning

**Reason**: There is a lot of code that is non-specific about ownership.

**Example**:

	???

**Note**: If the `T*` or `T&` is owning, mark it `owning`. If the `T*` is not owning, consider marking it `ptr`.
This will aide documentation and analysis.

**Enforcement**: Look at the initialization of raw member pointers and member references and see if an allocation is used.


<a name="Rc-dtor-ptr"></a>
### C.33: If a class has an owning pointer member, define a destructor

**Reason**: An owned object must be `deleted` upon destruction of the object that owns it.

**Example**: A pointer member may represent a resource.
[A `T*` should not do so](#Rr-ptr), but in older code, that's common.
Consider a `T*` a possible owner and therefore suspect.

	template<typename T>
	class Smart_ptr {
		T* p;	// BAD: vague about ownership of *p
		// ...
	public:
		// ... no user-defined default operations ...
	};

	void use(Smart_ptr<int> p1)
	{
		auto p2 = p1;	// error: p2.p leaked (if not nullptr and not owned by some other code)
	}

Note that if you define a destructor, you must define or delete [all default operations](#Rc-five):

	template<typename T>
	class Smart_ptr2 {
		T* p;	// BAD: vague about ownership of *p
		// ...
	public:
		// ... no user-defined copy operations ...
		~Smart_ptr2() { delete p; }		// p is an owner!
	};

	void use(Smart_ptr<int> p1)
	{
		auto p2 = p1;	// error: double deletion
	}

The default copy operation will just copy the `p1.p` into `p2.p` leading to a double destruction of `p1.p`. Be explicit about ownership:

	template<typename T>
	class Smart_ptr3 {
		owner<T>* p;	// OK: explicit about ownership of *p
		// ...
	public:
		// ...
		// ... copy and move operations ...
		~Smart_ptr3() { delete p; }
	};

	void use(Smart_ptr3<int> p1)
	{
		auto p2 = p1;	// error: double deletion
	}


 **Note**: Often the simplest way to get a destructor is to replace the pointer with a smart pointer (e.g., `std::unique_ptr`)
 and let the compiler arrange for proper destruction to be done implicitly.

**Note**: Why not just require all owning pointers to be "smart pointers"?
 That would sometimes require non-trivial code changes and may affect ABIs.

**Enforcement**:

* A class with a pointer data member is suspect.
* A class with an `owner<T>` should define its default operations.


<a name="Rc-dtor-ref"></a>
### C.34: If a class has an owning reference member, define a destructor

**Reason**: A reference member may represent a resource.
It should not do so, but in older code, that's common.
See [pointer members and destructors](#Rc-dtor ptr).
Also, copying may lead to slicing.

**Example, bad**:

	class Handle {		// Very suspect
		Shape& s;	// use reference rather than pointer to prevent rebinding
					// BAD: vague about ownership of *p
		// ...
	public:
		Handle(Shape& ss) : s{ss} { /* ... */ }
		// ...
	};

The problem of whether `Handle` is responsible for the destruction of its `Shape` is the same as for <a ref="#Rc-dtor ptr">the pointer case</a>:
If the `Handle` owns the object referred to by `s` it must have a destructor.

**Example**:

	class Handle {		// OK
		owner<Shape&> s;	// use reference rather than pointer to prevent rebinding
		// ...
	public:
		Handle(Shape& ss) : s{ss} { /* ... */ }
		~Handle() { delete &s; }
		// ...
	};

Independently of whether `Handle` owns its `Shape`, we must consider the default copy operations suspect:

	Handle x {*new Circle{p1,17}};	// the Handle had better own the Circle or we have a leak
	Handle y {*new Triangle{p1,p2,p3}};
	x = y;		// the default assignment will try *x.s=*y.s

That `x=y` is highly suspect.
Assigning a `Triangle` to a `Circle`?
Unless `Shape` has its [copy assignment `=deleted`](#Rc-copy-virtual), only the `Shape` part of `Triangle` is copied into the `Circle`.


**Note**: Why not just require all owning refereces to be replaced by "smart pointers"?
 Changing from references to smart pointers implies code changes.
 We don't (yet) havesmart references.
 Also, that may affect ABIs.

**Enforcement**:

* A class with a reference data member is suspect.
* A class with an `owner<T>` reference should define its default operations.


<a name="Rc-dtor-virtual"></a>
### C.35: A base class with a virtual function needs a virtual destructor

**Reason**: To prevent undefined behavior.
If an application attempts to delete a derived class object through a base class pointer, the result is undefined if the base class's destructor is non-virtual.
In general, the writer of a base class does not know the appropriate action to be done upon destruction.

**Example; bad**:

	struct Base {  // BAD: no virtual destructor
		virtual f();
	};

	struct D : Base {
		string s {"a resource needing cleanup"};
		~D() { /* ... do some cleanup ... */ }
		// ...
	};

    void use()
	{
	    unique_ptr<Base> p = make_unique<D>();
        // ...
	} // p's destruction calls ~Base(), not ~D(), which leaks D::s and possibly more

**Note**: A virtual function defines an interface to derived classes that can be used without looking at the derived classes.
Someone using such an interface is likely to also destroy using that interface.

**Note**: A destructor must be `public` or it will prevent stack allocation and normal heap allocation via smart pointer (or in legacy code explicit `delete`):

	class X {
		~X();	// private destructor
		// ...
	};

	void use()
	{
		X a;						// error: cannot destroy
		auto p = make_unique<X>();	// error: cannot destroy
	}

**Enforcement**: (Simple) A class with any virtual functions should have a virtual destructor.


<a name="Rc-dtor-fail"></a>
### C.36: A destructor may not fail

**Reason**: In general we do not know how to write error-free code if a destructor should fail.
The standard library requires that all classes it deals with have destructors that do not exit by throwing.

**Example**:

	class X {
	public:
		~X() noexcept;
		// ...
	};

	X::~X() noexcept
	{
		// ...
		if (cannot_release_a_resource) terminate();
		// ...
	}

**Note**: Many have tried to devise a fool-proof scheme for dealing with failure in destructors.
None have succeeded to come up with a general scheme.
This can be be a real practical problem: For example, what about a sockets that won't close?
The writer of a destructor does not know why the destructor is called and cannot "refuse to act" by throwing an exception.
See <a =href="#Sd dtor">discussion</a>.
To make the problem worse, many "close/release" operations are not retryable.
If at all possible, consider failure to close/cleanup a fundamental design error and terminate.

**Note**: Declare a destructor `noexcept`. That will ensure that it either completes normally or terminate the program.

**Note**: If a resource cannot be released and the program may not fail, try to signal the failure to the rest of the system somehow
(maybe even by modifying some global state and hope something will notice and be able to take care of the problem).
Be fully aware that this technique is special-purpose and error-prone.
Consider the "my connection will not close" example.
Probably there is a problem at the other end of the connection and only a piece of code responsible for both ends of the connection can properly handle the problem.
The destructor could send a message (somehow) to the responsible part of the system, consider that to have closed the connection, and return normally.

**Note**: If a destructor uses operations that may fail, it can catch exceptions and in some cases still complete successfully
(e.g., by using a different clean-up mechanism from the one that threw an exception).

**Enforcement**: (Simple) A destructor should be declared `noexcept`.


<a name="Rc-dtor-noexcept"></a>
### C.37: Make destructors `noexcept`

**Reason**: [A destructor may not fail](#Rc-dtor fail). If a destructor tries to exit with an exception, it's a bad design error and the program had better terminate.

**Enforcement**: (Simple) A destructor should be declared `noexcept`.



<a name="SS-ctor"></a>
## C.ctor: Constructors

A constuctor defined how an object is initialized (constructted).


<a name="Rc-ctor"></a>
### C.40: Define a constructor if a class has an invariant

**Reason**: That's what constructors are for.

**Example**:

	class Date {	// a Date represents a valid date
					// in the January 1, 1900 to December 31, 2100 range
		Date(int dd, int mm, int yy)
			:d{dd}, m{mm}, y{yy}
			{
				if (!is_valid(d,m,y)) throw Bad_date{};	// enforce invariant
			}
		// ...
	private:
		int d,m,y;
	};

It is often a good idea to express the invariant as an `Ensure` on the constructor.

**Note**: A constructor can be used for convenience even if a class does not have an invariant. For example:

	struct Rec {
		string s;
		int i {0};
		Rec(const string& ss) : s{ss} {}
		Rec(int ii) :i{ii} {}
	};

	Rec r1 {7};
	Rec r2 {"Foo bar"};

**Note**: The C++11 initializer list rules eliminates the need for many constructors. For example:

	struct Rec2{
		string s;
		int i;
		Rec2(const string& ss, int ii = 0} :s{ss}, i{ii} {}		// redundant
	};

	Rec r1 {"Foo",7};
	Rec r2 {"Bar};

The `Rec2` constructor is redundant.
Also, the default for `int` would be better done as a [member initializer](#Rc-in-class initializer).

**See also**: [construct valid object](#Rc-complete) and [constructor throws](#Rc-throw).

**Enforcement**:

* Flag classes with user-define copy operations but no destructor (a user-defined copy is a good indicator that the class has an invariant)


<a name="Rc-complete"></a>
### C.41: A constructor should create a fully initialized object

**Reason**: A constructor establishes the invariant for a class. A user of a class should be able to assume that a constructed object is usable.

**Example; bad**:

	class X1 {
		FILE* f;	// call init() before any other fuction
		// ...
	public:
		X1() {}
		void init();	// initialize f
		void read();	// read from f
		// ...
	};

	void f()
	{
		X1 file;
		file.read();	// crash or bad read!
		// ...
		file.init();	// too late
		// ...
	}

Compilers do not read comments.

**Exception**: If a valid object cannot conveniently be constructed by a constructor [use a factory function](#C factory).

**Note**: If a constructor acquires a resource (to create a valid object), that resource should be [released by the destructor](#Rc-release).
The idiom of having constructors acquire resources and destructors release them is called [RAII](Rr-raii) ("Resource Acquisitions Is Initialization").


<a name="Rc-throw"></a>
### C.42: If a constructor cannot construct a valid object, throw an exception

**Reason**: Leaving behind an invalid object is asking for trouble.

**Example**:

	class X2 {
		FILE* f;	// call init() before any other fuction
		// ...
	public:
		X2(const string& name)
			:f{fopen(name.c_str(),"r"}
		{
			if (f==nullptr) throw runrime_error{"could not open" + name};
			// ...
		}

		void read();	// read from f
		// ...
	};

	void f()
	{
		X2 file {"Zeno"}; // throws if file isn't open
		file.read();	  // fine
		// ...
	}

**Example, bad**:

	class X3 {			// bad: the constructor leaves a non-valid object behind
		FILE* f;	// call init() before any other fuction
		bool valid;;
		// ...
	public:
		X3(const string& name)
			:f{fopen(name.c_str(),"r"}, valid{false}
		{
			if (f) valid=true;
			// ...
		}

		void is_valid()() { return valid; }
		void read();		// read from f
		// ...
	};

	void f()
	{
		X3 file {Heraclides"};
		file.read();	// crash or bad read!
		// ...
		if (is_valid()()) {
			file.read();
			// ...
		}
		else {
			// ... handle error ...
		}
		// ...
	}

**Note**: For a variable definition (e.g., on the stack or as a member of another object) there is no explicit function call from which an error code could be returned. Leaving behind an invalid object an relying on users to consistently check an `is_valid()` function before use is tedious, error-prone, and inefficient.

**Exception**: There are domains, such as some hard-real-time systems (think airplane controls) where (without additional tool support) exception handling is not sufficiently predictable from a timing perspective. There the `is_valed()` technique must be used. In such cases, check `is_valid()` consistently and immediately to simulate [RAII](#Rc-raii).

**Alternative**: If you feel tempted to use some "post-constructor initialization" or "two-stage initialization" idiom, try not to do that. If you really have to, look at [factory functions](#Rc-factory).

**Enforcement**:
* (Simple) Every constructor should initialize every member variable (either explicitly, via a delegating ctor call or via default construction).
* (Unknown) If a constructor has an `Ensures` contract, try to see if it holds as a postcondition.


<a name="Rc-default0"></a>
### C.43: Give a class a default constructor

**Reason**: Many language and library facilities rely on default constructors,
e.g. `T a[10]` and `std::vector<T> v(10)` default initializes their elements.

**Example**:

	class Date {
	public:
		Date();
		// ...
	};

	vector<Date> vd1(1000);	// default Date needed here
	vector<Date> vd2(1000,Date{Month::october,7,1885});	// alternative

There is no "natural" default date (the big bang is too far back in time to be useful for most people), so this example is non-trivial.
`{0,0,0}` is not a valid date in most calendar systems, so choosing that would be introducing something like floating-point's NaN.
However, most realistic `Date` classes has a "first date" (e.g. January 1, 1970 is popular), so making that the default is usually trivial.

**Enforcement**:

* Flag classes without a default constructor


<a name="Rc-default00"></a>
### C.44: Prefer default constructors to be simple and non-throwing

**Reason**: Being able to set a value to "the default" without operations that might fail simplifies error handling and reasoning about move operations.

**Example, problematic**:

	template<typename T>
	class Vector0 {		// elem points to space-elem element allocated using new
	public:
		Vector0() :Vector0{0} {}
		Vector0(int n) :elem{new T[n]}, space{elem+n}, last{elem} {}
		// ...
	private:
		own<T*> elem;
		T* space;
		T* last;
	};

This is nice and general, but setting a `Vector0` to empty after an error involves an allocation, which may fail.
Also, having a default `Vector` represented as `{new T[0],0,0}` seems wasteful.
For example, `Vector0 v(100)` costs 100 allocations.

**Example**:

	template<typename T>
	class Vector1 {		// elem is nullptr or elem points to space-elem element allocated using new
	public:
		Vector1() noexcept {}	// sets the representation to {nullptr,nullptr,nullptr}; doesn't throw
		Vector1(int n) :elem{new T[n]}, space{elem+n}, last{elem} {}
		// ...
	private:
		own<T*> elem = nullptr;
		T* space = nullptr;
		T* last = nullptr;
	};

Using `{nullptr,nullptr,nullptr}` makes `Vector1{}` cheap, but a special case and implies run-time checks.
Setting a `Vector1` to empty after detecting an error is trivial.

**Enforcement**:

* Flag throwing default constructors


<a name="Rc-default"></a>
### C.45: Don't define a default constructor that only initializes data members; use in-class member initializers instead

**Reason**: Using in-class member initializers lets the compiler generate the function for you. The compiler-generated function can be more efficient.

**Example; bad**:

    class X1 { // BAD: doesn't use member initializers
        string s;
        int i;
    public:
        X1() :s{"default"}, i{1} { }
		// ...
    };

**Example**:

    class X2 {
        string s = "default";
        int i = 1;
    public:
        // use compiler-generated default constructor
		// ...
    };


**Enforcement**: (Simple) A default constructor should do more than just initialize member variables with constants.


<a name="Rc-explicit"></a>
### C.46: By default, declare single-argument constructors explicit

**Reason**: To avoid unintended conversions.

**Example; bad**:

	class String {
		// ...
	public:
		String(int);	// BAD
		// ...
	};

	String s = 10;	// surprise: string of size 10


**Exception**: If you really want an implicit conversion from the constructor argument type to the class type, don't use `explicit`:

	class Complex {
		// ...
	public:
		Complex(double d);	// OK: we want a conversion from d to {d,0}
		// ...
	};

	Complex z = 10.7;	// unsurprising conversion

**See also**: [Discussion of implicit conversions](#Ro-conversion).

**Enforcement**: (Simple) Single-argument constructors should be declared `explicit`. Good single argument non-`explicit` constructors are rare in most code based. Warn for all that are not on a "positive list".


<a name="Rc-order"></a>
### C.47: Define and initialize member variables in the order of member declaration

**Reason**: To minimize confusion and errors. That is the order in which the initialization happens (independent of the order of member initializers).

**Example; bad**:

	class Foo {
		int m1;
		int m2;
	public:
		Foo(int x) :m2{x}, m1{++x} { }	// BAD: misleading initializer order
		// ...
	};

	Foo x(1); // surprise: x.m1==x.m2==2

**Enforcement**: (Simple) A member initializer list should mention the members in the same order they are declared.

**See also**: [Discussion](#Sd order)


<a name="Rc-in-class-initializer"></a>
### C.48: Prefer in-class initializers to member initializers in constructors for constant initializers

**Reason**: Makes it explicit that the same value is expected to be used in all constructors. Avoids repetition. Avoids maintenance problems. It leads to the shortest and most efficient code.

**Example; bad**:

	class X {	// BAD
		int i;
		string s;
		int j;
	public:
		X() :i{666}, s{"qqq"} { }	// j is uninitialized
		X(int i) :i{ii} {}			// s is "" and j is uninitialized
		// ...
	};

How would a maintainer know whether `j` was deliberately uninitialized (probably a poor idea anyway) and whether it was intentional to give `s` the default value `""` in one case and `qqq` in another (almost certainly a bug)? The problem with `j` (forgetting to initialize a member) often happens when a new member is added to an existing class.

**Example**:

	class X2 {
		int i {666};
		string s {"qqq"};
		int j {0};
	public:
		X2() = default;			// all members are initialized to their defaults
		X2(int i) :i{ii} {}		// s and j initialized to their defaults
		// ...
	};

**Alternative**: We can get part of the benefits from default arguments to constructors, and that is not uncommon in older code. However, that is less explicit, causes more arguments to be passed, and is repetitive when there is more than one constructor:

	class X3 {	// BAD: inexplicit, argument passing overhead
		int i;
		string s;
		int j;
	public:
		X3(int ii = 666, const string& ss = "qqq", int jj = 0)
			:i{ii}, s{ss}, j{jj} { }		// all members are initialized to their defaults
		// ...
	};

**Enforcement**:
* (Simple) Every constructor should initialize every member variable (either explicitly, via a delegating ctor call or via default construction).
* (Simple) Default arguments to constructors suggest an in-class initalizer may be more appropriate.


<a name="Rc-initialize"></a>
### C.49: Prefer initialization to assignment in constructors

**Reason**: An initialization explicitly states that initialization, rather than assignment, is done and can be more elegant and efficient. Prevents "use before set" errors.

**Example; good**:

	class A {		// Good
	    string s1;
	public:
		A() : s1{"Hello, "} { }    // GOOD: directly construct
		// ...
	};

**Example; bad**:

	class B {		// BAD
    	string s1;
	public:
    	B() { s1 = "Hello, "; }   // BAD: default constuct followed by assignment
		// ...
	};

	class C {		// UGLY, aka very bad
		int* p;
	public:
		C() { cout << *p; p = new int{10};	}	// accidental use before initialized
		// ...
	};



<a name="Rc-factory"></a>
### C.50: Use a factory function if you need "virtual behavior" during initialization

**Reason**: If the state of a base class object must depend on the state of a derived part of the object,
 we need to use a virtual function (or equivalent) while minimizing the window of opportunity to misuse an imperfectly constructed object.

**Example; bad**:

	class B {
	public:
	    B()
		{
			// ...
			f(); 		// BAD: virtual call in constructor
			//...
		}

	    virtual void f() = 0;

	    // ...
	};

**Example*:

	class B {
	private:
	    B() { /* ... */ }					// create an imperfectly initialized object

	    virtual void PostInitialize()       // to be called right after construction
	    {
			// ...
			f();	// GOOD: virtual dispatch is safe
			// ...
		}

	public:
	    virtual void f() = 0;

	    template<class T>
	    static shared_ptr<T> Create()	// interface for creating objects
		{
	        auto p = make_shared<T>();
	        p->PostInitialize();
	        return p;
	    }
	};

	class D : public B { /* "¦ */ };			// some derived class

	shared_ptr<D> p = D::Create<D>();		// creating a D object

By making the constructor `private` we avoid an incompletely constructed object escaping into the wild.
By providing the factory function `Create()`, we make construction (on the free store) convenient.

**Note**: Conventional factory functions allocate on the free store, rather than on the stack or in an enclosing object.

**See also**: [Discussion](#Sd factory)


<a name="Rc-delegating"></a>
### C.51: Use delegating constructors to represent common actions for all constructors of a class

**Reason**: To avoid repetition and accidental differences

**Example; bad**:

	class Date {	// BAD: repettive
		int d;
		Month m;
		int y;
	public:
		Date(int ii, Month mm, year yy)
			:i{ii}, m{mm} y{yy}
			{ if (!valid(i,m,y)) throw Bad_date{}; }

		Date(int ii, Month mm)
			:i{ii}, m{mm} y{current_year()}
			{ if (!valid(i,m,y)) throw Bad_date{}; }
		// ...
	};

The common action gets tedious to write and may accidentally not be common.

**Example**:


	class Date2 {
		int d;
		Month m;
		int y;
	public:
		Date2(int ii, Month mm, year yy)
			:i{ii}, m{mm} y{yy}
			{ if (!valid(i,m,y)) throw Bad_date{}; }

		Date2(int ii, Month mm)
			:Date2{ii,mm,current_year{}} {}
		// ...
	};

**See also**: If the "repeated action" is a simple initialization, consider [an in-class member initializer](#Rc-in-class initializer).

**Enforcement**: (Moderate) Look for similar constructor bodies.


<a name="Rc-inheriting"></a>
### C.52: Use inheriting constructors to import constructors into a derived class that does not need further explicit initialization

**Reason**: If you need those constructors for a derived class, re-implementeing them is tedious and error prone.

**Example**: `std::vector` has a lot of tricky constructors, so it I want my own `vector`, I don't want to reimplement them:

	class Rec {
		// ... data and lots of nice constructors ...
	};

	class Oper : public Rec {
		using Rec::Rec;
		// ... no data members ...
		// ... lots of nice utility functions ...
	};

**Example; bad**:

	struct Rec2 : public Rec {
		int x;
		using Rec::Rec;
	};

	Rec2 r {"foo", 7};
	int val = r.x;	// uninitialized


**Enforcement**: Make sure that every member of the derived class is initialized.



<a name="SS-copy"></a>
## C.copy: Copy and move

Value type should generally be copyable, but interfaces in a class hierarchy should not.
Resource handles, may or may not be copyable.
Types can be defined to move for logical as well as performance reasons.


<a name="Rc-copy-assignment"></a>
### C.60: Make copy assignment non-`virtual`, take the parameter by `const&`, and return by non-`const&`

**Reason**: It is simple and efficient. If you want to optimize for rvalues, provide an overload that takes a `&&` (see [F.24](#Rf-pass-ref-ref)).

**Example**:

	class Foo {
	public:
		Foo& operator=(const Foo& x)
		{
            auto tmp = x;	// GOOD: no need to check for self-assignment (other than performance)
			std::swap(*this,tmp);
			return *this;
		}
		// ...
	};

	Foo a;
	Foo b;
	Foo f();

	a = b;		// assign lvalue: copy
	a = f();	// assign rvalue: potentially move

**Note**: The `swap` implementation technique offers the [strong guarantee](???).

**Example**: But what if you can get significant better performance by not making a temporary copy? Consider a simple `Vector` intended for a domain where assignment of large, equal-sized `Vector`s is common. In this case, the copy of elements implied by the `swap` implementation technique could cause an order of magnitude increase in cost:

	template<typename T>
	class Vector {
	public:
		Vector& operator=(const Vector&);
		// ...
	private:
		T* elem;
		int sz;
	};

	Vector& Vector::operator=(const Vector& a)
	{
		if (a.sz>sz)
		{
			// ... use the swap technique, it can't be bettered ...
			*return *this
		}
		// ... copy sz elements from *a.elem to elem ...
		if (a.sz<sz) {
			// ... destroy the surplus elements in *this* and adjust size ...
		}
		return *this*
	}

By writing directly to the target elements, we will get only [the basic guarantee](#???) rather than the strong guaranteed offered by the `swap` technique. Beware of [self assignment](#Rc-copy-self).

**Alternatives**: If you think you need a `virtual` assignment operator, and understand why that's deeply problematic, don't call it `operator=`. Make it a named function like `virtual void assign(const Foo&)`.
See [copy constructor vs. `clone()`](#Rc-copy-virtual).

**Enforcement**:

* (Simple) An assignment operator should not be virtual. Here be dragons!
* (Simple) An assignment operator should return `T&` to enable chaining, not alternatives like `const T&` which interfere with composability and putting objects in containers.
* (Moderate) An assignment operator should (implicitly or explicitly) invoke all base and member assignment operators.
Look at the destructor to determine if the type has pointer semantics or value semantics.


<a name="Rc-copy-semantic"></a>
### C.61: A copy operation should copy

**Reason**: That is the generally assumed semantics. After `x=y`, we should have `x==y`.
After a copy `x` and `y` can be independent objects (value semantics, the way non-pointer built-in types and the standard-library types work) or refer to a shared object (pointer semantics, the way pointers work).

**Example**:

	class X {	// OK: value sementics
	public:
		X();
		X(const X&);	// copy X
		void modify();	// change the value of X
		// ...
		~X() { delete[] p; }
	private:
		T* p;
		int sz;
	};

	bool operator==(const X& a, const X& b)
	{
		return sz==a.sz && equal(p,p+sz,a.p,a.p+sz);
	}

	X::X(const X& a)
		:p{new T}, sz{a.sz}
	{
		copy(a.p,a.p+sz,a.p);
	}

	X x;
	X y = x;
	if (x!=y) throw Bad{};
	x.modify();
	if (x==y) throw Bad{};	// assume value semantics

**Example**:

	class X2 {	// OK: pointer semantics
	public:
		X2();
		X2(const X&) = default;	// shallow copy
		~X2() = default;
		void modify();			// change the value of X
		// ...
	private:
		T* p;
		int sz;
	};

	bool operator==(const X2& a, const X2& b)
	{
		return sz==a.sz && p==a.p;
	}

	X2 x;
	X2 y = x;
	if (x!=y) throw Bad{};
	x.modify();
	if (x!=y) throw Bad{};	// assume pointer semantics

**Note**: Prefer copy semantics unless you are building a "smart pointer". Value semantics is the simplest to reason about and what the standard library facilities expect.

**Enforcement**: (Not enforceable).


<a name="Rc-copy-self"></a>
### C.62: Make copy assignment safe for self-assignment

**Reason**: If `x=x` changes the value of `x`, people will be surprised and bad errors will occur (often including leaks).

**Example**: The standard-library containers handle self-assignment elegantly and efficiently:

	std::vector<int> v = {3,1,4,1,5,9};
	v = v;
	// the value of v is still {3,1,4,1,5,9}

**Note**: The default assignment generated from members that handle self-assignment correctly handles self-assignment.

	struct Bar {
		vector<pair<int,int>> v;
		map<string,int> m;
		string s;
	};

	Bar b;
	// ...
	b = b;	// correct and efficient

**Note**: You can handle self-assignment by explicitly testing for self-assignment, but often it is faster and more elegant to cope without such a test (e.g., [using `swap`](#Rc-swap)).

	class Foo {
		string s;
		int i;
	public:
		Foo& operator=(const Foo& a);
		// ...
	};

	Foo& Foo::operator=(const Foo& a)	// OK, but there is a cost
	{
		if (this==&a) return *this;
		s = a.s;
		i = a.i;
		return *this;
	}

This is obviously safe and apparently efficient.
However, what if we do one self-assignment per million assignments?
That's about a million redundant tests (but since the answer is essentially always the same, the computer's branch predictor will guess right essentially every time).
Consider:

	Foo& Foo::operator=(const Foo& a)	// simpler, and probably much better
	{
		s = a.s;
		i = a.i;
		return *this;
	}

`std::string` is safe for self-assignment and so are `int`. All the cost is carried by the (rare) case of self-assignment.

**Enforcement**: (Simple) Assignment operators should not contain the pattern `if (this==&a) return *this;` ???


<a name="Rc-move-assignment"></a>
### C.63: Make move assignment non-`virtual`, take the parameter by `&&`, and return by non-`const `&`

**Reason**: It is simple and efficient.

**See**: [The rule for copy-assignment](#Rc-copy-assignment).

**Enforcement**: Equivalent to what is done for [copy-assignment](#Rc-copy-assignment).
* (Simple) An assignment operator should not be virtual. Here be dragons!
* (Simple) An assignment operator should return `T&` to enable chaining, not alternatives like `const T&` which interfere with composability and putting objects in containers.
* (Moderate) A move assignment operator should (implicitly or explicitly) invoke all base and member move assignment operators.


<a name="Rc-move-semantic"></a>
### C.64: A move operation should move and leave its source in valid state

**Reason**: That is the generally assumed semantics. After `x=std::move(y)` the value of `x` should be the value `y` had and `y` should be in a valid state.

**Example**:

	class X {	// OK: value sementics
	public:
		X();
		X(X&& a);		// move X
		void modify();	// change the value of X
		// ...
		~X() { delete[] p; }
	private:
		T* p;
		int sz;
	};


	X::X(X&& a)
		:p{a.p}, sz{a.sz}	// steal representation
	{
		a.p = nullptr;		// set to "empty"
		a.sz = 0;
	}

	void use()
	{
		X x{};
		// ...
		X y = std::move(x);
		x = X{};	// OK
	} // OK: x can be destroyed

**Note**: Ideally, that moved-from should be the default value of the type. Ensure that unless there is an exceptionally good reason not to. However, not all types have a default value and for some types establishing the default value can be expensive. The standard requires only that the moved-from object can be destroyed.
Often, we can easily and cheaply do better: The standard library assumes that it it possible to assign to a moved-from object. Always leave the moved-from object in some (necessarily specified) valid state.

**Note**: Unless there is an exceptionally strong reason not to, make `x=std::move(y); y=z;` work with the conventional semantics.

**Enforcement**: (Not enforceable) look for assignments to members in the move operation. If there is a default constructor, compare those assignments to the initializations in the default constructor.


<a name="Rc-move-self"></a>
### C.65: Make move assignment safe for self-assignment

**Reason**: If `x=x` changes the value of `x`, people will be surprised and bad errors may occur. However, people don't usually directly write a self-assignment that turn into a move, but it can occur. However, `std::swap` is implemented using move operations so if you accidentally do `swap(a,b)` where `a` and `b` refer to the same object, failing to handle self-move could be a serious and subtle error.

**Example**:

	class Foo {
		string s;
		int i;
	public:
		Foo& operator=(Foo&& a);
		// ...
	};

	Foo& Foo::operator=(Foo&& a)	// OK, but there is a cost
	{
		if (this==&a) return *this;	// this line is redundant
		s = std::move(a.s);
		i = a.i;
		return *this;
	}

The one-in-a-million argument against `if (this==&a) return *this;` tests from the discussion of [self-assignment](#Rc-copy self) is even more relevant for self-move.

**Note**: There is no know general way of avoiding a `if (this==&a) return *this;` test for a move assignment and still get a correct answer (i.e., after `x=x` the value of `x` is unchanged).

**Note** The ISO standard guarantees only a "valid but unspecified" state for the standard library containers. Apparently this has not been a problem in about 10 years of experimental and production use. Please contact the editors if you find a counter example. The rule here is more caution and insists on complete safety.

**Example**: Here is a way to move a pointer without a test (imagine it as code in the implementation a move assignment):

	// move from other.oter to this->ptr
	T* temp = other.ptr;
	other.ptr = nullptr;
	delete ptr;
	ptr = temp;

**Enforcement**:

* (Moderate) In the case of self-assignment, a move assignment operator should not leave the object holding pointer members that have been `delete`d or set to nullptr.
* (Not enforceable) Look at the use of standard-library container types (incl. `string`) and consider them safe for ordinary (not life-critical) uses.


<a name="Rc-move-noexcept"></a>
### C.66: Make move operations `noexcept`

**Reason**: A throwing move violates most people's reasonably assumptions.
A non-throwing move will be used more efficiently by standard-library and language facilities.

**Example**:

	class Vector {
		// ...
		Vector(Vector&& a) noexcept :elem{a.elem}, sz{a.sz} { a.sz=0; a.elem=nullptr; }
		Vector& operator=(Vector&& a) noexcept { elem=a.elem; sz=a.sz; a.sz=0; a.elem=nullptr; }
		//...
	public:
		T* elem;
		int sz;
	};

These copy operations do not throw.

**Example, bad**:

	class Vector2 {
		// ...
		Vector2(Vector2&& a) { *this = a; }				// just use the copy
		Vector2& operator=(Vector2&& a) { *this = a; }	// just use the copy
		//...
	public:
		T* elem;
		int sz;
	};

This `Vector2` is not just inefficient, but since a vector copy requires allocation, it can throw.

**Enforcement**: (Simple) A move operation should be marked `noexcept`.




<a name="Rc-copy-virtual"></a>
### C.67: A base class should suppress copying, and provide a virtual `clone` instead if "copying" is desired

**Reason**: To prevent slicing, because the normal copy operations will copy only the base portion of a derived object.

**Example; bad**:

    class B { // BAD: base class doesn't suppress copying
        int data;
        // ... nothing about copy operations, so uses default ...
    };

    class D : public B {
        string moredata; // add a data member
        // ...
    };

    auto d = make_unique<D>();
    auto b = make_unique<B>(d); // oops, slices the object; gets only d.data but drops d.moredata

**Example**:

    class B { // GOOD: base class suppresses copying
        B(const B&) =delete;
        B& operator=(const B&) =delete;
        virtual unique_ptr<B> clone() { return /* B object */; }
        // ...
    };

    class D : public B {
        string moredata; // add a data member
        unique_ptr<B> clone() override { return /* D object */; }
        // ...
    };

    auto d = make_unique<D>();
    auto b = d.clone(); // ok, deep clone

**Note**: It's good to return a smart pointer, but unlike with raw pointers the return type cannot be covariant (for example, `D::clone` can't return a `unique_ptr<D>`. Don't let this tempt you into returning an owning raw pointer; this is a minor drawback compared to the major robustness benefit delivered by the owning smart pointer.

**Enforcement**: A class with any virtual function should not have a copy constructor or copy assignment operator (compiler-generated or handwritten).



## C.other: Other default operations

???

<a name="Rc-=default"></a>
### C.80: Use `=default` if you have to be explicit about using the default semantics

**Reason**: The compiler is more likely to get the default semantics right and you cannot implement these function better than the compiler.

**Example**:

	class Tracer {
		string message;
	public:
		Tracer(const string& m) : message{m} { cerr << "entering " << message <<'\n'; }
		~Tracer() { cerr << "exiting " << message <<'\n'; }

		Tracer(const Tracer&) = default;
		Tracer& operator=(const Tracer&) = default;
		Tracer(Tracer&&) = default;
		Tracer& operator=(Tracer&&) = default;
	};

Because we defined the destructor, we must define the copy and move operations. The `=default` is the best and simplest way of doing that.

**Example, bad**:

	class Tracer2 {
		string message;
	public:
		Tracer2(const string& m) : message{m} { cerr << "entering " << message <<'\n'; }
		~Tracer2() { cerr << "exiting " << message <<'\n'; }

		Tracer2(const Tracer2& a) : message{a.message} {}
		Tracer2& operator=(const Tracer2& a) { message=a.message; }
		Tracer2(Tracer2&& a) :message{a.message} {}
		Tracer2& operator=(Tracer2&& a) { message=a.message; }
	};

Writing out the bodies of the copy and move operations is verbose, tedious, and error-prone. A compiler does it better.

**Enforcement**: (Moderate) The body of a special operation should not have the same accessibility and semantics as the compiler-generated version, because that would be redundant


<a name="Rc-=delete"></a>
### C.81: Use `=delete` when you want to disable default behavior (without wanting an alternative)

**Reason**: In a few cases, a default operation is not desirable.

**Example**:

	class Immortal {
	public:
		~Immortal() = delete;	// do not allow destruction
		// ...
	};

	void use()
	{
		Immortal ugh;	// error: ugh cannot be destroyed
		Immortal* p = new Immortal{};
		delete p;		// error: cannot destroy *p
	}

**Example**: A `unique_ptr` can be moved, but not copied. To achieve that its copy operations are deleted. To avoid copying it is necessary to `=delete` its copy operations from lvalues:

	template <class T, class D = default_delete<T>> class unique_ptr {
	public:
		// ...
		constexpr unique_ptr() noexcept;
		explicit unique_ptr(pointer p) noexcept;
		// ...
		unique_ptr(unique_ptr&& u) noexcept;	// move constructor
		// ...
		unique_ptr(const unique_ptr&) = delete; // disable copy from lvalue
		// ...
	};

	unique_ptr<int> make();	// make "something" and return it by moving

	void f()
	{
		unique_ptr<int> pi {};
		auto pi2 {pi};		// error: no move constructor from lvalue
		auto pi3 {make()};	// OK, move: the result of make() is an rvalue
	}

**Enforcement**: The elimination of a default operation is (should be) based on the desired semantics of the class. Consider such classes suspect, but maintain a "positive list" of classes where a human has asserted that the semantics is correct.

<a name="Rc-ctor-virtual"></a>
### C.82: Don't call virtual functions in constructors and destructors

**Reason**: The function called will be that of the object constructed so far, rather than a possibly overriding function in a derived class.
This can be most confusing.
Worse, a direct or indirect call to an unimplemented pure virtual function from a constructor or destructor results in undefined behavior.

**Example; bad**:

	class base {
	public:
	    virtual void f() = 0;   // not implemented
	    virtual void g();       // implemented with base version
	    virtual void h();       // implemented with base version
	};

	class derived : public base {
	public:
		void g() override;      // provide derived implementation
	    void h() final;         // provide derived implementation

	    derived()
		{
	        f();                // BAD: attempt to call an unimplemented virtual function

			g();                // BAD: will call derived::g, not dispatch further virtually
			derived::g();       // GOOD: explicitly state intent to call only the visible version

			h();                // ok, no qualification needed, h is final
	    }
	};

Note that calling a specific explicitly qualified function is not a virtual call even if the function is `virtual`.

**See also** [factory functions](#Rc-factory) for how to achieve the effect of a call to a derived class function without risking undefined behavior.


<a name="Rc-swap"></a>
### C.83: For value-like types, consider providing a `noexcept` swap function

**Reason**: A `swap` can be handy for implementing a number of idioms, from smoothly moving objects around to implementing assignment easily to providing a guaranteed commit function that enables strongly error-safe calling code. Consider using swap to implement copy assignment in terms of copy construction. See also [destructors, deallocation, and swap must never fail]("#Re-never-fail).

**Example; good**:

    class Foo {
		// ...
    public:
        void swap(Foo& rhs) noexcept
		{
            m1.swap(rhs.m1);
            std::swap(m2, rhs.m2);
        }
    private:
        Bar m1;
        int m2;
    };

Providing a nonmember `swap` function in the same namespace as your type for callers' convenience.

    void swap(Foo& a, Foo& b)
	{
		a.swap(b);
	}

**Enforcement**:
* (Simple) A class without virtual functions should have a `swap` member function declared.
* (Simple) When a class has a `swap` member function, it should be declared `noexcept`.

<a name="Rc-swap-fail"></a>
### C.84: A `swap` function may not fail

**Reason**: `swap` is widely used in ways that are assumed never to fail and programs cannot easily be written to work correctly in the presence of a failing `swap`. The The standard-library containers and algorithms will not work correctly if a swap of an element type fails.

**Example, bad**:

	void swap(My_vector& x, My_vector& y)
	{
		auto tmp = x;	// copy elements
		x = y;
		y = tmp;
	}

This is not just slow, but if a memory allocation occur for the elements in `tmp`, this `swap` may throw and would make STL algorithms fail is used with them.

**Enforcement**: (Simple) When a class has a `swap` member function, it should be declared `noexcept`.


<a name="Rc-swap-noexcept"></a>
### C.85: Make `swap` `noexcept`

**Reason**: [A `swap` may not fail](#Rc-swap-fail).
If a `swap` tries to exit with an exception, it's a bad design error and the program had better terminate.

**Enforcement**: (Simple) When a class has a `swap` member function, it should be declared `noexcept`.


<a name="Rc-eq"></a>
### C.86: Make `==` symmetric with respect to operand types and `noexcept`

**Reason**: Assymetric treatment of operands is surprising and a source of errors where conversions are possible.
`==` is a fundamental operations and programmers should be able to use it without fear of failure.

**Example**:

	class X {
		string name;
		int number;
	};

	bool operator==(const X& a, const X& b) noexcept { return a.name==b.name && a.number==b.number; }

**Example, bad**:

	class B {
		string name;
		int number;
		bool operator==(const B& a) const { return name==a.name && number==a.number; }
		// ...
	};

`B`'s comparison accpts conversions for its second operand, but not its first.

**Note**: If a class has a failure state, like `double`'s `NaN`, there is a temptation to make a comparison against the failure state throw.
The alternative is to make two failure states compare equal and any valid state compare false against the failure state.

**Enforcement**: ???


<a name="Rc-eq-base"></a>
### C.87: Beware of `==` on base classes

**Reason**: It is really hard to write a foolproof and useful `==` for a hierarchy.

**Example, bad**:

	class B {
		string name;
		int number;
		virtual bool operator==(const B& a) const { return name==a.name && number==a.number; }
		// ...
	};

// `B`'s comparison accpts conversions for its second operand, but not its first.

	class D :B {
		char character;
		virtual bool operator==(const D& a) const { return name==a.name && number==a.number && character==a.character; }
		// ...
	};

	B b = ...
	D d = ...
	b==d;	// compares name and number, ignores d's character
	d==b;	// error: no == defined
	D d2;
	d==d2;	// compares name, number, and character
	B& b2 = d2;
	b2==d;	// compares name and number, ignores d2's and d's character

Of course there are way of making `==` work in a hierarchy, but the naive approaches do not scale

**Enforcement**: ???


<a name="Rc-lt"></a>
### C.88: Make `<` symmetric with respect to operand types and `noexcept`

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rc-hash"></a>
### C.89: Make a `hash` `noexcept`

**Reason**: ???

**Example**:

	???

**Enforcement**: ???

<a name="SS-containers"</a>
## C.con: Containers and other resource handles

A container is an object holding a sequence of objects of some type; `std::vector` is the archetypical container.
A resource handle is a class that owns a resource; `std::vector` is the typical resource handle; it's resource is its sequence of elements.

Summary of container rules:

* [C.100: Follow the STL when defining a container](#Rcon-stl)
* [C.101: Give a container value semantics](#Rcon-val)
* [C.102: Give a container move operations](#Rcon-move)
* [C.103: Give a container an initializer list constructor](#Rcon-init)
* [C.104: Give a container a default constructor that sets it to empty](#Rcon-empty)
* [C.105: Give a constructor and `Extent` constructor](#Rcon-val)
* ???
* [C.109: If a resource handle has pointer semantics, provide `*` and `->`](#rcon-ptr)

**See also**: [Resources](#SS-resources)


<a name="SS-lambdas"></a>
## C.lambdas: Function objects and lambdas

A function object is an object supplying an overloaded `()` so that you can call it.
A lambda expression (colloquially often shortened to "a lambda") is a notation for generating a function object.

Summary:

* [F.50: Use a lambda when a function won't do (to capture local variables, or to write a local function)](#Rf-capture-vs-overload)
* [F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms](#Rf-reference-capture)
* [F.53: Avoid capturing by reference in lambdas that will be used nonlocally, including returned, stored on the heap, or passed to another thread](#Rf-value-capture)
* [ES.28: Use lambdas for complex initialization, especially of `const` variables](#Res-lambda-init)



<a name="SS-hier"></a>
## C.hier: Class hierarchies (OOP)

A class hierarchy is constructed to represent a set of hierarchically organized concepts (only).
Typically base classes act as interfaces.
There are two major uses for hierarchies, often named implementation inheritance and interface inheritance.

Class hierarchy rule summary:

* [C.120: Use class hierarchies to represent concepts with inherent hierarchical structure](#Rh-domain)
* [C.121: If a base class is used as an interface, make it a pure abstract class](#Rh-abstract)
* [C.122: Use abstract classes as interfaces when complete separation of interface and implementation is needed](#Rh-separation)

Designing rules for classes in a hierarchy summary:

* [C.126: An abstract class typically doesn't need a constructor](#Rh-abstract-ctor)
* [C.127: A class with a virtual function should have a virtual destructor](#Rh-dtor)
* [C.128: Use `override` to make overriding explicit in large class hierarchies](#Rh-override)
* [C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance](#Rh-kind)
* [C.130: Redefine or prohibit copying for a base class; prefer a virtual `clone` function instead](#Rh-copy)

* [C.131: Avoid trivial getters and setters](#Rh-get)
* [C.132: Don't make a function `virtual` without reason](#Rh-virtual)
* [C.133: Avoid `protected` data](#Rh-protected)
* [C.134: Ensure all data members have the same access level](#Rh-public)
* [C.135: Use multiple inheritance to represent multiple distinct interfaces](#Rh-mi-interface)
* [C.136: Use multiple inheritance to represent the union of implementation attributes](#Rh-mi-implementation)
* [C.137: Use `virtual` bases to avoid overly general base classes](#Rh-vbase)
* [C.138: Create an overload set for a derived class and its bases with `using`](#Rh-using)

Accessing objects in a hierarchy rule summary:

* [C.145: Access polymorphic objects through pointers and references](#Rh-poly)
* [C.146: Use `dynamic_cast` where class hierarchy navigation is unavoidable](#Rh-dynamic_cast)
* [C.147: Use `dynamic_cast` to a reference type when failure to find the required class is considered an error](#Rh-ptr-cast)
* [C.148: Use `dynamic_cast` to a pointer type when failure to find the required class is considered a valid alternative](#Rh-ref-cast)
* [C.149: Use `unique_ptr` or `shared_ptr` to avoid forgetting to `delete` objects created using `new`](#Rh-smart)
* [C.150: Use `make_unique()` to construct objects owned by `unique_ptr`s or another smart pointer](#Rh-make_unique)
* [C.151: Use `make_shared()` to construct objects owned by `shared_ptr`s](#Rh-make_shared)
* [C.152: Never assign a pointer to an array of derived class objects to a pointer to its base](#Rh-array)


<a name="Rh-domain"></a>
### C.120: Use class hierarchies to represent concepts with inherent hierarchical structure (only)

**Reason**: Direct representation of ideas in code eases comprehension and maintenance. Make sure the idea represented in the base class exactly matches all derived types and there is not a better way to express it than using the tight coupling of inheritance.

Do *not* use inheritance when simply having a data member will do. Usually this means that the derived type needs to override a base virtual function or needs access to a protected member.

**Example**:

	??? Good old Shape example?

**Example, bad**:
Do *not* represent non-hierarchical domain concepts as class hierarchies.

	template<typename T>
	class Container {
	public:
		// list operations:
		virtual T& get() = 0;
		virtual void put(T&) = 0;
		virtual void insert(Position) = 0;
		// ...
		// vector operations:
		virtual T& operator[](int) = 0;
		virtual void sort() = 0;
		// ...
		// tree operations:
		virtual void balance() = 0;
		// ...
	};

Here most overriding classes cannot implement most of the functions required in the interface well.
Thus the base class becomes an implementation burden.
Furthermore, the user of `Container` cannot rely on the member functions actually performing a meaningful operations reasonably efficiently;
it may throw an exception instead.
Thus users have to resort to run-time checking and/or
not using this (over)general interface in favor of a particular interface found by a run-time type inquiry (e.g., a `dynamic_cast`).

**Enforcement**:

* Look for classes with lots of members that do nothing but throw.
* Flag every use of a nonpublic base class where the derived class does not override a virtual function or access a protected base member.


<a name="Rh-abstract"></a>
### C.121: If a base class is used as an interface, make it a pure abstract class

**Reason**: A class is more stable (less brittle) if it does not contain data. Interfaces should normally be composed entirely of public pure virtual functions.

**Example**:

	???

**Enforcement**:

* Warn on any class that contains data members and also has an overridable (non-`final`) virtual function.


<a name="Rh-separation"></a?
### C.122: Use abstract classes as interfaces when complete separation of interface and implementation is needed

**Reason**: Such as on an ABI (link) boundary.

**Example**:

	???

**Enforcement**: ???



## C.hierclass: Designing classes in a hierarchy:


<a name="Rh-abstract-ctor"></a>
### C.126: An abstract class typically doesn't need a constructor

**Reason**: An abstract class typically does not have any data for a constructor to initialize.

**Example**:

	???

**Exceptions**:
* A base class constructor that does work, such as registering an object somewhere, may need a constructor.
* In extremely rare cases, you might find a reasonable for an abstract class to have a bit of data shared by all derived classes
(e.g., use statistics data, debug information, etc.); such classes tend to have constructors. But be warned: Such classes also tend to be prone to requiring virtual inheritance.

**Enforcement**: Flag abstract classes with constructors.


<a name="Rh-dtor"></a>
### C.127: A class with a virtual function should have a virtual destructor

**Reason**: A class with a virtual function is usually (and in general) used via a pointer to base, including that the last user has to call delete on a pointer to base, often via a smart pointer to base.

**Example, bad**:

	struct B {
		// ... no destructor ...
	};

	stuct D : B {		// bad: class with a resource derived from a class without a virtual destructor
		string s {"default"};
	};

	void use()
	{
		B* p = new B;
		delete p;	// leak the string
	}

**Note**: There are people who don't follow this rule because they plan to use a class only through a `shared_ptr`: `std::shared_ptr<B> p = std::make_shared<D>(args);` Here, the shared pointer will take care of deletion, so no leak will occur from and inappropriate `delete` of the base. People who do this consistently can get a false positive, but the rule is important -- what if one was allocated using `make_unique`? It's not safe unless the author of `B` ensures that it can never be misused, such as by making all constructors private and providing a factory functions to enforce the allocation with `make_shared`.

**Enforcement**:

* Flag a class with a virtual function and no virtual destructor. Note that this rule needs only be enforced for the first (base) class in which it occurs, derived classes inherit what they need. This flags the place where the problem arises, but can give false positives.
* Flag `delete` of a class with a virtual function but no virtual destructor.


<a name="Rh-override"></a>
### C.128: Use `override` to make overriding explicit in large class hierarchies

**Reason**: Readability. Detection of mistakes. Explicit `override` allows the compiler to catch mismatch of types and/or names between base and derived classes.

**Example, bad**:

	struct B {
		void f1(int);
		virtual void f2(int);
		virtual void f3(int);
		// ...
	};

	struct D : B {
		void f1(int);		// warn: D::f1() hides B::f1()
		void f2(int);		// warn: no explicit override
		void f3(double);	// warn: D::f3() hides B::f3()
		// ...
	};

**Enforcement**:

* Compare names in base and derived classes and flag uses of the same name that does not override.
* Flag overrides without `override`.


<a name="Rh-kind"><a>
### C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance

**Reason**: ??? Herb: I've become a non-fan of implementation inheritance -- seems most often an antipattern. Are there reasonable examples of it?

**Example**:

	???

**Enforcement**: ???


<a name="Rh-copy"></a>
### C.130: Redefine or prohibit copying for a base class; prefer a virtual `clone` function instead

**Reason**: Copying a base is usually slicing. If you really need copy semantics, copy deeply: Provide a virtual `clone` function that will copy the actual most-derived type, and in derived classes return the derived type (use a covariant return type).

**Example**:

	class base {
	public:
	    virtual base* clone() =0;
	};

	class derived : public base {
	public:
	    derived* clone() override;
	};

Note that because of language rules, the covariant return type cannot be a smart pointer.

**Enforcement**:

* Flag a class with a virtual function and a non-user-defined copy operation.
* Flag an assignment of base class objects (objects of a class from which another has been derived).


<a name="Rh-get"></a>
### C.131: Avoid trivial getters and setters

**Reason**: A trivial getter or setter adds no semantic value; the data item could just as well be `public`.

**Example**:

	class point {
	    int x;
		int y;
	public:
	    point(int xx, int yy) : x{xx}, y{yy} { }
	    int get_x() { return x; }
		void set_x(int xx) { x = xx; }
	    int get_y() { return y; }
		void set_y(int yy) { y = yy; }
		// no behavioral member functions
	};

Consider making such a class a `struct` -- that is, a behaviorless bunch of variables, all public data and no member functions.

	struct point {
		int x = 0;
		int y = 0;
	};

**Note**: A getter or a setter that converts from an internal type to an interface type is not trivial (it provides a form of information hiding).

**Enforcement**: Flag multiple `get` and `set` member functions that simply access a member without additional semantics.


<a name="Rh-virtual"></a>
### C.132: Don't make a function `virtual` without reason

**Reason**: Redundant `virtual` increases run-time and object-code size.
A virtual function can be overridden and is thus open to mistakes in a derived class.
A virtual function ensures code replication in a templated hierarchy.

**Example, bad**:

	template<class T>
	class Vector {
	public:
		// ...
		virtual int size() const { return sz; }	// bad: what good could a derived class do?
	private:
		T* elem;	// the elements
		int sz; 	// number of elements
	};

This kind of "vector" isn't meant to be used as a base class at all.

**Enforcement**:

* Flag a class with virtual functions but no derived classes.
* Flag a class where all member functions are virtual and have implementations.


<a name="Rh-protected"></a>
### C.133: Avoid `protected` data

**Reason**: `protected` data is a source of complexity and errors.
`protected` data complicated the statement of invariants.
`protected` data inherently violates the guidance against putting data in base classes, which usually leads to having to deal virtual inheritance as well.

**Example**:

	???

**Note**: Protected member function can be just fine.

**Enforcement**: Flag classes with `protected` data.


<a name="Rh-public"></a>
### C.134: Ensure all data members have the same access level

**Reason**: If they don't, the type is confused about what it's trying to do. Only if the type is not really an abstraction, but just a convenience bundle to group individual variables with no larger behavior (a behaviorless bunch of variables), make all data members `public` and don't provide functions with behavior. Otherwise, the type is an abstraction, so make all its data members `private`. Don't mix `public` and `private` data.

**Example**:

	???

**Enforcement**: Flag any class that has data members with different access levels.


<a name="Rh-mi-interface"></a>
### C.135: Use multiple inheritance to represent multiple distinct interfaces

**Reason**: Not all classes will necessarily support all interfaces, and not all callers will necessarily want to deal with all operations. Especially to break apart monolithic interfaces into "aspects" of behavior supported by a given derived class.

**Example**:

	???

**Note**: This is a very common use of inheritance because the need for multiple different interfaces to an implementation is common
and such interfaces are often not easily or naturally organized into a single-rooted hierarchy.

**Note**: Such interfaces are typically abstract classes.

**Enforcement**: ???


<a name="Rh-mi-implementation"></a>
### C.136: Use multiple inheritance to represent the union of implementation attributes

**Reason**: ??? Herb: Here's the second mention of implementation inheritance. I'm very skeptical, even of single implementation inheritance, never mind multiple implementation inheritance which just seems frightening -- I don't think that even policy-based design really needs to inherit from the policy types. Am I missing some good examples, or could we consider discouraging this as an anti-pattern?

**Example**:

	???

**Note**: This a relatively rare use because implementation can often be organized into a single-rooted hierarchy.

**Enforcement**: ??? Herb: How about opposite enforcement: Flag any type that inherits from more than one non-empty base class?


<a name="Rh-vbase"></a>
### C.137: Use `virtual` bases to avoid overly general base classes

**Reason**: ???

**Example**:

	???

**Note**: ???

**Enforcement**: ???

<a name="Rh-using"></a>
### C.138: Create an overload set for a derived class and its bases with `using`

**Reason**: ???

**Example**:

	???


## C.hier-access: Accessing objects in a hierarchy


<a name="Rh-poly"></a>
### C.145: Access polymorphic objects through pointers and references

**Reason**: If you have a class with a virtual function, you don't (in general) know which class provided the function to be used.

**Example**:

	struct B { int a; virtual f(); };
	struct D { int b; override f(); };

	void use(B b)
	{
		D d;
		B b2 = d;	// slice
		B b3 = b;
	}

	void use2()
	{
		D d;
		use(b);	// slice
	}

Both `d`s are sliced.

**Exeption**: You can safely access a named polymorphic object in the scope of its definition, just don't slice it.

	void use3()
	{
		D d;
		d.f();	// OK
	}

**Enforcement**: Flag all slicing.


<a name="Rh-dynamic_cast"></a>
### C.146: Use `dynamic_cast` where class hierarchy navigation is unavoidable

**Reason**: `dynamic_cast` is checked at run time.

**Example**:

	struct B {	// an interface
		virtual void f();
		virtual void g();
	};

	struct D : B {	// a wider interface
		void f() override;
		virtual void h();
	};

	void user(B* pb)
	{
		if (D* pd = dynamic_cast<D*>(pb)) {
			// ... use D's interface ...
		}
		else {
			// .. make do with B's interface ...
		}
	}

**Note**: Like other casts, `dynamic_cast` is overused.
[Prefer virtual functions to casting](#???).
Prefer [static polymorphism](#???) to hierarchy navigation where it is possible (no run-time resolution necessary)
and reasonably convenient.

**Exception**: If your implementation provided a really slow `dynamic_cast`, you may have to use a workaround.
However, all workarounds that cannot be statically resolved involve explicit casting (typically `static_cast`) and are error-prone.
You will basically be crafting your own special-purpose `dynamic_cast`.
So, first make sure that your `dynamic_cast` really is as slow as you think it is (there are a fair number of unsupported rumors about)
and that your use of `dynamic_cast` is really performance critical.

**Enforcement**: Flag all uses of `static_cast` for downcasts, including C-style casts that perform a `static_cast`.


<a name="Rh-ptr-cast"></a>
### C.147: Use `dynamic_cast` to a reference type when failure to find the required class is considered an error

**Reason**: Casting to a reference expresses that you intend to end up with a valid object, so the cast must succeed. `dynamic_cast` will then throw if it does not succeed.

**Example**:

	???

**Enforcement**: ???


<a name="Rh-ref-cast"></a>
### C.148: Use `dynamic_cast` to a pointer type when failure to find the required class is considered a valid alternative

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rh-smart"></a>
### C.149: Use `unique_ptr` or `shared_ptr` to avoid forgetting to `delete` objects created using `new`

**Reason**: Avoid resource leaks.

**Example**:

	void use(int i)
	{
		auto p = new int {7};			// bad: initialize local pointers with new
		auto q = make_unique<int>(9);	// ok: guarantee the release of the memory allocated for 9
		if(0<i) return;	// maybe return and leak
		delete p;		// too late
	}

**Enforcement**:

* Flag initialization of a naked pointer with the result of a `new`
* Flag `delete` of local variable


<a name="Rh-make_unique"></a>
### C.150: Use `make_unique()` to construct objects owne by `unique_ptr`s or other smart pointers

**Reason**: `make_unique` gives a more concise statement of the construction.

**Example**:

	unique_ptr<Foo> p {new<Foo>{7});	// OK: but repetitive

	auto q = make_unique<Foo>(7);		// Better: no repetition of Foo

**Enforcement**:

* Flag the repetitive usage of template specialization list `<Foo>`
* Flag variables declared to be `unique_ptr<Foo>`


<a name="Rh-make_shared"></a>
### C.151: Use `make_shared()` to construct objects owned by `shared_ptr`s

**Reason**: `make_shared` gives a more concise statement of the construction.
It also gives an opportunity to eliminate a separate allocation for the reference counts, by placing the `shared_ptr`'s use counts next to its object.

**Example**:

	shared_ptr<Foo> p {new<Foo>{7});	// OK: but repetitive; and separate allocations for the Foo and shared_ptr's use count

	auto q = make_shared<Foo>(7);		// Better: no repetition of Foo; one object

**Enforcement**:

* Flag the repetive usage of template specialization list`<Foo>`
* Flag variables declared to be `shared_ptr<Foo>`


<a name="Rh-array"></a>
### C.152: Never assign a pointer to an array of derived class objects to a pointer to its base

**Reason**: Subscripting the resulting base pointer will lead to invalid object access and probably to memory corruption.

**Example**:

	struct B { int x; };
	struct D : B { int y; };

	void use(B*);

	D a[] = { {1,2}, {3,4}, {5,6} };
	B* p = a;	// bad: a decays to &a[0] which is converted to a B*
	p[1].x = 7;	// overwrite D[0].y

	use(a);		// bad: a decays to &a[0] which is converted to a B*

**Enforcement**:

* Flag all combinations of array decay and base to derived conversions.
* Pass an array as an `array_view` rather than as a pointer, and don't let the array name suffer a derived-to-base conversion before getting into the `array_view`


<a name="SS-overload"></a>
# C.over: Overloading and overloaded operators

You can overload ordinary functions, template functions, and operators.
You cannot overload function objects.

Overload rule summary:

* [C.160: Define operators primarily to mimic conventional usage](#Ro-conventional)
* [C.161: Use nonmember functions for symmetric operators](#Ro-symmetric)
* [C.162: Overload operations that are roughly equivalent](#Ro-equivalent)
* [C.163: Overload only for operations that are roughly equivalent](#Ro-equivalent-2)
* [C.164: Avoid conversion operators](#Ro-conversion)
* [C.170: If you feel like overloading a lambda, use a generic lambda](#Ro-lambda)

<a name="Ro-conventional"></a>
### C.140: Define operators primarily to mimic conventional usage

**Reason**: Minimize surprises.

**Example, bad**:

	X operator+(X a, X b) { return a.v-b.v; }	// bad: makes + subtract

???. Non-member operators: namespace-level definition (traditional?) vs friend definition (as used by boost.operator, limits lookup to ADL only)

**Enforcement**: Possibly impossible.


<a name="Ro-symmetric"></a>
### C.141: Use nonmember functions for symmetric operators

**Reason**: If you use member functions, you need two.
Unless you use a non-member function for (say) `==`, `a==b` and `b==a` will be subtly different.

**Example**:

	bool operator==(Point a, Point b) { return a.x==b.x && a.y==b.y; }

**Enforcement**: Flag member operator functions.


<a name="Ro-equivalent"></a>
### C.142: Overload operations that are roughly equivalent

**Reason**: Having different names for logically equivalent operations on different argument types is confusing, leads to encoding type information in function names, and inhibits generic programming.

**Example**: Consider

	void print(int a);
	void print(int a, int base);
	void print(const string&);

These three functions all prints their arguments (appropriately). Conversely

	void print_int(int a);
	void print_based(int a, int base);
	void print_string(const string&);

These three functions all prints their arguments (appropriately). Adding to the name just introduced verbosity and inhibits generic code.

**Enforcement**: ???


<a name="Ro-equivalent-2"></a>
### C.143: Overload only for operations that are roughly equivalent

**Reason**: Having the same name for logically different functions is confusing and leads to errors when using generic programming.

**Example**: Consider

	void open_gate(Gate& g);	// remove obstacle from garage exit lane
	void fopen(const char*name, const char* mode);	// open file

The two operations are fundamentally different (and unrelated) so it is good that their names differ. Conversely:

	void open(Gate& g);	// remove obstacle from garage exit lane
	void open(const char*name, const char* mode ="r");	// open file

The two operations are still fundamentally different (and unrelated) but the names have been reduced to their (common) minimum, opening opportunities for confusion.
 Fortunately, the type system will catch many such mistakes.

**Note**: be particularly careful about common and popular names, such as `open`, `move`, `+`, and `==`.

**Enforcement**: ???


<a name="Ro-conversion"></a>
### C.144: Avoid conversion operators

**Reason**: Implicit conversions can be essential (e.g., `double` to '`int`) but often cause surprises (e.g., `String` to C-style string).

**Note**: Prefer explicitly named conversions until a serious need is demonstracted.
By "serious need" we mean a reason that is fundamental in the application domain (such as an integer to complex number conversion)
and frequently needed. Do not introduce implicit conversions (through conversion operators or non-`explicit` constructors)
just to gain a minor convenience.

**Example, bad**:

	class String {	// handle ownership and access to a sequence of characters
		// ...
		String(czstring p); // copy from *p to *(this->elem)
		// ...
		operator zstring() { return elem; }
		// ...
	};

	void user(zstring p)
	{
		if (*p=="") {
			String s {"Trouble ahead!"};
			// ...
			p = s;
		}
		// use p
	}

The string allocated for `s` and assigned to `p` is destroyed before it can be used.

**Enforcement**: Flag all conversion operators.

<a name="Ro-lambda"></a>
### C.170: If you feel like overloading a lambda, use a generic lambda

**Reason**: You can overload by defining two different lambdas with the same name

**Example**:

	void f(int);
	void f(double);
	auto f = [](char);	// error: cannot overload variable and function

	auto g = [](int) { /* ... */ };
	auto g = [](double) { /* ... */ };	// error: cannot overload variables

	auto h = [](auto) { /* ... */ };	// OK

**Enforcement**: The compiler catches attempt to overload a lambda.


<a name="SS-union"></a>
## C.union: Unions

???

Union rule summary:

* [C.180: Use `union`s to ???](#Ru-union)
* [C.181: Avoid "naked" `union`s](#Ru-naked)
* [C.182: Use anonymous `union`s to implement tagged unions](#Ru-anonymous)
* ???


<a name="Ru-union"></a>
### C.180: Use `union`s to ???

??? When should unions be used, if at all? What's a good future-proof way to re-interpret object representations of PODs?
??? variant

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Ru-naked"></a>
### C.181: Avoid "naked" `union`s

**Reason**: Naked unions are a source of type errors.

**Alternative**: Wrap them in a class together with a type field.

**Alternative**: Use `variant`.

**Example**:

	???

**Enforcement**: ???



<a name="Ru-anonymous"></a>
### C.182: Use anonymous `union`s to implement tagged unions

**Reason**: ???

**Example**:

	???

**Enforcement**: ???
