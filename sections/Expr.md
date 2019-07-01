# <a name="S-expr"></a>ES: 표현식과 구문

표현식(expression)과 구문(statement)은 행위와 연산에 대해 표현하는 가장 직접적인 방법들이다. 지역 유효범위 내에서의 선언 역시 구문에 포함된다.

이름 짓기와 주석, 들여쓰기 규칙에 대해서는, [NL: Naming and layout](./Naming.md#S-naming)을 참고하라.

일반적인 규칙:

* [ES.1: 다른 라이브러리나 "직접 짠 코드" 대신 표준 라이브러리를 사용하라](#Res-lib)
* [ES.2: 언어 기능을 바로 사용하기는 보다 적절히 추상화하라](#Res-abstr)

선언 규칙:

* [ES.5: 유효범위(scope)는 작게 유지하라](#Res-scope)
* [ES.6: for 문의 변수는 유효범위를 제한하기 위해 초기화와 조건 검사부분에서만 선언하라](#Res-cond)
* [ES.7: 일반적이거나 지역범위 변수들의 이름은 짧게, 그렇지 않다면 길게 하라](#Res-name-length)
* [ES.8: 비슷해보이는 이름은 피하라](#Res-name-similar)
* [ES.9: `ALL_CAPS` 같은 이름을 피하라](#Res-not-CAPS)
* [ES.10: 선언은 (오직) 하나의 이름을 선언해야 한다](#Res-name-one)
* [ES.11: 타입 이름의 불필요한 반복을 막을때는 `auto`를 사용하라](#Res-auto)
* [ES.12: 이름을 덮어쓰지 않도록 하라](#Res-reuse)
* [ES.20: 항상 개체를 초기화하라](#Res-always)
* [ES.21: 사용할 필요가 없을 때 변수나 상수를 선언하지 마라](#Res-introduce)
* [ES.22: 변수를 초기화할 값이 생길 때까지 선언하지 마라](#Res-init)
* [ES.23: `{}` 초기화 문법을 사용하라](#Res-list)
* [ES.24: 포인터는 `unique_ptr<T>`에 담아라](#Res-unique)
* [ES.25: 값을 변경하지 않는다면 개체를 `const` 혹은 `constexpr`로 선언하라](#Res-const)
* [ES.26: 서로 상관없는 목적에 하나의 변수를 사용하지 마라](#Res-recycle)
* [ES.27: 스택에서 사용되는 배열은 `std::array`나 `stack_array`를 사용하라](#Res-stack)
* [ES.28: 복잡한 초기화, 특히 `const` 변수의 초기화에는 람다를 사용하라](#Res-lambda-init)
* [ES.30: 프로그램 텍스트를 다루기(manipulate) 위해 매크로를 사용하지 마라](#Res-macros)
* [ES.31: 매크로를 상수나 "함수"에 사용하지 마라](#Res-macros2)
* [ES.32: 모든 매크로는 `ALL_CAPS` 형태로 선언하라](#Res-ALL_CAPS)
* [ES.33: 매크로를 사용해야만 한다면, 고유한 이름을 사용하라](#Res-MACROS)
* [ES.34: (C-스타일의) 가변인자 함수를 정의하지 마라](#Res-ellipses)

표현식 규칙:

* [ES.40: 복잡한 표현식을 피하라](#Res-complicated)
* [ES.41: 연산자 우선순위가 불분명하면, 소괄호를 사용하라(parenthesize)](#Res-parens)
* [ES.42: 포인터는 간단하고 직관적인 형태로 사용하라](#Res-ptr)
* [ES.43: 평가 순서가 정의되지 않은 표현식은 사용하지 마라](#Res-order)
* [ES.44: 함수 인자가 표현식 평가 순서의 영향을 받지 않게 하라](#Res-order-fct)
* [ES.45: 이유를 알 수 없는 상수(magic constant)를 사용하지 마라; 상징적인 상수를 사용하라](#Res-magic)
* [ES.46: 타입 범위를 축소하는 변환을 피하라](#Res-narrowing)
* [ES.47: `0` 혹은 `NULL`보다는 `nullptr`를 사용하라](#Res-nullptr)
* [ES.48: 타입 변환(cast)을 피하라](#Res-casts)
* [ES.49: 타입 변환을 사용해야만 한다면, 알려진 방법으로 변환(named cast)하라](#Res-casts-named)
* [ES.50: `const`를 제거하지 마라](#Res-casts-const)
* [ES.55: 범위 검사가 필요없게 하라](#Res-range-checking)
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

### <a name="Res-lib"></a>ES.1: 다른 라이브러리나 "직접 짠 코드" 대신 표준 라이브러리를 사용하라

##### Reason

라이브러리를 사용하는 코드는 언어의 기능을 직접적으로 사용하는 것보다 쉽고, 더 짧게 작성할 수 있고, 고수준의 추상화가 가능하다.
ISO C++ 표준 라이브러리는 널리 알려져있으며 테스트가 잘된 라이브러리다.
모든 C++ 구현체에서 제공하고 있다.

##### Example

```c++
    auto sum = accumulate(begin(a), end(a), 0.0);   // good
```

`accumulate`의 Ranges 버전이 더 낫다:

```c++
    auto sum = accumulate(v, 0.0); // better
```

잘 알려진 알고리즘을 직접 만들 필요는 없다:

```c++
    int max = v.size();   // bad: verbose, purpose unstated
    double sum = 0.0;
    for (int i = 0; i < max; ++i)
        sum = sum + v[i];
```

##### Exception

표준 라이브러리의 대다수가 동적 할당(자유 저장소)에 의존한다.
이런 부분은 알고리즘의 문제는 아닐지라도, 제한 시간 내에 응답성을 보장해야 하는 경우(hard real-time)나 임베디드 환경에는 적합하지 않다.
그런 경우는 비슷한 기능을 구현하여 사용하는 것을 고려해볼 수 있다. 예를 들면 표준 라이브러리 스타일로 구현된 메모리 풀 할당 컨테이너 같은 것들이다.

##### Enforcement

쉽지 않다.  
??? 지저분한 반복문, 중첩 반복문, 긴 함수, 함수 호출의 부재, 내장 타입이 아닌 타입을 거의 사용하지 않는 경우. 순환 복잡성?

### <a name="Res-abstr"></a>ES.2: 언어 기능을 바로 사용하기는 보다 적절히 추상화하라

##### Reason

"적절한 추상화"(예를 들어 라이브러리나 클래스 같은 것)가 언어보다 어플리케이션의 개념에 더 가깝다. 
코드를 짧고 명확하게 만들 수 있으며, 테스트하기에도 더 쉽다.

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

아래와 같은 전통적인 코드, 시스템 레벨과 거의 동등한 저수준(low-level) 코드는 길고, 지저분하고, 이해하기도 어렵고, 느리게 돌아간다:

```c++
    char** read2(istream& is, int maxelem, 
                 int maxstring, int* nread) // bad: verbose and incomplete
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

오버플로우나 오류처리 코드가 한 번 들어가게 되면, 코드는 급격히 지저분해진다. 
그리고, 반환하는 포인터와 배열로 구현되는 C 스타일의 문자열을 `delete`를 꼭 해줘야하는 문제도 있다.

##### Enforcement

쉽지 않다.  
??? 지저분한 반복문, 중첩 반복문, 긴 함수, 함수 호출의 부재, 내장 타입이 아닌 타입을 거의 사용하지 않는 경우. 순환 복잡성?

## ES.dcl: 선언(Declarations)

선언은 구문(statement)이다. 한 선언은 임의의 유효 범위에 하나의 이름을 만들거나 이름있는 개체(named object)를 생성할 수 있다.

### <a name="Res-scope"></a>ES.5: 유효범위(scope)는 작게 유지하라

##### Reason

가독성이 좋아진다. 리소스 점유를 최소화할 수 있다. 값의 잘못된 사용을 피할 수 있다.

##### Alternative Formulation

불필요하게 큰 범위에 변수를 선언하지 마라

##### Example

```c++
    void use()
    {
        int i;    // bad: i 가 반복문 이후에도 불필요하게 접근 가능하다
        for (i = 0; i < 20; ++i) { 
            /* ... */ 
        }

        // 위에서 선언한 i를 사용하지 않는다
        for (int i = 0; i < 20; ++i) { // good: i 는 for 반복문의 범위에서만 존재한다
            /* ... */
        }

        // good: pc 는 if 문의 범위에서만 존재한다
        if (auto pc = dynamic_cast<Circle*>(ps)) { 
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
        // ... 200 줄 코드 ...
    }
```

##### Enforcement

* 루프 바깥에서 루프 변수가 선언되고 이후에는 사용되지 않는 경우를 지적한다
* 파일 핸들이나 잠금과 같은 중요한 리소스를 사용하는 코드가 (적당히 큰) N줄 이상 계속되면 지적한다

### <a name="Res-cond"></a>ES.6: for 문의 변수는 유효범위를 제한하기 위해 초기화와 조건 검사부분에서만 선언하라

##### Reason

가독성. 시스템 자원 점유를 최소화한다.

##### Example

```c++
    void use()
    {
        for (string s; cin >> s;)
            v.push_back(s);

        // good: i 는 for 반복문의 범위에서만 존재한다
        for (int i = 0; i < 20; ++i) {
            // ...
        }

        // good: pc 는 if 문의 범위에서만 존재한다
        if (auto pc = dynamic_cast<Circle*>(ps)) {
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

C++17 에서는 `if`와 `switch`에 초기화 구문이 추가되었다. C++ 17을 지원하는 경우는 아래처럼 작성할 수 있다.

```c++
    map<int, string> mymap;

    if (auto result = mymap.insert(value); result.second) {
        // insert가 성공했고, 반환된 결과는 이 블록에서만 유효하다(valid)
        use(result.first);  // ok
        // ...
    } // result 는 이 시점에 파괴된다
```

##### C++17 enforcement (if using a C++17 compiler)

* 선택/반복 구문의 변수가 미리 선언되고 구문 이후에는 사용되지 않으면 지적하라
* (어려움) 선택/반복 구문의 변수가 미리 선언되고 구문 이후에 다른 상관없는 목적으로 사용되면 지적하라

### <a name="Res-name-length"></a>ES.7: 일반적이거나 지역범위 변수들의 이름은 짧게, 그렇지 않다면 길게 하라

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

인덱스는 관습적으로 `i`를 사용하고, 이 일반 함수에는 벡터의 의미를 알만한 힌트가 없으므로, `v`가 어떤 경우에든지 맞는 이름이다.

비교:

```c++
    template<typename Element_type>   // bad: 읽기 어렵다
    void print(ostream& target_stream, 
               const vector<Element_type>& current_vector)
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
    void complicated_algorithm(vector<Record>& vr, 
                               const vector<int>& vi,
                               map<string, int>& out)
        // read from events in vr (marking used Records) for the indices in
        // vi placing (name, index) pairs into out
    {
        // ... 500 lines of code using vr, vi, and out ...
    }
```

함수는 짧게 유지하는 것을 권장하지만, 이 규칙을 모두 적용시키긴 힘들 때가 있다.
그럴 경우엔 변수에 이름을 적절히 부여해야 한다.

##### Enforcement

지역 변수와 비지역 변수의 이름이 유지되는 범위의 길이를 확인한다. 동시에 함수의 길이를 함께 고려한다.

### <a name="Res-name-similar"></a>ES.8: 비슷해보이는 이름은 피하라

##### Reason

코드의 명확함과 가독성.
너무 비슷한 이름은 이해를 저해하고 오류가 발생할 소지를 낳는다.

##### Example; bad

```c++
    if (readable(i1 + l1 + ol + o1 + o0 + ol + o1 + I0 + l0))
        surprise();
```

##### Example; bad

같은 유효범위에서 타입이 아닌 것을 타입의 이름과 같은 이름으로 선언하지마라.
이러면 `struct` 혹은 `enum`을 사용해서 이름의 의미을 구분할 필요가 없게 된다.

`struct X` 같은 코드는 이름 탐색(lookup)이 실패하면 암묵적으로 `X`로 간주되기 때문에 오류의 원인을 제거하는 효과도 얻을 수 있다. 

```c++
    struct foo { int n; };

    struct foo foo();       // BAD, foo는 이미 타입의 이름으로 쓰이고 있다
    struct foo x = foo();   // 설명이 필요하다
```

##### Exception

좀 오래된(antique) 헤더 파일들은 타입이 아닌 것에 타입과 같은 이름을 붙여놓았을 수도 있다.

##### Enforcement

* 이미 알려진 (혼란을 일으키는) 글자 혹은 숫자 조합을 사용하는 이름이 있는지 검사한다
* 변수, 함수, 열거자(enumerator)의 선언이 같은 유효범위에서 선언된 클래스 혹은 열거형을 가리는(hide) 경우 지적한다

### <a name="Res-not-CAPS"></a>ES.9: `ALL_CAPS` 같은 이름을 피하라

##### Reason

이런 이름은 보통 매크로를 정의할 때 사용한다. 따라서 `ALL_CAPS` 형태의 이름은 매크로와 충돌될 가능성이 많다. 

##### Example

```c++
    // 어떤 헤더파일의 어느 지점:
    #define NE !=

    // 다른 어떤 헤더파일의 어느 지점:
    enum Coord { N, NE, NW, S, SE, SW, E, W };

    // 어느 불쌍한 프로그래머의 .cpp 파일 어느 지점:
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

### <a name="Res-name-one"></a>ES.10: 선언은 (오직) 하나의 이름을 선언해야 한다

##### Reason

한 줄에 선언 하나씩 하면 가독성을 향상시킬 수 있고, C/C++ 문법과 관련된 실수를 피할 수 있다. 
그리고 `//` 주석을 달 수 있는 공간이 생긴다.

##### Example, bad

```c++
    char *p, c, a[7], *pp[7], **aa[10];   // 윽 이게뭐야!
```

##### Exception

함수 선언은 다수의 함수 인자 선언을 포함할 수 있다.

##### Exception

C++ 17 의 structured binding은 여러 변수를 동시에 선언하기 위해 설계되었다:

```c++
    auto [iter, inserted] = m.insert_or_assign(k, val);
    if (inserted) {
        /* new entry was inserted */
    }
```

##### Example

```c++
    template <class InputIterator, class Predicate>
    bool any_of(InputIterator first, InputIterator last, Predicate pred);
```

컨셉(concepts)를 사용하면 이렇게 된다:

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

여러 변수들을 한번에 선언하는 것은 초기화되지 않은 변수를 간과하기 쉽다.

##### Enforcement

변수와 상수들을 한번에 선언을 한 곳을 지적한다.(예를 들어, `int* p, q;`)

### <a name="Res-auto"></a>ES.11: 타입 이름의 불필요한 반복을 막을때는 `auto`를 사용하라

##### Reason

* 간단한 내용이 반복되면 지루하고 오류에 취약하다.
* `auto`를 사용하면 선언된 개체(entity)가 그 지점에 고정되고, 가독성을 향상시킨다.
* 템플릿 함수 선언에서는 반환 타입이 멤버 타입일 수 있다.

##### Example

이런 코드를 생각해보자:

```c++
    auto p = v.begin();   // vector<int>::iterator
    auto h = t.future();
    auto q = make_unique<int[]>(s);
    auto f = [](int x){ return x + 10; };
```

각각의 경우, 타입들을 컴파일러가 이미 알고 있지만, 프로그래머가 기억하기 힘든 긴 이름의 타입을 작성할 필요가 없게 된다.

##### Example

```c++
    template<class T>
    auto Container<T>::first() -> Iterator;   // Container<T>::Iterator
```

##### Exception

초기화 리스트(initializer list), 그리고 초기화가 당신이 의도한(그리고 정확히 알고있는) 타입으로 변환되어야 하는 경우에는 `auto`의 사용을 피하라.

##### Example

```c++
    auto lst = { 1, 2, 3 };   // lst는 initializer_list<int> 타입이다

    auto x{1};  // C++ 17에서 x는 int 타입이지만,
                // C++ 11에서는 initializer_list로 처리된다
```

##### Note

컨셉(concepts)을 사용할 수 있게되면, 추론되는 타입을 좀 더 분명하게 표기할 수 있다.

```c++
    // ...
    ForwardIterator p = algo(x, y, z);
```

##### Example (C++17)

```c++
    auto [ quotient, remainder ] = div(123456, 73);   // 반환되는 div_t 타입의 멤버들을 분리해서 선언하게 된다
```

##### Enforcement

선언에서 장황한(redundant) 타입 이름이 반복되면 지적한다

### <a name="Res-reuse"></a>ES.12: 이름을 덮어쓰지 않도록 하라

##### Reason

어떤 변수가 사용되고 있는지 혼동하기 쉽다.
유지보수에 문제가 될수도 있다.

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

예시가 아주 큰 `if` 구문이었다면, 구문 안에서 새로운 `d`가 선언되는 것을 보지 못했을 수도 있다.
이런 코드는 버그의 원인으로 알려져있다.

보통 이렇게 더 깊은 유효범위에서 같은 이름을 사용하는 것을 "shadowing"이라고도 한다.

##### Note

Shadowing은 함수가 너무 크거나 복잡할때 문제가 된다.

##### Example

가장 바깥 범위에서 함수 인자들을 가리는 것은 언어에서 금지하고 있다:

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

멤버의 이름을 지역 변수로 사용하는 것 또한 문제가 된다:

```c++
    struct S {
        int m;
        void f(int x);
    };

    void S::f(int x)
    {
        m = 7;    // 멤버 변수에 대입한다
        if (x) {
            int m = 9;
            // ...
            m = 99; // 멤버 변수에 대입한다
            // ...
        }
    }
```

##### Exception

상위 클래스의 함수 이름을 하위 클래스에서 재사용하기도 한다:

```c++
    struct B {
        void f(int);
    };

    struct D : B {
        void f(double);
        using B::f;
    };
```

이는 오류에 취약하다.
예를 들어, using 선언을 하지 않았다면, `d.f(1)`의 호출은 `f(int)`를 찾지 못할 것이다.

??? 클래스 계층구조에서 shadowing/hiding에 대한 규칙이 필요할까요?

##### Enforcement

* 더 깊은(nested) 지역범위에서 이름을 재사용하면 지적하라
* 멤버 함수에서 멤버 이름을 지역변수의 이름으로 사용하면 지적하라
* 전역범위의 이름을 지역 변수 혹은 멤버의 이름으로 사용하면 지적하라
* 상위 클래스 멤버의 이름을 하위 클래스에서 재사용하면 지적하라 (함수 이름은 제외)

### <a name="Res-always"></a>ES.20: 항상 개체를 초기화하라

##### Reason

값을 저장하기 전에 사용하는 오류와 관련된 미정의 행동(undefined behavior)를 방지한다.
복잡한 초기화를 이해하면서 생기는 문제를 예방한다.
리팩토링이 쉬워진다.

##### Example

```c++
    void use(int arg)
    {
        int i;   // bad: 초기화가 안된 변수
        // ...
        i = 7;   // i를 초기화한다
    }
```

이런 코드는 좋지 않다. `i = 7`는 `i`를 초기화하는 것이 아니다; 변수에 값을 대입을 하는 것이다.
또한, `i`는 `...`부분에서 값을 읽을 수 있다. 이렇게 작성하는 것이 더 좋다:

```c++
    void use(int arg)   // OK
    {
        int i = 7;   // OK: 초기화한다
        string s;    // OK: 기본값으로 초기화한다
        // ...
    }
```

##### Note

*언제나 초기화하라*는 규칙은 *개체는 사용되기 전에 값을 가져야 한다*는 언어 규칙보다 (의도적으로) 엄격하다.
후자는 더 완화된 규칙으로, 기술적인 버그를 잡을수는 있다, 하지만:

* 가독성이 더 낮다
* 필요한 범위 이상으로 이름을 선언하도록 만든다
* 복잡한 코드로 인한 논리적 버그를 발생시킨다
* 리팩토링을 방해한다

*언제나 초기화하라*는 규칙은 값을 저장하지 않고 사용하는 오류를 막는 것 이외에도 유지보수성을 높이는데 초점을 둔 규칙이다.

##### Example

초기화에 대한 좀 더 약한 규칙이 필요한 경우를 보여주는 예시가 있다

```c++
    widget i;   // "widget" a type that's expensive to initialize, 
                // possibly a large POD
    widget j;

    if (cond) { // bad: i와 j가 "뒤늦게" 초기화된다
        i = f1();
        j = f2();
    }
    else {
        i = f3();
        j = f4();
    }
```

이런 코드는 `i`와 `j`를 초기화 하는 형태로 다시 작성하기 어렵다.
기본 생성자를 가지는 타입들에 대해서는, 초기화를 지연하는 코드는 기본 초기화를 하고 대입이 따라오는 구조가 된다.
이런 예시처럼 작성되는 이유는 보통 "효율적이기" 때문이다. 하지만 컴파일러가 값을 설정하지 않고 사용하는 오류를 찾을 수 있다면 초기화의 중복을 제거할수도 있다.

`i`와 `j`에 논리적인 상관관계가 있다고 가정하자. 이런 연결은 코드에 나타났을 것이다:

```c++
    pair<widget, widget> make_related_widgets(bool x)
    {
        return (x) ? {f1(), f2()} : {f3(), f4()};
    }

    auto [i, j] = make_related_widgets(cond);    // C++17
```

##### Note

복잡한 초기화는 수십년간 똑똑한 프로그래머들이 자주 사용해왔다.
동시에 복잡성과 오류의 주 원인이기도 했다.
이런 오류들은 초기 구현 이후 유지보수 단게에서 많이 발견되었다.

##### Example

이 규칙은 멤버 변수에도 적용된다.

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

컴파일러가 `cm3`가 `const`인데도 초기화되지 않은 것을 지적할 것이다. 하지만 이는 `m3`이 초기화되지 않은 것을 잡아내지는 않는다.

많은 경우, a rare spurious member initialization is worth the absence of errors from lack of initialization 
또 경우에 따라 최적화기에서 불필요한(redundant) 초기화를 제거할수도 있다.
(예컨대, 대입 직전에 초기화를 수행하는 경우)

##### Exception

입력에 따라 초기화되는 개체를 선언하고 있다면, 그 개체를 초기화 하는 것은 초기화를 2번 수행하는 것이다.
하지만, 이런 방법은 입력 이후에 초기화하지 않은 부분을 남길수도 있다는 점에 유의하라 -- 이는 보안과 관련된 오류의 온상(fertile source)이 되어왔다.

```c++
    constexpr int max = 8 * 1024;
    int buf[max];       // OK, but suspicious: uninitialized
    f.read(buf, max);
```

상황에 따라 배열을 초기화하는 비용이 막대할수도 있다.
하지만, 그런 경우는 초기화되지 않은 부분에 접근할 수 있도록 의도한 것이며, 초기화되지 않았다는 사실을 고려하면서 사용한다.

```c++
    constexpr int max = 8 * 1024;
    int buf[max] = {};  // 모든 원소들을 0으로 초기화한다
                        // 어떤 상황에서는 더 나은 방법이 된다
    f.read(buf, max);
```

할 수 있다면 오버플로우가 발생하지 않는 라이브러리 함수를 사용하라. 예를 들어:

```c++
    string s;   // s는 기본값 ""로 초기화된다
    cin >> s;   // s가 입력 문자열을 담기 위해 확장된다
```

입력 처리의 대상이 되는 변수들도 예외는 아니다:

```c++
    int i;   // bad
    // ...
    cin >> i;
```

보통 입력의 대상과 입력 처리가 분리되는 경우 (그러면 안되지만) used-before-set의 가능성이 발생한다.

```c++
    int i2 = 0;   // 좀 더 나은 코드, 0이 i2에 허용되는 값이라고 가정한다
    // ...
    cin >> i2;
```

좋은 최적화기는 입력 처리(operation)에 대해 알고있어야 하며, 중복되는 부분을 제거해야 한다

##### Example

"초기화되지 않은" 상태를 보여주는 값을 사용하는 것은 해결방법이 아니며 문제가 있는 것이다:

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

이 코드는 이제 컴파일러가 used-before-set을 쉽게 탐지할 수 없게 되었다.
더욱이 widget의 상태 공간을 더 복잡하게 만들었다: `uninit` 값을 가진 widget에는 어떤 처리가 허용(valid)되고 어떤 처리가 허용되지 않을까?

##### Note

어떨때는 람다를 초기화의 도구로 사용할수도 있다:

```c++
    error_code ec;
    Value v = [&] {
        auto p = get_value();   // get_value()에서 pair<error_code, Value>를 반환한다
        ec = p.first;
        return p.second;
    }();
```

또는 이렇게 할수도 있다:

```c++
    Value v = [] {
        auto p = get_value();   // get_value()에서 pair<error_code, Value>를 반환한다
        if (p.first) 
            throw Bad_value{p.first};
        return p.second;
    }();
```

##### See also

[ES.28](#Res-lambda-init)

##### Enforcement

* 모든 초기화되지 않은 변수들을 지적한다.  
  기본 생성자를 가진 사용자 정의 타입 변수들은 지적하지 않는다.
* 초기화되지 않은 버퍼가 선언 *하자마자* 쓰기-접근 되는지 검사하라.  
  비-`const` 참조 실행인자(argument)로 전달하는 경우는 쓰기-접근으로 가정한다. 

### <a name="Res-introduce"></a>ES.21: 사용할 필요가 없을 때 변수나 상수를 선언하지 마라

##### Reason

가독성.
변수가 사용될 수 있는 범위를 제한한다

##### Example

```c++
    int x = 7;
    // ... no use of x here ...
    ++x;
```
##### Enforcement

개체의 선언과 처음 사용되는 곳이 떨어져 있으면(distant from) 지적한다

### <a name="Res-init"></a>ES.22: 변수를 초기화할 값이 생길 때까지 선언하지 마라

##### Reason

가독성. 변수가 사용될 수 있는 범위를 제한한다.
used-before-set의 위험을 감수하지 마라.
대입보다 초기화가 더 효율적일 수 있다.

##### Example, bad

```c++
    string s;
    // ... s 를 사용하지 않는 부분 ...
    s = "what a waste";
```

##### Example, bad

```c++
    SomeLargeType var;  // ugly CaMeLcAsEvArIaBlE 
                        //      (Camel Case Variable)

    if (cond)   // 좀 중요한 조건
        Set(&var);
    else if (cond2 || !cond3) {
        var = Set2(3.14);
    }
    else {
        var = 0;
        for (auto& e : something)
            var += e;
    }

    // var를 사용한다; 
    // that this isn't done too early can be enforced statically with only control flow
```

`SomeLargeType`의 기본 초기화 비용이 크다면 이 코드는 괜찮다고 할 수 있다.
그렇지 않다면, 어떤 프로그래머는 저 복잡한 조건 속에서 모든 경우가 고려되었는지 의심할 것이다. 만약 허점이 있다면, "use before set" 버그가 된다. 유지보수의 함정인 것이다.

중간정도 복잡한 초기화에 대해서는, `const` 변수를 포함해서, 초기화에 람다를 사용하는 것을 고려해보라; [ES.28](#Res-lambda-init)를 참고하라.

##### Enforcement

* 처음 개체의 값을 읽기 전에 값을 대입하는 경우, 해당 개체가 기본 초기화로 선언되었으면 지적하라
* 초기화되지 않은 변수의 선언 이후에, 변수를 사용하기 전에 복잡한 처리(computation)이 있으면 지적하라

### <a name="Res-list"></a>ES.23: `{}` 초기화 문법을 사용하라

##### Reason

`{}` 초기화를 사용하라는 규칙은 쉽고, 더 일반적이며, 덜 모호하고, 다른 초기화 형태에 비해 안전하다.

##### Example

```c++
    int x {f(99)};
    vector<int> v = {1, 2, 3, 4, 5, 6};
```

##### Exception

컨테이너 타입들에 대해서는, `{...}`를 원소들을 나열하기 위해 사용하고 `(...)`는 크기를 나타내는데 사용한다는 전통(tradition)이 있다:

```c++
    vector<int> v1(10);    // vector of 10 elements with the default value 0
    vector<int> v2 {10};   // vector of 1 element with the value 10
```

##### Note

`{}` 초기화는 값의 범위가 줄어드는 타입변환(narrowing conversion)을 허용하지 않는다.

##### Example

```c++
    int x {7.9};   // error: narrowing
    int y = 7.9;   // OK: y becomes 7. Hope for a compiler warning
```

##### Note

`{}` 초기화는 다른 형태의 초기화와 달리 모든 경우의 초기화에 사용될 수 있다:

```c++
    auto p = new vector<int> {1, 2, 3, 4, 5};   // initialized vector

    D::D(int a, int b) :m{a, b} {   // member initializer 
                                    // (e.g., m might be a pair)
        // ...
    };
    X var {};   // initialize var to be empty
    struct S {
        int m {7};   // default initializer for a member
        // ...
    };
```

##### Note

`auto`로 선언한 변수를 하나의 값으로 초기화 하는것, 예를 들어, `{v}`형태는 C++ 17 이전까지는 예상밖의 결과를 낳는다.
C++ 17의 규칙은 상대적으로 덜 놀랍다:

```c++
    auto x1 {7};    // x1 is an int with the value 7
    auto x2 = {7};  // x2 is an initializer_list<int> with an element 7

    auto x11 {7, 8};    // error: two initializers
    auto x22 = {7, 8};  // x2 is an initializer_list<int> with elements 7 and 8
```

따라서 `initializer_list<T>`를 의도했다면 `={...}`를 사용하라.

```c++
    auto fib10 = {1, 1, 2, 3, 5, 8, 13, 21, 34, 55};   // fib10 is a list
```

##### Note

오래된 습관은 지우기 어렵다는 것을 생각하면, 이 규칙은 꾸준히 적용하기는 어렵다. 특히 `=`에 문제가 없는 경우가 너무나도 많다.

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

##### Enforcement

까다롭다(Tricky).

* 단순한 초기화를 위한 `=`는 지적하지 않는다
* `auto` 이후에 `=`가 사용된 경우를 찾는다

### <a name="Res-unique"></a>ES.24: 포인터는 `unique_ptr<T>`에 담아라

##### Reason

`std::unique_ptr`는 누수를 피하기 위한 가장 쉬운 방법이다.
이 방법은 믿을 수 있고, 타입 시스템이 안전한 소유권 관리를 위해 일하도록 만든다.
가독성을 향상시키고 실행시간 비용이 0에 가깝다.

##### Example

```c++
    void use(bool leak)
    {
        auto p1 = make_unique<int>(7);   // OK
        int* p2 = new int{7};            // bad: might leak

        // ... no assignment to p2 ...
        if (leak)
            return;

        // ... no assignment to p2 ...
        vector<int> v(7);
        v.at(7) = 0;                    // exception thrown
        // ...
    }
```

만약 `leak`이 `true`값을 가진다면 `p2`가 가리키는 개체가 누수된다. 하지만 `p1`이 가리키는 개체는 그렇지 않다.
`at()`이 예외를 던지는 경우에도 그렇다.

##### Enforcement

`new`, `malloc()` 혹은 그 결과를 반환하는 함수의 대상이 되는 원시 포인터를 찾는다.

### <a name="Res-const"></a>ES.25: 값을 변경하지 않는다면 개체를 `const` 혹은 `constexpr`로 선언하라

##### Reason

실수로 값을 바꾸는 걸 막을 수 있는 방법이다.
컴파일러에게 최적화를 위한 기회를 줄 수도 있다.

##### Example

```c++
    void f(int n)
    {
        const int bufmax = 2 * n + 2;  // good: bufmax가 이후의 코드에서 실수로 변경될 가능성이 없다
        int xmax = n;                  // suspicious: xmax를 나중에 바꾸려고 의도한 걸까?
        // ...
    }
```

##### Enforcement

변수가 실제로 값이 바뀌는지 안 바뀌는지 보고 바뀐다면 지적한다.
불행하게도, `const`가 아닌 개체가 값을 바꾸려 *의도*했는지 찾아내는 것은 불가능하다.

### <a name="Res-recycle"></a>ES.26: 서로 상관없는 목적에 하나의 변수를 사용하지 마라

##### Reason

가독성과 안전성

##### Example, bad

```c++
    void use()
    {
        int i;
        for (i = 0; i < 20; ++i) {
            /* ... */
        }
        for (i = 0; i < 200; ++i) { // bad: i 가 재사용된다
            /* ... */
        }
    }
```

##### Note

초기화를 위해서, buffer를 재사용하고 싶을수도 있다.
하지만 그렇더라도 변수의 범위를 최대한 제한하고 buffer에 남겨진 데이터로 인해 버그가 발생하지 않도록 주의하라.
재사용된 버퍼는 보안관련 버그의 원인이 되기도 한다.

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

### <a name="Res-stack"></a>ES.27: 스택에서 사용되는 배열은 `std::array`나 `stack_array`를 사용하라

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
        int a2[m];   // error: ISO C++가 아니다
        // ...
    }
```

##### Note

`a1` 변수선언은 C++에서는 적법하다. 이 방법을 사용한 코드가 많이 있다.
다만 이는 길이 값이 비지역 변수인 경우 잘못 사용하기 쉽다. 버퍼 오버플로우, 배열을 포인터로 변환하는 등의 "유명한" 오류 원인이 된다.

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

* 상수 길이를 가지지 않는 배열이라면 지적한다. (C 언어의 가변길이배열(VLA))
* 배열 길이로 지역 상수를 사용하지 않으면 지적한다

### <a name="Res-lambda-init"></a>ES.28: 복잡한 초기화, 특히 `const` 변수의 초기화에는 람다를 사용하라

##### Reason

멋지게 지역 초기화를 숨길 수 있다.
초기화 작업을 위해서만 필요한 변수를 포함해서 재사용할 것 같지 않은 함수를 생성할 필요도 없다.

약간의 초기화 작업 후에 `const`여야 하는 변수에도 사용할 수 있다.

##### Example, bad

```c++
    widget x;   // 가능하다면 const여야 한다, 하지만:
    for (auto i = 2; i <= N; ++i) {          // 이 부분이 x를 초기화하기 위한
        x += some_obj.do_something_with(i);  // 좀 긴 코드라고 하자
    }
    // 이 지점부터, x는 const가 되어야 한다. 
    // 하지만 이런 코딩 스타일에서는 그렇게 만들 수가 없다.
```

##### Example, good

```c++
    const widget x = [&]{
        widget val; // widget이 기본 생성자를 가진다고 가정하자
        for (auto i = 2; i <= N; ++i) {            // 이 부분이 x를 초기화하기 위한
            val += some_obj.do_something_with(i);  // 좀 긴 코드라고 하자
        }
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

### <a name="Res-macros"></a>ES.30: 프로그램 텍스트를 다루기(manipulate) 위해 매크로를 사용하지 마라

##### Reason

매크로는 버그의 주요 원인이다.
매크로는 일반적인 범위와 타입 규칙을 따르지 않는다.
매크로는 사람이 보는 것과 컴파일러가 보는 것을 다르게 한다.
매크로는 지원 도구를 만드는 것을 복잡하게 한다.

##### Example, bad

```c++
    #define Case break; case   /* BAD */
```

이 문제 없어 보이는 매크로는 `C`대신 `c`가 사용되면 악질적인(bad) 제어흐름 버그로 이어진다.

##### Note

이 규칙은 `#ifdef`문에서 설정제어를 위해 매크로를 사용하는 것은 막지 않는다.

미래에 모듈이 도입되면 설정을 제어하기 위한 매크로는 사라지게 될 것이다.

##### Note

이 규칙이 의도하는 것은 `#`을 사용해 문자를 만들어내거나 `##`를 사용해 접합(concat)하는 것이다. 
보통의 매크로들 처럼, "거의 무해한" 경우도 있다. 하지만 자동 완성기, 정적 분석기, 디버거와 같은 도구에게는 문제가 된다.

경우에 따라서는 근사한 매크로를 사용하는 것이 과도하게 복잡한 설계가 있다는 신호일 수 있다.
또한, `#`와 `##`를 사용하면 매크로를 정의하고 사용하도록 유도(encourage)한다:

```c++
    #define CAT(a, b) a ## b
    #define STRINGIFY(a) #a

    void f(int x, int y)
    {
        string CAT(x, y) = "asdf";   // BAD: 툴에서 다루기 어렵다 (그리고 못생겼다)
        string sx2 = STRINGIFY(x);
        // ...
    }
```

매크로없이 문자열을 조작하기 위한 방법(workaround)이 있다.
예를 들자면:

```c++
    string s = "asdf" "lkjh";   // 평범한 문자열 리터럴 접합 (literal concatenation)

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

매크로만큼 편리한 것은 아니지만, 쉽게 사용할 수 있고, 오버헤드를 발생시키지 않으며, 타입과 유효범위의 영향을 받는다.

미래에는 정적 리플렉션(static reflection)이 전처리기를 사용해 문자열을 다루는 것을 없애게 될 것이다.

##### Enforcement

소스제어(`#ifdef`같은)에 사용하지 않는 매크로를 본다면 소리를 질러라.

### <a name="Res-macros2"></a>ES.31: 매크로를 상수나 "함수"에 사용하지 마라

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

    template<typename T> 
    T square(T a, T b) { return a * b; }
```

##### Enforcement

소스제어(`#ifdef`같은)에 사용하지 않는 매크로를 본다면 소리를 질러라.

### <a name="Res-ALL_CAPS"></a>ES.32: 모든 매크로는 `ALL_CAPS` 형태로 선언하라

##### Reason

관습. 가독성. 매크로 구별.

##### Example

```c++
    #define forever for (;;)   /* 엄청 나쁜 코드 */

    #define FOREVER for (;;)   /* 여전히 사악하지만, 최소한 사람은 매크로라는걸 알 수 있다 */
```

##### Enforcement

소문자로 작성된 매크로를 본다면 소리를 질러라.

### <a name="Res-MACROS"></a>ES.33: 매크로를 사용해야만 한다면, 고유한 이름을 사용하라

##### Reason

매크로는 유효범위 규칙을 따르지 않는다.

##### Example

```c++
    #define MYCHAR        /* BAD, will eventually clash with someone else's MYCHAR*/

    #define ZCORP_CHAR    /* Still evil, but less likely to clash */
```

##### Note

가능하다면 매크로는 사용하지 마라: [ES.30](#Res-macros), [ES.31](#Res-macros2), 그리고 [ES.32](#Res-ALL_CAPS)를 참고하라.

안타깝게도, 매크로를 사용하거나 남용하는 긴 전통과 함께 매크로의 영향을 받는 코드가 수십억 줄은 있을 것이다. 매크로를 사용해야만 한다면, 긴 이름을 사용하고 고유한 접두사(prefix)를 붙여서 (당신이 속한 조직의 이름이라던지) 이름이 충돌할 가능성을 낮춰라.

##### Enforcement

짧은 매크로 이름에 대해서 경고하라.

### <a name="Res-ellipses"></a> ES.34: (C-스타일의) 가변인자 함수를 정의하지 마라

##### Reason

타입 안전하지 않다.
정확하게 동작하기 위해서 지저분한 변환/매크로 코드가 필요하다.

##### Example

```c++
    #include <cstdarg>

    // "severity" followed by a zero-terminated list of char*s; 
    // write the C-style strings to cerr
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
##### Alternative

중복 정의, 템플릿, 가변 템플릿을 사용하라

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

이 방법으로 `printf`가 구현되어있다.

##### Enforcement

* C-스타일 가변인자 함수를 정의하면 지적하라
* `#include <cstdarg>`와 `#include <stdarg.h>`를 지적하라

## ES.expr: 표현식

표현식은 값을 조작한다(manipulate).

### <a name="Res-complicated"></a>ES.40: 복잡한 표현식을 피하라

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

위의 연산식 중 몇은 의심의 여지없이 나쁘다. (정의되지 않은 행동이 일어나게 한다)
나머지는 꽤 복잡하거나 특이한 편이고, 심지어 능력있는 프로그래머도 잘못 이해하거나 문제를 간과해 버릴 만한 것도 있다.

##### Note

C++17 에서는 평가 순서를 규정하고 있다.  
오른쪽에서 왼쪽으로 대입되는 것을 제외하고 왼쪽에서 오른쪽 순서로 평가된다. 
함수의 실행인자 평가순서는 정의되어 있지 않다; [ES.43 를 참고하라](#Res-order)
하지만 이 규칙의 유무가 복잡한 표현식이 혼란을 만든다는 사실을 바꾸지는 않는다.

##### Note

프로그래머는 표현식의 기본적인 규칙들을 알고 사용해야 한다.

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
* 구현에 따라 달라지는 행동(implementation defined behavior)?
* ???

### <a name="Res-parens"></a>ES.41: 연산자 우선순위가 불분명하면, 소괄호를 사용하라(parenthesize)

##### Reason

오류가 발생하지 않게 하라. 가독성. 모든 사람들이 연산자 우선순위를 기억하지는 않는다.

##### Example

```c++
    const unsigned int flag = 2;
    unsigned int a = flag;

    if (a & flag != 0)  // bad: a & (flag != 0)를 의도했다
```

##### Note

프로그래머는 산술 연산, 논리 연산에 대해서 우선순위 테이블을 알고 있을 것을 기대한다.
다른 연산과 비트 연산을 섞어 사용할 때는 소괄호(parentheses)를 사용하기를 권한다.

```c++
    if ((a & flag) != 0)  // OK: 의도대로 동작한다
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
* 가장 왼쪽에 위치한 연산자(leftmost operator)가 할당 연산자가 아니라면 지적한다
* ???

### <a name="Res-ptr"></a>ES.42: 포인터는 간단하고 직관적인 형태로 사용하라

##### Reason

복잡한 포인터 계산은 주요한 오류 원인이 된다.

##### Note

포인터 대신 `gsl::span`를 사용하라.
포인터는 [오직 하나의 개체를 가리킬 때만 사용해야 한다](./Interfaces.md#Ri-array).
포인터의 산술연산은 잘못 사용하기 쉽고 수많은, 수많은 나쁜 버그와 보안  위험(violation)의 원인이다.
`span`은 경계를 검사하고, 안전하게 배열의 데이터에 접근하는 타입이다.
경계를 알 수 있는 배열에 대한 접근(subscript)에 상수를 사용하는 코드는 컴파일러가 평가할 수 있다.

##### Example, bad

```c++
    void f(int* p, int count)
    {
        if (count < 2)
            return;

        int* q = p + 1;    // BAD

        ptrdiff_t d;
        int n;
        d = (p - &n);      // OK
        d = (q - p);       // OK

        int n = *p++;      // BAD

        if (count < 6)
            return;

        p[4] = 1;          // BAD

        p[count - 1] = 2;  // BAD

        use(&p[0], 3);     // BAD
    }
```

##### Example, good

```c++
    void f(span<int> a) // BETTER: use span in the function declaration
    {
        if (a.size() < 2)
            return;

        int n = a[0];      // OK

        span<int> q = a.subspan(1); // OK

        if (a.size() < 6)
            return;

        a[4] = 1;          // OK

        a[a.size() - 1] = 2;  // OK

        use(a.data(), 3);  // OK
    }
```

##### Note

변수를 사용해 배열에 접근하는 코드가 안전한지 평가하는 것은 도구와 사람 모두에게 어렵다.
`span`은 실행 시간에 경계를 검사하기 때문에, 배열의 데이터에 접근할때 안전하다.
`at()`은 한번 접근할 때 경계를 검사하는 다른 방법이다.
배열에 접근할 때 반복자가 필요하다면, 배열에 대한 `span`을 생성하고 그에 대한 반복자를 사용하라.

##### Example, bad

```c++
    void f(array<int, 10> a, int pos)
    {
        a[pos / 2] = 1; // BAD
        a[pos - 1] = 2; // BAD
        a[-1] = 3;    // BAD (다만 도구에서 잡아낼 수 있다) -- 다른 방법이 없다. 그냥 이런 코드를 작성하지 마라
        a[10] = 4;    // BAD (다만 도구에서 잡아낼 수 있다) -- 다른 방법이 없다. 그냥 이런 코드를 작성하지 마라
    }
```

##### Example, good

`span`을 사용하면 이렇다:

```c++
    // A1: 매개변수 타입을 span을 사용하도록 바꾸었다
    void f1(span<int, 10> a, int pos) 
    {
        a[pos / 2] = 1; // OK
        a[pos - 1] = 2; // OK
    }

    // A2: 지역변수로 span을 만들어 사용한다
    void f2(array<int, 10> arr, int pos) 
    {
        span<int> a = {arr, pos};
        a[pos / 2] = 1; // OK
        a[pos - 1] = 2; // OK
    }
```

`at()`을 사용하면 이렇다:

```c++
    // ALTERNATIVE B: 원소에 접근할때 at()을 사용한다
    void f3(array<int, 10> a, int pos) 
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

`span`을 사용하면 이렇다:

```c++
    void f1()
    {
        int arr[COUNT];
        span<int> av = arr;
        for (int i = 0; i < COUNT; ++i)
            av[i] = i;
    }
```

범위 기반 `for`문에 `span`을 사용하면 이렇다:

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

접근할 때 `at()`를 사용하면 이렇다:

```c++
    void f2()
    {
        int arr[COUNT];
        for (int i = 0; i < COUNT; ++i)
            at(arr, i) = i;
    }
```

범위기반 `for`문은 이렇다:

```c++
    void f3()
    {
        int arr[COUNT];
        for (auto& e : arr)
             e = i++;
    }
```

##### Note

도구에서 실행시간에 결정되는 인덱스로 배열에 접근하는 표현식을 `at()`을 사용해 다시 작성하도록 할수도 있다:

```c++
    static int a[10];

    void f(int i, int j)
    {
        a[i + j] = 12;      // BAD, 이 코드를 다시 쓴다면 ...
        at(a, i + j) = 12;  // OK -- 경계를 검사한다
    }
```

##### Example

(그동안 언어에서 해왔던 것처럼) 배열을 포인터로 바꾸는 것은 경계 검사의 기회를 없애버린다. 지양하라.

```c++
    void g(int* p);

    void f()
    {
        int a[5];
        g(a);        // BAD: are we trying to pass an array?
        g(&a[0]);    // OK: passing one object
    }
```

배열을 전달하고 싶다면:

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

* 포인터 타입에 대한 산술연산을 수행하는 표현식은 지적하라
* 배열(정적 배열 혹은 `std::array`)에 인덱스를 사용해 접근하는 표현식을 지적하라. 이때 표현식은 배열 범위 안(`0`부터 배열 끝까지)에 해당하는 컴파일 시간 상수 표현식이 아니어야 한다
* 배열 타입에서 포인터 타입으로 묵시적 형변환에 의존하는 표현식을 지적하라

이 규칙은 [경계 안전성 검사](./Profile.md#SS-bounds)의 일부분이다.

### <a name="Res-order"></a>ES.43: 평가 순서가 정의되지 않은 표현식은 사용하지 마라

##### Reason

그런 코드가 어떻게 동작할지는 알 수가 없다. 이식성.
특정한 환경에는 맞을지는 몰라도, 다른 컴파일러 (혹은 사용 중인 컴파일러의 다음 버전)에서는 다를 수 있다. 
혹은 최적화 설정에 따라 다를 수도 있다.

##### Note

C++17 에서는 평가 순서를 규정하고 있다.  
오른쪽에서 왼쪽으로 대입되는 것을 제외하고 왼쪽에서 오른쪽 순서로 평가된다. 
함수의 실행인자 평가순서는 정의되어 있지 않다.

당신의 코드가 (Ctrl + C,V 되어서) C++ 17 이전의 컴파일러로 컴파일 될 수 있다는 것을 기억하라. 너무 영리할 필요는 없다.

##### Example

```c++
    v[i] = ++i;   // 결과는 알 수 없다(undefined)
```

가장 좋은 규칙은 값을 변경하는 표현식에서 값을 읽지 않는 것이다.

##### Enforcement

좋은 분석기를 사용해 찾을 수 있다.

### <a name="Res-order-fct"></a>ES.44: 함수 인자가 표현식 평가 순서의 영향을 받지 않게 하라

##### Reason

순서가 정의되어있지 않다.

##### Note

C++17 에서는 평가 순서를 규정하고 있다.  
오른쪽에서 왼쪽으로 대입되는 것을 제외하고 왼쪽에서 오른쪽 순서로 평가된다. 
함수의 실행인자 평가순서는 정의되어 있지 않다.

##### Example

```c++
    int i = 0;
    f(++i, ++i);
```

이 함수 호출은 `f(0, 1)`혹은 `f(1, 0)`일 것이다. 하지만 어떤 것이 될지는 알 수 없다.
기술적으로는, 어떻게 처리해야 하는지 정의되어 있지 않다.

C++ 17에서는 이 코드가 미정의 행동이 아니다. 하지만 여전히 어떤 인자가 먼저 평가되는지 분명하지 않다.

##### Example

중복정의된 연산자들은 평가순서 문제로 이어질 수 있다:

```c++
    f1()->m(f2());          // m( f1(), f2() )
    cout << f1() << f2();   // operator<<( operator<<( cout, f1() ), f2() )
```

C++ 17에서 이 예시는 기대한 대로 동작한다 (왼쪽에서 오른쪽으로 평가된다).
그리고 `=`의 바인딩이 오른쪽에서 왼쪽으로 수행되는 것처럼 대입은 오른쪽에서 왼쪽으로 평가된다. 

```c++
    f1() = f2();    // C++14 에서는 미정의 행동; 
                    // C++17 에서는 f2()가 f1()보다 먼저 평가된다
```

##### Enforcement

좋은 분석기를 사용해 찾을 수 있다.

### <a name="Res-magic"></a>ES.45: 이유를 알 수 없는 상수(magic constant)를 사용하지 마라; 상징적인 상수를 사용하라

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

### <a name="Res-narrowing"></a>ES.46: 타입 범위를 축소하는 변환을 피하라

##### Reason

정보를 파괴하고 전혀 기대하지 않은 값을 가지게 한다.

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

좋은 분석기는 범위가 축소되는 변환을 탐지할 수 있다.
하지만 모든 축소변환을 지적하는 것은 수많은 거짓 양성(false positive)로 이어질 것이다. 

제안:

* 부동 소수점에서 정수로의 변환을 지적한다 (`float`->`char` 이나 `double`->`int`만 지적할수도 있다. 더 많은 정보가 필요하다)
* 모든 `long`->`char` 변환을 지적한다. (`int`->`char` 변환은 상당히 일반적이다. 더 많은 정보가 필요하다)
* 함수의 실행인자에서 축소 변환이 발생하면 특히 의심스럽게 생각한다

### <a name="Res-nullptr"></a>ES.47: `0` 혹은 `NULL`보다는 `nullptr`를 사용하라

##### Reason

가독성의 문제다. 기대를 벗어나지 않게 한다.

`nullptr`는 `int`와 혼동의 여지가 없다. 
`nullptr`는 잘 명세된 (아주 까다로운) 타입을 가지고 있다. 그러니 `NULL` 혹은 `0`을 사용하면 잘못될 수 있는 타입 추론 상황에서 더 잘 동작할 것이다.

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

### <a name="Res-casts"></a>ES.48: 타입 변환(cast)을 피하라

##### Reason

잘 알려진 오류의 원인이다. 최적화를 신뢰할 수 없게 만들어 버린다.

##### Example, bad

```c++
    double d = 2;
    auto p = (long*)&d;
    auto q = (long long*)&d;
    cout << d << ' ' << *p << ' ' << *q << '\n';
```

이 코드는 어떤 값을 출력할까?
The result is at best implementation defined. 
필자의 환경에서는 이런 결과가 나온다

```
    2 0 4611686018427387904
```

아래의 내용을 더하면,

```c++
    *q = 666;
    cout << d << ' ' << *p << ' ' << *q << '\n';
```

이런 결과가 나온다

```
    3.29048e-321 666 666
```

놀라운가? 프로그램에서 크래시나 생기지 않은게 다행이다.

##### Note

타입 변환을 작성하는 프로그래머는 자신이 무엇을 하고있는지 안다고 생각한다. 또는 값을 사용할때 일반적인 규칙들을 접어두기도 한다.

중복정의 선택(overload resolution)이나 템플릿 실체화(instantiation)는 보통 인자 타입이 꼭 맞는 함수를 골라낸다. 그런 함수가 없다면, 어쩌면 고쳐서(local fix) 적용할 필요가 있을지도 모르지만, 오류가 된다.

##### Note

형변환은 시스템 프로그래밍 언어에 꼭 필요하다.
예를 들어, 디바이스 레지스터의 주소를 포인터로 얻어 올 때이다.
그러나 너무 남용하는 바람에 많은 오류가 발생하는 것도 사실이다.

##### Note

형변환을 너무 많이 쓴다고 생각된다면 설계에 근본적인 문제가 있을지도 모른다.

##### Exception

Casting to `(void)` is the Standard-sanctioned way to turn off `[[nodiscard]]` warnings. 
`[[nodiscard]]` 속성이 있는 함수를 호출하면서 반환 결과를 버리기를 원한다면,
우선 그 생각이 정말 좋은 생각인지 진지하게 고민하라
(무엇보다 함수의 반환 타입에 `[[nodiscard]]`를 작성한데는 보통 타당한 이유가 있다),
하지만 당신이 값을 버려도 적절하다고 생각하고, 당신과 함께 코드를 리뷰한 사람들이 동의한다면 `(void)`를 써서 경고를 없애라.

##### Alternatives

타입 변환은 널리 (잘못) 사용되고 있다. 모던 C++은 규칙을 두고 여러 방법으로 타입 변환이 필요 없도록 한다.

* 템플릿을 사용한다
* `std::variant`을 사용한다
* 포인터 타입 간의 잘 정의되고, 안전하고, 묵시적인 변환을 사용한다

##### Enforcement

* `[[nodiscard]]`로 반환하는 함수를 제외하고 C-스타일 타입 변환을 없애도록 강제한다
* Warn if there are many functional style casts (there is an obvious problem in quantifying 'many')
* The [type profile](./Profile.md#Pro-type-reinterpretcast) bans `reinterpret_cast`.
* Warn against [identity casts](./Profile.md#Pro-type-identitycast) between pointer types, where the source and target types are the same (#Pro-type-identitycast)
* 포인터가 [묵시적](./Profile.md#Pro-type-implicitpointercast)으로 변환될 수 있으면 경고한다

### <a name="Res-casts-named"></a>ES.49: 타입 변환을 사용해야만 한다면, 알려진 방법으로 변환(named cast)하라

##### Reason

가독성. 오류 예방.
Named cast들은 C 스타일이나 함수형 형변환보다 더 구체적이며, 컴파일러가 일부 오류를 잡아낼 수 있도록 한다.

> 역주:
> * C 스타일 형변환: `(int) a`
> * 함수형 형변환: `int(a)`

Named cast의 목록:

* `static_cast`
* `const_cast`
* `reinterpret_cast`
* `dynamic_cast`
* `std::move` // `move(x)`는 `x`에 대한 r-value 참조를 반환한다
* `std::forward` // `forward(x)`는 `x`에 대한 r-value 참조를 반환한다
* `gsl::narrow_cast` // `narrow_cast<T>(x)`는 `static_cast<T>(x)`와 동일하다
* `gsl::narrow` // `narrow<T>(x)`는 `static_cast<T>(x)`와 동일하며, 만약 `static_cast<T>(x) == x`가 아니면 예외 `narrowing_error`를 던진다

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

이 예시는 `D`가 `B`의 하위 타입이면서, 누군가 계층 구조를 리팩토링 했을때 발생한 실제 버그들을 종합한 것이다.
C 스타일 타입변환이 위험한 이유는 어떤 형태로의 변환도 수행할 수 있기 때문이다. 이는 실수로부터 우리를 보호해주지 않는다 (지금도, 앞으로도). 

##### Note

정보의 손실이 없는 타입 변환의 경우 (가령 `float`에서 `double`로, 혹은 `int32`에서 `int64`로 변환하는 경우), `{}` 초기화를 대신 사용할수도 있다.

```c++
    double d {some_float};
    int64_t i {some_int32};
```

이 코드는 타입 변환을 의도했다는 것이 분명히 드러나고 정확도를 잃을 수 있는 변환을 예방한다.
(예를 들자면 이런 코드에서 `float`를 `double`로 초기화 하는 것은 컴파일 오류가 된다)

##### Note

`reinterpret_cast`가 필수적일 수 있다. 하지만 본질적으로 타입 안전하지는 않다 (기계 주소를 포인터로 바꾼다던가):

```c++
    auto p = reinterpret_cast<Device_register>(0x800);  // 필연적으로 위험하다
```

##### Enforcement

* C스타일, 함수형 형변환이 있다면 지적한다
* The [type profile](./Profile.md#Pro-type-reinterpretcast) bans `reinterpret_cast`.
* The [type profile](./Profile.md#Pro-type-arithmeticcast) warns when using `static_cast` between arithmetic types.

### <a name="Res-casts-const"></a>ES.50: `const`를 제거하지 마라

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

경우에 따라서는 코드 중복을 피하고자 `const_cast`에 의존하고 싶을수도 있다.
이때는 두 함수가 구현은 유사하지만 오직 `const` 부분만 다를 것이다. 예를 들어:

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

대신, 구현 코드를 공유하도록 하라. 보통의 경우, 비-`const` 함수에서 `const` 함수를 호출할 수 있다.
하지만 그 구현에 복잡한 로직이 있다면 여전히 `const_cast`를 사용하는 다음과 같은 패턴을 쓰게 될 것이다:

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

이 패턴은 정확하게 사용되었을 때는 호출자가 비-`const` 개체를 통해 호출하기 때문에 안전하지만, 안전성이 검사기의 규칙만큼 자연스럽게 강제되기는 어렵기 때문에 이상적인 코드는 아니다.

이런 패턴 대신, 공통되는 코드는 공통된 보조 함수(helper function)에 배치하라 -- 그리고 `const`를 타입 추론에서 찾아내도록 템플릿으로 만들어라. 이는 `const_cast`이 완전히 필요없게 만든다:

```c++
    class Foo {
    public:                 // good
              Bar& get_bar()       { return get_bar_impl(*this); }
        const Bar& get_bar() const { return get_bar_impl(*this); }
    private:
        Bar my_bar;

        template<class T>   // good, deduces whether T is const or non-const
        static auto get_bar_impl(T& t) -> decltype(t.get_bar()){ 
            // the complex logic around getting 
            // a possibly-const reference to my_bar
        }
    };
```

##### Exception

You may need to cast away `const` when calling `const`-incorrect functions.
Prefer to wrap such functions in inline `const`-correct wrappers to encapsulate the cast in one place.

##### Example

보통 `const`를 없애버리는 이유는 변경할 수 없는 객체 속에 있는 일시적인 정보를 변경하기 위해서이다.
예를 들면 캐싱값, 임시계산값, 선계산값 등이다.
이런 값은 `const_cast`를 쓰는 것보다 `mutable`이나 간접적인 방법을 사용하면 더 쉽게 처리할 수 있다.

비용이 드는 처리를 거쳐서 계산한 결과를 유지하는 것을 고려해보라:

```c++
    int compute(int x); // compute a value for x; assume this to be costly

    class Cache {   // int->int 처리에서 캐시를 구현한 타입
    public:
        pair<bool, int> find(int x) const;  // x를 위한 값이 있는가?
        void set(int x, int v);             // x를 위한 값 y를 만든다
        // ...
    private:
        // ...
    };

    class X {
    public:
        int get_val(int x)
        {
            auto p = cache.find(x);
            if (p.first)
                return p.second;

            int val = compute(x);
            cache.set(x, val); // x에 값을 넣는다
            return val;
        }
        // ...
    private:
        Cache cache;
    };
```

여기서 `get_val()`는 논리적으로는 상수이다. 따라서 `const`멤버로 만들 수 있을 것이다.
이렇게 하려면 여전히 `cache`를 변경해야 한다. 일부는 그러지 않고 `const_cast`를 사용한다:

```c++
    class X {   // Suspicious solution based on casting
    public:
        int get_val(int x) const
        {
            auto p = cache.find(x);
            if (p.first)
                return p.second;
            int val = compute(x);
            const_cast<Cache&>(cache).set(x, val);   // ugly
            return val;
        }
        // ...
    private:
        Cache cache;
    };
```

다행히, 더 나은 해결책이 있다:
`cache`가 `const` 개체여도 변경 가능하다고 표기(state)하는 것이다:

```c++
    class X {   // better solution
    public:
        int get_val(int x) const
        {
            auto p = cache.find(x);
            if (p.first)
                return p.second;

            int val = compute(x);
            cache.set(x, val);
            return val;
        }
        // ...
    private:
        mutable Cache cache;
    };
```

다른 방법은 `cache`에 대한 포인터를 보관하는 것이다:

```c++
    class X {   // OK, but slightly messier solution
    public:
        int get_val(int x) const
        {
            auto p = cache->find(x);
            if (p.first)
                return p.second;

            int val = compute(x);
            cache->set(x, val);
            return val;
        }
        // ...
    private:
        unique_ptr<Cache> cache;
    };
```

이 해결책은 굉장히 유연하지만, `cache`로 가리키는 개체의 명시적인 생성과 소멸을 필요로 한다.
(아마 그 코드는 `X`의 생성자와 소멸자에 위치할 것이다).

멀티스레드 코드에서 `cache`에 데이터 경쟁이 발생하면 `std::mutex`를 사용해 보호해야 한다.

##### Enforcement

* `const_cast`를 지적한다.
* 이 규칙은 [타입 안정성 분석](#Pro-type-constcast)과 관련 있다

### <a name="Res-range-checking"></a>ES.55: 범위 검사가 필요없게 하라

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

### <a name="Res-move"></a>ES.56: `std::move()`는 개체를 다른 유효범위로 명시적으로 옮겨야 할때만 사용하라

##### Reason

복제를 막고 성능을 향상시키기 위해 복사보다는 이동을 사용한다.

이동 연산은 보통 빈 개체를 남긴다 ([C.64](#Rc-move-semantic)). 이는 기대밖의 결과 혹은 위험으로 이어질 수 있다. 가능하다면 lvalue로부터 이동하는 것을 피하려 해야한다 (lvalue에 나중에 접근할 수도 있다).

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

##### See also

* 수명주기 문제를 피하려면 [RAII](#Rr-raii)를 사용하라
* 수명주기 문제를 피하려면 [unique_ptr](#Rf-unique_ptr)를 사용하라
* 수명주기 문제를 피하려면 [shared_ptr](#Rf-shared_ptr)를 사용하라
* `nullptr`가 허용되지 않는다면 [references](#Rf-ptr-ref)를 사용하라
* 의도치 않은 `nullptr`를 일찍 잡아내기 위해 [not_null](#Rf-not_null)을 사용하라
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

        *p = 42;            // BAD, p는 분기문을 거쳤다면 유효하지 않은 값을 가지고 있다
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
