# <a name="S-class"></a> C: 클래스와 클래스 계층 구조
클래스는 사용자 정의 타입으로써, 프로그래머가 표현과 동작, 인터페이스를 정의할 수 있다.
클래스 계층 구조는 관련된 클래스들을 계층 구조로 구조화 할 때 사용된다.

클래스 규칙 요약:
* [C.1: 관련된 데이터를 구조를 사용해서 조직화 하라 (`struct`s or `class`es)](#Rc-org)
* [C.2: 클래스가 invariant 하다면 `class` 를 사용하고; 데이터 멤버가 독립적으로 달라질 수 있으면 `struct` 를 사용하라](#Rc-struct)
* [C.3: 클래스를 사용할 때 인터페이스 인지 구현인지 분명하게 표현하라](#Rc-interface)
* [C.4: 클래스의 표현에 직접 접근 할 필요가 있는 경우만 함수를 멤버로 만들어라](#Rc-member)
* [C.5: 헬퍼 함수들은 그들이 도와주는 클래스와 같은 네임스페이스에 두어라](#Rc-member)
* [C.6: 객체의 상태를 수정하지 않는 멤버 함수는 `const` 로 선언하라](#Rc-const)

하위 영역:
* [C.concrete: 구체적인 타입](#SS-concrete)
* [C.ctor: 생성자, 할당, 소멸자](#SS-ctor)
* [C.con: 컨테이너와 다른 리소스 핸들](#SS-containers)
* [C.lambdas: 함수 객체와 람다](#SS-lambdas)
* [C.hier: 클래스 계층구조 (OOP)](SS-hier)
* [C.over: 오버로딩과 오버로딩된 연산자](#SS-overload)
* [C.union: 유니온](#SS-union)

> # <a name="S-class"></a>C: Classes and Class Hierarchies
A class is a user-defined type, for which a programmer can define the representation, operations, and interfaces.
Class hierarchies are used to organize related classes into hierarchical structures.
>
Class rule summary:
* [C.1: Organize related data into structures (`struct`s or `class`es)](#Rc-org)
* [C.2: Use `class` if the class has an invariant; use `struct` if the data members can vary independently](#Rc-struct)
* [C.3: Represent the distinction between an interface and an implementation using a class](#Rc-interface)
* [C.4: Make a function a member only if it needs direct access to the representation of a class](#Rc-member)
* [C.5: Place helper functions in the same namespace as the class they support](#Rc-helper)
* [C.7: Don't define a class or enum and declare a variable of its type in the same statement](#Rc-standalone)
* [C.8: use `class` rather that `struct` if any member is non-public](#Rc-class)
* [C.9: minimize exposure of members](#Rc-private)
>
Subsections:
* [C.concrete: Concrete types](#SS-concrete)
* [C.ctor: Constructors, assignments, and destructors](#S-ctor)
* [C.con: Containers and other resource handles](#SS-containers)
* [C.lambdas: Function objects and lambdas](#SS-lambdas)
* [C.hier: Class hierarchies (OOP)](#SS-hier)
* [C.over: Overloading and overloaded operators](#SS-overload)
* [C.union: Unions](#SS-union)




### <a name="Rc-org"></a> C.1: 관련된 데이터를 구조를 사용해서 조직화 하라 (`struct`s or `class`es)

##### 근거: 
이해하기 쉽다. 근본적인 이유로 데이터가 관련이 있다면, 코드에 반영되어야 한다.

##### 예:

	void draw(int x, int y, int x2, int y2);	// BAD: unnecessary implicit relationships
	void draw(Point from, Point to)				// better

##### 참고 사항: 
가상 함수가 없는 간단한 클래스는 공간, 시간적인 오버헤드가 없다는 것을 의미한다.

##### 참고 사항: 
언어적인 관점에서 볼 때 `class` 와 `struct` 는 멤버의 기본적인 가시성만 다르다.

##### 시행하기: 
아마도 불가능하다. 데이터 항목들에 대한 경험적인 관점을 함께 사용하는 것은 가능할 것이다.


> ### <a name="Rc-org"></a> C.1: Organize related data into structures (`struct`s or `class`es)
>
##### Reason
Ease of comprehension. If data is related (for fundamental reasons), that fact should be reflected in code.
>
##### Example
```
    void draw(int x, int y, int x2, int y2);  // BAD: unnecessary implicit relationships
    void draw(Point from, Point to);          // better
```
>
##### Note
A simple class without virtual functions implies no space or time overhead.
>
##### Note
From a language perspective `class` and `struct` differ only in the default visibility of their members.
>
##### Enforcement
Probably impossible. Maybe a heuristic looking for data items used together is possible.




### <a name="Rc-struct"></a> C.2: 클래스가 invariant 하다면 `class` 를 사용하고; 데이터 멤버가 독립적으로 달라질 수 있으면 `struct` 를 사용하라

##### 근거: 
이해하기 쉽다. 프로그래머가 `class` 를 사용함으로써, invariant 가 필요하다는 것을 알게 된다.

##### 참고 사항: 
invariant 는 객체 멤버들의 논리적인 상태로써, 공개 멤버 함수들이 가정할 수 있도록 생성자가 설정 해 주어야 한다. invariant 가 설정된 후에 (일반적으로 생성자에 의해) 모든 멤버 함수는 객체를 통해 호출될 수 있다. invariant 는 형식에 구애받지 않고 (예. 주석) 기술될 수 있으며, 더 형식을 갖춘다면 `Expects` 를 사용한다.

##### 예:

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

##### 시행하기: 
모든 데이터가 비공개인 `struct` 와 모든 멤버가 공개인 `class` 들을 찾아보라.


> ### <a name="Rc-struct"></a>C.2: Use `class` if the class has an invariant; use `struct` if the data members can vary independently
>
##### Reason
Readability.
Ease of comprehension.
The use of `class` alerts the programmer to the need for an invariant.
This is a useful convention.
>
##### Note
An invariant is a logical condition for the members of an object that a constructor must establish for the public member functions to assume.
After the invariant is established (typically by a constructor) every member function can be called for the object.
An invariant can be stated informally (e.g., in a comment) or more formally using `Expects`.
If all data members can vary independently of each other, no invariant is possible.
>
##### Example
```
    struct Pair {  // the members can vary independently
        string name;
        int volume;
    };
```
> but:
```
    class Date {
    public:
        Date(int yy, Month mm, char dd);    // validate that {yy, mm, dd} is a valid date and initialize
        // ...
    private:
        int y;
        Month m;
        char d;    // day
    };
```
>   
##### Note
If a class has any `private` data, a user cannot completely initialize an object without the use of a constructor.
Hence, the class definer will provide a constructor and must specify its meaning.
This effectively means the definer need to define an invariant.
* See also [define a class with private data as `class`](#Rc-class).
* See also [Prefer to place the interface first in a class](#Rl-order).
* See also [minimize exposure of members](#Rc-private).
* See also [Avoid `protected` data](#Rh-protected).
>
##### Enforcement
Look for `struct`s with all data private and `class`es with public members.




### <a name="Rc-interface"></a> C.3: 클래스를 사용할 때 인터페이스 인지 구현인지 분명하게 표현하라

##### 근거: 
인터페이스와 구현에 대한 명시적인 구분은 가독성을 더 좋게 하고, 유지 보수를 단순하게 한다.

##### 예:

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

##### 참고 사항: 
인터페이스와 구현간의 구분을 표현하기 위해 클래스를 사용하는 것이 유일한 방법은 아니다.
예를 들면, 인터페이스를 표현하기 위한 개념으로 네임스페이스 안에 독립적인 함수들이나 추상 기본 클래스 혹은 템플릿 함수들을 선언해서 사용할 수 있다.
가장 중요한 이슈는 명시적으로 인터페이스와 그것들의 "상세한" 구현을 구분하는 것이다.
이상적으로, 그리고 전형적으로 인터페이스는 그 구현보다 훨씬 더 안정적이다.

##### 시행하기: 
???


> ### <a name="Rc-interface"></a>C.3: Represent the distinction between an interface and an implementation using a class
>
##### Reason
An explicit distinction between interface and implementation improves readability and simplifies maintenance.
>
##### Example
```
    class Date {
        // ... some representation ...
    public:
        Date();
        Date(int yy, Month mm, char dd);    // validate that {yy, mm, dd} is a valid date and initialize
>
        int day() const;
        Month month() const;
        // ...
    };
```
>
For example, we can now change the representation of a `Date` without affecting its users (recompilation is likely, though).
>
##### Note
Using a class in this way to represent the distinction between interface and implementation is of course not the only way.
For example, we can use a set of declarations of freestanding functions in a namespace, an abstract base class, or a template function with concepts to represent an interface.
The most important issue is to explicitly distinguish between an interface and its implementation "details."
Ideally, and typically, an interface is far more stable than its implementation(s).
>
##### Enforcement
???




### <a name="Rc-member"></a> C.4: 클래스의 표현에 직접 접근 할 필요가 있는 경우만 함수를 멤버로 만들어라

##### 근거: 
멤버 함수간 커플링을 작게하고, 객체 상태 변경에 의해 문제가 생기는 함수를 줄이고, 표현이 변경된 후에 수정될 필요가 있는 멤버 함수의 수를 줄인다.

##### 예:

	class Date {
		// ... relatively small interface ...
	};

	// helper functions:
	Date next_weekday(Date);
	bool operator==(Date, Date);

"헬퍼 함수"는 `Date` 의 표현에 직접 접근 할 필요가 없다.

##### 참고 사항: 
이 규칙은 C++17 에서 "uniform function call" 이 들어오면 더 좋아질 것이다. ???

##### 시행하기: 
데이터 멤버를 직접 건드리지 않는 멤버 함수를 찾아보라. 문제는 데이터 멤버를 직접 건드릴 필요가 없는 많은 멤버 함수들이 실제로 그렇게 한다는 것이다.


> ### <a name="Rc-member"></a>C.4: Make a function a member only if it needs direct access to the representation of a class
>
##### Reason
Less coupling than with member functions, fewer functions that can cause trouble by modifying object state, reduces the number of functions that needs to be modified after a change in representation.
>
##### Example
```
    class Date {
        // ... relatively small interface ...
    };
>
    // helper functions:
    Date next_weekday(Date);
    bool operator==(Date, Date);
```
The "helper functions" have no need for direct access to the representation of a `Date`.
>
##### Note
This rule becomes even better if C++17 gets "uniform function call." ???
>
##### Enforcement
Look for member function that do not touch data members directly.
The snag is that many member functions that do not need to touch data members directly do.




### <a name="Rc-member"></a> C.5: 헬퍼 함수들은 그들이 도와주는 클래스와 같은 네임스페이스에 두어라

##### 근거: 
헬퍼 함수는 (보통 클래스 작성자가 제공하는) 클래스의 표현에 직접 접근할 필요가 없는 함수이며, 클래스에 대한 유용한 인터페이스 중에 하나로 볼 수 있다.
헬퍼 함수들을 같은 네임스페이스에 넣으면 클래스에 대한 관계가 명확해지고, 인자 종속적인 검색에서 발견 할 수 있게 된다.

##### 예:

	namespace Chrono { // here we keep time-related services

		class Time { /* ... */ };
		class Date { /* ... */ };

		// helper functions:
		bool operator==(Date,Date);
		Date next_weekday(Date);
		// ...
	}

##### 시행하기:
* 하나의 네임스페이스에서 인자 타입을 취하는 전역함수들을 표시해 두어라.


> ### <a name="Rc-helper"></a> C.5: Place helper functions in the same namespace as the class they support
>
##### Reason
A helper function is a function (usually supplied by the writer of a class) that does not need direct access to the representation of the class, yet is seen as part of the useful interface to the class.
Placing them in the same namespace as the class makes their relationship to the class obvious and allows them to be found by argument dependent lookup.
>
##### Example
```
    namespace Chrono { // here we keep time-related services
>
        class Time { /* ... */ };
        class Date { /* ... */ };
>
        // helper functions:
        bool operator==(Date, Date);
        Date next_weekday(Date);
        // ...
    }
```
>   
##### Note
This is expecially important for [overloaded operators](#Ro-namespace).
>
##### Enforcement
* Flag global functions taking argument types from a single namespace.




### <a name="Rc-standalone"></a>C.7: Don't define a class or enum and declare a variable of its type in the same statement

##### Reason
Mixing a type definition and the definition of another entity in the same declaration is confusing and unnecessary.

##### Example; bad
```
    struct Data { /*...*/ } data{ /*...*/ };
```
##### Example; good
```
    struct Data { /*...*/ };
    Data data{ /*...*/ };
```
##### Enforcement
* Flag if the `}` of a class or enumeration definition is not followed by a `;`. The `;` is missing.




### <a name="Rc-class"></a>C.8: use `class` rather that `struct` if any member is non-public

##### Reason
Readability.
To make it clear that something is being hidden/abstracted.
This is a useful convention.

##### Example, bad
```
    struct Date {
        int d,m;

        Date(int i, Month m);
        // ... lots of functions ...
    private:
        int y;  // year
    };
```
There is nothing wrong with this code as far as the C++ language rules are concerned,
but nearly everything is wrong from a design perspective.
The private data is hidden far from the public data.
The data is split in different parts of the class declaration.
Different parts of the data has difference access.
All of this decreases readability and complicates maintenance.

##### Note
Prefer to place the interface first in a class [see](#Rl-order).

##### Enforcement
Flag classes declared with `struct` if there is a `private` or `public` member.




### <a name="Rc-private"></a>C.9: minimize exposure of members

##### Reason
Encapsulation.
Information hiding.
Minimize the chance of untended access.
This simplifies maintenance.

##### Example
```
    ???
```
##### Note
Prefer the order `public` members before `protected` members before `private` members [see](#Rl-order).

##### Enforcement
???




## <a name="SS-concrete"></a> C.concrete: 구체적인 타입

이상적인 클래스는 기본 타입이 되는 것이다.
대략 "`int` 처럼 동작하는 것"을 의미한다. 구체적인 타입은 가장 간단한 종류의 클래스이다.
기본 타입의 값은 복사 될 수 있고, 복사이 결과는 원본과 같은 값을 갖는 독립적인 객체이다.
구체적인 타입이 `=` 와 `==` 를 둘다 갖는다면, `a=b` 는 `a==b` 일 때 참이 될 것이다.
할당과 동등연산이 없는 구체적인 타입을 정의할 수 있지만, 이렇게 하는 것은 드물다. (그래야 한다.)
C++ 의 내장 타입들은 규칙적이며, 표준 라이브러리의 클래스인 `string`, `vector`, `map` 도 그렇다.
구체적인 타입들은 계층구조의 일부로 사용되는 클래스와 구분하기 위해 종종 값 타입으로 언급되기도 한다.

구체적인 타입 규칙 요약:

* [C.10: 복잡한 클래스들 보다는 구체적인 타입을 선호하라](#Rc-concrete)
* [C.11: 구체적인 타입을 기본 타입처럼 동작하도록 하라](#Rc-regular)


> ## <a name="SS-concrete"></a>C.concrete: Concrete types
>
One ideal for a class is to be a regular type.
That means roughly "behaves like an `int`." A concrete type is the simplest kind of class.
A value of regular type can be copied and the result of a copy is an independent object with the same value as the original.
If a concrete type has both `=` and `==`, `a=b` should result in `a == b` being `true`.
Concrete classes without assignment and equality can be defined, but they are (and should be) rare.
The C++ built-in types are regular, and so are standard-library classes, such as `string`, `vector`, and `map`.
Concrete types are also often referred to as value types to distinguish them from types uses as part of a hierarchy.
>
Concrete type rule summary:
>
* [C.10: Prefer a concrete type over more complicated classes](#Rc-concrete)
* [C.11: Make concrete types regular](#Rc-regular)




### <a name="Rc-concrete"></a> C.10 복잡한 클래스들 보다는 구체적인 타입을 선호하라

##### 근거: 
구체적인 타입은 근본적으로 계층구조 보다 단순하다:
디자인이 더 쉽고, 구현이 더 쉽고, 사용하기가 더 쉬우며, 추론하기 더 쉽다. 더 작고 더 빠르기도 하다.

##### 예

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

##### 참고 사항: 
구체적인 타입은 스택에 할당 될 수 있고, 다른 클래스의 멤버가 될 수 있다.

##### 참고 사항: 
실행시간 다형적 인터페이스를 위해 간접처리는 필수적이다.
할당과 해제의 추가비용은 그렇지 않다. (단지 가장 흔한 사례일 뿐이다)
패생 클래스의 scoped object 에 대한 인터페이스로 베이스 클래스를 사용할 수 있다.
동적 할당을 할 수 없으며, 플러그인과 같은 것들에게 안정적인 인터페이스를 제공하고자 할 때 이렇게 할 수 있다. (예, 하드 리얼타임)

##### 시행하기: 
???


> ### <a name="Rc-concrete"></a>C.10 Prefer a concrete type over more complicated classes
>
##### Reason
A concrete type is fundamentally simpler than a hierarchy:
easier to design, easier to implement, easier to use, easier to reason about, smaller, and faster.
You need a reason (use cases) for using a hierarchy.

> 
##### Example
```
    class Point1 {
        int x, y;
        // ... operations ...
        // ... no virtual functions ...
    };
>
    class Point2 {
        int x, y;
        // ... operations, some virtual ...
        virtual ~Point2();
    };
>
    void use()
    {
        Point1 p11 {1, 2};   // make an object on the stack
        Point1 p12 {p11};    // a copy
>
        auto p21 = make_unique<Point2>(1, 2);   // make an object on the free store
        auto p22 = p21.clone();                 // make a copy
        // ...
    }
```
>
If a class can be part of a hierarchy, we (in real code if not necessarily in small examples) must manipulate its objects through pointers or references.
That implies more memory overhead, more allocations and deallocations, and more run-time overhead to perform the resulting indirections.
>
##### Note
Concrete types can be stack allocated and be members of other classes.
>
##### Note
The use of indirection is fundamental for run-time polymorphic interfaces.
The allocation/deallocation overhead is not (that's just the most common case).
We can use a base class as the interface of a scoped object of a derived class.
This is done where dynamic allocation is prohibited (e.g. hard real-time) and to provide a stable interface to some kinds of plug-ins.
>
##### Enforcement
???


<a name="Rc-regular"></a>
### C.11: 구체적인 타입을 기본 타입처럼 동작하도록 하라]

##### 근거: 
기본 타입은 그렇지 못한 타입보다 이해하거나 추론하기 쉽다 (변칙적인 것들은 이해하고 사용하는데 추가적인 노력을 필요로 한다)

##### 예:

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

##### 시행하기: 
???


> ### <a name="Rc-regular"></a>C.11: Make concrete types regular
>
##### Reason
Regular types are easier to understand and reason about than types that are not regular (irregularities requires extra effort to understand and use).
>
##### Example
```
    struct Bundle {
        string name;
        vector<Record> vr;
    };
>
    bool operator==(const Bundle& a, const Bundle& b) { return a.name == b.name && a.vr == b.vr; }
>
    Bundle b1 { "my bundle", {r1, r2, r3}};
    Bundle b2 = b1;
    if (!(b1 == b2)) error("impossible!");
    b2.name = "the other bundle";
    if (b1 == b2) error("No!");
```
>
In particular, if a concrete type has an assignment also give it an equals operator so that `a=b` implies `a == b`.
>
##### Enforcement
???




## <a name="SS-ctor"></a> C.ctor: 생성자, 할당, 소멸자

이 함수들은 객체의 생명주기를 제어 한다: 생성, 복사, 이동, 그리고 파괴
생성자를 정의해서 클래스의 초기화를 보장하고 단순화 하라.

*기본적으로 있는 연산들*:

* 기본 생성자: `X()`
* 복사 생성자: `X(const X&)`
* 복사 할당자: `operator=(const X&)`
* 이동 생성자: `X(X&&)`
* 이동 할당자: `operator=(X&&)`
* 소멸자: `~X()`

각 연산들이 사용될 경우 컴파일러가 정의 해주는데, 컴파일러가 생성하지 않도록 할 수도 있다.

컴파일러가 생성하는 기본 연산들은 객체의 생명주기를 의미하는 연산들의 집합이다.
기본적으로, C++은 클래스를 값과 같은 타입으로 다루지만 모든 타입이 값 타입은 아니다.


> ## <a name="S-ctor"></a>C.ctor: Constructors, assignments, and destructors
>
These functions control the lifecycle of objects: creation, copy, move, and destruction.
Define constructors to guarantee and simplify initialization of classes.
>
These are *default operations*:
* a default constructor: `X()`
* a copy constructor: `X(const X&)`
* a copy assignment: `operator=(const X&)`
* a move constructor: `X(X&&)`
* a move assignment: `operator=(X&&)`
* a destructor: `~X()`
>
By default, the compiler defines each of these operations if it is used, but the default can be suppressed.  
The default operations are a set of related operations that together implement the lifecycle semantics of an object.  
By default, C++ treats classes as value-like types, but not all types are value-like.  


기본 연산들의 규칙들:
* [C.20: 기본 연산을 정의하지 않아도 되면 그렇게 하라](#Rc-zero)
* [C.21: 기본 연산을 정의 하거나 `=delete` 로 선언 한다면, 모두 정의하거나 모두 `=delete` 로 선언하라](#Rc-five)
* [C.22: 기본 연산들을 일관성 있도록 하라](#Rc-matched)

>
Set of default operations rules:
* [C.20: If you can avoid defining any default operations, do](#Rc-zero)
* [C.21: If you define or `=delete` any default operation, define or `=delete` them all](#Rc-five)
* [C.22: Make default operations consistent](#Rc-matched)


소멸자 규칙들:
* [C.30: 객체가 없어질 때, 명시적인 동작이 필요할 경우 소멸자를 정의하라](#Rc-dtor)
* [C.31: 클래스에 의해 얻어진 모든 리로스는 소멸자에서 해제되어야 한다](#Rc-dtor-release)
* [C.32: 클래스가 포인터(`T*`)나 참조(`T&`)를 갖고 있을 때, 소유하고 있는 것인지 고려해 보라](#Rc-dtor-ptr)
* [C.33: 클래스가 포인터 멤버를 소유하고 있다면, 파과자를 정의하거나 `=delete` 로 선언하라](#Rc-dtor-ptr)
* [C.34: 클래스가 참조 멤버를 소유하고 있다면, 파과자를 정의하거나 `=delete` 로 선언하라](#Rc-dtor-ref)
* [C.35: 가상 함수를 갖는 기본 클래스는 가상 소멸자가 필요하다](#Rc-dtor-virtual)
* [C.36: 소멸자는 실패하지 않을 것이다](#Rc-dtor-fail)
* [C.37: 소멸자를 `noexcept`로 하라](#Rc-dtor-noexcept)

>
Destructor rules:
* [C.30: Define a destructor if a class needs an explicit action at object destruction](#Rc-dtor)
* [C.31: All resources acquired by a class must be released by the class's destructor](#Rc-dtor-release)
* [C.32: If a class has a raw pointer (`T*`) or reference (`T&`), consider whether it might be owning](#Rc-dtor-ptr)
* [C.33: If a class has an owning pointer member, define or `=delete` a destructor](#Rc-dtor-ptr2)
* [C.34: If a class has an owning reference member, define or `=delete` a destructor](#Rc-dtor-ref)
* [C.35: A base class with a virtual function needs a virtual destructor](#Rc-dtor-virtual)
* [C.36: A destructor may not fail](#Rc-dtor-fail)
* [C.37: Make destructors `noexcept`](#Rc-dtor-noexcept)


생성자 규칙들:
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

>
Constructor rules:
* [C.40: Define a constructor if a class has an invariant](#Rc-ctor)
* [C.41: A constructor should create a fully initialized object](#Rc-complete)
* [C.42: If a constructor cannot construct a valid object, throw an exception](#Rc-throw)
* [C.43: Ensure that a class has a default constructor](#Rc-default0)
* [C.44: Prefer default constructors to be simple and non-throwing](#Rc-default00)
* [C.45: Don't define a default constructor that only initializes data members; use member initializers instead](#Rc-default)
* [C.46: By default, declare single-argument constructors `explicit`](#Rc-explicit)
* [C.47: Define and initialize member variables in the order of member declaration](#Rc-order)
* [C.48: Prefer in-class initializers to member initializers in constructors for constant initializers](#Rc-in-class-initializer)
* [C.49: Prefer initialization to assignment in constructors](#Rc-initialize)
* [C.50: Use a factory function if you need "virtual behavior" during initialization](#Rc-factory)
* [C.51: Use delegating constructors to represent common actions for all constructors of a class](#Rc-delegating)
* [C.52: Use inheriting constructors to import constructors into a derived class that does not need further explicit initialization](#Rc-inheriting)


복사와 이동 규칙들 : 
* [C.60: 복사연산을 `virtual`로 만들지 말아라. 매개변수는 `const&`로 받고, 비-`const&`로 반환하라](#Rc-copy-assignment)
* [C.61: 복사 연산은 복사를 수행해야 한다](#Rc-copy-semantic)
* [C.62: 복사 연산은 자기 대입에 안전하게 작성하라](#Rc-move-self)
* [C.63: 이동 연산은 `virtual`로 만들지 말아라, 매개변수는 `&&`를 사용하고, 비-`const&`로 반환하라](#Rc-move-assignment)
* [C.64: 이동 연산은 이동을 수행해야 하며, 원본 객체를 안전한(Valid) 상태로 남겨둬야 한다](#Rc-move-semantic)
* [C.65: 이동 연산은 자기 대입에 안전하게 작성하라](#Rc-copy-self)
* [C.66: 이동 연산은 `noexcept`로 만들어라](#Rc-move-noexcept)
* [C.67: 기본 클래스는 복사를 감추는게 낫다. 대신 복사가 요구된다면 가상 `clone`함수를 제공하라](#Rc-copy-virtual)

>
Copy and move rules:
* [C.60: Make copy assignment non-`virtual`, take the parameter by `const&`, and return by non-`const&`](#Rc-copy-assignment)
* [C.61: A copy operation should copy](#Rc-copy-semantic)
* [C.62: Make copy assignment safe for self-assignment](#Rc-move-self)
* [C.63: Make move assignment non-`virtual`, take the parameter by `&&`, and return by non-`const&`](#Rc-move-assignment)
* [C.64: A move operation should move and leave its source in a valid state](#Rc-move-semantic)
* [C.65: Make move assignment safe for self-assignment](#Rc-copy-self)
* [C.66: Make move operations `noexcept`](#Rc-move-noexcept)
* [C.67: A base class should suppress copying, and provide a virtual `clone` instead if "copying" is desired](#Rc-copy-virtual)


다른 기본 연산들에 대한 규칙 :
* [C.80: 기본 문맥(의미론)을 명시적으로 사용하려면 `=default` 키워드를 사용하라](#Rc-=default)
* [C.81: 기본 동작을 (대안을 원하지 않고) 금지하고 싶다면 `=delete`를 사용하라](#Rc-=delete)
* [C.82: 생성자 또는 소멸자에서 가상 함수를 호출하지 말아라](#Rc-ctor-virtual)
* [C.83: 값 형식 타입들에는, `noexcept` swap함수를 제공하는 것을 고려하라](#Rc-swap)
* [C.84: `swap`연산은 실패하지 않도록 한다](#Rc-swap-fail)
* [C.85: `swap`연산은 `noexcept`로 작성하라](#Rc-swap-noexcept)
* [C.86: `==`연산자는 피연산자 타입들에 대칭적이고, `noexcept`로 만들어라](#Rc-eq)
* [C.87: 기본 클래스에 있는 `==`에 주의하라](#Rc-eq-base)
* [C.88: `<` 연산자는 피연산자 타입에 대칭적으로 동작하고, `noexcept`로 작성하라](#Rc-lt)
* [C.89: `hash`는 `noexcept`로 작성하라 ](#Rc-hash)  

>
Other default operations rules:
* [C.80: Use `=default` if you have to be explicit about using the default semantics](#Rc-default)
* [C.81: Use `=delete` when you want to disable default behavior (without wanting an alternative)](#Rc-delete)
* [C.82: Don't call virtual functions in constructors and destructors](#Rc-ctor-virtual)
* [C.83: For value-like types, consider providing a `noexcept` swap function](#Rc-swap)
* [C.84: A `swap` may not fail](#Rc-swap-fail)
* [C.85: Make `swap` `noexcept`](#Rc-swap-noexcept)
* [C.86: Make `==` symmetric with respect of operand types and `noexcept`](#Rc-eq)
* [C.87: Beware of `==` on base classes](#Rc-eq-base)
* [C.89: Make a `hash` `noexcept`](#Rc-hash)






## <a name="SS-defop"></a> C.defop: 기본 연산들

기본적으로, 언어에서 기본적인 의미를 담는 기본 연산들을 제공한다.
그러나, 프로그래머는 기본적으로 제공되는 것들을 막거나 바꿀 수 있다.

> ## <a name="SS-defop"></a>C.defop: Default Operations
>
By default, the language supply the default operations with their default semantics.
However, a programmer can disable or replace these defaults.




### <a name="Rc-zero"></a> C.20: 기본 연산을 정의하지 않아도 되면 그렇게 하라

##### 근거: 
가장 단순하고, 가장 명료한 의미를 준다.

##### 예:

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

##### 참고 사항: 
"0의 규칙"으로 알려져 있다.

##### 시행하기: 
시행할 수 없더라도, 좋은 정적 분석기는 이 규칙에 맞는 가능한 개선사항들을 가르키는 패턴들을 찾을 수 있다.
예를 들면, 포인터와 크기를 멤버로 갖는 클래스가 있고 소멸자에서 그 포인터를 `delete` 한다면 아마도 `vector` 로 바꿀 수 있을 것이다.


> ### <a name="Rc-zero"></a>C.20: If you can avoid defining default operations, do
>
##### Reason
It's the simplest and gives the cleanest semantics.
>
##### Example
```
    struct Named_map {
    public:
        // ... no default operations declared ...
    private:
        string name;
        map<int, int> rep;
    };
>
    Named_map nm;        // default construct
    Named_map nm2 {nm};  // copy construct
```
>
Since `std::map` and `string` have all the special functions, no further work is needed.
>
##### Note
This is known as "the rule of zero".
>
##### Enforcement
(Not enforceable) While not enforceable, a good static analyzer can detect patterns that indicate a possible improvement to meet this rule.
For example, a class with a (pointer, size) pair of member and a destructor that `delete`s the pointer could probably be converted to a `vector`.




### <a name="Rc-five"></a> C.21: 기본 연산을 정의 하거나 `=delete` 로 선언 한다면, 모두 정의하거나 모두 `=delete` 로 선언하라

##### 근거: 
특별한 함수들의 의미는 아주 밀접하게 연관되어 있으며, 하나가 기본 제공 함수가 아니여야 한다면, 다른 것들도 수정이 필요하다.

##### 잘못된 예:

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

소멸자(여기서는, 할당해제)에 대한 "특별한 주의"가 필요하다고 한다면, 복사와 이동 할당(둘 다 묵시적으로 객체를 소멸할 것이다)이 정확하게 동작할 가능성은 적다. (여기서는, 두번 삭제를 시도할 것이다)

##### 참고 사항: 
기본 생성자를 중요하게 생각하는지에 달려있는데, 이것은 "다섯의 규칙" 혹은 "여섯의 규칙" 이라고 알려져 있다.

##### 참고 사항: 
다른 것은 정의 하더라도 기본 연산의 기본 구현이 필요하다면, `=default` 을 사용하여 해당 함수에 대한 의도를 표현하라.

##### 참고 사항: 
컴파일러는 이 규칙 대체적으로 시행하고, 이상적으로는 위반하는 것을 경고 해준다.

##### 참고 사항: 
클래스에 묵시적으로 생성된 복사 연산에 의존하는 것은 더 이상 사용되지 않는다.

**시행 하기**: (간단함) 클래스는 특별한 함수들에 대한 선언(`=delete` 도 포함하여)을 모두 갖거나 갖지 말아야 한다.


> ### <a name="Rc-five"></a>C.21: If you define or `=delete` any default operation, define or `=delete` them all
>
##### Reason
The semantics of the special functions are closely related, so if one needs to be non-default, the odds are that others need modification too.
>
##### Example, bad
```
    struct M2 {   // bad: incomplete set of default operations
    public:
        // ...
        // ... no copy or move operations ...
        ~M2() { delete[] rep; }
    private:
        pair<int, int>* rep;  // zero-terminated set of pairs
    };
>
    void use()
    {
        M2 x;
        M2 y;
        // ...
        x = y;   // the default assignment
        // ...
    }
```
>
Given that "special attention" was needed for the destructor (here, to deallocate), the likelihood that copy and move assignment (both will implicitly destroy an object) are correct is low (here, we would get double deletion).
>
##### Note
This is known as "the rule of five" or "the rule of six", depending on whether you count the default constructor.
>
##### Note
If you want a default implementation of a default operation (while defining another), write `=default` to show you're doing so intentionally for that function.
If you don't want a default operation, suppress it with `=delete`.
>
##### Note
Compilers enforce much of this rule and ideally warn about any violation.
>
##### Note
Relying on an implicitly generated copy operation in a class with a destructor is deprecated.
>
##### Enforcement
(Simple) A class should have a declaration (even a `=delete` one) for either all or none of the special functions.




<a name="Rc-matched"></a>
### C.22: 기본 연산들을 일관성 있도록 하라

##### 근거: 
기본 연산들은 개념적으로 맞춰져 있다. 그들의 의미는 내부적으로 연관 되어 있다.
사용자는 복사/이동 생성과 복사/이동 할당이 논리적으로 동일하고, 생성자와 소멸자가 리소스 관리에 대해 일관적으로 동작하며, 복사와 이동이 생성자와 소멸자가 동작하는 방식을 반영한다는 것을 기대 할 것이다.

##### 잘못된 예:

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

이 연산들은 복사 연산에 대한 의미가 일치하지 않는다. 이것은 혼란을 야기하고 버그를 만들 것이다.

##### 시행하기:
* (복잡함) 복사/이동 생성자와 이에 대응하는 복사/이동 할당 연산자는 동일한 레벨에서 동일한 멤버 변수를 변경하는 것이 좋다.
* (복잡함) 복사/이동 생성자에서 변경하는 멤버 변수들은 다른 생성자들에서도 초기화 하는 것이 좋다.
* (복잡함) 복사/이동 생성자는 멤버 변수에 대해 깊은 복사를 수행하고 나서, 소멸자는 멤버 변수를 수정하는 것이 좋다.
* (복잡함) 소멸자가 멤버 변수를 변경하면, 그 멤버 변수들은 복사/이동 생성자 혹은 할당 연산자에서 쓰여지는 것이 좋다.


> ### <a name="Rc-matched"></a>C.22: Make default operations consistent
>
##### Reason
The default operations are conceptually a matched set. Their semantics are interrelated.
Users will be surprised if copy/move construction and copy/move assignment do logically different things. Users will be surprised if constructors and destructors do not provide a consistent view of resource management. Users will be surprised if copy and move don't reflect the way constructors and destructors work.
>
##### Example, bad
```
    class Silly {   // BAD: Inconsistent copy operations
        class Impl {
            // ...
        };
        shared_ptr<Impl> p;
    public:
        Silly(const Silly& a) : p{a.p} { *p = *a.p; }   // deep copy
        Silly& operator=(const Silly& a) { p = a.p; }   // shallow copy
        // ...
    };
```
>
These operations disagree about copy semantics. This will lead to confusion and bugs.
>
##### Enforcement
* (Complex) A copy/move constructor and the corresponding copy/move assignment operator should write to the same member variables at the same level of dereference.
* (Complex) Any member variables written in a copy/move constructor should also be initialized by all other constructors.
* (Complex) If a copy/move constructor performs a deep copy of a member variable, then the destructor should modify the member variable.
* (Complex) If a destructor is modifying a member variable, that member variable should be written in any copy/move constructors or assignment operators.




<a name="SS-dtor"></a>
## C.dtor: 소멸자

소멸자가 필요한지 생각해 보는 것은 놀랍게도 디자인에 대한 강력한 질문이다.
대부분에 클래스들에 대한 답은 "아니오" 인데, 클래스가 리소스를 사용하지 않거나 소멸은 [0의 규칙](#Rc-zero)에 의해 다루어 지기 때문이다.
답이 "예" 라면, 클래스 디자인은 [다섯의 규칙](#Rc-five)을 따른다.

> ## <a name="SS-dtor"></a>C.dtor: Destructors
"Does this class need a destructor?" is a surprisingly powerful design question.
For most classes the answer is "no" either because the class holds no resources or because destruction is handled by [the rule of zero](#Rc-zero);
that is, its members can take care of themselves as concerns destruction.
If the answer is "yes", much of the design of the class follows (see [the rule of five](#Rc-five)).





### <a name="Rc-dtor"></a> C.30: 객체가 없어질 때, 명시적인 동작이 필요할 경우 소멸자를 정의하라

##### 근거: 
소멸자는 암묵적으로 객체의 생명주기의 마지막에 호출된다.
기본 소멸자로 충분하다면 그것을 사용하라.
단순하게 멤버의 소멸자를 호출하는 것이 아닌 코드가 필요할 경우 소멸자를 정의하라.

##### 예:
```
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
```
`Final_action` 의 목적은 소멸할 때 실행할 코드(보통 람다)를 얻는 것이다.

##### 참고 사항: 
사용자 정의 소멸자가 필요한 클래스를 보면, 일반적으로 두개의 카테고리가 있다.

* 리소스를 사용하는 클래스가 아직 소멸자가 없는 경우, 예, `vector` 혹은 트랜잭션 코드
* 소멸할 때 특정 코드를 실행하기 위해 사전에 만들어 둔 클래스, 추적자 혹은 `Final_action`

##### 잘못된 예:
```
	class Foo {		// bad; use the default destructor
	public:
		// ...
		~Foo() { s=""; i=0; vi.clear(); }  // clean up
	private:
		string s;
		int i;
		vector<int> vi;
	}
```
기본 소멸자가 더 잘 동작하고, 더 효과적이며, 틀리지 않는다.

##### 참고 사항: 
기본 소멸자가 필요하지만, 생성되지 않도록 되어 있다면 (예, 이동 생성자를 정의한 경우), `=default` 를 사용하라.

##### 시행하기: 
포인터나 참조와 같은 "암묵적인 자원"이 될 수 있는 것들을 찾아보라. 모든 데이터 멤버가 소멸자를 갖고 있더라도, 소멸자가 있는 클래스들을 찾아보라.


> ### <a name="Rc-dtor"></a>C.30: Define a destructor if a class needs an explicit action at object destruction
>
##### Reason
A destructor is implicitly invoked at the end of an object's lifetime.
If the default destructor is sufficient, use it.
Only define a non-default destructor if a class needs to execute code that is not already part of its members' destructors.
>
##### Example
```
    template<typename A>
    struct final_action {   // slightly simplified
        A act;
        final_action(A a) :act{a} {}
        ~final_action() { act(); }
    };
>
    template<typename A>
    final_action<A> finally(A act)   // deduce action type
    {
        return final_action<A>{act};
    }
>
    void test()
    {
        auto act = finally([]{ cout << "Exit test\n"; });  // establish exit action
        // ...
        if (something) return;   // act done here
        // ...
    } // act done here
```
>
The whole purpose of `final_action` is to get a piece of code (usually a lambda) executed upon destruction.
>
##### Note
There are two general categories of classes that need a user-defined destructor:
* A class with a resource that is not already represented as a class with a destructor, e.g., a `vector` or a transaction class.
* A class that exists primarily to execute an action upon destruction, such as a tracer or `final_action`.
>
##### Example, bad
```
    class Foo {   // bad; use the default destructor
    public:
        // ...
        ~Foo() { s = ""; i = 0; vi.clear(); }  // clean up
    private:
        string s;
        int i;
        vector<int> vi;
    };
```
>
The default destructor does it better, more efficiently, and can't get it wrong.
>
##### Note
If the default destructor is needed, but its generation has been suppressed (e.g., by defining a move constructor), use `=default`.
>
##### Enforcement
Look for likely "implicit resources", such as pointers and references. Look for classes with destructors even though all their data members have destructors.




### <a name="Rc-dtor-release"></a> C.31: 클래스에 의해 얻어진 모든 리소스는 소멸자에서 해제되어야 한다

##### 근거: 
리소스 누수를 막는다, 특히 에러 상황에서.

##### 참고사항: 
클래스로 표현되는 리소스들이 기본 연산자들만 갖고 있을 때 자동적으로 발생한다.

##### 예:
```
	class X {
		ifstream f;	// may own a file
		// ... no default operations defined or =deleted ...
	};
```
`X`의 `ifstream` 은 `X`가 소멸될 때 묵시적으로 열었던 파일을 모두 닫는다.  
`X`'s `ifstream` implicitly closes any file it may have open upon destruction of its `X`.

##### 잘못된 예:
```
	class X2 {	// bad
		FILE* f;	// may own a file
		// ... no default operations defined or =deleted ...
	};
```
`X2` 에서는 파일 핸들 누수가 생길 것이다.  

##### 참고사항: 
닫지 않은 소켓은 어떨까? 소멸자, 닫기, 정리 연산은 [실패하지 않는 것이 좋다](#Rc-dtor-fail).
그럼에도 불구 하고 발생한다면, 정말 좋은 해결책을 찾기 힘든 문제를 만나는 것이다.
초심자들은 소멸자를 작성할 때 왜 소멸자가 호출되고, 예외를 던짐으로써 "처리 거부"를 할 수 없는지 알지 못할 것이다.
[discussion](#Sd-never-fail)을 보라.
문제를 악화시키는 것은, 많은 "닫기/해제" 연산들이 재시도 할 수 없도록 되어있는 것이다.
이 문제를 풀려는 시도는 많았지만, 일반적인 해결책은 알려지지 않았다.
해결책이 없다면, 닫기/해제에 대한 실패를 디자인 오류로 간주하고 종료시키는 것을 고려해 보라.

##### 참고사항: 
클래스가 소유하고 있지 않은 객체에 대한 포인터나 참조를 갖고 있을 수 있다.
명백하게, 이 객체들은 클래스의 소멸자에서 `delete`되지 않아야 한다.
예:
```
	Preprocessor pp { /* ... */ };
	Parser p { pp, /* ... */ };
	Type_checker tc { p, /* ... */ };
```
`p`는 `pp`를 참조하지만, 소유하고 있지 않다.

##### 시행하기:
* (단순함) 클래스가 소유자인 포인터나 참조 멤버 변수를 갖고 있다면(예, 소유자는 `GSL::owner` 처럼 사용하는 것을 생각해보라), 소멸자에서 참조되는 것이 좋다.
* (어려움) 소유권에 대해 명시적으로 기술하지 않은 경우, 포인터나 참조 멤버 변수들이 소유자 인지 판단하라. (예, 생성자를 보라)


> ### <a name="Rc-dtor-release"></a>C.31: All resources acquired by a class must be released by the class's destructor
>
##### Reason
Prevention of resource leaks, especially in error cases.
>
##### Note
For resources represented as classes with a complete set of default operations, this happens automatically.
>
##### Example
```
    class X {
        ifstream f;   // may own a file
        // ... no default operations defined or =deleted ...
    };
```
>
`X`'s `ifstream` implicitly closes any file it may have open upon destruction of its `X`.
>
##### Example, bad
```
    class X2 {     // bad
        FILE* f;   // may own a file
        // ... no default operations defined or =deleted ...
    };
```
`X2` may leak a file handle.
>
##### Note
What about a sockets that won't close? A destructor, close, or cleanup operation [should never fail](#Rc-dtor-fail).
If it does nevertheless, we have a problem that has no really good solution.
For starters, the writer of a destructor does not know why the destructor is called and cannot "refuse to act" by throwing an exception.
See [discussion](#Sd-never-fail).
To make the problem worse, many "close/release" operations are not retryable.
Many have tried to solve this problem, but no general solution is known.
If at all possible, consider failure to close/cleanup a fundamental design error and terminate.
>
##### Note
A class can hold pointers and references to objects that it does not own.
Obviously, such objects should not be `delete`d by the class's destructor.
>
For example:
```
    Preprocessor pp { /* ... */ };
    Parser p { pp, /* ... */ };
    Type_checker tc { p, /* ... */ };
```
Here `p` refers to `pp` but does not own it.
>
##### Enforcement
* (Simple) If a class has pointer or reference member variables that are owners
  (e.g., deemed owners by using `gsl::owner`), then they should be referenced in its destructor.
* (Hard) Determine if pointer or reference member variables are owners when there is no explicit statement of ownership
  (e.g., look into the constructors).





### <a name="Rc-dtor-ptr"></a> C.32: 클래스가 포인터(`T*`)나 참조(`T&`)를 갖고 있을 때, 소유하고 있는 것인지 고려해 보라

##### 근거: 
소유권에 대해 특이할게 없는 코드들이 많다.

##### 예:
```
	???
```
##### 참고사항: 
`T*` 혹은 `T&` 가 소유를 나타낸다면, `소유한다는` 표시를 하라. `T*` 에 소유의 의미가 없다면 `ptr` 로 표시하는 것을 고려하라.
이것은 문서화와 분석에 도움이 될 것이다.

##### 시행하기: 
저수준 멤버 포인터나 멤버 참조의 초기화를 살펴보고, 할당이 사용되는지 보라.


> ### <a name="Rc-dtor-ptr"></a>C.32: If a class has a raw pointer (`T*`) or reference (`T&`), consider whether it might be owning
>
##### Reason
There is a lot of code that is non-specific about ownership.
>
##### Example
```
    ???
```
>
##### Note
If the `T*` or `T&` is owning, mark it `owning`. If the `T*` is not owning, consider marking it `ptr`.
This will aide documentation and analysis.
>
##### Enforcement
Look at the initialization of raw member pointers and member references and see if an allocation is used.





### <a name="Rc-dtor-ptr"></a> C.33: 클래스가 포인터 멤버를 소유하고 있다면, 소멸자를 정의하거나 `=delete` 로 선언하라

##### 근거: 
소유된 객체는 그것을 소유한 객체가 소멸될 때 `삭제`되어야 한다.

##### 예: 
포인터 멤버는 리소스일 것이다.
[`T*`는 리소스가 아니여야 한다](#Rr-ptr), 오래된 코드에서는 일반적이다.
`T*` 를 가능한 소유자라고 고려하고, 의심해보라.
```
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
```
소멸자를 정의 한다면, [모든 기본 연산들](#Rc-five)을 정의하거나 삭제해야 한다.
```
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
```
기본 복사 연산은 단지 `p1.p` 를 `p2.p` 로 복사하고, `p1.p` 가 두번 소멸되게 만들 것이다. 소유권에 대해 명시적이 되라:
```
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
```
##### 참고사항: 
가끔 소멸자를 사용하는 가장 단순한 방법은 포인터를 스마트 포인터(예, `std::unique_ptr`)로 교체하고, 컴파일러가
적절한 소멸자를 암묵적으로 호출하게 만들도록 놔두는 것이다.

##### 참고사항: 
왜 소유하고 있는 모든 포인터를 "스마트 포인터"로 사용하도록 하지 않는가?
가끔 중요하지 않는 코드 변경을 만들게 되고, ABI 에 영향을 줄 수 있다.

##### 시행하기:
* 포인터 데이터 맴버를 갖는 클래스를 의심하라.
* `owner<T>` 를 갖는 클래스는 기본 연산들을 정의 하는 것이 좋다.


> ### <a name="Rc-dtor-ptr2"></a>C.33: If a class has an owning pointer member, define a destructor
>
##### Reason
An owned object must be `deleted` upon destruction of the object that owns it.
>
##### Example
A pointer member may represent a resource.
[A `T*` should not do so](#Rr-ptr), but in older code, that's common.
Consider a `T*` a possible owner and therefore suspect.
```
    template<typename T>
    class Smart_ptr {
        T* p;   // BAD: vague about ownership of *p
        // ...
    public:
        // ... no user-defined default operations ...
    };
>
    void use(Smart_ptr<int> p1)
    {
        auto p2 = p1;   // error: p2.p leaked (if not nullptr and not owned by some other code)
    }
```
Note that if you define a destructor, you must define or delete [all default operations](#Rc-five):
```
    template<typename T>
    class Smart_ptr2 {
        T* p;   // BAD: vague about ownership of *p
        // ...
    public:
        // ... no user-defined copy operations ...
        ~Smart_ptr2() { delete p; }  // p is an owner!
    };
>
    void use(Smart_ptr<int> p1)
    {
        auto p2 = p1;   // error: double deletion
    }
```
>
The default copy operation will just copy the `p1.p` into `p2.p` leading to a double destruction of `p1.p`. Be explicit about ownership:
```
    template<typename T>
    class Smart_ptr3 {
        owner<T>* p;   // OK: explicit about ownership of *p
        // ...
    public:
        // ...
        // ... copy and move operations ...
        ~Smart_ptr3() { delete p; }
    };
>
    void use(Smart_ptr3<int> p1)
    {
        auto p2 = p1;   // error: double deletion
    }
```
>
##### Note
Often the simplest way to get a destructor is to replace the pointer with a smart pointer (e.g., `std::unique_ptr`) and let the compiler arrange for proper destruction to be done implicitly.
>
##### Note
Why not just require all owning pointers to be "smart pointers"?
That would sometimes require non-trivial code changes and may affect ABIs.
>
##### Enforcement
* A class with a pointer data member is suspect.
* A class with an `owner<T>` should define its default operations.




### <a name="Rc-dtor-ref"></a> C.34: 클래스가 참조 멤버를 소유하고 있다면, 파과자를 정의하라.

##### 근거: 
참조 멤버는 리소스를 표현할 수도 있다.
그렇지 않는 것이 좋지만, 오래된 코드에서는 일반적이다.
[포인터 멤버와 소멸자](#Rc-dtor ptr) 를 참고하라.
복사 역시 손실문제를 발생시킬 수 있다.

##### 잘못된 예:
```
	class Handle {		// Very suspect
		Shape& s;	// use reference rather than pointer to prevent rebinding
					// BAD: vague about ownership of *p
		// ...
	public:
		Handle(Shape& ss) : s{ss} { /* ... */ }
		// ...
	};
```
`Handle` 이 `Shape` 를 소멸할 책임이 있는지에 대한 문제는 <a ref="#Rc-dtor ptr">포인터의 경우</a> 와 동일하다:
`Handle` 이 `s` 로 참조되는 객체를 소유한다면, 소멸자가 있어야 한다.

##### 예:
```
	class Handle {		// OK
		owner<Shape&> s;	// use reference rather than pointer to prevent rebinding
		// ...
	public:
		Handle(Shape& ss) : s{ss} { /* ... */ }
		~Handle() { delete &s; }
		// ...
	};
```
`Handle` 이 `Shape` 를 소유하는지와는 별개로, 기본 복사 동작에 대해 의심해야 한다:
```
	Handle x {*new Circle{p1,17}};	// the Handle had better own the Circle or we have a leak
	Handle y {*new Triangle{p1,p2,p3}};
	x = y;		// the default assignment will try *x.s=*y.s
```
`x=y` 는 아주 의심스럽다.
`Triangle` 을 `Circle` 에 할당한다?
`Shape` 에 [복사 할당이 `=deleted`](#Rc-copy-virtual) 되어있지 않다면, `Triangle` 의 `Shape` 부분만 `Circle` 로 복사된다.

##### 참고사항: 
모든 소유하는 참조는 "스마트 포인터"로 대체하도록 하는 것은 어떤가?
참조를 스마트 포인터로 변경하는 것은 코드 변경을 의미한다.
아직 스마트 참조는 없다.
그것 역시 ABI 에 영향을 준다.

##### 시행하기:
* 참조 데이터 멤버를 갖는 클래스는 의심해보라.
* `owner<T>` 참조를 갖는 클래스는 기본 연산들을 정의하는 것이 좋다.


> ### <a name="Rc-dtor-ref"></a>C.34: If a class has an owning reference member, define a destructor
>
##### Reason
A reference member may represent a resource.
It should not do so, but in older code, that's common.
See [pointer members and destructors](#Rc-dtor-ptr).
Also, copying may lead to slicing.
>
##### Example, bad
```
    class Handle {  // Very suspect
        Shape& s;   // use reference rather than pointer to prevent rebinding
                    // BAD: vague about ownership of *p
        // ...
    public:
        Handle(Shape& ss) : s{ss} { /* ... */ }
        // ...
    };
```
The problem of whether `Handle` is responsible for the destruction of its `Shape` is the same as for [the pointer case](#Rc-dtor-ptr):
If the `Handle` owns the object referred to by `s` it must have a destructor.
>
##### Example
```
    class Handle {        // OK
        owner<Shape&> s;  // use reference rather than pointer to prevent rebinding
        // ...
    public:
        Handle(Shape& ss) : s{ss} { /* ... */ }
        ~Handle() { delete &s; }
        // ...
    };
```
Independently of whether `Handle` owns its `Shape`, we must consider the default copy operations suspect:
>
```
    Handle x {*new Circle{p1, 17}};  // the Handle had better own the Circle or we have a leak
    Handle y {*new Triangle{p1, p2, p3}};
    x = y;     // the default assignment will try *x.s = *y.s
```
That `x=y` is highly suspect.
Assigning a `Triangle` to a `Circle`?
Unless `Shape` has its [copy assignment `=deleted`](#Rc-copy-virtual), only the `Shape` part of `Triangle` is copied into the `Circle`.
>
##### Note
Why not just require all owning references to be replaced by "smart pointers"?
Changing from references to smart pointers implies code changes.
We don't (yet) have smart references.
Also, that may affect ABIs.
>
##### Enforcement
* A class with a reference data member is suspect.
* A class with an `owner<T>` reference should define its default operations.





### <a name="Rc-dtor-virtual"></a> C.35: 가상 함수를 갖는 기본 클래스는 가상 소멸자가 필요하다.

##### 근거: 
정의되지 않은 행위를 막는다.
애플리케이션에서 기본 클래스의 포인터로 파생된 클래스를 삭제하려고 할 때, 기본 클래스의 소멸자가 가상함수가 아니라면 결과는 정의되지 않는다.
일반적으로, 기본 클래스를 작성하는 사람은 소멸시점에 어떤 적절한 행동이 필요한지 모른다.

##### 잘못된 예:
```
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
```
##### 참고사항: 
가상함수는 파생클래스에 대한 인터페이스를 정의하며, 파생클래스를 몰라도 사용될 수 있다.
이런 인터페이스를 사용한다면, 역시 이 인터페이스를 통해서 파괴할 수도 있다.

##### 참고사항: 
소멸자는 `public` 이어야 하고 그렇지 않으면, 스택 할당과 스마트 포인터를 통한 일반적인 힙 할당이 해제되는 것을 막을 것이다. (혹은 레거시 코드에서 명시적인 `delete`)
```
	class X {
		~X();	// private destructor
		// ...
	};

	void use()
	{
		X a;						// error: cannot destroy
		auto p = make_unique<X>();	// error: cannot destroy
	}
```
##### 시행하기: 
(단순함) 어떤 가상함수라도 갖는 클래스는 가상 소멸자를 정의하는 것이 좋다.




> ### <a name="Rc-dtor-virtual"></a>C.35: A base class destructor should be either public and virtual, or protected and nonvirtual
>
##### Reason
To prevent undefined behavior.
If the destructor is public, then calling code can attempt to destroy a derived class object through a base class pointer, and the result is undefined if the base class's destructor is non-virtual.
If the destructor is protected, then calling code cannot destroy through a base class pointer and the destructor does not need to be virtual; it does need to be protected, not private, so that derived destructors can invoke it.
In general, the writer of a base class does not know the appropriate action to be done upon destruction.
>
##### Discussion
See [this in the Discussion section](#Sd-dtor).
>
##### Example, bad
```
    struct Base {  // BAD: no virtual destructor
        virtual f();
    };
>
    struct D : Base {
        string s {"a resource needing cleanup"};
        ~D() { /* ... do some cleanup ... */ }
        // ...
    };
>
    void use()
    {
        unique_ptr<Base> p = make_unique<D>();
        // ...
    } // p's destruction calls ~Base(), not ~D(), which leaks D::s and possibly more
```
>
##### Note
A virtual function defines an interface to derived classes that can be used without looking at the derived classes.
If the interface allows destroying, it should be safe to do so.
>
##### Note
A destructor must be nonprivate or it will prevent using the type :
```
    class X {
        ~X();   // private destructor
        // ...
    };
>
    void use()
    {
        X a;                        // error: cannot destroy
        auto p = make_unique<X>();  // error: cannot destroy
    }
```
>
##### Exception
We can imagine one case where you could want a protected virtual destructor: When an object of a derived type (and only of such a type) should be allowed to destroy *another* object (not itself) through a pointer to base. We haven't seen such a case in practice, though.
>
##### Enforcement
* A class with any virtual functions should have a destructor that is either public and virtual or else protected and nonvirtual.





### <a name="Rc-dtor-fail"></a> C.36: 소멸자는 실패하지 않을 것이다

##### 근거: 
일반적으로 코드를 작성하는 사람은 소멸자가 실패할 때 에러 없는 코드를 작성하는 방법을 모른다.
표준 라이브러리에서 다루는 모든 클래스들은 소멸자가 있어야 하고 예외를 던져서 빠져나가지 않도록 요구된다.

##### 예:
```
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
```
##### 참고사항: 
소멸자에서의 실패를 다루기 위해 실패할 염려가 없는 방법을 많이 고안해 왔다.
일반적인 방법이 제시되진 않았다.
이것은 정말 현실적인 문제가 될 수 있다: 예를 들면, 닫지 않은 소켓은 어떤가?
소멸자를 작성하는 사람은 왜 소멸자가 호출되고 예외를 던짐으로써 "동작을 거부하는 것"을 할 수 없는지 모른다.
<a =href="#Sd dtor">토론</a>부분을 보라.
문제를 악화시키는 것은, 많은 "닫기/해제" 연산이 재시도할 수 없게 되어있는 것이다.
전혀 가능하지 않다면, 닫기/정리에 대한 실패를 근본적인 디자인 오류로 간주하고 종료시켜라.

##### 참고사항: 
소멸자를 `noexcept`로 선언하라. 이것은 소멸자가 정상적으로 완료했거나 프로그램을 종료한다는 것을 보장한다.

##### 참고사항: 
만약 자원이 해제되지 않고 프로그램이 실패하지 않는다면, 어떤 방법으로든 시스템의 나머지 부분에서 실패 했다는 신호를 보내도록 하라.
(아마도 전역 상태를 수정함으로써 그렇게 할 수 있으며, 프로그램이 관리 될 수 있다.)
이 방식은 특별한 목적이 있고, 에러가 발생하기 쉽다는 것을 완전하게 인지하고 있어야 한다.
"내 연결은 닫히지 않을 것이다" 예제를 살펴보라.
아마도 연결의 다른 끝에 문제가 있을 수 있고, 연결의 양끝단에 관련된 코드들 만이 이 문제를 제대로 처리할 수 있다.
소멸자는 어떤 방식으로든 시스템의 책임이 있는 부분에 메세지를 보내고, 연결이 닫혔다고 보고 정상적으로 리턴할 수 있을 것이다.

##### 참고사항: 
소멸자가 실패할 수도 있는 연산을 사용한다면, 예외를 잡을 수 있고, 어떤 경우에는 성공적으로 완료할 수 있다.
(예, 예외를 던진 메커니즘과 다른 정리 메커니즘을 사용한다)

##### 시행하기: 
(단순함) 소멸자는 `noexcept`로 선언되는 것이 좋다.


> ### <a name="Rc-dtor-fail"></a>C.36: A destructor may not fail
>
##### Reason
In general we do not know how to write error-free code if a destructor should fail.
The standard library requires that all classes it deals with have destructors that do not exit by throwing.
>
##### Example
```
    class X {
    public:
        ~X() noexcept;
        // ...
    };
>
    X::~X() noexcept
    {
        // ...
        if (cannot_release_a_resource) terminate();
        // ...
    }
```
>
##### Note
Many have tried to devise a fool-proof scheme for dealing with failure in destructors.
None have succeeded to come up with a general scheme.
This can be a real practical problem: For example, what about a socket that won't close?
The writer of a destructor does not know why the destructor is called and cannot "refuse to act" by throwing an exception.
See [discussion](#Sd-dtor).
To make the problem worse, many "close/release" operations are not retryable.
If at all possible, consider failure to close/cleanup a fundamental design error and terminate.
>
##### Note
Declare a destructor `noexcept`. That will ensure that it either completes normally or terminate the program.
>
##### Note
If a resource cannot be released and the program may not fail, try to signal the failure to the rest of the system somehow
(maybe even by modifying some global state and hope something will notice and be able to take care of the problem).
Be fully aware that this technique is special-purpose and error-prone.
Consider the "my connection will not close" example.
Probably there is a problem at the other end of the connection and only a piece of code responsible for both ends of the connection can properly handle the problem.
The destructor could send a message (somehow) to the responsible part of the system, consider that to have closed the connection, and return normally.
>
##### Note
If a destructor uses operations that may fail, it can catch exceptions and in some cases still complete successfully
(e.g., by using a different clean-up mechanism from the one that threw an exception).
>
##### Enforcement
(Simple) A destructor should be declared `noexcept`.




### <a name="Rc-dtor-noexcept"></a> C.37 소멸자를 `noexcept`로 하라

##### 근거: 
[소멸자는 실패하지 않을 것이다](#Rc-dtor fail). 만약 소멸자가 예외로 인해 종료되려고 한다면, 좋지 않은 디자인 오류로 보고 종료하는 편이 더 좋다.

##### 시행하기: 
(단순함) 소멸자는 `noexcept`로 선언되는 것이 좋다.

> ### <a name="Rc-dtor-noexcept"></a>C.37: Make destructors `noexcept`
>
##### Reason
 [A destructor may not fail](#Rc-dtor-fail). If a destructor tries to exit with an exception, it's a bad design error and the program had better terminate.
>
##### Note
A destructor (either user-defined or compiler-generated) is implicitly declared `noexcept` (independently of what code is in its body) if all of the members of its class have `noexcept` destructors.
>
##### Enforcement
(Simple) A destructor should be declared `noexcept`.




## <a name="SS-ctor"></a> C.ctor: 생성자

생성자는 객체가 초기화되는(만들어지는) 방법을 정의 한다.

> ## C.ctor: Constructors
> A constuctor defined how an object is initialized (constructted).




### <a name="Rc-ctor"></a> C.40: 클래스가 불변조건을 가지면 생성자를 정의하라

##### 근거: 
이것이 생성자가 존재하는 이유이다.

##### 예:
```
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
```
가끔 생성자에서 `Ensure`로 불변조건을 표현하는 것은 좋은 아이디어 이다.

##### 참고 사항: 
생성자는 클래스가 불변조건이 아니더라도 편의를 위해 사용될 수 있다. 예:
```
	struct Rec {
		string s;
		int i {0};
		Rec(const string& ss) : s{ss} {}
		Rec(int ii) :i{ii} {}
	};

	Rec r1 {7};
	Rec r2 {"Foo bar"};
```
##### 참고 사항: 
C++11 초기화 리스트 규칙은 많은 생성자의 필요성을 제거한다. 예:
```
	struct Rec2{
		string s;
		int i;
		Rec2(const string& ss, int ii = 0} :s{ss}, i{ii} {}		// redundant
	};

	Rec r1 {"Foo",7};
	Rec r2 {"Bar};
```
`Rec2` 생성자는 중복이다.
`int`에 대한 기본값은 [멤버 초기화자](#Rc-in-class initializer)를 사용하는 편이 났다.

##### 참고 사항: 
[유효한 객체를 생성하라](#Rc-complete) and [생성자가 던지는 예외](#Rc-throw).

##### 시행하기:
* 사용자 정의 복사 연산이 있지만 소멸자가 없는 클래스를 표시해보라 (사용자 정의 복사는 클래스가 불변조건이라는 좋은 표시이다)


> ### <a name="Rc-ctor"></a>C.40: Define a constructor if a class has an invariant
>
##### Reason
That's what constructors are for.
>
##### Example
```
    class Date {  // a Date represents a valid date
                  // in the January 1, 1900 to December 31, 2100 range
        Date(int dd, int mm, int yy)
            :d{dd}, m{mm}, y{yy}
        {
            if (!is_valid(d, m, y)) throw Bad_date{};  // enforce invariant
        }
        // ...
    private:
        int d, m, y;
    };
```
It is often a good idea to express the invariant as an `Ensures` on the constructor.
>
##### Note
A constructor can be used for convenience even if a class does not have an invariant.   
For example:
```
    struct Rec {
        string s;
        int i {0};
        Rec(const string& ss) : s{ss} {}
        Rec(int ii) :i{ii} {}
    };
>
    Rec r1 {7};
    Rec r2 {"Foo bar"};
```
>
##### Note
The C++11 initializer list rule eliminates the need for many constructors. For example:
```
    struct Rec2{
        string s;
        int i;
        Rec2(const string& ss, int ii = 0) :s{ss}, i{ii} {}   // redundant
    };
>
    Rec r1 {"Foo", 7};
    Rec r2 {"Bar"};
```
The `Rec2` constructor is redundant.
Also, the default for `int` would be better done as a [member initializer](#Rc-in-class-initializer).
>
##### See also 
[construct valid object](#Rc-complete) and [constructor throws](#Rc-throw).
>
##### Enforcement
* Flag classes with user-defined copy operations but no constructor (a user-defined copy is a good indicator that the class has an invariant)




### <a name="Rc-complete"></a> C.41: 생성자는 완전히 초기화된 객체를 생성하는 것이 좋다

##### 근거: 
생성자는 클래스에 대한 불변조건을 정립한다. 클래스 사용자는 생성된 객체가 사용가능하다는 것을 가정할 수 있는 것이 좋다.

##### 잘못된 예:
```
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
```
컴파일러는 주석을 읽지 않는다.

##### 예외 사항: 
생성자만으로 유효한 객체를 쉽게 만들 수 없다면 [팩토리 함수를 사용하라](#C factory)

##### 참고 사항
생성자가 유효한 객체를 만들기 위해 자원을 얻는다면, 리소스는 [소멸자에 의해 해제](#Rc-release) 되는 것이 좋다.
생성자에서 자원을 얻고 소멸자에서 자원을 해제하는 것을 [RAII](Rr-raii) ("Resource Acquisitions Is Initialization") 라고 한다.


> ### <a name="Rc-complete"></a>C.41: A constructor should create a fully initialized object
>
##### Reason
A constructor establishes the invariant for a class. A user of a class should be able to assume that a constructed object is usable.
>
##### Example, bad
```
    class X1 {
        FILE* f;   // call init() before any other function
        // ...
    public:
        X1() {}
        void init();   // initialize f
        void read();   // read from f
        // ...
    };
>
    void f()
    {
        X1 file;
        file.read();   // crash or bad read!
        // ...
        file.init();   // too late
        // ...
    }
```
Compilers do not read comments.
>
##### Exception 
If a valid object cannot conveniently be constructed by a constructor [use a factory function](#Rc-factory).
>
##### Note
If a constructor acquires a resource (to create a valid object), that resource should be [released by the destructor](#Rc-dtor-release).
The idiom of having constructors acquire resources and destructors release them is called [RAII](#Rr-raii) ("Resource Acquisition Is Initialization").




### <a name="Rc-throw"></a> C.42: 생성자가 유효한 객체를 생성하지 못한다면, 예외를 던지도록 하라

##### 근거: 
유효하지 않은 객체를 남겨두는 것은 문제를 만드는 것이다.

##### 예:
```
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
```
##### 잘못된 예:
```
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
```
##### 참고 사항: 
변수 정의에 있어서 (예, 스택에 혹은 다른 객체의 멤버로써) 에러코드가 리턴되는 명시적인 함수 호출은 없다.
유효하지 않은 객체를 남겨두고 사용하기 전에 지속적으로 `is_valid()` 함수를 호출해야 하는 것은 번거롭고, 에러가 발생하기 쉬우며, 비효율적 이다.

##### 예외 사항: 
타이밍의 관점에서 볼 때 (추가적인 툴 지원 없이) 예외 처리가 충분하게 예측 가능하지 않은 고실시간 시스템(비행기 제어를 생각해 보라)과 같은 영역이 있다. `is_valid()` 와 같은 방법이 사용되어야 한다. 이와 같은 경우 [RAII](#Rc-raii) 를 시뮬레이션 하기 위해 지속적이고 즉시 `is_valid()` 로 체크하라.

##### 대안: 
"생성자 이후 초기화" 혹은 "두 단계 초기화"를 사용해야 한다면, 그렇게 하지 않도록 해보라. 정말 그렇게 해야 한다면 [팩토리 함수](#Rc-factory)를 검토해 보라.

##### 시행하기:
* (단순함) 모든 생성자는 모든 멤버 변수를 초기화 하는 것이 좋다. (명시적으로 생성자 호출을 위임하거나 기본 생성자를 통해)
* (알려지지 않음)  생성자가 `Ensure` 계약을 갖고 있다면, 사후 조건이 있는지 살펴보도록 하라.


> ### <a name="Rc-throw"></a>C.42: If a constructor cannot construct a valid object, throw an exception
>
##### Reason
Leaving behind an invalid object is asking for trouble.
>
##### Example
```
    class X2 {
        FILE* f;   // call init() before any other function
        // ...
    public:
        X2(const string& name)
            :f{fopen(name.c_str(), "r")}
        {
            if (f == nullptr) throw runtime_error{"could not open" + name};
            // ...
        }
>
        void read();      // read from f
        // ...
    };
>
    void f()
    {
        X2 file {"Zeno"}; // throws if file isn't open
        file.read();      // fine
        // ...
    }
```
>
##### Example, bad
```
    class X3 {     // bad: the constructor leaves a non-valid object behind
        FILE* f;   // call init() before any other function
        bool valid;
        // ...
    public:
        X3(const string& name)
            :f{fopen(name.c_str(), "r")}, valid{false}
        {
            if (f) valid = true;
            // ...
        }
>
        void is_valid() { return valid; }
        void read();   // read from f
        // ...
    };
>
    void f()
    {
        X3 file {"Heraclides"};
        file.read();   // crash or bad read!
        // ...
        if (is_valid()) {
            file.read();
            // ...
        }
        else {
            // ... handle error ...
        }
        // ...
    }
```
>
##### Note
For a variable definition (e.g., on the stack or as a member of another object) there is no explicit function call from which an error code could be returned.
Leaving behind an invalid object and relying on users to consistently check an `is_valid()` function before use is tedious, error-prone, and inefficient.
>
##### Exception
There are domains, such as some hard-real-time systems (think airplane controls) where (without additional tool support) exception handling is not sufficiently predictable from a timing perspective.
There the `is_valid()` technique must be used. In such cases, check `is_valid()` consistently and immediately to simulate [RAII](#Rr-raii).
>
##### Alternative
 If you feel tempted to use some "post-constructor initialization" or "two-stage initialization" idiom, try not to do that.
If you really have to, look at [factory functions](#Rc-factory).
>
##### Note
One reason people have used `init()` functions rather than doing the initialization work in a constructor has been to avoid code replication.
[Delegating constructors](#Rc-delegating) and [default member initialization](#Rc-in-class-initializer) do that better.
Another reason is been to delay initialization until an object is needed; the solution to that is often [not to declare a variable until it can be properly initialized](#Res-init)
>
##### Enforcement
* (Simple) Every constructor should initialize every member variable (either explicitly, via a delegating ctor call or via default construction).
* (Unknown) If a constructor has an `Ensures` contract, try to see if it holds as a postcondition.




### <a name="Rc-default0"></a> C.43: 클래스가 기본 생성자를 갖도록 하라

##### 근거: 
많은 언어나 라이브러리 설비들이 기본 생성자들에 의존한다,
예. `T a[10]` 나 `std::vector<T> v(10)` 는 기본 생성자들이 각 요소를 초기화 한다.

##### 예:
```
	class Date {
	public:
		Date();
		// ...
	};

	vector<Date> vd1(1000);	// default Date needed here
	vector<Date> vd2(1000,Date{Month::october,7,1885});	// alternative
```
자연스러은 기본 날짜는 없다, 그래서 이 예는 사소하지 않다. (대부분의 사람들에게 태초의 시간은 필요없다)
대부분의 달력 시스템에서 `{0,0,0}` 은 유효한 날짜가 아니다. 이것은 부동 소수점의 NaN 와 같은 것을 만드는 것이다.
그러나, 대부분의 현실적인 `Date` 클래스는 "첫째 날" (예. 1970년 1월 1일이 많이 쓰인다)을 갖기 때문에 이것을 기본으로 갖는 것이 보통 일반적이다.

##### 시행하기:
* 기본 생성자가 없는 클래스들을 표시하라.


> ### <a name="Rc-default0"></a>C.43: Ensure that a class has a default constructor
>
##### Reason
Many language and library facilities rely on default constructors to initialize their elements, e.g. `T a[10]` and `std::vector<T> v(10)`.
>
##### Example , bad
```
    class Date { // BAD: no default constructor
    public:
        Date(int dd, int mm, int yyyy);
        // ...
    };
>
    vector<Date> vd1(1000);   // default Date needed here
    vector<Date> vd2(1000, Date{Month::october, 7, 1885});   // alternative
```
The default constructor is only auto-generated if there is no user-declared constructor, hence it's impossible to initialize the vector `vd1` in the example above.
>
There is no "natural" default date (the big bang is too far back in time to be useful for most people), so this example is non-trivial.
`{0, 0, 0}` is not a valid date in most calendar systems, so choosing that would be introducing something like floating-point's `NaN`.
However, most realistic `Date` classes have a "first date" (e.g. January 1, 1970 is popular), so making that the default is usually trivial.
>
##### Example
```
    class Date {
    public:
        Date(int dd, int mm, int yyyy);
        Date() = default; // See also C.45
        // ...
    private:
        int dd = 1;
        int mm = 1;
        int yyyy = 1970;
        // ...
    };
>
    vector<Date> vd1(1000);
```
>
##### Note
A class with members that all have default constructors implicitly gets a default constructor:
```
    struct X {
        string s;
        vector v;
    };
>
    X x; // means X{ {}, {} }; that is the empty string and the empty vector
```
Beware that built-in types are not properly default constructed:
```
    struct X {
       string s;
       int i;
    };
>
    void f()
    {
       X x;    // x.s is initialized to the empty string; x.i is uninitialized
>
       cout << x.s << ' ' << x.i << '\n';
       ++x.i;
    }
```
Statically allocated objects of built-in types are by default initialized to `0`, but local built-in variables are not.
Beware that your compiler may default initialize local built-in variables, whereas an optimized build will not.
Thus, code like the example above may appear to work, but it relies on undefined behavior.
Assuming that you want initialization, an explicit default initialization can help:
```
    struct X {
       string s;
       int i {};   // default initialize (to 0)
    };
```
>
##### Enforcement
* Flag classes without a default constructor




### <a name="Rc-default00"></a> C.44: 단순하고 예외를 던지지 않는 기본 생성자를 선호하라

##### 근거: 
실패할 수 있는 연산없이 "기본"적인 값을 설정할 수 있다는 것은 에러 처리를 단순화 하고, 이동 연산을 추측 할 수 있도록 한다.

##### 잘못된 예:
```
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
```
이것은 일반적이고 좋지만, 에러 이후 `Vector0` 를 0으로 셋팅하는 것은 할당과 관련이 있고, 실패할 수 있다.
기본 `Vector` 를 `{new T[0],0,0}` 으로 표현하는 것 역시 낭비처럼 보인다.
예를 들면, `Vector0 v(100)` 는 100 만큼 할당하는 비용이 든다.

##### 예:
```
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
```
`Vector1{}` 를 `{nullptr,nullptr,nullptr}` 로 만드는 것은 비용이 적다, 이것은 특별한 경우이고 실행시간 체크가 필요하다.
에러를 발견하고 빈 `Vector1` 로 설정하는 것은 쉽다.

##### 시행하기:
* 예외를 던지는 기본 생성자를 표시하라.


> ### <a name="Rc-default00"></a>C.44: Prefer default constructors to be simple and non-throwing
>
##### Reason
Being able to set a value to "the default" without operations that might fail simplifies error handling and reasoning about move operations.
>
##### Example, problematic
```
    template<typename T>
    class Vector0 {   // elem points to space-elem element allocated using new
    public:
        Vector0() :Vector0{0} {}
        Vector0(int n) :elem{new T[n]}, space{elem + n}, last{elem} {}
        // ...
    private:
        own<T*> elem;
        T* space;
        T* last;
    };
```
This is nice and general, but setting a `Vector0` to empty after an error involves an allocation, which may fail.
Also, having a default `Vector` represented as `{new T[0], 0, 0}` seems wasteful.
For example, `Vector0 v(100)` costs 100 allocations.
>
##### Example
```
    template<typename T>
    class Vector1 {   // elem is nullptr or elem points to space-elem element allocated using new
    public:
        Vector1() noexcept {}   // sets the representation to {nullptr, nullptr, nullptr}; doesn't throw
        Vector1(int n) :elem{new T[n]}, space{elem + n}, last{elem} {}
        // ...
    private:
        own<T*> elem = nullptr;
        T* space = nullptr;
        T* last = nullptr;
    };
```
Using `{nullptr, nullptr, nullptr}` makes `Vector1{}` cheap, but a special case and implies run-time checks.
Setting a `Vector1` to empty after detecting an error is trivial.
>
##### Enforcement
* Flag throwing default constructors





<a name="Rc-default"></a>
### C.45: 데이터 멤버만을 초기화 하기 위해 기본 생성자를 정의하지 말라; 대신 클래스 내부 멤버 초기화자를 사용하라

##### 근거: 
클래스 내부 멤버 초기화자는 컴파일러가 작성자 대신 함수를 생성하도록 한다. 컴파일러가 생성하는 함수가 더 효율적일 수 있다.

##### 잘못된 예:
```
    class X1 { // BAD: doesn't use member initializers
        string s;
        int i;
    public:
        X1() :s{"default"}, i{1} { }
		// ...
    };
```
##### 예:
```
    class X2 {
        string s = "default";
        int i = 1;
    public:
        // use compiler-generated default constructor
		// ...
    };
```
##### 시행하기: 
(단순함) 기본 생성자는 상수로 멤버 변수들을 초기화 하는 것 보다 더 많은 것을 하는 것이 좋다.


> ### <a name="Rc-default"></a>C.45: Don't define a default constructor that only initializes data members; use in-class member initializers instead
>
##### Reason
Using in-class member initializers lets the compiler generate the function for you. The compiler-generated function can be more efficient.
>
##### Example, bad
```
    class X1 { // BAD: doesn't use member initializers
        string s;
        int i;
    public:
        X1() :s{"default"}, i{1} { }
        // ...
    };
```
>
##### Example
```
    class X2 {
        string s = "default";
        int i = 1;
    public:
        // use compiler-generated default constructor
        // ...
    };
```
>
##### Enforcement
(Simple) A default constructor should do more than just initialize member variables with constants.




<a name="Rc-explicit"></a>
### C.46: 기본적으로 하나의 인자를 받는 생성자는 `explicit` 로 선언하라

##### 근거: 
의도하지 않은 변환을 피한다.

##### 잘못된 예:
```
	class String {
		// ...
	public:
		String(int);	// BAD
		// ...
	};

	String s = 10;	// surprise: string of size 10
```
##### 예외 사항: 
생성자의 인자 타입에서 클래스 타입으로 묵시적 변환이 필요한 경우에는 `explicit` 를 사용하지 말라:

```
	class Complex {
		// ...
	public:
		Complex(double d);	// OK: we want a conversion from d to {d,0}
		// ...
	};

	Complex z = 10.7;	// unsurprising conversion
```
##### 참고 사항: 
[묵시적 변환에 대한 토론](#Ro-conversion).

##### 시행하기: 
(단순함) 하나의 인자를 받는 생성자는 `explicit` 로 선언되는 것이 좋다. 대부분의 코드에서 하나의 인자를 받는 생성자가 `explicit` 가 아닌데 좋은 경우는 드물다.
"긍정적인 목록"에 없는 모든 것에 대해 경고하라.


> ### <a name="Rc-explicit"></a>C.46: By default, declare single-argument constructors explicit
>
##### Reason
To avoid unintended conversions.
>
##### Example, bad
```
    class String {
        // ...
    public:
        String(int);   // BAD
        // ...
    };
>
    String s = 10;   // surprise: string of size 10
```
##### Exception
If you really want an implicit conversion from the constructor argument type to the class type, don't use `explicit`:
```
    class Complex {
        // ...
    public:
        Complex(double d);   // OK: we want a conversion from d to {d, 0}
        // ...
    };
>
    Complex z = 10.7;   // unsurprising conversion
```
>
##### See also 
[Discussion of implicit conversions](#Ro-conversion).
>
##### Enforcement
(Simple) Single-argument constructors should be declared `explicit`. Good single argument non-`explicit` constructors are rare in most code based. Warn for all that are not on a "positive list".




### <a name="Rc-order"></a> C.47: 멤버 변수들은 선언된 순서대로 정의하고 초기화 하라

##### 근거: 
혼란과 에러를 최소화 한다. 선언된 순서가 초기화가 발생하는 순서이다. (멤버 초기화자의 순서와 독립적임)

##### 잘못된 예:
```
	class Foo {
		int m1;
		int m2;
	public:
		Foo(int x) :m2{x}, m1{++x} { }	// BAD: misleading initializer order
		// ...
	};

	Foo x(1); // surprise: x.m1==x.m2==2
```
##### 시행하기: 
(단순함) 멤버 초기화자는 그들이 선언된 순서데로 멤버들을 기술해야 한다.

##### 참고 사항: 
[토론](#Sd order)


> ### <a name="Rc-order"></a>C.47: Define and initialize member variables in the order of member declaration
>
##### Reason
To minimize confusion and errors. That is the order in which the initialization happens (independent of the order of member initializers).
>
##### Example, bad
```
    class Foo {
        int m1;
        int m2;
    public:
        Foo(int x) :m2{x}, m1{++x} { }   // BAD: misleading initializer order
        // ...
    };
>
    Foo x(1); // surprise: x.m1 == x.m2 == 2
```
>
##### Enforcement
(Simple) A member initializer list should mention the members in the same order they are declared.
>
##### See also
[Discussion](#Sd-order)





### <a name="Rc-in-class-initializer"></a> C.48: 생성자에서 항상 동일하게 초기화 되는 멤버는 클래스내 초기화자를 선호하라

##### 근거: 
모든 생성자에서 같은 값이 예상되는 경우를 분명하게 하라. 반복을 피하라. 유지보수 문제를 피하라. 이것은 가장 짧고 효율적인 코드를 작성하도록 한다.

##### 잘못된 예:
```
	class X {	// BAD
		int i;
		string s;
		int j;
	public:
		X() :i{666}, s{"qqq"} { }	// j is uninitialized
		X(int i) :i{ii} {}			// s is "" and j is uninitialized
		// ...
	};
```
유지보수하는 사람이 어떻게 `j` 가 의도적으로 초기화 되지 않았는지 (어쨌든 나쁜 생각임) 알 것이며, 의도적으로 `s`의 기본 값으로 `""` 주었는지, 또 어떤 경우에 `qqq` 를 주는지 (대부분 확실해 버그임) 알 수 있겠는가? `j` 에 대한 문제 (멤버를 초기화 하는것을 잊는 것)는 가끔 기존 클래스에 새로운 멤버를 추가할 때 발생한다.

##### 예:
```
	class X2 {
		int i {666};
		string s {"qqq"};
		int j {0};
	public:
		X2() = default;			// all members are initialized to their defaults
		X2(int i) :i{ii} {}		// s and j initialized to their defaults
		// ...
	};
```
##### 대안: 
생성자의 기본 인자를 사용해서 부분적으로 이득을 얻을 수 있으며, 오래된 코드에서 일반적이다. 그러나 덜 명시적이고, 더 많은 인자를 넘겨야 하며 생성자가 하나 이상일 때 반복적인 작업이 필요하다:
```
	class X3 {	// BAD: inexplicit, argument passing overhead
		int i;
		string s;
		int j;
	public:
		X3(int ii = 666, const string& ss = "qqq", int jj = 0)
			:i{ii}, s{ss}, j{jj} { }		// all members are initialized to their defaults
		// ...
	};
```
##### 시행하기:
* (단순함) 모든 생성자는 모든 멤버 변수를 초기화 하는 것이 좋다. (명시적으로 생성자 호출을 위임하거나 기본 생성자를 통해서)
* (단순함) 생성자에서 기본 인자 보다 클래스 내부 초기화자를 제안하는 것이 더 적절한다.


> ### <a name="Rc-in-class-initializer"></a>C.48: Prefer in-class initializers to member initializers in constructors for constant initializers
>
##### Reason
Makes it explicit that the same value is expected to be used in all constructors. Avoids repetition. Avoids maintenance problems. It leads to the shortest and most efficient code.
>
##### Example, bad
```
    class X {   // BAD
        int i;
        string s;
        int j;
    public:
        X() :i{666}, s{"qqq"} { }   // j is uninitialized
        X(int ii) :i{ii} {}         // s is "" and j is uninitialized
        // ...
    };
```
How would a maintainer know whether `j` was deliberately uninitialized (probably a poor idea anyway) and whether it was intentional to give `s` the default value `""` in one case and `qqq` in another (almost certainly a bug)? The problem with `j` (forgetting to initialize a member) often happens when a new member is added to an existing class.
>
##### Example
```
    class X2 {
        int i {666};
        string s {"qqq"};
        int j {0};
    public:
        X2() = default;        // all members are initialized to their defaults
        X2(int ii) :i{ii} {}   // s and j initialized to their defaults
        // ...
    };
```
##### Alternative
We can get part of the benefits from default arguments to constructors, and that is not uncommon in older code. However, that is less explicit, causes more arguments to be passed, and is repetitive when there is more than one constructor:
```
    class X3 {   // BAD: inexplicit, argument passing overhead
        int i;
        string s;
        int j;
    public:
        X3(int ii = 666, const string& ss = "qqq", int jj = 0)
            :i{ii}, s{ss}, j{jj} { }   // all members are initialized to their defaults
        // ...
    };
```
>
##### Enforcement
* (Simple) Every constructor should initialize every member variable (either explicitly, via a delegating ctor call or via default construction).
* (Simple) Default arguments to constructors suggest an in-class initializer may be more appropriate.




<a name="Rc-initialize"></a>
### C.49: 생성자에서 할당 보다는 초기화를 선호하라

##### 근거: 
할당보다 명시적으로 초기화가 수행된다는 것을 기술하고, 더 우하하고 효율적일 수 있다. "값을 넣기 전에 사용하는" 에러를 방지한다.

##### 예:
```
	class A {		// Good
	    string s1;
	public:
		A() : s1{"Hello, "} { }    // GOOD: directly construct
		// ...
	};
```
##### 잘못된 예:
```
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
```


> ### <a name="Rc-initialize"></a>C.49: Prefer initialization to assignment in constructors
>
##### Reason
An initialization explicitly states that initialization, rather than assignment, is done and can be more elegant and efficient. Prevents "use before set" errors.
>
##### Example, good
```
    class A {   // Good
        string s1;
    public:
        A() : s1{"Hello, "} { }    // GOOD: directly construct
        // ...
    };
```
##### Example, bad
```
    class B {   // BAD
        string s1;
    public:
        B() { s1 = "Hello, "; }   // BAD: default constructor followed by assignment
        // ...
    };
>
    class C {   // UGLY, aka very bad
        int* p;
    public:
        C() { cout << *p; p = new int{10}; }   // accidental use before initialized
        // ...
    };
```




### <a name="Rc-factory"></a> C.50: 초기화 하는 동안 "가상함수 처리" 가 필요하다면 팩토리 함수를 사용하라

##### 근거: 
기본 클래스의 상태가 객체의 파생된 일부의 상태에 의존해야 한다면, 불완전하게 생성된 객체를 잘못 사용할 가능성을 줄이면서 가상함수(혹은 동등한)를 사용할 필요가 있다.

##### 잘못된 예:
```
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
```
##### 예:
```
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
```
생성자를 `private`으로 선언함으로써 완벽하게 생성되지 않은 객체를 노출하는 것을 피할 수 있다.
팩토리 함수 `Create()`를 제공함으로써, (힙에)생성을 편하게 만든다.

##### 참고 사항:
관습적으로 팩토리 함수는 스택이나 둘러싼 객체안쪽 보다 힙에 할당한다

##### 참고 사항: 
[토론](#Sd factory)


> ### <a name="Rc-factory"></a>C.50: Use a factory function if you need "virtual behavior" during initialization
>
##### Reason
If the state of a base class object must depend on the state of a derived part of the object, we need to use a virtual function (or equivalent) while minimizing the window of opportunity to misuse an imperfectly constructed object.
>
##### Example, bad
```  
    class B {
    public:
        B()
        {
            // ...
            f();   // BAD: virtual call in constructor
            // ...
        }
>
        virtual void f() = 0;
>
        // ...
    };
```
>
##### Example
```
    class B {
    protected:
        B() { /* ... */ }                   // create an imperfectly initialized object
>
        virtual void PostInitialize()       // to be called right after construction
        {
            // ...
            f();    // GOOD: virtual dispatch is safe
            // ...
        }
>
    public:
        virtual void f() = 0;
>
        template<class T>
        static shared_ptr<T> Create()    // interface for creating objects
        {
            auto p = make_shared<T>();
            p->PostInitialize();
            return p;
        }
    };
>
    class D : public B { /* "¦ */ };            // some derived class
>
    shared_ptr<D> p = D::Create<D>();        // creating a D object
```
>
By making the constructor `protected` we avoid an incompletely constructed object escaping into the wild.
By providing the factory function `Create()`, we make construction (on the free store) convenient.
>
##### Note
Conventional factory functions allocate on the free store, rather than on the stack or in an enclosing object.
>
##### See also
[Discussion](#Sd-factory)





### <a name="Rc-delegating"></a> C.51: 클래스의 모든 생성자들의 일반적인 동작을 표현하려면 위임 생성자를 사용하라

##### 근거: 
중복을 피하고 우연히 발생할 수 있는 잘못된 표기를 방지한다.

##### 잘못된 예:
```
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
```
일반적인 동작은 작성하기 지루하며, 뜻하지 않게 일반적이지 않을 수도 있다.  

##### 예:
```
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
```
##### 참고 사항: 
만약 "반복된 동작"이 단순한 초기화라면, [생성자에서 항상 동일하게 초기화 되는 멤버는 클래스내 초기화자를 선호하라](#Rc-in-class initializer)를 고려하라. 

##### 시행하기: 
(Moderate) 비슷한 생성자 내용을 찾아보라.


> ### <a name="Rc-delegating"></a>C.51: Use delegating constructors to represent common actions for all constructors of a class
>
##### Reason
To avoid repetition and accidental differences.
>
##### Example, bad
```
    class Date {   // BAD: repetitive
        int d;
        Month m;
        int y;
    public:
        Date(int ii, Month mm, year yy)
            :i{ii}, m{mm} y{yy}
            { if (!valid(i, m, y)) throw Bad_date{}; }
>
        Date(int ii, Month mm)
            :i{ii}, m{mm} y{current_year()}
            { if (!valid(i, m, y)) throw Bad_date{}; }
        // ...
    };
```
The common action gets tedious to write and may accidentally not be common.
>
##### Example
```
    class Date2 {
        int d;
        Month m;
        int y;
    public:
        Date2(int ii, Month mm, year yy)
            :i{ii}, m{mm} y{yy}
            { if (!valid(i, m, y)) throw Bad_date{}; }
>
        Date2(int ii, Month mm)
            :Date2{ii, mm, current_year()} {}
        // ...
    };
```
>
##### See also 
If the "repeated action" is a simple initialization, consider [an in-class member initializer](#Rc-in-class-initializer).
>
##### Enforcement
(Moderate) Look for similar constructor bodies.






### <a name="Rc-inheriting"></a> C.52: 명시적인 초기화를 필요로 하지 않는 파생 클래스에선, 상속 받는 생성자들을 사용하라.

##### 근거: 
파생 클래스를 위해서 생성자가 필요하다면, 그 생성자들을 다시 작성하는 것은 지루하며 에러에 취약하다.  

##### 예: 
`std::vector`는 교묘한 생성자들을 많가지고 있다. 자신만의 `vector`를 원한다면, 그 많은 생성자들을 다시 구현하고 싶진 않을 것이다.
```
    class Rec {
	   // ... 데이터와 많은 좋은 생성자들 ...
    };

    class Oper : public Rec {
	   using Rec::Rec;
	   // ... 데이터 멤버는 없고 ...
	   // ... 많은 보조 함수들만 있다 ...
    };
```
##### 잘못된 예:
```
    struct Rec2 : public Rec {
    	int x;
	   using Rec::Rec;
    };

    Rec2 r {"foo", 7};
    int val = r.x;	// 초기화 되지 않았다
```
##### 시행하기: 
파생된 클래스의 모든 멤버들이 반드시 초기화 되도록 하라. 


> ### <a name="Rc-inheriting"></a>C.52: Use inheriting constructors to import constructors into a derived class that does not need further explicit initialization
>
##### Reason
If you need those constructors for a derived class, re-implementing them is tedious and error prone.
>
##### Example
`std::vector` has a lot of tricky constructors, so if I want my own `vector`, I don't want to reimplement them:
```
    class Rec {
        // ... data and lots of nice constructors ...
    };
>
    class Oper : public Rec {
        using Rec::Rec;
        // ... no data members ...
        // ... lots of nice utility functions ...
    };
```
>
##### Example, bad
```
    struct Rec2 : public Rec {
        int x;
        using Rec::Rec;
    };
>
    Rec2 r {"foo", 7};
    int val = r.x;   // uninitialized
```
>
##### Enforcement
Make sure that every member of the derived class is initialized.


-----

<a name="SS-copy"></a>
## C.copy: 복사와 이동
값 타입들은 일반적으로 복사 가능해야 한다. 하지만 클래스 계층에서의 인터페이스들은 그렇지 않아야 한다.  
리소스 핸들의 경우, 복사가 가능할 수도, 그렇지 않을 수도 있다.  
타입들은 논리적인 또는 성능 상의 이유로 이동하도록 정의될 수 있다. 

> <a name="SS-copy"></a>
> ## C.copy: Copy and move
Value type should generally be copyable, but interfaces in a class hierarchy should not.
Resource handles, may or may not be copyable.
Types can be defined to move for logical as well as performance reasons.




<a name="Rc-copy-assignment"></a>
### C.60: 복사 대입은 `virtual`로 만들지 마라, 매개변수는 `const&`로 받고, 반환은 비-`const&`로 하라. 

##### 근거: 
이렇게 하는 것이 간단하고 효율적이다. r-value를 위해 최적화하길 원한다면, `&&`를 받는 대입 연산을 오버로드하여 제공하라. ([F.24](#Rf-pass-ref-ref)를 참조하라)

##### 예:
```
class Foo {
public:
	Foo& operator=(const Foo& x)
	{
		auto tmp = x;	// GOOD: 자기 대입을 위한 검사를 할 필요가 없다.(성능 말고는)
		std::swap(*this,tmp);
		return *this;
	}
	// ...
};

Foo a;
Foo b;
Foo f();

a = b;  	// l-value 대입 : 복사
a = f();	// r-value 대입 : 이동일 수도 있다
```
##### 참고 사항: 
`swap`함수의 구현은 [강한 예외 안전성 보장](???)을 가능하게 한다.

##### 예: 
하지만 만약 임시 사본을 만들지 않음으로써 굉장히 나은 성능을 얻을 수 있다면 어떨까? 
큰고 같은 크기의 `Vector`들의 대입이 빈번한 영역을 위한 간단한 `Vector`를 생각해보라.  
이 경우, `swap`구현 기법에 의한 원소들의 사본은 대규모의 비용을야기할 수 있다.
```
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
		// ... swap함수 기법을 사용한다. 이러면 최상의 구현이 된다 ...
		*return *this
	}
	// ... *a.elem으로부터 elem으로 sz만큼 원소들을 복사한다 ...
	if (a.sz<sz) {
		// ... destroy the surplus elements in *this* and adjust size ...
	}
	return *this*
}
```
대상 원소들에 직접 쓰기 연산을 함으로써, `swap`기법이 제공하는 강한 예외 보장 대신 [기본적인 예외 보장](#???)만 얻게 될 것이다. 
[자기 대입](#Rc-copy-self)에 주의하라.

##### 대안: 
만약 당신이 `virtual` 대입 연산자가 필요하다고 생각한다면, 그리고 어째서 그것이 문제를 야기할 수 있는지 이해한다면, 그 함수는 `operator=`라고 부르지 마라. 이름을 부여해서 `virtual void assign(const Foo&)`로 만들어라. 
[복사 생성 vs. `clone()`](#Rc-copy-virtual)를 참조하라. 

##### 시행하기:  
* (Simple) 대입 연산자는 가상함수여서는 안된다. 드래곤들만큼 위험하다!
* (Simple) 대입 연산자는 `T&`를 반환하면 안된다. 연쇄적인 호출을 위해선, 컨테이너로의 객체 대입과 코드 작성을 방해하는 `const T&`를 사용하지 말아라.
* (Moderate) 대입 연산자는 (암시적으로나 명시적으로나) 모든 기본 클래스와 멤버들의 대입 연산자를 호출해야 한다.  
해당 타입이 포인터 문맥이나 값 문맥을 가지는지 확인하기 위해 소멸자를 확인하라. 


> <a name="Rc-copy-assignment"></a>
> ### C.60: Make copy assignment non-`virtual`, take the parameter by `const&`, and return by non-`const&`
> **Reason**: It is simple and efficient. If you want to optimize for rvalues, provide an overload that takes a `&&` (see [F.24](#Rf-pass-ref-ref)).  
> **Example**:
>
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
>
	Foo a;
	Foo b;
	Foo f();
>
	a = b;		// assign lvalue: copy
	a = f();	// assign rvalue: potentially move

> **Note**: The `swap` implementation technique offers the [strong guarantee](???).  
> **Example**: But what if you can get significant better performance by not making a temporary copy? Consider a simple `Vector` intended for a domain where assignment of large, equal-sized `Vector`s is common. In this case, the copy of elements implied by the `swap` implementation technique could cause an order of magnitude increase in cost:
>
	template<typename T>
	class Vector {
	public:
		Vector& operator=(const Vector&);
		// ...
	private:
		T* elem;
		int sz;
	};
>
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
> By writing directly to the target elements, we will get only [the basic guarantee](#???) rather than the strong guaranteed offered by the `swap` technique. Beware of [self assignment](#Rc-copy-self).
> **Alternatives**: If you think you need a `virtual` assignment operator, and understand why that's deeply problematic, don't call it `operator=`. Make it a named function like `virtual void assign(const Foo&)`.
See [copy constructor vs. `clone()`](#Rc-copy-virtual).

> **Enforcement**:
>
* (Simple) An assignment operator should not be virtual. Here be dragons!
* (Simple) An assignment operator should return `T&` to enable chaining, not alternatives like `const T&` which interfere with composability and putting objects in containers.
* (Moderate) An assignment operator should (implicitly or explicitly) invoke all base and member assignment operators.
Look at the destructor to determine if the type has pointer semantics or value semantics.



<a name="Rc-copy-semantic"></a>
### C.61: 복사 연산은 복사를 수행해야 한다.

##### 근거: 
그렇게 하는 것이 일반적으로 생각되는 의미론이다. `x=y`가 수행된 후에는, `x==y`인 결과를 가져야 한다.
복사 후에는 `x`와 `y`가 독립적인 객체들일 수 있다. (값 의미론, 비-포인터 빌트인 타입들과 표준 라이브러리 타입들의 동작하는 방식) 또는 공유된 객체를 참조한다.(포인터 의미론, 포인터들이 동작하는 방식) 

##### 예:
```
	class X {	// OK: 값 의미론
	public:
		X();
		X(const X&);	// X를 복사한다.
		void modify();	// X의 값을 변경한다.
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
	if (x==y) throw Bad{};	// 값 의미론으로 가정한다.
```
##### 예:
```
	class X2 {	// OK: 포인터 의미론
	public:
		X2();
		X2(const X&) = default;	// 얕은 복사
		~X2() = default;
		void modify();			// X의 값을 변경한다.
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
	if (x!=y) throw Bad{};	// 포인터 의미론으로 가정한다.
```
##### 참고 사항: 
"스마트 포인터"를 만들고 있지 않다면 복사 의미론을 선호하라. 값 의미론은 가장 간단하며, 표준 라이브러리의 기능들이 기대하는 것이다.   

##### 시행하기: 
(강요할 수는 없다).

> <a name="Rc-copy-semantic"></a>
> ### C.61: A copy operation should copy
> **Reason**: That is the generally assumed semantics. After `x=y`, we should have `x==y`.  
> After a copy `x` and `y` can be independent objects (value semantics, the way non-pointer built-in types and the standard-library types work) or refer to a shared object (pointer semantics, the way pointers work).  
> **Example**:
>
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
>
	bool operator==(const X& a, const X& b)
	{
		return sz==a.sz && equal(p,p+sz,a.p,a.p+sz);
	}
>
	X::X(const X& a)
		:p{new T}, sz{a.sz}
	{
		copy(a.p,a.p+sz,a.p);
	}
>
	X x;
	X y = x;
	if (x!=y) throw Bad{};
	x.modify();
	if (x==y) throw Bad{};	// assume value semantics
> **Example**:
>
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
>
	bool operator==(const X2& a, const X2& b)
	{
		return sz==a.sz && p==a.p;
	}
>
	X2 x;
	X2 y = x;
	if (x!=y) throw Bad{};
	x.modify();
	if (x!=y) throw Bad{};	// assume pointer semantics
> **Note**: Prefer copy semantics unless you are building a "smart pointer". Value semantics is the simplest to reason about and what the standard library facilities expect.  
> **Enforcement**: (Not enforceable).


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




## C.other: 다른 기본 연산들
> ## C.other: Other default operations


<a name="Rc-=default"></a>
### C.80: 기본 문맥(의미론)을 명시적으로 사용하려면 `=default` 키워드를 사용하라 

##### 근거: 
컴파일러가 더 정확한 기본 의미론을 알고 있으며, 이보다 나은 코드를 작성할 수 없다. 

##### 예:
```
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
```
소멸자를 정의했기 때문에, 우리는 복사, 이동 연산들을 정의해야만 한다. 이를 위해선 `=default`가 가장 간단한 최선의 방법이다.  

##### 잘못된 예:
```
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
```
복사와 이동 연산들의 함수 본체를 작성하는 것은 번거롭고, 지루하며, 에러에 취약하다. 컴파일러가 이 작업을 더 잘 할수있다.

##### 시행하기: 
(Moderate) 특별한 연산들은 중복성을 피하기 위해 컴파일러가 만든 버전과 같은 접근성, 의미론을 가져서는 안된다.  

> <a name="Rc-=default"></a>
### C.80: Use `=default` if you have to be explicit about using the default semantics
> **Reason**: The compiler is more likely to get the default semantics right and you cannot implement these function better than the compiler.  
> **Example**:
>
	class Tracer {
		string message;
	public:
		Tracer(const string& m) : message{m} { cerr << "entering " << message <<'\n'; }
		~Tracer() { 
			cerr << "exiting " << message <<'\n'; }
		Tracer(const Tracer&) = default;
		Tracer& operator=(const Tracer&) = default;
		Tracer(Tracer&&) = default;
		Tracer& operator=(Tracer&&) = default;
	};
> Because we defined the destructor, we must define the copy and move operations. The `=default` is the best and simplest way of doing that.  
> **Example, bad**:
>
	class Tracer2 {
		string message;
	public:
		Tracer2(const string& m) : message{m} { cerr << "entering " << message <<'\n'; }
		~Tracer2() { 
			cerr << "exiting " << message <<'\n'; }
		Tracer2(const Tracer2& a) : message{a.message} {}
		Tracer2& operator=(const Tracer2& a) { message=a.message; }
		Tracer2(Tracer2&& a) :message{a.message} {}
		Tracer2& operator=(Tracer2&& a) { message=a.message; }
	};
> Writing out the bodies of the copy and move operations is verbose, tedious, and error-prone. A compiler does it better.  
> **Enforcement**: (Moderate) The body of a special operation should not have the same accessibility and semantics as the compiler-generated version, because that would be redundant



<a name="Rc-=delete"></a>
### C.81: 기본 동작을 (대안을 원하지 않고) 금지하고 싶다면 `=delete`를 사용하라

##### 근거: 
몇몇 경우에, 기본 연산들이 바람직하지 않기도 하다.

##### 예:
```
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
```
##### 예: `unique_ptr`는 이동 가능하지만, 복사는 불가능하다. 이 클래스의 복사를 막기 위해, 복사 연산들은 삭제된다. l-value로부터 복사 연산을 막기 위해 `=delete`가 필요하다.
```
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
```
##### 시행하기: 
기본 연산을 제거하는 것은 해당 클래스에 부합하는 근거가 있어야 한다. 의심하라. 하지만 사람이 보기에 문맥적으로 정확하다고 단정할 수 있도록 유지하라.   

> <a name="Rc-=delete"></a>
### C.81: Use `=delete` when you want to disable default behavior (without wanting an alternative)
> **Reason**: In a few cases, a default operation is not desirable.  
> **Example**:
>
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

> **Example**: A `unique_ptr` can be moved, but not copied. To achieve that its copy operations are deleted. To avoid copying it is necessary to `=delete` its copy operations from lvalues:
>
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
> **Enforcement**: The elimination of a default operation is (should be) based on the desired semantics of the class. Consider such classes suspect, but maintain a "positive list" of classes where a human has asserted that the semantics is correct.




<a name="Rc-ctor-virtual"></a>
### C.82: 생성자 또는 소멸자에서 가상 함수를 호출하지 말아라.

##### 근거: 
호출된 함수는 파생 클래스에서 오버라이드 하는 함수가 아니라, 생성된 객체의 함수이다. 이러한 동작은 혼란을 일으킬 수 있다. 나쁘게는, 생성자와 소멸자 내부에서 발생하는 구현되지 않은 순수 가상 함수에 대한 직접 또는 간접호출이 비정의된 동작을 일으킨다.  

##### 잘못된 예:
```
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
```
특정하게 명시적으로 한정된 함수는 `virtual`로 선언되었다고 하더라도 가상호출이 발생하지 않음을 기억하라.

##### 참고 사항 정의되지 않은 동작의 위험이 없이 파생 클래스의 함수를 호출하는 효과를 얻기 위해서는 [팩토리 함수들](#Rc-factory) 참고하라. 

> <a name="Rc-ctor-virtual"></a>
### C.82: Don't call virtual functions in constructors and destructors

> **Reason**: The function called will be that of the object constructed so far, rather than a possibly overriding function in a derived class.
This can be most confusing.
Worse, a direct or indirect call to an unimplemented pure virtual function from a constructor or destructor results in undefined behavior.

> **Example; bad**:
>
	class base {
	public:
	    virtual void f() = 0;   // not implemented
	    virtual void g();       // implemented with base version
	    virtual void h();       // implemented with base version
	};
>
	class derived : public base {
	public:
		void g() override;      // provide derived implementation
	    void h() final;         // provide derived implementation
>
	    derived()
		{
	        f();                // BAD: attempt to call an unimplemented virtual function
>
			g();                // BAD: will call derived::g, not dispatch further virtually
			derived::g();       // GOOD: explicitly state intent to call only the visible version
>
			h();                // ok, no qualification needed, h is final
	    }
	};
> Note that calling a specific explicitly qualified function is not a virtual call even if the function is `virtual`.

> **See also** [factory functions](#Rc-factory) for how to achieve the effect of a call to a derived class function without risking undefined behavior.



<a name="Rc-swap"></a>
### C.83: 값 형식 타입들에는, `noexcept` swap함수를 제공하는 것을 고려하라.

##### 근거: 
`swap`함수는  
객체 대입을 구현할 때 원활하게 객체를 이동하는 것에서, 에러가 발생하지 않는 것을 보장하는 함수를 제공하는 것까지 몇몇 함수들(idioms)을 구현하는데 유용하다. 
swap함수을 이용해서 복사 대입을 구현하는 것을 고려하라. [소멸자, 자원해제, 그리고 swap은 실패해선 안된다]("#Re-never-fail)를 확인하라.

##### 잘못된 예:
```
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
```
호출자들의 편의를 위해서 같은 네임스페이스에 비 멤버 `swap`함수를 제공하라.
```
void swap(Foo& a, Foo& b)
{
	a.swap(b);
}
```
##### 시행하기:
* 가상 함수들이 없는 클래스는 `swap`멤버 함수 선언이 있어야 한다. 

* 클래스가 `swap` 멤버함수를 가지고 있다면, 그 함수는 `noexcept`로 선언되어야 한다.

> <a name="Rc-swap"></a>
### C.83: For value-like types, consider providing a `noexcept` swap function

> **Reason**: A `swap` can be handy for implementing a number of idioms, from smoothly moving objects around to implementing assignment easily to providing a guaranteed commit function that enables strongly error-safe calling code. Consider using swap to implement copy assignment in terms of copy construction. See also [destructors, deallocation, and swap must never fail]("#Re-never-fail).  
> **Example; good**:
> 
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
> Providing a nonmember `swap` function in the same namespace as your type for callers' convenience.
> 
    void swap(Foo& a, Foo& b)
	{
		a.swap(b);
	}
> **Enforcement**:
> * (Simple) A class without virtual functions should have a `swap` member function declared.
> * (Simple) When a class has a `swap` member function, it should be declared `noexcept`.




<a name="Rc-swap-fail"></a>
### C.84: `swap`연산은 실패하지 않도록 한다

##### 근거: 
`swap`연산은 많은 경우 실패하지 않을 것으로 전제하고 사용된다. 또한 실패 가능성이 있는 `swap`연산으로는 정확하게 동작하도록 프로그램이 작성되기 어렵다. 표준 라이브러리의 컨테이너들과 알고리즘들은 swap연산의 타입이 실패하면 정확하게 동작하지 않을 것이다.  

##### 잘못된 예:
```
void swap(My_vector& x, My_vector& y)
{
	auto tmp = x;	// copy elements
	x = y;
	y = tmp;
}
```
이 경우는 느릴 뿐만 아니라, `tmp`내의 원소들에 메모리 할당이 발생하면, 이 `swap` 연산은 예외를 던지고 이를 사용하는 STL 알고리즘들이 실패할 수 있다. 

##### 시행하기: 
클래스에 `swap` 멤버 함수가 있으면, `noexcept`로 선언되어야 한다.   


> <a name="Rc-swap-fail"></a>
### C.84: A `swap` function may not fail
> **Reason**: `swap` is widely used in ways that are assumed never to fail and programs cannot easily be written to work correctly in the presence of a failing `swap`. The The standard-library containers and algorithms will not work correctly if a swap of an element type fails.  
> **Example, bad**:
>
	void swap(My_vector& x, My_vector& y)
	{
		auto tmp = x;	// copy elements
		x = y;
		y = tmp;
	}
> This is not just slow, but if a memory allocation occur for the elements in `tmp`, this `swap` may throw and would make STL algorithms fail is used with them.  
> **Enforcement**: (Simple) When a class has a `swap` member function, it should be declared `noexcept`.


<a name="Rc-swap-noexcept"></a>
### C.85: `swap`연산은 `noexcept`로 작성하라

##### 근거: 
[`swap`연산은 실패하지 않도록 한다](#Rc-swap-fail).
만약 `swap`연산이 예외를 던지면서 종료하면, 그것은 좋지 않은 설계 오류이며 프로그램을 종료하는게 낫다.

##### 시행하기: 
클래스에 `swap` 멤버 함수가 있으면, `noexcept`로 선언되어야 한다.   

> <a name="Rc-swap-noexcept"></a>
### C.85: Make `swap` `noexcept`
> **Reason**: [A `swap` may not fail](#Rc-swap-fail).
If a `swap` tries to exit with an exception, it's a bad design error and the program had better terminate.  
> **Enforcement**: (Simple) When a class has a `swap` member function, it should be declared `noexcept`.



<a name="Rc-eq"></a>
### C.86: `==`연산자는 피연산자 타입들에 대칭적이고, `noexcept`로 만들어라.  

##### 근거: 
피연산자들에 비대칭적인 처리는 기대에 부합하지 않고, 형변환이 가능한 경우 에러를 유발할 수 있다. 
`==`는 기본적인 연산이며 프로그래머들이 이 연산을 사용할 때 연산 실패에 대한 고민이 없어야 한다.

##### 예:
```
class X {
	string name;
	int number;
};

bool operator==(const X& a, const X& b) noexcept {
	 return a.name==b.name 
	 	&& a.number==b.number; }
```
##### 잘못된 예:
```
class B {
	string name;
	int number;
	bool operator==(const B& a) const { 
		return name==a.name 
			&& number==a.number; }
	// ...
};
```
`B`의 비교 연산은 두번째 피연산자에 대해 형변환을 용인하지만, 첫번째 피연산자에 대해서는 그렇지 않다.

##### 참고 사항: 
만약 클래스가 `double`타입의 `NaN`처럼 실패 상태를 가진다면, 실패 상태와의 비교에서 예외를 던지도록 하는 것도 적합할 수 있다.
다른 방법으로는 실패 상태끼리의 비교는 동등하게 보고, 적합한 상태와 실패 상태의 비교에서는 거짓으로 판정할 수 있다.    

##### 시행하기: 
???


> <a name="Rc-eq"></a>
### C.86: Make `==` symmetric with respect to operand types and `noexcept`

> **Reason**: Assymetric treatment of operands is surprising and a source of errors where conversions are possible.
> `==` is a fundamental operations and programmers should be able to use it without fear of failure.

> **Example**:
>
	class X {
		string name;
		int number;
	};
>
	bool operator==(const X& a, const X& b) noexcept { return a.name==b.name && a.number==b.number; }
> **Example, bad**:
>
	class B {
		string name;
		int number;
		bool operator==(const B& a) const { return name==a.name && number==a.number; }
		// ...
	};
> `B`'s comparison accpts conversions for its second operand, but not its first.
> **Note**: If a class has a failure state, like `double`'s `NaN`, there is a temptation to make a comparison against the failure state throw.
The alternative is to make two failure states compare equal and any valid state compare false against the failure state.  
> **Enforcement**: ???




<a name="Rc-eq-base"></a>
### C.87: 기본 클래스에 있는 `==`에 주의하라

##### 근거: 
계층 구조에서 잘못 사용하기 어렵고 유용한  `==`를 작성하는 것은 어려운 일이다. 

##### 잘못된 예:
```
class B {
	string name;
	int number;
	virtual bool operator==(const B& a) const {
		return name==a.name 
		&& number==a.number; }
	// ...
};

// `B`'s comparison accpts conversions for its second operand, but not its first.

class D :B {
	char character;
	virtual bool operator==(const D& a) const { 
		return name==a.name 
			&& number==a.number 
			&& character==a.character; }
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
```
물론 계층 구조 안에서 `==`가 동작하도록 하는 방법들이 있지만, 고지식한 방법들은 고려하지 말아라.  

##### 시행하기: 
???

><a name="Rc-eq-base"></a>
### C.87: Beware of `==` on base classes
> **Reason**: It is really hard to write a foolproof and useful `==` for a hierarchy.  
> **Example, bad**:
>
	class B {
		string name;
		int number;
		virtual bool operator==(const B& a) const { 
			return name==a.name 
				&& number==a.number; }
		// ...
	};
> // `B`'s comparison accpts conversions for its second operand, but not its first.
>
	class D :B {
		char character;
		virtual bool operator==(const D& a) const { 
			return name==a.name 
				&& number==a.number 
				&& character==a.character; }
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
> Of course there are way of making `==` work in a hierarchy, but the naive approaches do not scale
> **Enforcement**: ???



<a name="Rc-lt"></a>
### C.88: `<` 연산자는 피연산자 타입에 대칭적으로 동작하고, `noexcept`로 작성하라

##### 근거: 
???  
##### 예:
```
	???
```
##### 시행하기: 
???

> <a name="Rc-lt"></a>
### C.88: Make `<` symmetric with respect to operand types and `noexcept`
> **Reason**: ???  
> **Example**:  
> 
	???
> **Enforcement**: ???



<a name="Rc-hash"></a>
### C.89: `hash`는 `noexcept`로 작성하라  
##### 근거: 
???  
##### 예:
```
???
```
##### 시행하기: 
???

> <a name="Rc-hash"></a>
### C.89: Make a `hash` `noexcept`
> **Reason**: ???  
> **Example**:  
>
	???
> **Enforcement**: ???


<a name="SS-containers"></a>
## C.con: 컨테이너와 다른 리소스 핸들   
> ## C.con: Containers and other resource handles

컨테이너는 어떤 타입의 객체들을 보유하는 객체를 의미한다; `std::vector`가 대표적인 컨테이너이다.   
리소스 핸들은 리소스를 소유한하는 클래스를 의미한다. `std::vector`는 리소스 핸들에 속한다; 여기서 리소스는 벡터가 보유한 원소들의 시퀀스이다. 


> A container is an object holding a sequence of objects of some type; `std::vector` is the archetypical container.   
> A resource handle is a class that owns a resource; `std::vector` is the typical resource handle; it's resource is its sequence of elements.


컨테이너 규칙 요약
* [C.100: 컨테이너를 정의할때는 STL을 따르라](#Rcon-stl)
* [C.101: 값 문맥을 적용하라](#Rcon-val)
* [C.102: move 연산을 제공하라](#Rcon-move)
* [C.103: 초기화 리스트 생성자를 지원하라](#Rcon-init)
* [C.104: 공백 값으로 설정하는 기본 생성자를 지원하라](#Rcon-empty)
* [C.105: 생성자와 '확장' 생성자를 지원하라](#Rcon-val)
* ???
* [C.109: 포인터 문맥을 따를 경우에는, `*` 과 `->` 연산자를 제공하라](#rcon-ptr)

> Summary of container rules:
>
* [C.100: Follow the STL when defining a container](#Rcon-stl)
* [C.101: Give a container value semantics](#Rcon-val)
* [C.102: Give a container move operations](#Rcon-move)
* [C.103: Give a container an initializer list constructor](#Rcon-init)
* [C.104: Give a container a default constructor that sets it to empty](#Rcon-empty)
* [C.105: Give a constructor and `Extent` constructor](#Rcon-val)
* ???
* [C.109: If a resource handle has pointer semantics, provide `*` and `->`](#rcon-ptr)

##### 참고 사항 : [Resources](#SS-resources)
>**See also**: [Resources](#SS-resources)



<a name="SS-lambdas"></a>
## C.lambdas: 함수 객체와 람다   

함수 객체는 호출할 수 있도록 재정의된 `()`를 지원하는 객체이다.   
람다 표현식(보통 "람다"라고 축약해 부른다)은 함수 객체를 생성하는 방법이다.


> ## C.lambdas: Function objects and lambdas

> A function object is an object supplying an overloaded `()` so that you can call it.  
> A lambda expression (colloquially often shortened to "a lambda") is a notation for generating a function object. 

요약 :

* [F.50: Use a lambda when a function won't do (to capture local variables, or to write a local function)](#Rf-capture-vs-overload)
* [F.52: 지역적으로 사용되면 참조 캡쳐를 선호하라.(알고리즘으로의 전달)](#Rf-reference-capture)
* [F.53: 비-지역적으로 사용될 경우 참조 캡쳐를 지양하라.(비-지역적 사용, Heap 영역에 저장, 다른 스레드로의 전달)](#Rf-value-capture)
* [ES.28: 복잡한 초기화에, 특히 상수 변수들에 람다를 사용하라.](#Res-lambda-init)


> Summary:  
>
* [F.50: Use a lambda when a function won't do (to capture local variables, or to write a local function)](#Rf-capture-vs-overload)
* [F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms](#Rf-reference-capture)
* [F.53: Avoid capturing by reference in lambdas that will be used nonlocally, including returned, stored on the heap, or passed to another thread](#Rf-value-capture)
* [ES.28: Use lambdas for complex initialization, especially of `const` variables](#Res-lambda-init)



<a name="SS-hier"></a>

## C.hier: 클래스 계층 구조 (OOP)

클래스 계층은 계층적으로 조직된 개념들의 집합을 표현하면서 생성된다.  
일반적으로 기본 클래스들은 인터페이스로서 동작한다.  
계층 구조는 크게 2가지 용도로 사용되는데, 종종 구현 상속과 인터페이스 상속으로 명명된다.

> ## C.hier: Class hierarchies (OOP)
> A class hierarchy is constructed to represent a set of hierarchically organized concepts (only).  
> Typically base classes act as interfaces.  
> There are two major uses for hierarchies, often named implementation inheritance and interface inheritance.  

클래스 계층 구조 규칙 요약

* [C.120: 클래스 계층은 상속 계층 구조의 개념을 표현하는데 사용하라](#Rh-domain)
* [C.121: 만약 기본 클래스가 인터페이스로써 사용되면, 순수 추상 클래스로 만들어라](#Rh-abstract)
* [C.122: 인터페이스와 구현의 분리가 필요하면, 추상 클래스를 인터페이스로 사용하라.](#Rh-separation)

  
> Class hierarchy rule summary:
>
* [C.120: Use class hierarchies to represent concepts with inherent hierarchical structure](#Rh-domain)
* [C.121: If a base class is used as an interface, make it a pure abstract class](#Rh-abstract)
* [C.122: Use abstract classes as interfaces when complete separation of interface and implementation is needed](#Rh-separation)

클래스 계층 구조에서 설계 규칙 요약 : 

* [C.126: 추상클래스는 일반적으로 생성자가 필요하지 않다](#Rh-abstract-ctor)
* [C.127: 가상 함수를 가진 클래스는 가상 소멸자를 가져야 한다](#Rh-dtor)
* [C.128: 클래스 계층구조가 클 경우에는 `override`를 명시하라](#Rh-override)
* [C.129: 클래스 계층구조를 설계할 때는, 인터페이스 상속과 구현의 상속을 구분하라](#Rh-kind)
* [C.130: 기본 클래스의 복사를 재정의하거나 금지하라; 대신 가상 `clone`함수를 사용하라](#Rh-copy)

* [C.131: 사소한 getter나 setter는 지양하라.](#Rh-get)
* [C.132: 가상함수로 만들때는 이유가 있어야 한다.](#Rh-virtual)
* [C.133: `protected`데이터를 지양하라.](#Rh-protected)
* [C.134: 모든 데이터 멤버들이 같은 접근 레벨에 있도록 하라.](#Rh-public)
* [C.135: 다중 상속을 다수의 인터페이스를 표현하기 위해 사용하라.](#Rh-mi-interface)
* [C.136: 다중 상속을 구현 속성들의 합집합(union)을 표현하기 위해 사용하라.](#Rh-mi-implementation)
* [C.137: 가상 기본 클래스는 과도하게 일반적인 기본 클래스들을 피하기 위해 사용하라.](#Rh-vbase)
* [C.138: `using`을 사용해서 파생 클래스와 기본 클래스의 Overload 집합을 만들어라.](#Rh-using)

> Designing rules for classes in a hierarchy summary:
>
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

계층 구조에서 객체 접근 규칙 요약 :

* [C.145: 다형성을 지닌 객체는 포인터나 참조자를 사용해서 접근하라](#Rh-poly)
* [C.146: `dynamic_cast`는 클래스 계층 구조에서 탐색이 불가피할때 사용하라](#Rh-dynamic_cast)
* [C.147: 필요한 타입을 찾는 데 실패하는 것이 에러로 간주될 때는, `dynamic_cast`를 참조자 타입에 사용하라](#Rh-ptr-cast)
* [C.148: 필요한 클래스를 찾는데 실패하는 것이 허용가능 하다면, `dynamic_cast`를 포인터 타입에 사용하라.](#Rh-ref-cast)
* [C.149: `new`를 사용해서 생성한 객체를 `delete`하지 않는 것을 예방하기 위해, `unique_ptr` 또는 `shared_ptr`를 사용하라.](#Rh-smart)
* [C.150: `unique_ptr`나 다른 스마트포인터가 소유하는 객체를 생성하기 위해선 `make_unique()`를 사용하라.](#Rh-make_unique)
* [C.151: `shared_ptr`들에 의해 소유되는 객체를 생성하기 위해서는 `make_shared()`를 사용하라.](#Rh-make_shared)
* [C.152: 파생 클래스 객체들의 포인터 배열에 기본 클래스의 포인터를 할당해서는 절대로 안된다.](#Rh-array)

> Accessing objects in a hierarchy rule summary:
>
* [C.145: Access polymorphic objects through pointers and references](#Rh-poly)
* [C.146: Use `dynamic_cast` where class hierarchy navigation is unavoidable](#Rh-dynamic_cast)
* [C.147: Use `dynamic_cast` to a reference type when failure to find the required class is considered an error](#Rh-ptr-cast)
* [C.148: Use `dynamic_cast` to a pointer type when failure to find the required class is considered a valid alternative](#Rh-ref-cast)
* [C.149: Use `unique_ptr` or `shared_ptr` to avoid forgetting to `delete` objects created using `new`](#Rh-smart)
* [C.150: Use `make_unique()` to construct objects owned by `unique_ptr`s or another smart pointer](#Rh-make_unique)
* [C.151: Use `make_shared()` to construct objects owned by `shared_ptr`s](#Rh-make_shared)
* [C.152: Never assign a pointer to an array of derived class objects to a pointer to its base](#Rh-array)



<a name="Rh-domain"></a>
### C.120: 클래스 계층은 상속 계층 구조의 개념을 표현하는데 사용하라 (only)

##### 근거:   
코드에 생각이 직접적으로 표현되는것은 이해와 유지보수를 쉽게 한다. 기본 클래스에 표현된 생각들이 모든 파생 타입들에서도 일치하도록 하라. 그리고 그것을 표현하는데는 상속을 이용한 밀접한 커플링보다 나은 방법은 없다. 

단지 데이터멤버를 가진것 만으로 상속을 사용해서는 안된다. 보통 이것은 파생된 타입이 기본 클래스의 가상 함수를 오버라이드 하거나 `protected` 멤버에 접근하는 것이 필요함을 의미한다.

##### 예:   
??? (추가 예시 없음)

##### 잘못된 예:   
계층적이지 않은 영역(domain)의 개념을 클래스 계층으로 표현해서는 안된다.

```
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
```

여기서 오버라이드 하는 클래스들은 인터페이스가 요구하는 함수들의 대부분을 구현할 수 없다. 그렇게 되면 기본 클래스는 가상함수들을 구현할 부담을 떠안게 된다. 더해서, `Container`를 사용자는 실제로 의미있고, 타당하며, 효율적인 연산을 수행하는데 멤버함수에 의존할 수 없게된다;  
멤버함수가 연산 대신 예외를 던질수도 있으며, 그로 인해 사용자는 실행시간 검사에 의존하거나 이 (과도하게)일반적인 인터페이스를 사용하지 않고, 실행시간 타입 검사로 찾아낸 특정한 인터페이스를 사용하게 된다.(e.g., `dynamic_cast`).

##### 시행하기:  
* 아무것도 하지 않고 throw만 하는 멤버를 많이 가진 클래스들을 확인하라.
* 파생 클래스가 가상함수를 오버라이드 하지 않거나 protected 기본 멤버에 접근하는 non-public 기본 클래스의 사용을 표시하라.


><a name="Rh-domain"></a>
### C.120: Use class hierarchies to represent concepts with inherent hierarchical structure (only) 
 
>**Reason**: Direct representation of ideas in code eases comprehension and maintenance. Make sure the idea represented in the base class exactly matches all derived types and there is not a better way to express it than using the tight coupling of inheritance.  
>Do *not* use inheritance when simply having a data member will do. Usually this means that the derived type needs to override a base virtual function or needs access to a protected member.

>**Example**:  
??? Good old Shape example?  
>**Example, bad**:   
Do *not* represent non-hierarchical domain concepts as class hierarchies.
> 
```
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
```

>Here most overriding classes cannot implement most of the functions required in the interface well.
Thus the base class becomes an implementation burden.
Furthermore, the user of `Container` cannot rely on the member functions actually performing a meaningful operations reasonably efficiently; it may throw an exception instead.
Thus users have to resort to run-time checking and/or
not using this (over)general interface in favor of a particular interface found by a run-time type inquiry (e.g., a `dynamic_cast`).

> **Enforcement**:
> * Look for classes with lots of members that do nothing but throw.
> * Flag every use of a nonpublic base class where the derived class does not override a virtual function or access a protected base member.




<a name="Rh-abstract"></a>
### C.121: 만약 기본 클래스가 인터페이스로써 사용되면, 순수 추상 클래스로 만들어라

##### 근거: 
클래스는 데이터를 보유하지 않게 되면 더 안정적이게(깨지지 않게) 된다. 인터페이스들은 일반적으로 public한 순수 가상함수들로 작성되어야 한다.  

##### 예:
```
	???
```
##### 시행하기:  
* 데이터 멤버를 포함하거나 오버라이드할 수 있는(`final`이 아닌) 가상 함수들을 포함하는 클래스에 대해서는 경고하라.  


> <a name="Rh-abstract"></a>
### C.121: If a base class is used as an interface, make it a pure abstract class

>**Reason**: A class is more stable (less brittle) if it does not contain data. Interfaces should normally be composed entirely of public pure virtual functions.

>**Example**:  
>  
	???
> **Enforcement**:
> * Warn on any class that contains data members and also has an overridable (non-`final`) virtual function.




<a name="Rh-separation"></a>
### C.122: 인터페이스와 구현의 완전한 분리가 필요하면, 추상 클래스를 인터페이스로 사용하라.

##### 근거: 
ABI(<a>링크</a>)와 같은 경우.

##### 예:
```
	???
```
##### 시행하기: 
???


> <a name="Rh-separation"></a>
### C.122: Use abstract classes as interfaces when complete separation of interface and implementation is needed  
>**Reason**: Such as on an ABI (link) boundary.  
>**Example**:
> 
	???
>**Enforcement**: ???


## C.hierclass: 계층 안에 있는 클래스들의 설계:
> ## C.hierclass: Designing classes in a hierarchy:


<a name="Rh-abstract-ctor"></a>
### C.126: 추상 클래스는 일반적으로 생성자가 필요하지 않다. 

##### 근거:
추상 클래스는 일반적으로 초기화 할 데이터를 가지지 않는다.  

##### 예:
```
	???
```

##### 예외 사항:
* 기본 클래스의 생성자가 어떤 작업을 수행하는 경우, 가령 생성된 객체를 어딘가에 기록할 경우, 생성자가 필요할 수 있다.
* 극도로 드문 경우지만, 추상 클래스가 파생 클래스들에 의해 공유되는 데이터를 보유할 이유를 찾을 수도 있다.
(e.g., 정적인 데이터, 디버깅 정보 등등..); 그러한 클래스는 생성자를 가진다. 하지만, 주의하라. 그러한 클래스들은 가상 상속을 사용할 때 취약하기도 하다.

##### 시행하기: 
생성자를 가지는 추상 클래스에는 표시를 하라.


> <a name="Rh-abstract-ctor"></a>
### C.126: An abstract class typically doesn't need a constructor

>**Reason**: An abstract class typically does not have any data for a constructor to initialize.

>**Example**:
>
	???

> **Exceptions**:
> * A base class constructor that does work, such as registering an object somewhere, may need a constructor.
> * In extremely rare cases, you might find a reasonable for an abstract class to have a bit of data shared by all derived classes
(e.g., use statistics data, debug information, etc.); such classes tend to have constructors. But be warned: Such classes also tend to be prone to requiring virtual inheritance.

> **Enforcement**: Flag abstract classes with constructors.




<a name="Rh-dtor"></a>
### C.127: 가상함수를 가진 크래스는 가상 소멸자를 가져야 한다.

##### 근거: 
가상 함수를 가진 클래스는 일반적으로 기본 클래스에 대한 포인터나 스마트 포인터를 통해서 사용된다. 사용자는 기본 클래스에 대한 포인터에 `delete`를 호출하는 경우를 포함한다.  

##### 잘못된 예:
```
struct B {
	// ... no destructor ...
};

stuct D : B {		
	// bad: class with a resource 
	// derived from a class without a virtual destructor
	string s {"default"};
};

void use()
{
	B* p = new B;
	delete p;	// leak the string
}
```

##### 참고 사항: 
`shared_ptr`: `std::shared_ptr<B> p = std::make_shared<D>(args);`를 통해서만 클래스를 사용하도록 계획한 경우 이 지침을 따르지 않을 수 있다. 이 경우, 공유 포인터가 개체의 소멸을 담당할 때, 기본 클래스에 대한 부적합한 `delete`호출로 인한 누수가 발생하지 않을 것이다. 일관적으로 이런 코드를 작성하는 사람이라면 따르지 않아도 좋다.(false positive) 하지만 이 지침 자체는 중요하다. -- 만약 `make_unique`를 통해서 객체가 할당된다면? `B`의 작성자가 절대로 오용되지 않는다고, 가령 팩토리 함수를 사용해 `make_shared`를 강제하여 할당하는 것처럼, 확신하지 않는다면 안전하지 않다.

##### 시행하기:
* 가상 함수를 가지지만 가상 소멸자를 가지지 않는 클래스에는 표시를 남겨라. 이 지침은 오직 첫번째 기본 클래스에만 필요하다. 파생 클래스들은 필요한 것들을 자동적으로 상속받게 된다. 이 표시를 통해 문제가 발생하는 지점을 알 수 있다. 하지만 따르지 않을 수도 있다.
* 가상 함수를 가지지만 가상 소멸자를 가지지 않는 클래스를 `delete`할때는 표시를 남겨라.


> <a name="Rh-dtor"></a>
### C.127: A class with a virtual function should have a virtual destructor

> **Reason**: A class with a virtual function is usually (and in general) used via a pointer to base, including that the last user has to call delete on a pointer to base, often via a smart pointer to base.

> **Example, bad**:
>
	struct B {
		// ... no destructor ...
	};
	stuct D : B {		
		// bad: class with a resource 
		// derived from a class without a virtual destructor
		string s {"default"};
	};
	void use()
	{
		B* p = new B;
		delete p;	// leak the string
	}

> **Note**: There are people who don't follow this rule because they plan to use a class only through a `shared_ptr`: `std::shared_ptr<B> p = std::make_shared<D>(args);` Here, the shared pointer will take care of deletion, so no leak will occur from and inappropriate `delete` of the base. People who do this consistently can get a false positive, but the rule is important -- what if one was allocated using `make_unique`? It's not safe unless the author of `B` ensures that it can never be misused, such as by making all constructors private and providing a factory functions to enforce the allocation with `make_shared`.

> **Enforcement**:  
> * Flag a class with a virtual function and no virtual destructor. Note that this rule needs only be enforced for the first (base) class in which it occurs, derived classes inherit what they need. This flags the place where the problem arises, but can give false positives.
> * Flag `delete` of a class with a virtual function but no virtual destructor.




<a name="Rh-override"></a>
### C.128: 클래스 계층구조가 클 경우에는 `override`를 명시하라.

##### 근거: 
가독성. 실수들의 확인. 명시적인 `override`는 컴파일러가 기본 클래스와 파생 클래스 사이의 타입/이름들의 불일치를 잡아낼 수 있도록 한다.

##### 잘못된 예:
```
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
```
##### 시행하기:
* 기본 클래스와 파생 클래스의 이름들을 비교하고 같은 이름을 사용하지만 오버라이드 하지 않는 경우에는 표시를 남겨라.
* `override`키워드를 사용하지 않은 오버라이드 함수들에는 표시를 남겨라.


> <a name="Rh-override"></a>
### C.128: Use `override` to make overriding explicit in large class hierarchies

> **Reason**: Readability. Detection of mistakes. Explicit `override` allows the compiler to catch mismatch of types and/or names between base and derived classes.  
> **Example, bad**:
> 
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

> **Enforcement**:
> * Compare names in base and derived classes and flag uses of the same name that does not override.
> * Flag overrides without `override`.




<a name="Rh-kind"></a>
### C.129: 클래스 계층구조를 설계할 때는, 인터페이스 상속과, 구현의 상속을 구분하라.  

##### 근거: 
???  
Herb: 구현 상속은 별로 좋지 않은 것 같습니다. -- 대부분 안티패턴으로 보입니다. 이에 대한 타당한 예시들이 있을까요?

##### 예:
```
	???
```
##### 시행하기: 
???


> <a name="Rh-kind"></a>
### C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance

> **Reason**: ??? Herb: I've become a non-fan of implementation inheritance -- seems most often an antipattern. Are there reasonable examples of it?  

> **Example**:
> 
	???
	
> **Enforcement**: ???



 
<a name="Rh-copy"></a>
### C.130: 기본 클래스의 복사를 재정의하거나 금지하라; 대신 가상 `clone`함수를 사용하라.

##### 근거: 
기본클래스를 복사하는 것은 일반적으로 슬라이싱(slicing)이다. 만약 복사 문맥(의미론)이 필요하면, 깊은 복사를 하라: 가상 `clone`함수를 제공하여 실제로 가장 파생된 타입을 복사하도록 하라. 그리고 파생된 클래스들은 파생 타입을 반환하도록 하라.(covariant(자신과 동일한?) 반환 타입을 사용하라.) 

##### 예:
```
class base {
public:
    virtual base* clone() =0;
};

class derived : public base {
public:
    derived* clone() override;
};
```
언어 규칙에 의해서, covariant 반환 타입은 스마트 포인터가 될 수 없다.

##### 시행하기:
* 가상 함수와 사용자가 정의하지 않은 복사 연산을 가진 클래스에 표시를 남겨라.
* 기본 클래스 객체들의 복사 대입에 표시를 남겨라. (파생 클래스들의 객체들) 


> <a name="Rh-copy"></a>
### C.130: Redefine or prohibit copying for a base class; prefer a virtual `clone` function instead

> **Reason**: Copying a base is usually slicing. If you really need copy semantics, copy deeply: Provide a virtual `clone` function that will copy the actual most-derived type, and in derived classes return the derived type (use a covariant return type).

> **Example**:
> 
	class base {
	public:
	    virtual base* clone() =0;
	};
	class derived : public base {
	public:
	    derived* clone() override;
	};
> Note that because of language rules, the covariant return type cannot be a smart pointer.

> **Enforcement**:
> * Flag a class with a virtual function and a non-user-defined copy operation.
> * Flag an assignment of base class objects (objects of a class from which another has been derived).




<a name="Rh-get"></a>
### C.131: 사소한 getter나 setter는 지양하라.

##### 근거: 
사소한 getter나 setter는 문맥적인 가치를 가지지 않는다; 데이터가 `public`이 될수도 있다.

##### 예:
```
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
```
클래스를 `struct`로 만드는 것을 생각해보라. -- 즉, 동작이 없는 변수들의 덩어리, 모든 데이터들이 public하고 멤버함수가 없도록 만드는 것을 생각해보라.
```
struct point {
	int x = 0;
	int y = 0;
};
```
##### 참고 사항: 
내부 타입에서 인터페이스 타입으로 전환하는 setter와 getter는 사소하지 않다.(이 경우는 정보 은닉을 제공하는 것이므로)

##### 시행하기: 
다수의 `get`과 `set` 멤버 함수가 별다른 문맥 없이 멤버에 접근할 경우 표시를 남겨라.


><a name="Rh-get"></a>
### C.131: Avoid trivial getters and setters

> **Reason**: A trivial getter or setter adds no semantic value; the data item could just as well be `public`.  
> **Example**:
> 
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
> Consider making such a class a `struct` -- that is, a behaviorless bunch of variables, all public data and no member functions.
> 
	struct point {
		int x = 0;
		int y = 0;
	};

> **Note**: A getter or a setter that converts from an internal type to an interface type is not trivial (it provides a form of information hiding).

> **Enforcement**: Flag multiple `get` and `set` member functions that simply access a member without additional semantics.




<a name="Rh-virtual"></a>
### C.132: Don't make a function `virtual` without reason

##### 근거: 
중첩된 `virtual`는 실행 시간과 객체의 코드 크기를 증가시킨다.
가상 함수는 오버라이드 될 수 있고, 그렇기 때문에 파생 클래스에서의 실수에 노출되어있다. 
가상 함수는 템플릿화 된 계층구조에서 코드 복제를 야기한다.

##### 잘못된 예:
```
template<class T>
class Vector {
public:
	// ...
	virtual int size() const { return sz; }	
	// bad: what good could a derived class do?
private:
	T* elem;	// the elements
	int sz; 	// number of elements
};
```
이러한 형태의 "vector"는 기본 클래스로 사용되는 것을 전혀 의도하지 않았다.

##### 시행하기:
* 가상 함수를 가지지만 파생 클래스가 없으면 표시를 남겨라.
* 모든 멤버 함수가 가상 함수이고 구현을 가지고 있으면 표시를 남겨라.


> <a name="Rh-virtual"></a>
### C.132: Don't make a function `virtual` without reason

> **Reason**: Redundant `virtual` increases run-time and object-code size. A virtual function can be overridden and is thus open to mistakes in a derived class. A virtual function ensures code replication in a templated hierarchy.

> **Example, bad**:
> 
	template<class T>
	class Vector {
	public:
		// ...
		virtual int size() const { return sz; }	
		// bad: what good could a derived class do?
	private:
		T* elem;	// the elements
		int sz; 	// number of elements
	};
> This kind of "vector" isn't meant to be used as a base class at all.

> **Enforcement**:
> * Flag a class with virtual functions but no derived classes.
> * Flag a class where all member functions are virtual and have implementations.




<a name="Rh-protected"></a>
### C.133: `protected` 데이터를 지양하라.

##### 근거:  
`protected` 데이터는 복잡성과 에러의 원인이다.
`protected` 데이터는 불변조건의 구문을 복잡하게 만든다.
`protected` 데이터는 근본적으로 데이터를 기본 클래스에 넣고 
data inherently violates the guidance against putting data in base classes, which usually leads to having to deal virtual inheritance as well.  

##### 예:
``` 
	???
```
##### 참고 사항: 
Protected 멤버 함수는 허용된다.

##### 시행하기: 
`protected` 데이터를 지닌 클래스들에 표시를 남겨라.


> <a name="Rh-protected"></a>
### C.133: Avoid `protected` data

> **Reason**: `protected` data is a source of complexity and errors.
> `protected` data complicated the statement of invariants.
> `protected` data inherently violates the guidance against putting data in base classes, which usually leads to having to deal virtual inheritance as well.

> **Example**:
> 
	???

> **Note**: Protected member function can be just fine.

> **Enforcement**: Flag classes with `protected` data.



 
<a name="Rh-public"></a>
### C.134: 모든 데이터 멤버들이 같은 접근 레벨에 있도록 하라.

##### 근거: 
만약 모든 데이터 멤버들이 같은 접근 레벨에 있지 않으면, 그 타입은 무엇을 시도하는 것인지 혼란을 가져올 수 있다.   
오직 타입이 실제로 추상화가 아니고, 편의를 위한 독립 변수들의 모임(행동이 없는 변수들의 덩어리)일 경우에만 모든 데이터 멤버들을 `public`으로 만들고 함수들을 제공하지 말아라.  
그렇지 않은 경우, 그 타입은 추상화이므로, 모든 데이터멤버를 `private`으로 만들고 `public`과 혼용하지 말아라.

##### 예:
```
	???
```
##### 시행하기: 
다른 접근레벨을 가진 데이터 멤버가 있는 클래스는 표시를 남겨라.


> <a name="Rh-public"></a>
### C.134: Ensure all data members have the same access level

> **Reason**: If they don't, the type is confused about what it's trying to do. > Only if the type is not really an abstraction, but just a convenience bundle to group individual variables with no larger behavior (a behaviorless bunch of variables), make all data members `public` and don't provide functions with behavior. Otherwise, the type is an abstraction, so make all its data members `private`. Don't mix `public` and `private` data.  

> **Example**:
> 
	???
> **Enforcement**: Flag any class that has data members with different access levels.




<a name="Rh-mi-interface"></a>
### C.135: 다중 상속을 다수의 인터페이스를 표현하기 위해 사용하라

##### 근거: 
모든 클래스들이 모든 인터페이스들을 지원하지는 않을 것이다. 그리고 모든 호출자(caller)들이 모든 연산들을 사용하길 원하지도 않을 것이다. (다중상속은) 특별히 단일한(monolitic) 인터페이스들을 파생 클래스가 지원하는 동작의 측면들로 나눌때 사용하라. 

##### 예:
```
	???
```
##### 참고 사항: 
다중상속의 이용은 매우 일반적인데, 이는 다수의 다른 인터페이스들이 구현될 필요가 있는 경우가 일반적이기 때문이다. 그리고 이런 인터페이스들은 종종 쉽게, 또는 자연적으로 단일 상속 구조에서는 조직되지 않는다. 

##### 참고 사항: 
이런 인터페이스들은 일반적으로 추상 클래스들이다.

##### 시행하기: 
???


> <a name="Rh-mi-interface"></a>
### C.135: Use multiple inheritance to represent multiple distinct interfaces

> **Reason**: Not all classes will necessarily support all interfaces, and not all callers will necessarily want to deal with all operations. Especially to break apart monolithic interfaces into "aspects" of behavior supported by a given derived class.

> **Example**:
> 
	???

> **Note**: This is a very common use of inheritance because the need for multiple different interfaces to an implementation is common
and such interfaces are often not easily or naturally organized into a single-rooted hierarchy.  
> **Note**: Such interfaces are typically abstract classes.
  
> **Enforcement**: ???



 
<a name="Rh-mi-implementation"></a>
### C.136: 다중 상속을 구현 속성들의 합집합(union)을 표현하기 위해 사용하라.

##### 근거: 
???   
Herb: 여기서 구현 상속에 대한 두번째 언급이 있군요. 전 매우 부정적입니다. 하나의 구현 상속마저도요. 다수의 구현 상속은 절대 생각하지 마세요. -- 저는 정책 기반의(policy-based) 설계라도 정말로 정책 타입들을 상속할 필요가 있다고 생각하지 않습니다.
제가 좋은 예시들을 놓치고 있는 걸까요, 아니면 이것을 안티패턴으로 보고 제외시키는 것을 고려해야 할까요? 

##### 예:
```
	???
```
##### 참고 사항: 
이것은 상대적으로 드문 경우인데, 구현은 종종 단일루트(single-root) 계층으로 조직화될 수 있기 때문입니다.

##### 시행하기: 
??? 
Herb: 정반대의 시행하기: 2개 이상의 (데이터 멤버가 있는)기본 클래스를 상속하는 타입에는 표시를 남겨라?  


> <a name="Rh-mi-implementation"></a>
### C.136: Use multiple inheritance to represent the union of implementation attributes

> **Reason**: ??? Herb: Here's the second mention of implementation inheritance. I'm very skeptical, even of single implementation inheritance, never mind multiple implementation inheritance which just seems frightening -- I don't think that even policy-based design really needs to inherit from the policy types. Am I missing some good examples, or could we consider discouraging this as an anti-pattern?

> **Example**:
> 
	???

> **Note**: This a relatively rare use because implementation can often be organized into a single-rooted hierarchy.

> **Enforcement**: ??? Herb: How about opposite enforcement: Flag any type that inherits from more than one non-empty base class?




<a name="Rh-vbase"></a>
### C.137: 가상 기본 클래스는 과도하게 일반적인 기본 클래스들을 피하기 위해 사용하라.

##### 근거: 
???

##### 예:
```
	???
```
##### 참고 사항: 
???

##### 시행하기: 
???

> <a name="Rh-vbase"></a>
### C.137: Use `virtual` bases to avoid overly general base classes

> **Reason**: ???  
> **Example**:
> 
	???
> **Note**: ???  
> **Enforcement**: ???




<a name="Rh-using"></a>
### C.138: `using`을 사용해서 파생 클래스와 기본 클래스의 Overload 집합을 만들어라

##### 근거: 
???

##### 예:
```
	???
```


> <a name="Rh-using"></a>
### C.138: Create an overload set for a derived class and its bases with `using`

> **Reason**: ???

> **Example**:
> 
	???



## C.hier-access: 계층 구조에서 객체 접근
> ## C.hier-access: Accessing objects in a hierarchy


<a name="Rh-poly"></a>
### C.145: 다형성을 지닌 객체는 포인터나 참조자를 사용해서 접근하라

##### 근거: 
가상 함수를 가진 클래스가 있다면, 당신은 (일반적으로) 어떤 클래스가 실행될 함수를 제공할지 알 수 없다.

##### 예:
```
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
```
use와 use2 양쪽의 `d` 모두 절단(slice)된다.  

##### 예외 사항: 
당신은 객체의 정의 범위 안의 이름이 있는 다형적 객체에 안전하게 접근할 수 있다. 단지 그것을 절단하지는 말아라.
```
void use3()
{
	D d;
	d.f();	// OK
}
```
##### 시행하기: 
모든 절단(slicing)에 표시를 남겨라.


> <a name="Rh-poly"></a>
### C.145: Access polymorphic objects through pointers and references

> **Reason**: If you have a class with a virtual function, you don't (in general) know which class provided the function to be used.

> **Example**:
> 
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

> Both `d`s are sliced.
  
> **Exeption**: You can safely access a named polymorphic object in the scope of its definition, just don't slice it.
> 
	void use3()
	{
		D d;
		d.f();	// OK
	}
	
> **Enforcement**: Flag all slicing.



<a name="Rh-dynamic_cast"></a>
### C.146: `dynamic_cast`는 클래스 계층 구조에서 탐색이 불가피할때 사용하라

##### 근거: 
`dynamic_cast`는 실행시간에 검사된다.

##### 예:
```
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
```
##### 참고 사항: 
다른 모든 캐스팅처럼, `dynamic_cast`는 너무 자주 사용된다.
[캐스팅 보다는 가상 함수들을 사용하라](#???).
가능한 한 클래스 계층을 탐색하는 것보다 [정적 다형성](#???)을 선호하라. (이렇게 하면 실행시간 실행시간 결정이 필요없다. 그리고 충분히 편리하다.)

##### 예외 사항: 
만약 당신의 구현 코드에 정말로 느린 `dynamic_cast`가 있다면, 대안을 찾아야 할 것이다. 
하지만, 정적으로 클래스를 결정할 수 없는 모든 대안은 명시적 캐스팅(일반적으로 `static_cast`)을 포함하고, 에러에 취약하다.  
당신만의 특별한 `dynamic_cast`를 만들수도 있을 것이다. 그러니, `dynamic_cast`가 정말로 당신이 생각하는 것 만큼 느리다는 것을 확실시하라. (근거 없는 루머들이 꽤 있다.) 그리고 `dynamic_cast`의 사용이 정말로 성능에 치명적이라는 것 또한 확실시 하라. 

##### 시행하기: 
하향식 캐스팅(C언어 스타일을 포함해서)에 사용되는 `static_cast`에 표시를 남겨라. 

> <a name="Rh-dynamic_cast"></a>
### C.146: Use `dynamic_cast` where class hierarchy navigation is unavoidable

> **Reason**: `dynamic_cast` is checked at run time.

> **Example**:
> 
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

> **Note**: Like other casts, `dynamic_cast` is overused.
> [Prefer virtual functions to casting](#???).
> Prefer [static polymorphism](#???) to hierarchy navigation where it is possible (no run-time resolution necessary) and reasonably convenient.

> **Exception**: If your implementation provided a really slow `dynamic_cast`, you may have to use a workaround.
> However, all workarounds that cannot be statically resolved involve explicit casting (typically `static_cast`) and are error-prone.
> You will basically be crafting your own special-purpose `dynamic_cast`.
> So, first make sure that your `dynamic_cast` really is as slow as you think it is (there are a fair number of unsupported rumors about) and that your use of `dynamic_cast` is really performance critical.

> **Enforcement**: Flag all uses of `static_cast` for downcasts, including C-style casts that perform a `static_cast`.




<a name="Rh-ptr-cast"></a>
### C.147: 필요한 타입을 찾는 데 실패하는 것이 에러로 간주될 때는, `dynamic_cast`를 참조자 타입에 사용하라. 

##### 근거: 
참조자에 대한 캐스팅은 당신이 정상적인 객체를 얻는 것을 의도했음을 표현한다. 따라서 캐스팅은 반드시 성공해야만 한다. `dynamic_cast`는 만약 실패한다면 예외를 던질 것이다.

##### 예:
```
	???
```
##### 시행하기: 
???


> <a name="Rh-ptr-cast"></a>
### C.147: Use `dynamic_cast` to a reference type when failure to find the required class is considered an error

> **Reason**: Casting to a reference expresses that you intend to end up with a valid object, so the cast must succeed. `dynamic_cast` will then throw if it does not succeed.  
> **Example**:
> 
	???
> **Enforcement**: ???




<a name="Rh-ref-cast"></a>
### C.148: 필요한 클래스를 찾는데 실패하는 것이 허용가능 하다면, `dynamic_cast`를 포인터 타입에 사용하라.

##### 근거: 
???

##### 예:
```
	???
```
##### 시행하기: 
???


> <a name="Rh-ref-cast"></a>
### C.148: Use `dynamic_cast` to a pointer type when failure to find the required class is considered a valid alternative

> **Reason**: ???  
> **Example**:
> 
	???
> **Enforcement**: ???




<a name="Rh-smart"></a>
### C.149: `new`를 사용해서 생성한 객체를 `delete`하지 않는 것을 예방하기 위해, `unique_ptr` 또는 `shared_ptr`를 사용하라

##### 근거: 
자원 누수를 방지한다.

##### 예:
```
void use(int i)
{
	auto p = new int {7};			// bad: initialize local pointers with new
	auto q = make_unique<int>(9);	// ok: guarantee the release of the memory allocated for 9
	if(0 < i) return;	// maybe return and leak
	delete p;		// too late
}
```
##### 시행하기:
* `new`를 사용한 naked 포인터의 초기화에 표시를 남겨라. 
* 지역 변수의 `delete`처리에 표시를 남겨라. 


> <a name="Rh-smart"></a>
### C.149: Use `unique_ptr` or `shared_ptr` to avoid forgetting to `delete` objects created using `new`

> **Reason**: Avoid resource leaks.  
> **Example**:
```
void use(int i)
{
	auto p = new int {7};			// bad: initialize local pointers with new
	auto q = make_unique<int>(9);	// ok: guarantee the release of the memory allocated for 9
	if(0<i) return;	// maybe return and leak
	delete p;		// too late
}
```
> **Enforcement**:  
> * Flag initialization of a naked pointer with the result of a `new`
> * Flag `delete` of local variable



 
<a name="Rh-make_unique"></a>
### C.150: unique_ptr나 다른 스마트포인터가 소유하는 객체를 생성하기 위해선 make_unique()를 사용하라

##### 근거: 
`make_unique`는 생성 구문을 보다 간결하게 만든다.

##### 예:
```
unique_ptr<Foo> p {new<Foo>{7});	// OK: but repetitive

auto q = make_unique<Foo>(7);		// Better: no repetition of Foo
```
##### 시행하기:
* 템플릿 특수화(specialization) 리스트 `<Foo>`의 반복적인 사용에 표시를 남겨라.
* `unique_ptr<Foo>`로 선언된 변수에 표시를 남겨라. 


> <a name="Rh-make_unique"></a>
### C.150: Use `make_unique()` to construct objects owne by `unique_ptr`s or other smart pointers

> **Reason**: `make_unique` gives a more concise statement of the construction.  
> **Example**:  
>
	unique_ptr<Foo> p {new<Foo>{7});	// OK: but repetitive
	auto q = make_unique<Foo>(7);		// Better: no repetition of Foo

> **Enforcement**
> * Flag the repetitive usage of template specialization list `<Foo>`
> * Flag variables declared to be `unique_ptr<Foo>`




<a name="Rh-make_shared"></a>
### C.151: shared_ptr들에 의해 소유되는 객체를 생성하기 위해서는 make_shared()를 사용하라

##### 근거: 
`make_shared`는 생성 구문을 더 간결하게 만들어준다.
또한 `make_shared`는 객체 옆에 `shared_ptr`의 사용횟수를 배치하면서, 멀리 떨어져 표기되지 않도록 해준다.

##### 예:
```
	shared_ptr<Foo> p {new<Foo>{7});	// OK: but repetitive; and separate allocations for the Foo and shared_ptr's use count

	auto q = make_shared<Foo>(7);		// Better: no repetition of Foo; one object
```
##### 시행하기:
* 반복적인 템플릿 특수화 리스트 `<Foo>`의 사용에 표시를 남겨라. 
* `shared_ptr<Foo>`로 선언된 변수들에 표시를 남겨라. 


> <a name="Rh-make_shared"></a>
### C.151: Use `make_shared()` to construct objects owned by `shared_ptr`s

> **Reason**: `make_shared` gives a more concise statement of the construction.
> It also gives an opportunity to eliminate a separate allocation for the reference counts, by placing the `shared_ptr`'s use counts next to its object.

> **Example**:
> 
	shared_ptr<Foo> p {new<Foo>{7});	// OK: but repetitive; and separate allocations for the Foo and shared_ptr's use count
	auto q = make_shared<Foo>(7);		// Better: no repetition of Foo; one object

> **Enforcement**:
> * Flag the repetive usage of template specialization list`<Foo>`
> * Flag variables declared to be `shared_ptr<Foo>`




<a name="Rh-array"></a>
### C.152: 파생 클래스 객체들의 포인터 배열에 기본 클래스의 포인터를 할당해서는 절대로 안된다

##### 근거: 
기본 자료형의 포인터를 기록하는 것은 부당한 객체 접근과 메모리 손상을 야기할 수 있다. 

##### 예:
```
struct B { int x; };
struct D : B { int y; };

void use(B*);

D a[] = { {1,2}, {3,4}, {5,6} };
B* p = a;	// bad: a decays to &a[0] which is converted to a B*
p[1].x = 7;	// overwrite D[0].y

use(a);		// bad: a decays to &a[0] which is converted to a B*
```
##### 시행하기:
* 모든 종류의 배열 해제와 기본 타입에서 파생 타입으로의 형변환에 표시를 남겨라. 
* 배열을 포인터 보다는 `array_view`로 전달하라, 그리고 `array_view`로 전달하기 전 파생클래스에서 기본클래스로의 변환이 없도록 하라.   

> <a name="Rh-array"></a>
### C.152: Never assign a pointer to an array of derived class objects to a pointer to its base

> **Reason**: Subscripting the resulting base pointer will lead to invalid object access and probably to memory corruption.

> **Example**:
>  
	struct B { int x; };
	struct D : B { int y; };
>	
	void use(B*);
>
	D a[] = { {1,2}, {3,4}, {5,6} };
	B* p = a;	// bad: a decays to &a[0] which is converted to a B*
	p[1].x = 7;	// overwrite D[0].y
>	
	use(a);		// bad: a decays to &a[0] which is converted to a B*

> **Enforcement**:
> * Flag all combinations of array decay and base to derived conversions.
> * Pass an array as an `array_view` rather than as a pointer, and don't let the array name suffer a derived-to-base conversion before getting into the `array_view`



<a name="SS-overload"></a>
# C.over: Overloading and overloaded operators

You can overload ordinary functions, template functions, and operators.
You cannot overload function objects.

Overload rule summary:

* [C.160: 연산자를 정의할때는 관례적인 사용을 모방하라](#Ro-conventional)
* [C.161: 대칭적인 연산자들에는 비멤버 함수들을 사용하라](#Ro-symmetric)
* [C.162: 거의 동등한 연산들을 오버로드하라](#Ro-equivalent)
* [C.163: 거의 동등한 연산들만 오버로드하라](#Ro-equivalent-2)
* [C.164: 형변환 연산자들을 정의하지 말아라](#Ro-conversion)
* [C.170: 람다를 오버로딩하는 기분이 든다면, 제네릭 람다를 사용하라](#Ro-lambda)


> <a name="SS-overload"></a>
# C.over: Overloading and overloaded operators
> You can overload ordinary functions, template functions, and operators.
You cannot overload function objects.

> Overload rule summary:
> 
* [C.160: Define operators primarily to mimic conventional usage](#Ro-conventional)
* [C.161: Use nonmember functions for symmetric operators](#Ro-symmetric)
* [C.162: Overload operations that are roughly equivalent](#Ro-equivalent)
* [C.163: Overload only for operations that are roughly equivalent](#Ro-equivalent-2)
* [C.164: Avoid conversion operators](#Ro-conversion)
* [C.170: If you feel like overloading a lambda, use a generic lambda](#Ro-lambda)



<a name="Ro-conventional"></a>
### C.160: 연산자를 정의할때는 관례적인 사용을 모방하라

##### 근거: 
뜻밖의 의미가 없도록 한다.

##### 잘못된 예:
```
X operator+(X a, X b) { return a.v-b.v; }	// bad: makes + subtract
```
???. 비멤버 연산자들: (전통적인?) 네임스페이스 레벨 정의 또는 `friend` 정의(boost.operator에서 사용되고 ADL(Argument-Dependent Lookup)만으로 제한되는)  

##### 시행하기: 
거의 불가능하다.

> <a name="Ro-conventional"></a>
### C.160: Define operators primarily to mimic conventional usage
> **Reason**: Minimize surprises.  
> **Example, bad**:
>
	X operator+(X a, X b) { return a.v-b.v; }	// bad: makes + subtract
> ???. Non-member operators: namespace-level definition (traditional?) vs friend definition (as used by boost.operator, limits lookup to ADL only)  

> **Enforcement**: Possibly impossible.




<a name="Ro-symmetric"></a>
### C.161: 대칭적인 연산자들에는 비멤버 함수들을 사용하라

##### 근거: 
만약 멤버 함수로 정의하게 되면, 비 멤버 함수를 쓰지 않는 한 2개의 함수가 필요하게 된다. 가령  `==`, `a==b` 그리고 `b==a` 는 미묘하게 다른 함수가 된다.

##### 예:
```
bool operator==(Point a, Point b) { return a.x==b.x && a.y==b.y; }
```
##### 시행하기: 
멤버 연산자 함수들에는 표시를 남겨라.

> <a name="Ro-symmetric"></a>
### C.161: Use nonmember functions for symmetric operators
> **Reason**: If you use member functions, you need two.
Unless you use a non-member function for (say) `==`, `a==b` and `b==a` will be subtly different.  
> **Example**:
>
	bool operator==(Point a, Point b) { return a.x==b.x && a.y==b.y; }

> **Enforcement**: Flag member operator functions.




<a name="Ro-equivalent"></a>
### C.162: `동등한` 연산들을 오버로드하라

##### 근거: 
다른 인자타입을 지니는 논리적으로 동등한 연산에 제각기 다른 이름을 붙이는 것은 혼란을 야기한다. 또한 함수 이름에 타입정보를 넣는 것은 제네릭 프로그래밍을 방해한다.  

##### 예: 
고려 중(consider)
```
void print(int a);
void print(int a, int base);
void print(const string&);
```
이 세 함수들은 모두 인자들을 출력한다. 반대로, 
```
void print_int(int a);
void print_based(int a, int base);
void print_string(const string&);
```
이 세 함수들은 모두 인자들을 출력하지만, 함수 이름을 길게 만들면서 일반화된 코드가 되지 못하게 한다.

##### 시행하기: 
???


> <a name="Ro-equivalent"></a>
### C.162: Overload operations that are roughly equivalent

> **Reason**: Having different names for logically equivalent operations on different argument types is confusing, leads to encoding type information in function names, and inhibits generic programming.

> **Example**: Consider
>
	void print(int a);
	void print(int a, int base);
	void print(const string&);
> These three functions all prints their arguments (appropriately). Conversely
> 
	void print_int(int a);
	void print_based(int a, int base);
	void print_string(const string&);
> These three functions all prints their arguments (appropriately). Adding to the name just introduced verbosity and inhibits generic code.

> **Enforcement**: ???




<a name="Ro-equivalent-2"></a>
### C.163: 거의 동등한 연산들'만' 오버로드하라

##### 근거: 
논리적으로 다른 함수들이 같은 이름을 가지는 것은 혼란을 야기하며, 제네릭 프로그래밍을 사용할 때 에러로 이어진다. 

##### 예: 
고려 중(Consider)
```
void open_gate(Gate& g);	// remove obstacle from garage exit lane
void fopen(const char*name, const char* mode);	// open file
```
두 연산은 근본적으로 다르다(그리고 연관성이 없다). 따라서 서로 함수명이 다른 것은 타당하다. 반대로,  
```
void open(Gate& g);	// remove obstacle from garage exit lane
void open(const char*name, const char* mode ="r");	// open file
```
이 두 연산은 여전히 근본적으로 다르고 연관성이 없지만, 이름이 혼란을 가져올 가능성이 있도록 축약되었다. 
다행히도, 타입 시스템이 혼란으로 인한 많은 잘못된 사용들을 잡아낼 것이다.

##### 참고 사항: 
가령 `open`, `move`, `+`, 그리고 `==` 같은 연산들처럼 일반적이고 자주 사용되는 이름에는 특히 신중하라. 

##### 시행하기:
???


> <a name="Ro-equivalent-2"></a>
### C.163: Overload only for operations that are roughly equivalent

> **Reason**: Having the same name for logically different functions is confusing and leads to errors when using generic programming.

> **Example**: Consider
>
	void open_gate(Gate& g);	// remove obstacle from garage exit lane
	void fopen(const char*name, const char* mode);	// open file
> The two operations are fundamentally different (and unrelated) so it is good that their names differ. Conversely:
>
	void open(Gate& g);	// remove obstacle from garage exit lane
	void open(const char*name, const char* mode ="r");	// open file
> The two operations are still fundamentally different (and unrelated) but the names have been reduced to their (common) minimum, opening opportunities for confusion.  
> Fortunately, the type system will catch many such mistakes.
  
> **Note**: be particularly careful about common and popular names, such as `open`, `move`, `+`, and `==`.

> **Enforcement**: ???



<a name="Ro-conversion"></a>
### C.164: 형변환 연산자들을 정의하지 말아라

##### 근거: 
암묵적 형변환은 필수적일 수도 있다. (`double` 에서 `int`로의 변환) 하지만 종종 기대밖의 동작이 일어난다. (`String` 에서 C-스타일 문자열로 변환) 

##### 참고 사항: 
중대한 필요가 발생하지 않는 한, 명시적으로 변환하는 것을 지향하라. 여기서 중대한 필요는 응용프로그램 안에서 필수적이거나 자주 사용되는 등의 이유를 의미한다.
(가령 정수에서 복소수로의 변환)  
암묵적 형변환을 제공하지 말아라. (형변환 연산자 또는 `explicit`을 명기하지 않은 생성자들) 약간의 불편함만 있을 뿐이다.

##### 잘못된 예:
```
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
```
`s`에 할당되고 `p`에 대입된 문자열이 사용될 수 있게 되기 전에 파괴된다.

##### 시행하기: 
모든 형변환 연산자에 표시를 남겨라.

> <a name="Ro-conversion"></a>
### C.164: Avoid conversion operators
> **Reason**: Implicit conversions can be essential (e.g., `double` to '`int`) but often cause surprises (e.g., `String` to C-style string).  
> **Note**: Prefer explicitly named conversions until a serious need is demonstracted.  

> By "serious need" we mean a reason that is fundamental in the application domain (such as an integer to complex number conversion)
> and frequently needed. Do not introduce implicit conversions (through conversion operators or non-`explicit` constructors) just to gain a minor convenience.

> **Example, bad**:
> 
	class String {	// handle ownership and access to a sequence of characters
		// ...
		String(czstring p); // copy from *p to *(this->elem)
		// ...
		operator zstring() { return elem; }
		// ...
	};
>
	void user(zstring p)
	{
		if (*p=="") {
			String s {"Trouble ahead!"};
			// ...
			p = s;
		}
		// use p
	}
> The string allocated for `s` and assigned to `p` is destroyed before it can be used.

> **Enforcement**: Flag all conversion operators.




<a name="Ro-lambda"></a>
### C.170: 람다를 오버로딩하는 기분이 든다면, 제네릭 람다를 사용하라

##### 근거: 
같은 이름으로 다른 람다 함수를 오버로드 할 수 있다. 

##### 예:
```
void f(int);
void f(double);
auto f = [](char);	// error: cannot overload variable and function

auto g = [](int) { /* ... */ };
auto g = [](double) { /* ... */ };	// error: cannot overload variables

auto h = [](auto) { /* ... */ };	// OK
```
##### 시행하기: 
컴파일러가 람다 함수에 대한 오버로드를 잡아낸다. 


> <a name="Ro-lambda"></a>
### C.170: If you feel like overloading a lambda, use a generic lambda

> **Reason**: You can overload by defining two different lambdas with the same name  
> **Example**:
> 
	void f(int);
	void f(double);
	auto f = [](char);	// error: cannot overload variable and function
>
	auto g = [](int) { /* ... */ };
	auto g = [](double) { /* ... */ };	// error: cannot overload variables
>
	auto h = [](auto) { /* ... */ };	// OK

> **Enforcement**: The compiler catches attempt to overload a lambda.



<a name="SS-union"></a>
## C.union: 공용체

공용체 규칙 요약:

* [C.180: ???에는 `union`을 이용하라](#Ru-union)
* [C.181: "naked" `union`를 지양하라](#Ru-naked)
* [C.182: 익명 공용체 `union`들은 tagged(중첩된?) 공용체를 구현하기 위해 사용하라](#Ru-anonymous)
* ???

> <a name="SS-union"></a>
## C.union: Unions
> ???  
> Union rule summary:
> * [C.180: Use `union`s to ???](#Ru-union)
> * [C.181: Avoid "naked" `union`s](#Ru-naked)
> * [C.182: Use anonymous `union`s to implement tagged unions](#Ru-anonymous)
> * ???



<a name="Ru-union"></a>
### C.180: ???에는 `union`을 사용하라

??? : 공용체는 언제 사용되어야 할까요? 어떤 방법이 POD(Plain-Old-Data) 객체 표현을 더 장래성있게(future-proof) 재해석하는 방법일까요?  
??? : 변형(variant)

##### 근거: 
???  
##### 예:
```
	???
```
##### 시행하기: 
???


> <a name="Ru-union"></a>
### C.180: Use `union`s to ???

> ??? When should unions be used, if at all? What's a good future-proof way to re-interpret object representations of PODs?
> ??? variant

> **Reason**: ???  
> **Example**:  
> 
	???
> **Enforcement**: ???




<a name="Ru-naked"></a>
### C.181: "naked" `union`을 지양하라.

##### 근거: 
Naked 공용체들은 타입 에러의 원인이다.  
##### 대안: 
타입 필드와 함께 클래스로 감싸라.  
##### 대안: 
`variant`를 사용하라.  

##### 예:
```
	???
```
##### 시행하기: 
???


> <a name="Ru-naked"></a>
### C.181: Avoid "naked" `union`s

> **Reason**: Naked unions are a source of type errors.  
> **Alternative**: Wrap them in a class together with a type field.  
> **Alternative**: Use `variant`.  
> **Example**:
> 
	???
> **Enforcement**: ???  




<a name="Ru-anonymous"></a>
### C.182: 익명 공용체 `union`들은 tagged(중첩된?) 공용체를 구현하기 위해 사용하라.

##### 근거: 
???  

**예시**:
```
	???
```
##### 시행하기: 
???


> <a name="Ru-anonymous"></a>
### C.182: Use anonymous `union`s to implement tagged unions  
> **Reason**: ???  
> **Example**:
> 
	???
> **Enforcement**: ???
