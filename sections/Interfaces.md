
# <a name="S-interfaces"></a>I: 인터페이스

인터페이스는 프로그램 두 부분 사이의 계약이다. 서비스의 공급자와 사용자가 기대하는 바를 정확하게 기술하는 것이 핵심이다.
좋은 인터페이스(이해하기 쉽고, 효율적인 사용을 장려하고, 쉽게 오류가 발생하지 않고, 테스트를 지원하는 등)는 아마도 코드 구성 중에서 가장 중요한 요소 중 하나다.

인터페이스 규칙 요약:

* [I.1: 인터페이스를 명확하게 만들어라](#Ri-explicit)
* [I.2: `const`가 아닌 전역 변수를 피하라](#Ri-global)
* [I.3: 싱글톤을 피하라](#Ri-singleton)
* [I.4: 인터페이스를 정확하게, 강타입으로 만들어라](#Ri-typed)
* [I.5: (하나라도 있다면) 사전 조건을 기술하라](#Ri-pre)
* [I.6: 사전 조건을 표현하고 싶다면 `Expects()`를 사용하라](#Ri-expects)
* [I.7: 사후 조건을 기술하라](#Ri-post)
* [I.8: 사후 조건을 표현하고 싶다면 `Ensures()`를 사용하라](#Ri-ensures)
* [I.9:  인터페이스가 템플릿이라면 컨셉(Concept)을 사용해서 매개 변수를 문서화하라](#Ri-concepts)
* [I.10: 요구된 작업의 수행 실패를 알리기 위해 예외를 사용하라](#Ri-except)
* [I.11: 원시 포인터(`T*`) 혹은 참조(`T&`)를 사용해 소유권을 넘기지 마라](#Ri-raw)
* [I.12: null이 되어선 안되는 포인터는 `not_null`로 선언하라](#Ri-nullptr)
* [I.13: 배열을 단일 포인터로 전달하지 마라](#Ri-array)
* [I.22: 전역 개체의 복잡한 초기화를 피하라](#Ri-global-init)
* [I.23: 함수 인자를 최소로 유지하라](#Ri-nargs)
* [I.24: 관련없는 동일 타입의 인접한 매개 변수는 피하라](#Ri-unrelated)
* [I.25: 클래스 계층에 대한 인터페이스로 추상 클래스를 사용하라](#Ri-abstract)
* [I.26: 크로스 컴파일러 ABI를 원한다면 C 스타일 코드를 사용하라](#Ri-abi)
* [I.27: For stable library ABI, consider the Pimpl idiom](#Ri-pimpl)
* [I.30: Encapsulate rule violations](#Ri-encapsulate)

**참고할 만한 내용**:

* [F: 함수(Functions)](#S-functions)
* [C.concrete: 실제 타입(Concrete types)](#SS-concrete)
* [C.hier: 클래스 계층(Class hierarchies)](#SS-hier)
* [C.over: 오버로딩과 오버로딩 된 연산자(Overloading and overloaded operators)](#SS-overload)
* [C.con: 컨테이너와 다른 리소스 핸들(Containers and other resource handles)](#SS-containers)
* [E: 오류 처리(Error handling)](#S-errors)
* [T: 템플릿과 제너릭 프로그래밍(Templates and generic programming)](#S-templates)

### <a name="Ri-explicit"></a>I.1: Make interfaces explicit

##### Reason

정확성. 인터페이스에 언급되지 않은 가정은 간과되기 쉽고 테스트하기 어렵다.

##### Example, bad

전역 (네임스페이스 범위에 있는) 변수를 통해 함수의 행동을 제어하는 것은 암시적이고 혼란스럽다.

예를 들어:

```c++
    int round(double d)
    {
        return (round_up) ? ceil(d) : d;    // don't: "invisible" dependency
    }
```

`rnd(7.2)`를 두 번 호출해서 서로 다른 결과가 발생한다면 호출하는 입장에서는 함수의 의미를 명확하게 알지 못할 것이다.

##### Exception

때때로 우리는 환경 변수를 통해 연산의 세부 사항을 제어한다. 예를 들어, 일반적인 출력 대 상세한 출력, 디버그 대 최적화 등이 있다.  
전역 제어를 사용하면 잠재적으로 혼란스럽다는 문제가 있기는 하지만, 그렇게 하지 않으면 고정되었을 의도를 구현하는데 세밀하게 제어할 수 있다.

##### Example, bad

R`errno`와 같이 전역 변수를 통해 오류를 보고하는 것은 무시되기 쉽다.

예를 들어:

```c++
    // don't: no test of printf's return value
    fprintf(connection, "logging: %d %d %d\n", x, y, s);
```

만약 연결(connection)이 다운되어 로그가 만들어지지 않는다면 어떨까?

**Alternative**: 예외를 발생시켜라. 예외는 무시할 수 없다.

**Alternative formulation**: 전역 또는 암시적 상태의 값을 통해 인터페이스로 정보가 전달되는 것을 피하라. `const`가 아닌 멤버 함수는 개체의 상태를 통해 다른 멤버 함수에게 정보를 전달한다는 점을 참고하라.

**Alternative formulation**: 인터페이스는 함수나 함수의 집합이어야 한다. 함수는 템플릿 함수일 수 있고 함수 집합은 클래스나 클래스 템플릿일 수 있다.

##### Enforcement

* (간단함) 함수는 네임스페이스 범위에서 선언된 변수 값에 따라 제어 흐름을 결정해서는 안된다
* (간단함) 함수는 네임스페이스 범위에서 선언된 변수에 값을 저장해서는 안된다

### <a name="Ri-global"></a>I.2: Avoid non-`const` global variables

##### Reason

`const`가 아닌 전역 변수는 의존성을 숨기고 예측하지 못한 변화에 의존하게 만든다.

##### Example

```c++
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
```

또 어디서 `data`를 수정할 수 있는가?

##### Note

전역 상수는 유용하다.

##### Note

전역 변수에 반하는 규칙은 네임스페이스 범위의 변수들에도 동일하게 적용된다.

**Alternative**: 복사를 피하기 위해 전역 변수(좀 더 일반화해서 네임스페이스 스코프의 데이터)를 사용한다면 `const` 레퍼런스로 개체를 전달하는 것을 고려하라.
다른 해결책은 개체의 상태로서의 데이터와 멤버 함수로서의 연산을 정의하는 것이다.

**Warning**: 데이터 경합을 조심하라. 하나의 스레드가 실행되는 동안 다른 스레드가 전역 데이터(또는 레퍼런스로 전달된 데이터)에 접근하려고 한다면, 데이터 경합이 발생할 수 있다.
Every pointer or reference to mutable data is a potential data race.

##### Note

변경할 수 없는 데이터에 대해서는 경합 조건(race condition)이 생기지 않는다.

**References**: [함수 호출에 대한 규칙](#SS-call)

##### Note

The rule is "avoid", not "don't use." Of course there will be (rare) exceptions, such as `cin`, `cout`, and `cerr`.

##### Enforcement

(간단함) 네임스페이스 범위 내에 정의된 `const`가 아닌 변수에 대해서 모두 보고하라.

### <a name="Ri-singleton"></a>I.3: Avoid singletons

##### Reason
싱글톤은 기본적으로 복잡한 전역 개체이다.

##### Example

```c++
    class Singleton {
        // ... lots of stuff to ensure that only one Singleton object is created,
        // that it is initialized properly, etc.
    };
```

싱글톤 아이디어에는 많은 변형이 있다.
That's part of the problem.

##### Note

전역 개체가 변경되지 않게 하려면, `const`나 `constexpr`로 선언하라.

##### Exception

제일 단순한 "싱글톤"(싱글톤으로 생각되지 않을 정도로 간단한)을 사용해 처음 사용시 초기화를 할 수 있다.

```c++
    X& myX()
    {
        static X my_x {3};
        return my_x;
    }
```

이 방식은 초기화 순서와 관련된 문제에 대해 가장 효과적인 해결책 중 하나다.
다중 스레드 환경에서 정적 개체의 초기화는 경합 조건을 유발하지 않는다. (단, 생성자 내에서 공유 개체에 부주의하게 접근하지 않아야 한다.)

Note that the initialization of a local `static` does not imply a race condition.
However, if the destruction of `X` involves an operation that needs to be synchronized we must use a less simple solution.
For example:

```c++
    X& myX()
    {
        static auto p = new X {3};
        return *p;  // potential leak
    }
```

Now someone must `delete` that object in some suitably thread-safe way.
That's error-prone, so we don't use that technique unless

* `myX` is in multi-threaded code,
* that `X` object needs to be destroyed (e.g., because it releases a resource), and
* `X`'s destructor's code needs to be synchronized.

하나의 개체만 생성해야 하는 클래스로 싱글톤을 정의한다면 `myX`와 같은 함수는 싱글톤이 아니다. 그리고 이 유용한 테크닉은 싱글톤이 아닌 규칙에 대한 예외는 아니다.

##### Enforcement

일반적으로 매우 힘들다.

* `singleton`을 포함하는 이름을 가진 클래스를 찾아라
* 개체를 세거나 생성자를 검사해 단일 개체만 만들어진 클래스를 찾아라
* If a class X has a public static function that contains a function-local static of the class' type X and returns a pointer or reference to it, ban that

### <a name="Ri-typed"></a>I.4: Make interfaces precisely and strongly typed

##### Reason

타입은 가장 단순하지만 최고의 문서이기도 하다. 잘 정의된 의미를 갖고 있고, 컴파일 타임 검사가 보장된다.

또한 정확하게 타입을 사용하는 코드는 더 잘 최적화된다.

##### Example, don't

다음을 고려해 보자:

```c++
    void pass(void* data);    // void* is suspicious
```

이제 피호출자에서 올바른 타입으로 사용하기 위해 `data` 포인터를 캐스팅해야 한다. 하지만 오류가 발생하기 쉽고, 구질구질하다.
특히 인터페이스에서 `void*`를 피하라.
대신 베이스 클래스를 가리키는 포인터나 `variant` 사용을 고려하라.

**Alternative**: 때때로 템플릿 매개 변수는 `void*`를 제거하고 `T*`나 `T&`로 변환할 수 있다.
For generic code these `T`s can be general or concept constrained template parameters.

##### Example, bad

다음을 고려해 보자:

```c++
    void draw_rect(int, int, int, int);   // great opportunities for mistakes

    draw_rect(p.x, p.y, 10, 20);          // what does 10, 20 mean?
```

`int`는 임의 형태의 정보를 전달할 수 있어서, 네 `int`가 각각 어떤 의미를 갖는지를 유추해야 한다.
아마도 처음 두 `int`는 `x`, `y` 좌표일 것 같지만, 나머지 두 `int`는 무엇을 의미할까?  
주석이나 매개 변수 이름이 도움을 줄 수 있지만, 명확히 할 수 있다:

```c++
    void draw_rectangle(Point top_left, Point bottom_right);
    void draw_rectangle(Point top_left, Size height_width);

    draw_rectangle(p, Point{10, 20});  // two corners
    draw_rectangle(p, Size{10, 20});   // one corner and a (height, width) pair
```

분명히 정적 타입 시스템을 통해 모든 오류를 잡아낼 수는 없다.
(예를 들어, 첫번째 인자가 왼쪽 상단에 있는 점이라는 사실은 이름이나 주석 등을 통해 편의상 정해져 있을 뿐이다.)

##### Example, bad

다음 예제에서 인터페이스만 보고는 `time_to_blink`이 무엇을 의미하는지 잘 모르겠다. 초를 의미할까? 밀리초를 의미할까?

```c++
    void blink_led(int time_to_blink) // bad -- the unit is ambiguous
    {
        // ...
        // do something with time_to_blink
        // ...
    }

    void use()
    {
        blink_led(2);
    }
```

##### Example, good

C++11에 도입된 `std::chrono::duration` 타입은 지속 시간의 단위를 명시적으로 표현하는데 도움이 된다.

```c++
    void blink_led(milliseconds time_to_blink) // good -- the unit is explicit
    {
        // ...
        // do something with time_to_blink
        // ...
    }

    void use()
    {
        blink_led(1500ms);
    }
```

어떤 종류의 지속 시간의 단위도 허용하도록 함수를 다음과 같이 작성할 수도 있다.

```c++
    template<class rep, class period>
    void blink_led(duration<rep, period> time_to_blink) // good -- accepts any unit
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
```

##### Enforcement

* (간단함) `void*`를 매개 변수나 리턴 타입으로 사용한다면 보고한다
* (잘하기 어려움) 다수의 내장 타입 인자를 갖는 멤버 함수를 찾는다

### <a name="Ri-pre"></a>I.5: State preconditions (if any)

##### Reason

인자는 피호출자에서 적절한 사용을 제한할 수 있는 의미를 갖는다.

##### Example

다음을 고려해 보자:

```c++
    double sqrt(double x);
```

여기서 `x`는 반드시 음수가 아니어야 한다. 타입 시스템으로는 이를 (있는 그대로 쉽게) 표현할 수 없고, 그래서 다른 수단을 사용해야 한다. 예를 들어,

```c++
    double sqrt(double x); // x must be nonnegative
```

일부 사전 조건은 단정문으로 표현하기도 한다.

예를 들어:

```c++
    double sqrt(double x) { Expects(x >= 0); /* ... */ }
```

이상적으로 `Expects(x >= 0)` 조건이 `sqrt()`의 인터페이스에 일부분이 되는게 가장 좋지만 그렇게 하기는 쉽지 않다. 따라서 지금은 함수 정의부(함수 본문)에 위치시킨다.

**References**: `Expects()`는 [GSL](#S-gsl)에 기술되어 있다.

##### Note

`Excepts(p != nullptr);`처럼 요구 사항의 공식적인 명세를 선호하라.
불가능하다면, `// the sequence [p:q) is ordered using <`와 같이 영문 텍스트 주석을 사용하라.

##### Note

대부분의 멤버 함수는 클래스의 불변 조건(invariant) 중 일부에 해당하는 선행 조건을 갖고 있다.
해당 불변 조건은 생성자에서 구성되는데 클래스 외부로부터 호출되는 모든 멤버 함수를 통해 재구성되어야 한다.
그래서 각 함수마다 언급할 필요는 없다.

##### Enforcement

(적용 불가능)

**See also**: 포인터 전달에 대한 규칙. ???

### <a name="Ri-expects"></a>I.6: Prefer `Expects()` for expressing preconditions

##### Reason

사전 조건임을 명확하게 표시하고 툴 사용을 가능하게 한다.

##### Example

```c++
    int area(int height, int width)
    {
        Expects(height > 0 && width > 0);            // good
        if (height <= 0 || width <= 0) my_error();   // obscure
        // ...
    }
```

##### Note

사전 조건은 `if`문, `assert()`문, 주석문 등으로 기술할 수 있다.
하지만 이런 구문은 일반 코드와 구분, 갱신하거나 툴로 조작하기 어렵고 잘못된 의미를 가질 수도 있다. (디버그 모드일 때는 중단시키고, 릴리즈 모드일때는 검사하고 싶은가?)

##### Note

사전 조건은 구현 부분보다는 인터페이스 부분에 포함시켜야 한다. 하지만 아직은 그렇게 할 수 있는 언어 기능이 없다.
Once language support becomes available (e.g., see the [contract proposal](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r1.pdf)) we will adopt the standard version of preconditions, postconditions, and assertions.

##### Note

`Expects()`은 알고리즘 중간에 조건을 검사하는데 사용할 수도 있다.

##### Note

No, using `unsigned` is not a good way to sidestep the problem of [ensuring that a value is nonnegative](#Res-nonnegative).

##### Enforcement

(적용 불가능) 사전 조건이 단정될 수 있는 다양한 방법을 찾는 것은 실현 가능하지 않다. 쉽게 식별할 수 있는 것들에 대해 경고(`assert()`)를 할 수 있는 언어 기능이 없다면 의심의 여지가 생길 수 밖에 없다.

### <a name="Ri-post"></a>I.7: State postconditions

##### Reason

결과에 대해 잘못 이해하고 있는 부분과 잘못된 구현을 찾아내기 위해서다.

##### Example, bad

다음을 고려해 보자:

```c++
    int area(int height, int width) { return height * width; }  // bad
```

여기서 부주의하게 높이와 폭이 양수여야 한다는 사전 조건 명세를 빠뜨렸다.
역시 넓이를 구하는 알고리즘(`hieght * width`)이 정수의 최댓값보다 클 수 있다는 사후 조건 명세를 빠뜨렸다.
오버플로우가 발생할 수 있다.

다음 사용을 고려해 보자:

```c++
    int area(int height, int width)
    {
        auto res = height * width;
        Ensures(res > 0);
        return res;
    }
```

##### Example, bad

악명 높은 보안 버그를 고려해 보자:

```c++
    void f()    // problematic
    {
        char buffer[MAX];
        // ...
        memset(buffer, 0, sizeof(buffer));
    }
```

버퍼가 초기화되어야 하고 최적화하면서 중복되는 `memset()`의 호출을 제거했음을 설명하는 사후 조건이 없다.

```c++
    void f()    // better
    {
        char buffer[MAX];
        // ...
        memset(buffer, 0, sizeof(buffer));
        Ensures(buffer[0] == 0);
    }
```

##### Note

사후 조건은 종종 함수의 목적을 기술하는 주석문에 비공식적으로 명시되어 있다. `Ensures()`를 사용함으로서 더 시스템적으로 검사하는 모습을 보여줄 수 있다.

##### Note

사후 조건은 사용되는 자료 구조의 상태처럼 반환된 결과에 간접적으로 영향을 미치는 무언가에 연관되어 있을 때 특히 중요하다.

##### Example

경합 조건을 피할 목적으로 `mutex`를 사용해 `Record`를 조작하는 함수를 고려해 보자:

```c++
    mutex m;

    void manipulate(Record& r)    // don't
    {
        m.lock();
        // ... no m.unlock() ...
    }
```

여기서 `mutex`를 해제해야 한다는 언급을 "잊어"버렸는데, `mutex` 해제를 언급하지 않은 것이 단순히 버그인지 의도한 것인지 알 수가 없다.
사후 조건을 기술함으로서 언급을 명확하게 할 수 있다:

```c++
    void manipulate(Record& r)    // postcondition: m is unlocked upon exit
    {
        m.lock();
        // ... no m.unlock() ...
    }
```

이제 버그가 분명하게 보인다. (단, 주석을 읽는 사람에게만 보인다.)

더 나은 방법은 [RAII](#Rc-raii)를 사용해 사후 조건("잠금을 해제해야 함")이 코드에 적용되도록 하는 것이다:

```c++
    void manipulate(Record& r)    // best
    {
        lock_guard<mutex> _ {m};
        // ...
    }
```

##### Note

이상적으로, 사후 조건은 사용자들이 쉽게 볼 수 있도록 인터페이스/선언부에 기술해야 한다.
사용자들에게 언급되어야 하는 사후 조건만 인터페이스에 기술해야 한다.
내부 상태에 대한 사후 조건은 정의/구현부에만 기술한다.

##### Enforcement

(Not enforceable) This is a philosophical guideline that is infeasible to check
directly in the general case. Domain specific checkers (like lock-holding
checkers) exist for many toolchains.

### <a name="Ri-ensures"></a>I.8: Prefer `Ensures()` for expressing postconditions

##### Reason

사후 조건이라는 것을 분명히 하기 위해, 또 분석 툴을 사용하기 위해서다.

##### Example

```c++
    void f()
    {
        char buffer[MAX];
        // ...
        memset(buffer, 0, MAX);
        Ensures(buffer[0] == 0);
    }
```

##### Note

사후 조건은 주석문, `if`문, `assert()`문 등 다양한 방식으로 기술할 수 있다.
이는 일반적인 코드와 구분을 어렵게 만들고, 갱신하기 어렵게 만들고, 툴로 조작하기 어렵게 만들며 잘못된 의미를 가질 수도 있다.

**Alternative**: 이 리소스는 반드시 해제되어야 한다" 형태의 사후 조건은 [RAII](#Rc-raii)를 통해 가장 잘 나타낼 수 있다.

##### Note

이상적으로 `Ensures`는 인터페이스의 일부가 되어야 하지만, 쉽게 할 수 있는 작업이 아니다.
현재로서는 정의 부분(함수 본문)에 위치시킨다.
Once language support becomes available (e.g., see the [contract proposal](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r1.pdf)) we will adopt the standard version of preconditions, postconditions, and assertions.

##### Enforcement

(적용 불가능) 사후 조건을 단정할 수 있는 다양한 방식을 찾는 것은 실현 가능하지 않다. 
Warning about those that can be easily identified (`assert()`) has questionable value in the absence of a language facility.

### <a name="Ri-concepts"></a>I.9: If an interface is a template, document its parameters using concepts

##### Reason

인터페이스를 정확하게, 그리고 (멀지 않은) 미래에 컴파일 타임 검사를 할 수 있게 만들어라.

##### Example

요구 사항 명세에 ISO Concept TS 스타일을 사용하라. 

예를 들어:

```c++
    template<typename Iter, typename Val>
    // requires InputIterator<Iter> && EqualityComparable<ValueType<Iter>>, Val>
    Iter find(Iter first, Iter last, Val v)
    {
        // ...
    }
```

##### Note

`//`가 제거되면 대부분의 컴파일러가 `requires` 구문을 검사할 수 있을 것이다.
현재 Concept은 GCC 6.1과 그 이후 버전에서 지원된다.

**See also**: [제너릭 프로그래밍](#SS-GP)과 [컨셉](#SS-t-concepts)을 보라

##### Enforcement

(아직 적용 불가능) 언어 명세를 작성중이다. 언어 명세가 정의된 후, 비가변 템플릿 매개 변수가 (선언이나 `requires`문에 언급되지 않은) 컨셉으로 제한되지 않는다면 경고하라.

### <a name="Ri-except"></a>I.10: Use exceptions to signal a failure to perform a required task

##### Reason

시스템이나 계산 결과를 정의되지 않은(예측 불가능한) 상태로 둘 수 없으므로 오류를 무시해서는 안된다.

이는 오류의 주 발생지가 된다.

##### Example

```c++
    int printf(const char* ...);    // bad: return negative number if output fails

    template <class F, class ...Args>
    // good: throw system_error if unable to start the new thread
    explicit thread(F&& f, Args&&... args);
```

##### Note

오류란 무엇인가?

오류란 함수가 (사후 조건 설정을 포함해) 의도한 목적을 이룰 수 없는 것을 의미한다.
오류를 무시하는 코드를 호출하면 정의되지 않은 시스템 상태나 잘못된 결과를 야기할 수 있다.
예를 들어, 리모트 서버에 연결할 수 없는 경우 자체적으로 오류가 발생하지 않는다.
서버는 다양한 이유로 연결을 거부할 수 있다. 따라서 호출하는 쪽에서 결과를 확인할 수 있도록 반환해 주는 것이 자연스럽다.
그러나 연결에 실패하는 것을 오류로 간주할 생각이라면, 실패할 경우 예외를 던져야 한다.

##### Exception

기존의 많은 인터페이스 함수들은 오류가 아닌 실제 상태 코드를 보고하기 위해 오류 코드(예를 들어, `errno`)를 사용한다. 이에 대해 별다른 대안이 없으므로 규칙을 위반하지는 않는다.

##### Alternative

만약 (예를 들어, 여러분의 코드가 예전 스타일의 처리되지 않은 포인터로 가득하거나 실시간 제약이 심각하기 때문에) 예외를 사용할 수 없다면, 한 쌍의 값을 반환하는 스타일을 사용하는 것을 고려해 보라:

```c++
    int val;
    int error_code;
    tie(val, error_code) = do_something();
    if (error_code) {
        // ... handle the error or exit ...
    }
    // ... use val ...
```

This style unfortunately leads to uninitialized variables.
A facility [structured bindings](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0144r1.pdf) to deal with that will become available in C++17.

```c++
    auto [val, error_code] = do_something();
    if (error_code) {
        // ... handle the error or exit ...
    }
    // ... use val ...
```

##### Note

"성능"이 예외를 사용하지 않아야 하는 타당한 이유라고는 생각하지 않는다.

* 명시적인 오류 검사 및 처리는 종종 예외 처리만큼 많은 시간과 공간을 쓰기도 한다.
* 간결한 코드는 종종 예외 처리를 포함하더라도 더 좋은 성능을 갖는다. (프로그램과 최적화를 통해 실행 경로를 단순화한다.)
* 성능이 중요한 코드의 좋은 규칙은 오류 검사를 코드의 핵심 부분 바깥으로 옮기는 것이다 ([검사](#Rper-checking)).
* 장기적으로 보면 정규 코드가 더 잘 최적화된다.
* Always carefully [measure](#Rper-measure) before making performance claims.

**See also**: 사전 조건, 사후 조건의 위반 보고를 위한 [I.5](#Ri-pre) 및 [I.7](#Ri-post).

##### Enforcement

* (적용 불가능) 철저한 점검이 불가능한 철학적 가이드라인이다
* `errno`를 살펴 봐라

### <a name="Ri-raw"></a>I.11: Never transfer ownership by a raw pointer (`T*`) or reference (`T&`)

##### Reason

호출하는 쪽이나 받는 쪽 중 누가 개체를 소유할 것인지 모른다면, 메모리 누수나 조기 파괴가 발생할 것이다.

##### Example

다음을 고려해 보자:

```c++
    X* compute(args)    // don't
    {
        X* res = new X{};
        // ...
        return res;
    }
```

반환된 `X`를 누가 삭제할 것인가? 만약 레퍼런스를 반환한다면 문제는 더 어려워질 것이다.
결과가 값으로 반환되었다고 고려해 보자 (결과로 반환되는 값의 크기가 크다면 이동 문법을 사용하라):

```c++
    vector<double> compute(args)  // good
    {
        vector<double> res(10000);
        // ...
        return res;
    }
```

**Alternative**: [Pass ownership](#Rr-smartptrparam) using a "smart pointer", such as `unique_ptr` (for exclusive ownership) and `shared_ptr` (for shared ownership).
However, that is less elegant and often less efficient than returning the object itself,
so use smart pointers only if reference semantics are needed.

**Alternative**: ABI 호환성 요구 사항 또는 리소스 부족으로 인해 오래된 코드를 수정할 수 없는 경우가 있다.
이 경우, [가이드라인 지원 라이브러리](#S-gsl)의 `owner`를 사용해 포인터의 소유권을 표시하라:

```c++
    owner<X*> compute(args)    // It is now clear that ownership is transferred
    {
        owner<X*> res = new X{};
        // ...
        return res;
    }
```

위의 코드는 분석 툴에게 `res`가 소유권자라고 알려준다.
즉, 이 값은 `return`을 통해 행해진 것처럼 `delete`되거나 다른 소유권자에게 넘겨줘야 한다.

`owner`는 리소스 핸들의 구현에서 비슷하게 사용된다.

##### Note

처리되지 않은 포인터(또는 반복자)로 전달된 모든 개체는 호출자가 소유한 것으로 간주되므로 호출자가 수명을 처리한다. Viewed another way:
ownership transferring APIs are relatively rare compared to pointer-passing APIs,
so the default is "no ownership transfer."

**See also**: [Argument passing](#Rf-conventional), [use of smart pointer arguments](#Rr-smartptrparam), and [value return](#Rf-value-return).

##### Enforcement

* (간단함) `owner`가 아닌 처리되지 않은 포인터의 `delete`에 대해 경고를 표시하라.
* (간단함) 모든 코드 경로에서 `owner` 포인터를 `reset`하거나 명시적으로 `delete`를 실패하게 되면 경고를 표시하라.
* (간단함) `new`의 반환 값이나 포인터 타입의 반환 값을 갖는 함수 호출이 처리되지 않은 포인터에 할당되면 경고를 표시하라.

### <a name="Ri-nullptr"></a>I.12: Declare a pointer that must not be null as `not_null`

##### Reason

`nullptr` 역참조 오류를 피하기 위해서다.
그리고 `nullptr`를 반복해서 검사하는 경우를 피해 성능을 향상시키기 위해서다.

##### Example

```c++
    int length(const char* p);            // it is not clear whether length(nullptr) is valid

    length(nullptr);                      // OK?

    int length(not_null<const char*> p);  // better: we can assume that p cannot be nullptr

    int length(const char* p);            // we must assume that p can be nullptr
```

소스 코드에 의도를 명시함으로써, 컴파일러와 툴이 정적 분석을 통해 일부 오류 클래스를 찾아내는 등의 보다 나은 진단을 제공하고 분기 및 널(NULL) 검사를 제거하는 등의 최적화 작업을 수행할 수 있다.

##### Note

`not_null`은 [가이드라인 지원 라이브러리](#S-gsl)에 정의되어 있다.

##### Note

`char`에 대한 포인터가 C-스타일 문자열(`\0`으로 끝나는 문자열)을 가리키고 있다는 가정은 여전히 암묵적이며 혼란과 오류를 발생시키는 원인이 될 수 있다. `const char*`보다는 `czstring`을 사용하라.

```c++
    // we can assume that p cannot be nullptr
    // we can assume that p points to a zero-terminated array of characters
    int length(not_null<zstring> p);
```

물론 `length()`는 `std::strlen()`이다.

##### Enforcement

* (간단함) ((기초)) 함수가 모든 제어-흐름 경로에서 포인터 매개 변수에 접근하기 전에 `nullptr`인지 검사한다면, `not_null`으로 선언되어야 한다는 경고하라
* (복잡함) 포인터 반환 값을 갖는 함수가 모든 반환 경로에서 `nullptr`이 아닌지 확인한다면, 리턴 타입을 `not_null`으로 선언해야 된다는 경고하라

### <a name="Ri-array"></a>I.13: Do not pass an array as a single pointer

##### Reason

(포인터, 크기)-스타일 인터페이스는 오류가 발생하기 쉽다. 또한 (배열에 대한) 일반 포인터는 호출을 받는 곳에서 크기를 결정할 수 있도록 몇 가지 관례에 의존해야 한다.

##### Example

다음을 고려해 보자:

```c++
    void copy_n(const T* p, T* q, int n); // copy from [p:p+n) to [q:q+n)
```

만약 `q`가 가리키는 배열의 원소 갯수가 `n`보다 적다면 어떻게 될까? 관계없는 메모리를 덮어쓰게 된다.
만약 `p`가 가리키는 배열의 원소 갯수가 `n`보다 적다면 어떻게 될까? 관계없는 메모리를 읽을 것이다.
어느 쪽이나 정의되지 않은 동작을 수행하거나 매우 불쾌한 버그가 발생할 수 있다.

##### Alternative

명시적인 범위 사용을 고려해 보라:

```c++
    void copy(span<const T> r, span<T> r2); // copy r to r2
```

##### Example, bad

다음을 고려해 보자:

```c++
    void draw(Shape* p, int n);  // poor interface; poor code
    Circle arr[10];
    // ...
    draw(arr, 10);
```

`n`의 인수로 `10`을 전달하는 것은 실수일 수 있다. 가장 일반적인 관례는 \[`0`:`n`)이라고 가정하는 것이지만, 이러한 내용이 어디에도 언급되어 있지 않다.
더 나쁜 부분은 어쨌든 `draw()` 호출이 컴파일된다는 것이다. 배열에서 포인터로의 암시적 변환(배열 부패)이 있었고, `Circle`에서 `Shape`로의 또 다른 암시적 변환이 있었다.
`draw()`가 그 배열을 통해 안전하게 반복문을 수행할 수 있는 방법은 없다. 왜냐하면 요소의 크기를 알 수 있는 방법이 없기 때문이다.

**Alternative**: 요소의 크기한계를 보장하고 위험한 암시적 변환을 방지하는 지원 클래스를 사용하라.

예를 들어:

```c++
    void draw2(span<Circle>);
    Circle arr[10];
    // ...
    draw2(span<Circle>(arr));  // deduce the number of elements
    draw2(arr);    // deduce the element type and array size

    void draw3(span<Shape>);
    draw3(arr);    // error: cannot convert Circle[10] to span<Shape>
```

이 `draw2()`는 같은 양의 정보를 `draw()`에 전달하지만, 명시적으로 `Circle`이 되어야 한다는 사실을 알 수 있다.

##### Exception

`zstring`과 `czstring`을 사용해 C-스타일의 `\0`로 끝나는 문자열을 나타내라.
But when doing so, use `string_span` from the [GSL](#GSL) to prevent range errors.

##### Enforcement

* (간단함) ((경계)) 배열 타입에서 포인터 타입으로의 암시적 변환에 의존하는 표현식에 대해 경고를 표시하라. string/czstring 포인터 타입에 대해서는 예외를 허용한다.
* (간단함) ((경계)) 포인터 타입의 값을 가져오는 포인터 타입의 표현식에 대한 산술 연산에 대해 경고를 표시하라. zstring/czstring 포인터 타입에 대해서는 예외를 허용한다.

### <a name="Ri-global-init"></a>I.22: Avoid complex initialization of global objects

##### Reason

Complex initialization can lead to undefined order of execution.

##### Example

```c++
    // file1.c
    extern const X x;
    const Y y = f(x);   // read x; write y

    // file2.c
    extern const Y y;
    const X x = g(y);   // read y; write x
```

Since `x` and `y` are in different translation units the order of calls to `f()` and `g()` is undefined;
one will access an uninitialized `const`.
This shows that the order-of-initialization problem for global (namespace scope) objects is not limited to global *variables*.

##### Note

Order of initialization problems become particularly difficult to handle in concurrent code.
It is usually best to avoid global (namespace scope) objects altogether.

##### Enforcement

* Flag initializers of globals that call non-`constexpr` functions
* Flag initializers of globals that access `extern` objects

### <a name="Ri-nargs"></a>I.23: Keep the number of function arguments low

##### Reason

인자 갯수가 많으면 혼란을 일으킬 수 있다.
인자를 많이 전달하는 것은 다른 대안에 비해 비용이 많이 든다.

##### Discussion

The two most common reasons why functions have too many parameters are:

1. *Missing an abstraction.*
   There is an abstraction missing, so that a compound value is being
   passed as individual elements instead of as a single object that enforces an invariant.
   This not only expands the parameter list, but it leads to errors because the component values
   are no longer protected by an enforced invariant.

2. *Violating "one function, one responsibility."*
   The function is trying to do more than one job and should probably be refactored.

##### Example

표준 라이브러리 `merge()`는 편하게 처리할 수 있는 한계에 있다:

```c++
    template<class InputIterator1, class InputIterator2, class OutputIterator, class Compare>
    OutputIterator merge(InputIterator1 first1, InputIterator1 last1,
                         InputIterator2 first2, InputIterator2 last2,
                         OutputIterator result, Compare comp);
```

Note that this is because of problem 1 above -- missing abstraction. Instead of passing a range (abstraction), STL passed iterator pairs (unencapsulated component values).

여기에 4개의 템플릿 인자와 6개의 함수 인자가 있다.
To simplify the most frequent and simplest uses, the comparison argument can be defaulted to `<`:

```c++
    template<class InputIterator1, class InputIterator2, class OutputIterator>
    OutputIterator merge(InputIterator1 first1, InputIterator1 last1,
                         InputIterator2 first2, InputIterator2 last2,
                         OutputIterator result);
```

이렇게 한다고 해서 전체적인 복잡성이 줄어들지는 않지만 사용자 입장에서 볼 때는 복잡성이 줄어든 것처럼 보인다.
실제로 인자 갯수를 줄이려면 인자를 좀 더 높은 수준의 추상화로 묶어야 한다:

```c++
    template<class InputRange1, class InputRange2, class OutputIterator>
    OutputIterator merge(InputRange1 r1, InputRange2 r2, OutputIterator result);
```

인자를 "묶어서" 그룹화하는 것은 인자의 갯수를 줄이고 검사할 기회를 늘리는 일반적인 기법이다.

Alternatively, we could use concepts (as defined by the ISO TS) to define the notion of three types that must be usable for merging:

```c++
    Mergeable{In1, In2, Out}
    OutputIterator merge(In1 r1, In2 r2, Out result);
```

##### Example

The safety Profiles recommend replacing

```c++
    void f(int* some_ints, int some_ints_length);  // BAD: C style, unsafe
```

with

```c++
    void f(gsl::span<int> some_ints);              // GOOD: safe, bounds-checked
```

Here, using an abstraction has safety and robustness benefits, and naturally also reduces the number of parameters.

##### Note

얼마나 많은 인자가 있어야 너무 많다고 말할 수 있을까? 인자가 4개라면 많다고 말할 수 있다.
4개의 인자로 가장 잘 표현할 수 있는 함수들도 있지만, 많지는 않다.

**Alternative**: Use better abstraction: 인자를 의미있는 개체로 그룹화하고 개체를 전달하라. (값에 의한 전달 또는 레퍼런스에 의한 전달)

**Alternative**:  더 적은 인자 갯수로 가장 일반적인 형태의 호출을 할 수 있는 디폴트 인자나 오버로드를 사용하라.

##### Enforcement

* 범위 또는 뷰가 아닌 동일한 타입의 반복자(포인터 포함)를 2개 이상 선언하는 함수가 있다면 경고를 표시하라.
* (적용 불가능) 철저한 점검이 불가능한 철학적 가이드라인이다.

### <a name="Ri-unrelated"></a>I.24: Avoid adjacent unrelated parameters of the same type

##### Reason

실수로 동일 타입의 인접한 인수를 쉽게 바꿀 수 있다.

##### Example, bad

다음을 고려해 보자:

```c++
    void copy_n(T* p, T* q, int n);  // copy from [p:p + n) to [q:q + n)
```

위 코드는 K&R C-스타일 인터페이스의 적절하지 못한 변형이다. "to"와 "from" 인수를 쉽게 바꿀 수 있다.

"from" 인자에 `const`를 사용하라:

```c++
    void copy_n(const T* p, T* q, int n);  // copy from [p:p + n) to [q:q + n)
```

##### Exception

If the order of the parameters is not important, there is no problem:

```c++
    int max(int a, int b);
```

##### Alternative

배열을 포인터로 전달하지 말고 범위를 나타내는 개체(예를 들어, `span`)로 전달하라.

```c++
    void copy_n(span<const T> p, span<T> q);  // copy from p to q
```

##### Alternative

Define a `struct` as the parameter type and name the fields for those parameters accordingly:

```c++
    struct SystemParams {
        string config_file;
        string output_path;
        seconds timeout;
    };
    void initialize(SystemParams p);
```

This tends to make invocations of this clear to future readers, as the parameters
are often filled in by name at the call site.

##### Enforcement

(간단함) 연속하는 두 매개 변수가 동일한 타입을 공유하는 경우 경고를 표시하라.

### <a name="Ri-abstract"></a>I.25: Prefer abstract classes as interfaces to class hierarchies

##### Reason

추상 클래스는 상태가 있는 베이스 클래스보다 안정적이다.

##### Example, bad

당신은 `Shape`가 어디선가 나타날 것이라고 알고 있었을 것이다. :-)

```c++
    class Shape {  // bad: interface class loaded with data
    public:
        Point center() const { return c; }
        virtual void draw() const;
        virtual void rotate(int);
        // ...
    private:
        Point c;
        vector<Point> outline;
        Color col;
    };
```

이렇게 하면 파생된 모든 클래스가 중심을 계산하게 된다.
비록 중요하지 않고 중심이 사용되지 않더라도 말이다.
비슷하게, 모든 `Shape`가 `Color`를 갖고 있는 것은 아니며 많은 `Shape`들은 일련의 `Point`로 정의된 윤곽선없이 가장 잘 표현된다.
추상 클래스는 사용자가 그러한 클래스를 작성하지 못하도록 만들기 위해 고안되었다.

```c++
    class Shape {    // better: Shape is a pure interface
    public:
        virtual Point center() const = 0;   // pure virtual functions
        virtual void draw() const = 0;
        virtual void rotate(int) = 0;
        // ...
        // ... no data members ...
        // ...
        virtual ~Shape() = default;
    };
```

##### Enforcement

(간단함) `C` 클래스를 가리키는 포인터가 `C`의 베이스를 가리키는 포인터에 할당되고 베이스 클래스에 데이터 멤버가 있으면 경고를 표시하라.

### <a name="Ri-abi"></a>I.26: If you want a cross-compiler ABI, use a C-style subset

##### Reason

컴파일러마다 클래스, 예외 처리, 함수 이름 및 기타 구현 세부 사항에 대해 서로 다른 바이너리 레이아웃을 구현한다.

##### Exception

신중하게 선택한 몇 가지 고급 수준의 C++ 타입을 사용해 인터페이스를 신중하게 만들 수 있다.

##### Exception

일반적인 ABI는 일부 플랫폼에서 점점 더 엄격한 제한으로부터 벗어나고 있다.

##### Note

단일 컴파일러를 사용하는 경우 인터페이스에서 C++ 전체를 사용할 수 있다. 새 컴파일러 버전으로 업그레이드 한 후에는 다시 컴파일해야 할 수 있다.

##### Enforcement

(적용 불가능) 인터페이스가 ABI의 일부가 되는 부분을 확실하게 식별하기는 어렵다.

### <a name="Ri-pimpl"></a>I.27: For stable library ABI, consider the Pimpl idiom

##### Reason

Because private data members participate in class layout and private member functions participate in overload resolution, changes to those
implementation details require recompilation of all users of a class that uses them. A non-polymorphic interface class holding a pointer to
implementation (Pimpl) can isolate the users of a class from changes in its implementation at the cost of an indirection.

##### Example

interface (widget.h)

```c++
    class widget {
        class impl;
        std::unique_ptr<impl> pimpl;
    public:
        void draw(); // public API that will be forwarded to the implementation
        widget(int); // defined in the implementation file
        ~widget();   // defined in the implementation file, where impl is a complete type
        widget(widget&&) = default;
        widget(const widget&) = delete;
        widget& operator=(widget&&); // defined in the implementation file
        widget& operator=(const widget&) = delete;
    };
```

implementation (widget.cpp)

```c++
    class widget::impl {
        int n; // private data
    public:
        void draw(const widget& w) { /* ... */ }
        impl(int n) : n(n) {}
    };
    void widget::draw() { pimpl->draw(*this); }
    widget::widget(int n) : pimpl{std::make_unique<impl>(n)} {}
    widget::~widget() = default;
    widget& widget::operator=(widget&&) = default;
```

##### Notes

See [GOTW #100](https://herbsutter.com/gotw/_100/) and [cppreference](http://en.cppreference.com/w/cpp/language/pimpl) for the trade-offs and additional implementation details associated with this idiom.

##### Enforcement

(Not enforceable) It is difficult to reliably identify where an interface forms part of an ABI.

### <a name="Ri-encapsulate"></a>I.30: Encapsulate rule violations

##### Reason

To keep code simple and safe.
Sometimes, ugly, unsafe, or error-prone techniques are necessary for logical or performance reasons.
If so, keep them local, rather than "infecting" interfaces so that larger groups of programmers have to be aware of the
subtleties.
Implementation complexity should, if at all possible, not leak through interfaces into user code.

##### Example

Consider a program that, depending on some form of input (e.g., arguments to `main`), should consume input
from a file, from the command line, or from standard input.
We might write

```c++
    bool owned;
    owner<istream*> inp;
    switch (source) {
    case std_in:        owned = false; inp = &cin;                       break;
    case command_line:  owned = true;  inp = new istringstream{argv[2]}; break;
    case file:          owned = true;  inp = new ifstream{argv[2]};      break;
    }
    istream& in = *inp;
```

This violated the rule [against uninitialized variables](#Res-always),
the rule against [ignoring ownership](#Ri-raw),
and the rule [against magic constants](#Res-magic).
In particular, someone has to remember to somewhere write

```c++
    if (owned) delete inp;
```

We could handle this particular example by using `unique_ptr` with a special deleter that does nothing for `cin`,
but that's complicated for novices (who can easily encounter this problem) and the example is an example of a more general
problem where a property that we would like to consider static (here, ownership) needs infrequently be addressed
at run time.
The common, most frequent, and safest examples can be handled statically, so we don't want to add cost and complexity to those.
But we must also cope with the uncommon, less-safe, and necessarily more expensive cases.
Such examples are discussed in [[Str15]](http://www.stroustrup.com/resource-model.pdf).

So, we write a class

```c++
    class Istream { [[gsl::suppress(lifetime)]]
    public:
        enum Opt { from_line = 1 };
        Istream() { }
        Istream(zstring p) :owned{true}, inp{new ifstream{p}} {}            // read from file
        Istream(zstring p, Opt) :owned{true}, inp{new istringstream{p}} {}  // read from command line
        ~Istream() { if (owned) delete inp; }
        operator istream& () { return *inp; }
    private:
        bool owned = false;
        istream* inp = &cin;
    };
```

Now, the dynamic nature of `istream` ownership has been encapsulated.
Presumably, a bit of checking for potential errors would be added in real code.

##### Enforcement

* Hard, it is hard to decide what rule-breaking code is essential
* Flag rule suppression that enable rule-violations to cross interfaces
