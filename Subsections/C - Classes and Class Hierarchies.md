# <a name="S-class"></a> C: 클래스와 클래스 계층 구조
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#S-class)

클래스는 사용자 정의 타입으로써, 타입의 표현과 연산, 인터페이스를 프로그래머가  정의할 수 있다.
클래스 계층 구조는 관련된 클래스들을 계층적으로 구조화 할 때 사용된다.

클래스 규칙 요약:

* [C.1: 관련된 데이터를 조직화 하라 (`struct` 와 `class`)](#Rc-org)
* [C.2: 타입이 불변조건을 가진다면, `class`를 사용하라; 데이터 멤버들에 대한 제약이 자유롭다면 `struct`를 사용하라](#Rc-struct)
* [C.3: 클래스를 사용해 인터페이스와 구현을 분리하라](#Rc-interface)
* [C.4: 클래스에 직접적으로 접근할 필요가 있는 경우에만 함수를 멤버함수로 작성하라](#Rc-member)
* [C.5: 보조 함수들은 관련 클래스와 같은 namespace에 배치하라](#Rc-helper)
* [C.7: Don't define a class or enum and declare a variable of its type in the same statement](#Rc-standalone)
* [C.8: non-public 멤버가 있다면 `struct`보단 `class`를 사용하라](#Rc-class)
* [C.9: 멤버들의 노출을 최소화하라](#Rc-private)

하위 영역:

* [C.concrete: 구체적인 타입들](#SS-concrete)
* [C.ctor: 생성자, 대입 연산자, 소멸자](#S-ctor)
* [C.con: 컨테이너와 리소스 핸들](#SS-containers)
* [C.lambdas: 함수 객체와 람다 표현식](#SS-lambdas)
* [C.hier: 클래스 계층 구조 (OOP)](#SS-hier)
* [C.over: 오버로딩](#SS-overload)
* [C.union: 공용체](#SS-union)


### <a name="Rc-org"></a>C.1: 관련된 데이터를 조직화 하라 (`struct` 와 `class`)

##### 근거
이해하기 쉽다. 근본적인 이유로 데이터가 관련이 있다면, 그 사실은 코드에 반영되어야 한다.

##### 예
```
    void draw(int x, int y, int x2, int y2);  // BAD: 암묵적인 관계를 지닌다
    void draw(Point from, Point to);          // 더 낫다.
```

##### 참고 사항
가상 함수가 없는 간단한 클래스는 공간, 시간적인 오버헤드가 없다.

##### 참고 사항
언어적인 관점에서 볼 때 `class` 와 `struct`의 차이는 멤버의 기본적인 가시성이다.

##### 시행하기
특별히 없다. 데이터 항목들에 대한 경험적인 관점들이 함께 반영될 수는 있을 것이다.


### <a name="Rc-struct"></a>C.2: 타입이 불변조건을 가진다면, `class`를 사용하라; 데이터 멤버들에 대한 제약이 자유롭다면 `struct`를 사용하라

##### 근거
가독성이 좋고 이해하기도 쉽다. 
`class` 를 사용함으로써, 프로그래머가 invariant가 필요하다는 것을 알게 된다.  
이 점은 유익한 관습이다.

##### 참고 사항
invariant는 객체 멤버들의 논리적인 상태로써, 공개 멤버 함수들이 가정할 수 있도록 생성자가 설정 해 주어야 한다. invariant 가 설정된 후에야 (일반적으로 생성자에 의해) 모든 멤버 함수는 객체를 통해 호출될 수 있다. 
invariant 는 형식에 구애받지 않고 (가령, 주석으로) 기술될 수 있으며, 더 형식을 갖춘다면 `Expects` 를 사용할 수 있다.

만약 모든 데이터 멤버들이 상호독립적이라면, 불변조건은 존재할 수 없다. 

##### 예
```
    // 멤버들이 독립적으로 달라질 수 있다.
    struct Pair {  
        string name;
        int volume;
    };
```
하지만:
```
    class Date {
    public:
        // 생성자가 {yy, mm, dd}를 확인하고 초기화한다.
        Date(int yy, Month mm, char dd);
        // ...
    private:
        int y;
        Month m;
        char d;    // day
        Date(int yy, Month mm, char dd);
    };
```
##### 참고 사항
클래스가 어떤 `private` 데이터를 가지고 있으면, 사용자는 생성자 호출 없이 객체를 초기화할 수 없다. 
따라서, 클래스를 정의하는 사람은 생성자를 제공하고 그 의미를 명시해야만 한다.
이는 클래스 작성자가 invariant를 정의해야 한다는 것을 의미한다.

* 참고 : [define a class with private data as `class`](#Rc-class).
* 참고 : [Prefer to place the interface first in a class](#Rl-order).
* 참고 : [minimize exposure of members](#Rc-private).
* 참고 : [Avoid `protected` data](#Rh-protected).

##### 시행하기
private 데이터를 가진 `struct`나 public 멤버를 가진 `class`들을 확인해보라.


### <a name="Rc-interface"></a>C.3: 클래스를 사용해 인터페이스와 구현을 분리하라

##### 근거
인터페이스와 구현에 대한 분명한 구분은 가독성을 더 좋게 하고, 유지 보수를 단순하게 한다.

##### 예
```
    class Date {
        // ... some representation ...
    public:
        Date();
        // {yy, mm, dd}이 유효한지 확인하고 초기화한다
        Date(int yy, Month mm, char dd);

        int day() const;
        Month month() const;
        // ...
    };
```
이러한 경우, 이제 사용자에게 영향을 주지 않고 `Date` 에 대한 representation을 변경할 수 있다. (비록 다시 컴파일 해야 하겠지만)

##### 참고 사항
인터페이스와 구현간의 구분을 표현하기 위해 클래스를 사용하는 것이 유일한 방법은 아니다.
예를 들면, 인터페이스를 표현하기 위한 개념으로 네임스페이스 안에 독립적인 함수들이나 추상 기본 클래스 혹은 템플릿 함수들을 선언해서 사용할 수 있다.
가장 중요한 것을 명시적으로 인터페이스와 그것들의 구현 "세부사항"을 구분하는 것이다.
이상적으로, 그리고 일반적으로, 인터페이스는 그 구현들보다 훨씬 더 안정적이다.

##### 시행하기
???


### <a name="Rc-member"></a>C.4: 클래스에 직접적으로 접근할 필요가 있는 경우에만 함수를 멤버함수로 작성하라

##### 근거
멤버 함수간 커플링을 줄이고, 객체 상태 변경에 의해 문제가 생기는 함수를 줄이고, representation이 변경된 후에 수정될 필요가 있는 멤버 함수의 수를 줄인다.

##### 예
```
    class Date {
        // ... 상대적으로 적은 인터페이스 ...
    };

    // helper functions:
    Date next_weekday(Date);
    bool operator==(Date, Date);
```

표시된  "helper functions"은 `Date`의 representation에 직접 접근할 필요가 없다.

##### 참고 사항
이 규칙은 C++17 에서 "uniform function call" 이 들어오면 더 좋아질 것이다. ???

##### 시행하기
데이터 멤버를 직접 조작하지 않는 멤버 함수를 찾아보라.   
문제는 데이터 멤버를 직접 건드릴 필요가 없는 멤버 함수들이 많이 있다는 것이다.


### <a name="Rc-helper"></a>C.5: 보조 함수들은 관련 클래스와 같은 namespace에 배치하라

##### 근거
보조 함수는 (보통 클래스 작성자가 제공하는) 클래스의 표현에 직접 접근할 필요가 없는 함수이며, 클래스에 대한 유용한 인터페이스 중에 하나로 볼 수 있다.
보조 함수들을 같은 네임스페이스에 넣으면 함수와 클래스의 관계가 명확해지고, 인자 종속적인 검색(Argument Dependent Lookup)에서 발견 할 수 있게 된다.

##### 예
```
    namespace Chrono { // here we keep time-related services

        class Time { /* ... */ };
        class Date { /* ... */ };

        // 보조(helper) 함수들:
        bool operator==(Date, Date);
        Date next_weekday(Date);
        // ...
    }
```
##### 참고 사항
이는 [overloaded operators](#Ro-namespace)를 위해서 매우 중요하다.

##### 시행하기
* 단일 네임스페이스에서 인자 타입을 취하는 전역함수들에 표시를 남겨라.


### <a name="Rc-standalone"></a>C.7: 클래스 또는 열거형에 대한 정의와 변수 선언을 같은 구문에 넣지 말아라

##### 근거
타입에 대한 정의와 다른 개체(entitiy)에 대한 정의를 같은 구문(statement)에 넣는 것은 혼동을 일으킬 수 있고, 불필요하다.

##### 잘못된 예
```
    struct Data { /*...*/ } data{ /*...*/ };
```
##### 좋은 예
```
    struct Data { /*...*/ };
    Data data{ /*...*/ };
```
##### 시행하기
* 클래스나 열거형의 정의에 있는 닫는 괄호 `}`에 `;`이 따라오도록 하라.



### <a name="Rc-class"></a>C.8: non-public 멤버가 있다면 `struct`보단 `class`를 사용하라

##### 근거
가독성에 좋다. 
무엇인가 숨겨져 있거나, 추상화되었다는 것을 분명하게 한다.  
유익한 관습이다.

##### 잘못된 예
```
    struct Date {
        int d, m;

        Date(int i, Month m);
        // ... 많은 함수들 ...
    private:
        int y;  // year
    };
```
C++ 언어 규칙을 고려했을 때 이 코드엔 잘못된 것이 없다.  
하지만 디자인 관점에서는 모든게 잘못되었다.
private 데이터가 public data와 멀리 떨어져 숨어있고, 클래스 선언의 다른 부분들로 분리되어 있다.  
이런 요소들은 가독성을 저해하고 유지보수를 복잡하게 한다.

##### 참고 사항
클래스 인터페이스를 먼저 배치하라. [해당 항목](#Rl-order) 

##### 시행하기
Flag classes declared with `struct` if there is a `private` or `public` member.


### <a name="Rc-private"></a>C.9: 멤버들의 노출을 최소화하라

##### 근거
캡슐화. 정보 은닉. 의도치 않은 접근을 최소화 하고, 유지보수를 쉽게 한다.

##### 예
```
    ???
```
##### 참고 사항
`public`멤버를 가장 앞에, `protected` 멤버를 다음에, `private`멤버를 마지막에 배치하라. [해당 항목](#Rl-order).

##### 시행하기
???



## <a name="SS-concrete"></a>C.concrete: Concrete types
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#SS-concrete)

클래스의 이상 중 하나는 기본 타입이 되는 것이다.

쉽게 말하면 "`int` 처럼 동작하는 것"이다. Concrete type은 가장 간단한 종류의 클래스를 의미한다.

기본 타입의 값은 복사 될 수 있고, 복사의 결과는 원본과 같은 값을 갖는 독립적인 객체이다.
타입이 `=` 와 `==` 를 모두 갖는다면, `a = b`를 실행한 이후에는 `a == b`에서 `true`가 반환되도록 해야 한다.

대입과 동등 비교가 없는 Concrete classes들은 정의될 수는 있지만, 그런 경우는 드물다(드물어야 한다). 
C++의 언어 내장(built-in) 타입들은 기본 타입들이고, `string`, `vector`, `map`같은 표준 라이브러리의 클래스들 또한 그렇다. 

Concrete type들은 종종 계층구조의 일부로 사용되는 타입들과 구분하여 값 타입으로 언급된다.

구체적인 타입 규칙 요약:

* [C.10: 복잡한 클래스들 보다 Concrete type을 선호하라](#Rc-concrete)
* [C.11: Concrete type은 일반적으로 만들어라](#Rc-regular)


### <a name="Rc-concrete"></a>C.10 복잡한 클래스들 보다 Concrete type을 선호하라

##### 근거
구체적인 타입은 근본적으로 계층구조 보다 단순하다:
디자인이 더 쉽고, 구현이 더 쉽고, 사용하기가 더 쉬우며, 추론하기 더 쉽다. 더 작고 더 빠르기도 하다.  
계층구조를 사용할 때는 마땅한 이유가 있어야 한다.

##### 예
```
    class Point1 {
        int x, y;
        // ... 연산들 ...
        // ... virtual 함수는 없다 ...
    };

    class Point2 {
        int x, y;
        // ... 연산들, 일부는 virtual 함수 ...
        virtual ~Point2();
    };

    void use()
    {
        Point1 p11 {1, 2};   // Stack에 객체 생성
        Point1 p12 {p11};    // 복사 생성

        // 자유 영역에 객체 생성
        auto p21 = make_unique<Point2>(1, 2);   
        
        // 복사
        auto p22 = p21.clone();
        // ...
    }
```
클래스가 계층구조의 일부가 될 수 있다면, 반드시 포인터나 레퍼런스로 객체를 다루어야 한다.
이는 간접 처리를 위해 더 많은 메모리를 사용하게 되고, 더 많은 할당과 해제, 실행시간 오버헤드가 발생하게 된다는 것을 의미한다.

##### 참고 사항
구체적인 타입은 스택에 할당 될 수 있고, 다른 클래스의 멤버가 될 수 있다.

##### 참고 사항
실행시간에 다형적 인터페이스를 위해 간접처리는 필수적이다.
할당과 해제의 추가비용은 그렇지 않다. (단지 가장 흔한 사례일 뿐이다)
패생 클래스의 제한된(특정된) 객체에 대한 인터페이스로써 기본 클래스를 사용할 수도 있다.
동적 할당을 할 수 없으며, 플러그인과 같은 것들에게 안정적인 인터페이스를 제공하고자 할 때 이렇게 할 수 있다. (예컨대,  hard real-time)

##### 시행하기
???


### <a name="Rc-regular"></a>C.11:Concrete type은 일반적으로 만들어라 

##### 근거

Regular types are easier to understand and reason about than types that are not regular (irregularities requires extra effort to understand and use).

##### 예
```
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
In particular, if a concrete type has an assignment also give it an equals operator so that `a=b` implies `a == b`.

##### 시행하기
???



## <a name="S-ctor"></a>C.ctor: 생성자, 대입 연산자, 소멸자
> [원문](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#S-ctor)

이 함수들은 객체의 생명주기를 제어 한다: 생성, 복사, 이동, 그리고 소멸.
생성자를 정의해서 클래스의 초기화를 보장하고 단순화 하라.

*기본 연산들*은 다음과 같다.

* 기본 생성자: `X()`
* 복사 생성자: `X(const X&)`
* 복사 대입 연산자: `operator=(const X&)`
* 이동 생성자: `X(X&&)`
* 이동 대입 연산자: `operator=(X&&)`
* 소멸자: `~X()`

컴파일러가 생성하는 기본 연산들은 객체의 생명주기를 의미하는 연산들의 집합이다.  
기본적으로, C++은 클래스를 값 타입 처럼 다루지만 모든 타입이 값 타입과 같은 것은 아니다.


기본 연산들 규칙들:

* [C.20: 기본 연산을 정의하지 않아도 되면 그렇게 하라](#Rc-zero)
* [C.21: 기본 연산을 정의 하거나 `=delete` 로 선언했다면, 나머지 모두 정의하거나 `=delete`하라](#Rc-five)
* [C.22: 기본 연산들을 일관성 있도록 하라](#Rc-matched)


소멸자 규칙들:

* [C.30: 객체가 없어질 때, 명시적인 동작이 필요할 경우 소멸자를 정의하라](#Rc-dtor)
* [C.31: 클래스에 의해 얻어진 모든 리로스는 소멸자에서 해제되어야 한다](#Rc-dtor-release)
* [C.32: 클래스가 포인터(`T*`)나 참조(`T&`)를 갖고 있을 때, 해당 클래스가 소유하고 있는지를 고려하라](#Rc-dtor-ptr)
* [C.33:클래스가 포인터 멤버를 소유하고 있다면, 소멸자를 정의하거나 `=delete` 로 선언하라](#Rc-dtor-ptr2)
* [C.34: If 클래스가 참조 멤버를 소유하고 있다면, 소멸자를 정의하거나 `=delete` 로 선언하라](#Rc-dtor-ref)
* [C.35: 가상 함수를 갖는 기본 클래스는 가상 소멸자가 필요하다](#Rc-dtor-virtual)
* [C.36: 소멸자는 실패해선 안된다](#Rc-dtor-fail)
* [C.37: 소멸자를 `noexcept`로 작성하라](#Rc-dtor-noexcept)


생성자 규칙들:

* [C.40: 클래스가 불변 조건을 가지면 생성자를 정의하라](#Rc-ctor)
* [C.41: 생성자는 완전히 초기화된 객체를 생성해야 한다](#Rc-complete)
* [C.42: 생성자가 유효한 객체를 생성하지 못한다면, 예외를 던지도록 하라](#Rc-throw)
* [C.43: 클래스가 기본 생성자를 갖도록 하라](#Rc-default0)
* [C.44: 기본 생성자는 되도록 단순하고 예외를 던지지 않게 작성하라](#Rc-default00)
* [C.45: 기본 생성자가 모든 멤버를 초기화하도록 하지 마라; 대신 멤버들이 스스로 초기화 하도록 하라](#Rc-default)
* [C.46: 단일 인자를 사용하는 생성자는 `explicit`으로 선언하라](#Rc-explicit)
* [C.47: 멤버 변수들은 선언된 순서대로 초기화 하라](#Rc-order)
* [C.48: 상수 초기화는 in-calss 멤버 초기화를 선호하라](#Rc-in-class-initializer)
* [C.49: 생성자 안에서의 대입 보다는 초기화를 선호하라](#Rc-initialize)
* [C.50: 초기화 과정에서 `virtual` 연산이 필요하다면, 팩토리 함수를 사용하라](#Rc-factory)
* [C.51: Use delegating constructors to represent common actions for all constructors of a class](#Rc-delegating)
* [C.52: 추가적인 초기화가 필요하지 않은 파생된 클래스에서 생성자를 사용할 때는 상속받은 생성자들을 사용하라](#Rc-inheriting)


복사와 이동 규칙들 : 

* [C.60: 복사연산을 `virtual`로 만들지 말아라. 매개변수는 `const&`로 받고, `const&`로 반환하지 말아라](#Rc-copy-assignment)
* [C.61: 복사 연산은 복사를 수행해야 한다](#Rc-copy-semantic)
* [C.62: 복사 연산은 자기 대입에 안전하게 작성하라](#Rc-move-self)
* [C.63: 이동 연산은 `virtual`로 만들지 말아라, 매개변수는 `&&`를 사용하고, `const&`로 반환하지 말아라](#Rc-move-assignment)
* [C.64: 이동 연산은 이동을 수행해야 하며, 원본 객체를 유효한 상태로 남겨놓아야 한다](#Rc-move-semantic)
* [C.65: 이동 연산은 자기 대입에 안전하게 작성하라](#Rc-copy-self)
* [C.66: 이동 연산은 `noexcept`로 만들어라](#Rc-move-noexcept)
* [C.67: 기본 클래스가 복사를 제한해야 하는데 복사가 요구된다면 virtual `clone`함수를 대신 제공하라](#Rc-copy-virtual)


다른 기본 연산들에 대한 규칙 :

* [C.80: 기본 의미론을 명시적으로 사용하려면 `=default` 키워드를 사용하라](#Rc-default)
* [C.81: 기본 동작을 (대안을 원하지 않고) 금지하고 싶다면 `=delete`를 사용하라](#Rc-delete)
* [C.82: 생성자 또는 소멸자에서 가상 함수를 호출하지 말아라](#Rc-ctor-virtual)
* [C.83: 값 타입들에는, `noexcept` swap함수를 제공하는 것을 고려하라](#Rc-swap)
* [C.84: `swap` 연산은 실패해선 안된다](#Rc-swap-fail)
* [C.85: `swap` 연산은 `noexcept`로 작성하라](#Rc-swap-noexcept)
* [C.86: `==`연산자는 피연산자 타입들에 대칭적이고, `noexcept`로 만들어라](#Rc-eq)
* [C.87: 기본 클래스에 있는 `==`에 주의하라](#Rc-eq-base)
* [C.89: `hash`는 `noexcept`로 작성하라](#Rc-hash)



## <a name="SS-defop"></a>C.defop: 기본 연산들

기본적으로, 언어에서 기본적인 의미를 담는 기본 연산들을 제공한다.
그러나, 프로그래머는 기본적으로 제공되는 것들을 막거나 바꿀 수 있다.


### <a name="Rc-zero"></a>C.20: 기본 연산을 정의하지 않아도 되면 그렇게 하라

##### 근거
가장 단순하고, 가장 명료한 의미를 준다.

##### 예
```
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

##### 참고 사항
"the rule of zero"로 알려져 있다.

##### 시행하기
시행할 수 없더라도, 좋은 정적 분석기는 이 규칙에 맞는 가능한 개선사항들을 알려주는 패턴들을 찾을 수 있다.
예를 들면, 포인터와 크기를 멤버로 갖는 클래스가 있고 소멸자에서 그 포인터를 `delete` 한다면 아마도 `vector` 로 바꿀 수 있을 것이다.



### <a name="Rc-five"></a>C.21: 기본 연산을 정의 하거나 `=delete` 로 선언했다면, 나머지 모두 정의하거나 `=delete`하라

##### 근거
특별한 함수들의 의미론들은 밀접하게 연관되어 있다. 만약 한 함수가 기본 제공 함수가 아니어야 한다면(non-default), 다른 함수들도 수정이 필요하다.

##### 잘못된 예
```
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
여기서는, 소멸자에 대한 "특별한 주의"가 필요하다고 한다면, 복사와 이동 할당(둘 다 묵시적으로 객체를 소멸할 것이다)이 정확하게 동작할 가능성은 적다. (여기서는, 두번 `delete`를 시도할 것이다)

##### 참고 사항
기본 생성자를 중요하게 생각하는지에 달려있는데, 이것은 "the rule of five" 혹은 "the rule of six" 이라고 알려져 있다.

##### 참고 사항
다른 것은 정의 하더라도 기본 연산의 기본 구현이 필요하다면, `=default` 을 사용하여 해당 함수에 대한 의도를 표현하라.
기본 연산을 원하지 않는다면, `=delete`를 써서 제한하라.

##### 참고 사항
컴파일러는 이 규칙을 강제하고 이상적으로는 위반사항이 발생하면 경고한다.

##### 참고 사항
클래스에 묵시적으로 생성된 복사 연산에 의존하는 것은 더 이상 사용되지 않는다.

##### 시행하기
(쉬움) 클래스는 특별한 함수들에 대한 선언(`=delete`도 포함하여)을 모두 갖거나 갖지 말아야 한다.



### <a name="Rc-matched"></a>C.22: 기본 연산들을 일관성 있도록 하라

##### 근거
기본 연산들은 개념적으로 잘 짜여진 집합이다. 연산들의 의미는 서로 연관되어 있다.
사용자는 복사/이동 생성과 복사/이동 할당이 논리적으로 동일하고, 생성자와 소멸자가 리소스 관리에 대해 일관적으로 동작하며, 복사와 이동이 생성자와 소멸자가 동작하는 방식을 반영한다는 것을 기대 할 것이다.


##### 잘못된 예
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
이 연산들은 복사 연산에 대한 의미가 일치하지 않는다. 이런 동작은 혼란을 야기하고 버그를 만들 것이다.

##### 시행하기

* (어려움) 복사/이동 생성자와 이에 대응하는 복사/이동 할당 연산자는 동일한 레벨에서 동일한 멤버 변수를 변경하는 것이 좋다.
* (어려움) 복사/이동 생성자에서 변경하는 멤버 변수들은 다른 생성자들에서도 초기화 하는 것이 좋다.
* (어려움) 복사/이동 생성자는 멤버 변수에 대해 깊은 복사를 수행하고 나서, 소멸자는 멤버 변수를 수정해야 한다.
* (어려움) 소멸자가 멤버 변수를 변경하면, 그 멤버 변수들은 복사/이동 생성자 혹은 할당 연산자에서 쓰여지는 것이 좋다.


## <a name="SS-dtor"></a>C.dtor: 소멸자
"이 클래스에 소멸자가 필요할까?"라는 것은 설계 측면에서 굉장히 강력한 질문이다.
For most classes the answer is "no" either because the class holds no resources or because destruction is handled by [the rule of zero](#Rc-zero);
that is, its members can take care of themselves as concerns destruction.
If the answer is "yes", much of the design of the class follows (see [the rule of five](#Rc-five)).

### <a name="Rc-dtor"></a>C.30: 객체가 없어질 때, 명시적인 동작이 필요할 경우 소멸자를 정의하라

##### 근거
소멸자는 암묵적으로 객체의 생명주기의 마지막에 호출된다.
기본 소멸자로 충분하다면 그것을 사용하라.
단순하게 멤버의 소멸자를 호출하는 것이 아닌 코드가 필요할 경우 소멸자를 정의하라.

##### 예
```
    template<typename A>
    struct final_action {   // 약간 단순화된 클래스
        A act;
        final_action(A a) :act{a} {}
        ~final_action() { act(); }
    };

    template<typename A>
    final_action<A> finally(A act)   // 타입 추론
    {
        return final_action<A>{act};
    }

    void test()
    {
        // 최종 동작을 지정
        auto act = finally([]{ cout << "Exit test\n"; });  
        // ...
        if (something) 
            return;   // 여기서 소멸자를 통해 호출된다
        // ...
    } // 여기서 소멸자를 통해 호출된다
```
`Final_action` 의 목적은 소멸할 때 실행할 코드(보통 람다)를 얻는 것이다.


##### 참고 사항
사용자 정의 소멸자가 필요한 클래스에는 보통 두 종류가 있다.
* 리소스를 사용하는 클래스가 소멸자가 없는 경우, 예컨대 `vector` 혹은 트랜잭션 코드
* A class that exists primarily to execute an action upon destruction, such as a tracer or `final_action`.

##### 잘못된 예
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
기본 소멸자가 더 잘 동작하고, 더 효과적이며, 틀리지 않는다.

##### 참고 사항
기본 소멸자가 필요하지만, 생성되지 않도록 되어 있다면 (예, 이동 생성자를 정의한 경우), `=default` 를 사용하라.

##### 시행하기
포인터나 참조와 같은 "암묵적인 자원"이 될 수 있는 것들을 찾아보라. 
모든 데이터 멤버가 소멸자를 갖고 있더라도, 사용자 지정 소멸자가 있는 클래스들을 찾아보라.


### <a name="Rc-dtor-release"></a>C.31: 클래스에 의해 얻어진 모든 리소스는 소멸자에서 해제되어야 한다

##### 근거
리소스 누수를 막는다, 특히 에러 상황에서.

##### 참고 사항
클래스로 표현되는 리소스들이 기본 연산 집합을 갖고 있을 때 소멸자에서의 리소스 해제가 자동으로 발생한다.

##### 예
```
    class X {
        ifstream f;   // may own a file
        // ... no default operations defined or =deleted ...
    };
```
`X`의 `ifstream` 은 `X`가 소멸될 때 묵시적으로 열었을 수 있는 파일을 닫는다.  


##### 잘못된 예
```
    class X2 {     // bad
        FILE* f;   // may own a file
        // ... no default operations defined or =deleted ...
    };
```
`X2` 에서는 파일 핸들 누수가 생길 것이다.  

##### 참고 사항
닫지 않은 소켓은 어떨까? 소멸자, 닫기, 정리 연산은 [실패하지 않는 것이 좋다](#Rc-dtor-fail).
그럼에도 불구 하고 발생한다면, 정말 좋은 해결책을 찾기 힘든 문제를 만나는 것이다.
초심자들은 소멸자를 작성할 때 왜 소멸자가 호출되고, 예외를 던짐으로써 "처리 거부"를 할 수 없는지 알지 못할 것이다.
[discussion](#Sd-never-fail)을 보라.
문제를 악화시키는 것은, 많은 "닫기/해제" 연산들이 재시도 할 수 없도록 되어있는 것이다.
이 문제를 풀려는 시도는 많았지만, 일반적인 해결책은 알려지지 않았다.
해결책이 없다면, 닫기/해제에 대한 실패를 디자인 오류로 간주하고 종료시키는 것을 고려해 보라.

##### 참고 사항
클래스가 소유하고 있지 않은 객체에 대한 포인터나 참조를 갖고 있을 수 있다.
명백하게, 이 객체들은 클래스의 소멸자에서 `delete`되지 않아야 한다.
예를 들면:
```
    Preprocessor pp { /* ... */ };
    Parser p { pp, /* ... */ };
    Type_checker tc { p, /* ... */ };
```
`p`는 `pp`를 참조하지만, 소유하고 있지 않다.

##### 시행하기

* (쉬움) 클래스가 소유자인 포인터나 참조 멤버 변수를 갖고 있다면 (가령, `gsl::owner`를 사용하여 소유하는 경우), 소멸자에서 참조되는 것이 좋다.
* (어려움) 소유권에 대해 명시적으로 기술하지 않은 경우, 포인터나 참조 멤버 변수들이 소유자 인지 판단하라. (예, 생성자를 보라)



### <a name="Rc-dtor-ptr"></a>C.32: 클래스가 날 포인터(`T*`)나 참조(`T&`)를 갖고 있을 때, 소유하고 있는 것인지 고려해 보라

##### 근거
소유권에 대해서 상세하지 않은 코드는 많이 있다.

##### 예
```
    ???
```
##### 참고 사항
`T*` 혹은 `T&` 가 소유를 의미한다면, **소유한다는** 표시를 하라. `T*` 에 소유의 의미가 없다면 `ptr` 로 표시하는 것을 고려하라.
이것은 문서화와 분석에 도움이 될 것이다.

##### 시행하기
저수준 일반 포인터나 멤버 참조의 초기화를 살펴보고, 할당이 사용되는지 보라.

Look at the initialization of raw member pointers and member references and see if an allocation is used.

### <a name="Rc-dtor-ptr2"></a>C.33: 클래스가 포인터 멤버를 소유하고 있다면, 소멸자를 정의하거나 `=delete` 로 선언하라


##### 근거
소유된 객체는 그것을 소유한 객체가 소멸될 때 `삭제`되어야 한다.

##### 예
포인터 멤버는 리소스일 것이다.
[`T*`는 리소스가 아니어야 한다](#Rr-ptr), 오래된 코드에서는 일반적이다.
`T*` 를 가능한 소유자라고 고려하고, 의심해보라.
```
    template<typename T>
    class Smart_ptr {
        T* p;   // BAD: vague about ownership of *p
        // ...
    public:
        // ... no user-defined default operations ...
    };

    void use(Smart_ptr<int> p1)
    {
        // error: p2.p leaked (if not nullptr and not owned by some other code)
        auto p2 = p1;
    }
```
소멸자를 정의 한다면, [모든 기본 연산들](#Rc-five)을 정의하거나 삭제해야 한다.
```
    template<typename T>
    class Smart_ptr2 {
        T* p;   // BAD: vague about ownership of *p
        // ...
    public:
        // ... no user-defined copy operations ...
        ~Smart_ptr2() { delete p; }  // p is an owner!
    };

    void use(Smart_ptr<int> p1)
    {
        auto p2 = p1;   // error: double deletion
    }
```
기본 복사 연산은 단지 `p1.p` 를 `p2.p` 로 복사하고, `p1.p` 가 두번 소멸되게 만들 것이다. 소유권을 명시하라:
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

    void use(Smart_ptr3<int> p1)
    {
        auto p2 = p1;   // error: double deletion
    }
```
##### 참고 사항
보통 소멸자를 사용하는 가장 단순한 방법은 포인터를 스마트 포인터(가령, `std::unique_ptr`)로 교체하고, 컴파일러가
적절한 소멸자를 암묵적으로 호출하게 만들도록 놔두는 것이다.

##### 참고 사항
왜 소유하고 있는 모든 포인터를 "스마트 포인터"로 사용하도록 하지 않는가?
가끔 중요하지 않은 코드 변경을 만들게 되고, ABI 에 영향을 줄 수 있다.

##### 시행하기
* 포인터 데이터 맴버를 갖는 클래스를 의심하라.
* `owner<T>` 를 갖는 클래스는 기본 연산들을 정의 해야한다.


### <a name="Rc-dtor-ref"></a>C.34: 클래스가 참조 멤버를 소유하고 있다면, 소멸자를 정의하거나 `=delete` 로 선언하라

##### 근거
참조 멤버는 클래스가 소유한 리소스일 수도 있다.
그래서는 안되지만, 오래된 코드에선 흔히 발견된다.
[포인터 멤버들과 소멸자](#Rc-dtor-ptr)를 보라.
또, 복사가 복사 손실로 이어질 수도 있다.

##### 잘못된 예
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
`Handle` 이 `Shape` 를 소멸할 책임이 있는지에 대한 문제는 [the pointer case](#Rc-dtor-ptr)과 동일하다.
만약 `Handle` 이 `s` 로 참조되는 객체를 소유한다면, 소멸자가 있어야 한다.

##### 예
```
    class Handle {        // OK
        // use reference rather than pointer to prevent rebinding
        owner<Shape&> s;  
        // ...
    public:
        Handle(Shape& ss) : s{ss} { /* ... */ }
        ~Handle() { delete &s; }
        // ...
    };
```
`Handle` 이 `Shape` 를 소유하는지와는 별개로, 기본 복사 동작에 대해 의심해야 한다:
```
    // the Handle had better own the Circle or we have a leak
    Handle x { *new Circle{p1, 17} };

    Handle y { *new Triangle{p1, p2, p3} };
    
    // the default assignment will try *x.s = *y.s
    x = y;     
```
코드에 사용된 `x = y` 는 굉장히 의심스럽다.
`Triangle` 을 `Circle` 에 할당한다?
`Shape` 에 [복사 대입을 `=deleted`](#Rc-copy-virtual) 하지 않았다면, `Triangle` 의 `Shape` 부분만 `Circle` 로 복사된다.

##### 참고 사항
모든 소유하는 참조는 "스마트 포인터"로 대체하도록 하는 것은 어떤가?
참조를 스마트 포인터로 변경하는 것은 코드 변경을 의미한다.
아직 스마트 참조는 없다. ABI 에 영향을 줄 수도 있다.

##### 시행하기
* 참조 데이터 멤버를 갖는 클래스는 의심스럽다.
* `owner<T>` 참조를 갖는 클래스는 기본 연산들을 정의하는 것이 좋다.


### <a name="Rc-dtor-virtual"></a>C.35: 기본 클래스의 소멸자는 `public` `virtual` 이거나, `protected` non-`virtual`이어야 한다.

##### 근거

To prevent undefined behavior.
If the destructor is public, then calling code can attempt to destroy a derived class object through a base class pointer, and the result is undefined if the base class's destructor is non-virtual.
If the destructor is protected, then calling code cannot destroy through a base class pointer and the destructor does not need to be virtual; it does need to be protected, not private, so that derived destructors can invoke it.
In general, the writer of a base class does not know the appropriate action to be done upon destruction.

##### Discussion

See [this in the Discussion section](#Sd-dtor).

##### 잘못된 예
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
##### 참고 사항

A virtual function defines an interface to derived classes that can be used without looking at the derived classes.
If the interface allows destroying, it should be safe to do so.

##### 참고 사항

A destructor must be nonprivate or it will prevent using the type :
```
    class X {
        ~X();   // private destructor
        // ...
    };

    void use()
    {
        X a;                        // error: cannot destroy
        auto p = make_unique<X>();  // error: cannot destroy
    }
```
##### 예외 사항

We can imagine one case where you could want a protected virtual destructor: When an object of a derived type (and only of such a type) should be allowed to destroy *another* object (not itself) through a pointer to base. We haven't seen such a case in practice, though.

##### 시행하기

* A class with any virtual functions should have a destructor that is either public and virtual or else protected and nonvirtual.

### <a name="Rc-dtor-fail"></a>C.36: 소멸자는 실패해선 안된다

##### 근거

In general we do not know how to write error-free code if a destructor should fail.
The standard library requires that all classes it deals with have destructors that do not exit by throwing.

##### 예
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
##### 참고 사항

Many have tried to devise a fool-proof scheme for dealing with failure in destructors.
None have succeeded to come up with a general scheme.
This can be a real practical problem: For example, what about a socket that won't close?
The writer of a destructor does not know why the destructor is called and cannot "refuse to act" by throwing an exception.
See [discussion](#Sd-dtor).
To make the problem worse, many "close/release" operations are not retryable.
If at all possible, consider failure to close/cleanup a fundamental design error and terminate.

##### 참고 사항

Declare a destructor `noexcept`. That will ensure that it either completes normally or terminate the program.

##### 참고 사항

If a resource cannot be released and the program may not fail, try to signal the failure to the rest of the system somehow
(maybe even by modifying some global state and hope something will notice and be able to take care of the problem).
Be fully aware that this technique is special-purpose and error-prone.
Consider the "my connection will not close" example.
Probably there is a problem at the other end of the connection and only a piece of code responsible for both ends of the connection can properly handle the problem.
The destructor could send a message (somehow) to the responsible part of the system, consider that to have closed the connection, and return normally.

##### 참고 사항

If a destructor uses operations that may fail, it can catch exceptions and in some cases still complete successfully
(e.g., by using a different clean-up mechanism from the one that threw an exception).

##### 시행하기

(쉬움) A destructor should be declared `noexcept`.

### <a name="Rc-dtor-noexcept"></a>C.37: 소멸자를 `noexcept`로 작성하라

##### 근거

 [A destructor may not fail](#Rc-dtor-fail). If a destructor tries to exit with an exception, it's a bad design error and the program had better terminate.

##### 참고 사항

A destructor (either user-defined or compiler-generated) is implicitly declared `noexcept` (independently of what code is in its body) if all of the members of its class have `noexcept` destructors.

##### 시행하기

(쉬움) A destructor should be declared `noexcept`.

## <a name="SS-ctor"></a>C.ctor: Constructors

A constructor defines how an object is initialized (constructed).

### <a name="Rc-ctor"></a>C.40: Define a constructor if a class has an invariant

##### 근거

That's what constructors are for.

##### 예
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

##### 참고 사항

A constructor can be used for convenience even if a class does not have an invariant. For example:
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
##### 참고 사항

The C++11 initializer list rule eliminates the need for many constructors. For example:
```
    struct Rec2{
        string s;
        int i;
        Rec2(const string& ss, int ii = 0) :s{ss}, i{ii} {}   // redundant
    };

    Rec r1 {"Foo", 7};
    Rec r2 {"Bar"};
```
The `Rec2` constructor is redundant.
Also, the default for `int` would be better done as a [member initializer](#Rc-in-class-initializer).

##### 함께 보기
[construct valid object](#Rc-complete) and [constructor throws](#Rc-throw).

##### 시행하기

* Flag classes with user-defined copy operations but no constructor (a user-defined copy is a good indicator that the class has an invariant)



### <a name="Rc-complete"></a>C.41: A constructor should create a fully initialized object

##### 근거

A constructor establishes the invariant for a class. A user of a class should be able to assume that a constructed object is usable.

##### 잘못된 예
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

##### 예외 사항
If a valid object cannot conveniently be constructed by a constructor [use a factory function](#Rc-factory).

##### 참고 사항

If a constructor acquires a resource (to create a valid object), that resource should be [released by the destructor](#Rc-dtor-release).
The idiom of having constructors acquire resources and destructors release them is called [RAII](#Rr-raii) ("Resource Acquisition Is Initialization").

### <a name="Rc-throw"></a>C.42: If a constructor cannot construct a valid object, throw an exception

##### 근거

Leaving behind an invalid object is asking for trouble.

##### 예
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

        void read();      // read from f
        // ...
    };

    void f()
    {
        X2 file {"Zeno"}; // throws if file isn't open
        file.read();      // fine
        // ...
    }
```
##### 잘못된 예
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

        void is_valid() { return valid; }
        void read();   // read from f
        // ...
    };

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
##### 참고 사항

For a variable definition (e.g., on the stack or as a member of another object) there is no explicit function call from which an error code could be returned.
Leaving behind an invalid object and relying on users to consistently check an `is_valid()` function before use is tedious, error-prone, and inefficient.

##### 예외 사항
There are domains, such as some hard-real-time systems (think airplane controls) where (without additional tool support) exception handling is not sufficiently predictable from a timing perspective.
There the `is_valid()` technique must be used. In such cases, check `is_valid()` consistently and immediately to simulate [RAII](#Rr-raii).

##### 대안
If you feel tempted to use some "post-constructor initialization" or "two-stage initialization" idiom, try not to do that.
If you really have to, look at [factory functions](#Rc-factory).

##### 참고 사항

One reason people have used `init()` functions rather than doing the initialization work in a constructor has been to avoid code replication.
[Delegating constructors](#Rc-delegating) and [default member initialization](#Rc-in-class-initializer) do that better.
Another reason is been to delay initialization until an object is needed; the solution to that is often [not to declare a variable until it can be properly initialized](#Res-init)

##### 시행하기

* (쉬움) Every constructor should initialize every member variable (either explicitly, via a delegating ctor call or via default construction).
* (Unknown) If a constructor has an `Ensures` contract, try to see if it holds as a postcondition.

### <a name="Rc-default0"></a>C.43: Ensure that a class has a default constructor

##### 근거

Many language and library facilities rely on default constructors to initialize their elements, e.g. `T a[10]` and `std::vector<T> v(10)`.

##### 예 , bad
```
    class Date { // BAD: no default constructor
    public:
        Date(int dd, int mm, int yyyy);
        // ...
    };

    vector<Date> vd1(1000);   // default Date needed here
    vector<Date> vd2(1000, Date{Month::october, 7, 1885});   // alternative
```
The default constructor is only auto-generated if there is no user-declared constructor, hence it's impossible to initialize the vector `vd1` in the example above.

There is no "natural" default date (the big bang is too far back in time to be useful for most people), so this example is non-trivial.
`{0, 0, 0}` is not a valid date in most calendar systems, so choosing that would be introducing something like floating-point's `NaN`.
However, most realistic `Date` classes have a "first date" (e.g. January 1, 1970 is popular), so making that the default is usually trivial.

##### 예
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

    vector<Date> vd1(1000);
```
##### 참고 사항

A class with members that all have default constructors implicitly gets a default constructor:
```
    struct X {
        string s;
        vector v;
    };

    X x; // means X{{}, {}}; that is the empty string and the empty vector
```
Beware that built-in types are not properly default constructed:
```
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
##### 시행하기

* Flag classes without a default constructor

### <a name="Rc-default00"></a>C.44: Prefer default constructors to be simple and non-throwing

##### 근거

Being able to set a value to "the default" without operations that might fail simplifies error handling and reasoning about move operations.

##### 예, problematic
```
    template<typename T>
    // elem points to space-elem element allocated using new
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
This is nice and general, but setting a `Vector0` to empty after an error involves an allocation, which may fail.
Also, having a default `Vector` represented as `{new T[0], 0, 0}` seems wasteful.
For example, `Vector0 v(100)` costs 100 allocations.

##### 예
```
    template<typename T>
    // elem is nullptr or elem points to space-elem element allocated using new
    class Vector1 {
    public:
        // sets the representation to {nullptr, nullptr, nullptr}; doesn't throw
        Vector1() noexcept {}
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

##### 시행하기

* Flag throwing default constructors

### <a name="Rc-default"></a>C.45: Don't define a default constructor that only initializes data members; use in-class member initializers instead

##### 근거

Using in-class member initializers lets the compiler generate the function for you. The compiler-generated function can be more efficient.

##### 잘못된 예
```
    class X1 { // BAD: doesn't use member initializers
        string s;
        int i;
    public:
        X1() :s{"default"}, i{1} { }
        // ...
    };
```
##### 예
```
    class X2 {
        string s = "default";
        int i = 1;
    public:
        // use compiler-generated default constructor
        // ...
    };
```
##### 시행하기

(쉬움) A default constructor should do more than just initialize member variables with constants.

### <a name="Rc-explicit"></a>C.46: By default, declare single-argument constructors explicit

##### 근거

To avoid unintended conversions.

##### 잘못된 예
```
    class String {
        // ...
    public:
        String(int);   // BAD
        // ...
    };

    String s = 10;   // surprise: string of size 10
```
##### 예외 사항

If you really want an implicit conversion from the constructor argument type to the class type, don't use `explicit`:
```
    class Complex {
        // ...
    public:
        Complex(double d);   // OK: we want a conversion from d to {d, 0}
        // ...
    };

    Complex z = 10.7;   // unsurprising conversion
```
##### 함께 보기
[Discussion of implicit conversions](#Ro-conversion).

##### 시행하기

(쉬움) Single-argument constructors should be declared `explicit`. Good single argument non-`explicit` constructors are rare in most code based. Warn for all that are not on a "positive list".

### <a name="Rc-order"></a>C.47: Define and initialize member variables in the order of member declaration

##### 근거

To minimize confusion and errors. That is the order in which the initialization happens (independent of the order of member initializers).

##### 잘못된 예
```
    class Foo {
        int m1;
        int m2;
    public:
        Foo(int x) :m2{x}, m1{++x} { }   // BAD: misleading initializer order
        // ...
    };

    Foo x(1); // surprise: x.m1 == x.m2 == 2
```
##### 시행하기

(쉬움) A member initializer list should mention the members in the same order they are declared.

##### 함께 보기
[Discussion](#Sd-order)

### <a name="Rc-in-class-initializer"></a>C.48: Prefer in-class initializers to member initializers in constructors for constant initializers

##### 근거

Makes it explicit that the same value is expected to be used in all constructors. Avoids repetition. Avoids maintenance problems. It leads to the shortest and most efficient code.

##### 잘못된 예
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

##### 예
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
##### 대안
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
##### 시행하기

* (쉬움) Every constructor should initialize every member variable (either explicitly, via a delegating ctor call or via default construction).
* (쉬움) Default arguments to constructors suggest an in-class initializer may be more appropriate.

### <a name="Rc-initialize"></a>C.49: Prefer initialization to assignment in constructors

##### 근거

An initialization explicitly states that initialization, rather than assignment, is done and can be more elegant and efficient. Prevents "use before set" errors.

##### 예, good
```
    class A {   // Good
        string s1;
    public:
        A() : s1{"Hello, "} { }    // GOOD: directly construct
        // ...
    };
```
##### 잘못된 예
```
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

##### 근거

If the state of a base class object must depend on the state of a derived part of the object, we need to use a virtual function (or equivalent) while minimizing the window of opportunity to misuse an imperfectly constructed object.

##### 잘못된 예
```
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
##### 예
```
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
        static shared_ptr<T> Create()  // interface for creating objects
        {
            auto p = make_shared<T>();
            p->PostInitialize();
            return p;
        }
    };

    class D : public B { /* ... */ };            // some derived class

    shared_ptr<D> p = D::Create<D>();  // creating a D object
```
By making the constructor `protected` we avoid an incompletely constructed object escaping into the wild.
By providing the factory function `Create()`, we make construction (on the free store) convenient.

##### 참고 사항

Conventional factory functions allocate on the free store, rather than on the stack or in an enclosing object.

##### 함께 보기
[Discussion](#Sd-factory)

### <a name="Rc-delegating"></a>C.51: Use delegating constructors to represent common actions for all constructors of a class

##### 근거

To avoid repetition and accidental differences.

##### 잘못된 예
```
    class Date {   // BAD: repetitive
        int d;
        Month m;
        int y;
    public:
        Date(int ii, Month mm, year yy)
            :i{ii}, m{mm} y{yy}
            { if (!valid(i, m, y)) throw Bad_date{}; }

        Date(int ii, Month mm)
            :i{ii}, m{mm} y{current_year()}
            { if (!valid(i, m, y)) throw Bad_date{}; }
        // ...
    };
```
The common action gets tedious to write and may accidentally not be common.

##### 예
```
    class Date2 {
        int d;
        Month m;
        int y;
    public:
        Date2(int ii, Month mm, year yy)
            :i{ii}, m{mm} y{yy}
            { if (!valid(i, m, y)) throw Bad_date{}; }

        Date2(int ii, Month mm)
            :Date2{ii, mm, current_year()} {}
        // ...
    };
```
##### 함께 보기
If the "repeated action" is a simple initialization, consider [an in-class member initializer](#Rc-in-class-initializer).

##### 시행하기

(중간) Look for similar constructor bodies.

### <a name="Rc-inheriting"></a>C.52: Use inheriting constructors to import constructors into a derived class that does not need further explicit initialization

##### 근거

If you need those constructors for a derived class, re-implementing them is tedious and error prone.

##### 예

`std::vector` has a lot of tricky constructors, so if I want my own `vector`, I don't want to reimplement them:
```
    class Rec {
        // ... data and lots of nice constructors ...
    };

    class Oper : public Rec {
        using Rec::Rec;
        // ... no data members ...
        // ... lots of nice utility functions ...
    };
```
##### 잘못된 예
```
    struct Rec2 : public Rec {
        int x;
        using Rec::Rec;
    };

    Rec2 r {"foo", 7};
    int val = r.x;   // uninitialized
```
##### 시행하기

Make sure that every member of the derived class is initialized.

## <a name="SS-copy"></a>C.copy: Copy and move

Value types should generally be copyable, but interfaces in a class hierarchy should not.
Resource handles may or may not be copyable.
Types can be defined to move for logical as well as performance reasons.

### <a name="Rc-copy-assignment"></a>C.60: Make copy assignment non-`virtual`, take the parameter by `const&`, and return by non-`const&`

##### 근거

It is simple and efficient. If you want to optimize for rvalues, provide an overload that takes a `&&` (see [F.24](#Rf-pass-ref-ref)).

##### 예
```
    class Foo {
    public:
        Foo& operator=(const Foo& x)
        {
            // GOOD: no need to check for self-assignment (other than performance)
            auto tmp = x;
            std::swap(*this, tmp);
            return *this;
        }
        // ...
    };

    Foo a;
    Foo b;
    Foo f();

    a = b;    // assign lvalue: copy
    a = f();  // assign rvalue: potentially move
```
##### 참고 사항

The `swap` implementation technique offers the [strong guarantee](???).

##### 예

But what if you can get significantly better performance by not making a temporary copy? Consider a simple `Vector` intended for a domain where assignment of large, equal-sized `Vector`s is common. In this case, the copy of elements implied by the `swap` implementation technique could cause an order of magnitude increase in cost:
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
        if (a.sz > sz) {
            // ... use the swap technique, it can't be bettered ...
            return *this
        }
        // ... copy sz elements from *a.elem to elem ...
        if (a.sz < sz) {
            // ... destroy the surplus elements in *this* and adjust size ...
        }
        return *this;
    }
```
By writing directly to the target elements, we will get only [the basic guarantee](#???) rather than the strong guarantee offered by the `swap` technique. Beware of [self assignment](#Rc-copy-self).

##### 대안s
If you think you need a `virtual` assignment operator, and understand why that's deeply problematic, don't call it `operator=`. Make it a named function like `virtual void assign(const Foo&)`.
See [copy constructor vs. `clone()`](#Rc-copy-virtual).

##### 시행하기

* (쉬움) An assignment operator should not be virtual. Here be dragons!
* (쉬움) An assignment operator should return `T&` to enable chaining, not alternatives like `const T&` which interfere with composability and putting objects in containers.
* (중간) An assignment operator should (implicitly or explicitly) invoke all base and member assignment operators.
  Look at the destructor to determine if the type has pointer semantics or value semantics.

### <a name="Rc-copy-semantic"></a>C.61: A copy operation should copy

##### 근거

That is the generally assumed semantics. After `x=y`, we should have `x == y`.
After a copy `x` and `y` can be independent objects (value semantics, the way non-pointer built-in types and the standard-library types work) or refer to a shared object (pointer semantics, the way pointers work).

##### 예
```
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
        copy(a.p, a.p + sz, a.p);
    }

    X x;
    X y = x;
    if (x != y) throw Bad{};
    x.modify();
    if (x == y) throw Bad{};   // assume value semantics
```
##### 예
```
    class X2 {  // OK: pointer semantics
    public:
        X2();
        X2(const X&) = default; // shallow copy
        ~X2() = default;
        void modify();          // change the value of X
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
##### 참고 사항

Prefer copy semantics unless you are building a "smart pointer". Value semantics is the simplest to reason about and what the standard library facilities expect.

##### 시행하기

(Not enforceable)

### <a name="Rc-copy-self"></a>C.62: Make copy assignment safe for self-assignment

##### 근거

If `x=x` changes the value of `x`, people will be surprised and bad errors will occur (often including leaks).

##### 예

The standard-library containers handle self-assignment elegantly and efficiently:
```
    std::vector<int> v = {3, 1, 4, 1, 5, 9};
    v = v;
    // the value of v is still {3, 1, 4, 1, 5, 9}
```
##### 참고 사항

The default assignment generated from members that handle self-assignment correctly handles self-assignment.
```
    struct Bar {
        vector<pair<int, int>> v;
        map<string, int> m;
        string s;
    };

    Bar b;
    // ...
    b = b;   // correct and efficient
```
##### 참고 사항

You can handle self-assignment by explicitly testing for self-assignment, but often it is faster and more elegant to cope without such a test (e.g., [using `swap`](#Rc-swap)).
```
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
This is obviously safe and apparently efficient.
However, what if we do one self-assignment per million assignments?
That's about a million redundant tests (but since the answer is essentially always the same, the computer's branch predictor will guess right essentially every time).
Consider:
```
    Foo& Foo::operator=(const Foo& a)   // simpler, and probably much better
    {
        s = a.s;
        i = a.i;
        return *this;
    }
```
`std::string` is safe for self-assignment and so are `int`. All the cost is carried by the (rare) case of self-assignment.

##### 시행하기

(쉬움) Assignment operators should not contain the pattern `if (this == &a) return *this;` ???

### <a name="Rc-move-assignment"></a>C.63: Make move assignment non-`virtual`, take the parameter by `&&`, and return by non-`const &`

##### 근거

It is simple and efficient.

##### 함께 보기
[The rule for copy-assignment](#Rc-copy-assignment).

##### 시행하기

Equivalent to what is done for [copy-assignment](#Rc-copy-assignment).

* (쉬움) An assignment operator should not be virtual. Here be dragons!
* (쉬움) An assignment operator should return `T&` to enable chaining, not alternatives like `const T&` which interfere with composability and putting objects in containers.
* (중간) A move assignment operator should (implicitly or explicitly) invoke all base and member move assignment operators.

### <a name="Rc-move-semantic"></a>C.64: A move operation should move and leave its source in valid state

##### 근거

That is the generally assumed semantics. After `x=std::move(y)` the value of `x` should be the value `y` had and `y` should be in a valid state.

##### 예
```
    template<typename T>
    class X {   // OK: value semantics
    public:
        X();
        X(X&& a);          // move X
        void modify();     // change the value of X
        // ...
        ~X() { delete[] p; }
    private:
        T* p;
        int sz;
    };


    X::X(X&& a)
        :p{a.p}, sz{a.sz}  // steal representation
    {
        a.p = nullptr;     // set to "empty"
        a.sz = 0;
    }

    void use()
    {
        X x{};
        // ...
        X y = std::move(x);
        x = X{};   // OK
    } // OK: x can be destroyed
```
##### 참고 사항

Ideally, that moved-from should be the default value of the type. Ensure that unless there is an exceptionally good reason not to. However, not all types have a default value and for some types establishing the default value can be expensive. The standard requires only that the moved-from object can be destroyed.
Often, we can easily and cheaply do better: The standard library assumes that it it possible to assign to a moved-from object. Always leave the moved-from object in some (necessarily specified) valid state.

##### 참고 사항

Unless there is an exceptionally strong reason not to, make `x = std::move(y); y = z;` work with the conventional semantics.

##### 시행하기

(Not enforceable) Look for assignments to members in the move operation. If there is a default constructor, compare those assignments to the initializations in the default constructor.

### <a name="Rc-move-self"></a>C.65: Make move assignment safe for self-assignment

##### 근거

If `x = x` changes the value of `x`, people will be surprised and bad errors may occur. However, people don't usually directly write a self-assignment that turn into a move, but it can occur. However, `std::swap` is implemented using move operations so if you accidentally do `swap(a, b)` where `a` and `b` refer to the same object, failing to handle self-move could be a serious and subtle error.

##### 예
```
    class Foo {
        string s;
        int i;
    public:
        Foo& operator=(Foo&& a);
        // ...
    };

    Foo& Foo::operator=(Foo&& a)       // OK, but there is a cost
    {
        if (this == &a) return *this;  // this line is redundant
        s = std::move(a.s);
        i = a.i;
        return *this;
    }
```
The one-in-a-million argument against `if (this == &a) return *this;` tests from the discussion of [self-assignment](#Rc-copy-self) is even more relevant for self-move.

##### 참고 사항

There is no know general way of avoiding a `if (this == &a) return *this;` test for a move assignment and still get a correct answer (i.e., after `x=x` the value of `x` is unchanged).

##### 참고 사항

The ISO standard guarantees only a "valid but unspecified" state for the standard library containers. Apparently this has not been a problem in about 10 years of experimental and production use. Please contact the editors if you find a counter example. The rule here is more caution and insists on complete safety.

##### 예

Here is a way to move a pointer without a test (imagine it as code in the implementation a move assignment):
```
    // move from other.ptr to this->ptr
    T* temp = other.ptr;
    other.ptr = nullptr;
    delete ptr;
    ptr = temp;
```
##### 시행하기

* (중간) In the case of self-assignment, a move assignment operator should not leave the object holding pointer members that have been `delete`d or set to nullptr.
* (Not enforceable) Look at the use of standard-library container types (incl. `string`) and consider them safe for ordinary (not life-critical) uses.

### <a name="Rc-move-noexcept"></a>C.66: Make move operations `noexcept`

##### 근거

A throwing move violates most people's reasonably assumptions.
A non-throwing move will be used more efficiently by standard-library and language facilities.

##### 예
```
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
These copy operations do not throw.

##### 잘못된 예
```
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
This `Vector2` is not just inefficient, but since a vector copy requires allocation, it can throw.

##### 시행하기

(쉬움) A move operation should be marked `noexcept`.

### <a name="Rc-copy-virtual"></a>C.67: A base class should suppress copying, and provide a virtual `clone` instead if "copying" is desired

##### 근거

To prevent slicing, because the normal copy operations will copy only the base portion of a derived object.

##### 잘못된 예
```
    class B { // BAD: base class doesn't suppress copying
        int data;
        // ... nothing about copy operations, so uses default ...
    };

    class D : public B {
        string moredata; // add a data member
        // ...
    };

    auto d = make_unique<D>();

    // oops, slices the object; gets only d.data but drops d.moredata
    auto b = make_unique<B>(d);
```
##### 예
```
    class B { // GOOD: base class suppresses copying
        B(const B&) = delete;
        B& operator=(const B&) = delete;
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
```
##### 참고 사항

It's good to return a smart pointer, but unlike with raw pointers the return type cannot be covariant (for example, `D::clone` can't return a `unique_ptr<D>`. Don't let this tempt you into returning an owning raw pointer; this is a minor drawback compared to the major robustness benefit delivered by the owning smart pointer.

##### 예외 사항s

If you need covariant return types, return an `owner<derived*>`. See [C.130](#Rh-copy).

##### 시행하기

A class with any virtual function should not have a copy constructor or copy assignment operator (compiler-generated or handwritten).

## C.other: Other default operation rules

In addition to the operations for which the language offer default implementations,
there are a few operations that are so foundational that it rules for their definition are needed:
comparisons, `swap`, and `hash`.

### <a name="Rc-default"></a>C.80: Use `=default` if you have to be explicit about using the default semantics

##### 근거

The compiler is more likely to get the default semantics right and you cannot implement these function better than the compiler.

##### 예
```
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
Because we defined the destructor, we must define the copy and move operations. The `=default` is the best and simplest way of doing that.

##### 잘못된 예
```
    class Tracer2 {
        string message;
    public:
        Tracer2(const string& m) : message{m} { cerr << "entering " << message << '\n'; }
        ~Tracer2() { cerr << "exiting " << message << '\n'; }

        Tracer2(const Tracer2& a) : message{a.message} {}
        Tracer2& operator=(const Tracer2& a) { message = a.message; }
        Tracer2(Tracer2&& a) :message{a.message} {}
        Tracer2& operator=(Tracer2&& a) { message = a.message; }
    };
```
Writing out the bodies of the copy and move operations is verbose, tedious, and error-prone. A compiler does it better.

##### 시행하기

(중간) The body of a special operation should not have the same accessibility and semantics as the compiler-generated version, because that would be redundant

### <a name="Rc-delete"></a>C.81: Use `=delete` when you want to disable default behavior (without wanting an alternative)

##### 근거

In a few cases, a default operation is not desirable.

##### 예
```
    class Immortal {
    public:
        ~Immortal() = delete;   // do not allow destruction
        // ...
    };

    void use()
    {
        Immortal ugh;   // error: ugh cannot be destroyed
        Immortal* p = new Immortal{};
        delete p;       // error: cannot destroy *p
    }
```
##### 예

A `unique_ptr` can be moved, but not copied. To achieve that its copy operations are deleted. To avoid copying it is necessary to `=delete` its copy operations from lvalues:
```
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

    unique_ptr<int> make();   // make "something" and return it by moving

    void f()
    {
        unique_ptr<int> pi {};
        auto pi2 {pi};      // error: no move constructor from lvalue
        auto pi3 {make()};  // OK, move: the result of make() is an rvalue
    }
```
##### 시행하기

The elimination of a default operation is (should be) based on the desired semantics of the class. Consider such classes suspect, but maintain a "positive list" of classes where a human has asserted that the semantics is correct.

### <a name="Rc-ctor-virtual"></a>C.82: Don't call virtual functions in constructors and destructors

##### 근거

The function called will be that of the object constructed so far, rather than a possibly overriding function in a derived class.
This can be most confusing.
Worse, a direct or indirect call to an unimplemented pure virtual function from a constructor or destructor results in undefined behavior.

##### 잘못된 예
```
    class base {
    public:
        virtual void f() = 0;   // not implemented
        virtual void g();       // implemented with base version
        virtual void h();       // implemented with base version
    };

    class derived : public base {
    public:
        void g() override;   // provide derived implementation
        void h() final;      // provide derived implementation

        derived()
        {
            // BAD: attempt to call an unimplemented virtual function
            f();

            // BAD: will call derived::g, not dispatch further virtually
            g();

            // GOOD: explicitly state intent to call only the visible version
            derived::g();

            // ok, no qualification needed, h is final
            h();
        }
    };
```
Note that calling a specific explicitly qualified function is not a virtual call even if the function is `virtual`.

##### 함께 보기
[factory functions](#Rc-factory) for how to achieve the effect of a call to a derived class function without risking undefined behavior.

##### 참고 사항

There is nothing inherently wrong with calling virtual functions from constructors and destructors.
The semantics of such calls is type safe.
However, experience shows that such calls are rarely needed, easily confuse maintainers, and become a source of errors when used by novices.

##### 시행하기

* Flag calls of virtual functions from constructors and destructors.

### <a name="Rc-swap"></a>C.83: For value-like types, consider providing a `noexcept` swap function

##### 근거

A `swap` can be handy for implementing a number of idioms, from smoothly moving objects around to implementing assignment easily to providing a guaranteed commit function that enables strongly error-safe calling code. Consider using swap to implement copy assignment in terms of copy construction. See also [destructors, deallocation, and swap must never fail](#Re-never-fail).

##### 예, good
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
Providing a nonmember `swap` function in the same namespace as your type for callers' convenience.
```
    void swap(Foo& a, Foo& b)
    {
        a.swap(b);
    }
```
##### 시행하기

* (쉬움) A class without virtual functions should have a `swap` member function declared.
* (쉬움) When a class has a `swap` member function, it should be declared `noexcept`.

### <a name="Rc-swap-fail"></a>C.84: A `swap` function may not fail

##### 근거

 `swap` is widely used in ways that are assumed never to fail and programs cannot easily be written to work correctly in the presence of a failing `swap`. The standard-library containers and algorithms will not work correctly if a swap of an element type fails.

##### 잘못된 예
```
    void swap(My_vector& x, My_vector& y)
    {
        auto tmp = x;   // copy elements
        x = y;
        y = tmp;
    }
```
This is not just slow, but if a memory allocation occurs for the elements in `tmp`, this `swap` may throw and would make STL algorithms fail if used with them.

##### 시행하기

(쉬움) When a class has a `swap` member function, it should be declared `noexcept`.

### <a name="Rc-swap-noexcept"></a>C.85: Make `swap` `noexcept`

##### 근거

 [A `swap` may not fail](#Rc-swap-fail).
If a `swap` tries to exit with an exception, it's a bad design error and the program had better terminate.

##### 시행하기

(쉬움) When a class has a `swap` member function, it should be declared `noexcept`.

### <a name="Rc-eq"></a>C.86: Make `==` symmetric with respect to operand types and `noexcept`

##### 근거

Asymmetric treatment of operands is surprising and a source of errors where conversions are possible.
`==` is a fundamental operations and programmers should be able to use it without fear of failure.

##### 예
```
    class X {
        string name;
        int number;
    };

    bool operator==(const X& a, const X& b) noexcept { return a.name == b.name && a.number == b.number; }
```
##### 잘못된 예
```
    class B {
        string name;
        int number;
        bool operator==(const B& a) const { return name == a.name && number == a.number; }
        // ...
    };
```
`B`'s comparison accepts conversions for its second operand, but not its first.

##### 참고 사항

If a class has a failure state, like `double`'s `NaN`, there is a temptation to make a comparison against the failure state throw.
The alternative is to make two failure states compare equal and any valid state compare false against the failure state.

#### Note

This rule applies to all the usual comparison operators: `!=`, `<`, `<=`, `>`, and `>=`.

##### 시행하기

* Flag an `operator==()` for which the argument types differ; same for other comparison operators: `!=`, `<`, `<=`, `>`, and `>=`.
* Flag member `operator==()`s; same for other comparison operators: `!=`, `<`, `<=`, `>`, and `>=`.

### <a name="Rc-eq-base"></a>C.87: Beware of `==` on base classes

##### 근거

It is really hard to write a foolproof and useful `==` for a hierarchy.

##### 잘못된 예
```
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
`B`'s comparison accepts conversions for its second operand, but not its first.
```
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
Of course there are ways of making `==` work in a hierarchy, but the naive approaches do not scale

#### Note

This rule applies to all the usual comparison operators: `!=`, `<`, `<=`, `>`, and `>=`.

##### 시행하기

* Flag a virtual `operator==()`; same for other comparison operators: `!=`, `<`, `<=`, `>`, and `>=`.

### <a name="Rc-hash"></a>C.89: Make a `hash` `noexcept`

##### 근거

Users of hashed containers use hash indirectly and don't expect simple access to throw.
It's a standard-library requirement.

##### 잘못된 예
```
    template<>
    struct hash<My_type> {  // thoroughly bad hash specialization
        using result_type = size_t;
        using argument_type = My_type;

        size_t operator() (const My_type & x) const
        {
            size_t xs = x.s.size();
            if (xs < 4) throw Bad_My_type{};    // "Nobody expects the Spanish inquisition!"
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
If you have to define a `hash` specialization, try simply to let it combine standard-library `hash` specializations with `^` (xor).
That tends to work better than "cleverness" for non-specialists.

##### 시행하기

* Flag throwing `hash`es.

## <a name="SS-containers"></a>C.con: Containers and other resource handles
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#SS-containers)

A container is an object holding a sequence of objects of some type; `std::vector` is the archetypical container.
A resource handle is a class that owns a resource; `std::vector` is the typical resource handle; its resource is its sequence of elements.


컨테이너 규칙 요약

* [C.100: 컨테이너를 정의할때는 STL을 따르라](#Rcon-stl)
* [C.101: 값 문맥을 적용하라](#Rcon-val)
* [C.102: move 연산을 제공하라](#Rcon-move)
* [C.103: 초기화 리스트 생성자를 지원하라](#Rcon-init)
* [C.104: 공백 값으로 설정하는 기본 생성자를 지원하라](#Rcon-empty)
* [C.105: 생성자와 '확장' 생성자를 지원하라](#Rcon-val)
* ???
* [C.109: 리소스 핸들이 포인터 문맥을 따를 경우에는, `*` 과 `->` 연산자를 제공하라](#rcon-ptr)

### 같이 보기
[Resources](#S-resource)


## <a name="SS-lambdas"></a>C.lambdas: 함수 객체와 람다 표현식
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#SS-lambdas)


A function object is an object supplying an overloaded `()` so that you can call it.
A lambda expression (colloquially often shortened to "a lambda") is a notation for generating a function object.
Function objects should be cheap to copy (and therefore [passed by value](#Rf-in)).

요약:

* [F.50: Use a lambda when a function won't do (to capture local variables, or to write a local function)](#Rf-capture-vs-overload)
* [F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms](#Rf-reference-capture)
* [F.53: Avoid capturing by reference in lambdas that will be used nonlocally, including returned, stored on the heap, or passed to another thread](#Rf-value-capture)
* [ES.28: Use lambdas for complex initialization, especially of `const` variables](#Res-lambda-init)



## <a name="SS-hier"></a>C.hier:  클래스 계층 구조 (OOP)
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#SS-hier)

A class hierarchy is constructed to represent a set of hierarchically organized concepts (only).
Typically base classes act as interfaces.
There are two major uses for hierarchies, often named implementation inheritance and interface inheritance.

Class hierarchy rule summary:

* [C.120: Use class hierarchies to represent concepts with inherent hierarchical structure](#Rh-domain)
* [C.121: If a base class is used as an interface, make it a pure abstract class](#Rh-abstract)
* [C.122: Use abstract classes as interfaces when complete separation of interface and implementation is needed](#Rh-separation)

Designing rules for classes in a hierarchy summary:

* [C.126: An abstract class typically doesn't need a constructor](#Rh-abstract-ctor)
* [C.127: A class with a virtual function should have a virtual or protected destructor](#Rh-dtor)
* [C.128: Use `override` to make overriding explicit in large class hierarchies](#Rh-override)
* [C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance](#Rh-kind)
* [C.130: Redefine or prohibit copying for a base class; prefer a virtual `clone` function instead](#Rh-copy)
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
* [C.147: Use `dynamic_cast` to a reference type when failure to find the required class is considered an error](#Rh-ptr-cast)
* [C.148: Use `dynamic_cast` to a pointer type when failure to find the required class is considered a valid alternative](#Rh-ref-cast)
* [C.149: Use `unique_ptr` or `shared_ptr` to avoid forgetting to `delete` objects created using `new`](#Rh-smart)
* [C.150: Use `make_unique()` to construct objects owned by `unique_ptr`s](#Rh-make_unique)
* [C.151: Use `make_shared()` to construct objects owned by `shared_ptr`s](#Rh-make_shared)
* [C.152: Never assign a pointer to an array of derived class objects to a pointer to its base](#Rh-array)

### <a name="Rh-domain"></a>C.120: Use class hierarchies to represent concepts with inherent hierarchical structure (only)

##### 근거

Direct representation of ideas in code eases comprehension and maintenance. Make sure the idea represented in the base class exactly matches all derived types and there is not a better way to express it than using the tight coupling of inheritance.

Do *not* use inheritance when simply having a data member will do. Usually this means that the derived type needs to override a base virtual function or needs access to a protected member.

##### 예
```
    ??? Good old Shape example?
```
##### 잘못된 예

Do *not* represent non-hierarchical domain concepts as class hierarchies.
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
Here most overriding classes cannot implement most of the functions required in the interface well.
Thus the base class becomes an implementation burden.
Furthermore, the user of `Container` cannot rely on the member functions actually performing a meaningful operations reasonably efficiently;
it may throw an exception instead.
Thus users have to resort to run-time checking and/or
not using this (over)general interface in favor of a particular interface found by a run-time type inquiry (e.g., a `dynamic_cast`).

##### 시행하기

* Look for classes with lots of members that do nothing but throw.
* Flag every use of a nonpublic base class `B` where the derived class `D` does not override a virtual function or access a protected member in `B`, and `B` is not one of the following: empty, a template parameter or parameter pack of `D`, a class template specialized with `D`.

### <a name="Rh-abstract"></a>C.121: If a base class is used as an interface, make it a pure abstract class

##### 근거

A class is more stable (less brittle) if it does not contain data.
Interfaces should normally be composed entirely of public pure virtual functions and a default/empty virtual destructor.

##### 예
```
    class my_interface {
    public:
        // ...only pure virtual functions here ...
        virtual ~my_interface() {}   // or =default
    };
```
##### 잘못된 예
```
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


##### 시행하기

* Warn on any class that contains data members and also has an overridable (non-`final`) virtual function.

### <a name="Rh-separation"></a>C.122: Use abstract classes as interfaces when complete separation of interface and implementation is needed

##### 근거

Such as on an ABI (link) boundary.

##### 예
```
    struct Device {
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
Furthermore, we can update `D1` and `D2` in a ways that are not binarily compatible with older versions as long as all access goes through `Device`.

##### 시행하기
```
    ???
```

## C.hierclass: Designing classes in a hierarchy:

### <a name="Rh-abstract-ctor"></a>C.126: An abstract class typically doesn't need a constructor

##### 근거

An abstract class typically does not have any data for a constructor to initialize.

##### 예
```
    ???
```
##### 예외 사항s

* A base class constructor that does work, such as registering an object somewhere, may need a constructor.
* In extremely rare cases, you might find it reasonable for an abstract class to have a bit of data shared by all derived classes
  (e.g., use statistics data, debug information, etc.); such classes tend to have constructors. But be warned: Such classes also tend to be prone to requiring virtual inheritance.

##### 시행하기

Flag abstract classes with constructors.

### <a name="Rh-dtor"></a>C.127: A class with a virtual function should have a virtual or protected destructor

##### 근거

A class with a virtual function is usually (and in general) used via a pointer to base. Usually, the last user has to call delete on a pointer to base, often via a smart pointer to base, so the destructor should be public and virtual. Less commonly, if deletion through a pointer to base is not intended to be supported, the destructor should be protected and nonvirtual; see [C.35](#Rc-dtor-virtual).

##### 잘못된 예
```
    struct B {
        virtual int f() = 0;
        // ... no user-written destructor, defaults to public nonvirtual ...
    };

    // bad: class with a resource derived from a class without a virtual destructor
    struct D : B {
        string s {"default"};
    };

    void use()
    {
        auto p = make_unique<D>();
        // ...
    } // calls B::~B only, leaks the string
```
##### 참고 사항

There are people who don't follow this rule because they plan to use a class only through a `shared_ptr`: `std::shared_ptr<B> p = std::make_shared<D>(args);` Here, the shared pointer will take care of deletion, so no leak will occur from an inappropriate `delete` of the base. People who do this consistently can get a false positive, but the rule is important -- what if one was allocated using `make_unique`? It's not safe unless the author of `B` ensures that it can never be misused, such as by making all constructors private and providing a factory function to enforce the allocation with `make_shared`.

##### 시행하기

* A class with any virtual functions should have a destructor that is either public and virtual or else protected and nonvirtual.
* Flag `delete` of a class with a virtual function but no virtual destructor.

### <a name="Rh-override"></a>C.128: Virtual functions should specify exactly one of `virtual`, `override`, or `final`

##### 근거

Readability. Detection of mistakes. Writing explicit `virtual`, `override`, or `final` is self-documenting and enables the compiler to catch mismatch of types and/or names between base and derived classes. However, writing more than one of these three is both redundant and a potential source of errors.

Use `virtual` only when declaring a new virtual function. Use `override` only when declaring an overrider. Use `final` only when declaring an final overrider.

##### 잘못된 예
```
    struct B {
        void f1(int);
        virtual void f2(int) const;
        virtual void f3(int);
        // ...
    };

    struct D : B {
        void f1(int);        // warn: D::f1() hides B::f1()
        void f2(int) const;  // warn: no explicit override
        void f3(double);     // warn: D::f3() hides B::f3()
        // ...
    };

    struct D2 : B {
        virtual void f2(int) final;  // BAD; pitfall, D2::f does not override B::f
    };
```
##### 시행하기

* Compare names in base and derived classes and flag uses of the same name that does not override.
* Flag overrides with neither `override` nor `final`.
* Flag function declarations that use more than one of `virtual`, `override`, and `final`.

### <a name="Rh-kind"></a>C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance

##### 근거

 ??? Herb: I've become a non-fan of implementation inheritance -- seems most often an anti-pattern. Are there reasonable examples of it?

##### 예
```
    ???
```
##### 시행하기

???

### <a name="Rh-copy"></a>C.130: Redefine or prohibit copying for a base class; prefer a virtual `clone` function instead

##### 근거

Copying a base is usually slicing. If you really need copy semantics, copy deeply: Provide a virtual `clone` function that will copy the actual most-derived type and return an owning pointer to the new object, and then in derived classes return the derived type (use a covariant return type).

##### 예
```
    class base {
    public:
        virtual owner<base*> clone() = 0;
        virtual ~base() = 0;

        base(const base&) = delete;
        base& operator=(const base&) = delete;
    };

    class derived : public base {
    public:
        owner<derived*> clone() override;
        virtual ~derived() override;
    };
```
Note that because of language rules, the covariant return type cannot be a smart pointer. See also [C.67](#Rc-copy-virtual).

##### 시행하기

* Flag a class with a virtual function and a non-user-defined copy operation.
* Flag an assignment of base class objects (objects of a class from which another has been derived).

### <a name="Rh-get"></a>C.131: Avoid trivial getters and setters

##### 근거

A trivial getter or setter adds no semantic value; the data item could just as well be `public`.

##### 예
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
Consider making such a class a `struct` -- that is, a behaviorless bunch of variables, all public data and no member functions.
```
    struct point {
        int x = 0;
        int y = 0;
    };
```
##### 참고 사항

A getter or a setter that converts from an internal type to an interface type is not trivial (it provides a form of information hiding).

##### 시행하기

Flag multiple `get` and `set` member functions that simply access a member without additional semantics.

### <a name="Rh-virtual"></a>C.132: Don't make a function `virtual` without reason

##### 근거

Redundant `virtual` increases run-time and object-code size.
A virtual function can be overridden and is thus open to mistakes in a derived class.
A virtual function ensures code replication in a templated hierarchy.

##### 잘못된 예
```
    template<class T>
    class Vector {
    public:
        // ...
        virtual int size() const { return sz; }   // bad: what good could a derived class do?
    private:
        T* elem;   // the elements
        int sz;    // number of elements
    };
```
This kind of "vector" isn't meant to be used as a base class at all.

##### 시행하기

* Flag a class with virtual functions but no derived classes.
* Flag a class where all member functions are virtual and have implementations.

### <a name="Rh-protected"></a>C.133: Avoid `protected` data

##### 근거

`protected` data is a source of complexity and errors.
`protected` data complicated the statement of invariants.
`protected` data inherently violates the guidance against putting data in base classes, which usually leads to having to deal virtual inheritance as well.

##### 예
```
    ???
```
##### 참고 사항

Protected member function can be just fine.

##### 시행하기

Flag classes with `protected` data.

### <a name="Rh-public"></a>C.134: Ensure all non-`const` data members have the same access level

##### 근거

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

##### 예외 사항s

Occasionally classes will mix A and B, usually for debug reasons. An encapsulated object may contain something like non-`const` debug instrumentation that isn't part of the invariant and so falls into category A -- it isn't really part of the object's value or meaningful observable state either. In that case, the A parts should be treated as A's (made `public`, or in rarer cases `protected` if they should be visible only to derived classes) and the B parts should still be treated like B's (`private` or `const`).

##### 시행하기

Flag any class that has non-`const` data members with different access levels.

### <a name="Rh-mi-interface"></a>C.135: Use multiple inheritance to represent multiple distinct interfaces

##### 근거

Not all classes will necessarily support all interfaces, and not all callers will necessarily want to deal with all operations. Especially to break apart monolithic interfaces into "aspects" of behavior supported by a given derived class.

##### 예
```
    ???
```
##### 참고 사항

This is a very common use of inheritance because the need for multiple different interfaces to an implementation is common
and such interfaces are often not easily or naturally organized into a single-rooted hierarchy.

##### 참고 사항
Such interfaces are typically abstract classes.

##### 시행하기
???

### <a name="Rh-mi-implementation"></a>C.136: Use multiple inheritance to represent the union of implementation attributes

##### 근거

 ??? Herb: Here's the second mention of implementation inheritance. I'm very skeptical, even of single implementation inheritance, never mind multiple implementation inheritance which just seems frightening -- I don't think that even policy-based design really needs to inherit from the policy types. Am I missing some good examples, or could we consider discouraging this as an anti-pattern?

##### 예
```
    ???
```
##### 참고 사항

This a relatively rare use because implementation can often be organized into a single-rooted hierarchy.

##### 시행하기

??? Herb: How about opposite enforcement: Flag any type that inherits from more than one non-empty base class?


### <a name="Rh-vbase"></a>C.137: Use `virtual` bases to avoid overly general base classes

##### 근거
 ???

##### 예
```
    ???
```
##### 참고 사항
???

##### 시행하기
???


### <a name="Rh-using"></a>C.138: Create an overload set for a derived class and its bases with `using`

##### 근거
???

##### 예
```
    ???
```
### <a name="Rh-final"></a>C.139: Use `final` sparingly

##### 근거

Capping a hierarchy with `final` is rarely needed for logical reasons and can be damaging to the extensibility of a hierarchy.
Capping an individual virtual function with `final` is error-prone as that `final` can easily be overlooked when defining/overriding a set of functions.

##### 잘못된 예
```
    class Widget { /* ... */ };

    class My_widget final : public Widget { /* ... */ };    // nobody will ever want to improve My_widget (or so you thought)

    class My_improved_widget : public My_widget { /* ... */ };  // error: can't do that
```
##### 잘못된 예
```
    struct Interface {
        virtual int f() = 0;
        virtual int g() = 0;
    };

    class My_implementation : public Interface {
        int f() override;
        int g() final;  // I want g() to be FAST!
        // ...
    };

    class Better_implementation : public My_implementation {
        int f();
        int g();
        // ...
    };

    void use(Interface* p)
    {
        int x = p->f();    // Better_implementation::f()
        int y = p->g();    // My_implementation::g() Surprise?
    }

    // ...

    use(new Better_interface{});
```
The problem is easy to see in a small example, but in a large hierarchy with many virtual functions, tools are required for reliably spotting such problems.
Consistent use of `override` would catch this.

##### 참고 사항

Claims of performance improvements from `final` should be substantiated.
Too often, such claims are based on conjecture or experience with other languages.

There are examples where `final` can be important for both logical and performance reasons.
One example is a performance-critical AST hierarchy in a compiler or language analysis tool.
New derived classes are not added every year and only by library implementers.
However, misuses are (or at least has been) far more common.

##### 시행하기

Flag uses of `final`.


## <a name="Rh-virtual-default-arg"></a>C.140: Do not provide different default arguments for a virtual function and an overrider

##### 근거

That can cause confusion: An overrider do not inherit default arguments..

##### 잘못된 예
```
    class base {
    public:
        virtual int multiply(int value, int factor = 2) = 0;
    };

    class derived : public base {
    public:
        int multiply(int value, int factor = 10) override;
    };

    derived d;
    base& b = d;

    b.multiply(10);  // these two calls will call the same function but
    d.multiply(10);  // with different arguments and so different results
```
##### 시행하기

Flag default arguments on virtual functions if they differ between base and derived declarations.

## C.hier-access: Accessing objects in a hierarchy

### <a name="Rh-poly"></a>C.145: Access polymorphic objects through pointers and references

##### 근거

If you have a class with a virtual function, you don't (in general) know which class provided the function to be used.

##### 예
```
    struct B { int a; virtual int f(); };
    struct D : B { int b; int f() override; };

    void use(B b)
    {
        D d;
        B b2 = d;   // slice
        B b3 = b;
    }

    void use2()
    {
        D d;
        use(d);   // slice
    }
```
Both `d`s are sliced.

##### 예외 사항

You can safely access a named polymorphic object in the scope of its definition, just don't slice it.
```
    void use3()
    {
        D d;
        d.f();   // OK
    }
```
##### 시행하기

Flag all slicing.

### <a name="Rh-dynamic_cast"></a>C.146: Use `dynamic_cast` where class hierarchy navigation is unavoidable

##### 근거

`dynamic_cast` is checked at run time.

##### 예
```
    struct B {   // an interface
        virtual void f();
        virtual void g();
    };

    struct D : B {   // a wider interface
        void f() override;
        virtual void h();
    };

    void user(B* pb)
    {
        if (D* pd = dynamic_cast<D*>(pb)) {
            // ... use D's interface ...
        }
        else {
            // ... make do with B's interface ...
        }
    }
```
##### 참고 사항

Like other casts, `dynamic_cast` is overused.
[Prefer virtual functions to casting](#???).
Prefer [static polymorphism](#???) to hierarchy navigation where it is possible (no run-time resolution necessary)
and reasonably convenient.

##### 참고 사항

Some people use `dynamic_cast` where a `typeid` would have been more appropriate;
`dynamic_cast` is a general "is kind of" operation for discovering the best interface to an object,
whereas `typeid` is a "give me the exact type of this object" operation to discover the actual type of an object.
The latter is an inherently simpler operation that ought to be faster.
The latter (`typeid`) is easily hand-crafted if necessary (e.g., if working on a system where RTTI is -- for some reason -- prohibited),
the former (`dynamic_cast`) is far harder to implement correctly in general.

Consider:
```
    struct B {
        const char * name {"B"};
        virtual const char* id() const { return name; }
        // ...
    };

    struct D : B {
        const char * name {"D"};
        const char* id() const override { return name; }
        // ...
    };

    void use()
    {
        B* pb1 = new B;
        B* pb2 = new D;

        cout << pb1->id(); // "B"
        cout << pb2->id(); // "D"

        if (pb1->id() == pb2->id()) // *pb1 is the same type as *pb2
        if (pb2 == "D") {         // looks innocent
                D* pd = static_cast<D*>(pb1);
                // ...
        }
        // ...
    }
```
The result of `pb2 == "D"` is actually implementation defined.
We added it to warn of the dangers of home-brew RTTI.
This code may work as expected for years, just to fail on a new machine, new compiler, or a new linker that does not unify character literals.

If you implement your own RTTI, be careful.

##### 예외 사항s

If your implementation provided a really slow `dynamic_cast`, you may have to use a workaround.
However, all workarounds that cannot be statically resolved involve explicit casting (typically `static_cast`) and are error-prone.
You will basically be crafting your own special-purpose `dynamic_cast`.
So, first make sure that your `dynamic_cast` really is as slow as you think it is (there are a fair number of unsupported rumors about)
and that your use of `dynamic_cast` is really performance critical.

We are of the opinion that current implementations of `dynamic_cast` are unnecessarily slow.
For example, under suitable conditions, it is possible to perform a `dynamic_cast` in [fast constant time](http://www.stroustrup.com/fast_dynamic_casting.pdf).
However, compatibility makes changes difficult even if all agree that an effort to optimize is worthwhile.

In very rare cases, if you have measured that the `dynamic_cast` overhead is material, you have other means to statically guarantee that a downcast will succeed (e.g., you are using CRTP carefully), and there is no virtual inheritance involved, consider tactically resorting `static_cast` with a prominent comment and disclaimer summarizing this paragraph and that human attention is needed under maintenance because the type system can't verify correctness. Even so, in our experience such "I know what I'm doing" situations are still a known bug source.

##### 시행하기

Flag all uses of `static_cast` for downcasts, including C-style casts that perform a `static_cast`.

### <a name="Rh-ptr-cast"></a>C.147: Use `dynamic_cast` to a reference type when failure to find the required class is considered an error

##### 근거

Casting to a reference expresses that you intend to end up with a valid object, so the cast must succeed. `dynamic_cast` will then throw if it does not succeed.

##### 예
```
    ???
```
##### 시행하기

???

### <a name="Rh-ref-cast"></a>C.148: Use `dynamic_cast` to a pointer type when failure to find the required class is considered a valid alternative

##### 근거

???

##### 예
```
    ???
```
##### 시행하기

???

### <a name="Rh-smart"></a>C.149: Use `unique_ptr` or `shared_ptr` to avoid forgetting to `delete` objects created using `new`

##### 근거

Avoid resource leaks.

##### 예
```
    void use(int i)
    {
        auto p = new int {7};           // bad: initialize local pointers with new
        auto q = make_unique<int>(9);   // ok: guarantee the release of the memory allocated for 9
        if (0 < i) return;              // maybe return and leak
        delete p;                       // too late
    }
```
##### 시행하기

* Flag initialization of a naked pointer with the result of a `new`
* Flag `delete` of local variable

### <a name="Rh-make_unique"></a>C.150: Use `make_unique()` to construct objects owned by `unique_ptr`s

##### 근거

 `make_unique` gives a more concise statement of the construction.
It also ensures exception safety in complex expressions.

##### 예
```
    unique_ptr<Foo> p {new<Foo>{7}};   // OK: but repetitive

    auto q = make_unique<Foo>(7);      // Better: no repetition of Foo

    // Not exception-safe: the compiler may interleave the computations of arguments as follows:
    //
    // 1. allocate memory for Foo,
    // 2. construct Foo,
    // 3. call bar,
    // 4. construct unique_ptr<Foo>.
    //
    // If bar throws, Foo will not be destroyed, and the memory allocated for it will leak.
    f(unique_ptr<Foo>(new Foo()), bar());

    // Exception-safe: calls to functions are never interleaved.
    f(make_unique<Foo>(), bar());
```
##### 시행하기

* Flag the repetitive usage of template specialization list `<Foo>`
* Flag variables declared to be `unique_ptr<Foo>`

### <a name="Rh-make_shared"></a>C.151: Use `make_shared()` to construct objects owned by `shared_ptr`s

##### 근거

 `make_shared` gives a more concise statement of the construction.
It also gives an opportunity to eliminate a separate allocation for the reference counts, by placing the `shared_ptr`'s use counts next to its object.

##### 예
```
    // OK: but repetitive; and separate allocations for the Foo and shared_ptr's use count
    shared_ptr<Foo> p {new<Foo>{7}};

    auto q = make_shared<Foo>(7);   // Better: no repetition of Foo; one object
```
##### 시행하기

* Flag the repetitive usage of template specialization list`<Foo>`
* Flag variables declared to be `shared_ptr<Foo>`

### <a name="Rh-array"></a>C.152: Never assign a pointer to an array of derived class objects to a pointer to its base

##### 근거

Subscripting the resulting base pointer will lead to invalid object access and probably to memory corruption.

##### 예
```
    struct B { int x; };
    struct D : B { int y; };

    void use(B*);

    D a[] = {{1, 2}, {3, 4}, {5, 6}};
    B* p = a;     // bad: a decays to &a[0] which is converted to a B*
    p[1].x = 7;   // overwrite D[0].y

    use(a);       // bad: a decays to &a[0] which is converted to a B*
```
##### 시행하기

* Flag all combinations of array decay and base to derived conversions.
* Pass an array as a `span` rather than as a pointer, and don't let the array name suffer a derived-to-base conversion before getting into the `span`


## <a name="SS-overload"></a>C.over: 오버로딩
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#SS-overload) 

You can overload ordinary functions, template functions, and operators.
You cannot overload function objects.

Overload rule summary:

* [C.160: 연산자를 정의할때는 관례적인 사용을 모방하라](#Ro-conventional)
* [C.161: 대칭적인 연산자들에는 비멤버 함수들을 사용하라](#Ro-symmetric)
* [C.162: 거의 동등한 연산들을 오버로드하라](#Ro-equivalent)
* [C.163: 거의 동등한 연산들만 오버로드하라](#Ro-equivalent-2)
* [C.164: 형변환 연산자들을 지양하라](#Ro-conversion)
* [C.165: Use `using` for customization points](#Ro-custom)
* [C.166: Overload unary `&` only as part of a system of smart pointers and references](#Ro-address-of)
* [C.167: Use an operator for an operation with its conventional meaning](#Ro-overload)
* [C.168: Define overloaded operators in the namespace of their operands](#Ro-namespace)
* [C.170: 람다를 오버로딩하는 기분이 든다면, 제네릭 람다를 사용하라](#Ro-lambda)

### <a name="Ro-conventional"></a>C.160: 연산자를 정의할때는 관례적인 사용을 모방하라

##### 근거
뜻밖의 의미가 없도록 한다.

##### 예
```
    class X {
    public:
        // ...
        X& operator=(const X&); // member function defining assignment
        friend bool operator==(const X&, const X&); // == needs access to representation
                                                    // after a=b we have a==b
        // ...
    };
```
Here, the conventional semantics is maintained: [Copies compare equal](#SS-copy).

##### 잘못된 예
```
    X operator+(X a, X b) { return a.v - b.v; }   // bad: makes + subtract
```
##### 참고 사항

Non-member operators should be either friends or defined in [the same namespace as their operands](#Ro-namespace).
[Binary operators should treat their operands equivalently](#Ro-symmetric).

##### 시행하기
Possibly impossible.


### <a name="Ro-symmetric"></a>C.161: 대칭적인 연산자들에는 비멤버 함수들을 사용하라

##### 근거

If you use member functions, you need two.
Unless you use a non-member function for (say) `==`, `a == b` and `b == a` will be subtly different.

##### 예
```
    bool operator==(Point a, Point b) { return a.x == b.x && a.y == b.y; }
```
##### 시행하기

Flag member operator functions.


### <a name="Ro-equivalent"></a>C.162: 거의 동등한 연산들을 오버로드하라

##### 근거

Having different names for logically equivalent operations on different argument types is confusing, leads to encoding type information in function names, and inhibits generic programming.

##### 예

Consider:
```
    void print(int a);
    void print(int a, int base);
    void print(const string&);
```
These three functions all print their arguments (appropriately). Conversely:
```
    void print_int(int a);
    void print_based(int a, int base);
    void print_string(const string&);
```
These three functions all print their arguments (appropriately). Adding to the name just introduced verbosity and inhibits generic code.

##### 시행하기

???


### <a name="Ro-equivalent-2"></a>C.163: 거의 동등한 연산들'만' 오버로드하라

##### 근거

Having the same name for logically different functions is confusing and leads to errors when using generic programming.

##### 예

Consider:
```
    void open_gate(Gate& g);   // remove obstacle from garage exit lane
    void fopen(const char* name, const char* mode);   // open file
```
The two operations are fundamentally different (and unrelated) so it is good that their names differ. Conversely:
```
    void open(Gate& g);   // remove obstacle from garage exit lane
    void open(const char* name, const char* mode ="r");   // open file
```
The two operations are still fundamentally different (and unrelated) but the names have been reduced to their (common) minimum, opening opportunities for confusion.
Fortunately, the type system will catch many such mistakes.

##### 참고 사항

Be particularly careful about common and popular names, such as `open`, `move`, `+`, and `==`.

##### 시행하기
???


### <a name="Ro-conversion"></a>C.164:형변환 연산자들을 지양하라

##### 근거
Implicit conversions can be essential (e.g., `double` to `int`) but often cause surprises (e.g., `String` to C-style string).

##### 참고 사항
Prefer explicitly named conversions until a serious need is demonstrated.
By "serious need" we mean a reason that is fundamental in the application domain (such as an integer to complex number conversion)
and frequently needed. Do not introduce implicit conversions (through conversion operators or non-`explicit` constructors)
just to gain a minor convenience.

##### 잘못된 예
```
    class String {   // handle ownership and access to a sequence of characters
        // ...
        String(czstring p); // copy from *p to *(this->elem)
        // ...
        operator zstring() { return elem; }
        // ...
    };

    void user(zstring p)
    {
        if (*p == "") {
            String s {"Trouble ahead!"};
            // ...
            p = s;
        }
        // use p
    }
```
`s`에 할당되고 `p`에 대입된 문자열이 사용될 수 있게 되기 전에 파괴된다.

##### 시행하기
모든 형변환 연산자에 표시를 남겨라.


### <a name="Ro-custom"></a>C.165: Use `using` for customization points

##### 근거
To find function objects and functions defined in a separate namespace to "customize" a common function.

##### 예
Consider `swap`. It is a general (standard library) function with a definition that will work for just about any type.
However, it is desirable to define specific `swap()`s for specific types.
For example, the general `swap()` will copy the elements of two `vector`s being swapped, whereas a good specific implementation will not copy elements at all.
```
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
```
    void f2(N::X& a, N::X& b)
    {
        swap(a, b);   // calls N::swap
    }
```
But that may not be what we wanted for generic code.
There, we typically want the specific function if it exists and the general function if not.
This is done by including the general function in the lookup for the function:
```
    void f3(N::X& a, N::X& b)
    {
        using std::swap;  // make std::swap available
        swap(a, b);        // calls N::swap if it exists, otherwise std::swap
    }
```
##### 시행하기

Unlikely, except for known customization points, such as `swap`.
The problem is that the unqualified and qualified lookups both have uses.


### <a name="Ro-address-of"></a>C.166: Overload unary `&` only as part of a system of smart pointers and references

##### 근거

The `&` operator is fundamental in C++.
Many parts of the C++ semantics assumes its default meaning.

##### 예
```
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
##### 참고 사항

If you "mess with" operator `&` be sure that its definition has matching meanings for `->`, `[]`, `*`, and `.` on the result type.
Note that operator `.` currently cannot be overloaded so a perfect system is impossible.
We hope to remedy that: [n4477.pdf](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4477.pdf).  
Note that `std::addressof()` always yields a built-in pointer.

##### 시행하기

Tricky. Warn if `&` is user-defined without also defining `->` for the result type.


### <a name="Ro-namespace"></a>C.168: Define overloaded operators in the namespace of their operands

##### 근거

Readability.
Ability for find operators using ADL.
Avoiding inconsistent definition in different namespaces

##### 예
```
    struct S { };
    bool operator==(S, S);   // OK: in the same namespace as S, and even next to S
    S s;

    bool s == s;
```
This is what a default `==` would do, if we had such defaults.

##### 예
```
    namespace N {
        struct S { };
        bool operator==(S, S);   // OK: in the same namespace as S, and even next to S
    }

    N::S s;

    bool s == s;  // finds N::operator==() by ADL
```
##### 잘못된 예
```
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

##### 참고 사항

If a binary operator is defined for two types that are defined in different namespaces, you cannot follow this rule.
For example:
```
    Vec::Vector operator*(const Vec::Vector&, const Mat::Matrix&);
```
This may be something best avoided.

##### 함께 보기

This is a special case of the rule that [helper functions should be defined in the same namespace as their class](#Rc-helper).

##### 시행하기

* Flag operator definitions that are not it the namespace of their operands


### <a name="Ro-overload"></a>C.167: Use an operator for an operation with its conventional meaning

##### 근거

Readability. Convention. Reusability. Support for generic code

##### 예
```
    void cout_my_class(const my_class& c) // confusing, not conventional,not generic
    {
        std::cout << /* class members here */;
    }

    std::ostream& operator<<(std::ostream& os, const my_class& c) //OK
    {
        return os << /* class members here */;
    }
```
By itself, `cout_my_class` would be OK, but it is not usable/composable with code that rely on the `<<` convention for output:
```
    My_class var { /* ... */ };
    // ...
    cout << "var = " << var << '\n';
```
##### 참고 사항

There are strong and vigorous conventions for the meaning most operators, such as

* comparisons (`==`, `!=`, `<`, `<=`, `>`, and `>=`),
* arithmetic operations (`+`, `-`, `*`, `/`, and `%`)
* access operations (`.`, `->`, unary `*`, and `[]`)
* assignment (`=`)

Don't define those unconventionally and don't invent your own names for them.

##### 시행하기
까다롭다. 의미론에 대한 통찰이 필요하다.


### <a name="Ro-lambda"></a>C.170: 람다를 오버로딩하는 기분이 든다면, 제네릭 람다를 사용하라


##### 근거
같은 이름으로 다른 람다 함수를 오버로드 할 수 없다. 

##### 예
```
    void f(int);
    void f(double);
    auto f = [](char);   // error: cannot overload variable and function

    auto g = [](int) { /* ... */ };
    auto g = [](double) { /* ... */ };   // error: cannot overload variables

    auto h = [](auto) { /* ... */ };   // OK
```
##### 시행하기
컴파일러가 람다 함수에 대한 오버로드 시도를 잡아낸다. 


## <a name="SS-union"></a>C.union: Unions
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#SS-union)

???

Unions 규칙 요약:

* [C.180: `union`은 ???에 사용하라](#Ru-union)
* [C.181: `union` 그대로 사용하는 것을 지양하라](#Ru-naked)
* [C.182: 익명 `union`은 tagged union을 구현하는데 사용하라](#Ru-anonymous)
* ???

### <a name="Ru-union"></a>C.180: Use `union`s to ???

??? When should unions be used, if at all? What's a good future-proof way to re-interpret object representations of PODs?
??? variant

##### 근거
 ???

##### 예
```
    ???
```

##### 시행하기
???


### <a name="Ru-naked"></a>C.181: Avoid "naked" `union`s

##### 근거
Naked unions are a source of type errors.

##### 대안
Wrap them in a class together with a type field.

##### 대안
Use `variant`.

##### 예
```
    ???
```


##### 시행하기
???


### <a name="Ru-anonymous"></a>C.182: Use anonymous `union`s to implement tagged unions

##### 근거
???

##### 예
```
    ???
```
##### 시행하기

???
