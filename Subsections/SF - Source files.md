#SF: 소스 파일 (Source Files)

인터페이스로 사용되는 선언, 구현으로 사용되는 정의를 구분해 놓아야한다.
헤더 파일은 인터페이스를 표현하고, 논리적 구조를 보여주기 위해서 사용해야 한다.
>Distinguish between declarations (used as interfaces) and definitions (used as implementations)
>Use header files to represent interfaces and to emphasize logical structure.

소스 파일에 대한 법칙:
>Source file rule summary:

* [SF.1: 코드 파일에 대해서는 `.cpp` 확장자를 사용하고, 인터페이스 파일에 대해서는 `.h` 확장자를 사용한다](#Rs-suffix)
* [SF.2: `.h` 파일은 객체에 대한 정의나, 인라인이 아닌 함수에 대한 정의를 포함하지 않도록 한다](#Rs-inline)
* [SF.3: 여러 소스 파일에서 사용되는 모든 선언들은 `.h` 파일에 적는다](#Rs-suffix)
* [SF.4: `.h` 파일들을 `포함`할 때는 다른 선언들 이전에 한다](#Rs-include-order)
* [SF.5: `.cpp` 파일은 자신의 인터페이스를 정의하고 있는 `.h` 파일들을 포함해야 한다](#Rs-consistency)
* [SF.6: `using` 지시자는 이름 공간의 전환, 기반 라이브러리, 혹은 지역적인 영역에서만 사용한다](#Rs-using)
* [SF.7: 헤더 파일에는 `using` 지시자를 사용하지 않는다](#Rs-using-directive)
* [SF.8: 모든 `.h` 파일에 `#include` 보호 문구를 사용한다](#Rs-guards)
* [SF.9: 소스 파일들 사이에는 순환 의존 구조를 만들지 않는다](#Rs-cycles)

* [SF.20: 논리적 구조를 표현하기 위하여 `namespace`를 사용한다](#Rs-namespace)
* [SF.21: 헤더에는 익명 이름공간을 사용하지 않는다](#Rs-unnamed)
* [SF.22: 내부에서만 사용하고 외부에 노출되지 않는 모든 항목들에 대해서는 익명 이름공간을 사용한다](#Rs-unnamed2)

>* [SF.1: Use a `.cpp` suffix for code files and `.h` for interface files](#Rs-suffix)
>* [SF.2: A `.h` file may not contain object definitions or non-inline function definitions](#Rs-inline)
>* [SF.3: Use `.h` files for all declarations used in multiple sourcefiles](#Rs-suffix)
>* [SF.4: Include `.h` files before other declarations in a file](#Rs-include-order)
>* [SF.5: A `.cpp` file must include the `.h` file(s) that defines its interface](#Rs-consistency)
>* [SF.6: Use `using`-directives for transition, for foundation libraries (such as `std`), or within a local scope](#Rs-using)
>* [SF.7: Don't put a `using`-directive in a header file](#Rs-using-directive)
>* [SF.8: Use `#include` guards for all `.h` files](#Rs-guards)
>* [SF.9: Avoid cyclic dependencies among source files](#Rs-cycles)
>
>* [SF.20: Use `namespace`s to express logical structure](#Rs-namespace)
>* [SF.21: Don't use an unnamed (anonymous) namespace in a header](#Rs-unnamed)
>* [SF.22: Use an unnamed (anonymous) namespace for all internal/nonexported entities](#Rs-unnamed2)



<a name="Rs-suffix"></a>
### SF.1: 코드 파일에 대해서는 `.cpp` 확장자를 사용하고, 인터페이스 파일에 대해서는 `.h` 확장자를 사용한다 (Use a `.cpp` suffix for code files and `.h` for interface files)

**근거**: 관습을 따르는 것이 좋다.
>**Reason**: Convention

**참고 사항**: `.h` 와 `.cpp` 를 사용하는 것이 권장되기는 하지만 필수는 아니다. 다른 이름들도 많이 사용된다.
예를 들자면 `.hh` 와 `.cxx` 같은 것이 있다. 이런 이름들도 동등한 방식으로 사용해야 한다.
>**Note**: The specific names `.h` and `.cpp` are not required (but recommended) and other names are in widespread use.
>Examples are `.hh` and `.cxx`. Use such names equivalently.

**예**:

	// foo.h:
	extern int a;	// 선언
	extern void foo();
	
	// foo.cpp:
	int a;	// 정의
	void foo() { ++a; }

**Example**:

	// foo.h:
	extern int a;	// a declaration
	extern void foo();
	
	// foo.cpp:
	int a;	// a definition
	void foo() { ++a; }

`foo.h` 는 `foo.cpp`에 대한 인터페이스를 제공한다. 전역 변수는 가능한 피해야 한다.
>`foo.h` provides the interface to `foo.cpp`. Global variables are best avoided.

**잘못된 예**:

	// foo.h:
	int a;	// 정의
	void foo() { ++a; }

**Example**, bad:

	// foo.h:
	int a;	// a definition
	void foo() { ++a; }

`#include<foo.h>` 문구가 프로그램 상에서 두 번 이상 처리될 경우 두 번 이상 정의가 되어있다며 링크 오류가 발생한다.
>`#include<foo.h>` twice in a program and you get a linker error for two one-definition-rule violations.
	

>**시행하기**:

* 컨벤션에 맞지 않는 파일 이름들 찾아내기
* `.h` 와 `.cpp`` (혹은 동일한 의미의 파일들)이 아래의 규칙들을 따르는지 검토하기
>**Enforcement**:
>
>* Flag non-conventional file names.
>* Check that `.h` and `.cpp`` (and equivalents) follow the rules below.


<a name="Rs-inline"></a>
### SF.2: `.h` 파일은 객체에 대한 정의나, 인라인이 아닌 함수에 대한 정의를 포함하지 않도록 한다 (A `.h` file may not contain object definitions or non-inline function definitions)

**근거**: 하나의 정의만 가져야하는 대상을 `포함`하게 되면 링크 오류로 발전할 수 있다.
>**Reason**: Including entities subject to the one-definition rule leads to linkage errors.

**Example**:

	???
	
**다른 공식**: `.h` 파일은 다음의 항목만 가질 수 있다:
>**Alternative formulation**: A `.h` file must contain only:

* 다른 `.h`파일에 대한 `#include` 구문 (#include 보호 문구와 함께)
* 템플릿
* 클래스 정의
* 함수 선언
* `extern` 선언
* `inline` 함수 정의
* `constexpr` 정의
* `const` 정의
* `using` 별칭 정의
* ???
>* `#include`s of other `.h` files (possibly with include guards
>* templates
>* class definitions
>* function declarations
>* `extern` declarations
>* `inline` function definitions
>* `constexpr` definitions
>* `const` definitions
>* `using` alias definitions
>* ???

**시행하기**: 위의 허용 목록을 검토한다.
>**Enforcement**: Check the positive list above.


<a name="Rs-suffix"></a>
### SF.3: 여러 소스 파일에서 사용되는 모든 선언들은 `.h` 파일에 적는다 (Use `.h` files for all declarations used in multiple sourcefiles)

**근거**: 관리가 편해지고 가독성이 올라간다.
>**Reason**: Maintainability. Readability.

**잘못된 예**:

	// bar.cpp:
	void bar() { cout << "bar\n"; }

	// foo.cpp:
	extern void bar();
	void foo() { bar(); }
	
**example, bad**:

	// bar.cpp:
	void bar() { cout << "bar\n"; }

	// foo.cpp:
	extern void bar();
	void foo() { bar(); }
	
`bar` 를 관리하는 사람은 `bar` 의 타입에 변경이 필요할 경우 `bar` 에 대한 모든 선언을 찾을 수가 없다.
`bar` 를 사용하는 사람은 이 인터페이스가 완전하고 올바른 것인지 알 수가 없다. 기껐해야, 나중에 링커로 부터 오류 메시지를 받는 것이 고작이다.
>A maintainer of `bar` cannot find all declarations of `bar` if its type needs changing.
>The user of `bar` cannot know if the interface used is complete and correct. At best, error messages come (late) from the linker.

**시행하기**:

* `.h` 파일에 없는 선언이 다른 소스 파일에 선언되어 있는 지 찾아내기
>**Enforcement**:
>
>* Flag declarations of entities in other source files not placed in a `.h`.


<a name="Rs-include-order"></a>
### SF.4: `.h` 파일들을 `포함`할 때는 다른 선언들 이전에 한다 (Include `.h` files before other declarations in a file)

**근거**: 문맥에 대한 종속성을 최소화하고 가독성을 높인다.
>**Reason**: Minimize context dependencies and increase readability.

**예**:

	#include<vector>
	#include<algorithms>
	#include<string>

	// ... 코드는 여기에 위치한다 ...

**Example**:

	#include<vector>
	#include<algorithms>
	#include<string>

	// ... my code here ...

**잘못된 예**:

	#include<vector>
	#include<algorithms>
	#include<string>

	// ... 코드는 여기에 위치한다 ...

**Example, bad**:

	#include<vector>
	#include<algorithms>
	#include<string>

	// ... my code here ...

**참고 사항**: 이 내용은 `.h` 와 `.cpp` 파일 모두에 해당한다.
>**Note**: This applies to both `.h` and `.cpp` files.

**예외 사항**: 좋은 예제 코드가 없나요?
>**Exception**: Are there any in good code?

**시행하기**: 쉽다.
>**Enforcement**: Easy.
	

<a name="Rs-consistency"></a>
### SF.5: `.cpp` 파일은 자신의 인터페이스를 정의하고 있는 `.h` 파일들을 포함해야 한다 (A `.cpp` file must include the `.h` file(s) that defines its interface)

**근거** 컴파일러가 좀 더 빠르게 일관성 검사를 진행할 수 있다.
>**Reason** This enables the compiler to do an early consistency check.

**잘못된 예**:

	// foo.h:
	void foo(int);
	int bar(long double);
	int foobar(int);
	
	// foo.cpp:
	void foo(int) { /* ... */ }
	int bar(double) { /* ... */ }
	double foobar(int);

**Example**, bad:

	// foo.h:
	void foo(int);
	int bar(long double);
	int foobar(int);
	
	// foo.cpp:
	void foo(int) { /* ... */ }
	int bar(double) { /* ... */ }
	double foobar(int);

`bar` 나 `foobar` 를 호출하는 프로그램은 링크할 때에야 오류를 확인할 수 있다.
>Thw errors will not be caught until link time for a program calling `bar` or `foobar`.

**예**:

	// foo.h:
	void foo(int);
	int bar(long double);
	int foobar(int);
	
	// foo.cpp:
	#include<foo.h>
		
	void foo(int) { /* ... */ }
	int bar(double) { /* ... */ }
	double foobar(int);		// 오류: 잘못된 반환 형

**Example**:

	// foo.h:
	void foo(int);
	int bar(long double);
	int foobar(int);
	
	// foo.cpp:
	#include<foo.h>
		
	void foo(int) { /* ... */ }
	int bar(double) { /* ... */ }
	double foobar(int);		// error: wrong return type

`foobar` 의 반환 형 오류가 이번엔 `foo.cpp` 파일이 컴파일될 때 즉시 발생한다.
`bar` 의 인자 형 오류는 오버로딩이 가능하기 때문에 링크 이전에는 발생하지 않지만,
체계적으로 `.h` 파일을 사용하게 되면, 프로그래머가 일찍 오류를 확인할 수 있게 해준다.
>The return-type error for `foobar` is now caught immediately when `foo.cpp` is compiled.
>The argument-type error for `bar` cannot be caught until link time because of the possibility of overloading,
>but systematic use of `.h` files increases the likelyhood that it is caught earlier by the programmer.

**시행하기**: ???
>**Enforcement**: ???


<a name="Rs-using"></a>
### SF.6: `using` 지시자는 이름 공간의 전환, 기반 라이브러리, 혹은 지역적인 영역에서만 사용한다 (Use `using`-directives for transition, for foundation libraries (such as `std`), or within a local scope)

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rs-using-directive"></a>
### SF.7: 헤더 파일에는 `using` 지시자를 사용하지 않는다 (Don't put a `using`-directive in a header file)

**근거** 헤더 파일에 `using` 지시자를 사용하는 경우 `#include` 를 사용하는 쪽에서 대체 구현을 효과적으로 구분할 수 있는 방안을 없애버린다.
>**Reason** Doing so takes away an `#include`r's ability to effectively disambiguate and to use alternatives.

**Example**:

	???

**Enforcement**: ???


<a name="Rs-guards"></a>
### SF.8: 모든 `.h` 파일에 `#include` 보호 문구를 사용한다 (Use `#include` guards for all `.h` files)

**근거**: 파일이 여러 번 `#include` 되는 것을 방지한다.
>**Reason**: To avoid files being `#include`d several times.

**예**:

	// file foobar.h:
	#ifndef FOOBAR_H
	#define FOOBAR_H
	// ... 선언들 ...
	#endif // FOOBAR_H

**Example**:

	// file foobar.h:
	#ifndef FOOBAR_H
	#define FOOBAR_H
	// ... declarations ...
	#endif // FOOBAR_H

**시행하기**: `#include` 보호 문구 없는 `.h` 파일 찾아내기
>**Enforcement**: Flag `.h` files without `#include` guards


<a name="Rs-cycles"></a>
### SF.9: 소스 파일들 사이에는 순환 의존 구조를 만들지 않는다 (Avoid cyclic dependencies among source files)

**근거**: 순환은 이해하기 어렵고, 컴파일 속도도 느려지게 한다.
향후 모듈 기능이 지원디는 경우 이 기능을 사용하도록 변경하기 어렵게 된다.
>**Reason**: Cycles complicates comprehension and slows down compilation.
>Complicates conversion to use language-supported modules (when they become available).

**참고 사항**: 단순히 `#include` 보호 장치로 처리하지 말고 실제 순환 구조를 없애야 한다.
>**Note**: Eliminate cycles; don't just break them with `#include` guards.

**잘못된 예**:

	// file1.h:
	#include "file2.h"
	
	// file2.h:
	#include "file3.h"
	
	// file3.h:
	#include "file1.h"

**Example, bad**:

	// file1.h:
	#include "file2.h"
	
	// file2.h:
	#include "file3.h"
	
	// file3.h:
	#include "file1.h"

**시행하기**: 모든 순환 구조를 찾아내기
>**Enforcement: Flag all cycles.


<a name="Rs-namespace"></a>
### SF.20: 논리적 구조를 표현하기 위하여 `namespace`를 사용한다 (Use `namespace`s to express logical structure)

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rs-unnamed"></a>
### SF.21: 헤더에는 익명 이름공간을 사용하지 않는다 (Don't use an unnamed (anonymous) namespace in a header)

**근거**: 헤더 파일에 있는 익명 이름공간 거의 대부분이 버그이다.
>**Reason**: It is almost always a bug to mention an unnamed namespace in a header file.

**Example**:

	???

**시행하기**:
* 헤더 파일에서 사용되는 익명 이름공간을 찾아내기
>**Enforcement**:
>* Flag any use of an anonymous namespace in a header file.



<a name="Rs-unnamed2"></a>
### SF.22: 내부에서만 사용하고 외부에 노출되지 않는 모든 항목들에 대해서는 익명 이름공간을 사용한다 (Use an unnamed (anonymous) namespace for all internal/nonexported entities)

**근거**:
어떤 외부에서도 내부의 익명 이름공간에 있는 항목들에 참조할 수 없다.
소스 파일에 정의되어 있는 모든 구현들 중 "외부에 노출되는" 항목의 정의를 뺀 나머지 모두는 익명 이름공간에 넣는다 생각하라.
>**Reason**:
>nothing external can depend on an entity in a nested unnamed namespace.
>Consider putting every definition in an implementation source file should be in an unnamed namespace unless that is defining an "external/exported" entity.

**예**: API 클래스와 그 멤버들은 익명 이름공간에 있을 수 없지만, 구현 소스 파일에 정의된 "도우미" 클래스나 함수들의 경우 익명 이름공간 영역에 정의되어야 한다.
>**Example**: An API class and its members can't live in an unnamed namespace; but any "helper" class or function that is defined in an implementation source file should be at an unnamed namespace scope.

	???

**Enforcement**:
* ???