
# <a name="S-gsl"></a>GSL: 가이드라인 지원 라이브러리

GSL 은 가이드라인 적용을 돕기 위해 설계된 작은 라이브러리다. 이런 기능이 없다면, 가이드라인은 언어 세부사항들에 대해 더 엄격해져야만 했을 것이다.

핵심 가이드라인 지원 라이브러리는 네임스페이스 `gsl`에 정의되어 있으며, 이 이름은 표준 라이브러리나 다른 잘 알려진 라이브러리에 대한 네임스페이스 별명일 수 있다. 이와 같은 (컴파일 시간) 우회는 지원 기능들에 대한 실험 혹은 변형을 가능하게 한다.

GSL은 헤더 파일로만 구성되며, [GSL: Guidelines support library](https://github.com/Microsoft/GSL)에서 확인할 수 있다.

이 라이브러리의 기능들은 전통적인 다른 방법들보다 오버헤드가 발생하지 않도록 극도로 가볍게(Zero-Overhead) 설계되었다. 디버깅 작업과 같이 특정 작업에 추가 기능(예를 들어 뭔가에 대한 확인)으로 "계측"할 때 사용될 수도 있다.

핵심 가이드라인에서는 `variant` 타입이 있다고 가정하지만, GSL에서는 이를 지원하지 않는다. [C++ 17에서 제안된 형태를 사용하라](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0088r3.html).

GSL 구성요소 요약:

* [GSL.view: Views](#SS-views)
* [GSL.owner](#SS-ownership)
* [GSL.assert: Assertions](#SS-assertions)
* [GSL.util: Utilities](#SS-utilities)
* [GSL.concept: Concepts](#SS-gsl-concepts)

We plan for a "ISO C++ standard style" semi-formal specification of the GSL.

ISO C++ 표준 라이브러리에 기반을 두며, GSL의 일부가 표준 라이브러리에 포함되기를 바란다.

## <a name="SS-views"></a>GSL.view: Views

이 타입들은 사용자가 소유권을 가진 포인터와 그렇지 않은 포인터들, 단일 개체에 대한 포인터와 첫번째 원소에 대한 포인터를 구분할 수 있도록 해준다

"view" 타입들은 자원을 소유하지 않는다.

참조 역시 자원을 소유하지 않는다. [R.4](#Rr-ref)를 참고하라. 
Note: References have many opportunities to outlive the objects they refer to (returning a local variable by reference, holding a reference to an element of a vector and doing `push_back`, binding to `std::max(x, y + 1)`, etc. The Lifetime safety profile aims to address those things, but even so `owner<T&>` does not make sense and is discouraged.

타입 이름들은 대부분 ISO 표준 라이브러리 스타일을 따른다(소문자와 '_'를 사용):

* `T*`      // The `T*` is not an owner, may be null; assumed to be pointing to a single element.
* `T&`      // The `T&` is not an owner and can never be a "null reference"; references are always bound to objects.

The "raw-pointer" notation (e.g. `int*`) is assumed to have its most common meaning; that is, a pointer points to an object, but does not own it.
Owners should be converted to resource handles (e.g., `unique_ptr` or `vector<T>`) or marked `owner<T*>`.

* `owner<T*>`   // a `T*` that owns the object pointed/referred to; may be `nullptr`.

`owner`는 소유권을 가진 포인터에 사용되며, 다음과 같은 이유로 이 타입은 적합한 리소스 핸들 타입으로 변환될 수 없다:

 * 변환 비용 발생
 * 포인터가 ABI에 사용되는 경우
 * 포인터가 리소스 핸들 구현의 일부인 경우

An `owner<T>` differs from a resource handle for a `T` by still requiring an explicit `delete`.

An `owner<T>` is assumed to refer to an object on the free store (heap).

If something is not supposed to be `nullptr`, say so:

* `not_null<T>`   // `T` is usually a pointer type (e.g., `not_null<int*>` and `not_null<owner<Foo*>>`) that may not be `nullptr`.
  `T` can be any type for which `==nullptr` is meaningful.

* `span<T>`       // `[p:p+n)`, constructor from `{p, q}` and `{p, n}`; `T` is the pointer type
* `span_p<T>`     // `{p, predicate}` `[p:q)` where `q` is the first element for which `predicate(*p)` is true
* `string_span`   // `span<char>`
* `cstring_span`  // `span<const char>`

A `span<T>` refers to zero or more mutable `T`s unless `T` is a `const` type.

"Pointer arithmetic" is best done within `span`s.
A `char*` that points to more than one `char` but is not a C-style string (e.g., a pointer into an input buffer) should be represented by a `span`.

* `zstring`    // a `char*` supposed to be a C-style string; that is, a zero-terminated sequence of `char` or `nullptr`
* `czstring`   // a `const char*` supposed to be a C-style string; that is, a zero-terminated sequence of `const` `char` or `nullptr`

논리적으로는, 마지막 두 별칭은 필요하지는 않지만, 하나의 `char`에 대한 포인터와 C 스타일 문자열에 대한 포인터를 명시적으로 구분하게 해준다. 0으로 끝나지 않는 문자열은 `zstring`보다는 `char*` 타입을 사용해야 한다.

Use `not_null<zstring>` for C-style strings that cannot be `nullptr`. ??? Do we need a name for `not_null<zstring>`? or is its ugliness a feature?

## <a name="SS-ownership"></a>GSL.owner: Ownership pointers

* `unique_ptr<T>`     // 독점적 소유권: `std::unique_ptr<T>`
* `shared_ptr<T>`     // 공유 소유권: `std::shared_ptr<T>` (a counted pointer)
* `stack_array<T>`    // A stack-allocated array. The number of elements are determined at construction and fixed thereafter. The elements are mutable unless `T` is a `const` type.
* `dyn_array<T>`      // ??? needed ??? A heap-allocated array. The number of elements are determined at construction and fixed thereafter.
  The elements are mutable unless `T` is a `const` type. Basically a `span` that allocates and owns its elements.

## <a name="SS-assertions"></a>GSL.assert: Assertions

* `Expects`     // precondition assertion. Currently placed in function bodies. Later, should be moved to declarations.
                // `Expects(p)` terminates the program unless `p == true`
                // `Expect` in under control of some options (enforcement, error message, alternatives to terminate)
* `Ensures`     // postcondition assertion. Currently placed in function bodies. Later, should be moved to declarations.

These assertions are currently macros (yuck!) and must appear in function definitions (only)
pending standard committee decisions on contracts and assertion syntax.
See [the contract proposal](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r1.pdf); using the attribute syntax,
for example, `Expects(p)` will become `[[expects: p]]`.

## <a name="SS-utilities"></a>GSL.util: Utilities

* `finally`        // `finally(f)` makes a `final_action{f}` with a destructor that invokes `f`
* `narrow_cast`    // `narrow_cast<T>(x)` is `static_cast<T>(x)`
* `narrow`         // `narrow<T>(x)` is `static_cast<T>(x)` if `static_cast<T>(x) == x` or it throws `narrowing_error`
* `[[implicit]]`   // "Marker" to put on single-argument constructors to explicitly make them non-explicit.
* `move_owner`     // `p = move_owner(q)` means `p = q` but ???
* `joining_thread` // a RAII style version of `std::thread` that joins.
* `index`          // a type to use for all container and array indexing (currently an alias for `ptrdiff_t`)

## <a name="SS-gsl-concepts"></a>GSL.concept: Concepts

이하의 개념들(타입의 동작에 대한 것들)은 Andrew Sutton의 라이브러리, Range 제안, ISO WG21 Palo Alto TR 에서 가져온 것이다. 이들은 ISO C++ 표준에 포함될 가능성이 높다.

표기법은 ISO WG21 [Concepts TS](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf)의 것을 따른다.
후술할 대부분의 개념들은 [the Ranges TS](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf)에 정의되어 있다.

* `Range`
* `String`   // ???
* `Number`   // ???
* `Sortable`
* `Pointer`  // A type with `*`, `->`, `==`, and default construction (default construction is assumed to set the singular "null" value); see [smart pointers](#SS-gsl-smartptrconcepts)
* `Unique_ptr`  // A type that matches `Pointer`, has move (not copy), and matches the Lifetime profile criteria for a `unique` owner type; see [smart pointers](#SS-gsl-smartptrconcepts)
* `Shared_ptr`   // A type that matches `Pointer`, has copy, and matches the Lifetime profile criteria for a `shared` owner type; see [smart pointers](#SS-gsl-smartptrconcepts)
* `EqualityComparable`   // ???Must we suffer CaMelcAse???
* `Convertible`
* `Common`
* `Boolean`
* `Integral`
* `SignedIntegral`
* `SemiRegular` // ??? Copyable?
* `Regular`
* `TotallyOrdered`
* `Function`
* `RegularFunction`
* `Predicate`
* `Relation`
* ...

### <a name="SS-gsl-smartptrconcepts"></a>GSL.ptr: Smart pointer concepts

See [Lifetimes paper](https://github.com/isocpp/CppCoreGuidelines/blob/master/docs/Lifetimes%20I%20and%20II%20-%20v0.9.1.pdf).
