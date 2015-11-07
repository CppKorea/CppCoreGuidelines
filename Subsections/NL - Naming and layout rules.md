<a name="S-naming"></a>
# NL: 이름짓기와 코드 배치 규칙
> NL: Naming and layout rules

만약 다른 이유가 없다면 일관된 이름짓기와 배치는 도움이 된다. 이는 "내 스타일이 당신의 것보다 더 좋다"라는 논의를 최소화할 수 있기 때문이다.
그러나 여기저기 매우 다양한 스타일들이 있어 사람들은 그것들에 대해 (찬반으로) 열렬하다.
게다가 대부분의 실제 프로젝트들은 많은 원점(sources)으로부터 코드를 포함하여, 모든 코드를 단일 스타일로 표준화하는 것은 흔히 불가능하다.
여기서 더 좋은 아이디어가 없다면 사용할 일련의 규칙들 소개하며, 진짜 목표는 특정 규칙 집합이 아닌 일관성이다.
IDE들과 툴들이 도와줄 수 있다. (게다가 방해도 할 것이다.)
> Consistent naming and layout are helpful. If for no other reason because it minimizes "my style is better than your style" arguments.
However, there are many, many, different styles around and people are passionate about them (pro and con).
Also, most real-world projects includes code from many sources, so standardizing on a single style for all code is often impossible.
We present a set of rules that you might use if you have no better ideas, but the real aim is consistency, rather than any particular rule set.
IDEs and tools can help (as well as hinder).

이름짓기와 배치 규칙:
> Naming and layout rules:

* [NL 1: Don't say in comments what can be clearly stated in code](#Rl-comments)
* [NL.2: State intent in comments](#Rl-comments-intent)
* [NL.3: Keep comments crisp](#Rl-comments-crisp)
* [NL.4: Maintain a consistent indentation style](#Rl-indent)
* [NL.5: Don't encode type information in names](#Rl-name-type)
* [NL.7: Make the length of a name roughly proportional to the length of its scope](#Rl-name-length)
* [NL.8: Use a consistent naming style](#Rl-name)
* [NL 9: Use `ALL_CAPS` for macro names only](#Rl-all-caps)
* [NL.10: Avoid CamelCase](#Rl-camel)
* [NL.15: Use spaces sparingly](#Rl-space)
* [NL.16: Use a conventional class member declaration order](#Rl-order)
* [NL.17: Use K&R-derived layout](#Rl-knr)
* [NL.18: Use C++-style declarator layout](#Rl-ptr)
* [NL.25: Don't use `void` as an argument type](#Rl-void)

대부분의 규칙들은 미학적이고 개발자들은 강한 의견을 유지한다.
IDE 역시 기본값 외에 다양한 대안을 가지고 있는 경향이다. 이 규칙들은 이유가 되지 않는다면 기본으로 따르도록 제안된다.
> Most of these rules are aesthetic and programmers hold strong opinions.
IDEs also tend to have defaults and a range of alternatives. These rules are suggested defaults to follow unless you have reasons not to.

더 구체적이고 세부적인 규칙들이 시행하기 쉬울 것이다.
> More specific and detailed rules are easier to enforce.

<a name="Rl-comments"></a>
### NL.1: 코드에서 말할 수 있는 내용을 주석문에 넣지 마라.
> ### NL.1: Don't say in comments what can be clearly stated in code

#####근거
> #####Reason

컴파일러는 주석문을 읽지 않는다.
주석문은 코드보다 덜 정확하다.
주석문은 코드와 같이 일관되게 업데이트되지 않는다.
> Compilers do not read comments.
Comments are less precise than code.
Comments are not updated as consistently as code.

##### 잘못된 예
> ##### Example, bad

    auto x = m*v1 + vv;	// multiply m with v1 and add the result to vv

##### 시행하기
> ##### Enforcement

구어체 문장을 번역하는 인공지능 프로그램을 만들고 C++로 잘 표현할 수 있는지 살펴봐라.
> Build an AI program that interprets colloquial English text and see if what is said could be better expressed in C++.

<a name="Rl-comments-intent"></a>
### NL.2: 주석문에 목적을 나타내라
> ### NL.2: State intent in comments

##### 근거
코드는 무엇을 할지가 아니라 무엇을 했는지를 말한다. 주석은 구현된 내용보다 목적이나 의도를 간결하고 명쾌하게 기술할 수 있다.
>
##### Reason
Code says what is done, not what is supposed to be done. Often intent can be stated more clearly and concisely than the implementation.

##### Example

    void stable_sort(Sortable& c)
        // sort c in the order determined by <, keep equal elements (as defined by ==) in their original relative order
    {
        // ... quite a few lines of non-trivial code ...
    }

##### Note

주석과 코드가 다르다면 둘다 틀렸을거다.
>If the comment and the code disagrees, both are likely to be wrong.

### <a name="Rl-comments-crisp"></a> NL.3: 주석을 간략하게 유지하라.
>### <a name="Rl-comments-crisp"></a> NL.3: Keep comments crisp

##### Reason

말이 많으면 이해도가 떨어지고 소스파일에 퍼져 보여서 코드를 읽기 어렵게 만든다.
>Verbosity slows down understanding and makes the code harder to read by spreading it around in the source file.

##### Enforcement

불가능하다.
>not possible.

### <a name="Rl-indent"></a> NL.4: 일관적인 들여쓰기 스타일을 유지하라.
>### <a name="Rl-indent"></a> NL.4: Maintain a consistent indentation style

##### Reason

가독성 향상. 멍청한 실수를 피하기 위해.
>Readability. Avoidance of "silly mistakes."

##### Example, bad

    int i;
    for (i = 0; i < max; ++i); // bug waiting to happen
    if (i == j)
        return i;

##### Enforcement

툴을 사용하라.
>Use a tool.

### <a name="Rl-name-type"></a> NL.5 이름 안에 타입 정보를 포함하지 마라.
>### <a name="Rl-name-type"></a> NL.5 Don't encode type information in names

**Rationale**: 이름을 기능보다는 타입을 반영한다면 기능을 다른 종류의 타입으로 변경시키기가 힘들 것이다.
타입을 포함한 이름은 부차적이거나 약간 이해 못할 수도 있다.
헝가리안 표기법은 최악이다.(적어도 정적 타입 언어에 있어서는)
>**Rationale**: If names reflects type rather than functionality, it becomes hard to change the types used to provide that functionality.
Names with types encoded are either verbose or cryptic.
Hungarian notation is evil (at least in a strongly statically-typed language).

##### Example

    ???

##### Note

어떤 스타일은 지역변수와 맴버변수를 구분하거나 전역변수를 구분하려고 한다.
>Some styles distinguishes members from local variable, and/or from global variable.

    struct S {
        int m_;
        S(int m) :m_{abs(m)} { }
    };

이건 그렇게 나쁘지는 않다.
>This is not evil.

##### Note

또 어떤 스타일은 타입과 비타입을 구분하려고 한다.
>Some styles distinguishes types from non-types.

    typename<typename T>
    class Hash_tbl {	// maps string to T
        // ...
    };

    Hash_tbl<int> index;

이것도 그렇게 나쁘지는 않다.
>This is not evil.

### <a name="Rl-name-length"></a> NL.7: 변수범위의 크기에 비례해서 이름의 길이를 맞춰라.
>### <a name="Rl-name-length"></a> NL.7: Make the length of a name roughly proportional to the length of its scope

**Rationale**: ???

##### Example

    ???

##### Enforcement

???

### <a name="Rl-name"></a> NL.8: 일관적인 이름짓기 스타일을 사용하라.
>### <a name="Rl-name"></a> NL.8: Use a consistent naming style

**Rationale**:  일관성 있게 이름을 정하면 가독성을 높여준다.
>**Rationale**: Consistence in naming and naming style increases readability.

##### Note

많은 스타일이 공존하고 복수개의 라이브러리를 사용할 때 모든 네이밍 방식을 따를 수는 없다.
다양한 라이브러리가 가진 고유 스타일을 버리고 자기 스타일을 선택하라.
>Where are many styles and when you use multiple libraries, you can't follow all their differences conventions.
Choose a "house style", but leave "imported" libraries with their original style.

##### Example

ISO 표준은 소문자, 숫자, `_`로 구분된 단어만 사용한다.
>ISO Standard, use lower case only and digits, separate words with underscores:

* `int`
* `vector`
* `my_map`

두개 짜리 `__`를 사용하지 마라.
>Avoid double underscores `__`.

##### Example

[Stroustrup](http://www.stroustrup.com/Programming/PPP-style.pdf):
대문자로 된 사용자 정의 타입과 컨셉을 지원하는 ISO 표준
>ISO Standard, but with upper case used for your own types and concepts:

* `int`
* `vector`
* `My_map`

##### Example

낙타표기법(CamelCase): 여러 단어로 구성된 식별자에서 단어의 첫글자를 대문자로 한다.
>CamelCase: capitalize each word in a multi-word identifier:

* `int`
* `vector`
* `MyMap`
* `myMap`

맨 첫글자를 소문자로 하는 경우도 있다.
>Some conventions capitalize the first letter, some don't.

##### Note

약어나 식별자 길이도 일관성을 유지하도록 하라.
>Try to be consistent in your use of acronyms, lengths of identifiers:

    int mtbf {12};
    int mean_time_between_failor {12};		// make up your mind

##### Enforcement

스타일이 다른 라이브러리를 사용할 때를 뻬고는 시행이 가능할 것이다.
>Would be possible except for the use of libraries with varying conventions.

### <a name="Rl-all-caps"></a> NL 9: 매크로 명칭에만 전체 대문자를 사용하라.
>### <a name="Rl-all-caps"></a> NL 9: Use `ALL_CAPS` for macro names only

##### Reason

범위와 타입 규칙을 지키는 이름에 대해서는 매크로와 혼돈하지 않도록 하기 위해.
>To avoid confusing macros from names that obeys scope and type rules.

##### Example

    void f()
    {
        const int SIZE{1000};  // Bad, use 'size' instead
        int v[SIZE];
    }

##### Note

이 규칙은 매크로가 아닌 상수에도 적용된다.
>This rule applies to non-macro symbolic constants:

    enum bad { BAD, WORSE, HORRIBLE }; // BAD

##### Enforcement

* 소문자로 된 매크로는 표시하라.
* 매크로가 아닌데 대문자로 된 것은 표시하라.

>* Flag macros with lower-case letters
>* Flag `ALL_CAPS` non-macro names

### <a name="Rl-camel"></a> NL.10: 카멜케이스를 피하라.
>### <a name="Rl-camel"></a> NL.10: Avoid CamelCase

##### Reason

이름을 구분하기 위해 `_`를 사용하는 것은 원래 C/C++ 스타일이고 C++ 표준 라이브러리에도 사용하고 있다.
카멜케이스를 좋아한다면 여러 종류 중에서 한 종류만 선택하라.
>The use of underscores to separate parts of a name is the original C and C++ style and used in the C++ standard library.
If you prefer CamelCase, you have to choose among different flavors of camelCase.

##### Note

이 규칙은 일단 카멜케이스를 쓰겠다고 했을때 기본 규칙이다.
카멜케이스를 안 쓴다면 [consistency](#Rl-name)에 정의된 스타일을 따라야 한다.
일관성 유지는 개인적 기호보다 우선한다.
>This rule is a default to use only if you have a choice.
Often, you don't have a choice and must follow an established style for [consistency](#Rl-name).
The need for consistency beats personal taste.

##### Example

[Stroustrup](http://www.stroustrup.com/Programming/PPP-style.pdf):
사용자 정의 타입, 개념에 대해 대문자를 사용하는 ISO 표준
>ISO Standard, but with upper case used for your own types and concepts:

* `int`
* `vector`
* `My_map`

##### Enforcement

불가능하다.
>Impossible.

### <a name="Rl-space"></a> NL.15: 스페이스를 아껴서 사용하라.
>### <a name="Rl-space"></a> NL.15: Use spaces sparingly

##### Reason

너무 많은 스페이스는 코드를 길고 산만하게 만들기 때문에.
>Too much space makes the text larger and distracts.

##### Example, bad

    #include < map >

    int main (int argc, char * argv [ ])
    {
        // ...
    }

##### Example

    #include<map>

    int main(int argc, char* argv[])
    {
        // ...
    }

##### Note

몇몇 IDE는 그들만의 확신에 따라 추가적인 스페이스를 사용한다.
>Some IDEs have their own opinions and add distracting space.

##### Note

잘 정리된 스페이스는 가독성 향상에 많은 도움이 된다. 과도하게 쓰지는 말자.
>We value well-placed whitespace as a significant help for readability. Just don't overdo it.

### <a name="Rl-order"></a> NL.16: 일반적인 클래스 맴버 선언 순서를 지켜라.
>### <a name="Rl-order"></a> NL.16: Use a conventional class member declaration order

##### Reason

멤버 선언 순서는 가독성을 높여준다.
>A conventional order of members improves readability.

클래스 선언시 다음 순서를 사용하라.
>When declaring a class use the following order

* 타입: class, enum, alias (`using`구문)
* 생성자, 복사 생성자, 해제자.
* 함수
* 데이터

>* types: classes, enums, and aliases (`using`)
* constructors, assignments, destructor
* functions
* data

`private`, `protected`, `public`순으로 선언하라.
>Use the `public` before `protected` before `private` order.

비공개 타입과 함수는 비공개 데이터 영역에 둔다.
>Private types and functions can be placed with private data.

##### Example

    ???

##### Enforcement

제안된 순서와 다르면 표시를 해라. 이 규칙을 따르지 않는 예전 코드가 많이 있을꺼다.
>Flag departures from the suggested order. There will be a lot of old code that doesn't follow this rule.

### <a name="Rl-knr"></a> NL.17: K&R 방식의 레이아웃을 사용하라.
>### <a name="Rl-knr"></a> NL.17: Use K&R-derived layout

##### Reason

이 방식이 원래 C/C++ 레이아웃이다. 수직적인 배치가 유지되고 함수나 클래스 같은 언어 요소를 구분하기가 쉽다.
>This is the original C and C++ layout. It preserves vertical space well. It distinguishes different language constructs (such as functions and classes well).

##### Note

C++에서는 "Stroustrup" 스타일이라고 부른다.
>In the context of C++, this style is often called "Stroustrup".

##### Example

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

**Note** `if`와 `(`사이에 스페이스를 한칸 넣는다.
>**Note** a space between `if` and `(`

##### Note

`if`문, `for`문의 내용은 다음 라인을 사용하라.
>Use separate lines for each statement, the branches of an `if`, and the body of a `for`.

##### Note

`class`와 `struct`는 `{`를 한 라인으로 붙이고, 함수는 `{`를 다음 라인에 둔다.
>The `{` for a `class` and a `struct` in *not* on a separate line, but the `{` for a function is.

##### Note

표준라이브러리 타입과 구분하기 위해 사용자 정의 타입은 대문자로 이름을 짓는다.
>Capitalize the names of your user-defined types to distinguish them from standards-library types.

##### Note

함수 이름에는 대문자를 쓰지 마라.
>Do not capitalize function names.

##### Enforcement

이걸 시행하려면, IDE로 다시 포맷을 맞춰라.
>If you want enforcement, use an IDE to reformat.

### <a name="Rl-ptr"></a> NL.18: C++ 스타일의 선언 방식을 사용하라.
>### <a name="Rl-ptr"></a> NL.18: Use C++-style declarator layout

##### Reason

C 스타일의 선언 방식은 연산식과 문법에서 사용하는데 강조를 둔다면, C++ 스타일은 타입을 강조한다.
선언에서 사용시 인자는 참조를 붙이지 않도록 한다.
>The C-style layout emphasizes use in expressions and grammar, whereas the C++-style emphasizes types.
The use in expressions argument doesn't hold for references.

##### Example

    T& operator[](size_t);	// OK
    T &operator[](size_t);	// just strange
    T & operator[](size_t);	// undecided

##### Enforcement

역사적으로 볼때 불가능.
>Impossible in the face of history.

### <a name="Rl-void"></a> NL.25: 인자 타입으로 `void`를 사용하지 마라.
>### <a name="Rl-void"></a> NL.25: Don't use `void` as an argument type

##### Reason

말이 많고 C 호환이 문제될 때만 필요하기에.
>It's verbose and only needed where C compatibility matters.

##### Example

    void f(void);   // bad

    void g();       // better

###### Note

데니스 리치도 `void f(void)`는 abomination 인정했다.
함수 인터페이스가 드물게 금지될 경우 C에서 그런 abomination에 대해서 인자를 추가할 수 있다.
>Even Dennis Ritchie deemed `void f(void)` an abomination.
You can make an argument for that abomination in C when function prototypes were rare so that banning:

    int f();
    f(1, 2, "weird but valid C89");   // hope that f() is defined int f(a, b, c) char* c; { /* ... */ }

중요한 문제를 야기할 것인데 현재 C나 C++에는 전혀 문제되지 않는다.
>would have caused major problems, but not in the 21st century and in C++.
