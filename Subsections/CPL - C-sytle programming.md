# CPL: C-style 프로그래밍
> CPL: C-style programming

C와 C++는 밀접하게 관련이 있는 언어다.
(그것들은) 둘 다 1978년 "Classic C"에서 유래되어 ISO위원회에서 발전해 왔다.
(그것들의) 호환성을 위한 많은 시도들이 있었으나, 어느쪽도 완전한 부분집합(subset)이 되지 못한다.
> C and C++ are closely related languages.
They both originate in "Classic C" from 1978 and have evolved in ISO committees since then.
Many attempts have been made to keep them compatible, but neither is a subset of the other.

C 규칙 요약:
> C rule summary:

* [CPL.1: C보다 C++를 택하자](Rcpl-C)
* [CPL.2: 만약 반드시 C를 써야한다면, C와 C++의 공용 하위 집합(subset)을 쓰고, C++로 코드를 컴파일하자](#Rcpl-subset)
* [CPL.3: 만약 인터페이스를 위해 C를 써야한다면, 인터페이스를 쓴 코드 안에 C++를 사용하자](#Rcpl-interface)

>
* [CPL.1: Prefer C++ to C]
* [CPL.2: If you must use C, use the common subset of C and C++, and compile the C code as C++]
* [CPL.3: If you must use C for interfaces, use C++ in the code using such interfaces]


<a name="Rcpl-C"></a>
### CPL.1: C보다 C++을 택하자
> ### CPL.1: Prefer C++ to C

**근거**: C++는 더 나은 형 검사(type checking)와 더 많은 표기법을 지원한다.
이것은 고수준의 프로그래밍을 잘 지원하고 대부분의 경우에 보다 더 빠른 코드를 생성하게 한다.
> **Reason**: C++ provides better type checking and more notational support.
It provides better support for high-level programming and often generates faster code.

**예**:
> **Example**:

	char ch = 7;
	void* pv = &ch;
	int* pi = pv;	// C++(style)이 아님 // not C++
	*pi = 999;		// &ch 근처의 sizeof(int) 바이트를 덮어씀 // overwrite sizeof(int) bytes near &ch

**시행하기**: C++ 컴파일러를 사용하라.
> **Enforcement**: Use a C++ compiler.


<a name="Rcpl-subset"></a>
### CPL.2: 만약 반드시 C를 써야한다면, C와 C++의 공통 부분집합(subset)을 쓰고, C++로 코드를 컴파일하자
> ### CPL.2: If you must use C, use the common subset of C and C++, and compile the C code as C++

**근거**: 그 하위 집합은 C와 C++ 컴파일러 양자에서 컴파일 될 수 있으며, 형 검사가 "순수한 C"보다 C++로 컴파일될 때에 더 났다.
> **Reason**: That subset can be compiled with both C and C++ compilers, and when compiled as C++ is better type checked than "pure C."

**예**:
> **Example**:

	int* p1 = malloc(10*sizeof(int));                      	// C++(style)이 아니다. // not C++
	int* p2 = static_cast<int*>(malloc(10*sizeof(int)));   // C++(style)이 아니며, C-style C++이다. // not C, C-style C++
	int* p3 = new int[10];                                 // C++(의 스타일)이 아니다. // not C
	int* p4 = (int*)malloc(10*sizeof(int));                // C와 C++ 양자(style)이다. // both C and C++

**시행하기**:
> **Enforcement**:

  * C로 컴파일하도록 빌드모드가 설정되어 있다면 표시한다.
	* C++ 컴파일러는 C 확장(extension) 옵션을 사용하지 않는다면 유효한 C++코드라고 가정하고 처리할 것이다.

>
    * Flag if using a build mode that compiles code as C.
	* The C++ compiler will enforce that the code is valid C++ unless you use C extension options.


<a name="Rcpl-interface"></a>
### CPL.3:
> ### CPL.3: If you must use C for interfaces, use C++ in the calling code using such interfaces

**근거**: C++는 C보다 표현력이 풍부하고 많은 타입에 대해서 더 잘 지원한다.
> **Reason**: C++ is more expressive than C and offer better support for many types of programming.

**예**:
예를 들어, 협력업체(3rd party) C 라이브러리나 C 시스템 인터페이스를 사용하기 위해, 좀더 정확한 타입체크용으로 하위 레벨 인터페이스를 C/C++의 공통부분만으로 정의하라.
가능하면 C++ 가이드라인들을 따르는 인터페이스(더 나은 추상화(abstraction), 메모리(memory) 안정성, 자원(resource)을 위해) 안의 저수준 인터페이스를 캡슐화하며 C++ 코드 안의 C++ 인터페이스를 사용한다.
> **Example**: For example, to use a 3rd party C library or C systems interface, define the low-level interface in the common subset of C and C++ for better type checking.
Whenever possible encapsulate the low-level interface in an interface that follows the C++ guidelines (for better abstraction, memory safety, and resource safety) and use that C++ inerface in C++ code.

> ***역자주: 본문의 inerface는 오타로 보인다. interface로 정정하여 번역하였다.***

**예**: C를 C++에서 호출할 수 있다.:
> **Example**: You can call C from C++:

	// C(코드) 안 // in C:
	double sqrt(double);

	// C++(코드) 안 // in C++:
	extern "C" double sqrt(double);

	sqrt(2);

**예**: C++을 C에서 호출할 수 있다.:
> **Example**: You can call C++ from C:

	// C(코드) 안 // in C:
	X call_f(struct Y*, int);

	// C++(코드) 안 // in C++:
	extern "C" X call_f(Y* p, int i)
	{
    		return p->f(i);	// 가상함수 호출일 수도 있다. // possibly a virtual function call
	}

**시행하기**: 아무것도 필요로 하지 않음
> **Enforcement**: None needed
