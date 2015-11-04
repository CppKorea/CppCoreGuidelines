# <a name="S-errors"></a> E: Error handling

에러처리는 다음과 관련이 있다:
>Error handling involves:

* 에러 찾아내기
* 에러에 대한 정보를 처리코드로 전송하기
* 프로그램을 올바른 상태로 유지시키기
* 리소스 누수를 피하기

>* Detecting an error
>* Transmitting information about an error to some handler code
>* Preserve the state of a program in a valid state
>* Avoid resource leaks

모든 에러를 복구하기란 불가능하다. 에러 복구가 여의치 않다면 정의된 방식으로 빨리 "빠져나가는" 것이 중요하다.
에러 처리 방법은 간단해야 한다. 그렇지 않으면 훨씬 더 안좋은 에러의 시작이 될 것이다.
>It is not possible to recover from all errors. If recovery from an error is not possible, it is important to quickly "get out" in a well-defined way. A strategy for error handling must be simple, or it becomes a source of even worse errors.

규칙들은 여러 종류의 에러를 피하기 위해 만들어졌다:
>The rules are designed to help avoid several kinds of errors:

* 타입 위반 (`union`의 오용과 형변환)
* 리소스 누수 (메모리 누수도 포함)
* 범위를 벗어나는 에러
* 변수 수명 에러 (`delete`된 객체 접근하기)
* 복잡성 에러 (아이디어를 복잡한 표현으로 만드는 논리적 에러)
* 인터페이스 에러 (기대하지 않은 값이 인터페이스로 넘어온다.)

>* Type violations (e.g., misuse of `union`s and casts)
>* Resource leaks (including memory leaks)
>* Bounds errors
>* Lifetime errors (e.g., accessing an object after is has been `delete`d)
>* Complexity errors (logical errors make likely by overly complex expression of ideas)
>* Interface errors (e.g., an unexpected value is passed through an interface)

에러 처리 규칙 요약:
>Error-handling rule summary:

* [E.1: 디자인 초기에 에러처리 전략을 수립하라.](#Re-design)
* [E.2: 함수가 맡은 작업을 처리할 수 없다는 것을 알리기 위해 예외를 발생시켜라.](#Re-throw)
* [E.3: 에러처리를 위해서만 예외를 사용하라.](#Re-errors)
* [E.4: 객체조건에 대한 에러처리 방법을 디자인하라.](#Re-design-invariants)
* [E.5: 생성자가 객체조건을 구성하게 하라. 할 수 없으면 에러를 발생시켜라.](#Re-invariant)
* [E.6: 메모리 누수를 막으려면 RAII를 사용하라.](#Re-raii)
* [E.7: 선행조건을 기술하라.](#Re-precondition)
* [E.8: 후행조건을 기술하라.](#Re-postcondition)

* [E.12: `throw`를 허용 안하거나 불가능할 때 `noexcept`문을 사용하라.](#Re-noexcept)
* [E.13: 객체의 직접 주인인 동안에는 예외를 발생시키지 마라.](#Re-never-throw)
* [E.14: 목적에 맞는 사용자 정의 타입을 예외로 사용하라. (내장 타입은 안됨.)](#Re-exception-types)
* [E.15: 참조형을 사용해서 상속받은 예외를 처리하라.](#Re-exception-ref)
* [E.16: 소멸자, 할당해제, 스왑은 절대 실패해선 안된다.](#Re-never-fail)
* [E.17: 모든 함수에서 발생하는 모든 예외를 처리하려고 하지 마라.](#Re-not-always)
* [E.18: 명시적인 `try`/`catch`문의 사용을 최소화하라.](#Re-catch)
* [E.19: 적당한 리소스 핸들을 사용할 수 없다면 처리방법을 나타낼 `final_action` 객체를 사용하라.](#Re-finally)

* [E.25: ??? 예외를 던질 수 없는 프로그램에서는 무엇을 할 것인가.](#Re-no-throw)
* ???

>* [E.1: Develop an error-handling strategy early in a design](#Re-design)
>* [E.2: Throw an exception to signal that a function can't perform its assigned task](#Re-throw)
>* [E.3: Use exceptions for error handling only](#Re-errors)
>* [E.4: Design your error-handling strategy around invariants](#Re-design-invariants)
>* [E.5: Let a constructor establish an invariant, and throw if it cannot](#Re-invariant)
>* [E.6: Use RAII to prevent leaks](#Re-raii)
>* [E.7: State your preconditions](#Re-precondition)
>* [E.8: State your postconditions](#Re-postcondition)

>* [E.12: Use `noexcept` when exiting a function because of a `throw` is impossible or unacceptable](#Re-noexcept)
>* [E.13: Never throw while being the direct owner of an object](#Re-never-throw)
>* [E.14: Use purpose-designed user-defined types as exceptions (not built-in types)](#Re-exception-types)
>* [E.15: Catch exceptions from a hierarchy by reference](#Re-exception-ref)
>* [E.16: Destructors, deallocation, and `swap` must never fail](#Re-never-fail)
>* [E.17: Don't try to catch every exception in every function](#Re-not-always)
>* [E.18: Minimize the use of explicit `try`/`catch`](#Re-catch)
>* [E.19: Use a `final_action` object to express cleanup if no suitable resource handle is available](#Re-finally)

>* [E.25: ??? What to do in programs where exceptions cannot be thrown](#Re-no-throw)
>* ???

### <a name="Re-design"></a> E.1: 디자인 초기에 에러처리 전략을 수립하라.
>### <a name="Re-design"></a> E.1: Develop an error-handling strategy early in a design

##### Reason

에러와 리소스 누수를 처리하기 위한 일관성있고 완벽한 전략은 기존 시스템에 새로 추가해 넣기가 아주 어렵다.
>A consistent and complete strategy for handling errors and resource leaks is hard to retrofit into a system.

### <a name="Re-throw"></a> E.2: 함수가 맡은 작업을 처리할 수 없다는 것을 알리기 위해 예외를 발생시켜라.
>### <a name="Re-throw"></a> E.2: Throw an exception to signal that a function can't perform its assigned task

##### Reason

에러 처리를 시스템적으로, 튼튼하게, 비반복적으로 하기 위해서.
>To make error handling systematic, robust, and non-repetitive.

##### Example

    struct Foo {
        vector<Thing> v;
        File_handle f;
        string s;
    };

    void use()
    {
        Foo bar { {Thing{1}, Thing{2}, Thing{monkey}}, {"my_file", "r"}, "Here we go!"};
        // ...
    }

여기 `vector`, `string`의 생성자는 충분한 메모리를 할당받을 수 없을지도 모른다. `vector`의 생성자는 `Thing`을 초기화 리스트에 복사할 수 없을지도 모른다.
`File_handle`은 파일을 오픈하지 못할지도 모른다.
이런 경우에 `use()`의 호출자가 처리할 수 있도록 예외를 발생시킨다. `use()`가 직접 `bar` 생성 실패를 처리하려고 하면 `try`/`catch`를 사용하면 된다.
이런 에러일 때 `Foo`를 생성하려고 시도한 것들에 제어권을 넘기기 전에, 이미 생성된 멤버에 대해서 `Foo` 생성자가 알아서 잘 소멸시킨다. (?? - 어렵다)
에러 코드를 포함하는 반환값이 없다는데 주목하라.
>Here, `vector` and `string`s constructors may not be able to allocate sufficient memory for their elements, `vector`s constructor may not be able copy the `Thing`s in its initializer list, and `File_handle` may not be able to open the required file.
In each case, they throw an exception for `use()`'s caller to handle.
If `use()` could handle the failure to construct `bar` it can take control using `try`/`catch`.
In either case, `Foo`'s constructor correctly destroys constructed members before passing control to whatever tried to create a `Foo`.
Note that there is no return value that could contain an error code.

`File_handle` 생성자를 아래처럼 정의했을 것이다:
>The `File_handle` constructor might defined like this:

    File_handle::File_handle(const string& name, const string& mode)
        :f{fopen(name.c_str(), mode.c_str())}
    {
        if (!f)
            throw runtime_error{"File_handle: could not open "S-+ name + " as " + mode"}
    }

##### Note

예외는 예외적인 이벤트나 실패를 알려주려는 의도라고들 한다.
그러나 무엇이 예외적이냐는 것 때문에 다소 순환적인 의미를 가진다.
예를 들면:
>It is often said that exceptions are meant to signal exceptional events and failures.
However, that's a bit circular because "what is exceptional?"
Examples:

* 만날 수 없는 선행조건
* 객체를 생성할 수 없는 생성자 (클래스의 [invariant](#Rc-struct)를 구성하는데 실패)
* 범위를 벗어나는 에러 (`v[v.size()] =7`)
* 리소스 획득에 실패 (네트웍 다운)

>* A precondition that cannot be met
>* A constructor that cannot construct an object (failure to establish its class's [invariant](#Rc-struct))
>* An out-of-range error (e.g., `v[v.size()] =7`)
>* Inability to acquire a resource (e.g., the network is down)

반대로 일반적인 루프의 종료는 예외적이 아니다.
루프가 무한하지 않다면 루프의 종료는 일반적이고 당연히 그래야 한다.
>In contrast, termination of an ordinary loop is not exceptional.
Unless the loop was meant to be infinite, termination is normal and expected.

##### Note

함수에서 값을 반환하는 또다른 방법으로 `throw`를 사용하지 마라.
>Don't use a `throw` as simply an alternative way of returning a value from a function.

**Exception**: 진짜 실시간 시스템은 실행 전에 이미 정해진 상수최대시간(일반적으로 짧은 시간)내에 작업이 완료됨을 보장해야 한다.
그런류의 시스템은 `throw`로부터 복구하기 위한 최대시간을 정확히 예측할 수 있는 툴이 있는 경우에만 예외를 사용할 수 있다.
>**Exception**: Some systems, such as hard-real time systems require a guarantee that an action is taken in a (typically short) constant maximum time known before execution starts. Such systems can use exceptions only if there is tool support for accurately predicting the maximum time to recover from a `throw`.

**See also**: [RAII](#Re-raii)

**See also**: [discussion](#Sd-noexcept)

### <a name="Re-errors"></a> E.3: 에러처리를 위해서만 예외를 사용하라.
>### <a name="Re-errors"></a> E.3: Use exceptions for error handling only

##### Reason

"정상적인 코드"와 분리해서 에러처리를 하기 위해서.
C++에서는 예외가 드물다는 가정에 기본해서 최적화를 하는 경향이 좀 있다.
>To keep error handling separated from "ordinary code."
C++ implementations tend to be optimized based on the assumption that exceptions are rare.

##### Example, don't

    int find_index(vector<string>& vec, const string& x)	// don't: exception not used for error handling
    {
        try {
            for (int i =0; i < vec.size(); ++i)
                if (vec[i] == x) throw i;  // found x
        } catch (int i) {
            return i;
        }
        return -1;	// not found
    }

위는 더 복잡하다. 다른 방법보다 더 느리게 동작할 것 같다.
`vector`에서 특정값을 찾는 것에 예외적인 상황은 없다.
>This is more complicated and most likely runs much slower than the obvious alternative.
There is nothing exceptional about finding a value in a `vector`.

### <a name="Re-design-invariants"></a> E.4: 객체조건에 대한 에러처리 방법을 디자인하라.
>### <a name="Re-design-invariants"></a> E.4: Design your error-handling strategy around invariants

##### Reason

객체를 사용하기 위해 올바른 상태에 있어야 한다(공식, 비공식적으로 객체조건을 만족해야).
그리고 에러복구를 위해 삭제되지 않은 모든 객체는 올바른 상태여야 한다.
>To use an objects it must be in a valid state (defined formally or informally by an invariant) and to recover from an error every object not destroyed must be in a valid state.

##### Note

[invariant](#Rc-struct)는 객체 멤버를 위한 논리적 조건이다. 생성자는 공용 멤버함수가 작동하도록 객체를 구성해야 한다.
(?? - 이것도 넘 어렵다.)
>An [invariant](#Rc-struct) is logical condition for the members of an object that a constructor must establish for the public member functions to assume.

### <a name="Re-invariant"></a> E.5: 생성자가 객체조건을 구성하게 하라. 할 수 없으면 에러를 발생시켜라.
>### <a name="Re-invariant"></a> E.5: Let a constructor establish an invariant, and throw if it cannot

##### Reason

조건을 만족하지 않는 객체를 구성하면 문제가 발생한다. 모든 멤버 함수를 호출할 수 없다.
>Leaving an object without its invariant established is asking for trouble.
Not all member function can be called.

##### Example

    ???

**See also**: [If a constructor cannot construct a valid object, throw an exception](#Rc-throw)

##### Enforcement

???

### <a name="Re-raii"></a> E.6: 메모리 누수를 막으려면 RAII를 사용하라.
>### <a name="Re-raii"></a> E.6: Use RAII to prevent leaks

##### Reason

새는 메모리는 받아 들일 수 없다.
메모리 누수를 막기 위해서는 RAII("Resource Acquisition is Initialization")방법이 가장 쉽고 시스템적이다.
>Leaks are typically unacceptable. RAII ("Resource Acquisition Is Initialization") is the simplest, most systematic way of preventing leaks.

##### Example

    void f1(int i)	// Bad: possibly leak
    {
        int* p = new int[12];
        // ...
        if (i < 17) throw Bad {"in f()", i};
        // ...
    }

예외를 발생시키기 전에 메모리를 해제할 수 있다.
>We could carefully release the resource before the throw:

    void f2(int i)	// Clumsy: explicit release
    {
        int* p = new int[12];
        // ...
        if (i < 17) {
            delete p;
            throw Bad {"in f()", i};
        }
        // ...
    }

구문이 길어진다. 코드가 커지면 `throw`마다 해제하기가 반복적인 작업이라 에러 발생이 쉬워진다.
>This is verbose. In larger code with multiple possible `throw`s explicit releases become repetitive and error-prone.

    void f3(int i)	// OK: resource management done by a handle
    {
        auto p = make_unique<int[12]>();
        // ...
        if (i < 17) throw Bad {"in f()", i};
        // ...
    }

호출하는 함수내에서 발생하는 `throw`에 대해서도 잘 동작한다.
>Note that this works even when the `throw` is implicit because it happened in a called function:

    void f4(int i)	// OK: resource management done by a handle
    {
        auto p = make_unique<int[12]>();
        // ...
        helper(i);	// may throw
        // ...
    }

포인터까지 필요없다면 지역변수를 사용해라:
>Unless you really need pointer semantics, use a local resource object:

    void f5(int i)	// OK: resource management done by local object
    {
        vector<int> v(12);
        // ...
        helper(i);	// may throw
        // ...
    }

##### Note

리소스 핸들이 아니라면 해제 작업은 [`final_action` object](#Re-finally)로 표현할 수도 있다.
>If there is no obvious resource handle, cleanup actions can be represented by a [`final_action` object](#Re-finally)

##### Note

그러나 예외를 사용할 수 없는 프로그램을 작성한다면 어떻게 해야 할까?
한번 그 생각에 도전해 보자; 예외를 반대하는 많은 의견이 있을 것이다.
그 중에 몇몇에 대해 알아 본다:
>But what do we do if we are writing a program where exceptions cannot be used?
First challenge that assumption; there are many anti-exceptions myths around.
We know of only a few good reasons:

* 예외를 적용하려면 메모리를 다 써버리는 아주 작은 시스템 상에 있다.
* 진짜 실시간 시스템이 있어서, 요구된 시간 안에 예외를 처리할 수 있는 툴을 가지고 있지 않다.
* 난해한 방식으로 포인터를 사용하는 몇 톤의 과거 코드를 가진 시스템 상에 있다.
(특히 소유권 관련된 정책이 보이지 않을 때) 예외가 메모리 누수를 야기할 수 있다.
* 매니저가 가진 오래된 지식에 도전하면 해고당한다.
>* We are on a system so small that the exception support would eat up most of our 2K or memory.
>* We are in a hard-real-time system and we don't have tools that allows us that an exception is handled within the required time.
>* We are in a system with tons of legacy code using lots of pointers in difficult-to-understand ways
  (in particular without a recognizable ownership strategy) so that exceptions could cause leaks.
>* We get fired if we challenge our manager's ancient wisdom.

이런 이유의 첫번째는 기본적이다. 가능하면 언제든지 RAII를 구현하도록 예외를 사용한다.
예외를 사용할 수 없다면 RAII를 흉내내라.
즉,  생성한 후와 소멸자에서 리소스 해제한 후에도 객체가 올바른지 시스템적으로 체크한다.
한가지 방법은 모든 리소스 핸들에 대해서 `valid()` 연산을 추가하는 것이다.
>Only the first of these reasons is fundamental, so whenever possible, use exception to implement RAII.
When exceptions cannot be used, simulate RAII.
That is, systematically check that objects are valid after construction and still release all resources in the destructor.
One strategy is to add a `valid()` operation to every resource handle:

    void f()
    {
        vector<string> vs(100);	// not std::vector: valid() added
        if (!vs.valid()) {
            // handle error or exit
        }

        Ifstream fs("foo");		// not std::ifstream: valid() added
        if (!fs.valid()) {
            // handle error or exit
        }

        // ...
    } // destructors clean up as usual

딱 보면 코드 길이가 늘어나고 `exceptions`을 암시적으로 전파하는 것을 허용하지 않는다.
그리고 `valid()` 체크를 까먹을 수도 있다.
예외를 써라.
>Obviously, this increases the size of the code, doesn't allow for implicit propagation of "exceptions" (`valid()` checks), and `valid()` checks can be forgotten.
Prefer to use exceptions.

**See also**: [discussion](#Sd-noexcept).

##### Enforcement

???

### <a name="Re-precondition"></a> E.7: 선행조건을 기술하라.
>### <a name="Re-precondition"></a> E.7: State your preconditions

##### Reason

인터페이스 오류를 줄이기 위해서.
>To avoid interface errors.

**See also**: [precondition rule](#Ri-pre).

### <a name="Re-postcondition"></a> E.8: 후행조건을 기술하라.
>### <a name="Re-postcondition"></a> E.8: State your postconditions

##### Reason

인터페이스 오류를 줄이기 위해서.
>To avoid interface errors.

**See also**: [postcondition rule](#Ri-post).

### <a name="Re-noexcept"></a> E.12: `throw`를 허용 안하거나 불가능할 때 `noexcept`문을 사용하라.
>### <a name="Re-noexcept"></a> E.12: Use `noexcept` when exiting a function because of a `throw` is impossible or unacceptable

##### Reason

시스템적으로, 튼튼하게, 효율적으로 에러를 관리하도록 만들기 위해서.
>To make error handling systematic, robust, and efficient.

##### Example

    double compute(double d) noexcept
    {
        return log(sqrt(d <= 0 ? 1 : d));
    }

`compute`는 예외가 없는 연산을 사용해서 예외를 발생시키지 않는다는 것을 알 수 있다.
`noexcept`로 정의함으로써 컴파일러나 사람들이 쉽게 이해할 수 있고 조작할 수 있는 정보를 제공해준다.
>Here, I know that `compute` will not throw because it is composed out of operations that don't throw. By declaring `compute` to be `noexcept` I give the compiler and human readers information that can make it easier for them to understand and manipulate `compute`.

##### Note

많은 표준 라이브러리 함수는 `noexcept`이다. C 표준함수에서 파생된 모든 함수를 포함해서이다.
>Many standard library functions are `noexcept` including all the standard library functions "inherited" from the C standard library.

##### Example

    vector<double> munge(const vector<double>& v) noexcept
    {
        vector<double> v2(v.size());
        // ... do something ...
    }

여기서 `noexcept`는 로컬 `vector`를 생성할 수 없는 상황을 관리하고 싶지 않다는 것을 뜻한다.
즉, 메모리 부족을 심각한 디자인적 오류라고 생각하고 메모리 부족이 발생하면 프로그램이 기꺼이 망가지는 것을 감수하고자 한다.
>The `noexcept` here states that I am not willing or able to handle the situation where I cannot construct the local `vector`. That is, I consider memory exhaustion a serious design error (on line with hardware failures) so that I'm willing to crash the program if it happens.

**See also**: [discussion](#Sd-noexcept).

### <a name="Re-never-throw"></a> E.13: 객체의 직접 주인인 동안에는 예외를 발생시키지 마라.
>### <a name="Re-never-throw"></a> E.13: Never throw while being the direct owner of an object

##### Reason

메모리 누수가 발생할 것이다.
>That would be a leak.

##### Example

    void leak(int x)	// don't: may leak
    {
        auto p = new int{7};
        if (x < 0) throw Get_me_out_of_here{}  // may leak *p
        // ...
        delete p;	// we may never get here
    }

이 문제를 피하는 한 방법은 리소스 핸들을 일관(?)되게 사용하는 것이다.
>One way of avoiding such problems is to use resource handles consistently:

    void no_leak(int x)
    {
        auto p = make_unique<int>(7);
        if (x < 0) throw Get_me_out_of_here{};  // will delete *p if necessary
        // ...
        // no need for delete p
    }

**See also**: ??? 리소스 규칙 ???
>**See also**: ???resource rule ???

### <a name="Re-exception-types"></a> E.14: 목적에 맞는 사용자 정의 타입을 예외로 사용하라. (내장 타입은 안됨.)
>### <a name="Re-exception-types"></a> E.14: Use purpose-designed user-defined types as exceptions (not built-in types)

##### Reason

사용자 정의 타입은 다른 분이 작성한 예외와 충돌이 나지 않을 것이다.
>A user-defined type is unlikely to clash with other people's exceptions.

##### Example

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

##### Example, don't

    void my_code()	// Don't
    {
        // ...
        throw 7;	// 7 means "moon in the 4th quarter"
        // ...
    }

    void your_code()	// Don't
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

##### Note

`exception`에서 파생된 표준 라이브러리 클래스는 상위클래스나 "generic" 처리를 하는 예외에 대해서만 사용해야 한다.
내장 타입 사용처럼 표준 라이브러리 클래스는 다른 사람들이 사용할 수 있으므로 충돌이 날 수 있다.
>The standard-library classes derived from `exception` should be used only as base classes or for exceptions that require only "generic" handling. Like built-in types, their use could clash with other people's use of them.

##### Example, don't

    void my_code()	// Don't
    {
        // ...
        throw runtime_error{"moon in the 4th quarter"};
        // ...
    }

    void your_code()	// Don't
    {
        try {
            // ...
            my_code();
            // ...
        }
        catch(runtime_error) {	// runtime_error means "input buffer too small"
            // ...
        }
    }

**See also**: [Discussion](#Sd-???)

##### Enforcement

내장 타입을 사용하는 `throw`, `catch`문을 찾아내라. 표준 라이브러리 `exception` 타입을 사용하는 것에 대해서도 경고하라.
분명히 `std::exception` 계층구조에서 파생된 예외면 충분하다.
>Catch `throw` and `catch` of a built-in type. Maybe warn about `throw` and `catch` using an standard-library `exception` type. Obviously, exceptions derived from the `std::exception` hierarchy is fine.

### <a name="Re-exception-ref"></a> E.15: 참조형을 사용해서 상속받은 예외를 처리하라.
>### <a name="Re-exception-ref"></a> E.15: Catch exceptions from a hierarchy by reference

##### Reason

복사손실(slicing)을 없애기 위해.
>To prevent slicing.

##### Example

    void f()
    try {
        // ...
    }
    catch (exception e) {	// don't: may slice
        // ...
    }

위 대신에 이렇게 해라:
>Instead, use:
```
    catch (exception& e) { /* ... */ }
```
##### Enforcement

상속받은 예외가 값으로 처리된다면 표시한다. (프로그램 전체 분석이 완벽해야만 가능하다.)
>Flag by-value exceptions if their type are part of a hierarchy (could require whole-program analysis to be perfect).

### <a name="Re-never-fail"></a> E.16: 소멸자, 할당해제, 스왑은 절대 실패해선 안된다.
>### <a name="Re-never-fail"></a> E.16: Destructors, deallocation, and `swap` must never fail

##### Reason

 소멸자, 스왑, 메모리 할당해제가 실패한다면 신뢰할 수 있는 프로그램을 어떻게 작성해야 할지 잘 모르겠다.; 예외때문에 종료되 버리거나 단순히 그 작업을 수행할 수 없다면 말이다.
>We don't know how to write reliable programs if a destructor, a swap, or a memory deallocation fails; that is, if it exits by an exception or simply doesn't perform its required action.

##### Example, don't

    class Connection {
        // ...
    public:
        ~Connection()	// Don't: very bad destructor
        {
            if (cannot_disconnect()) throw I_give_up{information};
            // ...
        }
    };

##### Note

이 규칙을 어기면서 많은 분들이 종료를 거부하는 네트웍 연결 같은 예제에 대해서 신뢰할 수 있는 코드를 작성하려고 노력해왔다.
우리가 알고 있는 사실은 아주 특수한 경우를 제외하고는 누구도 일반적인 방법을 발견하지 못했다는 점이고, 나중에 제거/소거할 수 있도록 상태를 설정해 두는 것 정도만 가능하다.
모든 예제들은 에러가 쉽게 발생했고, 특수화되고, 버그가 많았다. (?? - 문장이 너무 어렵다.)
>Many have tried to write reliable code violating this rule for examples such as a network connection that "refuses to close". To the best of our knowledge nobody has found a general way of doing this though occasionally, for very specific examples, you can get away with setting some state for future cleanup. Every example, we have seen of this is error-prone, specialized, and usually buggy.

##### Note

표준 라이브러리는 소멸자, 할당해제 함수(`operator delete`), `swap`은 예외를 발생시키지 않는 것을 가정하고 있다.
예외 발생을 가정했다면 표준 라이브러리 불변성이 깨진다. (? - 이것도 이상하다.)
>The standard library assumes that destructors, deallocation functions (e.g., `operator delete`), and `swap` do not throw. If they do, basic standard library invariants are broken.

##### Note

할당해제 함수, `delete` 연산자를 포함해서 `noexcept`여야 한다. `swap`함수도 `noexcept`여야 한다.
대부분의 소멸자는 암시적으로 `noexcept` 기본값을 가진다. 그러나 소멸자를 `noexcept`로 만들어라.
>Deallocation functions, including `operator delete`, must be `noexcept`. `swap` functions must be `noexcept`. Most destructors are implicitly `noexcept` by default. destructors, make them `noexcept`.

##### Enforcement

`throw`를 쓰는 소멸자, 할당해제 연산, `swap`함수를 찾아내라. `noexcept`를 쓰지 않는 그런 류를 찾아내라.
>Catch destructors, deallocation operations, and `swap`s that `throw`. Catch such operations that are not `noexcept`.

**See also**: [discussion](#Sd-never-fail)

### <a name="Re-not-always"></a> E.17: 모든 함수에서 발생하는 모든 예외를 처리하려고 하지 마라.
>### <a name="Re-not-always"></a> E.17: Don't try to catch every exception in every function

##### Reason

의미있을 정도의 복구작업을 하지도 않는 함수에서 예외를 처리하는 것은 복잡성만 야기하고 쓸데없다.
처리할 수 있는 함수에 도달할때까지 예외가 전파되도록 둬라.
[RAII](#Re-raii)에 의해 처리될 수 있는 위치에 소거기능을 둬라. (??)
>Catching an exception in a function that cannot take a meaningful recovery action leads to complexity and waste.
Let an exception propagate until it reaches a function that can handle it.
Let cleanup actions on the unwinding path be handled by [RAII](#Re-raii).

##### Example, don't

    void f()	// bad
    {
        try {
            // ...
        }
        catch (...) {
            throw;	// propagate exception
        }
    }

##### Enforcement

* 중첩된 try문이 있다면 표시한다.
* 함수 대비 try문이 너무 많다면 표시한다. (문제는 너무 많음을 정의하는 정도.)

>* Flag nested try-blocks.
* Flag source code files with a too high ratio of try-blocks to functions. (??? Problem: define "too high")

### <a name="Re-catch"></a> E.18: 명시적인 `try`/`catch`문의 사용을 최소화하라.
>### <a name="Re-catch"></a> E.18: Minimize the use of explicit `try`/`catch`

##### Reason

`try`/`catch`문은 좀 길어 보이고, 특이하게 사용하다가는 에러가 쉽게 발생한다.
>`try`/`catch` is verbose and non-trivial uses error-prone.

##### Example

    ???

##### Enforcement

???

### <a name="Re-finally"></a> E.19: 적당한 리소스 핸들을 사용할 수 없다면 처리방법을 나타낼 `final_action` 객체를 사용하라.
>### <a name="Re-finally"></a> E.19: Use a `final_action` object to express cleanup if no suitable resource handle is available

##### Reason

`finally`문 정도면 적절해서 `try`/`catch`문에 비해 나쁘지 않을 것이다.
>`finally` is less verbose and harder to get wrong than `try`/`catch`.

##### Example

    void f(int n)
    {
        void* p = malloc(1, n);
        auto _ = finally([p] { free(p); });
        // ...
    }

**See also** ????

### <a name="Re-no-throw"></a> E.25: ??? 예외를 던질 수 없는 프로그램에서는 무엇을 할 것인가.
>### <a name="Re-no-throw"></a> E.25: ??? What to do in programs where exceptions cannot be thrown

##### Note

??? 대부분은 예외처리할 여지는 있을 것이고 코드는 좀더 심플해 질 것이다. ???
>??? mostly, you can afford exceptions and code gets simpler with exceptions ???
**See also**: [Discussion](#Sd-???).
