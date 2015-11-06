# <a name="S-philosophy"></a> P: Philosophy

The rules in this section are very general.

Philosophy rules summary:

* [P.1: Express ideas directly in code](#Rp-direct)
* [P.2: Write in ISO Standard C++](#Rp-C++)
* [P.3: Express intent](#Rp-what)
* [P.4: Ideally, a program should be statically type safe](#Rp-typesafe)
* [P.5: Prefer compile-time checking to run-time checking](#Rp-compile-time)
* [P.6: What cannot be checked at compile time should be checkable at run time](#Rp-run-time)
* [P.7: Catch run-time errors early](#Rp-early)
* [P.8: Don't leak any resources](#Rp-leak)
* [P.9: Don't waste time or space](#Rp-waste)

Philosophical rules are generally not mechanically checkable.
However, individual rules reflecting these philosophical themes are.
Without a philosophical basis the more concrete/specific/checkable rules lack rationale.

### <a name="Rp-direct"></a> P.1: Express ideas directly in code

##### Reason

Compilers don't read comments (or design documents) and neither do many programmers (consistently).
What is expressed in code has defined semantics and can (in principle) be checked by compilers and other tools.

##### Example

    class Date {
        // ...
    public:
        Month month() const;  // do
        int month();          // don't
        // ...
    };

The first declaration of `month` is explicit about returning a `Month` and about not modifying the state of the `Date` object.
The second version leaves the reader guessing and opens more possibilities for uncaught bugs.

##### Example

    void do_something(vector<string>& v)
    {
        string val;
        cin >> val;
        // ...
        int index = 0;            // bad
        for (int i = 0; i < v.size(); ++i)
            if (v[i] == val) {
                index = i;
                break;
            }
        // ...
    }

That loop is a restricted form of `std::find`.
A much clearer expression of intent would be:

    void do_something(vector<string>& v)
    {
        string val;
        cin >> val;
        // ...
        auto p = find(v, val);  // better
        // ...
    }

A well-designed library expresses intent (what is to be done, rather than just how something is being done) far better than direct use of language features.

A C++ programmer should know the basics of the standard library, and use it where appropriate.
Any programmer should know the basics of the foundation libraries of the project being worked on, and use them appropriately.
Any programmer using these guidelines should know the [guideline support library](#S-gsl), and use it appropriately.

##### Example

    change_speed(double s);   // bad: what does s signify?
    // ...
    change_speed(2.3);

A better approach is to be explicit about the meaning of the double (new speed or delta on old speed?) and the unit used:

    change_speed(Speed s);    // better: the meaning of s is specified
    // ...
    change_speed(2.3);        // error: no unit
    change_speed(23m / 10s);  // meters per second

We could have accepted a plain (unit-less) `double` as a delta, but that would have been error-prone.
If we wanted both absolute speed and deltas, we would have defined a `Delta` type.

##### Enforcement

Very hard in general.

* use `const` consistently (check if member functions modify their object; check if functions modify arguments passed by pointer or reference)
* flag uses of casts (casts neuter the type system)
* detect code that mimics the standard library (hard)

### <a name="Rp-C++"></a> P.2: Write in ISO Standard C++

##### Reason

This is a set of guidelines for writing ISO Standard C++.

##### Note

There are environments where extensions are necessary, e.g., to access system resources.
In such cases, localize the use of necessary extensions and control their use with non-core Coding Guidelines.

##### Note

There are environments where restrictions on use of standard C++ language or library features are necessary, e.g., to avoid dynamic memory allocation as required by aircraft control software standards.
In such cases, control their (dis)use with non-core Coding Guidelines.

##### Enforcement

Use an up-to-date C++ compiler (currently C++11 or C++14) with a set of options that do not accept extensions.

### <a name="Rp-what"></a> P.3: Express intent

##### Reason

Unless the intent of some code is stated (e.g., in names or comments), it is impossible to tell whether the code does what it is supposed to do.

##### Example

    int i = 0;
    while (i < v.size()) {
        // ... do something with v[i] ...
    }

The intent of "just" looping over the elements of `v` is not expressed here. The implementation detail of an index is exposed (so that it might be misused), and `i` outlives the scope of the loop, which may or may not be intended. The reader cannot know from just this section of code.

Better:

    for (auto x : v) { /* do something with x */ }

Now, there is no explicit mention of the iteration mechanism, and the loop operates on a copy of elements so that accidental modification cannot happen. If modification is desired, say so:

    for (auto& x : v) { /* do something with x */ }

Sometimes better still, use a named algorithm:

    for_each(v, [](int x) { /* do something with x */ });
    for_each(parallel.v, [](int x) { /* do something with x */ });

The last variant makes it clear that we are not interested in the order in which the elements of `v` are handled.

A programmer should be familiar with

* [The guideline support library](#S-gsl)
* [The ISO C++ standard library](#S-stdlib)
* Whatever foundation libraries are used for the current project(s)

##### Note

Alternative formulation: Say what should be done, rather than just how it should be done.

##### Note

Some language constructs express intent better than others.

##### Example

If two `int`s are meant to be the coordinates of a 2D point, say so:

      drawline(int, int, int, int);  // obscure
      drawline(Point, Point);        // clearer

##### Enforcement

Look for common patterns for which there are better alternatives

* simple `for` loops vs. range-`for` loops
* `f(T*, int)` interfaces vs. `f(array_view<T>)` interfaces
* loop variables in too large a scope
* naked `new` and `delete`
* functions with many arguments of built-in types

There is a huge scope for cleverness and semi-automated program transformation.

### <a name="Rp-typesafe"></a> P.4: Ideally, a program should be statically type safe

##### Reason

Ideally, a program would be completely statically (compile-time) type safe.
Unfortunately, that is not possible. Problem areas:

* unions
* casts
* array decay
* range errors
* narrowing conversions

##### Note

These areas are sources of serious problems (e.g., crashes and security violations).
We try to provide alternative techniques.

##### Enforcement

We can ban, restrain, or detect the individual problem categories separately, as required and feasible for individual programs.
Always suggest an alternative.
For example:

* unions - use `variant`
* casts - minimize their use; templates can help
* array decay - use `array_view`
* range errors - use `array_view`
* narrowing conversions - minimize their use and use `narrow` or `narrow_cast` where they are necessary

### <a name="Rp-compile-time"></a> P.5: Prefer compile-time checking to run-time checking

##### Reason

Code clarity and performance. You don't need to write error handlers for errors caught at compile time.

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

**Alternative formulation**: Don't postpone to run time what can be done well at compile time.

##### Enforcement

* Look for pointer arguments.
* Look for run-time checks for range violations.

### <a name="Rp-run-time"></a> P.6: What cannot be checked at compile time should be checkable at run time

##### Reason

Leaving hard-to-detect errors in a program is asking for crashes and bad results.

##### Note

Ideally we catch all errors (that are not errors in the programmer's logic) at either compile-time or run-time. It is impossible to catch all errors at compile time and often not affordable to catch all remaining errors at run time. However, we should endeavor to write programs that in principle can be checked, given sufficient resources (analysis programs, run-time checks, machine resources, time).

##### Example, bad

    extern void f(int* p);  // separately compiled, possibly dynamically loaded

    void g(int n)
    {
        f(new int[n]);  // bad: the number of elements is not passed to f()
    }

Here, a crucial bit of information (the number of elements) has been so thoroughly "obscured" that static analysis is probably rendered infeasible and dynamic checking can be very difficult when `f()` is part of an ABI so that we cannot "instrument" that pointer. We could embed helpful information into the free store, but that requires global changes to a system and maybe to the compiler. What we have here is a design that makes error detection very hard.

##### Example, bad

We can of course pass the number of elements along with the pointer:

    extern void f2(int* p, int n);  // separately compiled, possibly dynamically loaded

    void g2(int n)
    {
        f2(new int[n], m);    // bad: the wrong number of elements can be passed to f()
    }

Passing the number of elements as an argument is better (and far more common) that just passing the pointer and relying on some (unstated) convention for knowing or discovering the number of elements. However (as shown), a simple typo can introduce a serious error. The connection between the two arguments of `f2()` is conventional, rather than explicit.

Also, it is implicit that `f2()` is supposed to `delete` its argument (or did the caller make a second mistake?).

##### Example, bad

The standard library resource management pointers fail to pass the size when they point to an object:

    extern void f3(unique_ptr<int[]>, int n);    // separately compiled, possibly dynamically loaded

    void g3(int n)
    {
        f3(make_unique<int[]>(n), m);    // bad: pass ownership and size separately
    }

##### Example

We need to pass the pointer and the number of elements as an integral object:

    extern void f4(vector<int>&);       // separately compiled, possibly dynamically loaded
    extern void f4(array_view<int>);    // separately compiled, possibly dynamically loaded

    void g3(int n)
    {
        vector<int> v(n);
        f4(v);                     // pass a reference, retain ownership
        f4(array_view<int>{v});    // pass a view, retain ownership
    }

This design carries the number of elements along as an integral part of an object, so that errors are unlikely and dynamic (run-time) checking is always feasible, if not always affordable.

##### Example

How do we transfer both ownership and all information needed for validating use?

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
* show how possible checks are avoided by interfaces that pass polymorphic base classes around, when they actually know what they need?
  Or strings as "free-style" options

##### Enforcement

* Flag (pointer, count)-style interfaces (this will flag a lot of examples that can't be fixed for compatibility reasons)
* ???

### <a name="Rp-early"></a> P.7: Catch run-time errors early

##### Reason

Avoid "mysterious" crashes.
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

Here we made a small error in `use1` that will lead to corrupted data or a crash.
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

Now, `m<=n` can be checked at the point of call (early) rather than later.
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

Don't repeatedly check the same value. Don't pass structured data as strings:

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

The date is validated twice (by the `Date` constructor) and passed as a character string (unstructured data).

##### Example

Excess checking can be costly.
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

The physical law for a jet (`e*e < x*x + y*y + z*z`) is not an invariant because of the possibility for measurement errors.

???

##### Enforcement

* Look at pointers and arrays: Do range-checking early
* Look at conversions: Eliminate or mark narrowing conversions
* Look for unchecked values coming from input
* Look for structured data (objects of classes with invariants) being converted into strings
* ???

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
