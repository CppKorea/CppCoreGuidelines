
# <a name="S-const"></a>Con: 상수와 불변성

상수에 대해서는 경쟁 상태가 발생하지 않는다.
프로그램 내의 객체 중 그 값이 바뀔 수 없는 것이 많다면 해석하기가 쉬울 것이다.
인자로 넘어가는 개체의 상태가 바뀌지 않는 것을 보장한다면 그 인터페이스는 가독성이 굉장히 높을 것이다.

상수 규칙 요약:

* [Con.1: 기본적으로 객체를 변경 불가능하도록 만들어라](#Rconst-immutable)
* [Con.2: 기본적으로 멤버 함수들은 `const`로 만들어라](#Rconst-fct)
* [Con.3: 기본적으로 포인터와 참조는 `const`로 전달하라](#Rconst-ref)
* [Con.4: 개체 생성 이후 변하지 않는 값은 `const`로 정의하라](#Rconst-const)
* [Con.5: 컴파일 시간에 계산될 수 있는 값은 `constexpr`을 사용하라](#Rconst-constexpr)

### <a name="Rconst-immutable"></a>Con.1: 기본적으로 객체를 변경 불가능하도록 만들어라

##### Reason

변하지 않는 개체는 이해하기가 더 쉽다. 값을 바꿀 필요가 있을 때에만 개체를 비-`const`로 만들어라.

의도하지 않았거나 알아차리기 어려운 값 변경을 예방한다.

##### Example

```c++
    for (const int i : c) cout << i << '\n';    // just reading: const

    for (int i : c) cout << i << '\n';          // BAD: just reading
```

##### Exception

함수의 인자들의 값이 변경되는 경우는 드물지만, 또한 이들이 const 로 선언되는 경우 역시 드물다.
혼동과 수많은 거짓 양성 (false positives) 테스트 결과를 피하기 위해, 이 규칙은 함수의 인자들에 대해서는 적용하지 않도록 한다.

```c++
    void f(const char* const p); // pedantic
    void g(const int i);        // pedantic
```

함수 내부의 변수는 지역 변수이므로 이를 변경하는 영향도 지역적임을 짚고 가자.

##### Enforcement

* 값이 변경되지 않는 비-`const` 변수들을 지적하라 (수많은 거짓 양성 테스트 결과를 피하기 위해 인자들에 대해서는 예외로 한다)

### <a name="Rconst-fct"></a>Con.2: 기본적으로 멤버 함수들은 `const`로 만들어라

##### Reason

멤버 함수가 해당 객체의 상태를 변경하지 않는다면 `const`로 표시하도록 한다.
이렇게 하면 디자인 의도를 더욱 명확하게 드러내고, 가독성을 높이며, 컴파일러가 더 많은 오류를 잡아낼 수 있게 도우며, 때에 따라서는 최적화를 더 잘 할 수 있도록 한다.

##### Example; bad

```c++
    class Point {
        int x, y;
    public:
        int getx() { return x; }    // BAD, should be const as it doesn't modify the object's state
        // ...
    };

    void f(const Point& pt) {
        int x = pt.getx();          // ERROR, doesn't compile because getx was not marked const
    }
```

##### Note

비-`const` 형식의 포인터나 참조자를 전달하는 것 자체가 근본적으로 나쁜 것은 아니지만,
이는 호출되는 함수가 해당 객체를 수정하는 경우에만 수행되어야 한다.
코드를 읽는 독자는 "plain" `T*` 나 `T&` 를 전달받는 함수가 해당 참조가 가리키는 객체를 수정할 것이라고 가정해야 한다.
비록 당장 현재에는 객체를 수정을 하지 않더라도, 이후에 재컴파일을 강제하지 않고도 객체를 수정하도록 변경될 수 있다.

##### Note

기존 코드/라이브러리 중 일부는 함수 내에서 `T`를 수정하지 않으면서도
`T*` 로 선언된 함수를 제공하기도 한다.
이는 코드를 "현대화(modernizing)" 하려는 사람들에게 문제가 될 수 있다.
다음 방법들을 사용할 수 있다

* 정확하게 `const`를 이용하도록 라이브러리를 업데이트한다; 장기적인 대안으로 좋다
* "`const`속성을 타입 변환(cast)을 통해 해지한다"; [가능한 피하는 것이 좋다](#Res-casts-const)
* wrapper 함수를 제공한다

Example:

```c++
    void f(int* p);   // old code: f() does not modify `*p`
    void f(const int* p) { f(const_cast<int*>(p)); } // wrapper
```

이렇게 wrapper를 이용하는 해결책은(라이브러리 안에 들어 있어 직접 수정할 수 없다든가 하는 이유로)
f()의 선언을 수정할 수 없는 경우에만 사용해야 하는 임시방편임에 유의하라.

##### Note

`const` 멤버 함수도 `mutable` 하거나 포인터 멤버를 통해 접근되는 객체에 대해서는 값을 수정할 수 있다.
이러한 방식의 흔한 예는 복잡한 계산을 반복적으로 하지 않기 위해 캐시를 유지하는 경우이다.
예를 들어, 아래 `Date` 는 반복적인 사용을 단순화하기 위해 날짜의 문자열 표현을 캐시(기억) 하고 있다:

```c++
    class Date {
    public:
        // ...
        const string& string_ref() const
        {
            if (string_val == "") compute_string_rep();
            return string_val;
        }
        // ...
    private:
        void compute_string_rep() const;    // compute string representation and place it in string_val
        mutable string string_val;
        // ...
    };
```

이에 대해서 다른 방식으로 이야기하면, `const`함은 전이적(transitive) 이지 않다는 것이다.
`const` 멤버 함수에게 있어 `mutable` 한 멤버의 값과
비-`const` 포인터를 통해 접근하는 객체의 값을 변경할 수 있기 때문이다.
클래스가 사용자들에게 제공하는 의미론(불변 조건)에 의거하여 타당하다고 여겨질 경우에만 이러한 값 변경이 이루어질 수 있도록 보장하는 것은
해당 클래스의 역할이다.

**See also**: [Pimpl](#Ri-pimpl)

##### Enforcement

* `const` 표시가 되어 있지 않지만, 어떠한 멤버 변수에 대해서도 비-`const` 작업을 수행하지 않는 멤버 함수를 지적하라

### <a name="Rconst-ref"></a>Con.3: 기본적으로 포인터와 참조는 `const`로 전달하라

##### Reason

 호출되는 함수가 예상치 못하게 값을 변경하는 것을 방지한다.
 호출되는 함수가 상태를 변경하지 않는다면 프로그램의 동작을 예상하기가 훨씬 수월하다.

##### Example

```c++
    void f(char* p);        // does f modify *p? (assume it does)
    void g(const char* p);  // g does not modify *p
```

##### Note

비-`const` 형식의 포인터나 참조자를 전달하는 것 자체가 근본적으로 나쁜 것은 아니지만,
이는 호출되는 함수가 해당 객체를 수정하는 경우에만 수행되어야 한다.

##### Note

[`const`속성을 타입 변환을 통해 없애지 말라](#Res-casts-const).

##### Enforcement

* 객체에 대한 포인터나 참조자를 비-`const` 형식으로 전달받지만 해당 객체를 수정하지는 않는 함수들을 지적하라
* 객체에 대한 포인터나 참조자를 `const` 형식으로 전달받지만 (타입 변환을 통해) 해당 객체를 수정하는 함수를 지적하라

### <a name="Rconst-const"></a>Con.4: 개체 생성 이후 변하지 않는 값은 `const`로 정의하라

##### Reason

 예기치 않은 객체의 값 변경으로 인해 뜻밖의 결과가 발생하는 것을 방지한다.

##### Example

```c++
    void f()
    {
        int x = 7;
        const int y = 9;

        for (;;) {
            // ...
        }
        // ...
    }
```

`x` 가 `const` 형식으로 선언되지 않았으므로, 해당 변수는 반복문 내 어디선가 값이 변경된다고 가정해야 한다.

##### Enforcement

* 변경되지 않는 비-`const` 변수들을 지적하라.

### <a name="Rconst-constexpr"></a>Con.5: 컴파일 시간에 계산될 수 있는 값은 `constexpr`을 사용하라

##### Reason

성능이 향상되고, 컴파일-시점 검사가 원활해지며, 컴파일-시점 연산(evaluation)이 보장되며, 경합 조건 (race condition) 위험도 피할 수 있다.

##### Example

```c++
    double x = f(2);            // possible run-time evaluation
    const double y = f(2);      // possible run-time evaluation
    constexpr double z = f(2);  // error unless f(2) can be evaluated at compile time
```

##### Note

See F.4.

##### Enforcement

* 상수 표현식을 통해 초기화되는 `const` 선언을 지적하라.