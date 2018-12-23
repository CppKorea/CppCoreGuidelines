
# <a name="S-functions"></a>F: 함수

함수는 하나의 시스템이 조화로운(consistent) 상태에서 다음 상태로 넘어가도록 하는 동작(action)이나 계산(computation)을 작성하는(specify) 것이다. 
함수는 프로그램의 만들어나가는 기초(building block)가 된다.

함수의 이름은 전달인자(argument)들의 요구사항을 드러내고, 인자들과 호출 결과간의 관계를 명확히 기술해야 한다.
구현은 명세와 다르다. 함수가 어떤 일을 수행하는지를 어떻게 수행하는지와 동등하게 고려하라. 함수들은 대부분의 인터페이스의 핵심이므로, 인터페이스 규칙도 확인하라.

>
> 역주:
> * Parameter: 매개변수
> * Argument: 전달인자
>

함수 규칙 요약:

함수 정의 규칙:

* [F.1: 의미있는 동작들을 "묶어서" 심사숙고해 함수 이름을 지어라](#Rf-package)
* [F.2: 함수는 하나의 논리적 동작만 수행하도록 하라](#Rf-logical)
* [F.3: 함수는 간결하고 단순하게 유지시켜라](#Rf-single)
* [F.4: 함수가 컴파일 타임에 평가되어야 한다면 `constexpr`로 선언하라](#Rf-constexpr)
* [F.5: 함수가 매우 짧고 수행시간이 중요하다면 `inline`으로 선언하라](#Rf-inline)
* [F.6: 함수가 예외를 던지지 않는다면 `noexcept`로 선언하라](#Rf-noexcept)
* [F.7: 보편성을 고려한다면, 스마트 포인터 대신에 `T*`나 `T&` 타입의 인자를 사용하라](#Rf-smart)
* [F.8: 순수 함수를 선호하라](#Rf-pure)
* [F.9: 사용되지 않는 인자는 이름이 없어야 한다](#Rf-unused)

매개변수(parameter) 전달 표현 규칙:

* [F.15: 정보를 전달 할 때 단순하고 관습적인 방법을 선호하라](#Rf-conventional)
* [F.16: "입력" 매개 변수는 복사 비용이 적게드는 값 타입을 사용하거나 상수 참조형으로 전달하라](#Rf-in)
* [F.17: "입출력" 매개 변수는 비상수 참조형으로 전달하라](#Rf-inout)
* [F.18: "소모성" 매개 변수는 `X&&`타입과 `std::move`로 전달하라](#Rf-consume)
* [F.19: "forward" 매개 변수는 `TP&&`타입과 `std::forward`로만 전달하라](#Rf-forward)
* [F.20: "출력" 매개 변수로 값을 반환하는 방법을 선호하라](#Rf-out)
* [F.21: "출력"값 여러 개를 반환할 때는 튜플이나 구조체를 선호하라](#Rf-out-multi)
* [F.60: "인자가 없을 수도" 있다면 `T&`보다는 `T*`를 선호하라](#Rf-ptr-ref)

매개변수 전달 의미구조(semantic) 규칙:

* [F.22: Use `T*` or `owner<T*>` to designate a single object](#Rf-ptr)
* [F.23: Use a `not_null<T>` to indicate that "null" is not a valid value](#Rf-nullptr)
* [F.24: Use a `span<T>` or a `span_p<T>` to designate a half-open sequence](#Rf-range)
* [F.25: Use a `zstring` or a `not_null<zstring>` to designate a C-style string](#Rf-zstring)
* [F.26: Use a `unique_ptr<T>` to transfer ownership where a pointer is needed](#Rf-unique_ptr)
* [F.27: Use a `shared_ptr<T>` to share ownership](#Rf-shared_ptr)

<a name="Rf-value-return"></a>값 반환 의미구조 규칙:

* [F.42: Return a `T*` to indicate a position (only)](#Rf-return-ptr)
* [F.43: Never (directly or indirectly) return a pointer or a reference to a local object](#Rf-dangle)
* [F.44: Return a `T&` when copy is undesirable and "returning no object" isn't needed](#Rf-return-ref)
* [F.45: Don't return a `T&&`](#Rf-return-ref-ref)
* [F.46: `int` is the return type for `main()`](#Rf-main)
* [F.47: Return `T&` from assignment operators](#Rf-assignment-op)
* [F.48: Don't `return std::move(local)`](#Rf-return-move-local)

기타 함수 규칙:

* [F.50: Use a lambda when a function won't do (to capture local variables, or to write a local function)](#Rf-capture-vs-overload)
* [F.51: Where there is a choice, prefer default arguments over overloading](#Rf-default-args)
* [F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms](#Rf-reference-capture)
* [F.53: Avoid capturing by reference in lambdas that will be used nonlocally, including returned, stored on the heap, or passed to another thread](#Rf-value-capture)
* [F.54: If you capture `this`, capture all variables explicitly (no default capture)](#Rf-this-capture)
* [F.55: Don't use `va_arg` arguments](#F-varargs)

함수는 람다와 함수개체와 강한 연관성을 가지고 있다.

##### See also

[C.lambdas: Function objects and lambdas](#SS-lambdas)

## <a name="SS-fct-def"></a>F.def: 함수 정의(definition)

함수 정의는 함수를 선언하고 본문를 구현하는 것을 의미한다.

### <a name="Rf-package"></a>F.1: "Package" meaningful operations as carefully named functions

##### Reason

공통 코드를 만드는 것은 가독성, 재사용성을 높이고, 복잡한 코드에서 오류 발생을 제한한다.
만약 어떤 동작이 잘 정의되어 있다면 주변 코드로부터 분리하고 이름을 부여하라.

##### Example, don't

```c++
    void read_and_print(istream& is)    // read and print an int
    {
        int x;
        if (is >> x)
            cout << "the int is " << x << '\n';
        else
            cerr << "no int on input\n";
    }
```

`read_and_print`는 대부분이 틀렸다.
이 함수는 읽고, (`ostream`에) 쓰고, (`ostream`에) 오류 메시지를 쓴다. 그리고 `int`만을 다룬다.
재사용 가능한 코드가 없고 논리적으로 분리된 동작들은 뒤섞여있으며 지역변수는 사용이 끝난 후에도 존재한다.
간단한 이 예제는 문제가 없어보이지만, 입력동작, 출력동작 그리고 오류처리가 좀 더 복잡해지면 코드가 뒤엉켜서 이해하기 어려워진다.

##### Note

만약 한 곳 이상에서 사용 될 중요한 람다 함수를 작성한다면 (비지역)변수에 할당하고 이름을 지어줘라.

##### Example

```c++
    sort(a, b, [](T x, T y) { return x.rank() < y.rank() && x.value() < y.value(); });
```

람다에 이름을 짓게되면 표현식을 여러개의 논리적 부분으로 나눌 수 있고, 람다가 어떤 일을 하는지 가늠할 수 있다.

```c++
    auto lessT = [](T x, T y) { return x.rank() < y.rank() && x.value() < y.value(); };

    sort(a, b, lessT);
    find_if(a, b, lessT);
```

짧은 코드가 항상 유지보수나 성능에 최선인 것은 아니다.

##### Exception

반복문(loop) 몸체, 반복문 몸체로 사용되는 람다는 이름을 가질 필요가 거의 없다.
하지만 수 십라인이 넘거나 수 페이지가 된다면 문제가 될 수 있다.
[함수를 간결하게 유지하라](#Rf-single) 규칙은 "반복문을 간결하게 유지하라"는 규칙을 내포한다. 콜백 인자로 사용되는 람다는 재사용하지 않지만 사소하게 볼 수 없는 경우도 있다.

##### Enforcement

* [함수를 간결하게 유지하라](#Rf-single)를 참고하라
* 동일하거나 매우 비슷한 람다가 여러 곳에서 사용되면 지적하라

### <a name="Rf-logical"></a>F.2: A function should perform a single logical operation

##### Reason

하나의 작업만 수행하는 함수는 이해하기 쉽고, 테스트하기 쉽고, 재사용하기 쉽다.

##### Example

다음을 고려해 보자:

```c++
    void read_and_print()    // bad
    {
        int x;
        cin >> x;
        // check for errors
        cout << x << "\n";
    }
```

위 함수는 특정한 입력에 속박되어 있고 다른 쓰임새는 찾을 수 없다. 대신, 함수를 의미있는 논리적 작업으로 나누고 매개변수를 사용하라:

```c++
    int read(istream& is)    // better
    {
        int x;
        is >> x;
        // check for errors
        return x;
    }

    void print(ostream& os, int x)
    {
        os << x << "\n";
    }
```

필요하다면 두 함수를 결합하면 된다:

```c++
    void read_and_print()
    {
        auto x = read(cin);
        print(cout, x);
    }
```

또한 필요하다면 `read()`와 `print()`에서 사용하는 데이터 타입, 입력 메커니즘, 오류에 대한 응답 등을 템플릿화 할 수 있다.

예를 들어:

```c++
    auto read = [](auto& input, auto& value)    // better
    {
        input >> value;
        // check for errors
    };

    auto print(auto& output, const auto& value)
    {
        output << value << "\n";
    }
```

##### Enforcement

* 출력 매개 변수가 2개 이상인 함수를 의심하라. 대신 반환값을 사용하라. 여러 반환값을 저장 할 수 있는 `tuple`을 사용해도 좋다.
* 편집기 화면에 다 나오지 않을 만큼 큰 함수를 의심하라. 이런 함수는 세부 동작을 갖는 더 작은 함수들로 (이름을 잘 지어서) 나누도록 한다.
* 7개 이상의 매개 변수를 갖는 함수를 의심하라.

### <a name="Rf-single"></a>F.3: Keep functions short and simple

##### Reason

긴 함수는 읽기 어렵고, 복잡하고, 변수가 필요한 유효범위 이상으로 사용되고 있을지도 모른다.  
복잡한 제어구조를 가진 함수는 더 길고 논리적 오류가 숨겨져 있을 수 있다

##### Example

Consider:

```c++
    double simple_func(double val, int flag1, int flag2)
        // simple_func: takes a value and calculates the expected ASIC output,
        // given the two mode flags.
    {
        double intermediate;
        if (flag1 > 0) {
            intermediate = func1(val);
            if (flag2 % 2)
                 intermediate = sqrt(intermediate);
        }
        else if (flag1 == -1) {
            intermediate = func1(-val);
            if (flag2 % 2)
                 intermediate = sqrt(-intermediate);
            flag1 = -flag1;
        }
        if (abs(flag2) > 10) {
            intermediate = func2(intermediate);
        }
        switch (flag2 / 10) {
        case 1: if (flag1 == -1) return finalize(intermediate, 1.171);
                break;
        case 2: return finalize(intermediate, 13.1);
        default: break;
        }
        return finalize(intermediate, 0.);
    }
```

이 함수는 너무 복잡하다 (그리고 너무 길다).
모든 경우가 올바르게 처리되는지 어떻게 알 수 있겠는가?
게다가, 이 함수는 다른 규칙들도 어기고 있다.

이렇게 바꿔쓸 수 있다:

```c++
    double func1_muon(double val, int flag)
    {
        // ???
    }

    double func1_tau(double val, int flag1, int flag2)
    {
        // ???
    }

    double simple_func(double val, int flag1, int flag2)
        // simple_func: takes a value and calculates the expected ASIC output,
        // given the two mode flags.
    {
        if (flag1 > 0)
            return func1_muon(val, flag2);
        if (flag1 == -1)
            // handled by func1_tau: flag1 = -flag1;
            return func1_tau(-val, flag1, flag2);
        return 0.;
    }
```

##### Note

"한 화면에 맞추기"는 "너무 크게 하지 않기"를 막는 좋은 실용적인 규칙이다.
함수의 길이는 최대 다섯줄을 기본으로 생각해야 한다.

##### Note

긴 함수는 응집성있고 의미있는 이름을 가진 작은 함수로 나누어야 한다. 작고 간결한 함수는 함수 호출 비용이 중요한 곳에서 `inline`처리될 수 있다.

##### Enforcement

* "한 화면에 맞지 않는" 함수는 지적한다.  
  화면은 어느정도 크기로 할 것인가? 한 줄에 140자, 60줄 화면을 사용해보라; 이는 대략 책의 한 페이지에 맞는 최대 크기이다.
* 너무 복잡한 함수는 지적한다.  
  너무 복잡한은 어느정도를 의미하는가? 순환 복잡도(cyclomatic complexity)를 쓸 수도 있다. "10개의 논리적 경로"를 사용해보라. 단순한 switch는 하나로 세어도 좋다.

### <a name="Rf-constexpr"></a>F.4: If a function may have to be evaluated at compile time, declare it `constexpr`

##### Reason

`constexpr`는 컴파일 시간에 평가하라고 컴파일러에게 지시하는데 사용된다.

##### Example

유명한 팩토리얼:

```c++
    constexpr int fac(int n)
    {
        constexpr int max_exp = 17;      // constexpr enables max_exp to be used in Expects
        Expects(0 <= n && n < max_exp);  // prevent silliness and overflow
        int x = 1;
        for (int i = 2; i <= n; ++i) x *= i;
        return x;
    }
```

C++14 에서는 이와 같이 작성할 수 있다. C++ 11 환경이라면, `fac()`를 재귀를 사용해 작성해야 한다.

##### Note

`constexpr`은 컴파일 타임 평가를 보장하지 않는다; 단지 프로그래머가 요구하거나 컴파일러가 최적화를 하기로 결정했을 때 상수 표현 인자에 대해서 컴파일 타임에 평가 될 수 있다는 것만을 보장 할 뿐이다.

```c++
    constexpr int min(int x, int y) { return x < y ? x : y; }

    void test(int v)
    {
        int m1 = min(-1, 2);            // probably compile-time evaluation
        constexpr int m2 = min(-1, 2);  // compile-time evaluation
        int m3 = min(-1, v);            // run-time evaluation
        constexpr int m4 = min(-1, v);  // error: cannot evaluate at compile time
    }
```

##### Note

`constexpr` 함수는 순수 함수들이며, 부수효과(side deffect)를 가지지 않는다.

```c++
    int dcount = 0;
    constexpr int double(int v)
    {
        ++dcount;   // error: attempted side effect from constexpr function
        return v + v;
    }
```

대체적으로 좋은 특성이다.

When given a non-constant argument, a `constexpr` function can throw.
If you consider exiting by throwing a side effect, a `constexpr` function isn't completely pure; if not, this is not an issue.

??? A question for the committee: can a constructor for an exception thrown by a `constexpr` function modify state?  
"No" would be a nice answer that matches most practice.

##### Note

모든 함수를 `constexpr`로 작성하지는 마라. 대부분의 계산은 실행시간에 최적으로 수행된다.

##### Note

어떤 API가 높은 수준의 실행시간 설정(configuration) 혹은 비즈니스 로직에 의존한다면 `constexpr`로 작성해선 안된다.
그와 같은 경우는 컴파일러에 의해 평가될 수 없으며, 그 API에 의존하는 `constexpr` 함수들은 재구성(refactored)되거나 `constexpr`를 포기(drop)하게 될 것이다.

##### Enforcement

불가능하며 불필요하다.  
컴파일러가 상수가 필요한 곳에 `constexpr`가 아닌 함수들이 사용되면 오류로 처리할 것이다.

### <a name="Rf-inline"></a>F.5: If a function is very small and time-critical, declare it `inline`

##### Reason

일부 최적화기(optimizer)는 별도로 힌트를 받지 않아도 함수 인라인화를 잘 하지만, 그에 의존해서는 안된다.
측정하라! 지난 40년간 우리는 컴파일러가 아무런 힌트가 없어도 사람보다 더 인라인화를 잘 할거라고 약속해 왔다.
그리고 그 약속은 아직 지켜지지 않았다. `inline`을 명시하는 것은 컴파일러가 더 나은 코드를 생성하도록 권장하는 것이다.

##### Example

```c++
    inline string cat(const string& s, const string& s2) { return s + s2; }
```

##### Exception

함수가 변하지 않을 것이라고 확신하지 않는 한, `inline`을 안정된 인터페이스 함수에 사용해선 안된다.
인라인 함수는 ABI의 일부이다.

##### Note

`constexpr`은 `inline`을 내포하고 있다.

##### Note

클래스 내에 정의된 멤버 함수들은 기본적으로 `inline`이 적용된다.

##### Exception

템플릿 함수(템플릿 멤버 함수 포함)들은 보통 헤더 파일에 정의되기 때문에 인라인 함수에 해당한다.

##### Enforcement

`inline`함수가 3 문장보다 길고 (클래스의 멤버 함수처럼) 다른 곳에 선언되었다면 지적한다.

### <a name="Rf-noexcept"></a>F.6: If your function may not throw, declare it `noexcept`

##### Reason

예외를 던지지 않기로 했다면, 프로그램은 오류를 처리하지 않을 것이기 때문에 가능한 빠르게 종료되어야 한다. `noexcept`를 선언하면 최적화기가 선택하는 여러가지 실행경로들을 사전에 제거할 수 있도록 도와준다.

##### Example

온전히 C언어로 구현이 되었거나 예외를 지원하지 않는 모든 함수에 `noexcept`를 추가하라. C++ 표준 라이브러리는 C 표준 라이브러리에 대해서 암시적으로 그렇게하고 있다.

##### Note

`constexpr` 함수가 실행시간에 평가된다면 예외가 발생할 수 있기 때문에, `noexcept`를 명시해야 할 수 있다

##### Example

예외를 던질 수 있는 함수에서 `noexcept`를 사용하는 것도 가능하다:

```c++
    vector<string> collect(istream& is) noexcept
    {
        vector<string> res;
        for (string s; is >> s;)
            res.push_back(s);
        return res;
    }
```

`collect()` 함수가 메모리를 모두 사용해 버리면 프로그램은 비정상적으로 종료하게 된다. 프로그램이 메모리 고갈 상태를 해결할 수 없다면, 이는 당연한 결과라고 할 수 있다;
`terminate()` 함수에서 적합한 오류 기록(error log)을 생성할 수 있다. (하지만 메모리 부족 상황에서는 제대로 할 수 있는 일이 거의 없다)

##### Note

함수에 `noexcept`를 사용할 때는 당신의 코드가 실행되는 환경에 대해 알고 있어야 한다. 이는 예외를 던지면서 발생하는 메모리 할당 때문이다.
완벽하게 일반화되어 사용되는 (표준 라이브러리나 정렬같은 유틸리티) 코드는 `bad_alloc` 예외가 제대로 처리되는 환경을 지원해야 한다.
하지만, 대부분의 프로그램과 실행환경은 할당이 실패하는 경우를 제대로 처리할 수 없고 그럴 때는 프로그램을 강제종료(abort)하는 것이 깔끔하고 단순한 처리방법이다.
만약 당신의 코드가 할당 실패를 처리할 수 없다면, 할당을 수행하는 함수에 `noexcept`를 사용하는 것이 적절할 수 있다.

Put another way:  
대부분의 프로그램에서는 함수들은 보통 예외를 던진다 (함수 안에서 `new`를 사용하거나 예외를 던지는 방식으로 실패를 알리는 함수/라이브러리를 사용하는 경우). 따라서 발생가능한 예외가 처리될 수 있는지 고민하지 않고 `noexcept`를 남발해서는 안된다.

`noexcept`는 빈번히 호출되는 저수준 함수들에 유용하다 (또한 정확하다).

##### Note

소멸자, `swap` 함수, move 연산 그리고 기본 생성자에서는 절대로 예외를 던지면 안된다.

##### Enforcement

* 예외를 던질 수 없는데도 `noexcept`가 없는 함수가 있다면 지적한다
* 예외를 던지는 `swap`, move 연산자, 소멸자 그리고 기본 생성자가 있다면 지적한다

### <a name="Rf-smart"></a>F.7: For general use, take `T*` or `T&` arguments rather than smart pointers

##### Reason

스마트 포인터를 인자로 사용하면 소유권이 이전되거나 공유된다. 이는 의도적인 경우에만 사용되어야 한다. ([R.30](#Rr-smartptrparam) 참고).
스마트 포인터를 인자로 사용하면 함수 호출 시 스마트 포인터를 사용해야한다는 제약이 생긴다.
공유 스마트 포인터를 인자로 사용하는 것은 (예, `std::shared_ptr`) 런타임시 추가 비용을 발생시킨다.

##### Example

```c++
    // accepts any int*
    void f(int*);

    // can only accept ints for which you want to transfer ownership
    void g(unique_ptr<int>);

    // can only accept ints for which you are willing to share ownership
    void g(shared_ptr<int>);

    // doesn't change ownership, but requires a particular ownership of the caller
    void h(const unique_ptr<int>&);

    // accepts any int
    void h(int&);
```

##### Example, bad

```c++
    // callee
    void f(shared_ptr<widget>& w)
    {
        // ...
        use(*w); // only use of w -- the lifetime is not used at all
        // ...
    };
```

[R.30](#Rr-smartptrparam)에서 관련 내용을 기술하고 있다.

##### Note

허상 포인터(dangling pointer)는 정적으로 잡아낼 수 있다. 때문에 허상 포인터로 인한 자원 관리에 의존할 필요는 없다.

##### See also

* [전달인자가 없는 경우가 허용된다면 `T&`보다는 `T*`를 선호하라](#Rf-ptr-ref)
* [스마트 포인터 규칙 요약](#Rr-summary-smartptrs)

##### Enforcement

소유권 의미구조를 사용하지 않는데 스마트 포인터 타입을 인자로 사용한다면 지적한다 (또는 `operator->`나 `operator*`를 중복정의한 타입). 이런 경우는

* 복사 가능하지만 복사/이동이 발생하지 않는다 혹은 이동 가능하지만 이동하지 않는다
* 값을 변경하지 않거나 변경하지 않는 다른 함수로 전달한다

### <a name="Rf-pure"></a>F.8: Prefer pure functions

##### Reason

간결한 함수는 이유를 이해하기 쉽고, 최적화하기 쉽고(병렬화를 포함한다), 메모이제이션하기 쉽다.

##### Example

```c++
    template<class T>
    auto square(T t) { return t * t; }
```

##### Note

`constexpr`는 순수 함수에 속한다.

상수가 아닌 전달인자로 호출된 경우, `constexpr`함수는 예외를 던질 수 있다.
If you consider exiting by throwing a side effect, a `constexpr` function isn't completely pure;
if not, this is not an issue.

??? A question for the committee: can a constructor for an exception thrown by a `constexpr` function modify state?
"No" would be a nice answer that matches most practice.

##### Enforcement

불가능하다.

### <a name="Rf-unused"></a>F.9: Unused parameters should be unnamed

##### Reason

가독성. "사용되지 않는 인자" 경고가 발생하지 않게 한다.

##### Example

```c++
    X* find(map<Blob>& m, const string& s, Hint);   // once upon a time, a hint was used
```

##### Note

이 문제를 다루기 위해 1980년대 초에 이름 없는 매개변수를 허용하게 되었다

##### Enforcement

이름이 있지만 사용되지 않는 매개변수를 지적한다.

## <a name="SS-call"></a>F.call: 인자 전달(Parameter passing)

함수에 인자를 전달하고 반환값을 받는데는 다양한 방법이 있다.

### <a name="Rf-conventional"></a>F.15: Prefer simple and conventional ways of passing information

##### Reason

"별나면서 교묘한" 기법은 깜짝놀랄만한 버그를 만들어내거나, 다른 프로그래머가 코드를 이해하는데 어렵게 만든다.
정말로 일반적인 기법을 넘어서는 방법으로 최적화를 해야 한다면 꼭 필요한 개선사항이라는것을 확신할 수 있어야하고, 이식성이 없을 수 있기 때문에 문서나 주석을 남겨야 한다.

아래의 표는 핵심 가이드라인의 조언(F.16-21)을 요약한 것이다.

매개변수 전달(Normal):

![Normal parameter passing table](../images/param-passing-normal.png)

매개변수 전달(Advanced):

![Advanced parameter passing table](../images/param-passing-advanced.png)

필요한 경우에만 고급 기술을 사용하고, 주석으로 문서화하라.

### <a name="Rf-in"></a>F.16: For "in" parameters, pass cheaply-copied types by value and others by reference to `const`

##### Reason

두 경우 모두 호출자가 전달인자를 변경하지 않는다는 것을 알 수 있다. 또한 r-value 초기화를 허용한다.

"큰 비용 없이 복사" 한다는 것은 실행기(machine)의 구조(architecture)에 따라 다르다. 하지만 보통 2,3개의 워드(double, 포인터, 참조)를 값으로 전달할때 최적이다.

비용이 적다면, 단순성과 안전성에서 복사보다 나은 방법은 없다. 또한 작은 개체(2,3개 워드까지)에 대해선 참조보다 복사가 빠른데 함수에서 간접(in-direct)접근없이 사용할 수 있기 때문이다.

##### Example

```c++
    void f1(const string& s);  // OK: pass by reference to const; always cheap

    void f2(string s);         // bad: potentially expensive

    void f3(int x);            // OK: Unbeatable

    void f4(const int& x);     // bad: overhead on access in f4()
```

"입력 전용" 매개변수로 전달된 r-value를 최적화하고자 한다면:

* 함수에서 무조건적으로 전달인자를 이동(move)받는다면, `&&`를 사용하라. [F.18](#Rf-consume) 참고
* 인자의 복사본을 사용한다면, 매개변수에 (l-value인 경우) `const&`를 사용하는 함수와 (r-value인 경우) `&&`를 받아 필요한 영역에 `std::move`하는 함수를 중복 정의하라. 원래 이는 "will-move-from"을 중복정의한 것이다. [F.18](#Rf-consume) 참고
* "입력 + 복사"가 여럿 발생하는 특별한 경우에는, "perfect forwarding" 사용을 고려하라. [F.19](#Rf-forward) 참고

##### Example

```c++
    int multiply(int, int); // just input ints, pass by value

    // suffix is input-only but not as cheap as an int, pass by const&
    string& concatenate(string&, const string& suffix);

    void sink(unique_ptr<widget>);  // input only, and moves ownership of the widget
```

아래와 같은 "난해한 기술"은 지양하라:

* "효율적이라서" 인자를 `T&&`로 전달한다. `&&`로 전달함으로써 발생하는 성능 향상에 대한 루머는 잘못되었고 깨지기 쉽다(속단하지 말고 [F.18](#Rf-consume)와 [F.19](#Rf-forward)를 참고하라)
* 대입에서 `const T&`를 반환하거나 비슷한 연산을 수행한다 ([F.47](#Rf-assignment-op) 참고)

##### Example

`Matrix`가 이동 연산을 지원한다고 가정하자(아마도 원소들을 `std::vector`에 보관하고 있다):

```c++
    Matrix operator+(const Matrix& a, const Matrix& b)
    {
        Matrix res;
        // ... fill res with the sum ...
        return res;
    }

    Matrix x = m1 + m2;  // move constructor

    y = m3 + m3;         // move assignment
```

##### Notes

반환 값 최적화는 대입에 대해서는 동작하지 않지만, 이동 대입의 경우에는 적용된다.
참조는 언어 규칙에 의해 유효한 개체를 가리킨다고 가정하기 때문에, null 참조는 발생하지 않는다.
optional 값에 대해 알고 있다면, 포인터를 사용하거나, `std::optional` 혹은 "값이 없음"을 의미하는 특별한 값을 사용하라.

##### Enforcement

* (Simple) ((Foundation)) Warn when a parameter being passed by value has a size greater than `4 * sizeof(int)`.
  Suggest using a reference to `const` instead.
* (Simple) ((Foundation)) Warn when a `const` parameter being passed by reference has a size less than `3 * sizeof(int)`. Suggest passing by value instead.
* (Simple) ((Foundation)) Warn when a `const` parameter being passed by reference is `move`d.

### <a name="Rf-inout"></a>F.17: For "in-out" parameters, pass by reference to non-`const`

##### Reason

호출자에게 값이 변경될 수 있다는 점을 분명히 할 수 있다.

##### Example

```c++
    void update(Record& r);  // assume that update writes to r
```

##### Note

`T&` 인자는 정보를 전달할 수도 있지만 받아올 수도 있다.
때문에 `T&`는 입출력 매개변수가 될 수 있다. 이로 인해 문제가 되거나 오류의 원인이 되기도 한다:

```c++
    void f(string& s)
    {
        s = "New York";  // non-obvious error
    }

    void g()
    {
        string buffer = ".................................";
        f(buffer);
        // ...
    }
```

여기서, `g()` 작성자는  `f()`에게 버퍼를 제공하고 있지만,  `f()`는 참조를 변경해버린다 (이는 문자들을 단순히 복사하는 것보다 비용이 좀 더 발생한다).
`g()`에서 `buffer`의 크기를 잘못 가정한다면 오류가 발생할 수 있다.

##### Enforcement

* (Moderate) ((Foundation)) Warn about functions regarding reference to non-`const` parameters that do *not* write to them.
* (Simple) ((Foundation)) Warn when a non-`const` parameter being passed by reference is `move`d.

### <a name="Rf-consume"></a>F.18: For "will-move-from" parameters, pass by `X&&` and `std::move` the parameter

##### Reason

효율적이고 호출하는 지점에서 버그를 없앤다: `X&&`는 r-value에 연결되며(bind), l-value를 전달하는 경우 명시적으로 `std::move`를 호출해야 한다.

##### Example

```c++
    void sink(vector<int>&& v) {   // sink takes ownership of whatever the argument owned
        // usually there might be const accesses of v here
        store_somewhere(std::move(v));
        // usually no more use of v here; it is moved-from
    }
```

`store_somewhere()`를 호출할 때 `std::move(v)`를 사용한 결과 `v`가 값을 넘겨준(moved-from) 상태로 만든다는 점에 주의하라. 
[이는 위험할 수도 있다](#Rc-move-semantic).

##### Exception

`unique_ptr`와 같은 유일한 소유자 타입들은 이동만 가능(move-only)하며 쉽게 이동된다(cheap-to-move). 이 타입들은 쉽게 값 전달(pass by value) 코드를 작성하고 수행할 수 있다. 값 전달은 이동 연산이 한번 더 발생하지만, 분명함과 단순함을 우선하라.

예를 들어:

```c++
    template <class T>
    void sink(std::unique_ptr<T> p) {
        // use p ... possibly std::move(p) onward somewhere else
    }   // p gets destroyed
```

##### Enforcement

* 모든 `std::move`없이 `X&&` 매개변수를 사용하면 지적한다 (이때 `X`는 템플릿 인자가 아니다)
* 값을 넘겨준(moved-from) 개체에 접근하면 지적한다
* 조건부로 개체를 이동시키지 말아라

### <a name="Rf-forward"></a>F.19: For "forward" parameters, pass by `TP&&` and only `std::forward` the parameter

##### Reason

만약 개체가 해당 함수에서 바로 사용되지 않고 다른 코드로 전달된다면, 그 함수는 전달인자가 상수(`const`)인 경우이거나 r-value인 경우에도 동작하도록 작성되어야 한다.

`TP`가 템플릿형 매개변수면 `TP&&`는 포워딩 참조가 된다 -- 이 때 상수 속성과 rvalue 속성은 *무시* 되기도하고 *보존* 되기도 한다. 그래서 `T&&`를 사용하는 코드는 변수의 상수 속성과 rvalue 속성에 게의치 않는다는 의미를 내포하지만 (어차피 무시되기 때문에), 값을 전달하는 코드에서는 상수 속성과 rvalue 속성을 신경쓴다 (보존이 되기 때문에). `TP&&`형 매개변수에 임시객체가 전달되면 함수가 실행되는 동안에는 유효하기 때문에 안전하다. `TP&&`형 매개변수는 항상 `std::forward`를 이용하여 함수의 몸체에서 전달되어야 한다.

##### Example

```c++
    template <class F, class... Args>
    inline auto invoke(F f, Args&&... args) {
        return f(forward<Args>(args)...);
    }

    ??? calls ???
```
##### Enforcement

* 모든 정적 경로에 대해 단 한번 `std::forward`하는 경우를 제외하고 `TP&&` 매개변수를 받는 함수를 지적한다 (`TP`는 템플릿 인자의 이름이다). 

### <a name="Rf-out"></a>F.20: For "out" output values, prefer return values to output parameters

##### Reason

A return value is self-documenting, whereas a `&` could be either in-out or out-only and is liable to be misused.

This includes large objects like standard containers that use implicit move operations for performance and to avoid explicit memory management.

If you have multiple values to return, [use a tuple](#Rf-out-multi) or similar multi-member type.

##### Example

```c++
    // OK: return pointers to elements with the value x
    vector<const int*> find_all(const vector<int>&, int x);

    // Bad: place pointers to elements with value x in-out
    void find_all(const vector<int>&, vector<const int*>& out, int x);
```

##### Note

A `struct` of many (individually cheap-to-move) elements may be in aggregate expensive to move.

It is not recommended to return a `const` value.
Such older advice is now obsolete; it does not add value, and it interferes with move semantics.

```c++
    const vector<int> fct();    // bad: that "const" is more trouble than it is worth

    vector<int> g(const vector<int>& vx)
    {
        // ...
        fct() = vx;   // prevented by the "const"
        // ...
        return fct(); // expensive copy: move semantics suppressed by the "const"
    }
```

The argument for adding `const` to a return value is that it prevents (very rare) accidental access to a temporary.
The argument against is prevents (very frequent) use of move semantics.

##### Exceptions

* For non-value types, such as types in an inheritance hierarchy, return the object by `unique_ptr` or `shared_ptr`.
* If a type is expensive to move (e.g., `array<BigPOD>`), consider allocating it on the free store and return a handle (e.g., `unique_ptr`), or passing it in a reference to non-`const` target object to fill (to be used as an out-parameter).
* To reuse an object that carries capacity (e.g., `std::string`, `std::vector`) across multiple calls to the function in an inner loop: [treat it as an in/out parameter and pass by reference](#Rf-out-multi).

##### Example

```c++
    struct Package {      // exceptional case: expensive-to-move object
        char header[16];
        char load[2024 - 16];
    };

    Package fill();       // Bad: large return value
    void fill(Package&);  // OK

    int val();            // OK
    void val(int&);       // Bad: Is val reading its argument
```

##### Enforcement

* Flag reference to non-`const` parameters that are not read before being written to and are a type that could be cheaply returned; they should be "out" return values.
* Flag returning a `const` value. To fix: Remove `const` to return a non-`const` value instead.

### <a name="Rf-out-multi"></a>F.21: To return multiple "out" values, prefer returning a struct or tuple

##### Reason

A return value is self-documenting as an "output-only" value.
Note that C++ does have multiple return values, by convention of using a `tuple` (including `pair`),
possibly with the extra convenience of `tie` at the call site.
Prefer using a named struct where there are semantics to the returned value. Otherwise, a nameless `tuple` is useful in generic code.

##### Example

```c++
    // BAD: output-only parameter documented in a comment
    int f(const string& input, /*output only*/ string& output_data)
    {
        // ...
        output_data = something();
        return status;
    }

    // GOOD: self-documenting
    tuple<int, string> f(const string& input)
    {
        // ...
        return make_tuple(status, something());
    }
```

사실, C++98의 표준 라이브러리에서는 `pair`가 개체 2개를 묶은 `tuple`과 같기 때문에 이 기능을 편리하게 사용하고 있었다.

예를 들어, `set<string> my_set`이 주어졌다고 가정하면:

```c++
    // C++98
    result = my_set.insert("Hello");
    if (result.second) do_something_with(result.first);    // workaround
```

C++11에서는 이렇게 작성할 수 있다, 결과값들을 이미 존재하는 지역변수에 대입한다:

```c++
    Sometype iter;                                // default initialize if we haven't already
    Someothertype success;                        // used these variables for some other purpose

    tie(iter, success) = my_set.insert("Hello");   // normal return value
    if (success) do_something_with(iter);
```

With C++17 we should be able to use "structured bindings" to declare and initialize the multiple variables:

```c++
    if (auto [ iter, success ] = my_set.insert("Hello"); success) do_something_with(iter);
```

##### Exception

Sometimes, we need to pass an object to a function to manipulate its state.
In such cases, passing the object by reference [`T&`](#Rf-inout) is usually the right technique.
Explicitly passing an in-out parameter back out again as a return value is often not necessary.
For example:

```c++
    istream& operator>>(istream& is, string& s);    // much like std::operator>>()

    for (string s; cin >> s; ) {
        // do something with line
    }
```

Here, both `s` and `cin` are used as in-out parameters.
We pass `cin` by (non-`const`) reference to be able to manipulate its state.
We pass `s` to avoid repeated allocations.
By reusing `s` (passed by reference), we allocate new memory only when we need to expand `s`'s capacity.
This technique is sometimes called the "caller-allocated out" pattern and is particularly useful for types,
such as `string` and `vector`, that needs to do free store allocations.

To compare, if we passed out all values as return values, we would something like this:

```c++
    pair<istream&, string> get_string(istream& is);  // not recommended
    {
        string s;
        is >> s;
        return {is, s};
    }

    for (auto p = get_string(cin); p.first; ) {
        // do something with p.second
    }
```

We consider that significantly less elegant with significantly less performance.

For a truly strict reading of this rule (F.21), the exception isn't really an exception because it relies on in-out parameters,
rather than the plain out parameters mentioned in the rule.
However, we prefer to be explicit, rather than subtle.

##### Note

In many cases, it may be useful to return a specific, user-defined type.
For example:

```c++
    struct Distance {
        int value;
        int unit = 1;   // 1 means meters
    };

    Distance d1 = measure(obj1);        // access d1.value and d1.unit
    auto d2 = measure(obj2);            // access d2.value and d2.unit
    auto [value, unit] = measure(obj3); // access value and unit; somewhat redundant
                                        // to people who know measure()
    auto [x, y] = measure(obj4);        // don't; it's likely to be confusing
```

The overly-generic `pair` and `tuple` should be used only when the value returned represents to independent entities rather than an abstraction.

Another example, use a specific type along the lines of `variant<T, error_code>`, rather than using the generic `tuple`.

##### Enforcement

* Output parameters should be replaced by return values.
  An output parameter is one that the function writes to, invokes a non-`const` member function, or passes on as a non-`const`.

### <a name="Rf-ptr"></a>F.22: Use `T*` or `owner<T*>` to designate a single object

##### Reason

Readability: it makes the meaning of a plain pointer clear.
Enables significant tool support.

##### Note

In traditional C and C++ code, plain `T*` is used for many weakly-related purposes, such as:

* Identify a (single) object (not to be deleted by this function)
* Point to an object allocated on the free store (and delete it later)
* Hold the `nullptr`
* Identify a C-style string (zero-terminated array of characters)
* Identify an array with a length specified separately
* Identify a location in an array

This makes it hard to understand what the code does and is supposed to do.
It complicates checking and tool support.

##### Example

```c++
    void use(int* p, int n, char* s, int* q)
    {
        p[n - 1] = 666; // Bad: we don't know if p points to n elements;
                        // assume it does not or use span<int>
        cout << s;      // Bad: we don't know if that s points to a zero-terminated array of char;
                        // assume it does not or use zstring
        delete q;       // Bad: we don't know if *q is allocated on the free store;
                        // assume it does not or use owner
    }
```

better

```c++
    void use2(span<int> p, zstring s, owner<int*> q)
    {
        p[p.size() - 1] = 666; // OK, a range error can be caught
        cout << s; // OK
        delete q;  // OK
    }
```

##### Note

`owner<T*>` represents ownership, `zstring` represents a C-style string.

**Also**: Assume that a `T*` obtained from a smart pointer to `T` (e.g., `unique_ptr<T>`) points to a single element.

##### See also

* [Support library](#S-gsl)
* [Do not pass an array as a single pointer](#Ri-array)

##### Enforcement

* (Simple) ((Bounds)) Warn for any arithmetic operation on an expression of pointer type that results in a value of pointer type.

### <a name="Rf-nullptr"></a>F.23: Use a `not_null<T>` to indicate that "null" is not a valid value

##### Reason

명확성. `not_null<T>`인자는 함수 호출자가 `nullptr` 검사를 해야만 한다고 명확하게 보여준다.
유사하게, `not_null<T>`을 반환한다면 함수 호출자는 반환 값이 `nullptr`인지 검사해야 할 필요가 없다.

##### Example

`not_null<T*>` makes it obvious to a reader (human or machine) that a test for `nullptr` is not necessary before dereference.
Additionally, when debugging, `owner<T*>` and `not_null<T>` can be instrumented to check for correctness.

Consider:

```c++
    int length(Record* p);
```

When I call `length(p)` should I check if `p` is `nullptr` first? Should the implementation of `length()` check if `p` is `nullptr`?

```c++
    // it is the caller's job to make sure p != nullptr
    int length(not_null<Record*> p);

    // the implementor of length() must assume that p == nullptr is possible
    int length(Record* p);
```

##### Note

A `not_null<T*>` is assumed not to be the `nullptr`; a `T*` may be the `nullptr`; both can be represented in memory as a `T*` (so no run-time overhead is implied).

##### Note

`not_null` is not just for built-in pointers. It works for `unique_ptr`, `shared_ptr`, and other pointer-like types.

##### Enforcement

* (Simple) Warn if a raw pointer is dereferenced without being tested against `nullptr` (or equivalent) within a function, suggest it is declared `not_null` instead.
* (Simple) Error if a raw pointer is sometimes dereferenced after first being tested against `nullptr` (or equivalent) within the function and sometimes is not.
* (Simple) Warn if a `not_null` pointer is tested against `nullptr` within a function.

* (단순) 원시 포인터를 `nullptr`인지 검사하지 않고 사용하면 경고한다 `not_null`를 쓰도록 제안한다
* (단순) 포인터가 역참조 될 때 `nullptr`를 검사할 때도 있고 검사하지 않을 때도 있다면 오류로 처리한다
* (단순) `not_null`이 `nullptr`인지 검사하는 경우 경고한다

### <a name="Rf-range"></a>F.24: Use a `span<T>` or a `span_p<T>` to designate a half-open sequence

##### Reason

Informal/non-explicit ranges are a source of errors.

##### Example
```c++
    X* find(span<X> r, const X& v);    // find v in r

    vector<X> vec;
    // ...
    auto p = find({vec.begin(), vec.end()}, X{});  // find X{} in vec
```
##### Note

Ranges are extremely common in C++ code. Typically, they are implicit and their correct use is very hard to ensure.
In particular, given a pair of arguments `(p, n)` designating an array `[p:p+n)`,
it is in general impossible to know if there really are `n` elements to access following `*p`.
`span<T>` and `span_p<T>` are simple helper classes designating a `[p:q)` range and a range starting with `p` and ending with the first element for which a predicate is true, respectively.

##### Example

A `span` represents a range of elements, but how do we manipulate elements of that range?
```c++
    void f(span<int> s)
    {
        // range traversal (guaranteed correct)
        for (int x : s) cout << x << '\n';

        // C-style traversal (potentially checked)
        for (gsl::index i = 0; i < s.size(); ++i) cout << s[i] << '\n';

        // random access (potentially checked)
        s[7] = 9;

        // extract pointers (potentially checked)
        std::sort(&s[0], &s[s.size() / 2]);
    }
```
##### Note

A `span<T>` object does not own its elements and is so small that it can be passed by value.

Passing a `span` object as an argument is exactly as efficient as passing a pair of pointer arguments or passing a pointer and an integer count.

##### See also

[Support library](#S-gsl)

##### Enforcement

(Complex) Warn where accesses to pointer parameters are bounded by other parameters that are integral types and suggest they could use `span` instead.

### <a name="Rf-zstring"></a>F.25: Use a `zstring` or a `not_null<zstring>` to designate a C-style string

##### Reason

C언어 형식의 문자열은 광범위하게 사용되고 있다. 관례적으로, 이들은 `\0`으로 끝나는 `char`배열이라고 정의되어 있다.
C 문자열은 `char` 1개에 대한 포인터와 구분되어야 한다.

##### Example

Consider:

```c++
    int length(const char* p);
```

`length(s)`를 호출 할 때 `s==nullptr`을 검사해야 하는가? `length()` 본문 안에서 `p`가 `nullptr`인지 검사해야 하는가?

```c++
    // the implementor of length() must assume that p == nullptr is possible
    int length(zstring p);

    // it is the caller's job to make sure p != nullptr
    int length(not_null<zstring> p);
```

##### Note

`zstring`은 소유권을 표현하지 않는다.

##### See also

[Support library](#S-gsl)

### <a name="Rf-unique_ptr"></a>F.26: Use a `unique_ptr<T>` to transfer ownership where a pointer is needed

##### Reason

Using `unique_ptr` is the cheapest way to pass a pointer safely.

##### See also

[C.50](#Rc-factory) regarding when to return a `shared_ptr` from a factory.

##### Example

```c++
    unique_ptr<Shape> get_shape(istream& is)  // assemble shape from input stream
    {
        auto kind = read_header(is); // read header and identify the next shape on input
        switch (kind) {
        case kCircle:
            return make_unique<Circle>(is);
        case kTriangle:
            return make_unique<Triangle>(is);
        // ...
        }
    }
```

##### Note

You need to pass a pointer rather than an object if what you are transferring is an object from a class hierarchy that is to be used through an interface (base class).

##### Enforcement

(Simple) Warn if a function returns a locally allocated raw pointer. Suggest using either `unique_ptr` or `shared_ptr` instead.

### <a name="Rf-shared_ptr"></a>F.27: Use a `shared_ptr<T>` to share ownership

##### Reason

`std::shared_ptr`로 소유권을 공유하는 것은 표준에서 사용하는 방법이다. 이를 사용하면, 마지막 소유자가 개체를 소멸시킨다.

##### Example

```c++
    shared_ptr<const Image> im { read_image(somewhere) };

    std::thread t0 {shade, args0, top_left, im};
    std::thread t1 {shade, args1, top_right, im};
    std::thread t2 {shade, args2, bottom_left, im};
    std::thread t3 {shade, args3, bottom_right, im};

    // detach threads
    // last thread to finish deletes the image
```

##### Note

소유자가 하나 뿐이라면 `shared_ptr`보다는 `unique_ptr`을 사용하라. 
`shared_ptr`는 소유권의 공유를 위한 것이다.

Note that pervasive use of `shared_ptr` has a cost (atomic operations on the `shared_ptr`'s reference count have a measurable aggregate cost).

##### Alternative

Have a single object own the shared object (e.g. a scoped object) and destroy that (preferably implicitly) when all users have completed.

##### Enforcement

(Not enforceable) 제대로 탐지하기엔 너무 복잡한 패턴을 띄고 있다.

### <a name="Rf-ptr-ref"></a>F.60: Prefer `T*` over `T&` when "no argument" is a valid option

##### Reason

A pointer (`T*`) can be a `nullptr` and a reference (`T&`) cannot, there is no valid "null reference".
Sometimes having `nullptr` as an alternative to indicated "no object" is useful, but if it is not, a reference is notationally simpler and might yield better code.

##### Example
```c++
    string zstring_to_string(zstring p) // zstring is a char*; that is a C-style string
    {
        if (!p) return string{};    // p might be nullptr; remember to check
        return string{p};
    }

    void print(const vector<int>& r)
    {
        // r refers to a vector<int>; no check needed
    }
```
##### Note

It is possible, but not valid C++ to construct a reference that is essentially a `nullptr` (e.g., `T* p = nullptr; T& r = (T&)*p;`).
That error is very uncommon.

##### Note

If you prefer the pointer notation (`->` and/or `*` vs. `.`), `not_null<T*>` provides the same guarantee as `T&`.

##### Enforcement

* Flag ???

### <a name="Rf-return-ptr"></a>F.42: Return a `T*` to indicate a position (only)

##### Reason

포인터는 이를 표현하기에 적절하다. 
소유권을 전달하기 위해 `T*`를 사용하는 것은 잘못된 방법이다.

##### Example

```c++
    Node* find(Node* t, const string& s)  // find s in a binary tree of Nodes
    {
        if (!t || t->name == s) return t;
        if ((auto p = find(t->left, s))) return p;
        if ((auto p = find(t->right, s))) return p;
        return nullptr;
    }
```

`nullptr`가 아니라면 `find`가 반환하는 포인터는 `s`를 가지는 `node`를 의미한다.
중요한점은 개체를 가리키는 포인터로는 소유권이 호출자까지 전달되지 않는다는 것이다.

##### Note

Positions can also be transferred by iterators, indices, and references.
A reference is often a superior alternative to a pointer [if there is no need to use `nullptr`](#Rf-ptr-ref) or [if the object referred to should not change](???).

##### Note

Do not return a pointer to something that is not in the caller's scope; see [F.43](#Rf-dangle).

##### See also

[discussion of dangling pointer prevention](#???)

##### Enforcement

* Flag `delete`, `std::free()`, etc. applied to a plain `T*`. Only owners should be deleted.
* Flag `new`, `malloc()`, etc. assigned to a plain `T*`. Only owners should be responsible for deletion.

### <a name="Rf-dangle"></a>F.43: Never (directly or indirectly) return a pointer or a reference to a local object

##### Reason

To avoid the crashes and data corruption that can result from the use of such a dangling pointer.

##### Example, bad

After the return from a function its local objects no longer exist:

```c++
    int* f()
    {
        int fx = 9;
        return &fx;  // BAD
    }

    void g(int* p)   // looks innocent enough
    {
        int gx;
        cout << "*p == " << *p << '\n';
        *p = 999;
        cout << "gx == " << gx << '\n';
    }

    void h()
    {
        int* p = f();
        int z = *p;  // read from abandoned stack frame (bad)
        g(p);        // pass pointer to abandoned stack frame to function (bad)
    }
```

Here on one popular implementation I got the output:

```c++
    *p == 999
    gx == 999
```

I expected that because the call of `g()` reuses the stack space abandoned by the call of `f()` so `*p` refers to the space now occupied by `gx`.

* Imagine what would happen if `fx` and `gx` were of different types.
* Imagine what would happen if `fx` or `gx` was a type with an invariant.
* Imagine what would happen if more that dangling pointer was passed around among a larger set of functions.
* Imagine what a cracker could do with that dangling pointer.

Fortunately, most (all?) modern compilers catch and warn against this simple case.

##### Note

This applies to references as well:

```c++
    int& f()
    {
        int x = 7;
        // ...
        return x;  // Bad: returns reference to object that is about to be destroyed
    }
```

##### Note

This applies only to non-`static` local variables.
All `static` variables are (as their name indicates) statically allocated, so that pointers to them cannot dangle.

##### Example, bad

Not all examples of leaking a pointer to a local variable are that obvious:

```c++
    int* glob;       // global variables are bad in so many ways

    template<class T>
    void steal(T x)
    {
        glob = x();  // BAD
    }

    void f()
    {
        int i = 99;
        steal([&] { return &i; });
    }

    int main()
    {
        f();
        cout << *glob << '\n';
    }
```

Here I managed to read the location abandoned by the call of `f`.
The pointer stored in `glob` could be used much later and cause trouble in unpredictable ways.

##### Note

The address of a local variable can be "returned"/leaked by a return statement, by a `T&` out-parameter, as a member of a returned object, as an element of a returned array, and more.

##### Note

Similar examples can be constructed "leaking" a pointer from an inner scope to an outer one;
such examples are handled equivalently to leaks of pointers out of a function.

A slightly different variant of the problem is placing pointers in a container that outlives the objects pointed to.

##### See also

Another way of getting dangling pointers is [pointer invalidation](#???).
It can be detected/prevented with similar techniques.

##### Enforcement

* Compilers tend to catch return of reference to locals and could in many cases catch return of pointers to locals.
* Static analysis can catch many common patterns of the use of pointers indicating positions (thus eliminating dangling pointers)

### <a name="Rf-return-ref"></a>F.44: Return a `T&` when copy is undesirable and "returning no object" isn't needed

##### Reason

언어가 `T&`는 객체를 가리키고 있다는 것을 보장하기 때문에 `nullptr`인지 시험하는 것은 필요없다.

##### See also

The return of a reference must not imply transfer of ownership:
[discussion of dangling pointer prevention](#???) and [discussion of ownership](#???).

##### Example

```c++
    class Car
    {
        array<wheel, 4> w;
        // ...
    public:
        wheel& get_wheel(int i) { Expects(i < w.size()); return w[i]; }
        // ...
    };

    void use()
    {
        Car c;
        wheel& w0 = c.get_wheel(0); // w0 has the same lifetime as c
    }
```

##### Enforcement

Flag functions where no `return` expression could yield `nullptr`

### <a name="Rf-return-ref-ref"></a>F.45: Don't return a `T&&`

##### Reason

이것은 소멸된 임시 개체에 대한 참조를 반환하는 것이다. `&&`는 임시 개체를 붙잡기 위한 것이다.

##### Example 

A returned rvalue reference goes out of scope at the end of the full expression to which it is returned:

```c++
    auto&& x = max(0, 1);   // OK, so far
    foo(x);                 // Undefined behavior
```

This kind of use is a frequent source of bugs, often incorrectly reported as a compiler bug.
An implementer of a function should avoid setting such traps for users.

The [lifetime safety profile](#SS-lifetime) will (when completely implemented) catch such problems.


##### Example

Returning an rvalue reference is fine when the reference to the temporary is being passed "downward" to a callee;
then, the temporary is guaranteed to outlive the function call (see [F.18](#Rf-consume) and [F.19](#Rf-forward)).
However, it's not fine when passing such a reference "upward" to a larger caller scope.
For passthrough functions that pass in parameters (by ordinary reference or by perfect forwarding) and want to return values, use simple `auto` return type deduction (not `auto&&`).

Assume that `F` returns by value:

```c++
    template<class F>
    auto&& wrapper(F f)
    {
        log_call(typeid(f)); // or whatever instrumentation
        return f();          // BAD: returns a reference to a temporary
    }
```

Better:

```c++
    template<class F>
    auto wrapper(F f)
    {
        log_call(typeid(f)); // or whatever instrumentation
        return f();          // OK
    }
```

##### Exception

`std::move` 와 `std::forward`는 `&&`를 반환하지만 이는 형변환일 뿐이다 -- used by convention only in expression contexts where a reference to a temporary object is passed along within the same expression before the temporary is destroyed. We don't know of any other good examples of returning `&&`.

##### Enforcement

`std::move` 와 `std::forward`를 제외하고 `&&`를 반환한다면 지적한다

### <a name="Rf-main"></a>F.46: `int` is the return type for `main()`

##### Reason

It's a language rule, but violated through "language extensions" so often that it is worth mentioning.
Declaring `main` (the one global `main` of a program) `void` limits portability.

##### Example

```c++
    void main() { /* ... */ };  // bad, not C++
    
    int main()
    {
        std::cout << "This is the way to do it\n";
    }
```

##### Note

We mention this only because of the persistence of this error in the community.

##### Enforcement

* The compiler should do it
* If the compiler doesn't do it, let tools flag it

### <a name="Rf-assignment-op"></a>F.47: Return `T&` from assignment operators

##### Reason

The convention for operator overloads (especially on value types) is for
`operator=(const T&)` to perform the assignment and then return (non-`const`)
`*this`.  This ensures consistency with standard-library types and follows the
principle of "do as the ints do."

##### Note

Historically there was some guidance to make the assignment operator return `const T&`.
This was primarily to avoid code of the form `(a = b) = c` -- such code is not common enough to warrant violating consistency with standard types.

##### Example

```c++
    class Foo
    {
     public:
        ...
        Foo& operator=(const Foo& rhs) {
          // Copy members.
          ...
          return *this;
        }
    };
```

##### Enforcement

This should be enforced by tooling by checking the return type (and return
value) of any assignment operator.


### <a name="Rf-return-move-local"></a>F.48: Don't `return std::move(local)`

##### Reason

With guaranteed copy elision, it is now almost always a pessimization to expressly use `std::move` in a return statement.

##### Example; bad

```c++
    S f()
    {
      S result;
      return std::move(result);
    }
```

##### Example; good

```c++
    S f()
    {
      S result;
      return result;
    }
```

##### Enforcement

This should be enforced by tooling by checking the return expression .


### <a name="Rf-capture-vs-overload"></a>F.50: Use a lambda when a function won't do (to capture local variables, or to write a local function)

##### Reason

함수는 지역변수를 캡쳐할 수 없고, 지역 유효범위로 선언될 수도 없다; 이런 기능이 필요하다면 람다를 사용하거나 직접 작성한 함수 개체를 사용해야 한다 (가능한 람다를 사용하라). 하지만, 람다와 함수개체는 오버로드가 되지 않는다; 오버로드가 필요하다면 함수를 사용하라. 두 방법 모두 가능하다면 함수를 선호하라; 단순한 방법을 사용하라.

##### Example

```c++
    // writing a function that should only take an int or a string
    // -- overloading is natural
    void f(int);
    void f(const string&);

    // writing a function object that needs to capture local state and appear
    // at statement or expression scope -- a lambda is natural
    vector<work> v = lots_of_work();
    for (int tasknum = 0; tasknum < max; ++tasknum) {
        pool.run([=, &v]{
            /*
            ...
            ... process 1 / max - th of v, the tasknum - th chunk
            ...
            */
        });
    }
    pool.join();
```

##### Exception

제네릭 람다는 함수 템플릿을 구현하는 간결한 방법을 제공하기 때문에 코드를 조금 더 작성하면 일반 함수 템플릿과 같은 기능을 사용할 수 있다.
미래에 모든 함수들이 Concept 인자를 사용할 수 있게 되면 이 기능은 사라질지도 모른다.

##### Enforcement

* Warn on use of a named non-generic lambda (e.g., `auto x = [](int i){ /*...*/; };`) that captures nothing and appears at global scope. Write an ordinary function instead.

### <a name="Rf-default-args"></a>F.51: Where there is a choice, prefer default arguments over overloading

##### Reason

Default arguments simply provide alternative interfaces to a single implementation.
There is no guarantee that a set of overloaded functions all implement the same semantics.
The use of default arguments can avoid code replication.

##### Note

There is a choice between using default argument and overloading when the alternatives are from a set of arguments of the same types.
For example:

```c++
    void print(const string& s, format f = {});
```

as opposed to

```c++
    void print(const string& s);  // use default format
    void print(const string& s, format f);
```

There is not a choice when a set of functions are used to do a semantically equivalent operation to a set of types. For example:

```c++
    void print(const char&);
    void print(int);
    void print(zstring);
```

##### See also

[Default arguments for virtual functions](#Rh-virtual-default-arg)

##### Enforcement

    ???

### <a name="Rf-reference-capture"></a>F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms

##### Reason

지역범위에서 람다를 사용할 때는 대부분의 경우 효율성과 정확성을 위해 참조캡쳐(capture by reference)를 선호할 것이다. 여기에는 함수가 반환하기 전에 병렬 알고리즘을 작성하거나 호출할때도 포함된다.

##### Discussion

The efficiency consideration is that most types are cheaper to pass by reference than by value.

The correctness consideration is that many calls want to perform side effects on the original object at the call site (see example below). Passing by value prevents this.

##### Note

Unfortunately, there is no simple way to capture by reference to `const` to get the efficiency for a local call but also prevent side effects.

##### Example

Here, a large object (a network message) is passed to an iterative algorithm, and is it not efficient or correct to copy the message (which may not be copyable):

```c++
    std::for_each(begin(sockets), end(sockets), [&message](auto& socket)
    {
        socket.send(message);
    });
```

##### Example

아래 예제는 간단한 3단계 병렬 파이프라인이다.
각 `stage` 개체는 `process` 함수를 통해 작업을 전달하고 작업 큐가 소진될 때까지 소멸되지 않는 작업용 스레드들을 캡슐화 한 것이다.

```c++
    void send_packets(buffers& bufs)
    {
        stage encryptor([] (buffer& b){ encrypt(b); });
        stage compressor([&](buffer& b){ compress(b); encryptor.process(b); });
        stage decorator([&](buffer& b){ decorate(b); compressor.process(b); });
        for (auto& b : bufs) { decorator.process(b); }
    }  // automatically blocks waiting for pipeline to finish
```

##### Enforcement

Flag a lambda that captures by reference, but is used other than locally within the function scope or passed to a function by reference. (Note: This rule is an approximation, but does flag passing by pointer as those are more likely to be stored by the callee, writing to a heap location accessed via a parameter, returning the lambda, etc. The Lifetime rules will also provide general rules that flag escaping pointers and references including via lambdas.)

### <a name="Rf-value-capture"></a>F.53: Avoid capturing by reference in lambdas that will be used nonlocally, including returned, stored on the heap, or passed to another thread

##### Reason

지역범위에 있는 포인터와 참조는 범위를 넘어서면 더 이상 존재하지 않는다.
참조캡쳐를 가진 람다는 지역 개체를 참조하고 있을 뿐이며, 지역범위를 넘어서면 더 이상 참조해서는 않된다.

##### Example, bad

```c++
    int local = 42;

    // Want a reference to local.
    // Note, that after program exits this scope,
    // local no longer exists, therefore
    // process() call will have undefined behavior!
    thread_pool.queue_work([&]{ process(local); });
```

##### Example, good

```c++
    int local = 42;
    // Want a copy of local.
    // Since a copy of local is made, it will
    // always be available for the call.
    thread_pool.queue_work([=]{ process(local); });
```

##### Enforcement

* (Simple) Warn when capture-list contains a reference to a locally declared variable
* (Complex) Flag when capture-list contains a reference to a locally declared variable and the lambda is passed to a non-`const` and non-local context

### <a name="Rf-this-capture"></a>F.54: If you capture `this`, capture all variables explicitly (no default capture)

##### Reason

It's confusing. Writing `[=]` in a member function appears to capture by value, but actually captures data members by reference because it actually captures the invisible `this` pointer by value. If you meant to do that, write `this` explicitly.

##### Example

```c++
    class My_class {
        int x = 0;
        // ...

        void f() {
            int i = 0;
            // ...

            auto lambda = [=]{ use(i, x); };   // BAD: "looks like" copy/value capture
            // [&] has identical semantics and copies the this pointer under the current rules
            // [=,this] and [&,this] are not much better, and confusing

            x = 42;
            lambda(); // calls use(0, 42);
            x = 43;
            lambda(); // calls use(0, 43);

            // ...

            auto lambda2 = [i, this]{ use(i, x); }; // ok, most explicit and least confusing

            // ...
        }
    };
```

##### Note

This is under active discussion in standardization, and may be addressed in a future version of the standard by adding a new capture mode or possibly adjusting the meaning of `[=]`. For now, just be explicit.

##### Enforcement

* Flag any lambda capture-list that specifies a default capture and also captures `this` (whether explicitly or via default capture)

### <a name="F-varargs"></a>F.55: Don't use `va_arg` arguments

##### Reason

Reading from a `va_arg` assumes that the correct type was actually passed.
Passing to varargs assumes the correct type will be read.
This is fragile because it cannot generally be enforced to be safe in the language and so relies on programmer discipline to get it right.

##### Example

```c++
    int sum(...) {
        // ...
        while (/*...*/)
            result += va_arg(list, int); // BAD, assumes it will be passed ints
        // ...
    }

    sum(3, 2); // ok
    sum(3.14159, 2.71828); // BAD, undefined

    template<class ...Args>
    auto sum(Args... args) { // GOOD, and much more flexible
        return (... + args); // note: C++17 "fold expression"
    }

    sum(3, 2); // ok: 5
    sum(3.14159, 2.71828); // ok: ~5.85987
```

##### Alternatives

* overloading
* variadic templates
* `variant` arguments
* `initializer_list` (homogeneous)

##### Note

Declaring a `...` parameter is sometimes useful for techniques that don't involve actual argument passing, notably to declare "take-anything" functions so as to disable "everything else" in an overload set or express a catchall case in a template metaprogram.

##### Enforcement

* Issue a diagnostic for using `va_list`, `va_start`, or `va_arg`.
* Issue a diagnostic for passing an argument to a vararg parameter of a function that does not offer an overload for a more specific type in the position of the vararg. To fix: Use a different function, or `[[suppress(types)]]`.
