# <a name="S-resource"></a>R: 자원 관리

이 장은 자원과 관련된 규칙을 포함하고 있다. 
자원이란 획득해야만 하고, (명시적 혹은 묵시적으로) 해제된다. 주로 메모리, 파일 핸들, 소켓, 잠금(lock) 같은 것들이다. 
반드시 해체되어야 하는 이유는 자원 부족인데, 지연된 형태의 해체조차도 이런 문제를 야기할 수 있다.
기본적인 목표는 어떤 자원도 누수가 발생하지 않고, 필요 이상으로 자원을 소유하지 않는 것이다.
자원을 해체하는 책임을 가지는 주체를 우리는 소유자(owner)라고 한다.

드물게 자원 누수가 용인되거나 최선인 경우가 있다:
입력을 기반으로 단순히 출력하는 프로그램을 구현하고 입력에 비례하여 필요한 메모리 양이 증가한다면, (성능과 프로그래밍을 용이하게 하기 위한) 최선의 전략은 어떤 자원도 삭제하지 않는 것이다.
가장 큰 입력을 처리하기 위해서 충분한 메모리를 가졌다면 자원이 소비되도록 내버려 둬라. 다만 뭔가 잘못을 했다면 상황에 알맞는 에러 메시지를 주도록 해라. 이런 경우는 더 이상 언급하지 않겠다.

자원 관리 규칙 요약:

* [R.1: 자원 핸들과 RAII(자원 획득시 초기화)를 사용해서 자동적으로 관리되도록 하라](#Rr-raii)
* [R.2: 인터페이스에서는, 포인터는 서로 다른 개체들을 표시하기 위해서만 사용하라](#Rr-use-ptr)
* [R.3: 원시 포인터(`T*`)는 소유를 의미하지 않는다](#Rr-ptr)
* [R.4: 참조(`T&`)는 소유를 의미하지 않는다](#Rr-ref)
* [R.5: 유효 범위 안의 개체를 선호하라. 불필요한 동적할당을 하지 마라](#Rr-scoped)
* [R.6: `const`가 아닌 전역 변수를 지양하라](#Rr-global)

할당과 해제 규칙 요약:

* [R.10: `malloc()`과 `free()`의 사용을 피하라](#Rr-mallocfree)
* [R.11: 명시적인 `new`와 `delete` 호출을 지양하라](#Rr-newdelete)
* [R.12: 명시적인 할당의 결과는 즉시 관리 개체에 전달하라](#Rr-immediate-alloc)
* [R.13: 하나의 표현식 구문에서 명시적 자원 할당은 최대 한번만 수행하라](#Rr-single-alloc)
* [R.14: `[]` 매개변수 대신 `span`을 사용하라](#Rr-ap)
* [R.15: 할당/해제가 짝을 이루도록 중복정의하라](#Rr-pair)

<a name="Rr-summary-smartptrs"></a>스마트 포인터 규칙 요약:

* [R.20: 소유권을 나타내기 위해 `unique_ptr` 혹은 `shared_ptr`를 사용하라](#Rr-owner)
* [R.21: 소유권을 공유할 필요가 없다면 `shared_ptr`보다는 `unique_ptr`를 선호하라](#Rr-unique)
* [R.22: `shared_ptr`를 만들때는 `make_shared()`를 사용하라](#Rr-make_shared)
* [R.23: `unique_ptr`를 만들때는 `make_unique()`를 사용하라](#Rr-make_unique)
* [R.24: `shared_ptr`의 순환참조를 부수기 위해 `weak_ptr`를 사용하라](#Rr-weak_ptr)
* [R.30: 수명주기 의미구조를 표현하기 위해서만 스마트 포인터를 매개변수로 사용하라](#Rr-smartptrparam)
* [R.31: 표준 스마트 포인터를 사용하지 않고 있다면, 표준에서 사용하는 기본 패턴을 사용하라](#Rr-smart)
* [R.32: 함수가 `widget`의 소유권을 맡는다는 것을 표현하기 위해 `unique_ptr<widget>`를 매개변수로 사용하라](#Rr-uniqueptrparam)
* [R.33: 함수가 `widget`을 새로 설정한다는 것을 표현하기 위해 `unique_ptr<widget>&`를 사용하라](#Rr-reseat)
* [R.34: 함수가 소유자 중 하나라는 것을 표현하기 위해 `shared_ptr<widget>`를 매개변수로 사용하라](#Rr-sharedptrparam-owner)
* [R.35: 함수가 공유 포인터를 재설정한다는 것을 표현하기 위해 `shared_ptr<widget>&`를 매개변수로 사용하라](#Rr-sharedptrparam)
* [R.36: 함수가 개체에 대한 참조 카운트를 유지한다는 것을 표현하기 위해 `const shared_ptr<widget>&`을 매개변수로 사용하라 ???](#Rr-sharedptrparam-const)
* [R.37: 재명명(aliased)된 스마트 포인터에서 획득한 포인터 혹은 참조를 전달하지 마라](#Rr-smartptrget)

### <a name="Rr-raii"></a>R.1: 자원 핸들과 RAII(자원 획득시 초기화)를 사용해서 자동적으로 관리되도록 하라

##### Reason

수동 자원 관리의 복잡성과 누출을 피하기 위한 방법을 알아본다. 
C++ 언어적 강제인 생성자 소멸자 대칭은 `fopen`/`fclose`,  그리고 `lock`/`unlock`, `new`/`delete`과 같은 자원 획득/해체 함수의 짝과 같은 구조를 가진다.

이 특징을 사용해서 자원의 획득/해체시 짝 함수 호출이 필요한 자원을 다룰 때는 생성자에서 자원을 획득하고 소멸자에서 해체가 강제되도록 개체로 리소스를 캡슐화해라.

##### Example, bad

다음과 같은 경우를 생각해보라:

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

다음과 같은 경우를 생각해보라:

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

##### See Also

[RAII](#Rr-raii)

### <a name="Rr-use-ptr"></a>R.2: 인터페이스에서는, 포인터는 서로 다른 개체들을 표시하기 위해서만 사용하라

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

컨테이너 또는 뷰, 반복자(iterator)가 아닌 포인터에서 주소 계산(++ 포함)을 삼가하라.
이 규칙이 오래된 코드에 적용된다면 수많은 false positive를 만들 수 있다.

* 간단한 포인터로 전달하는 배열 이름을 지적하라
* 컨테이너, 뷰, 반복자가 아닌 포인터 연산을 지적하라. (이는 `++`를 포함한다)  
* 배열을 포인터를 사용해 전달할 경우 지적하라

### <a name="Rr-ptr"></a>R.3: 원시 포인터는(`T*`) 소유를 의미하지 않는다

> raw pointer: 원시 포인터

##### Reason

C++ 표준 뿐만 아니라 대부분의 경우 원시 포인터는 소유를 하지 않는다.
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

이 모든 코드는 다시 작성 될순 없고 (좋은 코드 변환 소프트웨어가 있더라도) 적어도 당장은 아닐것이다. 이 문제는 모든 포인터를 `unique_ptr`와 `shared_ptr`로 대체하는 것으로는 해결할 수 없다. 부분적으로 이는 기초적인 자원 핸들을 구현할때, 마치 내부의 단순한 포인터들처럼 소유하는 원시 포인터들을 사용하고 또 필요로 하기 때문이다.

예를 들어, 일반적인 `vector`구현은 하나의 소유하는 포인터와 두개의 소유하지 않는 포인터들을 가진다.
많은 ABI들(그리고 특히 C 코드로 이어지는 모든 인터페이스들)은 `T*`를 사용한다. 어떤 경우는 소유를 의미하기도 한다. C와 호환성을 유지해야 한다면 단순히 `owner`를 사용할(annotate) 수 없다.
(이런 경우는 `owner`가 C++에서만 적용되도록 하는 매크로가 좋은 사용이 될 수도 있다).

##### Note

`owner<T*>`에는 `T*`이상의 의미가 없다. 이를 사용하는 코드를 변경하거나 ABI에 영향을 주지 않으면서 사용할 수 있다. 

이 타입은 프로그래머와 분석도구들을 위한 지표일 뿐이다.
예를 들어, `owner<T*>`가 어떤 클래스의 멤버라면, 그 클래스는 해당 멤버를 `delete`하는 소멸자를 가지는 것이 나을 것이다. 

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

[leak](#???)으로 인한 고통뿐만 아니라 이는 쓸데없이 많고 미심쩍은 할당과 해제를 야기할 수 있다. 
만약 Gadget을 함수 바깥으로 가져오는 비용이 크지 않다면, 단순히 값으로 반환하는 것도 한 방법이다. (["out" 반환](#Rf-out) 항목을 보라):

```c++
    Gadget make_gadget(int n)
    {
        Gadget g{n};
        // ...
        return g;
    }
```

##### Note

이 규칙은 팩토리 함수에 적용될 수 있다.

##### Note

만약 포인터 의미구조가 필요하다면 (상위 클래스(인터페이스)를 참조한다거나), 스마트 포인터를 반환하라

##### Enforcement

* (쉬움) `owner<T>`가 아닌 원시 포인터를 `delete`하면 경고하라.
* (중간) Warn on failure to either `reset` or explicitly `delete` an `owner<T>` pointer on every code path.
* (쉬움) `new`의 결과가 원시 포인터에 대입된다면 경고하라.
* (쉬움) 함수 안에서 이동 생성이 가능한 개체가 할당되는 경우 경고하라. 대신 값으로 반환하는 것을 고려하도록 제안하라

### <a name="Rr-ref"></a>R.4: 참조는(`T&`) 소유를 의미하지 않는다

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

##### See Also

[원시 포인터 규칙들](#Rr-ptr)

##### Enforcement

[원시 포인터 규칙들](#Rr-ptr)을 보라

### <a name="Rr-scoped"></a>R.5: 유효 범위 안의 개체를 선호하라. 불필요한 동적할당을 하지 마라

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

* (중간) 함수 내의 모든 경로에서 개체가 할당되고 해제된다면 경고하라. 스택을 사용하는 지역 `auto` 개체를 사용하도록 제안하라.
* (단순) 지역적으로 사용된 `unique_ptr` 또는 `shared_ptr`가 이동, 복사, 대입되거나 소멸하기 전에 `reset`되면 경고하라.

### <a name="Rr-global"></a>R.6: `const`가 아닌 전역 변수를 지양하라

##### Reason

전역 변수는 모든 곳에서 접근될 수 있고 명백히 관련 없는 개체들 사이에 말도 안되는 의존성을 만들 수 있다. 오류의 원인 중 잘 알려진 것이기도 하다.

**경고**: 전역 개체의 초기화 순서는 보장되지 않는다. 상수로 전역 개체를 초기화하고 싶다면, `const` 개체에 대해서도 초기화 순서가 정의되지 않았을 수 있다는 점을 명심하라.

##### Exception

싱글톤 패턴 보다는 전역 개체가 나을 수도 있다.

##### Exception

변경할 수 없는(`const`) 전역 개체는 이런 문제를 발생시키지 않는다.

##### Enforcement

(??? NM: `const`가 아닌 static 변수들에 대해서도 경고할 수 있을 것 같은데 ... 그렇게 해야 하는가?)

## <a name="SS-alloc"></a>R.alloc: 할당과 해제

### <a name="Rr-mallocfree"></a>R.10: `malloc()`과 `free()`의 사용을 피하라

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

`delete`나 `free()`가 동작할 수도 있지만, 실행시간 오류를 발생시킬수도 있다.

##### Exception

예외가 허용되지 않는 응용 프로그램이나 코드 구간이 있다. 생명이 달려있어서 주어진 시간 안에 반응해야 하는(life-critical hard-real-time) 경우가 이에 해당한다.
많은 예외에 대한 금지사항(ban)들이 (나쁜) 미신 혹은 체계적으로 자원을 관리하지 않은(불행하게도, 종종 필요한 경우가 생긴다) 오래된 코드에서 부터 나온 걱정에서 나왔다는 점을 인지하라.
그런 경우, `new` 연산자의 `nothrow` 버전을 고려하라

##### Enforcement

명시적인  `malloc`과 `free`의 사용을 지적하라

### <a name="Rr-newdelete"></a>R.11: 명시적인 `new`와 `delete` 호출을 지양하라

##### Reason

`new`로 반환된 포인터는 리소스 핸들(`delete`를 호출할 수 있는)에 종속되어야 한다.
`new`로 반환된 포인터가 원시 포인터에 할당되면 누수가 발생할 수 있다.

##### Note

규모가 큰 프로그램에서, 노출된 `delete`는 (이는 자원을 관리하는 코드가 아닌 다른 응용 프로그램 코드에 `delete`가 있는 경우를 의미한다) 버그와 같다: 만약 N번의 `delete` 호출이 있다면, N+1번 혹은 N-1번이 필요한지 어떻게 확신할 수 있겠는가?

버그가 숨어있을 수도 있다: 유지보수 중에 새롭게 나타날수도 있다.

노출된 `new`가 있다면, 아마도 어디선가 노출된 `delete`가 필요할 것이다. 그렇다면 버그가 있을 가능성이 높다.

##### Enforcement

(단순) `new`와 `delete`가 명시적으로 사용되면 경고하라. 대신 `make_unique`를 사용하도록 제안하라.

### <a name="Rr-immediate-alloc"></a>R.12: 명시적인 할당의 결과는 즉시 관리 개체에 전달하라

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

`buf`에서의 할당은 실패할 수 있고 그 경우 파일 핸들의 누수가 발생한다.

##### Example

```c++
    void f(const string& name)
    {
        ifstream f{name};   // open the file
        vector<char> buf(1024);
        // ...
    }
```

이와 같은 (`ifstream` 내부의) 파일 핸들 사용은 단순하며, 효과적이고, 안전하다.

##### Enforcement

* 포인터를 초기화하기 위해 명시적인 할당을 했다면 지적하라 (문제: 직접적인 자원 할당을 얼마나 많이 인지할 수 있을 것인가?)

### <a name="Rr-single-alloc"></a>R.13: 하나의 표현식 구문에서 명시적 자원 할당은 최대 한번만 수행하라

##### Reason

한 구문에서 두번의 명시적 자원 할당을 수행하면, 함수 인자를 포함해 하위 표현식의 불특정한 평가 순서에 따라 자원 누수가 발생할수도 있다. 

##### Example

```c++
    void fun(shared_ptr<Widget> sp1, shared_ptr<Widget> sp2);
```

이 `fun`호출은 아래와 같을 수 있다:

```c++
    // BAD: potential leak
    fun(shared_ptr<Widget>(new Widget(a, b)), shared_ptr<Widget>(new Widget(c, d)));
```

이는 예외 안전하지 않은데, 컴파일러가 함수의 두 인자를 생성하면서 순서를 바꿀 수도 있기 때문이다.

특히, 컴파일러는 두 표현식을 뒤섞어(interleave) 수행할수도 있다: (`operator new`를 호출함으로써) 두 개체의 메모리 할당이 먼저 수행되고, `Widget`의 생성자를 호출하려는 시도가 이어질 것이다. 만약 둘 중 하나가 예외를 던지면, 다른 한 개체는 해제되지 않는다!

이 미묘한 문제의 해결책은 간단하다: 한 표현식 구문 내에서 명시적 자원 할당을 한번 이상 하지마라.

예를 들어:

```c++
    shared_ptr<Widget> sp1(new Widget(a, b)); // Better, but messy
    fun(sp1, new Widget(c, d));
```

최상의 해결책은 소유하는 개체를 반환하는 팩토리 함수를 사용해서 명시적 할당을 완전히 피하는 것이다:

```c++
    fun(make_shared<Widget>(a, b), make_shared<Widget>(c, d)); // Best
```

팩토리 함수를 사용하고 있지 않다면 새롭게 작성하라.

##### Enforcement

* 표현식 내에서 여러번의 명시적 자원 할당이 있다면 경고하라(문제: 직접적인 자원 할당을 얼마나 많이 인지할 수 있을 것인가?)

### <a name="Rr-ap"></a>R.14: [] 파라미터는 피하고 span을 사용하라

> 역주:
> * Parameter: 매개변수
> * Argument: 전달인자

##### Reason

배열은 포인터로 축약(decay)되고, 그 결과 길이를 알 수 없게 된다. 이로 인해 범위 오류가 발생할 수 있다. 크기 정보를 보존하기 위해 `span`을 사용하라.

##### Example

```c++
    void f(int[]);          // not recommended

    void f(int*);           // not recommended for multiple objects
                            // (a pointer should point to a single object, do not subscript)

    void f(gsl::span<int>); // good, recommended
```

##### Enforcement

`[]` 파라미터 대신 `span`을 사용하라.

### <a name="Rr-pair"></a>R.15: 할당/해제가 짝을 이루도록 중복정의하라

##### Reason

연산이 불일치하면 혼돈을 겪게 될 것이다.

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

할당한 후 해제되지 않는 메모리를 원한다면, 해제 연산에 `=delete`를 사용하라. 선언이 없는 채로 남겨두지 마라.

##### Enforcement

연산이 짝을 이루지 않으면 지적한다.

## <a name="SS-smart"></a>R.smart: 스마트 포인터

### <a name="Rr-owner"></a>R.20: 소유권을 나타내기 위해 `unique_ptr` 혹은 `shared_ptr`를 사용하라

##### Reason

자원 누수를 막을 수 있다.

##### Example

다음과 같은 경우를 생각해보라:

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

위 코드는 `p1`를 초기화하는데 사용된 개체에(만) 누수가 발생한다.

##### Enforcement

(단순) `new` 혹은 포인터 타입의 반환값이 원시 포인터에 대입되면 경고한다.

### <a name="Rr-unique"></a>R.21: 소유권을 공유할 필요가 없다면 `shared_ptr`보다는 `unique_ptr`를 선호하라

##### Reason

`unique_ptr`는 개념적으로 단순하고 예측가능하며(파괴가 일어날 때를 알고) 빠르다 (사용 횟수를 암시적으로 관리하지 않는다).

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

### <a name="Rr-make_shared"></a>R.22: `shared_ptr`를 만들때는 `make_shared()`를 사용하라

##### Reason

만약 개체를 처음 만들고 `shared_ptr`의 생성자에 전달하면, `make_shared()`를 사용할 때보다 (거의 확실히) 할당(그리고 나중의 해제)을 한번 더 하게 된다. 개체와는 독립적으로 참조 카운트를 할당해야 하기 때문이다.

##### Example

다음과 같은 경우를 생각해보라:

```c++
    shared_ptr<X> p1 { new X{2} }; // bad
    auto p = make_shared<X>(2);    // good
```

`make_shared()` 버전은 `X`가 단 한 번만 사용되며, 그렇기에 명시적으로 `new`를 사용하는 버전보다 코드가 짧다(게다가 빠르다).

##### Enforcement

(쉬움) `shared_ptr`가 `make_shared`가 아니라 `new`의 결과에서 생성되면 경고한다.

### <a name="Rr-make_unique"></a>R.23: `unique_ptr`를 만들때는 `make_unique()`를 사용하라

##### Reason

편리하며 `shared_ptr`와 일관성을 가진다.

##### Note

`make_unique()`는 C++14 에 해당하지만, 별 문제 없이 사용할 수 있다. (쉽게 작성할 수 있다)

##### Enforcement

(쉬움) `unique_ptr`가 `make_unique`가 아니라 `new`의 결과로부터 생성된다면 경고하라

### <a name="Rr-weak_ptr"></a>R.24: `shared_ptr`의 순환참조를 부수기 위해 `weak_ptr`를 사용하라

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

### <a name="Rr-smartptrparam"></a>R.30: 수명주기 의미구조를 표현하기 위해서만 스마트 포인터를 매개변수로 사용하라

##### Reason

그저 `widget`그 자체만 필요한 함수에서 `widget`에 대한 스마트 포인터를 받는 것은 잘못된 것이다.
그런 함수는 특정한 종류의 스마트 포인터에 의해서 수명주기가 관리되는 `widget`이 아니라 어떤 `widget`개체라도 받을 수 있어야 한다. 
수명주기에 영향을 주지 않는 함수는 원시 포인터 혹은 참조를 사용해야 한다.

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

* (쉬움) 함수가 복사 가능한 스마트 포인터 타입(`operator->` 혹은 `operator*`를 중복정의한 타입)을 매개변수로 받고, 함수 내에서 `operator*`, `operator->` 혹은 `get()`만 사용하는 경우 경고하라.  
  `T*`혹은 `T&`를 사용하도록 제안하라.
* 스마트 포인터 타입 매개변수를 지적하라. 이때 해당 타입은 복사/이동이 가능하지만 함수 내에서 복사/이동되지 않고, 변경되지 않으며, 다른 함수로 전달되지 않아야 한다. 이는 소유권 의미구조가 사용되지 않는다는 것을 의미한다.  
  `T*`혹은 `T&`를 사용하도록 제안하라.

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

두 경우 모두 [`sharedptrparam` 가이드라인](#Rr-smartptrparam)에 맞지 않는다:  
`p`는 `shared_ptr`이지만, 공유에 대해서는 아무것도 하지 않고 있으며, 값에 의한 전달은 비효율적이다;
이 함수들이 widget의 생명주기에 영향을 미친다면 스마트 포인터를 넘겨받아야만 한다. 
widget이 `nullptr`이 될 수 있다면 `widget*`를 넘겨받아야 하고, 그게 아닌 이상적인 상황은 함수가 `widget&`를 넘겨받아야 한다.

이 스마트 포인터들은 `shared_ptr` 개념에 부합한다. 때문에 이 규칙은 고정관념과는 다르게 흔히 발생할 수 있는 비효율을 노출시킨다.

### <a name="Rr-uniqueptrparam"></a>R.32: 함수가 `widget`의 소유권을 맡는다는 것을 표현하기 위해 `unique_ptr<widget>`를 매개변수로 사용하라

##### Reason

`unique_ptr`를 사용하는 것은 함수를 문서화하면서 호출할 때 소유권 전달을 강제한다.

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

* (단순) 함수가 `unique_ptr<T>` 매개변수를 lvalue 참조로 받아 새로운 개체를 대입하거나 최소 한 경로에서 `reset()`을 호출하지 않으면 경고하라. `T*`혹은 `T&`의 사용을 제안하라
* (단순) ((기본사항)) 함수가 `unique_ptr<T>`의 `const` 참조를 매개변수로 받는다면 경고하라. `const T*` 혹은 `const T&`를 대신 사용하도록 제안하라.

### <a name="Rr-reseat"></a>R.33: 함수가 `widget`을 새로 설정한다는 것을 표현하기 위해 `unique_ptr<widget>&`를 사용하라

##### Reason

이렇게 `unique_ptr`를 사용하는 것은 함수를 문서화하고 함수 호출의 재설정(reseating) 의미구조를 강제한다.

##### Note

재설정(reseat)은 "포인터 혹은 스마트 포인터가 다른 개체를 참조하도록 만드는 것"을 의미한다.

##### Example

```c++
    void reseat(unique_ptr<widget>&); // "will" or "might" reseat pointer
```

##### Example, bad

```c++
    void thinko(const unique_ptr<widget>&); // usually not what you want
```

##### Enforcement

* (단순) 함수가 `unique_ptr<T>` 매개변수를 lvalue 참조로 받아 새로운 개체를 대입하거나 최소 한 경로에서 `reset()`을 호출하지 않으면 경고하라. `T*`혹은 `T&`의 사용을 제안하라
* (단순) ((기본사항)) 함수가 `unique_ptr<T>`의 `const` 참조를 매개변수로 받는다면 경고하라. `const T*` 혹은 `const T&`를 대신 사용하도록 제안하라.

### <a name="Rr-sharedptrparam-owner"></a>R.34: 함수가 소유자 중 하나라는 것을 표현하기 위해 `shared_ptr<widget>`를 매개변수로 사용하라

##### Reason

이는 함수가 소유권을 공유한다는 것을 명시적으로 만든다.

##### Example, good

```c++
    void share(shared_ptr<widget>);            // share -- "will" retain refcount

    void may_share(const shared_ptr<widget>&); // "might" retain refcount

    void reseat(shared_ptr<widget>&);          // "might" reseat ptr
```

##### Enforcement

* (쉬움) 함수가 `shared_ptr<T>`를 lvalue 참조로 받으면서 새로운 개체를 대입하지 않고 최소 한 경로에서 `reset()`을 호출하지 않는다면 경고하라. 대신 `T*` 혹은 `T&`를 사용하도록 제안하라.
* (쉬움) ((기본사항)) 함수가 `shared_ptr<T>`를 값 혹은 `const` 참조로 전달 받으면서 최소 한 경로에서 다른 `shared_ptr`에 복사하거나 이동하지 않으면 경고하라. 대신 `T*` 혹은 `T&`를 사용하도록 제안하라.
* (쉬움) ((기본사항)) `shared_ptr<T>`을 rvalue 참조로 전달받으면 경고하라. 대신 값으로 전달받도록 제안하라.

### <a name="Rr-sharedptrparam"></a>R.35: 함수가 공유 포인터를 재설정한다는 것을 표현하기 위해 `shared_ptr<widget>&`를 매개변수로 사용하라

##### Reason

이는 함수가 값을 변경한다는 것을 명시적으로 드러낸다.

##### Note

재설정(reseat)은 "포인터 혹은 스마트 포인터가 다른 개체를 참조하도록 만드는 것"을 의미한다.

##### Example, good

```c++
    void share(shared_ptr<widget>);            // share -- "will" retain refcount

    void reseat(shared_ptr<widget>&);          // "might" reseat ptr

    void may_share(const shared_ptr<widget>&); // "might" retain refcount
```

##### Enforcement

* (쉬움) 함수가 `shared_ptr<T>`를 lvalue 참조로 받으면서 새로운 개체를 대입하지 않고 최소 한 경로에서 `reset()`을 호출하지 않는다면 경고하라. 대신 `T*` 혹은 `T&`를 사용하도록 제안하라.
* (쉬움) ((기본사항)) 함수가 `shared_ptr<T>`를 값 혹은 `const` 참조로 전달 받으면서 최소 한 경로에서 다른 `shared_ptr`에 복사하거나 이동하지 않으면 경고하라. 대신 `T*` 혹은 `T&`를 사용하도록 제안하라.
* (쉬움) ((기본사항)) `shared_ptr<T>`을 rvalue 참조로 전달받으면 경고하라. 대신 값으로 전달받도록 제안하라.

### <a name="Rr-sharedptrparam-const"></a>R.36: 함수가 개체에 대한 참조 카운트를 유지한다는 것을 표현하기 위해 `const shared_ptr<widget>&`을 매개변수로 사용하라 ???

##### Reason

작성한 함수의 ???를 명시적으로 만든다.

##### Example, good

```c++
    void share(shared_ptr<widget>);            // share -- "will" retain refcount

    void reseat(shared_ptr<widget>&);          // "might" reseat ptr

    void may_share(const shared_ptr<widget>&); // "might" retain refcount
```

##### Enforcement

* (쉬움) 함수가 `shared_ptr<T>`를 lvalue 참조로 받으면서 새로운 개체를 대입하지 않고 최소 한 경로에서 `reset()`을 호출하지 않는다면 경고하라. 대신 `T*` 혹은 `T&`를 사용하도록 제안하라.
* (쉬움) ((기본사항)) 함수가 `shared_ptr<T>`를 값 혹은 `const` 참조로 전달 받으면서 최소 한 경로에서 다른 `shared_ptr`에 복사하거나 이동하지 않으면 경고하라. 대신 `T*` 혹은 `T&`를 사용하도록 제안하라.
* (쉬움) ((기본사항)) `shared_ptr<T>`을 rvalue 참조로 전달받으면 경고하라. 대신 값으로 전달받도록 제안하라.

### <a name="Rr-smartptrget"></a>R.37: 재명명(aliased)된 스마트 포인터에서 획득한 포인터 혹은 참조를 전달하지 마라

> 역주: [Pointer Aliasing](https://en.wikipedia.org/wiki/Pointer_aliasing)

##### Reason

이 규칙을 위반하는 것은 참조 수를 잃어버리고 허상 포인터가 남도록 만드는 가장 중요한 원인이다.  
함수는 호출이 깊어질 때 되도록 원시 포인터나 참조를 전달해야 한다. 스마트 포인터로부터 원시 포인터 혹은 참조를 획득하는 호출 트리의 최상단에서는 개체가 소멸하지 않도록 해야 한다.  
프로그래머는 소유권을 가진 스마트 포인터가 우연치 않게 호출 트리의 하단에서 바뀌지 않도록 해야한다.

##### Note

이를 위해서, 스마트 포인터의 지역 사본을 만들어야 할수도 있다. 이 스마트 포인터는 함수와 그 호출 트리가 지속되는 동안 개체가 살아있도록 만든다.

##### Example

다음과 같은 경우를 생각해보라:

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

다음과 같은 코드가 허용되어선 안된다:

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

이는 쉽게 수정할 수 있다 -- "참조 카운트를 유지하도록" 해당 포인터의 사본을 지역적으로 만드는 것이다:

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

* (쉬움) 비지역 스마트 포인터 변수(`unique_ptr` 혹은 `shared_ptr`)로부터 포인터가 참조를 획득하면 경고하라. 혹은 스마트 포인터가 다른 개체에 연결(aliased)되었을 수 있을때 함수 호출에 사용되면 경고하라. `shared_ptr`라면 해당 포인터를 통해 참조하거나 그 포인터의 지역 사본을 만들도록 제안하라.
