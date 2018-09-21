자원 관리
> # R: Resource management

이 장은 자원과 관련된 규칙을 포함하고 있다. 자원은 메모리나 파일 핸들, 소켓, 락과 같은 것을 말하며 반드시 획득되야 하고
명시적 또는 암묵적으로 해체되어야 한다. 해체되어야 하는 일반적인 이유는 한정 잉여 자원으로 인해 생기는 자원 부족인데 지연 해체조차도 이런 문제를 야기할 수 있다.
어떤 자원도 새지 않으며 필요한 기간 보다 길게 자원을 소유하고 있지 않는 것이 근본적으로 자원을 관리하는 목표다.
자원을 해체하는 책임을 가지는 주체를 우리는 오너(owner)라고 한다.

>This section contains rules related to resources. A resource is anything that must be acquired and (explicitly or implicitly) released, such as memory, file handles, sockets, and locks. The reason it must be released is typically that it can be in short supply, so even delayed release may do harm. The fundamental aim is to ensure that we don't leak any resources and that we don't hold a resource longer than we need to. An entity that is responsible for releasing a resource is called an owner.

자원 누출이 용인되고 심지어 최선인 경우가 몇몇 있긴 하다.
입력을 기반으로 단순히 출력하는 프로그램을 구현하고 입력에 비례하여 필요한 메모리 양이 증가한다면,  
(성능과 프로그래밍 용이성을 위해) 어떤 자원도 삭제하지 않는 것이 때로는 최선의 전략일 수 있다.
가장 큰 입력을 처리하기 위해서 충분한 메모리를 가졌다면 자원이 소비되도록 내버려 둬라. 다만 뭔가 잘못을 했다면 상황에 맞는
에러 메시지를 주도록 해라. 이제부터 이런 경우는 더 이상 언급하지 않겠다.

>There are a few cases where leaks can be acceptable or even optimal:
if you are writing a program that simply produces an output based on an input and the amount of memory needed is proportional to the size of the input,
the optimal strategy (for performance and ease of programming) is sometimes simply never to delete anything.
If you have enough memory to handle your largest input, leak away, but be sure to give a good error message if you are wrong.
Here, we ignore such cases.

자원 관리 규칙 요약
>Resource management rule summary:

* R.1: 자원 핸들과 RAII(자원 획득시 초기화)를 사용해서 자동으로 리소스를 관리해라.(#Rr-raii)  
* R.2: 인터페이스에서 각각 오브젝트를 나타낼때만 원시 포인터를 사용해라.(#Rr-use-ptr)
* R.3: 원시 포인터(T*)는 비 소유다.(#Rr-ptr)
* R.4: 원시 참조(T&)는 비소유이다.(#Rr-ref)
* R.5: 불필요하게 힙 할당을 하지마라.(#Rr-scoped)
* R.6: const가 아닌 전역 변수의 사용을 피해라.(#Rr-global)

>* [R.1: Manage resources automatically using resource handles and RAII (resource acquisition is initialization)](#Rr-raii)
* [R.2: In interfaces, use raw pointers to denote individual objects (only)](#Rr-use-ptr)
* [R.3: A raw pointer (a `T*`) is non-owning](#Rr-ptr)
* [R.4: A raw reference (a `T&`) is non-owning](#Rr-ref)
* [R.5: Prefer scoped objects](#Rr-scoped)
* [R.6: Avoid non-`const` global variables](#Rr-global)

>Alocation and deallocation rule summary:

* R.10: malloc()과 free()의 사용을 피해라.
* R.11: 명시적으로 new와 delete의 사용을 피해라.
* R.12: 명시적 리소스 할당 결과를 매니저 오브젝트에서 즉시 줘라.
* R.13: 하나의 표현식에서 최대 하나의 자원 할당을 수행해라.
* R.14: ??? 배열 vs 포인터 인자
* R.15: 항상 할당 해제 짝을 오버르드해서 사용해라.

>* [R.10: Avoid `malloc()` and `free()`](#Rr-mallocfree)
* [R.11: Avoid calling `new` and `delete` explicitly](#Rr-newdelete)
* [R.12: Immediately give the result of an explicit resource allocation to a manager object](#Rr-immediate-alloc)
* [R.13: Perform at most one explicit resource allocation in a single expression statement](#Rr-single-alloc)
* [R.14: ??? array vs. pointer parameter](#Rr-ap)
* [R.15: Always overload matched allocation/deallocation pairs](#Rr-pair)

<a name ="Rr-summary-smartptrs"></a>Smart pointer rule summary:

* [R.20: 소유권을 나타내기 위해 `unique_ptr`이나 `shared_ptr`을 사용하라.](#Rr-owner)
* [R.21: 소유권 공유가 필요없다면 `shared_ptr`보다 `unique_ptr`이 낫다.](#Rr-unique)
* [R.22: `shared_ptr`을 만드려면 `make_shared()`를 사용하라.](#Rr-make_shared)
* [R.23: `unique_ptr`을 만드려면 `make_unique()`를 사용하라.](#Rr-make_unique)
* [R.24: `shared_ptr`의 순환을 끊으려면 `std::weak_ptr`을 사용하라.](#Rr-weak_ptr)

> * [R.20: Use `unique_ptr` or `shared_ptr` to represent ownership](#Rr-owner)
* [R.21: Prefer `unique_ptr` over `shared_ptr` unless you need to share ownership](#Rr-unique)
* [R.22: Use `make_shared()` to make `shared_ptr`s](#Rr-make_shared)
* [R.23: Use `make_unique()` to make `unique_ptr`s](#Rr-make_unique)
* [R.24: Use `std::weak_ptr` to break cycles of `shared_ptr`s](#Rr-weak_ptr)

* [R.31: 만약 non-`std` 스마트 포인터를 사용 중이라면, `std`의 기본 패턴을 따라라.](#Rr-smart)
>* [R.30: Take smart pointers as parameters only to explicitly express lifetime semantics](#Rr-smartptrparam)
* [R.31: If you have non-`std` smart pointers, follow the basic pattern from `std`](#Rr-smart)
* [R.32: Take a `unique_ptr<widget>` parameter to express that a function assumes ownership of a `widget`](#Rr-uniqueptrparam)
* [R.33: Take a `unique_ptr<widget>&` parameter to express that a function reseats the`widget`](#Rr-reseat)
* [R.34: Take a `shared_ptr<widget>` parameter to express that a function is part owner](#Rr-sharedptrparam-owner)
* [R.35: Take a `shared_ptr<widget>&` parameter to express that a function might reseat the shared pointer](#Rr-sharedptrparam)
* [R.36: Take a `const shared_ptr<widget>&` parameter to express that it might retain a reference count to the object ???](#Rr-sharedptrparam-const&)
* [R.37: Do not pass a pointer or reference obtained from an aliased smart pointer](#Rr-smartptrget)


<a name ="Rr-raii"></a>

### Rule R.1 : 자원 핸들과 RAII(자원 획득시 초기화)를 사용해서 자동으로 리소스를 관리해라.
**이유** : 수동 자원 관리의 복잡성과 누출을 피하기 위한 방법을 알아본다. C++ 언어적 강제인 생성자 소멸자 대칭은 `fopen`/`fclose`,  그리고 `lock`/`unlock`, `new`/`delete`과 같은 자원 획득/해체 함수의 짝과 같은 구조를 가진다.
이 특징을 사용해서 자원의 획득/해체시 짝 함수 호출이 필요한 자원을 다룰 때 생성자에서 자원을 획득하고 소멸자에서 해체하는 방법으로 강제되도록 오브젝트에 리소스를 캡슐화해라.

**나쁜 예**

	void send( X* x, cstring_view destination ) {
        auto port = OpenPort(destination);
        my_mutex.lock();
        // ...
        Send(port, x);
        // ...
        my_mutex.unlock();
        ClosePort(port);
        delete x;
	}

이 코드에서 모든 경로에서 unlock 그리고 ClosePort, delete 호출을 해야 하고 한 번만 해야 한다. 그리고
`...`로 표시된 코드에서 예외가 던져진다면 `x`는 누출되고 `my_mutex`는 락을 해체하지 못한다.

**좋은 예**

	void send( unique_ptr<X> x, cstring_view destination ) { // x가 X를 소유했다.
        Port port{destination};            // port가 Port 핸들을 소유했다.
        lock_guard<mutex> guard{my_mutex}; // guard가 락을 소유했다.
        // ...
        Send(port, x);
        // ...
	} // 자동으로 my_mutex 그리고 x, port의 자원이 해체될 것이다.

모든 자원 정리가 자동화되었고 예외와 상관없이 모든 경로에서 한번 수행된다. 참고로 함수가 포인터 소유권을 가져간 것도 보여주고 있다.

`Port`는 어떻게 구현할 수 있을까? 자원을 캡슐화하는 간단한 래퍼로 구현할 수 있다.

    class Port {
        PortHandle port;
    public:
        Port( cstring_view destination ) : port{OpenPort(destination)} { }
        ~Port() { ClosePort(port); }
        operator PortHandle() { return port; }

	 // port 핸들은 보통 복제될 수 없다, 그래서 필요에 따라 복사, 대입 연산자를 disable시키자.
        Port(const Port&) =delete;
        Port& operator=(const Port&) =delete;
    };

**Note**: 소멸자를 가진 클래스로 표현되지 않고 다루기 힘든 자원인 경우 클래스로 감싸서 자원을 관리하거나 [`finally`](#S-GSL)를 사용해라.

**참고**: [RAII](Rr-raii).

>### Rule R.1: Manage resources automatically using resource handles and RAII (resource acquisition is initialization)

>**Reason**: To avoid leaks and the complexity of manual resource management.
 C++'s language-enforced constructor/destructor symmetry mirrors the symmetry inherent in resource acquire/release function pairs such as `fopen`/`fclose`, `lock`/`unlock`, and `new`/`delete`.
 Whenever you deal with a resource that needs paired acquire/release function calls,
 encapsulate that resource in an object that enforces pairing for you -- acquire the resource in its constructor, and release it in its destructor.

>**Example, bad**: Consider

>	void send( X* x, cstring_view destination ) {
        auto port = OpenPort(destination);
        my_mutex.lock();
        // ...
        Send(port, x);
        // ...
        my_mutex.unlock();
        ClosePort(port);
        delete x;
	}

>In this code, you have to remember to `unlock`, `ClosePort`, and `delete` on all paths, and do each exactly once.
Further, if any of the code marked `...` throws an exception, then `x` is leaked and `my_mutex` remains locked.

>**Example**: Consider

>	void send( unique_ptr<X> x, cstring_view destination ) { // x owns the X
        Port port{destination};            // port owns the PortHandle
        lock_guard<mutex> guard{my_mutex}; // guard owns the lock
        // ...
        Send(port, x);
        // ...
	} // automatically unlocks my_mutex and deletes the pointer in x

>Now all resource cleanup is automatic, performed once on all paths whether or not there is an exception. As a bonus, the function now advertises that it takes over ownership of the pointer.

>What is `Port`? A handy wrapper that encapsulates the resource:

>    class Port {
        PortHandle port;
    public:
        Port( cstring_view destination ) : port{OpenPort(destination)} { }
        ~Port() { ClosePort(port); }
        operator PortHandle() { return port; }

>        // port handles can't usually be cloned, so disable copying and assignment if necessary
        Port(const Port&) =delete;
        Port& operator=(const Port&) =delete;
    };

>**Note**: Where a resource is "ill-behaved" in that it isn't represented as a class with a destructor, wrap it in a class or use [`finally`](#S-GSL)

>**See also**: [RAII](Rr-raii).

<a name ="Rr-use-ptr"></a>
### R.2: 인터페이스에서 각각 오브젝트를 나타낼때만 원시 포인터를 사용해라.
>### R.2: In interfaces, use raw pointers to denote individual objects (only)

**이유**: 배열은 컨테이너 타입(예, 백터(소유하는)이나 span(비 소유하는))으로 가장 잘 표현된다. 이런 컨테이너와 뷰는 범위 확인을 위한 충분한 정보를 가지고 있다.
>**Reason**: Arrays are best represented by a container type (e.g., `vector` (owning)) or an `span` (non-owning).
Such containers and views hold sufficient information to do range checking.

** 나쁜 예**
>**Example, bad**:

	void f(int* p, int n)	// n은 p[]의 배열 길이다.
	{
		// ...
		p[2] = 7;	//bad: subscript raw pointer
		// ...
	}
> void f(int* p, int n)	// n is the number of elements in p[]
	{
		// ...
		p[2] = 7;	// bad: subscript raw pointer
		// ...
	}

컴파일러 주석을 이해하지 못하고 다른 코드를 읽지 않고는 'p'가 정말로 'n' 만큼을 가르키는지 알순 없다. 대신 span을 사용해라.
>The compiler does not read comments, and without reading other code you do not know whether `p` really points to `n` elements.
Use an `span` instead.

**예**:

  void g(int* p, int fmt)	// 포맷 #fmt를 사용해서 *p를 출력해라.
	{
		// ... *p과 p[0]만을 사용해라  ...
	}

>**Example**:

>	void g(int* p, int fmt)	// print *p using format #fmt
	{
		// ... uses *p and p[0] only ...
	}

**예외**: C 스타일 문자열은 null-termniated 문자열을 가르키는 포인터로 전달한다. 설명한 규칙을 따르기 위해 'char*'보다 zstring을 사용해라.
>**Exception**: C-style strings are passed as single pointers to a zero-terminated sequence of characters.
Use `zstring` rather than `char*` to indicate that you rely on that convention.

**주목**: 단독 엘리먼트를 가르키기 위해 참조자를 사용할 수 있다. 그러나 'nullptr'이 가능한 경우라면 참조자는 좋은 대안은 아니다.
>**Note**: Many current uses of pointers to a single element could be references.
However, where `nullptr` is a possible value, a reference may not be an reasonable alternative.

**시행하기**
>**Enforcement**:

* 컨테이너 또는 뷰, 반복자(iterator)가 아닌 포인터에서 주소 계산(++ 포함)을 삼가해라
이 규칙은 오래된 코드 베이스에 적용된다면 많은 양의 false positive를 만들 수 있다.
* 간단한 포인터로 전달하는 배열 이름을 삼가해라??
>* Flag pointer arithmetic (including `++`) on a pointer that is not part of a container, view, or iterator.
This rule would generate a huge number of false positives if applied to an older code base.
>* Flag array names passed as simple pointers



<a name="Rr-ptr"></a>
### R.3: 원시 포인터(T*)는 비 소유다.
>### R.3: A raw pointer (a `T*`) is non-owning

**이유**:
원시 포인터는 소유 가능하지 않다 (c++ 표준뿐만 아니라 대부분의 경우)
오브젝트를 가르키는 포인터를 신뢰성있고 효율적으로 삭제하기 위해서 식별되고 소유 가능한 포인터가 있어야한다.
>**Reason**: There is nothing (in the C++ standard or in most code) to say otherwise and most raw pointers are non-owning.
We want owning pointers identified so that we can reliably and efficiently delete the objects pointed to by owning pointers.

**예**
>**Example**:

  void f()
	{
		int* p1 = new int{7};			// 나쁜 예: 원시 소유 포인터
		auto p2 = make_unique<int>(7);	// 좋은 예: unique pointer는 int 타입를 소유한다.
		// ...
	}
>	void f()
	{
		int* p1 = new int{7};			// bad: raw owning pointer
		auto p2 = make_unique<int>(7);	// OK: the int is owned by a unique pointer
		// ...
	}

unique_ptr는 오브젝트의 삭제를 보장해줌으로써 메모리 누수를 차단해준다. (예외 발생에서도 마찬가지다.) 그러나 T*는 그렇지 못하다.
>The `unique_ptr` protects against leaks by guaranteeing the deletion of its object (even in the presence of exceptions). The `T*` does not.

**예**
>**Example**:

template<typename T>
	class X {
		// ...
	public:
		T* p;	// 나쁜 예: p가 소유할 수 있는 타입인지 알 수 없다.
		T* q;	// 나쁜 예: q가 소유할 수 있는 타입인지 알 수 없다.
	};
>	template<typename T>
	class X {
		// ...
	public:
		T* p;	// bad: it is unclear whether p is owning or not
		T* q;	// bad: it is unclear whether q is owning or not
	};

명시적인 소유권을 만듦으로써 문제를 해결할 수 있다.
>We can fix that problem by making ownership explicit:

  template<typename T>
	class X2 {
		// ...
	public:
		owner<T> p;	// OK: p는 소유한다.
		T* q;		// OK: q는 소유하지 않는다.
	};
>	template<typename T>
	class X2 {
		// ...
	public:
		owner<T> p;	// OK: p is owning
		T* q;		// OK: q is not owning
	};

**예외**:
주요 예외사항은 C와 호환성을 가져야하거나 C 또는 C 스타일의 C++으로 된 인터페이스가 ABI 호환성을 가져야 하는 레거시 코드들이 있다. 즉, 'T*'를 소유하는 방식을 위반하는 수 많은 코드가 존재한다는 사실을 무시할 순 없다.
20년 묵은 레거시 코드를 최신 C++ 코드로 변환할 수 있는 툴이 있으면 좋을것이다. 이런 차원에서 툴 개발과 툴의 사용을 독려할것이고 또한 이 가이드라인이 도움이 되었으면 좋겠다. 실제 개발 또는 관련된 리서치를 진행해왔는데, 가시적인 성과가 보일때까지 시간은 더 걸릴것이다. : 최신 코드로 바꿀수 있게 되기전에 레거시 코드가 더 빠르게 생성될지도 모른다.


>**Exceptions**:
A major class of exception is legacy code, especially code that must remain compilable as C or interface with C and C-style C++ through ABIs. The fact that there are billions of lines of code that violate this rule against owning T*s cannot be ignored. We'd love to see program transformation tools turning 20-year-old "legacy" code into shiny modern code, we encourage the development, deployment and use of such tools, we hope the guidelines will help the development of such tools, and we even contributed (and contribute) to the research and development in this area. However, it will take time: "legacy code" is generated faster than we can renovate old code, and so it will be for a few years.

이 모든 코드는 다시 작성 될순 없고 적어도 당장은 아닐것이다.

>This code cannot all be rewritten (ever assuming good code transformation software), especially not soon. This problem cannot be solved (at scale) by transforming all owning pointers to unique_ptrs and shared_ptrs, partly because we need/use owning "raw pointers" as well as simple pointers in the implementation of our fundamental resource handles. For example, common vector implementations have one owning pointer and two non-owning pointers. Many ABIs (and essentially all interfaces to C code) use T*s, some of them owning. Some interfaces cannot be simply annotated with owner because they need to remain compilable as C (although this would be a rare good use for a macro, that expands to owner in C++ mode only).

**Note**: `owner<T>` has no default semantics beyond `T*` it can be used without changing any code using it and without affecting ABIs.
It is simply a (most valuable) indicator to programmers and analysis tools.
For example, if an `owner<T>` is a member of a class, that class better have a destructor that `delete`s it.

원시 포인터를 반환하는것은 호출자에게 수명 관리에 불확실성을 심어준다. 다시 말해 누가 포인터를 지워야 할까?
>**Example**, bad:
Returning a (raw) pointer imposes a life-time management uncertainty on the caller; that is, who deletes the pointed-to object?

	Gadget* make_gadget(int n)
	{
		auto p = new Gadget{n};
		// ...
		return p;
  }

	void caller(int n)
	{
		auto p = make_gadget(n);	// remember to delete p
		// ...
		delete p;
	}

메모리 누출의 고통뿐만아니라 이것은 쓸데없이 많은 spurious 할당과 해제를 야기할 수 있다. 만약
Gadget의 비용이 함수에서 반환되도 문제없을 정도 싸다면( 사이즈가 작거나 효율적인 move 연산을 ) 그냥 값 복사를 사용해도 된다.
>In addition to suffering from the problem from leak, this adds a spurious allocation and deallocation operation, and is needlessly verbose. If Gadget is cheap to move out of a function (i.e., is small or has an efficient move operation), just return it "by value" (see "out" return values):

	Gadget make_gadget(int n)
	{
		Gadget g{n};
		// ...
		return g;
	}

**주목** : 이 규칙은 펙토리 함수에 적용될 수 있다.
>**Note**: This rule applies to factory functions.

**주목** : ??
**Note**: If pointer semantics is required (e.g., because the return type needs to refer to a base class of a class hierarchy (an interface)),
return a "smart pointer."

**Enforcement**:

* (Simple) Warn on `delete` of a raw pointer that is not an `owner<T>`.
* (Moderate) Warn on failure to either `reset` or explicitly `delete` an `owner<T>` pointer on every code path.
* (Simple) Warn if the return value of `new` or a function call with return value of pointer type is assigned to a raw pointer.
* (Simple) Warn if a function returns an object that was allocated within the function but has a move constructor.
Suggest considering returning it by value instead.

### R.4: 원시 참조(T&)는 비소유이다.
>### R.4: A raw reference (a `T&`) is non-owning

원시 참조는 소유 가능하지 않다 (c++ 표준뿐아니라 대부분의 경우)
소유된 포인터를 가르키는 오브젝트를 신뢰성있고 효율적으로 삭제하기 위해서 식별되는 소유자가 있어야한다.
>**Reason**: There is nothing (in the C++ standard or in most code) to say otherwise and most raw references are non-owning. We want owners identified so that we can reliably and efficiently delete the objects pointed to by owning pointers.

**Example**:
	void f()
	{
		int& r = *new int{7};			// bad: raw owning reference
		// ...
		delete &r;						// bad: violated the rule against deleting raw pointers
	}

**See also**: [The raw pointer rule](#Rr-ptr)

**Enforcement**: See [the raw pointer rule](#Rr-ptr)

###R.5: 불필요하게 힙 할당을 하지마라.
>###R.5: Don't heap-allocate unnecessarily

범위내 오브젝트(scoped object)는 지역 오브젝트 또는 전역 오브젝트, 멤버가 있다.
포함하는 범위 또는 오브젝트에 이미 쓰인 할당과 해체를 초과하는 비용은 없다.(??)
범위내 오브젝트의 멤버는 스스로 범위내(scoped)이며 오브젝트의 생성자와 소멸자는 맴버의 생명주기를 관리한다.
>**Reason**: A scoped object is a local object, a global object, or a member.
This implies that there is no separate allocation and deallocation cost in excess that already used for the containing scope or object.
The members of a scoped object are themselves scoped and the scoped object's constructor and destructor manage the members' lifetimes.

다음 예는 불핑요한 할당과 해재를 하기 때문에 비 효율적이고 예외에 취약하며 ... 부분은 메모리 누출을 야기할 수 있다. 그리고 장황하다.
>**Example**: the following example is inefficient (because it has unnecessary allocation and deallocation), vulnerable to exception throws and returns in the "¦ part (leading to leaks), and verbose:

	void some_function(int n)
	{
		auto p = new Gadget{n};
		// ...
		delete p;
	}

대신에 지역 변수를 사용해라.
>Instead, use a local variable:

	void some_function(int n)
	{
		Gadget g{n};
		// ...
	}

**Enforcement**:

* (Moderate) Warn if an object is allocated and then deallocated on all paths within a function. Suggest it should be a local `auto` stack object instead.
* (Simple) Warn if a local `Unique_ptr` or `Shared_ptr` is not moved, copied, reassigned or `reset` before its lifetime ends.

### R.6: const가 아닌 전역 변수의 사용을 피해라.
>### R.6: Avoid non-`const` global variables

**이유**: 전역 변수는 모든 곳에서 접근될 수 있고 명백히 관련 없는 오브젝트사이에 얘기치 않는 의존성을 만들수 있다. 이런것들은 에러의 원천으로 악명높다.
>**Reason**: Global variables can be accessed from everywhere so they can introduce surprising dependencies between apparently unrelated objects.
They are a notable source of errors.

**경고**: 전역 오브젝트의 전적으로 초기화 순서를 보장하지 않는다. 전역 오브젝트를 사용하고 싶다면 상수로 초기화 해라.
>**Warning**: The initialization of global objects is not totally ordered. If you use a global object initialize it with a constant.

**예외**: 정역 오브젝트가 때로는 싱글톤보단 낫기도 하다.
>**Exception**: a global object is often better than a singleton.

**예외**: 전역 오브젝트 사용을 금지하는것을 피하려면 수정 불가한 상수 전역으로 사용해라???
**Exception**: An immutable (`const`) global does not introduce the problems we try to avoid by banning global objects.

**Enforcement**: [[??? NM: Obviously we can warn about non-const statics....do we want to?]]

## R.alloc: 할당과 해제
>## R.alloc: Alocation and deallocation

### R.10: malloc()과 free()의 사용을 피해라.
>### R.10: Avoid `malloc()` and `free()`

**이유**: malloc()과 free()는 생성자와 소멸자를 지원하지 않는다. new 과 delete와 섞어서 사용하지 마라.
>**Reason**: `malloc()` and `free()` do not support construction and destruction, and do not mix well with `new` and `delete`.


**Example**:
		class Record {
			int id;
			string name;
			// ...
		};
		void use()
		{
			Record* p1 = static_cast<Record*>(malloc(sizeof(Record)));
			// p1 may be nullptr
			// *p1 is not initialized; in particular, that string isn't a string, but a string-sizes bag of bits

			auto p2 = new Record;

			// unless an exception is thrown, *p2 is default initialized
			auto p3 = new(nothrow) Record;
			// p3 may be nullptr; if not, *p2 is default initialized

			// ...

			delete p1;	// error: cannot delete object allocated by malloc()
			free(p2);	// error: cannot free() object allocatedby new
		}

In some implementaions that `delete` and that `free()` might work, or maybe they will cause run-time errors.

**Exception**: There are applications and sections of code where exceptions are not acceptable.
Some of the best such example are in life-critical hard real-time code.
Beware that many bans on exception use are based on superstition (bad)
or by concerns for older code bases with unsystematics resource management (unfortunately, but sometimes necessary).
In such cases, consider the `nothrow` versions of `new`.

**Enforcement**: Flag explicit use of `malloc` and `free`.

### R.11: 명시적으로 new와 delete의 사용을 피해라.
>### R.11: Avoid calling `new` and `delete` explicitly

**이유**: new로 반환된 포인터를 리소스 핸들(delete를 부를수 있는)에 종속되어야 한다.
new로 반환된 포인터가 원시 포이터에 할당되면 누수가 발생할 수 있다.
**Reason**: The pointer returned by `new` should belong to a resource handle (that can call `delete`).
If the pointer returned from `new` is assigned to a plain/naked pointer, the object can be leaked.

**Note**: In a large program, a naked `delete` (that is a `delete` in application code, rather than part of code devoted to resource management)
is a likely bug: if you have N `delete`s, how can you be certain that you don't need N+1 or N-1?
The bug may be latent: it may emerge only during maintenace.
If you have a naled `new`, you probably need a naked `delete` somewhere, so yu probably have a bug.

**Enforcement**: (Simple) Warn on any explicit use of `new` and `delete`. Suggest using `make_unique` instead.

### R.12: 명시적 리소스 할당 결과를 매니저 오브젝트에서 즉시 줘라.
>### R.12: Immediately give the result of an explicit resource allocation to a manager object

**Reason**: 그렇지 않으면, 예외나 반환이 자원 누수를 야기할 수 있다.
>**Reason**: If you don't, an exception or a return may lead to a leak.

**Example, bad**:
	void f(const string& name)
	{
		FILE* f = fopen(name,"r");			// open the file
		vector<char> buf(1024);
		auto _ = finally([] { fclose(f); }	// remember to close the file
		// ...
	}

The allocation of `buf` may fail and leak the file handle.

**Example**:

	void f(const string& name)
	{
		ifstream {name,"r"};			// open the file
		vector<char> buf(1024);
		// ...
	}

The use of the file handle (in `ifstream`) is simple, efficient, and safe.

**Enforcement**:

* Flag explicit allocations used to initialize pointers  (problem: how many direct resouce allocations can we recognize?)

### R.13: 하나의 표현식에서 최대 하나의 자원 할당을 수행해라.
### R.13: Perform at most one explicit resource allocation in a single expression statement

**Reason**: If you perform two explicit resource allocations in one statement,
you could leak resources because the order of evaluation of many subexpressions, including function arguments, is unspecified.

**Example**:

    void fun( shared_ptr<Widget> sp1, shared_ptr<Widget> sp2 );

This `fun` can be called like this:

    fun( shared_ptr<Widget>(new Widget(a,b)), shared_ptr<Widget>(new Widget(c,d)) );	// BAD: potential leak

This is exception-unsafe because the compiler may reorder the two expressions building the function's two arguments.
In particular, the compiler can interleave execution of the two expressions:
Memory allocation (by calling `operator new`) could be done first for both objects, followed by attempts to call the two `Widget` constructors.
If one of the constructor calls throws an exception, then the other object's memory will never be released!

This subtle problem has a simple solution: Never perform more than one explicit resource allocation in a single expression statement.
For example:

	    shared_ptr<Widget> sp1(new Widget(a,b)); // Better, but messy
	    fun( sp1, new Widget(c,d) );

The best solution is to avoid explicit allocation entirely use factory functions that return owning objects:

	    fun( make_shared<Widget>(a,b), make_shared<Widget>(c,d) ); // Best

Write your own factory wrapper if there is not one already.

**Enforcement**:

* Flag expressions with multiple explicit resource allocations (problem: how many direct resouce allocations can we recognize?)


<a name ="Rr-ap"></a>
### R.14: ??? 배열 vs 포인터 인자
>### R.14: ??? array vs. pointer parameter

**Reason**: An array decays to a pointer, thereby loosing its size, opening the opportunity for range errors.

**Example**:

	??? what do we recommend: f(int*[]) or f(int**) ???

**Alternative**: Use `array_view` to preserve size information.

**Enforcement**: Flag `[]` parameters.

### R.15: 항상 할당 해제 짝을 오버르드해서 사용해라.
>### R.15: Always overload matched allocation/deallocation pairs

**Reason**. Otherwise you get mismarched opertions and chaos.

**Example**:

		class X {
			// ...
			void* operator new(size_t s);
			void operator delete(void*);
			// ...
		};

**Note**: If you want memory that cannot be deallocated, `=delete` the deallocation operation.
Don't leave it undeclared.

**Enforcement**: Flag incomplate pairs.


<a name="SS-smart"></a>
## R.smart: Smart pointers


<a name="Rr-owner"></a>
### R.20: 소유권을 나타내기 위해 unique_ptr이나 shared_ptr을 사용하라.
>### Rule R.20: Use `unique_ptr` or `shared_ptr` to represent ownership

**이유**: 자원 누수를 막을 수 있다.
>**Reason**: They can prevent resource leaks.

**Example**: Consider

	void f()
	{
		X x;
		X* p1 { new X };			// see also ???
		unique_ptr<T> p2 { new X };	// unique ownership; see also ???
		shared_ptr<T> p3 { new X };	// shared ownership; see also ???
	}

이 코드는 초기화하는데만 사용된 p1 객체가 누수될 것이다.
>This will leak the object used to initialize `p1` (only).

**시행하기** new로 리턴되는 값 혹은 날 포인터로 할당된 포인터 타입을 리턴하는 함수 호출이 있다면 경고하라.
>**Enforcement:** (Simple) Warn if the return value of `new` or a function call with return value of pointer type is assigned to a raw pointer.


<a name="Rr-unique"></a>
### R.21: 소유권 공유가 필요없다면 `shared_ptr`보다 `unique_ptr`이 낫다.
>### Rule R.21: Prefer `unique_ptr` over `shared_ptr` unless you need to share ownership

**이유**: `unique_ptr`은 개념적으로 단순하고 예측가능하며(파괴가 일어날 때를 알고) 빠르다(사용 횟수를 암시적으로 관리하지 않는다).
>**Reason**: a `unique_ptr` is conceptually simpler and more predictable (you know when destruction happens) and faster (you don't implicitly maintain a use count).

**안 좋은 예**: 이 코드는 불필요하게 참조 횟수를 증가 및 유지하고 있다.

    void f()
    {
        shared_ptr<Base> base = make_shared<Derived>();
        // base를 복사하지 않은 채 지역적으로 사용했다 -- 참조 횟수가 절대 1을 초과하지 않는다.
    } // base 파괴

>**Example, bad**: This needlessly adds and maintains a reference count

    void f()
    {
         shared_ptr<Base> base = make_shared<Derived>();
         // use base locally, without copying it -- refcount never exceeds 1
    } // destroy base

**예**: 이 코드가 더 효율적이다.

    void f()
    {
        unique_ptr<Base> base = make_unique<Derived>();
        // base를 지역적으로 사용
    } // base 파괴


>**Example**: This is more efficient

    void f()
    {
        unique_ptr<Base> base = make_unique<Derived>();
        // use base locall
    } // destroy base


**시행하기**: 만약 함수 내에서 객체 할당에 `Shared_ptr`을 사용하지만, `Shared_ptr`을 리턴하지 않거나 `Shared_ptr&`를 필요로 하는 함수에 전달하고 있다면 경고하라. 대신 `unique_ptr` 사용을 권하라.
>**Enforcement**: (Simple) Warn if a function uses a `Shared_ptr` with an object allocated within the function, but never returns the `Shared_ptr` or passes it to a function requiring a `Shared_ptr&`. Suggest using `unique_ptr` instead.


<a name ="Rr-make_shared"></a>
### R.22: `shared_ptr`을 만드려면 `make_shared()`를 사용하라.
>### R.22: Use `make_shared()` to make `shared_ptr`s

**이유**: 만약 당신이 `make_shared()`를 사용하는 대신 객체를 처음 만들고나서 그 객체를 `shared_ptr` 생성자에게 건네고 있다면, 당신은 (아마도) 할당을(이 후 해제도) 한 번 더 하게 되는 것이다. 왜냐하면 참조 횟수는 객체와 별개로 할당되어지기 때문이다.
>**Reason**: If you first make an object and then gives it to a `shared_ptr` constructor, you (most likely) do one more allocation (and later deallocation) than if you use `make_shared()` because the reference counts must be allocated separately from the object.

**예**: 다음을 봐라
	shared_ptr<X> p1 { new X{2} }; // 나쁜 예
	auto p = make_shared<X>(2);    // 좋은 예

>**Example**: Consider

	shared_ptr<X> p1 { new X{2} }; // bad
	auto p = make_shared<X>(2);    // good

`make_shared()` 버전은 `X`가 단 한 번만 나오는데, 그래서 명시적으로 `new`를 사용하는 버전보다 코드가 대개 짧다(게다가 빠르고).
>The `make_shared()` version mentions `X` only once, so it is usually shorter (as well as faster) than the version with the explicit `new`.

**시행하기**: `shared_ptr`이 `make_shared` 대신 `new`로 생성되어졌다면 경고하라.
>**Enforcement**: (Simple) Warn if a `shared_ptr` is constructed from the result of `new` rather than `make_shared`.


<a name ="Rr-make_shared"></a>
### R.23: `unique_ptr`을 만드려면 `make_unique()`를 사용하라.
>### Rule R.23: Use `make_unique()` to make `unique_ptr`s

**이유**: 편리성과 `shared_ptr`과의 일관성을 위해서.
>**Reason**: for convenience and consistency with `shared_ptr`.

**참조**: `make_unique()`는 C++14에 있지만, 폭넓게 사용 가능하다(만들기 또한 간단하다).
>**Note**: `make_unique()` is C++14, but widely available (as well as simple to write).

**시행하기**: `Shared_ptr`이 `make_unique`대신 `new`로 생성되어졌다면 경고하라.
(의견 - `unique_ptr`의 오기로 `Shared_ptr`로 기술되어있는 것 같습니다.)
>**Enforcement**: (Simple) Warn if a `Shared_ptr` is constructed from the result of `new` rather than `make_unique`.


<a name ="Rr-weak_ptr"></a>
### R.24: `shared_ptr`의 순환을 끊으려면 `std::weak_ptr`을 사용하라.
>### R.24: Use `std::weak_ptr` to break cycles of `shared_ptr`s

**이유**: `shared_ptr`은 사용 횟수에 의존적인데 순환 구조의 사용 횟수는 절대로 0이 되지 않는다. 때문에 우리는 순환 구조를 파괴할 수 있는 방법이 필요하다.
>**Reason**: `shared_ptr's rely on use counting and the use count for a cyclic structure never goes to zero, so we need a mechanism to
be able to destroy a cyclic structure.

**예제**:

 ???

>**Example**:

	???

**참고**:  ??? [[HS: 많은 사람들은 "순환을 끊는"이라고 말하는 반면 나는 "소유권 임시 공유"가 더 적절하다고 생각한다.]]
??? [[BS: 순환 끊기는 반드시 해야할 일인데, 어떻게 "임시적으로 소유권 공유"를 할 것인가. 당신은 다른 `shared_ptr`을 사용함으로써 "임시적으로 소유권 공유"를 할 수 있다.]]
>**Note**:  ??? [[HS: A lot of people say "to break cycles", while I think "temporary shared ownership" is more to the point.]]
???[[BS: breaking cycles is what you must do; temporarily sharing ownership is how you do it.
You could "temporarily share ownership simply by using another `stared_ptr`.]]

**시행하기**: ???아마도 불가능하다. 만약 당신이 정적으로 순환을 찾아낼 수 있다면, 우리는 `weak_ptr`이 필요할 일이 없다.
>**Enforcement**: ???probably impossible. If we could statically detect cycles, we wouldn't need `weak_ptr`


### R.30: Take smart pointers as parameters only to explicitly express lifetime semantics

### R.31: 만약 non-`std` 스마트 포인터를 사용 중이라면, `std`의 기본 패턴을 따라라.
>### R.31: If you have non-`std` smart pointers, follow the basic pattern from `std`

**이유**: 다음 섹션들의 규칙들 또한 다른 종류의 서드파티 혹은 커스텀 스마트 포인터 등에서도 동작할 것이며 성능과 정확성 문제를 일으키는 흔한 스마트 포인터 에러에 대한 분석에 매우 유용할 것이다. 당신은 사용하고 있는 모든 스마트 포인터에 대해서 이 규칙이 작동하기 원한다.

단항 연산자 `*`와 `->`를 과하게 쓰는 (기본 또는 특수 템플릿을 포함한)어떠한 타입이던 스마트 포인터가 고려되어진다:

* 복사할 수 있다면, 참조 카운팅 되는 `Shared_ptr`로 지정된다.
* 복사할 수 없다면, 고유한 `Unique_ptr`로 지정된다.

>**Reason**: The rules in the following section also work for other kinds of third-party and custom smart pointers and are very useful for diagnosing common smart pointer errors that cause performance and correctness problems.
You want the rules to work on all the smart pointers you use.

Any type (including primary template or specialization) that overloads unary `*` and `->` is considered a smart pointer:

* If it is copyable, it is recognized as a reference-counted `Shared_ptr`.
* If it not copyable, it is recognized as a unique `Unique_ptr`.

**예제**:

    // Boost의 intrusive_ptr 사용
    #include <boost/intrusive_ptr.hpp>
    void f(boost::intrusive_ptr<widget> p) { 	// 'sharedptrparam' 규칙 위배
        p->foo();
    }

    // Microsoft의 CComPtr 사용
	#include <atlbase.h>
    void f(CComPtr<widget> p) {                // 'sharedptrparam' 규칙 위배
        p->foo();
    }

>**Example**:

    // use Boost's intrusive_ptr
    #include <boost/intrusive_ptr.hpp>
    void f(boost::intrusive_ptr<widget> p) { 	// error under rule 'sharedptrparam'
        p->foo();
    }

    // use Microsoft's CComPtr
	#include <atlbase.h>
    void f(CComPtr<widget> p) {                // error under rule 'sharedptrparam'
        p->foo();
    }

두 케이스는 [`sharedptrparam` 가이드라인]에서 오류가 있다.:
`p`는 `Shared_ptr`이지만, 공유에 대해서는 아무것도 하지 않고 있다. 함수 안에서만 사용되는데 값을 전달함으로써 묵시적인 비효율(pessimization<->optimization) 객체이다.
이 함수들은 widget의 생명주기를 관리하고자 한다면 스마트 포인터를 넘겨받아야만 한다. 아니면 widget이 `nullptr`이 될 수 있다면 `widget*`를 넘겨받아야 하고, 그게 아닌 이상적인 상황은 함수가 `widget&`를 넘겨받아야 한다.
이 스마트포인터들은 `Shared_ptr` 컨셉과 잘 맞는다. 때문에 이 가이드라인 시행 규칙은 공통적인 비효율 객체를 드러내고 끄집어내어 위 함수들에서 잘 동작한다.
>Both cases are an error under the [`sharedptrparam` guideline](#Rr-smartptrparam):
`p` is a `Shared_ptr`, but nothing about its sharedness is used here and passing it by value is a silent pessimization;
these functions should accept a smart pointer only if they need to participate in the widget's lifetime management. Otherwise they should accept a `widget*`, if it can be `nullptr`. Otherwise, and ideally, the function should accept a `widget&`.
These smart pointers match the `Shared_ptr` concept,
so these guideline enforcement rules work on them out of the box and expose this common pessimization.


### R.32: Take smart pointers as parameters only to explicitly express lifetime semantics

**Reason**: Accepting a smart pointer to a `widget` is wrong if the function just needs the `widget` itself.
It should be able to accept any `widget` object, not just ones whose lifetimes are managed by a particular kind of smart pointer.
A function that does not manipulate lifetime should take raw pointers or references instead.

**Example; bad**:

    // callee
    void f( shared_ptr<widget>& w ) {
        // ...
        use( *w ); // only use of w -- the lifetime is not used at all
        // ...
    };

    // caller
    shared_ptr<widget> my_widget = /*...*/;
    f( my_widget );

    widget stack_widget;
    f( stack_widget ); // error

**Example; good**:

    // callee
    void f( widget& w ) {
        // ...
        use( w );
        // ...
    };

    // caller
    shared_ptr<widget> my_widget = /*...*/;
    f( *my_widget );

    widget stack_widget;
    f( stack_widget ); // ok -- now this works

**Enforcement**:

* (Simple) Warn if a function takes a parameter of a type that is a `Unique_ptr` or `Shared_ptr` and the function only calls any of: `operator*`, `operator->` or `get()`).
	Suggest using a `T*` or `T&` instead.


<a name="Rr-uniqueptrparam"></a>
### R.33: Take a `unique_ptr<widget>` parameter to express that a function assumes ownership of a `widget`

**Reason**: Using `unique_ptr` in this way both documents and enforces the function call's ownership transfer.

**Example**:

	void sink(unique_ptr<widget>); // consumes the widget

    void sink(widget*); 			// just uses the widget

**Example; bad**:

    void thinko(const unique_ptr<widget>&); // usually not what you want


**Enforcement**:

* (Simple) Warn if a function takes a `Unique_ptr<T>` parameter by lvalue reference and does not either assign to it or call `reset()` on it on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `Unique_ptr<T>` parameter by reference to `const`. Suggest taking a `const T*` or `const T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `Unique_ptr<T>` parameter by rvalue reference. Suggest using pass by value instead.


<a name="Rr-reseat"></a>
### R.34: Take a `unique_ptr<widget>&` parameter to express that a function reseats the`widget`

**Reason**: Using `unique_ptr` in this way both documents and enforces the function call's reseating semantics.

**Note**: "reseat" means "making a reference or a smart pointer refer to a different object."

**Example**:

    void reseat( unique_ptr<widget>& ); // "will" or "might" reseat pointer

**Example; bad**:

    void thinko( const unique_ptr<widget>& ); // usually not what you want


**Enforcement**:

* (Simple) Warn if a function takes a `Unique_ptr<T>` parameter by lvalue reference and does not either assign to it or call `reset()` on it on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `Unique_ptr<T>` parameter by reference to `const`. Suggest taking a `const T*` or `const T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `Unique_ptr<T>` parameter by rvalue reference. Suggest using pass by value instead.


<a name="Rr-sharedptrparam-owner"></a>
### R.35: Take a `shared_ptr<widget>` parameter to express that a function is part owner

**Reason**: This makes the function's ownership sharing explicit.

**Example; good**:

    void share( shared_ptr<widget> );            // share – “will” retain refcount

    void reseat( shared_ptr<widget>& );          // “might” reseat ptr

    void may_share( const shared_ptr<widget>& ); // “might” retain refcount

**Enforcement**:

* (Simple) Warn if a function takes a `Shared_ptr<T>` parameter by lvalue reference and does not either assign to it or call `reset()` on it on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes  a `Shared_ptr<T>` by value or by reference to `const` and does not copy or move it to another `Shared_ptr` on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `Shared_ptr<T>` by rvalue reference. Suggesting taking it by value instead.


<a name="Rr-sharedptrparam"></a>
### R.36: Take a `shared_ptr<widget>&` parameter to express that a function might reseat the shared pointer

**Reason**: This makes the function's reseating explicit.

**Note**: "reseat" means "making a reference or a smart pointer refer to a different object."

**Example; good**:

    void share( shared_ptr<widget> );            // share – “will” retain refcount

    void reseat( shared_ptr<widget>& );          // “might” reseat ptr

    void may_share( const shared_ptr<widget>& ); // “might” retain refcount

**Enforcement**:

* (Simple) Warn if a function takes a `Shared_ptr<T>` parameter by lvalue reference and does not either assign to it or call `reset()` on it on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes  a `Shared_ptr<T>` by value or by reference to `const` and does not copy or move it to another `Shared_ptr` on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `Shared_ptr<T>` by rvalue reference. Suggesting taking it by value instead.


<a name="Rr-sharedptrparam-const&"></a>
### R.37: Take a `const shared_ptr<widget>&` parameter to express that it might retain a reference count to the object ???

**Reason**: This makes the function's ??? explicit.

**Example; good**:

    void share( shared_ptr<widget> );            // share – “will” retain refcount

    void reseat( shared_ptr<widget>& );          // “might” reseat ptr

    void may_share( const shared_ptr<widget>& ); // “might” retain refcount

**Enforcement**:

* (Simple) Warn if a function takes a `Shared_ptr<T>` parameter by lvalue reference and does not either assign to it or call `reset()` on it on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes  a `Shared_ptr<T>` by value or by reference to `const` and does not copy or move it to another `Shared_ptr` on at least one code path. Suggest taking a `T*` or `T&` instead.
* (Simple) ((Foundation)) Warn if a function takes a `Shared_ptr<T>` by rvalue reference. Suggesting taking it by value instead.


<a name="Rr-smartptrget"></a>
### R.38: Do not pass a pointer or reference obtained from an aliased smart pointer

**Reason**: Violating this rule is the number one cause of losing reference counts and finding yourself with a dangling pointer.
Functions should prefer to pass raw pointers and references down call chains.
At the top of the call tree where you obtain the raw pointer or reference from a smart pointer that keeps the object alive.
You need to be sure that smart pointer cannot be inadvertently be reset or reassigned from within the call tree below

**Note**: To do this, sometimes you need to take a local copy of a smart pointer, which firmly keeps the object alive for the duration of the function and the call tree.

**Example**: Consider this code:

    // global (static or heap), or aliased local...
    shared_ptr<widget> g_p = ...;

    void f( widget& w ) {
	    g();
		use(w); // A
    }

    void g() {
	    g_p = ... ; // oops, if this was the last shared_ptr to that widget, destroys the widget
	}

The following should not pass code review:

    void my_code() {
	    f( *g_p );	// BAD: passing pointer or reference obtained from a nonlocal smart pointer
	                //      that could be inadvertently reset somewhere inside f or it callees
		g_p->func(); // BAD: same reason, just passing it as a "this" pointer
	}

The fix is simple -- take a local copy of the pointer to "keep a ref count" for your call tree:

    void my_code() {
	    auto pin = g_p; // cheap: 1 increment covers this entire function and all the call trees below us
	    f( *pin );	    // GOOD: passing pointer or reference obtained from a local unaliased smart pointer
		pin->func();    // GOOD: same reason
	}

**Enforcement**:

* (Simple) Warn if a pointer or reference obtained from a smart pointer variable (`Unique_ptr` or `Shared_ptr`) that is nonlocal, or that is local but potentially aliased, is used in a function call. If the smart pointer is a `Shared_ptr` then suggest taking a local copy of the smart pointer and obtain a pointer or reference from that instead.
