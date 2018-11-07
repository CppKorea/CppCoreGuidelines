
# <a name="S-class"></a>C: 클래스와 클래스 계층 구조

클래스는 사용자 정의 타입으로써, 타입의 표현과 연산, 인터페이스를 프로그래머가  정의할 수 있다.
클래스 계층 구조는 관련된 클래스들을 계층적으로 구조화 할 때 사용된다.

클래스 규칙 요약:

* [C.1: 관련된 데이터를 조직화 하라 (`struct` 와 `class`)](#Rc-org)
* [C.2: 타입이 불변조건을 가진다면, `class`를 사용하라; 데이터 멤버들에 대한 제약이 자유롭다면 `struct`를 사용하라](#Rc-struct)
* [C.3: 클래스를 사용해 인터페이스와 구현을 분리하라](#Rc-interface)
* [C.4: 클래스에 직접적으로 접근할 필요가 있는 경우에만 함수를 멤버함수로 작성하라](#Rc-member)
* [C.5: 보조 함수들은 관련 클래스와 같은 namespace에 배치하라](#Rc-helper)
* [C.7: 클래스 또는 열거형에 대한 정의와 변수 선언을 같은 구문에 넣지 말아라](#Rc-standalone)
* [C.8: non-public 멤버가 있다면 `struct`보단 `class`를 사용하라](#Rc-class)
* [C.9: 멤버들의 노출을 최소화하라](#Rc-private)

하위 영역:

* [C.concrete: 실제 타입(Concrete types)](#SS-concrete)
* [C.ctor: 생성자, 대입 연산자, 소멸자](#S-ctor)
* [C.con: 컨테이너와 리소스 핸들](#SS-containers)
* [C.lambdas: 함수 객체와 람다 표현식](#SS-lambdas)
* [C.hier: 클래스 계층 구조 (OOP)](#SS-hier)
* [C.over: 오버로딩](#SS-overload)
* [C.union: 공용체](#SS-union)

### <a name="Rc-org"></a>C.1: 관련된 데이터를 조직화 하라 (`struct` 와 `class`)

##### Reason

이해하기 쉽다. 근본적인 이유로 데이터가 관련이 있다면, 그 사실은 코드에 반영되어야 한다.

##### Example

```c++
    void draw(int x, int y, int x2, int y2);  // BAD: unnecessary implicit relationships
    void draw(Point from, Point to);          // better
```

##### Note

가상 함수가 없는 간단한 클래스는 공간, 시간적인 오버헤드가 없다.

##### Note

언어적인 관점에서 볼 때 `class` 와 `struct`의 차이는 멤버들의 가시성(visibility)이다.

##### Enforcement

특별히 없다. 데이터 항목들에 대한 경험적인 관점들이 함께 반영될 수는 있을 것이다.

### <a name="Rc-struct"></a>C.2: Use `class` if the class has an invariant; use `struct` if the data members can vary independently

##### Reason

가독성이 좋고 이해하기도 쉽다.
`class` 를 사용함으로써, 프로그래머가 불변조건(invariant)이 필요하다는 것을 알게 된다.  
이 점은 유익한 관습이다.

##### Note

invariant는 객체 멤버들의 논리적인 상태로써, 공개 멤버 함수들이 가정할 수 있도록 생성자가 설정 해 주어야 한다. invariant 가 설정된 후에야 (일반적으로 생성자에 의해) 모든 멤버 함수는 객체를 통해 호출될 수 있다.
invariant 는 형식에 구애받지 않고 (가령, 주석으로) 기술될 수 있으며, 더 형식을 갖춘다면 `Expects` 를 사용할 수 있다.

만약 모든 데이터 멤버들이 상호독립적이라면, 불변조건은 존재할 수 없다.

##### Example

```c++
    struct Pair {  // the members can vary independently
        string name;
        int volume;
    };
```
하지만:
```c++
    class Date {
    public:
        // validate that {yy, mm, dd} is a valid date and initialize
        Date(int yy, Month mm, char dd);
        // ...
    private:
        int y;
        Month m;
        char d;    // day
    };
```

##### Note

클래스가 어떤 `private` 데이터를 가지고 있으면, 사용자는 생성자 호출 없이 객체를 초기화할 수 없다. 
따라서, 클래스를 정의하는 사람은 생성자를 제공하고 그 의미를 명시해야만 한다.
이는 클래스 작성자가 invariant를 정의해야 한다는 것을 의미한다.

**See also**:

* [define a class with private data as `class`](#Rc-class)
* [Prefer to place the interface first in a class](#Rl-order)
* [minimize exposure of members](#Rc-private)
* [Avoid `protected` data](#Rh-protected)

##### Enforcement

private 데이터를 가진 `struct`나 public 멤버를 가진 `class`들을 찾아낸다.

### <a name="Rc-interface"></a>C.3: 클래스를 사용해 인터페이스와 구현을 분리하라

##### Reason

인터페이스와 구현에 대한 분명한 구분은 가독성을 더 좋게 하고, 유지 보수를 단순하게 한다.

##### Example

```c++
    class Date {
        // ... some representation ...
    public:
        Date();
        // validate that {yy, mm, dd} is a valid date and initialize
        Date(int yy, Month mm, char dd);

        int day() const;
        Month month() const;
        // ...
    };
```

이러한 경우, 이제 사용자에게 영향을 주지 않고 `Date` 에 대한 representation을 변경할 수 있다. (비록 다시 컴파일 해야 하겠지만)

##### Note

인터페이스와 구현간의 구분을 표현하기 위해 클래스를 사용하는 것이 유일한 방법은 아니다.
예를 들면, 인터페이스를 표현하기 위한 개념으로 네임스페이스 안에 독립적인 함수들이나 추상 기본 클래스 혹은 템플릿 함수들을 선언해서 사용할 수 있다.
가장 중요한 것을 명시적으로 인터페이스와 그것들의 구현 "세부사항"을 구분하는 것이다.
이상적으로, 그리고 일반적으로, 인터페이스는 그 구현들보다 훨씬 더 안정적이다.

##### Enforcement

???

### <a name="Rc-member"></a>C.4: 클래스에 직접적으로 접근할 필요가 있는 경우에만 함수를 멤버함수로 작성하라

##### Reason

멤버 함수간 커플링을 줄이고, 객체 상태 변경에 의해 문제가 생기는 함수를 줄이고, representation이 변경된 후에 수정될 필요가 있는 멤버 함수의 수를 줄인다.

##### Example

```c++
    class Date {
        // ... 상대적으로 적은 인터페이스 ...
    };

    // helper functions:
    Date next_weekday(Date);
    bool operator==(Date, Date);
```

표시된 "helper functions"은 `Date`의 representation에 직접 접근할 필요가 없다.

##### Note

["uniform function call"](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0251r0.pdf)이 가능해지면 더 좋아질 것이다.

##### Exception

The language requires `virtual` functions to be members, and not all `virtual` functions directly access data.
In particular, members of an abstract class rarely do.

Note [multi-methods](https://parasol.tamu.edu/~yuriys/papers/OMM10.pdf).

##### Exception

The language requires operators `=`, `()`, `[]`, and `->` to be members.

##### Exception

An overload set may have some members that do not directly access `private` data:
```c++
    class Foobar {
    public:
        void foo(long x)    { /* manipulate private data */ }
        void foo(double x) { foo(std::lround(x)); }
        // ...
    private:
        // ...
    };
```
Similarly, a set of functions may be designed to be used in a chain:

```c++
    x.scale(0.5).rotate(45).set_color(Color::red);
```

Typically, some but not all of such functions directly access `private` data.

##### Enforcement

* Look for non-`virtual` member functions that do not touch data members directly.
The snag is that many member functions that do not need to touch data members directly do.
* Ignore `virtual` functions.
* Ignore functions that are part of an overload set out of which at least one function accesses `private` members.
* Ignore functions returning `this`.

### <a name="Rc-helper"></a>C.5: 보조 함수들은 관련 클래스와 같은 namespace에 배치하라

##### Reason

보조 함수(helper function)는 (보통 클래스 작성자가 제공하는) 클래스의 표현에 직접 접근할 필요가 없는 함수이며, 클래스에 대한 유용한 인터페이스 중에 하나로 볼 수 있다.
보조 함수들을 같은 네임스페이스에 넣으면 함수와 클래스의 관계가 명확해지고, 인자 종속적인 검색(Argument Dependent Lookup)에서 발견 할 수 있게 된다.

##### Example

```c++
    namespace Chrono { // here we keep time-related services

        class Time { /* ... */ };
        class Date { /* ... */ };

        // helper functions:
        bool operator==(Date, Date);
        Date next_weekday(Date);
        // ...
    }
```

##### Note

이는 [연산자 오버로딩](#Ro-namespace)을 위해서 매우 중요하다.

##### Enforcement

* 단일 네임스페이스에서 인자 타입을 취하는 전역함수들을 지적하라

### <a name="Rc-standalone"></a>C.7: 클래스 또는 열거형에 대한 정의와 변수 선언을 같은 구문에 넣지 말아라

##### Reason

타입에 대한 정의와 다른 개체(entitiy)에 대한 정의를 같은 구문(statement)에 넣는 것은 혼동을 일으킬 수 있고, 불필요하다.

##### Example; bad

```c++
    struct Data { /*...*/ } data{ /*...*/ };
```

##### Example; good

```c++
    struct Data { /*...*/ };
    Data data{ /*...*/ };
```

##### Enforcement

* 클래스나 열거형의 정의에 있는 닫는 괄호 `}`에 `;`이 바로 나타나지 않으면 지적하라

### <a name="Rc-class"></a>C.8: non-public 멤버가 있다면 `struct`보단 `class`를 사용하라

##### Reason

가독성에 좋다.  
무엇인가 숨겨져 있거나, 추상화되었다는 것을 분명하게 한다.  
유익한 관습이다.

##### Example, bad

```c++
    struct Date {
        int d, m;

        Date(int i, Month m);
        // ... lots of functions ...
    private:
        int y;  // year
    };
```

C++ 언어 규칙을 고려했을 때 이 코드엔 잘못된 것이 없다.  
하지만 디자인 관점에서는 모든게 잘못되었다.
private 데이터가 public data와 멀리 떨어져 숨어있고, 클래스 선언의 다른 부분들로 분리되어 있다.  
이런 요소들은 가독성을 저해하고 유지보수를 복잡하게 한다.

##### Note

클래스 인터페이스를 먼저 배치하라. [NL.16을 참고하라](#Rl-order)

##### Enforcement

Flag classes declared with `struct` if there is a `private` or `protected` member.

### <a name="Rc-private"></a>C.9: 멤버들의 노출을 최소화하라

##### Reason

캡슐화. 정보 은닉. 의도치 않은 접근을 최소화 하고, 유지보수를 쉽게 한다.

##### Example

```c++
    template<typename T, typename U>
    struct pair {
        T a;
        U b;
        // ...
    };
```

Whatever we do in the `//`-part, an arbitrary user of a `pair` can arbitrarily and independently change its `a` and `b`.
In a large code base, we cannot easily find which code does what to the members of `pair`.
This may be exactly what we want, but if we want to enforce a relation among members, we need to make them `private`
and enforce that relation (invariant) through constructors and member functions.

For example:
```c++
    class Distance {
    public:
        // ...
        double meters() const { return magnitude*unit; }
        void set_unit(double u)
        {
                // ... check that u is a factor of 10 ...
                // ... change magnitude appropriately ...
                unit = u;
        }
        // ...
    private:
        double magnitude;
        double unit;    // 1 is meters, 1000 is kilometers, 0.001 is millimeters, etc.
    };
```

##### Note

If the set of direct users of a set of variables cannot be easily determined, the type or usage of that set cannot be (easily) changed/improved.
For `public` and `protected` data, that's usually the case.

##### Example

A class can provide two interfaces to its users.
One for derived classes (`protected`) and one for general users (`public`).
For example, a derived class might be allowed to skip a run-time check because it has already guaranteed correctness:

```c++
    class Foo {
    public:
        int bar(int x) { check(x); return do_bar(x); }
        // ...
    protected:
        int do_bar(int x); // do some operation on the data
        // ...
    private:
        // ... data ...
    };

    class Dir : public Foo {
        //...
        int mem(int x, int y)
        {
            /* ... do something ... */
            return do_bar(x + y); // OK: derived class can bypass check
        }
    };

    void user(Foo& x)
    {
        int r1 = x.bar(1);      // OK, will check
        int r2 = x.do_bar(2);   // error: would bypass check
        // ...
    }
```
##### Note

[`protected` data is a bad idea](#Rh-protected).

##### Note

`public`멤버를 가장 앞에, `protected` 멤버를 다음에, `private`멤버를 마지막에 배치하라.

##### Enforcement

* [Flag protected data](#Rh-protected).
* Flag mixtures of `public` and private `data`

## <a name="SS-concrete"></a>C.concrete: 실제 타입(Concrete types)

이상적인 클래스는 정규(regular) 타입과 같아야 한다.
쉽게 말하면 "`int` 처럼 동작하는 것"이다. 실제 타입(Concrete type)이란 가장 간단한 종류의 클래스를 의미한다.

정규 타입의 값은 복사 될 수 있고, 복사의 결과는 원본과 같은 값을 갖는 독립적인 객체이다. 타입이 `=` 와 `==` 를 모두 갖는다면, `a = b`를 실행한 이후에는 `a == b`에서 `true`가 반환되도록 해야 한다.
Concrete classes without assignment and equality can be defined, but they are (and should be) rare.

C++의 언어 내장(built-in) 타입들은 정규적(regular)이고, `string`, `vector`, `map`같은 표준 라이브러리의 클래스들 또한 그렇다. 실제 타입들은 종종 계층구조의 일부로 사용되는 타입들과 구분하여 값 타입으로 언급된다.

실제 타입 규칙 요약:

* [C.10: 클래스 계층 보다 실제(Concrete) 타입들을 선호하라](#Rc-concrete)
* [C.11: 실제 타입들은 정규적으로 만들어라](#Rc-regular)

### <a name="Rc-concrete"></a>C.10: 클래스 계층 보다 실제(Concrete) 타입들을 선호하라

##### Reason

실제 타입은 근본적으로 계층구조 보다 단순하다:
디자인이 더 쉽고, 구현이 더 쉽고, 사용하기가 더 쉬우며, 추론하기 더 쉽다. 더 작고 더 빠르기도 하다.  
계층구조를 사용할 때는 타당한 이유가 있어야 한다.

##### Example

```c++
    class Point1 {
        int x, y;
        // ... operations ...
        // ... no virtual functions ...
    };

    class Point2 {
        int x, y;
        // ... operations, some virtual ...
        virtual ~Point2();
    };

    void use()
    {
        Point1 p11 {1, 2};   // make an object on the stack
        Point1 p12 {p11};    // a copy

        auto p21 = make_unique<Point2>(1, 2);   // make an object on the free store
        auto p22 = p21.clone();                 // make a copy
        // ...
    }
```

클래스가 계층구조의 일부가 될 수 있다면, 반드시 포인터나 레퍼런스로 객체를 다루어야 한다.
이는 간접 처리를 위해 더 많은 메모리를 사용하게 되고, 더 많은 할당과 해제, 실행시간 오버헤드가 발생하게 된다는 것을 의미한다.

##### Note

실제 타입은 스택에 할당 될 수 있고, 다른 클래스의 멤버가 될 수 있다.

##### Note

실행시간에 다형적 인터페이스를 위해 간접처리는 필수적이다.
할당과 해제의 추가비용은 그렇지 않다. (단지 가장 흔한 사례일 뿐이다)
패생 클래스의 제한된(특정된) 객체에 대한 인터페이스로써 기본 클래스를 사용할 수도 있다.
동적 할당을 할 수 없으며, 플러그인과 같은 것들에게 안정적인 인터페이스를 제공하고자 할 때 이렇게 할 수 있다. (예컨대, hard real-time)

##### Enforcement

???

### <a name="Rc-regular"></a>C.11: 실제 타입들은 정규적으로 만들어라

##### Reason

일반적인(regular) 타입은 이해하고 추론(reason)하기 쉽다.
(일반적이지 않은 타입들은 이해하고 사용하는데 추가적인 노력을 필요로 한다.)

##### Example

```c++
    struct Bundle {
        string name;
        vector<Record> vr;
    };

    bool operator==(const Bundle& a, const Bundle& b)
    {
        return a.name == b.name && a.vr == b.vr;
    }

    Bundle b1 { "my bundle", {r1, r2, r3}};
    Bundle b2 = b1;
    if (!(b1 == b2)) error("impossible!");
    b2.name = "the other bundle";
    if (b1 == b2) error("No!");
```

일반적인 경우, 만약 concrete type이 대입연산(`a = b`)을 지원한다면, 비교 연산(`a == b`)도 지원한다.

##### Enforcement

???

## <a name="S-ctor"></a>C.ctor: 생성자, 대입 연산자, 소멸자

이 함수들은 객체의 생명주기를 제어 한다: 생성, 복사, 이동, 그리고 소멸.
생성자를 정의해서 클래스의 초기화를 보장하고 단순화 하라.

*기본 연산*은 아래와 같은 연산들을 의미한다.

* 기본 생성자: `X()`
* 복사 생성자: `X(const X&)`
* 복사 대입 연산자: `operator=(const X&)`
* 이동 생성자: `X(X&&)`
* 이동 대입 연산자: `operator=(X&&)`
* 소멸자: `~X()`

이상의 연산들은 정의하지 않아도 코드에서 사용되면 컴파일러가 생성한다. but the default can be suppressed.

The default operations are a set of related operations that together implement the lifecycle semantics of an object.

기본적으로, C++은 클래스를 값 타입 처럼 다루지만 모든 타입이 값 타입처럼 동작하는 것은 아니다.

기본 연산 규칙들:

* [C.20:기본 연산을 정의하지 않아도 되면 그렇게 하라](#Rc-zero)
* [C.21: 기본 연산을 정의 하거나 `=delete` 로 선언했다면, 나머지 모두 정의하거나 `=delete`하라](#Rc-five)
* [C.22: 기본 연산들을 일관성 있도록 하라](#Rc-matched)

소멸자 규칙들:

* [C.30: 객체가 없어질 때, 명시적인 동작이 필요할 경우 소멸자를 정의하라](#Rc-dtor)
* [C.31: 클래스에 의해 얻어진 모든 리로스는 소멸자에서 해제되어야 한다](#Rc-dtor-release)
* [C.32: 클래스가 포인터(`T*`)나 참조(`T&`)를 갖고 있을 때, 해당 클래스가 소유하고 있는지를 고려하라](#Rc-dtor-ptr)
* [C.33: 클래스가 포인터 멤버를 소유하고 있다면, 소멸자를 정의하거나 `=delete` 로 선언하라](#Rc-dtor-ptr2)
* [C.35: 가상 함수를 갖는 기본 클래스는 가상 소멸자가 필요하다](#Rc-dtor-virtual)
* [C.36: 소멸자는 실패해선 안된다](#Rc-dtor-fail)
* [C.37: 소멸자를 `noexcept`로 작성하라](#Rc-dtor-noexcept)

생성자 규칙들:

* [C.40: 클래스가 불변 조건을 가지면 생성자를 정의하라](#Rc-ctor)
* [C.41: 생성자는 완전히 초기화된 객체를 생성해야 한다](#Rc-complete)
* [C.42: 생성자가 유효한 객체를 생성하지 못한다면, 예외를 던지도록 하라](#Rc-throw)
* [C.43: 복사 가능한 클래스는 반드시 기본 생성자를 갖도록 하라](#Rc-default0)
* [C.44: 기본 생성자는 되도록 단순하고 예외를 던지지 않도록 하라](#Rc-default00)
* [C.45: 멤버를 초기화 하기만 하는 기본 생성자는 정의하지 마라; 대신 멤버들이 스스로 초기화 하도록 하라](#Rc-default)
* [C.46: 단일 인자를 사용하는 생성자는 `explicit`으로 선언하라](#Rc-explicit)
* [C.47: 멤버 변수들은 선언된 순서대로 초기화 하라](#Rc-order)
* [C.48: 상수 초기화는 가능한 in-calss 멤버 초기화를 사용하라](#Rc-in-class-initializer)
* [C.49: 생성자 안에서의 대입 보다는 초기화를 선호하라](#Rc-initialize)
* [C.50: 초기화 과정에서 `virtual` 연산이 필요하다면, 팩토리 함수를 사용하라](#Rc-factory)
* [C.51: 클래스의 모든 생성자들을 위한 일반적인 동작을 표현할 때는 대리 생성자를 사용하라](#Rc-delegating)
* [C.52: 추가적인 초기화가 필요하지 않은 파생된 클래스에서 생성자를 사용할 때는 상속받은 생성자들을 사용하라](#Rc-inheriting)

복사와 이동 규칙들:

* [C.60: 복사연산을 `virtual`로 만들지 말아라. 매개변수는 `const&`로 받고, `const&`로 반환하지 말아라](#Rc-copy-assignment)
* [C.61: 복사 연산은 복사를 수행해야 한다](#Rc-copy-semantic)
* [C.62: 복사 연산은 자기 대입에 안전하게 작성하라](#Rc-copy-self)
* [C.63: 이동 연산은 `virtual`로 만들지 말아라, 매개변수는 `&&`를 사용하고, `const&`로 반환하지 말아라](#Rc-move-assignment)
* [C.64: 이동 연산은 이동을 수행해야 하며, 원본 개체를 유효한 상태로 남겨놓아야 한다](#Rc-move-semantic)
* [C.65: 이동 연산은 자기 대입에 안전하게 작성하라](#Rc-move-self)
* [C.66: 이동 연산은 `noexcept`로 만들어라](#Rc-move-noexcept)
* [C.67: 다형적인 클래스는 복사를 제한해야 한다](#Rc-copy-virtual)

다른 기본 연산들에 대한 규칙:

* [C.80: 기본 의미구조(semantic)를 명시적으로 사용하려면 `=default`를 사용하라](#Rc-eqdefault)
* [C.81: 기본 동작을 (대안을 원하지 않고) 금지하고 싶다면 `=delete`를 사용하라](#Rc-delete)
* [C.82: 생성자 또는 소멸자에서 가상 함수를 호출하지 말아라](#Rc-ctor-virtual)
* [C.83: 값 타입들에는, `noexcept` swap함수를 제공하는 것을 고려하라](#Rc-swap)
* [C.84: `swap` 연산은 실패해선 안된다](#Rc-swap-fail)
* [C.85: `swap` 연산은 `noexcept`로 작성하라](#Rc-swap-noexcept)
* [C.86: `==`연산자는 피연산자 타입들에 대칭적이고, `noexcept`로 만들어라](#Rc-eq)
* [C.87: 기본 클래스에 있는 `==`에 주의하라](#Rc-eq-base)
* [C.89: `hash`는 `noexcept`로 작성하라](#Rc-hash)

## <a name="SS-defop"></a>C.defop: 기본 연산들(Default Operations)

C++ 에서는 기본적인 의미를 가진 연산들을 제공한다.
프로그래머는 이 연산들을 금지하거나 교체할 수 있다.

### <a name="Rc-zero"></a>C.20: If you can avoid defining default operations, do

##### Reason

가장 단순하고, 명료한 의미를 준다.

##### Example

```c++
    struct Named_map {
    public:
        // ... no default operations declared ...
    private:
        string name;
        map<int, int> rep;
    };

    Named_map nm;        // default construct
    Named_map nm2 {nm};  // copy construct
```

`std::map` 과 `string` 은 모든 특수한 함수들을 갖고 있다, 추가적인 작업이 필요없다.

##### Note

"the rule of zero"로 알려져 있다.

##### Enforcement

(Not enforceable) 시행할 수 없더라도, 좋은 정적 분석기는 이 규칙에 맞는 가능한 개선사항들을 알려주는 패턴들을 찾을 수 있다.
예를 들면, 포인터와 크기를 멤버로 갖는 클래스가 있고 소멸자에서 그 포인터를 `delete` 한다면 아마도 `vector` 로 바꿀 수 있을 것이다.

### <a name="Rc-five"></a>C.21: If you define or `=delete` any default operation, define or `=delete` them all

##### Reason

The *special member functions* are the default constructor, copy constructor,
copy assignment operator, move constructor, move assignment operator, and
destructor.

이 특별한 함수들의 의미는 서로 밀접하게 연관되어 있다. 만약 한 함수가 기본 제공 함수가 아니어야 한다면(non-default), 다른 함수들도 수정이 필요하다.

Declaring any special member function except a default constructor,
even as `=default` or `=delete`, will suppress the implicit declaration
of a move constructor and move assignment operator.
Declaring a move constructor or move assignment operator, even as
`=default` or `=delete`, will cause an implicitly generated copy constructor
or implicitly generated copy assignment operator to be defined as deleted.
So as soon as any of the special functions is declared, the others should
all be declared to avoid unwanted effects like turning all potential moves
into more expensive copies, or making a class move-only.

##### Example, bad

```c++
    struct M2 {   // bad: incomplete set of default operations
    public:
        // ...
        // ... no copy or move operations ...
        ~M2() { delete[] rep; }
    private:
        pair<int, int>* rep;  // zero-terminated set of pairs
    };

    void use()
    {
        M2 x;
        M2 y;
        // ...
        x = y;   // the default assignment
        // ...
    }
```

여기서는, 소멸자에 대한 "특별한 주의"가 필요하다고 한다면, 복사와 이동 할당(둘 다 묵시적으로 개체를 소멸할 것이다)이 정확하게 동작할 가능성은 적다. (여기서는, 두번 `delete`를 시도할 것이다)

##### Note

기본 생성자를 중요하게 생각하는지에 달려있는데, 이것은 "the rule of five" 혹은 "the rule of six" 이라고 알려져 있다.

##### Note

다른 것은 정의 하더라도 기본 연산의 기본 구현이 필요하다면, `=default` 을 사용하여 해당 함수에 대한 의도를 표현하라.
기본 연산을 원하지 않는다면, `=delete`를 써서 제한하라.

##### Example, good

When a destructor needs to be declared just to make it `virtual`, it can be
defined as defaulted. To avoid suppressing the implicit move operations
they must also be declared, and then to avoid the class becoming move-only
(and not copyable) the copy operations must be declared:
```c++
    class AbstractBase {
    public:
      virtual ~AbstractBase() = default;
      AbstractBase(const AbstractBase&) = default;
      AbstractBase& operator=(const AbstractBase&) = default;
      AbstractBase(AbstractBase&&) = default;
      AbstractBase& operator=(AbstractBase&&) = default;
    };
```
Alternatively to prevent slicing as per [C.67](#Rc-copy-virtual),
the copy and move operations can all be deleted:
```c++
    class ClonableBase {
    public:
      virtual unique_ptr<ClonableBase> clone() const;
      virtual ~ClonableBase() = default;
      ClonableBase(const ClonableBase&) = delete;
      ClonableBase& operator=(const ClonableBase&) = delete;
      ClonableBase(ClonableBase&&) = delete;
      ClonableBase& operator=(ClonableBase&&) = delete;
    };
```
Defining only the move operations or only the copy operations would have the
same effect here, but stating the intent explicitly for each special member
makes it more obvious to the reader.

##### Note

컴파일러는 이 규칙을 강제하고, 이상적으로는 위반사항이 발생하면 경고한다.

##### Note

클래스에 묵시적으로 생성된 복사 연산에 의존하는 것은 더 이상 사용되지 않는다.

##### Note

Writing the six special member functions can be error prone.
Note their argument types:
```c++
    class X {
    public:
        // ...
        virtual ~X() = default;            // destructor (virtual if X is meant to be a base class)
        X(const X&) = default;             // copy constructor
        X& operator=(const X&) = default;  // copy assignment
        X(X&&) = default;                  // move constructor
        X& operator=(X&&) = default;       // move assignment
    };
```
A minor mistake (such as a misspelling, leaving out a `const`, using `&` instead of `&&`, or leaving out a special function) can lead to errors or warnings.
To avoid the tedium and the possibility of errors, try to follow the [rule of zero](#Rc-zero).

##### Enforcement

(쉬움) 클래스는 특별한 함수들에 대한 선언(`=delete`도 포함하여)을 모두 갖거나 하나도 없어야 한다.

### <a name="Rc-matched"></a>C.22: Make default operations consistent

##### Reason

기본 연산들은 개념적으로 잘 짜여진 집합이다. 연산들의 의미는 서로 연관되어 있다.
사용자는 복사/이동 생성과 복사/이동 할당이 논리적으로 동일하고, 생성자와 소멸자가 리소스 관리에 대해 일관적으로 동작하며, 복사와 이동이 생성자와 소멸자가 동작하는 방식을 반영한다는 것을 기대 할 것이다. Users will be surprised if copy and move don't reflect the way constructors and destructors work.

##### Example, bad

```c++
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

이 연산들은 복사 연산에 대한 의미가 일치하지 않는다. 이런 동작은 혼란을 야기하고 버그를 만들 것이다.

##### Enforcement

* (어려움) 복사/이동 생성자와 이에 대응하는 복사/이동 할당 연산자는 동일한 레벨에서 동일한 멤버 변수를 변경하는 것이 좋다
* (어려움) 복사/이동 생성자에서 변경하는 멤버 변수들은 다른 생성자들에서도 초기화 하는 것이 좋다
* (어려움) 복사/이동 생성자는 멤버 변수에 대해 깊은 복사를 수행하고 나서, 소멸자는 멤버 변수를 수정해야 한다
* (어려움) 소멸자가 멤버 변수를 변경하면, 그 멤버 변수들은 복사/이동 생성자 혹은 할당 연산자에서 쓰여지는 것이 좋다

## <a name="SS-dtor"></a>C.dtor: 소멸자

"이 클래스에 소멸자가 필요할까?"라는 것은 설계 측면에서 굉장히 강력한 질문이다.
대부분의 클래스들에 대해서 대답은 "no"인데, 그 이유는 해당 클래스가 자원들을 가지고 있지 않거나 소멸과정이 [the rule of zero](#Rc-zero)에 의해 처리되기 때문이다.

요컨대, 클래스의 멤버들이 스스로의 소멸을 관리한다는 것이다.
만약 대답이 "yes"라면, 그 클래스 설계의 대부분은 [the rule of five](#Rc-five)를 따르게 된다.

### <a name="Rc-dtor"></a>C.30: Define a destructor if a class needs an explicit action at object destruction

##### Reason

소멸자는 암묵적으로 객체의 생명주기의 마지막에 호출된다.
기본 소멸자로 충분하다면 그것을 사용하라.
단순하게 멤버의 소멸자를 호출하는 것이 아닌 코드가 필요할 경우 소멸자를 정의하라.

##### Example

```c++
    template<typename A>
    struct final_action {   // slightly simplified
        A act;
        final_action(A a) :act{a} {}
        ~final_action() { act(); }
    };

    template<typename A>
    final_action<A> finally(A act)   // deduce action type
    {
        return final_action<A>{act};
    }

    void test()
    {
        auto act = finally([]{ cout << "Exit test\n"; });  // establish exit action
        // ...
        if (something) return;   // act done here
        // ...
    } // act done here
```

`final_action` 의 목적은 소멸할 때 실행할 코드(보통 람다 표현식을 쓴다)를 얻는 것이다.

##### Note

사용자 정의 소멸자가 필요한 클래스에는 보통 두 종류가 있다:

* 리소스를 사용하는 클래스가 소멸자가 없는 경우, 예컨대 `vector` 혹은 트랜잭션 코드
* A class that exists primarily to execute an action upon destruction, such as a tracer or `final_action`.

##### Example, bad

```c++
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

기본 소멸자가 더 잘 동작하고, 더 효과적이며, 틀리지 않는다.

##### Note

기본 소멸자가 필요하지만, 생성되지 않도록 되어 있다면 (예, 이동 생성자를 정의한 경우), `=default` 를 사용하라.

##### Enforcement

포인터나 참조와 같은 "암묵적인 자원"이 될 수 있는 것들을 찾아보라. 
모든 데이터 멤버가 소멸자를 갖고 있더라도, 사용자 지정 소멸자가 있는 클래스들을 찾아보라.

### <a name="Rc-dtor-release"></a>C.31: All resources acquired by a class must be released by the class's destructor

##### Reason

리소스 누수를 막는다, 특히 오류가 발생한 상황에서 그렇다.

##### Note

클래스로 표현되는 리소스들이 기본 연산 집합을 갖고 있을 때 소멸자에서의 리소스 해제가 자동으로 발생한다.

##### Example

```c++
    class X {
        ifstream f;   // may own a file
        // ... no default operations defined or =deleted ...
    };
```

`X`의 `ifstream` 은 `X`가 소멸될 때 묵시적으로 열었을 수 있는 파일을 닫는다.  

##### Example, bad

```c++
    class X2 {     // bad
        FILE* f;   // may own a file
        // ... no default operations defined or =deleted ...
    };
```

`X2` 에서는 파일 핸들 누수가 생길 것이다.  

##### Note

닫지 않은 소켓은 어떨까? 소멸자, 닫기, 정리 연산은 [실패하지 않는 것이 좋다](#Rc-dtor-fail).
그럼에도 불구하고 발생한다면, 좋은 해결책을 찾기 정말 힘든 문제를 마주친 것이다.
초심자들은 소멸자를 작성할 때 왜 소멸자가 호출되고, 예외를 던짐으로써 "처리를 거부"를 할 수 없는지 알지 못할 것이다. 이에 대해서는 [discussion](#Sd-never-fail)을 보라.

문제를 악화시키는 것은, 많은 "닫기/해제" 연산들이 재시도 할 수 없도록 되어있는 것이다.
이 문제를 풀려는 시도는 많았지만, 일반적인 해결책은 알려지지 않았다.
해결책이 없다면, 닫기/해제에 대한 실패를 디자인 오류로 간주하고 종료시키는 것을 고려해 보라.

##### Note

클래스가 소유하고 있지 않은 객체에 대한 포인터나 참조를 갖고 있을 수 있다.
당연하지만, 이 객체들은 클래스의 소멸자에서 `delete`되지 않아야 한다.
예를 들면:

```c++
    Preprocessor pp { /* ... */ };
    Parser p { pp, /* ... */ };
    Type_checker tc { p, /* ... */ };
```

`p`는 `pp`를 참조하지만, 소유하고 있지 않다.

##### Enforcement

* (쉬움) 클래스가 소유자인 포인터나 참조 멤버 변수를 갖고 있다면 (가령, `gsl::owner`를 사용하여 소유하는 경우), 소멸자에서 참조되는 것이 좋다
* (어려움) 소유권에 대해 명시적으로 기술하지 않은 경우, 포인터나 참조 멤버 변수들이 소유자 인지 판단하라 (예, 생성자 본문을 확인한다).

### <a name="Rc-dtor-ptr"></a>C.32: If a class has a raw pointer (`T*`) or reference (`T&`), consider whether it might be owning

##### Reason

소유권에 대해서 상세하지 않은 코드는 많이 있다.

##### Example

    ???

##### Note

`T*` 혹은 `T&` 가 소유를 의미한다면, **소유한다는** 표시를 하라. `T*` 에 소유의 의미가 없다면 `ptr` 로 표시하는 것을 고려하라.
이것은 문서화와 분석에 도움이 될 것이다.

##### Enforcement

Look at the initialization of raw member pointers and member references and see if an allocation is used.

### <a name="Rc-dtor-ptr2"></a>C.33: If a class has an owning pointer member, define a destructor

##### Reason

소유된 객체는 그것을 소유한 객체가 소멸될 때 `삭제`되어야 한다.

##### Example

포인터 멤버는 리소스일 것이다. [`T*`는 리소스가 아니어야 한다](#Rr-ptr), 이는 오래된 코드에서는 일반적이다.
가능한 `T*`를 소유자라고 고려하고, 의심해보라.

```c++
    template<typename T>
    class Smart_ptr {
        T* p;   // BAD: *p 의 소유가 불분명하다
        // ...
    public:
        // ... 사용자가 복사 연산을 정의하지 않았다 ...
    };

    void use(Smart_ptr<int> p1)
    {
        // error: p2.p 에 누수가 발생한다. (nullptr가 아니거나 다른 코드에서 소유하지 않는다면)
        auto p2 = p1;
    }
```

소멸자를 정의 한다면, [모든 기본 연산들](#Rc-five)을 정의하거나 삭제해야 한다.

```c++
    template<typename T>
    class Smart_ptr2 {
        T* p;   // BAD: *p 의 소유가 불분명하다
        // ...
    public:
        // ... 사용자가 복사 연산을 정의하지 않았다 ...
        ~Smart_ptr2() { delete p; }   // p 가 자원을 소유하고 있었다!
    };

    void use(Smart_ptr2<int> p1)
    {
        auto p2 = p1;    // error: delete가 2번 호출된다.
    }
```

기본 복사 연산은 단지 `p1.p` 를 `p2.p` 로 복사하고, `p1.p` 가 두번 소멸되게 만들 것이다. 소유권을 명시하라:

```c++
    template<typename T>
    class Smart_ptr3 {
        owner<T*> p;  // OK: 명시적으로 *p 의 소유권을 가진다. 
        // ...
    public:
        // ...
        // ... 복사와 이동 연산들 ...
        ~Smart_ptr3() { delete p; }
    };

    void use(Smart_ptr3<int> p1)
    {
        auto p2 = p1;
    }
```

##### Note

보통 소멸자를 사용하는 가장 단순한 방법은 포인터를 스마트 포인터(가령, `std::unique_ptr`)로 교체하고, 컴파일러가
적절한 소멸자를 암묵적으로 호출하게 만들도록 놔두는 것이다.

##### Note

소유하고 있는 모든 포인터를 "스마트 포인터"로 변경하는 것은 어떤가?
드물게 중대한 코드 변경이 필요해지고 ABI 에 영향을 줄 수도 있다.

##### Enforcement

* 포인터 데이터 맴버를 갖는 클래스를 의심하라
* `owner<T>` 를 갖는 클래스는 기본 연산들을 정의 해야한다

### <a name="Rc-dtor-virtual"></a>C.35: A base class destructor should be either public and virtual, or protected and nonvirtual

##### Reason

미정의 동작(undefined behavior)을 막기 위한 규칙이다.

만약 소멸자가 `public` 이면, 호출하는 코드는 파생 클래스가 기본 클래스의 포인터를 통해 소멸될 것이라 생각한다. 그리고 기본 클래스의 소멸자가 `virtual`이 아니면 결과는 미정의 동작으로 이어진다.

만약 소멸자가 `protected`라면, 호출하는 코드는 기본 클래스의 포인터를 통해서 소멸시킬 수 없고, 따라서 소멸자는 `virtual`이 아니어도 문제가 없다. `private`가 아닌 `protected`여야 하는 이유는 파생 클래스의 소멸자가 호출할 수 있어야 하기 때문이다.

일반적으로, 기본 클래스의 작성자는 소멸 과정에서 어떤 동작이 적합한지 알 수 없다.  

##### Discussion

[토론](#Sd-dtor)을 함께 읽어보라.

##### Example, bad

```c++
    struct Base {  // BAD: virtual 소멸자가 없다
        virtual void f();
    };

    struct D : Base {
        string s {"a resource needing cleanup"};
        ~D() { /* ... 정리 작업을 한다 ... */ }
        // ...
    };

    void use()
    {
        unique_ptr<Base> p = make_unique<D>();
        // ...
    }
    // p 의 소멸은 ~Base()를 호출하지만, ~D() 는 호출하지 않는다.
    // 따라서 D::s 에 누수가 발생하고, 다른 자원들도 누수될 것이다.
```

##### Note

가상(`virtual`) 함수는 파생 클래스에 대한 인터페이스를 제공한다. 이 인터페이스를 통해 파생 클래스에 대해 신경을 쓰지 않게 된다.   
만약 인터페이스가 소멸을 지원한다면, 그 과정은 안전해야만 한다.

##### Note

소멸자는 private이 아니어야 한다. 만약 그럴 경우 해당 타입을 사용하지 못하게 될 것이다:

```c++
    class X {
        ~X();   // private 소멸자
        // ...
    };

    void use()
    {
        X a;                        // error: 소멸시킬 수 없다
        auto p = make_unique<X>();  // error: 소멸시킬 수 없다
    }
```

##### Exception

protected virtual 소멸자를 원하지 않는 경우를 상상해볼 수 있다. 파생 타입의 객체가 기본 타입 포인터를 통해 (그 자신이 아닌) *다른* 개체의 소멸을 하도록 허용해야 하는 경우가 그러하다. 하지만 아직까지 그런 사례를 볼 수 없었다.

##### Enforcement

* 가상 함수를 하나라도 가지는 클래스는 `public` 하고 `virtual`한 소멸자를 가져야 한다. 또는 `protected`이고 `virtual`이 아닌 소멸자를 가져야 한다.

### <a name="Rc-dtor-fail"></a>C.36: A destructor may not fail

##### Reason

일반적으로 소멸자가 실패할 때 오류 없는 코드를 작성하는 방법을 알 수 없다.
표준 라이브러리에서 다루는 모든 클래스들은 예외를 던지지 않는 소멸자를 요구한다.

##### Example

```c++
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

##### Note

소멸자에서의 실패를 다루기 위해 실패할 염려가 없는 방법(scheme)을 많이 고안해 왔다. 이에 대해선 일반적인 방법으로 성공한 예가 없다.

이것은 정말 현실적인 문제가 될 수 있다: 예를 들면, 닫지 않은 소켓은 어떤가?  
소멸자를 작성하는 사람은 왜 소멸자가 호출되고 예외를 던짐으로써 "동작을 거부하는 것"을 할 수 없는지 모른다.

[토론](#Sd-dtor)을 함께보라.
문제를 악화시키는 것은, 많은 "close/release" 연산이 재시도할 수 없게 되어있는 것이다.
가능하다면, close/failure에 대한 실패를 근본적인 디자인 오류로 간주하고 종료시켜라.

##### Note

소멸자를 `noexcept`로 선언하라. 이것은 소멸자가 정상적으로 완료했거나 프로그램을 종료한다는 것을 보장한다.

##### Note

만약 자원이 해제될 수 없고 프로그램이 실패하지 않는다면, 어떤 방법으로든 시스템의 나머지 부분에서 실패 했다는 신호를 보내도록 하라.
(전역 상태 변수를 수정하고 프로그램의 다른 부분이 그것을 확인하고 아마도 문제를 처리할 수 있을 것이다)

이 방식은 특별한 목적이 있고, 오류가 발생하기 쉽다는 것을 충분히 이해하라.

예시로 "닫히지 않는 연결"을 고려해보자.
어쩌면 연결의 반대편에 문제가 있을 수 있고, 이때 양쪽의 연결을 담당하는 코드만이 문제를 처리할 수 있다.
소멸자가 (어떤 방법으로) 시스템의 담당(responsible) 부분에 메세지를 보내고, 연결이 닫힌 것으로 간주한 뒤, 정상적으로 반환할 수도 있다.

##### Note

소멸자가 실패할 수도 있는 연산을 사용한다면, 예외를 잡을 수 있고, 어떤 경우에는 성공적으로 완료할 수 있다.
(가령, 예외를 던진 메커니즘과는 다른 정리(clean-up) 메커니즘을 사용하는 것이다)

##### Enforcement

(쉬움) 만약 예외 발생이 가능하면, 소멸자는 `noexcept`로 선언되어야 한다

### <a name="Rc-dtor-noexcept"></a>C.37: Make destructors `noexcept`

##### Reason

[소멸자는 실패해선 안된다](#Rc-dtor fail).  
만약 소멸자가 예외로 인해 종료되려고 한다면, 좋지 않은 디자인 오류로 보고 종료하는 편이 나을 것이다.

##### Note

A destructor (either user-defined or compiler-generated) is implicitly declared `noexcept` (independently of what code is in its body) if all of the members of its class have `noexcept` destructors. By explicitly marking destructors `noexcept`, an author guards against the destructor becoming implicitly `noexcept(false)` through the addition or modification of a class member.

##### Example

Not all destructors are noexcept by default; one throwing member poisons the whole class hierarchy

```c++
    struct X {
        Details x;  // happens to have a throwing destructor
        // ...
        ~X() { }    // implicitly noexcept(false); aka can throw
    };
```

So, if in doubt, declare a destructor noexcept.

##### Note

Why not then declare all destructors noexcept?
Because that would in many cases -- especially simple cases -- be distracting clutter.

##### Enforcement

(쉬움) 소멸자는 `noexcept`로 선언되어야 한다.

## <a name="SS-ctor"></a>C.ctor: 생성자

생성자는 개체가 생성되는(초기화되는) 방법을 정의 한다.

### <a name="Rc-ctor"></a>C.40: Define a constructor if a class has an invariant

##### Reason

생성자가 존재하는 이유다.

##### Example

```c++
    class Date {  // Date 클래스는 유효한 날짜를 표현한다
                  // 범위 : 1900년 1월 1일 ~ 2100년 12월 31일
        Date(int dd, int mm, int yy)
            :d{dd}, m{mm}, y{yy}
        {
            if (!is_valid(d, m, y))
                throw Bad_date{};  // 불변조건을 강제한다
        }
        // ...
    private:
        int d, m, y;
    };
```

생성자에서 `Ensures`로 불변조건을 표현하는 것도 좋은 생각이다.

##### Note

생성자는 클래스가 불변조건이 아니더라도 편의를 위해 사용될 수 있다:

```c++
    struct Rec {
        string s;
        int i {0};
        Rec(const string& ss) : s{ss} {}
        Rec(int ii) :i{ii} {}
    };

    Rec r1 {7};
    Rec r2 {"Foo bar"};
```

##### Note

C++11 초기화 리스트 규칙은 많은 생성자의 필요성을 제거한다:

```c++
    struct Rec2{
        string s;
        int i;
        Rec2(const string& ss, int ii = 0) :s{ss}, i{ii} {}   // redundant
    };

    Rec2 r1 {"Foo", 7};
    Rec2 r2 {"Bar"};
```

`Rec2` 생성자는 중복적이다. 그리고, `int`에 대한 기본값은 [member initializer](#Rc-in-class initializer)를 사용하는 편이 났다.

**See also**:

* [유효한 객체를 생성하라](#Rc-complete)
* [생성자가 던지는 예외](#Rc-throw)

##### Enforcement

* 사용자 정의 복사 연산이 있지만 소멸자가 없는 클래스를 지적한다 (사용자 정의 복사는 클래스가 불변조건을 가진다는 것을 알려준다)

### <a name="Rc-complete"></a>C.41: A constructor should create a fully initialized object

##### Reason

생성자는 클래스에 대한 불변조건을 설정한다. 클래스 사용자는 생성된 객체가 사용가능하다는 것을 가정할 수 있어야 한다.

##### Example, bad

```c++
    class X1 {
        FILE* f;   // 다른 함수에 앞서 init()을 호출한다
        // ...
    public:
        X1() {}
        void init();   // 멤버 f 초기화
        void read();   // 멤버 f 로부터 읽는다
        // ...
    };

    void f()
    {
        X1 file;
        file.read();   // crash 또는 bad read 가 발생한다.
        // ...
        file.init();   // 초기화 하기엔 너무 늦었다
        // ...
    }
```
컴파일러는 주석을 읽지 않는다.

##### Exception

생성자만으로 유효한 객체를 쉽게 만들 수 없다면 [팩토리 함수를 사용하라](#C factory)

##### Enforcement

* (Simple) Every constructor should initialize every member variable (either explicitly, via a delegating ctor call or via default construction).
* (Unknown) If a constructor has an `Ensures` contract, try to see if it holds as a postcondition.

##### Note

생성자가 유효한 객체를 만들기 위해 자원을 얻는다면, 리소스는 [소멸자에 의해 해제](#Rc-release)되어야 한다.
생성자에서 자원을 얻고 소멸자에서 자원을 해제하는 것을 [RAII](Rr-raii) ("Resource Acquisitions Is Initialization") 라고 한다.

### <a name="Rc-throw"></a>C.42: 생성자가 유효한 개체를 생성하지 못한다면, 예외를 던지도록 하라

##### Reason

유효하지 않은 객체를 남겨두는 것은 문제를 일으킬 것이다.

##### Example

```c++
    class X2 {
        FILE* f;
        // ...
    public:
        X2(const string& name)
            :f{fopen(name.c_str(), "r")}
        {
            if (!f) throw runtime_error{"could not open" + name};
            // ...
        }

        void read();      // 멤버 f 로부터 읽는다
        // ...
    };

    void f()
    {
        X2 file {"Zeno"}; // file이 열려있지 않으면 예외를 던진다
        file.read();      // 문제 없다
        // ...
    }
```

##### Example, bad

```c++
    class X3 {     // bad: 생성자가 유효하지 않은 객체를 남겨놓을 수 있다
        FILE* f;   // call is_valid() before any other function
        bool valid;
        // ...
    public:
        X3(const string& name)
            :f{fopen(name.c_str(), "r")}, valid{false}
        {
            if (f) valid = true;
            // ...
        }

        bool is_valid() { return valid; }
        void read();   // 멤버 f 로부터 읽는다
        // ...
    };

    void f()
    {
        X3 file {"Heraclides"};
        file.read();   // crash 또는 bad read가 발생한다!
        // ...
        if (file.is_valid()) {
            file.read();
            // ...
        }
        else {
            // ... 오류를 처리한다 ...
        }
        // ...
    }
```

##### Note

변수를 정의할 때는 (가령, 스택에 혹은 다른 객체의 멤버로써) 오류코드가 리턴되는 명시적인 함수 호출은 없다.
유효하지 않은 객체를 남겨두고 사용하기 전에 지속적으로 `is_valid()` 함수를 호출해야 하는 것은 번거롭고, 오류가 발생하기 쉬우며, 비효율적 이다.

##### Exception

(추가적인 툴 지원 없이) 예외 처리가 예측 가능한 시간 내로 수행되어야 하는 실시간 시스템(비행기 제어를 생각해 보라)과 같은 분야도 있다.

이런 경우엔 `is_valid()` 와 같은 방법이 반드시 사용되어야 한다. 이와 같은 경우 [RAII](#Rc-raii)처럼 동작하도록 하기 위해 지속적으로 `is_valid()` 로 확인하라.

##### Alternative

"생성자 이후 초기화" 혹은 "두 단계 초기화"를 사용해야 할 것 같다면, 그렇게 하지 않도록 해보라. 정말로 그렇게 해야 한다면 [팩토리 함수](#Rc-factory)를 검토하라.

##### Note

사람들이 생성자에서 초기화를 수행하지 않고 `init()`함수를 사용해온 이유 중 하나는 코드의 중복을 막기 위함이었다.
[대리 생성자](#Rc-delegating)와 [기본 멤버 초기화](#Rc-in-class-initializer)가 이런 작업을 더 잘 해낼 수 있다.

또 다른 이유로는 객체가 필요할 때까지 초기화를 지연시키는 것이다; 이러한 해법은 보통 [변수가 적절하게 초기화되기 전까지는 해당 변수를 선언하지 않는 것이다](#Res-init).

##### Enforcement

???

### <a name="Rc-default0"></a>C.43: Ensure that a copyable (value type) class has a default constructor

##### Reason

많은 언어나 라이브러리들이 기본 생성자에 의존하고 있다.  
예를 들면, `T a[10]` 나 `std::vector<T> v(10)` 는 기본 생성자들이 각 요소를 초기화 한다.
A default constructor often simplifies the task of defining a suitable [moved-from state](#???) for a type that is also copyable.

##### Note

A [value type](#SS-concrete) is a class that is copyable (and usually also comparable).
It is closely related to the notion of Regular type from [EoP](http://elementsofprogramming.com/) and [the Palo Alto TR](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3351.pdf).

##### Example

```c++
    class Date { // BAD: 기본 생성자가 없다
    public:
        Date(int dd, int mm, int yyyy);
        // ...
    };

    vector<Date> vd1(1000);   // Date의 기본 값이 필요하다
    vector<Date> vd2(1000, Date{Month::October, 7, 1885});   // 대안
```

기본 생성자는 다른 사용자 정의 생성자가 없을 때만 자동으로 생성된다. 때문에 이런 코드와 같은 경우엔 `vd1`을 초기화 하는 것은 불가능하다.
기본값이 없다는 것은 사용자에게는 날벼락 같은 상황일 수 있으며 타입의 사용을 어렵게 한다. 때문에 하나라도 의미있게 정의할 수 있다면, 정의되어야 한다.

`Date` is chosen to encourage thought:
There is no "natural" default date (the big bang is too far back in time to be useful for most people), so this example is non-trivial.
`{0, 0, 0}` is not a valid date in most calendar systems, so choosing that would be introducing something like floating-point's `NaN`.
하지만, 대부분의 현실적인 `Date` 클래스는 "첫째 날" (가령. 1970년 1월 1일이 많이 쓰인다)을 갖기 때문에 이것을 기본으로 사용하는 것이 일반적이다.

```c++
    class Date {
    public:
        Date(int dd, int mm, int yyyy);
        Date() = default; // [See also](#Rc-default)
        // ...
    private:
        int dd = 1;
        int mm = 1;
        int yyyy = 1970;
        // ...
    };

    vector<Date> vd1(1000);
```

##### Note

클래스의 모든 멤버들이 기본 생성자들을 가지고 있을 경우 묵시적으로 기본 생성자를 가진다:

```c++
    struct X {
        string s;
        vector<int> v;
    };

    X x; // means X{{}, {}}; that is the empty string and the empty vector
```

기본(built-in) 타입들은 적절하게 기본 생성(default construct)되지 않을 수도 있다:

```c++
    struct X {
        string s;
        int i;
    };

    void f()
    {
        X x;    // x.s is initialized to the empty string; x.i is uninitialized

        cout << x.s << ' ' << x.i << '\n';
        ++x.i;
    }
```

정적으로 할당된 내장 타입 객체들은 `0`으로 초기화 된다. 하지만 지역 변수들은 그렇지 않다.  
컴파일러의 최적화 빌드는 내장 타입 지역 변수들을 초기화하지 않을 수 있다는 점에 주의하라. 따라서, 위의 예시와 같은 코드가 나타난다면, 미정의 동작을 일으킬 수 있다.
초기화를 하고자 한다면, 명시적 기본 생성이 도움이 될 것이다:

```c++
    struct X {
        string s;
        int i {};   // 기본 초기화 (i는 0 이 된다)
    };
```

##### Notes

Classes that don't have a reasonable default construction are usually not copyable either, so they don't fall under this guideline.

For example, a base class is not a value type (base classes should not be copyable) and so does not necessarily need a default constructor:

```c++
    // Shape is an abstract base class, not a copyable value type.
    // It may or may not need a default constructor.
    struct Shape {
        virtual void draw() = 0;
        virtual void rotate(int) = 0;
        // =delete copy/move functions
        // ...
    };
```

A class that must acquire a caller-provided resource during construction often cannot have a default constructor, but it does not fall under this guideline because such a class is usually not copyable anyway:

```c++
    // std::lock_guard is not a copyable value type.
    // It does not have a default constructor.
    lock_guard g {mx};  // guard the mutex mx
    lock_guard g2;      // error: guarding nothing
```

A class that has a "special state" that must be handled separately from other states by member functions or users causes extra work
(and most likely more errors). Such a type can naturally use the special state as a default constructed value, whether or not it is copyable:

```c++
    // std::ofstream is not a copyable value type.
    // It does happen to have a default constructor
    // that goes along with a special "not open" state.
    ofstream out {"Foobar"};
    // ...
    out << log(time, transaction);
```

Similar special-state types that are copyable, such as copyable smart pointers that have the special state "==nullptr", should use the special state as their default constructed value.

However, it is preferable to have a default constructor default to a meaningful state such as `std::string`s `""` and `std::vector`s `{}`.

##### Enforcement

* Flag classes that are copyable by `=` without a default constructor
* Flag classes that are comparable with `==` but not copyable

### <a name="Rc-default00"></a>C.44: Prefer default constructors to be simple and non-throwing

##### Reason

실패할 수 있는 연산없이 "기본"적인 값을 설정할 수 있다는 것은 오류 처리를 단순화 하고, 이동 연산을 추측 할 수 있도록 한다.

##### Example, problematic

```c++
    template<typename T>
    // elem은 공간에 대한 포인터다 - new를 사용해 원소들이 할당된다.
    class Vector0 {
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

일반적이지만, 오류 이후 `Vector0` 를 공백으로 만드는 것은 할당과 관련이 있고, 실패할 수 있다.
또, 기본 `Vector` 를 `{ new T[0], 0, 0}` 으로 표현하는 것 역시 낭비처럼 보인다
예를 들면, `Vector0<int> v(100)`은 100 만큼 할당하는 비용이 든다.

##### Example

```c++
    template<typename T>
    // elem은 nullptr이거나, new를 사용해 할당된 공간을 가리킨다.
    class Vector1 {
    public:
        // {nullptr, nullptr, nullptr}과 동일하다. 예외를 던지지 않는다.
        Vector1() noexcept {}
        Vector1(int n) :elem{new T[n]}, space{elem + n}, last{elem} {}
        // ...
    private:
        own<T*> elem = nullptr;
        T* space = nullptr;
        T* last = nullptr;
    };
```

`{nullptr, nullptr, nullptr}`는 `Vector1{}` 를 만드는 비용을 줄여준다(cheap). 하지만 이는 특별한 경우이고 실행시간 평가가 필요하다.
오류를 발견하고 `Vector1`를 비우는 것은 간단하다.

##### Enforcement

* 예외를 던지는 기본 생성자를 지적한다

### <a name="Rc-default"></a>C.45: 멤버를 초기화 하기만 하는 기본 생성자는 정의하지 마라; 대신 멤버들이 스스로 초기화 하도록 하라

##### Reason

멤버들에게 초기화를 위임하면, 컴파일러가 효율적인 코드를 생성한다. 더 효율적일 수 있다.

##### Example, bad

```c++
    class X1 { // BAD: 멤버 초기화를 사용하지 않는다
        string s;
        int i;
    public:
        X1() :s{"default"}, i{1} { }
        // ...
    };
```

##### Example

```c++
    class X2 {
        string s = "default";
        int i = 1;
    public:
        // 컴파일러가 생성한 기본 생성자를 사용한다.
        // ...
    };
```

##### Enforcement

(쉬움) 명시적인 기본 생성자는 초기화 이외의 동작을 해야할 때 쓰는 것이 좋다.

### <a name="Rc-explicit"></a>C.46: By default, declare single-argument constructors explicit

##### Reason

의도치 않은 변환을 피한다.

##### Example, bad

```c++
    class String {
        // ...
    public:
        String(int);   // BAD
        // ...
    };

    String s = 10;   // 놀랍게도... 길이가 10인 문자열이 된다
```

##### Exception

If you really want an implicit conversion from the constructor argument type to the class type, don't use `explicit`:

```c++
    class Complex {
        // ...
    public:
        Complex(double d);   // OK: we want a conversion from d to {d, 0}
        // ...
    };

    Complex z = 10.7;   // unsurprising conversion
```

**See also**: [Discussion of implicit conversions](#Ro-conversion)

##### Note

Copy and move constructors should not be made `explicit` because they do not perform conversions. Explicit copy/move constructors make passing and returning by value difficult.

##### Enforcement

(Simple) Single-argument constructors should be declared `explicit`. Good single argument non-`explicit` constructors are rare in most code based. Warn for all that are not on a "positive list".

### <a name="Rc-order"></a>C.47: Define and initialize member variables in the order of member declaration

##### Reason

To minimize confusion and errors. That is the order in which the initialization happens (independent of the order of member initializers).

##### Example, bad

```c++
    class Foo {
        int m1;
        int m2;
    public:
        Foo(int x) :m2{x}, m1{++x} { }   // BAD: misleading initializer order
        // ...
    };

    Foo x(1); // surprise: x.m1 == x.m2 == 2
```

##### Enforcement

(Simple) A member initializer list should mention the members in the same order they are declared.

**See also**: [Discussion](#Sd-order)

### <a name="Rc-in-class-initializer"></a>C.48: Prefer in-class initializers to member initializers in constructors for constant initializers

##### Reason

Makes it explicit that the same value is expected to be used in all constructors. Avoids repetition. Avoids maintenance problems. It leads to the shortest and most efficient code.

##### Example, bad

```c++
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

##### Example

```c++
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

**Alternative**: We can get part of the benefits from default arguments to constructors, and that is not uncommon in older code. However, that is less explicit, causes more arguments to be passed, and is repetitive when there is more than one constructor:

```c++
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

##### Enforcement

* (Simple) Every constructor should initialize every member variable (either explicitly, via a delegating ctor call or via default construction).
* (Simple) Default arguments to constructors suggest an in-class initializer may be more appropriate.

### <a name="Rc-initialize"></a>C.49: Prefer initialization to assignment in constructors

##### Reason

An initialization explicitly states that initialization, rather than assignment, is done and can be more elegant and efficient. Prevents "use before set" errors.

##### Example, good

```c++
    class A {   // Good
        string s1;
    public:
        A() : s1{"Hello, "} { }    // GOOD: directly construct
        // ...
    };
```

##### Example, bad

```c++
    class B {   // BAD
        string s1;
    public:
        B() { s1 = "Hello, "; }   // BAD: default constructor followed by assignment
        // ...
    };

    class C {   // UGLY, aka very bad
        int* p;
    public:
        C() { cout << *p; p = new int{10}; }   // accidental use before initialized
        // ...
    };
```

### <a name="Rc-factory"></a>C.50: Use a factory function if you need "virtual behavior" during initialization

##### Reason

If the state of a base class object must depend on the state of a derived part of the object, we need to use a virtual function (or equivalent) while minimizing the window of opportunity to misuse an imperfectly constructed object.

##### Note

The return type of the factory should normally be `unique_ptr` by default; if some uses are shared, the caller can `move` the `unique_ptr` into a `shared_ptr`. However, if the factory author knows that all uses of the returned object will be shared uses, return `shared_ptr` and use `make_shared` in the body to save an allocation.

##### Example, bad

```c++
    class B {
    public:
        B()
        {
            // ...
            f();   // BAD: virtual call in constructor
            // ...
        }

        virtual void f() = 0;

        // ...
    };
```

##### Example

```c++
    class B {
    protected:
        B() { /* ... */ }              // create an imperfectly initialized object

        virtual void PostInitialize()  // to be called right after construction
        {
            // ...
            f();    // GOOD: virtual dispatch is safe
            // ...
        }

    public:
        virtual void f() = 0;

        template<class T>
        static shared_ptr<T> Create()  // interface for creating shared objects
        {
            auto p = make_shared<T>();
            p->PostInitialize();
            return p;
        }
    };

    class D : public B { /* ... */ };  // some derived class

    shared_ptr<D> p = D::Create<D>();  // creating a D object
```

By making the constructor `protected` we avoid an incompletely constructed object escaping into the wild.
By providing the factory function `Create()`, we make construction (on the free store) convenient.

##### Note

Conventional factory functions allocate on the free store, rather than on the stack or in an enclosing object.

**See also**: [Discussion](#Sd-factory)

### <a name="Rc-delegating"></a>C.51: Use delegating constructors to represent common actions for all constructors of a class

##### Reason

To avoid repetition and accidental differences.

##### Example, bad

```c++
    class Date {   // BAD: repetitive
        int d;
        Month m;
        int y;
    public:
        Date(int ii, Month mm, year yy)
            :i{ii}, m{mm}, y{yy}
            { if (!valid(i, m, y)) throw Bad_date{}; }

        Date(int ii, Month mm)
            :i{ii}, m{mm} y{current_year()}
            { if (!valid(i, m, y)) throw Bad_date{}; }
        // ...
    };
```

The common action gets tedious to write and may accidentally not be common.

##### Example

```c++
    class Date2 {
        int d;
        Month m;
        int y;
    public:
        Date2(int ii, Month mm, year yy)
            :i{ii}, m{mm}, y{yy}
            { if (!valid(i, m, y)) throw Bad_date{}; }

        Date2(int ii, Month mm)
            :Date2{ii, mm, current_year()} {}
        // ...
    };
```

**See also**: If the "repeated action" is a simple initialization, consider [an in-class member initializer](#Rc-in-class-initializer).

##### Enforcement

(Moderate) Look for similar constructor bodies.

### <a name="Rc-inheriting"></a>C.52: Use inheriting constructors to import constructors into a derived class that does not need further explicit initialization

##### Reason

If you need those constructors for a derived class, re-implementing them is tedious and error-prone.

##### Example

`std::vector` has a lot of tricky constructors, so if I want my own `vector`, I don't want to reimplement them:

```c++
    class Rec {
        // ... data and lots of nice constructors ...
    };

    class Oper : public Rec {
        using Rec::Rec;
        // ... no data members ...
        // ... lots of nice utility functions ...
    };
```

##### Example, bad

```c++
    struct Rec2 : public Rec {
        int x;
        using Rec::Rec;
    };

    Rec2 r {"foo", 7};
    int val = r.x;   // uninitialized
```

##### Enforcement

Make sure that every member of the derived class is initialized.

## <a name="SS-copy"></a>C.copy: 복사(Copy)와 이동(Move)

값 타입들은 일반적으로 복사 가능해야 한다. 하지만 클래스 계층에서의 인터페이스들은 그렇지 않아야 한다.  
리소스 핸들의 경우, 복사가 가능할 수도, 그렇지 않을 수도 있다.  
타입들은 논리적인 또는 성능 상의 이유로 이동하도록 정의될 수 있다.

### <a name="Rc-copy-assignment"></a>C.60: Make copy assignment non-`virtual`, take the parameter by `const&`, and return by non-`const&`

##### Reason

이렇게 하는 것이 간단하고 효율적이다. r-value를 위해 최적화하길 원한다면, `&&`를 받는 대입 연산을 오버로드하여 제공하라. (see [F.18](#Rf-consume)).

##### Example

```c++
    class Foo {
    public:
        Foo& operator=(const Foo& x)
        {
            // GOOD: 자기대입 검사를 할 필요가 없다. (성능은 어찌되었든)
            auto tmp = x;
            std::swap(*this, tmp);
            return *this;
        }
        // ...
    };

    Foo a;
    Foo b;
    Foo f();

    a = b;    // l-value 대입 : 복사
    a = f();  // r-value 대입 : 이동일수도 있다
```

##### Note

`swap`함수의 구현은 [강한 예외 안전성 보장](#Abrahams01)을 가능하게 한다.

##### Example

하지만 만약 임시 사본을 만들지 않음으로써 훨씬 더 좋은 성능을 얻을 수 있다면 어떨까? 
크고 같은 크기의 `Vector`들의 대입이 빈번한 영역을 위한 간단한 `Vector`를 생각해보라.  

이 경우, `swap`구현 기법에 의한 원소들의 사본은 상당한 비용 증가를 야기할 수 있다:

```c++
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
        if (a.sz > sz) {
            // ... swap함수 기법을 사용한다. 이러면 최상의 구현이 된다 ...
            return *this
        }
        // ... copy sz elements from *a.elem to elem ...
        if (a.sz < sz) {
            // ... *a.elem으로부터 elem으로 sz만큼 원소들을 복사한다 ...
        }
        return *this;
    }
```

대상 원소들에 직접 쓰기 연산을 함으로써, `swap`기법이 제공하는 강한 예외 보장 대신 [기본적인 예외 보장](#Abrahams01)만 얻게 될 것이다.  
[자기 대입](#Rc-copy-self)에 주의하라.

**Alternatives**: 만약 `virtual` 대입 연산자가 필요하다고 생각한다면, 그리고 그것이 어째서 문제를 야기할 수 있는지 이해한다면, 그 함수는 `operator=`라고 부르지 마라. 이름을 부여해서 `virtual void assign(const Foo&)`로 만들어라.
[복사 생성 vs. `clone()`](#Rc-copy-virtual)를 참조하라.

##### Enforcement

* (쉬움) 대입 연산자는 가상함수여서는 안된다. 드래곤들만큼 위험하다!
* (쉬움) 대입 연산자는 `T&`를 반환하면 안된다. 연쇄적인 호출을 위해선, 컨테이너로의 객체 대입과 코드 작성을 방해하는 `const T&`를 사용하지 말아라.
* (중간) 대입 연산자는 (암시적으로나 명시적으로나) 모든 기본 클래스와 멤버들의 대입 연산자를 호출해야 한다. 해당 타입이 포인터 문맥이나 값 문맥을 가지는지 확인하기 위해 소멸자를 확인하라.

### <a name="Rc-copy-semantic"></a>C.61: 복사 연산은 복사를 수행해야 한다

##### Reason

일반적으로 그렇게 할 것이라 생각한다. `x = y`가 수행된 후에는, `x == y`인 결과를 가져야 한다.
복사 후에는 `x`와 `y`가 독립적인 개체들일 수 있다. (값 의미구조, 비-포인터 기본 타입들과 표준 라이브러리 타입들의 동작하는 방식) 또는 공유된 객체를 참조한다(포인터 의미구조, 포인터들이 동작하는 방식).

##### Example

```c++
    class X {   // OK: value semantics
    public:
        X();
        X(const X&);     // copy X
        void modify();   // change the value of X
        // ...
        ~X() { delete[] p; }
    private:
        T* p;
        int sz;
    };

    bool operator==(const X& a, const X& b)
    {
        return a.sz == b.sz && equal(a.p, a.p + a.sz, b.p, b.p + b.sz);
    }

    X::X(const X& a)
        :p{new T[a.sz]}, sz{a.sz}
    {
        copy(a.p, a.p + sz, p);
    }

    X x;
    X y = x;
    if (x != y) throw Bad{};
    x.modify();
    if (x == y) throw Bad{};   // assume value semantics
```

##### Example

```c++
    class X2 {  // OK: pointer semantics
    public:
        X2();
        X2(const X2&) = default; // shallow copy
        ~X2() = default;
        void modify();          // change the pointed-to value
        // ...
    private:
        T* p;
        int sz;
    };

    bool operator==(const X2& a, const X2& b)
    {
        return a.sz == b.sz && a.p == b.p;
    }

    X2 x;
    X2 y = x;
    if (x != y) throw Bad{};
    x.modify();
    if (x != y) throw Bad{};  // assume pointer semantics
```

##### Note

"스마트 포인터"를 만들고 있지 않다면 복사 의미구조을 선호하라. 값 의미구조은 가장 간단하며, 표준 라이브러리의 기능들이 기대하는 것이다.

##### Enforcement

(특별히 없음)

### <a name="Rc-copy-self"></a>C.62: 복사 연산은 자기 대입에 안전하게 작성하라

##### Reason

`x = x`의 수행이 `x`의 값을 바꾼다면, 사람들은 놀랄 것이며 안좋은 에러들이 발생할 수 있다 (종종 자원 누수를 포함하기도 한다).

##### Example

표준 라이브러리 컨테이너들은 자기 대입을 우아하고 효율적인 방법으로 처리한다.

```c++
    std::vector<int> v = {3, 1, 4, 1, 5, 9};
    v = v;
    // the value of v is still {3, 1, 4, 1, 5, 9}
```

##### Note

멤버들로부터 생성된 기본 대입 연산은 자기 대입에 안전하다.

```c++
    struct Bar {
        vector<pair<int, int>> v;
        map<string, int> m;
        string s;
    };

    Bar b;
    // ...
    b = b;   // 정확하고, 효율적이다
```

##### Note

자기 대입을 명시적으로 검사함으로써 처리할 수도 있을 것이다. 하지만 종종 그런 검사 없이도 우아하고 빠르게 동작하도록 할 수 있다 (가령, [`swap` 사용법](#Rc-swap)).

```c++
    class Foo {
        string s;
        int i;
    public:
        Foo& operator=(const Foo& a);
        // ...
    };

    Foo& Foo::operator=(const Foo& a)   // OK, but there is a cost
    {
        if (this == &a) return *this;
        s = a.s;
        i = a.i;
        return *this;
    }
```

이 방법은 분명 안전하고 효율적이다.
하지만, 만약 백만번 마다 한번씩 자기 대입을 한다면 어떻겠는가?  
그 말은 백만번이나 장황한 검사를해야 한다는 것과 같다 (하지만 자기 대입의 결과는 반드시 자신과 같아야 하기 때문에, 컴퓨터의 분기 예측은 매번 맞아떨어질 것이다.  

이런 코드를 생각해볼 수 있다:

```c++
    Foo& Foo::operator=(const Foo& a)       // 간단하고, 아마도 훨씬 나을 것이다.
    {
        s = a.s;
        i = a.i;
        return *this;
    }
```

`std::string`은 자기 대입에 안전하고, `int` 역시 안전하다. (희소하게 발생하는) 자기 대입에 대해서만 비용이 발생하게 된다.

##### Enforcement

* (쉬움) 대입 연산자들은 `if (this == &a) return *this;`와 같은 패턴이 있어선 안된다.  

### <a name="Rc-move-assignment"></a>C.63: 이동 연산은 `virtual`로 만들지 말아라, 매개변수는 `&&`를 사용하고, `const&`로 반환하지 말아라

##### Reason

간단하고, 효율적이다.

**See**: [복사 대입을 위한 규칙들](#Rc-copy-assignment).

##### Enforcement

[복사 대입](#Rc-copy-assignment)에서와 동일하다.  

* (쉬움) 대입 연산자는 가상 함수여서는 안된다. 드래곤들만큼 위험하다!
* (쉬움) 대입 연산자는 `T&`를 반환하면 안된다. 연쇄적인 호출을 위해선, 컨테이너로의 객체 대입과 코드 작성을 방해하는 `const T&`를 사용하지 말아라.
* (중간) 이동 연산자는 (암시적으로나 명시적으로나) 모든 기본 클래스와 멤버들의 이동 연산자를 호출해야 한다.  

### <a name="Rc-move-semantic"></a>C.64: 이동 연산은 이동을 수행해야 하며, 원본 객체를 유효한 상태로 남겨놓아야 한다

##### Reason

일반적으로 기대하는 의미구조(semantics)이다. `x = std::move(y)`를 수행한 후에는, `x`의 값은 `y`여야 하며, `y`는 유효한 상태여야 한다.

##### Example

```c++
    template<typename T>
    class X {   // OK: value semantics
    public:
        X();
        X(X&& a) noexcept;  // X를 이동한다
        void modify();      // X의 값을 변경한다
        // ...
        ~X() { delete[] p; }
    private:
        T* p;
        int sz;
    };

    X::X(X&& a)
        :p{a.p}, sz{a.sz}  // 값을 가져간다
    {
        a.p = nullptr;     // empty 상태가 된다
        a.sz = 0;
    }

    void use()
    {
        X x{};
        // ...
        X y = std::move(x);
        x = X{};   // OK
    } // OK: x 는 소멸 가능하다
```

##### Note

이상적으로는, 이동연산을 해준 객체는 해당 타입의 기본 값이어야 한다. 그렇지 않아야 하는 이유가 있지 않는한 기본 값을 가지도록 확실히 하라. 
하지만, 모든 타입들이 기본 값을 가지는 것은 아니며, 또 일부 타입들에서는 기본 값을 만드는 것이 비싼 비용을 필요로 할 수도 있다.
표준에서 요구하는 것은, 이동연산을 해준 객체가 파괴될 수 있다는 것 뿐이다.  
종종, 쉽고 비용이 들지 않는 방법을 쓸수도 있다: 표준 라이브러리는 객체로부터 이동을 받을 수 있다고 가정한다. 이동을 해주는 객체는 유효한 상태로 (필요하다면 명시하여) 남겨놓아라.  

##### Note

이 가이드라인을 적용하지 않아야 할 예외적인 이유가 있지 않는 한, `x = std::move(y); y = z;`를 사용하라. 전통적인 의미구조에 부합한다.

##### Enforcement

(자유선택) 이동 연산에서 멤버들의 대입을 확인해보라. 기본 생성자가 있다면, 그 대입 연산들을 기본 생성자를 사용한 초기화와 비교해보라.  

### <a name="Rc-move-self"></a>C.65: 이동 연산은 자기 대입에 안전하게 작성하라

##### Reason

만약 `x = x`가 `x`의 값을 바꾼다면, 사람들은 놀랄 것이고 안좋은 에러들이 발생할 수 있다. 사람들은 주로 자기 대입을 이동연산으로 작성하지 않지만, 그럴 수도 있다. 
예를 들어, `std::swap`은 이동 연산들로 구현되었고 만약 당신이 우연히  `a`와 `b`가 같은 객체를 참조하는 상황에서 `swap(a, b)`를 사용한다면, 자기-이동의 실패는 심각하거나 찾기 어려운(subtle) 에러가 될 수 있다.

##### Example

```c++
    class Foo {
        string s;
        int i;
    public:
        Foo& operator=(Foo&& a);
        // ...
    };

    Foo& Foo::operator=(Foo&& a) noexcept   // OK, 하지만 비용이 든다
    {
        if (this == &a) return *this;       // 이 라인은 무의미하다
        s = std::move(a.s);
        i = a.i;
        return *this;
    }
```

백만번에 한번 발생하는 `if (this == &a) return *this;`에 대한 논쟁이 있다. [자기 대입](#Rc-copy-self) 항목에서의 논의는 자기 이동과 더 관련이 있다.

##### Note

`if (this == &a) return *this;`을 쓰지 않는 방법은 알려진 것이 없다. 이동 대입 연산에서 검사를 수행하고 정확한 결과를 얻으라.(가령, `x=x`를 수행한 뒤에 `x`가 변화하지 않는다.)  

##### Note

ISO 표준은 표준 라이브러리 컨테이너들에 대해 오직 "유효하지만 명시되지는 않은" 상태만을 보장한다. 이것은 10여년간의 실험적인 사용이나 상용 환경에서 문제가 되지 않았다. 만약 반례를 찾게 된다면 작성자에게 연락하라. 이 규칙은 주의를 필요로 하며 완전히 안전해야 한다.

##### Example

여기 검사 없이 포인터를 이동하는 방법이 있다.(마치 이동 대입을 구현한 코드라고 상상해보라.):

```c++
    // move from other.ptr to this->ptr
    T* temp = other.ptr;
    other.ptr = nullptr;
    delete ptr;
    ptr = temp;
```

##### Enforcement

* (중간) 이러한 자기 대입의 경우, 이동 대입 연산자는 대입 받는 객체의 포인터 멤버를 `delete`된 상태 또는 `nullptr`로 남겨놓아서는 안된다.
* (자유선택) 표준 라이브러리 컨테이너들의 사용법을 보라(`string`을 포함한다). 그리고 일반적인(객체 수명에 민감하지 않은) 사용에 그 컨테이너들이 안전하다고 생각하라.

### <a name="Rc-move-noexcept"></a>C.66: 이동 연산은 `noexcept`로 만들어라

##### Reason

예외를 던지는 이동 연산은 대다수의 사람들의 타당한 가정을 무너뜨린다.
예외를 던지지 않는 이동은 표준 라이브러리와 언어 특징들에 의해 더 효율적으로 사용될 수 있다.

##### Example

```c++
    template<typename T>
    class Vector {
        // ...
        Vector(Vector&& a) noexcept :elem{a.elem}, sz{a.sz} { a.sz = 0; a.elem = nullptr; }
        Vector& operator=(Vector&& a) noexcept { elem = a.elem; sz = a.sz; a.sz = 0; a.elem = nullptr; }
        // ...
    public:
        T* elem;
        int sz;
    };
```

이 복사 연산들은 예외를 던지지 않는다.

##### Example, bad

```c++
    template<typename T>
    class Vector2 {
        // ...
        Vector2(Vector2&& a) { *this = a; }             // just use the copy
        Vector2& operator=(Vector2&& a) { *this = a; }  // just use the copy
        // ...
    public:
        T* elem;
        int sz;
    };
```

이 `Vector2`는 비 효율적일 뿐만 아니라, 벡터가 메모리 할당을 요구하기 때문에 예외를 던질 수 있다.

##### Enforcement

(쉬움) 이동연산은 `noexcept`로 표시되어야 한다.

### <a name="Rc-copy-virtual"></a>C.67: A polymorphic class should suppress copying

##### Reason

A *polymorphic class* is a class that defines or inherits at least one virtual function. It is likely that it will be used as a base class for other derived classes with polymorphic behavior. If it is accidentally passed by value, with the implicitly generated copy constructor and assignment, we risk slicing: only the base portion of a derived object will be copied, and the polymorphic behavior will be corrupted.

##### Example, bad

```c++
    class B { // BAD: polymorphic base class doesn't suppress copying
    public:
        virtual char m() { return 'B'; }
        // ... nothing about copy operations, so uses default ...
    };

    class D : public B {
    public:
        char m() override { return 'D'; }
        // ...
    };

    void f(B& b) {
        auto b2 = b; // oops, slices the object; b2.m() will return 'B'
    }

    D d;
    f(d);
```

##### Example

```c++
    class B { // GOOD: polymorphic class suppresses copying
    public:
        B(const B&) = delete;
        B& operator=(const B&) = delete;
        virtual char m() { return 'B'; }
        // ...
    };

    class D : public B {
    public:
        char m() override { return 'D'; }
        // ...
    };

    void f(B& b) {
        auto b2 = b; // ok, compiler will detect inadvertent copying, and protest
    }

    D d;
    f(d);
```

##### Note

If you need to create deep copies of polymorphic objects, use `clone()` functions: see [C.130](#Rh-copy).

##### Exception

Classes that represent exception objects need both to be polymorphic and copy-constructible.

##### Enforcement

* Flag a polymorphic class with a non-deleted copy operation.
* Flag an assignment of polymorphic class objects.

## C.other: 다른 기본 연산 규칙들

언어가 제공하는 기본 연산들의 구현 이외에도, 비교, `swap`, 그리고 `hash`처럼 별도의 정의가 필요할 정도로 기초적인 몇몇 연산들이 있다:

### <a name="Rc-eqdefault"></a>C.80: 기본 의미구조(semantic)를 명시적으로 사용하려면 `=default`를 사용하라

##### Reason

컴파일러가 더 정확한 기본 의미구조를 알고 있으며, 이보다 나은 코드를 작성할 수 없다.

##### Example

```c++
    class Tracer {
        string message;
    public:
        Tracer(const string& m) : message{m} { cerr << "entering " << message << '\n'; }
        ~Tracer() { cerr << "exiting " << message << '\n'; }

        Tracer(const Tracer&) = default;
        Tracer& operator=(const Tracer&) = default;
        Tracer(Tracer&&) = default;
        Tracer& operator=(Tracer&&) = default;
    };
```

소멸자를 정의했기 때문에, 우리는 복사, 이동 연산들을 정의해야만 한다. 이를 위해선 `=default`가 가장 최선이고, 간단한 방법이다.

##### Example, bad

```c++
    class Tracer2 {
        string message;
    public:
        Tracer2(const string& m) : message{m} { cerr << "entering " << message << '\n'; }
        ~Tracer2() { cerr << "exiting " << message << '\n'; }

        Tracer2(const Tracer2& a) : message{a.message} {}
        Tracer2& operator=(const Tracer2& a) { message = a.message; return *this; }
        Tracer2(Tracer2&& a) :message{a.message} {}
        Tracer2& operator=(Tracer2&& a) { message = a.message; return *this; }
    };
```

복사와 이동 연산들의 함수들을 일일이 작성하는 것은 번거롭고, 지루하며, 에러에 취약하다. 컴파일러가 이 작업을 더 잘 할수있다.

##### Enforcement

* (중간) 특별한 연산들은 중복을 피하기 위해 컴파일러가 만든 함수들과 같은 접근성, 의미구조를 가져서는 안된다

### <a name="Rc-delete"></a>C.81: 기본 동작을 (대안을 원하지 않고) 금지하고 싶다면 `=delete`를 사용하라

##### Reason

드물게 기본 연산들이 바람직하지 않은 경우도 있다.

##### Example

```c++
    class Immortal {
    public:
        ~Immortal() = delete;   // 소멸이 금지되었다
        // ...
    };

    void use()
    {
        Immortal ugh;  // error: ugh은 소멸될 수 없다
        Immortal* p = new Immortal{};
        delete p;       // error: *p를 소멸시킬 수 없다
    }
```

##### Example

`unique_ptr`는 이동 가능하지만, 복사는 불가능하다. 이 클래스의 복사를 막기 위해, 복사 연산들은 삭제되었다. l-value로부터 복사 연산을 막기 위해서는 `=delete`가 필요하다:

```c++
    template <class T, class D = default_delete<T>> class unique_ptr {
    public:
        // ...
        constexpr unique_ptr() noexcept;
        explicit unique_ptr(pointer p) noexcept;
        // ...
        unique_ptr(unique_ptr&& u) noexcept;   // move constructor
        // ...
        unique_ptr(const unique_ptr&) = delete; // disable copy from lvalue
        // ...
    };

    unique_ptr<int> make();   // "무언가" 만든 뒤에 이동으로 반환한다

    void f()
    {
        unique_ptr<int> pi {};
        auto pi2 {pi};      // error: l-value로부터 생성할 수 없다.
        auto pi3 {make()};  // OK, 이동 생성: make()의 결과는 r-value이다
    }
```

Note that deleted functions should be public.

##### Enforcement

기본 연산을 제거하는 것은 해당 클래스에 부합하는 근거가 있어야 한다.
정말 이유가 있는지 의심하라.
하지만 사람이 보기에 문맥적으로 타당하다고 단언(assert)할 수 있도록 하라.

### <a name="Rc-ctor-virtual"></a>C.82: 생성자 또는 소멸자에서 가상 함수를 호출하지 말아라

##### Reason

호출된 함수는 파생 클래스에서 오버라이드 하는 함수가 아니라, 생성된 객체의 함수이다.
이러한 동작은 혼란을 일으킬 수 있다.
나쁘게는, 생성자와 소멸자 내부에서 발생하는 구현되지 않은 순수 가상 함수에 대한 직접 또는 간접호출이 비정의된 동작을 일으킨다.

##### Example, bad

```c++
    class Base {
    public:
        virtual void f() = 0;   // 구현되지 않았다
        virtual void g();       // 기본 버전을 구현하였다
        virtual void h();       // 기본 버전을 구현하였다
    };

    class Derived : public Base {
    public:
        void g() override;   // 파생 구현을 제공한다
        void h() final;      // 파생 구현을 제공한다

        Derived()
        {
            // BAD: 구현되지 않은 가상 함수를 호출한다
            f();

            // BAD: will call Derived::g, not dispatch further virtually
            g();

            // GOOD: 접근 가능한(visible) 함수를 명시적으로 호출한다
            Derived::g();

            // ok, 문제 없다. h함수는 final 구현체를 의미한다
            h();
        }
    };
```

특정하게 명시적으로 한정된 함수는 `virtual`로 선언되었다고 하더라도 가상호출이 발생하지 않음을 기억하라.

**See also** 정의되지 않은 동작의 위험이 없이 파생 클래스의 함수를 호출하는 효과를 얻기 위해서는 [팩토리 함수](#Rc-factory) 항목을 참고하라.

##### Note

There is nothing inherently wrong with calling virtual functions from constructors and destructors.
The semantics of such calls is type safe.
However, experience shows that such calls are rarely needed, easily confuse maintainers, and become a source of errors when used by novices.

##### Enforcement

* 생성자와 소멸자에서의 가상 함수 호출에는 표시를 남겨라.

### <a name="Rc-swap"></a>C.83: 값 타입들에는, `noexcept` swap함수를 제공하는 것을 고려하라

##### Reason

`swap`함수는 객체 대입을 구현할 때 원활하게 객체를 이동하는 것에서, 에러가 발생하지 않는 것을 보장하는 함수를 제공하는 것까지 몇몇 함수들(idioms)을 구현하는데 유용하다.
swap함수을 이용해서 복사 대입을 구현하는 것을 고려하라. [소멸자, 자원 해제, 그리고 swap은 실패해선 안된다]("#Re-never-fail).

##### Example, good

```c++
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

호출자들의 편의를 위해서 같은 네임스페이스에 비-멤버 `swap`함수를 제공하라.

```c++
    void swap(Foo& a, Foo& b)
    {
        a.swap(b);
    }
```

##### Enforcement

* (쉬움) 가상 함수들이 없는 클래스는 `swap`멤버 함수 선언이 있어야 한다.
* (쉬움) 클래스가 `swap` 멤버함수를 가지고 있다면, 그 함수는 `noexcept`로 선언되어야 한다.

### <a name="Rc-swap-fail"></a>C.84: `swap` 연산은 실패해선 안된다

##### Reason

`swap`연산은 많은 경우 실패하지 않을 것으로 전제하고 사용된다. 또한 실패 가능성이 있는 `swap`연산으로는 정확하게 동작하도록 프로그램이 작성되기 어렵다. 
표준 라이브러리의 컨테이너들과 알고리즘들은 swap연산의 타입이 실패하면 정확하게 동작하지 않을 것이다.

##### Example, bad

```c++
    void swap(My_vector& x, My_vector& y)
    {
        auto tmp = x;   // copy elements
        x = y;
        y = tmp;
    }
```

이 경우는 느릴 뿐만 아니라, `tmp`내의 원소들에 메모리 할당이 발생하면, 이 `swap` 연산은 예외를 던지고 이를 사용하는 STL 알고리즘들이 실패할 수 있다.

##### Enforcement

* (쉬움) 클래스에 `swap` 멤버 함수가 있으면, `noexcept`로 선언되어야 한다.

### <a name="Rc-swap-noexcept"></a>C.85: `swap` 연산은 `noexcept`로 작성하라

##### Reason

[`swap`연산은 실패하지 않도록 작성하라](#Rc-swap-fail).
`swap`연산이 예외를 던지면서 종료하려 한다면, 그것은 잘못된 설계 오류이며 프로그램을 종료하는게 낫다.

##### Enforcement

* (쉬움) 클래스에 `swap` 멤버 함수가 있으면, `noexcept`로 선언되어야 한다.

### <a name="Rc-eq"></a>C.86: `==`연산자는 피연산자 타입들에 대칭적이고, `noexcept`로 만들어라

##### Reason

피연산자들에 비대칭적인 처리는 기대에 부합하지 않고, 형변환이 가능한 경우 에러를 유발할 수 있다.
`==`는 기본적인 연산이며 프로그래머들이 이 연산을 사용할 때 연산 실패에 대한 고민이 없어야 한다.

##### Example

```c++
    struct X {
        string name;
        int number;
    };

    bool operator==(const X& a, const X& b) noexcept {
        return a.name == b.name && a.number == b.number;
    }
```

##### Example, bad

```c++
    class B {
        string name;
        int number;
        bool operator==(const B& a) const {
            return name == a.name && number == a.number;
        }
        // ...
    };
```

`B`의 비교 연산은 두번째 피연산자에 대해 형변환을 용인하지만, 첫번째 피연산자에 대해서는 그렇지 않다.

##### Note

만약 클래스가 `double`타입의 `NaN`처럼 실패 상태를 가진다면, 실패 상태와의 비교에서 예외를 던지도록 하는 것이 적합할 수도 있다.
다른 방법으로는 실패 상태끼리의 비교는 동등하게 보고, 적합한 상태와 실패 상태의 비교에서는 거짓으로 판정할 수 있다.

#### Note

이 규칙은 모든 일반 비교 연산자들에도 적용된다 : `!=`, `<`, `<=`, `>`, `>=`.

##### Enforcement

* 인자의 타입이 다른 `operator==()`를 지적하라. 다른 비교 연산자들도 마찬가지다 : `!=`, `<`, `<=`, `>`, `>=`.
* 멤버인  `operator==()` 함수들을 지적하라. 다른 비교 연산자들도 마찬가지다 : `!=`, `<`, `<=`, `>`, `>=`.

### <a name="Rc-eq-base"></a>C.87: 기본 클래스에 있는 `==`에 주의하라

##### Reason

계층 구조에서 잘못 사용하기 어렵고 유용한 `==`를 작성하는 것은 어려운 일이다.

##### Example, bad

```c++
    class B {
        string name;
        int number;
        virtual bool operator==(const B& a) const
        {
             return name == a.name && number == a.number;
        }
        // ...
    };
```

`B`의 비교 연산은 두번째 피연산자에 대해서 타입 변환을 허용하지만, 첫번째 피연산자에 대해서는 허용하지 않는다.

```c++
    class D :B {
        char character;
        virtual bool operator==(const D& a) const
        {
            return name == a.name && number == a.number && character == a.character;
        }
        // ...
    };

    B b = ...
    D d = ...
    b == d;    // compares name and number, ignores d's character
    d == b;    // error: no == defined
    D d2;
    d == d2;   // compares name, number, and character
    B& b2 = d2;
    b2 == d;   // compares name and number, ignores d2's and d's character
```

물론 계층 구조 안에서 `==`가 동작하도록 하는 방법들이 있지만, 단순한(naive) 방법들은 고려하지 말아라.

#### Note

이 규칙은 모든 일반 비교연산자에 대해서도 동일하다 : `!=`, `<`, `<=`, `>`, `>=`

##### Enforcement

* 가상 함수인 `operator==()`를 지적하라. 다른 비교 연산자들도 동일하다: `!=`, `<`, `<=`, `>`, `>=`.

### <a name="Rc-hash"></a>C.89: `hash`는 `noexcept`로 작성하라

##### Reason

해시 컴테이너들의 사용자들은 hash를 간접적으로 사용하며, 해시값을 위한 단순한 접근이 throw하지 않을 것으로 기대한다.  
이는 표준 라이브러리의 요구사항이다.  

##### Example, bad
```c++
    template<>
    struct hash<My_type> {  // 정말정말 안좋은 해시 특수화
        using result_type = size_t;
        using argument_type = My_type;

        size_t operator() (const My_type & x) const
        {
            size_t xs = x.s.size();
            if (xs < 4) throw Bad_My_type{};    // "이런 이단자 같으니!"
            return hash<size_t>()(x.s.size()) ^ trim(x.s);
        }
    };

    int main()
    {
        unordered_map<My_type, int> m;
        My_type mt{ "asdfg" };
        m[mt] = 7;
        cout << m[My_type{ "asdfg" }] << '\n';
    }
```
`hash` 특수화를 정의할 때는, 간단하게 `^` (xor)와 함께 표준 라이브러리의 `hash` 특수화와 통합되도록 하라.  
비 전문가들을 위해선 이 방법이 더 적합하다.

##### Enforcement

* 예외를 던지는 `hash`들을 지적하라.

## <a name="SS-containers"></a>C.con: 컨테이너와 리소스 핸들

컨테이너는 임의 타입의 연속된 개체들을 가진 개체를 의미한다;  `std::vector`는 대표적인 컨테이너 타입이다.
리소스 핸들은 자원을 소유하는 클래스를 의미한다; `std::vector`는 보통 리소스 핸들에 해당한다; 이 경우 자원은 연속된 원소들이다.

컨테이너 규칙 요약:

* [C.100: 컨테이너를 정의할때는 STL을 따르라](#Rcon-stl)
* [C.101: 값 문맥을 적용하라](#Rcon-val)
* [C.102: move 연산을 제공하라](#Rcon-move)
* [C.103: 초기화 리스트 생성자를 지원하라](#Rcon-init)
* [C.104: 공백 값으로 설정하는 기본 생성자를 지원하라](#Rcon-empty)
* [C.105: 생성자와 '확장' 생성자를 지원하라](#Rcon-val)
* ???
* [C.109: 리소스 핸들이 포인터 문맥을 따를 경우에는, `*` 과 `->` 연산자를 제공하라](#rcon-ptr)

**See also**: [Resources](#S-resource)

## <a name="SS-lambdas"></a>C.lambdas: 함수 개체와 람다 표현식(Function objects and lambdas)

함수 개체는 `()`를 오버로드해 호출을 지원하는 개체를 의미한다.
람다 표현식(줄여서 "람다"라고도 한다)은 함수 개체를 생성하도록 하는 표기를 의미한다.
함수 개체는 가능한 복사 비용을 발생시키지 않아야 한다 (또 그렇기에 [값에 의한 전달](#Rf-in)이 사용된다).

Summary:

* [F.50: Use a lambda when a function won't do (to capture local variables, or to write a local function)](#Rf-capture-vs-overload)
* [F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms](#Rf-reference-capture)
* [F.53: Avoid capturing by reference in lambdas that will be used nonlocally, including returned, stored on the heap, or passed to another thread](#Rf-value-capture)
* [ES.28: Use lambdas for complex initialization, especially of `const` variables](#Res-lambda-init)

## <a name="SS-hier"></a>C.hier: 클래스 계층 구조 (OOP)

A class hierarchy is constructed to represent a set of hierarchically organized concepts (only).
Typically base classes act as interfaces.
There are two major uses for hierarchies, often named implementation inheritance and interface inheritance.

Class hierarchy rule summary:

* [C.120: Use class hierarchies to represent concepts with inherent hierarchical structure (only)](#Rh-domain)
* [C.121: If a base class is used as an interface, make it a pure abstract class](#Rh-abstract)
* [C.122: Use abstract classes as interfaces when complete separation of interface and implementation is needed](#Rh-separation)

Designing rules for classes in a hierarchy summary:

* [C.126: An abstract class typically doesn't need a constructor](#Rh-abstract-ctor)
* [C.127: A class with a virtual function should have a virtual or protected destructor](#Rh-dtor)
* [C.128: Virtual functions should specify exactly one of `virtual`, `override`, or `final`](#Rh-override)
* [C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance](#Rh-kind)
* [C.130: For making deep copies of polymorphic classes prefer a virtual `clone` function instead of copy construction/assignment](#Rh-copy)
* [C.131: Avoid trivial getters and setters](#Rh-get)
* [C.132: Don't make a function `virtual` without reason](#Rh-virtual)
* [C.133: Avoid `protected` data](#Rh-protected)
* [C.134: Ensure all non-`const` data members have the same access level](#Rh-public)
* [C.135: Use multiple inheritance to represent multiple distinct interfaces](#Rh-mi-interface)
* [C.136: Use multiple inheritance to represent the union of implementation attributes](#Rh-mi-implementation)
* [C.137: Use `virtual` bases to avoid overly general base classes](#Rh-vbase)
* [C.138: Create an overload set for a derived class and its bases with `using`](#Rh-using)
* [C.139: Use `final` sparingly](#Rh-final)
* [C.140: Do not provide different default arguments for a virtual function and an overrider](#Rh-virtual-default-arg)

Accessing objects in a hierarchy rule summary:

* [C.145: Access polymorphic objects through pointers and references](#Rh-poly)
* [C.146: Use `dynamic_cast` where class hierarchy navigation is unavoidable](#Rh-dynamic_cast)
* [C.147: Use `dynamic_cast` to a reference type when failure to find the required class is considered an error](#Rh-ref-cast)
* [C.148: Use `dynamic_cast` to a pointer type when failure to find the required class is considered a valid alternative](#Rh-ptr-cast)
* [C.149: Use `unique_ptr` or `shared_ptr` to avoid forgetting to `delete` objects created using `new`](#Rh-smart)
* [C.150: Use `make_unique()` to construct objects owned by `unique_ptr`s](#Rh-make_unique)
* [C.151: Use `make_shared()` to construct objects owned by `shared_ptr`s](#Rh-make_shared)
* [C.152: Never assign a pointer to an array of derived class objects to a pointer to its base](#Rh-array)
* [C.153: Prefer virtual function to casting](#Rh-use-virtual)

### <a name="Rh-domain"></a>C.120: Use class hierarchies to represent concepts with inherent hierarchical structure (only)

##### Reason

Direct representation of ideas in code eases comprehension and maintenance. Make sure the idea represented in the base class exactly matches all derived types and there is not a better way to express it than using the tight coupling of inheritance.

Do *not* use inheritance when simply having a data member will do. Usually this means that the derived type needs to override a base virtual function or needs access to a protected member.

##### Example

```c++
    class DrawableUIElement {
    public:
        virtual void render() const = 0;
        // ...
    };

    class AbstractButton : public DrawableUIElement {
    public:
        virtual void onClick() = 0;
        // ...
    };

    class PushButton : public AbstractButton {
        virtual void render() const override;
        virtual void onClick() override;
        // ...
    };

    class Checkbox : public AbstractButton {
    // ...
    };
```

##### Example, bad

Do *not* represent non-hierarchical domain concepts as class hierarchies.

```c++
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

Here most overriding classes cannot implement most of the functions required in the interface well.
Thus the base class becomes an implementation burden.
Furthermore, the user of `Container` cannot rely on the member functions actually performing a meaningful operations reasonably efficiently;
it may throw an exception instead.
Thus users have to resort to run-time checking and/or
not using this (over)general interface in favor of a particular interface found by a run-time type inquiry (e.g., a `dynamic_cast`).

##### Enforcement

* Look for classes with lots of members that do nothing but throw.
* Flag every use of a nonpublic base class `B` where the derived class `D` does not override a virtual function or access a protected member in `B`, and `B` is not one of the following: empty, a template parameter or parameter pack of `D`, a class template specialized with `D`.

### <a name="Rh-abstract"></a>C.121: If a base class is used as an interface, make it a pure abstract class

##### Reason

A class is more stable (less brittle) if it does not contain data.
Interfaces should normally be composed entirely of public pure virtual functions and a default/empty virtual destructor.

##### Example

```c++
    class My_interface {
    public:
        // ...only pure virtual functions here ...
        virtual ~My_interface() {}   // or =default
    };
```

##### Example, bad

```c++
    class Goof {
    public:
        // ...only pure virtual functions here ...
        // no virtual destructor
    };

    class Derived : public Goof {
        string s;
        // ...
    };

    void use()
    {
        unique_ptr<Goof> p {new Derived{"here we go"}};
        f(p.get()); // use Derived through the Goof interface
        g(p.get()); // use Derived through the Goof interface
    } // leak
```

The `Derived` is `delete`d through its `Goof` interface, so its `string` is leaked.
Give `Goof` a virtual destructor and all is well.

##### Enforcement

* Warn on any class that contains data members and also has an overridable (non-`final`) virtual function.

### <a name="Rh-separation"></a>C.122: Use abstract classes as interfaces when complete separation of interface and implementation is needed

##### Reason

Such as on an ABI (link) boundary.

##### Example

```c++
    struct Device {
        virtual ~Device() = default;
        virtual void write(span<const char> outbuf) = 0;
        virtual void read(span<char> inbuf) = 0;
    };

    class D1 : public Device {
        // ... data ...

        void write(span<const char> outbuf) override;
        void read(span<char> inbuf) override;
    };

    class D2 : public Device {
        // ... different data ...

        void write(span<const char> outbuf) override;
        void read(span<char> inbuf) override;
    };
```

A user can now use `D1`s and `D2`s interchangeably through the interface provided by `Device`.
Furthermore, we can update `D1` and `D2` in a ways that are not binary compatible with older versions as long as all access goes through `Device`.

##### Enforcement

    ???

## C.hierclass: Designing classes in a hierarchy:

### <a name="Rh-abstract-ctor"></a>C.126: An abstract class typically doesn't need a constructor

##### Reason

An abstract class typically does not have any data for a constructor to initialize.

##### Example

    ???

##### Exception

* A base class constructor that does work, such as registering an object somewhere, may need a constructor.
* In extremely rare cases, you might find it reasonable for an abstract class to have a bit of data shared by all derived classes
  (e.g., use statistics data, debug information, etc.); such classes tend to have constructors. But be warned: Such classes also tend to be prone to requiring virtual inheritance.

##### Enforcement

Flag abstract classes with constructors.

### <a name="Rh-dtor"></a>C.127: A class with a virtual function should have a virtual or protected destructor

##### Reason

A class with a virtual function is usually (and in general) used via a pointer to base. Usually, the last user has to call delete on a pointer to base, often via a smart pointer to base, so the destructor should be public and virtual. Less commonly, if deletion through a pointer to base is not intended to be supported, the destructor should be protected and nonvirtual; see [C.35](#Rc-dtor-virtual).

##### Example, bad

```c++
    struct B {
        virtual int f() = 0;
        // ... no user-written destructor, defaults to public nonvirtual ...
    };

    // bad: derived from a class without a virtual destructor
    struct D : B {
        string s {"default"};
    };

    void use()
    {
        unique_ptr<B> p = make_unique<D>();
        // ...
    } // undefined behavior. May call B::~B only and leak the string
```

##### Note

There are people who don't follow this rule because they plan to use a class only through a `shared_ptr`: `std::shared_ptr<B> p = std::make_shared<D>(args);` Here, the shared pointer will take care of deletion, so no leak will occur from an inappropriate `delete` of the base. People who do this consistently can get a false positive, but the rule is important -- what if one was allocated using `make_unique`? It's not safe unless the author of `B` ensures that it can never be misused, such as by making all constructors private and providing a factory function to enforce the allocation with `make_shared`.

##### Enforcement

* A class with any virtual functions should have a destructor that is either public and virtual or else protected and nonvirtual.
* Flag `delete` of a class with a virtual function but no virtual destructor.

### <a name="Rh-override"></a>C.128: Virtual functions should specify exactly one of `virtual`, `override`, or `final`

##### Reason

Readability.
Detection of mistakes.
Writing explicit `virtual`, `override`, or `final` is self-documenting and enables the compiler to catch mismatch of types and/or names between base and derived classes. However, writing more than one of these three is both redundant and a potential source of errors.

It's simple and clear:

* `virtual` means exactly and only "this is a new virtual function."
* `override` means exactly and only "this is a non-final overrider."
* `final` means exactly and only "this is a final overrider."

If a base class destructor is declared `virtual`, one should avoid declaring derived class destructors  `virtual` or `override`. Some code base and tools might insist on `override` for destructors, but that is not the recommendation of these guidelines.

##### Example, bad

```c++
    struct B {
        void f1(int);
        virtual void f2(int) const;
        virtual void f3(int);
        // ...
    };

    struct D : B {
        void f1(int);        // bad (hope for a warning): D::f1() hides B::f1()
        void f2(int) const;  // bad (but conventional and valid): no explicit override
        void f3(double);     // bad (hope for a warning): D::f3() hides B::f3()
        // ...
    };
```

##### Example, good

```c++
    struct Better : B {
        void f1(int) override;        // error (caught): D::f1() hides B::f1()
        void f2(int) const override;
        void f3(double) override;     // error (caught): D::f3() hides B::f3()
        // ...
    };
```

#### Discussion

We want to eliminate two particular classes of errors:

* **implicit virtual**: the programmer intended the function to be implicitly virtual and it is (but readers of the code can't tell); or the programmer intended the function to be implicitly virtual but it isn't (e.g., because of a subtle parameter list mismatch); or the programmer did not intend the function to be virtual but it is (because it happens to have the same signature as a virtual in the base class)
* **implicit override**: the programmer intended the function to be implicitly an overrider and it is (but readers of the code can't tell); or the programmer intended the function to be implicitly an overrider but it isn't (e.g., because of a subtle parameter list mismatch); or the programmer did not intend the function to be an overrider but it is (because it happens to have the same signature as a virtual in the base class -- note this problem arises whether or not the function is explicitly declared virtual, because the programmer may have intended to create either a new virtual function or a new nonvirtual function)

##### Enforcement

* Compare names in base and derived classes and flag uses of the same name that does not override.
* Flag overrides with neither `override` nor `final`.
* Flag function declarations that use more than one of `virtual`, `override`, and `final`.

### <a name="Rh-kind"></a>C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance

##### Reason

Implementation details in an interface makes the interface brittle;
that is, makes its users vulnerable to having to recompile after changes in the implementation.
Data in a base class increases the complexity of implementing the base and can lead to replication of code.

##### Note

Definition:

* interface inheritance is the use of inheritance to separate users from implementations,
in particular to allow derived classes to be added and changed without affecting the users of base classes.
* implementation inheritance is the use of inheritance to simplify implementation of new facilities
by making useful operations available for implementers of related new operations (sometimes called "programming by difference").

A pure interface class is simply a set of pure virtual functions; see [I.25](#Ri-abstract).

In early OOP (e.g., in the 1980s and 1990s), implementation inheritance and interface inheritance were often mixed
and bad habits die hard.
Even now, mixtures are not uncommon in old code bases and in old-style teaching material.

The importance of keeping the two kinds of inheritance increases

* with the size of a hierarchy (e.g., dozens of derived classes),
* with the length of time the hierarchy is used (e.g., decades), and
* with the number of distinct organizations in which a hierarchy is used
(e.g., it can be difficult to distribute an update to a base class)

##### Example, bad

```c++
    class Shape {   // BAD, mixed interface and implementation
    public:
        Shape();
        Shape(Point ce = {0, 0}, Color co = none): cent{ce}, col {co} { /* ... */}

        Point center() const { return cent; }
        Color color() const { return col; }

        virtual void rotate(int) = 0;
        virtual void move(Point p) { cent = p; redraw(); }

        virtual void redraw();

        // ...
    private:
        Point cent;
        Color col;
    };

    class Circle : public Shape {
    public:
        Circle(Point c, int r) :Shape{c}, rad{r} { /* ... */ }

        // ...
    private:
        int rad;
    };

    class Triangle : public Shape {
    public:
        Triangle(Point p1, Point p2, Point p3); // calculate center
        // ...
    };
```

Problems:

* As the hierarchy grows and more data is added to `Shape`, the constructors gets harder to write and maintain.
* Why calculate the center for the `Triangle`? we may never us it.
* Add a data member to `Shape` (e.g., drawing style or canvas)
and all derived classes and all users needs to be reviewed, possibly changes, and probably recompiled.

The implementation of `Shape::move()` is an example of implementation inheritance:
we have defined `move()` once and for all for all derived classes.
The more code there is in such base class member function implementations and the more data is shared by placing it in the base,
the more benefits we gain - and the less stable the hierarchy is.

##### Example

This Shape hierarchy can be rewritten using interface inheritance:

```c++
    class Shape {  // pure interface
    public:
        virtual Point center() const = 0;
        virtual Color color() const = 0;

        virtual void rotate(int) = 0;
        virtual void move(Point p) = 0;

        virtual void redraw() = 0;

        // ...
    };
```

Note that a pure interface rarely have constructors: there is nothing to construct.

```c++
    class Circle : public Shape {
    public:
        Circle(Point c, int r, Color c) :cent{c}, rad{r}, col{c} { /* ... */ }

        Point center() const override { return cent; }
        Color color() const override { return col; }

        // ...
    private:
        Point cent;
        int rad;
        Color col;
    };
```

The interface is now less brittle, but there is more work in implementing the member functions.
For example, `center` has to be implemented by every class derived from `Shape`.

##### Example, dual hierarchy

How can we gain the benefit of the stable hierarchies from implementation hierarchies and the benefit of implementation reuse from implementation inheritance.
One popular technique is dual hierarchies.
There are many ways of implementing the idea of dual hierarchies; here, we use a multiple-inheritance variant.

First we devise a hierarchy of interface classes:

```c++
    class Shape {   // pure interface
    public:
        virtual Point center() const = 0;
        virtual Color color() const = 0;

        virtual void rotate(int) = 0;
        virtual void move(Point p) = 0;

        virtual void redraw() = 0;

        // ...
    };

    class Circle : public Shape {   // pure interface
    public:
        virtual int radius() = 0;
        // ...
    };
```

To make this interface useful, we must provide its implementation classes (here, named equivalently, but in the `Impl` namespace):

```c++
    class Impl::Shape : public Shape { // implementation
    public:
        // constructors, destructor
        // ...
        Point center() const override { /* ... */ }
        Color color() const override { /* ... */ }

        void rotate(int) override { /* ... */ }
        void move(Point p) override { /* ... */ }

        void redraw() override { /* ... */ }

        // ...
    };
```

Now `Shape` is a poor example of a class with an implementation,
but bear with us because this is just a simple example of a technique aimed at more complex hierarchies.

```c++
    class Impl::Circle : public Circle, public Impl::Shape {   // implementation
    public:
        // constructors, destructor

        int radius() override { /* ... */ }
        // ...
    };
```

And we could extend the hierarchies by adding a Smiley class (:-)):

```c++
    class Smiley : public Circle { // pure interface
    public:
        // ...
    };

    class Impl::Smiley : public Smiley, public Impl::Circle {   // implementation
    public:
        // constructors, destructor
        // ...
    }
```

There are now two hierarchies:

* interface: Smiley -> Circle -> Shape
* implementation: Impl::Smiley -> Impl::Circle -> Impl::Shape

Since each implementation derived from its interface as well as its implementation base class we get a lattice (DAG):

```
    Smiley     ->         Circle     ->  Shape
      ^                     ^               ^
      |                     |               |
    Impl::Smiley -> Impl::Circle -> Impl::Shape
```

As mentioned, this is just one way to construct a dual hierarchy.

The implementation hierarchy can be used directly, rather than through the abstract interface.

```c++
    void work_with_shape(Shape&);

    int user()
    {
        Impl::Smiley my_smiley{ /* args */ };   // create concrete shape
        // ...
        my_smiley.some_member();        // use implementation class directly
        // ...
        work_with_shape(my_smiley);     // use implementation through abstract interface
        // ...
    }
```

This can be useful when the implementation class has members that are not offered in the abstract interface
or if direct use of a member offers optimization opportunities (e.g., if an implementation member function is `final`)

##### Note

Another (related) technique for separating interface and implementation is [Pimpl](#Ri-pimpl).

##### Note

There is often a choice between offering common functionality as (implemented) base class functions and free-standing functions
(in an implementation namespace).
Base classes gives a shorter notation and easier access to shared data (in the base)
at the cost of the functionality being available only to users of the hierarchy.

##### Enforcement

* Flag a derived to base conversion to a base with both data and virtual functions
(except for calls from a derived class member to a base class member)
* ???

### <a name="Rh-copy"></a>C.130: For making deep copies of polymorphic classes prefer a virtual `clone` function instead of copy construction/assignment

##### Reason

Copying a polymorphic class is discouraged due to the slicing problem, see [C.67](#Rc-copy-virtual). If you really need copy semantics, copy deeply: Provide a virtual `clone` function that will copy the actual most-derived type and return an owning pointer to the new object, and then in derived classes return the derived type (use a covariant return type).

##### Example

```c++
    class B {
    public:
        virtual owner<B*> clone() = 0;
        virtual ~B() = 0;

        B(const B&) = delete;
        B& operator=(const B&) = delete;
    };

    class D : public B {
    public:
        owner<D*> clone() override;
        virtual ~D() override;
    };
```

Generally, it is recommended to use smart pointers to represent ownership (see [R.20](#Rr-owner)). However, because of language rules, the covariant return type cannot be a smart pointer: `D::clone` can't return a `unique_ptr<D>` while `B::clone` returns `unique_ptr<B>`. Therefore, you either need to consistently return `unique_ptr<B>` in all overrides, or use `owner<>` utility from the [Guidelines Support Library](#SS-views).


### <a name="Rh-get"></a>C.131: Avoid trivial getters and setters

##### Reason

A trivial getter or setter adds no semantic value; the data item could just as well be `public`.

##### Example

```c++
    class Point {   // Bad: verbose
        int x;
        int y;
    public:
        Point(int xx, int yy) : x{xx}, y{yy} { }
        int get_x() const { return x; }
        void set_x(int xx) { x = xx; }
        int get_y() const { return y; }
        void set_y(int yy) { y = yy; }
        // no behavioral member functions
    };
```

Consider making such a class a `struct` -- that is, a behaviorless bunch of variables, all public data and no member functions.

```c++
    struct Point {
        int x {0};
        int y {0};
    };
```

Note that we can put default initializers on member variables: [C.49: Prefer initialization to assignment in constructors](#Rc-initialize).

##### Note

The key to this rule is whether the semantics of the getter/setter are trivial. While it is not a complete definition of "trivial", consider whether there would be any difference beyond syntax if the getter/setter was a public data member instead. Examples of non-trivial semantics would be: maintaining a class invariant or converting between an internal type and an interface type.

##### Enforcement

Flag multiple `get` and `set` member functions that simply access a member without additional semantics.

### <a name="Rh-virtual"></a>C.132: Don't make a function `virtual` without reason

##### Reason

중첩된 `virtual`은 실행 시간과 객체의 코드 크기를 증가시킨다.
가상 함수는 오버라이드 될 수 있고, 그렇기 때문에 파생 클래스에서의 실수에 노출되어있다. 
가상 함수는 템플릿 계층구조에서 코드 복제를 야기한다.

##### Example, bad

```c++
    template<class T>
    class Vector {
    public:
        // ...
        virtual int size() const { return sz; }           // bad: 파생 클래스에서 다른 무슨 일을 하겠는가?
    private:
        T* elem;   // the elements
        int sz;    // number of elements
    };
```

이런 "vector"는 기본 클래스로 사용되는 것을 전혀 의도하지 않았다.

##### Enforcement

* 가상 함수를 가지지만 파생 클래스가 없으면 지적하라.
* 모든 멤버 함수가 가상 함수이고 구현을 가지고 있으면 지적하라.

### <a name="Rh-protected"></a>C.133: Avoid `protected` data

##### Reason

`protected` 데이터는 복잡성과 에러의 원인이다.  
`protected` 데이터는 불변조건의 구문을 복잡하게 만든다.  
`protected` data inherently violates the guidance against putting data in base classes, which usually leads to having to deal with virtual inheritance as well.

##### Example, bad

```c++
    class Shape {
    public:
        // ... interface functions ...
    protected:
        // data for use in derived classes:
        Color fill_color;
        Color edge_color;
        Style st;
    };
```

Now it is up to every derived `Shape` to manipulate the protected data correctly.
This has been popular, but also a major source of maintenance problems.
In a large class hierarchy, the consistent use of protected data is hard to maintain because there can be a lot of code,
spread over a lot of classes.

The set of classes that can touch that data is open: anyone can derive a new class and start manipulating the protected data.
Often, it is not possible to examine the complete set of classes, so any change to the representation of the class becomes infeasible.
There is no enforced invariant for the protected data; it is much like a set of global variables.
The protected data has de facto become global to a large body of code.

##### Note

Protected data often looks tempting to enable arbitrary improvements through derivation.
Often, what you get is unprincipled changes and errors.
[Prefer `private` data](#Rc-private) with a well-specified and enforced invariant.
Alternative, and often better, [keep data out of any class used as an interface](#Rh-abstract).

##### Note

Protected member function can be just fine.

##### Enforcement

Flag classes with `protected` data.

### <a name="Rh-public"></a>C.134: Ensure all non-`const` data members have the same access level

##### Reason

Prevention of logical confusion leading to errors.
If the non-`const` data members don't have the same access level, the type is confused about what it's trying to do.
Is it a type that maintains an invariant or simply a collection of values?

##### Discussion

The core question is: What code is responsible for maintaining a meaningful/correct value for that variable?

There are exactly two kinds of data members:

* A: Ones that don't participate in the object's invariant. Any combination of values for these members is valid.
* B: Ones that do participate in the object's invariant. Not every combination of values is meaningful (else there'd be no invariant). Therefore all code that has write access to these variables must know about the invariant, know the semantics, and know (and actively implement and enforce) the rules for keeping the values correct.

Data members in category A should just be `public` (or, more rarely, `protected` if you only want derived classes to see them). They don't need encapsulation. All code in the system might as well see and manipulate them.

Data members in category B should be `private` or `const`. This is because encapsulation is important. To make them non-`private` and non-`const` would mean that the object can't control its own state: An unbounded amount of code beyond the class would need to know about the invariant and participate in maintaining it accurately -- if these data members were `public`, that would be all calling code that uses the object; if they were `protected`, it would be all the code in current and future derived classes. This leads to brittle and tightly coupled code that quickly becomes a nightmare to maintain. Any code that inadvertently sets the data members to an invalid or unexpected combination of values would corrupt the object and all subsequent uses of the object.

Most classes are either all A or all B:

* *All public*: If you're writing an aggregate bundle-of-variables without an invariant across those variables, then all the variables should be `public`.
  [By convention, declare such classes `struct` rather than `class`](#Rc-struct)
* *All private*: If you're writing a type that maintains an invariant, then all the non-`const` variables should be private -- it should be encapsulated.

##### Exception

Occasionally classes will mix A and B, usually for debug reasons. An encapsulated object may contain something like non-`const` debug instrumentation that isn't part of the invariant and so falls into category A -- it isn't really part of the object's value or meaningful observable state either. In that case, the A parts should be treated as A's (made `public`, or in rarer cases `protected` if they should be visible only to derived classes) and the B parts should still be treated like B's (`private` or `const`).

##### Enforcement

Flag any class that has non-`const` data members with different access levels.

### <a name="Rh-mi-interface"></a>C.135: Use multiple inheritance to represent multiple distinct interfaces

##### Reason

모든 클래스들이 모든 인터페이스들을 지원하지는 않을 것이다. 그리고 모든 호출자(caller)들이 모든 연산들을 사용하길 원하지도 않을 것이다. (다중 상속은) 특별히 단일한(monolitic) 인터페이스들을 파생 클래스가 지원하는 동작의 "측면"들로 나눌때 사용하라.

##### Example

```c++
    class iostream : public istream, public ostream {   // 굉장히 단순하다
        // ...
    };
```

`istream` provides the interface to input operations; `ostream` provides the interface to output operations.
`iostream` provides the union of the `istream` and `ostream` interfaces and the synchronization needed to allow both on a single stream.

##### Note

This is a very common use of inheritance because the need for multiple different interfaces to an implementation is common
and such interfaces are often not easily or naturally organized into a single-rooted hierarchy.

##### Note

Such interfaces are typically abstract classes.

##### Enforcement

???

### <a name="Rh-mi-implementation"></a>C.136: Use multiple inheritance to represent the union of implementation attributes

##### Reason

Some forms of mixins have state and often operations on that state.
If the operations are virtual the use of inheritance is necessary, if not using inheritance can avoid boilerplate and forwarding.

##### Example

```c++
    class iostream : public istream, public ostream {   // very simplified
        // ...
    };
```

`istream` provides the interface to input operations (and some data); `ostream` provides the interface to output operations (and some data).
`iostream` provides the union of the `istream` and `ostream` interfaces and the synchronization needed to allow both on a single stream.

##### Note

이것은 상대적으로 드문 경우인데, 구현은 종종 단일루트(single-root) 계층으로 조직화될 수 있기 때문이다.

##### Example

Sometimes, an "implementation attribute" is more like a "mixin" that determine the behavior of an implementation and inject
members to enable the implementation of the policies it requires.
For example, see `std::enable_shared_from_this`
or various bases from boost.intrusive (e.g. `list_base_hook` or `intrusive_ref_counter`).

##### Enforcement

???

### <a name="Rh-vbase"></a>C.137: Use `virtual` bases to avoid overly general base classes

##### Reason

 Allow separation of shared data and interface.
 To avoid all shared data to being put into an ultimate base class.

##### Example

```c++
    struct Interface {
        virtual void f();
        virtual int g();
        // ... no data here ...
    };

    class Utility {  // with data
        void utility1();
        virtual void utility2();    // customization point
    public:
        int x;
        int y;
    };

    class Derive1 : public Interface, virtual protected Utility {
        // override Interface functions
        // Maybe override Utility virtual functions
        // ...
    };

    class Derive2 : public Interface, virtual protected Utility {
        // override Interface functions
        // Maybe override Utility virtual functions
        // ...
    };
```

Factoring out `Utility` makes sense if many derived classes share significant "implementation details."

##### Note

Obviously, the example is too "theoretical", but it is hard to find a *small* realistic example.
`Interface` is the root of an [interface hierarchy](#Rh-abstract)
and `Utility` is the root of an [implementation hierarchy](#Rh-kind).
Here is [a slightly more realistic example](https://www.quora.com/What-are-the-uses-and-advantages-of-virtual-base-class-in-C%2B%2B/answer/Lance-Diduck) with an explanation.

##### Note

Often, linearization of a hierarchy is a better solution.

##### Enforcement

Flag mixed interface and implementation hierarchies.

### <a name="Rh-using"></a>C.138: Create an overload set for a derived class and its bases with `using`

##### Reason

Without a using declaration, member functions in the derived class hide the entire inherited overload sets.

##### Example, bad

```c++
    #include <iostream>
    class B {
    public:
        virtual int f(int i) { std::cout << "f(int): "; return i; }
        virtual double f(double d) { std::cout << "f(double): "; return d; }
    };
    class D: public B {
    public:
        int f(int i) override { std::cout << "f(int): "; return i + 1; }
    };
    int main()
    {
        D d;
        std::cout << d.f(2) << '\n';   // prints "f(int): 3"
        std::cout << d.f(2.3) << '\n'; // prints "f(int): 3"
    }
```

##### Example, good

```c++
    class D: public B {
    public:
        int f(int i) override { std::cout << "f(int): "; return i + 1; }
        using B::f; // exposes f(double)
    };
```

##### Note

This issue affects both virtual and nonvirtual member functions

For variadic bases, C++17 introduced a variadic form of the using-declaration,

```c++
    template <class... Ts>
    struct Overloader : Ts... {
        using Ts::operator()...; // exposes operator() from every base
    };
```

##### Enforcement

Diagnose name hiding

### <a name="Rh-final"></a>C.139: Use `final` sparingly

##### Reason

Capping a hierarchy with `final` is rarely needed for logical reasons and can be damaging to the extensibility of a hierarchy.

##### Example, bad

```c++
    class Widget { /* ... */ };

    // nobody will ever want to improve My_widget (or so you thought)
    class My_widget final : public Widget { /* ... */ };

    class My_improved_widget : public My_widget { /* ... */ };  // error: can't do that
```

##### Note

Not every class is meant to be a base class.
Most standard-library classes are examples of that (e.g., `std::vector` and `std::string` are not designed to be derived from).
This rule is about using `final` on classes with virtual functions meant to be interfaces for a class hierarchy.

##### Note

Capping an individual virtual function with `final` is error-prone as `final` can easily be overlooked when defining/overriding a set of functions.
Fortunately, the compiler catches such mistakes: You cannot re-declare/re-open a `final` member in a derived class.

##### Note

Claims of performance improvements from `final` should be substantiated.
Too often, such claims are based on conjecture or experience with other languages.

There are examples where `final` can be important for both logical and performance reasons.
One example is a performance-critical AST hierarchy in a compiler or language analysis tool.
New derived classes are not added every year and only by library implementers.
However, misuses are (or at least have been) far more common.

##### Enforcement

Flag uses of `final`.

### <a name="Rh-virtual-default-arg"></a>C.140: Do not provide different default arguments for a virtual function and an overrider

##### Reason

That can cause confusion: An overrider does not inherit default arguments.

##### Example, bad

```c++
    class Base {
    public:
        virtual int multiply(int value, int factor = 2) = 0;
    };

    class Derived : public Base {
    public:
        int multiply(int value, int factor = 10) override;
    };

    Derived d;
    Base& b = d;

    b.multiply(10);  // these two calls will call the same function but
    d.multiply(10);  // with different arguments and so different results
```

##### Enforcement

Flag default arguments on virtual functions if they differ between base and derived declarations.

## C.hier-access: 계층 구조에서 개체 접근

### <a name="Rh-poly"></a>C.145: Access polymorphic objects through pointers and references

##### Reason

가상 함수를 가진 클래스가 있다면, 당신은 (일반적으로) 어떤 클래스가 실행될 함수를 제공할지 알 수 없다.

##### Example

```c++
    struct B { int a; virtual int f(); };
    struct D : B { int b; int f() override; };

    void use(B b)
    {
        D d;
        B b2 = d;   // 복사 손실(slice)
        B b3 = b;
    }

    void use2()
    {
        D d;
        use(d);   // 복사 손실(slice)
    }
```

Both `d`s are sliced.

##### Exception

객체가 정의된 범위 안에서는 이름이 있는 다형적 객체에 안전하게 접근할 수 있다. 단지 복사 손실이 생기지 않도록 하라.

```c++
    void use3()
    {
        D d;
        d.f();   // OK
    }
```

##### Enforcement

Flag all slicing.

### <a name="Rh-dynamic_cast"></a>C.146: Use `dynamic_cast` where class hierarchy navigation is unavoidable

##### Reason

`dynamic_cast`는 실행시간에 검사된다.

##### Example

```c++
    struct B {   // 인터페이스
        virtual void f();
        virtual void g();
    };

    struct D : B {   // 확장된 인터페이스
        void f() override;
        virtual void h();
    };

    void user(B* pb)
    {
        if (D* pd = dynamic_cast<D*>(pb)) {
            // ... D의 인터페이스를 사용한다 ...
        }
        else {
            // ... B의 인터페이스를 사용한다 ...
        }
    }
```

Use of the other casts can violate type safety and cause the program to access a variable that is actually of type `X` to be accessed as if it were of an unrelated type `Z`:

```c++
    void user2(B* pb)   // bad
    {
        D* pd = static_cast<D*>(pb);    // I know that pb really points to a D; trust me
        // ... use D's interface ...
    }

    void user3(B* pb)    // unsafe
    {
        if (some_condition) {
            D* pd = static_cast<D*>(pb);   // I know that pb really points to a D; trust me
            // ... use D's interface ...
        }
        else {
            // ... make do with B's interface ...
        }
    }

    void f()
    {
        B b;
        user(&b);   // OK
        user2(&b);  // bad error
        user3(&b);  // OK *if* the programmer got the some_condition check right
    }
```

##### Note

다른 모든 캐스팅처럼, `dynamic_cast`는 너무 자주 사용된다.

[캐스팅 보다는 가상 함수들을 사용하라](#Rh-use-virtual).
가능한 한 클래스 계층을 탐색하는 것보다 [정적 다형성](#???)을 선호하라. 이렇게 하면 실행시간 결정이 필요없다. 그리고 충분히 편리하다.

##### Note

Some people use `dynamic_cast` where a `typeid` would have been more appropriate;
`dynamic_cast` is a general "is kind of" operation for discovering the best interface to an object,
whereas `typeid` is a "give me the exact type of this object" operation to discover the actual type of an object.
The latter is an inherently simpler operation that ought to be faster.
The latter (`typeid`) is easily hand-crafted if necessary (e.g., if working on a system where RTTI is -- for some reason -- prohibited),
the former (`dynamic_cast`) is far harder to implement correctly in general.

Consider:

```c++
    struct B {
        const char* name {"B"};
        // if pb1->id() == pb2->id() *pb1 is the same type as *pb2
        virtual const char* id() const { return name; }
        // ...
    };

    struct D : B {
        const char* name {"D"};
        const char* id() const override { return name; }
        // ...
    };

    void use()
    {
        B* pb1 = new B;
        B* pb2 = new D;

        cout << pb1->id(); // "B"
        cout << pb2->id(); // "D"


        if (pb1->id() == "D") {         // looks innocent
            D* pd = static_cast<D*>(pb1);
            // ...
        }
        // ...
    }
```

The result of `pb2->id() == "D"` is actually implementation defined.
We added it to warn of the dangers of home-brew RTTI.
This code may work as expected for years, just to fail on a new machine, new compiler, or a new linker that does not unify character literals.

If you implement your own RTTI, be careful.

##### Exception

만약 당신의 구현 코드에 정말로 느린 `dynamic_cast`가 있다면, 다른 방법을 찾아야 할 것이다.
하지만, 정적으로 클래스를 결정할 수 없는 모든 대안은 명시적 캐스팅(일반적으로 `static_cast`)을 포함하고, 에러에 취약하다.  

당신만의 특별한 `dynamic_cast`를 만들수도 있을 것이다. 그러니, `dynamic_cast`가 정말로 당신이 생각하는 것 만큼 느리다는 것을 확실히하라. (근거 없는 루머들이 꽤 있다.) 그리고 `dynamic_cast`의 사용이 정말로 성능에 치명적이라는 것 또한 확인하라.

We are of the opinion that current implementations of `dynamic_cast` are unnecessarily slow.
For example, under suitable conditions, it is possible to perform a `dynamic_cast` in [fast constant time](http://www.stroustrup.com/fast_dynamic_casting.pdf).
However, compatibility makes changes difficult even if all agree that an effort to optimize is worthwhile.

In very rare cases, if you have measured that the `dynamic_cast` overhead is material, you have other means to statically guarantee that a downcast will succeed (e.g., you are using CRTP carefully), and there is no virtual inheritance involved, consider tactically resorting `static_cast` with a prominent comment and disclaimer summarizing this paragraph and that human attention is needed under maintenance because the type system can't verify correctness. Even so, in our experience such "I know what I'm doing" situations are still a known bug source.

##### Exception

Consider:

```c++
    template<typename B>
    class Dx : B {
        // ...
    };
```

##### Enforcement

* Flag all uses of `static_cast` for downcasts, including C-style casts that perform a `static_cast`.
* This rule is part of the [type-safety profile](#Pro-type-downcast)

### <a name="Rh-ref-cast"></a>C.147: Use `dynamic_cast` to a reference type when failure to find the required class is considered an error

##### Reason

참조자에 대한 캐스팅은 당신이 정상적인 객체를 얻는 것을 의도했음을 표현한다. 따라서 캐스팅은 반드시 성공해야만 한다. `dynamic_cast`는 만약 실패한다면 예외를 던질 것이다.

##### Example

    ???

##### Enforcement

???

### <a name="Rh-ptr-cast"></a>C.148: Use `dynamic_cast` to a pointer type when failure to find the required class is considered a valid alternative

##### Reason

The `dynamic_cast` conversion allows to test whether a pointer is pointing at a polymorphic object that has a given class in its hierarchy. Since failure to find the class merely returns a null value, it can be tested during run time. This allows writing code that can choose alternative paths depending on the results.

Contrast with [C.147](#Rh-ptr-cast), where failure is an error, and should not be used for conditional execution.

##### Example

The example below describes the `add` function of a `Shape_owner` that takes ownership of constructed `Shape` objects. The objects are also sorted into views, according to their geometric attributes.
In this example, `Shape` does not inherit from `Geometric_attributes`. Only its subclasses do.

```c++
    void add(Shape* const item)
    {
      // Ownership is always taken
      owned_shapes.emplace_back(item);

      // Check the Geometric_attributes and add the shape to none/one/some/all of the views

      if (auto even = dynamic_cast<Even_sided*>(item))
      {
        view_of_evens.emplace_back(even);
      }

      if (auto trisym = dynamic_cast<Trilaterally_symmetrical*>(item))
      {
        view_of_trisyms.emplace_back(trisym);
      }
    }
```

##### Notes

A failure to find the required class will cause `dynamic_cast` to return a null value, and de-referencing a null-valued pointer will lead to undefined behavior.
Therefore the result of the `dynamic_cast` should always be treated as if it may contain a null value, and tested.

##### Enforcement

* (Complex) Unless there is a null test on the result of a `dynamic_cast` of a pointer type, warn upon dereference of the pointer.

### <a name="Rh-smart"></a>C.149: Use `unique_ptr` or `shared_ptr` to avoid forgetting to `delete` objects created using `new`

##### Reason

자원 누수를 방지한다.

##### Example

```c++
    void use(int i)
    {
        auto p = new int {7};           // bad: initialize local pointers with new
        auto q = make_unique<int>(9);   // ok: guarantee the release of the memory-allocated for 9
        if (0 < i) return;              // maybe return and leak
        delete p;                       // too late
    }
```

##### Enforcement

* `new`를 사용한 일반(naked) 포인터의 초기화를 지적하라
* 지역 변수의 `delete`처리를 지적하라

### <a name="Rh-make_unique"></a>C.150: Use `make_unique()` to construct objects owned by `unique_ptr`s

##### Reason

 `make_unique` gives a more concise statement of the construction.
It also ensures exception safety in complex expressions.

##### Example

```c++
    unique_ptr<Foo> p {new<Foo>{7}};   // OK: but repetitive

    auto q = make_unique<Foo>(7);      // Better: no repetition of Foo

    // Not exception-safe: the compiler may interleave the computations of arguments as follows:
    //
    // 1. allocate memory for Foo,
    // 2. construct Foo,
    // 3. call bar,
    // 4. construct unique_ptr<Foo>.
    //
    // If bar throws, Foo will not be destroyed, and the memory-allocated for it will leak.
    f(unique_ptr<Foo>(new Foo()), bar());

    // Exception-safe: calls to functions are never interleaved.
    f(make_unique<Foo>(), bar());
```

##### Enforcement

* Flag the repetitive usage of template specialization list `<Foo>`
* Flag variables declared to be `unique_ptr<Foo>`

### <a name="Rh-make_shared"></a>C.151: Use `make_shared()` to construct objects owned by `shared_ptr`s

##### Reason

 `make_shared` gives a more concise statement of the construction.
It also gives an opportunity to eliminate a separate allocation for the reference counts, by placing the `shared_ptr`'s use counts next to its object.

##### Example

```c++
    void test() {
        // OK: but repetitive; and separate allocations for the Bar and shared_ptr's use count
        shared_ptr<Bar> p {new<Bar>{7}};

        auto q = make_shared<Bar>(7);   // Better: no repetition of Bar; one object
    }
```

##### Enforcement

* Flag the repetitive usage of template specialization list`<Bar>`
* Flag variables declared to be `shared_ptr<Bar>`

### <a name="Rh-array"></a>C.152: Never assign a pointer to an array of derived class objects to a pointer to its base

##### Reason

Subscripting the resulting base pointer will lead to invalid object access and probably to memory corruption.

##### Example

```c++
    struct B { int x; };
    struct D : B { int y; };

    void use(B*);

    D a[] = {{1, 2}, {3, 4}, {5, 6}};
    B* p = a;     // bad: a decays to &a[0] which is converted to a B*
    p[1].x = 7;   // overwrite D[0].y

    use(a);       // bad: a decays to &a[0] which is converted to a B*
```

##### Enforcement

* Flag all combinations of array decay and base to derived conversions.
* Pass an array as a `span` rather than as a pointer, and don't let the array name suffer a derived-to-base conversion before getting into the `span`

### <a name="Rh-use-virtual"></a>C.153: Prefer virtual function to casting

##### Reason

A virtual function call is safe, whereas casting is error-prone.
A virtual function call reaches the most derived function, whereas a cast may reach an intermediate class and therefore
give a wrong result (especially as a hierarchy is modified during maintenance).

##### Example

    ???

##### Enforcement

See [C.146](#Rh-dynamic_cast) and ???

## <a name="SS-overload"></a>C.over: 오버로딩

You can overload ordinary functions, template functions, and operators.
You cannot overload function objects.

Overload rule summary:

* [C.160: 연산자를 정의할때는 관례적인 사용을 모방하라](#Ro-conventional)
* [C.161: 대칭적인 연산자들에는 비멤버 함수들을 사용하라](#Ro-symmetric)
* [C.162: 거의 동등한 연산들을 오버로드하라](#Ro-equivalent)
* [C.163: 거의 동등한 연산들'만' 오버로드하라](#Ro-equivalent-2)
* [C.164: 형변환 연산자들을 지양하라](#Ro-conversion)
* [C.165: Use `using` for customization points](#Ro-custom)
* [C.166: Overload unary `&` only as part of a system of smart pointers and references](#Ro-address-of)
* [C.167: Use an operator for an operation with its conventional meaning](#Ro-overload)
* [C.168: Define overloaded operators in the namespace of their operands](#Ro-namespace)
* [C.170: 람다를 오버로딩하는 기분이 든다면, 제네릭 람다를 사용하라](#Ro-lambda)

### <a name="Ro-conventional"></a>C.160: Define operators primarily to mimic conventional usage

##### Reason

예상을 벗어나지 않게 한다.

##### Example

```c++
    class X {
    public:
        // ...
        X& operator=(const X&); // member function defining assignment
        friend bool operator==(const X&, const X&); // == needs access to representation
                                                    // after a = b we have a == b
        // ...
    };
```

Here, the conventional semantics is maintained: [Copies compare equal](#SS-copy).

##### Example, bad

```c++
    X operator+(X a, X b) { return a.v - b.v; }   // bad: makes + subtract
```

##### Note

Nonmember operators should be either friends or defined in [the same namespace as their operands](#Ro-namespace).
[Binary operators should treat their operands equivalently](#Ro-symmetric).

##### Enforcement

Possibly impossible.

### <a name="Ro-symmetric"></a>C.161: Use nonmember functions for symmetric operators

##### Reason

If you use member functions, you need two.
Unless you use a nonmember function for (say) `==`, `a == b` and `b == a` will be subtly different.

##### Example

```c++
    bool operator==(Point a, Point b) { return a.x == b.x && a.y == b.y; }
```

##### Enforcement

Flag member operator functions.

### <a name="Ro-equivalent"></a>C.162: Overload operations that are roughly equivalent

##### Reason

Having different names for logically equivalent operations on different argument types is confusing, leads to encoding type information in function names, and inhibits generic programming.

##### Example

Consider:

```c++
    void print(int a);
    void print(int a, int base);
    void print(const string&);
```

These three functions all print their arguments (appropriately). Conversely:

```c++
    void print_int(int a);
    void print_based(int a, int base);
    void print_string(const string&);
```

These three functions all print their arguments (appropriately). Adding to the name just introduced verbosity and inhibits generic code.

##### Enforcement

???

### <a name="Ro-equivalent-2"></a>C.163: Overload only for operations that are roughly equivalent

##### Reason

Having the same name for logically different functions is confusing and leads to errors when using generic programming.

##### Example

Consider:

```c++
    void open_gate(Gate& g);   // remove obstacle from garage exit lane
    void fopen(const char* name, const char* mode);   // open file
```

The two operations are fundamentally different (and unrelated) so it is good that their names differ. Conversely:

```c++
    void open(Gate& g);   // remove obstacle from garage exit lane
    void open(const char* name, const char* mode ="r");   // open file
```

The two operations are still fundamentally different (and unrelated) but the names have been reduced to their (common) minimum, opening opportunities for confusion.
Fortunately, the type system will catch many such mistakes.

##### Note

Be particularly careful about common and popular names, such as `open`, `move`, `+`, and `==`.

##### Enforcement

???

### <a name="Ro-conversion"></a>C.164: Avoid implicit conversion operators

##### Reason

Implicit conversions can be essential (e.g., `double` to `int`) but often cause surprises (e.g., `String` to C-style string).

##### Note

Prefer explicitly named conversions until a serious need is demonstrated.
By "serious need" we mean a reason that is fundamental in the application domain (such as an integer to complex number conversion)
and frequently needed. Do not introduce implicit conversions (through conversion operators or non-`explicit` constructors)
just to gain a minor convenience.

##### Example

```c++
    struct S1 {
        string s;
        // ...
        operator char*() { return s.data(); }  // BAD, likely to cause surprises
    };

    struct S2 {
        string s;
        // ...
        explicit operator char*() { return s.data(); }
    };

    void f(S1 s1, S2 s2)
    {
        char* x1 = s1;     // OK, but can cause surprises in many contexts
        char* x2 = s2;     // error (and that's usually a good thing)
        char* x3 = static_cast<char*>(s2); // we can be explicit (on your head be it)
    }
```

The surprising and potentially damaging implicit conversion can occur in arbitrarily hard-to spot contexts, e.g.,

```c++
    S1 ff();

    char* g()
    {
        return ff();
    }
```

The string returned by `ff()` is destroyed before the returned pointer into it can be used.

##### Enforcement

Flag all conversion operators.

### <a name="Ro-custom"></a>C.165: Use `using` for customization points

##### Reason

To find function objects and functions defined in a separate namespace to "customize" a common function.

##### Example

Consider `swap`. It is a general (standard-library) function with a definition that will work for just about any type.
However, it is desirable to define specific `swap()`s for specific types.
For example, the general `swap()` will copy the elements of two `vector`s being swapped, whereas a good specific implementation will not copy elements at all.

```c++
    namespace N {
        My_type X { /* ... */ };
        void swap(X&, X&);   // optimized swap for N::X
        // ...
    }

    void f1(N::X& a, N::X& b)
    {
        std::swap(a, b);   // probably not what we wanted: calls std::swap()
    }
```

The `std::swap()` in `f1()` does exactly what we asked it to do: it calls the `swap()` in namespace `std`.
Unfortunately, that's probably not what we wanted.
How do we get `N::X` considered?

```c++
    void f2(N::X& a, N::X& b)
    {
        swap(a, b);   // calls N::swap
    }
```

But that may not be what we wanted for generic code.
There, we typically want the specific function if it exists and the general function if not.
This is done by including the general function in the lookup for the function:

```c++
    void f3(N::X& a, N::X& b)
    {
        using std::swap;  // make std::swap available
        swap(a, b);        // calls N::swap if it exists, otherwise std::swap
    }
```

##### Enforcement

Unlikely, except for known customization points, such as `swap`.
The problem is that the unqualified and qualified lookups both have uses.

### <a name="Ro-address-of"></a>C.166: Overload unary `&` only as part of a system of smart pointers and references

##### Reason

The `&` operator is fundamental in C++.
Many parts of the C++ semantics assumes its default meaning.

##### Example

```c++
    class Ptr { // a somewhat smart pointer
        Ptr(X* pp) :p(pp) { /* check */ }
        X* operator->() { /* check */ return p; }
        X operator[](int i);
        X operator*();
    private:
        T* p;
    };

    class X {
        Ptr operator&() { return Ptr{this}; }
        // ...
    };
```

##### Note

If you "mess with" operator `&` be sure that its definition has matching meanings for `->`, `[]`, `*`, and `.` on the result type.
Note that operator `.` currently cannot be overloaded so a perfect system is impossible.
We hope to remedy that: <http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4477.pdf>.
Note that `std::addressof()` always yields a built-in pointer.

##### Enforcement

Tricky. Warn if `&` is user-defined without also defining `->` for the result type.

### <a name="Ro-overload"></a>C.167: Use an operator for an operation with its conventional meaning

##### Reason

Readability. Convention. Reusability. Support for generic code

##### Example

```c++
    void cout_my_class(const My_class& c) // confusing, not conventional,not generic
    {
        std::cout << /* class members here */;
    }

    std::ostream& operator<<(std::ostream& os, const my_class& c) // OK
    {
        return os << /* class members here */;
    }
```

By itself, `cout_my_class` would be OK, but it is not usable/composable with code that rely on the `<<` convention for output:

```c++
    My_class var { /* ... */ };
    // ...
    cout << "var = " << var << '\n';
```

##### Note

There are strong and vigorous conventions for the meaning most operators, such as

* comparisons (`==`, `!=`, `<`, `<=`, `>`, and `>=`),
* arithmetic operations (`+`, `-`, `*`, `/`, and `%`)
* access operations (`.`, `->`, unary `*`, and `[]`)
* assignment (`=`)

Don't define those unconventionally and don't invent your own names for them.

##### Enforcement

Tricky. Requires semantic insight.

### <a name="Ro-namespace"></a>C.168: Define overloaded operators in the namespace of their operands

##### Reason

Readability.
Ability for find operators using ADL.
Avoiding inconsistent definition in different namespaces

##### Example

```c++
    struct S { };
    bool operator==(S, S);   // OK: in the same namespace as S, and even next to S
    S s;

    bool x = (s == s);
```

This is what a default `==` would do, if we had such defaults.

##### Example

```c++
    namespace N {
        struct S { };
        bool operator==(S, S);   // OK: in the same namespace as S, and even next to S
    }

    N::S s;

    bool x = (s == s);  // finds N::operator==() by ADL
```

##### Example, bad

```c++
    struct S { };
    S s;

    namespace N {
        S::operator!(S a) { return true; }
        S not_s = !s;
    }

    namespace M {
        S::operator!(S a) { return false; }
        S not_s = !s;
    }
```

Here, the meaning of `!s` differs in `N` and `M`.
This can be most confusing.
Remove the definition of `namespace M` and the confusion is replaced by an opportunity to make the mistake.

##### Note

If a binary operator is defined for two types that are defined in different namespaces, you cannot follow this rule.
For example:

```c++
    Vec::Vector operator*(const Vec::Vector&, const Mat::Matrix&);
```

This may be something best avoided.

##### See also

This is a special case of the rule that [helper functions should be defined in the same namespace as their class](#Rc-helper).

##### Enforcement

* Flag operator definitions that are not it the namespace of their operands

### <a name="Ro-lambda"></a>C.170: If you feel like overloading a lambda, use a generic lambda

##### Reason

You cannot overload by defining two different lambdas with the same name.

##### Example

```c++
    void f(int);
    void f(double);
    auto f = [](char);   // error: cannot overload variable and function

    auto g = [](int) { /* ... */ };
    auto g = [](double) { /* ... */ };   // error: cannot overload variables

    auto h = [](auto) { /* ... */ };   // OK
```

##### Enforcement

The compiler catches the attempt to overload a lambda.

## <a name="SS-union"></a>C.union: 공용체(Union)

A `union` is a `struct` where all members start at the same address so that it can hold only one member at a time.
A `union` does not keep track of which member is stored so the programmer has to get it right;
this is inherently error-prone, but there are ways to compensate.

A type that is a `union` plus an indicator of which member is currently held is called a *tagged union*, a *discriminated union*, or a *variant*.

공용체(Unions) 규칙 요약:

* [C.180: Use `union`s to save Memory](#Ru-union)
* [C.181: Avoid "naked" `union`s](#Ru-naked)
* [C.182: Use anonymous `union`s to implement tagged unions](#Ru-anonymous)
* [C.183: Don't use a `union` for type punning](#Ru-pun)
* ???

### <a name="Ru-union"></a>C.180: Use `union`s to save memory

##### Reason

A `union` allows a single piece of memory to be used for different types of objects at different times.
Consequently, it can be used to save memory when we have several objects that are never used at the same time.

##### Example

```c++
    union Value {
        int x;
        double d;
    };

    Value v = { 123 };  // now v holds an int
    cout << v.x << '\n';    // write 123
    v.d = 987.654;  // now v holds a double
    cout << v.d << '\n';    // write 987.654
```

But heed the warning: [Avoid "naked" `union`s](#Ru-naked)

##### Example

```c++
    // Short-string optimization

    constexpr size_t buffer_size = 16; // Slightly larger than the size of a pointer

    class Immutable_string {
    public:
        Immutable_string(const char* str) :
            size(strlen(str))
        {
            if (size < buffer_size)
                strcpy_s(string_buffer, buffer_size, str);
            else {
                string_ptr = new char[size + 1];
                strcpy_s(string_ptr, size + 1, str);
            }
        }

        ~Immutable_string()
        {
            if (size >= buffer_size)
                delete string_ptr;
        }

        const char* get_str() const
        {
            return (size < buffer_size) ? string_buffer : string_ptr;
        }

    private:
        // If the string is short enough, we store the string itself
        // instead of a pointer to the string.
        union {
            char* string_ptr;
            char string_buffer[buffer_size];
        };

        const size_t size;
    };
```

##### Enforcement

???

### <a name="Ru-naked"></a>C.181: Avoid "naked" `union`s

##### Reason

A *naked union* is a union without an associated indicator which member (if any) it holds,
so that the programmer has to keep track.
Naked unions are a source of type errors.

##### Example, bad

```c++
    union Value {
        int x;
        double d;
    };

    Value v;
    v.d = 987.654;  // v holds a double
```

So far, so good, but we can easily misuse the `union`:

```c++
    cout << v.x << '\n';    // BAD, undefined behavior: v holds a double, but we read it as an int
```

Note that the type error happened without any explicit cast.
When we tested that program the last value printed was `1683627180` which it the integer value for the bit pattern for `987.654`.
What we have here is an "invisible" type error that happens to give a result that could easily look innocent.

And, talking about "invisible", this code produced no output:

```c++
    v.x = 123;
    cout << v.d << '\n';    // BAD: undefined behavior
```

##### Alternative

Wrap a `union` in a class together with a type field.

The soon-to-be-standard `variant` type (to be found in `<variant>`) does that for you:

```c++
    variant<int, double> v;
    v = 123;        // v holds an int
    int x = get<int>(v);
    v = 123.456;    // v holds a double
    w = get<double>(v);
```

##### Enforcement

???

### <a name="Ru-anonymous"></a>C.182: Use anonymous `union`s to implement tagged unions

##### Reason

A well-designed tagged union is type safe.
An *anonymous* union simplifies the definition of a class with a (tag, union) pair.

##### Example

This example is mostly borrowed from TC++PL4 pp216-218.
You can look there for an explanation.

The code is somewhat elaborate.
Handling a type with user-defined assignment and destructor is tricky.
Saving programmers from having to write such code is one reason for including `variant` in the standard.

```c++
    class Value { // two alternative representations represented as a union
    private:
        enum class Tag { number, text };
        Tag type; // discriminant

        union { // representation (note: anonymous union)
            int i;
            string s; // string has default constructor, copy operations, and destructor
        };
    public:
        struct Bad_entry { }; // used for exceptions

        ~Value();
        Value& operator=(const Value&);   // necessary because of the string variant
        Value(const Value&);
        // ...
        int number() const;
        string text() const;

        void set_number(int n);
        void set_text(const string&);
        // ...
    };

    int Value::number() const
    {
        if (type != Tag::number) throw Bad_entry{};
        return i;
    }

    string Value::text() const
    {
        if (type != Tag::text) throw Bad_entry{};
        return s;
    }

    void Value::set_number(int n)
    {
        if (type == Tag::text) {
            s.~string();      // explicitly destroy string
            type = Tag::number;
        }
        i = n;
    }

    void Value::set_text(const string& ss)
    {
        if (type == Tag::text)
            s = ss;
        else {
            new(&s) string{ss};   // placement new: explicitly construct string
            type = Tag::text;
        }
    }

    Value& Value::operator=(const Value& e)   // necessary because of the string variant
    {
        if (type == Tag::text && e.type == Tag::text) {
            s = e.s;    // usual string assignment
            return *this;
        }

        if (type == Tag::text) s.~string(); // explicit destroy

        switch (e.type) {
        case Tag::number:
            i = e.i;
            break;
        case Tag::text:
            new(&s)(e.s);   // placement new: explicit construct
            type = e.type;
        }

        return *this;
    }

    Value::~Value()
    {
        if (type == Tag::text) s.~string(); // explicit destroy
    }
```

##### Enforcement

???

### <a name="Ru-pun"></a>C.183: Don't use a `union` for type punning

##### Reason

It is undefined behavior to read a `union` member with a different type from the one with which it was written.
Such punning is invisible, or at least harder to spot than using a named cast.
Type punning using a `union` is a source of errors.

##### Example, bad

```c++
    union Pun {
        int x;
        unsigned char c[sizeof(int)];
    };
```

The idea of `Pun` is to be able to look at the character representation of an `int`.

```c++
    void bad(Pun& u)
    {
        u.x = 'x';
        cout << u.c[0] << '\n';     // undefined behavior
    }
```

If you wanted to see the bytes of an `int`, use a (named) cast:

```c++
    void if_you_must_pun(int& x)
    {
        auto p = reinterpret_cast<unsigned char*>(&x);
        cout << p[0] << '\n';     // OK; better
        // ...
    }
```

Accessing the result of an `reinterpret_cast` to a different type from the objects declared type is defined behavior (even though `reinterpret_cast` is discouraged),
but at least we can see that something tricky is going on.

##### Note

Unfortunately, `union`s are commonly used for type punning.
We don't consider "sometimes, it works as expected" a strong argument.

C++17 introduced a distinct type `std::byte` to facilitate operations on raw object representation.  Use that type instead of `unsigned char` or `char` for these operations.

##### Enforcement

???
