#GSL : 가이드라인 지원 라이브러리

># GSL: Guideline support library

GSL은 지원기능의 작은 라이브러리들에 대한 안내서입니다.
이런 기능들을 사용하지 않는다면, 훨씬 더 엄격하게 자세한 언어적인 정보에 대해서 신경써야 합니다.

>The GSL is a small library of facilities designed to support this set of guidelines.
Without these facilities, the guidelines would have to be far more restrictive on language details.

지원 라이브러리들은 `Guide` 네임스페이스에 정의되어 있으며, `std`나 다른 잘 알려진 라이브러리 이름으로 별칭이 지정되어 있는 경우가 많습니다.
`Guide` 네임스페이스로 (컴파일 타임에) 간접적으로 하면 지원되는 기능들의 여러가지 특성들을 실험할 수 있습니다. (?)

>The Core Guidelines support library is define in namespace `Guide` and the names may be aliases for standard library or other well-known library names.
Using the (compile-time) indirection through the `Guide` namespace allows for experimentation and for local variants of the support facilities.

지원 라이브러리는 매우 가벼워서 (제로-오버헤드) 기존에 사용하던 방법들에 비해 오버헤드가 더 생기지 않습니다.
어디에 적합하냐면, 디버깅 작업과 같이 특정 작업에 추가 기능 (예를 들어 뭔가에 대한 확인)으로 "계측"할 때 좋습니다.

>The support library facilities are designed to be extremely lightweight (zero-overhead) so that they impose no overhead compared to using conventional alternatives.
Where desirable, they can be "instrumented" with additional functionality (e.g., checks) for tasks such as debugging.

이 가이드라인은 `variant` 타입에 대해 가정을 합니다만 아직은 GSL에서 다루지는 않습니다. 왜냐면 표준화 위원회에서 수정중에 있기 때문입니다.

>These Guidelines assume a `variant` type, but this is not currently in GSL because the design is being actively refined in the standards committee.



<a name="SS-views"></a>
## GSL.view: Views

이 타입은 소유권이 있는 포인터와 없는 포인터, 하나의 객체를 가리키는 포인터와 배열의 첫번째 요소를 가리키는 포인터를 구분해줍니다.

"view"자체는 소유자가 될 수 없습니다. (소유 : 특정 객체에 대한 직접적인 소유권을 가지는 것)

참조도 소유자가 될 수 없습니다.

>These types allow the user to distinguish between owning and non-owning pointers and between pointers to a single object and pointers to the first element of a sequence.
>
>These "views" are never owners.
>
>References are never owners.

이름은 ISO 표준라이브러리 스타일에 가깝습니다. (소문자와 밑줄)

>The names are mostly ISO standard-library style (lower case and underscore):

* `T*`			// `T*`는 소유자가 아니며, `nullptr`일 것입니다. (1개의 요소를 가리키는 것으로 판단됩니다.)
* `char*`		// C스타일 string (0을 만나면 끝나는 문자열 배열); 아마 `nullptr` 이겠죠.
* `const char*`	        // 역시 C스타일 string; `nullptr` 이겠죠.
* `T&`			// `T&`는 소유자가 아닙니다. C++ 규칙에 의하면 아마 `&(T&)*nullptr` 형태로 해석될 것입니다.

>* `T*`			// The `T*` is not an owner, may be `nullptr` (Assumed to be pointing to a single element)
* `char*`		// A C-style string (a zero-terminated array of characters); can be `nullptr`
* `const char*`	// A C-style string; can be `nullptr`
* `T&`			// The `T&` is not an owner, may not be `&(T&)*nullptr` (language rule)

일반 포인터 표기법 (e.g. `int*`)의 일반적인 의미는 하나의 객체를 가리키는 포인터일 뿐이지 소유자는 아닙니다.
소유자는 리소스핸들 (e.g., `unique_ptr`, `vector<T>`)로 변환되거나 `owner<T*>`로 표현해야 합니다.

>The "raw-pointer" notation (e.g. `int*`) is assumed to have its most common meaning; that is, a pointer points to an object, but does not own it.
Owners should be converted to resource handles (e.g., `unique_ptr` or `vector<T>`) or marked `owner<T*>`

* `owner<T*>`		// `T*` 는 가리키는 객체를 소유합니다.; 여기서는 `nullptr`이겠죠.
* `owner<T&>`		// `T&`는 참조하는 객체를 소유합니다.

>* `owner<T*>`		// a `T*`that owns the object pointed/referred to; can be `nullptr`
* `owner<T&>`		// a `T&` that owns the object pointed/referred to

`owner`는 코드상에서 포인터에 대한 소유를 표시함으로써 리소스핸들을 수정하지 못하게 합니다.
다음과 같은 이유 때문입니다.

* 변환 비용 발생
* 포인터가 ABI에 사용되는 경우
* 포인터가 리소스핸들 구현의 일부인 경우

>`owner` is used to mark owning pointers in code that cannot be upgraded to use proper resource handles.
Reasons for that include
>
>* cost of conversion
* the pointer is used with an ABI
* the pointer is part of the implementation of a resource handle.

`owner<T>`는 명시적으로 `delete`를 해주어야 하는 `T`를 리소스 핸들로 사용하는 것과는 다릅니다.

`owner<T>`는 heap영역의 객체를 가리킵니다.

`nullptr`을 지원하지 않는다면 다음과 같이 사용하면 됩니다.

* `not_null<T>`		// `T` 는 `nullptr`이 아닌 포인터 타입 (e.g., `not_null<int*>` , `not_null<owner<Foo*>>`) 입니다. 
`T`는 `==nullptr`이 의미있는 어떤 타입도 가능합니다.

>An `owner<T>` differs from a resource handle for a `T` by still requiring and explicit `delete`.
>
>An `owner<T>` is assumed to refer to an object on the free store (heap).
>
>If something is not supposed to be `nullptr`, say so:
>
>* `not_null<T>`		// `T` is usually a pointer type (e.g., `not_null<int*>` and `not_null<owner<Foo*>>`) that may not be `nullptr`.
`T` can be any type for which `==nullptr` is meaningful.

* `array_view<T>`	// [`p`:`p+n`), `{p,q}`와 `{p,n}`로 부터의 생성자; `T`는 포인터 타입
* `array_view_p<T>`	// `{p,predicate}` [`p`:`q`) `q`는 `predicate(*p)`가 참인 첫번째 요소
* `string_view`		// `array_view<char>`
* `cstring_view`	// `array_view<const char>`

>* `array_view<T>`	// [`p`:`p+n`), constructor from `{p,q}` and `{p,n}`; `T` is the pointer type
* `array_view_p<T>`	// `{p,predicate}` [`p`:`q`) where `q` is the first element for which `predicate(*p)` is true
* `string_view`		// `array_view<char>`
* `cstring_view`	// `array_view<const char>`

`*_view<T>`는 하나 이상의 비상수 `T`를 참조합니다.

>A `*_view<T>` refer to zero or more mutable `T`s unless `T` is a `const` type.

"포인터 연산"은 `array_view`안에서 잘 수행됩니다.
C스타일 string(e.g. input buffer에 대한 포인터)가 아닌 뭔가를 가리키는 포인터인 `char*`는 `array_view`로 표시해야 합니다.
하나의 `char`에 대한 포인터를 표현하는 다른 좋은 방법은 없습니다. (`string_view{p,1}`는 가능합니다. `T`가 `char`인 경우의 `T*`는 C스타일 string으로 변환되지 않습니다.)

>"Pointer arithmetic" is best done within `array_view`s.
A `char*` that points to something that is not a C-style string (e.g., a pointer into an input buffer) should be represented by an `array_view`.
There is no really good way to say "pointer to a single `char` (`string_view{p,1}` can do that, and `T*` where `T` is a `char` in a template that has not been specialized for C-style strings).

* `zstring`		// C스타일 string을 지원하는 `char*`; NULL(0) 종료 문자열 혹은 `null_ptr`
* `czstring`	        // C스타일 string을 지원하는 `const char*`; NULL(0) 종료 상수문자열 혹은 `null_ptr`

>* `zstring`		// a `char*` supposed to be a C-style string; that is, a zero-terminated sequence of `char` or `null_ptr`
* `czstring`	// a `const char*` supposed to be a C-style string; that is, a zero-terminated sequence of `const` `char` ort `null_ptr`

논리적으로 마지막 2개의 별칭은 필요하지 않습니다만, (우리가 언제나 논리적인 것은 아니죠) 하나의 `char`를 가리키는 포인터와 C스타일 string을 가리키는 포인터를 명시적으로 구분되어지게 해주는 역할은 해 줍니다.
0으로 끝나는 것을 가정하지 않은 문자의 배열에 대해서는 `zstring`보다는 `char*`를 사용해야 합니다.
프랑스어의 악센트는 선택사항이죠. (뭔가 조크인것 같은데요....)

`not_null<zstring>`는 C스타일 string이면서 `nullptr`일수 없는 것에 사용됩니다. ??? `not_null<zstring>`에 대한 이름이 필요할까요 ? 아니면 추한 특징인가요 ?

>Logically, those last two aliases are not needed, but we are not always logical, and they make the distinction between a pointer to one `char` and a pointer to a C-style string explicit.
A sequence of characters that is not assumed to be zero-terminated sould be a `char*`, rather than a `zstring`.
French accent optional.
>
>Use `not_null<zstring>` for C-style strings that cannot be `nullptr`. ??? Do we need a name for `not_null<zstring>`? or is its ugliness a feature?


<a name="SS-ownership"></a>
## GSL.owner: Ownership pointers

* `unique_ptr<T>`	// unique ownership: `std::unique_ptr<T>`
* `shared_ptr<T>`	// shared ownership: `std::shared_ptr<T>` (a counted pointer)
* `stack_array<T>`	// A stack-allocated array. The number of elements are determined at construction and fixed thereafter. The elements are mutable unless `T` is a `const` type.
* `dyn_array<T>`	// ??? needed ??? A heap-allocated array. The number of elements are determined at construction and fixed thereafter.
The elements are mutable unless `T` is a `const` type. Basically an `array_view` that allocates and owns its elements.


<a name="SS-assertions"></a>
## GSL.assert: Assertions

* `Expects`		// precondition assertion. Currently placed in function bodies. Later, should be moved to declarations.
				// `Expects(p)` terminates the program unless `p==true`
				// ??? `Expect` in under control of some options (enforcement, error message, alternatives to terminate)
* `Ensures`		// postcondition assertion.	Currently placed in function bodies. Later, should be moved to declarations.


<a name="SS-utilities"></a>
## GSL.util: Utilities

* `finally`		// `finally(f)` makes a `Final_act{f}` with a destructor that invokes `f`
* `narrow_cast`	// `narrow_cast<T>(x)` is `static_cast<T>(x)`
* `narrow`		// `narrow<T>(x)` is `static_cast<T>(x)` if `static_cast<T>(x)==x` or it throws `narrowing_error`
* `implicit`	// "Marker" to put on single-argument constructors to explicitly make them non-explicit
(I don't know how to do that except with a macro: `#define implicit`).
* `move_owner`	// `p=move_owner(q)` means `p=q` but ???


<a name="SS-concepts"></a>
## GSL.concept: Concepts

These concepts (type predicates) are borrowed from Andrew Sutton's Origin library, the Range proposal, and the ISO WG21 Palo Alto TR.
They are likely to be very similar to what will become part of the ISO C++ standard.
The notation is that of the ISO WG21 Concepts TS (???ref???).

* `Range`
* `String`	// ???
* `Number`	// ???
* `Sortable`
* `Pointer`  // A type with `*`, `->`, `==`, and default construction (default construction is assumed to set the singular "null" value) [see smartptrconcepts](#Rr-smartptrconcepts)
* `Unique_ptr`  // A type that matches `Pointer`, has move (not copy), and matches the Lifetime profile criteria for a `unique` owner type [see smartptrconcepts](#Rr-smartptrconcepts)
* `Shared_ptr`   // A type that matches `Pointer`, has copy, and matches the Lifetime profile criteria for a `shared` owner type [see smartptrconcepts](#Rr-smartptrconcepts)
* `EqualityComparable`	// ???Must we suffer CaMelcAse???
* `Convertible`
* `Common`
* `Boolean`
* `Integral`
* `SignedIntegral`
* `SemiRegular`
* `Regular`
* `TotallyOrdered`
* `Function`
* `RegularFunction`
* `Predicate`
* `Relation`
* ...
