# <a name="S-templates"></a> T: Templates and generic programming

일반화 프로그래밍은 타입, 값, 알고리즘을 매개변수화하는 타입과 알고리즘을 사용하는 프로그래밍이다.
C++에서 일반화 프로그래밍은 `template` 언어적 장치로 지원하고 있다.
>Generic programming is programming using types and algorithms parameterized by types, values, and algorithms.
In C++, generic programming is supported by the `template` language mechanisms.

일반화된 함수의 인자는 인자타입과 관련된 값에 대한 요구사항들을 특징짓는다.
C++에서 이런 요구사항은 컨셉이라는 컴파일타임 서술어로 표현된다.
>Arguments to generic functions are characterized by sets of requirements on the argument types and values involved.
In C++, these requirements are expressed by compile-time predicates called concepts.

템플릿은 메타프로그래밍용으로 사용될 수 있다; 즉 컴파일 타임 때 코드를 만들어내는 프로그램이다.
>Templates can also be used for meta-programming; that is, programs that compose code at compile time.

템플릿 사용 규칙 요약:
>Template use rule summary:

* [T.1: Use templates to raise the level of abstraction of code](#Rt-raise)
* [T.2: Use templates to express algorithms that apply to many argument types](#Rt-algo)
* [T.3: Use templates to express containers and ranges](#Rt-cont)
* [T.4: Use templates to express syntax tree manipulation](#Rt-expr)
* [T.5: Combine generic and OO techniques to amplify their strengths, not their costs](#Rt-generic-oo)

컨셉 사용 규칙 요약:
>Concept use rule summary:

* [T.10: Specify concepts for all template arguments](#Rt-concepts)
* [T.11: Whenever possible use standard concepts](#Rt-std-concepts)
* [T.12: Prefer concept names over `auto` for local variables](#Rt-auto)
* [T.13: Prefer the shorthand notation for simple, single-type argument concepts](#Rt-shorthand)
* ???

컨셉 정의 규칙 요약:
>Concept definition rule summary:

* [T.20: Avoid "concepts" without meaningful semantics](#Rt-low)
* [T.21: Define concepts to define complete sets of operations](#Rt-complete)
* [T.22: Specify axioms for concepts](#Rt-axiom)
* [T.23: Differentiate a refined concept from its more general case by adding new use patterns](#Rt-refine)
* [T.24: Use tag classes or traits to differentiate concepts that differ only in semantics](#Rt-tag)
* [T.25: Avoid negating constraints](#Rt-not)
* [T.26: Prefer to define concepts in terms of use-patterns rather than simple syntax](#Rt-use)
* ???

템플릿 인터페이스 규칙 요약:
>Template interface rule summary:

* [T.40: Use function objects to pass operations to algorithms](#Rt-fo)
* [T.41: Require complete sets of operations for a concept](#Rt-operations)
* [T.42: Use template aliases to simplify notation and hide implementation details](#Rt-alias)
* [T.43: Prefer `using` over `typedef` for defining aliases](#Rt-using)
* [T.44: Use function templates to deduce class template argument types (where feasible)](#Rt-deduce)
* [T.46: Require template arguments to be at least `Regular` or `SemiRegular`](#Rt-regular)
* [T.47: Avoid highly visible unconstrained templates with common names](#Rt-visible)
* [T.48: If your compiler does not support concepts, fake them with `enable_if`](#Rt-concept-def)
* [T.49: Where possible, avoid type-erasure](#Rt-erasure)
* [T.50: Avoid writing an unconstrained template in the same namespace as a type](#Rt-unconstrained-adl)

템플릿 정의 규칙 요약:
>Template definition rule summary:

* [T.60: Minimize a template's context dependencies](#Rt-depend)
* [T.61: Do not over-parameterize members (SCARY)](#Rt-scary)
* [T.62: Place non-dependent template members in a non-templated base class](#Rt-nondependent)
* [T.64: Use specialization to provide alternative implementations of class templates](#Rt-specialization)
* [T.65: Use tag dispatch to provide alternative implementations of functions](#Rt-tag-dispatch)
* [T.66: Use selection using `enable_if` to optionally define a function](#Rt-enable_if)
* [T.67: Use specialization to provide alternative implementations for irregular types](#Rt-specialization2)
* [T.68: Use `{}` rather than `()` within templates to avoid ambiguities](#Rt-cast)
* [T.69: Inside a template, don't make an unqualified nonmember function call unless you intend it to be a customization point](#Rt-customization)

템플릿과 상속구조 규칙 요약:
>Template and hierarchy rule summary:

* [T.80: Do not naively templatize a class hierarchy](#Rt-hier)
* [T.81: Do not mix hierarchies and arrays](#Rt-array) // ??? somewhere in "hierarchies"
* [T.82: Linearize a hierarchy when virtual functions are undesirable](#Rt-linear)
* [T.83: Do not declare a member function template virtual](#Rt-virtual)
* [T.84: Use a non-template core implementation to provide an ABI-stable interface](#Rt-abi)
* [T.??: ????](#Rt-???)

가변인자 템플릿규칙 요약:
>Variadic template rule summary:

* [T.100: Use variadic templates when you need a function that takes a variable number of arguments of a variety of types](#Rt-variadic)
* [T.101: ??? How to pass arguments to a variadic template ???](#Rt-variadic-pass)
* [T.102: ??? How to process arguments to a variadic template ???](#Rt-variadic-process)
* [T.103: Don't use variadic templates for homogeneous argument lists](#Rt-variadic-not)
* [T.??: ????](#Rt-???)

메타프로그래밍 규칙 요약:
>Metaprogramming rule summary:

* [T.120: Use template metaprogramming only when you really need to](#Rt-metameta)
* [T.121: Use template metaprogramming primarily to emulate concepts](#Rt-emulate)
* [T.122: Use templates (usually template aliases) to compute types at compile time](#Rt-tmp)
* [T.123: Use `constexpr` functions to compute values at compile time](#Rt-fct)
* [T.124: Prefer to use standard-library TMP facilities](#Rt-std-tmp)
* [T.125: If you need to go beyond the standard-library TMP facilities, use an existing library](#Rt-lib)
* [T.??: ????](#Rt-???)

다른 템플릿 규칙 요약:
>Other template rules summary:

* [T.140: Name all nontrivial operations](#Rt-name)
* [T.141: Use an unnamed lambda if you need a simple function object in one place only](#Rt-lambda)
* [T.142: Use template variables to simplify notation](#Rt-var)
* [T.143: Don't write unintentionally nongeneric code](#Rt-nongeneric)
* [T.144: Don't specialize function templates](#Rt-specialize-function)
* [T.??: ????](#Rt-???)

## <a name="SS-GP"></a> T.gp: 일반화 프로그래밍
>## <a name="SS-GP"></a> T.gp: Generic programming

일반화 프로그래밍은 타입, 값, 알고리즘을 매개변수화하는 타입과 알고리즘을 사용하는 프로그래밍이다.
>Generic programming is programming using types and algorithms parameterized by types, values, and algorithms.

### <a name="Rt-raise"></a> T.1: 추상화 레벨을 올리기 위해 템플릿을 사용하라.
>### <a name="Rt-raise"></a> T.1: Use templates to raise the level of abstraction of code

##### Reason

일반화. 재사용성. 효율성. 사용자타입의 일관된 정의를 장려한다.
>Generality. Re-use. Efficiency. Encourages consistent definition of user types.

##### Example, bad

개념적으로, 아래 요구사항은 잘못됐다. 왜냐하면 우리가 원하는 `T`는 "증가될 수 있다"거나 "추가될 수 있다"는 하위레벨 컨셉 그 이상이다.
>Conceptually, the following requirements are wrong because what we want of `T` is more than just the very low-level concepts of "can be incremented" or "can be added":

    template<typename T, typename A>
        // requires Incrementable<T>
    A sum1(vector<T>& v, A s)
    {
        for (auto x : v) s+=x;
        return s;
    }

    template<typename T, typename A>
        // requires Simple_number<T>
    A sum2(vector<T>& v, A s)
    {
        for (auto x : v) s = s + x;
        return s;
    }

`Incrementable`이 `+`를 지원하지 않고, `Simple_number`이 `+=`를 지원하지 않는다고 가정하면 `sum1`과 `sum2`의 구현을 과도하게 제약해왔다.
그리고 이런 경우에는 일반화를 위해 기회를 놓쳤다.
>Assuming that `Incrementable` does not support `+` and `Simple_number` does not support `+=`, we have overconstrained implementers of `sum1` and `sum2`.
And, in this case, missed an opportunity for a generalization.

##### Example

    template<typename T, typename A>
        // requires Arithmetic<T>
    A sum(vector<T>& v, A s)
    {
        for (auto x : v) s+=x;
        return s;
    }

Assuming that `Arithmetic` requires both `+` and `+=`, we have constrained the user of `sum` to provide a complete arithmetic type.
That is not a minimal requirement, but it gives the implementer of algorithms much needed freedom and ensures that any `Arithmetic` type
can be user for a wide variety of algorithms.

For additional generality and reusability, we could also use a more general `Container` or `Range` concept instead of committing to only one container, `vector`.

##### Note

If we define a template to require exactly the operations required for a single implementation of a single algorithm
(e.g., requiring just `+=` rather than also `=` and `+`) and only those, we have overconstrained maintainers.
We aim to minimize requirements on template arguments, but the absolutely minimal requirements of an implementation is rarely a meaningful concept.

##### Note

템플릿은 모든 것을 표현하는데 사용할 수 있다.(모든 것은 튜링(?)에 닫혀있다.)
그러나 일반화 프로그래밍의 목표는 비슷한 의미특성을 가진 타입집합에 대한 연산/알고리즘을 효과적으로 일반화하는 것이다.
>Templates can be used to express essentially everything (they are Turing complete), but the aim of generic programming (as expressed using templates)
is to efficiently generalize operations/algorithms over a set of types with similar semantic properties.

##### Enforcement

* 컨셉없이 특정 연산자를 바로 사용하는 것같은, 과도하게 단순한 요구사항을 가진 알고리즘이 있다면 표시한다.
* 과도하게 단순한 컨셉 정의가 있다면 표시하지 않는다; 더 쓸만한 컨셉을 위해 블록을 만들지도 모른다.

>* Flag algorithms with "overly simple" requirements, such as direct use of specific operators without a concept.
>* Do not flag the definition of the "overly simple" concepts themselves; they may simply be building blocks for more useful concepts.

### <a name="Rt-algo"></a> T.2: 다수의 인자 타입에 적용되는 알고리즘을 표현하기 위해 템플릿을 사용하라.
>### <a name="Rt-algo"></a> T.2: Use templates to express algorithms that apply to many argument types

##### Reason

일반화. 소스코드 사이즈를 최소화한다. 상호운용성. 재사용.
>Generality. Minimizing the amount of source code. Interoperability. Re-use.

##### Example

STL의 기본이다. `find` 알고리즘은 모든 종류의 입력 범위에서 잘 작동한다.
>That's the foundation of the STL. A single `find` algorithm easily works with any kind of input range:

    template<typename Iter, typename Val>
        // requires Input_iterator<Iter>
        //       && Equality_comparable<Value_type<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }

##### Note

한개 이상의 템플릿 인자타입에 대한 현실적인 필요가 없다면 템플릿을 사용하지 마라.
과도하게 추상화하지 마라.
>Don't use a template unless you have a realistic need for more than one template argument type.
Don't overabstract.

##### Enforcement

??? 터프, 아마도 인간이 필요한다.
>??? tough, probably needs a human

### <a name="Rt-cont"></a> T.3: 컨테이너와 범위를 표현하기 위해 템플릿을 사용하라.
>### <a name="Rt-cont"></a> T.3: Use templates to express containers and ranges

##### Reason

컨테이너는 요소의 타입을 필요로 하고, 템플릿인자로 표현하는 것이 일반적이고, 재사용가능하고, 타입 안전하다.
불안정하고 비효율적인 해결방법을 피한다. 관례: STL이 하는 방법이다.
>Containers need an element type, and expressing that as a template argument is general, reusable, and type safe.
It also avoids brittle or inefficient workarounds. Convention: That's the way the STL does it.

##### Example

    template<typename T>
        // requires Regular<T>
    class Vector {
        // ...
        T* elem;	// points to sz Ts
        int sz;
    };

    vector<double> v(10);
    v[7] = 9.9;

##### Example, bad

    class Container {
        // ...
        void* elem;	// points to size elements of some type
        int sz;
    };

    Container c(10, sizeof(double));
    ((double*)c.elem)[] = 9.9;

이것은 프로그래머의 의도를 직접적으로 표현하지 않는다. 타입시스템과 옵티마이저로부터 프로그램의 구조를 숨긴다.
>This doesn't directly express the intent of the programmer and hides the structure of the program from the type system and optimizer.

메크로 뒤에서 `void*`를 숨김은 단순히 문제를 어렵게 한다. 혼란스럽게도 새로운 기회를 제공한다.
>Hiding the `void*` behind macros simply obscures the problems and introduces new opportunities for confusion.

**Exceptions**: ABI지원 인터페이스가 필요하다면 기본 구현을 제공하고, 타입안전 템플릿으로 표현해야 할지도 모른다.
[Stable base](#Rt-abi)를 참조하라.
>**Exceptions**: If you need an ABI-stable interface, you might have to provide a base implementation and express the (type-safe) template in terms of that.
See [Stable base](#Rt-abi).

##### Enforcement

* `void*`과 하위레벨 구현코드 외에 형변환을 사용한다면 표시한다.

>* Flag uses of `void*`s and casts outside low-level implementation code

### <a name="Rt-expr"></a> T.4: 문법 트리 조작을 표현하기 위해 템플릿을 사용하라.
>### <a name="Rt-expr"></a> T.4: Use templates to express syntax tree manipulation

##### Reason

 ???

##### Example

    ???

**Exceptions**: ???

### <a name="Rt-generic-oo"></a> T.5: 비용이 아니라 강점을 증폭시킬 일반화와 객체지향 기술을 결합하라.
>### <a name="Rt-generic-oo"></a> T.5: Combine generic and OO techniques to amplify their strengths, not their costs

##### Reason

일반화와 객체지향 기술은 상호보완적이다.
>Generic and OO techniques are complementary.

##### Example

정적은 동적을 돕는다: 동적으로 다형적 인터페이스를 구현하기 위해 정적 다형성을 사용하라.
>Static helps dynamic: Use static polymorphism to implement dynamically polymorphic interfaces.

    class Command {
        // pure virtual functions
    };

    // implementations
    template</*...*/>
    class ConcreteCommand : public Command {
        // implement virtuals
    };

##### Example

동적은 정적을 돕는다: 일반적인, 편안한, 정적으로 묶인 인터페이스, 그러나 내부적으로는 동적으로 dispatch(?)를 제공한다.
그래서 획일적인 객체 배치를 제공한다. `std::shared_ptr`의 제거자로써 타입제거를 예제로 포함할 수 있다.
([don't overuse type erasure](#Rt-erasure).)
>Dynamic helps static: Offer a generic, comfortable, statically bound interface, but internally dispatch dynamically, so you offer a uniform object layout. Examples include type erasure as with `std::shared_ptr`’s deleter. (But [don't overuse type erasure](#Rt-erasure).)

##### Note

클래스템플릿 속에 있는 비상속함수는 사용될때만 한번 인스턴스화된다. -- 가상함수는 매번 인스턴스화된다.
이것은 코드사이즈를 늘리고 필요치도 않는 함수를 인스턴스화함으로써 일반화 타입에 대한 제약을 심화시킬지도 모른다.
>In a class template, nonvirtual functions are only instantiated if they're used -- but virtual functions are instantiated every time. This can bloat code size, and may overconstrain a generic type by instantiating functionality that is never needed. Avoid this, even though the standard facets made this mistake.

##### Enforcement

* 새로운(상속하지 않는) 가상 함수를 정의하는 클래스 템플릿이 있다면 표시한다.

>* Flag a class template that declares new (non-inherited) virtual functions.

## <a name="SS-tpg-concepts"></a> TPG.concepts: 컨셉 규칙
>## <a name="SS-tpg-concepts"></a> TPG.concepts: Concept rules

컨셉은 템플릿 인자용 요구사항을 기술하기 위한 기능이다.
[ISO technical specification](#Ref-conceptsTS)이다. 아직 컴파일러가 지원하지 않는다.
그러나 컨셉은 일반화 프로그래밍, 미래의 C++ 라이브러리(표준과 다른 것)에 대한 많은 기초작업에 대해 고려할 때는 결정적이다.
>Concepts is a facility for specifying requirements for template arguments.
It is an [ISO technical specification](#Ref-conceptsTS), but not yet supported by currently shipping compilers.
Concepts are, however, crucial in the thinking about generic programming and the basis of much work on future C++ libraries
(standard and other).

컨셉 사용 규칙 요약:
>Concept use rule summary:

* [T.10: Specify concepts for all template arguments](#Rt-concepts)
* [T.11: Whenever possible use standard concepts](#Rt-std-concepts)
* [T.14: Prefer concept names over `auto`](#Rt-auto)
* [T.15: Prefer the shorthand notation for simple, single-type argument concepts](#Rt-shorthand)
* ???

컨셉 정의 규칙 요약:
>Concept definition rule summary:

* [T.20: Avoid "concepts" without meaningful semantics](#Rt-low)
* [T.21: Define concepts to define complete sets of operations](#Rt-complete)
* [T.22: Specify axioms for concepts](#Rt-axiom)
* [T.23: Differentiate a refined concept from its more general case by adding new use patterns](#Rt-refine)
* [T.24: Use tag classes or traits to differentiate concepts that differ only in semantics](#Rt-tag)
* [T.25: Avoid negating constraints](#Rt-not)
* [T.26: Prefer to define concepts in terms of use-patterns rather than simple syntax](#Rt-use)
* ???

## <a name="SS-concept-use"></a> T.con-use: 컨셉 사용
>## <a name="SS-concept-use"></a> T.con-use: Concept use

### <a name="Rt-concepts"></a> T.10: 모든 템플릿 인자를 위해 개념을 기술하라.
>### <a name="Rt-concepts"></a> T.10: Specify concepts for all template arguments

##### Reason

정확함과 가독성.
템플릿 인자의 가정된 의미(문법과 의미구조)는 템플릿 인터페이스의 기본이다.
컨셉은 문서화와 템플릿용 에러 처리를 드라마틱하게 개선시킨다.
템플릿 인자를 위한 개념을 기술하는 것은 강력한 디자인 도구이다.
>Correctness and readability.
The assumed meaning (syntax and semantics) of a template argument is fundamental to the interface of a template.
A concept dramatically improves documentation and error handling for the template.
Specifying concepts for template arguments is a powerful design tool.

##### Example

    template<typename Iter, typename Val>
        requires Input_iterator<Iter>
                 && Equality_comparable<Value_type<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }

또는 동등하게, 그리고 더 간결하게:
>or equivalently and more succinctly:

    template<Input_iterator Iter, typename Val>
        requires Equality_comparable<Value_type<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }

##### Note

컴파일러가 언어특징으로 컨셉을 지원할때까지 컨셉을 주석으로 묶어라:
>Until your compilers support the concepts language feature, leave the concepts in comments:

    template<typename Iter, typename Val>
        // requires Input_iterator<Iter>
        //       && Equality_comparable<Value_type<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }

##### Note

평범한 `typename`(또는 `auto`)는 가장 제약이 작은 컨셉이다.
타입이라는 것을 가정하지 않는다면 단지 아주 예외적으로만 써야 한다.
(템플릿 메타프로그래밍 코드의 한 부분으로써) 우리가 순수한 연산식 트리를 조작하고, 타입 체크를 연기할때만 필요하다.
>Plain `typename` (or `auto`) is the least constraining concept.
It should be used only rarely when nothing more than "it's a type" can be assumed.
This is typically only needed when (as part of template metaprogramming code) we manipulate pure expression trees, postponing type checking.

**References**: TC++PL4, Palo Alto TR, Sutton

##### Enforcement

컨셉없이 템플릿 타입 인자가 있다면 표시한다.
>Flag template type arguments without concepts

### <a name="Rt-std-concepts"></a> T.11: 가능하면 표준 컨셉을 사용하라.
>### <a name="Rt-std-concepts"></a> T.11: Whenever possible use standard concepts

##### Reason

우리만의 컨셉을 생각해 내는 작업을 도울 수 있는 (GSL, ISO 컨셉 TS 그리고 희망적이이게도 곧 ISO 표준이 될)표준 컨셉은
우리가 서둘러 한 것과 상호운용성을 개선한 것보다 더 신중히 고려되었다.

>"Standard" concepts (as provided by the GSL, the ISO concepts TS, and hopefully soon the ISO standard itself)
saves us the work of thinking up our own concepts, are better thought out than we can manage to do in a hurry, and improves interoperability.

##### Note

새로운 일반화 라이브러리를 만들지 않는다면 필요한 많은 컨셉은 이미 표준 라이브러리 안에 정의되어 있다.
>Unless you are creating a new generic library, most of the concepts you need will already be defined by the standard library.

##### Example

    concept<typename T>
    // don't define this: Sortable is in the GSL
    Ordered_container = Sequence<T> && Random_access<Iterator<T>> && Ordered<Value_type<T>>;

    void sort(Ordered_container& s);

이 `Ordered_container`는 아주 타당해 보인다. 그러나 GSL(그리고 Range TS)안에 있는 `Sortable` 컨셉과 아주 비슷하다.
더 좋냐? 더 올바르냐? `sort`에 대한 표준 요구사항을 정확하게 반영하고 있느냐?
`Sortable`을 사용하는 것이 더 좋고 단순하다.
>This `Ordered_container` is quite plausible, but it is very similar to the `Sortable` concept in the GSL (and the Range TS).
Is it better? Is it right? Does it accurately reflect the standard's requirements for `sort`?
It is better and simpler just to use `Sortable`:

    void sort(Sortable& s);		// better

##### Note

표준 컨셉집합은 실제 ISO 표준화로 진화하고 있다.
>The set of "standard" concepts is evolving as we approaches real (ISO) standardization.

##### Note

쓸만한 컨셉을 디자인하는 것은 굉장히 도전적인 일이다.
>Designing a useful concept is challenging.

##### Enforcement

어렵다.
>Hard.

* 제약조건이 없는 인자, 비표준 컨셉을 쓰는 템플릿, 공리없이 'homebrew' 컨셉을 쓰는 템플릿을 찾아라.
* 컨셉 발견 툴을 개발하라. [an early experiment](http://www.stroustrup.com/sle2010_webversion.pdf)을 참조하라.

>* Look for unconstrained arguments, templates that use "unusual"/non-standard concepts, templates that use "homebrew" concepts without axioms.
* Develop a concept-discovery tool (e.g., see [an early experiment](http://www.stroustrup.com/sle2010_webversion.pdf).

### <a name="Rt-auto"></a> T.12: 지역변수에 대해 `auto`외에 컨셉이름을 선호하라.
>### <a name="Rt-auto"></a> T.12: Prefer concept names over `auto` for local variables

##### Reason

`auto`는 가장 약한 컨셉이다. 컨셉 이름은 `auto`보다 더 많은 의미을 전달한다.
 >`auto` is the weakest concept. Concept names convey more meaning than just `auto`.

##### Example

    vector<string> v;
    auto& x = v.front();	// bad
    String& s = v.begin();	// good

##### Enforcement

* ???

### <a name="Rt-shorthand"></a> T.13: 한 종류 인자 컨셉에 대해서는 약칭 표기법을 선호하라.
>### <a name="Rt-shorthand"></a> T.13: Prefer the shorthand notation for simple, single-type argument concepts

##### Reason

가독성. 아이디어의 직접적인 표현.
>Readability. Direct expression of an idea.

##### Example

`T`는 `Sortable`라고 말하면:
>To say "`T` is `Sortable`":

    template<typename T>		// Correct but verbose: "The parameter is
        requires Sortable<T>	// of type T which is the name of a type
    void sort(T&);				// that is Sortable"

    template<Sortable T>		// Better: "The parameter is of type T
    void sort(T&);				// which is Sortable"

    void sort(Sortable&);		// Best: "The parameter is Sortable"

약칭 버전이 우리가 말하는 방법과 더 잘 일치한다. 많은 템플릿은 `template` 키워드를 사용할 필요가 없다는 점에 주목하라.
>The shorter versions better match the way we speak. Note that many templates don't need to use the `template` keyword.

##### Enforcement

* `<typename T>`과 `<class T>`를 변경할 때 짧은 단어로는 쓰기가 불가능하다.
* 후에 처음 typename을 도입하고, 그 후에 한 종류 인자 컨셉으로 제한되는 선언이 있다면 표시한다.

>* Not feasible in the short term when people convert from the `<typename T>` and `<class T`> notation.
>* Later, flag declarations that first introduces a typename and then constrains it with a simple, single-type-argument concept.

## <a name="SS=concept-def"></a> T.con-def: 컨셉 정의 규칙
>## <a name="SS=concept-def"></a> T.con-def: Concept definition rules

???

### <a name="Rt-low"></a> T.20: 의미없는 컨셉을 피하라.
>### <a name="Rt-low"></a> T.20: Avoid "concepts" without meaningful semantics

##### Reason

컨셉은 의미적인 개념을 표현하려는 의도이다. 예를 들면 "숫자", 요소들의 "범위", 그리고 "완전히 소팅된".
단순한 제약조건, `+`연산자를 가진다거나, `>`연산자를 가지는 것과 같은, 독자적으로 기술되어선 의미가 없다.
유저 코드보다는 의미있는 개념을 위한 블록을 구성하는데 사용되어야 한다.
>Concepts are meant to express semantic notions, such as "a number", "a range" of elements, and "totally ordered."
Simple constraints, such as "has a `+` operator" and "has a `>` operator" cannot be meaningfully specified in isolation
and should be used only as building blocks for meaningful concepts, rather than in user code.

##### Example, bad

    template<typename T>
    concept Addable = has_plus<T>;    // bad; insufficient

    template<Addable N> auto algo(const N& a, const N& b) // use two numbers
    {
        // ...
        return a + b;
    }

    int x = 7;
    int y = 9;
    auto z = plus(x, y);	// z = 16

    string xx = "7";
    string yy = "9";
    auto zz = plus(xx, yy);	// zz = "79"

아마도 스트링 연결일 수도 있고 어쩌면 실수였을 것이다. 동일하게 마이너스를 정의한다면 가능한 타입에 대해서 완전히 다른 집합을 보여 줄 것이다.
이 `Addable`은 +는 교환법칙(`a + b == b + a`)이 성립해야 한다는 수학적인 규칙에 위배된다.
>Maybe the concatenation was expected. More likely, it was an accident. Defining minus equivalently would give dramatically different sets of accepted types.
This `Addable` violates the mathematical rule that addition is supposed to be commutative: `a + b == b + a`.

##### Note

의미구조를 기술하는 능력은 문법 제약에 반대되는 진짜 컨셉을 정의하는 특징이다.
>The ability to specify a meaningful semantics is a defining characteristic of a true concept, as opposed to a syntactic constraint.

##### Example (using TS concepts)

    template<typename T>
    // The operators +, -, *, and / for a number are assumed to follow the usual mathematical rules
    concept Number = has_plus<T>
                     && has_minus<T>
                     && has_multiply<T>
                     && has_divide<T>;

    template<Number N> auto algo(const N& a, const N& b) // use two numbers
    {
        // ...
        return a + b;
    }

    int x = 7;
    int y = 9;
    auto z = plus(x, y);	// z = 18

    string xx = "7";
    string yy = "9";
    auto zz = plus(xx, yy);	// error: string is not a Number

##### Note

여러 연산자를 가진 컨셉은 한 연산자를 가진 컨셉에 비해 실수로 타입으로 만족될 가능성이 낮을 것이다.
>Concepts with multiple operations have far lower chance of accidentally matching a type than a single-operation concept.

##### Enforcement

* 다른 `concepts` 밖에서 사용되어진 한 연산자를 가진 `concepts`가 있다면 표시한다.
* 한 연산자를 가진 `concepts`를 시뮬레이트하는 `enable_if`가 사용되고 있다면 표시한다.
>* Flag single-operation `concepts` when used outside the definition of other `concepts`.
>* Flag uses of `enable_if` that appears to simulate single-operation `concepts`.

### <a name="Rt-complete"></a> T.21: 완벽한 연산자 집합을 정의하려면 컨셉을 정의하라.
>### <a name="Rt-complete"></a> T.21: Define concepts to define complete sets of operations

##### Reason

상호운용성 개선. 구현이나 유지보수 인력에게 도움이 됨.
>Improves interoperability. Helps implementers and maintainers.

##### Example, bad

    template<typename T> Subtractable = requires(T a, T, b) { a-b; }	// correct syntax?

의미적인 감각이 없다. 적어도 `+`, `-` 정도는 있어야 쓸만하다.
>This makes no semantic sense. You need at least `+` to make `-` meaningful and useful.

Examples of complete sets are

* `Arithmetic`: `+`, `-`, `*`, `/`, `+=`, `-=`, `*=`, `/=`
* `Comparable`: `<`, `>`, `<=`, `>=`, `==`, `!=`

##### Enforcement

???

### <a name="Rt-axiom"></a> T.22: 컨셉을 위해 공리를 기술하라.
>### <a name="Rt-axiom"></a> T.22: Specify axioms for concepts

##### Reason

의미있고 유용한 컨셉은 의미구조적이다.
비공식적인, 반쯤 공식적인, 공식적인 방법으로 의미구조를 표현하는 것은 컨셉을 이해할 수 있게 만든다.
그것을 표현하는 노력은 컨셉 에러를 잡는데 도움을 준다.
의미구조를 기술할 수 있다는 것은 강력한 디자인 도구이다.
>A meaningful/useful concept has a semantic meaning.
Expressing this semantics in a informal, semi-formal, or informal way makes the concept comprehensible to readers and the effort to express it can catch conceptual errors.
Specifying semantics is a powerful design tool.

##### Example

    template<typename T>
        // The operators +, -, *, and / for a number are assumed to follow the usual mathematical rules
        // axiom(T a, T b) { a + b == b + a; a - a == 0; a * (b + c) == a * b + a * c; /*...*/ }
        concept Number = requires(T a, T b) {
            {a + b} -> T;   // the result of a + b is convertible to T
            {a - b} -> T;
            {a * b} -> T;
            {a / b} -> T;
        }

##### Note

이것은 수학적 개념으로 공리이다: 증명이 없이도 자명한 것.
일반적으로 공리는 증명하지 않는다. 게다가 그것을 증명함은 컴파일러의 능력을 넘어서는 것이다.
공리가 일반적이지 않을지도 모른다. 그러나 템플릿 작성자는 실제로 사용하는 모든 입력에 대해서 공리를 가지고 있다고 가정하는게 좋다. (선행조건과 비슷하다)
>This is an axiom in the mathematical sense: something that may be assumed without proof.
In general, axioms are not provable, and when they are the proof is often beyond the capability of a compiler.
An axiom may not be general, but the template writer may assume that it holds for all inputs actually used (similar to a precondition).

##### Note

이 문맥에서 공리는 불리언 연산식이다. 그 예로 [Palo Alto TR](#S-references)를 참조하라.
현재 C++은 공리를 지원하지 않는다.(ISO 컨셉 TS에서도) 그래서 꽤 오래동안 주석으로 대신해야만 한다.
나중에 언어가 지원한다면 '//'를 없애면 된다.
>In this context axioms are Boolean expressions.
See the [Palo Alto TR](#S-references) for examples.
Currently, C++ does not support axioms (even the ISO Concepts TS), so we have to make do with comments for a longish while.
Once language support is available, the `//` in front of the axiom can be removed

##### Note

GSL 컨셉은 잘 정의된 의미구조를 가지고 있다; Palo Alto TR과 the Ranges TS를 참조하라.
>The GSL concepts have well defined semantics; see the Palo Alto TR and the Ranges TS.

##### Exception

현재 개발중인 새 "컨셉" 초기버전은 의미구조를 기술하지 않고 제약조건들을 정의하려고 한다.
좋은 의미구조는 노력과 시간이 필요하다.
불완전한 제약조건이라도 쓸만하다:
>Early versions of a new "concept" still under development will often just define simple sets of constraints without a well-specified semantics.
Finding good semantics can take effort and time.
An incomplete set of constraints can still be very useful:

    ??? binary tree: rotate(), ...

불완전하거나 의미구조가 잘 정의되지 않은 컨셉이라도 쓸만하다.
그러나 안정화됐다고 생각해서는 안된다. 새로운 사용예제를 발견하면 불완전한 컨셉이 개선될 것이다.
>A "concept" that is incomplete or without a well-specified semantics can still be useful.
However, it should not be assumed to be stable. Each new use case may require such an incomplete concepts to be improved.

##### Enforcement

* 컨셉 정의 주석 내에 "axiom" 단어를 찾아라.
>* Look for the word "axiom" in concept definition comments

### <a name="Rt-refine"></a> T.23: 정제된 컨셉과 새로운 사용패턴을 추가함으로써 더 일반화된 경우를 구별하라.
>### <a name="Rt-refine"></a> T.23: Differentiate a refined concept from its more general case by adding new use patterns.

##### Reason

그렇지 않으면 컴파일러가 자동으로 구분할 수 없다.
>Otherwise they cannot be distinguished automatically by the compiler.

##### Example

    template<typename I>
    concept bool Input_iterator = requires (I iter) { ++iter; };

    template<typename I>
    concept bool Fwd_iter = Input_iter<I> && requires (I iter) { iter++; }

컴파일러는 요구된 연산들에 기반해서 정제를 결정할 수 있다.
2개의 컨셉이 요구사항이 정확하게 동일하다면 그들은 논리적으로 동일하다.(정제가 없다.)
>The compiler can determine refinement based on the sets of required operations.
If two concepts have exactly the same requirements, they are logically equivalent (there is no refinement).

컨셉을 끌어들이기 위한 특별한 선언이 필요없기에 이것은 타입 구현자들의 짐을 좀 줄여준다.
>This also decreases the burden on implementers of these types since
they do not need any special declarations to "hook into the concept".

##### Enforcement

이미 본 다른 컨셉과 요구사항이 정확하게 일치하는 컨셉이 있다면 표시한다.
차이를 분명하게 보고 싶다면 [T.24](#Rt-tag)를 참조하라.
>* Flag a concept that has exactly the same requirements as another already-seen concept (neither is more refined). To disambiguate them, see [T.24](#Rt-tag).

### <a name="Rt-tag"></a> T.24: 의미구조만 다른 컨셉을 구별하기 위해서는 테그 클래스나 타입특성을 사용하라.
>### <a name="Rt-tag"></a> T.24: Use tag classes or traits to differentiate concepts that differ only in semantics.

##### Reason

동일한 문법을 요구하지만 의미구조가 다르다면 두 컨셉은 프로그래머가 구별시키지 않는다면 애매모호함을 야기한다.
>Two concepts requiring the same syntax but having different semantics leads to ambiguity unless the programmer differentiates them.

##### Example

    template<typename I>    // iterator providing random access
    concept bool RA_iter = ...;

    template<typename I>    // iterator providing random access to contiguous data
    concept bool Contiguous_iter =
        RA_iter<I> && is_contiguous<I>::value;  // ??? why not is_contiguous<I>() or is_contiguous_v<I>?

프로그래머가 `is_contiguous`를 적절하게 정의해야 한다.
>The programmer (in a library) must define `is_contiguous` (a trait) appropriately.

##### Note

트레잇은 트레잇 클래스나 타입 트레잇이다.
이들은 사용자가 정의했거나 표준라이브러리 중의 하나이다.
표준라이브러리를 선호하라.
>Traits can be trait classes or type traits.
These can be user-defined or standard-library ones.
Prefer the standard-library ones.

##### Enforcement

* 컴파일러는 동일한 컨셉을 애매하게 사용하는 것을 표시한다.
* 동일한 컨셉을 정의한다면 표시한다.

>* The compiler flags ambiguous use of identical concepts.
>* Flag the definition of identical concepts.

### <a name="Rt-not"></a> T.25: 정반대인 제약조건은 피하라.
>### <a name="Rt-not"></a> T.25: Avoid negating constraints.

##### Reason

단순명료, 유지보수가 좋음.
반대를 나타내려고 쓴 보완적인 요구사항이 있는 함수는 불안정하다.
>Clarity. Maintainability.
Functions with complementary requirements expressed using negation are brittle.

##### Example

초기에 사람들은 보완적인 요구사항을 가진 함수를 정의하려고 시도할 것이다.
>Initially, people will try to define functions with complementary requirements:

    template<typename T>
        requires !C<T>    // bad
    void f();

    template<typename T>
        requires C<T>
    void f();

아래가 더 낫다:
>This is better:

    template<typename T>	// general template
        void f();

    template<typename T>	// 컨셉으로 특수화
        requires C<T>
    void f();

`C<T>`가 만족되지 않을 때만 컴파일러는 제한조건이 없는 템플릿을 선택할 것이다.
제약조건이 없는`f()`를 정의하고 싶지 않다면 그냥 없애라.
>The compiler will choose the unconstrained template only when `C<T>` is
unsatisfied. If you do not want to (or cannot) define an unconstrained
version of `f()`, then delete it.

    template<typename T>
    void f() = delete;

컴파일러는 오버로드된 함수를 선택할 것이고 적당한 에러를 낼 것이다.
>The compiler will select the overload and emit an appropriate error.

##### Enforcement

* `C<T>`, `!C<T>`를 같이 가진 함수들이 있다면 표시한다.
* 정반대 제약조건이 있다면 모두 표시한다.

>* Flag pairs of functions with `C<T>` and `!C<T>` constraints
* Flag all constraint negation

### <a name="Rt-use"></a> T.27: 단순화된 문법보다는 사용패턴으로 컨셉을 정의하는 것을 선호하라.
>### <a name="Rt-use"></a> T.27: Prefer to define concepts in terms of use-patterns rather than simple syntax

##### Reason

정의가 더 읽기 쉽고 사용자가 작성하고 싶은 것과 직접적으로 일치한다.
형변환은 고려해야 한다. 모든 타입특성의 이름을 기억할 필요가 없다.
>The definition is more readable and corresponds directly to what a user has to write.
Conversions are taken into account. You don't have to remember the names of all the type traits.

##### Example

    ???

##### Enforcement

???

## <a name="SS-temp-interface"></a> 템플릿 인터페이스
>## <a name="SS-temp-interface"></a> Template interfaces

???

### <a name="Rt-fo"></a> T.40: 알고리즘에 연산을 전달하기 위해 함수객체를 사용하라.
>### <a name="Rt-fo"></a> T.40: Use function objects to pass operations to algorithms

##### Reason

함수객체는 "단순"" 함수 포인터에 비해 인터페이스를 통해 많은 정보를 전달할 수 있다.
보통은 함수객체를 전달하는 것이 함수포인터에 비해 더 나은 성능을 보인다.
>Function objects can carry more information through an interface than a "plain" pointer to function.
In general, passing function objects give better performance than passing pointers to functions.

##### Example

    bool greater(double x, double y) { return x>y; }
    sort(v, greater);                                   // 함수 포인터
    sort(v, [](double x, double y) { return x>y; });    // 함수객체
    sort(v, greater<>);                                 // 함수객체

    bool greater_than_7(double x) { return x>7; }
    auto x = find_if(v, greater_than_7);				// 함수포인터: 융통성없는
    auto y = find_if(v, [](double x) { return x>7; });	// 함수객체: 데이터를 더 전달한다.
    auto y = find_if(v, Greater_than<double>(7));		// 함수객체: 데이터를 더 전달한다.

    ??? 람다는 오토 매개변수를 위해 변경하는데 반대를 외치고 있다.

##### Note

람다는 함수객체를 생성한다.
>Lambdas generate function objects.

##### Note

성능문제는 컴파일러, 최적화 기술에 달린 것이다.
>The performance argument depends on compiler and optimizer technology.

##### Enforcement

* 함수 템플릿 인자에 포인터가 있다면 표시한다.
* 템플릿(위정 위험요소)에 인자로 전달하는 함수 포인터가 있다면 표시한다.

>* Flag pointer to function template arguments.
>* Flag pointers to functions passed as arguments to a template (risk of false positives).

### <a name="Rt-operations"></a> T.41: 컨셉에 대해서 완전한 연산집합을 요구하라.
>### <a name="Rt-operations"></a> T.41: Require complete sets of operations for a concept

##### Reason

이해가 쉬움.
개선된 상호운용성.
템플릿 구현자를 위한 유연성
>Ease of comprehension.
Improved interoperability.
Flexibility for template implementers.

##### Note

여기서 이슈는 템플릿 인자를 위한 최소한의 연산집합을 요구할지에 대한 것이다. (`!=`이 아니라 `==`, `+=`이 아니라 `+`)
이 규칙은 컨셉이 수학적으로 일관성있는 연산집합을 반영해야 한다는 의견을 지원한다.
>The issue here is whether to require the minimal set of operations for a template argument
(e.g., `==` but not `!=` or `+` but not `+=`).
The rule supports the view that a concept should reflect a (mathematically) coherent set of operations.

##### Example

    ???

##### Enforcement

???

### <a name="Rt-alias"></a> T.42: 상세한 구현내용을 감추고 명칭을 단순화하기 위해 템플릿 별칭을 사용하라.
>### <a name="Rt-alias"></a> T.42: Use template aliases to simplify notation and hide implementation details

##### Reason

개선된 가독성. 구현 내용 숨김.
템플릿 별칭이 타입을 계산하기 위한 특성의 많은 사용을 치환한다.
그것은 타입특성을 숨기는데 사용할 수도 있다.
>Improved readability. Implementation hiding. Note that template aliases replace many uses of traits to compute a type. They can also be used to wrap a trait.

##### Example

    template<typename T, size_t N>
    class matrix {
        // ...
        using Iterator = typename std::vector<T>::iterator;
        // ...
    };

`Matrix` 사용자들이 그 요소가 `vector`에 저장되는 점을 알 필요가 없게 한다. 반복적으로 `typename std::vector<T>::`를 타이핑하는 것을 줄여준다.
>This saves the user of `Matrix` from having to know that its elements are stored in a `vector` and also saves the user from repeatedly typing `typename std::vector<T>::`.

##### Example

    template<typename T>
    using Value_type<T> = container_traits<T>::value_type;

이것은 `Value_type`을 쓰는 사용자에게 `value_type`을 구현하는 테크닉을 알 필요가 없게 한다.
>This saves the user of `Value_type` from having to know the technique used to implement `value_type`s.

##### Enforcement

* `using`으로 선언이외에 중의성 제거용으로 `typename`을 사용한다면 표시한다.
>* Flag use of `typename` as a disambiguator outside `using` declarations.
>* ???

### <a name="Rt-using"></a> T.43: 별칭을 정의하기 위해 `typedef` 대신에 `using`을 선호하라.
>### <a name="Rt-using"></a> T.43: Prefer `using` over `typedef` for defining aliases

##### Reason

가독성 개선:  `using`을 사용하면 새 명칭은 선언 시에 뒤쪽 어딘가에 숨어 있기보다는 제일 앞에 나온다.
일반화: `using`은 템플릿 별칭으로 사용할 수 있다. 반면 `typedef`는 템플릿으로 쓸 수 없다.
일률성: `using`은 구문상 `auto`와 비슷하다.
>Improved readability: With `using`, the new name comes first rather than being embedded somewhere in a declaration.
Generality: `using` can be used for template aliases, whereas `typedef`s can't easily be templates.
Uniformity: `using` is syntactically similar to `auto`.

##### Example

    typedef int (*PFI)(int);	// OK, but convoluted

    using PFI2 = int (*)(int);	// OK, preferred

    template<typename T>
    typedef int (*PFT)(T);		// error

    template<typename T>
    using PFT2 = int (*)(T);	// OK

##### Enforcement

* `typedef`를 사용하고 있다면 표시한다. 그것은 아주 많은 'hits'를 줄 것이다. (? - 뭐지?)

>* Flag uses of `typedef`. This will give a lot of "hits" :-(

### <a name="Rt-deduce"></a> T.44: 클래스 템플릿 인자 타입을 추정하기 위해 함수 템플릿을 사용하라.
>### <a name="Rt-deduce"></a> T.44: Use function templates to deduce class template argument types (where feasible)

##### Reason

템플릿 인자 타입을 명시적으로 쓴다면 지루해질 수 있고 불필요하게 길어질 수 있다.
>Writing the template argument types explicitly can be tedious and unnecessarily verbose.

##### Example

    tuple<int, string, double> t1 = {1, "Hamlet", 3.14};	// 명시적인 타입.
    auto t2 = make_tuple(1, "Ophelia"s, 3.14);			// better; 추정된 타입.

스트링이 C스타일이 아니라 `std::string`이라는 것을 보장하기 위해 `s`어미를 사용한 것에 주목하라.
>Note the use of the `s` suffix to ensure that the string is a `std::string`, rather than a C-style string.

##### Note

당신이 `make_T` 함수를 작성하듯이 컴파일러도 그렇게 할 수 있다. 따라서 `make_T` 함수는 미래에는 불필요해질 것이다.
>Since you can trivially write a `make_T` function, so could the compiler. Thus, `make_T` functions may become redundant in the future.

##### Exception

때로는 템플릿 인자를 추정할 좋은 방법이 없어서 인자를 명시적으로 기술하고 싶을 것이다.
>Sometimes there isn't a good way of getting the template arguments deduced and sometimes, you want to specify the arguments explicitly:

    vector<double> v = { 1, 2, 3, 7.9, 15.99 };
    list<Record*> lst;

##### Enforcement

명시적으로 특수화된 타입이 정확하게 인자에 사용된 타입과 일치하도록 사용하고 있다면 표시한다.
>Flag uses where an explicitly specialized type exactly matches the types of the arguments used.

### <a name="Rt-regular"></a> T.46: 템플릿인자를 `Regular`, `SemiRegular`하도록 요청하라.
>### <a name="Rt-regular"></a> T.46: Require template arguments to be at least `Regular` or `SemiRegular`

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Rt-visible"></a> T.47: 동일한 명칭을 가진 잘 보이는 제한조건이 없는 템플릿을 피하라. (? - 내용을 이해할 수 없다.)
>### <a name="Rt-visible"></a> T.47: Avoid highly visible unconstrained templates with common names

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Rt-concept-def"></a> T.48: 컴파일러가 컨셉을 지원하지 않는다면 `enable_if`문을 훼이크로 사용하라.
>### <a name="Rt-concept-def"></a> T.48: If your compiler does not support concepts, fake them with `enable_if`

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Rt-erasure"></a> T.49: 가능한 곳이면 어디든 타입제거를 없애라.
>### <a name="Rt-erasure"></a> T.49: Where possible, avoid type-erasure

##### Reason

타입제거는 분리된 컴파일 범위로 인해 타입 정보가 없어지므로 추가적으로 간접효과을 초래한다.
>Type erasure incurs an extra level of indirection by hiding type information behind a separate compilation boundary.

##### Example

    ???

**Exceptions**: `std::function`에 대해서는 타입제거는 적절할 수 있다.
>**Exceptions**: Type erasure is sometimes appropriate, such as for `std::function`.

##### Enforcement

???

### <a name="Rt-unconstrained-adl"></a> T.50: 타입처럼 동일한 이름공간 내에 제약조건없이 템플릿을 작성하는 것을 피하라.
>### <a name="Rt-unconstrained-adl"></a> T.50: Avoid writing an unconstrained template in the same namespace as a type

##### Reason

ADL(Argument-Dependent Lookup)은 생각지도 못한 템플릿도 찾아낼 것이다.
>ADL will find the template even when you think it shouldn't.

##### Example

    ???

##### Note

이 규칙은 별 필요가 없어야 정상이다. C++위원회는 ADL를 어떻게 고칠지 의견이 일치하지 않지만,
적어도 제약조건없는 템플릿을 고려하지 않는 것이 실질적인 많은 문제를 해결하고 이 규칙이 필요없게 만들 수 있을 것이다.
>This rule should not be necessary; the committee cannot agree on how to fix ADL, but at least making it not consider unconstrained templates would solve many of the actual problems and remove the need for this rule.

##### Enforcement

??? 불행히도 이 규칙은 많은 위정(?)을 갖게 될 것이다;
많은 제약조건없는 템플릿과 타입을 `std` 이름공간 내에 둔 표준 라이브러리도 이 규칙을 위배하고 있다.
>??? unfortunately this will get many false positives; the standard library violates this widely, by putting many unconstrained templates and types into the single namespace `std`

## <a name="SS=temp-def"></a> TCP.def: 템플릿 정의하기
>## <a name="SS=temp-def"></a> TCP.def: Template definitions

???

### <a name="Rt-depend"></a> T.60: 템플릿의 문맥 의존도를 최소화하라.
>### <a name="Rt-depend"></a> T.60: Minimize a template's context dependencies

##### Reason

이해가 쉬움. 예상치 못한 의존성에 에러 발생 최소화. 툴 작성이 쉬움.
>Eases understanding. Minimizes errors from unexpected dependencies. Eases tool creation.

##### Example

    ???

##### Note

인자에만 템플릿을 동작하게 하는 것이 의존도를 최소한으로 줄일 수 있는 한가지 방법인데 일반적으로는 다루기 힘들다.
예를 들어 한 알고리즘은 다른 알고리즘을 사용하기 마련이다.
>Having a template operate only on its arguments would be one way of reducing the number of dependencies to a minimum, but that would generally be unmanageable. For example, an algorithm usually uses other algorithms.

##### Enforcement

??? 뺑끼(?)
>??? Tricky

### <a name="Rt-scary"></a> T.61: 멤버를 과도하게 매개변수화하지 마라. (SCARY)
>### <a name="Rt-scary"></a> T.61: Do not over-parameterize members (SCARY)

##### Reason

템플릿 매개변수로 쓰이지 않는 멤버는 구체적인 템플릿 매개변수를 제외하고 사용될 수 없다.
이것은 사용을 제한하고 보통 코드 사이즈를 증가시킨다.
>A member that does not depend on a template parameter cannot be used except for a specific template argument.
This limits use and typically increases code size.

##### Example, bad

    template<typename T, typename A = std::allocator{}>
        // requires Regular<T> && Allocator<A>
    class List {
    public:
        struct Link {	// A는 쓰이지 않는다.
            T elem;
            T* pre;
            T* suc;
        };

        using iterator = Link*;

        iterator first() const { return head; }

        // ...
    private:
        Node* head;
    };

    List<int> lst1;
    List<int, my_allocator> lst2;

    ???

충분히 순진해 보인다. 그러나 ???
>This looks innocent enough, but ???

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
        Node* head;
    };

    List<int> lst1;
    List<int, my_allocator> lst2;

    ???

##### Enforcement

* 모든 템플릿 매개변수에 쓰이지 않는 멤버 타입이 있다면 표시한다.
* 모든 템플릿 매개변수에 쓰이지 않는 멤버 함수가 있다면 표시한다.

>* Flag member types that do not depend on every template argument
* Flag member functions that do not depend on every template argument

### <a name="Rt-nondependent"></a> T.62: 템플릿이 아닌 기본클래스에 독립적인 템플릿 멤버를 둬라.
>### <a name="Rt-nondependent"></a> T.62: Place non-dependent template members in a non-templated base class

##### Reason

 ???

##### Example

    template<typename T>
    class Foo {
    public:
        enum { v1, v2 };
        // ...
    };

???

    struct Foo_base {
        enum { v1, v2 };
        // ...
    };

    template<typename T>
    class Foo : public Foo_base {
    public:
        // ...
    };

##### Note

이 규칙의 더 일반적인 버전은 다음과 같다.
"템플릿 클래스 멤버가 M 중에서 N 템플릿 매개변수만 쓰고 있다면, N 매개변수만 기본클래스 내에 넣어라."
N == 1에 대해서, [T.41](#Rt-scary)에서처럼 둘러싼 범위내에서 기본클래스에 넣을지 어떨지 결정해야 한다. (? - 어렵다.)
>A more general version of this rule would be
"If a template class member depends on only N template parameters out of M, place it in a base class with only N parameters."
For N == 1, we have a choice of a base class of a class in the surrounding scope as in [T.41](#Rt-scary).

??? 상수는 어떤가? 정적 클래스?
>??? What about constants? class statics?

##### Enforcement

* 표시한다. ???

>* Flag ???

### <a name="Rt-specialization"></a> T.64: 클래스 템플릿을 다르게 구현하기 위해 특수화를 사용하라.
>### <a name="Rt-specialization"></a> T.64: Use specialization to provide alternative implementations of class templates

##### Reason

템플릿은 일반화된 인터페이스를 정의한다.
특수화는 일반화된 인터페이스 구현을 위한 강력한 메카니즘을 제공한다.
>A template defines a general interface.
Specialization offers a powerful mechanism for providing alternative implementations of that interface.

##### Example

    ??? 문자열 특수화 (==)
    >??? string specialization (==)

    ??? 표현 특수화 ?
    >??? representation specialization ?

##### Note

???

##### Enforcement

???

### <a name="Rt-tag-dispatch"></a> T.65: 함수 구현을 제공하기 위해 태그 디스패치를 사용하라.
>### <a name="Rt-tag-dispatch"></a> T.65: Use tag dispatch to provide alternative implementations of a function

##### Reason

템플릿 일반화된 인터페이스를 정의한다. ???
>A template defines a general interface. ???

##### Example

    ??? 인자로 적당하다면 `memmove`호출로 컴파일하는 `std::copy`같이 어떻게 알고리즘을 얻을지에 대한 것이다.
>    ??? that's how we get algorithms like `std::copy` which compiles into a `memmove` call if appropriate for the arguments.

##### Note

`concept`이 가능하게 된다면 그런 대안은 바로 구별될 수 있을 것이다.
>When `concept`s become available such alternatives can be distinguished directly.

##### Enforcement

???

### <a name="Rt-enable_if"></a> T.66: 함수 정의할 때 `enable_if`를 옵션선택용으로 사용하라.
>### <a name="Rt-enable_if"></a> T.66: Use selection using `enable_if` to optionally define a function

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Rt-customization"></a> T.69: 변경지점으로 쓸 의도가 아니라면 템플릿 내에 비적격인 멤버이외의 함수를 호출하지 마라.
>### <a name="Rt-customization"></a> T.69: Inside a template, don't make an unqualified nonmember function call unless you intend it to be a customization point

##### Reason

의도적으로 유연성을 제공하기 위해 그리고 실수로 환경 변화를 막기 위해.
>To provide only intended flexibility, and avoid accidental environmental changes.

`t`값으로 템플릿 타입 매개변수로 된 `helper(t)`함수를 호출하려고 한다면, `::detail` 이름공간 내에 함수를 두고 `detail::helper(t)`로 호출 자격을 얻어라.
그렇지 않으면 그 호출은 `t` 타입의 이름공간 내에 있는 `helper` 함수가 호출될 수 있는 변형지점(?)이 될 수 있다.
아래 두번째 경우가 될 것이고, [의도적이지 않지만 제약받지 않는 `t`타입과 같은 이름공간에 있는 함수 템플릿을 호출하기](#Rt-unconstrained-adl)같은 문제를 야기할 것이다.
(? - 어렵다.)
>If you intend to call your own helper function `helper(t)` with a value `t` that depends on a template type parameter, put it in a `::detail` namespace and qualify the call as `detail::helper(t);`. Otherwise the call becomes a customization point where any function `helper` in the namespace of `t`'s type can be invoked instead -- falling into the second option below, and resulting in problems like [unintentionally invoking unconstrained function templates of that name that happen to be in the same namespace as `t`'s type](#Rt-unconstrained-adl).

템플릿을 변형하기 위해 코드를 호출하는 세가지 주된 방법이 있다.
>There are three major ways to let calling code customize a template.

* 멤버함수를 호출한다. 호출자는 그런 이름의 멤버함수를 가진 타입을 제공할 수 있다.

>* Call a member function. Callers can provide any type with such a named member function.

        template<class T>
        void test(T t)
        {
            t.f();    // require T to provide f()
        }

* 검증없이 비멤버함수를 호출한다. 호출자는 문맥 내에, 또는 이름공간 내에 있는 그런 함수를 가진 타입을 제공할 수 있다.

>* Call a nonmember function without qualification. Callers can provide any type for which there is such a function available in the caller's context or in the namespace of the type.

        template<class T>
        void test(T t)
        {
            f(t);     // require f(/*T*/) be available in caller's scope or in T's namespace
        }

* 타입특성(trait)을 호출한다. - 보통 타입을 결정하는 타입 별칭, 값을 결정하는 `constexpr` 함수, 드물게는 사용자 타입에 대해서 특수화된 전통적인 타입특성 템플릿.

>* Invoke a "trait" -- usually a type alias to compute a type, or a `constexpr` function to compute a value, or in rarer cases a traditional traits template to be specialized on the user's type.

        template<class T>
        void test(T t)
        {
            test_traits<T>::f(t);    // require customizing test_traits<> to get non-default functions/types
            test_traits<T>::value_type x;
        }

##### Enforcement

* 템플릿 이름공간 내에 같은 이름을 가진 비멤버함수가 있을 때  관련된 타입 변수를 넘기는 비멤버함수를 검증도 하지 않고 호출하고 있다면 표시한다.

>* In a template, flag an unqualified call to a nonmember function that passes a variable of dependent type when there is a nonmember function of the same name in the template's namespace.

## <a name="SS-temp-hier"></a> T.temp-hier: 템플릿과 계층 규칙:
>## <a name="SS-temp-hier"></a> T.temp-hier: Template and hierarchy rules:

템플릿은 객체지향 프로그래밍으로써 일반화 프로그래밍과 클래스 계층구조를 지원하는 C++의 기본이다.
이 두개 언어 기능은 합해서 효과적으로 사용할 수 있다. 몇몇 디자인 함정은 피해야 한다.
>Templates are the backbone of C++'s support for generic programming and class hierarchies the backbone of its support
for object-oriented programming.
The two language mechanisms can be use effectively in combination, but a few design pitfalls must be avoided.

### <a name="Rt-hier"></a> T.80: 클래스 계층구조를 순진하게 템플릿화하지 마라.
>### <a name="Rt-hier"></a> T.80: Do not naively templatize a class hierarchy

##### Reason

함수도 많고 가상함수도 많은 클래스 계층구조를 템플릿화하면 코드가 폭발적으로 증가할 것이다.
>Templatizing a class hierarchy that has many functions, especially many virtual functions, can lead to code bloat.

##### Example, bad

    template<typename T>
    struct Container {			// an interface
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

    vector<int> vi;
    vector<string> vs;

컨테이너의 멤버함수로 `sort`를 정의하는 건 좀 바보같은 생각이다.
들어 본 적이 없는건 아니지만 하지 말아야 할 좋은 본보기가 될 것이다.
>It is probably a dumb idea to define a `sort` as a member function of a container, but it is not unheard of and it makes a good example of what not to do.

컴파일러가 코드를 생성해야 하는데 `vector<int>::sort()`가 호출되는지 알 수가 없다. `vector<string>::sort()`에 대해서도 비슷하다.
두 함수가 호출하지 않으면 코드만 커진 꼴이다.
십여개의 멤버 함수와 십여개의 파생클래스를 가진 클래스 계층구조가 다양하게 인스턴스화되면 무엇을 할지 상상해보라.
>Given this, the compiler cannot know if `vector<int>::sort()` is called, so it must generate code for it.
Similar for `vector<string>::sort()`.
Unless those two functions are called that's code bloat.
Imagine what this would do to a class hierarchy with dozens of member functions and dozens of derived classes with many instantiations.

##### Note

많은 경우에 기본클래스를 매개변수화하지 않는다면 안정적인 인터페이스를 제공해 줄 수 있다; [Rule](#Rt-abi)를 참고하라.
>In many cases you can provide a stable interface by not parameterizing a base; see [Rule](#Rt-abi).

##### Enforcement

* 템플릿 인자에 의존하는 가상함수가 있다면 표시한다. ??? 긍정 오류.

>* Flag virtual functions that depend on a template argument. ??? False positives

### <a name="Rt-array"></a> T.81: 계층과 배열을 섞지 마라.
>### <a name="Rt-array"></a> T.81: Do not mix hierarchies and arrays

##### Reason

파생 클래스 배열은 기본클래스에 대한 포인터로 "decay"될 수 있는데 처참한 결과가 숨겨져 있다.
>An array of derived classes can implicitly "decay" to a pointer to a base class with potential disastrous results.

##### Example

`Apple`, `Pear`가 `Fruit`의 종류라고 가정해보자.
>Assume that `Apple` and `Pear` are two kinds of `Fruit`s.

    void maul(Fruit* p)
    {
        *p = Pear{};	// put a Pear into *p
        p[1] = Pear{};	// put a Pear into p[1]
    }

    Apple aa [] = { an_apple, another_apple };	// aa contains Apples (obviously!)

    maul(aa);
    Apple& a0 = &aa[0];	// Pear?
    Apple& a1 = &aa[1];	// Pear?

아마도 `aa[0]`는 `Pear`일 것이다. (형변환을 사용하지 않아도.)
`sizeof(Apple) != sizeof(Pear)`이므로 `aa[1]`에 접근하면 배열에서 객체의 올바른 시작위치로 정렬될 수 없을 것이다.
타입 위반이 되서 메모리값이 망가지게 되니 절대로 그런류의 코드를 작성하지 마라.
>Probably, `aa[0]` will be a `Pear` (without the use of a cast!).
If `sizeof(Apple) != sizeof(Pear)` the access to `aa[1]` will not be aligned to the proper start of an object in the array.
We have a type violation and possibly (probably) a memory corruption.
Never write such code.

`maul()`이 개별 객체 [Rule](#???)에 대해 `T*` 포인트를 위반한다는 것을 기억하라.
>Note that `maul()` violates the a `T*` points to an individual object [Rule](#???).

**Alternative**: 적당한 컨테이너를 사용하라:
>**Alternative**: Use a proper container:

    void maul2(Fruit* p)
    {
        *p = Pear{};	// put a Pear into *p
    }

    vector<Apple> aa = { an_apple, another_apple };	// aa contains Apples (obviously!)

    maul2(aa);		// error: vector<Apple>는 Fruit*로 변환할 수 없다.
    maul2(&aa[0]);	// you asked for it

    Apple& a0 = &aa[0];	// a Pear?

`maul2()` 내에 있는 대입이 슬라이스 안하기 [Rule](#???)을 위반한다는 것을 기억하라.
>Note that the assignment in `maul2()` violated the no-slicing [Rule](#???).

##### Enforcement

* 이 공포스러운 문제를 찾아라.

>* Detect this horror!

### <a name="Rt-linear"></a> T.82: 가상함수가 싫다면 계층구조를 직선화하라.
>### <a name="Rt-linear"></a> T.82: Linearize a hierarchy when virtual functions are undesirable

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Rt-virtual"></a> T.83: 멤버함수 템플릿을 가상으로 정의하지 마라.
>### <a name="Rt-virtual"></a> T.83: Do not declare a member function template virtual

##### Reason

C++이 지원하지 않는다.
vtbl을 링크타임까지 생성할 수 없기 때문에 보통은 동적링크로 구현해야 한다.
>C++ does not support that.
If it did, vtbls could not be generated until link time.
And in general, implementations must deal with dynamic linking.

##### Example, don't

    class Shape {
        // ...
        template<class T>
        virtual bool intersect(T* p);	// error: 템플릿은 가상적일 수 없다.
    };

##### Note

사람들이 계속 물어보는 관계로 규칙이 필요하다.
>We need a rule because people keep asking about this

##### Alternative

이중 디스패치, 방문자가 어떤 함수를 호출하는지 계산한다.
>Double dispatch, visitors, calculate which function to call

##### Enforcement

컴파일러가 다룬다.
>The compiler handles that.

### <a name="Rt-abi"></a> T.84: ABI용 인터페이스는 비템플릿으로 핵심부분을 구현하라.
>### <a name="Rt-abi"></a> T.84: Use a non-template core implementation to provide an ABI-stable interface

##### Reason

코드 안정성을 개선한다. 코드가 부풀어 오르는 걸 피한다.
>Improve stability of code. Avoids code bloat.

##### Example

기본클래스가 있다면:
>It could be a base class:

    struct Link_base {		// stable
        Link* suc;
        Link* pre;
    };

    template<typename T>	// 타입 안정성을 위한 템플릿 레퍼
    struct Link : Link_base {
        T val;
    };

    struct List_base {
        Link_base* first;	// first element (if any)
        int sz;				// number of elements
        void add_front(Link_base* p);
        // ...
    };

    template<typename T>
    class List : List_base {
    public:
        void put_front(const T& e) { add_front(new Link<T>{e}); }	// Link_base의 암시적 형변환
        T& front() { static_cast<Link<T>*>(first).val; }			// Link<T> 명시적 역형변환
        // ...
    };

    List<int> li;
    List<string> ls;

여기는 `List`의 요소를 연결하고 해제하는 함수 복사본이 하나 있다.
`Link`, `List` 클래스는 타입 조작만 한다.
>Now there is only one copy of the operations linking and unlinking elements of a `List`.
The `Link` and `List` classes does nothing but type manipulation.

별도의 기본 타입을 사용하는 대신에 일반적으로는 `void`, `void*`에 대해서 특수화하고 핵심 `void` 구현에서 안전하게 `T`로 형변환하도록 템플릿을 가지도록 한다. (? - 어렵다)
>Instead of using a separate "base" type, another common technique is to specialize for `void` or `void*` and have the general template for `T` be just the safely-encapsulated casts to and from the core `void` implementation.

**Alternative**: [PIMPL](#???)을 구현하라.
>**Alternative**: Use a [PIMPL](#???) implementation.

##### Enforcement

???

## <a name="SS-variadic"></a> T.var: 가변인자 템플릿 규칙
>## <a name="SS-variadic"></a> T.var: Variadic template rules

???

### <a name="Rt-variadic"></a> T.100: 타입별 가변 인자가 필요한 함수가 있다면 가변인자 템플릿을 사용하라.
>### <a name="Rt-variadic"></a> T.100: Use variadic templates when you need a function that takes a variable number of arguments of a variety of types

##### Reason

가변인자 템플릿이 가장 종합적인 방법이고 효과적이고 타입적으로 안전하다. C의 가변인자를 사용하지 마라.
>Variadic templates is the most general mechanism for that, and is both efficient and type-safe. Don't use C varargs.

##### Example

    ??? printf

##### Enforcement

* 코드에 `va_arg`이 있다면 표시한다.

>    * Flag uses of `va_arg` in user code.

### <a name="Rt-variadic-pass"></a> T.101: ??? 가변인자 템플릿에 인자를 처리하는 방법 ???
>### <a name="Rt-variadic-pass"></a> T.101: ??? How to pass arguments to a variadic template ???

##### Reason

 ???

##### Example

    ??? beware of move-only and reference arguments

##### Enforcement

???

### <a name="Rt-variadic-process"></a> T.102: 가변인자 템플릿에 인자를 처리하는 방법
>### <a name="Rt-variadic-process"></a> T.102: How to process arguments to a variadic template

##### Reason

 ???

##### Example

    ??? forwarding, type checking, references

##### Enforcement

???

### <a name="Rt-variadic-not"></a> T.103: 동형 인자목록으로 가변인자 템플릿을 사용하지 마라.
>### <a name="Rt-variadic-not"></a> T.103: Don't use variadic templates for homogeneous argument lists

##### Reason

`initializer_list`으로 같은 타입 인자열을 정의할 수 있다.
>There are more precise ways of specifying a homogeneous sequence, such as an `initializer_list`.

##### Example

    ???

##### Enforcement

???

## <a name="SS-meta"></a> T.meta: 템플릿 메타프로그래밍 (TMP)
>## <a name="SS-meta"></a> T.meta: Template metaprogramming (TMP)

템플릿은 컴파일타임 프로그래밍을 위한 종합적인 방법을 제공한다.
>Templates provide a general mechanism for compile-time programming.

메타프로그래밍은 하나 이상의 입력이나 결과 자체가 타입인 프로그래밍이다.
템플릿은 컴파일타임에 튜링(모듈로 메모리 용량)문제에 덕타이핑을 제공한다. (? - 용어가 어려음.)
필요한 문법과 기술은 아주 끔찍하다.
>Metaprogramming is programming where at least one input or one result is a type.
Templates offer Turing-complete (modulo memory capacity) duck typing at compile time.
The syntax and techniques needed are pretty horrendous.

### <a name="Rt-metameta"></a> T.120: 꼭 필요할 때만 템플릿 메타프로그래밍을 사용하라.
>### <a name="Rt-metameta"></a> T.120: Use template metaprogramming only when you really need to

##### Reason

템플릿 메타프로그래밍은 올바르게 쓰기가 어렵고, 컴파일 속도를 느리게 하고, 유지보수를 어렵게 한다.
그러나 템플릿 메타프로그래밍이 전문가 수준의 어셈블리 코드보다 성능이 더 좋은 예들이 있다.
게다가 런타임 코드보다 핵심사상을 더 잘 표현하는 실제 예들도 있다.
예를 들어 컴파일 타임에 AST(Abstract Syntax Tree)를 조작해야 한다면 C++에서는 다른 방법이 없다.
>Template metaprogramming is hard to get right, slows down compilation, and is often very hard to maintain.
However, there are real-world examples where template metaprogramming provides better performance that any alternative short of expert-level assembly code.
Also, there are real-world examples where template metaprogramming expresses the fundamental ideas better than run-time code.
For example, if you really need AST manipulation at compile time (e.g., for optional matrix operation folding) there may be no other way in C++.

##### Example, bad

    ???

##### Example, bad

    enable_if

대신에 컨셉을 사용하라. [언어가 지원하지 않는 컨셉을 에뮬레이트하는 방법](#Rt-emulate)을 참고하라.
>Instead, use concepts. But see [How to emulate concepts if you don't have language support](#Rt-emulate).

##### Example

    ??? good

**Alternative**: 타입이 아니라 값이라면 [`constexpr` function](#Rt-fct)를 사용하라.
>**Alternative**: If the result is a value, rather than a type, use a [`constexpr` function](#Rt-fct).

##### Note

템플릿 메타프로그래밍을 메크로로 대신하고 싶다고 느낀다면 너무 나간거다.
>If you feel the need to hide your template metaprogramming in macros, you have probably gone too far.

### <a name="Rt-emulate"></a> T.121: 주로 컨셉을 에뮬레이트하기 위해 템플릿 메타프로그래밍을 사용하라.
>### <a name="Rt-emulate"></a> T.121: Use template metaprogramming primarily to emulate concepts

##### Reason

컨셉 개념이 널리 사용될때까지 TMP를 사용해서 에뮬레이트해야 할 것이다.
컨셉이 필요한 사례(유스케이스)가 TMP를 사용하기에 가장 적절한 때이다.
>Until concepts become generally available, we need to emulate them using TMP.
Use cases that require concepts (e.g. overloading based on concepts) are among the most common (and simple) uses of TMP.

##### Example

    template<typename Iter>
        /*requires*/ enable_if<random_access_iterator<Iter>, void>
    advance(Iter p, int n) { p += n; }

    template<typename Iter>
        /*requires*/ enable_if<forward_iterator<Iter>, void>
    advance(Iter p, int n) { assert(n >= 0); while (n--) ++p;}

##### Note

아래 코드는 컨셉을 사용하면 엄청 쉬워진다:
>Such code is much simpler using concepts:

    void advance(RandomAccessIterator p, int n) { p += n; }

    void advance(ForwardIterator p, int n) { assert(n >= 0); while (n--) ++p;}

##### Enforcement

???

### <a name="Rt-tmp"></a> T.122: 컴파일 타임에 타입을 조작하려면 템플릿(보통은 템플릿 별칭)을 사용하라.
>### <a name="Rt-tmp"></a> T.122: Use templates (usually template aliases) to compute types at compile time

##### Reason

템플릿 메타프로그래밍은 컴파일 타임에 타입을 생성하기 위해 유일하게 직접적인 그리고 거진 원리화된 방법이다.
>Template metaprogramming is the only directly supported and half-way principled way of generating types at compile time.

##### Note

"Traits" 기술을 타입 계산을 위한 템플릿 별칭과 값계산용 `constexpr`함수로 바꿀 수 있다.
>"Traits" techniques are mostly replaced by template aliases to compute types and `constexpr` functions to compute values.

##### Example

    ??? big object / small object optimization

##### Enforcement

???

### <a name="Rt-fct"></a> T.123: 컴파일타임에 값을 계산하려면 `constexpr`를 사용하라.
>### <a name="Rt-fct"></a> T.123: Use `constexpr` functions to compute values at compile time

##### Reason

함수는 값계산을 표현하는데 가장 분명하고 일반적인 방법이다.
대체로 `constexpr`함수는 일반 함수보다 컴파일 비용이 적다.
>A function is the most obvious and conventional way of expressing the computation of a value.
Often a `constexpr` function implies less compile-time overhead than alternatives.

##### Note

"Traits" 기술을 타입 계산을 위한 템플릿 별칭과 값계산용 `constexpr`함수로 바꿀 수 있다.
>"Traits" techniques are mostly replaced by template aliases to compute types and `constexpr` functions to compute values.

##### Example

    template<typename T>
        // requires Number<T>
    constexpr T pow(T v, int n)	// power/exponential
    {
        T res = 1;
        while (n--) res *= v;
        return res;
    }

    constexpr auto f7 = pow(pi, 7);

##### Enforcement

    * Flag template metaprograms yielding a value. These should be replaced with `constexpr` functions.

### <a name="Rt-std-tmp"></a> T.124: 표준라이브러리 TMP(Template MetaProgramming) 기능을 사용하라.
>### <a name="Rt-std-tmp"></a> T.124: Prefer to use standard-library TMP facilities

##### Reason

`conditional`, `enable_if`, `tuple`같은 표준에서 정의한 기능이 호환이 좋고, 잘 알려져 있다.
>Facilities defined in the standard, such as `conditional`, `enable_if`, and `tuple`, are portable and can be assumed to be known.

##### Example

    ???

##### Enforcement

???

### <a name="Rt-lib"></a> T.125: 표준 라이브러리의 TMP 기능 이상이 필요하다면, 타 라이브러리를 사용하라.
>### <a name="Rt-lib"></a> T.125: If you need to go beyond the standard-library TMP facilities, use an existing library

##### Reason

상급 TMP기능은 쉽지 않고, 타 라이브러리를 쓰면 (희망스럽게 지원받는)공동체에 속하게 만든다.
진짜 해야만 할 때에만 당신 자신만의 "상급 TMP 지원"을 개발하라.
>Getting advanced TMP facilities is not easy and using a library makes you part of a (hopefully supportive) community.
Write your own "advanced TMP support" only if you really have to.

##### Example

    ???

##### Enforcement

???

## <a name="SS-temp-other"></a> 기타 템플릿 규칙
>## <a name="SS-temp-other"></a> Other template rules

### <a name="Rt-name"></a> T.140: 모든 중요한 기능에 이름을 붙여라.
>### <a name="Rt-name"></a> T.140: Name all nontrivial operations

##### Reason

문서화, 가독성, 재사용성.
>Documentation, readability, opportunity for reuse.

##### Example

    ???

##### Example, good

    ???

##### Note

함수, 람다, 연산자든 뭐든지.
>whether functions, lambdas, or operators.

##### Exceptions

* 지역적으로만 사용하는 람다, `for_each`문 인자, 유사한 제어흐름 알고리즘.
* 변수 초기화용 람다
>* Lambdas logically used only locally, such as an argument to `for_each` and similar control flow algorithms.
* Lambdas as [initializers](#???)

##### Enforcement

???

### <a name="Rt-lambda"></a> T.141: 한 군데에서만 함수 객체가 필요하면 이름없는 람다를 사용하라.
>### <a name="Rt-lambda"></a> T.141: Use an unnamed lambda if you need a simple function object in one place only

##### Reason

코드를 간결하게 만들고 다른 방법에 비해 지역화에 좋다.
>That makes the code concise and gives better locality than alternatives.

##### Example

    ??? for-loop equivalent

**Exception**: 한번만 사용한다고 해도 람다에 이름을 붙이면 분명해 보인다.
>**Exception**: Naming a lambda can be useful for clarity even if it is used only once

##### Enforcement

* 동일한, 거의 동일한 람다를 찾아라. (이름있는 함수로 바꾸던지, 이름있는 람다로 바꾸던지)
>* Look for identical and near identical lambdas (to be replaced with named functions or named lambdas).

### <a name="Rt-var"></a> T.142?: 단순하게 표기하려면 템플릿 변수를 사용하라.
>### <a name="Rt-var"></a> T.142?: Use template variables to simplify notation

##### Reason

가독성 개선
>Improved readability.

##### Example

    ???

##### Enforcement

???

### <a name="Rt-nongeneric"></a> T.143: 의도적으로라도 일반화 코드를 사용하라.
>### <a name="Rt-nongeneric"></a> T.143: Don't write unintentionally nongeneric code

##### Reason

일반화, 재사용성, 쓸데없이 자세하게 commit하지 마라; 가능한 가장 일반적인 기능을 사용하라.
>Generality. Reusability. Don't gratuitously commit to details; use the most general facilities available.

##### Example

반복자를 비교하기 위해 '<'대신에 '!='를 사용하라; '!='는 순서에 상관없기 때문에 다양한 객체에 잘 동작한다.
>Use `!=` instead of `<` to compare iterators; `!=` works for more objects because it doesn't rely on ordering.

    for (auto i = first; i < last; ++i) {	// less generic
        // ...
    }

    for (auto i = first; i != last; ++i) {	// good; more generic
        // ...
    }

물론 범위-for문이 원하는대로 쓰기가 더 좋다.
>Of course, range-for is better still where it does what you want.

##### Example

필요한 기능을 가진 기본클래스를 사용하라.
>Use the least-derived class that has the functionality you need.

    class base {
    public:
        void f();
        void g();
    };

    class derived1 : public base {
    public:
        void h();
    };

    class derived2 : public base {
    public:
        void j();
    };

    void myfunc(derived1& param)  // 나쁘다, derived1 객체만 사용할 특별한 이유가 없다면
    {
        use(param.f());
        use(param.g());
    }

    void myfunc(base& param)   	// 좋다, 기본 인터페이스만으로 동작하도록 사용가능.
	// good, uses only base interface so only commit to that
    {
        use(param.f());
        use(param.g());
    }

##### Enforcement

* 반복자 비교에 '!=' 대신에 '<'를 쓴다면 표시한다.
* `x.empty()`, `x.is_empty()`이 가능한데 `x.size == 0`를 쓴다면 표시한다.
  몇몇 컨테이너는 사이즈를 모르거나 개념적으로 범위제한이 없기 때문에 size()보다 비었음을 비교하는게 낫다.
* 상속된 타입에 대한 포인터나 참조를 가지고 있지만 기본 타입으로 선언된 함수만 사용하는 함수가 있다면 표시한다.
Flag functions that take a pointer or reference to a more-derived type but only use functions declared in a base type.

>* Flag comparison of iterators using `<` instead of `!=`.
>* Flag `x.size() == 0` when `x.empty()` or `x.is_empty()` is available. Emptiness works for more containers than size(), because some containers don't know their size or are conceptually of unbounded size.
>* Flag functions that take a pointer or reference to a more-derived type but only use functions declared in a base type.

### <a name="Rt-specialize-function"></a> T.144: 함수 템플릿을 특수화하지 마라.
>### <a name="Rt-specialize-function"></a> T.144: Don't specialize function templates

##### Reason

언어규칙에 따라 함수 템플릿을 부분적으로 특수화할 수 없다.
함수템플릿을 전부 특수화할 수 있지만 그 대신으로 오버로딩하고 싶을 것이다. 함수 템플릿 특수화는 오버로딩으로 해석되지 때문에 원하는대로 동작하지 않는다.
드물지만 적절히 특수화할 수 있는 클래스 템플릿과 연계함으로써 실제로 특수화를 할 수 있다.
>You can't partially specialize a function template per language rules. You can fully specialize a function template but you almost certainly want to overload instead -- because function template specializations don't participate in overloading, they don't act as you probably wanted. Rarely, you should actually specialize by delegating to a class template that you can specialize properly.

##### Example

    ???

**Exceptions**: 함수템플릿을 특수화할 타당한 이유가 있다면 클래스 템플릿에 위임할 단 한개의 함수템플릿을 작성해라.
그리고 클래스 템플릿을 특수화하라.(부분 특수화를 작성할 능력까지 포함해서.)
>**Exceptions**: If you do have a valid reason to specialize a function template, just write a single function template that delegates to a class template, then specialize the class template (including the ability to write partial specializations).

##### Enforcement

* 함수템플릿을 특수화하고 있다면 표시한다. 대신 오버로드를 사용하라.
>* Flag all specializations of a function template. Overload instead.
