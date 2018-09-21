# <a name="S-expr"></a> ES: 식과 문

식과 문은 가장 낮은 레벨에서 가장 직접적으로 행동과 연산을 표현하는 방식이다. 로컬 스코프에서 선언하는 것을 '문(statements)이라고 한다.
Expressions and statements are the lowest and most direct way of expressing actions and computation. Declarations in local scopes are statements.

네이밍과 주석달기, 들여쓰기 룰에 대해서는,[NL: Naming and layout](#S-naming)을 참고하길 바란다.
For naming, commenting, and indentation rules, see [NL: Naming and layout](#S-naming).

일반적인 규칙:
General rules:

* [ES.1: 다른 라이브러리나 "직접 짠 코드" 대신 표준 라이브러리를 쓰자](#Res-lib)
* [ES.2: 언어의 기능을 직접적으로 사용하기 보다는 적절한 추상화를 하자](#Res-abstr)
* [ES.1: Prefer the standard library to other libraries and to "handcrafted code"](#Res-lib)
* [ES.2: Prefer suitable abstractions to direct use of language features](#Res-abstr)

선언 규칙:
Declaration rules:

* [ES.5: 스코프를 작게 유지하자](#Res-scope)
* [ES.6: for-구문 초기화나 조건 설정에 사용하는 변수는 스코프 안으로 한정하자](#Res-cond)
* [ES.7: 자주 쓰는 변수, 지역변수는 이름을 짧게, 그렇지 않은 경우는 길게](#Res-name-length)
* [ES.8: 비슷해보이는 네이밍은 피하자](#Res-name-similar)
* [ES.9: `ALL_CAPS` 네이밍은 피하자](#Res-!CAPS)
* [ES.10: 한 선언당 (단) 하나의 변수만 선언하자 ](#Res-name-one)
* [ES.11: 타입명을 풍부하게 재사용할 수 있도록 'auto'를 쓰자](#Res-auto)
* [ES.20: 객체를 언제나 초기화하자](#Res-always)
* [ES.21: 변수(혹은 상수)를 필요하기도 전에 선언하지 말자](#Res-introduce)
* [ES.22: 초기화할 값도 없는데 변수부터 선언하지 말자](#Res-init)
* [ES.23: '{}' 초기화 구문을 쓰자](#Res-list)
* [ES.24: Use a `unique_ptr<T>` to hold pointers in code that may throw](#Res-unique)
* [ES.25: 나중에 값을 바꿀 게 아니라면 'const'나 constexpr'로 객체를 선언하자](#Res-const)
* [ES.26: 변수 하나를 연관 없는 두 목적을 위해 쓰지 말자](#Res-recycle)
* [ES.27: 배열과 스택을 만들 때는 'std::array' 나 'stack_array'를 쓰자](#Res-stack)
* [ES.28: 복잡한 초기화를 위해서는 람다식을 쓰자. 특히나 'const' 상수에 대해서는.](#Res-lambda-init)
* [ES.30: 프로그램 텍스트 변조를 할때 매크로를 사용하지말자](#Res-macros)
* [ES.31: 상수나 "함수"를 매크로로 정의하지 말자](#Res-macros2)
* [ES.32: 매크로는 `ALL_CAPS`네이밍을 하자](#Res-CAPS!)
* [ES.40: C 스타일의 variadic 함수를 정의하지 말자](#Res-ellipses)

* [ES.5: Keep scopes small](#Res-scope)
* [ES.6: Declare names in for-statement initializers and conditions to limit scope](#Res-cond)
* [ES.7: Keep common and local names short, and keep uncommon and nonlocal names longer](#Res-name-length)
* [ES.8: Avoid similar-looking names](#Res-name-similar)
* [ES.9: Avoid `ALL_CAPS` names](#Res-!CAPS)
* [ES.10: Declare one name (only) per declaration](#Res-name-one)
* [ES.11: Use `auto` to avoid redundant repetition of type names](#Res-auto)
* [ES.20: Always initialize an object](#Res-always)
* [ES.21: Don't introduce a variable (or constant) before you need to use it](#Res-introduce)
* [ES.22: Don't declare a variable until you have a value to initialize it with](#Res-init)
* [ES.23: Prefer the `{}`-initializer syntax](#Res-list)
* [ES.24: Use a `unique_ptr<T>` to hold pointers in code that may throw](#Res-unique)
* [ES.25: Declare an object `const` or `constexpr` unless you want to modify its value later on](#Res-const)
* [ES.26: Don't use a variable for two unrelated purposes](#Res-recycle)
* [ES.27: Use `std::array` or `stack_array` for arrays on the stack](#Res-stack)
* [ES.28: Use lambdas for complex initialization, especially of `const` variables](#Res-lambda-init)
* [ES.30: Don't use macros for program text manipulation](#Res-macros)
* [ES.31: Don't use macros for constants or "functions"](#Res-macros2)
* [ES.32: Use `ALL_CAPS` for all macro names](#Res-CAPS!)
* [ES.40: Don't define a (C-style) variadic function](#Res-ellipses)

식의 규칙 :
Expression rules:
* [ES.40: 복잡한 식은 피하자](#Res-complicated)
* [ES.41: 연산자 우선순위가 햇갈릴 때는 괄호를 쓰자](#Res-parens)
* [ES.42: 포인터는 단순하고 직관적으로 사용하자](#Res-ptr)
* [ES.43: 값을 구하는 순서가 정의되지 않은 식은 피하자](#Res-order)
* [ES.44: 함수 아규먼트의 연산 순서에 의지하지 말자](#Res-order-fct)
* [ES.45: 컨버전을 좁히는 것을 피하자](#Res-narrowing)
* [ES.46: "만능 상수"를 피하자. 상징적인 상수를 사용하자](#Res-magic)
* [ES.47: '0'이나 'NULL' 대신 `nullptr`을 쓰자](#Res-nullptr)
* [ES.48: 캐스팅을 피하자](#Res-casts)
* [ES.49: 캐스팅을 해야할 때는 명시된 캐스팅을 하자](#Res-casts-named)
* [ES.50: `const`를 내버려두지 말자](#Res-casts-const)
* [ES.55: 범위 체크를 불필요하게 하자](#Res-range-checking)
* [ES.60: 자원 관리 함수 바깥에서 `new` 와 `delete[]` 를 쓰는 것을 피하자](#Res-new)
* [ES.61: 배열은 `delete[]`로 배열이 아닌 것은 `delete`로 삭제하자](#Res-del)
* [ES.62: 상이한 배열로 포인터를 비교하지 말자](#Res-arr2)

* [ES.40: Avoid complicated expressions](#Res-complicated)
* [ES.41: If in doubt about operator precedence, parenthesize](#Res-parens)
* [ES.42: Keep use of pointers simple and straightforward](#Res-ptr)
* [ES.43: Avoid expressions with undefined order of evaluation](#Res-order)
* [ES.44: Don't depend on order of evaluation of function arguments](#Res-order-fct)
* [ES.45: Avoid narrowing conversions](#Res-narrowing)
* [ES.46: Avoid "magic constants"; use symbolic constants](#Res-magic)
* [ES.47: Use `nullptr` rather than `0` or `NULL`](#Res-nullptr)
* [ES.48: Avoid casts](#Res-casts)
* [ES.49: If you must use a cast, use a named cast](#Res-casts-named)
* [ES.50: Don't cast away `const`](#Res-casts-const)
* [ES.55: Avoid the need for range checking](#Res-range-checking)
* [ES.60: Avoid `new` and `delete[]` outside resource management functions](#Res-new)
* [ES.61: delete arrays using `delete[]` and non-arrays using `delete`](#Res-del)
* [ES.62: Don't compare pointers into different arrays](#Res-arr2)

구문 규칙:
Statement rules:

* [ES.70: 선택을 할때는 'if' 문대신 'switch' 문을 사용하자](#Res-switch-if)
* [ES.71: 선택을 할때는 그냥 'for'  대신 range-`for`를 사용하자](#Res-for-range)
* [ES.72: 확실한 반복 변수가 있을 때는 'while'보다는 'for' 문을 사용하자](#Res-for-while)
* [ES.73: 확실한 반복 변수가 없을 때는 'for'문보다 'while'문을 사용하자](#Res-while-for)
* [ES.74: 'for'문의 초기화 부분에서 반복 변수를 선언하자](#Res-for-init)
* [ES.75: `do`문 사용을 피하자](#Res-do)
* [ES.76: `goto`문 사용을 피하자](#Res-goto)
* [ES.77: ??? `continue`](#Res-continue)
* [ES.78: 내용이 있는 `case` 문은 `break`로 끝내자](#Res-break)
* [ES.79: ??? `default`](#Res-default)
* [ES.85: 빈 구문을 보이게 만들자](#Res-empty)

* [ES.70: Prefer a `switch`-statement to an `if`-statement when there is a choice](#Res-switch-if)
* [ES.71: Prefer a range-`for`-statement to a `for`-statement when there is a choice](#Res-for-range)
* [ES.72: Prefer a `for`-statement to a `while`-statement when there is an obvious loop variable](#Res-for-while)
* [ES.73: Prefer a `while`-statement to a `for`-statement when there is no obvious loop variable](#Res-while-for)
* [ES.74: Prefer to declare a loop variable in the initializer part of as `for`-statement](#Res-for-init)
* [ES.75: Avoid `do`-statements](#Res-do)
* [ES.76: Avoid `goto`](#Res-goto)
* [ES.77: ??? `continue`](#Res-continue)
* [ES.78: Always end a non-empty `case` with a `break`](#Res-break)
* [ES.79: ??? `default`](#Res-default)
* [ES.85: Make empty statements visible](#Res-empty)

연산 규칙:
Arithmetic rules:

* [ES.100: 부호가 있는 연산과 없는 연산을 섞지 말자](#Res-mix)
* [ES.101: 비트 조작을 할때는 부호가 없는(unsigned) 타입을 사용하자](#Res-unsigned)
* [ES.102: 연산할때는 부호가 있는(signed) 타입을 사용하자](#Res-signed)
* [ES.103: 오버플로우 금지](#Res-overflow)
* [ES.104: 언더플로우 금지](#Res-underflow)
* [ES.105: 0으로 나누기 금지](#Res-zero)

* [ES.100: Don't mix signed and unsigned arithmetic](#Res-mix)
* [ES.101: use unsigned types for bit manipulation](#Res-unsigned)
* [ES.102: Used signed types for arithmetic](#Res-signed)
* [ES.103: Don't overflow](#Res-overflow)
* [ES.104: Don't underflow](#Res-underflow)
* [ES.105: Don't divide by zero](#Res-zero)


### <a name="Res-lib"></a> ES.1: 다른 라이브러리나 "직접 짠 코드" 대신 표준 라이브러리를 쓰자
### <a name="Res-lib"></a> ES.1: Prefer the standard library to other libraries and to "handcrafted code"

##### 이유
##### Reason

라이브러리를 사용하는 코드는 언어의 기능을 직접적으로 사용하는 것보다 작성하기 쉽고, 더 짧게 작성할 수 있고, 고수준의 추상화가 된다. 그리고, 라이브러리의 코드는 대개는 이미 테스트되어 있다.
ISO C++ 표준 라이브러리는 널리 알려져있으며 테스트가 잘된 라이브러리다.
그리고 모든 C++ 구현체에서 사용할 수 있다.
Code using a library can be much easier to write than code working directly with language features, much shorter, tend to be of a higher level of abstraction, and the library code is presumably already tested.
The ISO C++ standard library is among the most widely know and best tested libraries.
It is available as part of all C++ Implementations.

##### 예
##### Example

    auto sum = accumulate(begin(a), end(a), 0.0);	// 괜찮음

`accumulate`의 범위 버전이 더 낫다:

    auto sum = accumulate(v, 0.0); // 더 좋음

그런데 잘 알려진 알고리즘을 직접 만들지는 말자:

    int max = v.size();		// 안좋다. 장황하고 목적이 불분명하다.
    double sum = 0.0;
    for (int i = 0; i < max; ++i)
        sum = sum + v[i];

**예외**: 표준라이브러리의 대다수가 동적 할당(자유 저장소)에 의존한다. 이런 부분은 알고리즘의 문제는 아닐지라도, 컨테이너 레벨에서 실시간 정확성이 요구되는 상황이나, 임베디드 어플리케이션에는 잘 맞지 않는다. 이런 경우에는, 비슷한 기능을 구현하여 사용하는 것을 고려해볼 수 있다. 예를 들면 표준 라이브러리 스타일로 구현된 메모리 풀 할당 컨테이너 같은 것들이다.

    auto sum = accumulate(begin(a), end(a), 0.0);	// good

a range version of `accumulate` would be even better:

    auto sum = accumulate(v, 0.0); // better

but don't hand-code a well-known algorithm:

    int max = v.size();		// bad: verbose, purpose unstated
    double sum = 0.0;
    for (int i = 0; i < max; ++i)
        sum = sum + v[i];

**Exception**: Large parts of the standard library rely on dynamic allocation (free store). These parts, notably the containers but not the algorithms, are unsuitable for some hard-real time and embedded applications. In such cases, consider providing/using similar facilities, e.g.,  a standard-library-style container implemented using a pool allocator.

##### 강제사항
##### Enforcement

Not easy. ??? Look for messy loops, nested loops, long functions, absence of function calls, lack of use of non-built-in types. Cyclomatic complexity?



### <a name="Res-abstr"></a> ES.2: 언어의 기능을 직접적으로 사용하기 보다는 적절한 추상화를 하자
### <a name="Res-abstr"></a> ES.2: Prefer suitable abstractions to direct use of language features

##### 이유
##### Reason

"적절한 추상화"(예를 들어 라이브러리나 클래스 같은 것)가 민짜 언어보다 어플리케이션의 컨셉에 더 가깝다. 적절한 추상화를 사용하면 코드를 짧고 간결하게 만들수 있으며, 테스트하기도 더 쉽다.
A "suitable abstraction" (e.g., library or class) is closer to the application concepts than the bare language, leads to shorter and clearer code, and is likely to be better tested.

##### 예
##### Example

    vector<string> read1(istream& is)	// 좋다
    {
        vector<string> res;
        for (string s; is >> s;)
            res.push_back(s);
        return res;
    }

아래와 같은 전통적인 코드, 시스템 레벨과 거의 동등한 로우레벨 코드는 길고, 지저분하고, 이해하기도 어렵고, 느리게 돌아간다.

    char** read2(istream& is, int maxelem, int maxstring, int* nread)	// 나쁘다: 장황하고, 미완성된 코드다
    {
        auto res = new char*[maxelem];
        int elemcount = 0;
        while (is && elemcount < maxelem) {
            auto s = new char[maxstring];
            is.read(s, maxstring);
            res[elemcount++] = s;
        }
        nread = elemcount;
        return res;
    }

오버플로우나 에러 핸들링 코드가 일단 한 번 들어가게 되면, 코드는 확 지저분해진다. 그리고, 리턴하는 포인터와 배열로 구현되는 C스타일의 스트링을 `delete`를 꼭 해줘야하는 문제도 있다.



    vector<string> read1(istream& is)	// good
    {
        vector<string> res;
        for (string s; is >> s;)
            res.push_back(s);
        return res;
    }

The more traditional and lower-level near-equivalent is longer, messier, harder to get right, and most likely slower:

    char** read2(istream& is, int maxelem, int maxstring, int* nread)	// bad: verbose and incomplete
    {
        auto res = new char*[maxelem];
        int elemcount = 0;
        while (is && elemcount < maxelem) {
            auto s = new char[maxstring];
            is.read(s, maxstring);
            res[elemcount++] = s;
        }
        nread = elemcount;
        return res;
    }

Once the checking for overflow and error handling has been added that code gets quite messy, and there is the problem remembering to `delete` the returned pointer and the C-style strings that array contains.

##### 강제사항
##### Enforcement

Not easy. ??? Look for messy loops, nested loops, long functions, absence of function calls, lack of use of non-built-in types. Cyclomatic complexity?

## ES.dcl: 선언
## ES.dcl: Declarations

선언은 문(statement)이다. 선언은 한 스코프에 변수를 알려주고, 명명된 객체를 생성하게 된다.
A declaration is a statement. a declaration introduces a name into a scope and may cause the construction of a named object.

### <a name="Res-scope"></a> ES.5: 스코프를 작게 유지하자
### <a name="Res-scope"></a> ES.5: Keep scopes small

##### 이유
##### Reason

가독성이 좋아진다. 리소스 점유를 최소화할 수 있다. 값의 잘못된 사용을 피할 수 있다.
Readability. Minimize resource retention. Avoid accidental misuse of value.

**달리 말하자면**: 불필요하게 큰 스코프에 변수를 선언하지 말자
**Alternative formulation**: Don't declare a name in an unnecessarily large scope.

##### 예
##### Example

    void use()
    {
        int i;									// 나쁘다: 루프 종료 후에도 i에 접근할 필요가 없다
        for (i = 0; i < 20; ++i) { /* ... */ }
        // 이 부분에서 i를 써서는 안된다
        for (int i = 0; i < 20; ++i) { /* ... */ }  // 좋다: i가 for 루프의 지역변수로 선언되었다
     
        if (auto pc = dynamic_cast<Circle*>(ps)) {  // 좋다: pc가 if문의 지역변수로 선언되었다
            // ... Circle과 관련된 코드 ...
        }
        else {
            // ... 에러 핸들 코드 ...
        }
    }


    void use()
    {
        int i;									// bad: i is needlessly accessible after loop
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

##### 나쁜 예
##### Example, bad

    void use(const string& name)
    {
        string fn = name+".txt";
        ifstream is {fn};
        Record r;
        is >> r;
        // ... 여기에는 fn과 is를 쓰면 안되는 200 줄짜리 코드가 들어간다 ...
    }

무엇보다 이 코드는 길다는 문제점이 있지만, `fn`의 값과 `is`가 갖고 있는 파일 핸들러 필요 이상으로 길게 값이 유지된다는 게 가장 큰 문제다. 이러면 함수의 뒷부분에서 `is`와 `fn`을 실수로 사용해버릴 수 있다. 이럴 때는, 분할해버리는 게 낫다.

    void fill_record(Record& r, const string& name)
    {
        string fn = name+".txt";
        ifstream is {fn};
        Record r;
        is >> r;
    }

    void use(const string& name)
    {
        Record r;
        fill_record(r, name);
        // ... 200 lines of code ...
    }

이 코드에서 `Record`는 사이즈가 크고, 이동 기능도 별로인 것 같다. 그래서, `Record`를 리턴하는 것보다는 아웃파라미터를 쓰는 게 더 나아보인다.

    void use(const string& name)
    {
        string fn = name+".txt";
        ifstream is {fn};
        Record r;
        is >> r;
        // ... 200 lines of code without intended use of fn or is ...
    }

This function is by most measure too long anyway, but the point is that the used by `fn` and the file handle held by `is`
are retained for much longer than needed and that unanticipated use of `is` and `fn` could happen later in the function.
In this case, it might be a good idea to factor out the read:

    void fill_record(Record& r, const string& name)
    {
        string fn = name+".txt";
        ifstream is {fn};
        Record r;
        is >> r;
    }

    void use(const string& name)
    {
        Record r;
        fill_record(r, name);
        // ... 200 lines of code ...
    }

I am assuming that `Record` is large and doesn't have a good move operation so that an out-parameter is preferable to returning a `Record`.

##### 강조사항
##### Enforcement

* 루프 바깥에서 루프 변수가 선언되고 이후에는 사용되지 않을 때를 주의하라
* 파일 핸들이나 락과 같은 중요한 리소스를 사용하는 코드가 장황해질 때 주의하라

* Flag loop variable declared outside a loop and not used after the loop
* Flag when expensive resources, such as file handles and locks are not used for N-lines (for some suitable N)

### <a name="Res-cond"></a> ES.6: for-구문 초기화나 조건 설정에 사용하는 변수는 스코프 안으로 한정하자
### <a name="Res-cond"></a> ES.6: Declare names in for-statement initializers and conditions to limit scope

##### 이유
##### Reason

가독성. 낮은 시스템 자원 점유.
Readability. Minimize resource retention.

##### 예
##### Example

    void use()
    {
        for (string s; cin >> s;)
            v.push_back(s);

        for (int i = 0; i < 20; ++i) {	// 좋다. i가 for 루프의 지역 변수로 선언됐다.
            // ...
        }

        if (auto pc = dynamic_cast<Circle*>(ps)) {	// 좋다. pc가 if문의 지역변수로 선언되었다.
            // ... Circle을 다루는 코드 ...
        }
        else {
            // ... 에러 핸들링 코드 ...
        }
    }


    void use()
    {
        for (string s; cin >> s;)
            v.push_back(s);

        for (int i = 0; i < 20; ++i) {	// good: i is local to for-loop
            // ...
        }

        if (auto pc = dynamic_cast<Circle*>(ps)) {	// good: pc is local to if-statement
            // ... deal with Circle ...
        }
        else {
            // ... handle error ...
        }
    }

##### 강조사항
##### Enforcement

* 루프 바깥에서 루프 변수가 선언되고 이후에는 사용되지 않을 때를 주의하라
* (중요) 루프 바깥에서 루프 변수를 선언하고, 루프가 끝난 뒤에 관계없는 목적으로 그 변수를 사용할 때를 주의하라

* Flag loop variables declared before the loop and not used after the loop
* (hard) Flag loop variables declared before the loop and used after the loop for an unrelated purpose.

### <a name="Res-name-length"></a> ES.7: 자주 쓰는 변수, 지역변수는 이름을 짧게, 그렇지 않은 경우는 길게
### <a name="Res-name-length"></a> ES.7: Keep common and local names short, and keep uncommon and nonlocal names longer

##### 이유
##### Reason

가독성.  관계없는 전역변수들의 충돌 확률을 낮추기 위하여.
Readability. Lowering the chance of clashes between unrelated non-local names.

##### 예
##### Example

관습적으로 쓰이는 짧은 지역변수명은 가독성을 향상시킨다.

    template<typename T>							// 좋다
    void print(ostream& os, const vector<T>& v)
    {
        for (int i = 0; i < v.end(); ++i)
            os << v[i] << '\n';
    }

인덱스는 관습적으로 `i`라고 쓰고, 이 일반 함수에는 벡터의 의미를 알만한 힌트가 없으므로, `v`가 어떤 경우에든지 맞는 이름이다. 아래와 비교를 해보자.

    template<typename Element_type>					// 별로다. 장황하고 가독성이 떨어진다
    void print(ostream& target_stream, const vector<Element_type>& current_vector)
    {
        for (int current_element_index = 0;
                current_element_index < current_vector.end();
                ++current_element_index
        )
        target_stream << current_vector[i] << '\n';
    }

과장해서 표현하긴 했지만, 이것보다 더 심한 것도 본적이 있다.


Conventional short, local names increase readability:

    template<typename T>							// good
    void print(ostream& os, const vector<T>& v)
    {
        for (int i = 0; i < v.end(); ++i)
            os << v[i] << '\n';
    }

An index is conventionally called `i` and there is no hint about the meaning of the vector in this generic function, so `v` is as good name as any. Compare

    template<typename Element_type>					// bad: verbose, hard to read
    void print(ostream& target_stream, const vector<Element_type>& current_vector)
    {
        for (int current_element_index = 0;
                current_element_index < current_vector.end();
                ++current_element_index
        )
        target_stream << current_vector[i] << '\n';
    }

Yes, it is a caricature, but we have seen worse.

##### 예
##### Example

관습에 따르지 않는 짧은 比지역 변수는 코드를 모호하게 만든다.

    void use1(const string& s)
    {
        // ...
        tt(s);		// 안좋다. tt()란 무엇일까?
        // ...
    }

전역 엔티티에 가독성있는 이름을 부여하면 좀 나아진다.

    void use1(const string& s)
    {
        // ...
        trim_tail(s);		// 이게 더 낫다
        // ...
    }

이렇게 하면, 코드를 읽는 사람이 `trim_tail`의 의미를 알 수 있게 되고, 기억할 수 있게 된다.

Unconventional and short non-local names obscure code:

    void use1(const string& s)
    {
        // ...
        tt(s);		// bad: what is tt()?
        // ...
    }

Better, give non-local entities readable names:

    void use1(const string& s)
    {
        // ...
        trim_tail(s);		// better
        // ...
    }

Here, there is a chance that the reader knows what `trim_tail` means and that the reader can remember it after looking it up.

#### 나쁜 예
##### Example, bad

큰 사이즈의 함수의 인자는 사실상 比지역변수라고 볼 수 있다. 그래서 의미를 갖춰야 한다.

    void complicated_algorithm(vector<Record>&vr, const vector<int>& vi, map<string, int>& out)
    // vr(사용된 레코드를 마킹한다)에서 이벤트를 읽어와서 vi의 인덱스로 넘기고 out으로 빼낸다
    {
        // ... vr, vi, out을 사용하는 500줄 코드 ...
    }

함수는 짧게 유지하는 것을 권장하지만, 이 룰을 모두 적용시키긴 힘들 때가 있다. 그럴 경우엔 변수명을 적절히 줘야 한다.


Argument names of large functions are de facto non-local and should be meaningful:

    void complicated_algorithm(vector<Record>&vr, const vector<int>& vi, map<string, int>& out)
    // read from events in vr (marking used Records) for the indices in vi placing (name, index) pairs into out
    {
        // ... 500 lines of code using vr, vi, and out ...
    }

We recommend keeping functions short, but that rule isn't universally adhered to and naming should reflect that.

##### 강조사항
##### Enforcement

지역 변수와 전역변수의 길이를 체크하자. 함수도 이런 점을 고려하자.
Check length of local and non-local names. Also take function length into account.

### <a name="Res-name-similar"></a> ES.8: Avoid similar-looking names

##### Reason

Such names slow down comprehension and increase the likelihood of error.

##### Example

    if (readable(i1 + l1 + ol + o1 + o0 + ol + o1 + I0 + l0)) surprise();

##### Enforcement

Check names against a list of known confusing letter and digit combinations.

### <a name="Res-!CAPS"></a> ES.9: `ALL_CAPS` 네이밍은 피하자
>### <a name="Res-!CAPS"></a> ES.9: Avoid `ALL_CAPS` names

##### 이유
>##### Reason

그런 변수명은 매크로를 정의할 때 흔히들 쓴다. 그러므로, `ALL_CAPS` 네이밍은 매크로와 충돌될 가능성이 많다. 
>Such names are commonly used for macros. Thus, `ALL_CAPS` name are vulnerable to unintended macro substitution.

##### 예
##### Example

    // 헤더의 한 부분:
    #define NE !=

    // 다른 헤더의 또 어떤 부분:
    enum Coord { N, NE, NW, S, SE, SW, E, W };

    // 이 헤더를 사용하는 어느 불쌍한 프로그래머의 cpp 파일
    switch (direction) {
    case N:
        // ...
    case NE:
        // ...
    // ...
    }

>
    // somewhere in some header:
    #define NE !=
>
    // somewhere else in some other header:
    enum Coord { N, NE, NW, S, SE, SW, E, W };
>
    // somewhere third in some poor programmer's .cpp:
    switch (direction) {
    case N:
        // ...
    case NE:
        // ...
    // ...
    }

##### 요약
>##### Note

단지 상수가 매크로처럼 쓰인다는 이유로 상수에 `ALL_CAPS` 네이밍을 하지는 말자.
>Do not use `ALL_CAPS` for constants just because constants used to be macros.

##### 시행하기
>##### Enforcement

ALL CAPS 변수가 어디서 쓰이고 있는지 체크하자. 옛날 코드라면, 매크로 변수에 ALL CAPS를 허용하고 대신 소문자로 쓰여진 매크로 변수를 체크하자.
>Flag all uses of ALL CAPS. For older code, accept ALL CAPS for macro names and flag all non-ALL-CAPS macro names.

### <a name="Res-name-one"></a> 한 선언당 (단) 하나의 변수만 선언하자
>### <a name="Res-name-one"></a> ES.10: Declare one name (only) per declaration

##### 이유
>##### Reason

한 줄에 선언 하나씩 하면 가독성을 향상시킬 수 있고, C/C++ 문법과 관련된 실수를 피할 수 있다. 그리고 `//`주석을 달 수 있는 공간이 생긴다.
>One-declaration-per line increases readability and avoid mistake related to the C/C++ grammar. It leaves room for a `//`-comment.

##### 나쁜 예
>##### Example, bad

       char *p, c, a[7], *pp[7], **aa[10];	// 우웩!
>       char *p, c, a[7], *pp[7], **aa[10];	// yuck!

**예외**: 함수 정의는 여러 함수 인자 정의를 포함할 수 있다.
>**Exception**: a function declaration can contain several function argument declarations.

##### 예
>##### Example
	
	template <class InputIterator, class Predicate>
    bool any_of(InputIterator first, InputIterator last, Predicate pred);

>
    template <class InputIterator, class Predicate>
    bool any_of(InputIterator first, InputIterator last, Predicate pred);

이것보다 더 나은 사용 예는..
>or better using concepts:

    bool any_of(InputIterator first, InputIterator last, Predicate pred);
>
	bool any_of(InputIterator first, InputIterator last, Predicate pred);

##### 예
##### Example

    double scalbn(double x, int n);	 	// 괜찮다: x*pow(FLT_RADIX, n); FLT_RADIX는 일반적으로 2입니다

또는,

    double scalbn(    // 좀 더 낫다: x*pow(FLT_RADIX, n); FLT_RADIX는 일반적으로 2입니다
        double x,     // 기수부입니다
        int n         // 지수부입니다
    );

또는,

    double scalbn(double base, int exponent);	// 좋다: base*pow(FLT_RADIX, exponent); FLT_RADIX는 일반적으로 2입니다

>
    double scalbn(double x, int n);	 	// OK: x*pow(FLT_RADIX, n); FLT_RADIX is usually 2
>
or:
>
    double scalbn(    // better: x*pow(FLT_RADIX, n); FLT_RADIX is usually 2
        double x,     // base value
        int n         // exponent
    );
>
or:
>
    double scalbn(double base, int exponent);	// better: base*pow(FLT_RADIX, exponent); FLT_RADIX is usually 2

##### 시행하기
##### Enforcement

함수 정의도 아니면서, 선언자로 다중 선언을 한 곳을 체크하자.(예를 들어, `int* p, q;`같은 것)
>Flag non-function arguments with multiple declarators involving declarator operators (e.g., `int* p, q;`)

### <a name="Res-auto"></a> ES.11: Use `auto` to avoid redundant repetition of type names

##### Reason

* Simple repetition is tedious and error prone.
* When you use `auto`, the name of the declared entity is in a fixed position in the declaration, increasing readability.
* In a template function declaration the return type can be a member type.

##### Example

Consider:

    auto p = v.begin();	// vector<int>::iterator
    auto s = v.size();
    auto h = t.future();
    auto q = new int[s];
    auto f = [](int x){ return x + 10; };

In each case, we save writing a longish, hard-to-remember type that the compiler already knows but a programmer could get wrong.

##### Example

    template<class T>
    auto Container<T>::first() -> Iterator;	// Container<T>::Iterator

**Exception**: Avoid `auto` for initializer lists and in cases where you know exactly which type you want and where an initializer might require conversion.

##### Example

    auto lst = { 1, 2, 3 };	// lst is an initializer list (obviously)
    auto x{1};	// x is an int (after correction of the C++14 standard; initializer_list in C++11)

##### Note

When concepts become available, we can (and should) be more specific about the type we are deducing:

    // ...
    ForwardIterator p = algo(x, y, z);

##### Enforcement

Flag redundant repetition of type names in a declaration.

### <a name="Res-always"></a> ES.20: Always initialize an object

##### Reason

Avoid used-before-set errors and their associated undefined behavior.
Avoid problems with comprehension of complex initialization.
Simplify refactoring.

##### Example

    void use(int arg)	// bad: uninitialized variable
    {
        int i;
        // ...
        i = 7;	// initialize i
    }

No, `i = 7` does not initialize `i`; it assigns to it. Also, `i` can be read in the `...` part. Better:

    void use(int arg)	// OK
    {
        int i = 7;		// OK: initialized
        string s;		// OK: default initialized
        // ...
    }

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

    widget i, j; // "widget" a type that's expensive to initialize, possibly a large POD

    if (cond) {     // bad: i and j are initialized "late"
        i = f1();
        j = f2();
    }
    else {
        i = f3();
        j = f4();
    }

This cannot trivially be rewritten to initialize `i` and `j` with initializers.
Note that for types with a default constructor, attempting to postpone initialization simply leads to a default initialization followed by an assignment.
A popular reason for such examples is "efficiency", but a compiler that can detect whether we made a used-before-set error can also eliminate any redundant double initialization.

At the cost of repeating `cond` we could write:

    widget i = (cond) ? f1() : f3();
    widget j = (cond) ? f2() : f4();

Assuming that there is a logical connection between `i` and `j`, that connection should probably be expressed in code:

    pair<widget,widget> make_related_widgets(bool x)
    {
        return (x) ? {f1(),f2()} : {f3(),f4() };
    }

    auto init = make_related_widgets(cond);
    widget i = init.first;
    widget j = init.second;

Obviously, what we really would like is a construct that initialized n variables from a `tuple`. For example:

    auto {i,j} = make_related_widgets(cond);    // Not C++14

Today, we might approximate that using `tie()`:

    widget i;       // bad: uninitialized variable
    widget j;
    tie(i,j) = make_related_widgets(cond);

This may be seen as an example of the *immediately initialize from input* exception below.

Creating optimal and equivalent code from all of these examples should be well within the capabilities of modern C++ compilers
(but don't make performance claims without measuring; a compiler may very well not generate optimal code for every example and
there may be language rules preventing some optimization that you would have liked in a particular case)..

##### Note

Complex initialization has been popular with clever programmers for decades.
It has also been a major source of errors and complexity.
Many such errors are introduced during maintenance years after the initial implementation.

##### Exception

It you are declaring an object that is just about to be initialized from input, initializing it would cause a double initialization.
However, beware that this may leave uninitialized data beyond the input - and that has been a fertile source of errors and security breaches:

    constexpr int max = 8*1024;
    int buf[max];					// OK, but suspicious: uninitialized
    f.read(buf, max);

The cost of initializing that array could be significant in some situations.
However, such examples do tend to leave uninitialized variables accessible, so they should be treated with suspicion.

    constexpr int max = 8*1024;
    int buf[max] = {0};				// better in some situations
    f.read(buf, max);

When feasible use a library function that is known not to overflow. For example:

    string s;	// s is default initialized to ""
    cin >> s;	// s expands to hold the string

Don't consider simple variables that are targets for input operations exceptions to this rule:

    int i;		// bad
    // ...
    cin >> i;

In the not uncommon case where the input target and the input operation get separated (as they should not) the possibility of used-before-set opens up.

    int i2 = 0;	// better
    // ...
    cin >> i;

A good optimizer should know about input operations and eliminate the redundant operation.

##### Example

Using an `uninitialized` value is a symptom of a problem and not a solution:

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

Now the compiler cannot even simply detect a used-before-set.

##### Note

Sometimes, a lambda can be used as an initializer to avoid an uninitialized variable:

    error_code ec;
    Value v = [&] {
        auto p = get_value();	// get_value() returns a pair<error_code, Value>
        ec = p.first;
        return p.second;
    }();

or maybe:

    Value v = [] {
        auto p = get_value();	// get_value() returns a pair<error_code, Value>
        if (p.first) throw Bad_value{p.first};
        return p.second;
    }();

**See also**: [ES.28](#Res-lambda-init)

##### Enforcement

* Flag every uninitialized variable.
  Don't flag variables of user-defined types with default constructors.
* Check that an uninitialized buffer is written into *immediately* after declaration.
  Passing a uninitialized variable as a non-`const` reference argument can be assumed to be a write into the variable.

### <a name="Res-introduce"></a> ES.21: Don't introduce a variable (or constant) before you need to use it

##### Reason

Readability. To limit the scope in which the variable can be used.

##### Example

    int x = 7;
    // ... no use of x here ...
    ++x;

##### Enforcement

Flag declaration that distant from their first use.

### <a name="Res-init"></a> ES.22: Don't declare a variable until you have a value to initialize it with

##### Reason

Readability. Limit the scope in which a variable can be used. Don't risk used-before-set. Initialization is often more efficient than assignment.

##### Example, bad

    string s;
    // ... no use of s here ...
    s = "what a waste";

##### Example, bad

    SomeLargeType var;	// ugly CaMeLcAsEvArIaBlE

    if (cond)	// some non-trivial condition
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

This would be fine if there was a default initialization for `SomeLargeType` that wasn't too expensive.
Otherwise, a programmer might very well wonder if every possible path through the maze of conditions has been covered.
If not, we have a "use before set" bug. This is a maintenance trap.

For initializers of moderate complexity, including for `const` variables, consider using a lambda to express the initializer; see [ES.28](#Res-lambda-init).

##### Enforcement

* Flag declarations with default initialization that are assigned to before they are first read.
* Flag any complicated computation after an uninitialized variable and before its use.

### <a name="Res-list"></a> ES.23: Prefer the `{}` initializer syntax

##### Reason

The rules for `{}` initialization are simpler, more general, less ambiguous, and safer than for other forms of initialization.

##### Example

       int x {f(99)};
       vector<int> v = {1, 2, 3, 4, 5, 6};

##### Exception

For containers, there is a tradition for using `{...}` for a list of elements and `(...)` for sizes:

    vector<int> v1(10);		// vector of 10 elements with the default value 0
    vector<int> v2 {10};	// vector of 1 element with the value 10

##### Note

`{}`-initializers do not allow narrowing conversions.

##### Example

    int x {7.9};	// error: narrowing
    int y = 7.9;	// OK: y becomes 7. Hope for a compiler warning

##### Note

`{}` initialization can be used for all initialization; other forms of initialization can't:

    auto p = new vector<int> {1, 2, 3, 4, 5};	// initialized vector
    D::D(int a, int b) :m{a, b} {	// member initializer (e.g., m might be a pair)
        // ...
    };
    X var {};				// initialize var to be empty
    struct S {
        int m {7};	// default initializer for a member
        // ...
    };

##### Note

Initialization of a variable declared `auto` with a single value `{v}` had surprising results until recently:

    auto x1 {7};        // x1 is an int with the value 7
    auto x2 = {7};      // x2 is an initializer_list<int> with an element 7

    auto x11 {7, 8};    // error: two initializers
    auto x22 = {7, 8};  // x2 is an initializer_list<int> with elements 7 and 8

##### Exception

Use `={...}` if you really want an `initializer_list<T>`

    auto fib10 = {0, 1, 2, 3, 5, 8, 13, 25, 38, 63};	// fib10 is a list

##### Example

    template<typename T>
    void f()
    {
        T x1(1);	// T initialized with 1
        T x0();		// bad: function declaration (often a mistake)

        T y1 {1};	// T initialized with 1
        T y0 {};	// default initialized T
        // ...
    }

**See also**: [Discussion](#???)

##### Enforcement

Tricky.

* Don't flag uses of `=` for simple initializers.
* Look for `=` after `auto` has been seen.

### <a name="Res-unique"></a> ES.24: Use a `unique_ptr<T>` to hold pointers in code that may throw

##### Reason

Using `std::unique_ptr` is the simplest way to avoid leaks. And it is free compared to alternatives

##### Example

    void use(bool leak)
    {
        auto p1 = make_unique<int>(7);	// OK
        int* p2 = new int{7};			// bad: might leak
        // ...
        if (leak) return;
        // ...
    }

If `leak == true` the object pointed to by `p2` is leaked and the object pointed to by `p1` is not.

##### Enforcement

Look for raw pointers that are targets of `new`, `malloc()`, or functions that may return such pointers.

### <a name="Res-const"></a> ES.25: 값을 바꾸고 싶지 않다면 `const`, `constexpr`로 객체를 선언하라.
>### <a name="Res-const"></a> ES.25: Declare an objects `const` or `constexpr` unless you want to modify its value later on

##### Reason

실수로 값을 바꾸는 걸 막을 수 있는 방법이다. 컴파일러에게 최적화를 위한 기회를 줄 수도 있다.
>That way you can't change the value by mistake. That way may offer the compiler optimization opportunities.

##### Example

    void f(int n)
    {
        const int bufmax = 2 * n + 2;  // good: we can't change bufmax by accident
        int xmax = n;                  // suspicious: is xmax intended to change?
        // ...
    }

##### Enforcement

변수가 실제로 값이 바뀌는지 안 바뀌는지 보고 바뀐다면 표시한다.
`const`가 아닌 객체가 값을 바꾸려 하는지 찾아 내는 건 불가능하다.
>Look to see if a variable is actually mutated, and flag it if not. Unfortunately, it may be impossible to detect when a non-`const` was not intended to vary.

### <a name="Res-recycle"></a> ES.26: 두개 이상의 다른 목적을 위해 한 변수를 사용하지 마라.
>### <a name="Res-recycle"></a> ES.26: Don't use a variable for two unrelated purposes

##### Reason

가독성.
>Readability.

##### Example, bad

    void use()
    {
        int i;
        for (i = 0; i < 20; ++i) { /* ... */ }
        for (i = 0; i < 200; ++i) { /* ... */ } // bad: i recycled
    }

##### Enforcement

재활용되는 변수가 있다면 표시한다.
>Flag recycled variables.

### <a name="Res-stack"></a> ES.27: 지역변수 배열은 `std::array`, `stack_array`를 사용하라.
>### <a name="Res-stack"></a> ES.27: Use `std::array` or `stack_array` for arrays on the stack

##### Reason

가독성이 높아지고, 모르게 포인터로 바뀌지 않는다.
언어가 지원하는 배열과 비표준적인 확장과 헷갈리지 않는다.
>They are readable and don't implicitly convert to pointers.
They are not confused with non-standard extensions of built-in arrays.

##### Example, bad

    const int n = 7;
    int m = 9;

    void f()
    {
        int a1[n];
        int a2[m];	// error: not ISO C++
        // ...
    }

##### Note

`a1` 변수선언은 C++에서는 적법하다. 그런 류의 코드가 많이 있다.
다만 배열크기가 전역값일 때는 에러가 나기 쉽고 버퍼 오버플로우, 배열을 포인터로 변환(?) 등의 유명한 에러가 나기 쉽다.
`a2` 변수선언은 C방식으로 C++에서는 쓰지 않으며 보안상 문제가 있는 것으로 간주한다.
>The definition of `a1` is legal C++ and has always been.
There is a lot of such code.
It is error-prone, though, especially when the bound is non-local.
Also, it is a "popular" source of errors (buffer overflow, pointers from array decay, etc.).
The definition of `a2` is C but not C++ and is considered a security risk

##### Example

    const int n = 7;
    int m = 9;

    void f()
    {
        array<int, n> a1;
        stack_array<int> a2(m);
        // ...
    }

##### Enforcement

* 상수 크기를 가지지 않는 배열이라면 표시한다. (C스타일의 가변길이배열)
* 지역상수 크기를 가지지 않는 배열이라면 표시한다.

>* Flag arrays with non-constant bounds (C-style VLAs)
>* Flag arrays with non-local constant bounds

### <a name="Res-lambda-init"></a> ES.28: 특히 `const` 변수같은 복잡한 초기화를 위해 람다를 사용하라.
>### <a name="Res-lambda-init"></a> ES.28: Use lambdas for complex initialization, especially of `const` variables

##### Reason

멋지게 지역 초기화를 숨길 수 있다.
초기화 작업을 위해서만 필요한 변수를 포함해서 재사용할 것 같지 않은 전역함수를 생성할 필요도 없다.
`const`여야만 하는 변수에도 작동하고 초기화 작업 후에도 잘 작동한다.
>It nicely encapsulates local initialization, including cleaning up scratch variables needed only for the initialization, without needing to create a needless nonlocal yet nonreusable function. It also works for variables that should be `const` but only after some initialization work.

##### Example, bad

    widget x;	// should be const, but:
    for (auto i = 2; i <= N; ++i) {             // this could be some
        x += some_obj.do_something_with(i);  // arbitrarily long code
    }                                        // needed to initialize x
    // from here, x should be const, but we can’t say so in code in this style

##### Example, good

    const widget x = [&]{
        widget val;                                // assume that widget has a default constructor
        for (auto i = 2; i <= N; ++i) {            // this could be some
            val += some_obj.do_something_with(i);  // arbitrarily long code
        }                                          // needed to initialize x
        return val;
    }();

##### Example

    string var = [&]{
        if (!in) return "";	// default
        string s;
        for (char c : in >> c)
            s += toupper(c);
        return s;
    }(); // note ()

가능하다면 `enum`같은 쉬운 방법으로 조건을 줄여라. 값선정과 초기화를 섞지 마라.
>If at all possible, reduce the conditions to a simple set of alternatives (e.g., an `enum`) and don't mix up selection and initialization.

##### Example

    owner<istream&> in = [&]{
        switch (source) {
        case default:       owned=false; return cin;
        case command_line:  owned=true;  return *new istringstream{argv[2]};
        case file:          owned=true;  return *new ifstream{argv[2]};
    }();

##### Enforcement

어렵다. 잘해야 휴리스틱. 루프문으로 값을 설정하는 초기화 안된 변수을 찾아라.
>Hard. At best a heuristic. Look for an uninitialized variable followed by a loop assigning to it.

### <a name="Res-macros"></a> ES.30: 프로그램 텍스트를 변경하기 위해서 매크로를 사용하지 마라.
>### <a name="Res-macros"></a> ES.30: Don't use macros for program text manipulation

##### Reason

매크로는 주요 버그 소스이다.
매크로는 일반적인 범위와 타입 규칙을 따르지 않는다.
매크로는 매개변수 넘기는 것에 대한 일반적인 규칙을 따르지 않는다.
매크로는 사람이 보는 것과 컴파일러가 보는 것이 다르다는 점을 보장한다. (??)
>Macros are a major source of bugs.
Macros don't obey the usual scope and type rules.
Macros ensure that the human reader see something different from whet the compiler sees.
Macros complicates tool building.

##### Example, bad

    #define Case break; case	/* BAD */

이 무해한 매크로는 소문자를 대문자로만 바꿨는데 나쁜 흐름제어 버그를 만든다.
>This innocuous-looking macro makes a single lower case `c` instead of a `C` into a bad flow-control bug.

##### Note

이 규칙은 `#ifdef`문에서 설정제어를 위해 매크로를 사용하는 것은 금하지 않는다.
>This rule does not ban the use of macros for "configuration control" use in `#ifdef`s, etc.

##### Enforcement

소스제어(`#ifdef`같은)에 사용하지 않는 매크로를 본다면 소리질러.
>Scream when you see a macro that isn't just use for source control (e.g., `#ifdef`)

### <a name="Res-macros2"></a> ES.31: 상수나 함수에 대해서는 매크로를 사용하지 마라.
>### <a name="Res-macros2"></a> ES.31: Don't use macros for constants or "functions"

##### Reason

매크로는 주요 버그 소스이다.
매크로는 일반적인 범위와 타입 규칙을 따르지 않는다.
매크로는 매개변수 넘기는 것에 대한 일반적인 규칙을 따르지 않는다.
매크로는 사람이 보는 것과 컴파일러가 보는 것이 다르다는 점을 보장한다. (??)
매크로는 툴 빌딩을 복잡하게 한다.
>Macros are a major source of bugs.
Macros don't obey the usual scope and type rules.
Macros don't obey the usual rules for argument passing.
Macros ensure that the human reader see something different from whet the compiler sees.
Macros complicates tool building.

##### Example, bad

    #define PI 3.14
    #define SQUARE(a, b) (a*b)

`SQUARE`에 잘 알려진 버그가 없다고 하더라도 더 잘 동작하는 대안이 있다.
예를 들면:
>Even if we hadn't left a well-know bug in `SQUARE` there are much better behaved alternatives; for example:

    constexpr double pi = 3.14;
    template<typename T> T square(T a, T b) { return a*b; }

##### Enforcement

소스제어(`#ifdef`같은)에 사용하지 않는 매크로를 본다면 소리질러.
>Scream when you see a macro that isn't just use for source control (e.g., `#ifdef`)

### <a name="Res-CAPS!"></a> ES.32: 매크로 이름에 대문자만 사용하라.
>### <a name="Res-CAPS!"></a> ES.32: Use `ALL_CAPS` for all macro names

##### Reason

관습. 가독성. 매크로 구별.
>Convention. Readability. Distinguishing macros.

##### Example

```
    #define forever for(;;)		/* very BAD */

    #define FOREVER for(;;)		/* Still evil, but at least visible to humans */
```

##### Enforcement

소문자로 된 매크로를 보면 소리질러.
>Scream when you see a lower case macro.

### <a name="Res-ellipses"></a> ES.40: C스타일 가변인자 함수를 정의하지 마라.
>### <a name="Res-ellipses"></a> ES.40: Don't define a (C-style) variadic function

##### Reason

타입이 안전하지 않기 때문에. 복잡한 형변환과 매크로가 잘 작동하는 코드를 요구한다.
>Not type safe. Requires messy cast-and-macro-laden code to get working right.

##### Example

    ??? <vararg>

**Alternative**: 오버로딩, 템플릿, 가변인자 템플릿.
>**Alternative**: Overloading. Templates. Variadic templates.

##### Note

`<vararg>`
>There are rare used of variadic functions in SFINAE code, but those don't actually run and don't need the `<vararg>` implementation mess.

##### Enforcement

C스타일의 가변인자 함수를 정의한다면 표시한다.
>Flag definitions of C-style variadic functions.

## ES.stmt: Statements

문장은 제어흐름을 통제한다.(연산식인 함수 호출, 예외 호출을 제외하고)
>Statements control the flow of control (except for function calls and exception throws, which are expressions).

### <a name="Res-switch-if"></a> ES.70: 선택할 수 있다면 `if`문보다 `switch`문을 선호하라.
>### <a name="Res-switch-if"></a> ES.70: Prefer a `switch`-statement to an `if`-statement when there is a choice

##### Reason

* 가독성.
* 효율성: 상수값에 대해서 `if`-`then`-`else`문의 연속보다 `switch`문이 더 잘 최적화될 수 있다.
* `switch` 문은 휴리스틱하게 일관성 체크를 할 수 있다. 예를 들어 `enum` 모든 값을 커버할 수 있는가? 없으면 `default`는 있는가?

>* Readability.
>* Efficiency: A `switch` compares against constants and is usually better optimized than a series of tests in an `if`-`then`-`else` chain.
>* a `switch` is enables some heuristic consistency checking. For example, has all values of an `enum` been covered? If not, is there a `default`?

##### Example

    void use(int n)
    {
        switch (n) {	// good
        case 0:	// ...
        case 7:	// ...
        }
    }

위 예제가 더 좋다:
>rather than:

    void use2(int n)
    {
        if (n == 0)   // bad: if-then-else chain comparing against a set of constants
            // ...
        else if (n == 7)
            // ...
    }

##### Enforcement

상수값에 대해서 체크하는 if-then-else 연속이라면 표시한다.
>Flag if-then-else chains that check against constants (only).

### <a name="Res-for-range"></a> ES.71: 선택할 수 있다면 범위형 `for`문을 선호하라.
>### <a name="Res-for-range"></a> ES.71: Prefer a range-`for`-statement to a `for`-statement when there is a choice

##### Reason

가독성. 에러 방지. 효율성.
>Readability. Error prevention. Efficiency.

##### Example

    for (int i = 0; i < v.size(); ++i)	// bad
            cout << v[i] << '\n';

    for (auto p = v.begin(); p != v.end(); ++p)	// bad
        cout << *p << '\n';

    for (auto& x : v) 	// OK
        cout << x << '\n';

    for (int i = 1; i < v.size(); ++i) // touches two elements: can't be a range-for
        cout << v[i] + v[-1] << '\n';

    for (int i = 1; i < v.size(); ++i) // possible side-effect: can't be a range-for
        cout << f(&v[i]) << '\n';

    for (int i = 1; i < v.size(); ++i) { // body messes with loop variable: can't be a range-for
        if (i % 2)
            ++i;	// skip even elements
        else
            cout << v[i] << '\n';
    }

프로그래머나 정적 분석기는 `f(&v[i])`에서 `v`에 대해서 값변경이 일어나지 않는다고 판단할지도 모른다.
루프를 최적화하기 위해서다.
>A human or a good static analyzer may determine that there really isn't a side effect on `v` in `f(&v[i])` so that the loop can be rewritten.

루프문 내에서 루프변수와 관련되서 엮인 부분이 있다면 제일 먼저 없애도록 한다.
>"Messing with the loop variable" in the body of a loop is typically best avoided.

##### Note

범위형 `for`문에 루프변수를 복사하여 사용하지 마라:
>Don't use expensive copies of the loop variable of a range-`for` loop:

    for (string s : vs) // ...

위는 `vs` 요소를 `s`로 복사한다. 개선하면
>This will copy each elements of `vs` into `s`. Better

    for (string& s : vs) // ...

##### Enforcement

루프를 보고 개별 요소들을 일렬로 참조하고 있고 부작용이 없어 보이면 `for`문으로 재작성하라.
>Look at loops, if a traditional loop just looks at each element of a sequence, and there are no side-effects on what it does with the elements, rewrite the loop to a for loop.

### <a name="Res-for-while"></a> ES.72:  루프 변수가 있다면 `while`문보다 `for`문을 선호하라.
>### <a name="Res-for-while"></a> ES.72: Prefer a `for`-statement to a `while`-statement when there is an obvious loop variable

##### Reason

가독성: 루프에 대한 전체 로직을 첫구문에서 볼 수 있다. 루프변수의 범위가 제한되는 점도 좋다.
>Readability: the complete logic of the loop is visible "up front". The scope of the loop variable can be limited.

##### Example

    for (int i = 0; i < vec.size(); i++) {
     // do work
    }

##### Example, bad

    int i = 0;
    while (i < vec.size()) {
     // do work
     i++;
    }

##### Enforcement

???

### <a name="Res-while-for"></a> ES.73: 루프 변수가 없다면 `for`문보다 `while`문을 선호하라.
>### <a name="Res-while-for"></a> ES.73: Prefer a `while`-statement to a `for`-statement when there is no obvious loop variable

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Res-for-init"></a> ES.74: `for`문 초기화 부분에 루프 변수를 선언하라.
>### <a name="Res-for-init"></a> ES.74: Prefer to declare a loop variable in the initializer part of as `for`-statement

##### Reason

루프 변수의 가시범위를 루프 범위 내로 제한하라.
루프문 뒤에서 다른 목적으로 루프 변수를 사용하지 못하게 하라.
>Limit the loop variable visibility to the scope of the loop.
Avoid using the loop variable for other purposes after the loop.

##### Example

    for (int i = 0; i < 100; ++i) {	// GOOD: i var is visible only inside the loop
        // ...
    }

##### Example, don't

    int j;						// BAD: j is visible outside the loop
    for (j = 0; j < 100; ++j) {
        // ...
    }
    // j is still visible here and isn't needed

**See also**: [Don't use a variable for two unrelated purposes](#Res-recycle)

##### Enforcement

`for`문 안에서만 변하는 변수가 루프 밖에 선언되어 있지만 루프 밖에서 사용되지 않고 있다면 경고하라.
(`for`문 안에서만 사용해야 하는 변수는 루프구문 안에 선언하라.)
>Warn when a variable modified inside the `for`-statement is declared outside the loop and not being used outside the loop.

**Discussion**: 루프변수를 루프구문내로 범위로 정하면 코드 최적화에 많은 도움이 된다.
유도 변수는 루프구문 안에서만 접근가능함을 파악하면 최적화시킬 수 있다.
위치이동시키기(hoisting), 강도줄이기(strength reduction), 루프내 불변코드 이동(loop-invariant code motion) 등이다.
>**Discussion**: Scoping the loop variable to the loop body also helps code optimizers greatly. Recognizing that the induction variable
is only accessible in the loop body unblocks optimizations such as hoisting, strength reduction, loop-invariant code motion, etc.

### <a name="Res-do"></a> ES.75: `do`문을 피하라.
>### <a name="Res-do"></a> ES.75: Avoid `do`-statements

##### Reason

가독성. 에러 회피.
종료 조건이 끝에 위치해 있고(못 보고 넘어가기 쉬운 위치.) 첫 루프에서 체크를 하지 않는다.
>Readability, avoidance of errors.
The termination conditions is at the end (where it can be overlooked) and the condition is not checked the first time through. ???

##### Example

    int x;
    do {
        cin >> x;
        x
    } while (x < 0);

##### Enforcement

???

### <a name="Res-goto"></a> ES.76: `goto`를 피하라.
>### <a name="Res-goto"></a> ES.76: Avoid `goto`

##### Reason

가독성. 에러 회피. 더 좋은 컨트롤 구조가 있다. `goto`는 생성코드에 좋은 구조이다.
>Readability, avoidance of errors. There are better control structures for humans; `goto` is for machine generated code.

##### Exception

중첩된 루프에서 빠져나오기. 그런 경우라면 항상 전방으로 점프한다.
>Breaking out of a nested loop. In that case, always jump forwards.

##### Example

    ???

##### Example

C에서 goto-exit를 상당히 많이 사용한다.:
>There is a fair amount of use of the C goto-exit idiom:

    void f()
    {
        // ...
            goto exit;
        // ...
            goto exit;
        // ...
    exit:
        ... common cleanup code ...
    }

이건 소멸자를 즉흥적으로 시뮬레이션한 것이다. 리소스를 해제할 수 있는 소멸자를 가진 핸들로 선언하라.
>This is an ad-hoc simulation of destructors. Declare your resources with handles with destructors that clean up.

##### Enforcement

* `goto`가 보이면 표시한다. 루프 다음문으로 점프하지 않는 중첩 루프 내의 `goto`는 모두 표시하면 더 좋다.

>* Flag `goto`. Better still flag all `goto`s that do not jump from a nested loop to the statement immediately after a nest of loops.

### <a name="Res-continue"></a> ES.77: ??? `continue`

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Res-break"></a> ES.78: 빈 `case`가 아니라면 `break`로 끝내라.
>### <a name="Res-break"></a> ES.78: Always end a non-empty `case` with a `break`

##### Reason

실수로 `break`없이 나가기는 꽤 많이 범하는 버그다.
고의적으로 `break`를 없애기는 유지보수의 위험요소이다.
 >Accidentally leaving out a `break` is a fairly common bug.
 A deliberate fallthrough is a maintenance hazard.

##### Example

    switch(eventType)
    {
    case Information:
        update_status_bar();
        break;
    case Warning:
        write_event_log();
    case Error:
        display_error_window(); // Bad
        break;
    }

`break`로 안 끝나는 사항은 간과하기 쉽다. 명확하게 해보자:
>It is easy to overlook the fallthrough. Be explicit:

    switch(eventType)
    {
    case Information:
        update_status_bar();
        break;
    case Warning:
        write_event_log();
        // fall through
    case Error:
        display_error_window(); // Bad
        break;
    }

`[[fallthrough]]`에 대한 제안도 있다.
>There is a proposal for a `[[fallthrough]]` annotation.

##### Note

단일문으로 된 여러개의 케이스 조건은 오케이:
>Multiple case labels of a single statement is OK:

    switch (x) {
    case 'a':
    case 'b':
    case 'f':
        do_something(x);
        break;
    }

##### Enforcement

빈 `case`문이 아닌데 break로 끝나지 않는다면 표시한다.
>Flag all fall throughs from non-empty `case`s.

### <a name="Res-default"></a> ES.79: ??? `default`

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Res-empty"></a> ES.85: 빈 문장을 보이게 만들어라.
>### <a name="Res-empty"></a> ES.85: Make empty statements visible

##### Reason

가독성.
>Readability.

##### Example

    for (i = 0; i < max; ++i);	// BAD: the empty statement is easily overlooked
        v[i] = f(v[i]);

    for (auto x : v) {		// better
        // nothing
    }

##### Enforcement

블록이 아니면서 주석문을 포함하지 않는 빈 문장이 있다면 표시한다.
>Flag empty statements that are not blocks and doesn't "contain" comments.

## ES.expr: Expressions

연산식은 값을 조작한다.
>Expressions manipulate values.

### <a name="Res-complicated"></a> ES.40: 복잡한 연산식은 피하라.
>### <a name="Res-complicated"></a> ES.40: Avoid complicated expressions

##### Reason

복잡한 연산식은 에러가 쉽게 발생하기 때문이다.
>Complicated expressions are error-prone.

##### Example

    while ((c = getc()) != -1)	// bad: assignment hidden in subexpression

    while ((cin >> c1, cin >> c2), c1 == c2) // bad: two non-local variables assigned in a sub-expressions

    for (char c1, c2; cin >> c1 >> c2 && c1 == c2;)	// better, but possibly still too complicated

    int x = ++i + ++j;	// OK: iff i and j are not aliased

    v[i] = v[j] + v[k];	// OK: iff i != j and i != k

    x = a + (b = f()) + (c = g()) * 7;	// bad: multiple assignments "hidden" in subexpressions

    x = a & b + c * d && e ^ f == 7;		// bad: relies on commonly misunderstood precedence rules

    x = x++ + x++ + ++x;		// bad: undefined behavior

위의 연산식 중 몇은 의심할 여지없이 나쁘다. (정의되지 않은 행동을 야기한다.)
나머지는 꽤 복잡하거나 특이한 편이고, 심지어 능력있는 프로그래머도 잘못 이해하거나 문제를 간과해 버릴 만한 것도 있다.
>Some of these expressions are unconditionally bad (e.g., they rely on undefined behavior). Others are simply so complicated and/or unusual that even good programmers could misunderstand them or overlook a problem when in a hurry.

##### Note

프로그래머는 연산식에 대해서 기본적인 규칙은 알고 사용했으면 한다.
>A programmer should know and use the basic rules for expressions.

##### Example

    x=k * y + z;              // OK

    auto t1 = k*y;        // bad: unnecessarily verbose
    x = t1 + z;

    if (0 <= x && x < max)     // OK

    auto t1 = 0 <= x;		    // bad: unnecessarily verbose
    auto t2 = x < max;
    if (t1 && t2)          // ...

##### Enforcement

트릭을 쓰자면(?). 연산식의 복잡성은 어떻게 고려할 것인가? 계산식을 하나의 연산으로만 구성된 문장들로 구성하기는 힘들다.
고려해볼 것:
>Tricky. How complicated must an expression be to be considered complicated? Writing computations as statements with one operation each is also confusing. Things to consider:

* 부작용: 다수의 비지역 변수에 대한 부작용을 의심할 수 있다. 특히 별도의 하위 연산식에 있다면.
* 가명(alias) 변수에 쓰기.
* N개 이상의 연산자.
* 비슷한 우선순위에 의존하기.
* 정의되지 않은 행동을 사용하기. (모든 정의되지 않은 행동을 찾아낼 수 있는가?)
* 정의된 실행 구현.
* ???

>* side effects: side effects on multiple non-local variables (for some definition of non-local) can be suspect, especially if the side effects are in separate subexpressions
>* writes to aliased variables
>* more than N operators (and what should N be?)
>* reliance of subtle precedence rules
>* uses undefined behavior (can we catch all undefined behavior?)
>* implementation defined behavior?
>* ???

### <a name="Res-parens"></a> ES.41: 연산자 우선순위에 의심이 든다면 괄호를 사용하라.
>### <a name="Res-parens"></a> ES.41: If in doubt about operator precedence, parenthesize

##### Reason

에러를 피하라. 가독성. 모든 사람들이 연산자 우선순위 테이블을 기억하지 않는다.
>Avoid errors. Readability. Not everyone has the operator table memorized.

##### Example

    const unsigned int flag = 2;
    unsigned int a = flag;

    if (a & flag != 0)  // bad: means a&(flag != 0)

Note: 프로그래머는 산술 연산, 논리 연산에 대해서 우선순위 테이블을 알고 있고, 다른 연산과 비트 연산을 섞어 사용할 때는 괄호를 고려해야 한다.
>Note: We recommend that programmers know their precedence table for the arithmetic operations, the logical operations, but consider mixing bitwise logical operations with other operators in need of parentheses.

    if ((a & flag) != 0)  // OK: works as intended

##### Note

아래에 대해서는 괄호가 필요없다는 정도는 알고 있을 것이다:
>You should know enough not to need parentheses for:

    if (a<0 || a<=max) {
        // ...
    }

##### Enforcement

* 비트 논리 연산자와 다른 연산자가 섞여 있다면 표시한다.
* 제일 왼쪽에 있는 연산자가 할당 연산자가 아니라면 표시한다.
* ???

>* Flag combinations of bitwise-logical operators and other operators.
>* Flag assignment operators not as the leftmost operator.
>* ???

### <a name="Res-ptr"></a> ES.42: 포인터를 단순하게 쉽게 사용하라.
>### <a name="Res-ptr"></a> ES.42: Keep use of pointers simple and straightforward

##### Reason

복잡한 포인터 계산은 주요한 에러 원인이 된다.
>Complicated pointer manipulation is a major source of errors.

* 모든 포인터 연산을 `array_view`로 해라.
* 포인터에 대한 포인터를 피하라.
>* Do all pointer arithmetic on an `array_view` (exception ++p in simple loop???)
>* Avoid pointers to pointers
>* ???

##### Example

    ???

##### Enforcement

포인터 연산문의 복잡성 한도 제한을 줄일 필요가 있다.
>We need a heuristic limiting the complexity of pointer arithmetic statement.

### <a name="Res-order"></a> ES.43: 계산 순서가 정의되지 않는 표현식은 피하라.
>### <a name="Res-order"></a> ES.43: Avoid expressions with undefined order of evaluation

##### Reason

그런 코드가 어떻게 동작할지는 알 수가 없다. 이식성.
당신에게는 맞을지는 몰라도, 다른 옵티마이저 세팅이나 다른 컴파일러라면 다르게 동작할지도 모른다.
>You have no idea what such code does. Portability.
Even if it does something sensible for you, it may do something different on another compiler (e.g., the next release of your compiler) or with a different optimizer setting.

##### Example

    v[i] = ++i;	//  the result is undefined

좋은 방법은 값을 쓰려고 할때 두번 이상 읽지 않는 것이다.
>A good rule of thumb is that you should not read a value twice in an expression where you write to it.

##### Example

    ???

##### Note

안전하게?
>What is safe?

##### Enforcement

좋은 분석기는 찾을 수 있다.
>Can be detected by a good analyzer.

### <a name="Res-order-fct"></a> ES.44: 함수인자의 계산순서에 의존하지 마라.
>### <a name="Res-order-fct"></a> ES.44: Don't depend on order of evaluation of function arguments

##### Reason

순서는 정의되지 않았기 때문에.
>Because that order is unspecified.

##### Example

    int i = 0;
    f(++i, ++i);

위의 호출은 `f(0, 1)`이거나 `f(1, 0)`이 될지도 모른다. 기술적으로 정의되지 않은 동작이다.
>The call will most likely be `f(0, 1)` or `f(1, 0)`, but you don't know which. Technically, the behavior is undefined.

##### Example

??? 오버로드된 연산자는 계산 순서 문제를 야기한다.
>??? overloaded operators can lead to order of evaluation problems (shouldn't :-()

    f1()->m(f2());	// m(f1(), f2())
    cout << f1() << f2();	// operator<<(operator<<(cout, f1()), f2())

##### Enforcement

좋은 분석기는 찾을 수 있다.
>Can be detected by a good analyzer.

### <a name="Res-magic"></a> ES.45: "매직 상수"를 피하라; 심볼 상수를 사용하라.
>### <a name="Res-magic"></a> ES.45: Avoid "magic constants"; use symbolic constants

##### Reason

표현식에 포함된 이름없는 상수는 쉽게 간과되고 있고 이해하기 어렵기 때문이다:
>Unnamed constants embedded in expressions are easily overlooked and often hard to understand:

##### Example

    for (int m = 1; m <= 12; ++m)	// don't: magic constant 12
        cout << month[m] << '\n';

1년에 12달이 숫자로만 되어 있다면 이해가 잘 안될 것이다. 더 좋게 고치면:
>No, we don't all know that there are 12 months, numbered 1..12, in a year. Better:

    constexpr int last_month = 12;	// months are numbered 1..12

    for (int m = first_month; m <= last_month; ++m)	// better
        cout << month[m] << '\n';

더 개선한다면, 상수를 없애면 된다:
>Better still, don't expose constants:

    for (auto m : month)
        cout << m << '\n';

##### Enforcement

코드에 문자가 있다면 표시한다. `0`, `1`, `nullptr`, `\n`, `""`, 다른 문자들에도 패스를 줘라. (?)
>Flag literals in code. Give a pass to `0`, `1`, `nullptr`, `\n`, `""`, and others on a positive list.

### <a name="Res-narrowing"></a> ES.46: 협소한, 값 자르기, 손실 있는 산술 변환을 피하라.
>### <a name="Res-narrowing"></a> ES.46: Avoid lossy (narrowing, truncating) arithmetic conversions

##### Reason

좁은 변환은 정보를 파괴하고 전혀 기대하지 않는 값을 가지게 한다.
>A narrowing conversion destroys information, often unexpectedly so.

##### Example

다음 예제는 기본적인 협소 변환이다:
>A key example is basic narrowing:

    double d = 7.9;
    int i = d;		// bad: narrowing: i becomes 7
    i = (int)d;     // bad: we're going to claim this is still not explicit enough

    void f(int x, long y, double d)
    {
        char c1 = x;	// bad: narrowing
        char c2 = y;	// bad: narrowing
        char c3 = d;	// bad: narrowing
    }

##### Note

gsl은  narrowing을 허용하고 값이 바뀔려고 하면 예외를 던지도록 정의한 `narrow` 연산을 제공한다.
>The guideline support library offers a `narrow` operation for specifying that narrowing is acceptable and a `narrow` ("narrow if") that throws an exception if a narrowing would throw away information:

    i = narrow_cast<int>(d);		// OK (you asked for it): narrowing: i becomes 7
    i = narrow<int>(d);				// OK: throws narrowing_error

손실있는 연산 형변환까지도 포함한다. 음의 실수 타입을 양의 정수타입으로 변환.
>We also include lossy arithmetic casts, such as from a negative floating point type to an unsigned integral type:

    double d = -7.9;
    unsigned u = 0;

    u = d;                          // BAD
    u = narrow_cast<unsigned>(d);   // OK (you asked for it): u becomes 0
    u = narrow<unsigned>(d);        // OK: throws narrowing_error

##### Enforcement

좋은 분석기는 모든 협의 변환을 찾아낼 수 있다. 그러나 모든 협의 변환을 표시하면 수많은 false positive를 야기하므로. 제안하면:
>A good analyzer can detect all narrowing conversions. However, flagging all narrowing conversions will lead to a lot of false positives. Suggestions:

*

>* flag all floating-point to integer conversions (maybe only float->char and double->int. Here be dragons! we need data)
>* flag all long->char (I suspect int->char is very common. Here be dragons! we need data)
>* consider narrowing conversions for function arguments especially suspect

### <a name="Res-nullptr"></a> ES.47: 0`, `NULL`보다 `nullptr`을 사용하라.
>### <a name="Res-nullptr"></a> ES.47: Use `nullptr` rather than `0` or `NULL`

##### Reason

가독성: `nullptr`가 `int`와 혼돈될 수 있으니 주의해라.
>Readability. Minimize surprises: `nullptr` cannot be confused with an `int`.

##### Example

다음을 보자:
>Consider:

    void f(int);
    void f(char*);
    f(0); 			// call f(int)
    f(nullptr); 	// call f(char*)

##### Enforcement

포인터에 `0`, `NULL`을 사용한다면 표시한다. 간단하게 프로그램으로 변환할 수 있으면 도움이 될거다.
>Flag uses of `0` and `NULL` for pointers. The transformation may be helped by simple program transformation.

### <a name="Res-casts"></a> ES.48: 형변환을 줄여라.
>### <a name="Res-casts"></a> ES.48: Avoid casts

##### Reason

형변환은 잘 알려진대로 에러의 원천이다. 최적화를 신뢰할 수 없게 만들어 버린다.
>Casts are a well-known source of errors. Makes some optimizations unreliable.

##### Example

    ???

##### Note

형변환을 쓰는 개발자는 자신들이 무엇을 하는지 잘 안다고 생각한다.
그러나 실제로는 값 사용에 대한 일반적인 규칙을 무력화시키고 있다.
오버로드 함수 선정과 템플릿 인스턴스화는 맞아떨어지는 함수가 있어야만 하지만, 없다면 형변환을 적용하게 된다.
>Programmer who write casts typically assumes that they know what they are doing.
In fact, they often disable the general rules for using values.
Overload resolution and template instantiation usually pick the right function if there is a right function to pick.
If there is not, maybe there ought to be, rather than applying a local fix (cast).

##### Note

형변환은 시스템용 프로그래밍 언어에 꼭 필요하다.
예를 들어, 디바이스 레지스터의 주소를 포인터로 얻어 올 때이다.
그러나 너무 남용하는 바람에 많은 에러가 발생하는 것도 사실이다.
>Casts are necessary in a systems programming language.
For example, how else would we get the address of a device register into a pointer.
However, casts are seriously overused as well as a major source of errors.

##### Note

형변환을 너무 많이 쓴다고 생각된다면 디자인적으로 문제가 있을지도 모른다.
>If you feel the need for a lot of casts, there may be a fundamental design problem.

##### Enforcement

* C스타일 형변환을 없애라.
* 네임드 형변환 이외에는 경고하라.
* 함수형 형변환이 많다면 경고하라.(많다는 게 문제의 소지다.)

>* Force the elimination of C-style casts
>* Warn against named casts
>* Warn if there are many functional style casts (there is an obvious problem in quantifying 'many').

### <a name="Res-casts-named"></a> ES.49: 형변환을 써야 한다면 네임드 형변환을 사용하라.
>### <a name="Res-casts-named"></a> ES.49: If you must use a cast, use a named cast

##### Reason

가독성, 에러 줄이기.
네임드 형변환은 C스타일이나 함수형 형변환보다 더 구체적이다. 컴파일러에게 에러를 알려주는 역할도 한다.
C스타일 형변환: (int) a
함수형 형변환: int(a)
>Readability. Error avoidance.
Named casts are more specific than a C-style or functional cast, allowing the compiler to catch some errors.

네임드 형변환:
>The named casts are:

* `static_cast`
* `const_cast`
* `reinterpret_cast`
* `dynamic_cast`
* `std::move`		// `move(x)` is an rvalue reference to `x`
* `std::forward`	// `forward(x)` is an rvalue reference to `x`
* `gsl::narrow_cast`	// `narrow_cast<T>(x)` is `static_cast<T>(x)`
* `gsl::narrow`			// `narrow<T>(x)` is `static_cast<T>(x)` if `static_cast<T>(x) == x` or it throws `narrowing_error`

##### Example

    ???

##### Note

???

##### Enforcement

C스타일, 함수형 형변환이 있다면 표시한다.
>Flag C-style and functional casts.

### <a name="Res-casts-const"></a> ES.50: `const`를 없애지 마라.
>## <a name="Res-casts-const"></a> ES.50: Don't cast away `const`

##### Reason

`const`가 아닌 것처럼 속이지 않기 위해.
>It makes a lie out of `const`.

##### Note

보통 `const`를 없애버리는 이유는 변경할 수 없는 객체 속에 있는 일시적인 정보를 변경하기 위해서이다.
예를 들면 캐싱값, 임시계산값, 선계산값 등이다.
이런 값은 `const_cast`를 쓰는 것보다 `mutable`이나 간접적인 방법을 사용하면 더 쉽게 처리할 수 있다.
>Usually the reason to "cast away `const`" is to allow the updating of some transient information of an otherwise immutable object.
Examples are cashing, memorization, and precomputation.
Such examples are often handled as well or better using `mutable` or an indirection than with a `const_cast`.

##### Example

    ???

##### Enforcement

`const_cast`이 있다면 표시한다.
>Flag `const_cast`s.

### <a name="Res-range-checking"></a> ES.55: 범위를 체크할 필요성을 없애라.
>### <a name="Res-range-checking"></a> ES.55: Avoid the need for range checking

##### Reason

범위를 벗어날 수 없는 구조라면 오히려 더 빠르게 실행될 수 있다.
>Constructs that cannot overflow, don't, and usually runs faster:

##### Example

    for (auto& x : v)		// print all elements of v
        cout << x << '\n';

    auto p = find(v, x);		// find x in v

##### Enforcement

명시적인 범위체크를 찾아라. 적절한 대안을 제안한다. (?)
>Look for explicit range checks and heuristically suggest alternatives.

### <a name="Res-new"></a> ES.60: 리소스 함수 외부에서는 `new`, `delete[]`를 쓰지 마라.
>### <a name="Res-new"></a> ES.60: Avoid `new` and `delete[]` outside resource management functions

##### Reason

프로그램 코드 내에서 직접적인 리소스 관리는 에러를 발생시키기 쉬우며 지루(?)하다.
>Direct resource management in application code is error-prone and tedious.

##### Note

"no naked `new`""로 알려짐. (C스타일 포인터 `T *`, C++은 std::shared_ptr<T>, std::weak_ptr<T>, std::unique_ptr<T>)
C스타일 포인터를 생짜 포인터(naked pointer 또는 raw pointer)라고 함.
>also known as "No naked `new`!"

##### Example, bad

    void f(int n)
    {
        auto p = new X[n];	// n default constructed Xs
        // ...
        delete[] p;
    }

`...`는 `delete`를 호출할 필요가 전혀 없는 코드라고 가정한다.
>There can be code in the `...` part that causes the `delete` never to happen.

**See also**: [R: Resource management](#S-resource).

##### Enforcement

생짜 `new`, `delete`이 있다면 표시한다.
>Flag naked `new`s and naked `delete`s.

### <a name="Res-del"></a> ES.61: `delete[]`로 배열 포인터를 해제하라. `delete`로 배열이 아닌 포인터를 해제하라.
>### <a name="Res-del"></a> ES.61: delete arrays using `delete[]` and non-arrays using `delete`

##### Reason

C++의 요구조건이고 잘못 사용하면 리소스 해제 에러가 나면서 메모리값이 엉망이 될 것이다.
>That's what the language requires and mistakes can lead to resource release errors and/or memory corruption.

##### Example, bad

    void f(int n)
    {
        auto p = new X[n];	// n default constructed Xs
        // ...
        delete p;			// error: just delete the object p, rather than delete the array p[]
    }

##### Note

이 예제는 [no naked `new` rule](#Res-new)를 위반할 뿐만 아니라 많은 다른 문제를 야기한다.
>This example not only violates the [no naked `new` rule](#Res-new) as in the previous example, it has many more problems.

##### Enforcement

* `new`, `delete`가 같은 영역범위에 있다면 오류여부를 표시해 줄 수 있다.
* `new`, `delete`가 생성자/소멸자 안에 있다면 오류여부를 표시해 줄 수 있다.

>* if the `new` and the `delete` is in the same scope, mistakes can be flagged.
* if the `new` and the `delete` are in a constructor/destructor pair, mistakes can be flagged.

### <a name="Res-arr2"></a> ES.62: 다른 배열간에 포인터를 비교하지 마라.
>### <a name="Res-arr2"></a> ES.62: Don't compare pointers into different arrays

##### Reason

결과는 예측불가하다.
>The result of doing so is undefined.

##### Example, bad

    void f(int n)
    {
        int a1[7];
        int a2[9];
        if (&a1[5] < &a2[7]) {}       // bad: undefined
        if (0 < &a1[5] - &a2[7]) {}   // bad: undefined
    }

##### Note

더 많은 문제가 내포되어 있다.
>This example has many more problems.

##### Enforcement

## <a name="SS-numbers"></a> 연산
>## <a name="SS-numbers"></a> Arithmetic

### <a name="Res-mix"></a> ES.100: 부호 있는 연산과 없는 연산을 섞지 마라.
>### <a name="Res-mix"></a> ES.100: Don't mix signed and unsigned arithmetic

##### Reason

결과가 잘못될 수 있기 때문에.
>Avoid wrong results.

##### Example

    unsigned x = 100;
    unsigned y = 102;
    cout << abs(x-y) << '\n'; // wrong result

##### Note

불행히도 C++은 배열인자에 대해서 부호있는 정수를 사용하고 표준 라이브러리는 컨테이너 인자에 부호없는 정수형을 사용한다.
일관성을 방해한다.
>Unfortunately, C++ uses signed integers for array subscripts and the standard library uses unsigned integers for container subscripts.
This precludes consistency.

##### Enforcement

컴파일러가 이미 알고 있는 상황이고 경고를 날려 줄 것이다.
>Compilers already know and sometimes warn.

### <a name="Res-unsigned"></a> ES.101: 비트연산 시에는 부호없는 타입을 사용하라.
>### <a name="Res-unsigned"></a> ES.101: use unsigned types for bit manipulation

##### Reason

부호없는 타입은 부호비트까지 포함해서 비트 연산할 수 있도록 지원하기 때문에.
>Unsigned types support bit manipulation without surprises from sign bits.

##### Example

    ???

**Exception**: 모듈러 연산을 하려면 부호없는 타입을 사용하라.
>**Exception**: Use unsigned types if you really want modulo arithmetic.

##### Enforcement

???

### <a name="Res-signed"></a> ES.102: 산술연산을 위해서는 부호있는 타입을 사용하라.
>### <a name="Res-signed"></a> ES.102: Used signed types for arithmetic

##### Reason

부호없는 타입은 부호비트까지 포함해서 비트 연산할 수 있도록 지원하기 때문에.
>Unsigned types support bit manipulation without surprises from sign bits.

##### Example

    ???

**Exception**: 모듈러 연산을 사용한다면 부호없는 타입을 사용하라.
>**Exception**: Use unsigned types if you really want modulo arithmetic.

##### Enforcement

???

### <a name="Res-overflow"></a> ES.103: 오버플로우를 내지마라.
>### <a name="Res-overflow"></a> ES.103: Don't overflow

##### Reason

오버플로우는 수식 알고리즘을 의미없게 만들어 버린다.
최대값 이상으로 증가시킨다면 메모리값이 망가지고 비정상적으로 작동한다.
>Overflow usually makes your numeric algorithm meaningless.
Incrementing a value beyond a maximum value can lead to memory corruption and undefined behavior.

##### Example, bad

    int a[10];
    a[10] = 7;		// bad

    int n = 0;
    while (n++ < 10)
        a[n - 1] = 9; // bad (twice)

##### Example, bad

    int n = numeric_limits<int>::max();
    int m = n + 1;	// bad

##### Example, bad

    int area(int h, int w) { return h * w; }

    auto a = area(10'000'000, 100'000'000);	// bad

**Exception**: 모듈러 연산을 사용한다면 부호없는 타입을 사용하라.
>**Exception**: Use unsigned types if you really want modulo arithmetic.

**Alternative**: 어느 정도의 오버헤드를 감수할 수 있는 대단히 중요한 프로그램에서는 정수 범위 체크나 부동소수점 타입을 사용하라.
>**Alternative**: For critical applications that can afford some overhead, use a range-checked integer and/or floating-point type.

##### Enforcement

???

### <a name="Res-underflow"></a> ES.104: 언더플로우를 내지 마라.
>### <a name="Res-underflow"></a> ES.104: Don't underflow

##### Reason

최소값 이하로 값이 내려가면 메모리값이 망가지고 비정상적으로 작동한다.
>Decrementing a value beyond a minimum value can lead to memory corruption and undefined behavior.

##### Example, bad

    int a[10];
    a[-2] = 7;		// bad

    int n = 101;
    while (n--)
        a[n - 1] = 9; // bad (twice)

**Exception**: 모듈러 연산을 사용한다면 부호없는 타입을 사용하라.
>**Exception**: Use unsigned types if you really want modulo arithmetic.

##### Enforcement

???

### <a name="Res-zero"></a> ES.105: 0으로 나누지 마라.
>### <a name="Res-zero"></a> ES.105: Don't divide by zero

##### Reason

결과는 예측할 수 없고 코어덤프가 날 것이다.
>The result is undefined and probably a crash.

##### Note

`%` 모듈러 연산도 같이 적용된다.
>this also applies to `%`.

##### Example

    ???

**Alternative**: 어느 정도의 오버헤드를 감수할 수 있는 대단히 중요한 프로그램에서는 정수 범위 체크나 부동소수점 타입을 사용하라.
>**Alternative**: For critical applications that can afford some overhead, use a range-checked integer and/or floating-point type.

##### Enforcement

???
