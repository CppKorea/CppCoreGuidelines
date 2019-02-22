
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
* [CP.9: 가능한 모든 경우에, 도구 (tool) 를 이용하여 자신의 병행 실행 코드를 검증하라](#Rconc-tools)

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

### <a name="Rconc-tools"></a>CP.9: 가능한 모든 경우에, 도구 (tool) 를 이용하여 자신의 병행 실행 코드를 검증하라

경험이 보여주듯, 병행 실행 코드는 올바르게 작성하기가 극히 어려우며,
컴파일 시점 점검, 실행 시점 점검 및 테스팅을 통한 병행 실행 문제를 검출하는 것이
순차적인 코드에서 일반적인 문제점을 찾는 것만큼 효과적이지 않다.
미묘한 병행 실행 오류들은 메모리 손상이나 교착 상태와 같은 심각한 폐해를 가져올 수 있다.

##### Example

    ???

##### Note

스레드 안전한 프로그램을 만드는 것은 숙달된 개발자들조차 종종 곤란하게 만드는 만만찮은 작업이다: 적절한 도구를 사용하는 것은 이러한 위험을 덜 수 있는 중요한 전략이다.
이러한 도구들은 상용 / 오픈소스 와 연구용 / 생산용을 가리지 않고 다양한 형태의 수많은 종류가 제공되고 있다.
안타깝게도, 각자의 요구 사항과 제약사항이 모두 다르므로, 특정 도구를 추천하기는 어려우나,
몇 가지를 거론할 수는 있다:

* 정적 지침 도구: [clang](http://clang.llvm.org/docs/ThreadSafetyAnalysis.html)
과 [GCC](https://gcc.gnu.org/wiki/ThreadSafetyAnnotation) 의
몇몇 지난 버전들은 스레드 안전의 특성과 관련된 몇 가지 정적 주해문 (static annotation) 을 지원한다.
이러한 기법의 일관성 있는 사용은 다양한 종류의 스레드 안전 관련 오류를 컴파일 시점 오류로 만들어준다.
이러한 주해문들은 일반적으로 지역적이며 (특정 멤버 변수를 특정 뮤텍스를 통해 보호되게 표시함으로써),
대개의 경우 쉽게 배울 수 있다. 그러나, 다른 많은 정적 도구들과 마찬가지로,
본디 검출되어야 하나 허용 처리되는 잘못된 거짓 음성 결과를 종종 제출할 수도 있다.

* 동적 지침 도구: Clang 의 [Thread Sanitizer](http://clang.llvm.org/docs/ThreadSanitizer.html) (aka TSAN)
은 강력한 동적 도구의 한 예이다: 이 도구는 프로그램의 메모리 접근을 장부에 기록할 수 있도록 빌드 및 실행을 변경하여,
해당 프로그램 실행 시 데이터 경쟁을 확실히 검출해 낼 수 있도록 한다.
이를 위해서는 메모리 사용랑 (대부분의 경우 5-10배) 과 CPU 성능 저하 (2-20배) 를 비용으로 지불해야 한다.
이러한 동적 도구들은 통합 테스트나 카나리아 테스트 혹은 복수의 스레드상에서 작동하는 단위 테스트에 적용하기 가장 알맞다.
작업량이 영향을 미친다: TSAN 이 검출해 낸 문제점은 거의 항상 실질적인 데이터 경쟁을 정확하게 짚어 내지만,
해당 실행 과정 중에 발견된 문제만이 검출 가능하다.

##### Enforcement

특정 프로그램들에 있어서 어떤 도구를 사용하는 것이 유용할 것인지 고르는 것은 해당 프로그램 제작자에게 달려 있다.

## <a name="SScp-con"></a>CP.con: 동시성

이 Section은 다수의 스레드들이 공유 데이터를 사용해 통신하는 부분에 대해 다룹니다.

* 병렬 알고리즘은 [parallelism](#SScp-par)를 참조하세요
* 명시적인 공유없는 작업간 통신방법은 [messaging](#SScp-mess)을 참조하세요
* For vector parallel code, see [vectorization](#SScp-vec)
* 무잠금 프로그래밍은 [lock free](#SScp-free)를 참조하세요

동시성 규칙 요약:

* [CP.20: `lock()`/`unlock()` 대신 RAII를 사용하라](#Rconc-raii)
* [CP.21: 복수의 `mutex` 획득을 위해서는 `std::lock()` 이나 `std::scoped_lock` 을 사용하라](#Rconc-lock)
* [CP.22: lock 을 사용 중일 때는 알 수 없는 코드를 호출하지 말아라(예: callback)](#Rconc-unknown)
* [CP.23: join 하는 `thread`를 유효범위 안의 컨테이너처럼 생각하라](#Rconc-join)
* [CP.24: `thread`를 전역 컨테이너처럼 생각하라](#Rconc-detach)
* [CP.25: `std::thread` 보다는 `gsl::joining_thread` 사용을 우선하여 고려하라](#Rconc-joining_thread)
* [CP.26: 스레드를 `detach()` 하지 말아라](#Rconc-detached_thread)
* [CP.31: 스레드들 간의 작은 데이터 전달은 참조나 포인터보다는 값으로 전달하라](#Rconc-data)
* [CP.32: 관련 없는 `thread`간의 소유권 공유는 `shared_ptr`를 사용하라](#Rconc-shared)
* [CP.40: 문맥 교환을 최소화하라](#Rconc-switch)
* [CP.41: 스레드 생성과 소멸을 최소화하라](#Rconc-create)
* [CP.42: 조건 없이 `wait`하지 말아라](#Rconc-wait)
* [CP.43: 임계 영역(Critical Section)에서의 시간을 최소화하라](#Rconc-time)
* [CP.44: `lock_guard`과 `unique_lock`에는 이름을 붙여라](#Rconc-name)
* [CP.50: `mutex` 를 보호(guard) 해야 하는 데이터와 함께 선언하라. 가능한 경우에는  `synchronized_value<T>` 를 사용하라](#Rconc-mutex)
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

### <a name="Rconc-lock"></a>CP.21: 복수의 `mutex` 획득을 위해서는 `std::lock()` 이나 `std::scoped_lock` 을 사용하라 

##### Reason

복수의 `mutex` 로 인한 교착상태를 방지한다.

##### Example

다음 코드는 교착 상태를 유발한다:

```c++
    // thread 1
    lock_guard<mutex> lck1(m1);
    lock_guard<mutex> lck2(m2);

    // thread 2
    lock_guard<mutex> lck2(m2);
    lock_guard<mutex> lck1(m1);
```

대신하여 `lock()` 을 사용하라 :

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

혹은 (더 좋은 방법이지만, C++17 에서만 가능한):

```c++
    // thread 1
    scoped_lock<mutex, mutex> lck1(m1, m2);

    // thread 2
    scoped_lock<mutex, mutex> lck2(m2, m1);
```

위 코드에서, `thread1` 과 `thread2` 의 작성자들은 여전히 `mutex` 들의 순서에 합의하지는 못했지만, 그 순서는 더 이상 문제가 되지 않는다.

##### Note

실제 코드에서는, `mutex` 들이 서로 간의 의도된 관계나 의도된 획득 순서를 프로그래머에게 쉽게 상기시킬 수 있도록 명명돼 있는 경우가 드물다.
또한 실제 코드에서는, `mutex` 들이 편리하게도 항상 연속된 줄에 획득이 이루어지지는 않는다.

C++17 에서는 간단히 아래와 같이 작성하여

```c++
    lock_guard lck1(m1, adopt_lock);
```

`mutex` 타입을 자동으로 추론되도록 할 수 있다.

##### Enforcement

복수의 `mutex` 획득을 검출해 내라.
일반적으로 이는 판별이 어려운 경우가 많지만, 흔히 볼 수 있는 간단한 (상단의 예제와 같은) 사례를 잡아내는 것은 어렵지 않다.

### <a name="Rconc-unknown"></a>CP.22: lock 을 사용 중일 때는 알 수 없는 코드를 호출하지 말아라 (예: callback)

##### Reason

어떤 동작을 할지 확실치 않은 코드를 호출하는 것은 교착 상태를 유발할 위험을 감수하는 것이다.

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

만약 당신이 `Foo::act` 가 어떤 작업을 하는지 모른다면 (해당 함수는 아직 작성되지 않은 파생 클래스의 멤버를 호출하는 가상 함수일 수 있다),
`do_this` 함수를 (재귀적으로) 부를 수도 있어 `my_mutex` 에 교착상태를 유발할 수 있다.
아니면 해당 함수가 다른 뮤텍스를 잠그고 적절한 시간 안에 해제를 하지 않아, `do_this` 를 호출하는 다른 코드에 있어 지연을 유발할 수도 있다.

##### Example

"미지의 코드 호출" 문제의 흔한 사례는 동일한 객체에 대해 잠금 권한을 얻으려 하는 함수를 호출하는 경우이다.
이러한 호출 문제는 `recursive_mutex` 를 사용함으로써 종종 해결이 가능하다. 예를 들어:

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

만약, 아마도 그러하겠지만, `f()` 가 `*this` 에 대해서 어떤 작업을 수행한다고 하면, 해당 함수를 호출하기 전에 객체의 불변 조건 (invariant) 이 유지되고 있는지 확인해야 한다.

##### Enforcement

* 재귀 용이 아닌 `mutex` 를 잠근 후 호출하는 가상 함수 호출을 표시하라
* 재귀 용이 아닌 `mutex` 를 잠근 후 호출하는 콜백 함수 호출을 표시하라

### <a name="Rconc-join"></a>CP.23: join 하는 `thread`를 유효범위 안의 컨테이너처럼 생각하라

##### Reason

포인터 사용의 안전성과 메모리 누수 방지를 위하여, 어떤 포인터들이 `thread` 에서 사용되는지 주의 깊게 살펴야 한다.
만약 `thread` 가 join 한다고 하면, `thread` 의 유효범위 및 포함되는 유효범위 내의 객체에 대한 포인터를 안전하게 전달할 수 있다.

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

`gsl::joining_thread` 는 join 하는 소멸자를 가지고 있으며 `detached()` 될 수 없는 `std::thread` 이다.
"OK" 라고 함은  `thread` 가 전달받은 포인터를 사용할 수 있는 한, 해당 객체가 유효범위 안에 ("살아") 있을 것임을 의미한다.
`thread` 들이 병행 실행된다는 사실이 여기에서는 존속 기간이나 소유권 관련 문제에 있어서 아무런 영향을 미치지 않는다;
이 `thread` 들은 단순히 `some_fct` 로부터 호출되는 함수 객체들로 볼 수 있다.

##### Enforcement

`joining_thread` 가 `detach()` 되지 않도록 확인하라.
확인이 끝난 후에는, (지역 객체들에 대한) 통상적인 존속 기간과 소유권 적용 방식을 따른다.

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

"OK" 라고 함은  `thread` 가 전달받은 포인터를 사용할 수 있는 한, 해당 객체가 유효범위 안에 ("살아") 있을 것임을 의미한다.
"bad" 라고 함은 `thread` 가 전달받은 포인터가 가리키는 객체가 소멸된 이후에 해당 포인터를 사용할 수도 있음을 의미한다.
`thread` 들이 병행 실행된다는 사실이 여기에서는 존속 기간이나 소유권 관련 문제에 있어서 아무런 영향을 미치지 않는다;
이 `thread` 들은 단순히 `some_fct` 로부터 호출되는 함수 객체들로 볼 수 있다.

##### Note

정적 저장소 존속 기간을 가진 객체들도 detach 된 스레드에서 사용 시 문제를 야기할 수 있다:
스레드가 프로그램의 종료 시점까지 수행된다면, 정적 저장소 존속 기간을 가진 객체들의 소멸 시점까지
병행 실행될 수 있으며, 그로 인해 해당 객체들에 대한 접근에 있어 경쟁이 유발될 수 있다.

##### Note

이 규칙은 [don't `detach()`](#Rconc-detached_thread) and [use `gsl::joining_thread`](#Rconc-joining_thread) 를 따른다면 불필요하다.
그러나, 해당 지침을 따르기 위해서 코드를 변경하는 것이 어렵거나 혹은 서드파티 라이브러리들의 경우에는 아예 불가능할 수도 있다.
이러한 경우에는, 이 규칙이 존속기간 및 타입 안전에 필수적일 수 있다.

일반적으로 `detach()` 가 특정 `thread` 에 대해 수행되는지는 판별이 어려운 경우가 많지만, 흔히 볼 수 있는 간단한 사례는 손쉽게 검출해 낼 수 있다.
만약 특정 `thread` 가 `detach()` 하지 않음을 입증할 수 없다면, 해당 스레드는 `detach()` 하며 최초 생성된 유효 범위를 넘어서까지 유지될 수 있다고 가정해야 한다;
확인이 끝난 후에는, (전역 객체들에 대한) 통상적인 존속기간과 소유권 적용 방식을 따른다.

##### Enforcement

지역 변수들을 `detach()` 할지도 모르는 스레드에 전달하는 시도들을 표시하라.

### <a name="Rconc-joining_thread"></a>CP.25: `std::thread` 보다는 `gsl::joining_thread` 사용을 우선하여 고려하라

##### Reason

`joining_thread`는 유효 범위가 끝날 때 join 한다.

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

`join()` 혹은 `detach()` 할지를 결정하는 코드는 매우 복잡할 수 있으며 심지어는 해당 스레드가 호출하는 함수 내에서 결정되거나 해당 스레드를 생성한 함수가 호출하는 다른 함수에 의해서 결정될 수도 있다:

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

이는 존속기간 분석을 심각할 정도로 복잡하게 만들며, 실제 일어날 수도 있는 어떤 경우들에 대해서는 분석 자체를 불가능하게도 만들 수 있다.
이는 우리가 스레드 내부에서 `use()` 함수 내의 지역 객체들을 참조하거나 `use()` 함수에서 해당 스레드 내의 지역 객체를 참조하는 것이 안전하지 않다는 것을 의미한다.

##### Note

"종료되지 않아야 하는 스레드" 들은  `detach()` 하기보다는, 전역으로 만들어 포함되는 유효 범위 내에나 혹은 자유 영역에 위치시키도록 하라.
[don't `detach`](#Rconc-detached_thread).

##### Note

`std::thread` 를 사용하는 오래된 코드나 서드파티 라이브러리들로 인해 이 규칙은 도입하기가 어려울 수 있다.

##### Enforcement

`std::thread` 이 사용되는 경우들을 표시하라:

* `gsl::joining_thread` 사용을 권장한다.
* 만약 detach 하는 경우라면, 포함되는 유효 범위에 ["exporting ownership"](#Rconc-detached_thread) 하는 것을 권장한다.
* join 을 하는지 detach 를 하는지 명확하지 않다면 심각하게 경고하라.

### <a name="Rconc-detached_thread"></a>CP.26: 스레드를 `detach()` 하지 말아라

##### Reason

보통, 생성된 유효범위를 넘어서서 실행이 유지되어야 하는 필요성은  `thread` 의 내재된 작업 특성이나,
해당 특성을 `detach` 를 사용함으로써 구현하는 것은 추적 관찰 (monitor) 및 해당 스레드와의 통신을 어렵게 만든다.
특히, (물론 불가능하지는 않지만) 스레드가 예상된 시점까지 실행이 완료되기를 보장하거나 예상된 시점까지 실행이 유지되기를 보장하는 것은 더욱 어렵다.

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

이는 `detach()` 가 흔히 사용되는, 스레드의 합리적인 사용 예이다.
그러나, 여전히 문제가 있다.
우리가 detach 된 스레드가 여전히 실행되고 있는지 어떻게 추적 관찰할 수 있는가?
때때로 heartbeat 상에 어떤 문제가 발생할 수 있으며, heartbeat 를 잃는 것은 그것을 필요하는 시스템 상에서는 심각한 문제일 수 있다.
따라서, 우리는 해당 스레드와 통신을 할 필요가 있다
(e.g., 메시지 스트림이나 `condition_variable` 을 이용한 알림 이벤트를 통해서).

많은 경우에 더 우월하다고 볼 수 있는 대안으로는 해당 스레드를 생성되는 (혹은 활성화되는)  유효 범위 밖에 배치함으로써 존속 기간을 제어하는 것이다.
예제:

```c++
    void heartbeat();

    gsl::joining_thread t(heartbeat);             // heartbeat is meant to run "forever"
```

이 heartbeat (에러나 하드웨어 문제 등을 방지하는) 는 프로그램이 실행되는 동안 유지될 것이다.

때로는, 생성 시점과 소유권 시점을 분리할 필요가 있다:

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

`detach()` 사용을 표시하라.

### <a name="Rconc-data-by-value"></a>CP.31: 스레드들 간의 작은 데이터 전달은 참조나 포인터보다는 값으로 전달하라

##### Reason

소량의 데이터 복사는 잠금장치를 이용하는 것보다 복사 및 접근 비용이 저렴하다.
복사는 자연히도 유일한 소유권을 제공하며(코드를 단순화할 수 있다) 데이터 경쟁의 가능성을 제거한다.

##### Note

"소량" 이 정확하게 어느 정도인지를 정의하기는 불가능하다.

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

`modify1` 호출은 두 번의 `string` 값 복사를 필요로 하지만, `modify2` 호출은 값 복사를 필요로 하지 않는다.
반면에 `modify1` 의 구현은 우리가 단일 스레드 환경에서 작성했을 법한 코드와 정확히 동일하나,
`modify2` 의 구현은 데이터 경쟁을 피하기 위하여 일정 형태의 잠금 기법을 이용해야 한다.
문자열이 짧다면 (10 글자 정도), `modify1` 호출은 놀라울 정도로 빠를것이다;
근본적으로 모든 필요한 비용은 `thread` 전환에 들어갈 뿐이기 때문이다. 만약 문자열이 길다면 (1,000,000 글자 정도),
이를 두 번이나 복사하는 것은 좋은 선택이 아닐 것이다.

이 논의는 `async` 등의 사용과는 아무 연관이 없음을 유의하라.
이는 메시지 전달 방식 혹은 공유 메모리 방식을 사용을 고려할 때도 동일하게 적용된다.

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

### <a name="Rconc-mutex"></a>CP.50: `mutex` 를 보호(guard) 해야 하는 데이터와 함께 선언하라. 가능한 경우에는  `synchronized_value<T>` 를 사용하라

##### Reason

코드를 읽는 사람에게 있어 데이터가 보호된다는 사실과 어떻게 보호될지를 분명히 전달한다. 이는 뮤텍스를 잠그지 않거나 엉뚱한 뮤텍스를 잠글 확률을 낮춘다.

`synchronized_value<T>` 를 사용하는 것은 해당 데이터가 뮤텍스를 가지고 있으며, 데이터 접근 시 해당되는 올바른 뮤텍스가 잠길 것임을 보장한다.
`synchronized_value` 를 이후의 TS 나 C++ 표준에 추가하기 위한 [WG21 proposal](http://wg21.link/p0290)) 을 참고하라.

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

??? 가능한가?

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

### <a name="Rconc-double"></a>CP.110: 초기화를 위한 독자적인 이중 확인 잠금 (double-checked locking) 코드를 작성하지 말라

##### Reason

C++11 이후부터는, 정적 지역 변수들이 스레드 안전 (thread-safe) 한 방식으로 초기화된다. 따라서 RAII 패턴과 함께 사용 시, 정적 지역 변수들의 사용은 초기화를 위한 독자적인 이중 확인 잠금 (double-checked locking) 코드를 작성해야 할 필요성을 대체한다. 또한 std::call_once 사용 역시 동일한 목적을 달성할 수 있다. 그러므로 독자적인 이중 확인 잠금 코드를 작성하기보다는 C++11 의 정적 지역 변수나 std::call_once 을 사용하라.

##### Example

std::call_once 를 사용하는 예제.

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

C++11 의 스레드 안전한 정적 지역 변수를 사용하는 예제.

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

### <a name="Rconc-double-pattern"></a>CP.111: 이중 확인 잠금이 꼭 필요할 경우에는 전통적인 패턴을 사용하라

##### Reason

이중 확인 잠금 코드는 엉망으로 만들기 쉽다. [CP.110: 초기화를 위한 독자적인 이중 확인 잠금 코드를 작성하지 말라](#Rconc-double) 및 [CP.100: 정말 필요할 때만 lock-free 프로그래밍을 사용하라](#Rconc-lockfree) 과 같은 규칙에도 불구하고 독자적인 이중 확인 잠금 코드를 작성해야만 한다면, 전통적인 패턴을 이용해서 작성하도록 하라.

스레드 안전하지 않은 (non-thread-safe) 특정 동작에 있어, 동작 자체가 복잡하고도 수행 빈도가 낮으며 해당 동작의 수행이 필요하지 않다는 것을 보장하는 (그 반대의 경우를 보장하기 위해서는 사용할 수 없는) 신속하고도 스레드-안전한 시험 방법이 존재하는 상황에서의 이중 확인 잠금 패턴의 사용이  [CP.110: 초기화를 위한 독자적인 이중 확인 잠금 코드를 작성하지 말라](#Rconc-double) 을 위반하지 않는 경우에 해당한다.

##### Example, bad

volatile 의 사용을 통한 아래의 첫 번째 시험은 스레드 안전하지 않다. [CP.200: `volatile`은 C++가 아닌 메모리에 대해서만 사용하라](#Rconc-volatile2) 을 참고하라.

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

획득 적재 (acquire load) 방식이 순차 일관 적재 (sequentially-consistent load) 방식보다 효율적인 경우, 미세 조정된 메모리 순서 (memory order) 를 통해 성능상 이점을 얻을 수 있다.

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

??? 저 idiom 확인할 수 있는 건가요?

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
