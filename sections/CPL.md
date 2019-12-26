# <a name="S-cpl"></a>CPL: C 스타일 프로그래밍

C와 C++는 밀접하게 관련된 언어다.
두 언어 모두 1978년 "Classic C"에서 유래되어 이후 ISO 위원회에서 발전해 왔다.
두 언어간의 호환성을 위해 많은 시도들이 있었으나, 어느 쪽도 다른 것의 부분집합이 아니다.

C 규칙 요약:

* [CPL.1: C보다 C++를 사용하라](#Rcpl-C)
* [CPL.2: 반드시 C를 써야한다면, C와 C++의 공통집합을 쓰고, C++로 코드를 컴파일하라](#Rcpl-subset)
* [CPL.3: C 인터페이스를 써야한다면, 호출되는 코드 안에서는 C++를 사용하라](#Rcpl-interface)

### <a name="Rcpl-C"></a>CPL.1: C보다 C++를 사용하라

##### Reason

C++는 더 나은 형 검사(type checking)와 더 많은 표기법을 지원한다.
이것은 고수준의 프로그래밍을 위해 더 나은 지원을 제공하고 대게 더 빠른 코드를 생성한다.

##### Example

```c++
    char ch = 7;
    void* pv = &ch;
    int* pi = pv;   // C++ (style)이 아님
    *pi = 999;      // &ch 근처의 sizeof(int) 바이트를 덮어쓴다
```

The rules for implicit casting to and from `void*` in C are subtle and unenforced.
In particular, this example violates a rule against converting to a type with stricter alignment.

##### Enforcement

C++ 컴파일러를 사용하라.

### <a name="Rcpl-subset"></a>CPL.2:반드시 C를 써야한다면, C와 C++의 공통집합을 쓰고, C++로 코드를 컴파일하라

##### Reason

C와 C++ 컴파일러 모두 그 코드를 컴파일 할 수 있으며, C++로 컴파일 할 경우 "순수한 C"보다 타입 검사가 더 잘 수행된다.

##### Example

```c++
    int* p1 = malloc(10 * sizeof(int));                      // not C++
    int* p2 = static_cast<int*>(malloc(10 * sizeof(int)));   // not C, C-style C++
    int* p3 = new int[10];                                   // not C
    int* p4 = (int*) malloc(10 * sizeof(int));               // both C and C++
```

##### Enforcement

* 빌드 시에 C 언어로 컴파일하고 있다면 표시를 남겨라
* C++ 컴파일러는 C 확장(extension) 옵션을 사용하지 않는다면 유효한 C++코드로 (가정하고) 처리할 것이다

### <a name="Rcpl-interface"></a>CPL.3: C 인터페이스를 써야한다면, 호출되는 코드 안에서는 C++를 사용하라

##### Reason
C++는 C보다 표현력이 풍부하며 많은 종류의 프로그래밍을 위해 더 나은 지원을 제공한다.

##### Example
예를 들어, 외부(3rd party) C 라이브러리나 C 시스템 인터페이스를 사용할 때, 더 정확한 타입 검사를 위해 저수준의 인터페이스는 C와 C++의 부분집합만으로 정의하자.
가능하면 C++ 가이드라인들을 따르는 인터페이스(더 나은 추상화(abstraction), 메모리(memory) 안정성, 자원(resource)을 위해) 안의 저수준 인터페이스를 캡슐화하며 C++ 코드 안의 C++ 인터페이스를 사용한다.

##### Example

You can call C from C++:

```c++
    // in C:
    double sqrt(double);

    // in C++:
    extern "C" double sqrt(double);

    sqrt(2);
```

##### Example

C++에서 C를 호출할 수 있다:

```c++
    // in C:
    X call_f(struct Y*, int);

    // in C++:
    extern "C" X call_f(Y* p, int i)
    {
        return p->f(i);   // 아마도 가상함수 호출일 것이다
    }
```

##### Enforcement

특별히 없음
