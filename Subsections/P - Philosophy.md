# <a name="S-philosophy"></a> P: Philosophy

이 단원의 규칙은 매우 일반적이다.
>The rules in this section are very general.

철학 규칙 요약:
>Philosophy rules summary:

* [P.1: Express ideas directly in code](#Rp-direct)
* [P.2: Write in ISO Standard C++](#Rp-C++)
* [P.3: Express intent](#Rp-what)
* [P.4: Ideally, a program should be statically type safe](#Rp-typesafe)
* [P.5: Prefer compile-time checking to run-time checking](#Rp-compile-time)
* [P.6: What cannot be checked at compile time should be checkable at run time](#Rp-run-time)
* [P.7: Catch run-time errors early](#Rp-early)
* [P.8: Don't leak any resources](#Rp-leak)
* [P.9: Don't waste time or space](#Rp-waste)

철학적 규칙은 보통 기계적으로 체크할 수 없다.
그러나 철학적인 테마를 반영하는 개별적인 규칙은 체크가능하다.
철학적인 기초가 없이 구체적이고/특수하고/체크가능한 규칙은 근거가 부족하다.
>Philosophical rules are generally not mechanically checkable.
However, individual rules reflecting these philosophical themes are.
Without a philosophical basis the more concrete/specific/checkable rules lack rationale.

### <a name="Rp-direct"></a> P.1: 아이디어를 직접 코드로 표현하라.
>### <a name="Rp-direct"></a> P.1: Express ideas directly in code

##### Reason

컴파일러는 주석문 (혹은 디자인 문서 등)을 읽지 않는다. 수 많은 프로그래머 또한 주석을 (일관되게) 읽지 않는다.
코드로 표현된 내용이라면 그 의미(의도)를 이미 정의 했을 것이며 (대체로) 컴파일러나 다른 툴로 체크할 수 있다.
>Compilers don't read comments (or design documents) and neither do many programmers (consistently).
What is expressed in code has defined semantics and can (in principle) be checked by compilers and other tools.

##### Example

    class Date {
        // ...
    public:
        Month month() const;  // do
        int month();          // don't
        // ...
    };

첫번째 `month` 함수는 명확히 `Month`를 반한 하도록 선언되어 있으며, `Date` 객체의 상태를 변경하지 않을 것 처럼 보인다. 
두번째 버전은 코드를 읽는 개발자들을 고민하게 만들며, 발견하기 어려운 버그를 유발할 가능성이 있다.
>The first declaration of `month` is explicit about returning a `Month` and about not modifying the state of the `Date` object.
The second version leaves the reader guessing and opens more possibilities for uncaught bugs.

##### Example

    void do_something(vector<string>& v)
    {
        string val;
        cin >> val;
        // ...
        int index = 0;            // 나쁨
        for (int i = 0; i < v.size(); ++i)
            if (v[i] == val) {
                index = i;
                break;
            }
        // ...
    }

위의 루프는 `std::find`를 이용하여 표현 가능하므로, 
의도를 더 명확하게 드러내기 위해서 아래와 같이 코드를 바꾸어 작성할 수 있다:
>That loop is a restricted form of `std::find`.
A much clearer expression of intent would be:

    void do_something(vector<string>& v)
    {
        string val;
        cin >> val;
        // ...
        auto p = find(v, val);  // 좋음
        // ...
    }

언어가 제공하는 기능을 직접 사용하기 보다 잘 설계된 라이브러리를 사용하여 그 의도(어떻게 수행되는지 보다 무엇이 수행되는지를) 표현하는 것이 훨씬 낫다.  
>A well-designed library expresses intent (what is to be done, rather than just how something is being done) far better than direct use of language features.

C++ 프로그래머는 표준 라이브러리의 기본을 반드시 이해하고 올바른 곳에 사용해야 한다.
어떤 프로그래머이든 프로젝트가 기반하고 있는 핵심 라이브러리의 기본을 반드시 이해하고 있어야 하며, 올바르게 사용할 줄 알아야 한다.
이 가이드라인을 사용하는 프로그래머는 [guideline support library](#S-gsl)를 반드시 알아야 하고 적절히 사용할 줄 알아야 한다.
>A C++ programmer should know the basics of the standard library, and use it where appropriate.
Any programmer should know the basics of the foundation libraries of the project being worked on, and use them appropriately.
Any programmer using these guidelines should know the [guideline support library](#S-gsl), and use it appropriately.

##### Example

    change_speed(double s);   // 나쁨: 무슨 의미일까?
    // ...
    change_speed(2.3);

더 좋은 접근법은 `double`(새 스피드, 혹은 이전 스피드와의 차이?)의 의미와 단위를 명확히 하는 것이다.
>A better approach is to be explicit about the meaning of the double (new speed or delta on old speed?) and the unit used:

    change_speed(Speed s);    // 좋음: s의 의미가 구체적임
    // ...
    change_speed(2.3);        // 에러: 단위가 없음
    change_speed(23m / 10s);  // 미터/초

단순한(단위가 없는) `double`을 델타(속도의 차)로 받아들일 수도 있겠지만, 아무래도 에러가 발생하기 쉽다.
속도와 델타 둘다 필요하다면, `Delta` 타입을 정의해야 할 것이다.
>We could have accepted a plain (unit-less) `double` as a delta, but that would have been error-prone.
If we wanted both absolute speed and deltas, we would have defined a `Delta` type.

##### Enforcement

일반화하여 적용하기는 매우 어려움.
>Very hard in general.

* 일관성 있게 `const`를 사용하라.(멤버함수가 객체를 변경하는지 확인하라; 포인터나 참조로 넘어온 인자를 변경하는지 확인하라.)
* 형변환은 그만 사용하라.(형변환은 타입시스템을 무력화시킨다)
* 표준 라이브러리를 흉내내는 코드를 찾아라. (어렵다)

>* use `const` consistently (check if member functions modify their object; check if functions modify arguments passed by pointer or reference)
* flag uses of casts (casts neuter the type system)
* detect code that mimics the standard library (hard)

### <a name="Rp-C++"></a> P.2: ISO 표준 C++로 작성하라.
>### <a name="Rp-C++"></a> P.2: Write in ISO Standard C++

##### Reason

ISO 표준 C++을 사용하기 위한 가이드라인 집합이다.
>This is a set of guidelines for writing ISO Standard C++.

##### Note

시스템 리소스를 접근하기 위해 확장 모듈이 필요한 환경이 있다.
이런 경우에는 필요한 확장모듈을 지역적으로 사용하라. 비핵심 코딩 가이드라인으로 사용을 제한하라.
>There are environments where extensions are necessary, e.g., to access system resources.
In such cases, localize the use of necessary extensions and control their use with non-core Coding Guidelines.

##### Note

표준 C++ 언어, 라이브러리 특성에 대한 제약이 필요한 환경도 있다. 예를 들면 비행기 제어 소프트웨어 표준에서 요구하는 동적 메모리 할당을 피하기.
이러 경우에는 비핵심 코딩 가이드라인으로 사용을 제한하라.
>There are environments where restrictions on use of standard C++ language or library features are necessary, e.g., to avoid dynamic memory allocation as required by aircraft control software standards.
In such cases, control their (dis)use with non-core Coding Guidelines.

##### Enforcement

확장을 허용하지 않는 옵션셋을 가진 최신 C++ 컴파일러를 사용하라. (현재 C++11, C++14)
>Use an up-to-date C++ compiler (currently C++11 or C++14) with a set of options that do not accept extensions.

### <a name="Rp-what"></a> P.3: 의도를 표현하라.
>### <a name="Rp-what"></a> P.3: Express intent

##### Reason

코드의 의도를 (이름이나 주석문으로) 기술하지 않는다면, 의도대로 코드가 실행되는지 어떤지 말하기가 불가능하다.
>Unless the intent of some code is stated (e.g., in names or comments), it is impossible to tell whether the code does what it is supposed to do.

##### Example

    int i = 0;
    while (i < v.size()) {
        // ... do something with v[i] ...
    }

여기에 `v`의 요소를 루프하는 의도가 표현되지 않는다.
인덱스의 상세한 구현은 보인다. (잘못 사용될지도 모르겠지만) 의도적이든 아니든 `i`는 루프 영역 외에서도 살아 있다.
읽는 사람은 이 코드 일부분으로는 알 수 있는 게 없다.
>The intent of "just" looping over the elements of `v` is not expressed here. The implementation detail of an index is exposed (so that it might be misused), and `i` outlives the scope of the loop, which may or may not be intended. The reader cannot know from just this section of code.

더 좋게:
>Better:

    for (auto x : v) { /* do something with x */ }

여기 구체적인 언급은 없다. 반복 메커니즘에 대한, 그리고 루프는 요소의 복사본을 가지고 동작한다.
실수로 수정되는 것을 막기 위해.
수정이 필요하다면 아래처럼 써라:
>Now, there is no explicit mention of the iteration mechanism, and the loop operates on a copy of elements so that accidental modification cannot happen. If modification is desired, say so:

    for (auto& x : v) { /* do something with x */ }

나아지기는 했지만, 이름붙인 알고리즘을 사용하라.:
>Sometimes better still, use a named algorithm:

    for_each(v, [](int x) { /* do something with x */ });
    for_each(parallel.v, [](int x) { /* do something with x */ });

마지막 수정은 `v` 요소의 순서에는 별다른 흥미가 없다는 것을 명확하게 만든다.
>The last variant makes it clear that we are not interested in the order in which the elements of `v` are handled.

프로그래머라면 다음과 익숙해져야 한다.
>A programmer should be familiar with

* [The guideline support library](#S-gsl)
* [The ISO C++ standard library](#S-stdlib)
* 기본 라이브러리를 무엇을 사용하던지.

>* Whatever foundation libraries are used for the current project(s)

##### Note

대안 공식: 무엇을 할지 말하라, 어떻게 할지 말하지 말고.
>Alternative formulation: Say what should be done, rather than just how it should be done.

##### Note

몇몇 언어의 구조는 의도를 잘 표현한다.
>Some language constructs express intent better than others.

##### Example

2개의 `int`값이 2D 포인터 좌표를 의미한다면 이렇게 써라:
>If two `int`s are meant to be the coordinates of a 2D point, say so:

      drawline(int, int, int, int);  // obscure
      drawline(Point, Point);        // clearer

##### Enforcement

대안이 더 좋은 공통 패턴을 찾아라.
>Look for common patterns for which there are better alternatives

* 단순 `for`문 루프 대 범위 `for`문
* `f(T*, int)` 인터페이스 대 `f(array_view<T>)` 인터페이스
* 아주 큰 범위에서 사용하는 루프 변수
* 생짜 `new`, `delete`
* 여러개의 내장 타입 인자를 가진 함수.

>* simple `for` loops vs. range-`for` loops
>* `f(T*, int)` interfaces vs. `f(array_view<T>)` interfaces
>* loop variables in too large a scope
>* naked `new` and `delete`
>* functions with many arguments of built-in types

영리함, 반자동 프로그램 변환을 위한 거대한 범위(?)가 있다. (? - scope 모르겠음.)
>There is a huge scope for cleverness and semi-automated program transformation.

### <a name="Rp-typesafe"></a> P.4: 이상적으로는 프로그램은 정적으로 타입이 안전해야 한다.
>### <a name="Rp-typesafe"></a> P.4: Ideally, a program should be statically type safe

##### Reason

이상적으로 프로그램은 완전히 정적으로 타입이 안전해야 한다.
불행하게도 불가능하다. 문제영역은:
>Ideally, a program would be completely statically (compile-time) type safe.
Unfortunately, that is not possible. Problem areas:

* 유니온
* 형변환
* 배열 값이 망가짐.
* 범위 에러
* 축소 형변환

>* unions
>* casts
>* array decay
>* range errors
>* narrowing conversions

##### Note

이 영역들은 심각한 문제의 원인이 된다. (프로그램 충돌과 보안 위반)
다른 방법을 제공하고자 한다.
>These areas are sources of serious problems (e.g., crashes and security violations).
We try to provide alternative techniques.

##### Enforcement

금지하고, 저지하고, 개별문제 영역을 따로따로 찾아 낼 수 있다. 요구된대로, 개별 프로그램에서 가능하도록.
항상 대안을 제시하라.
예를 들면:
>We can ban, restrain, or detect the individual problem categories separately, as required and feasible for individual programs.
Always suggest an alternative.
For example:

* 유니온 - `variant`을 사용하라.
* 형변환 - 사용을 최소화하라. 템플릿이 도움이 될 수 있다.
* 배열 부패 - `array_view`를 사용하라.
* 범위 에러 - `array_view`를 사용하라.
* 축소 형변환 - 사용을 최소화하라. 필요하면 `narrow`, `narrow_cast`를 사용하라.

>* casts - minimize their use; templates can help
>* unions - use `variant`
>* array decay - use `array_view`
>* range errors - use `array_view`
>* narrowing conversions - minimize their use and use `narrow` or `narrow_cast` where they are necessary

### <a name="Rp-compile-time"></a> P.5: 런타임 체크보다는 컴파일타임 체크를 선호하라.
>### <a name="Rp-compile-time"></a> P.5: Prefer compile-time checking to run-time checking

##### Reason

코드 명확성, 성능향상. 컴파일 타임에 발견되는 에러에 대해서는 에러 처리기를 작성할 필요가 없다.
>Code clarity and performance. You don't need to write error handlers for errors caught at compile time.

##### Example

    void initializer(Int x)
    // Int is an alias used for integers
    {
        static_assert(sizeof(Int) >= 4);	// do: compile-time check

        int bits = 0;         // don't: avoidable code
        for (Int i = 1; i; i <<= 1)
            ++bits;
        if (bits < 32)
            cerr << "Int too small\n";

        // ...
    }

##### Example don't

    void read(int* p, int n);   // read max n integers into *p

##### Example

    void read(array_view<int> r); // read into the range of integers r

**Alternative formulation**: 컴파일 타임에 할 수 있는 것을 런타임으로 연기하지 마라.
>**Alternative formulation**: Don't postpone to run time what can be done well at compile time.

##### Enforcement

* 포인터 인자를 찾아라.
* 범위 오류에 대해서 런타임 체크를 찾아라.

>* Look for pointer arguments.
>* Look for run-time checks for range violations.

### <a name="Rp-run-time"></a> P.6: 컴파일 타임에 체크할 수 없다면 런타임에 체크할 수 있어야 한다.
>### <a name="Rp-run-time"></a> P.6: What cannot be checked at compile time should be checkable at run time

##### Reason

프로그램 속에 찾아내기 어려운 에러를 남겨둔다면 프로그램 충돌이나 나쁜 결과를 야기한다.
>Leaving hard-to-detect errors in a program is asking for crashes and bad results.

##### Note

이상적으로 우리는 컴파일타임, 런타임에 모든 에러를 찾을 수 있다.(프로그래머의 논리에서는 에러가 아닌 것)
컴파일타임에 모든 에러를 찾아내는 건 불가능하고 런타임에 남아 있는 모든 에러를 찾는 것도 불가능하다.
그러나 충분한 리소스를 준다면 원론적으로 체크 가능한 프로그램을 작성하려고 노력해야 한다.
(분석 프로그램, 런타임 체크, 기계 리소스, 시간)
>Ideally we catch all errors (that are not errors in the programmer's logic) at either compile-time or run-time. It is impossible to catch all errors at compile time and often not affordable to catch all remaining errors at run time. However, we should endeavor to write programs that in principle can be checked, given sufficient resources (analysis programs, run-time checks, machine resources, time).

##### Example, bad

    extern void f(int* p);  // separately compiled, possibly dynamically loaded

    void g(int n)
    {
        f(new int[n]);  // bad: the number of elements is not passed to f()
    }

여기서 결정적인 정보(원소 갯수) 아주 철저하게 숨겨져 있어서,
`f()`가 그 포인터를 기구화하지 않는 ABI의 일부분일 때 정적 분석은 아마도 불가능해 보이고 동적 체크는 아주 어려울 수 있다. (? - 어렵다)
도움이 될만한 정보를 남은 공간에 넣을 수 있지만, 그것은 시스템이나 컴파일러에게 전반적인 변경을 요구한다.
우리가 가진 것은 에러 발견을 아주 어렵게 만드는 디자인이다.
>Here, a crucial bit of information (the number of elements) has been so thoroughly "obscured" that static analysis is probably rendered infeasible and dynamic checking can be very difficult when `f()` is part of an ABI so that we cannot "instrument" that pointer. We could embed helpful information into the free store, but that requires global changes to a system and maybe to the compiler. What we have here is a design that makes error detection very hard.

##### Example, bad

포인터와 함께 원소의 갯수도 같이 넘길 수 있다.:
>We can of course pass the number of elements along with the pointer:

    extern void f2(int* p, int n);  // separately compiled, possibly dynamically loaded

    void g2(int n)
    {
        f2(new int[n], m);    // bad: the wrong number of elements can be passed to f()
    }


인자로 원소의 갯수를 전달하는 것은 포인터를 넘기면서 관례에 따라 원소의 갯수를 구하는 것보다 낫다.
그러나, 단순히 철자 오류만으로도 심각한 에러를 야기한다. `f2()`의 두 인자간의 연결은 구체적이지 않고 관례에 따른 것이다.
>Passing the number of elements as an argument is better (and far more common) that just passing the pointer and relying on some (unstated) convention for knowing or discovering the number of elements. However (as shown), a simple typo can introduce a serious error. The connection between the two arguments of `f2()` is conventional, rather than explicit.

게다가 `f2()`가 인자를 `delete`할지는 정의되지 않았다.(호출자가 두번째 실수를 한 것인가?)
>Also, it is implicit that `f2()` is supposed to `delete` its argument (or did the caller make a second mistake?).

##### Example, bad

표준라이브러리 리소스관리 포인터는 객체를 가리키고 있을 때 사이즈를 넘길 수 없다.
>The standard library resource management pointers fail to pass the size when they point to an object:

    extern void f3(unique_ptr<int[]>, int n);    // separately compiled, possibly dynamically loaded

    void g3(int n)
    {
        f3(make_unique<int[]>(n), m);    // bad: pass ownership and size separately
    }

##### Example

포인터와 원소의 갯수를 하나의 객체로 합쳐서 전달할 필요가 있다.:
>We need to pass the pointer and the number of elements as an integral object:

    extern void f4(vector<int>&);       // separately compiled, possibly dynamically loaded
    extern void f4(array_view<int>);    // separately compiled, possibly dynamically loaded

    void g3(int n)
    {
        vector<int> v(n);
        f4(v);                     // pass a reference, retain ownership
        f4(array_view<int>{v});    // pass a view, retain ownership
    }

이 디자인은 에러 가능성이 없애고 항상 동적 체크가 가능하도록 원소의 갯수를 객체의 필수적인 부분으로 전달한다.
>This design carries the number of elements along as an integral part of an object, so that errors are unlikely and dynamic (run-time) checking is always feasible, if not always affordable.

##### Example

어떻게 소유권과 올바른 사용을 위한 모든 정보를 전달할 것인가?
>How do we transfer both ownership and all information needed for validating use?

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

##### Example

* ???
* 필요한 것을 실제로 알게 되면 다형 기본 클래스를 전달하는 인터페이스가 어떻게 체크를 피하는게 가능한지 보여준다.
  또는 자유 스타일 옵션같은 문자열 (? - 어렵다.)

>* ???
>* show how possible checks are avoided by interfaces that pass polymorphic base classes around, when they actually know what they need?
  Or strings as "free-style" options

##### Enforcement

* (포인터, 갯수)-스타일 인터페이스라면 표시한다. (호환성을 이유로 고칠 수 없는 많은 예제를 표시할 것이다.)
> * Flag (pointer, count)-style interfaces (this will flag a lot of examples that can't be fixed for compatibility reasons)
> * ???

### <a name="Rp-early"></a> P.7: 런타임 에러를 초기에 발견하라.
>### <a name="Rp-early"></a> P.7: Catch run-time errors early

##### Reason

이해하기 힘든 프로그램 충돌을 피하라.
잘못된(아마 몰랐을 수도 있는) 결과를 야기하는 에러를 피하라.
>Avoid "mysterious" crashes.
Avoid errors leading to (possibly unrecognized) wrong results.

##### Example

    void increment1(int* p, int n)    // bad: error prone
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

여기에서 우리는 `use1`에 데이터를 깨먹거나 프로그램 충돌을 야기할 수 있는 작은 실수를 했다.
(포인터, 크기) 스타일 인터페이스는 `increment1()`을 범위에러에 대해 방어할 수 있는 현실적인 방안을 없애 버린다.
범위를 벗어나는지에 대해서 배열첨자를 체크한다면, 에러는 `p[10]`까지 발견되지 않을 것이다.
좀더 빨리 체크하도록 코드를 개선해 보자:
>Here we made a small error in `use1` that will lead to corrupted data or a crash.
The (pointer, count)-style interface leaves `increment1()` with no realistic way of defending itself against out-of-range errors.
Assuming that we could check subscripts for out of range access, the error would not be discovered until `p[10]` was accessed.
We could check earlier and improve the code:

    void increment2(array_view<int> p)
    {
        for (int& x : p) ++x;
    }

    void use2(int m)
    {
        const int n = 10;
        int a[n] = {};
        // ...
        increment2({a, m});    // maybe typo, maybe m<=n is supposed
        // ...
    }

여기 `m<=n`를 호출 지점에서 체크할 수 있다.
`n`을 범위로 사용할려고 한 철자 오류였다면, 코드는 훨씬 단순했을 것이다.(에러 가능성도 없어지면서)
>Now, `m<=n` can be checked at the point of call (early) rather than later.
If all we had was a typo so that we meant to use `n` as the bound, the code could be further simplified (eliminating the possibility of an error):

    void use3(int m)
    {
        const int n = 10;
        int a[n] = {};
        // ...
        increment2(a);   // the number of elements of a need not be repeated
        // ...
    }

##### Example, bad

동일값을 반복적으로 체크하지 마라. 구조화된 데이터를 문자열로 넘기지 마라.
>Don't repeatedly check the same value. Don't pass structured data as strings:

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

날짜가 두번 계산된다. (`Date` 생성자에 의해서) 그리고 문자열로 전달된다. (비구조화된 데이터)
>The date is validated twice (by the `Date` constructor) and passed as a character string (unstructured data).

##### Example

지나치게 체크하면 비용이 든다.
값을 필요하지 어떨지도 모르기 때문에, 일찍 체크하는 것이 안 좋은 경우도 있고 전체 값보다 개별로 체크하는게 쉬운 경우도 있다.
>Excess checking can be costly.
There are cases where checking early is dumb because you may not ever need the value, or may only need part of the value that is more easily checked than the whole.

    class Jet {    // Physics says: e*e < x*x + y*y + z*z

        float fx, fy, fz, fe;
    public:
        Jet(float x, float y, float z, float e)
            :fx(x), fy(y), fz(z), fe(e)
        {
            // Should I check here that the values are physically meaningful?
        }

        float m() const
        {
            // Should I handle the degenerate case here?
            return sqrt(x*x + y*y + z*z - e*e);
        }

        ???
    };

`e*e < x*x + y*y + z*z` 젯에 대한 물리적 법칙은 측정오류 가능성 때문에 값이 바뀔 수 있다. (? - 어렵다.)
>The physical law for a jet (`e*e < x*x + y*y + z*z`) is not an invariant because of the possibility for measurement errors.

???

##### Enforcement

* 포인터와 배열을 찾아라: 빨리 범위를 체크하라.
* 타입 변환을 찾아라: 축소 변환에 대해서 표시하거나 제거하라.
* 입력으로 들어오는 값이 체크되지 않는지 찾아라.
* 문자열로 변환되고 있는 구조화된 데이터(불변조건을 가진 클래스의 객체)를 찾아라.
* ???

>* Look at pointers and arrays: Do range-checking early
>* Look at conversions: Eliminate or mark narrowing conversions
>* Look for unchecked values coming from input
>* Look for structured data (objects of classes with invariants) being converted into strings
>* ???

### <a name="Rp-leak"></a> P.8: 리소스를 누락시키지 마라.
>### <a name="Rp-leak"></a> P.8: Don't leak any resources

##### Reason

장시간 실행해야 하는 프로그램에서는 필수적이다. 효율성. 에러를 복구하는 능력.
>Essential for long-running programs. Efficiency. Ability to recover from errors.

##### Example, bad

    void f(char* name)
    {
        FILE* input = fopen(name, "r");
        // ...
        if (something) return;   // bad: if something == true, a file handle is leaked
        // ...
        fclose(input);
    }

[RAII](#Rr-raii)를 선호하라:
>Prefer [RAII](#Rr-raii):

    void f(char* name)
    {
        ifstream input {name};
        // ...
        if (something) return;   // OK: no leak
        // ...
    }

**See also**: [The resource management section](#S-resource)

##### Enforcement

* 포인터를 살펴봐라: 소유자와 비소유자로 구분해라.
  가능하다면 소유자를 표준라이브러리 리소스 핸들로 바꿔라.(위의 예제처럼.)
	또는 [the GSL](#S-gsl)에서 `owner`를 사용하는 것처럼 소유자를 표시하라.
* 생짜 `new`, `delete`를 찾아라.
* 잘 알려진 생짜 포인터를 반환하는 리소스 할당함수를 찾아라.
  (예를 들어 `fopen`, `malloc`, `strdup`)

>* Look at pointers: Classify them into non-owners (the default) and owners.
  Where feasible, replace owners with standard-library resource handles (as in the example above).
  Alternatively, mark an owner as such using `owner` from [the GSL](#S-gsl).
>* Look for naked `new` and `delete`
>* Look for known resource allocating functions returning raw pointers (such as `fopen`, `malloc`, and `strdup`)

### <a name="Rp-waste"></a> P.9: 시간이나 공간을 낭비하지 마라.
>### <a name="Rp-waste"></a> P.9: Don't waste time or space

##### Reason

이것이 C++이다.
>This is C++.

##### Note

개발속도, 안전한 리소스, 테스트 단순화 등의 목표를 수행하기 위해 제대로 소모하는 시간이나 공간은 낭비가 아니다.
>Time and space that you spend well to achieve a goal (e.g., speed of development, resource safety, or simplification of testing) is not wasted.

##### Example

쓸데없는 낭비에 대한 좋은 제안은 언제나 환영 ???
>??? more and better suggestions for gratuitous waste welcome ???

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
    	if (p == nullptr) throw Nullptr_error{};
        int n = strlen(p);
        auto buf = new char[n];
        if (buf == nullptr) throw Allocation_error{};
        for (int i = 0; i < n; ++i) buf[i] = p[i];
        // ... manipulate buffer ...
        X x;
        x.ch = 'a';
        x.s = string(n);    // give x.s space for *ps
        for (int i = 0; i < x.s.size(); ++i) x.s[i] = buf[i];	// copy buf into x.s
        delete buf;
        return x;
    }

    void driver()
    {
        X x = waste("Typical argument");
        // ...
    }

예스, 이건 조크이기는 하지만, 실제 코드에서 이런 각각의 실수는 매번 보아 왔다.
`X`의 배치는 적어도 6바이트(다 많을지도 모르지만)의 낭비가 있다는 점을 주목하자.
복사연산을 그럴싸하게 정의해 두다보니 이동 의미가 없어져 버렸다. 그래서 반환 기능이 느려졌다.
`buf`의 `new`, `delete`는 불필요하게 중복적이다.
진짜 지역 문자열을 원했다면 `string` 지역변수를 사용했을 것이다.
몇가지 성능 버그가 있고 쓸데없이 복잡한 것도 문제다.
>Yes, this is a caricature, but we have seen every individual mistake in production code, and worse.
Note that the layout of `X` guarantees that at least 6 bytes (and most likely more) bytes are wasted.
The spurious definition of copy operations disables move semantics so that the return operation is slow.
The use of `new` and `delete` for `buf` is redundant; if we really needed a local string, we should use a local `string`.
There are several more performance bugs and gratuitous complication.

##### Note

낭비에 대한 각각의 예제는 별로 중요하지 않다. 그리고 중요했다면 이미 전문가들이 쉽게 제거했을 것이다.
그러나 코드 전체에 걸쳐 자유롭게 낭비가 퍼져버리면 중요해 질 수 있고,
낭비를 제거하기 위해서 전문가들을 항상 원하는데로 데려다 쓸 수는 없다.
이 규칙(더 세부적인 규칙)의 목적은 큰 낭비가 일어나기 전에 C++을 사용해서 제거하는 것이다.
그런 후에 알고리즘이나 요구사항과 관련된 낭비요소를 찾아볼 수 있다. 그건 이 영역 범위 밖이다.
>An individual example of waste is rarely significant, and where it is significant, it is typically easily eliminated by an expert.
However, waste spread liberally across a code base can easily be significant and experts are not always as available as we would like.
The aim of this rule (and the more specific rules that support it) is to eliminate most waste related to the use of C++ before it happens.
After that, we can look at waste related to algorithms and requirements, but that is beyond the scope of these guidelines.

##### Enforcement

많은 특정한 규칙은 단순한 총괄 목표, 쓸데없는 낭비 제거를 노리고 있다.
>Many more specific rules aim at the overall goals of simplicity and elimination of gratuitous waste.
