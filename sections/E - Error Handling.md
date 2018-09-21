# <a name="S-errors"></a>E(Error handling): 오류 처리
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#S-errors)  

> 역주 : 에러 처리(Error handling)

에러 처리는 4종류로 나눌 수 있다:
 * 에러 확인하기(Detecting)
 * 에러에 대한 정보를 처리 코드로 전달하기
 * 프로그램을 올바른 상태(valid state)로 유지하기
 * 자원 누수(leak)를 방지하기

모든 에러를 복구하는 것은 불가능하다. 에러 복구가 여의치 않다면 정의된 방식으로 빨리 "빠져나가는" 것이 중요하다. 에러 처리 방법은 간단해야 한다. 그렇지 않으면 훨씬 더 안좋은 에러를 발생시킬 수 있다. 검증되지 않고(untested) 희소하게(rarely) 실행되는 에러 처리 코드는 그 자체로 많은 버그의 원인이 되기도 한다.

이 문서의 규칙들은 다음과 같은 에러들의 발생을 방지하는 것을 목표로 한다:

 * 타입 위반 (예: `union`의 오용과 형변환)
 * 리소스 누수 (메모리 누수를 포함)
 * 범위를 벗어나는 에러
 * 수명 에러 (예: `delete`된 객체 접근하기)
 * 복잡성 에러 (복잡한 표현으로 인해 발생하는 논리적 에러)
 * 인터페이스 에러 (예: 예상치 못한 값이 전달되는 경우)

> 역주 : 범위(Bound)

에러 처리 규칙 요약:

 * [E.1: 설계 과정 초기에 에러 처리에 대한 전략을 수립하라](#Re-design)
 * [E.2: 함수가 맡은 작업을 처리할 수 없다는 것을 알리기 위해 예외를 발생시켜라](#Re-throw)
 * [E.3: 예외는 에러 처리에만 사용하라](#Re-errors)
 * [E.4: 불변조건에 맞게 에러 처리 전략을 설계하라](#Re-design-invariants)
 * [E.5: 생성자에서 불변조건을 설정하고, 그럴 수 없으면 예외를 발생시켜라](#Re-invariant)
 * [E.6: RAII를 사용해 누수를 방지하라](#Re-raii)
 * [E.7: 선행조건을 기술하라](#Re-precondition)
 * [E.8: 후행조건을 기술하라](#Re-postcondition)


 * [E.12: `throw`를 허용하지 않거나 불가능한 함수에는 `noexcept`를 사용하라](#Re-noexcept)
 * [E.13: 객체를 소유하는 중에는 예외를 발생시켜선 안된다](#Re-never-throw)
 * [E.14: 목적에 맞게 설계된 사용자 정의 타입을 예외로 사용하라 (내장 타입은 안된다)](#Re-exception-types)
 * [E.15: 계층 구조가 있는 예외는 참조를 사용해서 처리하라](#Re-exception-ref)
 * [E.16: 소멸자, 자원해제, `swap`은 절대 실패해선 안된다](#Re-never-fail)
 * [E.17: 각각의 함수들에서 모든 예외를 처리하려고 하지 마라](#Re-not-always)
 * [E.18: 명시적인 `try`/`catch`문의 사용을 최소화하라](#Re-catch)
 * [E.19: 적당한 리소스 핸들을 사용할 수 없다면 처리방법을 나타낼 `final_action` 객체를 사용하라](#Re-finally)


* [E.25: 예외를 던질 수 없는 경우, RAII처럼 자원을 관리하라](#Re-no-throw-raii)
* [E.26: 예외를 던질 수 없는 경우, 빠른 실패 전략을 고려하라](#Re-no-throw-crash)
* [E.27: 예외를 던질 수 없는 경우, 체계적으로 에러 코드를 사용하라](#Re-no-throw-codes)
* [E.28: 전역 상태를 사용한 에러 처리를 지양하라(`errno`처럼)](#Re-no-throw)

> 역주 : 불변조건(invariant)   
> 역주 : 빠른 실패(failing fast)

### <a name="Re-design"></a>E.1: 설계 과정 초기에 에러 처리에 대한 전략을 수립하라

##### 근거

에러와 리소스 누수를 처리하기 위한 일관성있고 완전한 전략은 기존의 시스템에 새로 추가하기 아주 어렵다.



### <a name="Re-throw"></a>E.2: 함수가 맡은 작업을 처리할 수 없다는 것을 알리기 위해 예외를 발생시켜라

##### 근거

에러처리가 체계적이고, 견고하며, 반복적이지 않게 된다.

##### 예

```c++
    struct Foo 
    {
        vector<Thing> v;
        File_handle f;
        string s;
    };

    void use()
    {
        Foo bar { { Thing{1}, Thing{2}, Thing{monkey} }, 
                  { "my_file", "r"}, 
                  "Here we go!" };
        // ...
    }
```
여기 `vector`, `string`의 생성자는 충분한 메모리를 할당받지 못할 수 있다. `vector`의 생성자는 `Thing`을 초기화 리스트에 복사할 수 없을 수 있고,
`File_handle`은 파일을 열지 못할 수 있다. 각각의 경우, `use()`의 호출자가 처리할 수 있도록 예외를 발생시킨다. 혹은 `use()`가 직접 `bar` 생성 실패를 처리하려고 하면 `try`/`catch`를 사용하면 된다.

두 경우 모두, `Foo`의 생성자가 제어권을 넘기기 전에 이미 생성된 멤버들을 정확히 소멸시킨다. 이때는 에러 코드를 저장한 반환값이 없다는 점에 주의하라.

`File_handle`의 생성자는 다음과 같은 코드일 수 있다:
```c++
    File_handle::File_handle(const string& name, const string& mode)
        :f{ fopen(name.c_str(), mode.c_str()) }
    {
        if (!f)
            throw runtime_error{
                "File_handle: could not open " + name + " as " + mode};
    }

```

##### 참고사항
예외의 의도는 예외적인 사건이나 실패를 알리는데 있다.
하지만 "무엇이 예외적인가?"라는 점에서 순환적인 의미를 가진다.

예를 들면: 
 * 충족되지 않은 선행조건
 * 객체를 생성할 수 없는 생성자 (클래스의 [불변조건](#Rc-struct) 설정 실패)
 * 범위를 벗어나는(out-of-range) 에러 
 * 자원 획득의 실패(예: 네트워크 다운)

이와 반대로, 일반적은 루프의 종료는 예외적이지 않다. 무한 루프가 아니라면, 루프가 종료하는 것은 일반적으로 기대하는 것이다.

##### 참고사항

함수가 값을 반환하는 방법으로 `throw`를 사용하지 마라.

##### 예외
몇몇 제약이 심한 시스템들은, 실행되기 전에 (일반적으로 짧은) 고정된 최대 처리시간을 보장해야 한다. 이런 경우에는 정확한 시간을 준수하고 복원하기 위해 예외를 던질 수도 있다.

##### 같이 보기
 - [RAII](#Re-raii)
 - [discussion](#Sd-noexcept)

##### 참고사항

예외를 사용한 에러 처리를 좋아하지 않거나 결정을 내릴 수 없다면, [대안들](#Re-no-throw-raii)을 검토해보라.

에러 처리 방법들은 제각기 복잡한 부분과 문제가 있기 마련이다. 
가능하다면, 처리 방법의 효율성에 대해서 측정을 해보라.




### <a name="Re-errors"></a>E.3: 예외는 에러 처리에만 사용하라

##### 근거
"정상적인 코드"와 에러 처리를 분리하도록 해준다.
C++에서는 예외가 드물게 발생하다는 전제 하에 최적화를 적용한다.

##### 잘못된 예

```c++
    // 잘못된 코드:
    //   exception이 에러 처리를 위해 사용되지 않고 있다.
    int find_index(vector<string>& vec, const string& x)
    {
        try {
            for (int i = 0; i < vec.size(); ++i)
                if (vec[i] == x) throw i;  // found x
        } catch (int i) {
            return i;
        }
        return -1;   // not found
    }
```
이런 코드는 복잡하고 일반적인 for문 보다 훨씬 느릴 것이다. 
`vector`에서 값을 찾는데 예외적인 상황은 없다.

##### 시행하기
경험적인 면이 필요하다. 
`catch`구문이 예외 값을 "잡지 못하는지" 확인해보라.



### <a name="Re-design-invariants"></a>E.4: 불변조건에 맞게 에러 처리 전략을 설계하라

> 역주 : 불변조건(invariant)   
> 역주 : 올바른, 유효한(valid)   

##### 근거
객체를 사용하려면 그 객체는 (불변조건을 사용해서 정의된) 올바른 상태에 있어야 한다. 또 에러로부터 객체들을 복원하려면, 파괴되지 않은 객체들은 올바른 상태에 있어야 한다.

##### 참고사항

[invariant](#Rc-struct)는 생성자가 수립해야 하는 객체 멤버들에 대한 논리적 조건을 의미한다.
이런 조건들은 `public` 멤버 함수들에서 가정하는 것들이다.


##### 시행하기

???




### <a name="Re-invariant"></a>E.5: 생성자에서 불변조건을 설정하고, 그럴 수 없으면 예외를 발생시켜라

##### 근거

조건을 만족하지 않는 객체를 구성하면 문제를 발생시킬 수 있다. 
멤버 함수 중 일부만 호출할 수 있다.

##### 예

```c++
    // double들의 간단한 vector
    //   elem이 nullptr가 아니면 sz개의 double들을 가리키고 있다.
    class Vector 
    {  
    public:
        Vector() : elem{nullptr}, sz{0}{}
        Vector(int s) : elem{new double[sz]}, sz{s} { 
            /* 원소 초기화 */ 
        }
        ~Vector() { 
            delete[] elem; 
        }
        double& operator[](int s) { 
            return elem[s]; 
        }
        // ...
    private:
        owner<double*> elem;
        int sz;
    };
```
주석으로 표기된 이 클래스의 불변조건은 생성자에서 설정된다.
`new`는 요구받은 메모리를 할당하지 못할 경우 예외를 던진다. 
여기서 `operator []`는 불변조건에 의존하게 된다.

##### 같이 보기
[생성자가 올바른 객체를 생성하지 못한다면, 예외를 던져라](#Rc-throw)

##### 시행하기
생성자에서 설정하지 않는 `private` 상태에는 표시를 남겨라.



### <a name="Re-raii"></a>E.6: RAII를 사용해 누수를 방지하라

##### 근거
일반적으로 누수는 허용되어선 안된다.RAII("Resource Acquisition Is Initialization")는 누수를 방지하는 간단하고, 가장 체계적인 방법이다. 

##### 예

```c++
    void f1(int i)   // Bad: 누수가 발생할 수 있다
    {
        int* p = new int[12];
        // ...
        if (i < 17) throw Bad {"in f()", i};
        // ...
    }
```
예외를 발생시키기 전에 메모리를 해제할 수도 있다.

```c++
    void f2(int i)   // 명시적인 해제: 실수할 가능성이 있다 
    {
        int* p = new int[12];
        // ...
        if (i < 17) {
            delete[] p;
            throw Bad {"in f()", i};
        }
        // ...
    }
```
이런 코드는 번거롭다. 다수의 `throw`가 가능한 경우, 명시적인 해제는 반복적이고, 에러에 취약하다.

```c++
    void f3(int i)   // OK: p가 알아서 해제한다
    {
        auto p = make_unique<int[]>(12);
        // ...
        if (i < 17) throw Bad {"in f()", i};
        // ...
    }
```
RAII는 호출받은 함수에서 묵시적으로 `throw`하는 상황에서도 동작한다는 점에 주의하라:

```c++
    void f4(int i)   // OK: 어찌되었건 p가 알아서 해제한다
    {
        auto p = make_unique<int[]>(12);
        // ...
        helper(i);   // 예외를 던질 수도 있다
        // ...
    }
```
포인터가 반드시 필요하지 않다면, 지역 변수로 처리하라:

```c++
    void f5(int i)   // OK: 지역 객체에 의해서 자원이 관리된다
    {
        vector<int> v(12);
        // ...
        helper(i);   // 예외를 던질 수도 있다
        // ...
    }
```

##### 참고사항
자원을 해제해주는 리소스 핸들 객체가 없다면, 자원해제 동작들은 [`final_action` object](#Re-finally)을 사용해서 수행할 수도 있다.

##### 참고사항

그러나 예외를 사용할 수 없는 프로그램을 작성한다면 어떻게 해야 할까?

한번 그 생각에 도전해 보자; 예외를 반대하는 많은 의견이 있을 것이다.
그 중에 몇몇 타당한 의견들에 대해 알아 보자:


 * 예외를 적용하려면 메모리를 다 써버리는 아주 작은(2K보다 작은) 시스템 상에 있다.
 * 극한 실시간 시스템에서, 요구된 시간 안에 예외를 처리할 수 있는 툴을 가지고 있지 않다.
 * 난해한 방식으로 포인터를 사용하는 엄청난 양의 레거시 코드를 가진 시스템 상에 있다. (특히 소유권 관련된 정책이 보이지 않아서) 예외가 메모리 누수를 야기할 수 있다.
 * C++ 예외 메커니즘을 구현한 것이 적합하지 않다.(속도, 메모리 소비, DLL에서의 불분명한 처리실패 등) (예외 구현을 제공한 쪽에 항의하라; 불평이 없다면 개선도 없다.) 
 * 매니저가 가진 오래된 지식에 도전하면 해고당한다.  

> 역주: 극한 실시간 시스템(Hard-real-time System)

여기서 첫번째 이유만이 근본적인 경우라고 할 수 있다. 가능하다면, RAII를 구현하기 위해 예외를 사용하거나, RAII 객체들이 문제를 일으키지 않도록 설계하라. 

예외를 사용할 수 없다면, RAII를 흉내라도 내라. 즉, 체계적으로 객체가 생성된 후 유효한지 검사하고 소멸자에서 자원을 모두 해제하라. `valid()` 연산을 추가하는 것이 좋을 수도 있다.

```c++
    void f()
    {
        vector<string> vs(100);   // valid()가 추가된 구현
        if (!vs.valid()) {
            // 에러를 처리하거나 종료
        }

        ifstream fs("foo");   // valid()가 추가된 구현
        if (!fs.valid()) {
            // 에러를 처리하거나 종료
        }

        // ...
    } // 소멸자들이 평소처럼 자원을 해제한다
```

이런 방법은 코드의 길이가 늘어나지만, "exception"이 전파되는 것을 `valid()` 검사를 통해서 방지한다.
`valid()` 검사가 실수로 잊혀질 수 있으니 가능하다면 예외를 선호하라.

##### 같이 보기
[`noexcept`의 사용](#Se-noexcept).

##### 시행하기

???



### <a name="Re-precondition"></a>E.7: 선행조건을 기술하라

##### 근거
인터페이스 오류를 예방한다.

##### 같이 보기
[precondition rule](#Ri-pre).


### <a name="Re-postcondition"></a>E.8: 후행조건을 기술하라

##### 근거
인터페이스 오류를 예방한다.

##### 같이 보기
[postcondition rule](#Ri-post).




### <a name="Re-noexcept"></a>E.12: `throw`를 허용하지 않거나 불가능한 함수에는 `noexcept`를 사용하라

> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#Re-noexcept)


##### 근거
에러 처리를 체계적이고, 견고하며, 효율적으로 만든다.

##### 예

```c++
    double compute(double d) noexcept
    {
        return log(sqrt(d <= 0 ? 1 : d));
    }
```
이 코드에서는 `compute` 함수가 예외를 던지지 않도록 작성되었단 사실을 알수 있다. 
`noexcept`로 선언함으로써, 우리는 컴파일러와 다른 이들에게 `compute`함수를 이해하고 변경하는데 도움을 줄 수 있다.

##### 참고사항
C 표준 라이브러리에서 "승계된" 많은 C++ 표준 라이브러리 함수들은 `noexcept`로 정의되어있다.

##### 예

```c++
    vector<double> munge(const vector<double>& v) noexcept
    {
        vector<double> v2(v.size());
        // ... do something ...
    }
```
이 코드에서 `noexcept`는 지역변수 `vector`의 생성이 실패하는 상황을 처리하지 않는다고 표현하고 있다. 이 말인 즉, 메모리 고갈을 (하드웨어 실패와 같은)심각한 설계 오류로 간주한다는 것이다. 따라서 예외가 발생한다면 프로그램에서 크래시를 일으킬 것이다.

##### 같이 보기
[discussion](#Sd-noexcept).



### <a name="Re-never-throw"></a>E.13: 객체를 소유하는 중에는 예외를 발생시켜선 안된다

##### 근거
자원 누수를 발생시킬 수 있다.

##### 예

```c++
    void leak(int x)   // 좋지 않다: 누수가 발생할 수 있다
    {
        auto p = new int{7};
        if (x < 0) 
            throw Get_me_out_of_here{};  // *p에서 누수가 발생

        // ...
        delete p;   // 예외가 발생하면 해제하지 못한다
    }
```
이런 문제를 해결하기 위해서 일관적으로 자원 핸들(resource handle)을 사용할 수도 있다.

```c++
    void no_leak(int x)
    {
        auto p = make_unique<int>(7);
        if (x < 0) 
            throw Get_me_out_of_here{};  // 필요하다면 delete가 수행된다

        // ...
        // p가 알아서 자원을 해제한다
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

##### 같이 보기

??? 자원(resource)과 관련된 규칙들 ???




### <a name="Re-exception-types"></a>E.14: 목적에 맞게 설계된 사용자 정의 타입을 예외로 사용하라 (내장 타입은 안된다)

##### 근거
프로그래머가 정의한 타입들은 다른 사용자들의 예외와 충돌할 가능성이 적다.

##### 예
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
        catch(Bufferpool_exhausted) {
            // ...
        }
    }
```

##### 잘못된 예
```c++
    void my_code()     // Don't try this code at your computer
    {
        // ...
        throw 7;       // 이 7은 "moon in the 4th quarter"을 의미한다
        // ...
    }

    void your_code()   // 이런 코드를 작성하지 마시오!
    {
        try {
            // ...
            my_code();
            // ...
        }
        catch(int i) {  // 여기선 7이 "입력 버퍼가 부족함"을 의미한다
            // ...
        }
    }
```
##### 참고사항
표준 라이브러리의 예외 클래스들은 `std::exception`을 상속하고 있다. 이들에 대한 에러 처리는 `std::exception` 또는 "일반적인" 처리를 사용하여야 한다. 내장 타입들을 예외 객체로 사용하는 것은 다른 사용자들의 코드와 충돌을 발생시킬 수 있다.

> 역주 : 일반적인(generic) 처리. `catch(...)`를 의미하는 것으로 추정됨


##### 잘못된 예
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
        catch(runtime_error&) {   
            // 여기서 runtime_error는 "입력 버퍼가 부족함"을 의미한다
            // ...
        }
    }
```

##### 같이 보기

[Discussion](#Sd-???)

##### 시행하기
내장 타입을 사용하는 `throw`와 `catch`를 잡아내라. 표준 라이브러리에서 사용하는 `std::exception`기반의 계층구조에 포함된 예외 클래스를 사용하는 것이 더 낫다.




### <a name="Re-exception-ref"></a>E.15: 계층 구조가 있는 예외는 참조를 사용해서 처리하라

##### 근거
예외 객체의 복사 손실을 방지한다.

##### 예
```c++
    void f()
    try {
        // ...
    }
    catch (exception e) {   // don't: 복사 손실이 발생할 수 있다
        // ...
    }
```
Instead, use:

```c++
    catch (exception& e) { /* ... */ }
```
##### 시행하기
값으로 예외를 처리하는 경우 표시를 남겨라.(완벽히 처리하기 위해선 전체 프로그램에 대한 분석이 필요할 수 있다.)




### <a name="Re-never-fail"></a>E.16: 소멸자, 자원해제, `swap`은 절대 실패해선 안된다

##### 근거
소멸자, 메모리 해제, `swap`이 실패한다면 신뢰할 수 있는 프로그램을 작성할 수 없다; that is, if it exits by an exception or simply doesn't perform its required action.

##### 잘못된 예

```c++
    class Connection 
    {
        // ...
    public:
        ~Connection()   // Don't: 아주 나쁜 소멸자!
        {
            if (cannot_disconnect()) 
                throw I_give_up{information};
            // ...
        }
    };
```

##### 참고사항
예시에 나온 "close를 거부하는" 네트워크 연결 클래스처럼, 많은 이들이 이 규칙을 위반하면서 신뢰할 수 있는 코드를 작성하려고 시도해왔다. 
우리가 아는 선에서 누구도 이를 일반화하지 못했다.

때때로, 아주 예외적인 경우, 나중에 정리작업을 하기 위핸 상태를 설정하는 것이 가능할 것이다.
예컨대, close하길 원하지 않는 소켓은 "문제있는 소켓"목록에 추가하고 정기적으로 sweep할수도 있을 것이다.

여태껏 이 규칙을 위반하는 사례들은 에러에 취약하거나, 특별하거나, 때로는 버그투성이였다.


##### 참고사항
표준 라이브러리는 소멸자, (`operator delete`같은) 자원 해제 함수, `swap`함수가 예외를 던지지 않는다고 가정한다. 예외를 던진다면, 표준 라이브러리들의 불변조건이 충족되지 않을 것이다.

##### 참고사항
 * `operator delete`와 같은 해제 함수는 반드시 `noexcept`여야 한다.
 * `swap`함수는 반드시 `noexcept`여야 한다.
 * 소멸자들은 표기하지 않아도 기본적으로 `noexcept`이다.

또한, [move 연산도 `noexcept`로 만들어라](##Rc-move-noexcept).


##### 시행하기

예외를 던지는 소멸자, 자원해제 함수, `swap`을 잡아내라. 비슷한 역할을 하는 함수들이 `noexcept`가 아닌 경우에도 잡아내라. 

##### 같이 보기
[discussion](#Sd-never-fail)




### <a name="Re-not-always"></a>E.17: 각각의 함수들에서 모든 예외를 처리하려고 하지 마라

> 역주 : 각각의 함수들(every function)


##### 근거

함수 내에서 예외를 catch하는 것은 유의미한 복구 작업을 하기 어려우며, 복잡성을 높이고 낭비가 될 수 있다.
예외를 던질 때는 그 예외를 처리할 수 있는 함수로 전파되도록 남겨두어라. 이때 정리작업와 호출스택 되감기는 [RAII](#Re-raii)에게 맡겨라.

##### 잘못된 예

```c++
    void f()   // bad
    {
        try {
            // ...
        }
        catch (...) {
            // 아무런 처리를 하지 않는다.
            throw;   // 예외 전파
        }
    }
```
##### 시행하기
 * 중첩된 `try` 블럭들에 표시를 남겨라.
 * 함수에 지나치게 빈번한 `try`블럭들을 사용한 소스 코드에는 표시를 남겨라 ( "지나치게 빈번한"을 정의할 수 있을까요???)




### <a name="Re-catch"></a>E.18: 명시적인 `try`/`catch`문의 사용을 최소화하라

##### 근거
`try`/`catch`는 번거롭고 에러를 발생시키기도 쉽다.
`try`/`catch`는 저레벨 자원 관리 또는 에러 처리가 체계적이지 않다는 신호일 수 있다. 


##### 잘못된 예

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
            delete p;   // 생성에 실패한 경우라면?
            throw;
        }
    }
```
지저분한 코드다. `try` 블록 내에 있는 포인터 `p`에서 누수가 발상할 수 있다.
모든 예외를 처리하지 않는다. 온전히 생성되지 않은 객체를 `delete`하는 것은 잘못된 사용법이다.

더 나은 생성방법:
```
    void f2(zstring s)
    {
        Gadget g {s};
    }
```

##### 대안
 * 리소스 핸들과 [RAII](#Re-raii)를 사용하라
 * [`finally`](#Re-finally)

##### 시행하기

??? 어렵다. 경험적인(Heuristic) 접근이 필요하다.




### <a name="Re-finally"></a>E.19: 적당한 리소스 핸들을 사용할 수 없다면 처리방법을 나타낼 `final_action` 객체를 사용하라

##### 근거
`finally`를 사용하는 것이 덜 번거롭고, `try`/`catch`에 비해 잘못 사용하기 어렵다.

##### 예

```c++
    void f(int n)
    {
        void* p = malloc(1, n);

        auto _ = finally([p] { free(p); });
        // ...
    }
```
##### 참고사항

`finally`는 `try`/`catch`만큼 지저분하지는 않지만, 임시적인(ad-hoc) 방법이다. [적합한 리소스 관리 객체](#Re-raii)를 사용하라.

##### 참고사항
[`goto exit;` 방법](##Re-no-throw-codes)과 비교하면 `finally`는 언어 시스템을 사용하는 체계적이고 깔끔한 대안이다. 

##### 시행하기

경험적이다 : `goto exit;`이 있는지 확인해보라




### <a name="Re-no-throw-raii"></a>E.25: 예외를 던질 수 없는 경우, RAII로 자원을 관리하라

> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#e25-if-you-cant-throw-exceptions-simulate-raii-for-resource-management)

##### 근거
예외가 없다고 하더라도, [RAII](#Re-raii)는 보통 자원을 다루는 최고의, 가장 체계적인 방법이다.

##### 참고사항
C++에서 예외를 이용한 에러처리는 비-지역적 에러를 다루는 방법이다. 특히 객체 생성에서 비-침습적으로 생성 실패를 알리는 데는 예외가 필요하다. 

에러의 발생을 알리는(signaling) 것이 무시되지 못하게 하기 위해선 예외가 필요하다 

예외를 사용할 수 없다면, 당신이 할 수 있는 한 그와 유사한 방법을 사용하라.

예외에 대한 많은 우려들은 잘못된 것이다.

포인터나 복잡한 제어 구조들로 인해 난잡하지 않은 환경에서 사용될때는 예외는 거의 언제나 (시간과 공간적인 측면에서) 신뢰할 수 있는 방법이고, 더 나은 코드로 이어진다. 이는, 당연하게도, 예외 처리 메커니즘이 잘 구현된 것을 전제한다. 그리고 이는 모든 시스템에서 해당하진 않는다.

위와 같은 문제를 적용할 수 없는 경우도 있다. 일부 Hard-real-time 시스템들이 그런 경우인데, 정해진 시간 안에 에러나 정확한 답을 산출해야만 하는 연산들 때문이다. 시간을 측정할 수 있는 적합한 도구가 없다면, 예외가 이를 충족시키기는 어렵다. 그런 시스템(가령, 비행기 조종 소프트웨어)에서는 일반적으로 동적(heap) 메모리 사용 또한 배제되기도 한다.

결국, 에러 처리에 대한 최우선 가이드는 이렇다. "예외와 [RAII](#Re-raii)를 사용하라".
이 부분에서는 예외의 구현이나 예외가 효율적이지 않은 경우, 오래된 코드의 처리 스타일등 예외 처리를 사용할 수 없는 상황도 고려한다. 
(가령, 엄청난 수의 포인터, 잘못 정의된 소유권, 에러코드 검사에 기반한 체계적이지 않은 에러처리들 등등)

예외와 그 비용에 대해 비난하기 전에, [error codes](#Re-no-throw-codes)에 있는 사례들을 고려해보라. 비용과 에러 코드를 다룰때의 복잡함을 모두 고려하라. 

성능이 걱정된다면, 측정하라.

##### 예

이런 코드를 작성하고 싶다고 하자.
```c++

    void func(zstring arg)
    {
        Gadget g {arg};
        // ...
    }
```
`Gadget`이 제대로 생성되지 않았다면, `func`는 예외를 던지며 종료한다. 예외를 던질 수 없다면, `Gadget`에 `valid()`멤버 함수를 추가함으로써 RAII처럼 자원을 처리할 수도 있다.

```c++
    error_indicator func(zstring arg)
    {
        Gadget g {arg};
        if (!g.valid()) 
            return gadget_construction_error;
        // ...
        return 0;   // zero indicates "good"
    }
```
이 경우 호출자(caller)는 당연히 반환값 검사가 필요하다는 것을 알고 있어야 한다.

##### 같이 보기
[Discussion](#Sd-???).

##### 시행하기
이 아이디어의 특별한 경우: 예컨대, 리소스 핸들의 생성을 `valid()`로 시스템적으로 검사하는 등



### <a name="Re-no-throw-crash"></a>E.26: 예외를 던질 수 없는 경우, 빠른 실패 전략을 고려하라

> 역주 : 빠른 실패(failing fast)

##### 근거
에러로부터 복원하는 작업을 제대로 수행할 수 없다면, 최소한 문제가 더 커지기 전에 탈출할 수는 있다.

[RAII처럼 자원을 관리하라](#Re-no-throw-raii) 항목을 함께 보라.

##### 참고사항
지역적인 에러 처리를 제대로 할 수 없다면, "크래시 발생"을 사용하는 것을 고려해보라. 이는 에러를 확인한 문맥에서 복구하거나 지역적으로 처리할 수 없는 경우를 의미한다. `abort()`나 `quick_exit()`를 호출하라. 또는 시스템이 재시작하도록 하는 비슷한 기능의 함수를 호출하라.

다수의 프로세스나 컴퓨터들을 사용하는 시스템에서는 어찌되었건 치명적인 크래시를 처리하는 경우를 예상할 필요가 있다. 하드웨어에서 문제가 생길 수도 있다. 이런 경우, "크래시 발생"은 단순히 에러 처리를 시스템의 다음 레벨로 전달하는 것이 된다.

##### 예

```c++
    void f(int n)
    {
        // ...
        p = static_cast<X*>(malloc(n, X));
        if (p == nullptr) 
            abort();     // 메모리가 고갈되면 종료한다
        // ...
    }
```
대부분의 프로그램은 메모리 고갈을 아름답게 해결하기 어렵다. 이는 다음 코드에서도 동일하다.

```c++
    void f(int n)
    {
        // ...
        // 메모리가 고갈하면 예외를 던진다. 
        // 처리되지 않을 경우 기본적으로 종료한다.
        p = new X[n];    
        // ...
    }
```
일반적으로, "크래시"가 발생해서 종료하기 전 발생 원인에 대해 로그를 남기는 것은 현명한 생각이다.

##### 시행하기

Awkward



### <a name="Re-no-throw-codes"></a>E.27: 예외를 던질 수 없는 경우, 체계적으로 에러 코드를 사용하라

> 역주 : 체계적으로(systematically)

##### 근거

에러 핸들링 정책을 체계적으로 사용하는 것은 에러 처리를 잊어버릴 가능성을 최소화한다.

[RAII처럼 자원을 관리하라](#Re-no-throw-raii) 항목을 함께 보라.

##### 참고사항
몇가지 이슈를 다뤄보자:
 * 함수 바깥에서 에러 알림을 어떻게 전달할 것인가? 
 * 에러로 종료하기 이전에 함수에서의 자원 해제는 어떻게 처리하는가?
 * 에러의 지표로서 무엇을 사용하는가?

> 역주 : 에러 지표, 에러 알림(error indicator)

일반적으로, 에러 지표를 반환하는 것은 2가지 값을 반환한다는 것을 의미한다: 정상적인 결과와 에러 지표.

에러 지표는 `valid()` 함수를 가진 경우처럼 객체의 일부가 될수도 있다. 또는 함수가 한번에 2개의 값들을 반환할수도 있다.

##### 예

```c++
    Gadget make_gadget(int n)
    {
        // ...
    }

    void user()
    {
        Gadget g = make_gadget(17);
        if (!g.valid()) {
                // 에러 처리
        }
        // ...
    }
```
이 접근법은 [RAII처럼 자원을 관리하라](#Re-no-throw-raii) 항목에 부합한다.
`valid()`함수는 `error_indicator`를 반환할수도 있다. (예컨대, `error_indicator` 열거형 중에서 하나의 값을 반환한다).

##### 예
`Gadget`타입을 변경하길 원하지 않거나, 변경할 수 없는 경우는 어떨까? 그런 경우엔, 한 쌍의 값을 반환할 수 밖에 없다.

예를 들면:

```c++
    // 값 쌍을 반환한다
    std::pair<Gadget, error_indicator> make_gadget(int n)
    {
        // ...
    }

    void user()
    {
        auto r = make_gadget(17);
        if (!r.second) {
                // 에러 처리
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
                // 에러 처리
        }
        Gadget& g = r.val;
        // ...
    }
```

특별한 타입을 선호하는 이유중의 하나로는 `first`나 `second`같은 비밀스러운 이름보다는 멤버처럼 사용할 수 있기 때문이다. 이런 방법은 `std::pair`의 다른 사용법과 혼동되는 것을 방지한다.

##### 예
일반적으로, 에러로 인해 종료하기 전 정리 작업을 하게 된다. 이런 코드는 지저분하다:

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
RAII처럼 동작하도록 하는 것은 함수 내에서 다수의 자원과 다양한 에러가 발생할 수 있을 경우 더욱 중요하다. 이에 대한 해법 중 하나는 반복을 피하기 위해 함수의 끝부분에 정리 작업을 모아놓는 것이다:

```c++
    std::pair<int, error_indicator> user()
    {
        error_indicator err = 0;

        Gadget g1 = make_gadget(17);
        if (!g1.valid()) {
                err = g1_error;
                goto exit;  // 정리 작업으로 보낸다
        }

        Gadget g2 = make_gadget(17);
        if (!g2.valid()) {
                err = g2_error;
                goto exit;  // 정리 작업으로 보낸다
        }

        if (all_foobar(g1, g2)) {
            err = foobar_error;
            goto exit;  // 정리 작업으로 보낸다
        }
        // ...

    // 모아놓은 cleanup 코드
    exit:
      if (g1.valid()) cleanup(g1);
      if (g2.valid()) cleanup(g2);
      return {res, err};
    }
```
함수가 꽤 크다면, 이런 방법을 사용하게 될 가능성이 크다. `finally`를 사용하는 것이 [좀 더 쉬울 수 있다](#Re-finally). 또한, 프로그램이 커질수록 체계적으로 **에러 지표를 사용한 에러 처리 전략**을 체계적으로 적용하기 어려울 수 있다.

우리는 [예외 기반 에러 처리](#Re-throw)를 선호하며, [함수는 짧게](#Rf-single)작성하기를 권한다.

##### 같이 보기
 - [Discussion](#Sd-???).
 - [다수의 값을 반환하기](#Rf-out-multi).
 
##### 시행하기

Awkward.



### <a name="Re-no-throw"></a>E.28: 전역 상태를 사용한 에러 처리를 지양하라(`errno`처럼)

##### 근거
전역 상태는 관리하기 어렵고, 확인하는 것을 잊어버리기 쉽다. 마지막으로 `printf()`의 반환값을 확인한 것이 언제인가?

[RAII처럼 자원을 관리하라](#Re-no-throw-raii) 항목을 참고하라

##### 잘못된 예

```
    ???
```

##### 참고사항
C 스타일 에러처리는 전역 변수 `errno`에 의존한다, 때문에 이 가이드를 따르는 것이 필연적으로 불가할수도 있다.

##### 시행하기

Awkward.

