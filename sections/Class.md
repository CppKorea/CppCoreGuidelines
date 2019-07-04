
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
* [C.lambdas: 함수 개체와 람다 표현식](#SS-lambdas)
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

### <a name="Rc-struct"></a>C.2: 타입이 불변조건을 가진다면, `class`를 사용하라; 데이터 멤버들에 대한 제약이 자유롭다면 `struct`를 사용하라

##### Reason

가독성이 좋고 이해하기도 쉽다.
`class` 를 사용함으로써, 프로그래머가 불변조건(invariant)이 필요하다는 것을 알게 된다.  
이 점은 유익한 관습이다.

##### Note

invariant는 개체 멤버들의 논리적인 상태로써, 공개 멤버 함수들이 가정할 수 있도록 생성자가 설정 해 주어야 한다. invariant 가 설정된 후에야 (일반적으로 생성자에 의해) 모든 멤버 함수는 개체를 통해 호출될 수 있다.
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

클래스가 어떤 `private` 데이터를 가지고 있으면, 사용자는 생성자 호출 없이 개체를 초기화할 수 없다. 
따라서, 클래스를 정의하는 사람은 생성자를 제공하고 그 의미를 명시해야만 한다.
이는 클래스 작성자가 invariant를 정의해야 한다는 것을 의미한다.

##### See also

* [private 데이터를 가지고 있다면 `class`를 사용하라](#Rc-class)
* [클래스 정의에 인터페이스를 먼저 배치하라](#Rl-order)
* [멤버의 노출을 최소화하라](#Rc-private)
* [`protected` 데이터 사용을 지양하라](#Rh-protected)

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

멤버 함수간 커플링을 줄인다. 개체 상태 변경에 의해 문제가 생기는 함수를 줄인다. 표현이 변경된 후에 수정될 필요가 있는 멤버 함수의 수를 줄인다.

##### Example

```c++
    class Date {
        // ... 상대적으로 적은 인터페이스 ...
    };

    // helper functions:
    Date next_weekday(Date);
    bool operator==(Date, Date);
```

"helper functions"으로 표시된 함수들은 `Date`의 내부에 접근할 필요가 없다.

##### Note

["uniform function call"](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0251r0.pdf)이 가능해지면 더 좋아질 것이다.

##### Exception

C++ 에서는 멤버 함수만이 `virtual` 함수가 될 수 있지만, 모든 `virtual`가 멤버에 접근하는 것은 아니다. 특히 추상 클래스들은 멤버에 접근하는 경우가 드물다.

[multi-methods](https://parasol.tamu.edu/~yuriys/papers/OMM10.pdf)를 확인하라.

##### Exception

C++ 언어에서 `=`, `()`, `[]`, `->` 연산자는 멤버함수여야 한다.

##### Exception

중복정의 집합에 `private` 데이터에 직접 접근하지 않는 멤버가 있을 수 있다:
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

유사하게, 어떤 함수들은 연속적으로 호출하도록 설계되었을 수 있다:

```c++
    x.scale(0.5).rotate(45).set_color(Color::red);
```

일반적으로, 이런 함수들 중 일부는 `private` 데이터에 접근한다.

##### Enforcement

* 데이터 멤버에 직접 접근하지 않는 비 가상 멤버 함수를 찾아낸다. 이런 함수는 많은 멤버 함수들이 데이터 멤버를 직접 접근할 필요가 없음을 의미한다
* `virtual` 함수들은 무시한다
* 중복정의(overload) 하는 함수는 그 중 하나가 `private` 데이터 멤버에 접근하지 않는 한 무시한다
* `this`를 반환하는 함수들은 무시한다

### <a name="Rc-helper"></a>C.5: 보조 함수들은 관련 클래스와 같은 namespace에 배치하라

##### Reason

보조 함수(helper function)는 (보통 클래스 작성자가 제공하는) 클래스의 표현에 직접 접근할 필요가 없는 함수이며, 클래스에 대한 유용한 인터페이스 중에 하나로 볼 수 있다.
보조 함수들을 같은 네임스페이스에 넣으면 함수와 클래스의 관계가 명확해지고, Argument Dependent Lookup에서 발견 할 수 있게 된다.

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

클래스 인터페이스를 먼저 배치하라. [NL.16을 참고하라](./Naming.md#Rl-order)

##### Enforcement

`private` 혹은 `protected` 멤버를 가지지만 `struct`로 선언된 클래스를 지적한다

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

`//` 부분에 어떤 코드가 작성되건, `pair`의 사용자는 `a`와 `b`를 독립적으로 변경할 수 있다.
코드 규모가 큰 경우, `pair`의 멤버에 어떤 일이 일어나는지 찾기 어렵다.

독립적으로 변경하는 것이 의도에 맞을 수 있지만, 멤버간의 관계를 강제하고 싶다면, `private`로 변경하고 그 관계(불변조건)를 생성자와 멤버 함수들로 지키도록 해야 한다.

예를 들자면:

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

만약 변수들에 접근하는 코드를 쉽게 결정할 수 없다면, 그 타입이나 사용을 (쉽게) 변경하거나 개선하기 어렵다.
`public`과 `protected` 데이터는 보통 이 경우에 해당한다.

##### Example

클래스는 사용자에게 두가지 인터페이스를 제공할 수 있다. 하나는 상속받는 클래스에게 제공하는 `protected`이며 하나는 일반적으로 사용 가능한 `public`이다.
예를 들면, 하위 클래스는 상위 클래스의불변조건이 유지된다는 것을 확실히 할 수 있다면 실행시간 검사를 생략 할수도 있다:

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

[`protected` 데이터는 좋은 생각이 아니다](#Rh-protected).

##### Note

`public`멤버를 가장 앞에, `protected` 멤버를 다음에, `private`멤버를 마지막에 배치하라.

##### Enforcement

* [protected 데이터를 지적하라](#Rh-protected)
* `public`과 `private` 데이터가 함께 사용된 경우를 지적하라

## <a name="SS-concrete"></a>C.concrete: 실제 타입(Concrete types)

이상적인 클래스는 정규 타입(Regular Type)과 같아야 한다.
쉽게 말하면 "`int` 처럼 동작하는 것"이다. 실제 타입(Concrete type)이란 가장 간단한 종류의 클래스를 의미한다.

> 역주:  
> Regular Type은 다음의 조건을 모두 만족하는 타입을 의미합니다.
> - DefaultConstructible
> - CopyConstructible, CopyAssignable
> - MoveConstructible, MoveAssignable
> - Destructible
> - Swappable
> - EqualityComparable
>
> 예시로 언급된 `int`의 경우, 기본 연산(생성, 파괴, 복사, 이동)을 지원하면서 교환, 동등비교가 가능합니다

정규 타입의 값은 복사 될 수 있고, 복사의 결과는 원본과 같은 값을 갖는 독립적인 개체이다. 타입이 `=` 와 `==` 를 모두 갖는다면, `a = b`를 실행한 이후에는 `a == b`에서 `true`가 반환되도록 해야 한다.
실제 타입이 대입과 동등 비교를 지원하지 않을 수 있지만, 그런 경우는 드물다 (거의 없어야 한다).

C++의 언어 내장(built-in) 타입들은 정규적(Regular)이고, `string`, `vector`, `map`같은 표준 라이브러리의 클래스들 또한 그렇다. 실제 타입들은 종종 계층구조의 일부로 사용되는 타입들과 구분하여 값 타입으로 언급된다.

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

클래스가 계층구조의 일부가 될 수 있다면, 반드시 포인터나 레퍼런스로 개체를 다루어야 한다.
이는 간접 처리를 위해 더 많은 메모리를 사용하게 되고, 더 많은 할당과 해제, 실행시간 오버헤드가 발생하게 된다는 것을 의미한다.

##### Note

실제 타입은 스택에 할당 될 수 있고, 다른 클래스의 멤버가 될 수 있다.

##### Note

실행시간에 다형적 인터페이스를 위해 간접처리는 필수적이다.
할당과 해제의 추가비용은 그렇지 않다. (단지 가장 흔한 사례일 뿐이다)
패생 클래스의 제한된(특정된) 개체에 대한 인터페이스로써 기본 클래스를 사용할 수도 있다.
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

이 함수들은 개체의 생명주기를 제어 한다: 생성, 복사, 이동, 그리고 소멸.
생성자를 정의해서 클래스의 초기화를 보장하고 단순화 하라.

*기본 연산*은 아래와 같은 연산들을 의미한다.

* 기본 생성자: `X()`
* 복사 생성자: `X(const X&)`
* 복사 대입 연산자: `operator=(const X&)`
* 이동 생성자: `X(X&&)`
* 이동 대입 연산자: `operator=(X&&)`
* 소멸자: `~X()`

이상의 연산들은 정의하지 않아도 코드에서 사용되면 컴파일러가 생성한다. 하지만 기본연산을 제한하는 것도 가능하다.

기본 연산은 개체의 수명주기와 관련된 연산들의 집합을 의미한다.

코드가 명시하지 않는 한, C++은 클래스를 값 타입 처럼 다루지만 모든 타입이 값 타입처럼 동작하는 것은 아니다.

기본 연산 규칙들:

* [C.20: 기본 연산을 정의하지 않아도 되면 그렇게 하라](#Rc-zero)
* [C.21: 기본 연산을 정의 하거나 `=delete` 로 선언했다면, 나머지 모두 정의하거나 `=delete`하라](#Rc-five)
* [C.22: 기본 연산들을 서로 조화롭게 동작해야 한다](#Rc-matched)

소멸자 규칙들:

* [C.30: 개체가 없어질 때, 명시적인 동작이 필요할 경우 소멸자를 정의하라](#Rc-dtor)
* [C.31: 클래스가 획득한 모든 자원은 소멸자에서 해제되어야 한다](#Rc-dtor-release)
* [C.32: 클래스가 포인터(`T*`)나 참조(`T&`)를 가지고 있다면, 참조 대상을 소유하고 있는지를 고려하라](#Rc-dtor-ptr)
* [C.33: 클래스가 포인터로 대상을 소유하고 있다면, 소멸자를 정의하라](#Rc-dtor-ptr2)
* [C.35: 상위 클래스의 소멸자는 공개된 가상 소멸자 혹은 상속되는 비-가상 함수여야 한다](#Rc-dtor-virtual)
* [C.36: 소멸자는 실패해선 안된다](#Rc-dtor-fail)
* [C.37: 소멸자를 `noexcept`로 작성하라](#Rc-dtor-noexcept)

생성자 규칙들:

* [C.40: 클래스가 불변 조건을 가진다면 생성자를 정의하라](#Rc-ctor)
* [C.41: 생성자는 완전히 초기화된 개체를 생성해야 한다](#Rc-complete)
* [C.42: 생성자가 유효한 개체를 생성하지 못한다면, 예외를 던지도록 하라](#Rc-throw)
* [C.43: 복사 가능한 클래스(값 타입)는 반드시 기본 생성자를 갖도록 하라](#Rc-default0)
* [C.44: 기본 생성자는 가능한 단순하고 예외를 던지지 않도록 하라](#Rc-default00)
* [C.45: 멤버를 초기화 하기만 하는 기본 생성자는 정의하지 마라; 대신 멤버들이 스스로 초기화 하도록 하라](#Rc-default)
* [C.46: 단일 인자를 사용하는 생성자는 `explicit`으로 선언하라](#Rc-explicit)
* [C.47: 멤버 변수들은 선언된 순서대로 초기화하라](#Rc-order)
* [C.48: 상수 초기화는 가능한 클래스 내(in-class) 멤버 초기화를 사용하라](#Rc-in-class-initializer)
* [C.49: 생성자 안에서의 대입 보다는 초기화를 선호하라](#Rc-initialize)
* [C.50: 초기화 과정에서 `virtual` 동작이 필요하다면, 팩토리 함수를 사용하라](#Rc-factory)
* [C.51: 클래스의 모든 생성자들을 위한 일반적인 동작을 표현할 때는 대리 생성자를 사용하라](#Rc-delegating)
* [C.52: 추가적인 초기화가 필요하지 않은 파생된 클래스에서 생성자를 사용할 때는 상속 생성자들을 사용하라](#Rc-inheriting)

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

### <a name="Rc-zero"></a>C.20: 기본 연산을 정의하지 않아도 되면 그렇게하라

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

`std::map` 과 `string` 은 모든 특수한 함수들을 갖고 있다, 추가로 코드를 작성할 필요가 없다.

##### Note

"The rule of zero"로 알려져 있다.

##### Enforcement

(Not enforceable) 시행할 수 없더라도, 좋은 정적 분석기는 이 규칙에 맞는 가능한 개선사항들을 알려주는 패턴들을 찾을 수 있다.
예를 들면, 포인터와 크기를 멤버로 갖는 클래스가 있고 소멸자에서 그 포인터를 `delete` 한다면 아마도 `vector` 로 바꿀 수 있을 것이다.

### <a name="Rc-five"></a>C.21: 기본 연산을 정의 하거나 `=delete` 로 선언했다면, 나머지 모두 정의하거나 `=delete`하라

##### Reason

이 *특별한 멤버 함수들*은 기본 생성자, 복사 생성자, 복사 대입 연산자, 이동 생성자, 이동 대입 연산자, 소멸자를 의미한다.

이들의 의미는 서로 밀접하게 연관되어 있다. 만약 한 함수가 기본 제공 함수가 아니어야 한다면(non-default), 다른 함수들도 수정이 필요하다.

기본 생성자를 제외하고 이 특별한 멤버 함수들 중 하나를 `=default` 혹은 `=delete`로 선언할 경우, 컴파일러가 이동 생성자와 이동 대입 연산자를 묵시적으로 선언하지 않는다.
이동 생성자 또는 이동 대입 연산자를 선언하는 경우, 복사 생성자와 복사 대입 연산자가 이를 따른다. 이동 연산이 `=default`로 선언된 경우 복사 연산이 자동으로 정의되며, 이동 연산이 `=delete`로 선언된 경우 복사 연산도 `=delete`가 적용된다.
따라서, 이 특별 함수들 중 하나라도 선언되었다면, 의도치 않은 복사와 이동연산을 피하기 위해 나머지 함수들도 선언되어야 한다.

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

기본 생성자를 중요하게 생각하는지에 달려있는데, 이것은 "The rule of five" 혹은 "The rule of six" 이라고 알려져 있다.

##### Note

다른 것은 정의 하더라도 기본 연산의 기본 구현이 필요하다면, `=default` 을 사용하여 해당 함수에 대한 의도를 표현하라.
기본 연산을 원하지 않는다면, `=delete`를 써서 제한하라.

##### Example, good

단순히 `virtual`을 위해 소멸자가 선언되어야 한다면, `=default`를 사용해 정의할 수 있다. 
묵시적으로 이동 연산이 제한되는 것을 막고 싶다면 이동 연산들이 선언되어야 한다. 만약 이동 연산만 지원하는게 아니라면 (즉 복사가 가능해야 한다면) 복사 연산 역시 그에 맞게 선언해야 한다:

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

[C.67](#Rc-copy-virtual)을 고려해서 복사 절단(slicing) 문제를 예방하기 위해 복사와 이동연산을 제한할 수도 있다:

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

복사를 지원하지 않는다면 이동 연산들을 `=delete`로 정의하거나 복사연산을 `=delete`하는 경우 모두 같은 효과를 가진다. 하지만 타입의 의도를 분명히 전달하기 위해서는 모든 특별 함수들을 정의하는 것이 좋다.

##### Note

컴파일러는 이 규칙을 강제하고, 이상적으로는 위반사항이 발생하면 경고한다.

##### Note

클래스에 묵시적으로 생성된 복사 연산에 의존하는 것은 더 이상 사용되지 않는다.

##### Note

여섯개의 특별 함수들을 모두 작성하는 것은 오류에 취약할 수 있다. 아래 예시의 인자 타입에 주목하라:

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

스펠링을 잘못 적거나, `const`를 빠뜨리거나, `&&`대신 `&`을 사용하거나, 하나를 빠뜨리는 것 같은 사소한 실수가 오류나 경고로 이어질 수 있다.
이런 (지루한 코드로 인해 발생하는) 오류를 피하고자 한다면 [The rule of zero](#Rc-zero)를 따르는 것을 권한다.

##### Enforcement

(쉬움) 클래스는 특별한 함수들에 대한 선언(`=delete`도 포함하여)을 모두 갖거나 하나도 없어야 한다.

### <a name="Rc-matched"></a>C.22: 기본 연산들을 서로 조화롭게 동작해야 한다

##### Reason

기본 연산들은 개념적으로 잘 짜여진 집합이다. 연산들의 의미는 서로 연관되어 있다.
사용자는 복사/이동 생성과 복사/이동 할당이 논리적으로 동일하고, 생성자와 소멸자가 리소스 관리에 대해 일관적으로 동작하며, 복사와 이동이 생성자와 소멸자가 동작하는 방식을 반영한다는 것을 기대 할 것이다. 만약 복사와 이동이 생성과 소멸에 영향을 주지 않는다면 사용자에게 혼란을 줄 것이다.

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
대부분의 클래스들에 대해서 대답은 "no"인데, 그 이유는 해당 클래스가 자원들을 가지고 있지 않거나 소멸과정이 [The rule of zero](#Rc-zero)에 의해 처리되기 때문이다.

요컨대, 클래스의 멤버들이 스스로의 소멸을 관리한다는 것이다.
만약 대답이 "yes"라면, 그 클래스 설계의 대부분은 [The rule of five](#Rc-five)를 따르게 된다.

### <a name="Rc-dtor"></a>C.30: 개체가 없어질 때, 명시적인 동작이 필요할 경우 소멸자를 정의하라

##### Reason

소멸자는 암묵적으로 개체의 생명주기의 마지막에 호출된다.
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
* 트레이싱이나 `final_action`처럼 소멸시기에 어떤 동작을 발생시키기 위한 클래스

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

### <a name="Rc-dtor-release"></a>C.31: 클래스가 획득한 모든 자원은 소멸자에서 해제되어야 한다

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
초심자들은 소멸자를 작성할 때 왜 소멸자가 호출되고, 예외를 던짐으로써 "처리를 거부"를 할 수 없는지 알지 못할 것이다. 이에 대해서는 [소멸자는 실패해선 안된다](#Sd-never-fail)를 참고하라.

문제를 악화시키는 것은, 많은 "닫기/해제" 연산들이 재시도 할 수 없도록 되어있는 것이다.
이 문제를 풀려는 시도는 많았지만, 일반적인 해결책은 알려지지 않았다.
해결책이 없다면, 닫기/해제에 대한 실패를 디자인 오류로 간주하고 종료시키는 것을 고려해 보라.

##### Note

클래스가 소유하고 있지 않은 개체에 대한 포인터나 참조를 갖고 있을 수 있다.
당연하지만, 이 개체들은 클래스의 소멸자에서 `delete`되지 않아야 한다.
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

### <a name="Rc-dtor-ptr"></a>C.32: 클래스가 포인터(`T*`)나 참조(`T&`)를 가지고 있다면, 참조 대상을 소유하고 있는지를 고려하라

##### Reason

소유권에 대해서 상세하지 않은 코드는 많이 있다.

##### Example

```
    ???
```

##### Note

`T*` 혹은 `T&` 가 소유를 의미한다면, **소유한다는** 표시를 하라. `T*` 에 소유의 의미가 없다면 `ptr` 로 표시하는 것을 고려하라.
이것은 문서화와 분석에 도움이 될 것이다.

##### Enforcement

포인터나 참조를 초기화 할 때 자원할당이 발생하는지 확인하라.

### <a name="Rc-dtor-ptr2"></a>C.33: 클래스가 포인터로 대상을 소유하고 있다면, 소멸자를 정의하라

##### Reason

소유된 개체는 그것을 소유한 개체가 소멸될 때 `삭제`되어야 한다.

##### Example

포인터 멤버는 리소스일 것이다. [`T*`는 리소스가 아니어야 한다](./Resource.md#Rr-ptr), 이는 오래된 코드에서는 일반적이다.
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
드물게는 중대한 코드 변경이 필요해지고 ABI 에 영향을 줄 수도 있다.

##### Enforcement

* 포인터 데이터 맴버를 갖는 클래스를 의심하라
* `owner<T>` 를 갖는 클래스는 기본 연산들을 정의 해야한다

### <a name="Rc-dtor-virtual"></a>C.35: 상위 클래스의 소멸자는 공개된 가상 소멸자 혹은 상속되는 비-가상 함수여야 한다

##### Reason

미정의 동작(undefined behavior)을 막기 위한 규칙이다.

만약 소멸자가 `public` 이면, 호출하는 코드는 파생 클래스가 기본 클래스의 포인터를 통해 소멸될 것이라 생각한다. 그리고 기본 클래스의 소멸자가 `virtual`이 아니면 결과는 미정의 동작으로 이어진다.

만약 소멸자가 `protected`라면, 호출하는 코드는 기본 클래스의 포인터를 통해서 소멸시킬 수 없고, 따라서 소멸자는 `virtual`이 아니어도 문제가 없다. `private`가 아닌 `protected`여야 하는 이유는 파생 클래스의 소멸자가 호출할 수 있어야 하기 때문이다.

일반적으로, 기본 클래스의 작성자는 소멸 과정에서 어떤 동작이 적합한지 알 수 없다.  

##### Discussion

[토론](./Discussion.md#Sd-dtor)을 함께 읽어보라.

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

protected virtual 소멸자를 원하지 않는 경우를 상상해볼 수 있다. 파생 타입의 개체가 기본 타입 포인터를 통해 (그 자신이 아닌) *다른* 개체의 소멸을 하도록 허용해야 하는 경우가 그러하다. 하지만 아직까지 그런 사례를 볼 수 없었다.

##### Enforcement

* 가상 함수를 하나라도 가지는 클래스는 `public` 하고 `virtual`한 소멸자를 가져야 한다. 또는 `protected`이고 `virtual`이 아닌 소멸자를 가져야 한다.

### <a name="Rc-dtor-fail"></a>C.36: 소멸자는 실패해선 안된다

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

[토론](./Discussion.md#Sd-dtor)을 함께보라.
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

### <a name="Rc-dtor-noexcept"></a>C.37: 소멸자를 `noexcept`로 작성하라

##### Reason

[소멸자는 실패해선 안된다](#Rc-dtor fail).  
만약 소멸자가 예외로 인해 종료되려고 한다면, 좋지 않은 디자인 오류로 보고 종료하는 편이 나을 것이다.

##### Note

사용자가 정의하였건 컴파일러가 생성하였건 모든 멤버 변수들의 소멸자가 `noexcept`라면 소멸자는 암묵적으로 `noexcept`가 된다 (함수의 코드가 어떻게 작성되었는지는 고려되지 않는다).
명시적으로 소멸자를 `noexcept`로 표기함으로써, 그 코드의 작성자는 나중에 멤버가 추가되거나 변경되면서 소멸자가 `noexcept(false)`로 변하는 것을 차단할 수 있다.

##### Example

모든 소멸자가 `noexcept`를 기본으로 하지는 않는다; 예외를 던지는 하나의 멤버가 모든 클래스 계층구조에 영향을 줄 수 있다.

```c++
    struct X {
        Details x;  // happens to have a throwing destructor
        // ...
        ~X() { }    // implicitly noexcept(false); aka can throw
    };
```

만약 의심이 생긴다면, 소멸자는 `noexcept`로 선언하라.

##### Note

소멸자를 `noexcept`로 선언하지 않을 이유가 없다. `noexcept(false)`는 많은 경우 -- 특히 단순한 코드에서 -- 혼란을 발생시킨다.

##### Enforcement

(쉬움) 소멸자는 `noexcept`로 선언되어야 한다.

## <a name="SS-ctor"></a>C.ctor: 생성자

생성자는 개체가 생성되는(초기화되는) 방법을 정의 한다.

### <a name="Rc-ctor"></a>C.40: 클래스가 불변 조건을 가진다면 생성자를 정의하라

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

##### See also:

* [유효한 개체를 생성하라](#Rc-complete)
* [생성자가 던지는 예외](#Rc-throw)

##### Enforcement

* 사용자 정의 복사 연산이 있지만 소멸자가 없는 클래스를 지적한다 (사용자 정의 복사는 클래스가 불변조건을 가진다는 것을 알려준다)

### <a name="Rc-complete"></a>C.41: 생성자는 완전히 초기화된 개체를 생성해야 한다

##### Reason

생성자는 클래스에 대한 불변조건을 설정한다. 클래스 사용자는 생성된 개체가 사용가능하다는 것을 가정할 수 있어야 한다.

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

생성자만으로 유효한 개체를 쉽게 만들 수 없다면 [팩토리 함수를 사용하라](#Rc-factory)

##### Enforcement

* (단순) 모든 생성자는 해당 클래스의 모든 멤버변수들을 초기화해야 한다 (명시적인 생성자 위임 혹은 기본 생성을 통해서)
* (불분명함) 만약 생성자가 `Ensures` 계약을 가지고 있다면, 생성 후 조건이 존재하는지 확인하라

##### Note

생성자가 유효한 개체를 만들기 위해 자원을 얻는다면, 리소스는 [소멸자에 의해 해제](#Rc-release)되어야 한다.
생성자에서 자원을 얻고 소멸자에서 자원을 해제하는 것을 [RAII](./Resource.md#Rr-raii) ("Resource Acquisitions Is Initialization") 라고 한다.

### <a name="Rc-throw"></a>C.42: 생성자가 유효한 개체를 생성하지 못한다면, 예외를 던지도록 하라

##### Reason

유효하지 않은 개체를 남겨두는 것은 문제를 일으킬 것이다.

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
    class X3 {     // bad: 생성자가 유효하지 않은 개체를 남겨놓을 수 있다
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

변수를 정의할 때는 (가령, 스택에 혹은 다른 개체의 멤버로써) 오류코드가 리턴되는 명시적인 함수 호출은 없다.
유효하지 않은 개체를 남겨두고 사용하기 전에 지속적으로 `is_valid()` 함수를 호출해야 하는 것은 번거롭고, 오류가 발생하기 쉬우며, 비효율적 이다.

##### Exception

(추가적인 툴 지원 없이) 예외 처리가 예측 가능한 시간 내로 수행되어야 하는 실시간 시스템(비행기 제어를 생각해 보라)과 같은 분야도 있다.

이런 경우엔 `is_valid()` 와 같은 방법이 반드시 사용되어야 한다. 이와 같은 경우 [RAII](#Rc-raii)처럼 동작하도록 하기 위해 지속적으로 `is_valid()` 로 확인하라.

##### Alternative

"생성자 이후 초기화" 혹은 "두 단계 초기화"를 사용해야 할 것 같다면, 그렇게 하지 않도록 해보라. 정말로 그렇게 해야 한다면 [팩토리 함수](#Rc-factory)를 검토하라.

##### Note

사람들이 생성자에서 초기화를 수행하지 않고 `init()`함수를 사용해온 이유 중 하나는 코드의 중복을 막기 위함이었다.
[대리 생성자](#Rc-delegating)와 [기본 멤버 초기화](#Rc-in-class-initializer)가 이런 작업을 더 잘 해낼 수 있다.

또 다른 이유로는 개체가 필요할 때까지 초기화를 지연시키는 것이다; 이러한 해법은 보통 [변수가 적절하게 초기화되기 전까지는 해당 변수를 선언하지 않는 것이다](./Expr.md#Res-init).

##### Enforcement

???

### <a name="Rc-default0"></a>C.43: 복사 가능한 클래스(값 타입)는 반드시 기본 생성자를 갖도록 하라

##### Reason

많은 언어나 라이브러리들이 기본 생성자에 의존하고 있다.  
예를 들면, `T a[10]` 나 `std::vector<T> v(10)` 는 기본 생성자들이 각 요소를 초기화 한다.
기본 생성자는 보통 복사 가능한 타입이 적합한 이동 된 상태를 정의하기 쉽도록 한다.

##### Note

[값 타입](#SS-concrete)은 복사 가능한 타입을 의미한다 (많은 경우 보통 비교 가능하다).
[EoP](http://elementsofprogramming.com/)와 [The Palo Alto TR](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3351.pdf)에서 나온 정규 타입 개념과 밀접한 관련이 있다.

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

`Date`를 바탕으로 생각해보자:  
"자연적인" 기본 날짜라는 것은 존재하지 않는다 (빅뱅은 대부분의 사람들에게 너무 오래 전 이야기다). 그러니 이 예시는 사소한 고민은 아니라고 할 수 있다.
`{0, 0, 0}`는 대부분의 달력 체계에서 유효한 날짜가 아니다. 때문에 이 값을 사용하는 것은 부동 소수점에서 `NaN`을 사용하는 것과 같다.
하지만, 대부분의 현실적인 `Date` 클래스는 "첫째 날" (가령. 1970년 1월 1일이 많이 쓰인다)을 갖기 때문에 이것을 기본으로 사용하는 것이 일반적이다.

```c++
    class Date {
    public:
        Date(int dd, int mm, int yyyy);
        Date() = default; // C.45: 멤버를 초기화 하기만 하는 기본 생성자는 정의하지 마라
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

정적으로 할당된 내장 타입 개체들은 `0`으로 초기화 된다. 하지만 지역 변수들은 그렇지 않다.  
컴파일러의 최적화 빌드는 내장 타입 지역 변수들을 초기화하지 않을 수 있다는 점에 주의하라. 따라서, 위의 예시와 같은 코드가 나타난다면, 미정의 동작을 일으킬 수 있다.
초기화를 하고자 한다면, 명시적 기본 생성이 도움이 될 것이다:

```c++
    struct X {
        string s;
        int i {};   // 기본 초기화 (i는 0 이 된다)
    };
```

##### Notes

적절한 기본 생성을 가지지 않는 클래스들은 보통 복사 또한 불가하다. 때문에 이런 경우는 이 가이드라인에서 다루지 않는다.

예를 들어, 상위 클래스가 값 타입이 아니라면 (상위 클래스들은 복사되어선 안된다) 기본 생성자가 필수적이지 않다:

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

호출자가 제공하는 자원을 생성 과정에서 사용하는 경우라면 기본 생성자를 가질 수 없다. 이 경우도 가이드라인에 해당하지 않는데 보통 이런 클래스들은 복사가 불가능하기 때문이다:

```c++
    // std::lock_guard is not a copyable value type.
    // It does not have a default constructor.
    lock_guard g {mx};  // guard the mutex mx
    lock_guard g2;      // error: guarding nothing
```

다른 상태들과 다른 방식으로 처리되어야 하는 "특별한 상태"를 가지는 클래스들은 사용자가 더 많은 작업을 하게 만든다 (그 때문에 더 많은 오류를 만들기도 한다). 그런 타입은 (복사와 무관하게) 특별한 상태를 기본 생성 직후의 상태로 사용할 수도 있다.

```c++
    // std::ofstream is not a copyable value type.
    // It does happen to have a default constructor
    // that goes along with a special "not open" state.
    ofstream out {"Foobar"};
    // ...
    out << log(time, transaction);
```

복사 가능한 특별한 상태를 가진 타입의 예시로는, 복사 가능한 스마트 포인터를 들 수 있다. 여기서 기본 생성 상태(특별한 상태)는 `nullptr`를 들고있는 상태가 된다.

하지만, 기본 생성자를 지원하고 생성 후 유의미한 상태를 가지는 것이 권장된다. 이런 예로는 `std::string`이 `""`로 초기화되거나, `std::vector`가 공백상태(`{}`)를 가지는 것을 들 수 있다.

##### Enforcement

* 클래스가 기본 생성자 없이 복사 가능한 경우를 지적한다
* 클래스가 복사를 지원하지 않으면서 동등 비교(`==`)가 가능하면 지적한다

### <a name="Rc-default0"></a>C.44: 기본 생성자는 가능한 단순하고 예외를 던지지 않도록 하라

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

### <a name="Rc-explicit"></a>C.46: 단일 인자를 사용하는 생성자는 `explicit`으로 선언하라

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

생성 인자 타입으로부터의 묵시적인 변환을 허용한다면 `explicit`을 사용하지 않아야 한다:

```c++
    class Complex {
        // ...
    public:
        Complex(double d);   // OK: we want a conversion from d to {d, 0}
        // ...
    };

    Complex z = 10.7;   // 전혀 어색하지 않다
```

##### See also
[Discussion of implicit conversions](#Ro-conversion)

##### Note

복사와 이동 생성자는 변환(conversion)을 수행하지 않기 때문에 `explicit`이 되어야 한다. 명시적 복사/이동 생성자는 값으로 전달하거나 반환하는 것을 어렵게 한다.

##### Enforcement

(쉬움) 단일 인자를 사용하는 생성자는 `explicit`으로 선언하도록 한다. `explicit`이 아니면서 좋은 생성자는 드물다. 모범 사례에 포함되지 않는다면 경고하라.

### <a name="Rc-order"></a>C.47: 멤버 변수들은 선언된 순서대로 초기화하라

##### Reason

혼란과 오류를 최소화한다. 순서대로 진행하는 것이 초기화의 방식이다. (멤버 변수 초기화와는 무관하다).

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

(단순) 멤버 초기화 리스트는 선언과 같은 순서로 진행되어야 한다.

##### See also

[Discussion](./appendix/Discussion.md#Sd-order)

### <a name="Rc-in-class-initializer"></a>C.48: 상수 초기화는 가능한 클래스 내(in-class) 멤버 초기화를 사용하라

##### Reason

같은 변수가 사용될 것이라고 명시적으로 보여준다. 중복적으로 작성하지 않아도 된다. 유지보수 문제는 없앤다. 짧고 효율적인 코드가 된다.

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

코드를 유지보수하는 사람이 `j`가 의도적으로 초기화되지 않았다고 생각할 수 있다 (꽤 이상한 생각이지만). 또 어떤 의도로 `s`의 기본값으로 `""`와 `qqq`를 사용하는지 알 수 있을까? `j`와 같이 멤버 초기화가 생략되는 문제는 이미 있는 클래스에 새로운 멤버가 추가될 때 발생한다.

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

##### Alternative

생성자에 기본 인자를 주는 것을 생각해볼 수 있다. 오래된 코드에선 꽤 흔한 방법이다. 하지만, 이는 명시적이지 않고, 더 많은 인자가 전달되도록 만든다. 생성자가 여럿인 경우, 기본 인자 코드를 반복적으로 작성해야 한다:

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

* (쉬움) 모든 생성자는 모든 멤버 변수들을 초기화해야 한다 (명시적으로든, 생성자 위임이든, 기본 생성이든)
* (쉬움) 생성자의 기본인자는 클래스 내 초기화가 적합할 수 있다

### <a name="Rc-initialize"></a>C.49: 생성자 안에서의 대입 보다는 초기화를 선호하라

##### Reason

초기화는 대입 보다 깔끔하고 아름답게 수행될 수 있다. 값을 설정하기 전에 사용하는 오류("use before set")를 예방한다.

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

### <a name="Rc-factory"></a>C.50: 초기화 과정에서 `virtual` 동작이 필요하다면, 팩토리 함수를 사용하라

##### Reason

상위 클래스의 상태가 하위 개체에 의해 결정된다면, 불완전하게 생성된 개체를 사용할 가능성을 최소화 하면서 가상 함수를 사용해야 한다.

##### Note

팩토리 함수의 반환 탕비은 보통 `unique_ptr`가 적절하다; 만약 공유되어야 한다면, 함수를 호출한 쪽에서 `unique_ptr`를 `shared_ptr`로 이동시킬 수 있다. 다르게는, 팩토리 함수의 작성자가 반환 개체가 항상 공유된다는 것을 알고있다면, `make_shared`를 사용해서 `shared_ptr`를 반환하고 할당을 줄일 수 있다.

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

생성자를 `protected`로 만듦으로써, 개체가 불완전하게 생성된 후 사용자 코드에 노출되는 것을 막을 수 있다.
팩토리 함수 `Create()`를 지원하면 (자유 저장소 영역에) 개체 생성을 좀 더 쉽게할 수 있다.

##### Note

전통적인 팩토리 함수들은 스택이나 인접 개체보다는 자유 저장소에 생성한다.

##### See also

[Discussion](./appendix/Discussion.md#Sd-factory)

### <a name="Rc-delegating"></a>C.51: 클래스의 모든 생성자들을 위한 일반적인 동작을 표현할 때는 대리 생성자를 사용하라

##### Reason

코드 중복과 실수에 의한 코드 차이가 발생하지 않는다.

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

똑같은 동작을 작성하는 것은 지루하고 실수로 인해 똑같지 않을 수도 있다.

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

##### See also

만약 "반복된 동작"이 단순한 초기화라면, [클래스 내 멤버 초기화](#Rc-in-class-initializer)를 고려하라.

##### Enforcement

(중간) 유사한 생성자 구현(Body)을 찾는다

### <a name="Rc-inheriting"></a>C.52: 추가적인 초기화가 필요하지 않은 파생된 클래스에서 생성자를 사용할 때는 상속 생성자들을 사용하라

##### Reason

하위 클래스들을 위한 생성자가 필요하다면, 생성자를 다시 구현하도록 하는 것은 자루하고 오류의 소지가 많다.

##### Example

`std::vector`는 굉장히 많은 생성자를 지원한다. 때문에 자신만의 `vector`를 만들고자 한다면, 그 많은 생성자를 다시 작성하고 싶지는 않을 것이다:

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

하위 클래스의 모든 멤버들이 초기화되는지 검사한다

## <a name="SS-copy"></a>C.copy: 복사(Copy)와 이동(Move)

값 타입들은 일반적으로 복사 가능해야 한다. 하지만 클래스 계층에서의 인터페이스들은 그렇지 않아야 한다.  
리소스 핸들의 경우, 복사가 가능할 수도, 그렇지 않을 수도 있다.  
타입들은 논리적인 또는 성능 상의 이유로 이동하도록 정의될 수 있다.

### <a name="Rc-copy-assignment"></a>C.60: 복사연산을 `virtual`로 만들지 말아라. 매개변수는 `const&`로 받고, `const&`로 반환하지 말아라

##### Reason

이렇게 하는 것이 간단하고 효율적이다. r-value를 위해 최적화하길 원한다면, `&&`를 받는 대입 연산을 오버로드하여 제공하라. ([F.18](./Functions.md#Rf-consume)를 보라)

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

`swap`함수의 구현은 [강한 예외 안전성 보장](./Bibliography.md#Abrahams01)을 가능하게 한다.

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

대상 원소들에 직접 쓰기 연산을 함으로써, `swap`기법이 제공하는 강한 예외 보장 대신 [기본적인 예외 보장](./Bibliography.md#Abrahams01)만 얻게 될 것이다.  
[자기 대입](#Rc-copy-self)에 주의하라.

##### Alternatives

만약 `virtual` 대입 연산자가 필요하다고 생각한다면, 그리고 그것이 어째서 문제를 야기할 수 있는지 이해한다면, 그 함수는 `operator=`라고 부르지 마라. 이름을 부여해서 `virtual void assign(const Foo&)`로 만들어라.
[복사 생성 vs. `clone()`](#Rc-copy-virtual)를 참조하라.

##### Enforcement

* (쉬움) 대입 연산자는 가상함수여서는 안된다. 드래곤들만큼 위험하다!
* (쉬움) 대입 연산자는 `T&`를 반환하면 안된다. 연쇄적인 호출을 위해선, 컨테이너로의 개체 대입과 코드 작성을 방해하는 `const T&`를 사용하지 말아라.
* (중간) 대입 연산자는 (암시적으로나 명시적으로나) 모든 기본 클래스와 멤버들의 대입 연산자를 호출해야 한다. 해당 타입이 포인터 문맥이나 값 문맥을 가지는지 확인하기 위해 소멸자를 확인하라.

### <a name="Rc-copy-semantic"></a>C.61: 복사 연산은 복사를 수행해야 한다

##### Reason

일반적으로 그렇게 할 것이라 생각한다. `x = y`가 수행된 후에는, `x == y`인 결과를 가져야 한다.
복사 후에는 `x`와 `y`가 독립적인 개체들일 수 있다. (값 의미구조, 비-포인터 기본 타입들과 표준 라이브러리 타입들의 동작하는 방식) 또는 공유된 개체를 참조한다(포인터 의미구조, 포인터들이 동작하는 방식).

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

##### See also

[복사 대입을 위한 규칙들](#Rc-copy-assignment)

##### Enforcement

[복사 대입](#Rc-copy-assignment)에서와 동일하다.  

* (쉬움) 대입 연산자는 가상 함수여서는 안된다. 드래곤들만큼 위험하다!
* (쉬움) 대입 연산자는 `T&`를 반환하면 안된다. 연쇄적인 호출을 위해선, 컨테이너로의 개체 대입과 코드 작성을 방해하는 `const T&`를 사용하지 말아라
* (중간) 이동 연산자는 (암시적으로나 명시적으로나) 모든 기본 클래스와 멤버들의 이동 연산자를 호출해야 한다

### <a name="Rc-move-semantic"></a>C.64: 이동 연산은 이동을 수행해야 하며, 원본 개체를 유효한 상태로 남겨놓아야 한다

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

이상적으로는, 이동연산을 해준 개체는 해당 타입의 기본 값이어야 한다. 그렇지 않아야 하는 이유가 있지 않는한 기본 값을 가지도록 확실히 하라. 
하지만, 모든 타입들이 기본 값을 가지는 것은 아니며, 또 일부 타입들에서는 기본 값을 만드는 것이 비싼 비용을 필요로 할 수도 있다.
표준에서 요구하는 것은, 이동연산을 해준 개체가 파괴될 수 있다는 것 뿐이다.  
종종, 쉽고 비용이 들지 않는 방법을 쓸수도 있다: 표준 라이브러리는 개체로부터 이동을 받을 수 있다고 가정한다. 이동을 해주는 개체는 유효한 상태로 (필요하다면 명시하여) 남겨놓아라.  

##### Note

이 가이드라인을 적용하지 않아야 할 예외적인 이유가 있지 않는 한, `x = std::move(y); y = z;`를 사용하라. 전통적인 의미구조에 부합한다.

##### Enforcement

(자유선택) 이동 연산에서 멤버들의 대입을 확인해보라. 기본 생성자가 있다면, 그 대입 연산들을 기본 생성자를 사용한 초기화와 비교해보라.  

### <a name="Rc-move-self"></a>C.65: 이동 연산은 자기 대입에 안전하게 작성하라

##### Reason

만약 `x = x`가 `x`의 값을 바꾼다면, 사람들은 놀랄 것이고 안좋은 에러들이 발생할 수 있다. 사람들은 주로 자기 대입을 이동연산으로 작성하지 않지만, 그럴 수도 있다. 
예를 들어, `std::swap`은 이동 연산들로 구현되었고 만약 당신이 우연히  `a`와 `b`가 같은 개체를 참조하는 상황에서 `swap(a, b)`를 사용한다면, 자기-이동의 실패는 심각하거나 찾기 어려운(subtle) 에러가 될 수 있다.

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

아래는 검사 없이 포인터를 이동하는 방법이다(마치 이동 대입을 구현한 코드라고 상상해보라.):

```c++
    // move from other.ptr to this->ptr
    T* temp = other.ptr;
    other.ptr = nullptr;
    delete ptr;
    ptr = temp;
```

##### Enforcement

* (중간) 이러한 자기 대입의 경우, 이동 대입 연산자는 대입 받는 개체의 포인터 멤버를 `delete`된 상태 또는 `nullptr`로 남겨놓아서는 안된다.
* (자유선택) 표준 라이브러리 컨테이너들의 사용법을 보라(`string`을 포함한다). 그리고 일반적인(개체 수명에 민감하지 않은) 사용에 그 컨테이너들이 안전하다고 생각하라.

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

### <a name="Rc-copy-virtual"></a>C.67: 다형적인 클래스는 복사를 제한해야 한다

##### Reason

*다형적인 클래스*는 하나 이상의 가상 함수를 정의하거나 상속받은 클래스를 의미한다. 다른 하위 클래스들과 같이 상위 클래스를 통해서 사용될 것이다. 
만약 실수로 인해 묵시적으로 생성된 복사 생성자/대입 연산자와 함께 값으로 전달된 경우, 복사 절단(slicing)의 위험이 있다: 하위 개체에서 상위 개체에 해당하는 부분만 복사되고 다형성이 망가진다.

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

다형적인 개체에 깊은 복사를 사용해야 한다면, `clone()` 함수들을 사용하라: [C.130](#Rh-copy)를 참고하라.

##### Exception

예외 개체들을 위한 클래스는 다형적이면서 복사 생성이 가능해야 한다.

##### Enforcement

* 다형적인 클래스이면서 복사 연산을 지원하는 경우를 지적하라
* 다형적인 클래스의 개체가 복사되는 경우 지적하라

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

delete된 함수들이 public이라는 점에 주목하라

##### Enforcement

기본 연산을 제거하는 것은 해당 클래스에 부합하는 근거가 있어야 한다.
정말 이유가 있는지 의심하라.
하지만 사람이 보기에 문맥적으로 타당하다고 단언(assert)할 수 있도록 하라.

### <a name="Rc-ctor-virtual"></a>C.82: 생성자 또는 소멸자에서 가상 함수를 호출하지 말아라

##### Reason

호출된 함수는 파생 클래스에서 오버라이드 하는 함수가 아니라, 생성된 개체의 함수이다.
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

##### See also

정의되지 않은 동작의 위험이 없이 파생 클래스의 함수를 호출하는 효과를 얻기 위해서는 [팩토리 함수](#Rc-factory) 항목을 참고하라.

##### Note

가상 함수를 생성자와 소멸자에서 호출하는 행위가 반드시 잘못된 것은 아니다. 보통의 경우 이런 행위는 타입 안전한 의미구조를 가진다.
하지만, 경험적으로 그런 사용이 필요한 경우는 거의 발생하지 않으며, 유지보수 개발자를 혼란스럽게 한다. 초심자가 사용한다면 실수하는 원인이 될 수 있다.

##### Enforcement

* 생성자와 소멸자에서의 가상 함수 호출을 지적한다

### <a name="Rc-swap"></a>C.83: 값 타입들에는, `noexcept` swap함수를 제공하는 것을 고려하라

##### Reason

`swap`함수는 개체 대입을 구현할 때 원활하게 개체를 이동하는 것에서, 에러가 발생하지 않는 것을 보장하는 함수를 제공하는 것까지 몇몇 함수들(idioms)을 구현하는데 유용하다.
swap함수을 이용해서 복사 대입을 구현하는 것을 고려하라. [소멸자, 자원 해제, 그리고 swap은 실패해선 안된다](#Re-never-fail).

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

* (쉬움) 가상 함수들이 없는 클래스는 `swap`멤버 함수 선언이 있어야 한다
* (쉬움) 클래스가 `swap` 멤버함수를 가지고 있다면, 그 함수는 `noexcept`로 선언되어야 한다

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

해시 컨테이너들의 사용자들은 hash를 간접적으로 사용하며, 해시값을 위한 단순한 접근이 throw하지 않을 것으로 기대한다.  
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
* [C.101: 값 의미구조(value semantics)를 제공하라](#Rcon-val)
* [C.102: 이동 연산을 제공하라](#Rcon-move)
* [C.103: 초기화 리스트 생성자를 지원하라](#Rcon-init)
* [C.104: 기본 생성자는 빈 컨테이너를 만들도록 하라](#Rcon-empty)
* [C.109: 리소스 핸들이 포인터 의미구조를 따를 경우에는, `*` 과 `->` 연산자를 제공하라](#rcon-ptr)

##### See also

[R: 자원 관리](./Resource.md#S-resource)

### <a name="Rcon-stl"></a>C.100: 컨테이너를 정의할때는 STL을 따르라

##### Reason

C++ 프로그래머들에게 STL 컨테이너는 친숙하고 기본적으로 적절한(fundamentally sound) 설계를 가지고 있다.

##### Note

다른 fundamentally sound한 설계 방식들이 있고 표준 라이브러리의 방식을 사용하지 않을 이유가 있기도 하지만, 다른 방식을 택해야 할 확실한 이유가 없다면, 구현자와 사용자 모두 표준을 따르는 것이 단순하고 쉬운 방법이다.

특히, `std::vector`와 `std::map`은 단순하고 유용한 모델을 제공한다

##### Example

```c++
    // simplified (e.g., no allocators):

    template<typename T>
    class Sorted_vector {
        using value_type = T;
        // ... iterator types ...

        Sorted_vector() = default;
        Sorted_vector(initializer_list<T>);    // initializer-list constructor: sort and store
        Sorted_vector(const Sorted_vector&) = default;
        Sorted_vector(Sorted_vector&&) = default;
        Sorted_vector& operator=(const Sorted_vector&) = default;   // copy assignment
        Sorted_vector& operator=(Sorted_vector&&) = default;        // move assignment
        ~Sorted_vector() = default;

        Sorted_vector(const std::vector<T>& v);   // store and sort
        Sorted_vector(std::vector<T>&& v);        // sort and "steal representation"

        const T& operator[](int i) const { return rep[i]; }
        // no non-const direct access to preserve order

        void push_back(const T&);   // insert in the right place (not necessarily at back)
        void push_back(T&&);        // insert in the right place (not necessarily at back)

        // ... cbegin(), cend() ...
    private:
        std::vector<T> rep;  // use a std::vector to hold elements
    };

    template<typename T> bool operator==(const T&);
    template<typename T> bool operator!=(const T&);
    // ...
```

위 예시에선, 표준 템플릿 라이브러리 스타일을 불완전하게 따른다. 이런 경우는 흔히 볼 수 있다. 최소한의 기능만 제공하는 것은 특별히 구현된 컨테이너에게는 타당하다. 
핵심은 그 컨테이너에게 맞게 전통적인 생성자, 대입, 소멸, 그리고 반복자를 전통적인 의미구조를 지원하도록 정의하는 것이다. 
이를 바탕으로, 그 컨테이너는 필요한 만큼 확장될 수 있다. 이 예시에서는 `std::vector`를 사용하는 특별한 생성자가 추가되었다.

##### Enforcement

???

### <a name="Rcon-val"></a>C.101: 값 의미구조(value semantics)를 제공하라

##### Reason

정규 타입의 개체들은 고민없이 사용할 수 있다. 사용자들은 값 의미구조에 익숙하다.

##### Note

필요하다면, 컨테이너를 `Regular`(Concept 중 하나)하게 만들어라. 특히, 복사된 개체와 원래 개체를 비교했을 때 같도록 하라.

##### Example

```c++
    void f(const Sorted_vector<string>& v)
    {
        Sorted_vector<string> v2 {v};
        if (v != v2)
            cout << "insanity rules!\n";
        // ...
    }
```

##### Enforcement

???

### <a name="Rcon-move"></a>C.102: 이동 연산을 제공하라

##### Reason

컨테이너는 큰 규모로 사용되는 경향이 있다; 이동 연산이 없다면 비용이 많이 드는 복사가 사용될 것이고, 사람들이 포인터를 전달하도록 만듦으로써 자원 관리 문제를 일으킬 수 있다.

##### Example

```c++
    Sorted_vector<int> read_sorted(istream& is)
    {
        vector<int> v;
        cin >> v;   // assume we have a read operation for vectors
        Sorted_vector<int> sv = v;  // sorts
        return sv;
    }

    A user can reasonably assume that returning a standard-like container is cheap.
```

##### Enforcement

???

### <a name="Rcon-init"></a>C.103: 초기화 리스트 생성자를 지원하라

##### Reason

사람들은 값 집합을 사용해서 컨테이너 초기화가 가능할 것이라 기대한다. 자연스럽다.

##### Example

```c++
    Sorted_vector<int> sv {1, 3, -1, 7, 0, 0}; // Sorted_vector sorts elements as needed
```

##### Enforcement

???

### <a name="Rcon-empty"></a>C.104: 기본 생성자는 빈 컨테이너를 만들도록 하라

##### Reason

컨테이너를 정규(Regular) 타입으로 만들어준다.

##### Example

```c++
    vector<Sorted_sequence<string>> vs(100);    // 100 Sorted_sequences each with the value ""
```

##### Enforcement

???

### <a name="Rcon-ptr"></a>C.109: 리소스 핸들이 포인터 의미구조를 따를 경우에는, `*` 과 `->` 연산자를 제공하라

##### Reason

포인터에 기대되는 연산들이다. 포인터 사용자들에게는 이런 표현이 익숙하다.

##### Example

```
    ???
```

##### Enforcement

???

## <a name="SS-lambdas"></a>C.lambdas: 함수 개체와 람다 표현식(Function objects and lambdas)

함수 개체는 `()`를 오버로드해 호출을 지원하는 개체를 의미한다.
람다 표현식(줄여서 "람다"라고도 한다)은 함수 개체를 생성하도록 하는 표기를 의미한다.
함수 개체는 가능한 복사 비용을 발생시키지 않아야 한다 (또 그렇기에 [값에 의한 전달](./Functions.md#Rf-in)이 사용된다).

요약:

* [F.50: 함수를 사용할 수 없다면 람다를 사용하라 (지역 변수를 복사하거나, 지역적으로 사용되는 함수를 작성하는 경우)](./Functions.md#Rf-capture-vs-overload)
* [F.52: 지역적으로 사용되거나 알고리즘에 전달된다면 람다 캡쳐로는 참조를 선호하라](./Functions.md#Rf-reference-capture)
* [F.53: 반환되거나, 외부에 저장되거나, 다른 스레드로 전달하는 경우처럼 비 지역적으로 사용된다면 람다의 참조 캡쳐를 지양하라](./Functions.md#Rf-value-capture)
* [ES.28: 복잡한 초기화에는, 특히 상수 변수들의 초기화에는 람다를 사용하라](./Expr.md#Res-lambda-init)

## <a name="SS-hier"></a>C.hier: 클래스 계층 구조 (OOP)

클래스 계층구조는 계층적으로 조직화된 개념들의 집합을 표현하면서 (오직 그런 경우에만) 생성된다.
보통 상위 클래스(base class)들은 인터페이스로써 동작한다. 
계층구조를 사용하는 대표적인 예로는 2가지가 있는데, 구현 상속과 인터페이스 상속으로 불린다.

클래스 계층구조 규칙 요약:

* [C.120: 계층적인 구조를 가진 개념을 표현하기 위해서만 클래스 계층구조를 사용하라](#Rh-domain)
* [C.121: 상위 클래스가 인터페이스로 사용된다면, 순수 가상 클래스로 만들어라](#Rh-abstract)
* [C.122: 인터페이스와 구현의 분리가 필요하다면 추상 클래스들을 인터페이스로 사용하라](#Rh-separation)

계층 구조 내 클래스 설계 규칙 요약:

* [C.126: 일반적으로 추상 클래스는 생성자가 필요하지 않다](#Rh-abstract-ctor)
* [C.127: 가상함수를 가진 클래스는 공개(public)된 가상(virtual) 혹은 상속되는(protected) 소멸자를 가져야 한다](#Rh-dtor)
* [C.128: 가상 함수들은 `virtual`, `override`, 혹은 `final` 중 하나만 명시해야 한다](#Rh-override)
* [C.129: 클래스 계층구조를 정의할 때는 구현 상속과 인터페이스 상속을 구분하라](#Rh-kind)
* [C.130: 다형적인 클래스에서 깊은 복사를 지원하게 하려면 복사 생성/대입 보다는 가상 `clone`을 선호하라](#Rh-copy)
* [C.131: 자잘한 getter와 setter를 사용하지 말아라](#Rh-get)
* [C.132: 이유없이 함수를 `virtual`로 만들지 말아라](#Rh-virtual)
* [C.133: `protected` 데이터를 지양하라](#Rh-protected)
* [C.134: `const`가 아닌 모든 데이터 멤버들이 같은 접근 레벨을 가지도록 하라](#Rh-public)
* [C.135: 서로 다른 인터페이스를 표현하기 위해 다중 상속을 사용하라](#Rh-mi-interface)
* [C.136: 구현 특성(attribute)의 결합을 표현하기 위해 다중 상속을 사용하라](#Rh-mi-implementation)
* [C.137: 지나치게 일반적인 상위 클래스를 피하기 위해 `virtual`을 사용하라](#Rh-vbase)
* [C.138: `using`을 사용해 상위/하위 클래스를 위한 중복 정의 집합을 만들어라](#Rh-using)
* [C.139: `final`은 필요한 만큼만 사용하라](#Rh-final)
* [C.140: 가상 함수와 그 구현 함수에 서로 다른 기본 인자를 사용하지 마라](#Rh-virtual-default-arg)

계층 구조 내 개체 접근 규칙 요약:

* [C.145: 다형적인 개체들은 포인터와 참조를 통해 접근하라](#Rh-poly)
* [C.146: 클래스 계층구조 탐색이 불가피한 경우에만 `dynamic_cast`를 사용하라](#Rh-dynamic_cast)
* [C.147: 필요한 클래스를 찾지 못한 경우가 오류에 해당하는 경우 `dynamic_cast`를 참조 타입에 사용하라](#Rh-ref-cast)
* [C.148: 필요한 클래스를 찾지 못한 경우가 대안으로 사용된다면 `dynamic_cast`를 포인터 타입에 사용하라](#Rh-ptr-cast)
* [C.149: 동적 할당한 개체의 소멸을 잊지 않도록 `unique_ptr` 혹은 `shared_ptr`를 사용하라](#Rh-smart)
* [C.150: `unique_ptr`로 소유되는 개체를 생성하기 위해서는 `make_unique()`를 사용하라](#Rh-make_unique)
* [C.151: `shared_ptr`로 소유되는 개체를 생성하기 위해서는 `make_shared()`를 사용하라](#Rh-make_shared)
* [C.152: 절대로 하위 클래스의 포인터에 상위 클래스 포인터를 대입하지 마라](#Rh-array)
* [C.153: 타입 캐스팅보다 가상 함수를 선호하라](#Rh-use-virtual)

### <a name="Rh-domain"></a>C.120: 계층적인 구조를 가진 개념을 표현하기 위해서만 클래스 계층구조를 사용하라

##### Reason

생각이 바로 코드로 나타나는 것은 이해와 유지보수를 쉽게 만든다. 생각이 상위 클래스에서 나타나고 하위 타입에서 이를 따르게 하라. 이 목적을 담아내기 위한 방법으로 상속보다 좋은 방법은 없다.

데이터 멤버를 담기 위한 방법으로 상속을 *사용해선 안된다*. 많은 경우 상속은 하위 타입이 상위 가상 함수를 재정의하거나 멤버에 접근할 필요한 경우를 의미한다.

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

계층적이지 않은 개념을 클래스 계층구조로 표현해선 *안된다*.

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

위 예에서 대부분의 하위 클래스들은 인터페이스에서 요구하는 함수들을 잘 재정의(override)할 수 없다. 때문에 상위 클래스는 구현 부담을 발생시킨다.
나아가서, `Container`의 사용자는 멤버 함수들이 실제로 유의미한 연산들을 효율적으로 실행하는지 신뢰할 수 없다: 연산을 수행하지 않고 예외를 던질 수도 있다.

이 때문에 유저는 실행 시간 검사에 의존하거나 이와 같은 (과도하게) 일반적인 인터페이스를 사용하지 않고 (`dynamic_cast`와 같은) 실행시간 타입 확인을 사용할 것이다.

##### Enforcement

* 아무것도 하지 않고 예외를 던지는 멤버가 많은 클래스를 찾
아낸다
* 상위 클래스 `B`의 가상함수를 하위 클래스 `D`가 구현하지 않거나 `B` 멤버에 접근하지 않는 경우를 지적하라. 이때 `B`는 다음에 해당하지 않아야 한다: 멤버 변수를 가지지 않는다. `D`의 템플릿 인자 혹은 인자 묶음(pack)이 아니다. `D`를 사용해 특수화된 템플릿이다.

### <a name="Rh-abstract"></a>C.121: 상위 클래스가 인터페이스로 사용된다면, 순수 가상 클래스로 만들어라

##### Reason

클래스는 데이터를 가지지 않으면 더 안정적이다. 인터페이스는 일반적으로 순수 가상 함수와 기본/비어있는 가상 소멸자로 구성되어야 한다

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

`Derived` 클래스는 `Goof`를 통해서 소멸되기 때문에, 멤버 `string`에서 누수가 발생한다. `Goof`에서 가상 소멸자를 제공하면 모든게 원활하게 돌아간다.

##### Enforcement

* 클래스가 데이터 멤버를 가지면서 (`final`이 아닌) 가상 함수를 가지면 경고한다

### <a name="Rh-separation"></a>C.122: 인터페이스와 구현의 분리가 필요하다면 추상 클래스들을 인터페이스로 사용하라

##### Reason

예를 들어 ABI(Application Binary Interfae)를 사용하는 지점에서 이런 작업이 필요하다.

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

위와 같은 코드에서 사용자는 `Device`에서 제공하는 인터페이스를 통해서 `D1`과 `D2`를 교환하면서 사용할 수 있다. 나아가서, `Device`를 통해서 접근되는 한 `D1`과 `D2`를 구 버전과 호환되지 않도록(not binary compatible) 업데이트 할 수 있다.

##### Enforcement

???

## C.hierclass: 계층 구조 내 클래스 설계

### <a name="Rh-abstract-ctor"></a>C.126: 일반적으로 추상 클래스는 생성자가 필요하지 않다

##### Reason

추상 클래스는 데이터 멤버를 가지지 않으며 이를 초기화하기 위한 생성자 또한 가지지 않는다.

##### Example

```
    ???
```

##### Exception

* 개체를 어딘가에 등록하기 위한 상위 클래스 생성자가 필요할 수도 있다
* 극히 드문 경우이지만, 추상 클래스가 하위 클래스들이 공유하는 데이터를 가지는 것이 타당한 경우가 있을 수 있다 (예를 들어, 정적 데이터, 디버깅 정보 등); 그러한 클래스들은 생성자를 가진다. 하지만 주의하라; 그런 경우는 가상 상속에 취약하다

##### Enforcement

생성자를 가진 추상 클래스들을 지적하라

### <a name="Rh-dtor"></a>C.127: 가상함수를 가진 클래스는 공개(public)된 가상(virtual) 혹은 상속되는(protected) 소멸자를 가져야 한다

##### Reason

가상 함수를 가진 클래스들은 보통 상위 클래스의 포인터를 통해서 사용된다. 많은 경우, 마지막 사용자가 상위 클래스 포인터를 통해 delete 하거나 스마트 포인터를 사용해 소멸시킨다. 때문에 소멸자는 public 범위에 있으면서 가상 함수여야 한다. 
드물게는, 상위 클래스 포인터를 사용한 소멸을 의도적으로 지원하지 않는다면, 소멸자는 protected 범위에 있으면서 가상 함수가 아니어야 한다; [C.35](#Rc-dtor-virtual)를 참고하라

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

`shared_ptr`를 통해 클래스를 사용하기 때문에 이 규칙을 따르지 않는 사람들도 있다: `std::shared_ptr<B> p = std::make_shared<D>(args);` 이런 경우, 공유 포인터가 소멸을 담당할 것이다. 그러니 부적절한 상위 클래스의 `delete`로 인한 누수가 발생하지 않는다. 
이를 계속하는 사람들은 잘못된 코드로부터 정상적인 동작을 만들어낸다 (false positive), 그렇지만 이 규칙은 중요하다 -- 만약 누군가 `make_unique`를 사용해 할당하면 어떻게 될 것인가? `B`의 작성자가 모든 생성자를 private로 만들고 `make_shared`를 통해서만 생성이 가능하도록 팩토리 함수를 제공해서 잘못 사용될 것이라고 보장하지 않는 한, 이 코드는 안전하지 않다. 

##### Enforcement

* 가상 함수를 가진 클래스는 공개(public)된 가상(virtual) 혹은 상속되는(protected) 소멸자를 가져야 한다
* 가상 소멸자를 가지지 않고 가상 함수를 사용해 `delete`하는 클래스를 지적하라

### <a name="Rh-override"></a>C.128: 가상 함수들은 `virtual`, `override`, 혹은 `final` 중 하나만 명시해야 한다

##### Reason

가독성. 실수를 발견할 수 있다. 명시적으로 `virtual`, `override`, `final`을 사용하는 것은 함수 자체를 문서화한다. 동시에 컴파일러가 상위 클래스와 하위 클래스의 타입 혹은 이름이 불일치 하는 것을 잡아낼 수 있도록 돕는다.
하지만 이들을 하나 이상 작성하는 것은 중복적이면서 오류를 발생시킬 수 있다.

하나만 작성하는 것이 단순하고 명확하다:

* `virtual`는 "새로운 가상 함수"라는 것을 의미한다
* `override`는 "재정의 될 수 있는(non-`final`) 재정의 함수"라는 것을 의미한다
* `final` 는 "마지막 재정의 함수"라는 것을 의미한다

만약 상위 클래스의 소멸자가 `virtual`로 선언되었다면, 하위 클래스 소멸자들은 `virtual` 혹은 `override`가 된다. 어떤 코드 혹은 지원도구에서 소멸자에 `override`를 사용하도록 강요할 수도 있지만, 이 가이드라인에서는 그 방법은 권하지 않는다.

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

우리는 이 규칙을 통해 2가지 오류를 없애고자 한다:

* **암묵적 가상함수**
   * 프로그래머가 암묵적 가상 함수를 의도했으며, 실제로 그에 해당하는 경우  
     (하지만 코드를 읽는 사람은 알아볼 수 없다)
   * 프로그래머는 암묵적 가상 함수를 의도했으나 그렇지 않은 경우  
     (예를 들어 인자가 미묘하게 맞지 않았다거나하는 이유로)
   * 프로그래머가 가상 함수를 의도하지 않았으나 가상 함수가 된 경우  
     (상위 클래스의 가상 함수와 같은 시그니처를 가지는 바람에)     
* **암묵적 재정의**
   * 프로그래머는 함수가 암묵적으로 재정의되는 것을 의도했고 그렇게 된 경우  
     (하지만 코드를 읽는 사람은 알아볼 수 없다)
   * 프로그래머는 함수가 암묵적으로 재정의되는 것을 의도했으나 그렇지 않은 경우
     (예를 들어 인자가 미묘하게 맞지 않았다거나하는 이유로)
   * 프로그래머가 함수가 재정의 되는 것을 의도하지 않았으나 재정의 된 경우  
     (상위 클래스의 가상 함수와 같은 시그니처를 가지는 바람에 -- 이런 일은 그 함수가 virtual로 선언되지 않아도 발생한다는 점에 주의하라, 프로그래머가 새로운 가상 함수를 만들기를 원했는지 비 가상 함수를 원했는지 알 방법이 없기 때문이다)

##### Enforcement

* 상위 클래스와 하위 클래스들의 이름을 비교하고 같은 이름을 쓰면서 재정의하지 않는 경우를 지적하라
* `override`와 `final` 중 어느 하나도 사용하지 않은 재정의를 지적하라
* `virtual`, `override`, `final`중 2개 이상을 사용한 함수 선언을 지적하라

### <a name="Rh-kind"></a>C.129: 클래스 계층구조를 정의할 때는 구현 상속과 인터페이스 상속을 구분하라

##### Reason

인터페이스에서 구현 내용을 가지는 것은 인터페이스가 망가지기 쉽게 한다; 달리 말해, 인터페이스의 사용자들이 구현을 바꾼 뒤에 다시 컴파일할 때 영향을 받게 한다.
상위 클래스의 데이터는 상위 클래스를 구현하는 것을 어렵게 만들고 코드 중복을 발생시킬 수 있다.

##### Note

정의:

* 인터페이스 상속은 사용자 코드를 구현과 분리하기 위한 것이다. 하위 클래스에서 상위 클래스를 사용하는 코드에 영향을 미치지 않으면서 코드를 더하거나 변경하는데 사용된다
* 구현 상속은 상속을 사용해 새로운 구현내용을 하위 구현체들이 사용할 수 있도록하는 것이다 (보통 "programming by difference"라고 불린다).

순수한 인터페이스 클래스는 쉽게말해 순수 가상함수들의 집합이라고 할 수 있다; [I.25](./Interfaces.md#Ri-abstract)를 참고하라.

초창기의 개체지향 프로그래밍에서는 (1980년도와 1990년도), 구현 상속과 인터페이스 상속이 혼재되어 있었고 그런 인습이 아직도 남아있다. 오래된 코드나 교육자료에서는 흔히 볼 수 있다.

아래의 경우 2가지 상속을 구분하는 것이 중요하다

* 계층구조의 크기가 커진다(십수개의 하위 클래스가 존재한다)
* 계층구조를 사용하는 시간이 길어진다 (수십년)
* 서로 다른 조직이 계층구조를 사용하고 있다 (즉, 흩어진 상위 클래스를 업데이트 하기 어렵다)

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

위 예시는 아래와 같은 문제들을 가지고 있다:

* 계층구조가 늘고 `Shape`의 데이터가 늘어난다. 생성자를 작성하고 관리하기 어려워진다
* `Triangle`의 중심을 사용하지 않을지도 모르는데 계산할 필요가 있을까?
* `Shape`에 새로운 멤버를 추가되면 (예컨대 그리는 방법이라던가 그리는 캔버스), 모든 하위 클래스들이 변화되면서 새로 컴파일 되어야 한다

`Shape::move()`가 구현 상속의 한 사례이다: 상위 클래스와 모든 하위 클래스를 위해서 `move()`를 한번만 정의한다.
상위 클래스에 더 많은 멤버함수 코드가 작성될수록,더 많은 데이터가 공유될수록, 코드를 적게 작성하는 효용이 생기지만 계층구조가 불안정하게 된다.

##### Example

인터페이스 상속을 사용해 `Shape` 계층을 다시 작성하면 이렇다:

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

순수 인터페이스들이 생성자를 가지는 경우는 드물다는 점에 주의하라: 생성할 데이터가 존재하지 않는다.

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

인터페이스는 이제 좀 더 견고해졌지만, 멤버 함수의 구현을 위해 더 많은 작업을 해야 한다. 예를 들어, `center`는 모든 하위 클래스에서 제각기 구현해야 한다.

##### Example, dual hierarchy

어떻게 하면 인터페이스 상속에 의한 안정적인 계층구조와 구현 상속의 효율적인 재사용을 결합할 수 있을까?
관련해서 유명한 방식 중 하나는 이중 계층(dual hierarchies) 방식이다. 
여러 방식들이 있지만, 여기서는 다중 상속 방법을 사용한다.

첫번째로 인터페이스 상속을 만든다:

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

이 인터페이스를 유용하게 만드려면, 구현 클래스들가 필요하다. (여기서는 `Impl` 네임스페이스에서 클래스를 하나 더 정의한다):

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

이제 `Shape`는 구현을 가진 클래스의 지저분한 예시가 되었지만, 좀 더 복잡한 계층구조를 위한 단순한 예시이기 때문에 참아줄 수 있다.

```c++
    class Impl::Circle : public Circle, public Impl::Shape {   // implementation
    public:
        // constructors, destructor

        int radius() override { /* ... */ }
        // ...
    };
```

여기서 `Smiley`클래스를 더해 계층구조를 확장해보자:

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

여기에는 두 계층구조가 혼합되어 있다:

* 인터페이스: Smiley -> Circle -> Shape
* 구현: Impl::Smiley -> Impl::Circle -> Impl::Shape

인터페이스와 구현 양쪽에서 상속받기 때문에 격자 구조(유향 비순환 그래프)를 가지게 된다:

```
    Smiley     ->         Circle     ->  Shape
      ^                     ^               ^
      |                     |               |
    Impl::Smiley -> Impl::Circle -> Impl::Shape
```

앞서 언급한 것처럼, 이는 이중 계층을 구현하기 위한 한 방법일 뿐이다.

추상 인터페이스가 아니라 구현 계층을 통해서 바로 사용될수도 있다.

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

추상 인터페이스에서 지원하지 않는 멤버를 구현 클래스에서 지원하는 경우 유용할 수 있다. 
또 멤버를 직접 사용함으로써 최적화의 가능성도 제공한다 (가령, 구현 멤버함수가 `final`로 선언되었다면)

##### Note

인터페이스와 구현을 분리하기 위한 또 다른 (관련된) 방법은 [Pimpl](./Interfaces.md#Ri-pimpl)이다.

##### Note

공통적인 기능들은 (이미 구현된) 상위 클래스 함수로 제공하고 구현 namespace에서 자유롭게 선택하도록 할수도 있다.
상위 클래스는 더 짧은 표기를 할 수 있게 만들어주며, 기능적인 측면에서(at the cost of the functionality) 계층구조가 공유하는 데이터에 접근하는 유일한 존재가 될 수 있다. 유일한 접근자가 접근하기가 쉽다.

##### Enforcement

* 데이터와 가상함수에 대해 하위 타입에서 상위 타입으로의 변환을 지적하라  
  (상위 클래스 멤버 함수를 호출하는 것을 제외하고)
* ???

### <a name="Rh-copy"></a>C.130: 다형적인 클래스에서 깊은 복사를 지원하게 하려면 복사 생성/대입 보다는 가상 `clone`을 선호하라

##### Reason

다형적인 클래스를 복사하는 것은 절단 문제 때문에 권할만한 일이 아니다. [C.67](#Rc-copy-virtual)를 보라. 복사 문맥이 정말 필요하다면, 깊은 복사를 수행하라: 가상 `clone` 함수를 제공해서 실제 하위 타입을 복사하고 새로운 개체를 소유하는 포인터를 반환하라. 그리고 하위 클래스에서는 하위 클래스의 타입을 반환하라 (공변적인 반환 타입을 사용하라)

> 공변성: covariance

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

보편적인 경우, 소유권을 표현하기 위해 스마트 포인터를 사용하는 것이 권장된다.([R.20](./Resource.md#Rr-owner) 참고). 하지만, 언어 규칙으로 인해, 공변적인 반환타입은 스마트 포인터가 될 수 없다: `D::clone`은 `unique_ptr<D>`을 반환할 수 없는 반면 `B::clone`는 `unique_ptr<B>`를 반환할 수 있다. 이로 인해, 모든 재정의에서 항상 `unique_ptr<B>` 혹은 [Guidelines Support Library](./GSL.md#SS-views)의 `owner<>`를 반환할 수 밖에 없다.

### <a name="Rh-get"></a>C.131: 자잘한 getter와 setter를 사용하지 말아라

##### Reason

사소한 목적으로 작성된 getter와 setter는 의미구조적 가치가 없다; 단순히 `public`으로 공개해도 될 것이다.

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

이런 클래스를 `struct`로 만드는 것을 고려하라 -- 즉, 어떤 행위도 하지 않는 변수들을 public 데이터로 만들고 멤버함수를 가지지 않는 것이다.

```c++
    struct Point {
        int x {0};
        int y {0};
    };
```

멤버 변수들에 기본 초기화를 사용할 수 있다는 점에 유의하라: [C.49: 생성자 안에서의 대입 보다는 초기화를 선호하라](#Rc-initialize).

##### Note

이 규칙의 핵심은 getter/setter의 의미구조가 가치있는지 판단하는 것이다. 이것이 "사소함"에 대한 완전한 정의는 아니지만, 문법을 넘어서 getter/setter가 public 데이터 멤버였을 때를 고려해보라. 사소하지 않은 의미구조의 예를 든다면: 클래스의 불변조건을 유지하거나 내부(internal) 타입과 인터페이스 타입을 변환하는 것을 예로 들 수 있다.

##### Enforcement

별다른 의미구조 없이 단순히 멤버에 접근하기만 하는 `get`/`set` 멤버 함수를 여럿 가지고 있으면 지적한다.

### <a name="Rh-virtual"></a>C.132: 이유없이 함수를 `virtual`로 만들지 말아라

##### Reason

중첩된 `virtual`은 실행 시간과 개체의 코드 크기를 증가시킨다.
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

### <a name="Rh-protected"></a>C.133: `protected` 데이터를 지양하라

##### Reason

`protected` 데이터는 복잡성과 에러의 원인이다.  
`protected` 데이터는 불변조건의 구문을 복잡하게 만든다.  
`protected` 데이터는 상위 클래스에 데이터를 배치함으로써 필연적으로 가상 상속을 처리해야 하는 상황으로 이어질 수 있다.

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

이 예에서 모든 `Shape`의 하위 타입들은 protected 데이터를 정확하게 변경해야만 한다. 흔히 볼수 있으면서 유지보수 문제를 일으키는 주요 원인 중 하나에 해당한다. 클래스 계층구조가 큰 경우, 일관적으로 protected 데이터를 사용하는 것은 코드가 양적으로 많고 분산되어 있기 때문에 관리되기 어렵다.
상속되는 데이터를 변경할 수 있는 클래스는 더 늘어날 수 있다: 새로 클래스를 상속받아 protected 데이터를 변경하기 시작할 수 있다.
경우에 따라선 클래스들의 전체 집합을 찾는 것이 불가능할수도 있다. 이로 인해 클래스를 변경하는 것을 실행할 수 없을 수도 있다. protected 데이터에는 불변조건을 강요할 수 없다; 전역변수 집합과 같다고 할 수 있다. protected 데이터는 코드 규모가 커지면 실제로 전역변수가 된다.

##### Note

데이터를 protected를 사용해 상속하는 것은 임의적으로 개선하도록 할 수 있게 한다는 점에서 매력적으로 보일 수 있다. 하지만 이로 인해 제어되지 않는 변경과 오류를 발생시키게 된다.
잘 정의되고 불변조건을 강요하도록 [`private` 데이터를 선호하라](#Rc-private)
다른 방법으로는, [인터페이스 클래스는 데이터를 가지지 않도록 하라](#Rh-abstract).

##### Note

protected 멤버 함수에는 문제가 없다.

##### Enforcement

`protected` 데이터를 지적하라

### <a name="Rh-public"></a>C.134: `const`가 아닌 모든 데이터 멤버들이 같은 접근 레벨을 가지도록 하라

##### Reason

생각하기에 혼란스럽지 않아 오류를 예방한다.
`const` 멤버들이 서로 다른 접근 레벨을 가지고 있다면, 그 타입이 어떤 일을 하는 것인지 혼란스러울 것이다.
타입이 불변조건을 유지하기 위한 것인가? 혹은 단순히 값의 집합을 표현한 것인가?

##### Discussion

하나의 변수에 대해 어떤 코드가 유의미하고 정확한 값을 관리하는 책임이 있는지 고민해야 한다.

데이터 멤버에는 2가지 종류가 있다:

* A: 개체의 불변조건과 무관한 경우. 이 멤버들이 어떤 값(혹은 값의 조합)을 가져도 유효하다
* B: 개체의 불변조건으로 기능하는 경우, 모든 값의 조합이 의미를 가지지는 않는다. 따라서 이 변수들의 값을 변경하는 모든 코드는 불변조건을 알고, 유지하기 위한 규칙들을 고려해야 한다.

A 그룹에 속하는 데이터 멤버들은 단순히 `public` (혹은, 드물지만 하위 클래스에서 볼 수 있도록 `protected`)이면 된다. 캡슐화가 필요하지 않다. 멤버를 볼 수 있는 코드는 이들을 변경할 수 있다.

B 그룹에 속하는 데이터 멤버들은 `private`혹은 `const`여야 한다. 이 경우에는 캡슐화가 필요하기 때문이다. 이들이 `private`이나 `const`가 아니라는 것은 개체가 자신의 상태를 관리하지 않는다는 의미가 된다: 클래스의 다른 코드들이 불변조건을 알고 정확하게 유지해야 한다. 그리고 그런 코드가 제한없이 늘어날 수 있다. 이 변수들이 `public`이 되면 모든 사용 코드가 불변조건을 고려해야 하며, `protected`이라면 (현재와 미래의) 모든 하위 클래스들이 포함된다.
이는 망가지기 쉽고 강하게 결합된 코드를 만들게 된다. 유지보수가 악몽과 같을 것이다. 의도치 않게 어떤 코드가 데이터 멤버를 잘못된(invalid) 값으로 만들면 개체의 현재 상태와 이후의 사용에 영향을 줄 것이다.

대부분의 클래스들은 A와 B로 구분된다:

* *All public*: 변수들의 불변조건이 없는 집합을 만든다면 모든 변수가 `public`이 되어야 한다. [이런 경우는 `class` 보다는 `struct`로 선언하라](#Rc-struct)
* *All private*: 불변조건이 있다면, 모든 `const`가 아닌 변수들은 `private`이 되어야 한다 -- 캡슐화 하라

##### Exception

경우에 따라선 클래스들이 디버깅을 위해 A와 B를 혼합할 수도 있다. 캡슐화된 개체가 `const`가 아닌 디버깅 정보를 포할 수도 있다. 이는 불변조건에 포함되지 않고 -- 개체가 관리하는 값의 일부가 아니고 관찰되어야 하는 부분이 아니기 때문에 A에 속한다. 이때는 A에 해당하는 영역 (`public` 혹은 `protected`)은 A 처럼, 나머지 영역은(`private` or `const`) B 그룹 처럼 관리하면 된다.

##### Enforcement

`const`가 아닌 데이터 멤버들이 서로 다른 접근레벨을 가진 클래스를 지적한다.

### <a name="Rh-mi-interface"></a>C.135: 서로 다른 인터페이스를 표현하기 위해 다중 상속을 사용하라

##### Reason

모든 클래스들이 모든 인터페이스들을 지원하지는 않을 것이다. 그리고 모든 호출자(caller)들이 모든 연산들을 사용하길 원하지도 않을 것이다. (다중 상속은) 특별히 단일한(monolitic) 인터페이스들을 파생 클래스가 지원하는 동작의 "측면"들로 나눌때 사용하라.

##### Example

```c++
    class iostream : public istream, public ostream {   // 굉장히 단순하다
        // ...
    };
```

`istream`은 입력 연산에 필요한 인터페이스를 제공한다. `ostream`은 출력 연산에 필요한 인터페이스를 제공한다.
`iostream`은 `istream`과 `ostream` 인터페이스를 결합하면서 단일 스트림에서의 입출력 동기화를 제공한다.

##### Note

하나의 구현에 대해 여러 다른 인터페이스가 필요한 경우 쉽게 볼 수 있다. 그런 인터페이스들은 하나의 계층구조로 조직화하기 쉽지 않다.

##### Note

이런 인터페이스들은 보통 추상 클래스들이다.

##### Enforcement

???

### <a name="Rh-mi-implementation"></a>C.136: 구현 특성(attribute)의 결합을 표현하기 위해 다중 상속을 사용하라

##### Reason
 
Some forms of mixins have state and often operations on that state.
If the operations are virtual the use of inheritance is necessary, if not using inheritance can avoid boilerplate and forwarding.

##### Example

```c++
    class iostream : public istream, public ostream {   // very simplified
        // ...
    };
```

`istream`은 입력 연산에 필요한 인터페이스를 제공한다. `ostream`은 출력 연산에 필요한 인터페이스를 제공한다.
`iostream`은 `istream`과 `ostream` 인터페이스를 결합하면서 단일 스트림에서의 입출력 동기화를 제공한다.

##### Note

이것은 상대적으로 드문 경우인데, 구현은 종종 단일루트(single-root) 계층으로 조직화될 수 있기 때문이다.

##### Example

경우에 따라서는 "구현 특성(implementation attribute)"이 구현체의 행위를 결정하고 구현체가 요구받는 정책을 따르도록 멤버를 주입하는 "혼합(mixin)" 처럼 보인다.
`std::enable_shared_from_this` 혹은 boost.intrusive의 `list_base_hook` 혹은 `intrusive_ref_counter`를 예로 들 수 있다.

##### Enforcement

???

### <a name="Rh-vbase"></a>C.137: 지나치게 일반적인 상위 클래스를 피하기 위해 `virtual`을 사용하라

##### Reason

공유 데이터와 인터페이스의 분리가 가능하게 만든다. 공유 데이터가 최상위 클래스에 배치되는 것을 막는다.

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

`Utility`를 추출하는 것은 수많은 하위 클래스들이 "구현 내용(implementation details)"의 많은 부분을 공유한다면 이치에 맞는 작업이다.

##### Note

이 예시는 명백히 너무 "이론적"이다. 하지만 *작은 규모*의 실제적인 예시를 찾기는 어렵다.
`Interface`는 [인터페이스 계층](#Rh-abstract)의 정점이고, `Utility`는 [구현 계층](#Rh-kind)의 정점이다.
설명을 포함한 [좀 더 사실적인 예시는 여기에 있다](https://www.quora.com/What-are-the-uses-and-advantages-of-virtual-base-class-in-C%2B%2B/answer/Lance-Diduck).

##### Note

때로는 계층을 선형화(linearization)하는 것이 나은 방법일 수 있다

##### Enforcement

인터페이스 계층과 구현 계층이 혼재된 경우 지적한다.

### <a name="Rh-using"></a>C.138: `using`을 사용해 상위/하위 클래스를 위한 중복 정의 집합을 만들어라

##### Reason

using 선언이 없으면, 하위 클래스의 멤버 함수들이 상속받은 중복정의 집합을 덮어쓴다(hide).

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

이 이슈는 가상/비 가상 멤버 함수 모두에 적용된다.

상위 클래스를 가변적으로 결정할 수 있도록, C++17은 가변 using 선언을 추가했다(introduced).

```c++
    template <class... Ts>
    struct Overloader : Ts... {
        using Ts::operator()...; // exposes operator() from every base
    };
```

##### Enforcement

이름을 덮어쓰는 경우를 찾아낸다

### <a name="Rh-final"></a>C.139: `final`은 필요한 만큼만 사용하라

##### Reason

`final`은 계층구조의 확장성을 저해한다는 점 때문에 필요한 경우가 거의 없다.

##### Example, bad

```c++
    class Widget { /* ... */ };

    // nobody will ever want to improve My_widget (or so you thought)
    class My_widget final : public Widget { /* ... */ };

    class My_improved_widget : public My_widget { /* ... */ };  // error: can't do that
```

##### Note

모든 클래스가 상위 클래스로써 작성되지는 않는다. 대부분의 표준 라이브러리 클래스들이 이런 경우에 속한다 (예컨대, `std::vector`와 `std::string`는 하위 클래스를 고려하지 않는다). 
이 규칙은 클래스 계층구조 내에서 인터페이스로 동작하는 가상 함수들을 가진 클래스들에 `final`을 사용하는 경우에 대한 것이다.

##### Note

가상 함수들을 `final`로 마감하는 것은 `final`이 함수들의 정의/재정의를 찾아내지 못하도록 하기 때문에 오류를 발생시킬 수 있다. 
운좋게도, 컴파일러가 이런 실수를 찾아낸다: 하위 클래스의 `final`을 다시 선언하거나 재정의할 수 없다.

##### Note

`final`로 성능이 개선될 것이라는 주장에는 근거가 없다. 대부분 그런 주장은 추측이거나 다른 언어에서의 경험에 근거한 것이다.

`final`이 논리적인 이유와 성능 측면에서 중요한 예시가 있다. 

 * 컴파일러나 언어 분석 도구에서 사용하는 (성능기준이 높은) AST 계층
 * 시간이 지나도 새로운 하위 클래스가 추가되지 않고 라이브러리 구현자에 의해서만 추가된다

하지만 잘못 사용하는 경우가 훨씬 더 많다.

##### Enforcement

`final`을 사용하면 지적한다

### <a name="Rh-virtual-default-arg"></a>C.140: 가상 함수와 그 구현 함수에 서로 다른 기본 인자를 사용하지 마라

##### Reason

혼란을 일으킨다: 재정의한 코드가 기본 인자를 상속받지 않는다.

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

가상함수의 기본인자가 상위/하위 클래스의 선언에서 서로 다르면 지적한다

## C.hier-access: 계층 구조 내 개체 접근

### <a name="Rh-poly"></a>C.145: 다형적인 개체들은 포인터와 참조를 통해 접근하라

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

개체가 정의된 범위 안에서는 이름이 있는 다형적 개체에 안전하게 접근할 수 있다. 단지 복사 손실이 생기지 않도록 하라.

```c++
    void use3()
    {
        D d;
        d.f();   // OK
    }
```

##### Enforcement

모든 절단(slicing)을 지적하라.

### <a name="Rh-dynamic_cast"></a>C.146: 클래스 계층구조 탐색이 불가피한 경우에만 `dynamic_cast`를 사용하라

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

다른 캐스팅 방법은 타입 안전성을 해치고 프로그램이 실제로 `X`인 변수를 `Z`처럼 사용하게 만든다:

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
가능한 한 클래스 계층을 탐색하는 것보다 [정적 다형성(링크 없음))](#???)을 선호하라. 이렇게 하면 실행시간 결정이 필요없다. 그리고 충분히 편리하다.

##### Note

`typeid`가 더 적절한데 `dynamic_cast`를 쓰는 사람들이 있다;
`dynamic_cast`는 일반적으로 알려진 (개체에 가장 적합한 인터페이스를 찾기 위한) "is kind of" 연산이다.
반면 `typeid`는 "이 개체의 정확한 타입을 찾는" 연산이다. 후자는 단순하고 더 빠르게 처리될 것이 분명하다.
`typeid`는 필요하다면 쉽게 작성할 수 있다(모종의 이유로 RTTI가 지원되지 않는다면). `dynamic_cast`는 보통 정확하게 구현하기 훨씬 어렵다.

다음과 같은 예를 생각해보라:

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

`pb2->id() == "D"`의 결과는 실제로는 구현에 의해 결정된 것이다. 이는 직접 작성한 RTTI의 위험을 경고하기 위한 예시다.
수년간 이 코드가 기대한 대로 동작할수도 있지만, 새로운 기계, 컴파일러, 혹은 링커에서 소스코드 내 문자(character literals)를 통일하지 않으면 실패하게 된다.

RTTI를 직접 구현하고자 한다면, 주의하라.

##### Exception

만약 당신의 구현 코드에 정말로 느린 `dynamic_cast`가 있다면, 다른 방법을 찾아야 할 것이다.
하지만, 정적으로 클래스를 결정할 수 없는 모든 대안은 명시적 캐스팅(일반적으로 `static_cast`)을 포함하고, 에러에 취약하다.  

당신만의 특별한 `dynamic_cast`를 만들수도 있을 것이다. 그러니, `dynamic_cast`가 정말로 당신이 생각하는 것 만큼 느리다는 것을 확실히하라. (근거 없는 루머들이 꽤 있다.) 그리고 `dynamic_cast`의 사용이 정말로 성능에 치명적이라는 것 또한 확인하라.

이는 `dynamic_cast`의 현재 구현이 불필요하게 느린 경우에 대한 것이다. 
예를 들어, 적절한 조건 하에서는, `dynamic_cast`를 [O(1)시간 내로 수행](http://www.stroustrup.com/fast_dynamic_casting.pdf)할 수 있다. 하지만, 최적화를 위해 노력할 가치가 있다는데 모두가 동의하지 않으면 호환성은 코드 변경을 어렵게 한다.

매우 드물게, `dynamic_cast`의 오버헤드가 문제가 된다면, 하향 캐스팅이 성공한다고 정적으로 보장되는 다른 방법을 쓸 수도 있다 (예를 들어 Curiously Recurring Template Pattern을 주의해서 쓰는 방법처럼). 
가상 상속을 사용하지 않는다면, 확실한 주석과 함께 `static_cast`에 의존하거나 시스템이 정확성을 검증할 수 없기 때문에 유지보수에 사람이 필요하다는 조항을 작성하라.
그렇게 하더라도, 우리 경험으로 미루어 "나는 지금 뭘 하는지 몰라요"는 버그의 근원이다.

##### Exception

다음과 같은 코드는 예외로 볼 수 있다:

```c++
    template<typename B>
    class Dx : B {
        // ...
    };
```

##### Enforcement

* 모든 하위 타입으로의 `static_cast`를 지적하라. `static_cast`를 수행하는 C 스타일 변환도 포함하라
* 이 규칙은 [타입 안전성](./Profile.md#Pro-type-downcast) 규칙들의 일부이다

### <a name="Rh-ref-cast"></a>C.147: 필요한 클래스를 찾지 못한 경우가 오류에 해당하는 경우 `dynamic_cast`를 참조 타입에 사용하라

##### Reason

참조자에 대한 캐스팅은 당신이 정상적인 개체를 얻는 것을 의도했음을 표현한다. 따라서 캐스팅은 반드시 성공해야만 한다. `dynamic_cast`는 만약 실패한다면 예외를 던질 것이다.

##### Example

```
    ???
```

##### Enforcement

???

### <a name="Rh-ptr-cast"></a>C.148: 필요한 클래스를 찾지 못한 경우가 대안으로 사용된다면 `dynamic_cast`를 포인터 타입에 사용하라

##### Reason

`dynamic_cast`는 포인터가 계층구조 내에서 다형적 개체를 가리키고 있는지 검사할 수 있도록 해준다.
해당 클래스를 찾는데 실패할 경우 단순히 null 값을 반환하기 때문에, 실행시간에 검사하는 것이 가능하다. 이 덕분에 참색 결과에 따라 다른 방법을 선택하는 코드를 작성하는 것이 가능하다.

[C.147](#Rh-ptr-cast) 항목과 반대로, 탐색 실패가 오류라면, 조건부 실행에서 사용되어선 안된다.

##### Example

아래의 예시는 `Shape_owner`의 `add` 함수가 생성된 `Shape`의 소유권을 가져가는 것을 보여준다. 개체들은 기하학적 특성에 따라 정렬된다.

이 예시에선, `Shape`는 `Geometric_attributes`를 상속하지 않는다.

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

탐색 실패로 인해 `dynamic_cast`이 null을 반환하기 때문에, null 포인터 참조가 발생하고 미정의 동작으로 이어질 것이다.
따라서 `dynamic_cast`의 결과는 항상 null 값인지 검사되어야 한다.

##### Enforcement

* (복잡함) `dynamic_cast`의 포인터 타입 반환을 검사하는 코드가 없으면, 그 포인터의 사용을 경고하라

### <a name="Rh-smart"></a>C.149: 동적 할당한 개체의 소멸을 잊지 않도록 `unique_ptr` 혹은 `shared_ptr`를 사용하라

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

### <a name="Rh-make_unique"></a>C.150: `unique_ptr`로 소유되는 개체를 생성하기 위해서는 `make_unique()`를 사용하라

##### Reason

`make_unique`는 생성에 대한 보다 정확한 구문을 제공한다. 복잡한 표현식에서 예외 안전성을 보장한다.

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

* 반복적인 템플릿 특수화 `<Foo>`의 사용을 지적한다
* `unique_ptr<Foo>`로 선언된 변수들을 지적한다


### <a name="Rh-make_shared"></a>C.151: `shared_ptr`로 소유되는 개체를 생성하기 위해서는 `make_shared()`를 사용하라

##### Reason

`make_shared`는 생성에 대한 보다 정확한 구문을 제공한다. 참조 카운트에 대한 별도의 공간 할당이 필요없게 된다. `shared_ptr`는 개체의 옆(다음 영역)에 참조 카운트를 배치해 사용한다.

##### Example

```c++
    void test() {
        // OK: but repetitive; and separate allocations for the Bar and shared_ptr's use count
        shared_ptr<Bar> p {new<Bar>{7}};

        auto q = make_shared<Bar>(7);   // Better: no repetition of Bar; one object
    }
```

##### Enforcement


* 반복적인 템플릿 특수화 `<Bar>`의 사용을 지적한다
* `shared_ptr<Bar>`로 선언된 변수들을 지적한다

### <a name="Rh-array"></a>C.152: 절대로 하위 클래스의 포인터에 상위 클래스 포인터를 대입하지 마라

##### Reason

상위 타입 포인터를 대입하면 부적절한 개체 접근이 발생하고 아마도 메모리 손상을 일으킬 것이다.

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

* 배열 포인터의 변환이나 상위 타입에서 하위 타입으로의 변환을 지적한다
* 배열은 포인터보다는 `span`을 사용해서 전달하라. 그리고 `span`을 생성하기  전까지는 하위 타입에서 상위 타입으로 변환되지 않도록 하라

### <a name="Rh-use-virtual"></a>C.153: 타입 캐스팅보다 가상 함수를 선호하라

##### Reason

타입 캐스팅이 오류에 취약한 반면 가상함수 호출은 안전하디. 가상 함수 호출은 최종 구현을 사용하는 반면, 타입 캐스팅은 중간 클래스에 적용될수도 있다. 이로 인해 잘못된 결과를 반환할 수 있다 (계층 구조가 유지보수 과정에서 변경되었다면).

##### Example

```
    ???
```

##### Enforcement

[C.146](#Rh-dynamic_cast)를 참고하라

## <a name="SS-overload"></a>C.over: 중복정의(Overloading)

일반 함수, 템플릿 함수, 연산자를 중복 정의할 수 있다. 함수 개체는 중복정의할 수 없다.

중복정의 규칙 요약:

* [C.160: 연산자를 정의할때는 전통적인 사용을 모방하라](#Ro-conventional)
* [C.161: 대칭적인 연산자는 비멤버 함수로 정의하라](#Ro-symmetric)
* [C.162: 거의 동등한 연산들을 중복정의하라](#Ro-equivalent)
* [C.163: 거의 동등한 연산들'만' 중복정의하라](#Ro-equivalent-2)
* [C.164: 암묵적 형변환 연산자들을 지양하라](#Ro-conversion)
* [C.165: 커스터마이징이 필요하면 `using`을 사용하라](#Ro-custom)
* [C.166: 단항 연산자 `&`는 스마트 포인터와 참조 체계를 따르는 경우에만 중복정의하라](#Ro-address-of)
* [C.167: 연산자는 전통적인 의미를 수행하는데 사용하라](#Ro-overload)
* [C.168: 연산자를 중복정의는 피연산자의 네임스페이스에 하라](#Ro-namespace)
* [C.170: 람다를 중복 정의하고 싶다면, 제네릭 람다를 사용하라](#Ro-lambda)

### <a name="Ro-conventional"></a>C.160: 연산자를 정의할때는 전통적인 사용을 모방하라

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

이 예시에선 전통적인 의미구조를 따른다: [복사된 개체는 동등한 값을 가진다](#SS-copy).

##### Example, bad

```c++
    X operator+(X a, X b) { return a.v - b.v; }   // bad: makes + subtract
```

##### Note

멤버가 아닌 연산자들은 friend이거나 [피연산자들과 같은 네임스페이스에 정의되어야 한다](#Ro-namespace).
[이항 연산자들은 피연산자를 동등하게 다뤄야 한다](#Ro-symmetric).

##### Enforcement

거의 불가능하다

### <a name="Ro-symmetric"></a>C.161: 대칭적인 연산자는 비멤버 함수로 정의하라

##### Reason

연산자 정의에 멤버함수를 사용하면 피연산자 타입마다 멤버함수가 필요하다.
가령 `==` 연산자에 비 멤버 함수를 사용하지 않는다면, `a == b`와 `b == a`가 미묘하게 다를 것이다.

##### Example

```c++
    bool operator==(Point a, Point b) { return a.x == b.x && a.y == b.y; }
```

##### Enforcement

멤버 함수인 연산자들을 지적하라.

### <a name="Ro-equivalent"></a>C.162: 거의 동등한 연산들을 중복정의하라

##### Reason

논리적으로 같은 연산이 다른 타입에 다른 이름을 가지는 것은 혼란스럽고, 타입 정보를 함수 이름에 집어넣게 된다. 제네릭 프로그래밍에도 방해된다.

##### Example

다음과 같은 예를 생각해보라:

```c++
    void print(int a);
    void print(int a, int base);
    void print(const string&);
```

이 세 함수들은 인자를 출력한다. 다른 경우:

```c++
    void print_int(int a);
    void print_based(int a, int base);
    void print_string(const string&);
```

이 세 함수들은 인자를 출력한다. 인자 타입을 이름에 붙이는 것은 불필요하고 일반적인 코드를 작성하지 못하게 한다.

##### Enforcement

???

### <a name="Ro-equivalent-2"></a>C.163: 거의 동등한 연산들'만' 중복정의하라

##### Reason

논리적으로 다른 함수가 같은 이름을 가지는 것은 혼란을 일으키고 제네릭 프로그래밍에서 오류로 이어진다.

##### Example

다음과 같은 예를 생각해보라:

```c++
    void open_gate(Gate& g);   // remove obstacle from garage exit lane
    void fopen(const char* name, const char* mode);   // open file
```

이 두 함수는 근본적으로 다르고 연관성이 없다. 따라서 다른 이름을 가지는 것이 좋다. 다른 경우:

```c++
    void open(Gate& g);   // remove obstacle from garage exit lane
    void open(const char* name, const char* mode ="r");   // open file
```

이 두 연산은 여전히 근본적으로 다르고 연관성을 가지지 않는다. 하지만 이름이 축약되었고 혼란의 가능성을 만든다. 다행히도, 이들의 시그니처가 다르기 때문에 타입시스템이 실수를 잡아낼 것이다.

##### Note

`open`, `move`, `+`, `==`과 같이 일반적이고 많이 쓰이는 이름에는 특히 주의하라. 

##### Enforcement

???

### <a name="Ro-conversion"></a>C.164: 암묵적 형변환 연산자들을 지양하라

##### Reason

묵시적 변환이 필수적일 수 있다 (`double`에서 `int`로 바꾼다던지). 하지만 (`String`에서 C-style 문자열이 되는 것처럼) 의도치 않은 동작이 생기기도 한다. 

##### Note

정말 필요한 경우가 발생하지 않는다면 명시적 변환을 선호하라.
"정말 필요한"은 응용 프로그램의 영역에서 기본적이고 자연스러우며 자주 필요한 경우를 의미한다. (가령 정수를 복소수로 변환하는 것처럼) 
(변환 연산자 또는 암묵적 생성자를 통해서) 암묵적 변환을 사용하지 마라. 약간의 편안함만 얻을 수 있을 뿐이다.

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

이런 놀랍고 잠재적 피해가 발생할 수 있는 암묵적 변환은 찾아내기 어려운 문맥 속에서 발생할 수 있다. 예를 들어,

```c++
    S1 ff();

    char* g()
    {
        return ff();
    }
```

`ff()`에서 반환된 문자열이 포인터가 사용되기 전에 파괴된다.

##### Enforcement

모든 형변환 연산자를 지적하라

### <a name="Ro-custom"></a>C.165: 커스터마이징이 필요하면 `using`을 사용하라

##### Reason

다른 네임스페이스에 위치한 함수 개체와 함수들을 찾고 보편적인 함수로 "커스터마이즈" 할 수 있다.

##### Example

`swap`을 생각해보라. 이 함수는 일반적인 (표준 라이브러리) 함수이고, 어떤 타입에 대해서도 동작한다.
하지만, 어떤 타입들은 특별한 `swap()`을 정의할 필요가 있다.
예를 들어, 일반적인 `vector`의 `swap()`은 컨테이너 내 원소들을 복사한다. 좋은 구현이라면 복사를 수행하지 않을 것이다.

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

`f1()`함수 안에서 `std::swap()`을 사용하는 것은 코드 그대로 실행된다: `std` 네임스페이스의 `swap()`을 호출할 것이다.
불행히도, 그게 우리가 원하는 것은 아니다. `N::X` 타입에 맞게 호출되게 하려면 어떻게 해야 할까?

```c++
    void f2(N::X& a, N::X& b)
    {
        swap(a, b);   // calls N::swap
    }
```

이 방법은 우리가 원하는 일반화된 코드가 아닐 수 있다. 
우리는 보통 특별히 작성된 함수가 있으면 그 함수를 쓰고 그렇지 않은 경우에는 범용 함수(general function)를 호출하기를 원한다. 이는 함수 탐색에 범용 함수를 포함하는 것으로 해결할 수 있다:

```c++
    void f3(N::X& a, N::X& b)
    {
        using std::swap;  // std::swap이 탐색되도록 한다
        swap(a, b);       // N 네임스페이스에 swap이 존재하면 호출하고, 그렇지 않으면 std::swap을 호출한다
    }
```

##### Enforcement

`swap`같이 잘 알려진 커스터마이징을 제외하면 할 수 있는게 거의 없다.
문제는 qualified 탐색과 unqualified 탐색이 함께 사용된다는 것이다.

> 역주:  
> [이름 탐색](https://github.com/CppKorea/CppTemplateStudy/blob/master/7th%20Study/Chap13.md#2-이름-탐색) - C++ Korea Template Study

### <a name="Ro-address-of"></a>C.166: 단항 연산자 `&`는 스마트 포인터와 참조 체계를 따르는 경우에만 중복정의하라

##### Reason

`&` 연산자는 C++에서 필수적이다. C++ 에서 사용되는 의미구조의 많은 부분이 기본적인 의미를 전제하고 있다.

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

`&` 연산자로 "뭔가 하려면" `->`, `[]`, `*`, `.` 연산자들에 적합한 정의(반환 타입)를 가지도록 하라. `.` 연산자는 현재로써는 중복정의할 수 없기 때문에 완벽한 체계를 갖추는 것은 불가능하다.

다음 문서를 보기를 권한다: <http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4477.pdf>.
`std::addressof()`는 항상 내장 포인터 타입을 반환한다는 점에 유의하라.

##### Enforcement

까다롭다. `&` 연산자가 `->`와 함께 사용자 정의되지 않았다면 경고한다.

### <a name="Ro-overload"></a>C.167: 연산자는 전통적인 의미를 수행하는데 사용하라

##### Reason

가독성, 관례적 의미, 재사용성, 일반화된 코드에 도움이 된다

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

그 자체로, `cout_my_class`는 문제가 없다. 하지만 관례적으로 출력에 사용하는 `<<` 연산자에 맞게 작성할 수 없다:

```c++
    My_class var { /* ... */ };
    // ...
    cout << "var = " << var << '\n';
```

##### Note

대부분의 연산자들은 강력하고 흔히 사용되는 의미를 가지고 있다

* 비교 (`==`, `!=`, `<`, `<=`, `>`, `>=`),
* 산술 연산 (`+`, `-`, `*`, `/`, `%`)
* 접근 연산 (`.`, `->`, 단항 `*`, `[]`)
* 대입 (`=`)

관례적으로 사용되어온 의미와 다르게 정의하거나 새롭게 의미를 부여해서 사용하지 말아라.

##### Enforcement

까다롭다. 의미구조에 대한 통찰이 필요하다.

### <a name="Ro-namespace"></a>C.168: 연산자를 중복정의는 피연산자의 네임스페이스에 하라

##### Reason

가독성. 인자 기반 탐색(ADL)이 가능하다. 다른 네임스페이스에 정의하는 것은 일관적이지 않다.

##### Example

```c++
    struct S { };
    bool operator==(S, S);   // OK: in the same namespace as S, and even next to S
    S s;

    bool x = (s == s);
```

기본적인 `==` 연산자가 하는 일이다.

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

네임스페이스 `N`과 `M` 에서 `!s`의 의미가 달라진다. 굉장히 혼란스러울 수 있다. `namespace M`의 정의를 제거하면 실수가 발생할 가능성의 사라진다.

##### Note

이항 연산자가 다른 네임스페이스에 있는 두 타입에 대해서 정의되었다면, 이 규칙을 따를 수 없다.
예를 들어:

```c++
    Vec::Vector operator*(const Vec::Vector&, const Mat::Matrix&);
```

이런 경우는 피하는 것이 최선이다.

##### See also

[보조 함수들은 관련 클래스와 같은 namespace에 배치하라](#Rc-helper)는 규칙의 특별한 경우에 해당한다

##### Enforcement

* 피연산자의 네임스페이스에 위치하지 않은 연산자 정의를 지적한다

### <a name="Ro-lambda"></a>C.170: 람다를 중복 정의하고 싶다면, 제네릭 람다를 사용하라

##### Reason

같은 이름으로 두개의 서로 다른 람다를 중복 정의할 수 없다.

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

컴파일러가 람다를 중복 정의하려는 시도를 잡아낸다.

## <a name="SS-union"></a>C.union: 공용체(Union)

`union`은 모든 멤버가 같은 주소에서 시작하는 `struct`라고 할 수 있다. 따라서 한 시점에 하나의 멤버만 가지고 있다.
`union`은 어던 멤버가 저장되었는지 추적하지 않기 때문에 프로그래머가 정확하게 사용해야 한다; 이는 필연적으로 오류를 발생시키지만, 이를 보완할 방법은 있다

`union`에 어떤 멤버가 사용되고 있는지 알려주도록 표지(indicator)가 추가된 것을 *tagged union*, *discriminated union*, 혹은 *variant*이라고 부른다. 

공용체(Unions) 규칙 요약:

* [C.180: `union`은 메모리를 절약하기 위해 사용하라](#Ru-union)
* [C.181: 표지 없는(naked) `union`을 사용하지 말아라](#Ru-naked)
* [C.182: Tagged union 구현에는 익명 `union`을 사용하라](#Ru-anonymous)
* [C.183: 타입 재해석(type punning)을 위해 `union`을 사용하지 말아라](#Ru-pun)
* ???

### <a name="Ru-union"></a>C.180: `union`은 메모리를 절약하기 위해 사용하라

##### Reason

`union`은 하나의 메모리 조각이 다른 시각에 다른 타입의 개체들로 사용될 수 있도록 해준다. 결과적으로, 다른 개체들이 동시에 사용되지 않는다면 메모리를 절약하는데 사용할 수 있다.

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

하지만 경고를 유심히 보라: [표지 없는(naked) `union`을 사용하지 말아라](#Ru-naked)

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

### <a name="Ru-naked"></a>C.181: 표지 없는(naked) `union`을 사용하지 말아라

##### Reason

*표지 없는(naked) union*은 어떤 멤버를 사용하는지 알 수 없는 union을 의미한다. 이런 경우 프로그래머가 계속 추적해야 한다. 타입 오류의 원인이 된다.

##### Example, bad

```c++
    union Value {
        int x;
        double d;
    };

    Value v;
    v.d = 987.654;  // v holds a double
```

여기까진 문제가 없다. 하지만 `union`은 잘못 사용하기 쉽다:

```c++
    cout << v.x << '\n';    // BAD, undefined behavior: v holds a double, but we read it as an int
```

명시적 변환이 없음에도 타입 오류가 발생한 점에 주목하라.
마지막으로 이 프로그램을 테스트 했을떄 출력된 값은 `987.654`의 비트 패턴을 정수로 해석한 `1683627180`이었다.
이 예시에서는 코드에는 문제 없어 보이지만 "보이지 않는(invisible)" 타입 오류가 발생하는 것을 보여준다.

그리고, "보이지 않는" 오류로, 이 코드는 아무것도 출력하지 않는다:

```c++
    v.x = 123;
    cout << v.d << '\n';    // BAD: undefined behavior
```

##### Alternative

타입 필드를 추가해서 `union`을 클래스로 감싼다.

`<variant>` 헤더의 표준 `variant` 타입이 이 일을 대신 해준다:

```c++
    variant<int, double> v;
    v = 123;        // v holds an int
    int x = get<int>(v);
    v = 123.456;    // v holds a double
    w = get<double>(v);
```

##### Enforcement

???

### <a name="Ru-anonymous"></a>C.182: Tagged union 구현에는 익명 `union`을 사용하라

##### Reason

잘 설계된 Tagged union은 타입 안전성을 가지고 있다.
*익명(anonymous)* union은 (tag, union) 형태의 클래스 정의를 쉽게 만들어준다.

##### Example

이 예제는 대부분 TC++PL4 pp216-218 에서 발췌한 것이다. 설명을 원한다면 해당 문서를 참고하라.

예시 코드는 상세한 편이다. 이 타입에서 사용자가 정의한 대입 연산과 소멸자를 다루는 것은 꽤 신경써야 하는 작업이다. 
이런 작업을 프로그래머가 하지 않도록 하는 것이 `variant`가 표준에 추가된 이유 중 하나다.

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

### <a name="Ru-pun"></a>C.183: 타입 재해석(type punning)을 위해 `union`을 사용하지 말아라

##### Reason

`union`의 멤버를 한 타입으로 값을 쓰고 다른 타입으로 읽는 것은 미정의 동작(undefined behavior)이다.
이런 해석은 보이지 않고, 타입 이름을 사용하는 것보다 찾아내기 어렵다.
`union`을 사용한 타입 재해석은(type punning)은 오류의 원인이다.

##### Example, bad

```c++
    union Pun {
        int x;
        unsigned char c[sizeof(int)];
    };
```

`Pun` 타입의 의도는 `int`를 `char` 형태로 접근하는 것이다.

```c++
    void bad(Pun& u)
    {
        u.x = 'x';
        cout << u.c[0] << '\n';     // undefined behavior
    }
```

`int`의 바이트를 확인하고 싶다면, 타입 이름을 사용한 형변환(named cast)를 사용하라:

```c++
    void if_you_must_pun(int& x)
    {
        auto p = reinterpret_cast<unsigned char*>(&x);
        cout << p[0] << '\n';     // OK; better
        // ...
    }
```

`reinterpret_cast`을 사용해 타입을 바꿔서 접근하는 것은 정의된 행동(defined behavior)이다(비록  `reinterpret_cast`이 권장되지는 않지만).
이러면 최소한 신경을 많이 써야하는 것들이 사라진 것을 보여준다.

##### Note

불행하게도 `union`은 타입 재해석에 꽤 많이 사용된다.
"보통의 경우, 기대한 대로 동작한다"는 것은 강한 주장이라고 생각할 수 없다.

C++17 에서는 있는 그대로의 비트에 대해 연산을 수행할 수 있도록 `std::byte` 타입을 추가하였다. `unsigned char` 혹은 `char` 대신에 이 타입을 사용하라

##### Enforcement

???
