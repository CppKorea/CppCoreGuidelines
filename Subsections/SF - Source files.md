#SF: 소스 파일 (Source Files)

선언 (인터페이스로 사용된다.) 과 정의 (구현으로 사용된다.)에 대해서 구분해야한다.
헤더 파일은 인터페이스를 표현하고, 논리적 구조를 강조하기 위해서 사용해야 한다.
>Distinguish between declarations (used as interfaces) and definitions (used as implementations)
>Use header files to represent interfaces and to emphasize logical structure.

소스 파일에 대한 법칙 정리:
>Source file rule summary:

* [SF.1: 코드 파일에 대해서 `.cpp` 확장자를 사용하고, 인터페이스 파일에 대해서는 `.h` 확장자를 사용하라](#Rs-suffix)
>* [SF.1: Use a `.cpp` suffix for code files and `.h` for interface files](#Rs-suffix)
* [SF.2: `.h` 파일은 객체에 대한 정의나, 인라인이 아닌 함수에 대한 정의를 포함하지 않도록 한다](#Rs-inline)
>* [SF.2: A `.h` file may not contain object definitions or non-inline function definitions](#Rs-inline)
* [SF.3: 복수의 소스 파일에서 사용되는 모든 선언은 `.h` 파일을 사용하라](#Rs-suffix)
>* [SF.3: Use `.h` files for all declarations used in multiple sourcefiles](#Rs-suffix)
* [SF.4: `.h` 파일은 다른 선언들을 하기 이전에 포함하라](#Rs-include-order)
>* [SF.4: Include `.h` files before other declarations in a file](#Rs-include-order)
* [SF.5: `.cpp` 파일은 자신의 인터페이스를 정의하고 있는 `.h` 파일들을 포함시켜야 한다](#Rs-consistency)
>* [SF.5: A `.cpp` file must include the `.h` file(s) that defines its interface](#Rs-consistency)
* [SF.6: Use `using`-directives for transition, for foundation libraries (such as `std`), or within a local scope](#Rs-using)
>* [SF.6: Use `using`-directives for transition, for foundation libraries (such as `std`), or within a local scope](#Rs-using)
* [SF.7: 헤더 파일에는 `using` 지시자를 사용하지 마라](#Rs-using-directive)
>* [SF.7: Don't put a `using`-directive in a header file](#Rs-using-directive)
* [SF.8: 모든 `.h` 파일에 `#include` 보호 문구를 사용하라](#Rs-guards)
>* [SF.8: Use `#include` guards for all `.h` files](#Rs-guards)
* [SF.9: 소스 파일들 사이에는 순환 의존 구조를 만들지 말라](#Rs-cycles)
>* [SF.9: Avoid cyclic dependencies among source files](#Rs-cycles)

* [SF.20: 논리적 구조를 표현하기 위하여 `namespace`를 사용하라](#Rs-namespace)
>* [SF.20: Use `namespace`s to express logical structure](#Rs-namespace)
* [SF.21: 헤더에는 이름없는(익명의) 네임스페이스를 사용하지 마라](#Rs-unnamed)
>* [SF.21: Don't use an unnamed (anonymous) namespace in a header](#Rs-unnamed)
* [SF.22: 내부에서만 사용하는/외부에 노출하지 않는 모든 항목들에 대해서는 이름없는(익명의) 네임스페이를 사용하라](#Rs-unnamed2)
>* [SF.22: Use an unnamed (anonymous) namespace for all internal/nonexported entities](#Rs-unnamed2)



<a name="Rs-suffix"></a>
### SF.1: 코드 파일에 대해서 `.cpp` 확장자를 사용하고, 인터페이스 파일에 대해서는 `.h` 확장자를 사용하라 (Use a `.cpp` suffix for code files and `.h` for interface files)

**Reason**: 관습을 따르는 것이 좋다.
>**Reason**: Convention

**Note**: 정확히 `.h` 와 `.cpp` 를 사용하는 것이 (권장되기는 하지만) 필수는 아니고 다른 이름들도 많이 사용된다.
예를 들자면 `.hh` 와 `.cxx` 같은 것이 있다. 이런 이름들도 동등한 방식으로 사용해야 한다.
>**Note**: The specific names `.h` and `.cpp` are not required (but recommended) and other names are in widespread use.
>Examples are `.hh` and `.cxx`. Use such names equivalently.

**Example**:

	// foo.h:
	extern int a;	// a declaration
	extern void foo();
	
	// foo.cpp:
	int a;	// a definition
	void foo() { ++a; }

`foo.h` 는 `foo.cpp`에 대한 인터페이스를 제공한다. 전역 변수를 가능한 피해야한다.
>`foo.h` provides the interface to `foo.cpp`. Global variables are best avoided.

**Example**, bad:

	// foo.h:
	int a;	// a definition
	void foo() { ++a; }

`#include<foo.h>` 가 프로그램 상에서 두 번 이상 처리될 경우 두 번 이상 정의가 되어있다며 링크 오류가 발생하게 된다.
>`#include<foo.h>` twice in a program and you get a linker error for two one-definition-rule violations.
	

**Enforcement**:

* 컨벤션에 맞지 않는 파일 이름들 찾아내기
>* Flag non-conventional file names.
* `.h` 와 `.cpp`` (혹은 동일한 의미의 파일들) 이 아래의 규칙을 따르는지 검토하기
>* Check that `.h` and `.cpp`` (and equivalents) follow the rules below.


<a name="Rs-inline"></a>
### SF.2: `.h` 파일은 객체에 대한 정의나, 인라인이 아닌 함수에 대한 정의를 포함하지 않도록 한다 (A `.h` file may not contain object definitions or non-inline function definitions)

**이유**: 하나의 정의만 가져야하는 대상을 인클루드하게 되면 링크 오류가 발생한다.
>**Reason**: Including entities subject to the one-definition rule leads to linkage errors.

**Example**:

	???
	
**Alternative formulation**: `.h` 파일은 다음의 항목만 가질 수 있다:
>**Alternative formulation**: A `.h` file must contain only:

* 다른 `.h`파일에 대한 `#include` 구문 (인클루드 보호 장치와 함께)
>* `#include`s of other `.h` files (possibly with include guards
* 템플릿
>* templates
* 클래스 정의
>* class definitions
* 함수 선언
>* function declarations
* `extern` 선언
>* `extern` declarations
* `inline` 함수 정의
>* `inline` function definitions
* `constexpr` 정의
>* `constexpr` definitions
* `const` 정의
>* `const` definitions
* `using` 별칭 정의
>* `using` alias definitions
* ???

**Enforcement**: 위의 허용 목록을 검토한다.
>**Enforcement**: Check the positive list above.


<a name="Rs-suffix"></a>
### SF.3: 복수의 소스 파일에서 사용되는 모든 선언은 `.h` 파일을 사용하라 (Use `.h` files for all declarations used in multiple sourcefiles)

**Reason**: 관리성. 가독성
>**Reason**: Maintainability. Readability.

**example, bad**:

	// bar.cpp:
	void bar() { cout << "bar\n"; }

	// foo.cpp:
	extern void bar();
	void foo() { bar(); }
	
`bar` 를 관리하는 사람은 `bar` 의 타입에 변경이 필요할 경우 `bar` 에 대한 모든 선언을 찾을 수가 없다.
`bar` 를 사용하는 사람은 이 인터페이스가 완전한지, 올바른 것인지 알 수가 없다. 최선의 경우, 나중에 링커로 부터 오류 메시지를 받는 것이 고작이다.
>A maintainer of `bar` cannot find all declarations of `bar` if its type needs changing.
>The user of `bar` cannot know if the interface used is complete and correct. At best, error messages come (late) from the linker.

**Enforcement**:

* 다른 소스 파일에 있는 요소가 `.h` 파일이 아닌 곳에 선언되어 있는 것 찾아내기
>* Flag declarations of entities in other source files not placed in a `.h`.


<a name="Rs-include-order"></a>
### SF.4: `.h` 파일은 다른 선언들을 하기 이전에 포함하라 (Include `.h` files before other declarations in a file)

**Reason**: 문맥 의존성을 최소화하고 가독성을 높인다.
>**Reason**: Minimize context dependencies and increase readability.

**Example**:

	#include<vector>
	#include<algorithms>
	#include<string>

	// ... my code here ...

**Example, bad**:

	#include<vector>
	#include<algorithms>
	#include<string>

	// ... my code here ...

**Note**: 이 항목은 `.h` 와 `.cpp` 파일 모두에 적용된다.
>**Note**: This applies to both `.h` and `.cpp` files.

**Exception**: 좋은 예제 코드가 없나요?
>**Exception**: Are there any in good code?

**Enforcement**: 쉽다.
>**Enforcement**: Easy.
	

<a name="Rs-consistency"></a>
### SF.5: `.cpp` 파일은 자신의 인터페이스를 정의하고 있는 `.h` 파일들을 포함시켜야 한다 (A `.cpp` file must include the `.h` file(s) that defines its interface)

**Reason** 컴파일러가 빠른 일관성 검사를 진행하도록 한다.
>**Reason** This enables the compiler to do an early consistency check.

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

`foobar` 의 반환 타입 오류가 이번엔 `foo.cpp` 파일이 컴파일될 때 즉시 발생한다.
`bar` 의 인자 타입 오류는 오버로딩의 가능하기 때문에 링크 이전에는 발생하지는 않지만,
체계적으로 `.h` 파일을 사용하게 되면, 좀 더 일찍 프로그래머가 오류를 확인할 수 있는 가능성을 열어준다.
>The return-type error for `foobar` is now caught immediately when `foo.cpp` is compiled.
>The argument-type error for `bar` cannot be caught until link time because of the possibility of overloading,
>but systematic use of `.h` files increases the likelyhood that it is caught earlier by the programmer.

**Enforcement**: ???


<a name="Rs-using"></a>
### SF.6: Use `using`-directives for transition, for foundation libraries (such as `std`), or within a local scope

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rs-using-directive"></a>
### SF.7: 헤더 파일에는 `using` 지시자를 사용하지 마라 (Don't put a `using`-directive in a header file)

**Reason** 헤더 파일에 `using` 지시자를 사용하는 경우 `#include` 를 사용하는 쪽에서 효과적으로 모호함을 해결하여 다른 항목을 사용할 수 있는 능력을 없애버린다.
>**Reason** Doing so takes away an `#include`r's ability to effectively disambiguate and to use alternatives.

**Example**:

	???

**Enforcement**: ???


<a name="Rs-guards"></a>
### SF.8: 모든 `.h` 파일에 `#include` 보호 문구를 사용하라 (Use `#include` guards for all `.h` files)

**Reason**: 파일이 여러번 `#include` 되는 것을 방지한다.
>**Reason**: To avoid files being `#include`d several times.

**Example**:

	// file foobar.h:
	#ifndef FOOBAR_H
	#define FOOBAR_H
	// ... declarations ...
	#endif // FOOBAR_H

**Enforcement**: `#include` 보호 문구 없는 `.h` 파일 찾아내기
>**Enforcement**: Flag `.h` files without `#include` guards


<a name="Rs-cycles"></a>
### SF.9: 소스 파일들 사이에는 순환 의존 구조를 만들지 말라 (Avoid cyclic dependencies among source files)

**Reason**: 순환이 발생하면 이해하기 어렵고, 컴파일 속도를 느려지게 한다.
언어가 지원하는 모듈 기능이 사용 가능하게 된 경우 모듈 기능을 사용하도록 변환하기 어렵다.
>**Reason**: Cycles complicates comprehension and slows down compilation.
>Complicates conversion to use language-supported modules (when they become available).

**Note**: 순환 구조를 없애야 한다. 단순히 `#include` 보호 장치로 처리해서는 안 된다.
>**Note**: Eliminate cycles; don't just break them with `#include` guards.

**Example, bad**:

	// file1.h:
	#include "file2.h"
	
	// file2.h:
	#include "file3.h"
	
	// file3.h:
	#include "file1.h"

**Enforcement: 모든 순환 구조를 찾아내기
>**Enforcement: Flag all cycles.


<a name="Rs-namespace"></a>
### SF.20: 논리적 구조를 표현하기 위하여 `namespace` 를 사용하라 (Use `namespace`s to express logical structure)

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rs-unnamed"></a>
### SF.21: 헤더 파일에는 이름 없는 네임스페이스를 사용하지 말라 (Don't use an unnamed (anonymous) namespace in a header)

**Reason**: 헤더 파일에 이름 없는 네임스페이스가 있을 경우 거의 대부분이 버그 상황이다.
>**Reason**: It is almost always a bug to mention an unnamed namespace in a header file.

**Example**:

	???

**Enforcement**:
* 헤더 파일에서 사용되는 이름 없는 네임스페이스를 찾아내기
>* Flag any use of an anonymous namespace in a header file.



<a name="Rs-unnamed2"></a>
### SF.22: 모든 내부/외부로 노출되지 않는 항목들에 대해서는 이름 없는 네임스페이스를 사용하라 (Use an unnamed (anonymous) namespace for all internal/nonexported entities)

**Reason**:
어떤 외부에서도 내부의 이름 없는 네임스페이스에 있는 항목들에 의존할 수 없다.
소스 파일에 정의되어 있는 모든 구현들 중 "외부에 노출되는" 항목들에 대한 정의를 뺀 나머지 모두는 이름 없는 네임스페이스에 넣는 것을 염두에 두라.
>nothing external can depend on an entity in a nested unnamed namespace.
>Consider putting every definition in an implementation source file should be in an unnamed namespace unless that is defining an "external/exported" entity.

**Example**: API 클래스와 그 멤버들은 이름 없는 네임스페이스에 존재할 수 없지만, 구현 소스 파일에 정의된 "헬퍼" 클래스나 함수들의 경우 이름 없는 네임스페이스 영역에 정의되어야 한다.
>**Example**: An API class and its members can't live in an unnamed namespace; but any "helper" class or function that is defined in an implementation source file should be at an unnamed namespace scope.

	???

**Enforcement**:
* ???