
# <a name="S-profile"></a>Pro: 프로필(Profiles)

이상적으로는 우리의 코드가 이 가이드라인의 모든 규칙을 따를 것이다.
그렇게 함으로써 깔끔하고, 규칙적이면서 오류에 취약하지도 않은 코드가 될 것이다. 어쩌면 가장 빠른 코드가 될수도 있다.
불행하게도, 그렇게 되는것은 불가능에 가까운데, 보통 우리가 작성하는 코드가 이미 존재하는 코드들에 맞추거나 이미 존재하는 라이브러리들을 사용해야 하기 때문이다.
그런 코드가 수십년간 작성되어 왔고, 그 코드들은 이 가이드라인을 따르지 않는다.
[점진적으로 적용](./appendix/Modernizing.md)하는 것을 목표해야 한다.

점진적으로 적용하기 위한 전략이 무엇이던간에, 어떤 문제들에 일련의 서로 연관된 가이드라인들을 적용할 수 있게 되고, 나머지 문제들은 나중을 위해 남겨놓을 필요가 있다.
언제나 그런 것은 아니지만 "연관된 가이드라인들"과 같은 생각이 중요할 떄가 있다.
가이드라인이 이미 작성된 코드(code base)와 관련있다고 여겨지거나 특별한 어플리케이션 영역에 적용되기 위한 일련의 특별한 가이드라인이 적용되면
우리는 그런 가이드라인 묶음들을 "프로필"이라고 부른다.
우리의 의도는 이런 가이드라인들이 일관성을 가지고 "범위 오류들을 없애는 것"이나 "정적 타입 안전성"과 같은 특정한 목적을 이루는 것을 돕는 것이다.
각 프로필들은 오류의 한 종류(class)를 없애기 위해 설계되었다.
개별적인 규칙을을 "아무거나" 따르는 것은 이미 존재하는 코드에 해당 규칙에서 정의하는 개선(improvement)보다 교란(disruptive)을 낳을 가능성이 크다.

"프로필"은 결정론적(deterministic)이고 어디에서도 적용 가능(portably enforceable)하며 특정한 보증(specific guarantee)을 달성하기 위해 설계된 규칙(즉, 규정)들의 부분집합을 말한다.
"결정론적"이라는 표현의 의미는 지역적인 분석만을 필요로 하고 컴파일러 안에서 구현할 수 있는 것(비록 그럴 필요가 없더라도)을 의미한다.
"어디에도 적용가능"하다는 해당 규칙들이 언어의 규칙과 비슷하다는 것을 말한다. 따라서 프로그래머들은 같은 코드에 대해서 같은 해답을 제시하는 서로 다른 도구들을 사용할 수 있다.

언어 프로필을 사용해 경고가 없도록 작성된 코드는 프로필에 부합하는(conform) 것으로 생각할 수 있다.
프로필에 부합하는 코드는 작성하는 순간부터 해당 프로필이 목표하는 안전성 속성(safety property)에 대해서는 안전하다고 볼 수 있을것이다.
프로필에서 목표하는 속성에 대한 오류가 발생했을 때, 설령 해당 오류가 다른 코드, 라이브러리 혹은 외부 환경에 의해서 프로그램에 생겨났더라도 프로필에 부합하는 코드가 그 오류의 근본 원인(root cause)이 되지는 않을 것이다. 
프로필에서는 쉽고 정확하게 코드를 작성하도록 유도하기 위해 새로운 라이브러리 타입을 제시할수도 있다.

프로필 요약:

* [Pro.type: 타입 안전성](#SS-type)
* [Pro.bounds: 경계 안전성](#SS-bounds)
* [Pro.lifetime: 수명 안전성](#SS-lifetime)

미래에는, 훨씬 더 많은 프로필을 정의하고 이미 존재하는 프로필들에 검사 방법을 추가하기를 기대한다.
그 후보들로는 다음과 같은 것들이 있다:

* narrowing arithmetic promotions/conversions (likely part of a separate safe-arithmetic profile)
* arithmetic cast from negative floating point to unsigned integral type (ditto)
* selected undefined behavior: Start with Gabriel Dos Reis's UB list developed for the WG21 study group
* selected unspecified behavior: Addressing portability concerns.
* `const` violations: Mostly done by compilers already, but we can catch inappropriate casting and underuse of `const`.

프로필을 적용할 수 있도록 하는것은 구현에 달려있다(implementation defined); 보편적으로, 이는 사용되는 분석 도구들에 달려있다.

프로필 검사를 수행하지 않도록 만들고 싶다면, 언어의 Contract를 사용해서 `suppress`라고 표기하라. 예를 들어:

```c++
    [[suppress(bounds)]] char* raw_find(char* p, int n, char x)
        // find x in p[0]..p[n - 1]
    {
        // ...
    }
```

이제 `raw_find()`는 주어진 내용에 마음대로(scramble) 사용할 수 있다.
당연히, 이런 억제(suppression)는 거의 사용되지 않아야 한다.

## <a name="SS-type"></a>Pro.safety: 타입 안전성 프로필

이 프로필은 타입을 정확하게 사용하고, 부주의하게(inadvertent) 타입을 조작하지 않는 코드를 작성하는 것을 돕는다.
그 방법으로 타입을 위배(violate)하는 궁극적인 원인을 제거하는데 집중한다. 
원인에는 타입 변환을 사용하거나 공용체(`union`)의 사용이 포함된다.

이 목적을 위해서, 타입 안전성은 임의의 변수가 그 타입에서 정의한 규칙을 준수하지 않는 방법으로 사용되지 않는 것으로 정의한다.
타입 `T`로 접근하는 메모리 영역이 실제로는 전혀 상관없는 타입 `U`의 개체를 담고 있어서는 안된다.
타입 안전성이 [경계 안전성](#SS-bounds), [수명 안전성](#SS-lifetime)과 같이 충족되어야 완전해지도록 의도되었다는 점에 유의하라.

이 프로필의 구현체는 소스코드에서 아래의 패턴에 맞지 않는 부분을 찾아내고 진단할 수 있어야 한다.

타입 안전성 프로필 요약:

* <a name="Pro-type-avoidcasts"></a>Type.1: [Avoid casts](./Expr.md#Res-casts):
  * <a name="Pro-type-reinterpretcast"></a>Don't use `reinterpret_cast`; A strict version of [Avoid casts](#Res-casts) and [prefer named casts](./Expr.md#Res-casts-named)
  * <a name="Pro-type-arithmeticcast"></a>Don't use `static_cast` for arithmetic types; A strict version of [Avoid casts](./Expr.md#Res-casts) and [prefer named casts](./Expr.md#Res-casts-named).
  * <a name="Pro-type-identitycast"></a>Don't cast between pointer types where the source type and the target type are the same; A strict version of [Avoid casts](./Expr.md#Res-casts).
  * <a name="Pro-type-implicitpointercast"></a>Don't cast between pointer types when the conversion could be implicit; A strict version of [Avoid casts](./Expr.md#Res-casts).
* <a name="Pro-type-downcast"></a>Type.2: Don't use `static_cast` to downcast: [Use `dynamic_cast` instead](./Class.md#Rh-dynamic_cast).
* <a name="Pro-type-constcast"></a>Type.3: Don't use `const_cast` to cast away `const` (i.e., at all): [Don't cast away const](./Expr.md#Res-casts-const).
* <a name="Pro-type-cstylecast"></a>Type.4: Don't use C-style `(T)expression` or functional `T(expression)` casts: Prefer [construction](./Expr.md#Res-construct) or [named casts](./Expr.md#Res-cast-named).
* <a name="Pro-type-init"></a>Type.5: Don't use a variable before it has been initialized: [always initialize](./Expr.md#Res-always).
* <a name="Pro-type-memberinit"></a>Type.6: Always initialize a member variable: [always initialize](./Expr.md#Res-always), possibly using [default constructors](./Class.md#Rc-default0) or [default member initializers](./Class.md#Rc-in-class-initializers).
* <a name="Pro-type-unon"></a>Type.7: Avoid naked union: [Use `variant` instead](./Class.md#Ru-naked).
* <a name="Pro-type-varargs"></a>Type.8: Avoid varargs: [Don't use `va_arg` arguments](./Functions.md#F-varargs).

##### Impact

타입 안전성 프로필을 준수하면 모든 처리(operation)들이 유효한 개체에서 수행된다고 확신할 수 있다.
오류를 알리기 위해서 발생하는 예외는 정적으로는(컴파일 시간에) 탐지해낼 수 없다.
타입 안전성이 [경계 안전성](#SS-bounds), [수명 안전성](#SS-lifetime)과 같이 충족되어야 완전해지도록 의도되었다는 점에 유의하라.
이를 보장할 수 없다면, 임의의 메모리 영역에 어떤 개체, 개체들, 혹은 개체의 일부가 저장되어 있는지와 상관없는 접근이 발생할 수 있다.

## <a name="SS-bounds"></a>Pro.bounds: 경계(bound) 안전성 프로필

이 프로필은 메모리의 할당된 블록들의 경계 내에서 동작하는 코드를 작성하는 것을 돕는다
그 방법으로 경계를 위반하는 궁극적인 원인을 제거하는데 집중한다:
바로 포인터를 계산하거나 배열의 인덱스를 사용하는 부분이다.
이 프로필의 핵심적인 특징 중 하나는 포인터가 배열이 아니라 오직 하나의 개체만을 가리키도록 제한한다는 것이다. 

경계 안전성은 프로그램이 할당된 구간(range)을 벗어난 메모리에 위치한 개체를 사용하지 않는 것을 의미한다.
경계 안전성은 타입 안전성과 수명 안전성과 같이 충족되어야 완전해지도록 의도되었다. 다른 안전성들이 경계 위반을 허용하는 안전하지 않은 처리들에 대해 다룰 것이다.

경계 안전성 프로필 요약:

* <a href="Pro-bounds-arithmetic"></a>Bounds.1: Don't use pointer arithmetic. Use `span` instead: [Pass pointers to single objects (only)](#Ri-array) and [Keep pointer arithmetic simple](#Res-ptr).
* <a href="Pro-bounds-arrayindex"></a>Bounds.2: Only index into arrays using constant expressions: [Pass pointers to single objects (only)](#Ri-array) and [Keep pointer arithmetic simple](#Res-ptr).
* <a href="Pro-bounds-decay"></a>Bounds.3: No array-to-pointer decay: [Pass pointers to single objects (only)](#Ri-array) and [Keep pointer arithmetic simple](#Res-ptr).
* <a href="Pro-bounds-stdlib"></a>Bounds.4: Don't use standard-library functions and types that are not bounds-checked: [Use the standard library in a type-safe manner](#Rsl-bounds).

##### Impact

경계 안전성을 따르면 개체에 - 특히 배열에 - 접근할 때 개체에 할당된 메모리 너머에 접근하지 않게 된다.
이는 (악명높은) "버퍼 오버플로우"를 비롯해 찾기 어려운 오류들을 소멸시킨다.
이는 (경계를 벗어나서 값을 변경할 때 발생하는) 메모리 오염(corruption)의 유명한 원인을 비롯해 보안 약점도 함께 막는다.
경계를 벗어난 접근이 "단순히 읽기"만 수행하더라도, (접근 대상이 의도한 타입이 아닌 경우) 불변조건을 위반하거나 "이상한 값"을 반환할 수도 있다.

## <a name="SS-lifetime"></a>Pro.lifetime: 수명 안전성 프로필

아무것도 가리키지 않는 포인터를 통해 접근하는 것은 오류의 주 원인이다.
또한 전통적인 C 혹은 C++ 스타일의 프로그래밍은 이 문제를 피하기 매우 어렵다.
예를 들어 포인터가 초기화되지 않았거나, `nullptr`이거나, 배열의 범위를 벗어나거나, 삭제된 개체일 수 있다.
[현재 디자인 명세를 함께 보라](https://github.com/isocpp/CppCoreGuidelines/blob/master/docs/Lifetime.pdf).

수명 안전성 프로필 요약:

* <a href="Pro-lifetime-invalid-deref"></a>Lifetime.1: Don't dereference a possibly invalid pointer:[detect or avoid](#Res-deref).

##### Impact

코딩 스타일 규칙, 정적 분석, 라이브러리 지원이 완전히 함께 적용되면 이 프로필은

* C++의 짜증나는(nasty) 오류의 주 원인 중 하나를 없애버린다
* 잠재적인 보안 문제의 원인을 없앤다
* "편집증적인(paranoia)" 검사를 없애 성능을 향상시킨다
* 코드의 정확함에 자신감을 가지게 한다
* C++ 언어의 핵심 규칙을 따름으로써 미정의 행동이 발생하지 않게 한다.
