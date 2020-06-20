# <a name="S-stdlib"></a>SL: The Standard Library

언어 자체의 기능만을 사용한다면 (어떤 언어를 사용하더라도)  모든 작업이 번거롭지만
적절한 라이브러리를 사용한다면 어떠한 작업이든 상당히 간단해질 수 있다.

표준 라이브러리는 매년 꾸준히 향상되고 있으며
이제는 표준 라이브러리의 설명이 언어 자체의 설명보다 더 많아졌다.
그래서 이 라이브러리 섹션 가이드라인이 다른 모든 섹션의 양과 비슷하거나 결국 더 커질 것이다.

<< ??? We need another level of rule numbering ??? >>

C++ 표준 라이브러리의 구성 요소 요약:

* [SL.con: Containers](#SS-con)
* [SL.str: String](#SS-string)
* [SL.io: Iostream](#SS-io)
* [SL.regex: Regex](#SS-regex)
* [SL.chrono: Time](#SS-chrono)
* [SL.C: C 표준 라이브러리](#SS-clib)

표준 라이브러리의 규칙 요약:

* [SL.1: 가능한 곳 어디에서든 라이브러리를 사용하라](#Rsl-lib)
* [SL.2: 다른 라이브러리보다는 표준 라이브러리를 우선적으로 사용하라](#Rsl-sl)
* [SL.3: 비표준 개체를 네임스페이스 `std`에 추가하지 말라](#sl-std)
* [SL.4: 타입 안전성을 지키면서 표준 라이브러리를 사용하라](#sl-safe)
* ???

### <a name="Rsl-lib"></a>SL.1: 가능한 곳 어디에서든 라이브러리를 사용하라

##### Reason

처음부터 다시 만들지 말고 시간을 절약하라.
다른 사람이 이미 완료한 일을 중복해서 하지 말라.
다른 사람들이 향후 개선을 하게 되면 그 혜택을 누릴 수 있다.
또한 내가 직접 개선함으로써 다른 사람들을 도울 수 있다.

### <a name="Rsl-sl"></a>SL.2: 다른 라이브러리보다는 표준 라이브러리를 우선적으로 사용하라

##### Reason

많은 사람들이 표준 라이브러리를 알고 있으므로
스스로 작성한 코드나 다른 라이브러리 보다 더 안정적이고, 더 잘 관리되고, 더 광범위한 곳에 사용할 수 있다.


### <a name="sl-std"></a>SL.3: 비표준 개체를 네임스페이스 `std`에 추가하지 말라

##### Reason

`std` 네임스페이스에 무언가를 추가하는 작업은 다른 표준을 따르는 코드의 의미를 변경할 수도 있으며
`std`에 무언가를 추가하면 표준의 향후 버전에서 의도하지 않은 충돌을 일으킬 수 있다.

##### Example

    ???

##### Enforcement

가능하지만 std 네임스페이스이 지저분해질 수도 있고 일부 플랫폼에서 문제의 원인이 될 수도 있다.

### <a name="sl-safe"></a>SL.4: 타입 안전성을 지키면서 표준 라이브러리를 사용하라

##### Reason

명백하게도 이 규칙을 지키지 않으면 미정의 동작, 메모리 오류, 그리고 다른 모든 종류의 나쁜 에러들을 일으킬 수 있다.

##### Note

이것은 이를 지지하는 구체적인 규칙에 대한 많은 지원이 필요한 어느정도 철학적인 메타 규칙이다.
우리는 더 구체적인 규칙에 대한 보호의 용도로 이것이 필요하다.

더 구체적인 규칙에 대한 요약:

* [SL.4: 표준 라이브러리는 타입 안전성을 지키며 사용하라](#sl-safe)


## <a name="SS-con"></a>SL.con: Containers

???

컨테이너 규칙에 대한 요약:

* [SL.con.1: C 배열을 사용하기보다 STL의 `array`나 `vector`를 사용하라](#Rsl-arrays)
* [SL.con.2: 다른 컨테이너를 사용할 이유가 있지 않다면 STL `vector`를 기본으로 사용하라](#Rsl-vector)
* [SL.con.3: 경계조건에서 발생하는 에러를 피하라](#Rsl-bounds)
*  ???

### <a name="Rsl-arrays"></a>SL.con.1: C 배열을 사용하기보다 STL의 `array`나 `vector`를 사용하라

##### Reason

C 배열은 덜 안전하고 `array`나 `vector`에 비해 가지는 장점이 없다.
고정된 길이의 배열에는 `std::array`를 사용하라. 이는 함수에 전달될때 포인터 전달에 비해 나쁜것이 없으며 자기 자신의 크기를 알 수도 있다.
또한, 원래의 배열과 같이 스택에 할당되는 `std::array`는 자기 자신의 요소들 또한 스택에 보관한다.
가변길이 배열의 경우, `std::vector`를 사용하라. 이는 자신의 크기를 변경할 수 있으며 메모리 할당을 자체적으로 관리해준다.

##### Example

```c++
    int v[SIZE];                        // BAD

    std::array<int, SIZE> w;             // ok
```

##### Example

```c++
    int* v = new int[initial_size];     // BAD, owning raw pointer
    delete[] v;                         // BAD, manual delete

    std::vector<int> w(initial_size);   // ok
```

##### Note

컨테이너에 대한 비소유 참조의 경우 `gsl::span`을 사용하라.

##### Note

스택에 할당되는 고정된 길이의 배열과 힙 공간을 사용하는 `vector`의 성능 비교는 신뢰하기가 어렵지만
스택을 사용하는 `std::array`와 포인터로 접근하는 `malloc()`된 공간을 사용하는 것과 비교할 수 있다.
대부분의 코드는 스택에 할당되거나 힙 공간에 저장되든지 상관 없고, 단지 `vector`의 편의성과 안전성에 관심이 있다.
그 차이가 상관있는 경우에는 실제 코드를 작성하는 사람이 `array`나 `vector`를 적절히 선택할 수 있다.

##### Enforcement

(STL이 아닌 구형의 코드에서 발생하는 과도하고 시끄러운 경고를 피하기 위해) STL 컨테이너를 선언하는 함수나 클래스 안에 C 배열에 대한 선언이 있으면 지적하라.
이를 수정하기 위해서는 최소한 C 배열을 `std::array`로 바꿔서 사용하라.

### <a name="Rsl-vector"></a>SL.con.2: 다른 컨테이너를 사용할 이유가 있지 않다면 STL `vector`를 기본으로 사용하라

##### Reason

`vector`와 `array`는 다음의 세 가지 장점을 제공하는 유일한 표준 컨테이너들이다.

* 가장 빠른 범용 접근(random access, including being vectorization-friendly)
* 가장 빠른 기본적 접근 패턴(begin-to-end or end-to-begin is prefetcher-friendly)
* 가장 적은 공간 부담(contiguous layout has zero per-element overhead, which is cache-friendly)

일반적인 경우 컨테이너로부터 요소를 추가하고 제거할 필요가 있는 경우 기본적으로 `vector`를 사용하고,
만약 컨테이너의 크기를 변경할 일이 없다면 `array`를 사용하라.

만약 다른 컨테이너가 더 적절해 보인다면, 예를 들어 O(log N) 검색 성능을 위해 `map`을 사용하거나, 요소들 중간에 효율적으로 삽입을 하고 싶은 경우는 `list`를 사용할 수 있다.  그렇지 않다면 몇 KB 의 크기까지는 보통의 경우에 `vector`가 더 나은 성능을 보일 수 있다.

##### Note

`string`은 각각의 개별 문자들을 위한 컨테이너로 사용하지 말아야 한다. `string`은 텍스트로 구성된 문자열이고, 만약 개별 문자의 컨테이너가 필요하면 `vector</*char_type*/>` 또는 `array</*char_type*/>`를 대신 사용하라.

##### Exceptions

만약 다른 컨테이너를 사용할 좋은 이유가 있다면 사용하라. 예를 들면:

* 만약 `vector`가 당신의 요구사항에 적합하고, 컨테이너의 크기가 가변적이지 않다면 그냥 `array`를 사용하라.

* 만약 O(K) 또는 O(log N) 검색시간이 보장되어야 하면서 컨테이너가 몇 KB 보다 더 커질 수 있고, 빈번한 삽입이 일어나 정렬된 `vector`를 유지하기 어렵다면 `unordered_map`이나 `map`을 대신 사용하라.

##### Note

vector 를 요소의 크기만큼 초기화하려면, `()` 초기화를 사용하라.
vector 를 요소의 리스트 내용으로 초기화하려면, `{}` 초기화를 사용하라.

```c++
    vector<int> v1(20);  // v1 은 값이 0인 20개의 요소들을 갖는다. (vector<int>{})
    vector<int> v2 {20}; // v2 는 값이 20인 한개의 요소를 갖는다.
```

[{} 초기화 문법을 선호하라.](./Expr.md#Res-list).

##### Enforcement

* `vector` 생성 이후에 크기가 전혀 변경되지 않는 경우(예를 들면 컨테이너 변수가 `const`이거나 `const`가 아닌 함수가 하나도 호출되지 않는 경우)를 지적할 수 있다. 이를 수정하려면 `array`를 사용하라.

### <a name="Rsl-bounds"></a>SL.con.3: 경계조건에서 발생하는 에러를 피하라

##### Reason

할당된 요소들의 범위를 넘어서 읽거나 쓰는 경우에 나쁜 에러나 잘못된 결과 및 사고 또는 보안 침해를 일으킬 수 있다.

##### Note

요소 범위에 대한 사용이 가능한 표준 라이브러리 함수들은 (대부분) 모두 경계조건이 안전한지 검사에 필요한 연산을 추가하는 부담이 있다.
`vector`와 같은 표준 타입들은 경계조건 검사를 수행하도록
[bounds profile](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#SS-bounds)에 기반하여
([contracts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r1.pdf)를 추가하는 등의 호환 가능한 방식으로)
수정되거나 `at()`을 사용할 수 있다.

이상적으로는 메모리 접근이 경계안에 있음이 정적으로 보장되어야 한다.

예를 들면:

* range-`for`문은 대상 컨테이너가 접근 가능한 범위를 절대 넘어서 반복문을 수행하지 않는다.
* `v.begin(),v.end()`는 경계조건이 안전하다는 것을 쉽게 알 수 있다.

그러한 반복문들은 경계조건 검사가 없고, 안전하지 않은 요소 접근에 필요한 성능과 동일하다.

종종 간단한 검사를 미리 함으로서 인덱스를 접근할 때마다 매번 검사할 필요를 없앨 수 있다.

예를 들면

* `v.begin(),v.begin()+i`를 위해서는 간단히 `i`가 `v.size`를 넘는지 검사해 볼 수 있다.

이러한 반복문은 매번 요소에 접근할 때마다 경계 조건 검사를 하는 것 보다 훨씬 빠를 수 있다.

##### Example, bad

```c++
    void f()
    {
        array<int, 10> a, b;
        memset(a.data(), 0, 10);         // BAD, 배열의 길이를 넘어서는 에러 (length = 10 * sizeof(int))
        memcmp(a.data(), b.data(), 10);  // BAD, 배열의 길이를 넘어서는 에러 (length = 10 * sizeof(int))
    }
```

또한, `std::array<>::fill()`이나 `std::fill()` 또는 비어있는 초기화문이 `memset()`보다는 나은 후보이다.

##### Example, good

```c++
    void f()
    {
        array<int, 10> a, b, c{};       // c is initialized to zero
        a.fill(0);
        fill(b.begin(), b.end(), 0);    // std::fill()
        fill(b, 0);                     // std::fill() + Ranges TS

        if ( a == b ) {
          // ...
        }
    }
```

##### Example

만약 수정되지 않은 표준 라이브러리를 사용한다면, `std::array`와 `std::vector`를 경계 조건에 안전하게 사용할 수 있는 대략의 해결 방법이 있다.
해당 코드는 `std::out_of_range`예외를 발생시킬 수 있는 각 class 의 `.at()` 멤버 함수를 호출 할 수 있다.
아니면 경계조건 위반에 빠르게 실패하거나 사용자가 정의한 동작을 하는 `at()` 이외의 함수를 호출 할 수도 있다.

```c++
    void f(std::vector<int>& v, std::array<int, 12> a, int i)
    {
        v[0] = a[0];        // BAD
        v.at(0) = a[0];     // OK (alternative 1)
        at(v, 0) = a[0];    // OK (alternative 2)

        v.at(0) = a[i];     // BAD
        v.at(0) = a.at(i);  // OK (alternative 1)
        v.at(0) = at(a, i); // OK (alternative 2)
    }
```

##### Enforcement

* 경계조건 검사가 없는 표준 라이브러리 함수에 대한 호출이 있다면 지적하라.
??? insert link to a list of banned functions

이러한 규칙은 [bounds profile](#SS-bounds)의 일부분이다.


## <a name="SS-string"></a>SL.str: String

텍스트 조작은 거대한 주제이고, `std::string`은 이들 전부를 다루지 못한다.
이 섹션에서는 주로 `char*`, `zstring`, `string_view`, 그리고 `gsl::string_span`과 `std::string`의 관계에 대해 명확하게 하고자 한다.
`wchar_t`, 유니코드, 그리고 UTF-8 을 포함하는 ASCII가 아닌 문자셋과 인코딩과 같은 중요한 이슈는 다른곳에서 다루고자 한다.

**See also**: [regular expressions](#SS-regex)

String 요약:

* [SL.str.1: 문자열을 제대로 소유하기 위해서는 `std::string`을 사용하라](#Rstr-string)
* [SL.str.2: 문자들의 나열을 참조하기 위해서는 `std::string_view`나 `gsl::string_span`을 사용하라](#Rstr-view)
* [SL.str.3: 0으로 끝나는 문자들의 나열인 C 스타일의 문자열을 표현하려면 `zstring`나 `czstring`을 사용하라.](#Rstr-zstring)
* [SL.str.4: 하나의 단일 문자에 대한 참조를 의도할 때 `char*`를 사용하라](#Rstr-char*)
* [SL.str.5: 문자열을 표현할 필요없이 바이트 값을 참조하려면 `std::byte`를 사용하라](#Rstr-byte)
* [SL.str.10: 지역 언어에 민감한 문자열에 대한 처리가 필요할 때에는 `std::string`을 사용하라.](#Rstr-locale)
* [SL.str.11: 문자열을 수정할 필요가 있을때는 `std::string_view`보다는 `gsl::string_span`을 사용하라](#Rstr-span)
* [SL.str.12: 표준 라이브러리 `std::string`을 의도할 때는 `s` 접미사를 사용하라](#Rstr-s)

**See also**:

* [F.24 span](#Rf-range)
* [F.25 zstring](#Rf-zstring)


### <a name="Rstr-string"></a>SL.str.1: 문자열을 제대로 소유하기 위해서는 `std::string`을 사용하라

##### Reason

`string`은 올바른 방식으로 메모리 할당 및 소유, 복사, 점진적인 확장을 지원하며, 그리고 다양하고 유용한 기능들을 제공한다.

##### Example

```c++
    vector<string> read_until(const string& terminator)
    {
        vector<string> res;
        for (string s; cin >> s && s != terminator; ) // 한 단어 읽기
            res.push_back(s);
        return res;
    }
```

`string`에서는 (유용한 기능들의 예로) `>>`와 `!=` 연산자들을 제공하고, 여기에는 어떠한 명시적인
메모리 할당, 해제 또는 경계조건 검사 없이 `string`이 내부적으로 이들을 해결한다.

C++17 에서는 함수 호출자에게 더 유연함을 제공하기 위해 `const string*` 대신에 `string_view`를 함수 인자로 사용할 수도 있다.

```c++
    vector<string> read_until(string_view terminator)   // C++17
    {
        vector<string> res;
        for (string s; cin >> s && s != terminator; ) // 한 단어 읽기
            res.push_back(s);
        return res;
    }
```

`gsl::string_span`은 `std::string_view`의 대부분의 장점을 대체할 수 있는 현재 가능한 옵션일 수 있다. 간단한 예를 들면:

```c++
    vector<string> read_until(string_span terminator)
    {
        vector<string> res;
        for (string s; cin >> s && s != terminator; ) // 한 단어 읽기
            res.push_back(s);
        return res;
    }
```

##### Example, bad

trivial 하지 않은 메모리 관리가 필요한 동작에 대해서는 C 스타일의 문자열을 사용하지 말라.

```c++
    char* cat(const char* s1, const char* s2)   // beware!
        // return s1 + '.' + s2
    {
        int l1 = strlen(s1);
        int l2 = strlen(s2);
        char* p = (char*) malloc(l1 + l2 + 2);
        strcpy(p, s1, l1);
        p[l1] = '.';
        strcpy(p + l1 + 1, s2, l2);
        p[l1 + l2 + 1] = 0;
        return p;
    }
```

이 코드가 올바르게 문제를 해결했을까?
함수 호출자가 반환된 포인터를 `free()` 해야 한다고 기억해야 할까?
이 코드가 보안에 관한 리뷰를 통과할 수 있을까?

##### Note

직접적인 성능 측정 없이 `string`이 저수준의 기법보다 느리다고 단정하지도 말고, 모든 코드가 성능에 민감하다고 생각하지 말라.
[너무 이른 최적화를 하지 말라](#Rper-Knuth)

##### Enforcement

???

### <a name="Rstr-view"></a>SL.str.2: 문자들의 나열을 참조하기 위해서는 `std::string_view`나 `gsl::string_span`을 사용하라

##### Reason

`std::string_view`나 `gsl::string_span`은 문자들의 나열에 대해 간단하고 (잠재적으로) 안전한 접근을 제공하며 이는
이 문자들이 어떻게 메모리 공간에 할당되고 저장되었는지와 무관하다.

##### Example

```c++
    vector<string> read_until(string_span terminator);

    void user(zstring p, const string& s, string_span ss)
    {
        auto v1 = read_until(p);
        auto v2 = read_until(s);
        auto v3 = read_until(ss);
        // ...
    }
```

##### Note

C++17 에서 지원하는 `std::string_view`는 읽기 전용이다.

##### Enforcement

???

### <a name="Rstr-zstring"></a>SL.str.3: 0으로 끝나는 문자들의 나열인 C 스타일의 문자열을 표현하려면 `zstring`나 `czstring`을 사용하라.

##### Reason

이들을 사용하면 가독성과 의도에 대한 표현을 명확히 할 수 있다.
단순 `char*`는 하나의 문자에 대한 포인터이거나 문자들의 배열에 대한 포인터, null 로 끝나는 C 스타일의 문자열 또는 작은 정수에 대한 포인터로 다양하게 표현될 수 있다.
이러한 다양한 방식의 표현들을 잘 구분함으로서 불필요한 오해나 버그를 방지 할 수 있다.

##### Example

```c++
    void f1(const char* s); // s 는 아마도 문자열일 것이다.
```

우리가 알 수 있는 것은 이것이 nullptr 이거나 최소한 하나의 문자에 대한 포인터라는 것이다.

```c++
    void f1(zstring s);     // s 는 C 스타일의 문자열이거나 nullptr 이다.
    void f1(czstring s);    // s 는 C 스타일의 상수 문자열이거나 nullptr 이다.
    void f1(std::byte* s);  // s 는 하나의 바이트에 대한 포인터이다. (C++17)
```

##### Note

특별한 이유가 있지 않다면 C 스타일의 문자열을 `string`으로 단순하게 변환하지 말라.

##### Note

다른 일반적인 포인터와 같이 `zstring`은 소유권을 표현하면 안된다.

##### Note

수많은 C++ 코드가 이미 `char*`와 `const char*`를 사용 의도에 대한 설명 없이 사용하고 있다.
그들은 광범위하고 다양한 방식으로 사용되는데 이는 소유권에 대한 표현과 `void*`를 사용하는 대신 일반적인 메모리에 대한 포인터로 사용된다.
이들에 대한 사용법을 구분하는 것이 어렵기 때문에 이러한 가이드라인을 따르는 것이 어렵다.
이것이 C 와 C++ 프로그램에서 발생하는 가장 주요한 버그의 원인중에 하나이므로 가능한한 이 가이드라인을 따르는 것을 권장한다.

##### Enforcement

* `char*`에 대한 `[]` 사용이 있으면 지적하라.
* `char*`에 대해 `delete`를 사용하면 지적하라.
* `char*`에 대해 `free()`를 사용하면 지적하라.

### <a name="Rstr-char*"></a>SL.str.4: 하나의 단일 문자에 대한 참조를 의도할 때 `char*`를 사용하라

##### Reason

현재 코드에서 다양한 방법의 `char*`의 사용은 주요한 에러의 원인이 된다.

##### Example, bad

```c++
    char arr[] = {'a', 'b', 'c'};

    void print(const char* p)
    {
        cout << p << '\n';
    }

    void use()
    {
        print(arr);   // 런타임 에러; 잠재적으로 매우 안좋을 수 있다.
    }
```

`arr` 배열은 0 으로 끝나지 않으므로 C 스타일의 문자열이 아니다.

##### Alternative

[`zstring`](#Rstr-zstring), [`string`](#Rstr-string), [`string_span`](#Rstr-view)을 함께 보라.

##### Enforcement

* `char*` 타입에 대해서 `[]`의 사용이 있으면 지적하라.

### <a name="Rstr-byte"></a>SL.str.5: 문자열을 표현할 필요없이 바이트 값을 참조하려면 `std::byte`를 사용하라

##### Reason

단일 문자일 필요가 없는 것에 대한 포인터로 `char*`를 사용하는 것은 혼란을 유발하거나 일부 최적화를 불가능하게 만든다.

##### Example

    ???

##### Note

C++17

##### Enforcement

???


### <a name="Rstr-locale"></a>SL.str.10: 지역 언어에 민감한 문자열에 대한 처리가 필요할 때에는 `std::string`을 사용하라.

##### Reason

`std::string`은 표준 라이브러리인 [`locale` facilities](#Rstr-locale)을 지원한다.

##### Example

    ???

##### Note

???

##### Enforcement

???

### <a name="Rstr-span"></a>SL.str.11: 문자열을 수정할 필요가 있을때는 `std::string_view`보다는 `gsl::string_span`을 사용하라

##### Reason

`std::string_view`는 읽기 전용이다.

##### Example

???

##### Note

???

##### Enforcement

컴파일러는 `string_view`에 대한 쓰기 시도가 있으면 알릴 것이다.

### <a name="Rstr-s"></a>SL.str.12: 표준 라이브러리 `std::string`을 의도할 때는 `s` 접미사를 사용하라

##### Reason

원래 의도에 대한 직접적인 표현이 실수를 줄일 수 있다.

##### Example

```c++
    auto pp1 = make_pair("Tokyo", 9.00);         // {C-style string,double} intended?
    pair<string, double> pp2 = {"Tokyo", 9.00};  // a bit verbose
    auto pp3 = make_pair("Tokyo"s, 9.00);        // {std::string,double}    // C++14
    pair pp4 = {"Tokyo"s, 9.00};                 // {std::string,double}    // C++17
```



##### Enforcement

???


## <a name="SS-io"></a>SL.io: Iostream

`iostream`은 스트리밍 입출력을 위한 타입에 안전하고, 확장 가능하며, 포맷화되거나 되지 않은 입출력 라이브러리이다.
이것은 다양한(사용자 확장 가능한) 버퍼링 전략과 다양한 지역 언어를 지원한다.
이것은 일반적인 입출력을 할 수 있으며, (string stream 과 같이) 메모리에 읽기와 쓰기를 하거나,
(아직 표준이 되지 않은 asio와 같이) 네트워크로 스트리밍을 하는 것과 같은 사용자 정의 확장 모듈에 사용할 수 있다.

Iostream 규칙 요약:

* [SL.io.1: 단일 문자 단위의 입력은 꼭 필요할때에만 사용하라](#Rio-low)
* [SL.io.2: 항상 읽을때에는 형식화되지 않은 입력을 고려하라](#Rio-validate)
* [SL.io.3: 입출력을 위해서는 `iostream`을 선호하라](#Rio-streams)
* [SL.io.10: `printf`계열의 함수를 사용하지 않는다면 `ios_base::sync_with_stdio(false)`를 호출하라](#Rio-sync)
* [SL.io.50: `endl`의 사용을 피하라](#Rio-endl)
* [???](#???)

### <a name="Rio-low"></a>SL.io.1: 단일 문자 단위의 입력은 꼭 필요할때에만 사용하라

##### Reason

의도적으로 각각의 개별 문자를 다루려고 하지 않는 경우라면 단일 문자 단위 입력에 대한 사용은
잠재적으로 에러가 발생하기 쉽고, 문자열을 비효율적인 토큰의 조합으로 만들뿐이다.

##### Example

```c++
    char c;
    char buf[128];
    int i = 0;
    while (cin.get(c) && !isspace(c) && i < 128)
        buf[i++] = c;
    if (i == 128) {
        // ... handle too long string ....
    }
```

위의 코드보다는 아래의 코드가 훨씬 간단하고 아마도 더 빠를 것이다.

```c++
    string s;
    s.reserve(128);
    cin >> s;
```

여기서 `reserve(128)`는 꼭 필요한 것이 아닐 수도 있다.

##### Enforcement

???


### <a name="Rio-validate"></a>SL.io.2: 항상 읽을때에는 형식화되지 않은 입력을 고려하라

##### Reason

일반적으로 에러는 가능한한 일찍 처리되는 것이 최상이다.
만약 입력이 유효하지 않다면 (현실적이지 않게도) 모든 함수는 잘못된 데이터에 대응할 수 있도록 작성되어야 한다.

##### Example

    ???

##### Enforcement

???

### <a name="Rio-streams"></a>SL.io.3: 입출력을 위해서는 `iostream`을 선호하라

##### Reason

`iostream`은 안전하고 유연하며 확장 가능하다.

##### Example

```c++
    // write a complex number:
    complex<double> z{ 3, 4 };
    cout << z << '\n';
```

`complex`는 사용자 정의 타입이지만 `iostream` 라이브러리에 대한 수정 없이도 이에 대한 입출력이 구현된다.

##### Example

```c++
    // read a file of complex numbers:
    for (complex<double> z; cin >> z; )
        v.push_back(z);
```

##### Exception

??? performance ???

##### `iostream`과 `printf()` 계열에 대한 비교 논의

일반적으로 (그리고 정확하게도) `printf()` 계열은 `iostream`에 비해 유연한 포맷팅과 성능이라는 두가지 장점이 있다.
이들은 `iostream`의 다음의 장점들과 함께 비교해 보아야 한다.
* 사용자 정의 타입에 대한 확장성
* 보안 문제 발생시 복구 가능한 회복력
* 암묵적인 메모리 관리
* `locale` 지역 언어 처리

만약 I/O 성능이 필요하다면 거의 항상 `printf()` 보다는 더 잘할수 있다.

`s`를 사용하는 `gets()` `scanf()`와 `%s`를 사용하는 `printf()`는 (버퍼 오버플로우나 일반적인 에러를 발생시키기 취약한) 보안 위험이다.
C11 에서는 더 안전한 대안으로 `gets_s()`, `scanf_s()`, 그리고 `printf_s()`로 이들을 대체했지만 아직도 타입에 안전하지 못하다.

##### Enforcement

`<cstdio>`과 `<stdio.h>`에 대한 사용을 선택적으로 지적하라.

### <a name="Rio-sync"></a>SL.io.10: `printf`계열의 함수를 사용하지 않는다면 `ios_base::sync_with_stdio(false)`를 호출하라

##### Reason

`iostream`과 `printf` 스타일의 I/O 동기화는 부담이 많이 든다.
`cin`과 `cout`은 기본적으로 `printf`와 동기화 된다.

##### Example

```c++
    int main()
    {
        ios_base::sync_with_stdio(false);
        // ... use iostreams ...
    }
```

##### Enforcement

???

### <a name="Rio-endl"></a>SL.io.50: `endl`의 사용을 피하라

##### Reason

`endl` 조정자는 대체로 `'\n'`이나 `"\n"`와 동일하다.
이것은 일반적으로 많이 사용되는데 대부분의 경우에 중복되는 `flush()`명령을 수행하느라 출력을 느리게 만든다.
이러한 성능 저하는 `printf` 스타일의 출력 성능과 뚜렷하게 비교된다.

##### Example

```c++
    cout << "Hello, World!" << endl;    // 두번의 출력 명령과 한번의 flush
    cout << "Hello, World!\n";          // flush 없는 단 한번의 출력 명령
```

##### Note

`cin`/`cout`을 사용한 상호 작용에는 버퍼를 비우는 flush 를 할 이유가 없고, 이는 어차피 자동으로 이루어진다.
파일에 쓰기를 하는 경우에는 `flush`를 할 필요가 거의 없다.

##### Note

(때때로 중요하지만) 성능 이슈를 논외로 하면, `'\n'`나 `endl`에 대한 선택은 거의 완전하게 미학적일 뿐이다.


## <a name="SS-regex"></a>SL.regex: Regex

`<regex>`는 표준 C++ 정규 표현식 라이브러리이다.
이것은 다양한 방식의 정규 표현식 패턴 규약을 지원한다.

## <a name="SS-chrono"></a>SL.chrono: Time

(`std::chrono` 네임스페이스에 정의되어 있는) `<chrono>`는 `time_point`와 `duration`에 대한 개념을
시간에 대한 다양한 단위의 출력 함수와 함께 제공한다.
또한, 이것은 `time_points` 등록을 위한 다양한 clock 들도 제공한다.

## <a name="SS-clib"></a>SL.C: C 표준 라이브러리

???

C 표준 라이브러리 규칙 요약:

* [S.C.1: setjmp/longjmp를 사용하지 말라](#Rclib-jmp)
* [???](#???)
* [???](#???)

### <a name="Rclib-jmp"></a>SL.C.1: setjmp/longjmp를 사용하지 말라

##### Reason

`longjmp`는 소멸자를 무시하게 하는데 이는 RAII 에 의존적인 모든 자원 관리 전략을 무효화시킨다.

##### Enforcement

`longjmp`와 `setjmp`이 있다면 전부 지적하라.
