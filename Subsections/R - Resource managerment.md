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

* R.1 : 자원 핸들과 RAII(자원 획득시 초기화)를 사용해서 자동으로 리소스를 관리해라.  
>

* [R.1: Manage resources automatically using resource handles and RAII (resource acquisition is initialization)](#Rr-raii)
* [R.2: In interfaces, use raw pointers to denote individual objects (only)](#Rr-use-ptr)
* [R.3: A raw pointer (a `T*`) is non-owning](#Rr-ptr)
* [R.4: A raw reference (a `T&`) is non-owning](#Rr-ref)
* [R.5: Prefer scoped objects](#Rr-scoped)
* [R.6: Avoid non-`const` global variables](#Rr-global)
	
Alocation and deallocation rule summary:
	
* [R.10: Avoid `malloc()` and `free()`](#Rr-mallocfree)
* [R.11: Avoid calling `new` and `delete` explicitly](#Rr-newdelete)
* [R.12: Immediately give the result of an explicit resource allocation to a manager object](#Rr-immediate-alloc)
* [R.13: Perform at most one explicit resource allocation in a single expression statement](#Rr-single-alloc)
* [R.14: ??? array vs. pointer parameter](#Rr-ap)
* [R.15: Always overload matched allocation/deallocation pairs](#Rr-pair)
	
<a name ="Rr-summary-smartptrs"></a>Smart pointer rule summary:

* [R.20: Use `unique_ptr` or `shared_ptr` to represent ownership](#Rr-owner)
* [R.21: Prefer `unique_ptr` over `shared_ptr` unless you need to share ownership](#Rr-unique)
* [R.22: Use `make_shared()` to make `shared_ptr`s](#Rr-make_shared)
* [R.23: Use `make_unique()` to make `unique_ptr`s](#Rr-make_unique)
* [R.24: Use `std::weak_ptr` to break cycles of `shared_ptr`s](#Rr-weak_ptr)
* [R.30: Take smart pointers as parameters only to explicitly express lifetime semantics](#Rr-smartptrparam)
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
이 특징을 사용해서 자원의 획득/해체시 짝 함수 호출이 필요한 자원을 다룰 때 생성자에서 자원을 획득하고 소멸자에서 해체하는 방법으로 강제되도록 오브젝트에 리소스를 갭슐화해라.

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
`...`로 표시된 코드에서 예외가 던져진다면 `x`는 누출되고 `my_mutex`는 락을 유지한다.

**좋은 예**

	void send( unique_ptr<X> x, cstring_view destination ) { // x가 X를 소유했다.
        Port port{destination};            // port가 Port 핸들을 소유했다.
        lock_guard<mutex> guard{my_mutex}; // guard가 락을 소유했다.
        // ...
        Send(port, x);
        // ...
	} // 자동으로 my_mutex 그리고 x, port의 자원이 해체될 것이다.

모든 자원 정리가 자동화되었고 예외와 상관없이 모든 경로에서 한번 수행된다. 참고로 함수가 포인터 소유권을 가져간 것도 보여주고 있다.

`Port`는 무엇일까? 자원을 캡슐화하는 간단한 래퍼로 구현할 수 있다.

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

**Note**: 소멸자를 가진 클래스에서 표현되지 않는 다루기 힘든 자원인 경우 클래스로 감싸거나 [`finally`](#S-GSL) 를 사용해라.

**참고**: [RAII](Rr-raii).
>

### Rule R.1: Manage resources automatically using resource handles and RAII (resource acquisition is initialization)

**Reason**: To avoid leaks and the complexity of manual resource management.
 C++'s language-enforced constructor/destructor symmetry mirrors the symmetry inherent in resource acquire/release function pairs such as `fopen`/`fclose`, `lock`/`unlock`, and `new`/`delete`.
 Whenever you deal with a resource that needs paired acquire/release function calls,
 encapsulate that resource in an object that enforces pairing for you -- acquire the resource in its constructor, and release it in its destructor.

**Example, bad**: Consider

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

In this code, you have to remember to `unlock`, `ClosePort`, and `delete` on all paths, and do each exactly once.
Further, if any of the code marked `...` throws an exception, then `x` is leaked and `my_mutex` remains locked.

**Example**: Consider

	void send( unique_ptr<X> x, cstring_view destination ) { // x owns the X
        Port port{destination};            // port owns the PortHandle
        lock_guard<mutex> guard{my_mutex}; // guard owns the lock
        // ...
        Send(port, x);
        // ...
	} // automatically unlocks my_mutex and deletes the pointer in x

Now all resource cleanup is automatic, performed once on all paths whether or not there is an exception. As a bonus, the function now advertises that it takes over ownership of the pointer.

What is `Port`? A handy wrapper that encapsulates the resource:

    class Port {
        PortHandle port;
    public:
        Port( cstring_view destination ) : port{OpenPort(destination)} { }
        ~Port() { ClosePort(port); }
        operator PortHandle() { return port; }

        // port handles can't usually be cloned, so disable copying and assignment if necessary
        Port(const Port&) =delete;
        Port& operator=(const Port&) =delete;
    };

**Note**: Where a resource is "ill-behaved" in that it isn't represented as a class with a destructor, wrap it in a class or use [`finally`](#S-GSL)

**See also**: [RAII](Rr-raii).

<a name ="Rr-use-ptr"></a>
### R.2: In interfaces, use raw pointers to denote individual objects (only)

**Reason**: Arrays are best represented by a container type (e.g., `vector` (owning)) or an `array_view` (non-owning).
Such containers and views hold sufficient information to do range checking.

**Example, bad**:

	void f(int* p, int n)	// n is the number of elements in p[]
	{
		// ...
		p[2] = 7;	// bad: subscript raw pointer
		// ...
	}

The compiler does not read comments, and without reading other code you do not know whether `p` really points to `n` elements.
Use an `array_view` instead.

**Example**:

	void g(int* p, int fmt)	// print *p using format #fmt
	{
		// ... uses *p and p[0] only ...
	}
	
**Exception**: C-style strings are passed as single pointers to a zero-terminated sequence of characters.
Use `zstring` rather than `char*` to indicate that you rely on that convention.

**Note**: Many current uses of pointers to a single element could be references.
However, where `nullptr` is a possible value, a reference may not be an reasonable alternative.

**Enforcement**:

* Flag pointer arithmetic (including `++`) on a pointer that is not part of a container, view, or iterator.
This rule would generate a huge number of false positives if applied to an older code base.
* Flag array names passed as simple pointers



<a name="Rr-ptr"></a>
### R.3: A raw pointer (a `T*`) is non-owning

**Reason**: There is nothing (in the C++ standard or in most code) to say otherwise and most raw pointers are non-owning.
We want owning pointers identified so that we can reliably and efficiently delete the objects pointed to by owning pointers.

**Example**:

	void f()
	{
		int* p1 = new int{7};			// bad: raw owning pointer
		auto p2 = make_unique<int>(7);	// OK: the int is owned by a unique pointer
		// ...
	}

The `unique_ptr` protects against leaks by guaranteeing the deletion of its object (even in the presence of exceptions). The `T*` does not.

**Example**:

	template<typename T>
	class X {
		// ...
	public:
		T* p;	// bad: it is unclear whether p is owning or not
		T* q;	// bad: it is unclear whether q is owning or not
	};

We can fix that problem by making ownership explicit:

	template<typename T>
	class X2 {
		// ...
	public:
		owner<T> p;	// OK: p is nowning
		T* q;		// OK: q is not owning
	};

**Note**: The fact that there are billions of lines of code that violates this rule against owning `T*`s cannot be ignored.
This code cannot all be rewritten (ever assuming good code transformation software).
This problem cannot be solved (at scale) by transforming all owning pointer to `unique_ptr`s and `shared_ptr`s, partly because we need/use owning "raw pointers" in the implementation of our fundamental resource handles. For example, most `vector` implementations have one owning pointer and two non-owning pointers.
Also, many ABIs (and essentially all interfaces to C code) use `T*`s, some of them owning.

**Note**: `owner<T>` has no default semantics beyond `T*` it can be used without changing any code using it and without affecting ABIs.
It is simply a (most valuable) indicator to programmers and analysis tools.
For example, if an `owner<T>` is a member of a class, that class better have a destructor that `delete`s it.
	
**Example**, bad:
Returning a (raw) pointer imposes a life-time management burden on the caller; that is, who deletes the pointed-to object?

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

In addition to suffering from then problem from [leak](#???), this adds a spurious allocation and deallocation operation,
and is needlessly verbose. If Gadget is cheap to move out of a function (i.e., is small or has an efficient move operation),
just return it "by value:'

	Gadget make_gadget(int n)
	{
		Gadget g{n};
		// ...
		return g;
	}

**Note**: This rule applies to factory functions.

**Note**: If pointer semantics is required (e.g., because the return type needs to refer to a base class of a class hierarchy (an interface)),
return a "smart pointer."

**Enforcement**:

* (Simple) Warn on `delete` of a raw pointer that is not an `owner<T>`.
* (Moderate) Warn on failure to either `reset` or explicitly `delete` an `owner<T>` pointer on every code path.
* (Simple) Warn if the return value of `new` or a function call with return value of pointer type is assigned to a raw pointer.
* (Simple) Warn if a function returns an object that was allocated within the function but has a move constructor.
Suggest considering returning it by value instead.


<a name="Rr-ref"></a>
### R.4: A raw reference (a `T&`) is non-owning

**Reason**: There is nothing (in the C++ standard or in most code) to say otherwise and most raw references are non-owning.
We want owners identified so that we can reliably and efficiently delete the objects pointed to by owning pointers.

**Example**:

	void f()
	{
		int& r = *new int{7};			// bad: raw owning reference
		// ...
		delete &r;						// bad: violated the rule against deleting raw pointers
	}
	
**See also**: [The raw pointer rule](#Rr-ptr)

**Enforcement**: See [the raw pointer rule](#Rr-ptr)


<a name="Rr-scoped"></a>
### R.5: Prefer scoped objects

**Reason**: A scoped object is a local object, a global object, or a member.
This implies that there is no separate allocation and deallocation cost in excess that already used for the containing scope or object.
The members of a scoped object are themselves scoped and the scoped object's constructor and destructor manage the members' lifetimes.

**Example**: the following example is inefficient (because it has unnecessary allocation and deallocation), vulnerable to exception throws and returns in the "¦ part (leading to leaks), and verbose:

	void some_function(int n)
	{
		auto p = new Gadget{n};
		// ...
		delete p;
	}

Instead, use a local variable:

	void some_function(int n)
	{
		Gadget g{n};
		// ...
	}

**Enforcement**:

* (Moderate) Warn if an object is allocated and then deallocated on all paths within a function. Suggest it should be a local `auto` stack object instead.
* (Simple) Warn if a local `Unique_ptr` or `Shared_ptr` is not moved, copied, reassigned or `reset` before its lifetime ends.



<a name="Rr-global"></a>
### R.6: Avoid non-`const` global variables

**Reason**: Global variables can be accessed from everywhere so they can introduce surprising dependencies between apparently unrelated objects.
They are a notable source of errors.

**Warning**: The initialization of global objects is not totally ordered. If you use a global object initialize it with a constant.

**Exception**: a global object is often better than a singleton.

**Exception**: An immutable (`const`) global does not introduce the problems we try to avoid by banning global objects.

**Enforcement**: [[??? NM: Obviously we can warn about non-const statics....do we want to?]]


<a name="SS-alloc"></a>
## R.alloc: Alocation and deallocation


<a name ="Rr-mallocfree"></a>
### R.10: Avoid `malloc()` and `free()`

**Reason**: `malloc()` and `free()` do not support construction and destruction, and do not mix well with `new` and `delete`.

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


<a name ="Rr-newdelete"></a>
### R.11: Avoid calling `new` and `delete` explicitly

**Reason**: The pointer returned by `new` should belong to a resource handle (that can call `delete`).
If the pointer returned from `new` is assigned to a plain/naked pointer, the object can be leaked.

**Note**: In a large program, a naked `delete` (that is a `delete` in application code, rather than part of code devoted to resource management)
is a likely bug: if you have N `delete`s, how can you be certain that you don't need N+1 or N-1?
The bug may be latent: it may emerge only during maintenace.
If you have a naled `new`, you probably need a naked `delete` somewhere, so yu probably have a bug.

**Enforcement**: (Simple) Warn on any explicit use of `new` and `delete`. Suggest using `make_unique` instead.


<a name ="Rr-immediate alloc"></a>
### R.12: Immediately give the result of an explicit resource allocation to a manager object

**Reason**: If you don't, an exception or a return may lead to a leak.

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


<a name ="Rr-single alloc"></a>
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
### R.14: ??? array vs. pointer parameter

**Reason**: An array decays to a pointer, thereby loosing its size, opening the opportunity for range errors.

**Example**:

	??? what do we recommend: f(int*[]) or f(int**) ???

**Alternative**: Use `array_view` to preserve size information.

**Enforcement**: Flag `[]` parameters.


<a name="Rr-pair"></a>
### R.15: Always overload matched allocation/deallocation pairs

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
### Rule R.20: Use `unique_ptr` or `shared_ptr` to represent ownership

**Reason**: They can prevent resource leaks.

**Example**: Consider

	void f()
	{
		X x;
		X* p1 { new X };			// see also ???
		unique_ptr<T> p2 { new X };	// unique ownership; see also ???
		shared_ptr<T> p3 { new X };	// shared ownership; see also ???
	}

This will leak the object used to initialize `p1` (only).

**Enforcement:** (Simple) Warn if the return value of `new` or a function call with return value of pointer type is assigned to a raw pointer.


<a name="Rr-unique"></a>
### Rule R.21: Prefer `unique_ptr` over `shared_ptr` unless you need to share ownership

**Reason**: a `unique_ptr` is conceptually simpler and more predictable (you know when destruction happens) and faster (you don't implicitly maintain a use count).

**Example, bad**: This needlessly adds and maintains a reference count

    void f()
    {
        shared_ptr<Base> base = make_shared<Derived>();
        // use base locally, without copying it -- refcount never exceeds 1
    } // destroy base

**Example**: This is more efficient

    void f()
    {
        unique_ptr<Base> base = make_unique<Derived>();
        // use base locall
    } // destroy base


**Enforcement**: (Simple) Warn if a function uses a `Shared_ptr` with an object allocated within the function, but never returns the `Shared_ptr` or passes it to a function requiring a `Shared_ptr&`. Suggest using `unique_ptr` instead.


<a name ="Rr-make_shared"></a>
### R.22: Use `make_shared()` to make `shared_ptr`s

**Reason**: If you first make an object and then gives it to a `shared_ptr` constructor, you (most likely) do one more allocation (and later deallocation) than if you use `make_shared()` because the reference counts must be allocated separately from the object.

**Example**: Consider

	shared_ptr<X> p1 { new X{2} }; // bad
	auto p = make_shared<X>(2);    // good

The `make_shared()` version mentions `X` only once, so it is usually shorter (as well as faster) than the version with the explicit `new`.

**Enforcement**: (Simple) Warn if a `shared_ptr` is constructed from the result of `new` rather than `make_shared`.


<a name ="Rr-make_shared"></a>
### Rule R.23: Use `make_unique()` to make `unique_ptr`s

**Reason**: for convenience and consistency with `shared_ptr`.

**Note**: `make_unique()` is C++14, but widely available (as well as simple to write).

**Enforcement**: (Simple) Warn if a `Shared_ptr` is constructed from the result of `new` rather than `make_unique`.


<a name ="Rr-weak_ptr"></a>
### R.30: Use `std::weak_ptr` to break cycles of `shared_ptr`s

**Reason**: `shared_ptr's rely on use counting and the use count for a cyclic structure never goes to zero, so we need a mechanism to
be able to destroy a cyclic structure.

**Example**:

	???

**Note**:  ??? [[HS: A lot of people say "to break cycles", while I think "temporary shared ownership" is more to the point.]]
???[[BS: breaking cycles is what you must do; temporarily sharing ownership is how you do it.
You could "temporarily share ownership simply by using another `stared_ptr`.]]

**Enforcement**: ???probably impossible. If we could statically detect cycles, we wouldn't need `weak_ptr`


<a name="Rr-smart"></a>
### R.31: If you have non-`std` smart pointers, follow the basic pattern from `std`

**Reason**: The rules in the following section also work for other kinds of third-party and custom smart pointers and are very useful for diagnosing common smart pointer errors that cause performance and correctness problems.
You want the rules to work on all the smart pointers you use.

Any type (including primary template or specialization) that overloads unary `*` and `->` is considered a smart pointer:

* If it is copyable, it is recognized as a reference-counted `Shared_ptr`.
* If it not copyable, it is recognized as a unique `Unique_ptr`.

**Example**:

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

Both cases are an error under the [`sharedptrparam` guideline](#Rr-smartptrparam):
`p` is a `Shared_ptr`, but nothing about its sharedness is used here and passing it by value is a silent pessimization;
these functions should accept a smart pointer only if they need to participate in the widget's lifetime management. Otherwise they should accept a `widget*`, if it can be `nullptr`. Otherwise, and ideally, the function should accept a `widget&`.
These smart pointers match the `Shared_ptr` concept,
so these guideline enforcement rules work on them out of the box and expose this common pessimization.


<a name="Rr-smartptrparam"></a>
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
