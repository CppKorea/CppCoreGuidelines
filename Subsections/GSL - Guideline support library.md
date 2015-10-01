# GSL: Guideline support library

The GSL is a small library of facilities designed to support this set of guidelines.
Without these facilities, the guidelines would have to be far more restrictive on language details.

The Core Guidelines support library is define in namespace `Guide` and the names may be aliases for standard library or other well-known library names.Using the (compile-time) indirection through the `Guide` namespace allows for experimentation and for local variants of the support facilities.

The support library facilities are designed to be extremely lightweight (zero-overhead) so that they impose no overhead compared to using conventional alternatives.
Where desirable, they can be "instrumented" with additional functionality (e.g., checks) for tasks such as debugging.

These Guidelines assume a `variant` type, but this is not currently in GSL because the design is being actively refined in the standards committee.


<a name="SS-views"></a>
## GSL.view: Views

These types allow the user to distinguish between owning and non-owning pointers and between pointers to a single object and pointers to the first element of a sequence.

These "views" are never owners.

References are never owners.

The names are mostly ISO standard-library style (lower case and underscore):

* `T*`			// The `T*` is not an owner, may be `nullptr` (Assumed to be pointing to a single element)
* `char*`		// A C-style string (a zero-terminated array of characters); can be `nullptr`
* `const char*`	// A C-style string; can be `nullptr`
* `T&`			// The `T&` is not an owner, may not be `&(T&)*nullptr` (language rule)

The "raw-pointer" notation (e.g. `int*`) is assumed to have its most common meaning; that is, a pointer points to an object, but does not own it.
Owners should be converted to resource handles (e.g., `unique_ptr` or `vector<T>`) or marked `owner<T*>`

* `owner<T*>`		// a `T*`that owns the object pointed/referred to; can be `nullptr`
* `owner<T&>`		// a `T&` that owns the object pointed/referred to

`owner` is used to mark owning pointers in code that cannot be upgraded to use proper resource handles.
Reasons for that include

* cost of conversion
* the pointer is used with an ABI
* the pointer is part of the implementation of a resource handle.

An `owner<T>` differs from a resource handle for a `T` by still requiring and explicit `delete`.

An `owner<T>` is assumed to refer to an object on the free store (heap).

If something is not supposed to be `nullptr`, say so:

* `not_null<T>`		// `T` is usually a pointer type (e.g., `not_null<int*>` and `not_null<owner<Foo*>>`) that may not be `nullptr`.
`T` can be any type for which `==nullptr` is meaningful.

* `array_view<T>`	// [`p`:`p+n`), constructor from `{p,q}` and `{p,n}`; `T` is the pointer type
* `array_view_p<T>`	// `{p,predicate}` [`p`:`q`) where `q` is the first element for which `predicate(*p)` is true
* `string_view`		// `array_view<char>`
* `cstring_view`	// `array_view<const char>`

A `*_view<T>` refer to zero or more mutable `T`s unless `T` is a `const` type.

"Pointer arithmetic" is best done within `array_view`s.
A `char*` that points to something that is not a C-style string (e.g., a pointer into an input buffer) should be represented by an `array_view`.
There is no really good way to say "pointer to a single `char` (`string_view{p,1}` can do that, and `T*` where `T` is a `char` in a template that has not been specialized for C-style strings).

* `zstring`		// a `char*` supposed to be a C-style string; that is, a zero-terminated sequence of `char` or `null_ptr`
* `czstring`	// a `const char*` supposed to be a C-style string; that is, a zero-terminated sequence of `const` `char` ort `null_ptr`

Logically, those last two aliases are not needed, but we are not always logical,
and they make the distinction between a pointer to one `char` and a pointer to a C-style string explicit.
A sequence of characters that is not assumed to be zero-terminated sould be a `char*`, rather than a `zstring`.
French accent optional.

Use `not_null<zstring>` for C-style strings that cannot be `nullptr`. ??? Do we need a name for `not_null<zstring>`? or is its ugliness a feature?


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