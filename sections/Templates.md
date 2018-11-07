
# <a name="S-templates"></a>T: Templates and generic programming

제네릭(generic) 프로그래밍은 타입, 값, 알고리즘을 매개변수화하는 타입과 알고리즘을 사용하는 프로그래밍이다.
C++에서 제네릭 프로그래밍은 `template`을 언어적 장치로 지원하고 있다.

일반화된 함수의 인자는 인자타입과 관련된 값에 대한 요구사항들을 특징짓는다.
C++에서 이런 요구사항은 컨셉이라는 컴파일타임 서술어로 표현된다.

템플릿은 메타프로그래밍을 목적으로 사용될 수 있다; 즉 컴파일 타임 때 코드를 만들어내는 형태의 프로그램이다.

A central notion in generic programming is "concepts"; that is, requirements on template arguments presented as compile-time predicates.
"Concepts" are defined in an ISO Technical specification: [concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
A draft of a set of standard-library concepts can be found in another ISO TS: [ranges](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf)
Concepts are supported in GCC 6.1 and later.
Consequently, we comment out uses of concepts in examples; that is, we use them as formalized comments only.
If you use GCC 6.1 or later, you can uncomment them.

템플릿 사용 규칙 요약:

* [T.1: 더 높은 코드 추상화를 위해서 템플릿을 사용하라](#Rt-raise)
* [T.2: 다양한 인자 타입들에게 적용되는 알고리즘에 템플릿을 사용하라](#Rt-algo)
* [T.3: 컨테이너와 구간(range)을 표현하기 위해서 템플릿을 사용하라](#Rt-cont)
* [T.4: 문법 트리 조작을 표현하기 위해 템플릿을 사용하라](#Rt-expr)
* [T.5: 강점을 증폭시키도록 제네릭 프로그래밍과 개체지향 기술을 결합하라](#Rt-generic-oo)

컨셉 사용 규칙 요약:

* [T.10: 템플릿 인자들에 concept들을 명시하라](#Rt-concepts)
* [T.11: 가능하다면 표준 concept들을 사용하라](#Rt-std-concepts)
* [T.12: 지역 변수들에는 `auto`대신 concept 이름을 사용하라](#Rt-auto)
* [T.13: 단일 타입의 인자 concept들에는 단순한 표기를 사용하라](#Rt-shorthand)
* ???

컨셉 정의 규칙 요약:

* [T.20: 무의미한 "concepts"는 지양하라](#Rt-low)
* [T.21: Require a complete set of operations for a concept](#Rt-complete)
* [T.22: Specify axioms for concepts](#Rt-axiom)
* [T.23: 특수한 concept들을 새로운 사용 패턴을 추가해서 일반적인 concept들과 차별화 하라](#Rt-refine)
* [T.24: Use tag classes or traits to differentiate concepts that differ only in semantics](#Rt-tag)
* [T.25: Avoid complementary constraints](#Rt-not)
* [T.26: Prefer to define concepts in terms of use-patterns rather than simple syntax](#Rt-use)
* [T.30: Use concept negation (`!C<T>`) sparingly to express a minor difference](#Rt-not)
* [T.31: Use concept disjunction (`C1<T> || C2<T>`) sparingly to express alternatives](#Rt-or)
* ???

템플릿 인터페이스 규칙 요약:

* [T.40: Use function objects to pass operations to algorithms](#Rt-fo)
* [T.41: Require only essential properties in a template's concepts](#Rt-essential)
* [T.42: Use template aliases to simplify notation and hide implementation details](#Rt-alias)
* [T.43: Prefer `using` over `typedef` for defining aliases](#Rt-using)
* [T.44: Use function templates to deduce class template argument types (where feasible)](#Rt-deduce)
* [T.46: Require template arguments to be at least `Regular` or `SemiRegular`](#Rt-regular)
* [T.47: Avoid highly visible unconstrained templates with common names](#Rt-visible)
* [T.48: If your compiler does not support concepts, fake them with `enable_if`](#Rt-concept-def)
* [T.49: Where possible, avoid type-erasure](#Rt-erasure)

템플릿 정의 규칙 요약:

* [T.60: Minimize a template's context dependencies](#Rt-depend)
* [T.61: Do not over-parameterize members (SCARY)](#Rt-scary)
* [T.62: Place non-dependent class template members in a non-templated base class](#Rt-nondependent)
* [T.64: Use specialization to provide alternative implementations of class templates](#Rt-specialization)
* [T.65: Use tag dispatch to provide alternative implementations of functions](#Rt-tag-dispatch)
* [T.67: Use specialization to provide alternative implementations for irregular types](#Rt-specialization2)
* [T.68: Use `{}` rather than `()` within templates to avoid ambiguities](#Rt-cast)
* [T.69: Inside a template, don't make an unqualified nonmember function call unless you intend it to be a customization point](#Rt-customization)

템플릿과 상속구조 규칙 요약:

* [T.80: Do not naively templatize a class hierarchy](#Rt-hier)
* [T.81: Do not mix hierarchies and arrays](#Rt-array) // ??? somewhere in "hierarchies"
* [T.82: Linearize a hierarchy when virtual functions are undesirable](#Rt-linear)
* [T.83: Do not declare a member function template virtual](#Rt-virtual)
* [T.84: Use a non-template core implementation to provide an ABI-stable interface](#Rt-abi)
* [T.??: ????](#Rt-???)

가변인자 템플릿규칙 요약:

* [T.100: Use variadic templates when you need a function that takes a variable number of arguments of a variety of types](#Rt-variadic)
* [T.101: ??? How to pass arguments to a variadic template ???](#Rt-variadic-pass)
* [T.102: ??? How to process arguments to a variadic template ???](#Rt-variadic-process)
* [T.103: Don't use variadic templates for homogeneous argument lists](#Rt-variadic-not)
* [T.??: ????](#Rt-???)

메타프로그래밍 규칙 요약:

* [T.120: Use template metaprogramming only when you really need to](#Rt-metameta)
* [T.121: Use template metaprogramming primarily to emulate concepts](#Rt-emulate)
* [T.122: Use templates (usually template aliases) to compute types at compile time](#Rt-tmp)
* [T.123: Use `constexpr` functions to compute values at compile time](#Rt-fct)
* [T.124: Prefer to use standard-library TMP facilities](#Rt-std-tmp)
* [T.125: If you need to go beyond the standard-library TMP facilities, use an existing library](#Rt-lib)
* [T.??: ????](#Rt-???)

다른 템플릿 규칙 요약:

* [T.140: Name all operations with potential for reuse](#Rt-name)
* [T.141: Use an unnamed lambda if you need a simple function object in one place only](#Rt-lambda)
* [T.142: Use template variables to simplify notation](#Rt-var)
* [T.143: Don't write unintentionally nongeneric code](#Rt-nongeneric)
* [T.144: Don't specialize function templates](#Rt-specialize-function)
* [T.150: Check that a class matches a concept using `static_assert`](#Rt-check-class)
* [T.??: ????](#Rt-???)

## <a name="SS-GP"></a>T.gp: 제네릭 프로그래밍(Generic programming)

제네릭 프로그래밍은 타입, 값, 알고리즘을 매개변수로 사용하는 타입과 알고리즘을 사용하는 프로그래밍이다.

### <a name="Rt-raise"></a>T.1: Use templates to raise the level of abstraction of code

##### Reason

일반화. 재사용성. 효율성. 사용자 타입의 일관된 정의를 장려한다.

##### Example, bad

개념적으로, 아래 요구사항은 잘못됐다. 왜냐하면 우리가 원하는 `T`는 "증가될 수 있다"거나 "추가될 수 있다"는 하위레벨 컨셉 그 이상이다:

```c++
    template<typename T>
        // requires Incrementable<T>
    T sum1(vector<T>& v, T s)
    {
        for (auto x : v) s += x;
        return s;
    }

    template<typename T>
        // requires Simple_number<T>
    T sum2(vector<T>& v, T s)
    {
        for (auto x : v) s = s + x;
        return s;
    }
```

`Incrementable`이 `+`를 지원하지 않고, `Simple_number`이 `+=`를 지원하지 않는다고 가정하면 `sum1`과 `sum2`의 구현을 과도하게 제약해왔다.
그리고 이런 경우에는 일반화를 위해 기회를 놓친 것이다.

##### Example

```c++
    template<typename T>
        // requires Arithmetic<T>
    T sum(vector<T>& v, T s)
    {
        for (auto x : v) s += x;
        return s;
    }
```

`Arithmetic` concept가 `+`와 `+=`를 모두 요구한다고 가정하면, 우리는 `sum`의 사용자가 두 연산을 지원하는 산술 타입을 제공하도록 제약한 것이다.
That is not a minimal requirement, but it gives the implementer of algorithms much needed freedom and ensures that any `Arithmetic` type
can be used for a wide variety of algorithms.

더 일반적인, 재사용 가능한 코드를 위해서, `vector`와 같은 하나의 컨테이너만 사용하는 것이 아닌 `Container`또는 `Range`와 같은 더 일반적인 concept를 사용하는 것도 가능하다.

##### Note

If we define a template to require exactly the operations required for a single implementation of a single algorithm
(e.g., requiring just `+=` rather than also `=` and `+`) and only those, we have overconstrained maintainers.
We aim to minimize requirements on template arguments, but the absolutely minimal requirements of an implementation is rarely a meaningful concept.

##### Note

템플릿은 모든 것을 표현하는데 사용할 수 있다. (튜링 완전성을 지닌다)
그러나 일반화 프로그래밍의 목표는 비슷한 의미특성을 가진 타입집합에 대한 연산/알고리즘을 효과적으로 일반화하는 것이다.

> 역주: 튜링 완전성(https://ko.wikipedia.org/wiki/%ED%8A%9C%EB%A7%81_%EC%99%84%EC%A0%84)

##### Note

The `requires` in the comments are uses of `concepts`.
"Concepts" are defined in an ISO Technical specification: [concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
Concepts are supported in GCC 6.1 and later.
Consequently, we comment out uses of concepts in examples; that is, we use them as formalized comments only.
If you use GCC 6.1 or later, you can uncomment them.

##### Enforcement

* 컨셉없이 특정 연산자를 바로 사용하는 것같은, 과도하게 단순한 요구사항을 가진 알고리즘이 있다면 지적한다
* "과도하게 단순한" 컨셉 정의가 있다면 지적하지 않는다; 더 쓸만한 컨셉을 위해 만들었을지도 모른다.

### <a name="Rt-algo"></a>T.2: Use templates to express algorithms that apply to many argument types

##### Reason

일반화. 소스 코드의 크기를 최소화한다. 상호운용성. 재사용.

##### Example

STL의 기본사항이다. `find` 알고리즘은 모든 종류의 입력 범위에서 문제없이 작동한다:

```c++
    template<typename Iter, typename Val>
        // requires Input_iterator<Iter>
        //       && Equality_comparable<Value_type<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }
```

##### Note

한개 이상의 템플릿 인자타입에 대한 현실적인 필요가 없다면 템플릿을 사용하지 마라.
과도하게 추상화하지 마라.

##### Enforcement

??? tough, probably needs a human

### <a name="Rt-cont"></a>T.3: Use templates to express containers and ranges

> 역주 : 구간(range)

##### Reason

컨테이너는 원소들의 타입을 필요로 하고, 이는 템플릿 인자로 표현하는 것이 일반적이다. 재사용이 가능하고, 타입 안전(Type Safe)하다.
이는 불안정하고 비효율적인 해결방법을 피한다.  

STL은 이 접근법을 사용한다.

##### Example

```c++
    template<typename T>
        // requires Regular<T>
    class Vector {
        // ...
        T* elem;   // points to sz Ts
        int sz;
    };

    Vector<double> v(10);
    v[7] = 9.9;
```

##### Example, bad

```c++
    class Container {
        // ...
        void* elem;   // points to size elements of some type
        int sz;
    };

    Container c(10, sizeof(double));
    ((double*) c.elem)[7] = 9.9;
```

이 코드는 프로그래머의 의도를 직접적으로 표현하지 않는다. 또한 타입시스템과 최적화기(optimizer)가 프로그램의 구조를 알 수 없도록 한다.

메크로 뒤에서 `void*`를 숨기는 것은 그저 문제를 어렵게 할 뿐이다. 새로운 혼란을 야기한다.

**Exceptions**: 고정된 ABI 지원 인터페이스가 필요하다면 기본 구현을 제공하고, 그 형태에 따라 타입에 안전한 템플릿을 표현해야 한다. [Stable base](#Rt-abi)를 참조하라.

##### Enforcement

* `void*`과 하위레벨 구현코드 외에 형변환을 사용한다면 지적한다.

### <a name="Rt-expr"></a>T.4: Use templates to express syntax tree manipulation

##### Reason

 ???

##### Example

???

**Exceptions**: ???

### <a name="Rt-generic-oo"></a>T.5: Combine generic and OO techniques to amplify their strengths, not their costs

##### Reason

제네릭 프로그래밍과 개체지향 기술은 상호보완적이다.

##### Example

정적인 것은 동적인 것을 돕는다: 동적으로 다형적인 타입 인터페이스를 구현하기 위해 정적 다형성을 사용하라.

```c++
    class Command {
        // pure virtual functions
    };

    // implementations
    template</*...*/>
    class ConcreteCommand : public Command {
        // implement virtuals
    };
```

##### Example

동적인 것은 정적인 것을 돕는다: 일반적이고, 편리하며, 정적으로 결합된(bound) 인터페이스를 제공하라. 내부적으로는 (dispatch dynmically)동적으로 구현하라. 그렇게 함으로써 일관적인 개체를 만들어라(offer a uniform object layout).  

Examples include type erasure as with `std::shared_ptr`'s deleter (but [don't overuse type erasure](#Rt-erasure)).

##### Note

클래스 템플릿에 있는 비상속함수(nonvirtual function)는 사용되지 않으면 생성되지(instantiated) 않는다. -- 가상함수는 언제나 인스턴스화된다.
이는 코드크기를 늘리고 필요치도 않는 함수를 인스턴스화함으로써 일반화 타입에 대한 제약을 심화시킬지도 모른다.

##### See also

* ref ???
* ref ???
* ref ???

##### Enforcement

See the reference to more specific rules.

## <a name="SS-concepts"></a>T.concepts: Concept rules

컨셉(Concept)은 템플릿 인자들에 대한 요구사항을 지정하기 위한 기능이다.
컨셉은 [ISO technical specification](#Ref-conceptsTS)이지만, 현재는 GCC만 이를 지원하고 있다.  
컨셉은 제네릭 프로그래밍에서의 사고에 중요한 역할을 하며, 미래의 (표준을 비롯한) C++ 라이브러리들의 많은 작업의 기초가 될 것이다.

이후의 규칙들은 컨셉이 지원된다고 가정한다

컨셉 사용 규칙 요약:

* [T.10: Specify concepts for all template arguments](#Rt-concepts)
* [T.11: Whenever possible use standard concepts](#Rt-std-concepts)
* [T.12: Prefer concept names over `auto`](#Rt-auto)
* [T.13: Prefer the shorthand notation for simple, single-type argument concepts](#Rt-shorthand)
* ???

컨셉 정의 규칙 요약:

* [T.20: Avoid "concepts" without meaningful semantics](#Rt-low)
* [T.21: Require a complete set of operations for a concept](#Rt-complete)
* [T.22: Specify axioms for concepts](#Rt-axiom)
* [T.23: Differentiate a refined concept from its more general case by adding new use patterns](#Rt-refine)
* [T.24: Use tag classes or traits to differentiate concepts that differ only in semantics](#Rt-tag)
* [T.25: Avoid complimentary constraints](#Rt-not)
* [T.26: Prefer to define concepts in terms of use-patterns rather than simple syntax](#Rt-use)
* ???

## <a name="SS-concept-use"></a>T.con-use: 컨셉 사용(Concept use)

### <a name="Rt-concepts"></a>T.10: Specify concepts for all template arguments

##### Reason

정확함과 가독성.
템플릿 인자의 가정된 의미(문법과 의미구조)는 템플릿 인터페이스의 기본이다.
컨셉은 문서화와 템플릿용 오류 처리를 경이적으로 개선시킨다.
템플릿 인자를 위한 개념을 기술하는 것은 강력한 디자인 도구가 된다.

##### Example

```c++
    template<typename Iter, typename Val>
    //    requires Input_iterator<Iter>
    //             && Equality_comparable<Value_type<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }
```

위와 같은 의미를 가지지만, 좀 더 간결하게 하자면:

```c++
    template<Input_iterator Iter, typename Val>
    //    requires Equality_comparable<Value_type<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }
```

##### Note

"Concepts" are defined in an ISO Technical specification: [concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
A draft of a set of standard-library concepts can be found in another ISO TS: [ranges](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf)
Concepts are supported in GCC 6.1 and later.
Consequently, we comment out uses of concepts in examples; that is, we use them as formalized comments only.
If you use GCC 6.1 or later, you can uncomment them:

```c++
    template<typename Iter, typename Val>
        requires Input_iterator<Iter>
               && Equality_comparable<Value_type<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }
```

##### Note

`typename`(또는 `auto`)는 가장 제약이 작은 컨셉이다.
"이 인자는 임의의 타입이다"인 경우를 제외하곤 가능한 적게 사용되어야 한다.

템플릿 메타프로그래밍 코드의 한 부분으로써 우리가 표현식 트리를 조작하고, 타입 검사를 연기할때만 필요하다.

**References**: TC++PL4, Palo Alto TR, Sutton

##### Enforcement

컨셉을 사용하지 않은 템플릿 타입 인자가 있다면 지적한다

### <a name="Rt-std-concepts"></a>T.11: Whenever possible use standard concepts

##### Reason

"Standard" concepts (as provided by the [GSL](#S-GSL) and the [Ranges TS](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf), and hopefully soon the ISO standard itself)
saves us the work of thinking up our own concepts, are better thought out than we can manage to do in a hurry, and improves interoperability.

##### Note

새로운 일반화 라이브러리를 만들지 않는한 필요한 많은 컨셉은 이미 표준 라이브러리 안에 정의되어 있다.

##### Example (using TS concepts)

```c++
    template<typename T>
        // don't define this: Sortable is in the GSL
    concept Ordered_container = Sequence<T> && Random_access<Iterator<T>> && Ordered<Value_type<T>>;

    void sort(Ordered_container& s);
```

이 `Ordered_container`는 아주 타당해 보인다. 그러나 GSL(그리고 Range TS)안에 있는 `Sortable` 컨셉과 아주 비슷하다.
더 좋은가? 더 올바른가? `sort`에 대한 표준 요구사항을 정확하게 반영하고 있는가?
`Sortable`을 사용하는 것이 더 좋고 단순하다:

```c++
    void sort(Sortable& s);   // better
```

##### Note

"표준" 컨셉들은 ISO 표준화 과정에서 발전하고 있다.

##### Note

쓸만한 컨셉을 디자인하는 것은 굉장히 어려운(challenging) 일이다.

##### Enforcement

어렵다.

* 제약조건이 없는 인자, 비표준 컨셉을 쓰는 템플릿, 공리(axiom)없이 'homebrew' 컨셉을 쓰는 템플릿을 찾는다
* 컨셉 발견 툴을 개발하라. [초기 실험](http://www.stroustrup.com/sle2010_webversion.pdf) 참조

### <a name="Rt-auto"></a>T.12: Prefer concept names over `auto` for local variables

##### Reason

`auto`는 가장 약한 컨셉이다. 컨셉 이름은 단순히 `auto`만 사용하는 것 보다 더 많은 의미을 전달한다.

##### Example (using TS concepts)

```c++
    vector<string> v{ "abc", "xyz" };
    auto& x = v.front();     // bad
    String& s = v.front();   // good (String is a GSL concept)
```

##### Enforcement

* ???

### <a name="Rt-shorthand"></a>T.13: Prefer the shorthand notation for simple, single-type argument concepts

##### Reason

가독성. 직접적인 아이디어의 표현.

##### Example (using TS concepts)

`T`는 `Sortable` 컨셉을 만족하기 위해서는:

```c++
    template<typename T>       // Correct but verbose: "The parameter is
    //    requires Sortable<T>   // of type T which is the name of a type
    void sort(T&);             // that is Sortable"

    template<Sortable T>       // Better (assuming support for concepts): "The parameter is of type T
    void sort(T&);             // which is Sortable"

    void sort(Sortable&);      // Best (assuming support for concepts): "The parameter is Sortable"
```

짧은 버전이 우리가 말하는 방법과 더 잘 일치한다. 많은 템플릿은 `template` 키워드를 사용할 필요가 없다는 점에 주목하라.

##### Note

"Concepts" are defined in an ISO Technical specification: [concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
A draft of a set of standard-library concepts can be found in another ISO TS: [ranges](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf)
Concepts are supported in GCC 6.1 and later.
Consequently, we comment out uses of concepts in examples; that is, we use them as formalized comments only.
If you use a compiler that supports concepts (e.g., GCC 6.1 or later), you can remove the `//`.

##### Enforcement

* `<typename T>`과 `<class T>` 표기법을 변경할 때 짧은 단어로는 쓰기가 불가능하다
* 처음에는 `typename`을 사용하고, 그 후에 한 종류 인자 컨셉으로 제한되는 선언이 있다면 지적한다

## <a name="SS-concepts-def"></a>T.concepts.def: 컨셉 정의(Concept definition rules) 규칙

Defining good concepts is non-trivial.
Concepts are meant to represent fundamental concepts in an application domain (hence the name "concepts").
Similarly throwing together a set of syntactic constraints to be used for a the arguments for a single class or algorithm is not what concepts were designed for
and will not give the full benefits of the mechanism.

Obviously, defining concepts will be most useful for code that can use an implementation (e.g., GCC 6.1 or later),
but defining concepts is in itself a useful design technique and help catch conceptual errors and clean up the concepts (sic!) of an implementation.

### <a name="Rt-low"></a>T.20: Avoid "concepts" without meaningful semantics

##### Reason

컨셉은 의미적인 개념, 예를 들면, "숫자", 요소들의 "범위", 그리고 "완전히 정렬된" 같은 개념을 표현하기 위한 것이다.
단순한 제약조건, `+`연산자를 가진다거나, `>`연산자를 가지는 것, 처럼 독자적으로 기술되어선 의미가 없다.
유저 코드보다는 의미있는 개념을 위한 블록을 구성하는데 사용되어야 한다.

##### Example, bad (using TS concepts)

```c++
    template<typename T>
    concept Addable = has_plus<T>;    // bad; insufficient

    template<Addable N> auto algo(const N& a, const N& b) // use two numbers
    {
        // ...
        return a + b;
    }

    int x = 7;
    int y = 9;
    auto z = algo(x, y);   // z = 16

    string xx = "7";
    string yy = "9";
    auto zz = algo(xx, yy);   // zz = "79"
```

아마도 문자열 접합일 수도 있고, 실수였을 수도 있다. 여기서 뺄샘 연산을 지원하도록 한다면 완전히 다른 타입들로만 사용가능할 것이다.
이 `Addable`은 교환법칙이 성립해야 한다는 수학적인 규칙에 위배된다: `a+b == b+a`

##### Note

컨셉의 진정한 특징은 문법 제약과 달리 의미구조를 기술하는 능력이 있다는 것이다.

##### Example (using TS concepts)

```c++
    template<typename T>
    // The operators +, -, *, and / for a number are assumed to follow the usual mathematical rules
    concept Number = has_plus<T>
                     && has_minus<T>
                     && has_multiply<T>
                     && has_divide<T>;

    template<Number N> auto algo(const N& a, const N& b)
    {
        // ...
        return a + b;
    }

    int x = 7;
    int y = 9;
    auto z = algo(x, y);   // z = 16

    string xx = "7";
    string yy = "9";
    auto zz = algo(xx, yy);   // error: string is not a Number
```

##### Note

Concepts with multiple operations have far lower chance of accidentally matching a type than a single-operation concept.

##### Enforcement

* Flag single-operation `concepts` when used outside the definition of other `concepts`.
* Flag uses of `enable_if` that appears to simulate single-operation `concepts`.

### <a name="Rt-complete"></a>T.21: Require a complete set of operations for a concept

##### Reason

이해하기 쉽다. 상호운용성 개선. 구현이나 유지보수 인력에게 도움이 된다.

##### Note

This is a specific variant of the general rule that [a concept must make semantic sense](#Rt-low).

##### Example, bad (using TS concepts)

```c++
    template<typename T> concept Subtractable = requires(T a, T, b) { a-b; };
```

문맥적으로 무의미 하다. 
최소한 `+`, `-` 정도는 있어야 쓸만하다.

완전한 연산 집합의 예를 들자면 다음과 같다

* `Arithmetic`: `+`, `-`, `*`, `/`, `+=`, `-=`, `*=`, `/=`
* `Comparable`: `<`, `>`, `<=`, `>=`, `==`, `!=`

##### Note

This rule applies whether we use direct language support for concepts or not.
It is a general design rule that even applies to non-templates:

```c++
    class Minimal {
        // ...
    };

    bool operator==(const Minimal&, const Minimal&);
    bool operator<(const Minimal&, const Minimal&);

    Minimal operator+(const Minimal&, const Minimal&);
    // no other operators

    void f(const Minimal& x, const Minimal& y)
    {
        if (!(x == y)) { /* ... */ }    // OK
        if (x != y) { /* ... */ }       // surprise! error

        while (!(x < y)) { /* ... */ }  // OK
        while (x >= y) { /* ... */ }    // surprise! error

        x = x + y;          // OK
        x += y;             // surprise! error
    }
```

This is minimal, but surprising and constraining for users.
It could even be less efficient.

The rule supports the view that a concept should reflect a (mathematically) coherent set of operations.

##### Example

```c++
    class Convenient {
        // ...
    };

    bool operator==(const Convenient&, const Convenient&);
    bool operator<(const Convenient&, const Convenient&);
    // ... and the other comparison operators ...

    Minimal operator+(const Convenient&, const Convenient&);
    // .. and the other arithmetic operators ...

    void f(const Convenient& x, const Convenient& y)
    {
        if (!(x == y)) { /* ... */ }    // OK
        if (x != y) { /* ... */ }       // OK

        while (!(x < y)) { /* ... */ }  // OK
        while (x >= y) { /* ... */ }    // OK

        x = x + y;     // OK
        x += y;        // OK
    }
```

It can be a nuisance to define all operators, but not hard.
Ideally, that rule should be language supported by giving you comparison operators by default.

##### Enforcement

* Flag classes that support "odd" subsets of a set of operators, e.g., `==` but not `!=` or `+` but not `-`.
  Yes, `std::string` is "odd", but it's too late to change that.

### <a name="Rt-axiom"></a>T.22: Specify axioms for concepts

##### Reason

의미있고 유용한 컨셉은 의미구조에 영향을 준다.
비형식적으로든 형식적으로든 의미구조를 표현하는 것은 컨셉을 이해할 수 있게 만든다. 동시에 개념적인 오류를 잡아내도록 한다.

의미구조를 기술할 수 있다는 것은 강력한 디자인 도구이다.

##### Example (using TS concepts)

```c++
    template<typename T>
        // The operators +, -, *, and / for a number are assumed to follow the usual mathematical rules
        // axiom(T a, T b) { a + b == b + a; a - a == 0; a * (b + c) == a * b + a * c; /*...*/ }
        concept Number = requires(T a, T b) {
            {a + b} -> T;   // the result of a + b is convertible to T
            {a - b} -> T;
            {a * b} -> T;
            {a / b} -> T;
        }
```

##### Note

이것은 수학적 공리(axiom: 증명이 없이도 자명한 것)를 표현한 것이다.
일반적으로 공리는 증명하지 않는다. 경우에 따라 그 증명은 컴파일러의 능력을 넘어서는 것이다.
공리가 일반적이지 않을지도 모른다. 그러나 템플릿 작성자는 실제로 사용하는 모든 입력에 대해서 공리를 가지고 있다고 가정하는게 좋다 (선행조건과 비슷하다).

##### Note

이 문맥에서 공리는 불리언 연산식이다. 그 예로 [Palo Alto TR](#S-references)를 참조하라.
현재 C++은 공리를 지원하지 않는다. (ISO 컨셉 TS에서도) 그래서 꽤 오래동안 주석으로 대신해야만 한다.
나중에 언어가 지원한다면 '//'를 없애면 된다.

##### Note

GSL 컨셉은 잘 정의된 의미구조를 가지고 있다; Palo Alto TR과 Ranges TS를 참조하라.

##### Exception (using TS concepts)

현재 개발중인 새 "컨셉" 초기버전은 의미구조를 기술하지 않고 제약조건들을 정의하려고 한다.
좋은 의미구조는 노력과 시간이 필요하다.
불완전한 제약조건이라도 유용할 수 있다:

```c++
    // balancer for a generic binary tree
    template<typename Node> concept bool Balancer = requires(Node* p) {
        add_fixup(p);
        touch(p);
        detach(p);
    }
```

So a `Balancer` must supply at least thee operations on a tree `Node`,
but we are not yet ready to specify detailed semantics because a new kind of balanced tree might require more operations
and the precise general semantics for all nodes is hard to pin down in the early stages of design.

의미구조가 잘 정의되지 않은 "개념"이라도 유용할 수 있다.
For example, it allows for some checking during initial experimentation.
그러나 안정화됐다고 생각해서는 안된다. 새로운 용례가 발견되면 불완전한 컨셉은 개선될 것이다.

##### Enforcement

* 컨셉 정의 주석 내에 "axiom" 단어를 찾는다

### <a name="Rt-refine"></a>T.23: Differentiate a refined concept from its more general case by adding new use patterns

##### Reason

그렇지 않으면 컴파일러가 자동으로 구분할 수 없다.

##### Example (using TS concepts)

```c++
    template<typename I>
    concept bool Input_iter = requires(I iter) { ++iter; };

    template<typename I>
    concept bool Fwd_iter = Input_iter<I> && requires(I iter) { iter++; }
```

컴파일러는 요구된 연산들에 기반해서 어떤것이 필요한 연산인지 결정(determine refinement)할 수 있다. (예제에서는 `++`에 해당한다)
This decreases the burden on implementers of these types since
they do not need any special declarations to "hook into the concept".
2개의 컨셉이 요구사항이 정확하게 동일하다면 그들은 논리적 동치라고 할 수 있다 (there is no refinement).

##### Enforcement

* 다른 컨셉과 요구사항이 정확하게 일치하는 컨셉이 있다면 지적한다. 차이를 분명하게 하고 싶다면 [T.24](#Rt-tag)를 참조하라.

### <a name="Rt-tag"></a>T.24: Use tag classes or traits to differentiate concepts that differ only in semantics

##### Reason

프로그래머가 차별화하지 않는다면, 동일한 문법을 요구하지만 의미구조가 다른 두 컨셉은 모호함을 낳는다.

##### Example (using TS concepts)

```c++
    template<typename I>    // iterator providing random access
    concept bool RA_iter = ...;

    template<typename I>    // iterator providing random access to contiguous data
    concept bool Contiguous_iter =
        RA_iter<I> && is_contiguous<I>::value;  // using is_contiguous trait
```

라이브러리 프로그래머가 `is_contiguous`를 적절하게 정의해야 한다.

Wrapping a tag class into a concept leads to a simpler expression of this idea:

```c++
    template<typename I> concept Contiguous = is_contiguous<I>::value;

    template<typename I>
    concept bool Contiguous_iter = RA_iter<I> && Contiguous<I>;
```

The programmer (in a library) must define `is_contiguous` (a trait) appropriately.

##### Note

특성(trait)은 특성 클래스(trait class)나 타입 특성(type trait)을 말한다. 사용자 혹은 표준 라이브러리에서 정의했을 수 있다..
가능하다면 표준에서 정의한 특성을 선호하라.

##### Enforcement

* 컴파일러는 동일한 컨셉을 애매하게 사용하는 것을 지적한다
* 동일한 컨셉을 정의한다면 지적한다

### <a name="Rt-not"></a>T.25: Avoid complementary constraints

##### Reason

단순명료. 유지보수가 좋음.
Functions with complementary requirements expressed using negation are brittle.

##### Example (using TS concepts)

처음에는, 사람들이 보완적인 요구사항을 가진 함수를 정의하려고 할 것이다:

```c++
    template<typename T>
        requires !C<T>    // bad
    void f();

    template<typename T>
        requires C<T>
    void f();
```

아래 코드가 더 낫다:

```c++
    template<typename T>   // general template
        void f();

    template<typename T>   // specialization by concept
        requires C<T>
    void f();
```

`C<T>`가 만족되지 않을 때만 컴파일러는 제한조건이 없는 템플릿을 선택할 것이다.
제약조건이 없는 `f()`를 정의하고 싶지 않다면 그냥 없애라.

```c++
    template<typename T>
    void f() = delete;
```

컴파일러는 오버로드된 함수를 선택할 것이고 적당한 에러를 낼 것이다.

##### Note

Complementary constraints are unfortunately common in `enable_if` code:

```c++
    template<typename T>
    enable_if<!C<T>, void>   // bad
    f();

    template<typename T>
    enable_if<C<T>, void>
    f();
```

##### Note

Complementary requirements on one requirements is sometimes (wrongly) considered manageable.
However, for two or more requirements the number of definitions needs can go up exponentially (2,4,9,16,...):

```
    C1<T> && C2<T>
    !C1<T> && C2<T>
    C1<T> && !C2<T>
    !C1<T> && !C2<T>
```

Now the opportunities for errors multiply.

##### Enforcement

* `C<T>`, `!C<T>`를 같이 가진 함수들이 있다면 지적한다
* 정반대 제약조건이 있다면 모두 지적한다

### <a name="Rt-use"></a>T.26: Prefer to define concepts in terms of use-patterns rather than simple syntax

##### Reason

정의가 더 읽기 쉽고 사용자가 작성하고 싶은 것과 직접적으로 일치한다.
형변환도 고려해야 한다. 모든 타입특성의 이름을 기억할 필요가 없다.

##### Example (using TS concepts)

You might be tempted to define a concept `Equality` like this:

```c++
    template<typename T> concept Equality = has_equal<T> && has_not_equal<T>;
```

Obviously, it would be better and easier just to use the standard `EqualityComparable`,
but - just as an example - if you had to define such a concept, prefer:

```c++
    template<typename T> concept Equality = requires(T a, T b) {
        bool == { a == b }
        bool == { a != b }
        // axiom { !(a == b) == (a != b) }
        // axiom { a = b; => a == b }  // => means "implies"
    }
```

as opposed to defining two meaningless concepts `has_equal` and `has_not_equal` just as helpers in the definition of `Equality`.
By "meaningless" we mean that we cannot specify the semantics of `has_equal` in isolation.

##### Enforcement

???

## <a name="SS-temp-interface"></a> 템플릿 인터페이스(Template interfaces)

Over the years, programming with templates have suffered from a weak distinction between the interface of a template
and its implementation.
Before concepts, that distinction had no direct language support.
However, the interface to a template is a critical concept - a contract between a user and an implementer - and should be carefully designed.

### <a name="Rt-fo"></a>T.40: Use function objects to pass operations to algorithms

##### Reason

함수객체는 함수에 대한 "단순" 포인터에 비해 인터페이스를 통해 많은 정보를 전달할 수 있다.
보통은 함수개체를 전달하는 것이 함수포인터에 비해 더 나은 성능을 보인다.

##### Example (using TS concepts)

```c++
    bool greater(double x, double y) { return x > y; }
    sort(v, greater);                                    // pointer to function: potentially slow
    sort(v, [](double x, double y) { return x > y; });   // function object
    sort(v, std::greater<>);                             // function object

    bool greater_than_7(double x) { return x > 7; }
    auto x = find_if(v, greater_than_7);                 // pointer to function: inflexible
    auto y = find_if(v, [](double x) { return x > 7; }); // function object: carries the needed data
    auto z = find_if(v, Greater_than<double>(7));        // function object: carries the needed data
```

You can, of course, generalize those functions using `auto` or (when and where available) concepts. For example:

```c++
    auto y1 = find_if(v, [](Ordered x) { return x > 7; }); // require an ordered type
    auto z1 = find_if(v, [](auto x) { return x > 7; });    // hope that the type has a >
```

##### Note

람다 표현식은 함수개체를 생성한다.

##### Note

성능문제는 컴파일러, 최적화 기술에 달려있다.

##### Enforcement

* 함수 템플릿 인자에 포인터가 있다면 지적한다
* 템플릿에 함수 포인터가 인자로 전달된다면 지적한다 (false positives의 위험이 있다)

### <a name="Rt-essential"></a>T.41: Require only essential properties in a template's concepts

##### Reason

템플릿이 쉽고 안정적인 상태로 유지된다.

##### Example (using TS concepts)

Consider, a `sort` instrumented with (oversimplified) simple debug support:

```c++
    void sort(Sortable& s)  // sort sequence s
    {
        if (debug) cerr << "enter sort( " << s <<  ")\n";
        // ...
        if (debug) cerr << "exit sort( " << s <<  ")\n";
    }
```

Should this be rewritten to:

```c++
    template<Sortable S>
        requires Streamable<S>
    void sort(S& s)  // sort sequence s
    {
        if (debug) cerr << "enter sort( " << s <<  ")\n";
        // ...
        if (debug) cerr << "exit sort( " << s <<  ")\n";
    }
```

After all, there is nothing in `Sortable` that requires `iostream` support.
On the other hand, there is nothing in the fundamental idea of sorting that says anything about debugging.

##### Note

If we require every operation used to be listed among the requirements, the interface becomes unstable:
Every time we change the debug facilities, the usage data gathering, testing support, error reporting, etc.
The definition of the template would need change and every use of the template would have to be recompiled.
This is cumbersome, and in some environments infeasible.

Conversely, if we use an operation in the implementation that is not guaranteed by concept checking,
we may get a late compile-time error.

By not using concept checking for properties of a template argument that is not considered essential,
we delay checking until instantiation time.
We consider this a worthwhile tradeoff.

Note that using non-local, non-dependent names (such as `debug` and `cerr`) also introduces context dependencies that may lead to "mysterious" errors.

##### Note

It can be hard to decide which properties of a type is essential and which are not.

##### Enforcement

???

### <a name="Rt-alias"></a>T.42: Use template aliases to simplify notation and hide implementation details

##### Reason

가독성을 좋게 하며, 구현 내용을 숨긴다.
템플릿 별칭은 타입을 계산하기 위한 많은 특성(traits)을 사용하는 것을 대체해준다.
타입 특성을 숨기기 위해 쓰일 수도 있다.

##### Example

```c++
    template<typename T, size_t N>
    class Matrix {
        // ...
        using Iterator = typename std::vector<T>::iterator;
        // ...
    };
```

`Matrix` 사용자들이 그 요소가 `vector`에 저장되는 점을 알 필요가 없게 한다. 반복적으로 `typename std::vector<T>::`를 타이핑하는 것을 줄여준다.

##### Example

```c++
    template<typename T>
    void user(T& c)
    {
        // ...
        typename container_traits<T>::value_type x; // bad, verbose
        // ...
    }

    template<typename T>
    using Value_type = typename container_traits<T>::value_type;
```

이것은 `value_type`을 쓰는 사용자가 `value_type`의 구현을 알 필요가 없게 한다.

```c++
    template<typename T>
    void user2(T& c)
    {
        // ...
        Value_type<T> x;
        // ...
    }
```

##### Note

A simple, common use could be expressed: "Wrap traits!"

##### Enforcement

* `using`으로 선언이외에 중의성 제거용으로 `typename`을 사용한다면 지적한다
* ???

### <a name="Rt-using"></a>T.43: Prefer `using` over `typedef` for defining aliases

##### Reason

가독성: `using`을 사용하면 새 명칭은 선언 시에 뒤쪽 어딘가에 숨어 있기보다는 제일 앞에 나타난다.  
일반성(Generality): `using`은 템플릿 별칭으로 사용할 수 있다. 반면 `typedef`는 템플릿에는 쓰기 어렵다.  
일률성(Uniformity): `using`은 구문상 `auto`와 비슷하다.  

##### Example

```c++
    typedef int (*PFI)(int);   // OK, but convoluted

    using PFI2 = int (*)(int);   // OK, preferred

    template<typename T>
    typedef int (*PFT)(T);      // error

    template<typename T>
    using PFT2 = int (*)(T);   // OK
```

##### Enforcement

* `typedef`를 사용하고 있다면 지적하라. "아주 많이" 나올 것이다. :-(

### <a name="Rt-deduce"></a>T.44: Use function templates to deduce class template argument types (where feasible)

##### Reason

템플릿 인자 타입을 명시적으로 쓴다면 불필요하게 길어질 수 있다.

##### Example

```c++
    tuple<int, string, double> t1 = {1, "Hamlet", 3.14};   // explicit type
    auto t2 = make_tuple(1, "Ophelia"s, 3.14);         // better; deduced type
```

문자열이 C 스타일이 아니라 `std::string`이라는 것을 보장하기 위해 `s`를 뒤쪽에 붙인 것에 주목하라.

##### Note

당신이 간단한 `make_T` 함수를 작성하듯이 컴파일러도 그렇게 할 수 있다. 아마 미래에는 `make_T` 함수가 불필요해질 것이다.

##### Exception

템플릿 인자를 추정할 좋은 방법이 없어서 인자를 명시적으로 기술할 수도 있다:

```c++
    vector<double> v = { 1, 2, 3, 7.9, 15.99 };
    list<Record*> lst;
```

##### Note

Note that C++17 will make this rule redundant by allowing the template arguments to be deduced directly from constructor arguments:
[Template parameter deduction for constructors (Rev. 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0091r1.html).

For example:

```c++
    tuple t1 = {1, "Hamlet"s, 3.14}; // deduced: tuple<int, string, double>
```

##### Enforcement

명시적으로 특수화된 타입이 사용된 인자들의 타입과 정확히 일치하는 부분을 지적하라.

### <a name="Rt-regular"></a>T.46: Require template arguments to be at least `Regular` or `SemiRegular`

##### Reason

Readability.
Preventing surprises and errors.
Most uses support that anyway.

##### Example

```c++
    class X {
            // ...
    public:
        explicit X(int);
        X(const X&);            // copy
        X operator=(const X&);
        X(X&&) noexcept;                 // move
        X& operator=(X&&) noexcept;
        ~X();
        // ... no more constructors ...
    };

    X x {1};    // fine
    X y = x;      // fine
    std::vector<X> v(10); // error: no default constructor
```

##### Note

Semiregular requires default constructible.

##### Enforcement

* Flag types that are not at least `SemiRegular`.

### <a name="Rt-visible"></a>T.47: Avoid highly visible unconstrained templates with common names

##### Reason

An unconstrained template argument is a perfect match for anything so such a template can be preferred over more specific types that require minor conversions.
This is particularly annoying/dangerous when ADL is used.
 Common names make this problem more likely.

##### Example

```c++
    namespace Bad {
        struct S { int m; };
        template<typename T1, typename T2>
        bool operator==(T1, T2) { cout << "Bad\n"; return true; }
    }

    namespace T0 {
        bool operator==(int, Bad::S) { cout << "T0\n"; return true; }  // compare to int

        void test()
        {
            Bad::S bad{ 1 };
            vector<int> v(10);
            bool b = 1 == bad;
            bool b2 = v.size() == bad;
        }
    }
```

This prints `T0` and `Bad`.

Now the `==` in `Bad` was designed to cause trouble, but would you have spotted the problem in real code?
The problem is that `v.size()` returns an `unsigned` integer so that a conversion is needed to call the local `==`;
the `==` in `Bad` requires no conversions.
Realistic types, such as the standard-library iterators can be made to exhibit similar anti-social tendencies.

##### Note

If an unconstrained template is defined in the same namespace as a type,
that unconstrained template can be found by ADL (as happened in the example).
That is, it is highly visible.

##### Note

This rule should not be necessary, but the committee cannot agree to exclude unconstrained templated from ADL.

Unfortunately this will get many false positives; the standard library violates this widely, by putting many unconstrained templates and types into the single namespace `std`.

##### Enforcement

Flag templates defined in a namespace where concrete types are also defined (maybe not feasible until we have concepts).

### <a name="Rt-concept-def"></a>T.48: If your compiler does not support concepts, fake them with `enable_if`

##### Reason

Because that's the best we can do without direct concept support.
`enable_if` can be used to conditionally define functions and to select among a set of functions.

##### Example

    enable_if<???>

##### Note

Beware of [complementary constraints](# T.25).
Faking concept overloading using `enable_if` sometimes forces us to use that error-prone design technique.

##### Enforcement

???

### <a name="Rt-erasure"></a>T.49: Where possible, avoid type-erasure

> 역주 : 타입 제거(type-erasure)

##### Reason

타입제거는 서로 다른 컴파일 범위로 전달되는 타입 정보가 없어지므로 특별한 간접효과가 발생한다.

##### Example

    ???

**Exceptions**: `std::function`와 같은 경우처럼 때로는 타입제거가 적절할 수 있다.

##### Enforcement

???

##### Note

???

## <a name="SS-temp-def"></a>T.def: 템플릿 정의(Template definitions)

A template definition (class or function) can contain arbitrary code, so only a comprehensive review of C++ programming techniques would cover this topic.
However, this section focuses on what is specific to template implementation.
In particular, it focuses on a template definition's dependence on its context.

### <a name="Rt-depend"></a>T.60: Minimize a template's context dependencies

##### Reason

이해가 쉽다.
예상치 못한 의존성 오류 발생을 최소화.
툴 작성을 쉽게 한다.

##### Example

```c++
    template<typename C>
    void sort(C& c)
    {
        std::sort(begin(c), end(c)); // necessary and useful dependency
    }

    template<typename Iter>
    Iter algo(Iter first, Iter last) {
        for (; first != last; ++first) {
            auto x = sqrt(*first); // potentially surprising dependency: which sqrt()?
            helper(first, x);      // potentially surprising dependency:
                                   // helper is chosen based on first and x
            TT var = 7;            // potentially surprising dependency: which TT?
        }
    }
```

##### Note

Templates typically appear in header files so their context dependencies are more vulnerable to `#include` order dependencies than functions in `.cpp` files.

##### Note

인자에만 템플릿을 동작하게 하는 것이 의존도를 최소한으로 줄일 수 있는 한가지 방법인데 일반적으로는 다루기 힘들다.
For example, an algorithm usually uses other algorithms and invoke operations that does not exclusively operate on arguments.
And don't get us started on macros!

**See also**: [T.69](#Rt-customization)

##### Enforcement

??? 까다롭다(Tricky)

### <a name="Rt-scary"></a>T.61: Do not over-parameterize members (SCARY)

##### Reason

템플릿 매개변수로 쓰이지 않는 멤버는 구체적인 템플릿 매개변수를 제외하고 사용될 수 없다.
이는 템플릿 사용을 제한하고 보통 코드 사이즈를 증가시킨다.

##### Example, bad

```c++
    template<typename T, typename A = std::allocator{}>
        // requires Regular<T> && Allocator<A>
    class List {
    public:
        struct Link {   // does not depend on A
            T elem;
            T* pre;
            T* suc;
        };

        using iterator = Link*;

        iterator first() const { return head; }

        // ...
    private:
        Link* head;
    };

    List<int> lst1;
    List<int, My_allocator> lst2;
```

아무 문제 없어 보인다. but now `Link` formally depends on the allocator (even though it doesn't use the allocator). This forces redundant instantiations that can be surprisingly costly in some real-world scenarios.
Typically, the solution is to make what would have been a nested class non-local, with its own minimal set of template parameters.

```c++
    template<typename T>
    struct Link {
        T elem;
        T* pre;
        T* suc;
    };

    template<typename T, typename A = std::allocator{}>
        // requires Regular<T> && Allocator<A>
    class List2 {
    public:
        using iterator = Link<T>*;

        iterator first() const { return head; }

        // ...
    private:
        Link* head;
    };

    List<int> lst1;
    List<int, My_allocator> lst2;
```

Some people found the idea that the `Link` no longer was hidden inside the list scary, so we named the technique
[SCARY](http://www.open-std.org/jtc1/sc22/WG21/docs/papers/2009/n2911.pdf).From that academic paper: 
"The acronym SCARY describes assignments and initializations that are Seemingly erroneous (appearing Constrained by conflicting generic parameters), but Actually work with the Right implementation (unconstrained bY the conflict due to minimized dependencies."

##### Enforcement

* 멤버 타입이 의존하지 않는 템플릿 매개변수가 있다면 지적한다
* 멤버 함수가 의존하지 않는 템플릿 매개변수가 있다면 지적한다

### <a name="Rt-nondependent"></a>T.62: Place non-dependent class template members in a non-templated base class

##### Reason

 Allow the base class members to be used without specifying template arguments and without template instantiation.

##### Example

```c++
    template<typename T>
    class Foo {
    public:
        enum { v1, v2 };
        // ...
    };
```

???

```c++
    struct Foo_base {
        enum { v1, v2 };
        // ...
    };

    template<typename T>
    class Foo : public Foo_base {
    public:
        // ...
    };
```

##### Note

A more general version of this rule would be
"If a template class member depends on only N template parameters out of M, place it in a base class with only N parameters."
For N == 1, we have a choice of a base class of a class in the surrounding scope as in [T.61](#Rt-scary).

??? What about constants? class statics?

##### Enforcement

* Flag ???

### <a name="Rt-specialization"></a>T.64: Use specialization to provide alternative implementations of class templates

##### Reason

템플릿은 일반적인 인터페이스를 정의한 것이다.
특수화는 그 인터페이스의 다른(alternative) 구현을 위한 강력한 메커니즘을 제공한다.

##### Example

    ??? string specialization (==)

    ??? representation specialization ?

##### Note

???

##### Enforcement

???

### <a name="Rt-tag-dispatch"></a>T.65: Use tag dispatch to provide alternative implementations of a function

##### Reason

* A template defines a general interface.
* Tag dispatch allows us to select implementations based on specific properties of an argument type.
* Performance.

##### Example

`std::copy`를 단순화 하면 다음과 같다 (메모리 상에서 인접하지 않을 가능성을 배제하였다).

```c++
    struct pod_tag {};
    struct non_pod_tag {};

    template<class T> struct copy_trait { using tag = non_pod_tag; };   // T is not "plain old data"

    template<> struct copy_trait<int> { using tag = pod_tag; };         // int is "plain old data"

    template<class Iter>
    Out copy_helper(Iter first, Iter last, Iter out, pod_tag)
    {
        // use memmove
    }

    template<class Iter>
    Out copy_helper(Iter first, Iter last, Iter out, non_pod_tag)
    {
        // use loop calling copy constructors
    }

    template<class Itert>
    Out copy(Iter first, Iter last, Iter out)
    {
        return copy_helper(first, last, out, typename copy_trait<Iter>::tag{})
    }

    void use(vector<int>& vi, vector<int>& vi2, vector<string>& vs, vector<string>& vs2)
    {
        copy(vi.begin(), vi.end(), vi2.begin()); // uses memmove
        copy(vs.begin(), vs.end(), vs2.begin()); // uses a loop calling copy constructors
    }
```

위 코드는 컴파일 시간에 알고리즘을 선택하는 일반적이고 강력한 방법을 보여준다.

##### Note

`concept`이 가능하게 된다면 이런 대안은 바로 구별될 수 있을 것이다:

```c++
    template<class Iter>
        requires Pod<Value_type<iter>>
    Out copy_helper(In, first, In last, Out out)
    {
        // use memmove
    }

    template<class Iter>
    Out copy_helper(In, first, In last, Out out)
    {
        // use loop calling copy constructors
    }
```

##### Enforcement

???

### <a name="Rt-specialization2"></a>T.67: Use specialization to provide alternative implementations for irregular types

##### Reason

 ???

##### Example

???

##### Enforcement

???

### <a name="Rt-cast"></a>T.68: Use `{}` rather than `()` within templates to avoid ambiguities

##### Reason

 `()` is vulnerable to grammar ambiguities.

##### Example

```c++
    template<typename T, typename U>
    void f(T t, U u)
    {
        T v1(x);    // is v1 a function of a variable?
        T v2 {x};   // variable
        auto x = T(u);  // construction or cast?
    }

    f(1, "asdf"); // bad: cast from const char* to int
```

##### Enforcement

* flag `()` initializers
* flag function-style casts

### <a name="Rt-customization"></a>T.69: Inside a template, don't make an unqualified nonmember function call unless you intend it to be a customization point

##### Reason

* Provide only intended flexibility.
* Avoid vulnerability to accidental environmental changes.

##### Example

There are three major ways to let calling code customize a template.

```c++
    template<class T>
        // Call a member function
    void test1(T t)
    {
        t.f();    // require T to provide f()
    }

    template<class T>
    void test2(T t)
        // Call a nonmember function without qualification
    {
        f(t);  // require f(/*T*/) be available in caller's scope or in T's namespace
    }

    template<class T>
    void test3(T t)
        // Invoke a "trait"
    {
        test_traits<T>::f(t); // require customizing test_traits<>
                              // to get non-default functions/types
    }
```

A trait is usually a type alias to compute a type,
a `constexpr` function to compute a value,
or a traditional traits template to be specialized on the user's type.

##### Note

`t`값으로 템플릿 타입 매개변수로 된 `helper(t)`함수를 호출하려고 한다면, `::detail` 네임스페이스에 함수를 두고 `detail::helper(t);`로 호출하라.
그렇지 않으면 `t` 타입의 네임스페이스에서 찾을 수 있는 다른 `helper` 함수가 호출될 수도 있다. [함수에 제약이 없는 경우 ADL](#Rt-unconstrained-adl)을 참고하라

##### Enforcement

* In a template, flag an unqualified call to a nonmember function that passes a variable of dependent type when there is a nonmember function of the same name in the template's namespace.

## <a name="SS-temp-hier"></a>T.temp-hier: 템플릿과 클래스 계층

C++에서 템플릿은 제네릭 프로그래밍을 지원하기 위한 핵심기능이다. 동시에 클래스 계층은 개체지향 프로그래밍의 근간이라고 할 수 있다.
이 두개 언어 기능은 합해서 효과적으로 사용할 수 있다. 다만 몇가지 디자인 함정은 피해야 한다.

### <a name="Rt-hier"></a>T.80: Do not naively templatize a class hierarchy

##### Reason

함수도 많고 가상함수도 많은 클래스 계층구조를 템플릿화하면 코드가 폭발적으로 증가할 수 있다.

##### Example, bad

```c++
    template<typename T>
    struct Container {         // an interface
        virtual T* get(int i);
        virtual T* first();
        virtual T* next();
        virtual void sort();
    };

    template<typename T>
    class Vector : public Container<T> {
    public:
        // ...
    };

    Vector<int> vi;
    Vector<string> vs;
```

컨테이너의 멤버함수로 `sort`를 정의하는 건 별로 좋은 생각이 아니다.
들어 본 적이 없는건 아니지만 하지 말아야 할 좋은 본보기가 될 것이다.

컴파일러가 코드를 생성해야 하는데 `vector<int>::sort()`가 호출되는지 알 수가 없다. `vector<string>::sort()`에 대해서도 비슷하다.
두 함수가 호출하지 않으면 코드만 커진 꼴이다.
십여개의 멤버 함수와 십여개의 파생클래스를 가진 클래스 계층구조가 다양하게 인스턴스화되면 무엇을 할지 상상해보라.

##### Note

In many cases you can provide a stable interface by not parameterizing a base;
see ["stable base"](#Rt-abi) and [OO and GP](#Rt-generic-oo)

##### Enforcement

* 템플릿 인자에 의존하는 가상함수가 있다면 지적하라

### <a name="Rt-array"></a>T.81: Do not mix hierarchies and arrays

##### Reason

파생 클래스 배열은 기본클래스에 대한 포인터로 "decay"될 수 있는데 처참한(disastrous) 결과로 이어질 수 있다.

##### Example

`Apple`, `Pear`가 `Fruit`의 종류라고 가정하자.

```c++
    void maul(Fruit* p)
    {
        *p = Pear{};     // put a Pear into *p
        p[1] = Pear{};   // put a Pear into p[1]
    }

    Apple aa [] = { an_apple, another_apple };   // aa contains Apples (obviously!)

    maul(aa);
    Apple& a0 = &aa[0];   // a Pear?
    Apple& a1 = &aa[1];   // a Pear?
```

형변환을 사용하진 않았지만, `aa[0]`는 `Pear`일 것이다.
`sizeof(Apple) != sizeof(Pear)`이므로 `aa[1]`에 접근하면 배열에서 오브젝트의 올바른 시작위치로 정렬될 수 없을 것이다.
따라서 타입 위반이 되고, 메모리값이 망가지게 된다.

절대로 이런 코드를 작성하지 마라.

Note that `maul()` violates the a [`T*` points to an individual object rule](#Rf-ptr).

**Alternative**: 적당한 (템플릿) 컨테이너를 사용하라.

```c++
    void maul2(Fruit* p)
    {
        *p = Pear{};   // put a Pear into *p
    }

    vector<Apple> va = { an_apple, another_apple };   // va contains Apples (obviously!)

    maul2(va);       // error: cannot convert a vector<Apple> to a Fruit*
    maul2(&va[0]);   // you asked for it

    Apple& a0 = &va[0];   // a Pear?
```

`maul2()`에서의 대입은 복사 손실이 없도록 하라는 [규칙](#Res-slice)을 위반한다는 점에 유의하라.

##### Enforcement

* 이 공포스러운 문제를 찾아내야 한다!

### <a name="Rt-linear"></a>T.82: Linearize a hierarchy when virtual functions are undesirable

##### Reason

???

##### Example

???

##### Enforcement

???

### <a name="Rt-virtual"></a>T.83: Do not declare a member function template virtual

##### Reason

C++이 지원하지 않는다.
가상함수 테이블(vtbl)은 링크 시간까지는 생성할 수 없다. 때문에 보통은 동적링킹으로 구현해야 한다.

##### Example, don't

```c++
    class Shape {
        // ...
        template<class T>
        virtual bool intersect(T* p);   // error: template cannot be virtual
    };
```

##### Note

사람들이 이와 관련해 계속 물어보고 있다. 규칙이 필요하다.

##### Alternative

Double dispatch, Visitor 패턴, 어떤 함수를 호출하는지 계산

##### Enforcement

컴파일러가 처리한다.

### <a name="Rt-abi"></a>T.84: Use a non-template core implementation to provide an ABI-stable interface

##### Reason

코드 안정성을 개선한다.
코드가 급증하는 것을 막아준다.

##### Example

기본 클래스로 만들수도 있다:

```c++
    struct Link_base {   // stable
        Link_base* suc;
        Link_base* pre;
    };

    template<typename T>   // templated wrapper to add type safety
    struct Link : Link_base {
        T val;
    };

    struct List_base {
        Link_base* first;   // first element (if any)
        int sz;             // number of elements
        void add_front(Link_base* p);
        // ...
    };

    template<typename T>
    class List : List_base {
    public:
        void put_front(const T& e) { add_front(new Link<T>{e}); }   // implicit cast to Link_base
        T& front() { static_cast<Link<T>*>(first).val; }   // explicit cast back to Link<T>
        // ...
    };

    List<int> li;
    List<string> ls;
```

여기는 `List`의 요소를 연결하고 해제하는 함수가 한벌(one copy)만 있다.
`Link`, `List` 클래스는 타입 조작만 한다.

기본 타입을 분리하는 대신에 일반적으로는 `void`, `void*`에 대해서 특수화하고 핵심 `void` 구현에서 안전하게 `T`로 형변환하도록 템플릿을 가지도록 한다.

**Alternative**: [Pimpl](#Ri-pimpl)을 사용할수도 있다

##### Enforcement

???

## <a name="SS-variadic"></a>T.var: Variadic template rules

???

### <a name="Rt-variadic"></a>T.100: Use variadic templates when you need a function that takes a variable number of arguments of a variety of types

##### Reason

가변인자 템플릿은 가장 일반화 메커니즘이면서 동시에 효율적이고, 타입안정성을 지닌다. C 형식의 가변인자를 사용하지 말아라.

##### Example

    ??? printf

##### Enforcement

* 코드에 `va_arg`이 있다면 지적한다

### <a name="Rt-variadic-pass"></a>T.101: ??? How to pass arguments to a variadic template ???

##### Reason

 ???

##### Example

    ??? 이동만 가능하거나 인자 참조에 주의하라

##### Enforcement

???

### <a name="Rt-variadic-process"></a>T.102: How to process arguments to a variadic template

##### Reason

 ???

##### Example

    ??? forwarding, type checking, references

##### Enforcement

???

### <a name="Rt-variadic-not"></a>T.103: Don't use variadic templates for homogeneous argument lists

##### Reason

`initializer_list`같은 형태로 같은 타입의 인자들을 더 정확히 지정할 수 있다.

##### Example

    ???

##### Enforcement

???

## <a name="SS-meta"></a>T.meta: 템플릿 메타프로그래밍 (TMP)

템플릿은 컴파일-타임 프로그래밍을 위한 일반적인 매커니즘을 제공한다.
메타프로그래밍은 하나 이상의 입력이나 결과 자체가 타입인 프로그래밍을 말한다.

템플릿은 튜링 완전성을 가진 (modulo memory capacity) 덕타이핑을 제공한다.이를 위해 필요한 문법과 기술은 아주 끔찍하다(horrendous).

### <a name="Rt-metameta"></a>T.120: Use template metaprogramming only when you really need to

##### Reason

템플릿 메타프로그래밍은 올바르게 쓰기가 어렵고, 컴파일 속도를 느리게 하고, 유지보수를 어렵게 한다.
그러나 템플릿 메타프로그래밍이 전문가 수준의 어셈블리 코드보다 성능이 더 좋은 예들이 있다.
게다가 런타임 코드보다 핵심사상을 더 잘 표현하는 실제 예들도 있다.

예를 들어 컴파일 타임에 AST(Abstract Syntax Tree)를 조작해야 한다면 (예컨대 선택적으로 행렬 연산을 전파(folding)한다던지) C++에서는 다른 방법이 없다.

##### Example, bad

    ???

##### Example, bad

    enable_if

대안으로, 컨셉을 사용하라. [언어가 지원하지 않는 컨셉을 사용하는 방법](#Rt-emulate)을 참고하라.

##### Example

    ??? good

**Alternative**: 만약 결과가 타입이 아니라 값이라면 [`constexpr` 함수](#Rt-fct)를 사용하라.

##### Note

템플릿 메타프로그래밍을 전처리 매크로로 대신하고 싶다고 느낀다면 너무 나간것이다.

### <a name="Rt-emulate"></a>T.121: Use template metaprogramming primarily to emulate concepts

##### Reason

컨셉 개념이 널리 사용될때까지 TMP를 사용해서 에뮬레이트해야 할 것이다.
Use cases that require concepts (e.g. overloading based on concepts) are among the most common (and simple) uses of TMP.

##### Example

```c++
    template<typename Iter>
        /*requires*/ enable_if<random_access_iterator<Iter>, void>
    advance(Iter p, int n) { p += n; }

    template<typename Iter>
        /*requires*/ enable_if<forward_iterator<Iter>, void>
    advance(Iter p, int n) { assert(n >= 0); while (n--) ++p;}
```

##### Note

아래 코드는 컨셉을 사용하면 엄청 쉬워진다:

```c++
    void advance(RandomAccessIterator p, int n) { p += n; }

    void advance(ForwardIterator p, int n) { assert(n >= 0); while (n--) ++p;}
```

##### Enforcement

???

### <a name="Rt-tmp"></a>T.122: Use templates (usually template aliases) to compute types at compile time

##### Reason

템플릿 메타프로그래밍은 컴파일 타임에 타입을 생성하기 위한 유일하고 직접적인 방법이다.

##### Note

"특성(Traits)" 사용방법은 대부분 템플릿 별칭이나 `constexpr`함수로 대체되었다.

##### Example

    ??? big object / small object optimization

##### Enforcement

???

### <a name="Rt-fct"></a>T.123: Use `constexpr` functions to compute values at compile time

##### Reason

함수는 값계산을 표현하는데 가장 분명하고 일반적인 방법이다.
때때로 `constexpr` 함수는 일반 함수보다 컴파일 비용이 적다.

##### Note

"특성(Traits)" 사용방법은 대부분 템플릿 별칭이나 `constexpr`함수로 대체되었다.

##### Example

```c++
    template<typename T>
        // requires Number<T>
    constexpr T pow(T v, int n)   // power/exponential
    {
        T res = 1;
        while (n--) res *= v;
        return res;
    }

    constexpr auto f7 = pow(pi, 7);
```

##### Enforcement

* Flag template metaprograms yielding a value. These should be replaced with `constexpr` functions.

### <a name="Rt-std-tmp"></a>T.124: Prefer to use standard-library TMP facilities

##### Reason

`conditional`, `enable_if`, `tuple`같은 표준에서 정의한 기능이 호환이 좋고, 잘 알려져 있다.

##### Example

    ???

##### Enforcement

???

### <a name="Rt-lib"></a>T.125: If you need to go beyond the standard-library TMP facilities, use an existing library

##### Reason

Getting advanced TMP facilities is not easy and using a library makes you part of a (hopefully supportive) community.

정말 필요한 경우에만 자신만의 "상급 TMP 지원"을 작성하라.

##### Example

    ???

##### Enforcement

???

## <a name="SS-temp-other"></a> 기타 템플릿 규칙

### <a name="Rt-name"></a>T.140: Name all operations with potential for reuse

##### Reason

문서화, 가독성, 재사용 가능성.

##### Example

```c++
    struct Rec {
        string name;
        string addr;
        int id;         // unique identifier
    };

    bool same(const Rec& a, const Rec& b)
    {
        return a.id == b.id;
    }

    vector<Rec*> find_id(const string& name);    // find all records for "name"

    auto x = find_if(vr.begin(), vr.end(),
        [&](Rec& r) {
            if (r.name.size() != n.size()) return false; // name to compare to is in n
            for (int i = 0; i < r.name.size(); ++i)
                if (tolower(r.name[i]) != tolower(n[i])) return false;
            return true;
        }
    );
```

There is a useful function lurking here (case insensitive string comparison), as there often is when lambda arguments get large.

```c++
    bool compare_insensitive(const string& a, const string& b)
    {
        if (a.size() != b.size()) return false;
        for (int i = 0; i < a.size(); ++i) if (tolower(a[i]) != tolower(b[i])) return false;
        return true;
    }

    auto x = find_if(vr.begin(), vr.end(),
        [&](Rec& r) { compare_insensitive(r.name, n); }
    );
```

Or maybe (if you prefer to avoid the implicit name binding to n):

```c++
    auto cmp_to_n = [&n](const string& a) { return compare_insensitive(a, n); };

    auto x = find_if(vr.begin(), vr.end(),
        [](const Rec& r) { return cmp_to_n(r.name); }
    );
```

##### Note

함수, 람다, 연산자. 모든 것에 적용된다.

##### Exception

* 지역 유효범위에서만 사용하는 람다, `for_each`문 인자, 유사한 제어흐름 알고리즘
* [변수 초기화를 위한](#???) 람다

##### Enforcement

* (hard) flag similar lambdas
* ???

### <a name="Rt-lambda"></a>T.141: Use an unnamed lambda if you need a simple function object in one place only

##### Reason

코드를 간결하게 만들고 다른 방법에 비해 인접성(locality)도 좋다.

##### Example

```c++
    auto earlyUsersEnd = std::remove_if(users.begin(), users.end(),
                                        [](const User &a) { return a.id > 100; });
```

##### Exception

한번만 사용한다고 해도 람다에 이름을 붙이면 분명해 보인다.

##### Enforcement

* 동일한, 거의 동일한 람다를 찾는다 (이름있는 함수 혹은 이름있는 람다로 바꾸도록 한다).

### <a name="Rt-var"></a>T.142?: Use template variables to simplify notation

##### Reason

가독성 개선

##### Example

    ???

##### Enforcement

???

### <a name="Rt-nongeneric"></a>T.143: Don't write unintentionally nongeneric code

##### Reason

일반화, 재사용성, 필요 이상으로 자세하게 하지 마라; 가능한 일반적인 기능을 사용하라.

##### Example

반복자(iterator) 비교에는 `<`대신 `!=`를 사용하라; `!=`는 순서의 영향을 받지 않기 때문에 더 많은 경우에 사용할 수 있다.

```c++
    for (auto i = first; i < last; ++i) {   // less generic
        // ...
    }

    for (auto i = first; i != last; ++i) {   // good; more generic
        // ...
    }
```

물론, 범위기반-`for`문이 쓰기가 더 좋다.

##### Example

필요한 기능을 가진 기본클래스를 사용하라.

```c++
    class Base {
    public:
        Bar f();
        Bar g();
    };

    class Derived1 : public Base {
    public:
        Bar h();
    };

    class Derived2 : public Base {
    public:
        Bar j();
    };

    // bad, unless there is a specific reason for limiting to Derived1 objects only
    void my_func(Derived1& param)
    {
        use(param.f());
        use(param.g());
    }

    // good, uses only Base interface so only commit to that
    void my_func(Base& param)
    {
        use(param.f());
        use(param.g());
    }
```

##### Enforcement

* 반복자 비교에 `!=` 대신에 `<`를 쓴다면 지적하라
* Flag `x.size() == 0` when `x.empty()` or `x.is_empty()` is available. Emptiness works for more containers than size(), because some containers don't know their size or are conceptually of unbounded size.
* 상속된 타입에 대한 포인터나 참조를 가지고 있지만 기본 타입으로 선언된 함수만 사용하는 함수가 있다면 지적한다

### <a name="Rt-specialize-function"></a>T.144: Don't specialize function templates

##### Reason

언어규칙에 따라 함수 템플릿을 부분적으로 특수화할 수 없다.
함수템플릿을 전부 특수화할 수 있지만 그 대신으로 오버로딩하고 싶을 것이다. 함수 템플릿 특수화는 오버로딩으로 해석되지 때문에 원하는대로 동작하지 않는다.
드물지만 적절히 특수화할 수 있는 클래스 템플릿과 연계함으로써 실제로 특수화를 할 수 있다.

##### Example

    ???

**Exceptions**: 함수템플릿을 특수화할 타당한 이유가 있다면 클래스 템플릿에 위임할 단 한개의 함수템플릿을 작성해라.
그리고 클래스 템플릿을 특수화하라. (부분 특수화를 작성하는 것까지 포함하라)

##### Enforcement

* 함수템플릿을 특수화하고 있다면 지적하라. 가능하다면 오버로딩으로 대신하라

### <a name="Rt-check-class"></a>T.150: Check that a class matches a concept using `static_assert`

##### Reason

If you intend for a class to match a concept, verifying that early saves users pain.

##### Example

```c++
    class X {
    public:
        X() = delete;
        X(const X&) = default;
        X(X&&) = default;
        X& operator=(const X&) = default;
        // ...
    };
```

Somewhere, possibly in an implementation file, let the compiler check the desired properties of `X`:

```c++
    static_assert(Default_constructible<X>);    // error: X has no default constructor
    static_assert(Copyable<X>);                 // error: we forgot to define X's move constructor
```

##### Enforcement

Not feasible.
