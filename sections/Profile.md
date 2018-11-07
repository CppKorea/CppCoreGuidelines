
# <a name="S-profile"></a>Pro: 분석

Ideally, we would follow all of the guidelines.
That would give the cleanest, most regular, least error-prone, and often the fastest code.
Unfortunately, that is usually impossible because we have to fit our code into large code bases and use existing libraries.
Often, such code has been written over decades and does not follow these guidelines.
We must aim for [gradual adoption](#S-modernizing).

Whatever strategy for gradual adoption we adopt, we need to be able to apply sets of related guidelines to address some set
of problems first and leave the rest until later.
A similar idea of "related guidelines" becomes important when some, but not all, guidelines are considered relevant to a code base
or if a set of specialized guidelines is to be applied for a specialized application area.
We call such a set of related guidelines a "profile".
We aim for such a set of guidelines to be coherent so that they together help us reach a specific goal, such as "absence of range errors"
or "static type safety."
Each profile is designed to eliminate a class of errors.
Enforcement of "random" rules in isolation is more likely to be disruptive to a code base than delivering a definite improvement.

"프로파일"은 특정한 보장하기 위한 "결정성" "이식성" 규칙들(즉, 제한사항) 입니다.
여기서 "결정성"은 로컬 분석이 되어야 하며, 컴파일러에 의해 구현 될수 있음(꼭 그러지 않아도 됨)을 의미합니다.
"이식성"은 언어 규칙같이 같은 코드에 따른 그 결과가 같아야 한다는 것을 의미합니다.

이런 프로파일을 이용하여 경고가 없도록 작성된 코드는 프로파일에 일치하는 것이라 말할 수 있습니다.
규격에 부합하는 코드는 프로파일 상의 안전성을 지켜서 작성된 것이므로 안전하다고 판단할 수 있습니다.
규격에 부합하는 코드는 다른 코드나, 라이브러리 또는 외부환경에 의해서 에러가 발생할 수는 있어도 그 자체에서 오류가 발생하지 않습니다.
프로파일은 올바른 코드의 작성을 도와주는 라이브러리들을 소개해 드릴 것입니다.

분석 요약:

* [Pro.type: 타입 안전성](#SS-type)
* [Pro.bounds: 범위 안전성](#SS-bounds)
* [Pro.lifetime: 수명 안전성](#SS-lifetime)

In the future, we expect to define many more profiles and add more checks to existing profiles.
Candidates include:

* narrowing arithmetic promotions/conversions (likely part of a separate safe-arithmetic profile)
* arithmetic cast from negative floating point to unsigned integral type (ditto)
* selected undefined behavior: Start with Gabriel Dos Reis's UB list developed for the WG21 study group
* selected unspecified behavior: Addressing portability concerns.
* `const` violations: Mostly done by compilers already, but we can catch inappropriate casting and underuse of `const`.

Enabling a profile is implementation defined; typically, it is set in the analysis tool used.

To suppress enforcement of a profile check, place a `suppress` annotation on a language contract. For example:

```c++
    [[suppress(bounds)]] char* raw_find(char* p, int n, char x)    // find x in p[0]..p[n - 1]
    {
        // ...
    }
```

Now `raw_find()` can scramble memory to its heart's content.
Obviously, suppression should be very rare.

## <a name="SS-type"></a>Pro.safety: 타입 안전성 분석

이 유형의 프로파일은 타입을 정확히 사용하며, 부주의한 타입변형을 방지하면서 코드를 작성하도록 도와드릴 것입니다.
안전하지 않은 캐스팅과 `union`의 사용을 포함하여 타입이 잘못사용되는 것에 대한 주요 원인을 제거하는 것에 초점을 맞춰서 진행하겠습니다.

이 장의 목적인,
타입 안전성을 정의하자면, 프로그램 상에서 변수를 원래 타입과 다르게 사용하지 않는 것이라 하겠습니다. 실제 `U` 타입으로 정의된 객체가 저장된 메모리에 `T`타입으로 읽어서 사용하지 말자는 겁니다.
(타입 안전성 하나만 지켜서는 코드가 안전하다는 보장을 받기 힘듭니다. [범위 안전성](#SS-bounds)과 [수명 안전성](#SS-lifetime)을 함께 지켰을 때 비로서 완전히 안전성이 보장된다고 할 수 있습니다.)

An implementation of this profile shall recognize the following patterns in source code as non-conforming and issue a diagnostic.

Type safety profile summary:

* <a name="Pro-type-avoidcasts"></a>Type.1: [Avoid casts](#Res-casts):
  * <a name="Pro-type-reinterpretcast"></a>Don't use `reinterpret_cast`; A strict version of [Avoid casts](#Res-casts) and [prefer named casts](#Res-casts-named)
  * <a name="Pro-type-arithmeticcast"></a>Don't use `static_cast` for arithmetic types; A strict version of [Avoid casts](#Res-casts) and [prefer named casts](#Res-casts-named).
  * <a name="Pro-type-identitycast"></a>Don't cast between pointer types where the source type and the target type are the same; A strict version of [Avoid casts](#Res-casts).
  * <a name="Pro-type-implicitpointercast"></a>Don't cast between pointer types when the conversion could be implicit; A strict version of [Avoid casts](#Res-casts).
* <a name="Pro-type-downcast"></a>Type.2: Don't use `static_cast` to downcast: [Use `dynamic_cast` instead](#Rh-dynamic_cast).
* <a name="Pro-type-constcast"></a>Type.3: Don't use `const_cast` to cast away `const` (i.e., at all): [Don't cast away const](#Res-casts-const).
* <a name="Pro-type-cstylecast"></a>Type.4: Don't use C-style `(T)expression` or functional `T(expression)` casts: Prefer [construction](#Res-construct) or [named casts](#Res-cast-named).
* <a name="Pro-type-init"></a>Type.5: Don't use a variable before it has been initialized: [always initialize](#Res-always).
* <a name="Pro-type-memberinit"></a>Type.6: Always initialize a member variable: [always initialize](#Res-always), possibly using [default constructors](#Rc-default0) or [default member initializers](#Rc-in-class-initializers).
* <a name="Pro-type-unon"></a>Type.7: Avoid naked union: [Use `variant` instead](#Ru-naked).
* <a name="Pro-type-varargs"></a>Type.8: Avoid varargs: [Don't use `va_arg` arguments](#F-varargs).

##### Impact

With the type-safety profile you can trust that every operation is applied to a valid object.
Exception may be thrown to indicate errors that cannot be detected statically (at compile time).
Note that this type-safety can be complete only if we also have [Bounds safety](#SS-bounds) and [Lifetime safety](#SS-lifetime).
Without those guarantees, a region of memory could be accessed independent of which object, objects, or parts of objects are stored in it.

## <a name="SS-bounds"></a>Pro.bounds: 범위 안전성 분석

이 프로파일은 메모리 블록 할당 작업에 대해 코드 작성을 쉽게 해 줍니다.
포인터 연산, 배열 인덱스 연산 등에서 발생하는 범위 위반 사항을 제거하는 것에 초점을 맞춰서 진행하겠습니다.
이 프로파일의 주요 기능중 하나는 (하나의 객체가 아니라) 배열을 참조하는 포인터를 제한하자는 것입니다.

범위-안정성이란 변수가 할당된 범위 외부에서 해당 변수를 사용하지 않는 것을 의미합니다.
[타입 안전성](#SS-type)과 [수명 안전성](#SS-lifetime)을 함께 지켰을 때 범위 안전성도 그 의미가 있습니다.

Bounds safety profile summary:

* <a href="Pro-bounds-arithmetic"></a>Bounds.1: Don't use pointer arithmetic. Use `span` instead: [Pass pointers to single objects (only)](#Ri-array) and [Keep pointer arithmetic simple](#Res-ptr).
* <a href="Pro-bounds-arrayindex"></a>Bounds.2: Only index into arrays using constant expressions: [Pass pointers to single objects (only)](#Ri-array) and [Keep pointer arithmetic simple](#Res-ptr).
* <a href="Pro-bounds-decay"></a>Bounds.3: No array-to-pointer decay: [Pass pointers to single objects (only)](#Ri-array) and [Keep pointer arithmetic simple](#Res-ptr).
* <a href="Pro-bounds-stdlib"></a>Bounds.4: Don't use standard-library functions and types that are not bounds-checked: [Use the standard library in a type-safe manner](#Rsl-bounds).

##### Impact

Bounds safety implies that access to an object - notably arrays - does not access beyond the object's memory allocation.
This eliminates a large class of insidious and hard-to-find errors, including the (in)famous "buffer overflow" errors.
This closes security loopholes as well as a prominent source of memory corruption (when writing out of bounds).
Even an out-of-bounds access is "just a read", it can lead to invariant violations (when the accessed isn't of the assumed type)
and "mysterious values."

## <a name="SS-lifetime"></a>Pro.lifetime: 수명 안전성 분석

Accessing through a pointer that doesn't point to anything is a major source of errors,
and very hard to avoid in many traditional C or C++ styles of programming.
For example, a pointer may be uninitialized, the `nullptr`, point beyond the range of an array, or to a deleted object.

[See the current design specification here.](https://github.com/isocpp/CppCoreGuidelines/blob/master/docs/Lifetime.pdf)

Lifetime safety profile summary:

* <a href="Pro-lifetime-invalid-deref"></a>Lifetime.1: Don't dereference a possibly invalid pointer:[detect or avoid](#Res-deref).

##### Impact

Once completely enforced through a combination of style rules, static analysis, and library support, this profile

* eliminates one of the major sources of nasty errors in C++
* eliminates a major source of potential security violations
* improves performance by eliminating redundant "paranoia" checks
* increases confidence in correctness of code
* avoids undefined behavior by enforcing a key C++ language rule
