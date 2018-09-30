
# <a name="S-concurrency"></a>CP: 동시성과 병렬성

종종 우리 컴퓨터가 동시에 많은 작업들을 해주길 (최소한 그렇게 보이기를) 원할 때가 있죠.

이유는 다양합니다. (예를 들면, 단일 프로세서만 쓰면서 여러 이벤트를 기다리기를 원하거나, 동시에 다수의 데이터 스트림을 처리하길 원하거나, 또는 하드웨어 기능들을 사용하길 원할 수도 있겠죠.) 
동시성과 병렬성을 표현하기 위한 기본 기능들 또한 그러합니다.

여기서는, ISO 표준 C++에서 제공하는 기초적인 동시성과 병렬성을 위한 기능에 관한 몇몇 일반적인 원칙과 규칙에 대해서 기술합니다. 

동시적이고 병렬적인 프로그래밍을 위한 기계 수준의 지원(The core machine support)은 바로 스레드입니다. 스레드들을 같은 메모리를 공유하면서도 다수의 프로그램들을 독립적으로 실행할 수 있도록 해줍니다.  

많은 이유로 인해 동시성 프로그래밍은 까다로운데, 무엇보다도 스레드들 사이에 적절한 동기화(synchronization)가 없다면 한 스레드에서 데이터를 읽는 동안 다른 스레드에서 같은 데이터를 덮어쓸 때의 동작을 정의할 수 없기 때문(undefined behavior)입니다.

이미 존재하는 단일 스레드 코드는 `std::async` 또는 `std::thread`를 전략적으로 추가하는 것으로 간단하게 병렬화할 수 있습니다. 또는 코드 전체를 완전히 다시 작성해야 할 수도 있죠. 이는 원래의 코드가 스레드에 적합하게(thread-friendly) 작성되었느냐에 따라 다릅니다.

핵심 가이드라인의 동시성/병렬성 규칙들은 3가지 목표를 가지고 설계되었습니다:

 * 멀티스레드 환경에서 실행되는 코드를 작성할 수 있도록 돕는 것 
 * 표준 라이브러리에서 제공하는 스레드 기본연산(primitives)을 사용하는 깔끔하고, 안전한 방법을 보여주는 것
 * 동시성과 병렬성이 기대만큼의 성능 향상을 가져오지 못할 때의 가이드를 제공하는 것

C++에서의 동시성은 아직 진행 중이라는 것을 기억하길 바랍니다. C++ 11에서는 많은 동시성 기본연산을 소개했었고, C++14와 C++17에서는 발전이 이루어졌습니다. 
C++를 사용한 동시적인(concurrent) 프로그램들을 쉽게 작성하는 것에 대한 관심도 늘어났죠. 
이 문서에 있는 일부 라이브러리와 관련된 가이드도 시간이 지남에 따라 계속 바뀌기를 기대합니다.

(당연하게도) 이 부분은 많은 작업이 필요합니다.
핵심 가이드라인이 전문가가 아닌 사람들을 위한 규칙으로 시작한다는 점을 유의해주십시오. 당신이 전문가라면, 조금 더 기다리셔야 할겁니다; 
이 문서에 기여하는 것도 좋구요. 하지만 정확하고, 빠른 동시적인 프로그램을 작성하길 원하는 대부분의 프로그래머들을 생각해 주시기 바랍니다.

동시성과 병렬성 규칙 요약:  

* [CP.1: 코드가 멀티스레드 프로그램의 일부로 동작할 것이라 가정하라](#Rconc-multi)
* [CP.2: 데이터 경쟁을 피하라](#Rconc-races)
* [CP.3: 쓰기 가능한 데이터의 명시적인 공유를 최소화하라](#Rconc-data)
* [CP.4: 스레드보단 작업 단위로 생각하라](#Rconc-task)
* [CP.8: 동기화를 위해 `volatile`을 사용하지 말아라](#Rconc-volatile)
* [CP.9: Whenever feasible use tools to validate your concurrent code](#Rconc-tools)

**참고**:

* [CP.con: 동시성(Concurrency)](#SScp-con)
* [CP.par: 병렬성(Parallelism)](#SScp-par)
* [CP.mess: 메세지 전달(Message passing)](#SScp-mess)
* [CP.vec: 벡터화(Vectorization)](#SScp-vec)
* [CP.free: 무잠금 프로그래밍(Lock-free programming)](#SScp-free)
* [CP.etc: 기타 동시성 규칙들](#SScp-etc)

### <a name="Rconc-multi"></a>CP.1: 코드가 멀티스레드 프로그램의 일부로 동작할 것이라 가정하라

##### Reason
동시성이 지금, 그리고 미래의 언젠가 사용되지 않을 것이라고 확정하기 어렵다. 코드는 재사용된다.  
스레드를 사용하는 라이브러리들은 다른 프로그램의 일부로 사용되기 마련이다. 
이 규칙은 단일 응용 프로그램보다는 라이브러리 코드에 더 시급하게 적용되어야 한다. 하지만, Ctrl C, V의 마법 덕분에, 코드가 예상치 않은 곳에서 나타날 수 있다. 

##### Example

```c++
    double cached_computation(double x)
    {
        static double cached_x = 0.0;
        static double cached_result = COMPUTATION_OF_ZERO;
        double result;

        if (cached_x == x)
            return cached_result;
        result = computation(x);
        cached_x = x;
        cached_result = result;
        return result;
    }
```

비록 `cached_computation`함수는 단일 스레드 환경에선 완벽하게 동작하지만, 멀티 스레드 환경에선 두 개의 `static` 변수들이 데이터 경쟁으로 이어지고, 비정의 동작(undefined behavior)을 유발할 것이다.

멀티 스레드 환경에서 이런 코드를 안전하게 바꿀 수 있는 방법들이 있다:

* 동시성에 대한 사항을 상위 호출자에게 위임하라
* `static` 변수들을 `thread_local`로 만들어라 (아마 캐싱을 덜 효율적으로 만들 수도 있다) 
* 동시적인 제어를 구현하라, 예컨대, 두 `static`변수를 `static` 잠금으로 보호하라 (성능을 감소시킬 수 있다) 
* 호출자가 캐시로 사용할 메모리를 제공하도록 하라. 그렇게 함으로써 메모리 할당과 동시성에 대한 문제를 호출자에게 위임하라
* 멀티 스레드 환경에서의 빌드, 실행을 거부하라
* 두 가지 구현을 지원하라. 단일 스레드 환경을 위한 구현과 멀티 스레드 환경을 위한 것 모두

##### Exception

멀티 스레드 환경에서 절대 실행되지 않을 코드.  

주의: 멀티 스레드 프로그램에서 절대로 돌아가지 않을 줄 "알았던" 코드들이 몇년 뒤엔 멀티 스레드 프로그램의 일부가 된 사례는 많이 있다. 일반적으로, 그런 프로그램들은 데이터 경쟁을 없애기 위한 고통스러운 노력으로 이어진다. 
때문에, 멀티 스레드 환경에서 실행되는 것을 의도하지 않은 코드들은 분명하게 해당 사항이 기술되어야 하고, 이상적으로는 컴파일 또는 실행시간에 버그를 일찍 찾을 수 있는 메커니즘이 함께 사용되되어야 한다.

### <a name="Rconc-races"></a>CP.2: 데이터 경쟁을 피하라

##### Reason

데이터 경쟁이 있으면, 아무것도 보장할 수 없으며 미묘한 에러들이 계속될 것이다.

##### Note

쉽게 말해서, 만약 두 스레드가 같은 객체를 (동기화 없이) 동시적으로 접근할 수 있다면, 그리고 한 스레드가 쓰기(non-`const` 연산)를 수행한다면, 데이터 경쟁이 있는 것이다.  
어떻게 동기화를 사용하고 데이터 경쟁을 없앨 것인지 더 알고 싶다면 동시성에 대한 좋은 책들을 참고하라.

##### Example, bad

데이터 경쟁이 존재하는 예시는 많다. 지금 이 순간 실행 중인 production 소프트웨어들 중에도 존재할 것이다. 

간단한 예시를 들자면:

```c++
    int get_id() {
      static int id = 1;
      return id++;
    }
```

여기서 증가 연산은 데이터 경쟁의 예시다.

이 코드는 다음과 같이 잘못될 수 있다:

* 스레드 A가 `id`를 로드하고, OS가 A를 중지시킨다. 그 사이 다른 스레드가 ID를 수백 개 생성한다. 그 후에 A는 다시 실행되고, A의 문맥에서 증가한 `id`값이 다시 써지게 된다.
* 스레드 A와 B가 `id`를 로드하고 동시에 증가시킨다. 그 결과 두 스레드는 같은 ID값을 가진다.

지역 정적 변수들은 데이터 경쟁의 일반적인 원인이다.

##### Example, bad:

```c++
    void f(fstream&  fs, regex pat)
    {
        array<double, max> buf;
        int sz = read_vec(fs, buf, max);            // read from fs into buf
        gsl::span<double> s {buf};
        // ...
        auto h1 = async([&]{ sort(par, s); });     // spawn a task to sort
        // ...
        auto h2 = async([&]{ return find_all(buf, sz, pat); });   // spawn a task to find matches
        // ...
    }
```

이 코드에서는, `buf`의 원소들에 대한 데이터 경쟁이 있다. (`sort`가 읽기/쓰기 모두 수행할 것이다).
좋은 데이터 경쟁이란 없다.
이 코드에선 스택의 데이터에 대한 데이터 경쟁이 발생하는데, 모든 데이터 경쟁이 이처럼 찾아내기 쉽지는 않다.

##### Example, bad:

```c++
    // code not controlled by a lock

    unsigned val;

    if (val < 5) {
        // ... other thread can change val here ...
        switch (val) {
        case 0: // ...
        case 1: // ...
        case 2: // ...
        case 3: // ...
        case 4: // ...
        }
    }
```

이 경우, `val`이 바뀔수도 있다는 것을 모르는 컴파일러는 `switch`를 5개의 엔트리를 지닌 점프 테이블로 구현할 것이다. 그러면, `[0..4]`범위 밖의 값을 가진 `val`은 프로그램의 어딘가로 점프를 할 것이고, 그 지점부터 실행될 것이다.  
데이터 경쟁이 있으면 그 무엇도 장담할 수 없다.  
실제로, 더 안좋은 결과로 이어질 수도 있다: 컴파일 결과 코드를 보면서 길잃은 점프가 특정 값에 따라서 어디로 갈지 알 수 있을지도 모르지만, 이는 보안 위험이 될수도 있다.

##### Enforcement

최소한 무엇이라도 하라.  
이 문제를 해결하기 위해 상업용 그리고 오픈소스 도구들을 사용하라. 하지만 정적 도구들은 종종 잘못된 코드를 용인할수도 있다. 또 런타임 도구들은 종종 상당한 비용을 필요로 한다. 
더 나은 도구들이 나오기를 바란다. 여러 도구들을 사용하는 것이 하나를 사용하는 것보다 많은 문제를 잡아낼 수 있을 것이다.

데이터 경쟁 문제를 완화하기 위한 방법이 몇가지 있다:

* 더 적은 전역 데이터
* 더 적은 `static` 변수들
* 스택 메모리 중심의 사용 (포인터들을 너무 많이 던지지 말아라)
* 변경할 수 없는 데이터를 더 많이 써라. (리터럴, `constexpr`, `const`)

### <a name="Rconc-data"></a>CP.3: 쓰기 가능한 데이터의 명시적인 공유를 최소화하라

##### Reason

쓰기 가능한 데이터를 공유하지 않는다면, 데이터 경쟁을 원천적으로 막을 수 있다.
공유를 줄일 수록, 동기화 접근과 데이터 경쟁의 가능성을 줄일 수 있다. 
더해서, 대기와 잠금 역시 줄어든다. (따라서 성능이 향상될 것이다)

##### Example

```c++
    bool validate(const vector<Reading>&);
    Graph<Temp_node> temperature_gradiants(const vector<Reading>&);
    Image altitude_map(const vector<Reading>&);
    // ...

    void process_readings(const vector<Reading>& surface_readings)
    {
        auto h1 = async([&] { if (!validate(surface_readings)) throw Invalid_data{}; });
        auto h2 = async([&] { return temperature_gradiants(surface_readings); });
        auto h3 = async([&] { return altitude_map(surface_readings); });
        // ...
        h1.get();
        auto v2 = h2.get();
        auto v3 = h3.get();
        // ...
    }
```

위 코드에서 `const`들이 없다면, `surface_readings`에 대한 잠재적 데이터 경쟁 때문에 모든 비동기적인 함수 호출을 다시 점검해야 할 것이다.  
`surface_readings`을 (이 함수에 대해서) `const`로 만들면 문제 영역을 함수 안쪽으로 한정할 수 있다.

##### Note

변경할 수 없는 데이터는 안전하고 효율적으로 공유될 수 있다.  
여기엔 잠금이 필요하지 않다: 상수에 대해서는 데이터 경쟁이 발생하지 않는다.

[CP.mess: Message Passing](#SScp-mess)와 [CP.31: prefer pass by value](#Rconc-data-by-value)를 함께 확인하라.

##### Enforcement

???


### <a name="Rconc-task"></a>CP.4: 스레드보단 작업 단위로 생각하라

##### Reason

`thread`자체는 구현에 대한 개념이다. 이는 기계에 대해서 생각하는 것이다.  
작업은 프로그램((Application)에 대해서 생각하는 것이다. 당신의 생각과 더 가까울 것이고, 아마도 다른 작업들과 동시적일 것이다.  
응용개념은 추론하기도 쉽다.

##### Example

```c++
    void some_fun() {
        std::string  msg, msg2;
        std::thread publisher([&] { msg = "Hello"; });       // bad: less expressive
                                                             //      and more error-prone
        auto pubtask = std::async([&] { msg2 = "Hello"; });  // OK
        // ...
        publisher.join();
    }
```

##### Note
`async()`를 제외하면, 표준 라이브러리 기능들은 저-레벨에, 기계 중심적(machine-oriented)이고, 스레드-잠금 레벨에 위치한다. 
이 기능들은 필수적이지만, 우리는 생산성, 신뢰성, 그리고 성능을 위해서 추상화의 수준을 더 높일 필요가 있다.

상위 레벨의, 응용 프로그램 중심적인 라이브러리들을 사용하는 것에 대해선 논의의 여지가 있다.
(가능하다면 표준 라이브러리 기능들을 사용하라).

##### Enforcement

???

### <a name="Rconc-volatile"></a>CP.8: 동기화를 위해 `volatile`을 사용하지 말아라

##### Reason

C++ 에선, 다른 언어와는 다르게, `volatile`이 원자성과 스레드간 동기화를 제공하지 않는다.
또한 `volatile`은 명령어 재배치(컴파일러와 하드웨어 모두)를 제한하지도 않는다.

`volatile`은 동시성과 무관하다.

##### Example, bad:

```c++
    int free_slots = max_slots; // current source of memory for objects

    Pool* use()
    {
        if (int n = free_slots--) return &pool[n];
    }
```

이 코드엔 문제가 있다:

단일 스레드 프로그램이라면, 이 코드는 아무런 문제가 없다. 하지만 두 스레드가 동시에 실행한다면, `free_slots`에 대한 경쟁 상태가 발생한다. 따라서 두 스레드는 같은 `free_slots`값을 가질 수도 있다.  

이는 (명백하게) 잘못된 결과로 이어진다. 다른 언어들에 익숙한 사람들은 이 코드를 다음과 같이 수정할 것이다.:

```c++
    volatile int free_slots = max_slots; // current source of memory for objects

    Pool* use()
    {
        if (int n = free_slots--) return &pool[n];
    }
```

`volatile`로 바꿨지만, 동기화엔 아무런 영향이 없다. 이 코드엔 여전히 데이터 경쟁이 존재한다!

C++ 에선, 동기화를 원한다면 `atomic` 타입들을 사용해야 한다:

```c++
    atomic<int> free_slots = max_slots; // current source of memory for objects

    Pool* use()
    {
        if (int n = free_slots--) return &pool[n];
    }
```

이제 `--`연산은 원자적이다. 이는 다른 스레드가 끼여들 여지가 있는 읽기-증가-쓰기 과정과는 다르다.

##### Alternative

다른 언어에서 `volatile`을 사용했다면, C++에선 `atomic`을 사용하라.
더 복잡한 경우라면 `mutex`를 사용하라.

##### See also
[`volatile`의 적절한 사용](#Rconc-volatile2)

### <a name="Rconc-tools"></a>CP.9: Whenever feasible use tools to validate your concurrent code

Experience shows that concurrent code is exceptionally hard to get right
and that compile-time checking, run-time checks, and testing are less effective at finding concurrency errors
than they are at finding errors in sequential code.
Subtle concurrency errors can have dramatically bad effects, including memory corruption and deadlocks.

##### Example

    ???

##### Note

Thread safety is challenging, often getting the better of experienced programmers: tooling is an important strategy to mitigate those risks.
There are many tools "out there", both commercial and open-source tools, both research and production tools.
Unfortunately people's needs and constraints differ so dramatically that we cannot make specific recommendations,
but we can mention:

* Static enforcement tools: both [clang](http://clang.llvm.org/docs/ThreadSafetyAnalysis.html)
and some older versions of [GCC](https://gcc.gnu.org/wiki/ThreadSafetyAnnotation)
have some support for static annotation of thread safety properties.
Consistent use of this technique turns many classes of thread-safety errors into compile-time errors.
The annotations are generally local (marking a particular member variable as guarded by a particular mutex),
and are usually easy to learn. However, as with many static tools, it can often present false negatives;
cases that should have been caught but were allowed.

* dynamic enforcement tools: Clang's [Thread Sanitizer](http://clang.llvm.org/docs/ThreadSanitizer.html) (aka TSAN)
is a powerful example of dynamic tools: it changes the build and execution of your program to add bookkeeping on memory access,
absolutely identifying data races in a given execution of your binary.
The cost for this is both memory (5-10x in most cases) and CPU slowdown (2-20x).
Dynamic tools like this are best when applied to integration tests, canary pushes, or unittests that operate on multiple threads.
Workload matters: When TSAN identifies a problem, it is effectively always an actual data race,
but it can only identify races seen in a given execution.

##### Enforcement

It is up to an application builder to choose which support tools are valuable for a particular applications.

## <a name="SScp-con"></a>CP.con: 동시성

이 Section은 다수의 스레드들이 공유 데이터를 사용해 통신하는 부분에 대해 다룹니다.

* 병렬 알고리즘은 [parallelism](#SScp-par)를 참조하세요
* 명시적인 공유없는 작업간 통신방법은 [messaging](#SScp-mess)을 참조하세요
* For vector parallel code, see [vectorization](#SScp-vec)
* 무잠금 프로그래밍은 [lock free](#SScp-free)를 참조하세요

동시성 규칙 요약:

* [CP.20: `lock()`/`unlock()` 대신 RAII를 사용하라](#Rconc-raii)
* [CP.21: Use `std::lock()` or `std::scoped_lock` to acquire multiple `mutex`es](#Rconc-lock)
* [CP.22: lock을 사용 중일때는 알 수 없는 코드를 호출하지 말아라(e.g., a callback)](#Rconc-unknown)
* [CP.23: join하는 `thread`를 유효범위 안의 컨테이너처럼 생각하라](#Rconc-join)
* [CP.24: `thread`를 전역 컨테이너처럼 생각하라](#Rconc-detach)
* [CP.25: Prefer `gsl::joining_thread` over `std::thread`](#Rconc-joining_thread)
* [CP.26: Don't `detach()` a thread](#Rconc-detached_thread)
* [CP.31: 스레드들 간의 작은 데이터 전달은 참조나 포인터보다는 값으로 전달하라](#Rconc-data)
* [CP.32: 관련 없는 `thread`간의 소유권 공유는 `shared_ptr`를 사용하라](#Rconc-shared)
* [CP.40: 문맥 교환을 최소화하라](#Rconc-switch)
* [CP.41: 스레드 생성과 소멸을 최소화하라](#Rconc-create)
* [CP.42: 조건 없이 `wait`하지 말아라](#Rconc-wait)
* [CP.43: 임계 영역(Critical Section)에서의 시간을 최소화하라](#Rconc-time)
* [CP.44: `lock_guard`과 `unique_lock`에는 이름을 붙여라](#Rconc-name)
* [CP.50: Define a `mutex` together with the data it guards. Use `synchronized_value<T>` where possible](#Rconc-mutex)
* ??? 언제 스핀락(spinlock)을 사용하는가
* ??? 언제 `try_lock()`을 사용하는가
* ??? 언제 `unique_lock`보다 `lock_guard`를 쓰는가
* ??? Time multiplexing
* ??? 언제/어떻게 `new thread`를 사용하는가

### <a name="Rconc-raii"></a>CP.20: `lock()`/`unlock()` 대신 RAII를 사용하라

##### Reason

잠금을 해제하지 않음으로 인한 오류를 예방한다.

##### Example, bad

```c++
    mutex mtx;

    void do_stuff()
    {
        mtx.lock();
        // ... do stuff ...
        mtx.unlock();
    }
```

얼마 후 또는 나중에, 누군가 `mtx.unlock()`을 잊어버린다. 그리곤 `return`을 `... do stuff ...`에 붙이거나, 예외나 무언가를 던진다.

```c++
    mutex mtx;

    void do_stuff()
    {
        unique_lock<mutex> lck {mtx};
        // ... do stuff ...
    }
```

##### Enforcement
 `lock()`과 `unlock()`의 호출에 표식을 남겨라  ???

### <a name="Rconc-lock"></a>CP.21: Use `std::lock()` or `std::scoped_lock` to acquire multiple `mutex`es

##### Reason

To avoid deadlocks on multiple `mutex`es.

##### Example

This is asking for deadlock:

```c++
    // thread 1
    lock_guard<mutex> lck1(m1);
    lock_guard<mutex> lck2(m2);

    // thread 2
    lock_guard<mutex> lck2(m2);
    lock_guard<mutex> lck1(m1);
```

Instead, use `lock()`:

```c++
    // thread 1
    lock(m1, m2);
    lock_guard<mutex> lck1(m1, adopt_lock);
    lock_guard<mutex> lck2(m2, adopt_lock);

    // thread 2
    lock(m2, m1);
    lock_guard<mutex> lck2(m2, adopt_lock);
    lock_guard<mutex> lck1(m1, adopt_lock);
```

or (better, but C++17 only):

```c++
    // thread 1
    scoped_lock<mutex, mutex> lck1(m1, m2);

    // thread 2
    scoped_lock<mutex, mutex> lck2(m2, m1);
```

Here, the writers of `thread1` and `thread2` are still not agreeing on the order of the `mutex`es, but order no longer matters.

##### Note

In real code, `mutex`es are rarely named to conveniently remind the programmer of an intended relation and intended order of acquisition.
In real code, `mutex`es are not always conveniently acquired on consecutive lines.

In C++17 it's possible to write plain

```c++
    lock_guard lck1(m1, adopt_lock);
```

and have the `mutex` type deduced.

##### Enforcement

Detect the acquisition of multiple `mutex`es.
This is undecidable in general, but catching common simple examples (like the one above) is easy.

### <a name="Rconc-unknown"></a>CP.22: Never call unknown code while holding a lock (e.g., a callback)

##### Reason

If you don't know what a piece of code does, you are risking deadlock.

##### Example

```c++
    void do_this(Foo* p)
    {
        lock_guard<mutex> lck {my_mutex};
        // ... do something ...
        p->act(my_data);
        // ...
    }
```

If you don't know what `Foo::act` does (maybe it is a virtual function invoking a derived class member of a class not yet written),
it may call `do_this` (recursively) and cause a deadlock on `my_mutex`.
Maybe it will lock on a different mutex and not return in a reasonable time, causing delays to any code calling `do_this`.

##### Example

A common example of the "calling unknown code" problem is a call to a function that tries to gain locked access to the same object.
Such problem can often be solved by using a `recursive_mutex`. For example:

```c++
    recursive_mutex my_mutex;

    template<typename Action>
    void do_something(Action f)
    {
        unique_lock<recursive_mutex> lck {my_mutex};
        // ... do something ...
        f(this);    // f will do something to *this
        // ...
    }
```

If, as it is likely, `f()` invokes operations on `*this`, we must make sure that the object's invariant holds before the call.

##### Enforcement

* Flag calling a virtual function with a non-recursive `mutex` held
* Flag calling a callback with a non-recursive `mutex` held

### <a name="Rconc-join"></a>CP.23: Think of a joining `thread` as a scoped container

##### Reason

To maintain pointer safety and avoid leaks, we need to consider what pointers are used by a `thread`.
If a `thread` joins, we can safely pass pointers to objects in the scope of the `thread` and its enclosing scopes.

##### Example

```c++
    void f(int* p)
    {
        // ...
        *p = 99;
        // ...
    }
    int glob = 33;

    void some_fct(int* p)
    {
        int x = 77;
        joining_thread t0(f, &x);           // OK
        joining_thread t1(f, p);            // OK
        joining_thread t2(f, &glob);        // OK
        auto q = make_unique<int>(99);
        joining_thread t3(f, q.get());      // OK
        // ...
    }
```

A `gsl::joining_thread` is a `std::thread` with a destructor that joins and that cannot be `detached()`.
By "OK" we mean that the object will be in scope ("live") for as long as a `thread` can use the pointer to it.
The fact that `thread`s run concurrently doesn't affect the lifetime or ownership issues here;
these `thread`s can be seen as just a function object called from `some_fct`.

##### Enforcement

Ensure that `joining_thread`s don't `detach()`.
After that, the usual lifetime and ownership (for local objects) enforcement applies.

### <a name="Rconc-detach"></a>CP.24: `thread`를 전역 컨테이너처럼 생각하라

##### Reason

포인터를 안전하게 남겨두고 누수(leak)을 방지하기 위해선, 어떤 포인터들이 `thread`에 의해서 사용되는지 고려해야 한다.

만약 `thread`가 detach되었다면, 정적 객체 또는 자유 영역에 있는 객체들에 대한 포인터만 안전하게 전달할 수 있다.

##### Example

```c++
    void f(int* p)
    {
        // ...
        *p = 99;
        // ...
    }

    int glob = 33;

    void some_fct(int* p)
    {
        int x = 77;
        std::thread t0(f, &x);           // bad
        std::thread t1(f, p);            // bad
        std::thread t2(f, &glob);        // OK
        auto q = make_unique<int>(99);
        std::thread t3(f, q.get());      // bad
        // ...
        t0.detach();
        t1.detach();
        t2.detach();
        t3.detach();
        // ...
    }
```

By "OK" we mean that the object will be in scope ("live") for as long as a `thread` can use the pointers to it.
By "bad" we mean that a `thread` may use a pointer after the pointed-to object is destroyed.
The fact that `thread`s run concurrently doesn't affect the lifetime or ownership issues here;
these `thread`s can be seen as just a function object called from `some_fct`.

##### Note

Even objects with static storage duration can be problematic if used from detached threads: if the
thread continues until the end of the program, it might be running concurrently with the destruction
of objects with static storage duration, and thus accesses to such objects might race.

##### Note

This rule is redundant if you [don't `detach()`](#Rconc-detached_thread) and [use `gsl::joining_thread`](#Rconc-joining_thread).
However, converting code to follow those guidelines could be difficult and even impossible for third-party libraries.
In such cases, the rule becomes essential for lifetime safety and type safety.

In general, it is undecidable whether a `detach()` is executed for a `thread`, but simple common cases are easily detected.
If we cannot prove that a `thread` does not `detach()`, we must assume that it does and that it outlives the scope in which it was constructed;
After that, the usual lifetime and ownership (for global objects) enforcement applies.

##### Enforcement

Flag attempts to pass local variables to a thread that might `detach()`.

### <a name="Rconc-joining_thread"></a>CP.25: Prefer `gsl::joining_thread` over `std::thread`

##### Reason

`raii_thread`는 유효 범위가 끝날때 join한다.

Detach한 스레드들은 관찰(monitor)하기가 어렵다. 
(잠재적으로 detach할 스레드를 포함해서) 이 스레드들에서 오류가 없다고 확신하기 어렵다.  

##### Example, bad

```c++
    void f() { std::cout << "Hello "; }

    struct F {
        void operator()() { std::cout << "parallel world "; }
    };

    int main()
    {
        std::thread t1{f};      // f() executes in separate thread
        std::thread t2{F()};    // F()() executes in separate thread
    }  // spot the bugs
```

##### Example

```c++
    void f() { std::cout << "Hello "; }

    struct F {
        void operator()() { std::cout << "parallel world "; }
    };

    int main()
    {
        std::thread t1{f};      // f() executes in separate thread
        std::thread t2{F()};    // F()() executes in separate thread

        t1.join();
        t2.join();
    }  // one bad bug left
```

##### Example, bad

The code determining whether to `join()` or `detach()` may be complicated and even decided in the thread of functions called from it or functions called by the function that creates a thread:

```c++
    void tricky(thread* t, int n)
    {
        // ...
        if (is_odd(n))
            t->detach();
        // ...
    }

    void use(int n)
    {
        thread t { tricky, this, n };
        // ...
        // ... should I join here? ...
    }
```

This seriously complicates lifetime analysis, and in not too unlikely cases makes lifetime analysis impossible.
This implies that we cannot safely refer to local objects in `use()` from the thread or refer to local objects in the thread from `use()`.

##### Note

Make "immortal threads" globals, put them in an enclosing scope, or put them on the free store rather than `detach()`.
[don't `detach`](#Rconc-detached_thread).

##### Note

Because of old code and third party libraries using `std::thread` this rule can be hard to introduce.

##### Enforcement

Flag uses of `std::thread`:

* Suggest use of `gsl::joining_thread`.
* Suggest ["exporting ownership"](#Rconc-detached_thread) to an enclosing scope if it detaches.
* Seriously warn if it is not obvious whether if joins of detaches.

### <a name="Rconc-detached_thread"></a>CP.26: Don't `detach()` a thread

##### Reason

Often, the need to outlive the scope of its creation is inherent in the `thread`s task,
but implementing that idea by `detach` makes it harder to monitor and communicate with the detached thread.
In particular, it is harder (though not impossible) to ensure that the thread completed as expected or lives for as long as expected.

##### Example

```c++
    void heartbeat();

    void use()
    {
        std::thread t(heartbeat);             // don't join; heartbeat is meant to run forever
        t.detach();
        // ...
    }
```

This is a reasonable use of a thread, for which `detach()` is commonly used.
There are problems, though.
How do we monitor the detached thread to see if it is alive?
Something might go wrong with the heartbeat, and losing a heartbeat can be very serious in a system for which it is needed.
So, we need to communicate with the heartbeat thread
(e.g., through a stream of messages or notification events using a `condition_variable`).

An alternative, and usually superior solution is to control its lifetime by placing it in a scope outside its point of creation (or activation).
For example:

```c++
    void heartbeat();

    gsl::joining_thread t(heartbeat);             // heartbeat is meant to run "forever"
```

This heartbeat will (barring error, hardware problems, etc.) run for as long as the program does.

Sometimes, we need to separate the point of creation from the point of ownership:

```c++
    void heartbeat();

    unique_ptr<gsl::joining_thread> tick_tock {nullptr};

    void use()
    {
        // heartbeat is meant to run as long as tick_tock lives
        tick_tock = make_unique<gsl::joining_thread>(heartbeat);
        // ...
    }
```

#### Enforcement

Flag `detach()`.

### <a name="Rconc-data-by-value"></a>CP.31: Pass small amounts of data between threads by value, rather than by reference or pointer

##### Reason

Copying a small amount of data is cheaper to copy and access than to share it using some locking mechanism.
Copying naturally gives unique ownership (simplifies code) and eliminates the possibility of data races.

##### Note

Defining "small amount" precisely is impossible.

##### Example

```c++
    string modify1(string);
    void modify2(string&);

    void fct(string& s)
    {
        auto res = async(modify1, s);
        async(modify2, s);
    }
```

The call of `modify1` involves copying two `string` values; the call of `modify2` does not.
On the other hand, the implementation of `modify1` is exactly as we would have written it for single-threaded code,
whereas the implementation of `modify2` will need some form of locking to avoid data races.
If the string is short (say 10 characters), the call of `modify1` can be surprisingly fast;
essentially all the cost is in the `thread` switch. If the string is long (say 1,000,000 characters), copying it twice
is probably not a good idea.

Note that this argument has nothing to do with `async` as such. It applies equally to considerations about whether to use
message passing or shared memory.

##### Enforcement

???

### <a name="Rconc-shared"></a>CP.32: 관련 없는 `thread`간의 소유권 공유는 `shared_ptr`를 사용하라

##### Reason
만약 스레드들이 서로 무관하다면 (달리 말해, 같은 유효범위 안에 없거나 한 스레드가 다른 스레드의 생애를 포함하지 않는 경우),
그리고 소멸되어야 하는 자유 영역 메모리를 공유할 필요가 있다면, `shared_ptr`또는 동등한 것이 소멸을 보장할 수 있다.

##### Example

    ???

##### Note

* 정적(예컨대 전역) 객체는 한 스레드가 객체의 소멸에 책임이 있다는 점에서 공유할 수 있다
* 소멸되지 않는 자유 영역의 객체는 공유할 수 있다.
* 한 스레드가 객체를 소유하고, 다른 스레드가 소유자 스레드보다 오래 살아남지 않으면 객체를 안전하게 공유할 수 있다.

##### Enforcement

???

### <a name="Rconc-switch"></a>CP.40: 문맥 교환을 최소화하라

##### Reason
문맥 교환(Context switches)은 매우 높은 비용을 필요로 한다.

##### Example

    ???

##### Enforcement

???

### <a name="Rconc-create"></a>CP.41: 스레드 생성과 소멸을 최소화하라

##### Reason

스레드 생성은 매우 높은 비용을 필요로 한다.

##### Example

```c++
    void worker(Message m)
    {
        // process
    }

    void master(istream& is)
    {
        for (Message m; is >> m; )
            run_list.push_back(new thread(worker, m));
    }
```

이 코드는 메세지마다 `thread`를 생성한다. 그리고 `run_list`는 아마도 작업들이 끝나면 스레드를 파괴할 것이다.

대신, 미리 만들어둔 작업자(worker) 스레드들이 메세지를 처리하도록 할 수 있다.

```c++
    Sync_queue<Message> work;

    void master(istream& is)
    {
        for (Message m; is >> m; )
            work.put(m);
    }

    void worker()
    {
        for (Message m; m = work.get(); ) {
            // process
        }
    }

    void workers()  // set up worker threads (specifically 4 worker threads)
    {
        joining_thread w1 {worker};
        joining_thread w2 {worker};
        joining_thread w3 {worker};
        joining_thread w4 {worker};
    }
```

##### Note

시스템에서 잘 만든 스레드 풀을 지원한다면, 그것을 사용하라.

시스템에서 잘 만든 메세지 큐를 지원하면, 그것을 사용하라.

##### Enforcement

???

### <a name="Rconc-wait"></a>CP.42: 조건 없이 `wait`하지 말아라

##### Reason

조건을 주지 않고 `wait`하는 것은 작업을 위해 깨어나는 것(wakeup) 또는 단순히 작업이 없다는 것을 확인하기 위해 깨어나는 것을 놓치게 한다.

##### Example, bad

```c++
    std::condition_variable cv;
    std::mutex mx;

    void thread1()
    {
        while (true) {
            // do some work ...
            std::unique_lock<std::mutex> lock(mx);
            cv.notify_one();    // wake other thread
        }
    }

    void thread2()
    {
        while (true) {
            std::unique_lock<std::mutex> lock(mx);
            cv.wait(lock);    // might block forever
            // do work ...
        }
    }
```

여기서, 코드에 표현되지 않은 다른 `thread`가 다른 `thread1`의 알림(notification)을 소비하면, `thread2`는 영원히 대기할 수도 있다.

##### Example

```c++
    template<typename T>
    class Sync_queue {
    public:
        void put(const T& val);
        void put(T&& val);
        void get(T& val);
    private:
        mutex mtx;
        condition_variable cond;    // this controls access
        list<T> q;
    };

    template<typename T>
    void Sync_queue<T>::put(const T& val)
    {
        lock_guard<mutex> lck(mtx);
        q.push_back(val);
        cond.notify_one();
    }

    template<typename T>
    void Sync_queue<T>::get(T& val)
    {
        unique_lock<mutex> lck(mtx);
        cond.wait(lck, [this]{ return !q.empty(); });    // prevent spurious wakeup
        val = q.front();
        q.pop_front();
    }
```

이제 만약 큐가 비어있는 상태에서 `get()`을 실행하는 스레드가 깨어나게 된다면(예를 들면 다른 스레드가 먼저 `get()`을 실행해서 가져갔다거나), 
해당 스레드는 그대로 다시 sleep 상태가 될 것이다.

##### Enforcement

조건이 없는 모든 `waits`를 지적하라.

### <a name="Rconc-time"></a>CP.43: 임계 영역(Critical Section)에서의 시간을 최소화하라

##### Reason

`mutex`를 가진 시간이 짧을 수록, 다른 `thread`가 대기해야 하는 경우가 줄어들 것이다. 
그리고 `thread`의 중지(suspection)와 재실행(resumption)은 많은 비용을 필요로 한다.

##### Example

```c++
    void do_something() // bad
    {
        unique_lock<mutex> lck(my_lock);
        do0();  // preparation: does not need lock
        do1();  // transaction: needs locking
        do2();  // cleanup: does not need locking
    }
```

이 코드에선 필요 이상으로 오랫동안 잠금을 소유하고 있다:

필요하지 않다면 잠금을 가져서는 안되며, 정리를 시작하기 전에 잠금을 해제해야 한다.  
이렇게 다시 작성할 수 있을 것이다.

```c++
    void do_something() // bad
    {
        do0();  // preparation: does not need lock
        my_lock.lock();
        do1();  // transaction: needs locking
        my_lock.unlock();
        do2();  // cleanup: does not need locking
    }
```

하지만 이러한 코드는 안전성에 대해서 타협할 뿐이고, [RAII를 사용하라](#Rconc-raii) 규칙을 위반한다.

대신, 임계영역에 유효범위 블록을 추가하라:

```c++
    void do_something() // OK
    {
        do0();  // preparation: does not need lock
        {
            unique_lock<mutex> lck(my_lock);
            do1();  // transaction: needs locking
        }
        do2();  // cleanup: does not need locking
    }
```

##### Enforcement

일반적으로 불가능하다.
RAII를 적용하지 않은 `lock()`과 `unlock()`을 지적하라.

### <a name="Rconc-name"></a>CP.44: `lock_guard`과 `unique_lock`에는 이름을 붙여라

##### Reason

이름이 없는 지역 개체들은 일시적으로 생성되고 즉시 소멸된다

##### Example

```c++
    unique_lock<mutex>(m1);
    lock_guard<mutex> {m2};
    lock(m1, m2);
```

이 코드는 별로 문제 없어 보이지만, 그렇지 않다.

##### Enforcement

이름을 부여하지 않은 `lock_guard`와 `unique_lock`을 지적하라

### <a name="Rconc-mutex"></a>CP.50: Define a `mutex` together with the data it guards. Use `synchronized_value<T>` where possible

##### Reason

It should be obvious to a reader that the data is to be guarded and how. This decreases the chance of the wrong mutex being locked, or the mutex not being locked.

Using a `synchronized_value<T>` ensures that the data has a mutex, and the right mutex is locked when the data is accessed.
See the [WG21 proposal](http://wg21.link/p0290)) to add `synchronized_value` to a future TS or revision of the C++ standard.

##### Example

```c++
    struct Record {
        std::mutex m;   // take this mutex before accessing other members
        // ...
    };

    class MyClass {
        struct DataRecord {
           // ...
        };
        synchronized_value<DataRecord> data; // Protect the data with a mutex
    };
```

##### Enforcement

??? Possible?

## <a name="SScp-par"></a>CP.par: 병렬성(Parallelism)

여기서 병렬성(parallelism)은 많은 데이터에 대해서 동시에 한 작업을 병렬적으로 수행하는 것을 의미한다.

병렬성 규칙 요약:

* ???
* ???
* 적합하다고 판단되면, 표준 라이브러리의 병렬 알고리즘들을 사용하라.
* 병렬성을 위해 설계된 알고리즘들을 사용하라. 불필요한 의존성을 지닌 알고리즘을 사용하지 말아라.

## <a name="SScp-mess"></a>CP.mess: 메세지 전달(Message passing)

표준 라이브러리에서 제공하는 기능들은 저수준에 속한다. 즉, `thread`, `mutex`, `atomic`을 사용한 하드웨어와 가까운 프로그래밍에 초점을 두고 있다. 
대부분의 사람들은 이 레벨에서 작업해선 안된다: 에러를 만들기 쉽고, 개발이 느리다.

가능하다면 메세징 라이브러리, 병렬 알고리즘, 그리고 벡터화같은 상위 레벨의 기능을 사용하라.
이 영역에선 프로그래머가 명시적으로 동기화 할 필요가 없는 메세지 전달에 대해 살펴본다.

메세지 전달 규칙 요약:

* [CP.60: 동시적인 작업으로부터 반환값을 받는데 `future`를 사용하라](#Rconc-future)
* [CP.61: 동시적인 작업을 생성하기 위해선 `async()`를 사용하라](#Rconc-async)
* 메세지 큐
* 메세징 라이브러리들

???? should there be a "use X rather than `std::async`" where X is something that would use a better specified thread pool?

??? Is `std::async` worth using in light of future (and even existing, as libraries) parallelism facilities? What should the guidelines recommend if someone wants to parallelize, e.g., `std::accumulate` (with the additional precondition of commutativity), or merge sort?

### <a name="Rconc-future"></a>CP.60: 동시적인 작업으로부터 반환값을 받는데 `future`를 사용하라

##### Reason

`future`는 비동기 작업에 대한 일반적인 함수 호출-반환 문맥을 남겨두는데 사용된다. 명시적인 잠금이 없고 정확한 반환값과 에러(예외) 반환 모두 간단히 처리된다.

##### Example

    ???

##### Note

???

##### Enforcement

???

### <a name="Rconc-async"></a>CP.61: 동시적인 작업을 생성하기 위해선 `async()`를 사용하라

##### Reason

(원문 오기로 인한 내용 없음)

##### Example

    ???

##### Note

불행하게도 `async()`는 완벽하지 않다.
예를 들면, 스레드의 생성을 최소화하기 위해 스레드 풀이 사용된다는 보장을 하지 않는다. 실제로, 대부분의 동시적인 `async()`들은 스레드 풀을 구현하고 있지 않다.

그렇지만, `async()`는 단순하고 논리적으로 정확하다. 더 나은 무언가가 나오거나 많은 비동기적인 작업들을 최적화 해야만 하지 않다면, `async()`를 계속 써도 무방하다.

##### Enforcement

???

## <a name="SScp-vec"></a>CP.vec: 벡터화(Vectorization)

벡터화란 명시적인 동기화를 사용하지 않고 몇개의 작업들을 동시적으로 수행하는 기술을 의미한다.
연산은 단순하게 자료구조(벡터, 배열 등등)의 원소들에 병렬적으로 작용된다.
벡터화는 비 지역적 변화를 필요로 하지 않는다는 점에서 흥미로운 속성을 지닌다. 하지만, 벡터화는 단순한 자료구조들과 병렬적 연산을 가능하게 하는 특별한 알고리즘이 함께 써야 최적의 성능을 보인다.

벡터화 규칙 요약:

* ???
* ???

## <a name="SScp-free"></a>CP.free: 무잠금(Lock-free) 프로그래밍

`mutex`와 `condition_variable`를 사용한 동기화는 상대적으로 높은 비용을 지불해야 할 수 있다. 나아가서, 데드락(deadlock)으로 이어질 가능성 또한 존재한다.
때때로, 성능과 데드락의 가능성을 없애기 위해선, 까다로운 저-수준의 "무잠금" 기능들을 사용해야 한다. 
이 기능들은 일시적으로 배타적인(원자적인) 메모리 접근을 사용한다.  
Lock-free 프로그래밍은 `thread`, `mutex`와 같은 상위레벨의 동시성 메커니즘을 구현하는데 사용되기도 한다.  

Lock-free 프로그래밍 규칙 요약:

* [CP.100: 정말 필요할 때만 lock-free 프로그래밍을 사용하라](#Rconc-lockfree)
* [CP.101: 하드웨어/컴파일러 조합을 불신하라](#Rconc-distrust)
* [CP.102: 문헌을 세심하가 공부하라](#Rconc-literature)
* 언제/어떻게 atomic을 사용할 것인가
* 기아상태를 회피하라
* 정교하게 조작한 lock-free 메모리 접근보다는 lock-free 자료구조를 사용하라
* [CP.110: Do not write your own double-checked locking for initialization](#Rconc-double)
* [CP.111: Use a conventional pattern if you really need double-checked locking](#Rconc-double-pattern)
* 언제/어떻게 CAS(Compare And Swap) 연산을 사용하는가

### <a name="Rconc-lockfree"></a>CP.100: 정말 필요할 때만 lock-free 프로그래밍을 사용하라

##### Reason

lock-free 프로그래밍은 에러를 발생시키기 쉽고 언어 기능, 머신 아키텍처, 자료구조에 대한 전문가 수준의 지식을 필요로 한다.

##### Example, bad

```c++
    // 공유되는 연결리스트의 헤드(시작점)
    extern atomic<Link*> head;

    Link* nh = new Link(data, nullptr);    // 링크에 삽입할 준비를 한다
    Link* h = head.load();                 // 리스트의 공유 head를 읽는다

    do {
        // 조건을 충족하면, 다른 곳에 삽입한다.
        if (h->data <= data) break;
        // 다음 원소는 이전의 head이다
        nh->next = h;
    } while (!head.compare_exchange_weak(h, nh));
    // nh를 h 또는 head에 삽입한다.
```

버그를 찾아보라.  
테스팅을 통해서 찾기 정말정말 어려울 것이다.  
ABA 문제에 대해서 읽어보라.

##### Exception

기본적으로 적용되는 순차 무모순 메모리 모델 (`memory_order_seq_cst`)을 사용하는 한, [원자적 변수들](#???) 단순성과 안전성을 위해 사용될 수 있다.

##### Note

상위 레벨 동시성 메커니즘들은 `thread`와 `mutex` 같은 lock-free 프로그래밍으로 구현되었다.

**Alternative**: 다른 사람들에 의해서 구현되었거나 라이브러리의 일부로 존재하는 lock-free 자료구조를 사용하라.

### <a name="Rconc-distrust"></a>CP.101: 하드웨어/컴파일러 조합을 불신하라

##### Reason

lock-free프로그래밍에 사용되는 저-수준 하드웨어 인터페이스는 가장 구현하기 어렵고 미묘한 이식성 문제가 발생할 수 있다.  
성능을 위해 lock-free 프로그래밍을 사용하고 있다면, 오히려 성능 저하(regression)가 있지는 않은지 확인해야 한다.

##### Note

(정적/동적) 명령어 재배치(Instruction Reordering) 때문에 이 레벨에서는 효율적으로 생각하는 것이 어렵다 (특히 relaxed 메모리 모델을 사용하고 있다면).
경험과, (준)형식적인 lock-free 모델들, 그런 모델들에 대한 점검이 유용할 수 있다.
테스팅 - 종종 엄청나게 넓은 영역에 대한 - 은 필수적이다.
"Don't fly too close to the wind."

##### Enforcement

하드웨어, 운영체제, 컴파일러, 그리고 라이브러리의 모든 변화를 포함하는 강력한 테스팅 규칙을 세워라.

### <a name="Rconc-literature"></a>CP.102: 문헌을 세심하게 공부하라

##### Reason

atomic에서의 예외와 표준적인 패턴들, lock-free 프로그래밍은 온전히 전문가를 위한 내용이다. 
lock-free 코드를 적용하기에 앞서서 전문가가 되어라.

##### References

* Anthony Williams: C++ concurrency in action. Manning Publications.
* Boehm, Adve, You Don't Know Jack About Shared Variables or Memory Models , Communications of the ACM, Feb 2012.
* Boehm, "Threads Basics", HPL TR 2009-259.
* Adve, Boehm, "Memory Models: A Case for Rethinking Parallel Languages and Hardware", Communications of the ACM, August 2010.
* Boehm, Adve, "Foundations of the C++ Concurrency Memory Model", PLDI 08.
* Mark Batty, Scott Owens, Susmit Sarkar, Peter Sewell, and Tjark Weber, "Mathematizing C++ Concurrency", POPL 2011.
* Damian Dechev, Peter Pirkelbauer, and Bjarne Stroustrup: Understanding and Effectively Preventing the ABA Problem in Descriptor-based Lock-free Designs. 13th IEEE Computer Society ISORC 2010 Symposium. May 2010.
* Damian Dechev and Bjarne Stroustrup: Scalable Non-blocking Concurrent Objects for Mission Critical Code. ACM OOPSLA'09. October 2009
* Damian Dechev, Peter Pirkelbauer, Nicolas Rouquette, and Bjarne Stroustrup: Semantically Enhanced Containers for Concurrent Real-Time Systems. Proc. 16th Annual IEEE International Conference and Workshop on the Engineering of Computer Based Systems (IEEE ECBS). April 2009.

### <a name="Rconc-double"></a>CP.110: Do not write your own double-checked locking for initialization

##### Reason

Since C++11, static local variables are now initialized in a thread-safe way. When combined with the RAII pattern, static local variables can replace the need for writing your own double-checked locking for initialization. std::call_once can also achieve the same purpose. Use either static local variables of C++11 or std::call_once instead of writing your own double-checked locking for initialization.

##### Example

Example with std::call_once.

```c++
    void f()
    {
        static std::once_flag my_once_flag;
        std::call_once(my_once_flag, []()
        {
            // do this only once
        });
        // ...
    }
```

Example with thread-safe static local variables of C++11.

```c++
    void f()
    {
        // Assuming the compiler is compliant with C++11
        static My_class my_object; // Constructor called only once
        // ...
    }

    class My_class
    {
    public:
        My_class()
        {
            // do this only once
        }
    };
```

##### Enforcement

??? 저 idiom 확인할 수 있는 건가요?

### <a name="Rconc-double-pattern"></a>CP.111: Use a conventional pattern if you really need double-checked locking

##### Reason

Double-checked locking is easy to mess up. If you really need to write your own double-checked locking, in spite of the rules [CP.110: Do not write your own double-checked locking for initialization](#Rconc-double) and [CP.100: Don't use lock-free programming unless you absolutely have to](#Rconc-lockfree), then do it in a conventional pattern.

The uses of the double-checked locking pattern that are not in violation of [CP.110: Do not write your own double-checked locking for initialization](#Rconc-double) arise when a non-thread-safe action is both hard and rare, and there exists a fast thread-safe test that can be used to guarantee that the action is not needed, but cannot be used to guarantee the converse.

##### Example, bad

The use of volatile does not make the first check thread-safe, see also [CP.200: Use `volatile` only to talk to non-C++ memory](#Rconc-volatile2)

```c++
    mutex action_mutex;
    volatile bool action_needed;

    if (action_needed) {
        std::lock_guard<std::mutex> lock(action_mutex);
        if (action_needed) {
            take_action();
            action_needed = false;
        }
    }
```

##### Example, good

```c++
    mutex action_mutex;
    atomic<bool> action_needed;

    if (action_needed) {
        std::lock_guard<std::mutex> lock(action_mutex);
        if (action_needed) {
            take_action();
            action_needed = false;
        }
    }
```

Fine-tuned memory order may be beneficial where acquire load is more efficient than sequentially-consistent load

```c++
    mutex action_mutex;
    atomic<bool> action_needed;

    if (action_needed.load(memory_order_acquire)) {
        lock_guard<std::mutex> lock(action_mutex);
        if (action_needed.load(memory_order_relaxed)) {
            take_action();
            action_needed.store(false, memory_order_release);
        }
    }
```

##### Enforcement

??? Is it possible to detect the idiom?

## <a name="SScp-etc"></a>CP.etc: 기타 동시성 규칙들

이 규칙들은 분류를 적용하지 않는다:

* [CP.200:  `volatile`은 C++가 아닌 메모리에 대해서만 사용하라](#Rconc-volatile2)
* [CP.201: ??? 시그널](#Rconc-signal)

### <a name="Rconc-volatile2"></a>CP.200: `volatile`은 C++가 아닌 메모리에 대해서만 사용하라

##### Reason

`volatile`은 "C++가 아닌" 코드 또는 C++ 메모리 모델을 따르지 않는 하드웨어에 있는 객체를 참조하기 위해 사용된다.

##### Example

```c++
    const volatile long clock;
```

이 코드는 시계 회로가 지속적으로 업데이트하는 레지스터를 의미한다.
`clock`은 `volatile`인데, 이는 C++ 프로그램 내에서는 아무것도 하지 않았지만 값이 바뀔 수 있기 때문이다.
예컨대, `clock`을 두번 읽는 것은 다른 값을 반환할 수도 있다, 때문에 최적화기(optimizer)는 반복된 읽기 과정을 최적화해서 없애지 않는 것이 나을 것이다:

```c++
    long t1 = clock;
    // ... no use of clock here ...
    long t2 = clock;
```

`clock`은 프로그램에서 값을 쓰려고 하면 안되기 때문에 `const`로 사용된다.

##### Note

하드웨어를 직접 조작하는 최저 레벨의 코드를 작성하는 게 아니라면, `volatile`는 피하는게 좋은 난해한(esoteric) 기능이라고 생각하라.

##### Example

일반적으로 C++ 코드는 다른 어딘가(하드웨어 또는 다른 언어)에 속한 메모리를 가져올 때  `volatile`을 사용한다:

```c++
    int volatile* vi = get_hardware_memory_location();
        // note: we get a pointer to someone else's memory here
        // volatile says "treat this with extra respect"
```

때때로 C++ 코드는  `volatile` 메모리를 할당하고 의도적으로 포인터를 다른 어딘가(하드웨어 또는 다른언어)와 공유하기도 한다:

```c++
    static volatile long vl;
    please_use_this(&vl);   // escape a reference to this to "elsewhere" (not C++)
```

##### Example; bad

`volatile` 지역변수들은 거의 대부분의 경우 잘못 사용된 것이다 --  local variables are nearly always wrong -- 그 변수들이 일시적(emphemeral)이라면 어떻게 다른 언어/기계들과 공유될 수 있겠는가?

멤버 변수들의 경우도 같은 이유로 `volatile`의 잘못된 사용일 수 있다.

```c++
    void f() {
        volatile int i = 0; // bad, volatile local variable
        // etc.
    }

    class My_type {
        volatile int i = 0; // suspicious, volatile member variable
        // etc.
    };
```

##### Note

C++ 에서는, 다른 언어와는 달리, `volatile`은 동기화와 관련해 [아무것도 하지 않는다](#Rconc-volatile).

##### Enforcement

* `volatile T` 을 사용하는 지역, 멤버 변수들을 지적하라; 대부분의 경우  `atomic<T>` 를 의도했을 것이다.
* ???

### <a name="Rconc-signal"></a>CP.201: ??? 시그널

???UNIX signal handling???. May be worth reminding how little is async-signal-safe, and how to communicate with a signal handler (best is probably "not at all")
