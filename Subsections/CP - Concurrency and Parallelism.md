# CP: 병행성과 병렬성
# CP: Concurrency and Parallelism

???

병행성과 병렬성 규칙 요약:
Concurrency and parallism rule summary:

* [CP.1: 당신의 코드가 멀티스레드 프로그램 속에서 작동한다고 가정하라](#Rconc-multi)
* [CP.1: Assume that your code will run as part of a multi-threaded program](#Rconc-multi)
* [CP.2: 데이터 경쟁 (data race)를 피하라](#Rconc-races)
* [CP.2: Avoid data races](#Rconc-races)

See also:

* [CP.con: Concurrency](#SScp-con)
* [CP.par: Parallelism](#SScp-par)
* [CP.simd: SIMD](#SScp-simd)
* [CP.free: Lock-free programming](#SScp-free)


<a name="Rconc-multi"></a>
### CP.1: 당신의 코드가 멀티스레드 프로그램 속에서 작동한다고 가정하라
### CP.1: Assume that your code will run as part of a multi-threaded program

**이유**: 병행성이 지금이나 추후에 사용될지 확신하기 어렵다.
**Reason**: It is hard to be certain that concurrency isn't used now or sometime in the future.
코드는 재사용된다.
Code gets re-used.
스레드를 사용하는 라이브러리가 프로그램의 다른 부분에서 사용될 수 있다.
Libraries using threads my be used from some other part of the program.

**Example**:

	???

**예외**: 전혀 멀티스레드 환경에서 작동하지 않는 코드의 예도 있다.
**Exception**: There are examples where code will never be run in a multi-threaded environment.
하지만, 멀티스레드 프로그램으로 결코 작동하지 않을 것으로 알려진 코드가 수년 후 멀티스레드 프로그램에서 작동하는 예 많이 있다.
However, here are also many examples where code that was "known" to never run in a multi-threaded program
was run as part of a multi-threaded program. Often years later.
일반적으로 이런 프로그램에서 데이터 경쟁을 없애기란 아주 고통스럽다.
Typically, such programs leads to a painful efford to remove data races.

<a name="Rconc-races"></a>
### CP.2: 데이터 경쟁을 피하라
### CP.2: Avoid data races

**이유**: 어떤 것도 제대로 작동한다고 보장할 수 없고 사소한 오류는 계속 남을 것이다.
**Reason**: Unless you do, nothing is guaranteed to work and subtle errors will persist.

**노트**: 데이터 경쟁이 무엇을 뜻하는지 잘 모르겠으면 책을 읽어라.
**Note**: If you have any doubts about what this means, go read a book.

**강제하기**
**Enforcement**: Some is possible, do at least something.


<a name="SScp-con"></a>
## CP.con: 병행성
## CP.con: Concurrency

???

병행성 규칙 요약:
Concurrency rule summary:

* ???
* ???

???? should there be a "use X rather than std::async" where X is something that would use a better specified thread pool?
Â 
Speaking of concurrency, should there be a note about the dangers of std::atomic (weapons)?
A lot of people, myself included, like to experiment with std::memory_order, but it is perhaps best to keep a close watch on those things in production code.
Even vendors mess this up: Microsoft had to fix their `shared_ptr`
(weak refcount decrement wasn't synchronized-with the destructor, if I recall correctly, although it was only a problem on ARM, not Intel)
and everyone (gcc, clang, Microsoft, and intel) had to fix their c`ompare_exchange_*` this year,
after an implementation bug caused losses to some finance company and they were kind enough to let the community know.

It should definitely mention that `volatile` does not provide atomicity, does not synchronize between threads, and does not prevent instruction reordering (neither compiler nor hardware), and simply has nothing to do with concurrency.

	if(source->pool != YARROW_FAST_POOL && source->pool != YARROW_SLOW_POOL) {
		THROW( YARROW_BAD_SOURCE );
	}
	
??? Is `std::async` worth using in light of future (and even existing, as libraries) parallelism facilities? What should the guidelines recommend if someone wants to parallelize, e.g., `std::accumulate` (with the additional precondition of commutativity), or merge sort?

???UNIX signal handling???. May be worth reminding how little is async-signal-safe, and how to communicate with a signal handler (best is probably "not at all")


<a name="SScp-par"></a>
## CP.par: Parallelism

???

Parallelism rule summary:

* ???
* ???


<a name="SScp-simd"></a>
## CP.simd: SIMD

???

SIMD rule summary:

* ???
* ???

<a name="SScp-free"></a>
## CP.free: Lock-free programming

???

Lock-free programming rule summary:

* ???
* ???

### <a name="Rconc">Don't use lock-free programming unless you absolutely have to</a>

**Reason**: It's error-prone and requires expert level knowledge of language features, machine architecture, and data structures.

**Alternative**: Use lock-free data structures implemented by others as part of some library.