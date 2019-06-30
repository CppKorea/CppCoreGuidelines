# <a name="S-expr"></a>ES: 표현식과 구문

표현식(expression)과 구문(statement)은 행위와 연산에 대해 표현하는 가장 직접적인 방법들이다. 구문은 지역 유효범위 내에서 선언하는 것을 말한다.

이름 짓기와 주석, 들여쓰기 규칙에 대해서는, [NL: Naming and layout](./Naming.md#S-naming)을 참고하라.

일반적인 규칙:

* [ES.1: 다른 라이브러리나 "직접 짠 코드" 대신 표준 라이브러리를 사용하라](#Res-lib)
* [ES.2: 언어의 기능을 직접적으로 사용하기 보다는 적절한 추상화를 하라](#Res-abstr)

선언 규칙:

* [ES.5: 유효범위(scope)는 작게 유지하라](#Res-scope)
* [ES.6: for-구문에서 관련 변수들은 구문 범위 안에 두어라](#Res-cond)
* [ES.7: 일반적이거나 지역 변수들의 이름은 짧게, 그렇지 않다면 길게 하라](#Res-name-length)
* [ES.8: 유사해보이는 이름은 사용하지 마라](#Res-name-similar)
* [ES.9: 이름은 `ALL_CAPS` 형태가 아니어야 한다](#Res-not-CAPS)
* [ES.10: 선언은 (오직) 하나의 이름을 선언해야 한다](#Res-name-one)
* [ES.11: 타입 이름의 불필요한 반복을 막기 위해 `auto`를 사용하라](#Res-auto)
* [ES.12: 이름을 덮어쓰지 않게 하라](#Res-reuse)
* [ES.20: 항상 개체를 초기화하라](#Res-always)
* [ES.21: 사용할 필요가 없을 때 변수나 상수를 선언하지 마라](#Res-introduce)
* [ES.22: 변수를 초기화할 값이 생길 때까지 선언하지 마라](#Res-init)
* [ES.23: `{}`초기화 문법을 사용하라](#Res-list)
* [ES.24: Use a `unique_ptr<T>` to hold pointers](#Res-unique)
* [ES.25: 값을 변경하지 않는다면 개체를 `const` 혹은 `constexpr`로 선언하라](#Res-const)
* [ES.26: 서로 상관없는 목적에 하나의 변수를 사용하지 마라](#Res-recycle)
* [ES.27: `std::array`나 `stack_array`를 사용하라](#Res-stack)
* [ES.28: 복잡한 초기화, 특히 `const` 변수의 초기화에는 람다를 사용하라](#Res-lambda-init)
* [ES.30: 프로그램 텍스트를 바꾸기 위해 매크로를 사용하지 마라](#Res-macros)
* [ES.31: 매크로를 상수나 "함수"에 사용하지 마라](#Res-macros2)
* [ES.32: 모든 매크로는 `ALL_CAPS`형태로 선언하라](#Res-ALL_CAPS)
* [ES.33: 매크로를 사용해야만 한다면, 고유한 이름을 사용하라](#Res-MACROS)
* [ES.34: (C-스타일의) 가변인자 함수를 정의하지 마라](#Res-ellipses)

표현식 규칙:

* [ES.40: 복잡한 표현식을 피하라](#Res-complicated)
* [ES.41: 연산자 우선순위가 혼동된다면, 소괄호를 사용하라](#Res-parens)
* [ES.42: 포인터는 간단하고 직관적인 형태로 사용하라](#Res-ptr)
* [ES.43: Avoid expressions with undefined order of evaluation](#Res-order)
* [ES.44: 함수 인자가 표현식 평가 순서의 영향을 받지 않게 하라](#Res-order-fct)
* [ES.45: "만능 상수"를 피하자. 상징적인 상수를 사용하자](#Res-magic)
* [ES.46: 타입 범위를 축소하는 변환을 피하라](#Res-narrowing)
* [ES.47: `0` 혹은 `NULL`보다는 `nullptr`를 사용하라](#Res-nullptr)
* [ES.48: Avoid casts](#Res-casts)
* [ES.49: If you must use a cast, use a named cast](#Res-casts-named)
* [ES.50: `const`를 제거하지 마라](#Res-casts-const)
* [ES.55: 범위 검사가 필요하지 않도록 하라](#Res-range-checking)
* [ES.56: `std::move()`는 개체를 다른 유효범위로 명시적으로 옮겨야 할때만 사용하라](#Res-move)
* [ES.60: 자원을 관리하는 함수 외부에서 `new`와 `delete` 사용을 피하라](#Res-new)
* [ES.61: 배열은 delete[]`, 단일 개체는 `delete`를 사용해서 해제하라](#Res-del)
* [ES.62: Don't compare pointers into different arrays](#Res-arr2)
* [ES.63: Don't slice](#Res-slice)
* [ES.64: 개체 생성에는 `T{e}`을 사용하라](#Res-construct)
* [ES.65: Don't dereference an invalid pointer](#Res-deref)

구문 규칙:

* [ES.70: 선택을 하는 경우에는 `switch`-구문을 사용하라](#Res-switch-if)
* [ES.71: 가능하다면 범위기반 `for`-구문을 사용하라](#Res-for-range)
* [ES.72: 루프 변수가 있다면 `while`-구문보다 `for`-구문을 사용하라](#Res-for-while)
* [ES.73: 루프 변수가 없다면 `for`-구문보다 `while`-구문을 사용하라](#Res-while-for)
* [ES.74:  루프 변수는 `for`-구문의 초기화 부분에서 선언하라](#Res-for-init)
* [ES.75: `do`-구문을 사용하지 마라](#Res-do)
* [ES.76: `goto`를 사용하지 마라](#Res-goto)
* [ES.77: `break`와 `continue`의 사용을 최소화하라](#Res-continue)
* [ES.78: 내용이 있는 `case`는 `break`하라](#Res-break)
* [ES.79: 일반적인 경우를 처리하기 위해서 `default`를 사용하라](#Res-default)
* [ES.84: 이름이 없는 지역변수는 선언하지 마라](#Res-noname)
* [ES.85: Make empty statements visible](#Res-empty)
* [ES.86: for 루프에서 루프 변수를 변경하지 마라](#Res-loop-counter)
* [ES.87: 조건에 불필요한 `==`나 `!=`를 사용하지 마라](#Res-if)

산술연산 규칙:

* [ES.100: 부호가 있는 타입과 없는 타입을 함께 연산하지 마라](#Res-mix)
* [ES.101: 비트 조작시에는 부호가 없는(unsigned) 타입을 사용하라](#Res-unsigned)
* [ES.102: 연산에는 부호가 있는(signed) 타입을 사용하라](#Res-signed)
* [ES.103: Overflow가 발생하지 않게 하라](#Res-overflow)
* [ES.104: Underflow가 발생하지 않게 하라](#Res-underflow)
* [ES.105: 나눗셈의 제수(divisor)가 0이 되지 않게 하라](#Res-zero)
* [ES.106: `unsigned`로 음수값을 막으려 하지 마라](#Res-nonnegative)
* [ES.107: 배열 접근에는 `unsigned`보다는 `gsl::index`를 사용하라](#Res-subscripts)

### <a name="Res-lib"></a>ES.1: Prefer the standard library to other libraries and to "handcrafted code"

##### Reason

라이브러리를 사용하는 코드는 언어의 기능을 직접적으로 사용하는 것보다 쉽고, 더 짧게 작성할 수 있고, 고수준의 추상화가 가능하다.
ISO C++ 표준 라이브러리는 널리 알려져있으며 테스트가 잘된 라이브러리다.
모든 C++ 구현체에서 제공하고 있다.

##### Example

```c++
    auto sum = accumulate(begin(a), end(a), 0.0);   // good
```
`accumulate`의 범위 버전이 더 낫다:
```c++
    auto sum = accumulate(v, 0.0); // better
```
그런데 잘 알려진 알고리즘을 직접 만들지는 말자:
```c++
    int max = v.size();   // bad: verbose, purpose unstated
    double sum = 0.0;
    for (int i = 0; i < max; ++i)
        sum = sum + v[i];
```

##### Exception

표준라이브러리의 대다수가 동적 할당(자유 저장소)에 의존한다. 이런 부분은 알고리즘의 문제는 아닐지라도, 제한 시간 내에 응답성을 보장해야 하는 경우(hard-real-time)나 임베디드 환경에는 적합하지 않다.
그런 경우는 비슷한 기능을 구현하여 사용하는 것을 고려해볼 수 있다. 예를 들면 표준 라이브러리 스타일로 구현된 메모리 풀 할당 컨테이너 같은 것들이다.

##### Enforcement

Not easy. ??? Look for messy loops, nested loops, long functions, absence of function calls, lack of use of non-built-in types. Cyclomatic complexity?

### <a name="Res-abstr"></a>ES.2: Prefer suitable abstractions to direct use of language features

##### Reason

"적절한 추상화"(예를 들어 라이브러리나 클래스 같은 것)가 언어보다 어플리케이션의 개념에 더 가깝다. 코드를 짧고 명확하게 만들 수 있으며, 테스트하기에도 더 쉽다.

##### Example

```c++
    vector<string> read1(istream& is)   // good
    {
        vector<string> res;
        for (string s; is >> s;)
            res.push_back(s);
        return res;
    }
```
아래와 같은 전통적인 코드, 시스템 레벨과 거의 동등한 로우레벨 코드는 길고, 지저분하고, 이해하기도 어렵고, 느리게 돌아간다:
```c++
    char** read2(istream& is, int maxelem, int maxstring, int* nread)   // bad: verbose and incomplete
    {
        auto res = new char*[maxelem];
        int elemcount = 0;
        while (is && elemcount < maxelem) {
            auto s = new char[maxstring];
            is.read(s, maxstring);
            res[elemcount++] = s;
        }
        nread = &elemcount;
        return res;
    }
```
오버플로우나 에러 핸들링 코드가 일단 한 번 들어가게 되면, 코드는 확 지저분해진다. 그리고, 리턴하는 포인터와 배열로 구현되는 C 스타일의 문자열을 `delete`를 꼭 해줘야하는 문제도 있다.

##### Enforcement

Not easy. ??? Look for messy loops, nested loops, long functions, absence of function calls, lack of use of non-built-in types. Cyclomatic complexity?

## ES.dcl: 선언(Declarations)

선언은 구문(statement)이다. 한 선언은 임의의 유효 범위에 하나의 이름을 만들거나 이름있는 개체(named object)를 생성할 수 있다.

### <a name="Res-scope"></a>ES.5: Keep scopes small

##### Reason

가독성이 좋아진다. 리소스 점유를 최소화할 수 있다. 값의 잘못된 사용을 피할 수 있다.

**Alternative formulation**: 불필요하게 큰 스코프에 변수를 선언하지 마라

##### Example

```c++
    void use()
    {
        int i;    // bad: i is needlessly accessible after loop
        for (i = 0; i < 20; ++i) { /* ... */ }
        // no intended use of i here
        for (int i = 0; i < 20; ++i) { /* ... */ }  // good: i is local to for-loop

        if (auto pc = dynamic_cast<Circle*>(ps)) {  // good: pc is local to if-statement
            // ... deal with Circle ...
        }
        else {
            // ... handle error ...
        }
    }
```

##### Example, bad

```c++
    void use(const string& name)
    {
        string fn = name + ".txt";
        ifstream is {fn};
        Record r;
        is >> r;
        // ... 여기에는 fn과 is를 쓰면 안되는 200 줄짜리 코드가 들어간다 ...
    }
```
이 코드는 길다는 문제점이 있지만, `fn`의 값과 `is`가 갖고 있는 파일 핸들이 필요 이상으로 훨씬 길게 유지된다는 게 문제다.
이러면 함수의 뒷부분에서 `is`와 `fn`을 실수로 사용해버릴 수 있다.

이럴 때는, 분할해버리는 게 낫다:
```c++
    Record load_record(const string& name)
    {
        string fn = name + ".txt";
        ifstream is {fn};
        Record r;
        is >> r;
        return r;
    }

    void use(const string& name)
    {
        Record r = load_record(name);
        // ... 200 lines of code ...
    }
```

##### Enforcement

* 루프 바깥에서 루프 변수가 선언되고 이후에는 사용되지 않는 경우를 지적하라
* 파일 핸들이나 잠금과 같은 중요한 리소스를 사용하는 코드가 N  (이 값은 적당히 크다)줄 이상 계속되면 지적하라

### <a name="Res-cond"></a>ES.6: Declare names in for-statement initializers and conditions to limit scope

##### Reason

가독성. 시스템 자원 점유를 최소화한다.

##### Example

```c++
    void use()
    {
        for (string s; cin >> s;)
            v.push_back(s);

        for (int i = 0; i < 20; ++i) {   // good: i is local to for-loop
            // ...
        }

        if (auto pc = dynamic_cast<Circle*>(ps)) {   // good: pc is local to if-statement
            // ... deal with Circle ...
        }
        else {
            // ... handle error ...
        }
    }
```

##### Enforcement

* 루프 바깥에서 루프 변수가 선언되고 이후에는 사용되지 않을 때 지적하라
* (어려움) 루프 바깥에서 루프 변수를 선언하고, 루프가 끝난 뒤에 관계없는 목적으로 그 변수를 사용하는 경우 지적하라

##### C++17 example

Note: C++17 also adds `if` and `switch` initializer statements. These require C++17 support.
```c++
    map<int, string> mymap;

    if (auto result = mymap.insert(value); result.second) {
        // insert succeeded, and result is valid for this block
        use(result.first);  // ok
        // ...
    } // result is destroyed here
```

##### C++17 enforcement (if using a C++17 compiler)

* Flag selection/loop variables declared before the body and not used after the body
* (hard) Flag selection/loop variables declared before the body and used after the body for an unrelated purpose.

### <a name="Res-name-length"></a>ES.7: Keep common and local names short, and keep uncommon and nonlocal names longer

##### Reason

가독성. 관계없는 비-지역(non-local) 변수 간의 충돌 확률을 낮춘다.

##### Example

관습적으로 쓰이는 짧은 지역변수명은 가독성을 향상시킨다:
```c++
    template<typename T>    // good
    void print(ostream& os, const vector<T>& v)
    {
        for (gsl::index i = 0; i < v.size(); ++i)
            os << v[i] << '\n';
    }
```
인덱스는 관습적으로 `i`라고 쓰고, 이 일반 함수에는 벡터의 의미를 알만한 힌트가 없으므로, `v`가 어떤 경우에든지 맞는 이름이다.

비교:
```c++
    template<typename Element_type>   // bad: verbose, hard to read
    void print(ostream& target_stream, const vector<Element_type>& current_vector)
    {
        for (gsl::index current_element_index = 0;
             current_element_index < current_vector.size();
             ++current_element_index
        )
        target_stream << current_vector[current_element_index] << '\n';
    }
```
과장해서 표현하긴 했지만, 이것보다 더 심한 것도 본적이 있다.

##### Example

관습에 따르지 않는 짧은 비지역 변수는 코드를 모호하게 만든다:
```c++
    void use1(const string& s)
    {
        // ...
        tt(s);   // bad: what is tt()?
        // ...
    }
```
비지역 개체들에는 좀 더 가독성 있는 이름을 쓰면 나아진다:
```c++
    void use1(const string& s)
    {
        // ...
        trim_tail(s);   // better
        // ...
    }
```
이렇게 하면, 코드를 읽는 사람이 `trim_tail`의 의미를 알 수 있게 되고, 기억할 수 있게 된다.

##### Example, bad

내용이 긴 함수의 인자는 사실상 비지역 변수라고 볼 수 있다. 따라서 인자들의 이름은 적절한 의미를 담아야 한다:
```c++
    void complicated_algorithm(vector<Record>& vr, const vector<int>& vi, map<string, int>& out)
    // read from events in vr (marking used Records) for the indices in
    // vi placing (name, index) pairs into out
    {
        // ... 500 lines of code using vr, vi, and out ...
    }
```
함수는 짧게 유지하는 것을 권장하지만, 이 룰을 모두 적용시키긴 힘들 때가 있다. 그럴 경우엔 변수명을 적절히 줘야 한다.

##### Enforcement

지역 변수와 비지역 변수의 이름이 유지되는 범위의 길이를 확인한다. 동시에 함수의 길이를 함께 고려한다.

### <a name="Res-name-similar"></a>ES.8: Avoid similar-looking names

##### Reason

Code clarity and readability. Too-similar names slow down comprehension and increase the likelihood of error.

##### Example; bad

```c++
    if (readable(i1 + l1 + ol + o1 + o0 + ol + o1 + I0 + l0)) surprise();
```

##### Example; bad

Do not declare a non-type with the same name as a type in the same scope. This removes the need to disambiguate with a keyword such as `struct` or `enum`. It also removes a source of errors, as `struct X` can implicitly declare `X` if lookup fails.
```c++
    struct foo { int n; };
    struct foo foo();       // BAD, foo is a type already in scope
    struct foo x = foo();   // requires disambiguation
```

##### Exception

Antique header files might declare non-types and types with the same name in the same scope.

##### Enforcement

* Check names against a list of known confusing letter and digit combinations.
* Flag a declaration of a variable, function, or enumerator that hides a class or enumeration declared in the same scope.

### <a name="Res-not-CAPS"></a>ES.9: Avoid `ALL_CAPS` names

##### Reason

이런 이름은 보통 매크로를 정의할 때 사용한다. 따라서 `ALL_CAPS` 형태의 이름은 매크로와 충돌될 가능성이 많다. 

##### Example

```c++
    // somewhere in some header:
    #define NE !=

    // somewhere else in some other header:
    enum Coord { N, NE, NW, S, SE, SW, E, W };

    // somewhere third in some poor programmer's .cpp:
    switch (direction) {
    case N:
        // ...
    case NE:
        // ...
    // ...
    }
```

##### Note

단지 상수가 매크로처럼 쓰인다는 이유로 상수에 `ALL_CAPS` 형태의 이름을 사용하면 안된다

##### Enforcement

대문자만을 사용한 이름을 지적하라. 오래된 코드에 대해서는 매크로 이름으로 소문자를 섞어 사용한 경우를 지적하라

### <a name="Res-name-one"></a>ES.10: Declare one name (only) per declaration

##### Reason

한 줄에 선언 하나씩 하면 가독성을 향상시킬 수 있고, C/C++ 문법과 관련된 실수를 피할 수 있다. 
그리고 `//` 주석을 달 수 있는 공간이 생긴다.

##### Example, bad

```c++
    char *p, c, a[7], *pp[7], **aa[10];   // yuck!
```

##### Exception

함수 선언은 다수의 함수 인자 선언을 포함할 수 있다.

##### Exception

A structured binding (C++17) is specifically designed to introduce several variables:
```c++
    auto [iter, inserted] = m.insert_or_assign(k, val);
    if (inserted) { /* new entry was inserted */ }
```

##### Example

```c++
    template <class InputIterator, class Predicate>
    bool any_of(InputIterator first, InputIterator last, Predicate pred);
```
좀 더 나은 용례:
```c++
    bool any_of(InputIterator first, InputIterator last, Predicate pred);
```

##### Example

```c++
    double scalbn(double x, int n);   // OK: x * pow(FLT_RADIX, n); FLT_RADIX is usually 2
```
또는:
```c++
    double scalbn(    // better: x * pow(FLT_RADIX, n); FLT_RADIX is usually 2
        double x,     // base value
        int n         // exponent
    );
```
또는:
```c++
    // better: base * pow(FLT_RADIX, exponent); FLT_RADIX is usually 2
    double scalbn(double base, int exponent);
```

##### Example

```c++
    int a = 7, b = 9, c, d = 10, e = 3;
```
In a long list of declarators is is easy to overlook an uninitialized variable.

##### Enforcement

변수와 상수들을 한번에 선언을 한 곳을 지적한다.(예를 들어, `int* p, q;`)

### <a name="Res-auto"></a>ES.11: Use `auto` to avoid redundant repetition of type names

##### Reason

* Simple repetition is tedious and error-prone.
* When you use `auto`, the name of the declared entity is in a fixed position in the declaration, increasing readability.
* In a template function declaration the return type can be a member type.

##### Example

Consider:
```c++
    auto p = v.begin();   // vector<int>::iterator
    auto h = t.future();
    auto q = make_unique<int[]>(s);
    auto f = [](int x){ return x + 10; };
```
In each case, we save writing a longish, hard-to-remember type that the compiler already knows but a programmer could get wrong.

##### Example

```c++
    template<class T>
    auto Container<T>::first() -> Iterator;   // Container<T>::Iterator
```

##### Exception

Avoid `auto` for initializer lists and in cases where you know exactly which type you want and where an initializer might require conversion.

##### Example

```c++
    auto lst = { 1, 2, 3 };   // lst is an initializer list
    auto x{1};   // x is an int (in C++17; initializer_list in C++11)
```

##### Note

When concepts become available, we can (and should) be more specific about the type we are deducing:
```c++
    // ...
    ForwardIterator p = algo(x, y, z);
```

##### Example (C++17)

```c++
    auto [ quotient, remainder ] = div(123456, 73);   // break out the members of the div_t result
```

##### Enforcement

Flag redundant repetition of type names in a declaration.

### <a name="Res-reuse"></a>ES.12: Do not reuse names in nested scopes

##### Reason

It is easy to get confused about which variable is used.
Can cause maintenance problems.

##### Example, bad

```c++
    int d = 0;
    // ...
    if (cond) {
        // ...
        d = 9;
        // ...
    }
    else {
        // ...
        int d = 7;
        // ...
        d = value_to_be_returned;
        // ...
    }

    return d;
```
If this is a large `if`-statement, it is easy to overlook that a new `d` has been introduced in the inner scope.
This is a known source of bugs.
Sometimes such reuse of a name in an inner scope is called "shadowing".

##### Note

Shadowing is primarily a problem when functions are too large and too complex.

##### Example

Shadowing of function arguments in the outermost block is disallowed by the language:
```c++
    void f(int x)
    {
        int x = 4;  // error: reuse of function argument name

        if (x) {
            int x = 7;  // allowed, but bad
            // ...
        }
    }
```
##### Example, bad

Reuse of a member name as a local variable can also be a problem:
```c++
    struct S {
        int m;
        void f(int x);
    };

    void S::f(int x)
    {
        m = 7;    // assign to member
        if (x) {
            int m = 9;
            // ...
            m = 99; // assign to member
            // ...
        }
    }
```

##### Exception

We often reuse function names from a base class in a derived class:
```c++
    struct B {
        void f(int);
    };

    struct D : B {
        void f(double);
        using B::f;
    };
```
This is error-prone.
For example, had we forgotten the using declaration, a call `d.f(1)` would not have found the `int` version of `f`.

??? Do we need a specific rule about shadowing/hiding in class hierarchies?

##### Enforcement

* Flag reuse of a name in nested local scopes
* Flag reuse of a member name as a local variable in a member function
* Flag reuse of a global name as a local variable or a member name
* Flag reuse of a base class member name in a derived class (except for function names)

### <a name="Res-always"></a>ES.20: Always initialize an object

##### Reason

Avoid used-before-set errors and their associated undefined behavior.
Avoid problems with comprehension of complex initialization.
Simplify refactoring.

##### Example

```c++
    void use(int arg)
    {
        int i;   // bad: uninitialized variable
        // ...
        i = 7;   // initialize i
    }
```
No, `i = 7` does not initialize `i`; it assigns to it. Also, `i` can be read in the `...` part. Better:
```c++
    void use(int arg)   // OK
    {
        int i = 7;   // OK: initialized
        string s;    // OK: default initialized
        // ...
    }
```

##### Note

The *always initialize* rule is deliberately stronger than the *an object must be set before used* language rule.
The latter, more relaxed rule, catches the technical bugs, but:

* It leads to less readable code
* It encourages people to declare names in greater than necessary scopes
* It leads to harder to read code
* It leads to logic bugs by encouraging complex code
* It hampers refactoring

The *always initialize* rule is a style rule aimed to improve maintainability as well as a rule protecting against used-before-set errors.

##### Example

Here is an example that is often considered to demonstrate the need for a more relaxed rule for initialization
```c++
    widget i;    // "widget" a type that's expensive to initialize, possibly a large POD
    widget j;

    if (cond) {  // bad: i and j are initialized "late"
        i = f1();
        j = f2();
    }
    else {
        i = f3();
        j = f4();
    }
```
This cannot trivially be rewritten to initialize `i` and `j` with initializers.
Note that for types with a default constructor, attempting to postpone initialization simply leads to a default initialization followed by an assignment.
A popular reason for such examples is "efficiency", but a compiler that can detect whether we made a used-before-set error can also eliminate any redundant double initialization.

Assuming that there is a logical connection between `i` and `j`, that connection should probably be expressed in code:
```c++
    pair<widget, widget> make_related_widgets(bool x)
    {
        return (x) ? {f1(), f2()} : {f3(), f4() };
    }

    auto [i, j] = make_related_widgets(cond);    // C++17
```

##### Note

Complex initialization has been popular with clever programmers for decades.
It has also been a major source of errors and complexity.
Many such errors are introduced during maintenance years after the initial implementation.

##### Example

This rule covers member variables.
```c++
    class X {
    public:
        X(int i, int ci) : m2{i}, cm2{ci} {}
        // ...

    private:
        int m1 = 7;
        int m2;
        int m3;

        const int cm1 = 7;
        const int cm2;
        const int cm3;
    };
```
The compiler will flag the uninitialized `cm3` because it is a `const`, but it will not catch the lack of initialization of `m3`.
Usually, a rare spurious member initialization is worth the absence of errors from lack of initialization and often an optimizer
can eliminate a redundant initialization (e.g., an initialization that occurs immediately before an assignment).

##### Exception

If you are declaring an object that is just about to be initialized from input, initializing it would cause a double initialization.
However, beware that this may leave uninitialized data beyond the input -- and that has been a fertile source of errors and security breaches:
```c++
    constexpr int max = 8 * 1024;
    int buf[max];         // OK, but suspicious: uninitialized
    f.read(buf, max);
```
The cost of initializing that array could be significant in some situations.
However, such examples do tend to leave uninitialized variables accessible, so they should be treated with suspicion.
```c++
    constexpr int max = 8 * 1024;
    int buf[max] = {};   // zero all elements; better in some situations
    f.read(buf, max);
```
When feasible use a library function that is known not to overflow. For example:
```c++
    string s;   // s is default initialized to ""
    cin >> s;   // s expands to hold the string
```
Don't consider simple variables that are targets for input operations exceptions to this rule:
```c++
    int i;   // bad
    // ...
    cin >> i;
```
In the not uncommon case where the input target and the input operation get separated (as they should not) the possibility of used-before-set opens up.
```c++
    int i2 = 0;   // better, assuming that zero is an acceptable value for i2
    // ...
    cin >> i2;
```
A good optimizer should know about input operations and eliminate the redundant operation.

##### Example

Using a value representing "uninitialized" is a symptom of a problem and not a solution:
```c++
    widget i = uninit;  // bad
    widget j = uninit;

    // ...
    use(i);         // possibly used before set
    // ...

    if (cond) {     // bad: i and j are initialized "late"
        i = f1();
        j = f2();
    }
    else {
        i = f3();
        j = f4();
    }
```
Now the compiler cannot even simply detect a used-before-set. Further, we've introduced complexity in the state space for widget: which operations are valid on an `uninit` widget and which are not?

##### Note

Sometimes, a lambda can be used as an initializer to avoid an uninitialized variable:
```c++
    error_code ec;
    Value v = [&] {
        auto p = get_value();   // get_value() returns a pair<error_code, Value>
        ec = p.first;
        return p.second;
    }();
```
or maybe:
```c++
    Value v = [] {
        auto p = get_value();   // get_value() returns a pair<error_code, Value>
        if (p.first) throw Bad_value{p.first};
        return p.second;
    }();
```
**See also**: [ES.28](#Res-lambda-init)

##### Enforcement

* Flag every uninitialized variable.
  Don't flag variables of user-defined types with default constructors.
* Check that an uninitialized buffer is written into *immediately* after declaration.
  Passing an uninitialized variable as a reference to non-`const` argument can be assumed to be a write into the variable.

### <a name="Res-introduce"></a>ES.21: Don't introduce a variable (or constant) before you need to use it

##### Reason

Readability. To limit the scope in which the variable can be used.

##### Example

```c++
    int x = 7;
    // ... no use of x here ...
    ++x;
```
##### Enforcement

Flag declarations that are distant from their first use.

### <a name="Res-init"></a>ES.22: Don't declare a variable until you have a value to initialize it with

##### Reason

Readability. Limit the scope in which a variable can be used. Don't risk used-before-set. Initialization is often more efficient than assignment.

##### Example, bad

```c++
    string s;
    // ... no use of s here ...
    s = "what a waste";
```

##### Example, bad

```c++
    SomeLargeType var;   // ugly CaMeLcAsEvArIaBlE

    if (cond)   // some non-trivial condition
        Set(&var);
    else if (cond2 || !cond3) {
        var = Set2(3.14);
    }
    else {
        var = 0;
        for (auto& e : something)
            var += e;
    }

    // use var; that this isn't done too early can be enforced statically with only control flow
```
This would be fine if there was a default initialization for `SomeLargeType` that wasn't too expensive.
Otherwise, a programmer might very well wonder if every possible path through the maze of conditions has been covered.
If not, we have a "use before set" bug. This is a maintenance trap.

For initializers of moderate complexity, including for `const` variables, consider using a lambda to express the initializer; see [ES.28](#Res-lambda-init).

##### Enforcement

* Flag declarations with default initialization that are assigned to before they are first read.
* Flag any complicated computation after an uninitialized variable and before its use.

### <a name="Res-list"></a>ES.23: Prefer the `{}` initializer syntax

##### Reason

The rules for `{}` initialization are simpler, more general, less ambiguous, and safer than for other forms of initialization.

##### Example

```c++
    int x {f(99)};
    vector<int> v = {1, 2, 3, 4, 5, 6};
```

##### Exception

For containers, there is a tradition for using `{...}` for a list of elements and `(...)` for sizes:
```c++
    vector<int> v1(10);    // vector of 10 elements with the default value 0
    vector<int> v2 {10};   // vector of 1 element with the value 10
```

##### Note

`{}`-initializers do not allow narrowing conversions.

##### Example

```c++
    int x {7.9};   // error: narrowing
    int y = 7.9;   // OK: y becomes 7. Hope for a compiler warning
```

##### Note

`{}` initialization can be used for all initialization; other forms of initialization can't:
```c++
    auto p = new vector<int> {1, 2, 3, 4, 5};   // initialized vector
    D::D(int a, int b) :m{a, b} {   // member initializer (e.g., m might be a pair)
        // ...
    };
    X var {};   // initialize var to be empty
    struct S {
        int m {7};   // default initializer for a member
        // ...
    };
```

##### Note

Initialization of a variable declared using `auto` with a single value, e.g., `{v}`, had surprising results until C++17.
The C++17 rules are somewhat less surprising:
```c++
    auto x1 {7};        // x1 is an int with the value 7
    auto x2 = {7};  // x2 is an initializer_list<int> with an element 7

    auto x11 {7, 8};    // error: two initializers
    auto x22 = {7, 8};  // x2 is an initializer_list<int> with elements 7 and 8
```
So use `={...}` if you really want an `initializer_list<T>`
```c++
    auto fib10 = {1, 1, 2, 3, 5, 8, 13, 21, 34, 55};   // fib10 is a list
```

##### Note

Old habits die hard, so this rule is hard to apply consistently, especially as there are so many cases where `=` is innocent.

##### Example

```c++
    template<typename T>
    void f()
    {
        T x1(1);    // T initialized with 1
        T x0();     // bad: function declaration (often a mistake)

        T y1 {1};   // T initialized with 1
        T y0 {};    // default initialized T
        // ...
    }
```
**See also**: [Discussion](#???)

##### Enforcement

Tricky.

* Don't flag uses of `=` for simple initializers.
* Look for `=` after `auto` has been seen.

### <a name="Res-unique"></a>ES.24: Use a `unique_ptr<T>` to hold pointers

##### Reason

Using `std::unique_ptr` is the simplest way to avoid leaks. It is reliable, it
makes the type system do much of the work to validate ownership safety, it
increases readability, and it has zero or near zero run-time cost.

##### Example

```c++
    void use(bool leak)
    {
        auto p1 = make_unique<int>(7);   // OK
        int* p2 = new int{7};            // bad: might leak
        // ... no assignment to p2 ...
        if (leak) return;
        // ... no assignment to p2 ...
        vector<int> v(7);
        v.at(7) = 0;                    // exception thrown
        // ...
    }
```
If `leak == true` the object pointed to by `p2` is leaked and the object pointed to by `p1` is not.
The same is the case when `at()` throws.

##### Enforcement

Look for raw pointers that are targets of `new`, `malloc()`, or functions that may return such pointers.

### <a name="Res-const"></a>ES.25: Declare an object `const` or `constexpr` unless you want to modify its value later on

##### Reason

실수로 값을 바꾸는 걸 막을 수 있는 방법이다. 컴파일러에게 최적화를 위한 기회를 줄 수도 있다.

##### Example

```c++
    void f(int n)
    {
        const int bufmax = 2 * n + 2;  // good: we can't change bufmax by accident
        int xmax = n;                  // suspicious: is xmax intended to change?
        // ...
    }
```

##### Enforcement

변수가 실제로 값이 바뀌는지 안 바뀌는지 보고 바뀐다면 지적한다.
불행하게도, `const`가 아닌 개체가 값을 바꾸려 *의도*했는지 찾아내는 것은 불가능하다.

### <a name="Res-recycle"></a>ES.26: Don't use a variable for two unrelated purposes

##### Reason

가독성과 안전성

##### Example, bad

```c++
    void use()
    {
        int i;
        for (i = 0; i < 20; ++i) { /* ... */ }
        for (i = 0; i < 200; ++i) { /* ... */ } // bad: i recycled
    }
```

##### Note

As an optimization, you may want to reuse a buffer as a scratch pad, but even then prefer to limit the variable's scope as much as possible and be careful not to cause bugs from data left in a recycled buffer as this is a common source of security bugs.
```c++
    void write_to_file() {
        std::string buffer;             // to avoid reallocations on every loop iteration
        for (auto& o : objects)
        {
            // First part of the work.
            generate_first_String(buffer, o);
            write_to_file(buffer);

            // Second part of the work.
            generate_second_string(buffer, o);
            write_to_file(buffer);

            // etc...
        }
    }
```

##### Enforcement

재활용되는 변수가 있다면 지적한다.

### <a name="Res-stack"></a>ES.27: Use `std::array` or `stack_array` for arrays on the stack

##### Reason

가독성이 높아지고, 묵시적으로 포인터로 바뀌지 않는다.
언어가 지원하는 배열의 비표준적인 확장과 헷갈리지 않는다.

##### Example, bad

```c++
    const int n = 7;
    int m = 9;

    void f()
    {
        int a1[n];
        int a2[m];   // error: not ISO C++
        // ...
    }
```

##### Note

`a1` 변수선언은 C++에서는 적법하다. 그런 류의 코드가 많이 있다.
다만 이는 길이값이 비지역 변수일 대 잘못 사용하기 쉽다. 버퍼 오버플로우, 배열을 포인터로 변환하는 등의 "유명한" 오류 원인이 된다.

`a2` 변수선언은 C 방식으로 C++ 에서는 쓰지 않으며 보안상 문제가 있는 것으로 간주한다.

##### Example

```c++
    const int n = 7;
    int m = 9;

    void f()
    {
        array<int, n> a1;
        stack_array<int> a2(m);
        // ...
    }
```

##### Enforcement

* 상수 길이를 가지지 않는 배열이라면 표시한다. (C 언어의 가변길이배열(VLA))
* 배열 길이로 지역 상수를 사용하지 않으면 지적한다

### <a name="Res-lambda-init"></a>ES.28: Use lambdas for complex initialization, especially of `const` variables

##### Reason

멋지게 지역 초기화를 숨길 수 있다.
초기화 작업을 위해서만 필요한 변수를 포함해서 재사용할 것 같지 않은 함수를 생성할 필요도 없다.

약간의 초기화 작업 후에 `const`여야 하는 변수에도 사용할 수 있다.

##### Example, bad

```c++
    widget x;   // should be const, but:
    for (auto i = 2; i <= N; ++i) {          // this could be some
        x += some_obj.do_something_with(i);  // arbitrarily long code
    }                                        // needed to initialize x
    // from here, x should be const, but we can't say so in code in this style
```

##### Example, good

```c++
    const widget x = [&]{
        widget val;                                // assume that widget has a default constructor
        for (auto i = 2; i <= N; ++i) {            // this could be some
            val += some_obj.do_something_with(i);  // arbitrarily long code
        }                                          // needed to initialize x
        return val;
    }();
```

##### Example

```c++
    string var = [&]{
        if (!in) return "";   // default
        string s;
        for (char c : in >> c)
            s += toupper(c);
        return s;
    }(); // note ()
```
가능하다면 `enum`같은 쉬운 방법으로 조건을 줄여라. 분기 선택과 초기화를 뒤섞어선 안된다.

##### Enforcement

어렵다. 잘 해도 경험적인(heuristic) 수준. 
루프를 사용해 값을 설정하는 초기화 안된 변수을 찾아라.

### <a name="Res-macros"></a>ES.30: Don't use macros for program text manipulation

##### Reason

매크로는 버그의 주요 원인이다.
매크로는 일반적인 범위와 타입 규칙을 따르지 않는다.
매크로는 사람이 보는 것과 컴파일러가 보는 것을 다르게 한다.
매크로는 지원 도구를 만드는 것을 복잡하게 한다.

##### Example, bad

```c++
    #define Case break; case   /* BAD */
```
This innocuous-looking macro makes a single lower case `c` instead of a `C` into a bad flow-control bug.

##### Note

이 규칙은 `#ifdef`문에서 설정제어를 위해 매크로를 사용하는 것은 금하지 않는다.

In the future, modules are likely to eliminate the need for macros in configuration control.

##### Note

This rule is meant to also discourage use of `#` for stringification and `##` for concatenation.
As usual for macros, there are uses that are "mostly harmless", but even these can create problems for tools,
such as auto completers, static analyzers, and debuggers.
Often the desire to use fancy macros is a sign of an overly complex design.
Also, `#` and `##` encourages the definition and use of macros:
```c++
    #define CAT(a, b) a ## b
    #define STRINGIFY(a) #a

    void f(int x, int y)
    {
        string CAT(x, y) = "asdf";   // BAD: hard for tools to handle (and ugly)
        string sx2 = STRINGIFY(x);
        // ...
    }
```
There are workarounds for low-level string manipulation using macros. For example:
```c++
    string s = "asdf" "lkjh";   // ordinary string literal concatenation

    enum E { a, b };

    template<int x>
    constexpr const char* stringify()
    {
        switch (x) {
        case a: return "a";
        case b: return "b";
        }
    }

    void f(int x, int y)
    {
        string sx = stringify<x>();
        // ...
    }
```
This is not as convenient as a macro to define, but as easy to use, has zero overhead, and is typed and scoped.

In the future, static reflection is likely to eliminate the last needs for the preprocessor for program text manipulation.

##### Enforcement

소스제어(`#ifdef`같은)에 사용하지 않는 매크로를 본다면 소리를 질러라.

### <a name="Res-macros2"></a>ES.31: Don't use macros for constants or "functions"

##### Reason

매크로는 버그의 주요 원인이다.  
매크로는 일반적인 범위와 타입 규칙을 따르지 않는다.  
매크로는 사람이 보는 것과 컴파일러가 보는 것을 다르게 한다.  
매크로는 지원 도구를 만드는 것을 복잡하게 한다.  

##### Example, bad

```c++
    #define PI 3.14
    #define SQUARE(a, b) (a * b)
```
`SQUARE`에 잘 알려진 버그가 없다고 하더라도 더 잘 동작하는 대안이 있다.

예를 들면:
```c++
    constexpr double pi = 3.14;
    template<typename T> T square(T a, T b) { return a * b; }
```

##### Enforcement

소스제어(`#ifdef`같은)에 사용하지 않는 매크로를 본다면 소리를 질러라.

### <a name="Res-ALL_CAPS"></a>ES.32: Use `ALL_CAPS` for all macro names

##### Reason

관습. 가독성. 매크로 구별.

##### Example

```c++
    #define forever for (;;)   /* very BAD */

    #define FOREVER for (;;)   /* Still evil, but at least visible to humans */
```

##### Enforcement

소문자로 작성된 매크로를 본다면 소리를 질러라.

### <a name="Res-MACROS"></a>ES.33: If you must use macros, give them unique names

##### Reason

Macros do not obey scope rules.

##### Example

```c++
    #define MYCHAR        /* BAD, will eventually clash with someone else's MYCHAR*/

    #define ZCORP_CHAR    /* Still evil, but less likely to clash */
```

##### Note

Avoid macros if you can: [ES.30](#Res-macros), [ES.31](#Res-macros2), and [ES.32](#Res-ALL_CAPS).
However, there are billions of lines of code littered with macros and a long tradition for using and overusing macros.
If you are forced to use macros, use long names and supposedly unique prefixes (e.g., your organization's name) to lower the likelihood of a clash.

##### Enforcement

Warn against short macro names.

### <a name="Res-ellipses"></a> ES.34: Don't define a (C-style) variadic function

##### Reason

Not type safe.
Requires messy cast-and-macro-laden code to get working right.

##### Example

```c++
    #include <cstdarg>

    // "severity" followed by a zero-terminated list of char*s; write the C-style strings to cerr
    void error(int severity ...)
    {
        va_list ap;             // a magic type for holding arguments
        va_start(ap, severity); // arg startup: "severity" is the first argument of error()

        for (;;) {
            // treat the next var as a char*; no checking: a cast in disguise
            char* p = va_arg(ap, char*);
            if (!p) break;
            cerr << p << ' ';
        }

        va_end(ap);             // arg cleanup (don't forget this)

        cerr << '\n';
        if (severity) exit(severity);
    }

    void use()
    {
        error(7, "this", "is", "an", "error", nullptr);
        error(7); // crash
        error(7, "this", "is", "an", "error");  // crash
        const char* is = "is";
        string an = "an";
        error(7, "this", "is", an, "error"); // crash
    }
```
**Alternative**: Overloading. Templates. Variadic templates.
```c++
    #include <iostream>

    void error(int severity)
    {
        std::cerr << '\n';
        std::exit(severity);
    }

    template <typename T, typename... Ts>
    constexpr void error(int severity, T head, Ts... tail)
    {
        std::cerr << head;
        error(severity, tail...);
    }

    void use()
    {
        error(7); // No crash!
        error(5, "this", "is", "not", "an", "error"); // No crash!

        std::string an = "an";
        error(7, "this", "is", "not", an, "error"); // No crash!

        error(5, "oh", "no", nullptr); // Compile error! No need for nullptr.
    }
```

##### Note

This is basically the way `printf` is implemented.

##### Enforcement

* Flag definitions of C-style variadic functions.
* Flag `#include <cstdarg>` and `#include <stdarg.h>`

## ES.expr: Expressions

Expressions manipulate values.

### <a name="Res-complicated"></a>ES.40: Avoid complicated expressions

##### Reason

표현식이 복잡하면 오류가 발생하기 쉽다.

##### Example

```c++
    // bad: assignment hidden in subexpression
    while ((c = getc()) != -1)

    // bad: two non-local variables assigned in a sub-expressions
    while ((cin >> c1, cin >> c2), c1 == c2)

    // better, but possibly still too complicated
    for (char c1, c2; cin >> c1 >> c2 && c1 == c2;)

    // OK: if i and j are not aliased
    int x = ++i + ++j;

    // OK: if i != j and i != k
    v[i] = v[j] + v[k];

    // bad: multiple assignments "hidden" in subexpressions
    x = a + (b = f()) + (c = g()) * 7;

    // bad: relies on commonly misunderstood precedence rules
    x = a & b + c * d && e ^ f == 7;

    // bad: undefined behavior
    x = x++ + x++ + ++x;
```
위의 연산식 중 몇은 의심할 여지없이 나쁘다. (정의되지 않은 행동을 야기한다.)
나머지는 꽤 복잡하거나 특이한 편이고, 심지어 능력있는 프로그래머도 잘못 이해하거나 문제를 간과해 버릴 만한 것도 있다.

##### Note

C++17 tightens up the rules for the order of evaluation
(left-to-right except right-to-left in assignments, and the order of evaluation of function arguments is unspecified; [see ES.43](#Res-order)),
but that doesn't change the fact that complicated expressions are potentially confusing.

##### Note

A programmer should know and use the basic rules for expressions.

##### Example

```c++
    x = k * y + z;             // OK

    auto t1 = k * y;           // bad: unnecessarily verbose
    x = t1 + z;

    if (0 <= x && x < max)   // OK

    auto t1 = 0 <= x;        // bad: unnecessarily verbose
    auto t2 = x < max;
    if (t1 && t2)            // ...
```

##### Enforcement

까다롭다. "표현식이 얼마나 복잡한가"를 어떻게 판단할 것인가? 어떻게 고려할 것인가? 
계산을 하나의 연산으로만 구성된 문장들로 구성하기는 힘들다.

고려할만한 것들:
* 부수 효과(side-effect): 다수의 비지역 변수에 대한 부수 효과을 의심할 수 있다. 특히 별도의 하위 연산식에 있는 경우
* writes to aliased variables
* N개 이상의 연산자 (N은 얼마가 되어야 하는가?)
* 미묘한 우선순위규칙에 의존하기
* 미정의 행동 (undefined behavior: 모든 미정의 행동을 잡아낼 수 있는가?)
* implementation defined behavior?
* ???

### <a name="Res-parens"></a>ES.41: If in doubt about operator precedence, parenthesize

##### Reason

오류가 발생하지 않게 하라. 가독성. 모든 사람들이 연산자 우선순위를 기억하지는 않는다.

##### Example

```c++
    const unsigned int flag = 2;
    unsigned int a = flag;

    if (a & flag != 0)  // bad: means a&(flag != 0)
```
Note: 프로그래머는 산술 연산, 논리 연산에 대해서 우선순위 테이블을 알고 있고, 다른 연산과 비트 연산을 섞어 사용할 때는 소괄호(parentheses)를 사용하기를 권한다.
```c++
    if ((a & flag) != 0)  // OK: works as intended
```

##### Note

아래에 대해서는 소괄호가 필요없다는 정도는 알고 있을 것이다:
```c++
    if (a < 0 || a <= max) {
        // ...
    }
```

##### Enforcement

* 비트 논리 연산자와 다른 연산자가 섞여 있다면 지적한다
* 가장 왼쪽의 연산자(leftmost operator)가 할당 연산자가 아니라면 지적한다
* ???

### <a name="Res-ptr"></a>ES.42: Keep use of pointers simple and straightforward

##### Reason

복잡한 포인터 계산은 주요한 오류 원인이 된다.

##### Note

Use `gsl::span` instead.
Pointers should [only refer to single objects](#Ri-array).
Pointer arithmetic is fragile and easy to get wrong, the source of many, many bad bugs and security violations.
`span` is a bounds-checked, safe type for accessing arrays of data.
Access into an array with known bounds using a constant as a subscript can be validated by the compiler.

##### Example, bad

```c++
    void f(int* p, int count)
    {
        if (count < 2) return;

        int* q = p + 1;    // BAD

        ptrdiff_t d;
        int n;
        d = (p - &n);      // OK
        d = (q - p);       // OK

        int n = *p++;      // BAD

        if (count < 6) return;

        p[4] = 1;          // BAD

        p[count - 1] = 2;  // BAD

        use(&p[0], 3);     // BAD
    }
```

##### Example, good

```c++
    void f(span<int> a) // BETTER: use span in the function declaration
    {
        if (a.size() < 2) return;

        int n = a[0];      // OK

        span<int> q = a.subspan(1); // OK

        if (a.size() < 6) return;

        a[4] = 1;          // OK

        a[a.size() - 1] = 2;  // OK

        use(a.data(), 3);  // OK
    }
```

##### Note

Subscripting with a variable is difficult for both tools and humans to validate as safe.
`span` is a run-time bounds-checked, safe type for accessing arrays of data.
`at()` is another alternative that ensures single accesses are bounds-checked.
If iterators are needed to access an array, use the iterators from a `span` constructed over the array.

##### Example, bad

```c++
    void f(array<int, 10> a, int pos)
    {
        a[pos / 2] = 1; // BAD
        a[pos - 1] = 2; // BAD
        a[-1] = 3;    // BAD (but easily caught by tools) -- no replacement, just don't do this
        a[10] = 4;    // BAD (but easily caught by tools) -- no replacement, just don't do this
    }
```

##### Example, good

Use a `span`:
```c++
    void f1(span<int, 10> a, int pos) // A1: Change parameter type to use span
    {
        a[pos / 2] = 1; // OK
        a[pos - 1] = 2; // OK
    }

    void f2(array<int, 10> arr, int pos) // A2: Add local span and use that
    {
        span<int> a = {arr, pos};
        a[pos / 2] = 1; // OK
        a[pos - 1] = 2; // OK
    }
```
Use a `at()`:
```c++
    void f3(array<int, 10> a, int pos) // ALTERNATIVE B: Use at() for access
    {
        at(a, pos / 2) = 1; // OK
        at(a, pos - 1) = 2; // OK
    }
```

##### Example, bad

```c++
    void f()
    {
        int arr[COUNT];
        for (int i = 0; i < COUNT; ++i)
            arr[i] = i; // BAD, cannot use non-constant indexer
    }
```

##### Example, good

Use a `span`:
```c++
    void f1()
    {
        int arr[COUNT];
        span<int> av = arr;
        for (int i = 0; i < COUNT; ++i)
            av[i] = i;
    }
```
Use a `span` and range-`for`:
```c++
    void f1a()
    {
         int arr[COUNT];
         span<int, COUNT> av = arr;
         int i = 0;
         for (auto& e : av)
             e = i++;
    }
```
Use `at()` for access:
```c++
    void f2()
    {
        int arr[COUNT];
        for (int i = 0; i < COUNT; ++i)
            at(arr, i) = i;
    }
```
Use a range-`for`:
```c++
    void f3()
    {
        int arr[COUNT];
        for (auto& e : arr)
             e = i++;
    }
```

##### Note

Tooling can offer rewrites of array accesses that involve dynamic index expressions to use `at()` instead:
```c++
    static int a[10];

    void f(int i, int j)
    {
        a[i + j] = 12;      // BAD, could be rewritten as ...
        at(a, i + j) = 12;  // OK -- bounds-checked
    }
```

##### Example

Turning an array into a pointer (as the language does essentially always) removes opportunities for checking, so avoid it
```c++
    void g(int* p);

    void f()
    {
        int a[5];
        g(a);        // BAD: are we trying to pass an array?
        g(&a[0]);    // OK: passing one object
    }
```
If you want to pass an array, say so:
```c++
    void g(int* p, size_t length);  // old (dangerous) code

    void g1(span<int> av); // BETTER: get g() changed.

    void f2()
    {
        int a[5];
        span<int> av = a;

        g(av.data(), av.size());   // OK, if you have no choice
        g1(a);                     // OK -- no decay here, instead use implicit span ctor
    }
```

##### Enforcement

* Flag any arithmetic operation on an expression of pointer type that results in a value of pointer type.
* Flag any indexing expression on an expression or variable of array type (either static array or `std::array`) where the indexer is not a compile-time constant expression with a value between `0` or and the upper bound of the array.
* Flag any expression that would rely on implicit conversion of an array type to a pointer type.

This rule is part of the [bounds-safety profile](#SS-bounds).

### <a name="Res-order"></a>ES.43: Avoid expressions with undefined order of evaluation

##### Reason

그런 코드가 어떻게 동작할지는 알 수가 없다. 이식성.
특정한 환경에는 맞을지는 몰라도, 다른 컴파일러 (혹은 사용 중인 컴파일러의 다음 버전)에서는 다를 수 있다. 혹은 최적화 설정에 따라 다를 수도 있다.

##### Note

C++17 tightens up the rules for the order of evaluation:
left-to-right except right-to-left in assignments, and the order of evaluation of function arguments is unspecified.

However, remember that your code may be compiled with a pre-C++17 compiler (e.g., through cut-and-paste) so don't be too clever.

##### Example

```c++
    v[i] = ++i;   //  the result is undefined
```
A good rule of thumb is that you should not read a value twice in an expression where you write to it.

##### Enforcement

좋은 분석기를 사용해 찾을 수 있다.

### <a name="Res-order-fct"></a>ES.44: Don't depend on order of evaluation of function arguments

##### Reason

순서가 정의되어있지 않다.

##### Note

C++17 tightens up the rules for the order of evaluation, but the order of evaluation of function arguments is still unspecified.

##### Example

```c++
    int i = 0;
    f(++i, ++i);
```
The call will most likely be `f(0, 1)` or `f(1, 0)`, but you don't know which.
Technically, the behavior is undefined.
In C++17, this code does not have undefined behavior, but it is still not specified which argument is evaluated first.

##### Example

Overloaded operators can lead to order of evaluation problems:
```c++
    f1()->m(f2());          // m(f1(), f2())
    cout << f1() << f2();   // operator<<(operator<<(cout, f1()), f2())
```
In C++17, these examples work as expected (left to right) and assignments are evaluated right to left (just as ='s binding is right-to-left)
```c++
    f1() = f2();    // undefined behavior in C++14; in C++17, f2() is evaluated before f1()
```

##### Enforcement

좋은 분석기를 사용해 찾을 수 있다.

### <a name="Res-magic"></a>ES.45: Avoid "magic constants"; use symbolic constants

##### Reason

표현식에 포함된 이름없는 상수는 간과되기 쉽고 이해하기 어렵다:

##### Example

```c++
    for (int m = 1; m <= 12; ++m)   // don't: magic constant 12
        cout << month[m] << '\n';
```
1년에 12달이 숫자로만 되어 있다면 이해가 잘 안될 것이다. 

더 좋게 고치면:
```c++
    // months are indexed 1..12
    constexpr int first_month = 1;
    constexpr int last_month = 12;

    for (int m = first_month; m <= last_month; ++m)   // better
        cout << month[m] << '\n';
```
아예 상수를 사용하지 않으면 더 낫다:
```c++
    for (auto m : month)
        cout << m << '\n';
```

##### Enforcement

코드에 리터럴이 있다면 지적한다. `0`, `1`, `nullptr`, `\n`, `""` 등 가능한 목록은 허용하라.

### <a name="Res-narrowing"></a>ES.46: Avoid lossy (narrowing, truncating) arithmetic conversions

##### Reason

정보를 파괴하고 전혀 기대하지 않는 값을 가지게 한다.

##### Example, bad

기본적인 예제:
```c++
    double d = 7.9;
    int i = d;    // bad: narrowing: i becomes 7
    i = (int) d;  // bad: we're going to claim this is still not explicit enough

    void f(int x, long y, double d)
    {
        char c1 = x;   // bad: narrowing
        char c2 = y;   // bad: narrowing
        char c3 = d;   // bad: narrowing
    }
```

##### Note

gsl은 narrowing을 허용하는 `narrow_cast`와 변환시 값이 바뀌면 예외를 던지는 `narrow`("narrow if")를 제공한다:
```c++
    i = narrow_cast<int>(d);   // OK (you asked for it): narrowing: i becomes 7
    i = narrow<int>(d);        // OK: throws narrowing_error
```
이 규칙은 부동 소수점 타입의 음수를 부호 없는 정수타입으로 변환하는 등의 손실있는 형변환까지도 포함한다:
```c++
    double d = -7.9;
    unsigned u = 0;

    u = d;                          // BAD
    u = narrow_cast<unsigned>(d);   // OK (you asked for it): u becomes 0
    u = narrow<unsigned>(d);        // OK: throws narrowing_error
```

##### Enforcement

A good analyzer can detect all narrowing conversions. However, flagging all narrowing conversions will lead to a lot of false positives. Suggestions:

* flag all floating-point to integer conversions (maybe only `float`->`char` and `double`->`int`. Here be dragons! we need data)
* flag all `long`->`char` (I suspect `int`->`char` is very common. Here be dragons! we need data)
* consider narrowing conversions for function arguments especially suspect

### <a name="Res-nullptr"></a>ES.47: Use `nullptr` rather than `0` or `NULL`

##### Reason

가독성의 문제다. 기대를 벗어나지 않게 한다.

`nullptr`는 `int`와 혼동의 여지가 없다. `nullptr` also has a well-specified (very restrictive) type, and thus
works in more scenarios where type deduction might do the wrong thing on `NULL`
or `0`.

##### Example

참고하라:
```c++
    void f(int);
    void f(char*);
    f(0);         // call f(int)
    f(nullptr);   // call f(char*)
```

##### Enforcement

포인터에 `0`, `NULL`을 사용한다면 지적한다. 프로그램으로 간단하게 변환할 수 있으면 도움이 될 것이다.

### <a name="Res-casts"></a>ES.48: Avoid casts

##### Reason

잘 알려진 오류의 원인이다. 최적화를 신뢰할 수 없게 만들어 버린다.

##### Example, bad

```c++
    double d = 2;
    auto p = (long*)&d;
    auto q = (long long*)&d;
    cout << d << ' ' << *p << ' ' << *q << '\n';
```
What would you think this fragment prints? The result is at best implementation defined. I got
```
    2 0 4611686018427387904
```
Adding
```c++
    *q = 666;
    cout << d << ' ' << *p << ' ' << *q << '\n';
```
I got
```
    3.29048e-321 666 666
```
Surprised? I'm just glad I didn't crash the program.

##### Note

Programmers who write casts typically assume that they know what they are doing,
or that writing a cast makes the program "easier to read".
In fact, they often disable the general rules for using values.
Overload resolution and template instantiation usually pick the right function if there is a right function to pick.
If there is not, maybe there ought to be, rather than applying a local fix (cast).

##### Note

형변환은 시스템 프로그래밍 언어에 꼭 필요하다.
예를 들어, 디바이스 레지스터의 주소를 포인터로 얻어 올 때이다.
그러나 너무 남용하는 바람에 많은 오류가 발생하는 것도 사실이다.

##### Note

형변환을 너무 많이 쓴다고 생각된다면 근본적인 설계 문제가 있을지도 모른다.

##### Exception

Casting to `(void)` is the Standard-sanctioned way to turn off `[[nodiscard]]` warnings. If you are calling a function with a `[[nodiscard]]` return and you deliberately want to discard the result, first think hard about whether that is really a good idea (there is usually a good reason the author of the function or of the return type used `[[nodiscard]]` in the first place), but if you still think it's appropriate and your code reviewer agrees, write `(void)` to turn off the warning.

##### Alternatives

Casts are widely (mis) used. Modern C++ has rules and constructs that eliminate the need for casts in many contexts, such as

* Use templates
* Use `std::variant`
* Rely on the well-defined, safe, implicit conversions between pointer types

##### Enforcement

* Force the elimination of C-style casts, except on a function with a `[[nodiscard]]` return
* Warn if there are many functional style casts (there is an obvious problem in quantifying 'many')
* The [type profile](#Pro-type-reinterpretcast) bans `reinterpret_cast`.
* Warn against [identity casts](#Pro-type-identitycast) between pointer types, where the source and target types are the same (#Pro-type-identitycast)
* Warn if a pointer cast could be [implicit](#Pro-type-implicitpointercast)

### <a name="Res-casts-named"></a>ES.49: If you must use a cast, use a named cast

##### Reason

가독성. 오류 예방.

Readability. Error avoidance.
Named cast들은 C스타일이나 함수형 형변환보다 더 구체적이며, 컴파일러가 일부 오류를 잡아낼 수 있도록 한다.

> 역주:
> * C 스타일 형변환: `(int) a`
> * 함수형 형변환: `int(a)`

The named casts are:

* `static_cast`
* `const_cast`
* `reinterpret_cast`
* `dynamic_cast`
* `std::move`         // `move(x)` is an rvalue reference to `x`
* `std::forward`      // `forward(x)` is an rvalue reference to `x`
* `gsl::narrow_cast`  // `narrow_cast<T>(x)` is `static_cast<T>(x)`
* `gsl::narrow`       // `narrow<T>(x)` is `static_cast<T>(x)` if `static_cast<T>(x) == x` or it throws `narrowing_error`

##### Example

```c++
    class B { /* ... */ };
    class D { /* ... */ };

    template<typename D> D* upcast(B* pb)
    {
        D* pd0 = pb;                        // error: no implicit conversion from B* to D*
        D* pd1 = (D*)pb;                    // legal, but what is done?
        D* pd2 = static_cast<D*>(pb);       // error: D is not derived from B
        D* pd3 = reinterpret_cast<D*>(pb);  // OK: on your head be it!
        D* pd4 = dynamic_cast<D*>(pb);      // OK: return nullptr
        // ...
    }
```
The example was synthesized from real-world bugs where `D` used to be derived from `B`, but someone refactored the hierarchy.
The C-style cast is dangerous because it can do any kind of conversion, depriving us of any protection from mistakes (now or in the future).

##### Note

When converting between types with no information loss (e.g. from `float` to
`double` or `int64` from `int32`), brace initialization may be used instead.
```c++
    double d {some_float};
    int64_t i {some_int32};
```
This makes it clear that the type conversion was intended and also prevents
conversions between types that might result in loss of precision. (It is a
compilation error to try to initialize a `float` from a `double` in this fashion,
for example.)

##### Note

`reinterpret_cast` can be essential, but the essential uses (e.g., turning a machine address into pointer) are not type safe:
```c++
    auto p = reinterpret_cast<Device_register>(0x800);  // inherently dangerous
```

##### Enforcement

* C스타일, 함수형 형변환이 있다면 지적한다.
* The [type profile](#Pro-type-reinterpretcast) bans `reinterpret_cast`.
* The [type profile](#Pro-type-arithmeticcast) warns when using `static_cast` between arithmetic types.

### <a name="Res-casts-const"></a>ES.50: Don't cast away `const`

##### Reason

`const`를 거짓말로 만든다. 대상 변수가 정말로 `const`로 선언되었다면, `const`를 제거한 결과는 미정의 행동(undefined behavior)이 된다.

##### Example, bad

```c++
    void f(const int& i)
    {
        const_cast<int&>(i) = 42;   // BAD
    }

    static int i = 0;
    static const int j = 0;

    f(i); // silent side effect
    f(j); // undefined behavior
```

##### Example

Sometimes, you may be tempted to resort to `const_cast` to avoid code duplication, such as when two accessor functions that differ only in `const`-ness have similar implementations. For example:
```c++
    class Bar;

    class Foo {
    public:
        // BAD, duplicates logic
        Bar& get_bar() {
            /* complex logic around getting a non-const reference to my_bar */
        }

        const Bar& get_bar() const {
            /* same complex logic around getting a const reference to my_bar */
        }
    private:
        Bar my_bar;
    };
```
Instead, prefer to share implementations. Normally, you can just have the non-`const` function call the `const` function. However, when there is complex logic this can lead to the following pattern that still resorts to a `const_cast`:
```c++
    class Foo {
    public:
        // not great, non-const calls const version but resorts to const_cast
        Bar& get_bar() {
            return const_cast<Bar&>(static_cast<const Foo&>(*this).get_bar());
        }
        const Bar& get_bar() const {
            /* the complex logic around getting a const reference to my_bar */
        }
    private:
        Bar my_bar;
    };
```
Although this pattern is safe when applied correctly, because the caller must have had a non-`const` object to begin with, it's not ideal because the safety is hard to enforce automatically as a checker rule.

Instead, prefer to put the common code in a common helper function -- and make it a template so that it deduces `const`. This doesn't use any `const_cast` at all:
```c++
    class Foo {
    public:                         // good
              Bar& get_bar()       { return get_bar_impl(*this); }
        const Bar& get_bar() const { return get_bar_impl(*this); }
    private:
        Bar my_bar;

        template<class T>           // good, deduces whether T is const or non-const
        static auto get_bar_impl(T& t) -> decltype(t.get_bar())
            { /* the complex logic around getting a possibly-const reference to my_bar */ }
    };
```

##### Exception

You may need to cast away `const` when calling `const`-incorrect functions.
Prefer to wrap such functions in inline `const`-correct wrappers to encapsulate the cast in one place.

##### Example

보통 `const`를 없애버리는 이유는 변경할 수 없는 객체 속에 있는 일시적인 정보를 변경하기 위해서이다.
예를 들면 캐싱값, 임시계산값, 선계산값 등이다.
이런 값은 `const_cast`를 쓰는 것보다 `mutable`이나 간접적인 방법을 사용하면 더 쉽게 처리할 수 있다.

Consider keeping previously computed results around for a costly operation:
```c++
    int compute(int x); // compute a value for x; assume this to be costly

    class Cache {   // some type implementing a cache for an int->int operation
    public:
        pair<bool, int> find(int x) const;   // is there a value for x?
        void set(int x, int v);             // make y the value for x
        // ...
    private:
        // ...
    };

    class X {
    public:
        int get_val(int x)
        {
            auto p = cache.find(x);
            if (p.first) return p.second;
            int val = compute(x);
            cache.set(x, val); // insert value for x
            return val;
        }
        // ...
    private:
        Cache cache;
    };
```
Here, `get_val()` is logically constant, so we would like to make it a `const` member.
To do this we still need to mutate `cache`, so people sometimes resort to a `const_cast`:
```c++
    class X {   // Suspicious solution based on casting
    public:
        int get_val(int x) const
        {
            auto p = cache.find(x);
            if (p.first) return p.second;
            int val = compute(x);
            const_cast<Cache&>(cache).set(x, val);   // ugly
            return val;
        }
        // ...
    private:
        Cache cache;
    };
```
Fortunately, there is a better solution:
State that `cache` is mutable even for a `const` object:
```c++
    class X {   // better solution
    public:
        int get_val(int x) const
        {
            auto p = cache.find(x);
            if (p.first) return p.second;
            int val = compute(x);
            cache.set(x, val);
            return val;
        }
        // ...
    private:
        mutable Cache cache;
    };
```
An alternative solution would to store a pointer to the `cache`:
```c++
    class X {   // OK, but slightly messier solution
    public:
        int get_val(int x) const
        {
            auto p = cache->find(x);
            if (p.first) return p.second;
            int val = compute(x);
            cache->set(x, val);
            return val;
        }
        // ...
    private:
        unique_ptr<Cache> cache;
    };
```
That solution is the most flexible, but requires explicit construction and destruction of `*cache`
(most likely in the constructor and destructor of `X`).

In any variant, we must guard against data races on the `cache` in multi-threaded code, possibly using a `std::mutex`.

##### Enforcement

* `const_cast`를 지적한다.
* 이 규칙은 [타입 안정성 분석](#Pro-type-constcast)과 관련 있다

### <a name="Res-range-checking"></a>ES.55: Avoid the need for range checking

##### Reason

오버플로우가 발생할 소지가 없다 (또한 더 빠르게 실행될 수 있다).

##### Example

```c++
    for (auto& x : v)      // print all elements of v
        cout << x << '\n';

    auto p = find(v, x);   // find x in v
```

##### Enforcement

명시적인 범위검사를 찾아서 적절한 대안을 제시한다.

### <a name="Res-move"></a>ES.56: Write `std::move()` only when you need to explicitly move an object to another scope

##### Reason

We move, rather than copy, to avoid duplication and for improved performance.

A move typically leaves behind an empty object ([C.64](#Rc-move-semantic)), which can be surprising or even dangerous, so we try to avoid moving from lvalues (they might be accessed later).

##### Notes

Moving is done implicitly when the source is an rvalue (e.g., value in a `return` treatment or a function result), so don't pointlessly complicate code in those cases by writing `move` explicitly. Instead, write short functions that return values, and both the function's return and the caller's accepting of the return will be optimized naturally.

In general, following the guidelines in this document (including not making variables' scopes needlessly large, writing short functions that return values, returning local variables) help eliminate most need for explicit `std::move`.

Explicit `move` is needed to explicitly move an object to another scope, notably to pass it to a "sink" function and in the implementations of the move operations themselves (move constructor, move assignment operator) and swap operations.

##### Example, bad

```c++
    void sink(X&& x);   // sink takes ownership of x

    void user()
    {
        X x;
        // error: cannot bind an lvalue to a rvalue reference
        sink(x);
        // OK: sink takes the contents of x, x must now be assumed to be empty
        sink(std::move(x));

        // ...

        // probably a mistake
        use(x);
    }
```
Usually, a `std::move()` is used as an argument to a `&&` parameter.
And after you do that, assume the object has been moved from (see [C.64](#Rc-move-semantic)) and don't read its state again until you first set it to a new value.
```c++
    void f() {
        string s1 = "supercalifragilisticexpialidocious";

        string s2 = s1;             // ok, takes a copy
        assert(s1 == "supercalifragilisticexpialidocious");  // ok

        // bad, if you want to keep using s1's value
        string s3 = move(s1);

        // bad, assert will likely fail, s1 likely changed
        assert(s1 == "supercalifragilisticexpialidocious");
    }
```

##### Example

```c++
    void sink(unique_ptr<widget> p);  // pass ownership of p to sink()

    void f() {
        auto w = make_unique<widget>();
        // ...
        sink(std::move(w));               // ok, give to sink()
        // ...
        sink(w);    // Error: unique_ptr is carefully designed so that you cannot copy it
    }
```

##### Notes

`std::move()` is a cast to `&&` in disguise; it doesn't itself move anything, but marks a named object as a candidate that can be moved from.
The language already knows the common cases where objects can be moved from, especially when returning values from functions, so don't complicate code with redundant `std::move()`'s.

Never write `std::move()` just because you've heard "it's more efficient."
In general, don't believe claims of "efficiency" without data (???).
In general, don't complicate your code without reason (??)

##### Example, bad

```c++
    vector<int> make_vector() {
        vector<int> result;
        // ... load result with data
        return std::move(result);       // bad; just write "return result;"
    }
```
Never write `return move(local_variable);`, because the language already knows the variable is a move candidate.
Writing `move` in this code won't help, and can actually be detrimental because on some compilers it interferes with RVO (the return value optimization) by creating an additional reference alias to the local variable.

##### Example, bad

```c++
    vector<int> v = std::move(make_vector());   // bad; the std::move is entirely redundant
```
Never write `move` on a returned value such as `x = move(f());` where `f` returns by value.
The language already knows that a returned value is a temporary object that can be moved from.

##### Example

```c++
    void mover(X&& x) {
        call_something(std::move(x));         // ok
        call_something(std::forward<X>(x));   // bad, don't std::forward an rvalue reference
        call_something(x);                    // suspicious, why not std::move?
    }

    template<class T>
    void forwarder(T&& t) {
        call_something(std::move(t));         // bad, don't std::move a forwarding reference
        call_something(std::forward<T>(t));   // ok
        call_something(t);                    // suspicious, why not std::forward?
    }
```

##### Enforcement

* Flag use of `std::move(x)` where `x` is an rvalue or the language will already treat it as an rvalue, including `return std::move(local_variable);` and `std::move(f())` on a function that returns by value.
* Flag functions taking an `S&&` parameter if there is no `const S&` overload to take care of lvalues.
* Flag a `std::move`s argument passed to a parameter, except when the parameter type is one of the following: an `X&&` rvalue reference; a `T&&` forwarding reference where `T` is a template parameter type; or by value and the type is move-only.
* Flag when `std::move` is applied to a forwarding reference (`T&&` where `T` is a template parameter type). Use `std::forward` instead.
* Flag when `std::move` is applied to other than an rvalue reference. (More general case of the previous rule to cover the non-forwarding cases.)
* Flag when `std::forward` is applied to an rvalue reference (`X&&` where `X` is a concrete type). Use `std::move` instead.
* Flag when `std::forward` is applied to other than a forwarding reference. (More general case of the previous rule to cover the non-moving cases.)
* Flag when an object is potentially moved from and the next operation is a `const` operation; there should first be an intervening non-`const` operation, ideally assignment, to first reset the object's value.

### <a name="Res-new"></a>ES.60: Avoid `new` and `delete` outside resource management functions

##### Reason

프로그램 코드 내에서 직접적인 리소스 관리는 에러를 발생시키기 쉬우며 지루(tedious)하다.

##### Note

"No naked `new`!"로 알려져있다

##### Example, bad

```c++
    void f(int n)
    {
        auto p = new X[n];   // n default constructed Xs
        // ...
        delete[] p;
    }
```
`...`에 `delete`가 발생하지 않게 만드는 코드가 있을 수 있다.

**See also**: [R: 리소스 관리](#S-resource)

##### Enforcement

그대로 노출된 `new`와 `delete`를 지적한다.

### <a name="Res-del"></a>ES.61: Delete arrays using `delete[]` and non-arrays using `delete`

##### Reason

C++언어가 요구하는 것이며, 리소스 해제 오류와 메모리 오염(memory corruption) 문제가 발생할 수 있다. 

##### Example, bad

```c++
    void f(int n)
    {
        auto p = new X[n];   // n default constructed Xs
        // ...
        delete p;   // error: just delete the object p, rather than delete the array p[]
    }
```

##### Note

이 예제는 [no naked `new` rule](#Res-new)를 위반할 뿐만 아니라 많은 다른 문제를 야기한다.

##### Enforcement

* `new`, `delete`가 같은 영역범위에 있다면 오류여부를 지적할 수 있다
* `new`, `delete`가 생성자/소멸자 안에 있다면 오류여부를 지적할 수 있다

### <a name="Res-arr2"></a>ES.62: Don't compare pointers into different arrays

##### Reason

결과가 정의되지 않았다.

##### Example, bad

```c++
    void f(int n)
    {
        int a1[7];
        int a2[9];
        if (&a1[5] < &a2[7]) {}       // bad: undefined
        if (0 < &a1[5] - &a2[7]) {}   // bad: undefined
    }
```

##### Note

더 많은 문제가 내포되어 있다.

##### Enforcement

???

### <a name="Res-slice"></a>ES.63: Don't slice

##### Reason

Slicing -- that is, copying only part of an object using assignment or initialization -- most often leads to errors because
the object was meant to be considered as a whole.
In the rare cases where the slicing was deliberate the code can be surprising.

##### Example

```c++
    class Shape { /* ... */ };
    class Circle : public Shape { /* ... */ Point c; int r; };

    Circle c {{0, 0}, 42};
    Shape s {c};    // copy Shape part of Circle
```
The result will be meaningless because the center and radius will not be copied from `c` into `s`.
The first defense against this is to [define the base class `Shape` not to allow this](#Rc-copy-virtual).

##### Alternative

If you mean to slice, define an explicit operation to do so.
This saves readers from confusion.
For example:
```c++
    class Smiley : public Circle {
        public:
        Circle copy_circle();
        // ...
    };

    Smiley sm { /* ... */ };
    Circle c1 {sm};  // ideally prevented by the definition of Circle
    Circle c2 {sm.copy_circle()};
```

##### Enforcement

Warn against slicing.

### <a name="Res-construct"></a>ES.64: Use the `T{e}`notation for construction

##### Reason

The `T{e}` construction syntax makes it explicit that construction is desired.
The `T{e}` construction syntax doesn't allow narrowing.
`T{e}` is the only safe and general expression for constructing a value of type `T` from an expression `e`.
The casts notations `T(e)` and `(T)e` are neither safe nor general.

##### Example

For built-in types, the construction notation protects against narrowing and reinterpretation
```c++
    void use(char ch, int i, double d, char* p, long long lng)
    {
        int x1 = int{ch};     // OK, but redundant
        int x2 = int{d};      // error: double->int narrowing; use a cast if you need to
        int x3 = int{p};      // error: pointer to->int; use a reinterpret_cast if you really need to
        int x4 = int{lng};    // error: long long->int narrowing; use a cast if you need to

        int y1 = int(ch);     // OK, but redundant
        int y2 = int(d);      // bad: double->int narrowing; use a cast if you need to
        int y3 = int(p);      // bad: pointer to->int; use a reinterpret_cast if you really need to
        int y4 = int(lng);    // bad: long long->int narrowing; use a cast if you need to

        int z1 = (int)ch;     // OK, but redundant
        int z2 = (int)d;      // bad: double->int narrowing; use a cast if you need to
        int z3 = (int)p;      // bad: pointer to->int; use a reinterpret_cast if you really need to
        int z4 = (int)lng;    // bad: long long->int narrowing; use a cast if you need to
    }
```
The integer to/from pointer conversions are implementation defined when using the `T(e)` or `(T)e` notations, and non-portable
between platforms with different integer and pointer sizes.

##### Note

[Avoid casts](#Res-casts) (explicit type conversion) and if you must [prefer named casts](#Res-casts-named).

##### Note

When unambiguous, the `T` can be left out of `T{e}`.
```c++
    complex<double> f(complex<double>);

    auto z = f({2*pi, 1});
```

##### Note

The construction notation is the most general [initializer notation](#Res-list).

##### Exception

`std::vector` and other containers were defined before we had `{}` as a notation for construction.
Consider:
```c++
    vector<string> vs {10};                           // ten empty strings
    vector<int> vi1 {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};  // ten elements 1..10
    vector<int> vi2 {10};                             // one element with the value 10
```
How do we get a `vector` of 10 default initialized `int`s?
```c++
    vector<int> v3(10); // ten elements with value 0
```
The use of `()` rather than `{}` for number of elements is conventional (going back to the early 1980s), hard to change, but still
a design error: for a container where the element type can be confused with the number of elements, we have an ambiguity that
must be resolved.
The conventional resolution is to interpret `{10}` as a list of one element and use `(10)` to distinguish a size.

This mistake need not be repeated in new code.
We can define a type to represent the number of elements:
```c++
    struct Count { int n; };

    template<typename T>
    class Vector {
    public:
        Vector(Count n);                     // n default-initialized elements
        Vector(initializer_list<T> init);    // init.size() elements
        // ...
    };

    Vector<int> v1{10};
    Vector<int> v2{Count{10}};
    Vector<Count> v3{Count{10}};    // yes, there is still a very minor problem
```
The main problem left is to find a suitable name for `Count`.

##### Enforcement

Flag the C-style `(T)e` and functional-style `T(e)` casts.

### <a name="Res-deref"></a>ES.65: Don't dereference an invalid pointer

##### Reason

Dereferencing an invalid pointer, such as `nullptr`, is undefined behavior, typically leading to immediate crashes,
wrong results, or memory corruption.

##### Note

This rule is an obvious and well-known language rule, but can be hard to follow.
It takes good coding style, library support, and static analysis to eliminate violations without major overhead.
This is a major part of the discussion of [C++'s resource- and type-safety model](#Stroustrup15).

**See also**:

* Use [RAII](#Rr-raii) to avoid lifetime problems.
* Use [unique_ptr](#Rf-unique_ptr) to avoid lifetime problems.
* Use [shared_ptr](#Rf-shared_ptr) to avoid lifetime problems.
* Use [references](#Rf-ptr-ref) when `nullptr` isn't a possibility.
* Use [not_null](#Rf-not_null) to catch unexpected `nullptr` early.
* Use the [bounds profile](#SS-bounds) to avoid range errors.

##### Example

```c++
    void f()
    {
        int x = 0;
        int* p = &x;

        if (condition()) {
            int y = 0;
            p = &y;
        } // invalidates p

        *p = 42;            // BAD, p might be invalid if the branch was taken
    }
```
To resolve the problem, either extend the lifetime of the object the pointer is intended to refer to, or shorten the lifetime of the pointer (move the dereference to before the pointed-to object's lifetime ends).
```c++
    void f1()
    {
        int x = 0;
        int* p = &x;

        int y = 0;
        if (condition()) {
            p = &y;
        }

        *p = 42;            // OK, p points to x or y and both are still in scope
    }
```
Unfortunately, most invalid pointer problems are harder to spot and harder to fix.

##### Example

```c++
    void f(int* p)
    {
        int x = *p; // BAD: how do we know that p is valid?
    }
```
There is a huge amount of such code.
Most works -- after lots of testing -- but in isolation it is impossible to tell whether `p` could be the `nullptr`.
Consequently, this is also a major source of errors.
There are many approaches to dealing with this potential problem:
```c++
    void f1(int* p) // deal with nullptr
    {
        if (!p) {
            // deal with nullptr (allocate, return, throw, make p point to something, whatever
        }
        int x = *p;
    }
```
There are two potential problems with testing for `nullptr`:

* it is not always obvious what to do what to do if we find `nullptr`
* the test can be redundant and/or relatively expensive
* it is not obvious if the test is to protect against a violation or part of the required logic.

```c++
    void f2(int* p) // state that p is not supposed to be nullptr
    {
        assert(p);
        int x = *p;
    }
```
This would carry a cost only when the assertion checking was enabled and would give a compiler/analyzer useful information.
This would work even better if/when C++ gets direct support for contracts:
```c++
    void f3(int* p) // state that p is not supposed to be nullptr
        [[expects: p]]
    {
        int x = *p;
    }
```
Alternatively, we could use `gsl::not_null` to ensure that `p` is not the `nullptr`.
```c++
    void f(not_null<int*> p)
    {
        int x = *p;
    }
```
These remedies take care of `nullptr` only.
Remember that there are other ways of getting an invalid pointer.

##### Example

```c++
    void f(int* p)  // old code, doesn't use owner
    {
        delete p;
    }

    void g()        // old code: uses naked new
    {
        auto q = new int{7};
        f(q);
        int x = *q; // BAD: dereferences invalid pointer
    }
```
##### Example

```c++
    void f()
    {
        vector<int> v(10);
        int* p = &v[5];
        v.push_back(99); // could reallocate v's elements
        int x = *p; // BAD: dereferences potentially invalid pointer
    }
```

##### Enforcement

This rule is part of the [lifetime safety profile](#SS-lifetime)

* Flag a dereference of a pointer that points to an object that has gone out of scope
* Flag a dereference of a pointer that may have been invalidated by assigning a `nullptr`
* Flag a dereference of a pointer that may have been invalidated by a `delete`
* Flag a dereference to a pointer to a container element that may have been invalidated by dereference

## ES.stmt: Statements

Statements control the flow of control (except for function calls and exception throws, which are expressions).

### <a name="Res-switch-if"></a>ES.70: Prefer a `switch`-statement to an `if`-statement when there is a choice

##### Reason

* 가독성.
* 효율성: 상수값에 대해서 비교를 수행하므로 `if`-`then`-`else`문의 연속보다 `switch`문이 더 잘 최적화될 수 있다.
* `switch` 문은 경험적인 형태의 일관성 검사를 할 수 있게 한다. 예를 들자면, `enum` 모든 값을 확인하고 있는가? 그렇지 않다면 `default`는 있는가?

##### Example

```c++
    void use(int n)
    {
        switch (n) {   // good
        case 0:
            // ...
            break;
        case 7:
            // ...
            break;
        default:
            // ...
            break;
        }
    }
```
위의 예제가 더 좋다:
```c++
    void use2(int n)
    {
        if (n == 0)   // bad: if-then-else chain comparing against a set of constants
            // ...
        else if (n == 7)
            // ...
    }
```

##### Enforcement

상수값에 대해서 체크하는 if-then-else 연속이라면 지적한다. (이 경우에만)

### <a name="Res-for-range"></a>ES.71: Prefer a range-`for`-statement to a `for`-statement when there is a choice

##### Reason

가독성. 오류 예방. 효율성.

##### Example

```c++
    for (gsl::index i = 0; i < v.size(); ++i)   // bad
            cout << v[i] << '\n';

    for (auto p = v.begin(); p != v.end(); ++p)   // bad
        cout << *p << '\n';

    for (auto& x : v)    // OK
        cout << x << '\n';

    for (gsl::index i = 1; i < v.size(); ++i) // touches two elements: can't be a range-for
        cout << v[i] + v[i - 1] << '\n';

    for (gsl::index i = 0; i < v.size(); ++i) // possible side effect: can't be a range-for
        cout << f(v, &v[i]) << '\n';

    for (gsl::index i = 0; i < v.size(); ++i) { // body messes with loop variable: can't be a range-for
        if (i % 2 == 0)
            continue;   // skip even elements
        else
            cout << v[i] << '\n';
    }
```
프로그래머나 좋은 정적 분석기는 `f(&v[i])`에서 `v`에 대해서 부수효과(side effect)가 일어나지 않는다고 판단할지도 모른다. 이 경우 루프를 최적화할 수 있다.

루프문 내에서 "루프변수를 변경"하는 경우가 없어야 한다.

##### Note

범위 기반 `for`문에서 루프변수를 복사하여 사용하지 마라:
```c++
    for (string s : vs) // ...
```
위 코드는 `vs`의 원소를 `s`로 복사한다. 개선하면:
```c++
    for (string& s : vs) // ...
```
만약 루프 변수(`s`)가 변경되거나 복사되지 않는다면:
```c++
    for (const string& s : vs) // ...
```

##### Enforcement

루프를 보고 개별 요소들을 일렬로 참조하고 있고 부수효과(side effect)가 없어 보이면 `for`문으로 재작성한다.

### <a name="Res-for-while"></a>ES.72: Prefer a `for`-statement to a `while`-statement when there is an obvious loop variable

##### Reason

가독성: 루프에 대한 전체 로직을 첫구문에서 볼 수 있다. 루프변수의 범위가 제한되는 점도 좋다.

##### Example

```c++
    for (gsl::index i = 0; i < vec.size(); i++) {
        // do work
    }
```

##### Example, bad

```c++
    int i = 0;
    while (i < vec.size()) {
        // do work
        i++;
    }
```

##### Enforcement

???

### <a name="Res-while-for"></a>ES.73: Prefer a `while`-statement to a `for`-statement when there is no obvious loop variable

##### Reason

가독성

##### Example

```c++
    int events = 0;
    for (; wait_for_event(); ++events) {  // bad, confusing
        // ...
    }
```
The "event loop" is misleading because the `events` counter has nothing to do with the loop condition (`wait_for_event()`).
Better
```c++
    int events = 0;
    while (wait_for_event()) {      // better
        ++events;
        // ...
    }
```

##### Enforcement

Flag actions in `for`-initializers and `for`-increments that do not relate to the `for`-condition.

### <a name="Res-for-init"></a>ES.74: Prefer to declare a loop variable in the initializer part of a `for`-statement

##### Reason

루프 변수의 가시범위를 루프 범위 내로 제한하라.
루프문 뒤에서 다른 목적으로 루프 변수를 사용하지 못하게 하라.

##### Example

```c++
    for (int i = 0; i < 100; ++i) {   // GOOD: i var is visible only inside the loop
        // ...
    }
```

##### Example, don't

```c++
    int j;                            // BAD: j is visible outside the loop
    for (j = 0; j < 100; ++j) {
        // ...
    }
    // j is still visible here and isn't needed
```
**See also**: [Don't use a variable for two unrelated purposes](#Res-recycle)

##### Example

```c++
    for (string s; cin >> s; ) {
        cout << s << '\n';
    }
```

##### Enforcement

`for`문 안에서만 변하는 변수가 루프 밖에 선언되어 있지만 루프 밖에서 사용되지 않고 있다면 경고한다.

**Discussion**: 루프변수를 루프구문내로 범위로 정하면 코드 최적화에 많은 도움이 된다.
귀납 변수(induction variable)가 루프구문 안에서만 접근가능함을 파악하면 
위치이동(hoisting), 연산 부담 완화(strength reduction), 루프 내 불변코드 이동(loop-invariant code motion) 등의 최적화가 가능해진다.

> 번역 참고:
> * https://en.wikipedia.org/wiki/Strength_reduction
> * https://code-examples.net/ko/docs/gcc~7/optimize-options

### <a name="Res-do"></a>ES.75: Avoid `do`-statements

##### Reason

가독성. 오류 회피.
종료 조건이 끝에 위치해 있고(못 보고 넘어가기 쉬운 위치.) 첫 루프에서 체크를 하지 않는다.

##### Example

```c++
    int x;
    do {
        cin >> x;
        // ...
    } while (x < 0);
```

##### Note

Yes, there are genuine examples where a `do`-statement is a clear statement of a solution, but also many bugs.

##### Enforcement

Flag `do`-statements.

### <a name="Res-goto"></a>ES.76: Avoid `goto`

##### Reason

가독성. 오류 회피. 사람을 위해서는 더 좋은 컨트롤 구조가 있다;
`goto`는 기계코드(machine generated code)를 위한 것이다.

##### Exception

중첩된 루프에서 탈출.
이런 경우라면 항상 처리의 진행방향으로 점프하라.

Breaking out of a nested loop.
In that case, always jump forwards.
```c++
    for (int i = 0; i < imax; ++i)
        for (int j = 0; j < jmax; ++j) {
            if (a[i][j] > elem_max) goto finished;
            // ...
        }
    finished:
    // ...
```

##### Example, bad

C에서 goto-exit 형태(idiom)를 꽤 많이 사용한다:
```c++
    void f()
    {
        // ...
            goto exit;
        // ...
            goto exit;
        // ...
    exit:
        // ... common cleanup code ...
    }
```
이건 소멸자를 시뮬레이션한 것이다. 리소스를 해제하는 소멸자를 가진 핸들로 선언하라.

If for some reason you cannot handle all cleanup with destructors for the variables used,
consider `gsl::finally()` as a cleaner and more reliable alternative to `goto exit`

##### Enforcement

* `goto`가 보이면 지적한다. 루프 다음문으로 점프하지 않는 중첩 루프 내의 `goto`는 모두 표시하면 더 좋다.

### <a name="Res-continue"></a>ES.77: Minimize the use of `break` and `continue` in loops

##### Reason

In a non-trivial loop body, it is easy to overlook a `break` or a `continue`.

A `break` in a loop has a dramatically different meaning than a `break` in a `switch`-statement
 (and you can have `switch`-statement in a loop and a loop in a `switch`-case).

##### Example

    ???

##### Alternative

Often, a loop that requires a `break` is a good candidate for a function (algorithm), in which case the `break` becomes a `return`.

    ???

Often. a loop that uses `continue` can equivalently and as clearly be expressed by an `if`-statement.

    ???

##### Note

If you really need to break out a loop, a `break` is typically better than alternatives such as [modifying the loop variable](#Res-loop-counter) or a [`goto`](#Res-goto):

##### Enforcement

???

### <a name="Res-break"></a>ES.78: Always end a non-empty `case` with a `break`

##### Reason

실수로 `break`를 붙이지 않는 것은 꽤 많이 발생하는 버그다.
고의적으로 `break`를 없애는 것(fallthrough)은 유지보수의 위험요소가 된다.

##### Example

```c++
    switch (eventType) {
    case Information:
        update_status_bar();
        break;
    case Warning:
        write_event_log();
        // Bad - implicit fallthrough
    case Error:
        display_error_window();
        break;
    }
```
`break`로 안 끝나는 사항은 간과하기 쉽다. 명확하게 하라:
```c++
    switch (eventType) {
    case Information:
        update_status_bar();
        break;
    case Warning:
        write_event_log();
        // fallthrough
    case Error:
        display_error_window();
        break;
    }
```
C++17에서는, `[[fallthrough]]`를 사용하라:
```c++
    switch (eventType) {
    case Information:
        update_status_bar();
        break;
    case Warning:
        write_event_log();
        [[fallthrough]];        // C++17
    case Error:
        display_error_window();
        break;
    }
```

##### Note

단일문으로 된 여러개의 케이스 조건은 허용된다:
```c++
    switch (x) {
    case 'a':
    case 'b':
    case 'f':
        do_something(x);
        break;
    }
```

##### Enforcement

빈 `case`문이 아닌데 break로 끝나지 않는다면 지적한다.

### <a name="Res-default"></a>ES.79: Use `default` to handle common cases (only)

##### Reason

코드의 명확성.
오류를 탐지할 기회를 늘려준다.

##### Example

```c++
    enum E { a, b, c , d };

    void f1(E x)
    {
        switch (x) {
        case a:
            do_something();
            break;
        case b:
            do_something_else();
            break;
        default:
            take_the_default_action();
            break;
        }
    }
```
Here it is clear that there is a default action and that cases `a` and `b` are special.

##### Example

But what if there is no default action and you mean to handle only specific cases?
In that case, have an empty default or else it is impossible to know if you meant to handle all cases:
```c++
    void f2(E x)
    {
        switch (x) {
        case a:
            do_something();
            break;
        case b:
            do_something_else();
            break;
        default:
            // do nothing for the rest of the cases
            break;
        }
    }
```
If you leave out the `default`, a maintainer and/or a compiler may reasonably assume that you intended to handle all cases:
```c++
    void f2(E x)
    {
        switch (x) {
        case a:
            do_something();
            break;
        case b:
        case c:
            do_something_else();
            break;
        }
    }
```
Did you forget case `d` or deliberately leave it out?
Forgetting a case typically happens when a case is added to an enumeration and the person doing so fails to add it to every
switch over the enumerators.

##### Enforcement

Flag `switch`-statements over an enumeration that don't handle all enumerators and do not have a `default`.
This may yield too many false positives in some code bases; if so, flag only `switch`es that handle most but not all cases
(that was the strategy of the very first C++ compiler).

### <a name="Res-noname"></a>ES.84: Don't (try to) declare a local variable with no name

##### Reason

There is no such thing.
What looks to a human like a variable without a name is to the compiler a statement consisting of a temporary that immediately goes out of scope.
To avoid unpleasant surprises.

##### Example, bad

```c++
    void f()
    {
        lock<mutex>{mx};   // Bad
        // ...
    }
```
This declares an unnamed `lock` object that immediately goes out of scope at the point of the semicolon.
This is not an uncommon mistake.
In particular, this particular example can lead to hard-to find race conditions.
There are exceedingly clever uses of this "idiom", but they are far rarer than the mistakes.

##### Note

Unnamed function arguments are fine.

##### Enforcement

Flag statements that are just a temporary

### <a name="Res-empty"></a>ES.85: Make empty statements visible

##### Reason

가독성.

##### Example

```c++
    for (i = 0; i < max; ++i);   // BAD: the empty statement is easily overlooked
    v[i] = f(v[i]);

    for (auto x : v) {           // better
        // nothing
    }
    v[i] = f(v[i]);
```

##### Enforcement

블록이 아니면서 주석문을 포함하지 않는 빈 문장이 있다면 지적한다.

### <a name="Res-loop-counter"></a>ES.86: Avoid modifying loop control variables inside the body of raw for-loops

##### Reason

The loop control up front should enable correct reasoning about what is happening inside the loop. Modifying loop counters in both the iteration-expression and inside the body of the loop is a perennial source of surprises and bugs.

##### Example

```c++
    for (int i = 0; i < 10; ++i) {
        // no updates to i -- ok
    }

    for (int i = 0; i < 10; ++i) {
        //
        if (/* something */) ++i; // BAD
        //
    }

    bool skip = false;
    for (int i = 0; i < 10; ++i) {
        if (skip) { skip = false; continue; }
        //
        if (/* something */) skip = true;  // Better: using two variable for two concepts.
        //
    }
```

##### Enforcement

Flag variables that are potentially updated (have a non-`const` use) in both the loop control iteration-expression and the loop body.

### <a name="Res-if"></a>ES.87: Don't add redundant `==` or `!=` to conditions

##### Reason

Doing so avoids verbosity and eliminates some opportunities for mistakes.
Helps make style consistent and conventional.

##### Example

By definition, a condition in an `if`-statement, `while`-statement, or a `for`-statement selects between `true` and `false`.
A numeric value is compared to `0` and a pointer value to `nullptr`.
```c++
    // These all mean "if `p` is not `nullptr`"
    if (p) { ... }            // good
    if (p != 0) { ... }       // redundant `!=0`; bad: don't use 0 for pointers
    if (p != nullptr) { ... } // redundant `!=nullptr`, not recommended
```
Often, `if (p)` is read as "if `p` is valid" which is a direct expression of the programmers intent,
whereas `if (p != nullptr)` would be a long-winded workaround.

##### Example

This rule is especially useful when a declaration is used as a condition
```c++
    if (auto pc = dynamic_cast<Circle>(ps)) { ... } // execute is ps points to a kind of Circle, good

    if (auto pc = dynamic_cast<Circle>(ps); pc != nullptr) { ... } // not recommended
```

##### Example

Note that implicit conversions to bool are applied in conditions.
For example:
```c++
    for (string s; cin >> s; ) v.push_back(s);
```
This invokes `istream`'s `operator bool()`.

##### Note

Explicit comparison of an integer to `0` is in general not redundant.
The reason is that (as opposed to pointers and Booleans) an integer often has more than two reasonable values.
Furthermore `0` (zero) is often used to indicate success.
Consequently, it is best to be specific about the comparison.
```c++
    void f(int i)
    {
        if (i)            // suspect
        // ...
        if (i == success) // possibly better
        // ...
    }
```
Always remember that an integer can have more than two values.

##### Example, bad

It has been noted that
```c++
    if(strcmp(p1, p2)) { ... }   // are the two C-style strings equal? (mistake!)
```
is a common beginners error.
If you use C-style strings, you must know the `<cstring>` functions well.
Being verbose and writing
```c++
    if(strcmp(p1, p2) != 0) { ... }   // are the two C-style strings equal? (mistake!)
```
would not in itself save you.

##### Note

The opposite condition is most easily expressed using a negation:
```c++
    // These all mean "if `p` is `nullptr`"
    if (!p) { ... }           // good
    if (p == 0) { ... }       // redundant `== 0`; bad: don't use `0` for pointers
    if (p == nullptr) { ... } // redundant `== nullptr`, not recommended
```

##### Enforcement

Easy, just check for redundant use of `!=` and `==` in conditions.

## <a name="SS-numbers"></a>산술연산(Arithmetic)

### <a name="Res-mix"></a>ES.100: Don't mix signed and unsigned arithmetic

##### Reason

결과가 잘못될 수 있다.

##### Example

```c++
    int x = -3;
    unsigned int y = 7;

    cout << x - y << '\n';  // unsigned result, possibly 4294967286
    cout << x + y << '\n';  // unsigned result: 4
    cout << x * y << '\n';  // unsigned result, possibly 4294967275
```
It is harder to spot the problem in more realistic examples.

##### Note

불행히도 C++은 배열인자에 대해서 부호있는 정수를 사용하고 표준 라이브러리는 컨테이너 인자에 부호없는 정수형을 사용한다.

이는 일관적이지 않다. 배열 접근이라면 `gsl::index`를 사용하라. [ES.107](#Res-subscripts)을 참고하라.

##### Enforcement

* 컴파일러가 이미 알고 있는 상황이고 경고할 것이다
* (To avoid noise) Do not flag on a mixed signed/unsigned comparison where one of the arguments is `sizeof` or a call to container `.size()` and the other is `ptrdiff_t`.

### <a name="Res-unsigned"></a>ES.101: Use unsigned types for bit manipulation

##### Reason

부호없는 타입은 부호비트까지 포함해서 비트 연산할 수 있도록 지원하기 때문에 의도한 대로 동작한다.

##### Example

```c++
    unsigned char x = 0b1010'1010;
    unsigned char y = ~x;   // y == 0b0101'0101;
```

##### Note

Unsigned types can also be useful for modulo arithmetic.
However, if you want modulo arithmetic add
comments as necessary noting the reliance on wraparound behavior, as such code
can be surprising for many programmers.

##### Enforcement

* Just about impossible in general because of the use of unsigned subscripts in the standard library
* ???

### <a name="Res-signed"></a>ES.102: Use signed types for arithmetic

##### Reason

대부분의 산술 연산은 부호를 고려한다;
모듈러 연산과 같이 특별한 경우가 아니라면 `x - y`는 `y > x`인 경우 음수값이 나오길 기대한다.

##### Example

Unsigned arithmetic can yield surprising results if you are not expecting it.
This is even more true for mixed signed and unsigned arithmetic.
```c++
    template<typename T, typename T2>
    T subtract(T x, T2 y)
    {
        return x - y;
    }

    void test()
    {
        int s = 5;
        unsigned int us = 5;
        cout << subtract(s, 7) << '\n';       // -2
        cout << subtract(us, 7u) << '\n';     // 4294967294
        cout << subtract(s, 7u) << '\n';      // -2
        cout << subtract(us, 7) << '\n';      // 4294967294
        cout << subtract(s, us + 2) << '\n';  // -2
        cout << subtract(us, s + 2) << '\n';  // 4294967294
    }
```
Here we have been very explicit about what's happening,
but if you had seen `us - (s + 2)` or `s += 2; ...; us - s`, would you reliably have suspected that the result would print as `4294967294`?

##### Exception

Use unsigned types if you really want modulo arithmetic - add
comments as necessary noting the reliance on overflow behavior, as such code
is going to be surprising for many programmers.

##### Example

The standard library uses unsigned types for subscripts.
The built-in array uses signed types for subscripts.
This makes surprises (and bugs) inevitable.
```c++
    int a[10];
    for (int i = 0; i < 10; ++i) a[i] = i;
    vector<int> v(10);
    // compares signed to unsigned; some compilers warn, but we should not
    for (gsl::index i = 0; i < v.size(); ++i) v[i] = i;

    int a2[-2];         // error: negative size

    // OK, but the number of ints (4294967294) is so large that we should get an exception
    vector<int> v2(-2);
```
 Use `gsl::index` for subscripts; [see ES.107](#Res-subscripts).

##### Enforcement

* Flag mixed signed and unsigned arithmetic
* Flag results of unsigned arithmetic assigned to or printed as signed.
* Flag unsigned literals (e.g. `-2`) used as container subscripts.
* (To avoid noise) Do not flag on a mixed signed/unsigned comparison where one of the arguments is `sizeof` or a call to container `.size()` and the other is `ptrdiff_t`.

### <a name="Res-overflow"></a>ES.103: Don't overflow

##### Reason

오버플로우는 수식 알고리즘을 의미없게 만들어 버린다.
최대값 이상으로 증가시킨다면 메모리값이 망가지고 비정상적으로 작동한다.

##### Example, bad

```c++
    int a[10];
    a[10] = 7;   // bad

    int n = 0;
    while (n++ < 10)
        a[n - 1] = 9; // bad (twice)
```

##### Example, bad

```c++
    int n = numeric_limits<int>::max();
    int m = n + 1;   // bad
```

##### Example, bad

```c++
    int area(int h, int w) { return h * w; }

    auto a = area(10'000'000, 100'000'000);   // bad
```

##### Exception

모듈러 연산(modulo arithmetic)을 사용한다면 부호없는 타입을 사용하라.

**Alternative**: 어느 정도의 오버헤드를 감수할 수 있는 대단히 중요한 프로그램에서는 범위 검사를 수행하거나 부동소수점 타입을 사용하라.

##### Enforcement

???

### <a name="Res-underflow"></a>ES.104: Don't underflow

##### Reason

최소값 이하로 값이 내려가면 메모리값이 망가지고 비정상적으로 작동한다.

##### Example, bad

```c++
    int a[10];
    a[-2] = 7;   // bad

    int n = 101;
    while (n--)
        a[n - 1] = 9;   // bad (twice)
```

##### Exception

모듈러 연산(modulo arithmetic)을 사용한다면 부호없는 타입을 사용하라.

##### Enforcement

???

### <a name="Res-zero"></a>ES.105: Don't divide by zero

##### Reason

결과가 정의되지 않았으며 크래시를 발생시킬 것이다.

##### Note

모듈러 연산(`%`)에도 적용된다

##### Example; bad

```c++
    double divide(int a, int b) {
        // BAD, should be checked (e.g., in a precondition)
        return a / b;
    }
```
##### Example; good

```c++
    double divide(int a, int b) {
        // good, address via precondition (and replace with contracts once C++ gets them)
        Expects(b != 0);
        return a / b;
    }

    double divide(int a, int b) {
        // good, address via check
        return b ? a / b : quiet_NaN<double>();
    }
```
**Alternative**: 어느 정도의 오버헤드를 감수할 수 있는 대단히 중요한 프로그램에서는 범위 검사를 수행하거나 부동소수점 타입을 사용하라.

##### Enforcement

* Flag division by an integral value that could be zero

### <a name="Res-nonnegative"></a>ES.106: Don't try to avoid negative values by using `unsigned`

##### Reason

Choosing `unsigned` implies many changes to the usual behavior of integers, including modulo arithmetic,
can suppress warnings related to overflow,
and opens the door for errors related to signed/unsigned mixes.
Using `unsigned` doesn't actually eliminate the possibility of negative values.

##### Example

```c++
    unsigned int u1 = -2;   // Valid: the value of u1 is 4294967294
    int i1 = -2;
    unsigned int u2 = i1;   // Valid: the value of u2 is 4294967294
    int i2 = u2;            // Valid: the value of i2 is -2
```
These problems with such (perfectly legal) constructs are hard to spot in real code and are the source of many real-world errors.
Consider:
```c++
    unsigned area(unsigned height, unsigned width) { return height*width; } // [see also](#Ri-expects)
    // ...
    int height;
    cin >> height;
    auto a = area(height, 2);   // if the input is -2 a becomes 4294967292
```
Remember that `-1` when assigned to an `unsigned int` becomes the largest `unsigned int`.
Also, since unsigned arithmetic is modulo arithmetic the multiplication didn't overflow, it wrapped around.

##### Example

```c++
    unsigned max = 100000;    // "accidental typo", I mean to say 10'000
    unsigned short x = 100;
    while (x < max) x += 100; // infinite loop
```
Had `x` been a signed `short`, we could have warned about the undefined behavior upon overflow.

##### Alternatives

* use signed integers and check for `x >= 0`
* use a positive integer type
* use an integer subrange type
* `Assert(-1 < x)`

For example
```c++
    struct Positive {
        int val;
        Positive(int x) :val{x} { Assert(0 < x); }
        operator int() { return val; }
    };

    int f(Positive arg) { return arg; }

    int r1 = f(2);
    int r2 = f(-2);  // throws
```

##### Note

???

##### Enforcement

Hard: there is a lot of code using `unsigned` and we don't offer a practical positive number type.

### <a name="Res-subscripts"></a>ES.107: Don't use `unsigned` for subscripts, prefer `gsl::index`

##### Reason

To avoid signed/unsigned confusion.
To enable better optimization.
To enable better error detection.
To avoid the pitfalls with `auto` and `int`.

##### Example, bad

```c++
    vector<int> vec = /*...*/;

    for (int i = 0; i < vec.size(); i += 2)                    // may not be big enough
        cout << vec[i] << '\n';
    for (unsigned i = 0; i < vec.size(); i += 2)               // risk wraparound
        cout << vec[i] << '\n';
    for (auto i = 0; i < vec.size(); i += 2)                   // may not be big enough
        cout << vec[i] << '\n';
    for (vector<int>::size_type i = 0; i < vec.size(); i += 2) // verbose
        cout << vec[i] << '\n';
    for (auto i = vec.size()-1; i >= 0; i -= 2)                // bug
        cout << vec[i] << '\n';
    for (int i = vec.size()-1; i >= 0; i -= 2)                 // may not be big enough
        cout << vec[i] << '\n';
```

##### Example, good

```c++
    vector<int> vec = /*...*/;

    for (gsl::index i = 0; i < vec.size(); i += 2)             // ok
        cout << vec[i] << '\n';
    for (gsl::index i = vec.size()-1; i >= 0; i -= 2)          // ok
        cout << vec[i] << '\n';
```

##### Note

The built-in array uses signed subscripts.
The standard-library containers use unsigned subscripts.
Thus, no perfect and fully compatible solution is possible (unless and until the standard-library containers change to use signed subscripts someday in the future).
Given the known problems with unsigned and signed/unsigned mixtures, better stick to (signed) integers of a sufficient size, which is guaranteed by `gsl::index`.

##### Example

```c++
    template<typename T>
    struct My_container {
    public:
        // ...
        T& operator[](gsl::index i);    // not unsigned
        // ...
    };
```

##### Example

    ??? demonstrate improved code generation and potential for error detection ???

##### Alternatives

Alternatives for users

* use algorithms
* use range-for
* use iterators/pointers

##### Enforcement

* Very tricky as long as the standard-library containers get it wrong.
* (To avoid noise) Do not flag on a mixed signed/unsigned comparison where one of the arguments is `sizeof` or a call to container `.size()` and the other is `ptrdiff_t`.
