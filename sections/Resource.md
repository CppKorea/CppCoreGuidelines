# <a name="S-resource"></a>R: 자원 관리

이 장은 자원과 관련된 규칙을 포함하고 있다. 
자원이란 획득해야만 하고, (명시적 혹은 묵시적으로) 해제된다. 주로 메모리, 파일 핸들, 소켓, 잠금(lock) 같은 것들이다. 
반드시 해체되어야 하는 이유는 자원 부족인데, 지연된 형태의 해체조차도 이런 문제를 야기할 수 있다.
기본적인 목표는 어떤 자원도 누수가 발생하지 않고, 필요 이상으로 자원을 소유하지 않는 것이다.
자원을 해체하는 책임을 가지는 주체를 우리는 소유자(owner)라고 한다.

드물게 자원 누수가 용인되거나 최선인 경우가 있다:
입력을 기반으로 단순히 출력하는 프로그램을 구현하고 입력에 비례하여 필요한 메모리 양이 증가한다면, (성능과 프로그래밍을 용이하게 하기 위한) 최선의 전략은 어떤 자원도 삭제하지 않는 것이다.
가장 큰 입력을 처리하기 위해서 충분한 메모리를 가졌다면 자원이 소비되도록 내버려 둬라. 다만 뭔가 잘못을 했다면 상황에 알맞는 에러 메시지를 주도록 해라. 이런 경우는 더 이상 언급하지 않겠다.

* 자원 관리 규칙 요약:

  * [R.1: 자원 핸들과 RAII(자원 획득시 초기화)를 사용해서 자동적으로 관리되도록 하라](#Rr-raii)
  * [R.2: 인터페이스에서는, 서로 다른 오브젝트를 나타낼 경우에만 원시 포인터를 사용하라](#Rr-use-ptr)
  * [R.3: 원시 포인터(`T*`)는 소유를 의미하지 않는다](#Rr-ptr)
  * [R.4: 참조(a `T&`)는 소유를 의미하지 않는다](#Rr-ref)
  * [R.5: 가능한 자동 변수를 사용하라, 불필요한 동적 할당을 하지마라](#Rr-scoped)
  * [R.6: `const`가 아닌 전역 변수의 사용을 피하라](#Rr-global)

* 할당과 해제 규칙 요약:

  * [R.10: `malloc()`과 `free()`의 사용을 피하라](#Rr-mallocfree)
  * [R.11: 직접적으로 `new`와 `delete` 호출하는 것을 피하라](#Rr-newdelete)
  * [R.12: 명시적으로 자원이 생성되는 경우 즉시 관리 개체에게 결과를 전달하라](#Rr-immediate-alloc)
  * [R.13: 하나의 표현식에서는 한번의 자원 할당을 수행하라](#Rr-single-alloc)
  * [R.14: ??? 배열 혹은 포인터 인자 전달](#Rr-ap)
  * [R.15: 짝을 이루는 할당과 해제는 항상 오버로드 하라](#Rr-pair)

* <a name="Rr-summary-smartptrs"></a>스마트 포인터 규칙 요약:

  * [R.20: 소유권을 나타낼 때는 `unique_ptr`나 `shared_ptr`를 사용하라](#Rr-owner)
  * [R.21: 소유권을 공유하지 않는다면 `shared_ptr`보다 `unique_ptr`를 사용하라](#Rr-unique)
  * [R.22: `shared_ptr`를 만들 때는 `make_shared()`를 사용하라](#Rr-make_shared)
  * [R.23: `unique_ptr`를 만들 때는 `make_unique()`를 사용하라](#Rr-make_unique)
  * [R.24: `shared_ptr`의 순환 참조를 막기 위해 `std::weak_ptr`를 사용하라](#Rr-weak_ptr)
  * [R.30: 수명주기를 표현하고자 할 때만 스마트 포인터를 인자로 사용하라](#Rr-smartptrparam)
  * [R.31: 표준 스마트 포인터를 사용하지 않고 있다면, 표준에서 사용하는 기본 패턴을 사용하라](#Rr-smart)
  * [R.32: 함수가 `widget`의 소유권을 맡는다는 것을 표현하기 위해 `unique_ptr<widget>`인자를 사용하라](#Rr-uniqueptrparam)
  * [R.33: 함수가 `widget`을 생성한다는 것을 표현하기 위해 `unique_ptr<widget>&`를 인자로 사용하라](#Rr-reseat)
  * [R.34: 함수가 소유자 중 하나라는 것을 표현하기 위해 `shared_ptr<widget>`을 인자로 사용하라](#Rr-sharedptrparam-owner)
  * [R.35: 함수가 공유 포인터를 생성한다는 것을 표현하기 위해 `shared_ptr<widget>&`를 인자로 사용하라](#Rr-sharedptrparam)
  * [R.36: Take a `const shared_ptr<widget>&` parameter to express that it might retain a reference count to the object ???](#Rr-sharedptrparam-const)
  * [R.37: Do not pass a pointer or reference obtained from an aliased smart pointer](#Rr-smartptrget)

### <a name="Rr-raii"></a>R.1:자원 핸들과 RAII(자원 획득시 초기화)를 사용해서 자동적으로 관리되도록 하라

##### Reason

수동 자원 관리의 복잡성과 누출을 피하기 위한 방법을 알아본다. 
C++ 언어적 강제인 생성자 소멸자 대칭은 `fopen`/`fclose`,  그리고 `lock`/`unlock`, `new`/`delete`과 같은 자원 획득/해체 함수의 짝과 같은 구조를 가진다.

이 특징을 사용해서 자원의 획득/해체시 짝 함수 호출이 필요한 자원을 다룰 때는 생성자에서 자원을 획득하고 소멸자에서 해체가 강제되도록 개체로 리소스를 캡슐화해라.

##### Example, bad

Consider:

```c++
    void send(X* x, cstring_span destination)
    {
        auto port = open_port(destination);
        my_mutex.lock();
        // ...
        send(port, x);
        // ...
        my_mutex.unlock();
        close_port(port);
        delete x;
    }
```

이 코드에서는 모든 경우에 `unlock`, `close_port`, `delete`가 정확히 순서대로 호출되어야 한다는 점을 고려해야 한다. 만약 `...`로 표시된 코드에서 예외가 던져지면, 그로인해 `x`는 누출되고 `my_mutex`는 잠금을 해제하지 않게 된다.

##### Example

Consider:

```c++
    void send(unique_ptr<X> x, cstring_span destination)  // x owns the X
    {
        Port port{destination};            // port owns the PortHandle
        lock_guard<mutex> guard{my_mutex}; // guard owns the lock
        // ...
        send(port, x);
        // ...
    } // automatically unlocks my_mutex and deletes the pointer in x
```

모든 자원 관리가 자동화되었고 예외와 상관없이 모든 경로에서 한번 수행된다. 추가적으로 함수가 포인터 소유권을 가져간 것도 보여주고 있다.

`Port`는 어떻게 구현할 수 있을까? 자원을 캡슐화하는 간단한 래퍼로 구현할 수 있다:

```c++
    class Port {
        PortHandle port;
    public:
        Port(cstring_span destination) : port{open_port(destination)} { }
        ~Port() { close_port(port); }
        operator PortHandle() { return port; }

        // port handles can't usually be cloned, so disable copying and assignment if necessary
        Port(const Port&) = delete;
        Port& operator=(const Port&) = delete;
    };
```

##### Note

소멸자를 가진 클래스로 표현되지 않고 다루기 힘든 자원인 경우 클래스로 감싸서 자원을 관리하거나 [`finally`](#S-GSL)를 사용하라.

**See also**: [RAII](#Rr-raii)

### <a name="Rr-use-ptr"></a>R.2: In interfaces, use raw pointers to denote individual objects (only)

##### Reason

배열은 컨테이너 타입(가령, `vector`(소유)이나 `span`(비 소유))으로 가장 잘 표현된다. 이런 컨테이너와 뷰는 범위 검사를 위한 충분한 정보를 가지고 있다.

##### Example, bad

```c++
    void f(int* p, int n)   // n is the number of elements in p[]
    {
        // ...
        p[2] = 7;   // bad: subscript raw pointer
        // ...
    }
```

컴파일러 주석을 읽지 않는다. 또한 다른 코드를 읽지 않고는 `p`가 정말로 `n` 만큼을 가르키는지 알 수 없다. 
대신 `span`을 사용하라.

##### Example

```c++
    void g(int* p, int fmt)   // print *p using format #fmt
    {
        // ... uses *p and p[0] only ...
    }
```

##### Exception

C 스타일 문자열은 0으로 끝나는 문자 배열을 포인터로 전달하기도 한다.
관례를 따른다는 것을 보여주기 위해 `char*`보다는 `zstring`을 사용하라

##### Note

하나의 원소를 위해서는 참조자를 사용할 수도 있다. 그러나 `nullptr`이 가능한 경우라면 참조가 좋은 대안은 아니다.

##### Enforcement

컨테이너 또는 뷰, 반복자(iterator)가 아닌 포인터에서 주소 계산(++ 포함)을 삼가해라
이 규칙은 오래된 코드 베이스에 적용된다면 많은 양의 false positive를 만들 수 있다.

* 간단한 포인터로 전달하는 배열 이름을 삼가해라??
* 컨테이너, 뷰, 반복자가 아닌 포인터 연산에는 표시를 남겨라. (이는 `++`를 포함한다)  
  이 규칙은 오래된 코드에서는 엄청나게 많은 거짓 양성(false positive)을 만들 것이다.
* 배열을 포인터를 사용해 전달할 경우 표시를 남겨라

### <a name="Rr-ptr"></a>R.3: A raw pointer (a `T*`) is non-owning

##### Reason

C++ 표준뿐만 아니라 대부분의 경우 원시 포인터는 소유를 하지 않는다.
신뢰할 수 있고 효과적인 방법으로 개체를 제거하기 위해서는 개체를 소유하는 포인터가 필요하다.

##### Example

```c++
    void f()
    {
        int* p1 = new int{7};           // bad: raw owning pointer
        auto p2 = make_unique<int>(7);  // OK: the int is owned by a unique pointer
        // ...
    }
```

`unique_ptr`는 개체의 제거를 보장하기 때문에 메모리 누수를 차단해준다. (예외 발생에서도 마찬가지다.) `T*`는 그렇지 않다.

##### Example

```c++
    template<typename T>
    class X {
        // ...
    public:
        T* p;   // bad: it is unclear whether p is owning or not
        T* q;   // bad: it is unclear whether q is owning or not
    };
```

명시적인 소유권을 만들어 이 문제를 해결할 수 있다:

```c++
    template<typename T>
    class X2 {
        // ...
    public:
        owner<T*> p;  // OK: p is owning
        T* q;         // OK: q is not owning
    };
```

##### Exception

주요 예외사항은 레거시 코드라고 할 수 있다. 특히 ABI를 통해서 C 혹은 C 스타일 C++ 인터페이스와 호환성을 가져야 하는 경우가 그렇다. `T*`를 소유하는 방식을 위반하는 억 단위의 코드가 존재한다는 사실을 무시할 수는 없다.
20년 묵은 "레거시" 코드를 최신 C++ 코드로 변환할 수 있는 툴이 있으면 좋을것이다. 이런 툴의 개발과 툴의 사용을 독려할것이고 또한 이 가이드라인이 도움이 되었으면 좋겠다. 
가시적인 성과가 보일때까지 몇 년은 더 걸릴것이다: 최신 코드로 바꿀수 있게 되기전에 "레거시 코드"가 더 빠르게 생성될지도 모른다.

이 모든 코드는 다시 작성 될순 없고 (좋은 코드 변환 소프트웨어가 있더라도) 적어도 당장은 아닐것이다. 이 문제는 모든 포인터를  `unique_ptr`와 `shared_ptr`로 대체하는 것으로는 해결할 수 없다. 
partly because we need/use owning "raw pointers" as well as simple pointers in the implementation of our fundamental resource handles.
For example, common `vector` implementations have one owning pointer and two non-owning pointers.
Many ABIs (and essentially all interfaces to C code) use `T*`s, some of them owning.
Some interfaces cannot be simply annotated with `owner` because they need to remain compilable as C
(although this would be a rare good use for a macro, that expands to `owner` in C++ mode only).

##### Note

`owner<T*>` has no default semantics beyond `T*`. It can be used without changing any code using it and without affecting ABIs.
It is simply a indicator to programmers and analysis tools.
For example, if an `owner<T*>` is a member of a class, that class better have a destructor that `delete`s it.

##### Example, bad

원시 포인터를 반환하는것은 호출자에게 수명 관리에 불확실성을 심어준다; 다시 말해, 누가 포인터를 통해 개체를 제거해야 하는가?

```c++
    Gadget* make_gadget(int n)
    {
        auto p = new Gadget{n};
        // ...
        return p;
    }

    void caller(int n)
    {
        auto p = make_gadget(n);   // remember to delete p
        // ...
        delete p;
    }
```

[leak](#???)로 인한 고통뿐만 아니라 이는 쓸데없이 많고 미심쩍은 할당과 해제를 야기할 수 있다. 
만약 Gadget을 함수 바깥으로 가져오는 비용이 크지 않다면, 단순히 값으로 반환하는 것도 한 방법이다. (["out" return values](#Rf-out)를 보라):

```c++
    Gadget make_gadget(int n)
    {
        Gadget g{n};
        // ...
        return g;
    }
```

##### Note
이 규칙은 펙토리 함수에 적용될 수 있다.

##### Note

If pointer semantics are required (e.g., because the return type needs to refer to a base class of a class hierarchy (an interface)), return a "smart pointer."

##### Enforcement

* (Simple) Warn on `delete` of a raw pointer that is not an `owner<T>`.
* (Moderate) Warn on failure to either `reset` or explicitly `delete` an `owner<T>` pointer on every code path.
* (Simple) Warn if the return value of `new` is assigned to a raw pointer.
* (Simple) Warn if a function returns an object that was allocated within the function but has a move constructor.
  Suggest considering returning it by value instead.

### <a name="Rr-ref"></a>R.4: A raw reference (a `T&`) is non-owning

##### Reason

C++ 표준뿐만 아니라 대부분의 경우 참조는 소유를 하지 않는다. 
신뢰할 수 있고 효과적인 방법으로 개체를 제거하기 위해서는 개체를 소유하는 포인터가 필요하다.

##### Example

```c++
    void f()
    {
        int& r = *new int{7};  // bad: raw owning reference
        // ...
        delete &r;             // bad: violated the rule against deleting raw pointers
    }
```

**See also**: [The raw pointer rule](#Rr-ptr)

##### Enforcement

See [the raw pointer rule](#Rr-ptr)

### <a name="Rr-scoped"></a>R.5: Prefer scoped objects, don't heap-allocate unnecessarily

##### Reason

유효범위 내 개체는 지역 개체, 전역 개체, 혹은 멤버를 의미한다. 이는 해당 시점의 범위 내에서 별도의 할당/해제 비용이 발생하지 않는다는 것을 의미한다.
유효범위 내 개체의 멤버들은 생성자와 소멸자에 의해 수명이 관리된다.

##### Example

다음 예는 불필요한 할당화 해제를 하기 떄문에 비효율적이고, 예외에 취약하며, `...` 부분에서는 누수가 발생할 수 있다:

```c++
    void f(int n)
    {
        auto p = new Gadget{n};
        // ...
        delete p;
    }
```

대신, 지역 변수를 사용하라:

```c++
    void f(int n)
    {
        Gadget g{n};
        // ...
    }
```

##### Enforcement

* (Moderate) Warn if an object is allocated and then deallocated on all paths within a function. Suggest it should be a local `auto` stack object instead.
* (Simple) Warn if a local `Unique_ptr` or `shared_ptr` is not moved, copied, reassigned or `reset` before its lifetime ends.

### <a name="Rr-global"></a>R.6: Avoid non-`const` global variables

##### Reason

전역 변수는 모든 곳에서 접근될 수 있고 명백히 관련 없는 개체들 사이에 말도 안되는 의존성을 만들 수 있다. 에러의 원인 중 잘 알려진 것이기도 하다.

**Warning**: 전역 개체의 초기화 순서는 보장되지 않는다. 상수로 전역 개체를 초기화하고 싶다면, `const` 개체에 대해서도 초기화 순서가 정의되지 않았을 수 있다는 점을 명심하라.

##### Exception

싱글톤 패턴 보다는 전역 개체가 나을 수도 있다.

##### Exception

변경할 수 없는(`const`) 전역 개체는 이런 문제를 발생시키지 않는다.

##### Enforcement

(??? NM: Obviously we can warn about non-`const` statics ... do we want to?)

## <a name="SS-alloc"></a>R.alloc: Allocation and deallocation

### <a name="Rr-mallocfree"></a>R.10: Avoid `malloc()` and `free()`

##### Reason

`malloc()`과 `free()`는 생성자와 소멸자를 지원하지 않는다. `new` 과 `delete`와 섞어서 사용하지 마라.

##### Example

```c++
    class Record {
        int id;
        string name;
        // ...
    };

    void use()
    {
        // p1 may be nullptr
        // *p1 is not initialized; in particular,
        // that string isn't a string, but a string-sized bag of bits
        Record* p1 = static_cast<Record*>(malloc(sizeof(Record)));

        auto p2 = new Record;

        // unless an exception is thrown, *p2 is default initialized
        auto p3 = new(nothrow) Record;
        // p3 may be nullptr; if not, *p3 is default initialized

        // ...

        delete p1;    // error: cannot delete object allocated by malloc()
        free(p2);    // error: cannot free() object allocated by new
    }
```

In some implementations that `delete` and that `free()` might work, or maybe they will cause run-time errors.

##### Exception

There are applications and sections of code where exceptions are not acceptable.
Some of the best such examples are in life-critical hard-real-time code.
Beware that many bans on exception use are based on superstition (bad)
or by concerns for older code bases with unsystematic resource management (unfortunately, but sometimes necessary).
In such cases, consider the `nothrow` versions of `new`.

##### Enforcement

Flag explicit use of `malloc` and `free`.

### <a name="Rr-newdelete"></a>R.11: Avoid calling `new` and `delete` explicitly

##### Reason

`new`로 반환된 포인터는 리소스 핸들(`delete`를 호출할 수 있는)에 종속되어야 한다.
`new`로 반환된 포인터가 원시 포인터에 할당되면 누수가 발생할 수 있다.

##### Note

In a large program, a naked `delete` (that is a `delete` in application code, rather than part of code devoted to resource management)
is a likely bug: if you have N `delete`s, how can you be certain that you don't need N+1 or N-1?
The bug may be latent: it may emerge only during maintenance.
If you have a naked `new`, you probably need a naked `delete` somewhere, so you probably have a bug.

##### Enforcement

(Simple) Warn on any explicit use of `new` and `delete`. Suggest using `make_unique` instead.

### <a name="Rr-immediate-alloc"></a>R.12: Immediately give the result of an explicit resource allocation to a manager object

##### Reason

그렇지 않으면, 예외나 반환이 자원 누수를 야기할 수 있다.

##### Example, bad

```c++
    void f(const string& name)
    {
        FILE* f = fopen(name, "r");            // open the file
        vector<char> buf(1024);
        auto _ = finally([f] { fclose(f); });  // remember to close the file
        // ...
    }
```

The allocation of `buf` may fail and leak the file handle.

##### Example

```c++
    void f(const string& name)
    {
        ifstream f{name};   // open the file
        vector<char> buf(1024);
        // ...
    }
```

The use of the file handle (in `ifstream`) is simple, efficient, and safe.

##### Enforcement

* Flag explicit allocations used to initialize pointers (problem: how many direct resource allocations can we recognize?)

### <a name="Rr-single-alloc"></a>R.13: Perform at most one explicit resource allocation in a single expression statement

##### Reason

If you perform two explicit resource allocations in one statement, you could leak resources because the order of evaluation of many subexpressions, including function arguments, is unspecified.

##### Example

```c++
    void fun(shared_ptr<Widget> sp1, shared_ptr<Widget> sp2);
```

This `fun` can be called like this:

```c++
    // BAD: potential leak
    fun(shared_ptr<Widget>(new Widget(a, b)), shared_ptr<Widget>(new Widget(c, d)));
```

This is exception-unsafe because the compiler may reorder the two expressions building the function's two arguments.
In particular, the compiler can interleave execution of the two expressions:
Memory allocation (by calling `operator new`) could be done first for both objects, followed by attempts to call the two `Widget` constructors.
If one of the constructor calls throws an exception, then the other object's memory will never be released!

This subtle problem has a simple solution: Never perform more than one explicit resource allocation in a single expression statement.
For example:

```c++
    shared_ptr<Widget> sp1(new Widget(a, b)); // Better, but messy
    fun(sp1, new Widget(c, d));
```

The best solution is to avoid explicit allocation entirely use factory functions that return owning objects:

```c++
    fun(make_shared<Widget>(a, b), make_shared<Widget>(c, d)); // Best
```

Write your own factory wrapper if there is not one already.

##### Enforcement

* Flag expressions with multiple explicit resource allocations (problem: how many direct resource allocations can we recognize?)

### <a name="Rr-ap"></a>R.14: ??? array vs. pointer parameter

##### Reason

An array decays to a pointer, thereby losing its size, opening the opportunity for range errors.

##### Example

```
    ??? what do we recommend: f(int*[]) or f(int**) ???
```

**Alternative**: Use `span` to preserve size information.

##### Enforcement

Flag `[]` parameters.

### <a name="Rr-pair"></a>R.15: Always overload matched allocation/deallocation pairs

##### Reason

Otherwise you get mismatched operations and chaos.

##### Example

```c++
    class X {
        // ...
        void* operator new(size_t s);
        void operator delete(void*);
        // ...
    };
```

##### Note

If you want memory that cannot be deallocated, `=delete` the deallocation operation.
Don't leave it undeclared.

##### Enforcement

Flag incomplete pairs.

## <a name="SS-smart"></a>R.smart: Smart pointers

### <a name="Rr-owner"></a>R.20: Use `unique_ptr` or `shared_ptr` to represent ownership

##### Reason

자원 누수를 막을 수 있다.

##### Example

Consider:

```c++
    void f()
    {
        X x;
        X* p1 { new X };              // see also ???
        unique_ptr<T> p2 { new X };   // unique ownership; see also ???
        shared_ptr<T> p3 { new X };   // shared ownership; see also ???
        auto p4 = make_unique<X>();   // unique_ownership, preferable to the explicit use "new"
        auto p5 = make_shared<X>();   // shared ownership, preferable to the explicit use "new"
    }
```

This will leak the object used to initialize `p1` (only).

##### Enforcement

(Simple) Warn if the return value of `new` or a function call with return value of pointer type is assigned to a raw pointer.

### <a name="Rr-unique"></a>R.21: Prefer `unique_ptr` over `shared_ptr` unless you need to share ownership

##### Reason

`unique_ptr`는 개념적으로 단순하고 예측가능하며(파괴가 일어날 때를 알고) 빠르다(사용 횟수를 암시적으로 관리하지 않는다).

##### Example, bad
이 코드는 불필요하게 참조 횟수를 증가 및 유지하고 있다.

```c++
    void f()
    {
        shared_ptr<Base> base = make_shared<Derived>();
        // use base locally, without copying it -- refcount never exceeds 1
    } // destroy base
```

##### Example

이 코드가 더 효율적이다:

```c++
    void f()
    {
        unique_ptr<Base> base = make_unique<Derived>();
        // use base locally
    } // destroy base
```

##### Enforcement

(쉬움) 만약 함수 내에서 개체 할당에 `shared_ptr`을 사용하지만, `shared_ptr`을 리턴하지 않거나 `shared_ptr&`를 필요로 하는 함수에 전달하고 있다면 경고하라. 대신 `unique_ptr` 사용을 권하라.

### <a name="Rr-make_shared"></a>R.22: Use `make_shared()` to make `shared_ptr`s

##### Reason

만약 개체를 처음 만들고 `shared_ptr`의 생성자에 전달하면, `make_shared()`를 사용할 때보다 (거의 확실히) 할당(그리고 나중의 해제)을 한번 더 하게 된다. 개체와는 독립적으로 참조 카운트를 할당해야 하기 때문이다.

##### Example

Consider:

```c++
    shared_ptr<X> p1 { new X{2} }; // bad
    auto p = make_shared<X>(2);    // good
```

`make_shared()` 버전은 `X`가 단 한 번만 사용되며, 그렇기에 명시적으로 `new`를 사용하는 버전보다 코드가 짧다(게다가 빠르다).

##### Enforcement

(Simple) Warn if a `shared_ptr` is constructed from the result of `new` rather than `make_shared`.

### <a name="Rr-make_unique"></a>R.23: Use `make_unique()` to make `unique_ptr`s

##### Reason

For convenience and consistency with `shared_ptr`.

##### Note

`make_unique()` is C++14, but widely available (as well as simple to write).

##### Enforcement

(Simple) Warn if a `unique_ptr` is constructed from the result of `new` rather than `make_unique`.

### <a name="Rr-weak_ptr"></a>R.24: Use `std::weak_ptr` to break cycles of `shared_ptr`s

##### Reason

`shared_ptr`은 참조 카운트를 사용하는데, 순환 구조에서 이는 절대로 0이 되지 않는다.  때문에 우리는 순환 구조를 파괴할 수 있는 방법이 필요하다.

##### Example

```c++
    #include <memory>

    class bar;

    class foo
    {
    public:
      explicit foo(const std::shared_ptr<bar>& forward_reference)
        : forward_reference_(forward_reference)
      { }
    private:
      std::shared_ptr<bar> forward_reference_;
    };

    class bar
    {
    public:
      explicit bar(const std::weak_ptr<foo>& back_reference)
        : back_reference_(back_reference)
      { }
      void do_something()
      {
        if (auto shared_back_reference = back_reference_.lock()) {
          // Use *shared_back_reference
        }
      }
    private:
      std::weak_ptr<foo> back_reference_;
    };
```

##### Note

??? (HS: 많은 사람들은 "순환을 끊는"이라고 말하는 반면 나는 "일시적인 임시 공유"가 더 적절하다고 생각한다.)   
??? (BS: 순환 끊기는 반드시 해야할 일인데, 어떻게 "일시적인 소유권 공유"를 할 것인가. `shared_ptr`을 사용함으로써 "일시적으로 소유권 공유"를 할 수 있다.)

##### Enforcement

??? 아마도 불가능하다. 정적으로 순환 구조를 찾아낼 수 있다면, `weak_ptr`를 사용할 필요가 없다.

### <a name="Rr-smartptrparam"></a>R.30: Take smart pointers as parameters only to explicitly express lifetime semantics

##### Reason

Accepting a smart pointer to a `widget` is wrong if the function just needs the `widget` itself.
It should be able to accept any `widget` object, not just ones whose lifetimes are managed by a particular kind of smart pointer.
A function that does not manipulate lifetime should take raw pointers or references instead.

##### Example, bad

```c++
    // callee
    void f(shared_ptr<widget>& w)
    {
        // ...
        use(*w); // only use of w -- the lifetime is not used at all
        // ...
    };

    // caller
    shared_ptr<widget> my_widget = /* ... */;
    f(my_widget);

    widget stack_widget;
    f(stack_widget); // error
```

##### Example, good

```c++
    // callee
    void f(widget& w)
    {
        // ...
        use(w);
        // ...
    };

    // caller
    shared_ptr<widget> my_widget = /* ... */;
    f(*my_widget);

    widget stack_widget;
    f(stack_widget); // ok -- now this works
```

##### Enforcement

* (Simple) Warn if a function takes a parameter of a smart pointer type (that overloads `operator->` or `operator*`) that is copyable but the function only calls any of: `operator*`, `operator->` or `get()`.
  Suggest using a `T*` or `T&` instead.
* Flag a parameter of a smart pointer type (a type that overloads `operator->` or `operator*`) that is copyable/movable but never copied/moved from in the function body, and that is never modified, and that is not passed along to another function that could do so. That means the ownership semantics are not used.
  Suggest using a `T*` or `T&` instead.

### <a name="Rr-smart"></a>R.31: 표준 스마트 포인터를 사용하지 않고 있다면, 표준에서 사용하는 기본 패턴을 사용하라

##### Reason

다음 섹션들의 규칙들 또한 다른 종류의 서드파티 혹은 커스텀 스마트 포인터 등에서도 동작할 것이며 성능과 정확성 문제를 일으키는 흔한 스마트 포인터 에러에 대한 분석에 매우 유용할 것이다. 당신은 사용하고 있는 모든 스마트 포인터에 대해서 이 규칙이 작동해야 한다.

스마트 포인터는 단항 연산자 `*`와 `->`를 오버로드하는 (기본 또는 특수 템플릿을 포함한) 타입을 의미한다:

* 복사할 수 있다면, 참조 카운트를 유지하는 `shared_ptr`처럼 동작한다
* 복사할 수 없다면, 고유한 `unique_ptr`처럼 동작한다

##### Example

```c++
    // use Boost's intrusive_ptr
    #include <boost/intrusive_ptr.hpp>
    void f(boost::intrusive_ptr<widget> p)  // error under rule 'sharedptrparam'
    {
        p->foo();
    }

    // use Microsoft's CComPtr
    #include <atlbase.h>
    void f(CComPtr<widget> p)               // error under rule 'sharedptrparam'
    {
        p->foo();
    }
```

두 경우 모두 [`sharedptrparam` 가이드라인](#Rr-smartptrparam)에서는 오류가 있다:  
`p`는 `shared_ptr`이지만, 공유에 대해서는 아무것도 하지 않고 있으며, 값에 의한 전달은 비효율적이다;
이 함수들이 widget의 생명주기에 영향을 미친다면 스마트 포인터를 넘겨받아야만 한다. 
widget이 `nullptr`이 될 수 있다면 `widget*`를 넘겨받아야 하고, 그게 아닌 이상적인 상황은 함수가 `widget&`를 넘겨받아야 한다.

이 스마트 포인터들은 `shared_ptr` 개념에 부합한다. 때문에 이 규칙은 고정관념과는 다르게 흔히 발생할 수 있는 비효율을 노출시킨다.

### <a name="Rr-uniqueptrparam"></a>R.32: Take a `unique_ptr<widget>` parameter to express that a function assumes ownership of a `widget`

##### Reason

Using `unique_ptr` in this way both documents and enforces the function call's ownership transfer.

##### Example

```c++
    void sink(unique_ptr<widget>); // takes ownership of the widget

    void uses(widget*);            // just uses the widget
```

##### Example, bad

```c++
    void thinko(const unique_ptr<widget>&); // usually not what you want
```

##### Enforcement

* (Simple) Warn if a function takes a `Unique_ptr<T>` parameter by lvalue reference and does not either assign to it or call `reset()` on it on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `Unique_ptr<T>` parameter by reference to `const`. Suggest taking a `const T*` or `const T&` instead.

### <a name="Rr-reseat"></a>R.33: Take a `unique_ptr<widget>&` parameter to express that a function reseats the`widget`

##### Reason

Using `unique_ptr` in this way both documents and enforces the function call's reseating semantics.

##### Note

"reseat" means "making a pointer or a smart pointer refer to a different object."

##### Example

```c++
    void reseat(unique_ptr<widget>&); // "will" or "might" reseat pointer
```

##### Example, bad

```c++
    void thinko(const unique_ptr<widget>&); // usually not what you want
```

##### Enforcement

* (Simple) Warn if a function takes a `Unique_ptr<T>` parameter by lvalue reference and does not either assign to it or call `reset()` on it on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `Unique_ptr<T>` parameter by reference to `const`. Suggest taking a `const T*` or `const T&` instead.

### <a name="Rr-sharedptrparam-owner"></a>R.34: Take a `shared_ptr<widget>` parameter to express that a function is part owner

##### Reason

This makes the function's ownership sharing explicit.

##### Example, good

```c++
    void share(shared_ptr<widget>);            // share -- "will" retain refcount

    void may_share(const shared_ptr<widget>&); // "might" retain refcount

    void reseat(shared_ptr<widget>&);          // "might" reseat ptr
```

##### Enforcement

* (Simple) Warn if a function takes a `shared_ptr<T>` parameter by lvalue reference and does not either assign to it or call `reset()` on it on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `shared_ptr<T>` by value or by reference to `const` and does not copy or move it to another `shared_ptr` on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `shared_ptr<T>` by rvalue reference. Suggesting taking it by value instead.

### <a name="Rr-sharedptrparam"></a>R.35: Take a `shared_ptr<widget>&` parameter to express that a function might reseat the shared pointer

##### Reason

This makes the function's reseating explicit.

##### Note

"reseat" means "making a reference or a smart pointer refer to a different object."

##### Example, good

```c++
    void share(shared_ptr<widget>);            // share -- "will" retain refcount

    void reseat(shared_ptr<widget>&);          // "might" reseat ptr

    void may_share(const shared_ptr<widget>&); // "might" retain refcount
```

##### Enforcement

* (Simple) Warn if a function takes a `shared_ptr<T>` parameter by lvalue reference and does not either assign to it or call `reset()` on it on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `shared_ptr<T>` by value or by reference to `const` and does not copy or move it to another `shared_ptr` on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `shared_ptr<T>` by rvalue reference. Suggesting taking it by value instead.

### <a name="Rr-sharedptrparam-const"></a>R.36: Take a `const shared_ptr<widget>&` parameter to express that it might retain a reference count to the object ???

##### Reason

This makes the function's ??? explicit.

##### Example, good

```c++
    void share(shared_ptr<widget>);            // share -- "will" retain refcount

    void reseat(shared_ptr<widget>&);          // "might" reseat ptr

    void may_share(const shared_ptr<widget>&); // "might" retain refcount
```

##### Enforcement

* (Simple) Warn if a function takes a `shared_ptr<T>` parameter by lvalue reference and does not either assign to it or call `reset()` on it on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `shared_ptr<T>` by value or by reference to `const` and does not copy or move it to another `shared_ptr` on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `shared_ptr<T>` by rvalue reference. Suggesting taking it by value instead.

### <a name="Rr-smartptrget"></a>R.37: Do not pass a pointer or reference obtained from an aliased smart pointer

##### Reason

Violating this rule is the number one cause of losing reference counts and finding yourself with a dangling pointer.
Functions should prefer to pass raw pointers and references down call chains.
At the top of the call tree where you obtain the raw pointer or reference from a smart pointer that keeps the object alive.
You need to be sure that the smart pointer cannot inadvertently be reset or reassigned from within the call tree below.

##### Note

To do this, sometimes you need to take a local copy of a smart pointer, which firmly keeps the object alive for the duration of the function and the call tree.

##### Example

Consider this code:

```c++
    // global (static or heap), or aliased local ...
    shared_ptr<widget> g_p = ...;

    void f(widget& w)
    {
        g();
        use(w);  // A
    }

    void g()
    {
        g_p = ...; // oops, if this was the last shared_ptr to that widget, destroys the widget
    }
```

The following should not pass code review:

```c++
    void my_code()
    {
        // BAD: passing pointer or reference obtained from a nonlocal smart pointer
        //      that could be inadvertently reset somewhere inside f or it callees
        f(*g_p);

        // BAD: same reason, just passing it as a "this" pointer
         g_p->func();
    }
```

The fix is simple -- take a local copy of the pointer to "keep a ref count" for your call tree:

```c++
    void my_code()
    {
        // cheap: 1 increment covers this entire function and all the call trees below us
        auto pin = g_p;

        // GOOD: passing pointer or reference obtained from a local unaliased smart pointer
        f(*pin);

        // GOOD: same reason
        pin->func();
    }
```

##### Enforcement

* (Simple) Warn if a pointer or reference obtained from a smart pointer variable (`Unique_ptr` or `shared_ptr`) that is nonlocal, or that is local but potentially aliased, is used in a function call. If the smart pointer is a `shared_ptr` then suggest taking a local copy of the smart pointer and obtain a pointer or reference from that instead.
