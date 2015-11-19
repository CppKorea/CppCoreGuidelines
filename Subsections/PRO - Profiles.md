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

For the purposes of this section, type-safety is defined to be the property that a program does not use a variable as a type it is not. Memory accessed as a type `T` should not be valid memory that actually contains an object of an unrelated type `U`. (Note that the safety is intended to be complete when combined also with [Bounds safety](#SS-bounds) and [Lifetime safety](#SS-lifetime).)

The following are under consideration but not yet in the rules below, and may be better in other profiles:

   - narrowing arithmetic promotions/conversions (likely part of a separate safe-arithmetic profile)
   - arithmetic cast from negative floating point to unsigned integral type (ditto)
   - selected undefined behavior: ??? this is a big bucket, start with Gaby's UB list
   - selected unspecified behavior: ??? would this really be about safety, or more a portability concern?
   - constness violations? if we rely on it for safety

An implementation of this profile shall recognize the following patterns in source code as non-conforming and issue a diagnostic.


<a name="Pro-type-reinterpretcast"></a>
### Type.1: Don't use `reinterpret_cast`.

**Reason**:
Use of these casts can violate type safety and cause the program to access a variable that is actually of type `X` to be accessed as if it were of an unrelated type `Z`.

**Example; bad**:

    std::string s = "hello world";
    double* p = reinterpret_cast<double*>(&s); // BAD

**Enforcement**: Issue a diagnostic for any use of `reinterpret_cast`. To fix: Consider using a `variant` instead.


<a name="Pro-type-downcast"></a>
### Type.2: Don't use `static_cast` downcasts. Use `dynamic_cast` instead.

**Reason**:
Use of these casts can violate type safety and cause the program to access a variable that is actually of type `X` to be accessed as if it were of an unrelated type `Z`.

**Example; bad**:

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

**Enforcement**: Issue a diagnostic for any use of `static_cast` to downcast, meaning to cast from a pointer or reference to `X` to a pointer or reference to a type that is not `X` or an accessible base of `X`. To fix: If this is a downcast or cross-cast then use a `dynamic_cast` instead, otherwise consider using a `variant` instead.


<a name="Pro-type-constcast"></a>
### Type.3: Don't use `const_cast` to cast away `const` (i.e., at all).

**Reason**:
Casting away `const` is a lie. If the variable is actually declared `const`, it's a lie punishable by undefined behavior.

**Example; bad**:

    void f(const int& i) {
        const_cast<int&>(i) = 42;   // BAD
    }

    static int i = 0;
    static const int j = 0;

    f(i); // silent side effect
    f(j); // undefined behavior
    

**Exception**: You may need to cast away `const` when calling `const`-incorrect functions. Prefer to wrap such functions in inline `const`-correct wrappers to encapsulate the cast in one place.

**Enforcement**: Issue a diagnostic for any use of `const_cast`. To fix: Either don't use the variable in a non-`const` way, or don't make it `const`.


<a name="Pro-type-cstylecast"></a>
### Type.4: Don't use C-style `(T)expression` casts that would perform a `static_cast` downcast, `const_cast`, or `reinterpret_cast`.

**Reason**:
Use of these casts can violate type safety and cause the program to access a variable that is actually of type `X` to be accessed as if it were of an unrelated type `Z`.
Note that a C-style `(T)expression` cast means to perform the first of the following that is possible: a `const_cast`, a `static_cast`, a `static_cast` followed by a `const_cast`, a `reinterpret_cast`, or a `reinterpret_cast` followed by a `const_cast`. This rule bans `(T)expression` only when used to perform an unsafe cast.

**Example; bad**:

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

**Enforcement**: Issue a diagnostic for any use of a C-style `(T)expression` cast that would invoke a `static_cast` downcast, `const_cast`, or `reinterpret_cast`. To fix: Use a `dynamic_cast`, `const`-correct declaration, or `variant`, respectively.



<a name="Pro-type-init"></a>
### Type.5: Don't use a variable before it has been initialized.

[ES.20: Always initialize an object](#Res-always) is required.



<a name="Pro-type-memberinit"></a>
### Type.6: Always initialize a member variable.

**Reason**:
Before a variable has been initialized, it does not contain a deterministic valid value of its type. It could contain any arbitrary bit pattern, which could be different on each call.

**Example**:

    struct X { int i; };

    X x;
    use(x); // BAD, x hs not been initialized

    X x2{}; // GOOD
    use(x2);

**Enforcement**:
   - Issue a diagnostic for any constructor of a non-trivially-constructible type that does not initialize all member variables. To fix: Write a data member initializer, or mention it in the member initializer list.
   - Issue a diagnostic when constructing an object of a trivially constructible type without `()` or `{}` to initialize its members. To fix: Add `()` or `{}`.


<a name="Pro-type-unions"></a>
### Type.7: Avoid accessing members of raw unions. Prefer `variant` instead.

**Reason**:
Reading from a union member assumes that member was the last one written, and writing to a union member assumes another member with a nontrivial destructor had its destructor called. This is fragile because it cannot generally be enforced to be safe in the language and so relies on programmer discipline to get it right.

**Example**:

    union U { int i; double d; };
    
    U u;
    u.i = 42;
    use(u.d); // BAD, undefined
    
    variant<int,double> u;
    u = 42; // u  now contains int
    use(u.get<int>()); // ok
    use(u.get<double>()); // throws ??? update this when standardization finalizes the variant design
    
Note that just copying a union is not type-unsafe, so safe code can pass a union from one piece of unsafe code to another.

**Enforcement**:
   - Issue a diagnostic for accessing a member of a union. To fix: Use a `variant` instead.
   
 

<a name="Pro-type-varargs"></a>
### Type.8: Avoid reading from varargs or passing vararg arguments. Prefer variadic template parameters instead.

**Reason**:
Reading from a vararg assumes that the correct type was actually passed. Passing to varargs assumes the correct type will be read. This is fragile because it cannot generally be enforced to be safe in the language and so relies on programmer discipline to get it right.

**Example**:

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

Note: Declaring a `...` parameter is sometimes useful for techniques that don't involve actual argument passing, notably to declare “take-anything” functions so as to disable "everything else" in an overload set or express a catchall case in a template metaprogram.
    
**Enforcement**:
   - Issue a diagnostic for using `va_list`, `va_start`, or `va_arg`. To fix: Use a variadic template parameter list instead.
   - Issue a diagnostic for passing an argument to a vararg parameter. To fix: Use a different function, or `[[suppress(types)]]`.



<a name="SS-bounds"></a>
## Bounds safety profile

This profile makes it easier to construct code that operates within the bounds of allocated blocks of memory. It does so by focusing on removing the primary sources of bounds violations: pointer arithmetic and array indexing. One of the core features of this profile is to restrict pointers to only refer to single objects, not arrays.

For the purposes of this document, bounds-safety is defined to be the property that a program does not use a variable to access memory outside of the range that was allocated and assigned to that variable. (Note that the safety is intended to be complete when combined also with [Type safety](#SS-type) and [Lifetime safety](#SS-lifetime), which cover other unsafe operations that allow bounds violations, such as type-unsafe casts that 'widen' pointers.)

The following are under consideration but not yet in the rules below, and may be better in other profiles:

   -

An implementation of this profile shall recognize the following patterns in source code as non-conforming and issue a diagnostic.


<a name="Pro-bounds-arithmetic"></a>
### Bounds.1: Don't use pointer arithmetic. Use `array_view` instead.

**Reason**:
Pointers should only refer to single objects, and pointer arithmetic is fragile and easy to get wrong. `array_view` is a bounds-checked, safe type for accessing arrays of data.

**Example; bad**:

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

**Example; good**:

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

**Enforcement**:
Issue a diagnostic for any arithmetic operation on an expression of pointer type that results in a value of pointer type.


<a name="Pro-bounds-arrayindex"></a>
### Bounds.2: Only index into arrays using constant expressions.

**Reason**:
Dynamic accesses into arrays are difficult for both tools and humans to validate as safe. `array_view` is a bounds-checked, safe type for accessing arrays of data. `at()` is another alternative that ensures single accesses are bounds-checked. If iterators are needed to access an array, use the iterators from an `array_view` constructed over the array.

**Example; bad**:

    void f(array<int,10> a, int pos)
    {
        a[pos/2] = 1; // BAD
        a[pos-1] = 2; // BAD
        a[-1] = 3;    // BAD - no replacement, just don't do this
        a[10] = 4;    // BAD - no replacement, just don't do this
    }

**Example; good**:

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

**Example; bad**:

    void f()
    {
        int arr[COUNT];
        for (int i = 0; i < COUNT; ++i)
            arr[i] = i; // BAD, cannot use non-constant indexer
    }

**Example; good**:

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


**Enforcement**:
Issue a diagnostic for any indexing expression on an expression or variable of array type (either static array or `std::array`) where the indexer is not a compile-time constant expression.

Issue a diagnostic for any indexing expression on an expression or variable of array type (either static array or `std::array`) where the indexer is not a value between `0` or and the upper bound of the array.

**Rewrite support**: Tooling can offer rewrites of array accesses that involve dynamic index expressions to use `at()` instead:

    static int a[10];

    void f(int i, int j)
    {
    	a[i + j] = 12; 		// BAD, could be rewritten as...
        at(a, i + j) = 12; 	// OK - bounds-checked
    }


<a name="Pro-bounds-decay"></a>
### Bounds.3: No array-to-pointer decay.

**Reason**:
Pointers should not be used as arrays. `array_view` is a bounds-checked, safe alternative to using pointers to access arrays.

**Example; bad**:

    void g(int* p, size_t length);

    void f()
    {
        int a[5];
        g(a, 5);        // BAD
        g(&a[0], 1);    // OK
    }

**Example; good**:

    void g(int* p, size_t length);
    void g1(array_view<int> av); // BETTER: get g() changed.

    void f()
    {
        int a[5];
        array_view av = a;

        g(a.data(), a.length());	// OK, if you have no choice
        g1(a);                      // OK - no decay here, instead use implicit array_view ctor
    }

**Enforcement**:
Issue a diagnostic for any expression that would rely on implicit conversion of an array type to a pointer type.


<a name="Pro-bounds-stdlib"></a>
### Bounds.4: Don't use standard library functions and types that are not bounds-checked.

**Reason**:
These functions all have bounds-safe overloads that take `array_view`. Standard types such as `vector` can be modified to perform bounds-checks under the bounds profile (in a compatible way, such as by adding contracts), or used with `at()`.

**Example; bad**:

    void f()
    {
        array<int,10> a, b;
        memset(a.data(), 0, 10); 		// BAD, and contains a length error
        memcmp(a.data(), b.data(), 10); // BAD, and contains a length error
    }

**Example; good**:

    void f()
    {
        array<int,10> a, b;
        memset(a, 0); 	// OK
        memcmp({a,b}); 	// OK
    }

**Example**: If code is using an unmodified standard library, then there are still workarounds that enable use of `std::array` and `std::vector` in a bounds-safe manner. Code can call the `.at()` member function on each class, which will result in an `std::out_of_range` exception being thrown. Alternatively, code can call the `at()` free function, which will result in fail-fast (or a customized action) on a bounds violation.

    void f(std::vector<int>& v, std::array<int, 12> a, int i)
    {
        v[0] = a[0];        // BAD
        v.at(0) = a[0];     // OK (alternative 1)
        at(v, 0) = a[0];    // OK (alternative 2)

        v.at(0) = a[i];     // BAD
        v.at(0) = a.at(i)   // OK (alternative 1)
        v.at(0) = at(a, i); // OK (alternative 2)
    }

**Enforcement**:
   - Issue a diagnostic for any call to a standard library function that is not bounds-checked. ??? insert link to a list of banned functions

**TODO Notes**:
   - Impact on the standard library will require close coordination with WG21, if only to ensure compatibility even if never standardized.
   - We are considering specifying bounds-safe overloads for stdlib (especially C stdlib) functions like `memcmp` and shipping them in the GSL.
   - For existing stdlib functions and types like `vector` that are not fully bounds-checked, the goal is for these features to be bounds-checked when called from code with the bounds profile on, and unchecked when called from legacy code, possibly using constracts (concurrently being proposed by several WG21 members).




<a name="SS-lifetime"></a>
## Lifetime safety profile
