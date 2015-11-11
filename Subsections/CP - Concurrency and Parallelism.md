# <a name="S-concurrency"></a> CP: Concurrency and Parallelism

???

동시성과 병렬성 규칙 요약:
>Concurrency and parallelism rule summary:

* [CP.1: 코드가 다중스레드 프로그램의 한 부분으로 동작한다고 가정하라.](#Rconc-multi)
* [CP.2: 데이터 경쟁을 피하라.](#Rconc-races)

See also:

* [CP.con: 동시성](#SScp-con)
* [CP.par: 병렬성](#SScp-par)
* [CP.simd: SIMD](#SScp-simd)
* [CP.free: 락없는 프로그래밍](#SScp-free)

### <a name="Rconc-multi"></a> CP.1: 코드가 다중스레드 프로그램의 한 부분으로 동작한다고 가정하라.
>### <a name="Rconc-multi"></a> CP.1: Assume that your code will run as part of a multi-threaded program

##### Reason

추후 동기성이 사용되지 않는다고 확신하는 것은 어렵다.
코드는 재사용된다.
스레드를 사용하는 라이브러리는 프로그램의 다른 부분에서도 사용될지도 모른다.
>It is hard to be certain that concurrency isn't used now or sometime in the future.
Code gets re-used.
Libraries using threads my be used from some other part of the program.

##### Example

    ???

**Exception**: 멀티스레드 환경에서 코드가 결코 실행되지 않으려는 예들도 있다.
그러나 멀티스레드 프로그램에서 결코 실행되지 않는 코드가 멀티스레드 프로그램의 한 부분으로 동작하는 많은 예제들이 있다.
전형적으로 그런 프로그램은 데이터 경쟁을 없애기 위해 고통스런 노력을 야기한다.
>**Exception**: There are examples where code will never be run in a multi-threaded environment.
However, there are also many examples where code that was "known" to never run in a multi-threaded program
was run as part of a multi-threaded program. Often years later.
Typically, such programs lead to a painful effort to remove data races.

### <a name="Rconc-races"></a> CP.2: 데이터 경쟁을 피하라.
>### <a name="Rconc-races"></a> CP.2: Avoid data races

##### Reason

그렇게 하지 않으면, 동작을 보장할 수 없고 감지하기 힘든 에러가 끈질기게 계속될 것이다.
>Unless you do, nothing is guaranteed to work and subtle errors will persist.

##### Note

이 의미에 의심이 든다면 책을 읽어라.
>If you have any doubts about what this means, go read a book.

##### Enforcement

일부는 가능하지 그거라도 해라.
>Some is possible, do at least something.

## <a name="SScp-con"></a> CP.con: 동시성
>## <a name="SScp-con"></a> CP.con: Concurrency

???

동시성 규칙 요약:
>Concurrency rule summary:

* ???
* ???

????  X가 구체적인 스레드풀을 사용하는 것인 "`std::async`보다는 X를 사용하기"가 있어야 하나? (? - 어렵다)
>???? should there be a "use X rather than `std::async`" where X is something that would use a better specified thread pool?

동시성에 대해서 말해보면, `std::atomic`의 위험에 대해서 노트가 있어야 하는가?
나를 포함해서 많은 사람들은 `std::memory_order`로 실험하는 걸 좋아한다. 실제 코드 안에서 쓰이는 걸 가까이에서 지켜보는 것이 제일 좋다. (? - 어렵다.)
심지어 벤더들이 망쳐 버렸다: 마이크로소프트는 `shared_ptr`을 고쳐야 했다.
(약한 참조카운트가 소멸자에서 동기화되지 않았다. 내가 맞게 기억한다면, ARM만의 문제였을지라도.)
그리고 올해 모든 사람들(gcc, clang, Microsoft, Intel)은 `compare_exchange_*`을 고쳐야 했다.
구현 버그는 몇몇 금융회사에 손실을 야기했고, 그런 사실을 커뮤니티에 알릴 정도로 친절한 편이었다.
>Speaking of concurrency, should there be a note about the dangers of `std::atomic` (weapons)?
A lot of people, myself included, like to experiment with `std::memory_order`, but it is perhaps best to keep a close watch on those things in production code.
Even vendors mess this up: Microsoft had to fix their `shared_ptr` (weak refcount decrement wasn't synchronized-with the destructor, if I recall correctly, although it was only a problem on ARM, not Intel)
and everyone (gcc, clang, Microsoft, and Intel) had to fix their `compare_exchange_*` this year, after an implementation bug caused losses to some finance company and they were kind enough to let the community know.

`volatile`은 원자성을 제공하지 않고, 스레드간 동기화가 되지 않고, 명령순서 재조정을 방해하지 않고, (컴파일러, 하드웨어) 동기성과는 관련이 없다.
분명히 언급되어야 한다. (? - 사족이다.)
>It should definitely be mentioned that `volatile` does not provide atomicity, does not synchronize between threads, and does not prevent instruction reordering (neither compiler nor hardware), and simply has nothing to do with concurrency.

    if (source->pool != YARROW_FAST_POOL && source->pool != YARROW_SLOW_POOL) {
        THROW(YARROW_BAD_SOURCE);
    }

??? 미래 병렬 시설을 고려하여 `std::async`이 사용할 가치가 있는가?
만약 병렬화하고 싶다면 가이드라인은 무엇을 추천하는가? `std::accumulate`, 머지 소팅 등. (교환가능한 추가적인 선행조건을 가진)
>??? Is `std::async` worth using in light of future (and even existing, as libraries) parallelism facilities? What should the guidelines recommend if someone wants to parallelize, e.g., `std::accumulate` (with the additional precondition of commutativity), or merge sort?

???UNIX 시스널 처리???. 비동기 시그널이 얼마나 안전하지 않은지, 시그널 처리기와 소통하는 법을 기억해낼 가치가 있을 것이다. (최선은 아마도 "결코 아니다")
>???UNIX signal handling???. May be worth reminding how little is async-signal-safe, and how to communicate with a signal handler (best is probably "not at all")

## <a name="SScp-par"></a> CP.par: 병렬성
>## <a name="SScp-par"></a> CP.par: Parallelism

???

병렬성 규칙 요약:

* ???
* ???

## <a name="SScp-simd"></a> CP.simd: SIMD

???

Single Instruction Multiple Data 규칙 요약:
>SIMD rule summary:

* ???
* ???

## <a name="SScp-free"></a> CP.free: 락없는 프로그래밍

???

락없는 프로그래밍:

* ???
* ???

### <a name="Rconc"></a> 아예 할 수 없다면 락없는 프로그래밍을 하지 마라.
>### <a name="Rconc"></a> Don't use lock-free programming unless you absolutely have to

##### Reason

에러 발생이 쉽고 언어 특성, 기계구조, 데이터구조에 대한 전문가 레벨의 경험이 필요하다.
>It's error-prone and requires expert level knowledge of language features, machine architecture, and data structures.

**Alternative**: 라이브러리의 한부분으로 구현된 락없는 데이터 구조를 사용하라.
>**Alternative**: Use lock-free data structures implemented by others as part of some library.
