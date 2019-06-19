
# <a name="S-philosophy"></a>P: 철학

이 장의 규칙들은 매우 일반적이다.

철학 규칙 요약:

* [P.1: 아이디어를 직접 코드로 표현하라](#Rp-direct)
* [P.2: ISO 표준 C++로 작성하라](#Rp-Cplusplus)
* [P.3: 의도를 표현하라](#Rp-what)
* [P.4: 이상적으로 프로그램은 정적으로 타입 안전해야 한다](#Rp-typesafe)
* [P.5: 런타임 검사보다는 컴파일 타임 검사를 선호하라](#Rp-compile-time)
* [P.6: 컴파일 타임에 검사할 수 없다면 런타임에 검사할 수 있어야 한다](#Rp-run-time)
* [P.7: 런타임 오류는 초기에 잡아라](#Rp-early)
* [P.8: 리소스가 새도록 하지 마라](#Rp-leak)
* [P.9: 시간이나 공간을 낭비하지 마라](#Rp-waste)
* [P.10: 변경 가능한 데이터보다 변경 불가능한 데이터를 더 자주 사용하라](#Rp-mutable)
* [P.11: 복잡한 생성과정은 캡슐하라](#Rp-library)
* [P.12: 지원 도구를 적절히 활용하라](#Rp-tools)
* [P.13: 지원 라이브러리를 적절히 활용하라](#Rp-lib)

철학적 규칙은 보통 기계적으로 검사할 수 **없다**.
그러나 철학적인 테마를 반영하는 개별적인 규칙들은 검사할 수 하다.
철학적인 기초가 없이 구체적이고/특수하고/검사 가능한 규칙은 근거가 부족하다.

### <a name="Rp-direct"></a>P.1: 아이디어를 직접 코드로 표현하라

##### Reason

컴파일러는 주석문(또는 디자인 문서)을 읽지 않는다. 수많은 프로그래머 또한 주석을 (일관되게) 읽지 않는다.
코드로 표현된 내용이라면 그 의미(의도)를 이미 정의했을 것이며 (대체로) 컴파일러나 다른 툴로 검사할 수 있다.

##### Example

```c++
    class Date {
        // ...
    public:
        Month month() const;  // do
        int month();          // don't
        // ...
    };
```

첫번째 `month` 함수는 명확히 `Month`를 반환하도록 선언되어 있으며, `Date` 개체의 상태를 변경하지 않을 것처럼 보인다.
두번째 버전은 코드를 읽는 개발자들을 고민하게 만들며, 발견하기 어려운 버그를 유발할 가능성이 있다.

##### Example; bad

아래의 반복문은 `std::find`를 이용해 표현 가능하다.

```c++
    void f(vector<string>& v)
    {
        string val;
        cin >> val;
        // ...
        int index = -1;                    // bad, plus should use gsl::index
        for (int i = 0; i < v.size(); ++i) {
            if (v[i] == val) {
                index = i;
                break;
            }
        }
        // ...
    }
```

##### Example; good

의도를 더 명확하게 드러내기 위해 아래와 같이 코드를 바꿀 수 있다:

```c++
    void f(vector<string>& v)
    {
        string val;
        cin >> val;
        // ...
        auto p = find(begin(v), end(v), val);  // better
        // ...
    }
```

언어가 제공하는 기능을 직접 사용하기보다 잘 설계된 라이브러리를 사용해 그 의도(어떻게 수행되는지보다 무엇이 수행되는지를) 표현하는 것이 훨씬 낫다.

C++ 프로그래머는 표준 라이브러리의 기본 내용을 반드시 이해하고 올바른 곳에 사용해야 한다.
어떤 프로그래머든 프로젝트에 기반하고 있는 핵심 라이브러리의 기본 내용을 반드시 이해하고 있어야 하며, 올바르게 사용할 줄 알아야 한다.
이 가이드라인을 사용하는 프로그래머는 [가이드라인 지원 라이브러리](#S-gsl)을 반드시 알아야 하고 적절히 사용할 줄 알아야 한다.

##### Example

```c++
    change_speed(double s);   // bad: what does s signify?
    // ...
    change_speed(2.3);
```

더 좋은 접근법은 `double`의 의미와 단위(새로운 속도 값인지, 혹은 이전 속도에서의 증분을 의미하는지)를 명확히 하는 것이다:

```c++
    change_speed(Speed s);    // better: the meaning of s is specified
    // ...
    change_speed(2.3);        // error: no unit
    change_speed(23m / 10s);  // meters per second
```

(단위가 없는) 단순한 `double`을 변화량(속도의 차)으로 받아들일 수도 있겠지만, 아무래도 오류가 발생하기 쉽다.
속도와 변화량 둘 다 필요하다면, `Delta` 타입을 정의해야 할 것이다.

##### Enforcement

일반적으로 매우 어렵다.

* 일관성 있게 `const`를 사용하라 (멤버 함수가 개체를 변경하는지 확인하라. 그리고 포인터나 레퍼런스로 넘어온 인자를 변경하는지 확인하라)
* 타입 변환의 사용을 지적하라 (타입 변환은 타입 시스템을 무력화시킨다)
* 표준 라이브러리를 흉내내는 코드를 찾아라 (찾기 어렵다)

### <a name="Rp-Cplusplus"></a>P.2: ISO 표준 C++로 작성하라

##### Reason

이 문서는 ISO 표준 C++로 작성하는 가이드라인들을 모아둔 것이다.

##### Note

시스템 리소스에 접근하는 등의 작업을 수행하기 위해서 확장 기능이 필요할 수 있다.
이런 경우에는 필요한 확장 기능을 지역적으로 제한해서 사용하고, 핵심 가이드라인이 아닌 다른 코딩 가이드라인을 활용해 관리하라. If possible, build interfaces that encapsulate the extensions so they can be turned off or compiled away on systems that do not support those extensions.

Extensions often do not have rigorously defined semantics.  Even extensions that
are common and implemented by multiple compilers may have slightly different
behaviors and edge case behavior as a direct result of *not* having a rigorous
standard definition.  With sufficient use of any such extension, expected
portability will be impacted.

##### Note

Using valid ISO C++ does not guarantee portability (let alone correctness).
Avoid dependence on undefined behavior (e.g., [undefined order of evaluation](#Res-order))
and be aware of constructs with implementation defined meaning (e.g., `sizeof(int)`).

##### Note

표준 C++ 언어의 기능이나 라이브러리조차 제한적으로 사용할 수 밖에 없는 환경도 있다.
예를 들면, 항공기 제어 소프트웨어 개발 표준에는 동적 메모리 할당을 피할 것을 주문하고 있다.
이런 경우에는 특정 환경에 맞춘 코딩 가이드라인을 확장해서 사용하거나 사용하지 않아야 하는 기능들을 관리하라.

##### Enforcement

확장을 허용하지 않도록 기능 설정이 가능한 최신 C++ 컴파일러(C++17, C++14 혹은 C++11)를 사용하라.

### <a name="Rp-what"></a>P.3: 의도를 표현하라

##### Reason

(이름이나 주석을 통해) 코드의 의도를 제대로 드러내지 못한다면, 코드가 제대로 수행되는지조차 말할 수 없을 것이다.

##### Example

```c++
    gsl::index i = 0;
    while (i < v.size()) {
        // ... do something with v[i] ...
    }
```

위 코드만 보았을 때는 `v`의 각 요소를 순회하겠다는 의도가 드러나지 않는다.
인덱스에 대한 세부적인 구현부가 노출된다 (따라서 잘못 사용될 지도 모른다). 그리고 의도적인지는 알 수 없지만 `i`를 반복문 밖에서도 여전히 사용할 수 있다. 이 부분의 코드만 읽어서는 그 의도를 알지 못한다.

개선:

```c++
    for (const auto& x : v) { /* do something with the value of x */ }
```

위 코드를 보면 v에 대한 순회 메커니즘을 명시적으로 언급하지 않는다. 그리고 순회하는 동안 v의 각 `const` 요소의 레퍼런스에 대해 동작이 일어나기 때문에 각 요소를 수정할 수 없다. 만약 수정이 필요한 경우라면 아래와 같이 코드를 작성해야 한다.

```c++
    for (auto& x : v) { /* modify x */ }
```

For more details about for-statements, see [ES.71](#Res-for-range).
Sometimes better still, use a named algorithm. This example uses the `for_each` from the Ranges TS because it directly expresses the intent:

```c++
    for_each(v, [](int x) { /* do something with the value of x */ });
    for_each(par, v, [](int x) { /* do something with the value of x */ });
```

마지막 예는 `v`의 각 요소가 처리되는 순서에 관심이 없다는 점을 명확히 하고 있다.

프로그래머라면 다음에 익숙해져야 한다.

* [가이드라인 지원 라이브러리](#S-gsl)
* [ISO C++ 표준 라이브러리](#S-stdlib)
* 현재 프로젝트에서 사용되고 있는 모든 기본 라이브러리들

##### Note

공식 대안: 어떻게 작업이 수행되는지를 말하지 말고 무엇이 수행될지를 말하라.

##### Note

언어의 기본 요소는 다른 무엇보다도 그 의도를 더 잘 표현한다.

##### Example

2개의 `int` 값으로 2차원 좌표를 표현하고 싶다면, 다음과 같이 써라:

```c++
    draw_line(int, int, int, int);  // obscure
    draw_line(Point, Point);        // clearer
```

##### Enforcement

좀 더 나은 대안이 있는 범용적인 패턴을 찾아보라.

* 단순한 `for`문 대 범위 기반 `for`문
* `f(T*, int)` 인터페이스 대 `f(span<T>)` 인터페이스
* 아주 큰 스코프에서 순회하는 변수
* 그대로 노출된 `new`와 `delete`
* 인자로 내장 타입을 여러개 갖는 함수

There is a huge scope for cleverness and semi-automated program transformation.

### <a name="Rp-typesafe"></a>P.4: 이상적으로 프로그램은 정적으로 타입 안전해야 한다

##### Reason

이상적으로 프로그램은 완전히 정적으로 타입 안전해야 한다.
하지만 불행하게도 불가능하다. 왜냐하면 다음처럼 문제가 되는 영역들이 존재하기 때문이다.

* 공용체
* 타입 변환
* 배열 붕괴
* 범위 오류
* 축소 타입 변환

##### Note

이러한 영역들은 심각한 문제의 원인이 된다. (예를 들어, 크래시와 보안 위반)
따라서 다른 기법을 제공하고자 한다.

##### Enforcement

각 프로그램에 대해 필요하고 실현 가능하도록 각 문제 범주를 개별적으로 금지, 억제 또는 탐지할 수 있다.
항상 대안을 제시하라.

예시:

* 공용체 -- (C++17에 있는) `variant`를 사용하라.
* 타입 변환 -- 사용을 최소화하라. 템플릿이 도움이 될 수 있다.
* 배열 붕괴 -- (GSL에 있는) `span`을 사용하라.
* 범위 오류 -- `span`을 사용하라.
* 축소 타입 변환 -- 사용을 최소화하라. 필요하면 `narrow`나 (GSL에 있는) `narrow_cast`를 사용하라.

### <a name="Rp-compile-time"></a>P.5:런타임 검사보다는 컴파일 타임 검사를 선호하라

##### Reason

코드 명확성, 성능 향상.
컴파일 타임에 발견되는 오류에 대해서는 처리하는 부분을 따로 작성할 필요가 없다.

##### Example

```c++
    // Int is an alias used for integers
    int bits = 0;         // don't: avoidable code
    for (Int i = 1; i; i <<= 1)
        ++bits;
    if (bits < 32)
        cerr << "Int too small\n";
```

This example fails to achieve what it is trying to achieve (because overflow is undefined) and should be replaced with a simple `static_assert`:

```c++
    // Int is an alias used for integers
    static_assert(sizeof(Int) >= 4);    // do: compile-time check
```

Or better still just use the type system and replace `Int` with `int32_t`.

##### Example

```c++
    void read(int* p, int n);   // read max n integers into *p

    int a[100];
    read(a, 1000);    // bad, off the end
```

better

```c++
    void read(span<int> r); // read into the range of integers r

    int a[100];
    read(a);        // better: let the compiler figure out the number of elements
```

**Alternative formulation**: 컴파일 타임에 할 수 있는 것을 런타임으로 연기하지 마라.

##### Enforcement

* Look for pointer arguments.
* Look for run-time checks for range violations.

### <a name="Rp-run-time"></a>P.6: 컴파일 타임에 검사할 수 없다면 런타임에 검사할 수 있어야 한다

##### Reason

프로그램 안에 찾기 어려운 오류를 남겨둔다면 크래시나 나쁜 결과를 야기한다.

##### Note

이상적으로 우리는 컴파일 타임, 런타임에 (프로그래머의 논리에서는 오류가 아닌) 모든 오류를 찾을 수 있다.
컴파일 타임에 모든 오류를 찾아내는 건 불가능하고 런타임에 남아 있는 모든 오류를 찾는 것도 불가능하다.
그러나 충분한 리소스를 준다면 원론적으로 검사 가능한 프로그램을 작성하려고 노력해야 한다. (분석 프로그램, 런타임 검사, 컴퓨터 리소스, 시간)

##### Example, bad

```c++
    // separately compiled, possibly dynamically loaded
    extern void f(int* p);

    void g(int n)
    {
        // bad: the number of elements is not passed to f()
        f(new int[n]);
    }
```

여기서 결정적인 정보(요소의 갯수)가 아주 철저하게 숨겨져 있어서, `f()`가 ABI의 일부일 때 정적 분석은 아마도 불가능해 보이고 동적 검사는 매우 어려울 수 있다. 따라서 해당 포인터를 "측정"할 수 없다.
도움이 될만한 정보를 남은 공간에 넣을 수 있지만, 이는 시스템이나 컴파일러에게 전반적인 변경을 요구한다.
예제에 있는 코드는 오류 발견을 아주 어렵게 만드는 디자인이다.

##### Example, bad

We can of course pass the number of elements along with the pointer:

```c++
    // separately compiled, possibly dynamically loaded
    extern void f2(int* p, int n);

    void g2(int n)
    {
        f2(new int[n], m);  // bad: a wrong number of elements can be passed to f()
    }
```

인자로 요소의 갯수를 전달하는 것은 포인터를 전달하면서 관례에 따라 요소의 갯수를 구하는 것보다 낫다.
그러나, 단순히 철자 하나만 틀려도 심각한 오류를 야기한다. `f2()`에서 두 인자간의 연결은 구체적이지 않고 관례에 따른 것이다.

게다가 `f2()`가 인자를 `delete`할 것인지 알 수 없다. (호출자가 두번째 실수를 한 것인가?)

##### Example, bad

표준 라이브러리에 있는 리소스 관리 포인터는 개체를 가리키고 있을 때 크기를 넘길 수 없다:

```c++
    // separately compiled, possibly dynamically loaded
    // NB: this assumes the calling code is ABI-compatible, using a
    // compatible C++ compiler and the same stdlib implementation
    extern void f3(unique_ptr<int[]>, int n);

    void g3(int n)
    {
        f3(make_unique<int[]>(n), m);    // bad: pass ownership and size separately
    }
```

##### Example

포인터와 요소의 갯수를 하나의 개체로 합쳐서 전달해야 한다:

```c++
    extern void f4(vector<int>&);   // separately compiled, possibly dynamically loaded
    extern void f4(span<int>);      // separately compiled, possibly dynamically loaded
                                    // NB: this assumes the calling code is ABI-compatible, using a
                                    // compatible C++ compiler and the same stdlib implementation

    void g3(int n)
    {
        vector<int> v(n);
        f4(v);                     // pass a reference, retain ownership
        f4(span<int>{v});          // pass a view, retain ownership
    }
```

이 디자인은 개체의 필수 부분으로 요소의 갯수를 전달하므로 오류가 발생하지 않고 항상 저렴하지는 않지만 동적(런타임) 검사를 할 수 있다.

##### Example

올바른 사용을 위해 어떻게 소유권과 모든 정보를 전달할 것인가?

```c++
    vector<int> f5(int n)    // OK: move
    {
        vector<int> v(n);
        // ... initialize v ...
        return v;
    }

    unique_ptr<int[]> f6(int n)    // bad: loses n
    {
        auto p = make_unique<int[]>(n);
        // ... initialize *p ...
        return p;
    }

    owner<int*> f7(int n)    // bad: loses n and we might forget to delete
    {
        owner<int*> p = new int[n];
        // ... initialize *p ...
        return p;
    }
```

##### Example

* ???
* 필요한 것을 실제로 알고 있을 때, 다형 기본 클래스를 전달하는 인터페이스가 어떻게 검사를 피할 수 있는지 보여준다.  
  또는 "자유형" 옵션으로 문자열이 있다.

##### Enforcement

* (포인터, 갯수)-스타일 인터페이스라면 표시한다. (호환성을 이유로 고칠 수 없는 많은 예제를 표시할 것이다.)
* ???

### <a name="Rp-early"></a>P.7: 런타임 오류는 초기에 잡아라

##### Reason

"미스터리"한 크래시를 피한다.
(아마 몰랐을 수도 있는) 잘못된 결과를 야기하는 오류를 피한다.

##### Example

```c++
    void increment1(int* p, int n)    // bad: error-prone
    {
        for (int i = 0; i < n; ++i) ++p[i];
    }

    void use1(int m)
    {
        const int n = 10;
        int a[n] = {};
        // ...
        increment1(a, m);   // maybe typo, maybe m <= n is supposed
                            // but assume that m == 20
        // ...
    }
```

여기서 우리는 `use1`에 데이터가 손실되거나 크래시를 야기할 수 있는 작은 오류를 범했다.
(포인터, 크기)-스타일 인터페이스는 범위 오류에 대해 `increment1()`에서 방어할 수 있는 현실적인 방안을 없애버린다.
배열 첨자가 범위를 벗어나는지 검사한다고 가정하면, `p[10]`까지 오류가 발견되지 않을 것이다.
좀 더 빨리 검사하도록 코드를 개선해 보자:

```c++
    void increment2(span<int> p)
    {
        for (int& x : p) ++x;
    }

    void use2(int m)
    {
        const int n = 10;
        int a[n] = {};
        // ...
        increment2({a, m});    // maybe typo, maybe m <= n is supposed
        // ...
    }
```

이제 `m<=n`은 호출 시점에서 (일찍) 확인할 수 있다.
우리가 가진 모든 것이 오타이므로 `n`을 범위로 사용한다면, 코드를 더 단순하게 만들 수 있다. (오류의 가능성 제거)

```c++
    void use3(int m)
    {
        const int n = 10;
        int a[n] = {};
        // ...
        increment2(a);   // the number of elements of a need not be repeated
        // ...
    }
```

##### Example, bad

동일한 값을 반복적으로 검사하지 마라. 구조화된 데이터를 문자열로 넘기지 마라:

```c++
    Date read_date(istream& is);    // read date from istream

    Date extract_date(const string& s);    // extract date from string

    void user1(const string& date)    // manipulate date
    {
        auto d = extract_date(date);
        // ...
    }

    void user2()
    {
        Date d = read_date(cin);
        // ...
        user1(d.to_string());
        // ...
    }
```

(`Date` 생성자에 의해) 날짜가 두 번 계산되고 (비구조화된 데이터인) 문자열로 전달된다.

##### Example

지나친 검사는 비용이 많이 든다.
값이 필요한지도 모르기 때문에 일찍 검사하는 것이 안 좋은 경우도 있고 전체가 아닌 값의 일부만 검사하는 것이 쉬운 경우도 있다.
Similarly, don't add validity checks that change the asymptotic behavior of your interface (e.g., don't add a `O(n)` check to an interface with an average complexity of `O(1)`).

```c++
    class Jet {    // Physics says: e * e < x * x + y * y + z * z
        float x;
        float y;
        float z;
        float e;
    public:
        Jet(float x, float y, float z, float e)
            :x(x), y(y), z(z), e(e)
        {
            // Should I check here that the values are physically meaningful?
        }

        float m() const
        {
            // Should I handle the degenerate case here?
            return sqrt(x * x + y * y + z * z - e * e);
        }

        ???
    };
```

제트기에 대한 물리적 법칙(`e * e < x * x + y * y + z * z`)은 측정 오류의 가능성 때문에 값이 바뀔 수 있다.

???

##### Enforcement

* 포인터와 배열을 찾아라: 범위를 빨리 검사하고 반복되지 않게 하라.
* 타입 변환을 찾아라: 축소 변환을 표시하거나 제거하라.
* 입력된 값 중 검사되지 않은 값을 찾아라.
* 문자열로 변환되고 있는 구조화된 데이터(불변 조건을 갖는 클래스의 개체)를 찾아라.
* ???

### <a name="Rp-leak"></a>P.8: 리소스가 새도록 하지 마라

##### Reason

Even a slow growth in resources will, over time, exhaust the availability of those resources.
This is particularly important for long-running programs, but is an essential piece of responsible programming behavior.

##### Example, bad

```c++
    void f(char* name)
    {
        FILE* input = fopen(name, "r");
        // ...
        if (something) return;   // bad: if something == true, a file handle is leaked
        // ...
        fclose(input);
    }
```

[RAII](#Rr-raii)를 사용한 개선:

```c++
    void f(char* name)
    {
        ifstream input {name};
        // ...
        if (something) return;   // OK: no leak
        // ...
    }
```

**See also**: [리소스 관리](#S-resource)

##### Note

A leak is colloquially "anything that isn't cleaned up."
The more important classification is "anything that can no longer be cleaned up."
For example, allocating an object on the heap and then losing the last pointer that points to that allocation.
This rule should not be taken as requiring that allocations within long-lived objects must be returned during program shutdown.
For example, relying on system guaranteed cleanup such as file closing and memory deallocation upon process shutdown can simplify code.
However, relying on abstractions that implicitly clean up can be as simple, and often safer.

##### Note

Enforcing [the lifetime safety profile](#SS-lifetime) eliminates leaks.
When combined with resource safety provided by [RAII](#Rr-raii), it eliminates the need for "garbage collection" (by generating no garbage).
Combine this with enforcement of [the type and bounds profiles](#SS-force) and you get complete type- and resource-safety, guaranteed by tools.

##### Enforcement

* 포인터를 살펴봐라: 소유자와 비소유자로 구분해라.
  가능하다면 소유자를 (위의 예제처럼) 표준 라이브러리 리소스 핸들로 바꿔라.
  또는 [GSL](#S-gsl)에서 `owner`를 사용하는 것처럼 소유자를 표시하라.
* 처리되지 않은 `new`, `delete`를 찾아라.
* 처리되지 않은 포인터를 반환하는 잘 알려진 리소스 할당 함수를 찾아라. (예를 들어, `fopen`, `malloc`, `strdup`)

### <a name="Rp-waste"></a>P.9: 시간이나 공간을 낭비하지 마라

##### Reason

이것이 C++이다.

##### Note

어떤 목표(예를 들어, 개발 속도, 리소스 안전성, 또는 테스트 단순화)를 달성하기 위해 소모하는 시간과 공간은 낭비가 아니다. "효율성을 위해 노력할 때 얻을 수 있는 또다른 이점은 그러한 행위가 문제를 더 깊이 이해할 수 있도록 강제한다는 겁니다." - Alex Stepanov

##### Example, bad

```c++
    struct X {
        char ch;
        int i;
        string s;
        char ch2;

        X& operator=(const X& a);
        X(const X&);
    };

    X waste(const char* p)
    {
        if (!p) throw Nullptr_error{};
        int n = strlen(p);
        auto buf = new char[n];
        if (!buf) throw Allocation_error{};
        for (int i = 0; i < n; ++i) buf[i] = p[i];
        // ... manipulate buffer ...
        X x;
        x.ch = 'a';
        x.s = string(n);    // give x.s space for *p
        for (gsl::index i = 0; i < x.s.size(); ++i) x.s[i] = buf[i];  // copy buf into x.s
        delete[] buf;
        return x;
    }

    void driver()
    {
        X x = waste("Typical argument");
        // ...
    }
```

그렇다. 풍자를 위한 예제이기는 하지만, 실제 코드에서 이보다 심각한 실수도 본 적이 있다.
`X`의 레이아웃에 (더 많을지도 모르지만) 적어도 6바이트의 낭비가 있다는 점을 주목하라.
복사 연산을 그럴싸하게 정의해 두다 보니 이동 연산이 비활성화 됨으로써 반환 연산이 느려졌다(여기에선 반환값 최적화(Return Value Optimization, RVO)가 보장되지 않음에 주목하라).
`buf`에서 `new`, `delete`의 사용이 중복된다. 진짜 지역 문자열을 원했다면, `string` 지역 변수를 사용했을 것이다.
더 많은 성능 버그와 상황을 더 복잡하게 만드는 불필요한 문제가 있다.

##### Example, bad

```c++
    void lower(zstring s)
    {
        for (int i = 0; i < strlen(s); ++i) s[i] = tolower(s[i]);
    }
```

그렇다, 이것은 실제 코드로부터 가져온 예제이다.
우리는 이 예제를 독자가 직접 무엇이 낭비되는지 찾게 하기 위해 남겨둔다.

##### Note

낭비에 대한 각 예제는 별로 중요하지 않다. 그리고 중요했다면 이미 전문가들이 쉽게 제거했을 것이다.
그러나 코드 전체에 걸쳐 낭비가 퍼져버리면 중요해 질 수 있고, 낭비를 제거하기 위해서 전문가들을 항상 원하는데로 데려올 수는 없다.
이 규칙(그리고 보다 구체적인 규칙)의 목적은 C++ 사용과 관련된 대부분의 낭비를 발생하기 전에 없애기 위해서다.
그 후에 알고리즘이나 요구 사항과 관련된 낭비를 살펴볼 수 있다. 하지만 그건 이 가이드라인의 범위를 벗어난다.

##### Enforcement

더 많은 특정 규칙들은 쓸데없는 낭비의 단순화와 제거를 전반적인 목표로 하고 있다.
* 사용자가 정의한 기본 정의가 아닌 접미 연산자 ++ 또는 -- 함수의 사용하지 않은 반환값에 표시를 한다. 대신 접두 연산자를 사용하는 것이 좋다. (주의: "사용자가 정의한 기본 정의가 아닌"은 잡음을 줄이려는 의도가 있다. 실사용에서 여전히 너무 잡음이 많을때, 이 시행을 검토하라.)

### <a name="Rp-mutable"></a>P.10: 변경 가능한 데이터보다 불변의 데이터를 선호하라

##### Reason

변수보다는 상수에 대한 것이 추론을 하기가 더 쉽다.
불변한 것은 예상치 못하게 변하지 않는다.
때때로 불변성은 더 나은 최적화가 가능하게 한다.
당신은 상수를 경쟁상태(Race condition)로 만들 수 없다.

[Con: 상수와 불변성](./Const.md)을 참조하라.

### <a name="Rp-library"></a>P.11: 지저분한 구조가 코드를 통해 퍼지기 보단, 캠슐화를 하라.

##### Reason

지저분한 코드는 버그를 숨기고 쓰기가 더 어렵다.
좋은 인터페이스는 사용하기 더 쉽고 더 안전하다.
지저분한, 저수준의 코드는 그런 코드를 더 많이 낳는다.

##### Example

```c++
    int sz = 100;
    int* p = (int*) malloc(sizeof(int) * sz);
    int count = 0;
    // ...
    for (;;) {
        // ... read an int into x, exit loop if end of file is reached ...
        // ... check that x is valid ...
        if (count == sz)
            p = (int*) realloc(p, sizeof(int) * sz * 2);
        p[count++] = x;
        // ...
    }
```

이 것은 저수준이고, 장황하며, 에러가 발생하기 쉽다.
예를 들어, 우리는 메모리 소모를 테스트하는 것을 잊었다.
대신, 우리는 `vector`를 사용할 수 있었다:

```c++
    vector<int> v;
    v.reserve(100);
    // ...
    for (int x; cin >> x; ) {
        // ... check that x is valid ...
        v.push_back(x);
    }
```

##### Note

표준 라이브러리와 GSL은 이런 철학의 예들 이다.
예를 들어, `vector`, `span`, `lock_guard`, `future`와 같이 핵심적인 추상화 구현에 필요한, 
배열들과 유니온들, 캐스팅, 까다로운 생명주기 문제, `gsl::owner` 등으로 지저분하게 하는 대신, 
우리는 우리보다 전문적이고 더 많은 시간을 투자한 사람들이 설계하고 구현한 라이브러리들을 사용한다.
마찬가지로, 우리는 사용자들(흔히 우리 자신)을 반복적으로 낮은 수준의 코드를 얻기 쉬운 환경에 버려두지 말고,
더 전문화된 라이브러리들을 설계하고 구현해야하고 할 수 있다.
이것은 본 지침의 기초가 되는 [subset of superset principle](Introduction.md#R0)의 변형이다.

##### Enforcement

* 복잡한 포인터 조작이나 추상화 구현 밖에서의 캐스팅 같은 "지저분한 코드"를 찾아봐라.

### <a name="Rp-tools"></a>P.12: Use supporting tools as appropriate

##### Reason

There are many things that are done better "by machine".
Computers don't tire or get bored by repetitive tasks.
We typically have better things to do than repeatedly do routine tasks.

##### Example

Run a static analyzer to verify that your code follows the guidelines you want it to follow.

##### Note

See

* [Static analysis tools](???)
* [Concurrency tools](#Rconc-tools)
* [Testing tools](???)

There are many other kinds of tools, such as source code repositories, build tools, etc.,
but those are beyond the scope of these guidelines.

##### Note

Be careful not to become dependent on over-elaborate or over-specialized tool chains.
Those can make your otherwise portable code non-portable.

### <a name="Rp-lib"></a>P.13: Use support libraries as appropriate

##### Reason

Using a well-designed, well-documented, and well-supported library saves time and effort;
its quality and documentation are likely to be greater than what you could do
if the majority of your time must be spent on an implementation.
The cost (time, effort, money, etc.) of a library can be shared over many users.
A widely used library is more likely to be kept up-to-date and ported to new systems than an individual application.
Knowledge of a widely-used library can save time on other/future projects.
So, if a suitable library exists for your application domain, use it.

##### Example

```c++
    std::sort(begin(v), end(v), std::greater<>());
```

Unless you are an expert in sorting algorithms and have plenty of time,
this is more likely to be correct and to run faster than anything you write for a specific application.
You need a reason not to use the standard library (or whatever foundational libraries your application uses) rather than a reason to use it.

##### Note

By default use

* The [ISO C++ Standard Library](#S-stdlib)
* The [Guidelines Support Library](#S-gsl)

##### Note

If no well-designed, well-documented, and well-supported library exists for an important domain,
maybe you should design and implement it, and then use it.
