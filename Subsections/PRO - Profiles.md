# 프로파일

># Profiles

"프로파일"은 특정한 보장하기 위한 "결정성" "이식성" 규칙들(즉, 제한사항) 입니다.
여기서 "결정성"은 로컬 분석이 되어야 하며, 컴파일러에 의해 구현 될수 있음(꼭 그러지 않아도 됨)을 의미합니다.
"이식성"은 언어 규칙같이 같은 코드에 따른 그 결과가 같아야 한다는 것을 의미합니다.

>A "profile" is a set of deterministic and portably enforceable subset rules (i.e., restrictions) that are designed to achieve a specific guarantee. "Deterministic" means they require only local analysis and could be implemented in a compiler (though they don't need to be). "Portably enforceable" means they are like language rules, so programmers can count on enforcement tools giving the same answer for the same code.

이런 프로파일을 이용하여 경고가 없도록 작성된 코드는 프로파일에 일치하는 것이라 말할 수 있습니다.
규격에 부합하는 코드는 프로파일 상의 안전성을 지켜서 작성된 것이므로 안전하다고 판단할 수 있습니다.
규격에 부합하는 코드는 다른 코드나, 라이브러리 또는 외부환경에 의해서 에러가 발생할 수는 있어도 그 자체에서 오류가 발생하지 않습니다.
프로파일은 올바른 코드의 작성을 도와주는 라이브러리들을 소개해 드릴 것입니다.

>Code written to be warning-free using such a language profile is considered to conform to the profile. Conforming code is considered to be safe by construction with regard to the safety properties targeted by that profile. Conforming code will not be the root cause of errors for that property, although such errors may be introduced into a program by other code, libraries or the external environment. A profile may also introduce additional library types to ease conformance and encourage correct code.

프로파일 요약:

* [Pro.type: 타입 안전성](#SS-type)
* [Pro.bounds: 범위 안전성](#SS-bounds)
* [Pro.lifetime: 수명 안전성](#SS-lifetime)

>Profiles summary:
>
>* [Pro.type: Type safety](#SS-type)
* [Pro.bounds: Bounds safety](#SS-bounds)
* [Pro.lifetime: Lifetime safety](#SS-lifetime)



<a name="SS-type"></a>
## 타입 안전성 프로파일

>## Type safety profile

이 유형의 프로파일은 타입을 정확히 사용하며, 부주의한 타입변형을 방지하면서 코드를 작성하도록 도와드릴 것입니다.
안전하지 않은 캐스팅과 `union`의 사용을 포함하여 타입이 잘못사용되는 것에 대한 주요 원인을 제거하는 것에 초점을 맞춰서 진행하겠습니다.

>This profile makes it easier to construct code that uses types correctly and avoids inadvertent type punning. It does so by focusing on removing the primary sources of type violations, including unsafe uses of casts and unions.

이 장의 목적인, 타입 안전성을 정의하자면, 프로그램 상에서 변수를 원래 타입과 다르게 사용하지 않는 것이라 하겠습니다. 실제 `U` 타입으로 정의된 객체가 저장된 메모리에 `T`타입으로 읽어서 사용하지 말자는 겁니다. (타입 안전성 하나만 지켜서는 코드가 안전하다는 보장을 받기 힘듭니다. [범위 안전성](#SS-bounds)과 [수명 안전성](#SS-lifetime)을 함께 지켰을 때 비로서 완전히 안전성이 보장된다고 할 수 있습니다.)

>For the purposes of this section, type-safety is defined to be the property that a program does not use a variable as a type it is not. Memory accessed as a type `T` should not be valid memory that actually contains an object of an unrelated type `U`. (Note that the safety is intended to be complete when combined also with [Bounds safety](#SS-bounds) and [Lifetime safety](#SS-lifetime).)

아래 사항들에 대해서는 고려중에 있지만 이번에 다루지는 않을 것이며, 다른 프로파일에 더 있을 수도 있습니다. :

>The following are under consideration but not yet in the rules below, and may be better in other profiles:

   - 내로우잉 산술 승격/변환 (안전한 산술 프로파일 단위로 나누는 것 같은 식의)
   - 음수 부동소수점 연산을 양수 정수형 타입으로 산술 변환 (상동)
   - 정의되지 않은 동작 선택 : ??? 너무 큰 주제라서, Gaby의 UB 목록에서 다뤄야할지...
   - 지정되지 않은 동작 선택 : ??? 이건 안전성에 관한거라기 보단 이식성에 더 가깝지 않을까요 ?
   - 상수성 위반 ? 이것이 안정성에 관계된거라면요.

>   - narrowing arithmetic promotions/conversions (likely part of a separate safe-arithmetic profile)
   - arithmetic cast from negative floating point to unsigned integral type (ditto)
   - selected undefined behavior: ??? this is a big bucket, start with Gaby's UB list
   - selected unspecified behavior: ??? would this really be about safety, or more a portability concern?
   - constness violations? if we rely on it for safety

이러한 프로파일의 구현은 소스코드 상에서 다음과 같은 패턴들을 부적합한 것으로 판단하고 수정 조치합니다.

>An implementation of this profile shall recognize the following patterns in source code as non-conforming and issue a diagnostic.


<a name="Pro-type-reinterpretcast"></a>
### Type.1: `reinterpret_cast`를 사용하지 마세요.

**근거**:
타입 안전성을 위반하고 있으며, 실제로 `X`타입을 서로 호환되지 않는 타입 `Z`인 것처럼 읽을 여지가 있습니다.

>### Type.1: Don't use `reinterpret_cast`.
>
>**Reason**:
Use of these casts can violate type safety and cause the program to access a variable that is actually of type `X` to be accessed as if it were of an unrelated type `Z`.

**잘못된 예**:

    std::string s = "hello world";
    double* p = reinterpret_cast<double*>(&s); // BAD

>**Example; bad**:

    std::string s = "hello world";
    double* p = reinterpret_cast<double*>(&s); // BAD
    
**시행하기**: `reinterpret_cast`가 사용된 곳을 찾아내서 대신 `variant`를 사용하세요.

>**Enforcement**: Issue a diagnostic for any use of `reinterpret_cast`. To fix: Consider using a `variant` instead.


<a name="Pro-type-downcast"></a>
### Type.2: `static_cast`를 다운캐스팅하는 곳에 사용하지 말고, 대신 `dynamic_cast`를 사용하세요.

**근거**:
타입 안전성을 위반하고 있으며, 실제로 `X`타입을 서로 호환되지 않는 타입 `Z`인 것처럼 읽을 여지가 있습니다.

>### Type.2: Don't use `static_cast` downcasts. Use `dynamic_cast` instead.
>
>**Reason**:
Use of these casts can violate type safety and cause the program to access a variable that is actually of type `X` to be accessed as if it were of an unrelated type `Z`.

**잘못된 예**:

    class base { public: virtual ~base() =0; };

    class derived1 : public base { };

    class derived2 : public base {
        std::string s;
    public:
        std::string get_s() { return s; }
    };

    derived1 d1;
    base* p = &d1; // OK. base 타입포인터로 암시적 변환은 괜찮습니다.

    derived2* p2 = static_cast<derived2*>(p); // BAD, d1은 a derived2가 아닌데도 그렇게 취급하려 합니다.
    cout << p2.get_s(); // d1에는 string 멤버가 없는데도 제어하려 합니다. 결과적으로 d1 근처 임의의 바이트를 출력하게 됩니다.

>**Example; bad**:

    class base { public: virtual ~base() =0; };

    class derived1 : public base { };

    class derived2 : public base {
        std::string s;
    public:
        std::string get_s() { return s; }
    };

    derived1 d1;
    base* p = &d1; // ok, implicit conversion to pointer to base is fine

    derived2* p2 = static_cast<derived2*>(p); // BAD, tries to treat d1 as a derived2, which it is not
    cout << p2.get_s(); // tries to access d1's nonexistent string member, instead sees arbitrary bytes near d1

**시행하기**: `static_cast`로 다운캐스팅하는 곳을 찾으세요. 즉, `X`의 포인터나 참조 중 `X`가 아니거나 `X`의 부모클래스로 부터 참조된 것들을 찾으세요. 그 대신 `dynamic_cast`나 `variant`를 사용하세요.

>**Enforcement**: Issue a diagnostic for any use of `static_cast` to downcast, meaning to cast from a pointer or reference to `X` to a pointer or reference to a type that is not `X` or an accessible base of `X`. To fix: If this is a downcast or cross-cast then use a `dynamic_cast` instead, otherwise consider using a `variant` instead.


<a name="Pro-type-constcast"></a>
### Type.3: `const`를 없애기 위해 `const_cast`를 사용하지 마세요.

**근거**:
`const` 제거는 말도 안됩니다. 실제로 `const`로 정의된 변수라면, 정의되지 않은 동작이 발생하게 됩니다.

>### Type.3: Don't use `const_cast` to cast away `const` (i.e., at all).
>
>**Reason**:
Casting away `const` is a lie. If the variable is actually declared `const`, it's a lie punishable by undefined behavior.

**잘못된 예**:

    void f(const int& i) {
        const_cast<int&>(i) = 42;   // BAD
    }

    static int i = 0;
    static const int j = 0;

    f(i); // 조용히 부작용이 발생합니다.
    f(j); // 정의되지 않은 동작이 발생합니다.

>**Example; bad**:

    void f(const int& i) {
        const_cast<int&>(i) = 42;   // BAD
    }

    static int i = 0;
    static const int j = 0;

    f(i); // silent side effect
    f(j); // undefined behavior
    

**예외**: 잘못된 `const` 함수를 호출할 때는 `const`를 없에야 합니다. 이런 함수들을 올바른 `const`  inline 래퍼로 한 곳에서 캐스팅하는 것을 캡슐화 하는거보다는 `const_cast`를 사용하는 것이 더 좋습니다.

**시행하기**: `const_cast`가 사용된 곳을 찾아서 값을 변경하지 말거나 아니면 아에 `const`로 만들지 마세요.

>**Exception**: You may need to cast away `const` when calling `const`-incorrect functions. Prefer to wrap such functions in inline `const`-correct wrappers to encapsulate the cast in one place.
>
>**Enforcement**: Issue a diagnostic for any use of `const_cast`. To fix: Either don't use the variable in a non-`const` way, or don't make it `const`.


<a name="Pro-type-cstylecast"></a>
### Type.4: `static_cast` 다운캐스팅, `const_cast`, `reinterpret_cast` 처럼 사용 될 수 있는 C-스타일의 `(T)expression`을 사용하지 마세요.

**근거**:
이런 변환들은 타입 안전성을 위반하고 있으며, 실제로 `X`타입을 서로 호환되지 않는 타입 `Z`인 것처럼 읽을 여지가 있습니다.
C-스타일의 `(T)expression`는 실제로 다음의 순서대로 가능한 방법을 판단합니다: `const_cast`, `static_cast`, `const_cast` 이후 `static_cast`, `reinterpret_cast`, `const_cast` 이후 `reinterpret_cast`. 이 규칙은 안전하지 않은 변환으로 사용될 때에만 `(T)expression` 사용을 금지시킵니다.

>### Type.4: Don't use C-style `(T)expression` casts that would perform a `static_cast` downcast, `const_cast`, or `reinterpret_cast`.
>
>**Reason**:
Use of these casts can violate type safety and cause the program to access a variable that is actually of type `X` to be accessed as if it were of an unrelated type `Z`.
Note that a C-style `(T)expression` cast means to perform the first of the following that is possible: a `const_cast`, a `static_cast`, a `static_cast` followed by a `const_cast`, a `reinterpret_cast`, or a `reinterpret_cast` followed by a `const_cast`. This rule bans `(T)expression` only when used to perform an unsafe cast.

**잘못된 예**:

    std::string s = "hello world";
    double* p = (double*)(&s); // BAD

    class base { public: virtual ~base() = 0; };

    class derived1 : public base { };

    class derived2 : public base {
        std::string s;
    public:
        std::string get_s() { return s; }
    };

    derived1 d1;
    base* p = &d1; // OK. 부모 class로의 암시적 변환은 괜찮아요.

    derived2* p2 = (derived2*)(p); // BAD, d1은 derived2가 아닌데도 그렇게 취급하려 합니다.
    cout << p2.get_s(); // d1에는 string 멤버가 없는데도 제어하려 합니다. 결과적으로 d1 근처 임의의 바이트를 출력하게 됩니다.

    void f(const int& i) {
        (int&)(i) = 42;   // BAD
    }

    static int i = 0;
    static const int j = 0;

    f(i); // 조용히 부작용이 발생합니다.
    f(j); // 정의되지 않은 동작이 발생합니다.

>**Example; bad**:

    std::string s = "hello world";
    double* p = (double*)(&s); // BAD

    class base { public: virtual ~base() = 0; };

    class derived1 : public base { };

    class derived2 : public base {
        std::string s;
    public:
        std::string get_s() { return s; }
    };

    derived1 d1;
    base* p = &d1; // ok, implicit conversion to pointer to base is fine

    derived2* p2 = (derived2*)(p); // BAD, tries to treat d1 as a derived2, which it is not
    cout << p2.get_s(); // tries to access d1's nonexistent string member, instead sees arbitrary bytes near d1

    void f(const int& i) {
        (int&)(i) = 42;   // BAD
    }

    static int i = 0;
    static const int j = 0;

    f(i); // silent side effect
    f(j); // undefined behavior

**시행하기**: `static_cast` 다운캐스팅, `const_cast`, `reinterpret_cast`로 동작하는 C-스타일 `(T)expression`를 찾아서 `static_cast` 다운캐스팅는 `dynamic_cast`, `const_cast`는 제대로 된 `const` 선언, `reinterpret_cast`는 `variant`로 수정하세요.

>**Enforcement**: Issue a diagnostic for any use of a C-style `(T)expression` cast that would invoke a `static_cast` downcast, `const_cast`, or `reinterpret_cast`. To fix: Use a `dynamic_cast`, `const`-correct declaration, or `variant`, respectively.


<a name="Pro-type-init"></a>
### Type.5: 초기화되지 않은 변수를 사용하지 마세요.

[ES.20: Always initialize an object](#Res-always) 참조.

>### Type.5: Don't use a variable before it has been initialized.
>
>[ES.20: Always initialize an object](#Res-always) is required.


<a name="Pro-type-memberinit"></a>
### Type.6: 멤버 변수는 반드시 초기화하세요.

**근거**:
변수가 초기화되기 전에는 해당 타입에 부합하는 값을 가지고 있지 않을 수 있습니다. 임의의 비트 패턴값을 가지고 있어서 호출 할때마다 다른 값으로 보일 수 있습니다.

>### Type.6: Always initialize a member variable.
>
>**Reason**:
Before a variable has been initialized, it does not contain a deterministic valid value of its type. It could contain any arbitrary bit pattern, which could be different on each call.

**예**:

    struct X { int i; };

    X x;
    use(x); // BAD, x는 초기화되지 않았습니다.

    X x2{}; // GOOD
    use(x2);

>**Example**:

    struct X { int i; };

    X x;
    use(x); // BAD, x hs not been initialized

    X x2{}; // GOOD
    use(x2);

**시행하기**:
   - 모든 멤버 변수를 초기화하지 않는 생성자를 찾아서 초기화 하도록 수정하거나, 초기화 목록에 멤버들을 작성하세요.
   - `()`나 `{}`를 이용하여 멤버를 초기화하지 않는 생성자를 찾아서 `()`나 `{}`를 추가하세요.

>**Enforcement**:
   - Issue a diagnostic for any constructor of a non-trivially-constructible type that does not initialize all member variables. To fix: Write a data member initializer, or mention it in the member initializer list.
   - Issue a diagnostic when constructing an object of a trivially constructible type without `()` or `{}` to initialize its members. To fix: Add `()` or `{}`.


<a name="Pro-type-unions"></a>
### Type.7: `union`을 이용하여 멤버들을 제어하지 말고, `variant`를 사용하세요.

**근거**:
`union` 멤버를 읽으면 가장 마지막에 저장된 것을 읽게되며, 저장을 하면 다른 멤버의 해제자가 호출되게 됩니다.
그로인해 일반적인 C++언어의 안전성에 의존 할 수가 없게되며, 프로그래머가 판단하여야 하므로 취약할 수 밖에 없습니다.

>### Type.7: Avoid accessing members of raw unions. Prefer `variant` instead.
>
>**Reason**:
Reading from a union member assumes that member was the last one written, and writing to a union member assumes another member with a nontrivial destructor had its destructor called. This is fragile because it cannot generally be enforced to be safe in the language and so relies on programmer discipline to get it right.

**예**:

    union U { int i; double d; };
    
    U u;
    u.i = 42;
    use(u.d); // BAD, 아직 정의되지 않았음
    
    variant<int,double> u;
    u = 42; // 이제 u는 int 값을 가지고 있음
    use(u.get<int>()); // OK
    use(u.get<double>()); // 예외발생 ??? variant에 대한 표준화작업이 아직 완료되지 않아서, 그 이후 업데이트 하겠음

>**Example**:

    union U { int i; double d; };
    
    U u;
    u.i = 42;
    use(u.d); // BAD, undefined
    
    variant<int,double> u;
    u = 42; // u  now contains int
    use(u.get<int>()); // ok
    use(u.get<double>()); // throws ??? update this when standardization finalizes the variant design
    
단지 `union`을 복사하는 것 자체가 타입안전성에 위배되는 것은 아닙니다. 그래서, 안전한 코드도 `union`을 통해 안전하지 않은 코드를 다른 곳으로 전달이 가능합니다.

**시행하기**:
   - `union`이 사용된 곳을 찾아서 대신 `variant`를 사용하세요.

>Note that just copying a union is not type-unsafe, so safe code can pass a union from one piece of unsafe code to another.
>
>**Enforcement**:
   - Issue a diagnostic for accessing a member of a union. To fix: Use a `variant` instead.
   
 

<a name="Pro-type-varargs"></a>
### Type.8: 가변 인자를 사용하지 말고 가변 인자 템플릿을 사용하세요.

**근거**:
가변인자를 읽는 것은 정확한 타입으로 전달된 것으로 간주합니다.
가변인자로 보내는 것은 정확한 타입으로 읽을 것이라 간주합니다.
그로인해 일반적인 C++언어의 안전성에 의존 할 수가 없게되며, 프로그래머가 판단하여야 하므로 취약할 수 밖에 없습니다.

>### Type.8: Avoid reading from varargs or passing vararg arguments. Prefer variadic template parameters instead.
>
>**Reason**:
Reading from a vararg assumes that the correct type was actually passed. Passing to varargs assumes the correct type will be read. This is fragile because it cannot generally be enforced to be safe in the language and so relies on programmer discipline to get it right.

**예**:

    int sum(...) {
        // ...
        while( /*...*/ )
            result += va_arg(list, int); // NG : int로 전달되었다 가정합니다.
        // ...
    }
        
    sum( 3, 2 ); // OK
    sum( 3.14159, 2.71828 ); // NG : 위 함수에서 int라 가정했는데 double을 전달하였습니다.
    
    template<class ...Args>
    auto sum(Args... args) { // GOOD, 좀 더 유연하게 사용이 가능한 형태입니다.
        return (... + args); // note: C++17 "fold 표현식"
    }
        
    sum( 3, 2 ); // OK : 5
    sum( 3.14159, 2.71828 ); // OK : ~5.85987

>**Example**:

    int sum(...) {
        // ...
        while( /*...*/ )
            result += va_arg(list, int); // BAD, assumes it will be passed ints
        // ...
    }
        
    sum( 3, 2 ); // ok
    sum( 3.14159, 2.71828 ); // BAD, undefined
    
    template<class ...Args>
    auto sum(Args... args) { // GOOD, and much more flexible
        return (... + args); // note: C++17 "fold expression"
    }
        
    sum( 3, 2 ); // ok: 5
    sum( 3.14159, 2.71828 ); // ok: ~5.85987

참고: `...`로 인자를 선언하는 것은 실제 인수 전달과 관련이 없는 경우에 대해서는 유용합니다. 템플릿메타 프로그래밍에서 모든 것을 받는 경우를 표현하거나 중복정의(overload)를 매번 할 수 없기 때문에 아무거나 다 받는 함수를 정의하는 것입니다.
    
**시행하기**:
   - `va_list`, `va_start`, `va_arg`이 사용된 곳을 찾아서 가변인자 템플릿으로 수정하세요.
   - 가변인자를 사용하는 곳을 찾아서 다른 함수로 만들거나 `[[suppress(types)]]`를 참조하세요.

>Note: Declaring a `...` parameter is sometimes useful for techniques that don't involve actual argument passing, notably to declare “take-anything” functions so as to disable "everything else" in an overload set or express a catchall case in a template metaprogram.
>
>**Enforcement**:
   - Issue a diagnostic for using `va_list`, `va_start`, or `va_arg`. To fix: Use a variadic template parameter list instead.
   - Issue a diagnostic for passing an argument to a vararg parameter. To fix: Use a different function, or `[[suppress(types)]]`.


<a name="SS-bounds"></a>
## 범위 안전성 프로파일

이 프로파일은 메모리 블록 할당 작업에 대해 코드 작성을 쉽게 해 줍니다.
포인터 연산, 배열 인덱스 연산 등에서 발생하는 범위 위반 사항을 제거하는 것에 초점을 맞춰서 진행하겠습니다.
이 프로파일의 주요 기능중 하나는 (하나의 객체가 아니라) 배열을 참조하는 포인터를 제한하자는 것입니다.

이 문서에 목적에 따르면, 범위-안정성이란 변수가 할당된 범위 외부에서 해당 변수를 사용하지 않는 것을 의미합니다.
([타입 안전성](#SS-type)과 [수명 안전성](#SS-lifetime)을 함께 지켰을 때 범위 안전성도 그 의미가 있습니다. 예를 들엇 포인터 간의 변환에서 타입 안전성이 깨진 경우에는 범위 안전성 만으로 해당 작업에 대한 안전성을 확보하지 못합니다.)

아래 사항들에 대해서는 고려중에 있지만 이번에 다루지는 않을 것이며, 다른 프로파일에 더 있을 수도 있습니다. :

   -

이러한 프로파일의 구현은 소스코드 상에서 다음과 같은 패턴들을 부적합한 것으로 판단하고 수정 조치합니다.

>## Bounds safety profile
>
>This profile makes it easier to construct code that operates within the bounds of allocated blocks of memory. It does so by focusing on removing the primary sources of bounds violations: pointer arithmetic and array indexing. One of the core features of this profile is to restrict pointers to only refer to single objects, not arrays.
>
>For the purposes of this document, bounds-safety is defined to be the property that a program does not use a variable to access memory outside of the range that was allocated and assigned to that variable. (Note that the safety is intended to be complete when combined also with [Type safety](#SS-type) and [Lifetime safety](#SS-lifetime), which cover other unsafe operations that allow bounds violations, such as type-unsafe casts that 'widen' pointers.)
>
>The following are under consideration but not yet in the rules below, and may be better in other profiles:
>
>   -
>
>An implementation of this profile shall recognize the following patterns in source code as non-conforming and issue a diagnostic.


<a name="Pro-bounds-arithmetic"></a>
### Bounds.1: 포인터 연산을 사용하지 말고, 대신  `array_view`를 사용하세요.

**근거**:
포인터는 단일 객체를 참조할때만 써야 하며, 포인터 연산은 잘못되기 쉽습니다. `array_view`는 범위 체크가 되어서 배열 안의 데이터를 제어하기에 안전한 타입입니다.

>### Bounds.1: Don't use pointer arithmetic. Use `array_view` instead.
>
>**Reason**:
Pointers should only refer to single objects, and pointer arithmetic is fragile and easy to get wrong. `array_view` is a bounds-checked, safe type for accessing arrays of data.

**잘못된 예**:

    void f(int* p, int count)
    {
        if (count < 2) return;

        int* q = p + 1; // BAD

        ptrdiff_t d;
        int n;
        d = (p - &n); // OK
        d = (q - p); // OK

        int n = *p++; // BAD

        if (count < 6) return;

        p[4] = 1; // BAD

        p[count - 1] = 2; // BAD

        use(&p[0], 3); // BAD
    }

**올바른 예**:

    void f(array_view<int> a) // BETTER: 함수 선언에 array_view를 사용하였습니다.
    {
        if (a.length() < 2) return;

        int n = *a++; // OK

        array_view<int> q = a + 1; // OK

        if (a.length() < 6) return;

        a[4] = 1; // OK

        a[count – 1] = 2; // OK

        use(a.data(), 3); // OK
    }

>**Example; bad**:

    void f(int* p, int count)
    {
        if (count < 2) return;

        int* q = p + 1; // BAD

        ptrdiff_t d;
        int n;
        d = (p - &n); // OK
        d = (q - p); // OK

        int n = *p++; // BAD

        if (count < 6) return;

        p[4] = 1; // BAD

        p[count - 1] = 2; // BAD

        use(&p[0], 3); // BAD
    }

>**Example; good**:

    void f(array_view<int> a) // BETTER: use array_view in the function declaration
    {
        if (a.length() < 2) return;

        int n = *a++; // OK

        array_view<int> q = a + 1; // OK

        if (a.length() < 6) return;

        a[4] = 1; // OK

        a[count – 1] = 2; // OK

        use(a.data(), 3); // OK
    }

**시행하기**:
포인터 타입 표현식에 대한 연산식에의해  포인터 타입의 결과값이 어떻게 되는지 진단하세요.

>**Enforcement**:
Issue a diagnostic for any arithmetic operation on an expression of pointer type that results in a value of pointer type.


<a name="Pro-bounds-arrayindex"></a>
### Bounds.2: 배열의 인덱스로 상수표현식 만을 사용하세요.

**근거**:
배열에 동적으로 제어하면서 안전성을 확인하는 것은 사람이든 툴이든 어렵습니다. 
`array_view`는 범위체크를 해주어서 배열의 데이터를 제어하기에 안전한 타입입니다.
`at()`는 하나의 요소를 제어하면서 범위체크를 해줍니다.
배열을 제어하기 위해 반복자(iterator)를 사용해야할 경우라면 배열로 부터 생성한 `array_view`의 반복자를 사용하세요.

>### Bounds.2: Only index into arrays using constant expressions.
>
>**Reason**:
Dynamic accesses into arrays are difficult for both tools and humans to validate as safe. `array_view` is a bounds-checked, safe type for accessing arrays of data. `at()` is another alternative that ensures single accesses are bounds-checked. If iterators are needed to access an array, use the iterators from an `array_view` constructed over the array.

**잘못된 예**:

    void f(array<int,10> a, int pos)
    {
        a[pos/2] = 1; // BAD
        a[pos-1] = 2; // BAD
        a[-1] = 3;    // BAD - 잘못된 인덱스
        a[10] = 4;    // BAD - 잘못된 인덱스
    }

**올바른 예**:

    // 대안 A: array_view 사용

	// A1: 인자 타입을 array_view로 변경
    void f(array_view<int,10> a, int pos)
    {
        a[pos/2] = 1; // OK
        a[pos-1] = 2; // OK
    }

    // A2: 내부적으로 array_view를 선언해서 사용
    void f(array<int,10> arr, int pos)
    {
        array_view<int> a = arr, int pos)
        a[pos/2] = 1; // OK
        a[pos-1] = 2; // OK
    }

    // 대안 B: 제어하는데 at() 사용
    void f()(array<int,10> a, int pos)
    {
        at(a, pos/2) = 1; // OK
        at(a, pos-1) = 2; // OK
    }

**잘못된 예**:

    void f()
    {
        int arr[COUNT];
        for (int i = 0; i < COUNT; ++i)
            arr[i] = i; // BAD, 비상수 값을 인덱스로 전달하면 안됨
    }

**올바른 예**:

    // 대안 A: array_view 사용
    void f()
    {
        int arr[COUNT];
		array_view<int> av = arr;
        for (int i = 0; i < COUNT; ++i)
            av[i] = i;
    }

    // 대안 B:제어하는데 at() 사용
    void f()
    {
        int arr[COUNT];
        for (int i = 0; i < COUNT; ++i)
            at(arr,i) = i;
    }

>**Example; bad**:

    void f(array<int,10> a, int pos)
    {
        a[pos/2] = 1; // BAD
        a[pos-1] = 2; // BAD
        a[-1] = 3;    // BAD - no replacement, just don't do this
        a[10] = 4;    // BAD - no replacement, just don't do this
    }

>**Example; good**:

    // ALTERNATIVE A: Use an array_view

	// A1: Change parameter type to use array_view
    void f(array_view<int,10> a, int pos)
    {
        a[pos/2] = 1; // OK
        a[pos-1] = 2; // OK
    }

    // A2: Add local array_view and use that
    void f(array<int,10> arr, int pos)
    {
        array_view<int> a = arr, int pos)
        a[pos/2] = 1; // OK
        a[pos-1] = 2; // OK
    }

    // ALTERNATIVE B: Use at() for access
    void f()(array<int,10> a, int pos)
    {
        at(a, pos/2) = 1; // OK
        at(a, pos-1) = 2; // OK
    }

>**Example; bad**:

    void f()
    {
        int arr[COUNT];
        for (int i = 0; i < COUNT; ++i)
            arr[i] = i; // BAD, cannot use non-constant indexer
    }

>**Example; good**:

    // ALTERNATIVE A: Use an array_view
    void f()
    {
        int arr[COUNT];
		array_view<int> av = arr;
        for (int i = 0; i < COUNT; ++i)
            av[i] = i;
    }

    // ALTERNATIVE B: Use at() for access
    void f()
    {
        int arr[COUNT];
        for (int i = 0; i < COUNT; ++i)
            at(arr,i) = i;
    }


**시행하기**:
컴파일 타임에 상수표현식이 아닌 값을 인덱스로 사용하는 배열 타입 변수 (정적 배열, `std::array` 등...) 이나 인덱스 표현식을 진단하세요.

배열 범위 밖의 값을 인덱스로 사용하는 배열 타입 변수 (정적 배열, `std::array` 등...) 이나 인덱스 표현식을 진단하세요.

>**Enforcement**:
Issue a diagnostic for any indexing expression on an expression or variable of array type (either static array or `std::array`) where the indexer is not a compile-time constant expression.

>Issue a diagnostic for any indexing expression on an expression or variable of array type (either static array or `std::array`) where the indexer is not a value between `0` or and the upper bound of the array.

**수정 지원**: IDE에서 동적 인덱스 표현식으로 배열을 제어하는 곳을 `at()`로 고치도록 제안할 수 있습니다 :

    static int a[10];

    void f(int i, int j)
    {
    	a[i + j] = 12; 		// BAD, 아래와 같이 수정해야 합니다.
        at(a, i + j) = 12; 	// OK - 범위 체크
    }

>**Rewrite support**: Tooling can offer rewrites of array accesses that involve dynamic index expressions to use `at()` instead:

    static int a[10];

    void f(int i, int j)
    {
    	a[i + j] = 12; 		// BAD, could be rewritten as...
        at(a, i + j) = 12; 	// OK - bounds-checked
    }

<a name="Pro-bounds-decay"></a>
### Bounds.3: 배열을 포인터로 붕괴시키지 맙시다.

**근거**:
포인터를 배열처럼 사용할 수 없습니다. 배열의 값을 제어하는데 포인터를 사용하지 말고 범위체크가 되는 `array_view`를 사용하는 것이 안전한 대안입니다.

>### Bounds.3: No array-to-pointer decay.
>
>**Reason**:
Pointers should not be used as arrays. `array_view` is a bounds-checked, safe alternative to using pointers to access arrays.


**잘못된 예**:

    void g(int* p, size_t length);

    void f()
    {
        int a[5];
        g(a, 5);        // BAD
        g(&a[0], 1);    // OK
    }

**올바른 예**:

    void g(int* p, size_t length);
    void g1(array_view<int> av); // BETTER: get g() changed.

    void f()
    {
        int a[5];
        array_view av = a;

        g(a.data(), a.length());    // OK, 다른 대안이 없는 경우라면
        g1(a);                      // OK - 묵시적 aray_biew 생성자를 사용함으로 붕괴현상이 발생하지 않음
    }

>**Example; bad**:

    void g(int* p, size_t length);

    void f()
    {
        int a[5];
        g(a, 5);        // BAD
        g(&a[0], 1);    // OK
    }

>**Example; good**:

    void g(int* p, size_t length);
    void g1(array_view<int> av); // BETTER: get g() changed.

    void f()
    {
        int a[5];
        array_view av = a;

        g(a.data(), a.length());	// OK, if you have no choice
        g1(a);                      // OK - no decay here, instead use implicit array_view ctor
    }


**시행하기**:
배열을 포인터로 묵시적 변환하는 표현들을 찾아서 진단하세요.

>**Enforcement**:
Issue a diagnostic for any expression that would rely on implicit conversion of an array type to a pointer type.


<a name="Pro-bounds-stdlib"></a>
### Bounds.4: 범위체크를 하지 않는 표준 라이브러리 함수나 타입을 사용하지 마세요.

**근거**:
이런 함수들은 모두 `array_view`를 활용한 범위 안전 중복정의(overload)를 가지고 있습니다.
`vector`와 같은 표준 타입은 범위 프로파일 상의 범위체크를 하도록 수정하거나 (제약사항을 추가하는 등의 호환되는 방식), `at()`을 사용하세요.

>### Bounds.4: Don't use standard library functions and types that are not bounds-checked.
>
>**Reason**:
These functions all have bounds-safe overloads that take `array_view`. Standard types such as `vector` can be modified to perform bounds-checks under the bounds profile (in a compatible way, such as by adding contracts), or used with `at()`.

**잘못된 예**:

    void f()
    {
        array<int,10> a, b;
        memset(a.data(), 0, 10); 	// BAD, 길이 부분이 잘못되었습니다.
        memcmp(a.data(), b.data(), 10); // BAD, 길이 부분이 잘못되었습니다.
    }

**올바른 예**:

    void f()
    {
        array<int,10> a, b;
        memset(a, 0); 	// OK
        memcmp({a,b}); 	// OK
    }

>**Example; bad**:

    void f()
    {
        array<int,10> a, b;
        memset(a.data(), 0, 10); 		// BAD, and contains a length error
        memcmp(a.data(), b.data(), 10); // BAD, and contains a length error
    }

>**Example; good**:

    void f()
    {
        array<int,10> a, b;
        memset(a, 0); 	// OK
        memcmp({a,b}); 	// OK
    }

**예**: 
코드가 수정되지 않은 표준 라이브러리를 사용하는 경우, `std::array`와 `std::vector`를 범위 안전성 방법으로 사용함으로 해결하는 방법이 있습니다.
`std::out_of_range` 예외가 발생하는 클래스는 `.at()` 멤버 함수를 호출 할 수 있습니다.
또한, 범위 위반에서 fail-fast (또는 사용자 정의 행동)을 하는 경우 `at()` 함수를 이용 할 수도 있습니다.

    void f(std::vector<int>& v, std::array<int, 12> a, int i)
    {
        v[0] = a[0];        // BAD
        v.at(0) = a[0];     // OK (대안 1)
        at(v, 0) = a[0];    // OK (대안 2)

        v.at(0) = a[i];     // BAD
        v.at(0) = a.at(i)   // OK (대안 1)
        v.at(0) = at(a, i); // OK (대안 2)
    }

>**Example**: If code is using an unmodified standard library, then there are still workarounds that enable use of `std::array` and `std::vector` in a bounds-safe manner. Code can call the `.at()` member function on each class, which will result in an `std::out_of_range` exception being thrown. Alternatively, code can call the `at()` free function, which will result in fail-fast (or a customized action) on a bounds violation.

    void f(std::vector<int>& v, std::array<int, 12> a, int i)
    {
        v[0] = a[0];        // BAD
        v.at(0) = a[0];     // OK (alternative 1)
        at(v, 0) = a[0];    // OK (alternative 2)

        v.at(0) = a[i];     // BAD
        v.at(0) = a.at(i)   // OK (alternative 1)
        v.at(0) = at(a, i); // OK (alternative 2)
    }

**시행하기**:
   - 범위검사를 하지 않는 표준 라이브러리 함수를 호출하는 부분을 진단하세요. ??? 해당 함수 목록에 대한 링크 추가 예정

>**Enforcement**:
   - Issue a diagnostic for any call to a standard library function that is not bounds-checked. ??? insert link to a list of banned functions


**앞으로 작업할 내용**:
   - 이미 표준화가 된 것에 대한 것이라도 호환성을 보장하고자 한다면 WG21 위원회와의 긴밀한 협조를 하여 표준 라이브러리에 반영해야 합니다.
   - `memcmp`의 경우같이 범위 안전성이 보장되는 stdlib의 항목에 대해서 GSL에 기재하는 것을 고려하고 있습니다.
   - 이번 장의 목적은 `vector`의 경우같이 stdlib의 함수나 타입 중 완벽하게 범위 체크가 되지 않는 항목들에 대해서 범위 프로파일을 적용하여 범위 안정성을 확복하는 것입니다. 그리고, 예전(legacy) 코드에서 범위 안정성을 확인하지 않는 부분에 대해서는 계속해서 WG21의 여러 회원들에게 제안하고자 합니다.

>**TODO Notes**:
   - Impact on the standard library will require close coordination with WG21, if only to ensure compatibility even if never standardized.
   - We are considering specifying bounds-safe overloads for stdlib (especially C stdlib) functions like `memcmp` and shipping them in the GSL.
   - For existing stdlib functions and types like `vector` that are not fully bounds-checked, the goal is for these features to be bounds-checked when called from code with the bounds profile on, and unchecked when called from legacy code, possibly using constracts (concurrently being proposed by several WG21 members).




<a name="SS-lifetime"></a>
## 수명 안전성 프로파일

>## Lifetime safety profile
