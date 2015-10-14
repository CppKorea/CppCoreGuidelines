# CPL: C-style 프로그래밍
> CPL: C-style programming

C와 C++는 밀접하게 관계된 언어다.
(그것들은) 둘 다 "Classic C", 1978에서 유래되어 이후 ISO위원회에서 발전해왔다.
(그것들의) 호환성을 위해 많은 시도들이 있었으나, 어느 쪽도 다른 것의 하위 집합(subset)이 아니다.
> C and C++ are closely related languages.
They both originate in "Classic C" from 1978 and have evolved in ISO committees since then.
Many attempts have been made to keep them compatible, but neither is a subset of the other.

C 규칙 요약:
> C rule summary:

* [CPL.1: C보다 C++를 택하자](Rcpl-C)
* [CPL.2: 만약 반드시 C를 써야한다면, C와 C++의 공용 하위 집합(subset)을 쓰며, C++로 코드를 컴파일하자](#Rcpl-subset)
* [CPL.3: 만약 인터페이스를 위해 C를 써야한다면, 인터페이스를 쓴 코드 안에 C++를 사용하자](#Rcpl-interface)

>
* [CPL.1: Prefer C++ to C]
* [CPL.2: If you must use C, use the common subset of C and C++, and compile the C code as C++]
* [CPL.3: If you must use C for interfaces, use C++ in the code using such interfaces]


<a name="Rcpl-C"></a>
### CPL.1: C보다 C++을 택하자
> ### CPL.1: Prefer C++ to C

**이유**: C++는 더 나은 형 검사(type checking)와 더 많은 표기법을 지원한다.
이것은 더 나은 고수준의 프로그래밍과 자주 더 빠른 코드 생성을 제공한다.
> **Reason**: C++ provides better type checking and more notational support.
It provides better support for high-level programming and often generates faster code.

**예제**:
> **Example**:

```C++
char ch = 7;
void* pv = &ch;
int* pi = pv;	// C++(의 표기)가 아님 // not C++
*pi = 999;		// &ch 근처의 sizeof(int) 바이트를 덮어씀 // overwrite sizeof(int) bytes near &ch
```

**시행; 강제**: C++ 컴파일러를 사용하라. 
> **Enforcement**: Use a C++ compiler.


<a name="Rcpl-subset"></a>
### CPL.2: If you must use C, use the common subset of C and C++, and compile the C code as C++

**Reason**: That subset can be compiled with both C and C++ compilers, and when compiled as C++ is better type checked than "pure C."

**Example**:

	int* p1 = malloc(10*sizeof(int));                      // not C++
	int* p2 = static_cast<int*>(malloc(10*sizeof(int)));   // not C, C-style C++
	int* p3 = new int[10];                                 // not C
	int* p4 = (int*)malloc(10*sizeof(int));                // both C and C++

**Enforcement**:

    * Flag if using a build mode that compiles code as C.
	* The C++ compiler will enforce that the code is valid C++ unless you use C extension options.


<a name="Rcpl-interface"></a>
### CPL.3: If you must use C for interfaces, use C++ in the calling code using such interfaces

**Reason**: C++ is more expressive than C and offer better support for many types of programming.

**Example**: For example, to use a 3rd party C library or C systems interface, define the low-level interface in the common subset of C and C++ for better type checking.
Whenever possible encapsulate the low-level interface in an interface that follows the C++ guidelines (for better abstraction, memory safety, and resource safety) and use that C++ inerface in C++ code.

**Example**: You can call C from C++:

    // in C:
    double sqrt(double);

    // in C++:
    extern "C" double sqrt(double);

    sqrt(2);

**Example**: You can call C++ from C:

    // in C:
    X call_f(struct Y*, int);

    // in C++:
    extern "C" X call_f(Y* p, int i)
    {
        return p->f(i);	// possibly a virtual function call
    }

**Enforcement**: None needed
