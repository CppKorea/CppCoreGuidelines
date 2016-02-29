# <a name="S-concurrency"></a> CP: Concurrency and Parallelism

???

동시성과 병렬성 규칙 요약 :
* [CP.1: 당신의 코드가 멀티 스레드 프로그램의 일부로 실행된다고 가정하라](#Rconc-multi)
* [CP.2: 데이터 경쟁을 피하라](#Rconc-races)

참고:
* [CP.con: Concurrency](#SScp-con)
* [CP.par: Parallelism](#SScp-par)
* [CP.simd: SIMD](#SScp-simd)
* [CP.free: Lock-free programming](#SScp-free)


> Concurrency and parallelism rule summary:
>
* [CP.1: Assume that your code will run as part of a multi-threaded program](#Rconc-multi)
* [CP.2: Avoid data races](#Rconc-races)

> See also:
> 
* [CP.con: Concurrency](#SScp-con)
* [CP.par: Parallelism](#SScp-par)
* [CP.simd: SIMD](#SScp-simd)
* [CP.free: Lock-free programming](#SScp-free)



-----
### <a name="Rconc-multi"></a> CP.1: 당신의 코드가 멀티 스레드 프로그램의 일부로 실행된다고 가정하라

##### 근거
지금 또는 미래에 병행성이 사용되지 않는다고 확정할 수 없다.
코드는 재사용된다.
스레드들을 사용하는 라이브러리들은 프로그램의 다른 부분에 사용될수도 있다.
이 가이드라인은 라이브러리 코드에는 가장 긴급하게 요구되며, 단일 응용 프로그램에 대해서는 가장 덜 긴급하게 요구된다.

##### 예
```
double cached_computation(double x)
{
    static double cached_x = 0.0;
    static double cached_result = COMPUTATION_OF_ZERO;
    double result;
 
    if (cached_x = x)
        return cached_result;
    result = computation(x);
    cached_x = x;
    cached_result = result;
    return result;
}
```
비록 `cached_computation`함수는 단일-스레드 환경에서는 완벽하게 동작하지만, 다중-스레드 환경에서는 2개의 `static`변수들이 데이터 경쟁으로 귀결되고 미정의된 동작을 발생시킨다.

이런 예시가 멀티 스레드 환경에서 안전해질 수 있는 몇가지 방법이 있다:
* 동시성에 대한 관심을 상위 호출자에게 위임하라. 
* `static`변수들을 `thread_local`로 표기하라 (캐싱이 비효율적으로 이루어질 수 있다).
* 동시적인 컨트롤을 구현하라. 예컨대, 두 `static`변수를 `static` 잠금으로 보호할 수 있다. (성능 감소를 야기한다)
* 호출자가 캐시를 위한 메모리를 제공하도록 하라. 그럼으로써 메모리 할당과 동시성에 대한 고려를 상위 호출자에게 위임하라. 
* 멀티-스레드 환경에서의 빌드나 실행을 거부하라.
* 2가지 구현들을 제공하라. 하나는 싱글 스레드 환경에서 사용되는 것이며, 다른 하나는 멀티 스레드 환경에서 사용되는 것이다.

##### 예외 사항
멀티 스레드 환경에서 절대 실행되지 않을 코드의 사례들이 있다. 하지만, 멀티 스레드 환경에서 절대 실행되지 않을 것이라고 **알고 있었던** 코드의 사례들 역시 많이 있다.
수년 후에, 일반적으로, 그런 프로그램들은 데이터 경쟁을 피하기 위한 고통스러운 노력으로 이어질 것이다.
따라서, 멀티 스레드 환경에서의 실행을 고려하지 않은 코드는 이상적으로 컴파일 중에 또는 실행 시간에 실행 메커니즘에서 버그들을 일찍 잡아낼 수 있도록 분명하게 표시되어야 한다.   


> ### <a name="Rconc-multi"></a> CP.1: Assume that your code will run as part of a multi-threaded program

> ##### Reason
> It is hard to be certain that concurrency isn't used now or sometime in the future.
Code gets re-used.
Libraries using threads may be used from some other part of the program.
Note that this applies most urgently to library code and least urgently to stand-alone applications.

> ##### Example
>
    double cached_computation(double x)
    {
        static double cached_x = 0.0;
        static double cached_result = COMPUTATION_OF_ZERO;
        double result;
 >
        if (cached_x = x)
            return cached_result;
        result = computation(x);
        cached_x = x;
        cached_result = result;
        return result;
    }
    
> Although `cached_computation` works perfectly in a single-threaded environment, in a multi-threaded environment the two `static` variables result in data races and thus undefined behavior.

> There are several ways that this example could be made safe for a multi-threaded environment:
* Delegate concurrency concerns upwards to the caller.
* Mark the `static` variables as `thread_local` (which might make caching less effective).
* Implement concurrency control, for example, protecting the two `static` variables with a `static` lock (which might reduce performance).
* Have the caller provide the memory to be used for the cache, thereby delegating both memory allocation and concurrency concerns upwards to the caller.
* Refuse to build and/or run in a multi-threaded environment.
* Provide two implementations, one which is used in single-threaded environments and another which is used in multi-threaded environments.

> **Exception**: There are examples where code will never be run in a multi-threaded environment.
However, there are also many examples where code that was "known" to never run in a multi-threaded program
was run as part of a multi-threaded program. Often years later.
Typically, such programs lead to a painful effort to remove data races.
Therefore, code that is never intended to run in a multi-threaded environment should be clearly labeled as such and ideally come with compile or run-time enforcement mechanisms to catch those usage bugs early.




### <a name="Rconc-races"></a>CP.2: 데이터 경쟁을 피하라

##### 근거

그렇게 하지 않으면, 무엇도 동작한다고 보장될 수 없으며 사소한 에러들이 계속 지속될 것이다.  

##### 참고 사항

쉽게 말해서, 만약 두 스레드들이 동시에 같은 명명된 객체에 접근할 수 있다면(동기화 없이), 그리고 한 스레드가 쓰기를 한다면(비-`const` 연산을 수행), 데이터 경쟁이 있는 것이다. 데이터 경쟁을 없애기 위해 어떻게 동기화를 잘할 수 있는지 알고 싶다면, 동시성에 대한 좋은 책을 참고하라.   

##### 시행하기

어떤 방법들은 가능하다. 최소한 무엇이라도 하라.

> ### <a name="Rconc-races"></a>CP.2: Avoid data races

> ##### Reason
> Unless you do, nothing is guaranteed to work and subtle errors will persist.

> ##### Note
> In a nutshell, if two threads can access the same named object concurrently (without synchronization), and at least one is a writer (performing a non-`const` operation), you have a data race. For further information of how to use synchronization well to eliminate data races, please consult a good book about concurrency.

> ##### Enforcement
> Some is possible, do at least something.


-----

## <a name="SScp-con"></a>CP.con: Concurrency

???

Concurrency rule summary:

* ???
* ???

???? should there be a "use X rather than `std::async`" where X is something that would use a better specified thread pool?

Speaking of concurrency, should there be a note about the dangers of `std::atomic` (weapons)?
A lot of people, myself included, like to experiment with `std::memory_order`, but it is perhaps best to keep a close watch on those things in production code.
Even vendors mess this up: Microsoft had to fix their `shared_ptr` (weak refcount decrement wasn't synchronized-with the destructor, if I recall correctly, although it was only a problem on ARM, not Intel)
and everyone (gcc, clang, Microsoft, and Intel) had to fix their `compare_exchange_*` this year, after an implementation bug caused losses to some finance company and they were kind enough to let the community know.

It should definitely be mentioned that `volatile` does not provide atomicity, does not synchronize between threads, and does not prevent instruction reordering (neither compiler nor hardware), and simply has nothing to do with concurrency.

    if (source->pool != YARROW_FAST_POOL && source->pool != YARROW_SLOW_POOL) {
        THROW(YARROW_BAD_SOURCE);
    }

??? Is `std::async` worth using in light of future (and even existing, as libraries) parallelism facilities? What should the guidelines recommend if someone wants to parallelize, e.g., `std::accumulate` (with the additional precondition of commutativity), or merge sort?

???UNIX signal handling???. May be worth reminding how little is async-signal-safe, and how to communicate with a signal handler (best is probably "not at all")


-----

## <a name="SScp-par"></a>CP.par: Parallelism

???

Parallelism rule summary:

* ???
* ???

-----

## <a name="SScp-simd"></a> CP.simd: SIMD

???

SIMD rule summary:

* ???
* ???

-----

## <a name="SScp-free"></a> CP.free: Lock-free 프로그래밍
???

> ## <a name="SScp-free"></a> CP.free: Lock-free programming
> ???


무잠금 프로그래밍 규칙 요약 : 
* ???
* ???

> Lock-free programming rule summary:
>
* ???
* ???

### <a name="Rconc"></a> 정말로 필요할때만 lock-free 프로그래밍을 사용하라
##### 근거
lock-free 프로그래밍은 에러에 취약하고 언어 특징, 머신 아키텍처, 그리고 자료구조에 대한 전문가 수준의 지식을 요구한다.

**대안**: 다른 라이브러리의 일부분으로 구현된 lock-free 자료구조들을 사용하라.

> ### <a name="Rconc"></a> Don't use lock-free programming unless you absolutely have to
> ##### Reason
> It's error-prone and requires expert level knowledge of language features, machine architecture, and data structures.
> **Alternative**: Use lock-free data structures implemented by others as part of some library.
