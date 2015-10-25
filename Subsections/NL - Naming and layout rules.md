# <a name="S-naming"></a> NL: Naming and layout rules

Consistent naming and layout are helpful. If for no other reason because it minimizes "my style is better than your style" arguments.
However, there are many, many, different styles around and people are passionate about them (pro and con).
Also, most real-world projects includes code from many sources, so standardizing on a single style for all code is often impossible.
We present a set of rules that you might use if you have no better ideas, but the real aim is consistency, rather than any particular rule set.
IDEs and tools can help (as well as hinder).

Naming and layout rules:

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

Most of these rules are aesthetic and programmers hold strong opinions.
IDEs also tend to have defaults and a range of alternatives.These rules are suggested defaults to follow unless you have reasons not to.

More specific and detailed rules are easier to enforce.

### <a name="Rl-comments"></a> NL.1: Don't say in comments what can be clearly stated in code

##### Reason

Compilers do not read comments.
Comments are less precise than code.
Comments are not updated as consistently as code.

##### Example, bad

    auto x = m*v1 + vv;	// multiply m with v1 and add the result to vv

##### Enforcement

Build an AI program that interprets colloquial English text and see if what is said could be better expressed in C++.

### <a name="Rl-comments-intent"></a> NL.2: State intent in comments

##### Reason

Code says what is done, not what is supposed to be done. Often intent can be stated more clearly and concisely than the implementation.

##### Example

    void stable_sort(Sortable& c)
        // sort c in the order determined by <, keep equal elements (as defined by ==) in their original relative order
    {
        // ... quite a few lines of non-trivial code ...
    }

##### Note

If the comment and the code disagrees, both are likely to be wrong.

### <a name="Rl-comments-crisp"></a> NL.3: Keep comments crisp

##### Reason

Verbosity slows down understanding and makes the code harder to read by spreading it around in the source file.

##### Enforcement

not possible.

### <a name="Rl-indent"></a> NL.4: Maintain a consistent indentation style

##### Reason

Readability. Avoidance of "silly mistakes."

##### Example, bad

    int i;
    for (i = 0; i < max; ++i); // bug waiting to happen
    if (i == j)
        return i;

##### Enforcement

Use a tool.

### <a name="Rl-name-type"></a> NL.5 Don't encode type information in names

**Rationale**: If names reflects type rather than functionality, it becomes hard to change the types used to provide that functionality.
Names with types encoded are either verbose or cryptic.
Hungarian notation is evil (at least in a strongly statically-typed language).

##### Example

    ???

##### Note

Some styles distinguishes members from local variable, and/or from global variable.

    struct S {
        int m_;
        S(int m) :m_{abs(m)} { }
    };

This is not evil.

##### Note

Some styles distinguishes types from non-types.

    typename<typename T>
    class Hash_tbl {	// maps string to T
        // ...
    };

    Hash_tbl<int> index;

This is not evil.

### <a name="Rl-name-length"></a> NL.7: Make the length of a name roughly proportional to the length of its scope

**Rationale**: ???

##### Example

    ???

##### Enforcement

???

### <a name="Rl-name"></a> NL.8: Use a consistent naming style

**Rationale**: Consistence in naming and naming style increases readability.

##### Note

Where are many styles and when you use multiple libraries, you can't follow all their differences conventions.
Choose a "house style", but leave "imported" libraries with their original style.

##### Example

ISO Standard, use lower case only and digits, separate words with underscores:

* `int`
* `vector`
* `my_map`

Avoid double underscores `__`.

##### Example

[Stroustrup](http://www.stroustrup.com/Programming/PPP-style.pdf):
ISO Standard, but with upper case used for your own types and concepts:

* `int`
* `vector`
* `My_map`

##### Example

CamelCase: capitalize each word in a multi-word identifier:

* `int`
* `vector`
* `MyMap`
* `myMap`

Some conventions capitalize the first letter, some don't.

##### Note

Try to be consistent in your use of acronyms, lengths of identifiers:

    int mtbf {12};
    int mean_time_between_failor {12};		// make up your mind

##### Enforcement

Would be possible except for the use of libraries with varying conventions.

### <a name="Rl-all-caps"></a> NL 9: Use `ALL_CAPS` for macro names only

##### Reason

To avoid confusing macros from names that obeys scope and type rules.

##### Example

    void f()
    {
        const int SIZE{1000};  // Bad, use 'size' instead
        int v[SIZE];
    }

##### Note

This rule applies to non-macro symbolic constants:

    enum bad { BAD, WORSE, HORRIBLE }; // BAD

##### Enforcement

* Flag macros with lower-case letters
* Flag `ALL_CAPS` non-macro names

### <a name="Rl-camel"></a> NL.10: Avoid CamelCase

##### Reason

The use of underscores to separate parts of a name is the original C and C++ style and used in the C++ standard library.
If you prefer CamelCase, you have to choose among different flavors of camelCase.

##### Note

This rule is a default to use only if you have a choice.
Often, you don't have a choice and must follow an established style for [consistency](#Rl-name).
The need for consistency beats personal taste.

##### Example

[Stroustrup](http://www.stroustrup.com/Programming/PPP-style.pdf):
ISO Standard, but with upper case used for your own types and concepts:

* `int`
* `vector`
* `My_map`

##### Enforcement

Impossible.

### <a name="Rl-space"></a> NL.15: Use spaces sparingly

##### Reason

Too much space makes the text larger and distracts.

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

Some IDEs have their own opinions and add distracting space.

##### Note

We value well-placed whitespace as a significant help for readability. Just don't overdo it.

### <a name="Rl-order"></a> NL.16: Use a conventional class member declaration order

##### Reason

A conventional order of members improves readability.

When declaring a class use the following order

* types: classes, enums, and aliases (`using`)
* constructors, assignments, destructor
* functions
* data

Use the `public` before `protected` before `private` order.

Private types and functions can be placed with private data.

##### Example

    ???

##### Enforcement

Flag departures from the suggested order. There will be a lot of old code that doesn't follow this rule.

### <a name="Rl-knr"></a> NL.17: K&R 방식의 레이아웃을 사용하라.
>### <a name="Rl-knr"></a> NL.17: Use K&R-derived layout

##### Reason

이 방식이 원래 C/C++ 레이아웃이다. vertical space를 잘 보존한다.
다른 언어 구성을 구별시킬 수 있다.(함수와 클래스를 잘.)
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
