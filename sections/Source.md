
# <a name="S-source"></a>SF: 소스 파일

선언(인터페이스)과 정의(구현)를 구분하라. 
헤더 파일은 인터페이스를 표현하고 논리적 구조를 강조하기 위해서 사용해야 한다.

소스 파일 규칙 요약:

* [SF.1: 다른 관례를 따르는 중이 아니라면 `.cpp`는 코드 파일에, `.h`는 인터페이스 파일에 사용하라](#Rs-file-suffix)
* [SF.2: `.h` 파일에는 개체 변수(object definition) 혹은 inline이 아닌 함수의 정의가 있어서는 안된다](#Rs-inline)
* [SF.3: `.h` 파일은 여러 소스 파일에서 사용되는 선언을 담아라](#Rs-declaration-header)
* [SF.4: SF.4: 파일에서 무언가 선언하기 전에 `.h`를 include하라](#Rs-include-order)
* [SF.5: `.cpp`파일은 반드시 해당 인터페이스를 정의하는 `.h`를 include해야 한다](#Rs-consistency)
* [SF.6: `using namespace`는 네임스페이스의 이름 바꾸기, `std`처럼 기본적인 라이브러리, 혹은 지역 유효범위 안에서(만) 사용하라](#Rs-using)
* [SF.7: 헤더파일에서는 전체 유효범위(global scope)에 주는 `using namespace`를 작성하지 마라](#Rs-using-directive)
* [SF.8: 모든 `.h`파일에서 `#include` 가드(guard)를 사용하라](#Rs-guards)
* [SF.9: 소스 파일들이 순환 의존(cyclic dependencies)하게 하지마라](#Rs-cycles)
* [SF.10: 묵시적으로 `#include`된 이름이 필요하지 않도록 하라](#Rs-implicit)
* [SF.11: 헤더 파일은 독립적으로 사용할 수 있게(self-contained) 만들어라](#Rs-contained)
* [SF.20: `namespace`는 논리적 구조를 표현할 때 사용하라](#Rs-namespace)
* [SF.21: 헤더에서 이름없는(anonymous) 네임스페이스를 사용하지 마라](#Rs-unnamed)
* [SF.22: 이름없는(anonymous) 네임스페이스는 내부(internal)/노출시키지 않는(non-exported) 개체에 사용하라](#Rs-unnamed2)

### <a name="Rs-file-suffix"></a>SF.1: 다른 관례를 따르는 중이 아니라면 `.cpp`는 코드 파일에, `.h`는 인터페이스 파일에 사용하라

##### Reason

오래된 관례다. 다만 일관성이 더 중요하다. 
프로젝트에서 이미 어떤 파일 확장자 규칙을 사용하고 있다면, 그대로 따라가라

##### Note

이 관례는 코드 사용패턴에 영향을 준다:
헤더는 C 언어와 함께 사용되는 경우가 자주 있기 때문에 일반적으로 `.h`를 사용한다.
그리고 그렇게 사용되는것을 의도했다면 다른 파일 확장자를 쓰는것보다 모두가 `.h`를 사용하는것이 쉽다.
반면에, 구현 파일이 C 언어와 함께 사용되는 경우는 드물기 때문에 `.c` 파일들과는 구분될 필요가 있다. 모든 C++ 파일들이 `.cpp`처럼 다른 확장자를 사용하는게 최선의 방법이다.

`.h` 와 `.cpp`가 기본적으로 권장되기는 하지만 필수는 아니다. 다른 이름들도 광범위하게 사용된다.
예를 들자면 `.hh`, `.C`, `.cxx` 같은 것이 있다. 이런 이름을 같이 써도 좋다.

이 문서에서는, 실제로는 다른 확장자를 사용할수도 있겠지만, `.h`를 헤더파일에 대한 약칭(shorthand)으로, `.cpp`를 구현파일에 대한 약칭으로 사용한다.

당신이 사용하고있는 IDE에서는 특정한 확장자만 지원할수도 있다.

##### Example

```c++
    // foo.h:
    extern int a;   // 선언
    extern void foo();

    // foo.cpp:
    int a;   // 정의
    void foo() { ++a; }
```

`foo.h` 는 `foo.cpp`에 대한 인터페이스를 제공한다. 전역 변수는 피해야 한다.

##### Example, bad

```c++
    // foo.h:
    int a;   // 헤더 파일에 정의가 있다
    void foo() { ++a; }
```

`#include <foo.h>` 문구가 한 프로그램 내에 2회 이상 포함된다면 
단일 정의 규칙(one-definition-rule)에 위배된다고 링커가 오류를 낼 것이다.

##### Enforcement

* 관례에 맞지 않는 파일 이름들을 지적한다
* `.h`와 `.cpp`가 (그리고 비슷한 파일들이) 아래 규칙을 따르는지 확인한다

### <a name="Rs-inline"></a>SF.2: `.h` 파일에는 개체 변수(object definition) 혹은 inline이 아닌 함수의 정의가 있어서는 안된다

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

`file1.cpp`와 `file2.cpp`가 링킹될 때 링커 오류가 발생할 것이다.

**Alternative Formulation**:

`.h` 파일은 다음의 항목만을 가진다

* (include guard와 함께) 다른 `.h`의 `#include`
* 템플릿
* 클래스 정의(definition)
* 함수 선언(declaration)
* `extern` 선언
* `inline` 함수 정의
* `constexpr` 정의
* `const` 정의
* `using` 별칭
* ???

##### Enforcement

위의 목록에서 허용되는 것들을 검토한다

### <a name="Rs-declaration-header"></a>SF.3: `.h` 파일은 여러 소스 파일에서 사용되는 선언을 담아라

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
`bar`를 사용하는 입장에서는 이 인터페이스가 완벽한지 알 수가 없다.
기껏해야 (나중에) 링커로부터 오류메시지를 받는 것이 고작이다.

##### Enforcement

* 개체의 선언이 `.h`가 아니라 다른 소스파일에 있으면 지적하라

### <a name="Rs-include-order"></a>SF.4: 파일에서 무언가 선언하기 전에 `.h`를 include하라

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

하지만

* that only works for one file (at one level): Use that technique in a header included with other headers and the vulnerability reappears.
* a namespace (an "implementation namespace") can protect against many context dependencies.
* full protection and flexibility require modules.

##### See also

* [Working Draft, Extensions to C++ for Modules](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4592.pdf)
* [Modules, Componentization, and Transition](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0141r0.pdf)

##### Enforcement

쉽다

### <a name="Rs-consistency"></a>SF.5: `.cpp`파일은 반드시 해당 인터페이스를 정의하는 `.h`를 include해야 한다

##### Reason

컴파일러가 좀더 일찍 일관성을 검사할 수 있도록 한다.

##### Example, bad

```c++
    // foo.h:
    void foo(int);
    int bar(long);
    int foobar(int);

    // foo.cpp:
    void foo(int) {
        /* ... */
    }

    int bar(double) {
        /* ... */
    }

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

    void foo(int) {
        /* ... */
    }
    int bar(double) {
        /* ... */
    }
    double foobar(int);   // error: 반환 타입이 다르다
```

이제 `foobar`의 반환 타입 오류는 `foo.cpp`를 컴파일 할때 알 수 있다.
`bar`의 인자타입이 다른 것은 중복정의일 수 있으므로 오류는 링크 시간에 확인할 수 있다.
하지만 `.h`를 사용하는 것으로 프로그래머가 더 일찍 오류를 잡아낼 수 있게 한다.

##### Enforcement

???

### <a name="Rs-using"></a>SF.6: `using namespace`는 네임스페이스의 이름 바꾸기, `std`처럼 기본적인 라이브러리, 혹은 지역 유효범위 안에서(만) 사용하라

##### Reason

`using namespace`를 쓰면 이름 충돌이 일어날 수 있다. 가능한 필요한 경우에만(sparingly) 사용되어야 한다.
하지만, 사용자 코드에서 항상 모든 이름을 네임스페이스까지 분명히 하는 것(to qualify every name)이 가능한 것은 아니다.
그리고 때로는 어느 네임스페이스가 너무 기본적이고 많은 곳에서 사용되기도 한다. 그런 경우 매번 네임스페이스를 명시(qualification)하는 것은 코드를 장황하게 만들고 집중하기 어렵게 만든다.

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

이 코드는 (명백하게) 표준 라이브러리를 여럿 사용하고 있으며 다른 라이브러리는 사용하지 않는다는 것이 드러난다. 
따라서 모든 곳에서 `std::`를 작성하도록 하는 것은 코드에 집중할 수 없게 만들 것이다.

##### Example

`using namespace std;`를 사용한다는 것은 표준 라이브러리에서 사용중인 이름과 충돌이 발생할 수 있도록 허용하는 것이다.

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

다만 이 예시는 오류가 아니도록 처리될 가능성이 특히 낮은 경우다.
그리고 `using namespace std`를 사용하는 사람은 `std`에 무엇이 있고 어떤 위험이 있는지 이해하고 있을 것이다.

##### Note

하나의 `.cpp`파일은 지역 범위로 생각할 수 있다.
N-줄짜리 `.cpp`가 `using namespace X`를 사용했을 때 충돌 가능성과 
N-줄짜리 함수가 `using namespace X`를 사용했을 때,
N-줄짜리 함수 M개가 각각 `using namespace X`를 사용했을 때는 차이가 있다.

##### Note

[`using namespace`는 헤더파일에 작성하지 마라](#Rs-using-directive).

##### Enforcement

소스 파일에서 다른 네임스페이스에 대해 `using namespace`가 여러차례 나타나면 지적하라

### <a name="Rs-using-directive"></a>SF.7: 헤더파일에서는 전체 유효범위(global scope)에 주는 `using namespace`를 작성하지 마라

##### Reason

헤더 파일에 `using` 지시자를 사용하는 경우 `#include`를 사용하는 쪽에서 다른 구현을 효과적으로 구분할 수 있는 방안을 없애버린다.
동시에 그 헤더가 `#include`되는 순서를 신경쓰도록 만든다(order-dependent).
이는 헤더의 순서가 바뀌면 의미가 달라지는것과 같다.

##### Example

```c++
    // bad.h
    #include <iostream>
    using namespace std; // bad

    // user.cpp
    #include "bad.h"

    bool copy(/*... some parameters ...*/);    // some function that happens to be named copy

    int main() {
        copy(/*...*/);  // now overloads local
                        //  ::copy and std::copy, 
                        // could be ambiguous
    }
```

##### Enforcement

Flag `using namespace` at global scope in a header file.

### <a name="Rs-guards"></a>SF.8: 모든 `.h`파일에서 `#include` 가드(guard)를 사용하라

> 역주:  
> `#include` guard(보호 문구)는 어떤 헤더 파일이 여러차례 include 되었을 때 
> redefinition이 발생하지 않도록 
> Macro를 사용해 오직 처음 include할때만 그 내용이 활성화 되도록하는 트릭(Trick)을 말합니다

##### Reason

파일이 여러 번 `#include`되는 것을 방지한다.

가드의 이름이 충돌하는 것을 막기 위해, 단순히 파일의 이름을 따라서 가드들의 이름을 지어서는 안된다.
가드의 이름에 헤더파일이 담당하는 라이브러리 혹은 컴포넌트의 이름과 같은 핵심과 차별성(a key and good differentiator)이 담기게 하라.

##### Example

```c++
    // file foobar.h:
    #ifndef LIBRARY_FOOBAR_H
    #define LIBRARY_FOOBAR_H

    // ... declarations ...

    #endif // LIBRARY_FOOBAR_H
```

##### Enforcement

`#include`가드가 없는 `.h` 파일이 있다면 표시한다

##### Note

어떤 경우는 컴파일러에서 제공하는 확장(vendor extnsion)인 `#pragma once`를 대신 사용하기도 한다.
이는 표준이 아니며 모든 컴파일러가 제공하는 것은 아니다(not portable).
이 방법은 당신의 프로그램을 생성할 때 빌드를 수행하는 기계의 파일시스템 문맥을 사용하도록 만든다. 그 결과 해당 컴파일러/기계의 제공자(vendor)에 의존하게 된다.

ISO C++ 를 따라서 작성할 것을 권한다: [P.2](./Philosophy.md#Rp-Cplusplus)를 읽어보라

### <a name="Rs-cycles"></a>SF.9: 소스 파일들이 순환 의존(cyclic dependencies)하게 하지마라

##### Reason

순환은 이해하기 어렵고, 컴파일 속도도 느려지게 한다.
향후 언어에서 모듈 기능을 지원할 때 이 기능을 사용하도록 변경하기 어렵게 된다.

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

순환이 있으면 지적한다.

### <a name="Rs-implicit"></a>SF.10: 묵시적으로 `#include`된 이름이 필요하지 않도록 하라

##### Reason

이상 행동(surprise)을 막는다.
`#include`되는 파일이 바뀌었을 때 `#include`하는 코드가 바뀔 필요가 없어야 한다.
구현 세부사항이나 해더파일에 있는 논리적으로 분리된 개체에 의존하게 되지 않도록 한다.

##### Example

```c++
    #include <iostream>
    using namespace std;

    void use()                  // bad
    {
        string s;
        cin >> s;               // fine
        getline(cin, s);        // error: getline()이 정의되지 않았다
        if (s == "surprise") {  // 컴파일 오류. == 연산자가 정의되지 않았다
            // ...
        }
    }
```

`<iostream>`은 `std::string`의 정의를 사용할 수 있게 노출시킨다 ("어째서?"는 꽤 재미있는 질문이 될 것이다).
하지만 `<string>`헤더를 사용해서 그 내용을 전파시키는(by transitively) 방식을 사용해야 한다고 어떤 요구사항이 존재하는 것은 아니다. 
그 결과 많은 초심자들이 "왜 `getline(cin,s);`가 동작하지 않는거죠?"라거나, 때로는 "문자열을 `==` 연산자로 비교할수 없어요"라고 질문한다.

해결방법은 명시적으로 `#include <string>`를 추가하는 것이다:

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

어떤 헤더파일들은 그저 여러 헤더들을 똑같은 형태로(일관적으로) 가져오기 위해서만 존재하기도 한다.
예를 들어:

```c++
    // basic_std_lib.h:

    #include <vector>
    #include <string>
    #include <map>
    #include <iostream>
    #include <random>
    #include <vector>
```

이렇게 하면 사용자는 한번의 `#include`로 일련의 선언들을 가져올 수 있다.

```c++
    #include "basic_std_lib.h"
```

이 규칙은 "묵시적 include가 편의를 위해 사용되기 위한 기능이 아니다"라는 규칙에 반대된다.

> implicit inclusion is not meant to prevent such deliberate aggregation

##### Enforcement

이 규칙을 적용하려면 어떤 헤더파일이 사용자에게 "노출"되는지 알아야 하고 어떤 파일이 구현에서만 사용되는지 알아야 한다.
Module 기능을 사용할 수 있을때 까지는 좋은 해결방법이 마땅히 없다.

### <a name="Rs-contained"></a>SF.11: 헤더 파일은 독립적으로 사용할 수 있게(self-contained) 만들어라

##### Reason

사용성, 헤더는 단순하게 사용할 수 있어야 하며 그 자신만 있어도 동작해야 한다.
헤더는 제공하는 기능을 캡슐화해야 한다.
헤더를 사용하는 쪽에서 헤더의 의존성을 관리하게 하지마라.

##### Example

```c++
    #include "helpers.h"
    // helpers.h depends on std::string and includes <string>
```

##### Note

Failing to follow this results in difficult to diagnose errors for clients of a header.

##### Enforcement

A test should verify that the header file itself compiles or that a cpp file which only includes the header file compiles.

### <a name="Rs-namespace"></a>SF.20: `namespace`는 논리적 구조를 표현할 때 사용하라

##### Reason

???

##### Example

```
    ???
```

##### Enforcement

???

### <a name="Rs-unnamed"></a>SF.21: 헤더에서 이름없는(anonymous) 네임스페이스를 사용하지 마라

##### Reason

헤더 파일에 있는 익명 네임스페이스 거의 대부분이 버그이다.

##### Example

```
    ???
```

##### Enforcement

* 헤더 파일에서 사용되는 익명 네임스페이스을 찾아내 표시한다

### <a name="Rs-unnamed2"></a>SF.22: 이름없는(anonymous) 네임스페이스는 내부(internal)/노출시키지 않는(non-exported) 개체에 사용하라

##### Reason

어떤 외부에서도 내부의 익명 네임스페이스에 있는 항목들에 참조할 수 없다.
소스 파일에 정의되어 있는 모든 구현들 중 "외부에 노출되는" 항목의 정의를 뺀 나머지 모두는 익명 네임스페이스에 넣는다 생각하라.

##### Example

API 클래스와 그 멤버들은 익명 네임스페이스에 있을 수 없지만, 구현 소스 파일에 정의된 "도우미" 클래스나 함수들의 경우 익명 네임스페이스 영역에 정의되어야 한다.

```
    ???
```

##### Enforcement

* ???
