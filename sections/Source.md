
# <a name="S-source"></a>SF: 소스 파일

선언(인터페이스)과 정의(구현)를 구분하라. 
헤더 파일은 인터페이스를 표현하고 논리적 구조를 강조하기 위해서 사용해야 한다.

소스 파일 규칙 요약:

* [SF.1: Use a `.cpp` suffix for code files and `.h` for interface files if your project doesn't already follow another convention](#Rs-file-suffix)
* [SF.2: A `.h` file may not contain object definitions or non-inline function definitions](#Rs-inline)
* [SF.3: Use `.h` files for all declarations used in multiple source files](#Rs-declaration-header)
* [SF.4: Include `.h` files before other declarations in a file](#Rs-include-order)
* [SF.5: A `.cpp` file must include the `.h` file(s) that defines its interface](#Rs-consistency)
* [SF.6: Use `using namespace` directives for transition, for foundation libraries (such as `std`), or within a local scope (only)](#Rs-using)
* [SF.7: Don't write `using namespace` at global scope in a header file](#Rs-using-directive)
* [SF.8: Use `#include` guards for all `.h` files](#Rs-guards)
* [SF.9: Avoid cyclic dependencies among source files](#Rs-cycles)
* [SF.10: Avoid dependencies on implicitly `#include`d names](#Rs-implicit)
* [SF.11: Header files should be self-contained](#Rs-contained)

* [SF.20: Use `namespace`s to express logical structure](#Rs-namespace)
* [SF.21: Don't use an unnamed (anonymous) namespace in a header](#Rs-unnamed)
* [SF.22: Use an unnamed (anonymous) namespace for all internal/nonexported entities](#Rs-unnamed2)

### <a name="Rs-file-suffix"></a>SF.1: Use a `.cpp` suffix for code files and `.h` for interface files if your project doesn't already follow another convention

##### Reason
오래된 관례이다. 하지만 일관성이 더 중요하다. 프로젝트가 무엇인가 사용하고 있다면, 그대로 따라가라

##### Note

This convention reflects a common use pattern:
Headers are more often shared with C to compile as both C++ and C, which typically uses `.h`,
and it's easier to name all headers `.h` instead of having different extensions for just those headers that are intended to be shared with C.
On the other hand, implementation files are rarely shared with C and so should typically be distinguished from `.c` files,
so it's normally best to name all C++ implementation files something else (such as `.cpp`).

`.h` 와 `.cpp`가 기본적으로 권장되기는 하지만 필수는 아니다. 다른 이름들도 광범위하게 사용된다.
예를 들자면 `.hh`, `.C`, `.cxx` 같은 것이 있다. 이런 이름을 같이 써도 좋다.

In this document, we refer to `.h` and `.cpp` as a shorthand for header and implementation files,
even though the actual extension may be different.

Your IDE (if you use one) may have strong opinions about suffices.

##### Example

```c++
    // foo.h:
    extern int a;   // a declaration
    extern void foo();

    // foo.cpp:
    int a;   // a definition
    void foo() { ++a; }
```

`foo.h` 는 `foo.cpp`에 대한 인터페이스를 제공한다. 전역 변수는 피해야 한다.

##### Example, bad

```c++
    // foo.h:
    int a;   // a definition
    void foo() { ++a; }
```

`#include<foo.h>` 문구가 한 프로그램 내에 두 번 이상 포함된다면 단일 정의 규칙(one-definition-rule)에 위배된다고 링커가 오류를 낼 것이다.

##### Enforcement

* Flag non-conventional file names.
* Check that `.h` and `.cpp` (and equivalents) follow the rules below.

### <a name="Rs-inline"></a>SF.2: A `.h` file may not contain object definitions or non-inline function definitions

##### Reason

하나의 정의만 가져야하는 대상을 `포함`하게 되면 링킹 에러로 이어진다.

##### Example

```c++
    // file.h:
    namespace Foo {
        int x = 7;
        int xx() { return x+x; }
    }

    // file1.cpp:
    #include <file.h>
    // ... more ...

     // file2.cpp:
    #include <file.h>
    // ... more ...
```

Linking `file1.cpp` and `file2.cpp` will give two linker errors.

**Alternative formulation**: `.h` 파일은 다음의 항목만을 가진다:

* `#include`s of other `.h` files (possibly with include guards)
* templates
* class definitions
* function declarations
* `extern` declarations
* `inline` function definitions
* `constexpr` definitions
* `const` definitions
* `using` alias definitions
* ???

##### Enforcement

위의 목록에서 허용되는 것들을 검토한다

### <a name="Rs-declaration-header"></a>SF.3: Use `.h` files for all declarations used in multiple source files

##### Reason

관리가 편해지고 가독성이 향상된다.

##### Example, bad

```c++
    // bar.cpp:
    void bar() { cout << "bar\n"; }

    // foo.cpp:
    extern void bar();
    void foo() { bar(); }
```

`bar`를 관리하는 사람이 그 타입을 바꾸고자 하더라도 `bar`의 모든 선언을 찾을 수가 없다.
`bar`를 사용하는 입장에서는 이 인터페이스가 완벽한지 알 수가 없다. 기껏해야 (나중에) 링커로부터 오류메시지를 받는 것이 고작이다.

##### Enforcement

* Flag declarations of entities in other source files not placed in a `.h`.

### <a name="Rs-include-order"></a>SF.4: Include `.h` files before other declarations in a file

##### Reason

문맥에 대한 종속성을 최소화하고 가독성을 높인다.

##### Example

```c++
    #include <vector>
    #include <algorithm>
    #include <string>

    // ... my code here ...
```

##### Example, bad

```c++
    #include <vector>

    // ... my code here ...

    #include <algorithm>
    #include <string>
```

##### Note

이 내용은 `.h` 와 `.cpp` 파일 모두에 해당한다.

##### Note

There is an argument for insulating code from declarations and macros in header files by `#including` headers *after* the code we want to protect
(as in the example labeled "bad").
However

* that only works for one file (at one level): Use that technique in a header included with other headers and the vulnerability reappears.
* a namespace (an "implementation namespace") can protect against many context dependencies.
* full protection and flexibility require modules.

**See also**:

* [Working Draft, Extensions to C++ for Modules](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4592.pdf)
* [Modules, Componentization, and Transition](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0141r0.pdf)

##### Enforcement

Easy.

### <a name="Rs-consistency"></a>SF.5: A `.cpp` file must include the `.h` file(s) that defines its interface

##### Reason

컴파일러가 좀더 일찍 일관성을 검사할 수 있도록 한다.

##### Example, bad

```c++
    // foo.h:
    void foo(int);
    int bar(long);
    int foobar(int);

    // foo.cpp:
    void foo(int) { /* ... */ }
    int bar(double) { /* ... */ }
    double foobar(int);
```

`bar` 나 `foobar` 를 호출하는 프로그램을 링크하는 시점에서야 오류를 확인할 수 있다.

##### Example

```c++
    // foo.h:
    void foo(int);
    int bar(long);
    int foobar(int);

    // foo.cpp:
    #include <foo.h>

    void foo(int) { /* ... */ }
    int bar(double) { /* ... */ }
    double foobar(int);   // error: wrong return type
```

The return-type error for `foobar` is now caught immediately when `foo.cpp` is compiled.
The argument-type error for `bar` cannot be caught until link time because of the possibility of overloading, but systematic use of `.h` files increases the likelihood that it is caught earlier by the programmer.

##### Enforcement

???

### <a name="Rs-using"></a>SF.6: Use `using namespace` directives for transition, for foundation libraries (such as `std`), or within a local scope (only)

##### Reason

 `using namespace` can lead to name clashes, so it should be used sparingly.
 However, it is not always possible to qualify every name from a namespace in user code (e.g., during transition)
 and sometimes a namespace is so fundamental and prevalent in a code base, that consistent qualification would be verbose and distracting.

##### Example

```c++
    #include <string>
    #include <vector>
    #include <iostream>
    #include <memory>
    #include <algorithm>

    using namespace std;

    // ...
```

Here (obviously), the standard library is used pervasively and apparently no other library is used, so requiring `std::` everywhere
could be distracting.

##### Example

The use of `using namespace std;` leaves the programmer open to a name clash with a name from the standard library

```c++
    #include <cmath>
    using namespace std;

    int g(int x)
    {
        int sqrt = 7;
        // ...
        return sqrt(x); // error
    }
```

However, this is not particularly likely to lead to a resolution that is not an error and
people who use `using namespace std` are supposed to know about `std` and about this risk.

##### Note

A `.cpp` file is a form of local scope.
There is little difference in the opportunities for name clashes in an N-line `.cpp` containing a `using namespace X`,
an N-line function containing a `using namespace X`,
and M functions each containing a `using namespace X`with N lines of code in total.

##### Note

[Don't write `using namespace` in a header file](#Rs-using-directive).

##### Enforcement

Flag multiple `using namespace` directives for different namespaces in a single source file.

### <a name="Rs-using-directive"></a>SF.7: Don't write `using namespace` at global scope in a header file

##### Reason

헤더 파일에 `using` 지시자를 사용하는 경우 `#include`를 사용하는 쪽에서 대체 구현을 효과적으로 구분할 수 있는 방안을 없애버린다.
It also makes `#include`d headers order-dependent as they may have different meaning when included in different orders.

##### Example

```c++
    // bad.h
    #include <iostream>
    using namespace std; // bad

    // user.cpp
    #include "bad.h"

    bool copy(/*... some parameters ...*/);    // some function that happens to be named copy

    int main() {
        copy(/*...*/);    // now overloads local ::copy and std::copy, could be ambiguous
    }
```

##### Enforcement

Flag `using namespace` at global scope in a header file.

### <a name="Rs-guards"></a>SF.8: Use `#include` guards for all `.h` files

##### Reason

파일이 여러 번 `#include`되는 것을 방지한다.

In order to avoid include guard collisions, do not just name the guard after the filename.
Be sure to also include a key and good differentiator, such as the name of library or component
the header file is part of.

##### Example

```c++
    // file foobar.h:
    #ifndef LIBRARY_FOOBAR_H
    #define LIBRARY_FOOBAR_H
    // ... declarations ...
    #endif // LIBRARY_FOOBAR_H
```

##### Enforcement

`#include`보호 문구가 없는 `.h` 파일이 있다면 표시한다

##### Note

Some implementations offer vendor extensions like `#pragma once` as alternative to include guards.
It is not standard and it is not portable.  It injects the hosting machine's filesystem semantics
into your program, in addition to locking you down to a vendor.
Our recommendation is to write in ISO C++: See [rule P.2](#Rp-Cplusplus).

### <a name="Rs-cycles"></a>SF.9: Avoid cyclic dependencies among source files

##### Reason
순환은 이해하기 어렵고, 컴파일 속도도 느려지게 한다.
향후 언어에서 모듈 기능을 지원할 때 이를 사용하도록 변경하기 어렵게 된다.

##### Note

단순히 `#include` 보호 장치로 처리하지 말고 실제 순환 구조를 없애야 한다.

##### Example, bad

```c++
    // file1.h:
    #include "file2.h"

    // file2.h:
    #include "file3.h"

    // file3.h:
    #include "file1.h"
```

##### Enforcement

Flag all cycles.

### <a name="Rs-implicit"></a>SF.10: Avoid dependencies on implicitly `#include`d names

##### Reason

Avoid surprises.
Avoid having to change `#include`s if an `#include`d header changes.
Avoid accidentally becoming dependent on implementation details and logically separate entities included in a header.

##### Example

```c++
    #include <iostream>
    using namespace std;

    void use()                  // bad
    {
        string s;
        cin >> s;               // fine
        getline(cin, s);        // error: getline() not defined
        if (s == "surprise") {  // error == not defined
            // ...
        }
    }
```

`<iostream>` exposes the definition of `std::string` ("why?" makes for a fun trivia question),
but it is not required to do so by transitively including the entire `<string>` header,
resulting in the popular beginner question "why doesn't `getline(cin,s);` work?"
or even an occasional "`string`s cannot be compared with `==`).

The solution is to explicitly `#include <string>`:

```c++
    #include <iostream>
    #include <string>
    using namespace std;

    void use()
    {
        string s;
        cin >> s;               // fine
        getline(cin, s);        // fine
        if (s == "surprise") {  // fine
            // ...
        }
    }
```

##### Note

Some headers exist exactly to collect a set of consistent declarations from a variety of headers.
For example:

```c++
    // basic_std_lib.h:

    #include <vector>
    #include <string>
    #include <map>
    #include <iostream>
    #include <random>
    #include <vector>
```

a user can now get that set of declarations with a single `#include`

```c++
    #include "basic_std_lib.h"
```

This rule against implicit inclusion is not meant to prevent such deliberate aggregation.

##### Enforcement

Enforcement would require some knowledge about what in a header is meant to be "exported" to users and what is there to enable implementation.
No really good solution is possible until we have modules.

### <a name="Rs-contained"></a>SF.11: Header files should be self-contained

##### Reason

Usability, headers should be simple to use and work when included on their own.
Headers should encapsulate the functionality they provide.
Avoid clients of a header having to manage that header's dependencies.

##### Example

```c++
    #include "helpers.h"
    // helpers.h depends on std::string and includes <string>
```

##### Note

Failing to follow this results in difficult to diagnose errors for clients of a header.

##### Enforcement

A test should verify that the header file itself compiles or that a cpp file which only includes the header file compiles.

### <a name="Rs-namespace"></a>SF.20: Use `namespace`s to express logical structure

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Rs-unnamed"></a>SF.21: Don't use an unnamed (anonymous) namespace in a header

##### Reason

헤더 파일에 있는 익명 이름공간 거의 대부분이 버그이다.

##### Example

    ???

##### Enforcement

* 헤더 파일에서 사용되는 익명 이름공간을 찾아내 표시한다

### <a name="Rs-unnamed2"></a>SF.22: Use an unnamed (anonymous) namespace for all internal/nonexported entities

##### Reason

어떤 외부에서도 내부의 익명 이름공간에 있는 항목들에 참조할 수 없다.
소스 파일에 정의되어 있는 모든 구현들 중 "외부에 노출되는" 항목의 정의를 뺀 나머지 모두는 익명 이름공간에 넣는다 생각하라.

##### Example

API 클래스와 그 멤버들은 익명 이름공간에 있을 수 없지만, 구현 소스 파일에 정의된 "도우미" 클래스나 함수들의 경우 익명 이름공간 영역에 정의되어야 한다.

    ???

##### Enforcement

* ???
