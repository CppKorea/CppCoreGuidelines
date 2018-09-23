
# <a name="S-stdlib"></a>SL: The Standard Library

Using only the bare language, every task is tedious (in any language).
Using a suitable library any task can be reasonably simple.

The standard library has steadily grown over the years.
Its description in the standard is now larger than that of the language features.
So, it is likely that this library section of the guidelines will eventually grow in size to equal or exceed all the rest.

<< ??? We need another level of rule numbering ??? >>

C++ Standard Library component summary:

* [SL.con: Containers](#SS-con)
* [SL.str: String](#SS-string)
* [SL.io: Iostream](#SS-io)
* [SL.regex: Regex](#SS-regex)
* [SL.chrono: Time](#SS-chrono)
* [SL.C: The C Standard Library](#SS-clib)

Standard-library rule summary:

* [SL.1: Use libraries wherever possible](#Rsl-lib)
* [SL.2: Prefer the standard library to other libraries](#Rsl-sl)
* [SL.3: Do not add non-standard entities to namespace `std`](#sl-std)
* [SL.4: Use the standard library in a type-safe manner](#sl-safe)
* ???

### <a name="Rsl-lib"></a>SL.1:  Use libraries wherever possible

##### Reason

Save time. Don't re-invent the wheel.
Don't replicate the work of others.
Benefit from other people's work when they make improvements.
Help other people when you make improvements.

### <a name="Rsl-sl"></a>SL.2: Prefer the standard library to other libraries

##### Reason

More people know the standard library.
It is more likely to be stable, well-maintained, and widely available than your own code or most other libraries.


### <a name="sl-std"></a>SL.3: Do not add non-standard entities to namespace `std`

##### Reason

Adding to `std` may change the meaning of otherwise standards conforming code.
Additions to `std` may clash with future versions of the standard.

##### Example

    ???

##### Enforcement

Possible, but messy and likely to cause problems with platforms.

### <a name="sl-safe"></a>SL.4: Use the standard library in a type-safe manner

##### Reason

Because, obviously, breaking this rule can lead to undefined behavior, memory corruption, and all kinds of other bad errors.

##### Note

This is a semi-philosophical meta-rule, which needs many supporting concrete rules.
We need it as an umbrella for the more specific rules.

Summary of more specific rules:

* [SL.4: Use the standard library in a type-safe manner](#sl-safe)


## <a name="SS-con"></a>SL.con: Containers

???

Container rule summary:

* [SL.con.1: Prefer using STL `array` or `vector` instead of a C array](#Rsl-arrays)
* [SL.con.2: Prefer using STL `vector` by default unless you have a reason to use a different container](#Rsl-vector)
* [SL.con.3: Avoid bounds errors](#Rsl-bounds)
*  ???

### <a name="Rsl-arrays"></a>SL.con.1: Prefer using STL `array` or `vector` instead of a C array

##### Reason

C arrays are less safe, and have no advantages over `array` and `vector`.
For a fixed-length array, use `std::array`, which does not degenerate to a pointer when passed to a function and does know its size.
Also, like a built-in array, a stack-allocated `std::array` keeps its elements on the stack.
For a variable-length array, use `std::vector`, which additionally can change its size and handles memory allocation.

##### Example
```c++
    int v[SIZE];                        // BAD

    std::array<int, SIZE> w;             // ok
```
##### Example
```c++
    int* v = new int[initial_size];     // BAD, owning raw pointer
    delete[] v;                         // BAD, manual delete

    std::vector<int> w(initial_size);   // ok
```
##### Note

Use `gsl::span` for non-owning references into a container.

##### Note

Comparing the performance of a fixed-sized array allocated on the stack against a `vector` with its elements on the free store is bogus.
You could just as well compare a `std::array` on the stack against the result of a `malloc()` accessed through a pointer.
For most code, even the difference between stack allocation and free-store allocation doesn't matter, but the convenience and safety of `vector` does.
People working with code for which that difference matters are quite capable of choosing between `array` and `vector`.

##### Enforcement

* Flag declaration of a C array inside a function or class that also declares an STL container (to avoid excessive noisy warnings on legacy non-STL code). To fix: At least change the C array to a `std::array`.

### <a name="Rsl-vector"></a>SL.con.2: Prefer using STL `vector` by default unless you have a reason to use a different container

##### Reason

`vector` and `array` are the only standard containers that offer the fastest general-purpose access (random access, including being vectorization-friendly), the fastest default access pattern (begin-to-end or end-to-begin is prefetcher-friendly), and the lowest space overhead (contiguous layout has zero per-element overhead, which is cache-friendly).
Usually you need to add and remove elements from the container, so use `vector` by default; if you don't need to modify the container's size, use `array`.

Even when other containers seem more suited, such a `map` for O(log N) lookup performance or a `list` for efficient insertion in the middle, a `vector` will usually still perform better for containers up to a few KB in size.

##### Note

`string` should not be used as a container of individual characters. A `string` is a textual string; if you want a container of characters, use `vector</*char_type*/>` or `array</*char_type*/>` instead.

##### Exceptions

If you have a good reason to use another container, use that instead. For example:

* If `vector` suits your needs but you don't need the container to be variable size, use `array` instead.

* If you want a dictionary-style lookup container that guarantees O(K) or O(log N) lookups, the container will be larger (more than a few KB) and you perform frequent inserts so that the overhead of maintaining a sorted `vector` is infeasible, go ahead and use an `unordered_map` or `map` instead.

##### Enforcement

* Flag a `vector` whose size never changes after construction (such as because it's `const` or because no non-`const` functions are called on it). To fix: Use an `array` instead.

### <a name="Rsl-bounds"></a>SL.con.3: Avoid bounds errors

##### Reason

Read or write beyond an allocated range of elements typically leads to bad errors, wrong results, crashes, and security violations.

##### Note

The standard-library functions that apply to ranges of elements all have (or could have) bounds-safe overloads that take `span`.
Standard types such as `vector` can be modified to perform bounds-checks under the bounds profile (in a compatible way, such as by adding contracts), or used with `at()`.

Ideally, the in-bounds guarantee should be statically enforced.
For example:

* a range-`for` cannot loop beyond the range of the container to which it is applied
* a `v.begin(),v.end()` is easily determined to be bounds safe

Such loops are as fast as any unchecked/unsafe equivalent.

Often a simple pre-check can eliminate the need for checking of individual indices.
For example

* for `v.begin(),v.begin()+i` the `i` can easily be checked against `v.size()`

Such loops can be much faster than individually checked element accesses.

##### Example, bad
```c++
    void f()
    {
        array<int, 10> a, b;
        memset(a.data(), 0, 10);         // BAD, and contains a length error (length = 10 * sizeof(int))
        memcmp(a.data(), b.data(), 10);  // BAD, and contains a length error (length = 10 * sizeof(int))
    }
```
Also, `std::array<>::fill()` or `std::fill()` or even an empty initializer are better candidate than `memset()`.

##### Example, good
```c++
    void f()
    {
        array<int, 10> a, b, c{};       // c is initialized to zero
        a.fill(0);
        fill(b.begin(), b.end(), 0);    // std::fill()
        fill(b, 0);                     // std::fill() + Ranges TS

        if ( a == b ) {
          // ...
        }
    }
```
##### Example

If code is using an unmodified standard library, then there are still workarounds that enable use of `std::array` and `std::vector` in a bounds-safe manner. Code can call the `.at()` member function on each class, which will result in an `std::out_of_range` exception being thrown. Alternatively, code can call the `at()` free function, which will result in fail-fast (or a customized action) on a bounds violation.
```c++
    void f(std::vector<int>& v, std::array<int, 12> a, int i)
    {
        v[0] = a[0];        // BAD
        v.at(0) = a[0];     // OK (alternative 1)
        at(v, 0) = a[0];    // OK (alternative 2)

        v.at(0) = a[i];     // BAD
        v.at(0) = a.at(i);  // OK (alternative 1)
        v.at(0) = at(a, i); // OK (alternative 2)
    }
```
##### Enforcement

* Issue a diagnostic for any call to a standard-library function that is not bounds-checked.
??? insert link to a list of banned functions

This rule is part of the [bounds profile](#SS-bounds).

**TODO Notes**:

* Impact on the standard library will require close coordination with WG21, if only to ensure compatibility even if never standardized.
* We are considering specifying bounds-safe overloads for stdlib (especially C stdlib) functions like `memcmp` and shipping them in the GSL.
* For existing stdlib functions and types like `vector` that are not fully bounds-checked, the goal is for these features to be bounds-checked when called from code with the bounds profile on, and unchecked when called from legacy code, possibly using contracts (concurrently being proposed by several WG21 members).



## <a name="SS-string"></a>SL.str: String

Text manipulation is a huge topic.
`std::string` doesn't cover all of it.
This section primarily tries to clarify `std::string`'s relation to `char*`, `zstring`, `string_view`, and `gsl::string_span`.
The important issue of non-ASCII character sets and encodings (e.g., `wchar_t`, Unicode, and UTF-8) will be covered elsewhere.

**See also**: [regular expressions](#SS-regex)

Here, we use "sequence of characters" or "string" to refer to a sequence of characters meant to be read as text (somehow, eventually).
We don't consider

String summary:

* [SL.str.1: Use `std::string` to own character sequences](#Rstr-string)
* [SL.str.2: Use `std::string_view` or `gsl::string_span` to refer to character sequences](#Rstr-view)
* [SL.str.3: Use `zstring` or `czstring` to refer to a C-style, zero-terminated, sequence of characters](#Rstr-zstring)
* [SL.str.4: Use `char*` to refer to a single character](#Rstr-char*)
* [SL.str.5: Use `std::byte` to refer to byte values that do not necessarily represent characters](#Rstr-byte)

* [SL.str.10: Use `std::string` when you need to perform locale-sensitive string operations](#Rstr-locale)
* [SL.str.11: Use `gsl::string_span` rather than `std::string_view` when you need to mutate a string](#Rstr-span)
* [SL.str.12: Use the `s` suffix for string literals meant to be standard-library `string`s](#Rstr-s)

**See also**:

* [F.24 span](#Rf-range)
* [F.25 zstring](#Rf-zstring)


### <a name="Rstr-string"></a>SL.str.1: Use `std::string` to own character sequences

##### Reason

`string` correctly handles allocation, ownership, copying, gradual expansion, and offers a variety of useful operations.

##### Example
```c++
    vector<string> read_until(const string& terminator)
    {
        vector<string> res;
        for (string s; cin >> s && s != terminator; ) // read a word
            res.push_back(s);
        return res;
    }
```
Note how `>>` and `!=` are provided for `string` (as examples of useful operations) and there are no explicit
allocations, deallocations, or range checks (`string` takes care of those).

In C++17, we might use `string_view` as the argument, rather than `const string*` to allow more flexibility to callers:
```c++
    vector<string> read_until(string_view terminator)   // C++17
    {
        vector<string> res;
        for (string s; cin >> s && s != terminator; ) // read a word
            res.push_back(s);
        return res;
    }
```
The `gsl::string_span` is a current alternative offering most of the benefits of `std::string_view` for simple examples:
```c++
    vector<string> read_until(string_span terminator)
    {
        vector<string> res;
        for (string s; cin >> s && s != terminator; ) // read a word
            res.push_back(s);
        return res;
    }
```
##### Example, bad

Don't use C-style strings for operations that require non-trivial memory management
```c++
    char* cat(const char* s1, const char* s2)   // beware!
        // return s1 + '.' + s2
    {
        int l1 = strlen(s1);
        int l2 = strlen(s2);
        char* p = (char*) malloc(l1 + l2 + 2);
        strcpy(p, s1, l1);
        p[l1] = '.';
        strcpy(p + l1 + 1, s2, l2);
        p[l1 + l2 + 1] = 0;
        return p;
    }
```
Did we get that right?
Will the caller remember to `free()` the returned pointer?
Will this code pass a security review?

##### Note

Do not assume that `string` is slower than lower-level techniques without measurement and remember than not all code is performance critical.
[Don't optimize prematurely](#Rper-Knuth)

##### Enforcement

???

### <a name="Rstr-view"></a>SL.str.2: Use `std::string_view` or `gsl::string_span` to refer to character sequences

##### Reason

`std::string_view` or `gsl::string_span` provides simple and (potentially) safe access to character sequences independently of how
those sequences are allocated and stored.

##### Example
```c++
    vector<string> read_until(string_span terminator);

    void user(zstring p, const string& s, string_span ss)
    {
        auto v1 = read_until(p);
        auto v2 = read_until(s);
        auto v3 = read_until(ss);
        // ...
    }
```
##### Note

`std::string_view` (C++17) is read-only.

##### Enforcement

???

### <a name="Rstr-zstring"></a>SL.str.3: Use `zstring` or `czstring` to refer to a C-style, zero-terminated, sequence of characters

##### Reason

Readability.
Statement of intent.
A plain `char*` can be a pointer to a single character, a pointer to an array of characters, a pointer to a C-style (zero-terminated) string, or even to a small integer.
Distinguishing these alternatives prevents misunderstandings and bugs.

##### Example
```c++
    void f1(const char* s); // s is probably a string
```
All we know is that it is supposed to be the nullptr or point to at least one character
```c++
    void f1(zstring s);     // s is a C-style string or the nullptr
    void f1(czstring s);    // s is a C-style string constant or the nullptr
    void f1(std::byte* s);  // s is a pointer to a byte (C++17)
```
##### Note

Don't convert a C-style string to `string` unless there is a reason to.

##### Note

Like any other "plain pointer", a `zstring` should not represent ownership.

##### Note

There are billions of lines of C++ "out there", most use `char*` and `const char*` without documenting intent.
They are used in a wide variety of ways, including to represent ownership and as generic pointers to memory (instead of `void*`).
It is hard to separate these uses, so this guideline is hard to follow.
This is one of the major sources of bugs in C and C++ programs, so it is worthwhile to follow this guideline wherever feasible..

##### Enforcement

* Flag uses of `[]` on a `char*`
* Flag uses of `delete` on a `char*`
* Flag uses of `free()` on a `char*`

### <a name="Rstr-char*"></a>SL.str.4: Use `char*` to refer to a single character

##### Reason

The variety of uses of `char*` in current code is a major source of errors.

##### Example, bad
```c++
    char arr[] = {'a', 'b', 'c'};

    void print(const char* p)
    {
        cout << p << '\n';
    }

    void use()
    {
        print(arr);   // run-time error; potentially very bad
    }
```
The array `arr` is not a C-style string because it is not zero-terminated.

##### Alternative

See [`zstring`](#Rstr-zstring), [`string`](#Rstr-string), and [`string_span`](#Rstr-view).

##### Enforcement

* Flag uses of `[]` on a `char*`

### <a name="Rstr-byte"></a>SL.str.5: Use `std::byte` to refer to byte values that do not necessarily represent characters

##### Reason

Use of `char*` to represent a pointer to something that is not necessarily a character causes confusion
and disables valuable optimizations.

##### Example

    ???

##### Note

C++17

##### Enforcement

???


### <a name="Rstr-locale"></a>SL.str.10: Use `std::string` when you need to perform locale-sensitive string operations

##### Reason

`std::string` supports standard-library [`locale` facilities](#Rstr-locale)

##### Example

    ???

##### Note

???

##### Enforcement

???

### <a name="Rstr-span"></a>SL.str.11: Use `gsl::string_span` rather than `std::string_view` when you need to mutate a string

##### Reason

`std::string_view` is read-only.

##### Example

???

##### Note

???

##### Enforcement

The compiler will flag attempts to write to a `string_view`.

### <a name="Rstr-s"></a>SL.str.12: Use the `s` suffix for string literals meant to be standard-library `string`s

##### Reason

Direct expression of an idea minimizes mistakes.

##### Example
```c++
    auto pp1 = make_pair("Tokyo", 9.00);         // {C-style string,double} intended?
    pair<string, double> pp2 = {"Tokyo", 9.00};  // a bit verbose
    auto pp3 = make_pair("Tokyo"s, 9.00);        // {std::string,double}    // C++14
    pair pp4 = {"Tokyo"s, 9.00};                 // {std::string,double}    // C++17
```

##### Enforcement

???


## <a name="SS-io"></a>SL.io: Iostream

`iostream`s is a type safe, extensible, formatted and unformatted I/O library for streaming I/O.
It supports multiple (and user extensible) buffering strategies and multiple locales.
It can be used for conventional I/O, reading and writing to memory (string streams),
and user-defines extensions, such as streaming across networks (asio: not yet standardized).

Iostream rule summary:

* [SL.io.1: Use character-level input only when you have to](#Rio-low)
* [SL.io.2: When reading, always consider ill-formed input](#Rio-validate)
* [SL.io.3: Prefer iostreams for I/O](#Rio-streams)
* [SL.io.10: Unless you use `printf`-family functions call `ios_base::sync_with_stdio(false)`](#Rio-sync)
* [SL.io.50: Avoid `endl`](#Rio-endl)
* [???](#???)

### <a name="Rio-low"></a>SL.io.1: Use character-level input only when you have to

##### Reason

Unless you genuinely just deal with individual characters, using character-level input leads to the user code performing potentially error-prone
and potentially inefficient composition of tokens out of characters.

##### Example
```c++
    char c;
    char buf[128];
    int i = 0;
    while (cin.get(c) && !isspace(c) && i < 128)
        buf[i++] = c;
    if (i == 128) {
        // ... handle too long string ....
    }
```
Better (much simpler and probably faster):
```c++
    string s;
    s.reserve(128);
    cin >> s;
```
and the `reserve(128)` is probably not worthwhile.

##### Enforcement

???


### <a name="Rio-validate"></a>SL.io.2: When reading, always consider ill-formed input

##### Reason

Errors are typically best handled as soon as possible.
If input isn't validated, every function must be written to cope with bad data (and that is not practical).

##### Example

    ???

##### Enforcement

???

### <a name="Rio-streams"></a>SL.io.3: Prefer `iostream`s for I/O

##### Reason

`iostream`s are safe, flexible, and extensible.

##### Example
```c++
    // write a complex number:
    complex<double> z{ 3, 4 };
    cout << z << '\n';
```
`complex` is a user-defined type and its I/O is defined without modifying the `iostream` library.

##### Example
```c++
    // read a file of complex numbers:
    for (complex<double> z; cin >> z; )
        v.push_back(z);
```
##### Exception

??? performance ???

##### Discussion: `iostream`s vs. the `printf()` family

It is often (and often correctly) pointed out that the `printf()` family has two advantages compared to `iostream`s:
flexibility of formatting and performance.
This has to be weighed against `iostream`s advantages of extensibility to handle user-defined types, resilient against security violations,
implicit memory management, and `locale` handling.

If you need I/O performance, you can almost always do better than `printf()`.

`gets()` `scanf()` using `s`, and `printf()` using `%s` are security hazards (vulnerable to buffer overflow and generally error-prone).
In C11, they are replaced by `gets_s()`, `scanf_s()`, and `printf_s()` as safer alternatives, but they are still not type safe.

##### Enforcement

Optionally flag `<cstdio>` and `<stdio.h>`.

### <a name="Rio-sync"></a>SL.io.10: Unless you use `printf`-family functions call `ios_base::sync_with_stdio(false)`

##### Reason

Synchronizing `iostreams` with `printf-style` I/O can be costly.
`cin` and `cout` are by default synchronized with `printf`.

##### Example
```c++
    int main()
    {
        ios_base::sync_with_stdio(false);
        // ... use iostreams ...
    }
```
##### Enforcement

???

### <a name="Rio-endl"></a>SL.io.50: Avoid `endl`

##### Reason

The `endl` manipulator is mostly equivalent to `'\n'` and `"\n"`;
as most commonly used it simply slows down output by doing redundant `flush()`s.
This slowdown can be significant compared to `printf`-style output.

##### Example
```c++
    cout << "Hello, World!" << endl;    // two output operations and a flush
    cout << "Hello, World!\n";          // one output operation and no flush
```
##### Note

For `cin`/`cout` (and equivalent) interaction, there is no reason to flush; that's done automatically.
For writing to a file, there is rarely a need to `flush`.

##### Note

Apart from the (occasionally important) issue of performance,
the choice between `'\n'` and `endl` is almost completely aesthetic.

## <a name="SS-regex"></a>SL.regex: Regex

`<regex>` is the standard C++ regular expression library.
It supports a variety of regular expression pattern conventions.

## <a name="SS-chrono"></a>SL.chrono: Time

`<chrono>` (defined in namespace `std::chrono`) provides the notions of `time_point` and `duration` together with functions for
outputting time in various units.
It provides clocks for registering `time_points`.

## <a name="SS-clib"></a>SL.C: The C Standard Library

???

C Standard Library rule summary:

* [S.C.1: Don't use setjmp/longjmp](#Rclib-jmp)
* [???](#???)
* [???](#???)

### <a name="Rclib-jmp"></a>SL.C.1: Don't use setjmp/longjmp

##### Reason

a `longjmp` ignores destructors, thus invalidating all resource-management strategies relying on RAII

##### Enforcement

Flag all occurrences of `longjmp`and `setjmp`

