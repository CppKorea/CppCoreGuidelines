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
* [ES.6: for-구문 초기화나 조건 설정에 사용한 이름은 스코프 안으로 한정하자](#Res-cond)
* [ES.7: 자주 쓰거나 지역변수는 이름을 짧게, 그렇지 않은 경우는 길게](#Res-name-length)
* [ES.8: 비슷해보이는 네이밍은 피하자](#Res-name-similar)
* [ES.9: `ALL_CAPS` 네이밍은 피하자](#Res-!CAPS)
* [ES.10: 한 선언당 (단) 하나의 이름만 선언하자 ](#Res-name-one)
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
* [ES.101: 비트 조작을 할때는 unsigned호타입을 사용하자](#Res-unsigned)
* [ES.102: 연산할때는 signed 타입을 사용하자](#Res-signed)
* [ES.103: 오버플로우 금지](#Res-overflow)
* [ES.104: 언더플로우 금지](#Res-underflow)
* [ES.105: 0으로 나누기 금지](#Res-zero)

* [ES.100: Don't mix signed and unsigned arithmetic](#Res-mix)
* [ES.101: use unsigned types for bit manipulation](#Res-unsigned)
* [ES.102: Used signed types for arithmetic](#Res-signed)
* [ES.103: Don't overflow](#Res-overflow)
* [ES.104: Don't underflow](#Res-underflow)
* [ES.105: Don't divide by zero](#Res-zero)

### <a name="Res-lib"></a> ES.1: Prefer the standard library to other libraries and to "handcrafted code"

##### Reason

Code using a library can be much easier to write than code working directly with language features, much shorter, tend to be of a higher level of abstraction, and the library code is presumably already tested.
The ISO C++ standard library is among the most widely know and best tested libraries.
It is available as part of all C++ Implementations.

##### Example

    auto sum = accumulate(begin(a), end(a), 0.0);	// good

a range version of `accumulate` would be even better:

    auto sum = accumulate(v, 0.0); // better

but don't hand-code a well-known algorithm:

    int max = v.size();		// bad: verbose, purpose unstated
    double sum = 0.0;
    for (int i = 0; i < max; ++i)
        sum = sum + v[i];

**Exception**: Large parts of the standard library rely on dynamic allocation (free store). These parts, notably the containers but not the algorithms, are unsuitable for some hard-real time and embedded applications. In such cases, consider providing/using similar facilities, e.g.,  a standard-library-style container implemented using a pool allocator.

##### Enforcement

Not easy. ??? Look for messy loops, nested loops, long functions, absence of function calls, lack of use of non-built-in types. Cyclomatic complexity?

### <a name="Res-abstr"></a> ES.2: Prefer suitable abstractions to direct use of language features

##### Reason

A "suitable abstraction" (e.g., library or class) is closer to the application concepts than the bare language, leads to shorter and clearer code, and is likely to be better tested.

##### Example

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

##### Enforcement

Not easy. ??? Look for messy loops, nested loops, long functions, absence of function calls, lack of use of non-built-in types. Cyclomatic complexity?

## ES.dcl: Declarations

A declaration is a statement. a declaration introduces a name into a scope and may cause the construction of a named object.

### <a name="Res-scope"></a> ES.5: Keep scopes small

##### Reason

Readability. Minimize resource retention. Avoid accidental misuse of value.

**Alternative formulation**: Don't declare a name in an unnecessarily large scope.

##### Example

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

##### Example, bad

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

##### Enforcement

* Flag loop variable declared outside a loop and not used after the loop
* Flag when expensive resources, such as file handles and locks are not used for N-lines (for some suitable N)

### <a name="Res-cond"></a> ES.6: Declare names in for-statement initializers and conditions to limit scope

##### Reason

Readability. Minimize resource retention.

##### Example

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

##### Enforcement

* Flag loop variables declared before the loop and not used after the loop
* (hard) Flag loop variables declared before the loop and used after the loop for an unrelated purpose.

### <a name="Res-name-length"></a> ES.7: Keep common and local names short, and keep uncommon and nonlocal names longer

##### Reason

Readability. Lowering the chance of clashes between unrelated non-local names.

##### Example

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

##### Example

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

##### Example, bad

Argument names of large functions are de facto non-local and should be meaningful:

    void complicated_algorithm(vector<Record>&vr, const vector<int>& vi, map<string, int>& out)
    // read from events in vr (marking used Records) for the indices in vi placing (name, index) pairs into out
    {
        // ... 500 lines of code using vr, vi, and out ...
    }

We recommend keeping functions short, but that rule isn't universally adhered to and naming should reflect that.

##### Enforcement

Check length of local and non-local names. Also take function length into account.

### <a name="Res-name-similar"></a> ES.8: Avoid similar-looking names

##### Reason

Such names slow down comprehension and increase the likelihood of error.

##### Example

    if (readable(i1 + l1 + ol + o1 + o0 + ol + o1 + I0 + l0)) surprise();

##### Enforcement

Check names against a list of known confusing letter and digit combinations.

### <a name="Res-!CAPS"></a> ES.9: Avoid `ALL_CAPS` names

##### Reason

Such names are commonly used for macros. Thus, `ALL_CAPS` name are vulnerable to unintended macro substitution.

##### Example

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

##### Note

Do not use `ALL_CAPS` for constants just because constants used to be macros.

##### Enforcement

Flag all uses of ALL CAPS. For older code, accept ALL CAPS for macro names and flag all non-ALL-CAPS macro names.

### <a name="Res-name-one"></a> ES.10: Declare one name (only) per declaration

##### Reason

One-declaration-per line increases readability and avoid mistake related to the C/C++ grammar. It leaves room for a `//`-comment.

##### Example, bad

       char *p, c, a[7], *pp[7], **aa[10];	// yuck!

**Exception**: a function declaration can contain several function argument declarations.

##### Example

    template <class InputIterator, class Predicate>
    bool any_of(InputIterator first, InputIterator last, Predicate pred);

or better using concepts:

    bool any_of(InputIterator first, InputIterator last, Predicate pred);

##### Example

    double scalbn(double x, int n);	 	// OK: x*pow(FLT_RADIX, n); FLT_RADIX is usually 2

or:

    double scalbn(    // better: x*pow(FLT_RADIX, n); FLT_RADIX is usually 2
        double x,     // base value
        int n         // exponent
    );

or:

    double scalbn(double base, int exponent);	// better: base*pow(FLT_RADIX, exponent); FLT_RADIX is usually 2

##### Enforcement

Flag non-function arguments with multiple declarators involving declarator operators (e.g., `int* p, q;`)

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

### <a name="Res-const"></a> ES.25: Declare an objects `const` or `constexpr` unless you want to modify its value later on

##### Reason

That way you can't change the value by mistake. That way may offer the compiler optimization opportunities.

##### Example

    void f(int n)
    {
        const int bufmax = 2 * n + 2;  // good: we can't change bufmax by accident
        int xmax = n;                  // suspicious: is xmax intended to change?
        // ...
    }

##### Enforcement

Look to see if a variable is actually mutated, and flag it if not. Unfortunately, it may be impossible to detect when a non-`const` was not intended to vary.

### <a name="Res-recycle"></a> ES.26: Don't use a variable for two unrelated purposes

##### Reason

Readability.

##### Example, bad

    void use()
    {
        int i;
        for (i = 0; i < 20; ++i) { /* ... */ }
        for (i = 0; i < 200; ++i) { /* ... */ } // bad: i recycled
    }

##### Enforcement

Flag recycled variables.

### <a name="Res-stack"></a> ES.27: Use `std::array` or `stack_array` for arrays on the stack

##### Reason

They are readable and don't implicitly convert to pointers.
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

The definition of `a1` is legal C++ and has always been.
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

* Flag arrays with non-constant bounds (C-style VLAs)
* Flag arrays with non-local constant bounds

### <a name="Res-lambda-init"></a> ES.28: Use lambdas for complex initialization, especially of `const` variables

##### Reason

It nicely encapsulates local initialization, including cleaning up scratch variables needed only for the initialization, without needing to create a needless nonlocal yet nonreusable function. It also works for variables that should be `const` but only after some initialization work.

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

If at all possible, reduce the conditions to a simple set of alternatives (e.g., an `enum`) and don't mix up selection and initialization.

##### Example

    owner<istream&> in = [&]{
        switch (source) {
        case default:       owned=false; return cin;
        case command_line:  owned=true;  return *new istringstream{argv[2]};
        case file:          owned=true;  return *new ifstream{argv[2]};
    }();

##### Enforcement

Hard. At best a heuristic. Look for an uninitialized variable followed by a loop assigning to it.

### <a name="Res-macros"></a> ES.30: Don't use macros for program text manipulation

##### Reason

Macros are a major source of bugs.
Macros don't obey the usual scope and type rules.
Macros ensure that the human reader see something different from whet the compiler sees.
Macros complicates tool building.

##### Example, bad

    #define Case break; case	/* BAD */

This innocuous-looking macro makes a single lower case `c` instead of a `C` into a bad flow-control bug.

##### Note

This rule does not ban the use of macros for "configuration control" use in `#ifdef`s, etc.

##### Enforcement

Scream when you see a macro that isn't just use for source control (e.g., `#ifdef`)

### <a name="Res-macros2"></a> ES.31: Don't use macros for constants or "functions"

##### Reason

Macros are a major source of bugs.
Macros don't obey the usual scope and type rules.
Macros don't obey the usual rules for argument passing.
Macros ensure that the human reader see something different from whet the compiler sees.
Macros complicates tool building.

##### Example, bad

    #define PI 3.14
    #define SQUARE(a, b) (a*b)

Even if we hadn't left a well-know bug in `SQUARE` there are much better behaved alternatives; for example:

    constexpr double pi = 3.14;
    template<typename T> T square(T a, T b) { return a*b; }

##### Enforcement

Scream when you see a macro that isn't just use for source control (e.g., `#ifdef`)

### <a name="Res-CAPS!"></a> ES.32: Use `ALL_CAPS` for all macro names

##### Reason

Convention. Readability. Distinguishing macros.

##### Example

    #define forever for(;;)		/* very BAD */

    #define FOREVER for(;;)		/* Still evil, but at least visible to humans */

##### Enforcement

Scream when you see a lower case macro.

### <a name="Res-ellipses"></a> ES.40: Don't define a (C-style) variadic function

##### Reason

Not type safe. Requires messy cast-and-macro-laden code to get working right.

##### Example

    ??? <vararg>

**Alternative**: Overloading. Templates. Variadic templates.

##### Note

There are rare used of variadic functions in SFINAE code, but those don't actually run and don't need the `<vararg>` implementation mess.

##### Enforcement

Flag definitions of C-style variadic functions.

## ES.stmt: Statements

Statements control the flow of control (except for function calls and exception throws, which are expressions).

### <a name="Res-switch-if"></a> ES.70: Prefer a `switch`-statement to an `if`-statement when there is a choice

##### Reason

* Readability.
* Efficiency: A `switch` compares against constants and is usually better optimized than a series of tests in an `if`-`then`-`else` chain.
* a `switch` is enables some heuristic consistency checking. For example, has all values of an `enum` been covered? If not, is there a `default`?

##### Example

    void use(int n)
    {
        switch (n) {	// good
        case 0:	// ...
        case 7:	// ...
        }
    }

rather than:

    void use2(int n)
    {
        if (n == 0)   // bad: if-then-else chain comparing against a set of constants
            // ...
        else if (n == 7)
            // ...
    }

##### Enforcement

Flag if-then-else chains that check against constants (only).

### <a name="Res-for-range"></a> ES.71: Prefer a range-`for`-statement to a `for`-statement when there is a choice

##### Reason

Readability. Error prevention. Efficiency.

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

A human or a good static analyzer may determine that there really isn't a side effect on `v` in `f(&v[i])` so that the loop can be rewritten.

"Messing with the loop variable" in the body of a loop is typically best avoided.

##### Note

Don't use expensive copies of the loop variable of a range-`for` loop:

    for (string s : vs) // ...

This will copy each elements of `vs` into `s`. Better

    for (string& s : vs) // ...

##### Enforcement

Look at loops, if a traditional loop just looks at each element of a sequence, and there are no side-effects on what it does with the elements, rewrite the loop to a for loop.

### <a name="Res-for-while"></a> ES.72: Prefer a `for`-statement to a `while`-statement when there is an obvious loop variable

##### Reason

Readability: the complete logic of the loop is visible "up front". The scope of the loop variable can be limited.

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

### <a name="Res-while-for"></a> ES.73: Prefer a `while`-statement to a `for`-statement when there is no obvious loop variable

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Res-for-init"></a> ES.74: Prefer to declare a loop variable in the initializer part of as `for`-statement

##### Reason

Limit the loop variable visibility to the scope of the loop.
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

Warn when a variable modified inside the `for`-statement is declared outside the loop and not being used outside the loop.

**Discussion**: Scoping the loop variable to the loop body also helps code optimizers greatly. Recognizing that the induction variable
is only accessible in the loop body unblocks optimizations such as hoisting, strength reduction, loop-invariant code motion, etc.

### <a name="Res-do"></a> ES.75: Avoid `do`-statements

##### Reason

Readability, avoidance of errors.
The termination conditions is at the end (where it can be overlooked) and the condition is not checked the first time through. ???

##### Example

    int x;
    do {
        cin >> x;
        x
    } while (x < 0);

##### Enforcement

???

### <a name="Res-goto"></a> ES.76: Avoid `goto`

##### Reason

Readability, avoidance of errors. There are better control structures for humans; `goto` is for machine generated code.

##### Exception

Breaking out of a nested loop. In that case, always jump forwards.

##### Example

    ???

##### Example

There is a fair amount of use of the C goto-exit idiom:

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

This is an ad-hoc simulation of destructors. Declare your resources with handles with destructors that clean up.

##### Enforcement

* Flag `goto`. Better still flag all `goto`s that do not jump from a nested loop to the statement immediately after a nest of loops.

### <a name="Res-continue"></a> ES.77: ??? `continue`

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Res-break"></a> ES.78: Always end a non-empty `case` with a `break`

##### Reason

 Accidentally leaving out a `break` is a fairly common bug.
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

It is easy to overlook the fallthrough. Be explicit:

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

There is a proposal for a `[[fallthrough]]` annotation.

##### Note

Multiple case labels of a single statement is OK:

    switch (x) {
    case 'a':
    case 'b':
    case 'f':
        do_something(x);
        break;
    }

##### Enforcement

Flag all fall throughs from non-empty `case`s.

### <a name="Res-default"></a> ES.79: ??? `default`

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Res-empty"></a> ES.85: Make empty statements visible

##### Reason

Readability.

##### Example

    for (i = 0; i < max; ++i);	// BAD: the empty statement is easily overlooked
        v[i] = f(v[i]);

    for (auto x : v) {		// better
        // nothing
    }

##### Enforcement

Flag empty statements that are not blocks and doesn't "contain" comments.

## ES.expr: Expressions

Expressions manipulate values.

### <a name="Res-complicated"></a> ES.40: Avoid complicated expressions

##### Reason

Complicated expressions are error-prone.

##### Example

    while ((c = getc()) != -1)	// bad: assignment hidden in subexpression

    while ((cin >> c1, cin >> c2), c1 == c2) // bad: two non-local variables assigned in a sub-expressions

    for (char c1, c2; cin >> c1 >> c2 && c1 == c2;)	// better, but possibly still too complicated

    int x = ++i + ++j;	// OK: iff i and j are not aliased

    v[i] = v[j] + v[k];	// OK: iff i != j and i != k

    x = a + (b = f()) + (c = g()) * 7;	// bad: multiple assignments "hidden" in subexpressions

    x = a & b + c * d && e ^ f == 7;		// bad: relies on commonly misunderstood precedence rules

    x = x++ + x++ + ++x;		// bad: undefined behavior

Some of these expressions are unconditionally bad (e.g., they rely on undefined behavior). Others are simply so complicated and/or unusual that even good programmers could misunderstand them or overlook a problem when in a hurry.

##### Note

A programmer should know and use the basic rules for expressions.

##### Example

    x=k * y + z;              // OK

    auto t1 = k*y;        // bad: unnecessarily verbose
    x = t1 + z;

    if (0 <= x && x < max)     // OK

    auto t1 = 0 <= x;		    // bad: unnecessarily verbose
    auto t2 = x < max;
    if (t1 && t2)          // ...

##### Enforcement

Tricky. How complicated must an expression be to be considered complicated? Writing computations as statements with one operation each is also confusing. Things to consider:

* side effects: side effects on multiple non-local variables (for some definition of non-local) can be suspect, especially if the side effects are in separate subexpressions
* writes to aliased variables
* more than N operators (and what should N be?)
* reliance of subtle precedence rules
* uses undefined behavior (can we catch all undefined behavior?)
* implementation defined behavior?
* ???

### <a name="Res-parens"></a> ES.41: If in doubt about operator precedence, parenthesize

##### Reason

Avoid errors. Readability. Not everyone has the operator table memorized.

##### Example

    const unsigned int flag = 2;
    unsigned int a = flag;

    if (a & flag != 0)  // bad: means a&(flag != 0)

Note: We recommend that programmers know their precedence table for the arithmetic operations, the logical operations, but consider mixing bitwise logical operations with other operators in need of parentheses.

    if ((a & flag) != 0)  // OK: works as intended

##### Note

You should know enough not to need parentheses for:

    if (a<0 || a<=max) {
        // ...
    }

##### Enforcement

* Flag combinations of bitwise-logical operators and other operators.
* Flag assignment operators not as the leftmost operator.
* ???

### <a name="Res-ptr"></a> ES.42: Keep use of pointers simple and straightforward

##### Reason

Complicated pointer manipulation is a major source of errors.

* Do all pointer arithmetic on an `array_view` (exception ++p in simple loop???)
* Avoid pointers to pointers
* ???

##### Example

    ???

##### Enforcement

We need a heuristic limiting the complexity of pointer arithmetic statement.

### <a name="Res-order"></a> ES.43: Avoid expressions with undefined order of evaluation

##### Reason

You have no idea what such code does. Portability.
Even if it does something sensible for you, it may do something different on another compiler (e.g., the next release of your compiler) or with a different optimizer setting.

##### Example

    v[i] = ++i;	//  the result is undefined

A good rule of thumb is that you should not read a value twice in an expression where you write to it.

##### Example

    ???

##### Note

What is safe?

##### Enforcement

Can be detected by a good analyzer.

### <a name="Res-order-fct"></a> ES.44: Don't depend on order of evaluation of function arguments

##### Reason

Because that order is unspecified.

##### Example

    int i = 0;
    f(++i, ++i);

The call will most likely be `f(0, 1)` or `f(1, 0)`, but you don't know which. Technically, the behavior is undefined.

##### Example

??? overloaded operators can lead to order of evaluation problems (shouldn't :-()

    f1()->m(f2());	// m(f1(), f2())
    cout << f1() << f2();	// operator<<(operator<<(cout, f1()), f2())

##### Enforcement

Can be detected by a good analyzer.

### <a name="Res-magic"></a> ES.45: Avoid "magic constants"; use symbolic constants

##### Reason

Unnamed constants embedded in expressions are easily overlooked and often hard to understand:

##### Example

    for (int m = 1; m <= 12; ++m)	// don't: magic constant 12
        cout << month[m] << '\n';

No, we don't all know that there are 12 months, numbered 1..12, in a year. Better:

    constexpr int last_month = 12;	// months are numbered 1..12

    for (int m = first_month; m <= last_month; ++m)	// better
        cout << month[m] << '\n';

Better still, don't expose constants:

    for (auto m : month)
        cout << m << '\n';

##### Enforcement

Flag literals in code. Give a pass to `0`, `1`, `nullptr`, `\n`, `""`, and others on a positive list.

### <a name="Res-narrowing"></a> ES.46: Avoid lossy (narrowing, truncating) arithmetic conversions

##### Reason

A narrowing conversion destroys information, often unexpectedly so.

##### Example

A key example is basic narrowing:

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

The guideline support library offers a `narrow` operation for specifying that narrowing is acceptable and a `narrow` ("narrow if") that throws an exception if a narrowing would throw away information:

    i = narrow_cast<int>(d);		// OK (you asked for it): narrowing: i becomes 7
    i = narrow<int>(d);				// OK: throws narrowing_error

We also include lossy arithmetic casts, such as from a negative floating point type to an unsigned integral type:

    double d = -7.9;
    unsigned u = 0;

    u = d;                          // BAD
    u = narrow_cast<unsigned>(d);   // OK (you asked for it): u becomes 0
    u = narrow<unsigned>(d);        // OK: throws narrowing_error

##### Enforcement

A good analyzer can detect all narrowing conversions. However, flagging all narrowing conversions will lead to a lot of false positives. Suggestions:

* flag all floating-point to integer conversions (maybe only float->char and double->int. Here be dragons! we need data)
* flag all long->char (I suspect int->char is very common. Here be dragons! we need data)
* consider narrowing conversions for function arguments especially suspect

### <a name="Res-nullptr"></a> ES.47: Use `nullptr` rather than `0` or `NULL`

##### Reason

Readability. Minimize surprises: `nullptr` cannot be confused with an `int`.

##### Example

Consider:

    void f(int);
    void f(char*);
    f(0); 			// call f(int)
    f(nullptr); 	// call f(char*)

##### Enforcement

Flag uses of `0` and `NULL` for pointers. The transformation may be helped by simple program transformation.

### <a name="Res-casts"></a> ES.48: Avoid casts

##### Reason

Casts are a well-known source of errors. Makes some optimizations unreliable.

##### Example

    ???

##### Note

Programmer who write casts typically assumes that they know what they are doing.
In fact, they often disable the general rules for using values.
Overload resolution and template instantiation usually pick the right function if there is a right function to pick.
If there is not, maybe there ought to be, rather than applying a local fix (cast).

##### Note

Casts are necessary in a systems programming language.
For example, how else would we get the address of a device register into a pointer.
However, casts are seriously overused as well as a major source of errors.

##### Note

If you feel the need for a lot of casts, there may be a fundamental design problem.

##### Enforcement

* Force the elimination of C-style casts
* Warn against named casts
* Warn if there are many functional style casts (there is an obvious problem in quantifying 'many').

### <a name="Res-casts-named"></a> ES.49: If you must use a cast, use a named cast

##### Reason

Readability. Error avoidance.
Named casts are more specific than a C-style or functional cast, allowing the compiler to catch some errors.

The named casts are:

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

Flag C-style and functional casts.

## <a name="Res-casts-const"></a> ES.50: Don't cast away `const`

##### Reason

It makes a lie out of `const`.

##### Note

Usually the reason to "cast away `const`" is to allow the updating of some transient information of an otherwise immutable object.
Examples are cashing, memorization, and precomputation.
Such examples are often handled as well or better using `mutable` or an indirection than with a `const_cast`.

##### Example

    ???

##### Enforcement

Flag `const_cast`s.

### <a name="Res-range-checking"></a> ES.55: Avoid the need for range checking

##### Reason

Constructs that cannot overflow, don't, and usually runs faster:

##### Example

    for (auto& x : v)		// print all elements of v
        cout << x << '\n';

    auto p = find(v, x);		// find x in v

##### Enforcement

Look for explicit range checks and heuristically suggest alternatives.

### <a name="Res-new"></a> ES.60: Avoid `new` and `delete[]` outside resource management functions

##### Reason

Direct resource management in application code is error-prone and tedious.

##### Note

also known as "No naked `new`!"

##### Example, bad

    void f(int n)
    {
        auto p = new X[n];	// n default constructed Xs
        // ...
        delete[] p;
    }

There can be code in the `...` part that causes the `delete` never to happen.

**See also**: [R: Resource management](#S-resource).

##### Enforcement

Flag naked `new`s and naked `delete`s.

### <a name="Res-del"></a> ES.61: delete arrays using `delete[]` and non-arrays using `delete`

##### Reason

That's what the language requires and mistakes can lead to resource release errors and/or memory corruption.

##### Example, bad

    void f(int n)
    {
        auto p = new X[n];	// n default constructed Xs
        // ...
        delete p;			// error: just delete the object p, rather than delete the array p[]
    }

##### Note

This example not only violates the [no naked `new` rule](#Res-new) as in the previous example, it has many more problems.

##### Enforcement

* if the `new` and the `delete` is in the same scope, mistakes can be flagged.
* if the `new` and the `delete` are in a constructor/destructor pair, mistakes can be flagged.

### <a name="Res-arr2"></a> ES.62: Don't compare pointers into different arrays

##### Reason

The result of doing so is undefined.

##### Example, bad

    void f(int n)
    {
        int a1[7];
        int a2[9];
        if (&a1[5] < &a2[7]) {}       // bad: undefined
        if (0 < &a1[5] - &a2[7]) {}   // bad: undefined
    }

##### Note

This example has many more problems.

##### Enforcement

## <a name="SS-numbers"></a> Arithmetic

### <a name="Res-mix"></a> ES.100: Don't mix signed and unsigned arithmetic

##### Reason

Avoid wrong results.

##### Example

    unsigned x = 100;
    unsigned y = 102;
    cout << abs(x-y) << '\n'; // wrong result

##### Note

Unfortunately, C++ uses signed integers for array subscripts and the standard library uses unsigned integers for container subscripts.
This precludes consistency.

##### Enforcement

Compilers already know and sometimes warn.

### <a name="Res-unsigned"></a> ES.101: use unsigned types for bit manipulation

##### Reason

Unsigned types support bit manipulation without surprises from sign bits.

##### Example

    ???

**Exception**: Use unsigned types if you really want modulo arithmetic.

##### Enforcement

???

### <a name="Res-signed"></a> ES.102: 산술연산을 위해서는 부호있는 타입을 사용하라.
>### <a name="Res-signed"></a> ES.102: Used signed types for arithmetic

##### Reason

부호없는 타입은 부호비트부터 비트 연산을 해버리기 때문이다. (?)
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

`%` 모듈라도 같이 적용된다.
>this also applies to `%`.

##### Example

    ???

**Alternative**: 어느 정도의 오버헤드를 감수할 수 있는 대단히 중요한 프로그램에서는 정수 범위 체크나 부동소수점 타입을 사용하라.
>**Alternative**: For critical applications that can afford some overhead, use a range-checked integer and/or floating-point type.

##### Enforcement

???
