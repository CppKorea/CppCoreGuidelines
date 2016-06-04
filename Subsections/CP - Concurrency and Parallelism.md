# <a name="S-concurrency"></a>CP: 동시성과 병렬성
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#S-concurrency)    
> 번역 기준문서 : 16/05/26 일자

종종 우리 컴퓨터가 동시에 많은 작업들을 해주길 (최소한 그렇게 보이기를) 원할 때가 있죠.
이유는 다양합니다. (예를 들면, 단일 프로세서만 쓰면서 여러 이벤트를 기다리기를 원하거나, 동시에 다수의 데이터 스트림을 처리하길 원하거나, 또는 하드웨어 기능들을 사용하길 원할 수도 있겠죠.) 
동시성과 병렬성을 표현하기 위한 기본 기능들 또한 그러합니다.

여기서는, ISO 표준 C++에서 제공하는 기본적인 동시성과 병렬성을 위한 기능에 관한 몇몇 일반적인 원칙과 규칙에 대해서 기술합니다. 

스레드는 머신에서 지원하는 동시적이고 병렬적인 프로그래밍을 위한 핵심 기능(The core machine support)입니다. 
스레드들을 같은 메모리를 공유하면서도 다수의 프로그램 instance들을 독립적으로 실행할 수 있도록 합니다.  

동시적인 프로그래밍은 많은 이유로 인해 까다롭습니다, 대표적으로 한 스레드에서 데이터를 읽는 동안 다른 스레드에서 같은 데이터를 덮어쓸 때 발생하는 비정의 동작(undefined behavior)이 있죠. 이 스레드들 사이에 적절한 동기화(synchronization)가 없다면 말입니다.

`std::async` 또는 `std::thread`를 전략적으로 추가하면 이미 존재하는 단일 스레드 코드를 간단하게 병렬적으로 실행할 수 있습니다. 또는 코드 전체를 완전히 다시 작성해야 할 수도 있죠. 이는 원래의 코드가 스레드에 적합하게(thread-friendly) 작성되었느냐에 따라 다릅니다.
  

이 문서의 동시성/병렬성 규칙들은 3가지 목표를 가지고 설계되었습니다.
* To help you write code that is amenable to being used in a threaded
  environment
* To show clean, safe ways to use the threading primitives offered by the
  standard library
* To offer guidance on what to do when concurrency and parallelism aren't giving
  you the performance gains you need

C++에서의 동시성은 아직 진행 중이라는 것을 기억하길 바랍니다.
C++11에선 많은 핵심적인 동시성 연산들이 소개되었고, C++14에서는 개선이 이루어졌습니다. 그리고 C++를 사용한 동시적인 프로그램들을 작성하는 것에 대한 관심도 늘어났죠. 
We expect some of the library-related guidance here to change significantly over time.

이 부분은 많은 작업이 필요합니다. 우리가 비 전문가를 위한 규칙들로 시작한다는 점을 유의해주십시오. 당신이 전문가라면, 조금 더 기다리셔야 할겁니다; 이 문서에 기여하는 것도 좋구요. 하지만 정확하고, 빠른 동시적인 프로그램을 작성하길 원하는 대부분의 프로그래머들을 생각해 주시기 바랍니다.


동시성과 병렬성 규칙 요약:  

* [CP.1: 코드가 멀티스레드 프로그램의 일부로 동작할 것이라 가정하라](#Rconc-multi)
* [CP.2: 데이터 경쟁을 피하라](#Rconc-races)
* [CP.3: 쓰기 가능한 데이터의 명시적인 공유를 최소화하라](#Rconc-data)
* [CP.4: 스레드보단 작업 단위로 생각하라](#Rconc-task)
* [CP.8: 동기화를 위해 `volatile`을 사용하지 말아라](#Rconc-volatile)

하위 영역:  

* [CP.con: 동시성(Concurrency)](#SScp-con)
* [CP.par: 병렬성(Parallelism)](#SScp-par)
* [CP.mess: 메세지 전달(Message passing)](#SScp-mess)
* [CP.vec: 벡터화(Vectorization)](#SScp-vec)
* [CP.free: 무잠금 프로그래밍(Lock-free programming)](#SScp-free)
* [CP.etc: 기타 동시성 규칙들(Etc. concurrency rules)](#SScp-etc)



### <a name="Rconc-multi"></a>CP.1: 코드가 멀티스레드 프로그램의 일부로 동작할 것이라 가정하라

##### 근거
동시성이 지금, 그리고 미래의 언젠가 사용되지 않을 것이라고 확정하기 어렵다. 코드는 재사용된다.   
스레드를 사용한 라이브러리들은 다른 프로그램의 일부로 사용되기 마련이다. 
이 규칙은 단일 응용 프로그램보다는 라이브러리 코드에 더 시급하게 적용되어야 한다. 하지만, Ctrl C+V 의 마법 덕분에, 코드 조각들은 예상치 않은 곳에서 나타날 수 있다.   

##### 예
```
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
비록 `cached_computation`함수는 단일 스레드 환경에선 완벽하게 동작하지만, 멀티 스레드 환경에선 두 개의 `static` 변수들이 데이터 경쟁으로 이어지고, 비정의 동작을 유발할 것이다.

멀티 스레드 환경에서 이런 예시가 안전하게 만들어질 수 있는 몇가지 방법들이 있다:
* 동시성에 대한 사항을 상위 호출자에게 위임하라
* `static` 변수들을 `thread_local`로 만들어라. (아마 캐싱을 덜 효율적으로 만들 수도 있다) 
* 동시적인 제어를 구현하라, 예컨대, 두 `static`변수를 `static` 잠금으로 보호하라. (성능을 감소시킬 수 있다) 
* 호출자가 캐시로 사용할 메모리를 제공하도록 하라. 그렇게 함으로써 메모리 할당과 동시성에 대한 문제를 호출자에게 위임하라.
* 멀티 스레드 환경에서의 빌드, 실행을 거부하라.
* 두 가지 구현을 지원하라. 단일 스레드 환경을 위한 구현과 멀티 스레드 환경을 위한 것 모두

##### 예외 사항
멀티 스레드 환경에서 절대 실행되지 않을 코드.  
주의: 멀티 스레드 프로그램에서 절대로 돌아가지 않을 줄 "알았던" 코드들이 몇년 뒤엔 멀티 스레드 프로그램의 일부가 된 사례는 많이 있다.   
일반적으로, 그런 프로그램들은 데이터 경쟁을 없애기 위한 고통스러운 노력으로 이어진다. 따라서, 멀티 스레드 환경에서 실행되는 것을 의도하지 않은 코드들은 분명하게 해당 사항이 기술되어야 하고, 이상적으로는 컴파일 또는 실행시간에 버그를 일찍 찾을 수 있는 메커니즘이 함께 사용되되어야 한다.



### <a name="Rconc-races"></a>CP.2: 데이터 경쟁을 피하라

##### 근거
데이터 경쟁이 있으면, 아무것도 보장할 수 없으며 에러들이 남을 것이다.

##### 참고 사항
쉽게 말해서, 만약 두 스레드가 같은 객체를 (동기화 없이) 동시적으로 접근할 수 있다면, 그리고 한 스레드가 쓰기(non-`const` 연산)를 수행한다면, 데이터 경쟁이 있는 것이다.     
어떻게 동기화를 사용하고 데이터 경쟁을 없앨 것인지 더 알고 싶다면 동시성에 대한 좋은 책들을 참고하라.

##### 잘못된 예
데이터 경쟁이 존재하는 예시는 많다. 지금 이 순간 실행 중인 production 소프트웨어들 중에도 존재할 것이다. 간단한 예시를 들자면:
```
    int get_id() {
      static int id = 1;
      return id++;
    }
```
여기서 증가 연산은 데이터 경쟁의 예시다.   
이 연산은 다음과 같이 잘못될 수 있다:
* 스레드 A가 `id`를 로드하고, OS가 A를 중지시킨다. 그 사이 다른 스레드가 ID를 수백 개 생성한다. 그 후에 A는 다시 실행되고, A의 문맥에서 증가한 `id`값이 다시 써지게 된다. 
* 스레드 A와 B가 `id`를 로드하고 동시에 증가시킨다. 그 결과 두 스레드는 같은 ID값을 가진다.

지역 정적 변수들은 데이터 경쟁의 일반적인 원인이다. 

##### 잘못된 예
```
    void f(fstream&  fs, regex pat)
    {
        array<double, max> buf;
        int sz = read_vec(fs, buf, max);      // fs에서 buf로 읽는다
        gsl::span<double> s {buf, max};
        // ...
        auto h1 = async([&]{ sort(par, s); }); // 정렬을 위한 task 생성
        // ...
        
        // 일치하는 원소를 찾기 위한 task 생성
        auto h2 = async([&]{ return find_all(buf, sz, pat); });   
        // ...
    }
```
이 코드에서는, `buf`의 원소들에 대한 데이터 경쟁이 있다. (`sort`가 읽기/쓰기 모두 수행할 것이다).
좋은 데이터 경쟁이란 없다. 이 코드에선 스택의 데이터에 대한 데이터 경쟁이 발생하는데, 모든 데이터 경쟁이 이처럼 찾아내기 쉽지는 않다.

##### 잘못된 예
```
    // 잠금으로 통제하지 않는 코드

    unsigned val;

    if (val < 5) {
        // ... 다른 스레드가 val을 바꿀수도 있다 ...
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

##### 시행하기
최소한 무엇이라도 하라.   
이 문제를 해결하기 위해 상업용 그리고 오픈소스 도구들을 사용하라. 하지만 정적 도구들은 종종 잘못된 코드를 용인할수도 있다. 또 런타임 도구들은 상당한 비용을 필요로 한다. 

더 나은 도구들이 나오기를 바란다. 도구들이 제대로 동작하도록 하라:
* 더 적은 전역 데이터
* 더 적은 `static` 변수들
* 스택 메모리 중심의 사용 (포인터들을 너무 많이 던지지 말아라)
* 변경할 수 없는 데이터를 더 많이 써라. (리터럴, `constexpr`, `const`)


### <a name="Rconc-data"></a>CP.3: 쓰기 가능한 데이터의 명시적인 공유를 최소화하라

##### 근거
쓰기 가능한 데이터를 공유하지 않는다면, 데이터 경쟁을 원천적으로 막을 수 있다.
공유를 줄일 수록, 동기화 접근과 데이터 경쟁의 가능성을 줄일 수 있다. 
더해서, 대기와 잠금 역시 줄어든다. (따라서 성능이 향상될 것이다)

##### 예
```
    bool validate(const vector<Reading>&);
    Graph<Temp_node> validate(const vector<Reading>&);
    Image validate(const vector<Reading>&);
    // ...

    void process_readings(istream& socket1)
    {
        vector<Reading> surface_readings;
        socket1 >> surface_readings;
        if (!socket1) throw Bad_input{};

        auto h1 = async([&] { if (!validate(surface_readings) throw Invalide_data{}; });
        auto h2 = async([&] { return temparature_gradiants(surface_readings); });
        auto h3 = async([&] { return altitude_map(surface_readings); });
        // ...
        auto v1 = h1.get();
        auto v2 = h2.get();
        auto v3 = h3.get();
        // ...
    }
```
위 코드에서 `const`들이 없다면, `surface_readings`에 대한 잠재적 데이터 경쟁 때문에 모든 비동기적인 함수 호출을 다시 점검해야 할 것이다. 

##### 참고 사항
변경할 수 없는 데이터는 안전하고 효율적으로 공유될 수 있다.   
여기엔 잠금이 필요하지 않다. 상수에 대한 데이터 경쟁은 존재하지 않는다.

##### 시행하기

???


### <a name="Rconc-task"></a>CP.4: 스레드보단 작업 단위로 생각하라

##### 근거
`thread`자체는 구현에 대한 개념이다. 이는 기계에 대해서 생각하는 것이다.  
작업은 응용(Application)에서의 개념(notion)이다. 당신의 생각과 더 가까울 것이고, 아마도 다른 작업들과 동시적일 것이다.  
응용개념은 그 이유에 대해 생각하기도 쉽다.

##### 예
```
    ???
```

###### 참고 사항
`async()`를 제외하면, 표준 라이브러리 기능들은 저-레벨에, 기계 중심적(machine-oriented)이고, 스레드-잠금 레벨에 위치한다. 
이 기능들은 필수적이지만, 우리는 생산성, 신뢰성, 그리고 성능을 위해서 추상화의 수준을 더 높일 필요가 있다.    
상위 레벨의, 응용 프로그램 중심적인 라이브러리들을 사용하는 것에 대해선 논의의 여지가 있다.
(if possibly, built on top of standard-library facilities).

##### 시행하기
???


### <a name="Rconc-volatile"></a>CP.8 동기화를 위해 `volatile`을 사용하지 말아라

##### 근거
C++ 에선, 다른 언어와는 다르게, `volatile`이 원자성과 스레드간 동기화를 제공하지 않는다.
또한 `volatile`은 명령어 재배치(컴파일러와 하드웨어 모두)를 제한하지도 않는다.
`volatile`은 동시성과 무관하다.

##### 잘못된 예
```
    // current source of memory for objects
    int free_slots = max_slots; 

    Pool* use()
    {
        if (int n = free_slots--) return &pool[n];
    }
```
이 코드엔 문제가 있다:
단일 스레드 프로그램이라면, 이 코드는 아무런 문제가 없다. 하지만 두 스레드가 동시에 실행한다면, `free_slots`에 대한 경쟁 상태가 발생한다. 따라서 두 스레드는 같은 `free_slots`값을 가질 수도 있다.  
이는 (명백하게) 잘못된 결과로 이어진다.다른 언어들에 익숙한 사람들은 이 코드를 다음과 같이 수정할 것이다.:
```
    // current source of memory for objects
    volatile int free_slots = max_slots; 

    Pool* use()
    {
        if (int n = free_slots--) return &pool[n];
    }
```
`volatile`로 바꿨지만, 동기화엔 아무런 영향이 없다. 이 코드엔 여전히 데이터 경쟁이 존재한다! 

C++ 에선, 동기화를 원한다면 `atomic` 타입들을 사용해야 한다:
```
    // current source of memory for objects
    atomic<int> free_slots = max_slots; 

    Pool* use()
    {
        if (int n = free_slots--) return &pool[n];
    }
```
이제 `--`연산은 원자적이다. 이는 다른 스레드가 끼여들 여지가 있는 읽기-증가-쓰기 과정과는 다르다.

##### 대안
다른 언어에서 `volatile`을 사용했다면, C++에선 `atomic`을 사용하라.
더 복잡한 경우라면 `mutex`를 사용하라.

##### 함께 보기
[(rare) `volatile`의 적절한 사용](#Rconc-volatile2)




## <a name="SScp-con"></a>CP.con: 동시성
이 Section은 다수의 스레드들이 공유 데이터를 사용해 통신하는 부분에 대해 다룹니다.

* 병렬 알고리즘은 [parallelism](#SScp-par)를 참조하세요.
* 명시적인 공유없는 작업간 통신방법은 [messaging](#SScp-mess)을 참조하세요.
* For vector parallel code, see [vectorization](#SScp-vec)
* 무잠금 프로그래밍은 [lock free](#SScp-free)를 참조하세요.


동시성 규칙 요약:

* [CP.20: 단순한 `lock()`/`unlock()`대신 RAII를 사용하라](#Rconc-raii)
* [CP.21: `std::lock()`은 다수의 `mutex`들에 사용하라](#Rconc-lock)
* [CP.22: lock을 사용 중일때는 알 수 없는 코드를 호출하지 말아라(e.g., a callback)](#Rconc-unknown)
* [CP.23: join하는 `thread`를 유효범위 안의 컨테이너처럼 생각하라](#Rconc-join)
* [CP.24: detach한 `thread`를 전역 컨테이너처럼 생각하라](#Rconc-detach)
* [CP.25: `detach()`를 쓰지 않는다면 `std::thread`보다는 `gsl::raii_thread`를 사용하라](#Rconc-raii_thread)
* [CP.26: `detach()`를 쓴다면 `std::thread`보다는 `gsl::detached_thread`를 사용하라](#Rconc-detached_thread)
* [CP.27: 실행시간 조건에 따라 detach하는 `thread`에는 `std::thread`를 사용하라](#Rconc-thread)
* [CP.28: `detach()`하지 않은 유효범위 안의 `thread`는 join하도록 하라](#Rconc-join)
* [CP.30: `raii_thread'가 아닌 스레드들에게 지역변수에 대한 포인터를 전달하지 말아라](#Rconc-pass)
* [CP.31: 스레드들 간의 작은 데이터 전달은 참조나 포인터보다는 값으로 전달하라](#Rconc-data)
* [CP.32: 관련 없는 `thread`간의 소유권 공유는 `shared_ptr`를 사용하라](#Rconc-shared)
* [CP.40: 문맥 교환을 최소화하라](#Rconc-switch)
* [CP.41: 스레드 생성과 소멸을 최소화하라](#Rconc-create)
* [CP.42: 조건 없이 `wait`하지 말아라](#Rconc-wait)
* [CP.43: Critical section의 사용시간을 최소화하라](#Rconc-time)
* [CP.44: `lock_guard`과 `unique_lock`에는 이름을 붙여라](#Rconc-name)
* [CP.50: `mutex`는 보호할 데이터와 함께 정의하라](#Rconc-mutex)
* ??? 언제 스핀락(spinlock)을 사용하는가
* ??? 언제 `try_lock()`을 사용하는가
* ??? 언제 `unique_lock`보다 `lock_guard`를 쓰는가
* ??? Time multiplexing
* ??? 언제/어떻게 `new thread`를 사용하는가



### <a name="Rconc-raii"></a>CP.20: 단순한 `lock()`/`unlock()`대신 RAII를 사용하라

##### 근거
잠금을 해제하지 않음으로 인한 오류를 예방한다.

##### 잘못된 예
```
    mutex mtx;

    void do_stuff()
    {
        mtx.lock();
        // ... do stuff ...
        mtx.unlock();
    }
```
얼마 후 또는 나중에, 누군가 `mtx.unlock()`을 잊어버린다. 그리곤 `return`을 `... do stuff ...`에 붙이거나, 예외나 무언가를 던진다. 
```
    mutex mtx;

    void do_stuff()
    {
        unique_lock<mutex> lck {mtx};
        // ... do stuff ...
    }
```

##### 시행하기
 `lock()`과 `unlock()`의 호출에 표식을 남겨라  ???


### <a name="Rconc-lock"></a>CP.21: Use `std::lock()` to acquire multiple `mutex`es

##### 근거
To avoid deadlocks on multiple `mutex`s

##### 예
This is asking for deadlock:
```
    // thread 1
    lock_guard<mutex> lck1(m1);
    lock_guard<mutex> lck2(m2);

    // thread 2
    lock_guard<mutex> lck2(m2);
    lock_guard<mutex> lck1(m1);
```
Instead, use `lock()`:
```
    // thread 1
    lock_guard<mutex> lck1(m1,defer_lock);
    lock_guard<mutex> lck2(m2,defer_lock);
    lock(lck1,lck2);

    // thread 2
    lock_guard<mutex> lck2(m2,defer_lock);
    lock_guard<mutex> lck1(m1,defer_lock);
    lock(lck2,lck1);
```
Here, the writers of `thread1` and `thread2` are still not agreeing on the order of the `mutex`es, but order no longer matters.

##### 참고사항
In real code, `mutex`es are rarely named to conveniently remind the programmer of an intended relation and intended order of acquisition.
In real code, `mutex`es are not always conveniently aquired on consequtive lines.

I'm really looking forward to be able to write plain
```
    lock_guard lck1(m1,defer_lock);
```
and have the `mutex` type deduced.

##### 시행하기
Detect the acquistion of multiple `mutex`es.
This is undecidable in general, but catching common simple examples (like the one above) is easy.


### <a name="Rconc-unknown"></a>CP.22: Never call unknown code while holding a lock (e.g., a callback)

##### 근거
If you don't know what a piece of code does, you are risking deadlock.

##### 예
```
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

##### 예
A common example of the "calling unknown code" problem is a call to a function that tries to gain locked access to the same object.
Such problem cal often be solved by using a `recursive_mutex`. For example:
```
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

##### 시행하기
* Flag calling a virtual function with a non-recursive `mutex` held
* Flag calling a callback with a non-recursive `mutex` held


### <a name="Rconc-join"></a>CP.23: Think of a joining `thread` as a scoped container

##### 근거
To maintain pointer safety and avoid leaks, we need to consider what pointers a used by a `thread`.
If a `thread` joins, we can safely pass pointers to objects in the scope of the `thread` and its enclosing scopes.

##### 예
```
    void f(int * p)
    {
        // ...
        *p = 99;
        // ...
    }
    int glob = 33;

    void some_fct(int* p)
    {
        int x = 77;
        raii_thread t0(f,&x);           // OK
        raii_thread t1(f,p);            // OK
        raii_thread t2(f,&glob);        // OK
        auto q = make_unique<int>(99);
        raii_thread t3(f,q.get());      // OK
        // ...
     }
```
An `raii_thread` is a `std::thread` with a destructor that joined and cannot be `detached()`.
By "OK" we mean that the object will be in scope ("live") for as long as a `thread` can use the pointer to it.
The fact that `thread`s run concurrently doesn't affect the lifetime or ownership issues here;
these `thread`s can be seen as just a function object called from `some_fct`.

##### 시행하기
Ensure that `raii_thread`s don't `detach()`.
After that, the usual lifetime and ownership (for local objects) enforcement applies.


### <a name="Rconc-detach"></a>CP.24: Think of a detached `thread` as a global container
detach된 `thread`를 전역 컨테이너 처럼 생각하라.

##### 근거
포인터를 안전하게 남겨두고 누수(leak)을 방지하기 위해선, 어떤 포인터들이 `thread`에 의해서 사용되는지 고려해야 한다.
만약 `thread`가 detach되었다면, 우리는 정적 객체 또는 자유 영역에 있는 객체들에 대한 포인터만 안전하게 전달할 수 있다.

##### 예
```
    void f(int * p)
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

##### 시행하기
In general, it is undecidable whether a `detach()` is executed for a `thread`, but simple common cases are easily detected.
If we cannot prove that a `thread` does not `detatch()`, we must assune that it does and that it outlives the scope in which it was constructed;
After that, the usual lifetime and ownership (for global objects) enforcement applies.


### <a name="Rconc-raii_thread"></a>CP.25: `detach()`를 쓰지 않는다면 `std::thread`보다는 `gsl::raii_thread`를 사용하라

##### 근거
`raii_thread`는 유효 범위가 끝날때 join한다.
Detach된 스레드들은 관찰(monitor)하기가 어렵다. 

??? 모든 "immortal threads"들은 `detach()`하기 보다는 자유 영역에 배치하라?

##### 예
```
    ???
```
##### 시행하기
???


### <a name="Rconc-detached_thread"></a>CP.26: `detach()`를 쓴다면 `std::thread`보다는 `gsl::detached_thread`를 사용하라

##### 근거
종종, `detach`의 필요는 `thread`의 작업에 기인한다. 
문서화가 이해롸 정적 분석을 도울 수 있다. 

##### 예
```
    void heartbeat();

    void use()
    {
        gsl::detached_thread t1(heartbeat);    // 명백하게 join할 필요가 없다.
        
        // 이 경우 join해야 하는가? (heartbeat()의 코드를 확인)
        std::thread t2(heartbeat);             
        // ...
    }
```
평범한(plain) `thread`에서 조건이 부여되지 않은 `detach`에는 표시를 남겨라.



### <a name="Rconc-thread"></a>CP.27: 실행시간 조건에 따라 detach하는 `thread`에는 `std::thread`를 사용하라

##### 근거
무조건적으로  `join` 하거나 무조건적으로 `detach`하도록 의도된 `thread`들은 분명히 식별된다.   
평범한 `thread`들은 `std::thread`의 일반성을 최대한 사용하도록 해야한다. 

##### 예
```
    void tricky(thread* t, int n)
    {
        // ...
        if (is_odd(n))
            t->detach();
        // ...
    }

    void use(int n)
    {
        thread t { thricky, this, n };
        // ...
        // ... 이때 여기서 join해야 하는가? ...
    }
```
##### 시행하기
???



### <a name="Rconc-join"></a>CP.28: `detach()`하지 않은 유효범위 안의 `thread`는 join하도록 하라

##### 근거
`detach()`하지 않은 `thread`의 소멸은 프로그램을 종료시킨다.

##### 잘못된 예
```
    void f() { std::cout << "Hello "; }

    struct F {
        void operator()() { std::cout << "parallel world "; }
    };

    int main()
    {
        std::thread t1{f};      // f() 는 다른 스레드에서 실행된다
        std::thread t2{F()};    // F()() 는 다른 스레드에서 실행된다
    }  // 버그 발견
```

##### 예
```
    void f() { std::cout << "Hello "; }

    struct F {
        void operator()() { std::cout << "parallel world "; }
    };

    int main()
    {
        std::thread t1{f};      // f() 는 다른 스레드에서 실행된다
        std::thread t2{F()};    // F()() 는 다른 스레드에서 실행된다

        t1.join();
        t2.join();
    }  // 버그가 하나 남았다
```
??? `cout`이 동기화 되나요?

##### 시행하기
* `raii_thread`들의 `join`에 표시를 남겨라 ???
* `detached_thread`들의 `detach`에 표시를 남겨라.


### <a name="RRconc-pass"></a>CP.30: `raii_thread'가 아닌 스레드들에게 지역변수에 대한 포인터를 전달하지 말아라

###### 근거
일반적으로, `raii_thread`가 아닌 스레드가 포인터를 전달한 스레드보다 오래  남아있을지 알수 없다. (이런 경우 전달한 포인터들은 모두 무용지물이 될 것이다) 

##### 잘못된 예
```
    void use()
    {
        int x = 7;
        thread t0 { f, ref(x) };
        // ...
        t0.detach();
    }
```
`detach()`는 찾기 어려울 수도 있다. `raii_thread`를 쓰거나 포인터를 전달하지 말아라. 

##### 잘못된 예
```
    ??? put pointer to a local on a queue that is read by a longer-lived thread ???
```
##### 시행하기
평범한(plain) `thread`의 생성자에 지역변수에 대한 포인터를 전달하는 경우 표시를 남겨라.



### <a name="Rconc-switch"></a>CP.31: Pass small amounts of data between threads by value, reather by reference or pointer

##### 근거

Copying a small amount of data is cheaper to copy and access than to share it using some locking mechanism.
Copying naturally gives unique ownership (simplifies code) and eliminates the possibility of data races.

##### 참고사항
"작은 크기"를 정확하게 정의하는 것은 불가능하다.

##### 예
```
    string modify1(string);
    void modify2(shared_ptr<string);

    void fct(string& s)
    {
        auto res = async(modify1,string);
        async(modify2,&s);
    }
```
The call of `modify1` involves copying two `string` values; the call of `modify2` does not.
On the other hand, the implementation of `modify1` is exactly as we would have written in for single-threaded code,
wheread the implementation of `modify2` will need some form of locking to avoid data races.
If the string is short (say 10 characters), the call of `modify1` can be surprisingly fast;
essentially all the cost is in the `thread` switch. If the string is long (say 1,000,000 characters), copying it twice
is probably not a good idea.

Note that this argument has nothing to do with `sync` as sunch. It applies equally to considerations about whether to use
message passing or shared memory.

##### 시행하기
???


### <a name="Rconc-shared"></a>CP.32: 관련 없는 `thread`간의 소유권 공유는 `shared_ptr`를 사용하라

##### 근거
만약 스레드들이 서로 무관하다면 (달리 말해, 같은 유효범위 안에 없거나 한 스레드가 다른 스레드의 생애를 포함하지 않는 경우), 그리고 소멸되어야 하는 자유 영역 메모리를 공유할 필요가 있다면, `shared_ptr`또는 동등한 것이 소멸을 보장할 수 있다. 

##### 예
```
    ???
```

##### 참고사항
* 정적(예컨대 전역) 객체는 한 스레드가 객체의 소멸에 책임이 있다는 점에서 공유할 수 있다
* 소멸되지 않는 자유 영역의 객체는 공유할 수 있다.
* 한 스레드가 객체를 소유하고, 다른 스레드가 소유자 스레드보다 오래 살아남지 않으면 객체를 안전하게 공유할 수 있다.

##### 시행하기
???


### <a name="Rconc-switch"></a>CP.40: Minimize context switching

##### 근거
문맥 교환(Context switches)은 높은 비용을 필요로 한다.

##### 예
```
    ???
```
##### 시행하기
???



### <a name="Rconc-create"></a>CP.41: Minimize thread creation and destruction

##### 근거
스레드 생성은 높은 비용을 필요로 한다.

##### 예
```
    void worker(Message m)
    {
        // 프로세스
    }

    void master(istream& is)
    {
       for (Message m; is>>m; )
            run_list.push_back(new thread(worker,m);}
    }
```
이 코드는 메세지마다 `thread`를 생성한다. 그리고 `run_list`는 아마도 작업들이 끝나면 스레드를 파괴할 것이다. 

대신, 미리 만들어둔 작업자(worker) 스레드들이 메세지를 처리하도록 할 수 있다.
```
    Sync_queue<Message> work;

    void master(istream& is)
    {
       for (Message m; is >> m; )
            work.put(n);
    }

    void worker()
    {
        for (Message m; m = work.get(); ) {
               // process
        }
    }

    // 작업자 스레드들을 준비한다. (4개)
    void workers()  
    {
        raii_thread w1 {worker};
        raii_thread w2 {worker};
        raii_thread w3 {worker};
        raii_thread w4 {worker};
    }
```
###### 참고사항
시스템에서 잘 만든 스레드 풀을 지원한다면, 그것을 사용하라.
시스템에서 잘 만든 메세지 큐를 지원하면, 그것을 사용하라.

##### 시행하기
???


### <a name="Rconc-wait"></a>CP.42: Don't `wait` without a condition

##### 근거
조건을 주지 않고 `wait`하는 것은 작업을 위해 깨어나는 것(wakeup) 또는 단순히 작업이 없다는 것을 확인하기 위해 깨어나는 것을 놓칠 수 있다.

##### 잘못된 예
```
    std::condition_variable cv;
    std::mutex mx;

    void thread1()
    {
        while (true) {
            // 작업 수행 ...
            std::unique_lock<std::mutex> lock(mx);
            cv.notify_one();    // 다른 스레드를 깨운다
        }
    }

    void thread2()
    {
        while (true) {
            std::unique_lock<std::mutex> lock(mx);
            cv.wait(lock);    // 영원히 block될 수도 있다.
            // 작업 수행 ...
        }
    }
```
여기서, 코드에 표현되지 않은 다른 `thread`가 다른 `thread1`의 알림(notification)을 소비하면, `thread2`는 영원히 대기할 수도 있다. 

##### 예
```
    template<typename T>
    class Sync_queue {
    public:
        void put(const T& val);
        void put(T&& val);
        void get(T& val);
    private:
        mutex mtx;
        condition_variable cond;    // 접근 동기화를 위한 조건변수 
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
        cond.wait(lck, [this]{ return !q.empty(); });    // spurious wakeup 방지
        val = q.front();
        q.pop_front();
    }
```
이제 만약 큐가 비어있는 상태에서 `get()`을 실행하는 스레드가 깨어나게 된다면(예를 들면 다른 스레드가 먼저 `get()`을 실행해서 가져갔다거나), 
해당 스레드는 그대로 다시 sleep 상태가 될 것이다.

##### 시행하기
조건이 없는 모든 `waits`에 표시를 남겨라.


### <a name="Rconc-time"></a>CP.43: Minimize time spent in a critical section

##### 근거
`mutex`를 가진 시간이 짧을 수록, 다른 `thread`가 대기해야 하는 경우가 줄어들 것이다. 그리고 `thread`의 중지(suspection)와 재실행(resumption)은 많은 비용을 필요로 한다.

##### 예
```
    void do_something() // 잘못된 예
    {
        unique_lock<mutex> lck(my_lock);
        do0();  // 준비     : 잠금이 필요하지 않은 부분
        do1();  // 트랜잭션 : 잠금이 필요한 부분
        do2();  // 정리     : 잠금이 필요하지 않은 부분
    }
```
이 코드에선 필요 이상으로 오랫동안 잠금을 소유하고 있다. 
필요하지 않다면 잠금을 가져서는 안되며, 정리를 시작하기 전에 잠금을 해제해야 한다.  
이렇게 다시 작성할 수 있을 것이다.
We could rewrite this to
```
    void do_something() // 잘못된 예
    {
        do0();  // 준비     : 잠금이 필요하지 않은 부분
        my_lock.lock();
        do1();  // 트랜잭션 : 잠금이 필요한 부분
        my_lock.unluck();
        do2();  // 정리     : 잠금이 필요하지 않은 부분
    }
```
하지만 이러한 코드는 안전성에 대해서 타협할 뿐이고, [use RAII](#Rconc-raii) 규칙을 위반한다. 
대신, critical section에 블록을 추가하라:
```
    void do_something() // OK
    {
        do0();       // 준비     : 잠금이 필요하지 않은 부분
        {
            unique_lock<mutex> lck(my_lock);
            do1();  // 트랜잭션 : 잠금이 필요한 부분
        }
        do2();      // 정리     : 잠금이 필요하지 않은 부분
    }
```
##### 시행하기
일반적으로 불가능하다. 
RAII를 적용하지 않은 `lock()`과 `unlock()`에는 표시를 남겨라.


### <a name="Rconc-name"></a>CP.44: Remember to name your `lock_guard`s and `unique_lock`s

##### 근거
An unnamed local objects is a temporary that immediately goes out of scope.

##### 예
```
    unique_lock<mutex>(m1);
    lock_guard<mutex> {m2};
    lock(m1,m2);
```
이 코드는 별로 문제 없어 보이지만, 그렇지 않다.

##### 시행하기
이름을 부여하지 않은 `lock_guard`와 `unique_lock`에 표시를 남겨라 


### <a name="Rconc-mutex"></a>P.50: Define a `mutex` together with the data it guards

##### 근거
데이터에 대한 보호는 필요하고, 어떻게 보호할 것인지 독자는 이해할 것이다.

##### 예
```
    struct Record {
        std::mutex m;   // 다른 멤버에 접근하기 전에 이 뮤텍스를 점유한다
        // ...
    };
```
##### 시행하기
??? 가능한가요?


## <a name="SScp-par"></a>CP.par: 병렬성
여기서 병렬성(parallelism)은 많은 데이터에 대해서 동시에 한 작업을 병렬적으로 수행하는 것을 의미한다.

병렬성 규칙 요약:

* ???
* ???
* 적합하다고 판단되면, 표준 라이브러리의 병렬 알고리즘들을 사용하라.
* 병렬성을 위해 설계된 알고리즘들을 사용하라. 불필요한 의존성을 지닌 알고리즘을 사용하지 말아라.


## <a name="SScp-mess"></a>CP.mess: 메세지 전달

The standard-library facilities are quite low level, focused on the needs of close-to the hardware critical programming using `thread`s, `mutex`ex, `atomic` types, etc.
대부분의 사람들은 이 레벨에서 작업해선 안된다. 에러를 만들기 쉽고, 개발이 느리다.
가능하다면, 메세징 라이브러리, 병렬 알고리즘, 그리고 벡터화같은 상위 레벨의 기능을 사용하라.
이 영역에선 프로그래머가 명시적으로 동기화 할 필요가 없는 메세지 전달에 대해 살펴본다.


메세지 전달 규칙 요약:

* [CP.60: 동시적인 작업으로부터 반환값을 받는데 `future`를 사용하라](#Rconc-future)
* [CP.61: 동시적인 작업을 생성하기 위해선 `async()`를 사용하라](#Rconc-async)
* 메세지 큐
* 메세징 라이브러리들

???? "`std::async`보다는 X를 사용하라"같은 규칙이 필요할까요? X가 더 잘 정의된 스레드 풀일 수 있을텐데도?

??? Is `std::async` worth using in light of future (and even existing, as libraries) parallelism facilities? What should the guidelines recommend if someone wants to parallelize, e.g., `std::accumulate` (with the additional precondition of commutativity), or merge sort?


### <a name="Rconc-future"></a>CP.60: 동시적인 작업으로부터 반환값을 받는데 `future`를 사용하라

##### 근거
`future`는 비동기 작업에 대한 일반적인 함수 호출-반환 문맥을 남겨두는데 사용된다. 명시적인 잠금이 없고 정확한 반환값과 에러(예외) 반환 모두 간단히 처리된다. 

##### 예
```
    ???
```
##### 참고사항
???

##### 시행하기
???

### <a name="Rconc-async"></a>CP.61: 동시적인 작업을 생성하기 위해선 `async()`를 사용하라

##### 근거
(원문 오기로 인한 내용 없음)

##### 예

    ???

##### 참고사항
불행하게도 `async()`는 완벽하지 않다.
예를 들면, 스레드의 생성을 최소화하기 위해 스레드 풀이 사용된다는 보장을 하지 않는다. 실제로, 대부분의 동시적인 `async()`들은 스레드 풀을 구현하고 있지 않다.

그렇지만, `async()`는 단순하고 논리적으로 정확하다. 더 나은 무언가가 나오거나 많은 비동기적인 작업들을 최적화 해야만 하지 않다면, `async()`를 계속 써도 무방하다.

##### 시행하기
???



## <a name="SScp-vec"></a>CP.vec: 벡터화
벡터화란 명시적인 동기화를 사용하지 않고 몇개의 작업들을 동시적으로 수행하는 기술을 의미한다.
연산은 단순하게 자료구조(벡터, 배열 등등)의 원소들에 병렬적으로 작용된다.
벡터화는 비 지역적 변화를 필요로 하지 않는다는 점에서 흥미로운 속성을 지닌다. 하지만, 벡터화는 단순한 자료구조들과 병렬적 연산을 가능하게 하는 특별한 알고리즘이 함께 써야 최적의 성능을 보인다.

벡터화 규칙 요약:

* ???
* ???



## <a name="SScp-free"></a>CP.free: Lock-free 프로그래밍
`mutex`와 `condition_variable`를 사용한 동기화는 상대적으로 높은 비용을 지불해야 할 수 있다. 나아가서, 데드락(deadlock)으로 이어질 가능성 또한 존재한다.
때때로, 성능과 데드락의 가능성을 없애기 위해선, 까다로운 저-레벨 "무잠금" 기능들을 사용해야 한다. 이 기능들은 일시적으로 배타적인(원자적인) 메모리 접근을 사용한다.  
Lock-free 프로그래밍은 `thread`, `mutex`와 같은 상위레벨의 동시성 메커니즘을 구현하는데 사용되기도 한다.  

Lock-free 프로그래밍 규칙 요약:

* [CP.100: 정말 필요할 때만 lock-free 프로그래밍을 사용하라](#Rconc-lockfree)
* [CP.101: 하드웨어/컴파일러 조합을 불신하라](#Rconc-distrust)
* [CP.102: 문헌을 세심하가 공부하라](#Rconc-litterature)
* 언제/어떻게 atomic을 사용할 것인가
* 기아상태를 회피하라
* 정교하게 조작한 lock-free 메모리 접근보다는 lock-free 자료구조를 사용하라
* [CP.110: double-checkd locking에는 전통적인 패턴을 사용하라](#Rconc-double)
* 언제/어떻게 CAS(Compare And Swap) 연산을 사용하는가


### <a name="Rconc-lockfree"></a>CP.100: 정말 필요할 때만 lock-free 프로그래밍을 사용하라

##### 근거
lock-free 프로그래밍은 에러를 발생시키기 쉽고 언어 기능, 머신 아키텍처, 자료구조에 대한 전문가 수준의 지식을 필요로 한다. 

##### 잘못된 예
```
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
버그를 찾아보라. 테스팅을 통해서 찾기 정말정말 어려울 것이다.
ABA 문제에 대해서 읽어보라.

##### 예외 사항
[원자적 변수들](#???) 단순성과 안전성을 위해 사용될 수 있다.

##### 참고사항
상위 레벨 동시성 메커니즘들은 `thread`와 `mutex` 같은 lock-free 프로그래밍으로 구현되었다.

##### 대안
다른 사람들에 의해서 구현되었거나 라이브러리의 일부로 존재하는 lock-free 자료구조를 사용하라.


### <a name="Rconc-distrust"></a>CP.101: 하드웨어/컴파일러 조합을 불신하라

##### 근거
lock-free프로그래밍에 사용되는 저-레벨 하드웨어 인터페이스는 가장 구현하기 어렵고 미묘한 이식성 문제가 발생할 수 있다.  
성능을 위해 lock-free 프로그래밍을 사용하고 있다면, 오히려 성능 저하(regression)가 있지는 않은지 확인해야 한다.

##### 참고사항
(정적/동적) 명령어 재배치(Instruction Reordering) 때문에 이 레벨에서는 효율적으로 생각하는 것이 어렵다 (특히 relaxed 메모리 모델을 사용하고 있다면).
경험과, (준)형식적인 lock-free 모델들, 그런 모델들에 대한 점검이 유용할 수 있다.
테스팅 - 종종 엄청나게 넓은 영역에 대한 - 은 필수적이다.
"Don't fly too close to the wind."

##### 시행하기
하드웨어, 운영체제, 컴파일러, 그리고 라이브러리의 모든 변화를 포함하는 강력한 테스팅 규칙을 세워라.


### <a name="Rconc-litterature"></a>CP.102: 문헌을 세심하가 공부하라

##### 근거
atomic에서의 예외와 표준적인 패턴들, lock-free 프로그래밍은 온전히 전문가를 위한 내용이다. 
lock-free 코드를 적용하기에 앞서서 전문가가 되어라.

##### 참조 문헌
* Anthony Williams: C++ concurrency in action. Manning Publications.
* Boehm, Adve, You Don't Know Jack About Shared Variables or Memory Models , Communications of the ACM, Feb 2012.
* Boehm, "Threads Basics", HPL TR 2009-259.
* Adve, Boehm, "Memory Models: A Case for Rethinking Parallel Languages and Hardware", Communications of the ACM, August 2010.
* Boehm, Adve, "Foundations of the C++ Concurrency Memory Model", PLDI 08.
* Mark Batty, Scott Owens, Susmit Sarkar, Peter Sewell, and Tjark Weber, "Mathematizing C++ Concurrency", POPL 2011.
* Damian Dechev, Peter Pirkelbauer, and Bjarne Stroustrup: Understanding and Effectively Preventing the ABA Problem in Descriptor-based Lock-free Designs. 13th IEEE Computer Society ISORC 2010 Symposium. May 2010.
* Damian Dechev and Bjarne Stroustrup: Scalable Non-blocking Concurrent Objects for Mission Critical Code. ACM OOPSLA'09. October 2009
* Damian Dechev, Peter Pirkelbauer, Nicolas Rouquette, and Bjarne Stroustrup: Semantically Enhanced Containers for Concurrent Real-Time Systems. Proc. 16th Annual IEEE International Conference and Workshop on the Engineering of Computer Based Systems (IEEE ECBS). April 2009.


### <a name="Rconc-double"></a>CP.110: double-checkd locking에는 전통적인 패턴을 사용하라

##### 근거
이중 확인 잠금(Double-checked locking)은 잘못되기 쉽다.

##### 예
```
    atomic<bool> x_init;

    if (!x_init.load(memory_order_acquire) {
        lock_guard<mutex> lck(x_mutex);
        if (!x_init.load(memory_order_relaxed) {
            // ... x를 초기화 ...
            x_init.store(true, memory_order_release);
        }
    }

    // ... x 를 사용 ...
```

##### 시행하기
??? 저 idiom 확인할 수 있는 건가요?



## <a name="SScp-etc"></a>CP.etc: Etc. 동시성 규칙들
이 규칙들은 분류를 적용하지 않는다.

* [CP.200:  `volatile`은 C++가 아닌 메모리에 대해서만 사용하라](#Rconc-volatile2)
* [CP.201: ??? Signals](#Rconc-signal)

### <a name="Rconc-volatile2"></a>CP.200: `volatile`은 C++가 아닌 메모리에 대해서만 사용하라

##### 근거
`volatile`은 "C++가 아닌" 코드 또는 C++ 메모리 모델을 따르지 않는 하드웨어에 있는 객체를 참조하기 위해 사용된다.

##### 예
```
    const volatile long clock;
```
이 코드는 시계 회로가 지속적으로 업데이트하는 레지스터를 의미한다. 
`clock`은 `volatile`인데, 이는 C++ 프로그램 내에서는 아무것도 하지 않았지만 값이 바뀔 수 있기 때문이다.
예컨대, `clock`을 두번 읽는 것은 다른 값을 반환할 수도 있다, 때문에 최적화기(optimizer)는 반복된 읽기 과정을 최적화해서 없애지 않는 것이 나을 것이다.
```
    long t1 = clock;
    // ... no use of clock here ...
    long t2 = clock;
```
`clock`은 프로그램에서 값을 쓰려고 하면 안되기 때문에 `const`로 사용된다.

###### 참고사항
하드웨어를 직접 조작하는 최저 레벨의 코드를 작성하는 게 아니라면, `volatile`는 피하는게 좋은 난해한(esoteric) 기능이라고 생각하라.

###### 예
일반적으로 C++ 코드는 다른 어딘가(하드웨어 또는 다른 언어)에 속한 메모리를 가져올 때  `volatile`을 사용한다. 
```
    int volatile* vi = get_hardware_memory_location();
        // 참고: 여기선 외부 메모리의 포인터를 가져온다.
        // volatile은 "여기에 추가적인 주의를 기울여라"라고 말하는 것이다.
```
때때로 C++ 코드는  `volatile` 메모리를 할당하고 의도적으로 포인터를 다른 어딘가(하드웨어 또는 다른언어)와 공유하기도 한다:
```
    static volatile long vl;
    please_use_this(&vl);   
    // vl에 대한 참조를 C++가 아닌 "어딘가"에 넘겨준다.
```
##### 잘못된 예
`volatile` 지역변수들은 거의 대부분의 경우 잘못 사용된 것이다 --  local variables are nearly always wrong -- 그 변수들이 일시적(emphemeral)이라면 어떻게 다른 언어/기계들과 공유될 수 있겠는가?
멤버 변수들의 경우도 같은 이유로 `volatile`의 잘못된 사용일 수 있다.
```
    void f() {
        volatile int i = 0; // 좋지 않다. volatile 지역 변수
        // etc.
    }

    class mytype {
        volatile int i = 0; // 의심스러운 코드. volatile 멤버
        // etc.
    };
```

##### 참고사항
C++ 에서는, 다른 언어와는 달리, `volatile`은 동기화와 관련해 [아무것도 하지 않습니다](#Rconc-volatile).

##### 시행하기

* `volatile T` 을 사용하는 지역, 멤버 변수들에는 표시를 남겨라; 대부분의 경우  `atomic<T>` 를 의도했을 것이다.
* ???



### <a name="Rconc-signal"></a>CP.201: ??? 시그널(Signals)

???UNIX signal handling???. May be worth reminding how little is async-signal-safe, and how to communicate with a signal handler (best is probably "not at all")

