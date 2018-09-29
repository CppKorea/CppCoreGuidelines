
# <a name="S-naming"></a>NL: 이름과 코드 배치

별다른 이유없이 "내 스타일이 당신 것보다 더 좋다"는 논란을 줄이기 위해서라면 일관성있는 이름짓기와 레이아웃은 도움이 된다.
굉장히 많고 많은 스타일이 존재하고 사람들은 그 스타일에 대해 열렬하게 관심(찬성 혹은 반대)을 보인다.

현실의 프로젝트들은 많은 소스코드를 가지고 있고, 모든 코드에 하나의 표준화된 스타일을 적용하는 것은 거의 불가능에 가깝다.

여기서 더 좋은 아이디어가 없다면 도입해도 무방한 규칙들을 소개하겠다. 그러나 규칙을 소개하는 진짜 목적은 특정한 규칙 그 자체가 아니라 일관성이다.

IDE나 툴들이 이것을 도와줄 것이다. (어쩌면 방해할지도 모른다.)

이름과 코드 배치 규칙 요약:

* [NL.1: Don't say in comments what can be clearly stated in code](#Rl-comments)
* [NL.2: State intent in comments](#Rl-comments-intent)
* [NL.3: Keep comments crisp](#Rl-comments-crisp)
* [NL.4: Maintain a consistent indentation style](#Rl-indent)
* [NL.5: Avoid encoding type information in names](#Rl-name-type)
* [NL.7: Make the length of a name roughly proportional to the length of its scope](#Rl-name-length)
* [NL.8: Use a consistent naming style](#Rl-name)
* [NL.9: Use `ALL_CAPS` for macro names only](#Rl-all-caps)
* [NL.10: Prefer `underscore_style` names](#Rl-camel)
* [NL.11: Make literals readable](#Rl-literals)
* [NL.15: Use spaces sparingly](#Rl-space)
* [NL.16: Use a conventional class member declaration order](#Rl-order)
* [NL.17: Use K&R-derived layout](#Rl-knr)
* [NL.18: Use C++-style declarator layout](#Rl-ptr)
* [NL.19: Avoid names that are easily misread](#Rl-misread)
* [NL.20: Don't place two statements on the same line](#Rl-stmt)
* [NL.21: Declare one name (only) per declaration](#Rl-dcl)
* [NL.25: Don't use `void` as an argument type](#Rl-void)
* [NL.26: Use conventional `const` notation](#Rl-const)

규칙들 대부분이 심미적이고 개발자들은 자기 의견을 강하게 표현한다.
IDE 역시 기본값 외에 몇가지 대안을 가지고 있는 상황이다. 특별한 이유가 없다면 이 규칙들을 기본으로 사용할 것을 제안한다.

We have had comments to the effect that naming and layout are so personal and/or arbitrary that we should not try to "legislate" them.
We are not "legislating" (see the previous paragraph).
However, we have had many requests for a set of naming and layout conventions to use when there are no external constraints.

더 자세한 내용들이 적용하기 쉬울 것이다.

These rules bear a strong resemblance to the recommendations in the [PPP Style Guide](http://www.stroustrup.com/Programming/PPP-style.pdf)
written in support of Stroustrup's [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html).

### <a name="Rl-comments"></a>NL.1: 코드만으로 알 수 있는 내용을 주석문에 넣지 마라

##### Reason

컴파일러는 주석을 읽지 않는다.   
주석이 코드만큼 정확하지 않다.  
주석은 코드와 동시에 업데이트되지 않는다.  

##### Example, bad
```c++
    auto x = m * v1 + vv;   // multiply m with v1 and add the result to vv
```
##### Enforcement

구어체 문장을 번역하는 인공지능 프로그램을 만들고 C++로 잘 표현할 수 있는지 검사한다.

### <a name="Rl-comments-intent"></a>NL.2: 주석에는 의도를 기술하라

##### Reason

코드는 무엇을 할지가 아니라 무엇을 했는지를 말한다. 주석은 구현된 내용보다 목적이나 의도를 간결하고 명쾌하게 기술할 수 있다.

##### Example
```c++
    void stable_sort(Sortable& c)
        // sort c in the order determined by <, keep equal elements (as defined by ==) in
        // their original relative order
    {
        // ... quite a few lines of non-trivial code ...
    }
```
##### Note
주석과 코드가 다르다면 둘다 틀렸을거다.

### <a name="Rl-comments-crisp"></a>NL.3: 주석을 간략하게 유지하라

##### Reason

말이 많으면 이해도가 떨어지고 소스파일에 퍼져 보여서 코드를 읽기 어렵게 만든다.

##### Note

Use intelligible English.
I may be fluent in Danish, but most programmers are not; the maintainers of my code may not be.
Avoid SMS lingo and watch your grammar, punctuation, and capitalization.
Aim for professionalism, not "cool."

##### Enforcement

not possible.

### <a name="Rl-indent"></a>NL.4: 들여쓰기 스타일을 일관성 있게 하라

##### Reason

가독성 향상. 멍청한 실수를 피하기 위해.

##### Example, bad
```c++
    int i;
    for (i = 0; i < max; ++i); // bug waiting to happen
    if (i == j)
        return i;
```
##### Note

Always indenting the statement after `if (...)`, `for (...)`, and `while (...)` is usually a good idea:
```c++
    if (i < 0) error("negative argument");

    if (i < 0)
        error("negative argument");
```
##### Enforcement

툴을 사용하라.

### <a name="Rl-name-type"></a>NL.5: 이름 안에 타입 정보를 포함하지 마라

##### Rationale

이름이 기능보다는 타입을 반영한다면, 기능을 제공하는 타입들을 바꾸기 어려워진다.
또한, 타입이 바뀌면 타입을 사용하는 변수도 함께 바뀌어야 한다. Minimize unintentional conversions.

##### Example, bad
```c++
    void print_int(int i);
    void print_string(const char*);

    print_int(1);          // repetitive, manual type matching
    print_string("xyzzy"); // repetitive, manual type matching
```
##### Example, good
```c++
    void print(int i);
    void print(string_view);    // also works on any string-like sequence

    print(1);              // clear, automatic type matching
    print("xyzzy");        // clear, automatic type matching
```
##### Note

Names with types encoded are either verbose or cryptic.
```
    printS  // print a std::string
    prints  // print a C-style string
    printi  // print an int
```
Requiring techniques like Hungarian notation to encode a type in a name is needed in C, but is generally unnecessary and actively harmful in a strongly statically-typed language like C++, because the annotations get out of date (the warts are just like comments and rot just like them) and they interfere with good use of the language (use the same name and overload resolution instead).

##### Note

Some styles use very general (not type-specific) prefixes to denote the general use of a variable.
```c++
    auto p = new User();
    auto p = make_unique<User>();
    // note: "p" is not being used to say "raw pointer to type User,"
    //       just generally to say "this is an indirection"

    auto cntHits = calc_total_of_hits(/*...*/);
    // note: "cnt" is not being used to encode a type,
    //       just generally to say "this is a count of something"
```
This is not harmful and does not fall under this guideline because it does not encode type information.

##### Note

어떤 스타일은 지역변수와 멤버변수를 구분하거나 전역변수를 구분하려고 한다.
```c++
    struct S {
        int m_;
        S(int m) :m_{abs(m)} { }
    };
```
이는 유해하지 않고 타입 정보를 포함하지 않기 때문에 가이드라인에도 위배되지 않는다.

##### Note

어떤 스타일은 타입과 비타입을 구분하려고 한다.
For example, by capitalizing type names, but not the names of functions and variables.
```c++
    typename<typename T>
    class HashTable {   // maps string to T
        // ...
    };

    HashTable<int> index;
```
이는 유해하지 않고 타입 정보를 포함하지 않기 때문에 가이드라인에도 위배되지 않는다.

### <a name="Rl-name-length"></a>NL.7: Make the length of a name roughly proportional to the length of its scope

**Rationale**: The larger the scope the greater the chance of confusion and of an unintended name clash.

##### Example
```c++
    double sqrt(double x);   // return the square root of x; x must be non-negative

    int length(const char* p);  // return the number of characters in a zero-terminated C-style string

    int length_of_string(const char zero_terminated_array_of_char[])    // bad: verbose

    int g;      // bad: global variable with a cryptic name

    int open;   // bad: global variable with a short, popular name
```
The use of `p` for pointer and `x` for a floating-point variable is conventional and non-confusing in a restricted scope.

##### Enforcement

???

### <a name="Rl-name"></a>NL.8: 일관적인 이름짓기 스타일을 사용하라

**Rationale**: 일관성 있게 이름을 정하면 가독성을 높여준다.

##### Note

많은 스타일이 공존하고 복수개의 라이브러리를 사용할 때 모든 네이밍 방식을 따를 수는 없다.
"가져와 사용하는" 라이브러리가 가진 고유 스타일은 그대로 두고자기 스타일을 선택하라.

##### Example
ISO 표준은 소문자, 숫자, `_`로 구분된 단어만 사용한다:

* `int`
* `vector`
* `my_map`

두개 짜리 `__`를 사용하지 마라.

##### Example

[Stroustrup](http://www.stroustrup.com/Programming/PPP-style.pdf):
ISO Standard, but with upper case used for your own types and concepts:

* `int`
* `vector`
* `My_map`

##### Example

낙타표기법(CamelCase): 여러 단어로 구성된 식별자에서 단어의 첫글자를 대문자로 한다:

* `int`
* `vector`
* `MyMap`
* `myMap`

맨 첫글자를 소문자로 하는 경우도 있다.

##### Note
약어(acronym)나 식별자 길이도 일관성을 유지하도록 하라:
```c++
    int mtbf {12};
    int mean_time_between_failures {12}; // make up your mind
```
##### Enforcement
스타일이 다른 라이브러리를 사용할 때를 뻬고는 시행이 가능할 것이다.

### <a name="Rl-all-caps"></a>NL.9: 매크로 명칭에만 전체 대문자를 사용하라

##### Reason

범위와 타입 규칙을 지키는 이름에 대해서는 매크로와 혼돈하지 않도록 하기 위해.

##### Example
```c++
    void f()
    {
        const int SIZE{1000};  // Bad, use 'size' instead
        int v[SIZE];
    }
```
##### Note
이 규칙은 매크로가 아닌 상수에도 적용된다:
```c++
    enum bad { BAD, WORSE, HORRIBLE }; // BAD
```
##### Enforcement

* 소문자로 된 매크로를 지적하라
* 매크로가 아닌데 대문자로 된 경우를 지적하라

### <a name="Rl-camel"></a>NL.10: `underscore_style`형태의 이름을 선호하라

##### Reason

이름의 각 부분을 구분하기 위해 `_`를 사용하는 것은 원래 C/C++ 스타일이고 C++ 표준 라이브러리에도 사용하고 있다.

##### Note
이 규칙은 선택이 필요할 때의 기본 규칙이다. 이미 정립된 스타일과의 [일관성](#Rl-name)을 지켜야 하는 경우도 있을 수 있다. 
개인의 취향보다는 일관성이 중요하다.

This is a recommendation for [when you have no constraints or better ideas](#S-naming).
Thus rule was added after many requests for guidance.

##### Example

[Stroustrup](http://www.stroustrup.com/Programming/PPP-style.pdf):
사용자 정의 타입, 개념에 대해 대문자를 사용하는 ISO 표준:

* `int`
* `vector`
* `My_map`

##### Enforcement

불가능하다.

### <a name="Rl-space"></a>NL.15: 스페이스를 아껴서 사용하라

##### Reason

너무 많은 스페이스는 코드를 길고 산만하게 만든다

##### Example, bad
```c++
    #include < map >

    int main(int argc, char * argv [ ])
    {
        // ...
    }
```
##### Example
```c++
    #include <map>

    int main(int argc, char* argv[])
    {
        // ...
    }
```
##### Note

몇몇 IDE는 그들만의 확신에 따라 추가적인 스페이스를 사용한다.

This is a recommendation for [when you have no constraints or better ideas](#S-naming).
Thus rule was added after many requests for guidance.

##### Note

잘 정리된 스페이스는 가독성 향상에 많은 도움이 된다. 과도하지만 않으면 된다.

### <a name="Rl-literals"></a>NL.11: Make literals readable

##### Reason

Readability.

##### Example

Use digit separators to avoid long strings of digits
```c++
    auto c = 299'792'458; // m/s2
    auto q2 = 0b0000'1111'0000'0000;
    auto ss_number = 123'456'7890;
```
##### Example

Use literal suffixes where clarification is needed
```c++
    auto hello = "Hello!"s; // a std::string
    auto world = "world";   // a C-style string
    auto interval = 100ms;  // using <chrono>
```
##### Note

Literals should not be sprinkled all over the code as ["magic constants"](#Res-magic),
but it is still a good idea to make them readable where they are defined.
It is easy to make a typo in a long string of integers.

##### Enforcement

Flag long digit sequences. The trouble is to define "long"; maybe 7.

### <a name="Rl-order"></a>NL.16: 일반적인 클래스 멤버 선언 순서를 지켜라

##### Reason

멤버 선언 순서는 가독성을 높여준다.

클래스 선언시 다음 순서를 사용하라
* 타입: class, enum, alias (`using`구문)
* 생성자, 복사 생성자, 해제자.
* 함수
* 데이터

`public`, `protected`, `private` 순으로 선언하라.

This is a recommendation for [when you have no constraints or better ideas](#S-naming).
이 규칙은 수많은 요청에 의해 추가되었다.

##### Example
```c++
    class X {
    public:
        // interface
    protected:
        // unchecked function for use by derived class implementations
    private:
        // implementation details
    };
```
##### Example

Sometimes, the default order of members conflicts with a desire to separate the public interface from implementation details.
In such cases, private types and functions can be placed with private data.
```c++
    class X {
    public:
        // interface
    protected:
        // unchecked function for use by derived class implementations
    private:
        // implementation details (types, functions, and data)
    };
```
##### Example, bad

Avoid multiple blocks of declarations of one access (e.g., `public`) dispersed among blocks of declarations with different access (e.g. `private`).
```c++
    class X {   // bad
    public:
        void f();
    public:
        int g();
        // ...
    };
```
The use of macros to declare groups of members often leads to violation of any ordering rules.
However, macros obscures what is being expressed anyway.

##### Enforcement

제안된 순서와 다르면 지적하라. 이 규칙을 따르지 않는 예전 코드가 많이 있을 것이다.

### <a name="Rl-knr"></a>NL.17: K&R 방식의 레이아웃을 사용하라

##### Reason
이 방식이 C/C++의 고유한 레이아웃이다. 수직적인 배치가 유지되고 (함수나 클래스 같은) 언어 요소를 구분하기가 쉽다.

##### Note

C++에서는 "Stroustrup" 스타일이라고 부르기도 한다.

This is a recommendation for [when you have no constraints or better ideas](#S-naming).
Thus rule was added after many requests for guidance.

##### Example
```c++
    struct Cable {
        int x;
        // ...
    };

    double foo(int x)
    {
        if (0 < x) {
            // ...
        }

        switch (x) {
        case 0:
            // ...
            break;
        case amazing:
            // ...
            break;
        default:
            // ...
            break;
        }

        if (0 < x)
            ++x;

        if (x < 0)
            something();
        else
            something_else();

        return some_value;
    }
```
`if`와 `(`사이에 스페이스를 한칸 넣는다는 점에 유의하라.

##### Note
Use separate lines for each statement, the branches of an `if`, and the body of a `for`.

##### Note

The `{` for a `class` and a `struct` is *not* on a separate line, but the `{` for a function is.

##### Note

표준라이브러리 타입과 구분하기 위해 사용자 정의 타입은 대문자로 이름을 짓는다.

##### Note

함수 이름에는 대문자를 쓰지 마라.

##### Enforcement

IDE로 다시 포맷을 맞춰라.

### <a name="Rl-ptr"></a>NL.18: C++ 스타일의 선언 방식을 사용하라

##### Reason

C 스타일의 선언 방식은 표현식과 문법에서 사용하는데 강조를 둔다면, C++ 스타일은 타입을 강조한다.
선언에서 사용시 인자는 참조를 붙이지 않도록 한다.

##### Example
```c++
    T& operator[](size_t);   // OK
    T &operator[](size_t);   // just strange
    T & operator[](size_t);  // undecided
```
##### Note

This is a recommendation for [when you have no constraints or better ideas](#S-naming).
Thus rule was added after many requests for guidance.

##### Enforcement

역사적으로 볼때 불가능.


### <a name="Rl-misread"></a>NL.19: Avoid names that are easily misread

##### Reason

Readability.
Not everyone has screens and printers that make it easy to distinguish all characters.
We easily confuse similarly spelled and slightly misspelled words.

##### Example
```c++
    int oO01lL = 6; // bad

    int splunk = 7;
    int splonk = 8; // bad: splunk and splonk are easily confused
```
##### Enforcement

???

### <a name="Rl-stmt"></a>NL.20: Don't place two statements on the same line

##### Reason

Readability.
It is really easy to overlook a statement when there is more on a line.

##### Example
```c++
    int x = 7; char* p = 29;    // don't
    int x = 7; f(x);  ++x;      // don't
```
##### Enforcement

Easy.

### <a name="Rl-dcl"></a>NL.21: Declare one name (only) per declaration

##### Reason

Readability.
Minimizing confusion with the declarator syntax.

##### Note

For details, see [ES.10](#Res-name-one).


### <a name="Rl-void"></a>NL.25: 인자 타입으로 `void`를 사용하지 마라

##### Reason

C 호환이 문제될 때만 필요하다.

##### Example
```c++
    void f(void);   // bad

    void g();       // better
```
##### Note

데니스 리치도 `void f(void)`는 싫어한다고 말했다.
함수 인터페이스가 드물게 금지될 경우 C에서 그런 abomination에 대해서 인자를 추가할 수 있다:
```c++
    int f();
    f(1, 2, "weird but valid C89");   // hope that f() is defined int f(a, b, c) char* c; { /* ... */ }
```
중요한 문제를 야기할 것인데 21세기의 C나 C++에는 전혀 문제되지 않는다.

### <a name="Rl-const"></a>NL.26: Use conventional `const` notation

##### Reason

Conventional notation is more familiar to more programmers.
Consistency in large code bases.

##### Example
```c++
    const int x = 7;    // OK
    int const y = 9;    // bad

    const int *const p = nullptr;   // OK, constant pointer to constant int
    int const *const p = nullptr;   // bad, constant pointer to constant int
```
##### Note

We are well aware that you could claim the "bad" examples more logical than the ones marked "OK",
but they also confuse more people, especially novices relying on teaching material using the far more common, conventional OK style.

As ever, remember that the aim of these naming and layout rules is consistency and that aesthetics vary immensely.

This is a recommendation for [when you have no constraints or better ideas](#S-naming).
Thus rule was added after many requests for guidance.

##### Enforcement

Flag `const` used as a suffix for a type.
