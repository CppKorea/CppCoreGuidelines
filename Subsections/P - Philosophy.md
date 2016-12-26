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
