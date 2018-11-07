
# <a name="S-errors"></a>E: 오류 처리

오류 처리는 다음을 포함한다:

* 오류를 발견
* 오류에 대한 정보를 처리하는 부분으로 전달
* 프로그램을 유효한 상태로 유지
* 리소스 누수를 방지

모든 오류를 복구하는 것은 불가능하다. 만약 어떤 오류의 복구가 불가능 하다면, 잘 정의된 방법으로 빠르게 "빠져나가는" 것이 중요하다. 오류를 처리하는 전략은 단순해야 한다. 그렇지 않으면 더 악화된 오류들의 원인이 된다. 검증되지 않고 드물게 수행되는 오류 처리 코드는 그 자체로 많은 버그들의 원인이 된다.

이 장의 규칙들은 몇몇 종류의 오류들을 피하는 것에 도움을 주기 위해 설계되었다:

* 타입 위반들 (`공용체`와 형변환의 오용)
* 리소스 누수 (메모리 누수를 포함)
* 한계 오류
* 수명 오류 (`delete`된 객체에 접근)
* 복잡성 오류 (지나치게 복잡한 표현으로 아이디어들을 표현함으로 인해 발생하는 논리적 에러)
* 인터페이스 오류 (예상치 못한 값이 인터페이스를 통해 전달됨)

오류 처리 규칙 요약:

* [E.1: 설계 과정 초기에 오류 처리에 대한 전략을 수립하라](#Re-design)
* [E.2: 함수가 맡은 작업을 처리할 수 없다는 것을 알리기 위해 예외를 발생시켜라](#Re-throw)
* [E.3: 예외는 오류 처리에만 사용하라](#Re-errors)
* [E.4: 불변조건을 중심으로 오류 처리 전략을 설계하라](#Re-design-invariants)
* [E.5: 생성자에서 불변조건을 설정하도록 하고, 그렇게 할 수 없으면 예외를 발생시켜라](#Re-invariant)
* [E.6: RAII를 사용하여 누수를 방지하라](#Re-raii)
* [E.7: 선행조건을 기술하라](#Re-precondition)
* [E.8: 후행조건을 기술하라](#Re-postcondition)
* [E.12: `throw`를 허용하지 않거나 불가능한 함수에는 `noexcept`를 사용하라](#Re-noexcept)
* [E.13: 객체를 직접적으로 소유하는 중에는 예외를 발생시켜선 안된다](#Re-never-throw)
* [E.14: 목적에 맞게 설계된 사용자 정의 타입을 예외로 사용하라 (내장 타입은 안된다)](#Re-exception-types)
* [E.15: 계층 구조가 있는 예외는 참조를 사용해서 잡아라](#Re-exception-ref)
* [E.16: 소멸자, 자원해제, `swap`은 절대 실패해선 안된다](#Re-never-fail)
* [E.17: 각각의 함수들에서 모든 예외를 처리하려고 하지 마라](#Re-not-always)
* [E.18: 명시적인 `try`/`catch`문의 사용을 최소화하라](#Re-catch)
* [E.19: 적당한 리소스 핸들을 사용할 수 없다면 해제방법을 나타낼 `final_action` 객체를 사용하라](#Re-finally)
* [E.25: 예외를 던질 수 없다면, 자원관리를 위해 RAII를 시뮬레이션 하라](#Re-no-throw-raii)
* [E.26: 예외를 던질 수 없다면, 빠른 실패 전략을 고려하라](#Re-no-throw-crash)
* [E.27: 예외를 던질 수 없다면, 체계적으로 에러 코드를 사용하라](#Re-no-throw-codes)
* [E.28: 전역 상태에 기반한 에러 처리를 지양하라(`errno`처럼)](#Re-no-throw)
* [E.30: 예외 명세를 사용하지 마라](#Re-specifications)
* [E.31: 적합한 순서로 `catch`를 배치하라](#Re_catch)

### <a name="Re-design"></a>E.1: 설계 과정 초기에 오류 처리에 대한 전략을 수립하라

##### Reason

오류들과 리소스 누수들을 처리하기 위한 일관적이고 완전한 전략은 시스템에 새로 추가하기가 아주 어렵다.

### <a name="Re-throw"></a>E.2: 함수가 맡은 작업을 처리할 수 없다는 것을 알리기 위해 예외를 발생시켜라

##### Reason

오류 처리가 체계적이고, 견고하며, 반복적이지 않게 된다.

##### Example

```c++
    struct Foo {
        vector<Thing> v;
        File_handle f;
        string s;
    };

    void use()
    {
        Foo bar {{Thing{1}, Thing{2}, Thing{monkey}}, {"my_file", "r"}, "Here we go!"};
        // ...
    }
```

여기서, `vector` 와 `string`의 생성자는 엘리먼트들이 필요로 하는 충분한 메모리를 할당받지 못할 수 있으며, `vector`의 생성자는 초기화 목록의 `Thing`을 복사할 수 없을 수 있고, `File_handle`은 필요한 파일을 열지 못할 수 있다.
각각의 경우, `use()`의 호출자가 처리할 수 있도록 예외를 발생시킨다.
만약에 `use()`가 `bar` 생성 실패를 처리할 수 있으면 `try`/`catch`를 사용하여 제어할 수 있다.

어느 경우든, `Foo`의 생성자는 제어권을 넘기기전에 `Foo`를 만들려고 시도한 모든 생성된 멤버들을 올바르게 소멸시킨다.
이 때는 오류 코드를 포함한 반환 값이 없다는 점에 주의하라.

`File_handle`의 생성자는 다음과 같은 코드일 수 있다:

```c++
    File_handle::File_handle(const string& name, const string& mode)
        :f{fopen(name.c_str(), mode.c_str())}
    {
        if (!f)
            throw runtime_error{"File_handle: could not open " + name + " as " + mode};
    }
```

##### Note

예외의 의도는 예외적인 사건이나 실패를 알리는데 있다.
하지만 "무엇이 예외적인가?"라는 점에서 순환적인 의미를 가진다.

예를 들면:

* 충족되지 않은 선행조건
* 객체를 생성할 수 없는 생성자 (클래스의 [불변조건](#Rc-struct) 설정 실패)
* 범위를 벗어나는(out-of-range) 에러. (예컨대 `v[v.size()] = 7`)
* 자원 획득의 실패(예: 네트워크 다운)

이와 반대로, 일반적인 루프의 종료는 예외적이지 않다. 무한 루프가 아니라면, 루프가 종료하는 것은 일반적으로 기대하는 것이다.

##### Note

함수가 값을 반환하는 방법으로 `throw`를 사용하지 마라.

##### Exception

몇몇 제약이 심한 시스템들은, 실행되기 전에 (일반적으로 짧은) 고정된 최대 처리시간을 보장해야 한다. 이런 경우에는 `throw`로부터 복원하기 위한 정확한 시간을 예측하는 도구가 있어야만 예외를 사용할 수 있다.

**See also**: [RAII](#Re-raii)

**See also**: [discussion](#Sd-noexcept)

##### Note

예외를 사용한 에러 처리를 좋아하지 않거나 결정을 내릴 수 없다면, [대안들](#Re-no-throw-raii)을 검토해보라.

에러 처리 방법들은 제각기 복잡한 부분과 문제가 있기 마련이다. 
가능하다면, 처리 방법의 효율성에 대해서 측정을 해보라.

### <a name="Re-errors"></a>E.3: 예외는 오류 처리에만 사용하라

##### Reason

"정상적인 코드"와 오류 처리를 분리하도록 해준다.
C++에서는 예외가 드물게 발생하다는 전제 하에 최적화를 적용한다.

##### Example, don't

```c++
    // exception이 에러 처리를 위해 사용되지 않고 있다.
    int find_index(vector<string>& vec, const string& x)
    {
        try {
            for (gsl::index i = 0; i < vec.size(); ++i)
                if (vec[i] == x) throw i;  // found x
        } catch (int i) {
            return i;
        }
        return -1;   // not found
    }
```

이런 코드는 복잡하고 일반적인 for문 보다 훨씬 느릴 것이다. 
`vector`에서 값을 찾는데 예외적인 상황은 없다.

##### Enforcement

경험적으로 접근해야한다.
`catch`구문이 예외 값을 "잡지 못하는지" 확인해보라.

### <a name="Re-design-invariants"></a>E.4: 불변조건을 중심으로 오류 처리 전략을 설계하라

##### Reason

개체를 사용하려면 그 개체는 (불변조건을 사용해서 정의된) 올바른 상태에 있어야 한다. 또 오류로부터 개체들을 복원하려면, 파괴되지 않은 개체들은 올바른 상태에 있어야 한다.

##### Note

[불변조건(invariant)](#Rc-struct)은 생성자가 수립해야 하는 개체 멤버들에 대한 논리적 조건을 의미한다.
이런 조건들은 `public` 멤버 함수들에서 가정하는 것들이다.

##### Enforcement

???

### <a name="Re-invariant"></a>E.5: 생성자에서 불변조건을 설정하도록 하고, 그렇게 할 수 없으면 예외를 발생시켜라

##### Reason

조건을 만족하지 않는 개체를 구성하면 문제를 발생시킬 수 있다.
멤버 함수 중 일부만 호출할 수 있다.

##### Example

```c++
    class Vector {  // very simplified vector of doubles
        // if elem != nullptr then elem points to sz doubles
    public:
        Vector() : elem{nullptr}, sz{0}{}
        Vector(int s) : elem{new double[s]}, sz{s} { /* initialize elements */ }
        ~Vector() { delete [] elem; }
        double& operator[](int s) { return elem[s]; }
        // ...
    private:
        owner<double*> elem;
        int sz;
    };
```

주석으로 표기된 이 클래스의 불변조건은 생성자에서 설정된다.
`new`는 요구받은 메모리를 할당하지 못할 경우 예외를 던진다.
여기서 `operator []`는 불변조건에 의존하게 된다.

**See also**: [생성자가 올바른 개체를 생성하지 못한다면, 예외를 던져라](#Rc-throw)

##### Enforcement

생성자를 사용하지 않는 클래스는 접근 지정자 `private`으로 지정하도록 표시한다.

### <a name="Re-raii"></a>E.6: RAII를 사용해 누수를 방지하라

##### Reason

일반적으로 누수는 허용되어선 안된다.RAII(자원 획득은 초기화)는 누수를 방지하는 간단하고, 가장 체계적인 방법이다.

##### Example

```c++
    void f1(int i)   // Bad: possibly leak
    {
        int* p = new int[12];
        // ...
        if (i < 17) throw Bad{"in f()", i};
        // ...
    }
```

예외를 발생시키기 전에 메모리를 해제할 수도 있다:

```c++
    void f2(int i)   // Clumsy and error-prone: explicit release
    {
        int* p = new int[12];
        // ...
        if (i < 17) {
            delete[] p;
            throw Bad{"in f()", i};
        }
        // ...
    }
```

이런 코드는 지저분하다. 다수의 `throw`가 가능한 경우, 명시적인 해제는 반복적이고, 오류에 취약하다.

```c++
    void f3(int i)   // OK: resource management done by a handle (but see below)
    {
        auto p = make_unique<int[]>(12);
        // ...
        if (i < 17) throw Bad{"in f()", i};
        // ...
    }
```

RAII는 호출받은 함수에서 묵시적으로 `throw`하는 상황에서도 동작한다는 점에 주의하라:

```c++
    void f4(int i)   // OK: resource management done by a handle (but see below)
    {
        auto p = make_unique<int[]>(12);
        // ...
        helper(i);   // may throw
        // ...
    }
```

포인터가 반드시 필요하지 않다면, 지역 변수로 처리하라:

```c++
    void f5(int i)   // OK: resource management done by local object
    {
        vector<int> v(12);
        // ...
        helper(i);   // may throw
        // ...
    }
```

더 간단하고 안전하다. 경우에 따라선 더 효율적이기도 하다.

##### Note

자원을 해제해주는 리소스 핸들 개체가 없다면 혹은 적합한 RAII 개체/핸들을 사용하는 것이 불가하다면, 자원해제 동작들은 [`final_action` 개체](#Re-finally)를 사용해서 수행할 수도 있다.

##### Note

그러나 예외를 사용할 수 없는 프로그램을 작성한다면 어떻게 해야 할까?

한번 그 생각에 도전해 보자; 예외를 반대하는 많은 의견이 있을 것이다.
그 중에 몇몇 타당한 의견들에 대해 알아 보자:

* 예외를 적용하려면 메모리를 다 써버리는 아주 작은(2K보다 작은) 시스템 상에 있다.
* 제한 시간 안에 반드시 응답을 보장해야 하는 경우, 요구된 시간 안에 예외를 처리할 수 있는 도구를 가지고 있지 않다.
* 난해한 방식으로 포인터를 사용하는 엄청난 양의 레거시 코드를 가진 시스템 상에 있다. (특히 소유권 관련된 정책이 보이지 않아서) 예외가 메모리 누수를 야기할 수 있다.
* C++ 예외 메커니즘을 구현한 것이 적합하지 않다.(속도, 메모리 소비, DLL에서의 불분명한 처리실패 등) (예외 구현을 제공한 쪽에 항의하라; 불평이 없다면 개선도 없다.)
* 매니저가 가진 오래된 지식에 도전하면 해고당한다

여기서 첫번째 이유만이 근본적인 경우라고 할 수 있다. 가능하다면, RAII를 구현하기 위해 예외를 사용하거나, RAII 개체들이 문제를 일으키지 않도록 설계하라.

예외를 사용할 수 없다면, RAII를 흉내라도 내라. 즉, 체계적으로 개체가 생성된 후 유효한지 검사하고 소멸자에서 자원을 모두 해제하라. `valid()` 연산을 추가하는 것이 좋을 수도 있다.

```c++
    void f()
    {
        vector<string> vs(100);   // not std::vector: valid() added
        if (!vs.valid()) {
            // handle error or exit
        }

        ifstream fs("foo");   // not std::ifstream: valid() added
        if (!fs.valid()) {
            // handle error or exit
        }

        // ...
    } // destructors clean up as usual
```

이런 방법은 코드의 길이가 늘어나지만, "exception"이 전파되는 것을 `valid()` 검사를 통해서 방지한다.
`valid()` 검사가 실수로 잊혀질 수 있으니 가능하다면 예외를 선호하라.

**See also**: [`noexcept`의 사용](#Se-noexcept)

##### Enforcement

???

### <a name="Re-precondition"></a>E.7: 선행조건을 기술하라

##### Reason

인터페이스 오류를 예방한다.

**See also**: [precondition rule](#Ri-pre)

### <a name="Re-postcondition"></a>E.8: 후행조건을 기술하라

##### Reason

인터페이스 오류를 예방한다.

**See also**: [postcondition rule](#Ri-post)

### <a name="Re-noexcept"></a>E.12: `throw`를 허용하지 않거나 불가능한 함수에는 `noexcept`를 사용하라

##### Reason

오류 처리를 체계적이고, 견고하며, 효율적으로 만든다.

##### Example

```c++
    double compute(double d) noexcept
    {
        return log(sqrt(d <= 0 ? 1 : d));
    }
```

이 코드에서는 `compute` 함수가 예외를 던지지 않도록 작성되었단 사실을 알수 있다. 
`noexcept`로 선언함으로써, 우리는 컴파일러와 다른 이들에게 `compute`함수를 이해하고 변경하는데 도움을 줄 수 있다.

##### Note

C 표준 라이브러리에서 "승계된" 많은 C++ 표준 라이브러리 함수들은 `noexcept`로 정의되어있다.

##### Example

```c++
    vector<double> munge(const vector<double>& v) noexcept
    {
        vector<double> v2(v.size());
        // ... do something ...
    }
```

이 코드에서 `noexcept`는 지역변수 `vector`의 생성이 실패하는 상황을 처리하지 않는다고 표현하고 있다. 이 말인 즉, 메모리 고갈을 (하드웨어 실패와 같은)심각한 설계 오류로 간주한다는 것이다. 따라서 예외가 발생한다면 프로그램에서 크래시를 일으킬 것이다.

##### Note

전통적인 [예외 명세법](#Re-specifications)은 사용하지 마라

##### See also

[discussion](#Sd-noexcept).

### <a name="Re-never-throw"></a>E.13: 개체를 직접적으로 소유하는 중에는 예외를 발생시켜선 안된다

##### Reason

자원 누수를 발생시킬 수 있다.

##### Example

```c++
    void leak(int x)   // don't: may leak
    {
        auto p = new int{7};
        if (x < 0) throw Get_me_out_of_here{};  // may leak *p
        // ...
        delete p;   // we may never get here
    }
```

이런 문제를 해결하기 위해서 일관적으로 자원 핸들(resource handle)을 사용할 수도 있다:

```c++
    void no_leak(int x)
    {
        auto p = make_unique<int>(7);
        if (x < 0) throw Get_me_out_of_here{};  // will delete *p if necessary
        // ...
        // no need for delete p
    }
```

다른 (더 나은) 해결책은 포인터를 대신해 지역 변수를 사용하는 것이다:

```c++
    void no_leak_simplified(int x)
    {
        vector<int> v(7);
        // ...
    }
```

##### Note

정리 작업이 필요하지만 소멸자가 없는 지역 변수들이 있다면, 그 작업들은 `throw`에 앞서 수행되어야 한다. 
[`finally()`](#Re-finally)는 그와 같은 시스템에서 누락될 수 있는 작업들을 좀 더 관리하기 쉽게 해준다.

### <a name="Re-exception-types"></a>E.14:목적에 맞게 설계된 사용자 정의 타입을 예외로 사용하라 (내장 타입은 안된다)

##### Reason

프로그래머가 정의한 타입들은 다른 사용자들의 예외와 충돌할 가능성이 적다.

##### Example

```c++
    void my_code()
    {
        // ...
        throw Moonphase_error{};
        // ...
    }

    void your_code()
    {
        try {
            // ...
            my_code();
            // ...
        }
        catch(const Bufferpool_exhausted&) {
            // ...
        }
    }
```

##### Example, don't

```c++
    void my_code()     // Don't
    {
        // ...
        throw 7;       // 7 means "moon in the 4th quarter"
        // ...
    }

    void your_code()   // Don't
    {
        try {
            // ...
            my_code();
            // ...
        }
        catch(int i) {  // i == 7 means "input buffer too small"
            // ...
        }
    }
```

##### Note

표준 라이브러리의 예외 클래스들은 `std::exception`을 상속하고 있다. 이들에 대한 오류 처리는 `std::exception` 또는 "일반적인" 처리를 사용하여야 한다. 내장 타입들을 예외 개체로 사용하는 것은 다른 사용자들의 코드와 충돌을 발생시킬 수 있다.

##### Example, don't

```c++
    void my_code()   // Don't
    {
        // ...
        throw runtime_error{"moon in the 4th quarter"};
        // ...
    }

    void your_code()   // Don't
    {
        try {
            // ...
            my_code();
            // ...
        }
        catch(const runtime_error&) {   // runtime_error means "input buffer too small"
            // ...
        }
    }
```

**See also**: [Discussion](#Sd-???)

##### Enforcement

내장 타입을 사용하는 `throw`와 `catch`를 잡아내라. 표준 라이브러리에서 사용하는 `std::exception`기반의 계층구조에 포함된 예외 클래스를 사용하는 것이 더 낫다.

### <a name="Re-exception-ref"></a>E.15: 계층 구조가 있는 예외는 참조를 사용해서 처리하라

##### Reason

예외 개체의 복사 손실을 방지한다.

##### Example

```c++
    void f()
    {
        try {
            // ...
        }
        catch (exception e) {   // don't: may slice
            // ...
        }
    }
```

대신 참조를 사용하라:

```c++
    catch (exception& e) { /* ... */ }
```

`const` 참조를 사용하면 더 좋다:

```c++
    catch (const exception& e) { /* ... */ }
```

Most handlers do not modify their exception and in general we [recommend use of `const`](#Res-const).

##### Note

To rethrow a caught exception use `throw;` not `throw e;`. Using `throw e;` would throw a new copy of `e` (sliced to the static type `std::exception`) instead of rethrowing the original exception of type `std::runtime_error`. (But keep [Don't try to catch every exception in every function](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#Re-not-always) and [Minimize the use of explicit `try`/`catch`](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#Re-catch) in mind.)

##### Enforcement

값으로 예외를 처리하는 경우 지적하라. 완벽히 처리하기 위해선 전체 프로그램에 대한 분석이 필요할 수 있다.

### <a name="Re-never-fail"></a>E.16: 소멸자, 자원해제, `swap`은 절대 실패해선 안된다

##### Reason
소멸자, 메모리 해제, `swap`이 실패한다면 신뢰할 수 있는 프로그램을 작성할 수 없다; 예외가 발생하거나 해야 할 작업을 수행하지 않고 종료하는 것을 말한다.

##### Example, don't

```c++
    class Connection {
        // ...
    public:
        ~Connection()   // Don't: very bad destructor
        {
            if (cannot_disconnect()) throw I_give_up{information};
            // ...
        }
    };
```

##### Note

예시에 나온 "close를 거부하는" 네트워크 연결 클래스처럼, 많은 이들이 이 규칙을 위반하면서 신뢰할 수 있는 코드를 작성하려고 시도해왔다. 
우리가 아는 선에서 누구도 이를 일반화하지 못했다.

때때로, 아주 예외적인 경우, 나중에 정리작업을 하기 위핸 상태를 설정하는 것이 가능할 것이다.
예컨대, close하길 원하지 않는 소켓은 "문제있는 소켓"목록에 추가하고 정기적으로 sweep할수도 있을 것이다.

여태껏 이 규칙을 위반하는 사례들은 오류에 취약하거나, 특별하거나, 때로는 버그투성이였다.

##### Note

표준 라이브러리는 소멸자, (`operator delete`같은) 자원 해제 함수, `swap`함수가 예외를 던지지 않는다고 가정한다. 예외를 던진다면, 표준 라이브러리들의 불변조건이 충족되지 않을 것이다.

##### Note

`operator delete`와 같은 해제 함수는 반드시 `noexcept`여야 한다.
`swap`함수는 반드시 `noexcept`여야 한다.
소멸자들은 표기하지 않아도 기본적으로 `noexcept`로 간주된다.
[move 연산도 `noexcept`로 만들어라](#Rc-move-noexcept).

##### Enforcement

예외를 던지는 소멸자, 자원해제 함수, `swap`을 잡아내라. 비슷한 역할을 하는 함수들이 `noexcept`가 아닌 경우에도 잡아내야 한다.

**See also**: [discussion](#Sd-never-fail)

### <a name="Re-not-always"></a>E.17: 각각의 함수들에서 모든 예외를 처리하려고 하지 마라

##### Reason

함수 내에서 예외를 catch하는 것은 유의미한 복구 작업을 하기 어려우며, 복잡성을 높이고 낭비가 될 수 있다.
예외를 던질 때는 그 예외를 처리할 수 있는 함수로 전파되도록 남겨두어라. 이때 정리작업와 호출스택 되감기는 [RAII](#Re-raii)에게 맡겨라.

##### Example, don't

```c++
    void f()   // bad
    {
        try {
            // ...
        }
        catch (...) {
            // no action
            throw;   // propagate exception
        }
    }
```

##### Enforcement

* 중첩된 `try` 구문들을 지적하라
* 함수에 지나치게 빈번한 `try`블럭들을 사용한 소스 코드를 지적하라 ( "지나치게 빈번한"을 정의할 수 있는가??)

### <a name="Re-catch"></a>E.18:  명시적인 `try`/`catch`문의 사용을 최소화하라

##### Reason
`try`/`catch`는 번거롭고 오류를 발생시키기도 쉽다.
`try`/`catch`는 저레벨 자원 관리 또는 오류 처리가 체계적이지 않다는 신호일 수 있다.

##### Example, Bad

```c++
    void f(zstring s)
    {
        Gadget* p;
        try {
            p = new Gadget(s);
            // ...
            delete p;
        }
        catch (Gadget_construction_failure) {
            delete p;
            throw;
        }
    }
```

지저분한 코드다.
`try` 블록 내에 있는 포인터 `p`에서 누수가 발상할 수 있다.
모든 예외를 처리하지 않는다. 온전히 생성되지 않은 개체를 `delete`하는 것은 잘못된 사용법이다.

더 나은 생성방법:

```c++
    void f2(zstring s)
    {
        Gadget g {s};
    }
```

##### Alternatives

* 리소스 핸들과 [RAII](#Re-raii)를 사용하라
* [`finally`](#Re-finally)

##### Enforcement

 ??? 어렵다. 경험적인(Heuristic) 접근이 필요하다.

### <a name="Re-finally"></a>E.19: 적당한 리소스 핸들을 사용할 수 없다면 해제방법을 나타낼 `final_action` 개체를 사용하라

##### Reason

`finally`를 사용하는 것이 덜 번거롭고, `try`/`catch`에 비해 잘못 사용하기 어렵다.

##### Example

```c++
    void f(int n)
    {
        void* p = malloc(1, n);
        auto _ = finally([p] { free(p); });
        // ...
    }
```

##### Note

`finally`는 `try`/`catch`만큼 지저분하지는 않지만, 임시적인(ad-hoc) 방법이다. [적합한 리소스 관리 개체](#Re-raii)를 사용하라.
`finally`는 최후의 방법으로 생각하라.

##### Note

[`goto exit;` 방법](##Re-no-throw-codes)과 비교하면 `finally`는 언어 시스템을 사용하는 체계적이고 깔끔한 대안이다.

##### Enforcement

경험적이다 : `goto exit;`이 있는지 확인하라

### <a name="Re-no-throw-raii"></a>E.25: 예외를 던질 수 없는 경우, RAII로 자원을 관리하라

##### Reason

예외가 없다고 하더라도, [RAII](#Re-raii)는 보통 자원을 다루는 최고의, 가장 체계적인 방법이다.

##### Note

C++에서 예외를 이용한 오류처리는 지역적이지 않은 오류를 다루기 위한 방법이다. 특히 개체 생성에서 비-침습적으로 생성 실패를 알리는 데는 예외가 필요하다. 
오류의 발생을 알리는(signaling) 것이 무시되지 못하게 하기 위해선 예외가 필요하다. 예외를 사용할 수 없다면, 당신이 할 수 있는 한 그와 유사한 방법을 사용하라.

예외에 대한 많은 우려들은 잘못된 것이다.
포인터나 복잡한 제어 구조들로 인해 난잡하지 않은 환경에서 사용될때는 예외는 거의 언제나 (시간과 공간적인 측면에서) 신뢰할 수 있는 방법이고, 더 나은 코드로 이어진다. 이는 모든 시스템에서 해당하진 않지만 예외 처리 메커니즘이 잘 구현된 것을 전제한다.

위와 같은 문제를 적용할 수 없는 경우도 있다. 짧은 시간 내에 응답을 보장해야하는(Hard-real-time) 시스템들이 그런 경우인데, 정해진 시간 안에 오류나 정확한 답을 산출해야만 하는 연산들 때문이다. 시간을 측정할 수 있는 적합한 도구가 없다면, 예외가 이를 충족시키기는 어렵다. 그런 시스템(가령, 비행기 조종 소프트웨어)에서는 일반적으로 동적(heap) 메모리 사용 또한 배제되기도 한다.

결국, 오류 처리에 대한 최우선 가이드는 이렇다. "예외와 [RAII](#Re-raii)를 사용하라".
이 부분에서는 예외의 구현이나 예외가 효율적이지 않은 경우, 오래된 코드의 처리 스타일등 예외 처리를 사용할 수 없는 상황도 고려한다.
(가령, 엄청난 수의 포인터, 잘못 정의된 소유권, 오류코드 검사에 기반한 체계적이지 않은 오류처리들 등등)

결국, 오류 처리에 대한 최우선 가이드는 이렇다. "예외와 [RAII](#Re-raii)를 사용하라".
이 부분에서는 예외의 구현이나 예외가 효율적이지 않은 경우, 오래된 코드의 처리 스타일등 예외 처리를 사용할 수 없는 상황도 고려한다.
(가령, 엄청난 수의 포인터, 잘못 정의된 소유권, 오류코드 검사에 기반한 체계적이지 않은 오류처리들 등등)

예외와 그 비용에 대해 비난하기 전에, [error codes](#Re-no-throw-codes)에 있는 사례들을 고려해보라. 비용과 오류 코드를 다룰때의 복잡함을 모두 고려하라.

성능이 걱정된다면, 측정하라.

##### Example

이런 코드를 작성하고 싶다고 하자.

```c++
    void func(zstring arg)
    {
        Gadget g {arg};
        // ...
    }
```

`gadget`이 제대로 생성되지 않았다면, `func`는 예외를 던지며 종료한다. 예외를 던질 수 없다면, `gadget`에 `valid()`멤버 함수를 추가함으로써 RAII처럼 자원을 처리할 수도 있다:

```c++
    error_indicator func(zstring arg)
    {
        Gadget g {arg};
        if (!g.valid()) return gadget_construction_error;
        // ...
        return 0;   // zero indicates "good"
    }
```

이 경우 호출자(caller)는 당연히 반환값 검사가 필요하다는 것을 알고 있어야 한다.

**See also**: [Discussion](#Sd-???)

##### Enforcement

특별한 경우: 예컨대, 리소스 핸들의 생성을 `valid()`로 시스템적으로 검사하는 등의 방법이 가능하다

### <a name="Re-no-throw-crash"></a>E.26: 예외를 던질 수 없다면, 빠른 실패 전략을 고려하라

##### Reason

오류로부터 복원하는 작업을 제대로 수행할 수 없다면, 최소한 문제가 더 커지기 전에 탈출할 수는 있다.

**See also**: [Simulating RAII](#Re-no-throw-raii)

##### Note

지역적인 오류 처리를 제대로 할 수 없다면, "크래시 발생"을 사용하는 것을 고려해보라. 이는 오류를 확인한 문맥에서 복구하거나 지역적으로 처리할 수 없는 경우를 의미한다. `abort()`나 `quick_exit()`를 호출하라. 또는 시스템이 재시작하도록 하는 비슷한 기능의 함수를 호출하라.

다수의 프로세스나 컴퓨터들을 사용하는 시스템에서는 어찌되었건 치명적인 크래시를 처리하는 경우를 예상할 필요가 있다. 하드웨어에서 문제가 생길 수도 있다. 이런 경우, "크래시 발생"은 단순히 오류 처리를 시스템의 다음 레벨로 전달하는 것이 된다.

##### Example

```c++
    void f(int n)
    {
        // ...
        p = static_cast<X*>(malloc(n, X));
        if (!p) abort();     // abort if memory is exhausted
        // ...
    }
```

대부분의 프로그램은 메모리 고갈을 아름답게 해결하기 어렵다. 이는 다음 코드에서도 동일하다.

```c++
    void f(int n)
    {
        // ...
        p = new X[n];    // throw if memory is exhausted (by default, terminate)
        // ...
    }
```

일반적으로, "크래시"가 발생해서 종료하기 전 발생 원인에 대해 로그를 남기는 것은 현명한 생각이다.

##### Enforcement

Awkward

### <a name="Re-no-throw-codes"></a>E.27: 예외를 던질 수 없다면, 체계적으로 오류 코드를 사용하라

##### Reason

오류 핸들링 정책을 체계적으로 사용하는 것은 오류 처리를 잊어버릴 가능성을 최소화한다.

**See also**: [RAII](#Re-no-throw-raii)

##### Note

몇가지 이슈가 있다:

* 함수 바깥에서 오류 알림을 어떻게 전달할 것인가? 
* 오류로 종료하기 이전에 함수에서의 자원 해제는 어떻게 처리하는가?
* 오류의 지표(error indicator)로서 무엇을 사용하는가?

일반적으로, 오류 지표를 반환하는 것은 2가지 값을 반환한다는 것을 의미한다: 정상적인 결과와 오류 지표.
오류 지표는 `valid()` 함수를 가진 경우처럼 개체의 일부가 될수도 있다. 또는 함수가 한번에 2개의 값들을 반환할수도 있다.

##### Example

```c++
    Gadget make_gadget(int n)
    {
        // ...
    }

    void user()
    {
        Gadget g = make_gadget(17);
        if (!g.valid()) {
                // error handling
        }
        // ...
    }
```

이 접근법은 [RAII처럼 자원을 관리하라](#Re-no-throw-raii) 항목에 부합한다.
This approach fits with [simulated RAII resource management](#Re-no-throw-raii). `valid()`함수는 `error_indicator`를 반환할수도 있다. (예컨대, `error_indicator` 열거형 중에서 하나의 값을 반환한다).

##### Example

`Gadget`타입을 변경하길 원하지 않거나, 변경할 수 없는 경우는 어떨까? 그런 경우엔, 한 쌍의 값을 반환할 수 밖에 없다.

예를 들면:

```c++
    std::pair<Gadget, error_indicator> make_gadget(int n)
    {
        // ...
    }

    void user()
    {
        auto r = make_gadget(17);
        if (!r.second) {
                // error handling
        }
        Gadget& g = r.first;
        // ...
    }
```

이처럼, `std::pair`가 반환타입으로 사용 가능하다. 일부는 특별한 타입을 선호하기도 한다.

예를 들면:

```c++
    Gval make_gadget(int n)
    {
        // ...
    }

    void user()
    {
        auto r = make_gadget(17);
        if (!r.err) {
                // error handling
        }
        Gadget& g = r.val;
        // ...
    }
```

특별한 타입을 선호하는 이유중의 하나로는 `first`나 `second`같은 비밀스러운 이름보다는 멤버처럼 사용할 수 있기 때문이다. 이런 방법은 `std::pair`의 다른 사용법과 혼동되는 것을 방지한다.

##### Example

일반적으로, 오류로 인해 종료하기 전 정리 작업을 하게 된다. 이런 코드는 지저분하다:

```c++
    std::pair<int, error_indicator> user()
    {
        Gadget g1 = make_gadget(17);
        if (!g1.valid()) {
                return {0, g1_error};
        }

        Gadget g2 = make_gadget(17);
        if (!g2.valid()) {
                cleanup(g1);
                return {0, g2_error};
        }

        // ...

        if (all_foobar(g1, g2)) {
            cleanup(g1);
            cleanup(g2);
            return {0, foobar_error};
        // ...

        cleanup(g1);
        cleanup(g2);
        return {res, 0};
    }
```

RAII처럼 동작하도록 하는 것은 함수 내에서 다수의 자원과 다양한 오류가 발생할 수 있을 경우 더욱 중요하다. 드물게 사용되는 해법 중 하나는 반복을 피하기 위해 함수의 끝부분에 정리 작업을 모아놓는 것이다 (`goto`가 컴파일 되려면 `g2` 근처에서  불가피하게 추가로 유효 범위를 지정해야 한다):

```c++
    std::pair<int, error_indicator> user()
    {
        error_indicator err = 0;

        Gadget g1 = make_gadget(17);
        if (!g1.valid()) {
                err = g1_error;
                goto exit;
        }

        {
            Gadget g2 = make_gadget(17);
            if (!g2.valid()) {
                    err = g2_error;
                    goto exit;
            }

            if (all_foobar(g1, g2)) {
                err = foobar_error;
                goto exit;
            }
            // ...
        }

    exit:
      if (g1.valid()) cleanup(g1);
      if (g2.valid()) cleanup(g2);
      return {res, err};
    }
```

함수가 꽤 크다면, 이런 방법을 사용하게 될 가능성이 크다. `finally`를 사용하는 것이 [좀 더 쉬울 수 있다](#Re-finally). 또한, 프로그램이 커질수록 체계적으로 오류 지표(error indicator)를 사용한 오류 처리 전략을 체계적으로 적용하기 어려울 수 있다.

[예외 기반 오류 처리](#Re-throw)를 선호하며, [함수는 짧게](#Rf-single)작성하기를 권한다.

**See also**: [Discussion](#Sd-???)

**See also**: [다수의 값을 반환](#Rf-out-multi)

##### Enforcement

Awkward.

### <a name="Re-no-throw"></a>E.28: 전역 상태를 사용한 오류 처리를 지양하라(`errno`처럼)

##### Reason

전역 상태는 관리하기 어렵고, 확인하는 것을 잊어버리기 쉽다. 마지막으로 `printf()`의 반환값을 확인한 것이 언제인가?

**See also**: [RAII](#Re-no-throw-raii)

##### Example, bad

    ???

##### Note

C 스타일 오류처리는 전역 변수 `errno`에 의존한다, 때문에 이 가이드를 따르는 것이 필연적으로 불가할수도 있다.

##### Enforcement

Awkward.

### <a name="Re-specifications"></a>E.30: 예외 명세를 사용하지 마라

##### Reason

예외 명세는 오류 처리를 깨지기 쉽게 만들고, 실행시간 비용을 발생시키며, C++ 표준에서는 제거되었다.

##### Example

```c++
    int use(int arg)
        throw(X, Y)
    {
        // ...
        auto x = f(arg);
        // ...
    }
```

`f()`가 `X`나 `Y`가 아닌 다른 예외를 던진다면 엉뚱한 오류 처리 루틴이 호출된다. 만약 아무것도 없다면 프로그램은 종료된다.
이런 종료 자체는 괜찮지만 `f()`가 새로운 예외 `Z`를 던지도록 변경되었다고 가정해보자, `use()`를 변경하지 않는 한 계속 프로그램이 강제로 종료될 것이다 (결국 모두 새로 테스트 해봐야 한다).
`f()`가 라이브러리의 함수라서 손댈 수 없으면 `use()`는 난처해진다. `use()`가 `Z`를 전달하도록 할 수 있지만, `use()`를 호출하는 쪽에서도 변경이 필요하다. 문제가 순식간에 다룰 수 없게 되어버린다.

대안이 있다면, `use()`에서 `try`-`catch`를 사용해 `Z`를 다른 예외로 처리하는 것이다. 이 방법 또한 문제를 키우게 된다.
발생 가능한 예외들이 달라지는 경우가 시스템의 저수준에서 일어난다는 점이 중요하다 (예를 들어, 네트워크 라이브러리나 미들웨어가 변경되었다거나). 이 경우, 문제가 여러 수준의 많은 함수들을 거쳐서 나타나게 된다
코드의 규모가 크다면, 이는 마지막 사용자가 변경하기 전까지 새로운 버전으로 업데이트 할 수 없게 된다.
만약 `use()`가 라이브러리의 일부라면, 알수 없는 사용자들에게 미치는 영향때문에 업데이트를 할 수 없게 된다.

예외가 전파되도록 하는 전략은 이런 경우에 대응할 수 있다는 것이 지난 수년간의 경험으로 증명되었다.

##### Note

No. This would not be any better had exception specifications been statically enforced.
그 예로 [Stroustrup94](#Stroustrup94)를 참고하라.

##### Note

예외가 발생하지 않는다면, [`noexcept`](#Re-noexcept)를 지정하라. 그렇지 않으면 `throw()`와 동일하다.

##### Enforcement

예외 명세가 사용된 부분을 지적한다.

### <a name="Re_catch"></a>E.31: 적합한 순서로 `catch`를 배치하라

##### Reason

`catch` 구절은 순서대로 평가되기 때문에 다른 구절에 도달하지 못할 수도 있다.

##### Example

```c++
    void f()
    {
        // ...
        try {
                // ...
        }
        catch (Base& b) { /* ... */ }
        catch (Derived& d) { /* ... */ }
        catch (...) { /* ... */ }
        catch (std::exception& e){ /* ... */ }
    }
```

`Derived`가 `Base`를 상속받은 경우, `Derived`를 처리하는 부분은 절대로 호출되지 않는다. 
마찬가지로 "모든 것"을 잡는 핸들러로 인해 `std::exception`는 호출되지 않게 된다.

##### Enforcement

다른 핸들러를 숨기는 경우를 지적한다.
