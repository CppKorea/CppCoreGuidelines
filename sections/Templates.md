
# <a name="S-templates"></a>T: 템플릿과 제네릭 프로그래밍

제네릭 프로그래밍(generic programming)은 **타입, 값, 알고리즘을 매개변수로 사용하는 타입과 알고리즘**을 사용한 프로그래밍을 말한다.
C++에서는 제네릭 프로그래밍을 위한 언어적 장치로 `template`을 지원하고 있다.

제네릭 프로그래밍의 함수들에 사용되는 인자들은 (일련의 요구사항에 따라서) 각각의 타입과 값이 규정된다. 
C++에서는 이런 요구사항을 컨셉(concepts)이라는 컴파일 시간 검사를 사용해 코드에 나타나도록 한다.

템플릿은 메타프로그래밍(meta-programming)에도 사용될 수 있다; 메타프로그래밍이란, 컴파일 시간에 코드를 만들어내는(compose) 프로그램을 의미한다.

제네릭 프로그래밍에서 중심이 되는 생각은 "컨셉"이다; 이는 템플릿 실행인자에 대한 요구사항들을 컴파일 시간에 검사할 수 있도록 기술하는 것이다.

"컨셉(Concepts)"은 ISO TS에 정의되어 있다: [concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
표준 라이브러리 컨셉들의 초안은 다른 ISO TS에 있다: [ranges](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf)
컨셉은 GCC 6.1이후 버전에서 사용할 수 있다.
그에 따라, 예시들에서 컨셉 부분은 정형화된 주석으로만 표기할 것이다. 당신이 GCC 6.1이후 버전을 사용한다면, 주석을 해제할 수 있다:

템플릿 사용 규칙 요약:

* [T.1: 코드의 추상화 수준을 높이기 위해 템플릿을 사용하라](#Rt-raise)
* [T.2: 여러가지 실행인자 타입들에 적용되는 알고리즘을 표현할 때 템플릿을 사용하라](#Rt-algo)
* [T.3: 컨테이너와 구간(range)을 표현할때 템플릿을 사용하라](#Rt-cont)
* [T.4: 문법 트리 조작을 표현하기 위해 템플릿을 사용하라](#Rt-expr)
* [T.5:  제네릭 프로그래밍과 개체지향 기술을 결합해 비용이 아닌 강점을 증폭시켜라](#Rt-generic-oo)

컨셉 사용 규칙 요약:

* [T.10: 모든 템플릿 인자에 컨셉을 명시하라](#Rt-concepts)
* [T.11: 가능한 모든 경우 표준 컨셉을 사용하라](#Rt-std-concepts)
* [T.12: 지역 변수에 `auto` 보다는 컨셉의 이름을 사용하라](#Rt-auto)
* [T.13: 단순하거나 단일 타입을 인자로 받는 컨셉에는 약식 표기를 사용하라](#Rt-shorthand)
* ???

컨셉 정의 규칙 요약:

* [T.20: 유의미한 의미구조가 없는 "컨셉"을 피하라](#Rt-low)
* [T.21: 컨셉에서는 연산들의 완전한 집합(complete set)을 요구하라](#Rt-complete)
* [T.22: 컨셉에서 쓸 수 있도록 공리를 명시하라](#Rt-axiom)
* [T.23: 정제된 컨셉(refined concept)은 더 범용적인 경우에 새로운 패턴을 더해서 차별화하라](#Rt-refine)
* [T.24: 의미구조만 다른 컨셉은 Tag 클래스 혹은 Traits를 사용해 차별화하라](#Rt-tag)
* [T.25: 제약이 서로 대비되지 않게 하라](#Rt-not)
* [T.26: 단순한 문법보다는 사용 패턴을 고려해서 컨셉을 정의하라](#Rt-use)
* ???

템플릿 인터페이스 규칙 요약:

* [T.40: 알고리즘에 연산(operation)을 전달할때는 함수 개체를 사용하라](#Rt-fo)
* [T.41: 템플릿의 컨셉에서는 오직 필요한 특성(property)만 요구하라](#Rt-essential)
* [T.42: 템플릿 별칭을 구현사항을 숨기거나 표기를 간단히 할 때 사용하라](#Rt-alias)
* [T.43: 타입에 별명을 붙일때 `typedef` 보다는 `using`을 사용하라](#Rt-using)
* [T.44: (할 수 있다면) 함수 템플릿은 클래스 템플릿의 인자 타입을 유도(deduce)할 때 사용하라](#Rt-deduce)
* [T.46: 템플릿 인자들은 최소한 `Regular`혹은 `SemiRegular`하도록 하라](#Rt-regular)
* [T.47: 일반적인 이름을 가지고 있으면서 쉽게 찾을 수 있고 제약이 거의 없는 템플릿은 지양하라](#Rt-visible)
* [T.48: 컴파일러가 컨셉을 지원하지 않는다면 `enable_if`로 유사하게 작성하라](#Rt-concept-def)
* [T.49: 가능하다면 타입 제거는 피하라](#Rt-erasure)

템플릿 정의 규칙 요약:

* [T.60: 문맥에 따라 달라질 수 있는 템플릿은 최소화 하라](#Rt-depend)
* [T.61: 너무 많은 멤버를 매개변수화 하지 말라 (SCARY)](#Rt-scary)
* [T.62: 종속적이지 않은 클래스 템플릿 멤버들은 템플릿이 아닌 상위 클래스에 배치하라](#Rt-nondependent)
* [T.64: 클래스 템플릿의 대안적(alternative) 구현을 제공할 때는 특수화를 사용하라](#Rt-specialization)
* [T.65: 함수의 대안적(alternative) 구현을 제공할 때는 Tag dispatch를 사용하라](#Rt-tag-dispatch)
* [T.67: 정규적이지 않은 타입(irregular types)들의 대안적 구현을 제공할 때는 특수화를 사용하라](#Rt-specialization2)
* [T.68: 모호하지 않도록 템플릿에서는 `()` 보다는 `{}`를 사용하라](#Rt-cast)
* [T.69: 제약없는(unqualified) 비-멤버 함수 호출은 해당 부분이 변경될 수 있도록 하려는게 아니라면 템플릿 내에서 사용하지 말아라](#Rt-customization)

템플릿과 상속구조 규칙 요약:

* [T.80: 충분한 고려 없이 클래스 계층구조를 템플릿으로 바꾸지 말아라](#Rt-hier)
* [T.81: 계층구조와 배열을 섞어서 사용하지 말아라](#Rt-array)
* [T.82: 가상 함수를 원하지 않는다면 계층 구조를 제거하라(linearize)](#Rt-linear)
* [T.83: 멤버 함수는 템플릿이면서 virtual한 함수로 선언해선 안된다](#Rt-virtual)
* [T.84: 안정된 ABI를 지원하고자 할때는 핵심 구현에 템플릿을 쓰지 마라](#Rt-abi)
* ???

가변인자 템플릿규칙 요약:

* [T.100: 다양한 타입을 가지고 가변적인 수의 실행인자들을 처리하는 함수가 필요할 때는 가변 템플릿을 사용하라](#Rt-variadic)
* [T.101: ??? How to pass arguments to a variadic template ???](#Rt-variadic-pass)
* [T.102: ??? How to process arguments to a variadic template ???](#Rt-variadic-process)
* [T.103: 동일 타입의 실행인자들을 위해서 가변템플릿을 사용하지 마라](#Rt-variadic-not)
* ???

메타프로그래밍 규칙 요약:

* [T.120: 정말 필요한 경우에만 템플릿 메타프로그래밍을 사용하라](#Rt-metameta)
* [T.121: 컨셉을 모방(emulate)하기 위해 템플릿 메타프로그래밍을 사용하라](#Rt-emulate)
* [T.122: 템플릿(대부분 템플릿 별칭)은 컴파일 시간에 타입을 계산할때 사용하라](#Rt-tmp)
* [T.123: 컴파일 시간에 값을 계산한다면 `constexpr` 함수를 사용하라](#Rt-fct)
* [T.124: 가능한 표준 라이브러리의 TMP 기능들을 사용하라](#Rt-std-tmp)
* [T.125: 만약 표준 라이브러리의 TMP 기능으로 충분하지 않다면, 가능한 이미 존재하는 라이브러리를 사용하라](#Rt-lib)
* ???

다른 템플릿 규칙 요약:

* [T.140: 재사용 가능성이 있는 모든 연산은 이름을 붙여라](#Rt-name)
* [T.141: 단순한 함수 개체가 한곳에서만 필요하다면 이름없는 람다를 사용하라](#Rt-lambda)
* [T.142: 템플릿 변수로 표기를 간단히 하라](#Rt-var)
* [T.143: 범용적이지 않은 코드는 계획없이 작성하지 말아라](#Rt-nongeneric)
* [T.144: 함수 템플릿은 특수화하지 말라](#Rt-specialize-function)
* [T.150: 해당 클래스가 개념에 부합하는지를 `static_assert`를 사용해 확인하라](#Rt-check-class)
* ???

## <a name="SS-GP"></a>T.gp: 제네릭 프로그래밍(generic programming)

제네릭 프로그래밍은 **타입, 값, 알고리즘을 매개변수로 사용하는 타입과 알고리즘**을 사용한 프로그래밍을 말한다.

### <a name="Rt-raise"></a>T.1: 코드의 추상화 수준을 높이기 위해 템플릿을 사용하라

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
        for (auto x : v)
            s += x;
        return s;
    }
```

`Arithmetic` 컨셉이 `+`와 `+=`를 모두 요구한다고 가정하면, 우리는 `sum`의 사용자가 두 연산을 지원하는 산술 타입을 제공하도록 제약한 것이다.
이는 연산을 구현에 필요한 만큼만 요구한 것은 아니다, 하지만 이는 알고리즘의 구현자에게 필요한 만큼의 자유를 부여하며 `Arithmetic` 타입이 알고리즘 전반에 사용될 수 있다는 것을 보증한다.

더 일반적인, 재사용 가능한 코드를 위해서, `vector`와 같은 하나의 컨테이너만 사용하는 것이 아닌 `Container`또는 `Range`와 같은 더 일반적인 컨셉을 사용하는 것도 가능하다.

##### Note

만약 템플릿이 하나의 알고리즘을 구현하기 위한 정확히 하나의 연산만을 요구한다면 (예컨대 `=`와 `+`를 요구하지 않고 `+=`만 요구하는 것), 유지보수를 지나치게 제약한 것이다.

우리의 의도는 템플릿 인자에 대한 요구사항을 최소화하는 것이다. 하지만 구현에서 필요한 만큼만 요구하는 것은 유의미한 컨셉일 가능성이 낮다.
> We aim to minimize requirements on template arguments, but the absolutely minimal requirements of an implementation is rarely a meaningful concept.

##### Note

템플릿은 모든 것을 표현할 수 있다 ([튜링 완전성]((https://ko.wikipedia.org/wiki/%ED%8A%9C%EB%A7%81_%EC%99%84%EC%A0%84))을 지닌다).
그러나 제네릭 프로그래밍의 목표는 비슷한 특성(property)을 가진 타입집합에 대한 연산/알고리즘을 효과적으로 일반화하는 것이다.

##### Note

주석처리된 `requires`는 `concepts`를 사용하는 부분이다.

"컨셉(Concepts)"은 ISO TS에 정의되어 있다: [concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
표준 라이브러리 컨셉들의 초안은 다른 ISO TS에 있다: [ranges](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf)
컨셉은 GCC 6.1이후 버전에서 사용할 수 있다.
그에 따라, 예시들에서 컨셉 부분은 정형화된 주석으로만 표기할 것이다. 당신이 GCC 6.1이후 버전을 사용한다면, 주석을 해제할 수 있다:

##### Enforcement

* 컨셉없이 특정 연산자를 바로 사용하는 것같은, 과도하게 단순한 요구사항을 가진 알고리즘이 있다면 지적한다
* "과도하게 단순한" 컨셉 정의가 있다면 지적하지 않는다; 더 쓸만한 컨셉을 위해 만들었을지도 모른다.

### <a name="Rt-algo"></a>T.2: 여러가지 실행인자 타입들에 적용되는 알고리즘을 표현할 때 템플릿을 사용하라

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

??? 어렵다, 사람이 해야할수도 있다

### <a name="Rt-cont"></a>T.3: 컨테이너와 구간(range)을 표현할때 템플릿을 사용하라

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

##### Exceptions

고정된 ABI 지원 인터페이스가 필요하다면 기본 구현을 제공하고, 그 형태에 따라 타입에 안전한 템플릿을 표현해야 한다. [안정된 기본 클래스 규칙](#Rt-abi)을 참조하라.

##### Enforcement

* `void*`과 하위레벨 구현코드 외에 형변환을 사용한다면 지적한다.

### <a name="Rt-expr"></a>T.4: 문법 트리 조작을 표현하기 위해 템플릿을 사용하라

##### Reason

???

##### Example

???

##### Exceptions

???

### <a name="Rt-generic-oo"></a>T.5: 제네릭 프로그래밍과 개체지향 기술을 결합해 비용이 아닌 강점을 증폭시켜라

##### Reason

제네릭 프로그래밍과 개체지향 기술은 상호보완적(complementary)이다.

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

동적인 것은 정적인 것을 돕는다: 일반적이고, 편리하며, 정적으로 결합된(bound) 인터페이스를 제공하라. 내부적으로는 동적으로 구현하라. 그렇게 함으로써 일관적인 개체를 만들어라(offer a uniform object layout).  

예시로는 `std::shared_ptr`의 제거 함수(deleter) 타입 제거를 들 수 있다. (하지만 [타입 제거를 남용하지 마라](#Rt-erasure))
```c++
#include <memory>

class Object {
public:
    template<typename T>
    Object(T&& obj)
        : concept_(std::make_shared<ConcreteCommand<T>>(std::forward<T>(obj))) {}

    int get_id() const { return concept_->get_id(); }

private:
    struct Command {
        virtual ~Command() {}
        virtual int get_id() const = 0;
    };

    template<typename T>
    struct ConcreteCommand final : Command {
        ConcreteCommand(T&& obj) noexcept : object_(std::forward<T>(obj)) {}
        int get_id() const final { return object_.get_id(); }

    private:
        T object_;
    };

    std::shared_ptr<Command> concept_;
};

class Bar {
public:
    int get_id() const { return 1; }
};

struct Foo {
public:
    int get_id() const { return 2; }
};

Object o(Bar{});
Object o2(Foo{});
```

##### Note

클래스 템플릿에 있는 비상속함수(nonvirtual function)는 사용되지 않으면 생성되지(instantiated) 않는다. -- 가상함수는 언제나 실체화(instantiate)된다.
이는 코드크기를 늘리고 필요치도 않는 함수를 실체화함으로써 일반화 타입에 대한 제약을 심화시킬지도 모른다.

##### See also

* ref ???
* ref ???
* ref ???

##### Enforcement

보다 구체적인 규칙들은 위의 참조링크로 확인하라.

## <a name="SS-concepts"></a>T.concepts: 컨셉 규칙들

컨셉(Concepts)은 템플릿 인자들에 대한 요구사항을 지정하기 위한 C++20 기능이다.
컨셉은 제네릭 프로그래밍에서의 사고에 중요한 역할을 하며, 미래의 (표준을 비롯한) C++ 라이브러리들의 많은 작업의 기초가 될 것이다.

이 항목의 규칙들은 컨셉이 지원된다고 가정한다

컨셉 사용 규칙 요약:

* [T.10: 모든 템플릿 인자에 컨셉을 명시하라](#Rt-concepts)
* [T.11: 가능한 모든 경우 표준 컨셉을 사용하라](#Rt-std-concepts)
* [T.12: 지역 변수에 `auto` 보다는 컨셉의 이름을 사용하라](#Rt-auto)
* [T.13: 단순하거나 단일 타입을 인자로 받는 컨셉에는 약식 표기를 사용하라](#Rt-shorthand)
* ???

컨셉 정의 규칙 요약:

* [T.20: 유의미한 의미구조가 없는 "컨셉"을 피하라](#Rt-low)
* [T.21: 컨셉에서는 연산들의 완전한 집합(complete set)을 요구하라](#Rt-complete)
* [T.22: 컨셉에서 쓸 수 있도록 공리를 명시하라](#Rt-axiom)
* [T.23: 정제된 컨셉(refined concept)은 더 범용적인 경우에 새로운 패턴을 더해서 차별화하라](#Rt-refine)
* [T.24: 의미구조만 다른 컨셉은 Tag 클래스 혹은 Traits를 사용해 차별화하라](#Rt-tag)
* [T.25: 제약이 서로 대비되지 않게 하라](#Rt-not)
* [T.26: 단순한 문법보다는 사용 패턴을 고려해서 컨셉을 정의하라](#Rt-use)
* ???

## <a name="SS-concept-use"></a>T.con-use: 컨셉 사용(Concept use)

### <a name="Rt-concepts"></a>T.10: 모든 템플릿 인자에 컨셉을 명시하라

##### Reason

정확함과 가독성.
템플릿 인자의 가정된 의미(문법과 의미구조)는 템플릿 인터페이스의 기본이다.
컨셉은 문서화와 템플릿용 오류 처리를 경이적으로 개선시킨다.
템플릿 인자에 적용되는 개념을 기술하는 것은 강력한 디자인 도구가 된다.

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

"컨셉(Concepts)"은 ISO TS에 정의되어 있다: [concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
표준 라이브러리 컨셉들의 초안은 다른 ISO TS에 있다: [ranges](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf)
컨셉은 GCC 6.1이후 버전에서 사용할 수 있다.
그에 따라, 예시들에서 컨셉 부분은 정형화된 주석으로만 표기할 것이다. 당신이 GCC 6.1이후 버전을 사용한다면, 주석을 해제할 수 있다:

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

##### References

TC++PL 4, Palo Alto TR, Sutton

##### Enforcement

컨셉을 사용하지 않은 템플릿 타입 인자가 있다면 지적한다

### <a name="Rt-std-concepts"></a>T.11: 가능한 모든 경우 표준 컨셉을 사용하라

##### Reason

"표준" 컨셉(GSL과 [Ranges TS](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf)에서 제공하는)은 우리 자신만의 컨셉을 생각할 필요가 없도록 한다. 동시에 우리가 조급하게 관리하는 것보다 더 깊이 생각한 결과물이며, 상호 동작성을 개선한다.

##### Note

새로운 일반화 라이브러리를 만들지 않는 한 필요한 많은 컨셉이 이미 표준 라이브러리 안에 정의되어 있다.

##### Example (using TS concepts)

```c++
    template<typename T>
        // 이런 컨셉을 정의할 필요 없다: GSL에 Sortable이 정의되어 있다
    concept Ordered_container = Sequence<T> && Random_access<Iterator<T>> && Ordered<Value_type<T>>;

    void sort(Ordered_container& s);
```

이 `Ordered_container`는 아주 타당해 보인다. 그러나 GSL(그리고 Range TS)안에 있는 `Sortable` 컨셉과 아주 비슷하다.
더 좋은가? 더 올바른가? `sort`에 대한 표준 요구사항을 정확하게 반영하고 있는가?
`Sortable`을 사용하는 것이 더 좋고 단순하다:

```c++
    void sort(Sortable& s);   // 더 나은 방법
```

##### Note

"표준" 컨셉들은 ISO 표준화 과정에서 발전하고 있다.

##### Note

쓸만한 컨셉을 디자인하는 것은 굉장히 어려운(challenging) 일이다.

##### Enforcement

어렵다.

* 제약조건이 없는 인자, 비표준 컨셉을 쓰는 템플릿, 공리(axiom)없이 'homebrew' 컨셉을 쓰는 템플릿을 찾는다
* 컨셉 발견 툴을 개발하라. [초기 실험](http://www.stroustrup.com/sle2010_webversion.pdf) 참조

### <a name="Rt-auto"></a>T.12: 지역 변수에 `auto` 보다는 컨셉의 이름을 사용하라

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

### <a name="Rt-shorthand"></a>T.13: 단순하거나 단일 타입을 인자로 받는 컨셉에는 약식 표기를 사용하라

> 역주: 약식 표기(shorthand notation)

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

"컨셉(Concepts)"은 ISO TS에 정의되어 있다: [concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
표준 라이브러리 컨셉들의 초안은 다른 ISO TS에 있다: [ranges](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf)
컨셉은 GCC 6.1이후 버전에서 사용할 수 있다.
그에 따라, 예시들에서 컨셉 부분은 정형화된 주석으로만 표기할 것이다. 당신이 GCC 6.1이후 버전을 사용한다면, 주석을 해제할 수 있다.

##### Enforcement

* `<typename T>`과 `<class T>` 표기법을 변경할 때 짧은 단어로는 쓰기가 불가능하다
* 처음에는 `typename`을 사용하고, 그 후에 한 종류 인자 컨셉으로 제한되는 선언이 있다면 지적한다

## <a name="SS-concepts-def"></a>T.concepts.def: 컨셉 정의(Concept definition rules) 규칙

좋은 컨셉을 정의하는 것은 사소한 일이 아니다.
컨셉은 Application 범위에서 기초적인 개념을 표현하기 위한 것이다 (그렇기 때문에 "concepts"라고 이름지어졌다).

하나의 클래스나 알고리즘을 문법적으로 제약하는 것은 컨셉이 의도하는 바가 아니며, 컨셉의 메커니즘을 적용했을 때의 효율을 완전히 끌어낼 수 없다.

분명, 컨셉을 정의하는 것은 구현을 사용할 수 있는 (예를 들어, C++20 또는 그 이후) 코드에 유용할 것이다.
그 이외에도 컨셉을 정의하는것 자체가 유용한 설계 기술이 될 것이며 개념 차원의 오류를 잡아내거나 구현 코드의 개념을 정리하도록 도울 것이다. 

### <a name="Rt-low"></a>T.20: 유의미한 의미구조가 없는 "컨셉"을 피하라

##### Reason

컨셉은 의미적인 개념, 예를 들면, "숫자", 요소들의 "범위", 그리고 "완전히 정렬된" 같은 개념을 표현하기 위한 것이다.
단순한 제약조건, `+`연산자를 가진다거나, `>`연산자를 가지는 것, 처럼 독자적으로 기술되어선 의미가 없다.
유저 코드보다는 의미있는 개념을 위한 블록을 구성하는데 사용되어야 한다.

##### Example, bad (using TS concepts)

```c++
    template<typename T>
    concept Addable = has_plus<T>;    // bad; 불충분하다

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
예시의 `Addable`은 교환법칙이 성립해야 한다는 수학적인 규칙에 위배된다: `a+b == b+a`

##### Note

컨셉의 진정한 특징은 문법 제약과 달리 의미구조를 기술하는 능력이 있다는 것이다.

##### Example (using TS concepts)

```c++
    template<typename T>
    // The operators +, -, *, and / for a number are assumed to 
    //  follow the usual mathematical rules
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

다수의 연산으로 정의한 컨셉은 하나의 연산으로 정의한 컨셉보다 의도치 않은 타입을 허용할 가능성이 낮다.

##### Enforcement

* 하나의 연산으로 정의한 `concepts`이 다른 `concepts`들을 정의하는 코드 이외의 코드에서 사용되면 지적하라 
* `enable_if`가 하나의 연산으로 정의한 `concepts`처럼 사용되고 있으면 지적하라

### <a name="Rt-complete"></a>T.21: 컨셉에서는 연산들의 완전한 집합을 요구하라

> Require a complete set of operations for a concept  

##### Reason

이해하기 쉽다. 상호운용성 개선. 구현이나 유지보수 인력에게 도움이 된다.

##### Note

이 규칙은 보다 일반적인 규칙인 [컨셉은 의미구조적으로 적절해야 한다](#Rt-low)에서 파생된 것이다.

##### Example, bad (using TS concepts)

```c++
    template<typename T> concept Subtractable = requires(T a, T, b) { a-b; };
```

문맥적으로 무의미 하다. 
최소한 `+`, `-` 정도는 있어야 쓸만하다.

연산들의 완전한 집합(complete set)을 예시하자면 다음과 같다

* 산술(Arithmetic)연산의 집합: `+`, `-`, `*`, `/`, `+=`, `-=`, `*=`, `/=`
* 비교(Comparable)연산의 집합: `<`, `>`, `<=`, `>=`, `==`, `!=`

##### Note

이 규칙은 컨셉의 지원여부와 무관하게 적용될 수 있다.
이는 비-템플릿 코드에 적용되는 일반적인 설계 규칙이다:

```c++
    class Minimal {
        // ...
    };

    bool operator==(const Minimal&, const Minimal&);
    bool operator<(const Minimal&, const Minimal&);

    Minimal operator+(const Minimal&, const Minimal&);
    // 다른 연산자는 없다

    void f(const Minimal& x, const Minimal& y)
    {
        if (!(x == y)) { /* ... */ }    // OK
        if (x != y) { /* ... */ }       // 이런! 컴파일 오류다

        while (!(x < y)) { /* ... */ }  // OK
        while (x >= y) { /* ... */ }    // 이런! 컴파일 오류다

        x = x + y;          // OK
        x += y;             // 이런! 컴파일 오류다
    }
```

이는 아주 작은 예시지만, 사용자들의 기대에 어긋나고 코드를 제약한다(surprising and constraining).
능률을 떨어뜨릴 수도 있다.

이 규칙은 연산들이 수학적으로 일관성 있는 집합(coherent set of operations)을 반영해야 한다는 관점을 뒷받침한다.

##### Example

```c++
    class Convenient {
        // ...
    };

    bool operator==(const Convenient&, const Convenient&);
    bool operator<(const Convenient&, const Convenient&);
    // ... and the other comparison operators ...

    Convenient operator+(const Convenient&, const Convenient&);
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

모든 연산자를 정의해야 한다는 것이 번거로울 수는 있지만, 어려운 것은 아니다.
이상적으로는, 언어가 비교 연산들을 기본적으로 제공함으로써 이 규칙을 지원해야 한다.

##### Enforcement

* 클래스가 연산들의 "완전하지 않은(odd)" 부분집합을 가지면 지적한다.
  가령, `==`를 정의하지만 `!=`를 정의하지 않거나, `+`는 정의하지만 `-`를 정의하지 않는 경우이다.
  `std::string`의 연산 집합은 "완전하지 않지만", 수정하기엔 너무 늦었다.

### <a name="Rt-axiom"></a>T.22: 컨셉에서 쓸 수 있도록 공리를 명시하라

> 역주: 공리(axiom: 증명이 없이도 자명한 것)

##### Reason

의미있고 유용한 컨셉은 의미구조에 영향을 준다.
비형식적으로든 형식적으로든 의미구조를 표현하는 것은 컨셉을 이해할 수 있게 만든다. 동시에 개념적인 오류를 잡아내도록 한다.

의미구조를 기술할 수 있다는 것은 강력한 디자인 도구이다.

##### Example (using TS concepts)

```c++
    template<typename T>
        // 사칙연산은 기본적인 수학 규칙들을 따른다고 가정한다
        // axiom(T a, T b) { 
        //      a + b == b + a; 
        //      a - a == 0; 
        //      a * (b + c) == a * b + a * c; 
        //      /*...*/ 
        // }
        concept Number = requires(T a, T b) {
            {a + b} -> T;   // the result of a + b is convertible to T
            {a - b} -> T;
            {a * b} -> T;
            {a / b} -> T;
        }
```

##### Note

이것은 수학적 공리를 표현한 것이다.
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

위에 따르면 `Balancer`는 최소한 트리의 `Node`에 대한 3개 연산을 지원해야 한다.
하지만 우리는 더 자세한 의미구조를 명시할 준비가 되지 않았는데, 새로운 형태의 균형 트리(balanced tree)가 더 많은 연산을 필요로 하거나 초기 디자인에서 모든 노드에 적용할 정확한 의미구조를 확정하기 어려울 수 있기 때문이다.

의미구조가 잘 정의되지 않은 "개념"이라도 유용할 수 있다.
가령, 개념을 정의하는 것은 초기 실험단계에서 몇몇 사항을 검사할 수 있도록 한다.
다만 안정화됐다고 생각해서는 안된다. 새로운 용례가 발견되면 불완전한 컨셉은 개선될 것이다.

##### Enforcement

* 컨셉 정의 주석 내에 "axiom" 단어를 찾는다

### <a name="Rt-refine"></a>T.23: 정제된 컨셉(refined concept)은 더 범용적인 경우에 새로운 패턴을 더해서 차별화하라

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
이는 타입을 구현하는 측의 부담을 줄여주는데, "컨셉을 만족하도록" 특별한 선언을 작성할 필요가 없기 때문이다.
2개의 컨셉이 요구사항이 정확하게 동일하다면 그들은 논리적 동치라고 할 수 있다 (개선된 점이 없다).

##### Enforcement

* 다른 컨셉과 요구사항이 정확하게 일치하는 컨셉이 있다면 지적한다. 차이를 분명하게 하고 싶다면 [T.24](#Rt-tag)를 참조하라.

### <a name="Rt-tag"></a>T.24: 의미구조만 다른 컨셉은 Tag 클래스 혹은 Traits를 사용해 차별화하라

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

컨셉으로 Tag 클래스를 감싸면 비슷한 표현이 된다:

```c++
    template<typename I> concept Contiguous = is_contiguous<I>::value;

    template<typename I>
    concept bool Contiguous_iter = RA_iter<I> && Contiguous<I>;
```

이렇게 하면 라이브러리 프로그래머가 `is_contiguous` 특성(trait)을 적절히 정의해야만 한다.

##### Note

특성(trait)은 특성 클래스(trait class)나 타입 특성(type trait)을 말한다. 사용자 혹은 표준 라이브러리에서 정의했을 수 있다..
가능하다면 표준에서 정의한 특성을 선호하라.

##### Enforcement

* 컴파일러는 동일한 컨셉을 애매하게 사용하는 것을 지적한다
* 동일한 컨셉을 정의한다면 지적한다

### <a name="Rt-not"></a>T.25: 제약이 서로 대비되지 않게 하라

> Avoid complementary constraints

##### Reason

단순명료하다. 유지보수하기 좋다.

부정(negation)을 사용해 서로 대비되는 요구사항을 가지는 함수들은 믿고 쓰기 어렵다(brittle).

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

불행하게도 `enable_if`를 사용하는 코드에서 서로 대비되는 제약을 자주 확인할 수 있다:

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

### <a name="Rt-use"></a>T.26: 단순한 문법보다는 사용 패턴을 고려해서 컨셉을 정의하라

##### Reason

정의가 더 읽기 쉽고 사용자가 작성하고 싶은 것과 직접적으로 일치한다.
형변환도 고려해야 한다. 모든 타입특성의 이름을 기억할 필요가 없다.

##### Example (using TS concepts)

`Equality`를 아래처럼 정의하고 싶을 것이다:

```c++
    template<typename T> concept Equality = has_equal<T> && has_not_equal<T>;
```

표준에서 `EqualityComparable`을 지원하는게 더 낫고 쉬울 것이라는 점은 명백하다. 하지만 - 예를 들어 - 그런 컨셉을 정의해야 한다면:

```c++
    template<typename T> concept Equality = requires(T a, T b) {
        bool == { a == b }
        bool == { a != b }
        // axiom { !(a == b) == (a != b) }
        // axiom { a = b; => a == b }  // => means "implies"
    }
```

`Equality`의 정의 안에 무의미한 컨셉인 `has_equal`와 `has_not_equal` 2개를 정의하는 것 대신에, 위와 같은 형태를 지향하라.
여기서 "무의미한"이란 `has_equal`의 의미구조를 독럽적으로(in isolation) 정의할 수 없다는 것을 의미한다.

##### Enforcement

???

## <a name="SS-temp-interface"></a> 템플릿 인터페이스(Template Interfaces)

지난 수년동안, 템플릿을 사용한 프로그래밍은 템플릿의 인터페이스와 구현을 분명히 구분하지 않아 고통받았다.
컨셉 이전에는, 이 구분을 위한 언어차원의 지원이 없었다. 그러나 템플릿에 대한 인터페이스는 매우 중요한 개념- 사용자와 구현자간의 계약 -이고, 마땅히 신중하게 설계되어야 한다.

### <a name="Rt-fo"></a>T.40: 알고리즘에 연산(operation)을 전달할때는 함수 개체를 사용하라

##### Reason

함수 개채(function object)는 함수에 대한 "단순" 포인터에 비해 인터페이스를 통해 많은 정보를 전달할 수 있다.
보통은 함수 개체를 전달하는 것이 함수 포인터에 비해 더 나은 성능을 보인다.

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

물론 저런 함수들을 `auto` 또는 컨셉을 사용해서 일반화 할 수 있다. 예를 들어:

```c++
    auto y1 = find_if(v, [](Ordered x) { return x > 7; }); // Ordered 컨셉을 만족하는 타입
    auto z1 = find_if(v, [](auto x) { return x > 7; });    // 해당 타입이 > 연산자를 지원하기를 기대한다
```

##### Note

람다 표현식은 함수개체를 생성한다.

##### Note

성능문제는 컴파일러, 최적화 기술에 달려있다.

##### Enforcement

* 함수 템플릿 인자에 포인터가 있다면 지적한다
* 템플릿에 함수 포인터가 인자로 전달된다면 지적한다 (false positive의 위험이 있다)

### <a name="Rt-essential"></a>T.41: 템플릿의 컨셉에서는 오직 필요한 특성(property)만 요구하라

##### Reason

템플릿이 쉽고 안정적인 상태로 유지된다.

##### Example (using TS concepts)

이런 경우를 생각해보라, `sort`는 (좀 심하게 단순하지만) 디버깅 지원을 포함하고 있다:

```c++
    void sort(Sortable& s)  // sort sequence s
    {
        if (debug) cerr << "enter sort( " << s <<  ")\n";
        // ...
        if (debug) cerr << "exit sort( " << s <<  ")\n";
    }
```

아래처럼 다시 작성되어야 한다:

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

결론적으로 `Sortable`에는 `iostream`을 지원한다는 요구사항이 전혀 없다는 것이다.
반대로 말해, 정렬이라는 개념은 디버깅과는 완전히 무관하다.

##### Note

수행하는 모든 처리(operation)를 요구사항으로 나열하면, 그 인터페이스는 불안정하게 된다:
디버깅 관련 기능을 변경하거나, 사용 데이터를 수집하거나, 테스트를 지원하거나, 오류를 보고하거나 등등

템플릿의 정의가 바뀌고 해당 템플릿의 모든 사용코드가 다시 컴파일되어야 할 것이다.
이는 다루기 힘든 문제고, 어떤 환경에서는 현실적이지 않다.

반대로, 컨셉을 사용한 검사를 보장하지 않는 구현을 사용한다면, 컴파일 시간 오류를 뒤늦게 확인하게 될 것이다.

필수적이지 않은 템플릿 실행인자들의 속성을 컨셉을 사용해 검사하지 않는 경우, 이는 실체화 시간까지 검사를 미루게 된다.
우리는 이 방식이 그럴만한 가치가 있는 타협(worthwhile tradeoff)이라고 생각한다.

지역적이지 않고, 의존적이지 않은 이름들을(`debug` 나 `cerr` 같은) 사용하는 것은 문맥 의존적인 코드를 낳고, 이는 "원인이 분명하지 않은(mysterious)" 오류들로 이어질 수 있다는 점에 주의하라.

##### Note

타입의 어떤 속성이 필수적인지 결정하기 어려울 수 있다.

##### Enforcement

???

### <a name="Rt-alias"></a>T.42: 템플릿 별칭을 구현사항을 숨기거나 표기를 간단히 할 때 사용하라

##### Reason

가독성을 좋게 하며, 구현 내용(implementation detail)을 숨긴다.
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
    using value_type = typename container_traits<T>::value_type;
```

이것은 `value_type`을 쓰는 사용자가 `value_type`의 구현을 알 필요가 없게 한다.

```c++
    template<typename T>
    void user2(T& c)
    {
        // ...
        value_type<T> x;
        // ...
    }
```

##### Note

간단하고 일반적인 사용은 "특성(traits)으로 감싸라!"라고 할 수 있겠다

##### Enforcement

* `using`으로 선언 이외에 중의성 제거용으로 `typename`을 사용한다면 지적한다
* ???

### <a name="Rt-using"></a>T.43: 타입에 별명을 붙일때 `typedef` 보다는 `using`을 사용하라

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

### <a name="Rt-deduce"></a>T.44: (할 수 있다면) 함수 템플릿은 클래스 템플릿의 인자 타입을 유도(deduce)할 때 사용하라

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

C++ 17 에서는 이 규칙처럼 템플릿 인자들을 생성자의 실행 인자들로부터 바로 유도할 수 있도록 하고 있다.
[Template parameter deduction for constructors (Rev. 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0091r1.html).

예를 들어:

```c++
    tuple t1 = {1, "Hamlet"s, 3.14}; // deduced: tuple<int, string, double>
```

##### Enforcement

명시적으로 특수화된 타입이 사용된 인자들의 타입과 정확히 일치하는 부분을 지적하라.

### <a name="Rt-regular"></a>T.46: 템플릿 인자들은 최소한 `Regular`혹은 `SemiRegular`하도록 하라

##### Reason

가독성. 뜻하지 않은 오류를 예방한다. 대부분의 타입들이 이를 지원한다.

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

`SemiRegular` 타입은 기본생성이 가능하다.

##### Enforcement

* 템플릿의 인자가 `SemiRegular`하지 않으면 지적한다

### <a name="Rt-visible"></a>T.47: 일반적인 이름을 가지고 있으면서 쉽게 찾을 수 있고 제약이 거의 없는 템플릿은 지양하라

##### Reason

제약하지 않은 템플릿 인자는 어떤 타입도 들어맞을 수 있으므로 그런 템플릿들이 약간의 변환이 필요한 타입들에 적용되어버릴 수 있다.
이는 ADL(Argument Dependent Lookup)이 사용되었을때 특히 짜증나고 위험한 코드가 된다. 일반적인 이름(common name)이 사용되면 이 문제가 더 자주 발생할 수 있다.

##### Example

```c++
    namespace Bad {
        struct S { int m; };
        template<typename T1, typename T2>
        bool operator==(T1, T2) { 
            cout << "Bad\n"; 
            return true; 
        }
    }

    namespace T0 {
        bool operator==(int, Bad::S) { // compare to int
            cout << "T0\n"; 
            return true;
        }  

        void test()
        {
            Bad::S bad{ 1 };
            vector<int> v(10);
            bool b = 1 == bad;
            bool b2 = v.size() == bad;
        }
    }
```

이 코드는 `T0`와 `Bad`를 출력한다.

여기서 `Bad`에 있는 `==` 중복정의가 문제를 일으키고 있다, 실제 코드에서 문제를 눈치챌 수 있었는가?
문제는 `v.size()`가 `unsigned` 정수를 반환하고, 이것이 타입 변환을 필요로하는 지역 `==` 대신 타입변환이 필요 없는 `Bad`의 `==`를 선택하도록 한다는 것이다.
표준 라이브러리의 반복자같은 현실적으로 사용되는 타입(realistic type)들은 이런 반-사회적인 기술들을 금하도록 만들 수 있다.

##### Note

같은 네임스페이스에서 제약없는 템플릿이 정의되어 있다면, 그 템플릿은 ADL에 의해서 (예제에서 발생한 것처럼) 발견될 것이다. 
이 말인 즉, 그 템플릿이 쉽게 찾을 수 있다(highly visible)는 것이다. 

##### Note

이 규칙은 필수가 되어선 안된다. 하지만 표준 위원회에서는 ADL에서 제약 없는 템플릿을 제외하는것에 동의하지 않는다.

불행하게도 이 규칙이 많은 false positives로 이어질 수 있다; 표준 라이브러리는 수많은 제약 없는 템플릿은 하나의 네임스페이스(`std`)에 배치함으로써 이 규칙을 광범위하게 위반한다. 

##### Enforcement

네임스페이스에서 구체적인 타입을 사용하는 템플릿이 같이 정의되어 있다면 지적한다. (컨셉이 가능해지기 전까지는 현실적으로 실행하기 어려울 수 있다)

### <a name="Rt-concept-def"></a>T.48: 컴파일러가 컨셉을 지원하지 않는다면 `enable_if`로 유사하게 작성하라

##### Reason

컨셉 지원이 없는 상황에서 최선이다.
`enable_if`는 조건에 따라 함수를 정의하거나 여러 함수 중 하나를 선택할때 사용할 수 있다.

##### Example

```c++
    template<typename T>
    enable_if_t<is_integral_v<T>>
    f(T v)
    {
        // ...
    }

    // Equivalent to:
    template<Integral T>
    void f(T v)
    {
        // ...
    }
```

##### Note

[complementary constraints](#T.25)에 유의하라.
컨셉 오버로딩을 `enable_if`로 꾸미는(fake) 것은 오류에 취약한 설계 기법을 쓰도록 할 수도 있다.

##### Enforcement

???

### <a name="Rt-erasure"></a>T.49: 가능하다면 타입 제거는 피하라

> 역주 : 타입 제거(type-erasure)

##### Reason

타입 제거는 서로 다른 컴파일 범위로 전달되는 타입 정보가 없어지므로 특별한 간접효과가 발생한다.

##### Example

    ???

##### Exception

`std::function`와 같은 경우처럼 때로는 타입제거가 적절할 수 있다.

##### Enforcement

???

##### Note

???

## <a name="SS-temp-def"></a>T.def: 템플릿 정의(Template definitions)

템플릿 정의 (클래스 혹은 함수)는 임의의 코드를 포함할 수 있기 때문에, C++ 프로그래밍 기술에 대한 종합적인 설명만이 이 내용을 다룬다.
이 부분에서는 특별한 템플릿 구현, 특히, 그 코드의 문맥에 의존하는 템플릿 정의에 대해 중점적으로 다룰 것이다.

### <a name="Rt-depend"></a>T.60: 문맥에 따라 달라질 수 있는 템플릿은 최소화 하라

##### Reason

이해가 쉽다.
예상치 못한 의존성 오류 발생을 최소화.
툴 작성을 쉽게 한다.

##### Example

```c++
    template<typename C>
    void sort(C& c)
    {
        std::sort(begin(c), end(c)); // 필요하고 실용적인 의존성
    }

    template<typename Iter>
    Iter algo(Iter first, Iter last) {
        for (; first != last; ++first) {
            auto x = sqrt(*first); // 잠재적 의존성: 어떤 sqrt를 의미하는 것인가?
            helper(first, x);      // 잠재적 의존성:
                                   //   helper는 first와 x에 의해서 결정된다
            TT var = 7;            // 잠재적 의존성: TT 는 어떤 타입 의미하는가?
        }
    }
```

##### Note

템플릿들은 일반적으로 헤더 파일들에 정의되기 때문에 `.cpp`파일에서 `#include`순서의 영향을 받을 소지가 더 크다(more vulnerable).

##### Note

인자만을 사용해서 템플릿을 동작하게 하는 것이 의존도를 최소한으로 줄일 수 있는 한가지 방법인데 일반적으로는 힘들다.
예를 들어, 다른 알고리즘을 사용하는 한 알고리즘은 실행 인자만 사용하는 것은 아니다. 매크로를 사용하게 하지 말라!

##### See also

[T.69](#Rt-customization)

##### Enforcement

??? 까다롭다(Tricky)

### <a name="Rt-scary"></a>T.61: 너무 많은 멤버를 매개변수화 하지 말라 (SCARY)

##### Reason

템플릿 매개변수로 쓰이지 않는 멤버는 구체적인 템플릿 매개변수를 제외하고 사용될 수 없다.
이는 보통 템플릿 사용을 제한하고 코드 사이즈를 증가시킨다.

##### Example, bad

```c++
    template<typename T, typename A = std::allocator<T>>
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

아무 문제 없어 보인다.
하지만 이 코드에서 `Link`는 allocator에 종속적이다(비록 allocator를 사용하지 않더라도).
이런 코드는 중복적인 실체화(instantiation)가 발생하도록 강제하고, 실제 사례에서는 생각 이상의 비용을 발생시킬 수 있다.
일반적으로, 이에 대한 해결책은 최소한의 템플릿 매개변수로 내포된 클래스(nested class)를 분리하는 것이다.

```c++
    template<typename T>
    struct Link {
        T elem;
        T* pre;
        T* suc;
    };

    template<typename T, typename A = std::allocator<T>>
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

어떤 이들은 `Link` 타입이 `List`안에 숨겨지지 않음으로써 템플릿 매개변수 충돌로 인한 오류가 생길 것 같지만 실제로는 제대로 구현된다는 것을 알게 되었다. 이런 기교(technique)를 
[SCARY](http://www.open-std.org/jtc1/sc22/WG21/docs/papers/2009/n2911.pdf)라 한다.

해당 문서에 따르면:
"The acronym SCARY describes assignments and initializations that are Seemingly erroneous (appearing Constrained by conflicting generic parameters), but Actually work with the Right implementation (unconstrained bY the conflict due to minimized dependencies."

##### Note

템플릿 매개변수에 의존하지 않는 람다에도 적용된다.

##### Enforcement

* 모든 템플릿 매개 변수에 의존하지 않는 멤버 타입을 표시하라.
* 모든 템플릿 매개 변수에 의존하지 않는 멤버 함수를 표시하라.
* 모든 템플릿 매개 변수에 의존하지 않는 람다나 변수 템플릿을 표시하라.

### <a name="Rt-nondependent"></a>T.62: 종속적이지 않은 클래스 템플릿 멤버들은 템플릿이 아닌 상위 클래스에 배치하라

##### Reason

템플릿 매개변수를 명시하거나 템플릿을 실체화시키지 않고도 상위 클래스의 멤버를 사용하게 한다. 

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

이 규칙을 좀 더 일반화 하자면 

"템플릿 클래스의 멤버가 M개의 템플릿 매개변수 중 N개에 의존적이라면, 이들은 N개의 매개변수만을 사용하는 기본 클래스에 배치하라"
N이 1일때는 [T.61](#Rt-scary)규칙을 충족하는 클래스 중에서 기본 클래스를 선택할 수 있다.

??? 상수나 static 멤버는 어떠한가?

##### Enforcement

* ???를 지적하라

### <a name="Rt-specialization"></a>T.64: 클래스 템플릿의 대안적(alternative) 구현을 제공할 때는 특수화를 사용하라

##### Reason

템플릿은 일반적인 인터페이스를 정의한 것이다.
특수화(specialization)는 그 인터페이스의 대안적 구현(alternative implementation)을 위한 강력한 메커니즘을 제공한다.

##### Example

```
    ??? string specialization (==)

    ??? representation specialization ?
```

##### Note

???

##### Enforcement

???

### <a name="Rt-tag-dispatch"></a>T.65: 함수의 대안적(alternative) 구현을 제공할 때는 Tag dispatch를 사용하라

##### Reason

* 템플릿은 일반화된 인터페이스를 정의한다
* Tag dispatch는 실행 인자의 타입을 사용해서 특정한 속성에 따른 구현을 선택할 수 있도록 한다
* 성능

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

Concept가 적용 가능해지면 이런 대안은 바로 구별될 수 있을 것이다:

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

### <a name="Rt-specialization2"></a>T.67: 정규적이지 않은 타입(irregular types)들의 대안적 구현을 제공할 때는 특수화를 사용하라

##### Reason

???

##### Example

???

##### Enforcement

???

### <a name="Rt-cast"></a>T.68: 모호하지 않도록 템플릿에서는 `()` 보다는 `{}`를 사용하라

##### Reason

`()` 는 문법적으로 모호한 경우가 발생한다(vulnerable to grammar ambiguities).

##### Example

```c++
    template<typename T, typename U>
    void f(T t, U u)
    {
        T v1(x);    // v1은 변수를 사용하는 함수인가?
        T v2 {x};   // v2가 변수라는 것을 바로 알 수 있다
        auto x = T(u);  // 개체 생성인가? 타입 변환인가?
    }

    f(1, "asdf"); // bad: cast from const char* to int
```

> 참고: [Why C++ Sails When the Vasa Sank](https://youtu.be/ltCgzYcpFUI?t=350)


##### Enforcement

* 초기화에 `()`를 사용하면 지적하라
* 함수처럼 보이는 타입 변환(cast)을 지적하라

### <a name="Rt-customization"></a>T.69: 제약없는(unqualified) 비-멤버 함수 호출은 해당 부분이 변경될 수 있도록 하려는게 아니라면 템플릿 내에서 사용하지 말아라

##### Reason

* 의도한만큼만 유연성을 제공하라.
* 의도치 않은 환경적 변화에 대한 취약함이 발생하지 않도록 한다.

##### Example

호출하는 코드에서 템플릿을 의도적으로 조정하도록 허용하는데는 주로 3가지 방법이 있다.

```c++
    template<class T>
        // 1. 멤버 함수를 호출한다
    void test1(T t)
    {
        t.f();    // f() 함수를 제공하는 타입 T를 요구한다
    }

    template<class T>
    void test2(T t)
        // 2. 제한하지 않은(without qualification) 비-멤버함수 
    {
        f(t);  // 호출자의 범위(scope) 혹은 T의 네임스페이스에서 사용할 수 있는 f(/*T*/)를 요구한다
    }

    template<class T>
    void test3(T t)
        // 3. "trait" 을 통해서 호출한다
    {
        test_traits<T>::f(t); // 기본 함수/타입을 사용하지 않을때는 
                              // test_traits<> 를 조작(customize)하도록 한다. 
    }
```

trait은 보통 타입을 계산하기 위한 타입이거나, 값을 계산하는 `constexpr` 함수이거나, 사용자의 타입으로 특수화되는 전통적인 traits 템플릿을 의미한다.

##### Note

`t`값으로 템플릿 타입 매개변수로 된 `helper(t)`함수를 호출하려고 한다면, `::detail` 네임스페이스에 함수를 두고 `detail::helper(t);`로 호출하라.
그렇지 않으면 `t` 타입의 네임스페이스에서 찾을 수 있는 다른 `helper` 함수가 호출될 수도 있다. [함수에 제약이 없는 경우 ADL](#Rt-unconstrained-adl)을 참고하라

##### Enforcement

* In a template, flag an unqualified call to a nonmember function that passes a variable of dependent type when there is a nonmember function of the same name in the template's namespace.

## <a name="SS-temp-hier"></a>T.temp-hier: 템플릿과 클래스 계층구조

C++에서 템플릿은 제네릭 프로그래밍을 지원하기 위한 핵심기능이다. 동시에 클래스 계층은 개체지향 프로그래밍의 근간이라고 할 수 있다.
이 두개 언어 기능은 합해서 효과적으로 사용할 수 있다. 다만 몇가지 디자인 함정은 피해야 한다.

### <a name="Rt-hier"></a>T.80: 충분한 고려 없이 클래스 계층구조를 템플릿으로 바꾸지 말아라

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

많은 경우 기본 클래스를 파라미터로 사용하지 않음으로써 안정적인 인터페이스를 지원할 수 있다;
["stable base"](#Rt-abi) 와 [OO and GP](#Rt-generic-oo)를 함께보라.

##### Enforcement

* 템플릿 인자에 의존하는 가상함수가 있다면 지적하라

### <a name="Rt-array"></a>T.81: 계층구조와 배열을 섞어서 사용하지 말아라

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

`maul()`이 [`T*`는 개별적인(individual) 개체를 가리킨다는 규칙](#Rf-ptr)을 위반한다는 점을 눈여겨 보라.

##### Alternative

적당한 (템플릿) 컨테이너를 사용하라.

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

### <a name="Rt-linear"></a>T.82: 가상 함수를 원하지 않는다면 계층 구조를 제거하라(linearize)

##### Reason

???

##### Example

???

##### Enforcement

???

### <a name="Rt-virtual"></a>T.83: 멤버 함수는 템플릿이면서 virtual한 함수로 선언해선 안된다

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

Double dispatch, Visitor 패턴, 어떤 함수를 호출하는지 분석한다.

##### Enforcement

컴파일러가 처리한다.

### <a name="Rt-abi"></a>T.84: 안정된 ABI를 지원하고자 할때는 핵심 구현에 템플릿을 쓰지 마라

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

##### Alternative

[Pimpl](#Ri-pimpl)을 사용할수도 있다

##### Enforcement

???

## <a name="SS-variadic"></a>T.var: 가변 템플릿 규칙들

???

### <a name="Rt-variadic"></a>T.100: 다양한 타입을 가지고 가변적인 수의 실행인자들을 처리하는 함수가 필요할 때는 가변 템플릿을 사용하라

##### Reason

가변인자 템플릿은 가장 일반화된 메커니즘이면서 동시에 효율적이고, 타입안정성을 지닌다. C 형식의 가변인자를 사용하지 말아라.

##### Example

```
    ??? printf
```

##### Enforcement

* 코드에 `va_arg`이 있다면 지적한다

### <a name="Rt-variadic-pass"></a>T.101: ??? How to pass arguments to a variadic template ???

##### Reason

 ???

##### Example

```
    ??? 이동만 가능하거나 인자 참조에 주의하라
```

##### Enforcement

???

### <a name="Rt-variadic-process"></a>T.102: ??? How to process arguments to a variadic template ???

##### Reason

 ???

##### Example

```
    ??? forwarding, type checking, references
```

##### Enforcement

???

### <a name="Rt-variadic-not"></a>T.103: 동일 타입의 실행인자들을 위해서 가변템플릿을 사용하지 마라

##### Reason

같은 타입의 인자들은 `initializer_list` 형태로 더 정확히 명시할 수 있다.

##### Example

```
    ???
```

##### Enforcement

???

## <a name="SS-meta"></a>T.meta: 템플릿 메타프로그래밍 (TMP)

템플릿은 컴파일-타임 프로그래밍을 위한 일반적인 매커니즘을 제공한다.
메타프로그래밍은 하나 이상의 입력이나 결과 자체가 타입인 프로그래밍을 말한다.

템플릿은 튜링 완전성을 가진 (modulo memory capacity) 덕타이핑을 제공한다.이를 위해 필요한 문법과 기술은 아주 끔찍하다(horrendous).

### <a name="Rt-metameta"></a>T.120: 정말 필요한 경우에만 템플릿 메타프로그래밍을 사용하라

##### Reason

템플릿 메타프로그래밍은 올바르게 쓰기가 어렵고, 컴파일 속도를 느리게 하고, 유지보수를 어렵게 한다.
그러나 템플릿 메타프로그래밍이 전문가 수준의 어셈블리 코드보다 성능이 더 좋은 예들이 있다.
게다가 런타임 코드보다 핵심사상을 더 잘 표현하는 실제 예들도 있다.

예를 들어 컴파일 타임에 AST(Abstract Syntax Tree)를 조작해야 한다면 (예컨대 선택적으로 행렬 연산을 전파(folding)한다던지) C++에서는 다른 방법이 없다.

##### Example, bad

```
    ???
```

##### Example, bad

```
    enable_if
```


대안으로, 컨셉을 사용하라. [언어가 지원하지 않는 컨셉을 사용하는 방법](#Rt-emulate)을 참고하라.

##### Example

```
    ??? 좋은 예제가 필요하다
```

##### Alternative

만약 결과가 타입이 아니라 값이라면 [`constexpr` 함수](#Rt-fct)를 사용하라.

##### Note

템플릿 메타프로그래밍을 전처리 매크로로 대신하고 싶다고 느낀다면 너무 나간것이다.

### <a name="Rt-emulate"></a>T.121: 컨셉을 모방(emulate)하기 위해 템플릿 메타프로그래밍을 사용하라

> Use template metaprogramming primarily to emulate concepts

##### Reason

컨셉 개념이 널리 사용될때까지 TMP를 사용해서 에뮬레이트해야 할 것이다.
(컨셉에 기반한 중복정의와 같이) 컨셉이 필요한 경우(usecase)들은 보통 TMP를 사용하고 있다. 

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

### <a name="Rt-tmp"></a>T.122: 템플릿(대부분 템플릿 별칭)은 컴파일 시간에 타입을 계산할때 사용하라

##### Reason

템플릿 메타프로그래밍은 컴파일 타임에 타입을 생성하기 위한 유일하고 직접적인 방법이다.

##### Note

"특성(Traits)" 사용방법은 대부분 템플릿 별칭이나 `constexpr`함수로 대체되었다.

##### Example

```
    ??? big object / small object optimization
```

##### Enforcement

???

### <a name="Rt-fct"></a>T.123: 컴파일 시간에 값을 계산한다면 `constexpr` 함수를 사용하라

##### Reason

함수는 값 계산을 표현하는데 가장 분명하고 일반적인 방법이다.
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
        while (n--) 
            res *= v;
        return res;
    }

    constexpr auto f7 = pow(pi, 7);
```

##### Enforcement

* 값을 계산해내는 템플릿 메타프로그램들을 지적하라. 이들은 `constexpr` 함수로 대체되어야 한다.

### <a name="Rt-std-tmp"></a>T.124: 가능한 표준 라이브러리의 TMP 기능들을 사용하라

##### Reason

`conditional`, `enable_if`, `tuple`같은 표준에서 정의한 기능이 호환이 좋고, 잘 알려져 있다.

##### Example

```
    ???
```

##### Enforcement

???

### <a name="Rt-lib"></a>T.125: 만약 표준 라이브러리의 TMP 기능으로 충분하지 않다면, 가능한 이미 존재하는 라이브러리를 사용하라

##### Reason

좀더 고급한(advanced) TMP 기능들은 작성하는 것은 쉽지 않은 일이고, 라이브러리를 사용하는 것은 커뮤니티에 도움이 된다(hopefully supportive).

정말 필요한 경우에만 자신만의 "고급 TMP 지원"을 작성하라.

##### Example

```
    ???
```

##### Enforcement

???

## <a name="SS-temp-other"></a> 기타 템플릿 규칙

### <a name="Rt-name"></a>T.140: 재사용 가능성이 있는 모든 연산은 이름을 붙여라

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
            if (r.name.size() != n.size()) // name to compare to is in n
                return false; 
            for (int i = 0; i < r.name.size(); ++i)
                if (tolower(r.name[i]) != tolower(n[i])) 
                    return false;
            return true;
        }
    );
```

이 코드에는 유용한 함수(대소문자 구분이 없는 문자열 비교)가 숨어있다. 람다의 실행인자(argument)들이 많아지면 이런일이 생긴다.

```c++
    bool compare_insensitive(const string& a, const string& b)
    {
        if (a.size() != b.size()) 
            return false;
        for (int i = 0; i < a.size(); ++i) 
            if (tolower(a[i]) != tolower(b[i])) 
                return false;
        return true;
    }

    auto x = find_if(vr.begin(), vr.end(),
        [&](Rec& r) { compare_insensitive(r.name, n); }
    );
```

혹은, (`n`에 묵시적으로 이름이 묶이는(binding) 것을 피하고 싶다면) 아래처럼 작성할수도 있다:

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

* (어려움) 유사한 람다를 지적하라
* ???

### <a name="Rt-lambda"></a>T.141: 단순한 함수 개체가 한곳에서만 필요하다면 이름없는 람다를 사용하라

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

### <a name="Rt-var"></a>T.142?: 템플릿 변수로 표기를 간단히 하라

##### Reason

더 읽기 좋다

##### Example

```
    ???
```

##### Enforcement

???

### <a name="Rt-nongeneric"></a>T.143: 범용적이지 않은 코드는 계획없이 작성하지 말아라

> Don't write unintentionally nongeneric code

##### Reason

일반화, 재사용성, 필요 이상으로 자세하게 하지 마라; 가능한 일반적인 기능을 사용하라.

##### Example

반복자(iterator) 비교에는 `<`대신 `!=`를 사용하라; `!=`는 순서의 영향을 받지 않기 때문에 더 많은 경우에 사용할 수 있다.

```c++
    for (auto i = first; i < last; ++i) {   // 비교가 가능해야만 사용할 수 있다
        // ...
    }

    for (auto i = first; i != last; ++i) {  // good; 좀 더 범용적(generic)이다 
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
* `x.empty()` 혹은 `x.is_empty()` 표현이 가능하다면 `x.size() == 0`의 사용을 지적한다. 보다 많은 컨테이너 타입들이 `size()`보다는 비어있는지를 검사하는 것을 지원한다. 이런 경우는 크기를 알 수 없거나, 개념적으로 크기 제한이 없기 때문이다.
* 상속된 타입에 대한 포인터나 참조를 가지고 있지만 기본 타입으로 선언된 함수만 사용하는 함수가 있다면 지적한다

### <a name="Rt-specialize-function"></a>T.144: 함수 템플릿은 특수화하지 말라

##### Reason

언어규칙에 따라 함수 템플릿을 부분적으로 특수화할 수 없다.
함수 템플릿을 전부 특수화할 수 있지만 그 대신으로 오버로딩하고 싶을 것이다. 함수 템플릿 특수화는 오버로딩으로 해석되기 때문에 원하는대로 동작하지 않는다.
드물지만 적절히 특수화할 수 있는 클래스 템플릿과 연계함으로써 실제로 특수화를 할 수 있다.

##### Example

```
    ???
```

##### Exceptions

함수 템플릿을 특수화할 타당한 이유가 있다면 클래스 템플릿을 사용하는 함수 템플릿을 하나만 작성하라.
그리고 클래스 템플릿을 특수화하라. (부분 특수화를 작성하는 것까지 포함하라)

##### Enforcement

* 함수 템플릿을 특수화하고 있다면 지적하라. 가능하다면 오버로딩으로 대신하라

### <a name="Rt-check-class"></a>T.150: 해당 클래스가 개념에 부합하는지를 `static_assert`를 사용해 확인하라

##### Reason

클래스가 컨셉을 만족시키는지 확인해야 한다면, 일찍 검사하는 것이 사용자들의 고통을 줄여준다.

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

구현파일 안의 어디선가 컴파일러가 `X`가 의도한 속성(desired properties)을 검사할 수 있도록 하라:

```c++
    static_assert(default_constructible<X>);    // error: X 는 기본 생성자가 없다.
    static_assert(copyable<X>);                 // error: X의 이동 생성자를 정의하지 않았다.
```

##### Enforcement

마땅히 검사할 방법이 없다(Not feasible).
