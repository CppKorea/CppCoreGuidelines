# PER: Performance  
# 성능  

???should this section be in the main guide???

This section contains rules for people who needs high performance or low-latency.
That is, rules that relates to how to use as little time and as few resources as possible to achieve a task in a predictably short time.
The rules in this section are more restrictive and intrusive than what is needed for many (most) applications.
Do not blindly try to follow them in general code because achieving the goals of low latency requires extra work.

이 장에서는 높은 성능 또는 저 지연을 필요로 하는 사람들을 위한 규칙을 포함하고 있다. 다시 말해 예상하는 짧은 시간 안에 
태스크를 완료하기 위해 가능한 적은 시간과 리소스를 어떻게 사용하는지와 관련된 규칙이다.
그래서 이러한 규칙은 대부분의 어플리케이션에 필요한 것보다 더 제한적이고 어떻게 보면 거슬릴 수도 있는 것이다.
저 지연 목적을 달성하는 것은 추가적인 노력이 필요하기 때문에 일반적은 코드에 이러한 것들을 맹목적으로 따라 하지 말았으면 좋겠다.

Performance rule summary:

* [PER.1: Don't optimize without reason](#Rper-reason)
* [PER.2: Don't optimize prematurely](#Rper-Knuth)
* [PER.3: Don't optimize something that's not performance critical](#Rper-critical)
* [PER.4: Don't assume that complicated code is necessarily faster than simple code](#Rper-simple)
* [PER.5: Don't assume that low-level code is necessarily faster than high-level code](#Rper-low)
* [PER.6: Don't make claims about performance without measurements](#Rper-measure)
* [PER.10: Rely on the static type system](#Rper-type)
* [PER.11: Move computation from run time to compile time](#Rper-Comp)
* [PER.12: Eliminate redundant aliases](#Rper-alias)
* [PER.13: Eliminate redundant indirections](#Rper-indirect)
* [PER.14: Minimize the number of allocations and deallocations](#Rper-alloc)
* [PER.15: Do not allocate on a critical branch](#Rper-alloc0)
* [PER.16: Use compact data structures](#Rper-compact)
* [PER.17: Declare the most used member of a time critical struct first](#Rper-struct)
* [PER.18: Space is time](#Rper-space)
* [PER.19: Access memory predictably](#Rper-access)
* [PER.30: Avoid context switches on the critical path](#Rper-context)


<a name="Rper-reason"></a>
### PER.1: Don't optimize without reason  
### 이유없이 최적화하지 마라.  

**Reason**: If there is no need for optimization, the main result of the effort will be more errors and higher maintenance costs.

**Note**: Some people optimize out of habit or because it's fun.

???


<a name="Rper-Knuth"></a>
### PER.2: Don't optimize prematurely  
### 성급하게 최적화하지 마라

**Reason**: Elaborately optimized code is usually larger and harder to change than unoptimized code.

???


<a name="Rper-critical"></a>
### PER.3: Don't optimize something that's not performance critical  
### 성능 크리티컬하지 않는 곳에는 최적화를 하지 마라.  
**Reason**: Optimizing a non-performance-critical part of a program has no effect on system performance.

**Note**: If your program spends most of its time waiting for the web or for a human, optimization of in-memory computation is problably useless.

???



<a name="Rper-simple"></a>
### PER.4: Don't assume that complicated code is necessarily faster than simple code  
### 복잡한 코드가 간단한 코드보다 빠르다고 추측하지 마라.  

**Reason**: Simple code can be very fast. Optimizers sometimes do marvels with simple code

**Note**: ???

???


<a name="Rper-low"></a>
### PER.5: Don't assume that low-level code is necessarily faster than high-level code
### 저 수준 코드가 고 수준 코드보다 필연적으로 빠를 것이라고 추측하지 마라.

**Reason**: Low-level code sometimes inhibits optimizations. Optimizers sometimes do marvels with high-level code

**Note**: ???

???


<a name="Rper-measure"></a>
### PER.6: Don't make claims about performance without measurements
### 측정없는 성능 개선을 주장하지 마라.

**Reason**: The field of performance is littered with myth and bogus folklore.
Modern hardware and optimizers defy naive assumptions; even experts are regularly surprised.

**Note**: Getting good performance measurements can be hard and require specialized tools.

**Note**: A few simple microbenchmarks using Unix `time` or the standard library `<chrono>` can help dispell the most obvious myths.
If you can't measure your complete system accurately, at least try to measure a few of your key operations and algorithms.
A profiler can help tell you which parts of your system are performance critical.
Often, you will be surprised.

???


<a name="Rper-type"></a>
### PER.10: Rely on the static type system
### 정적 타입 시스템에 의지해라.

**Reason**: Type violations, weak types (e.g. `void*`s), and low level code (e.g., manipulation of sequences as individual bytes)
make the job of the optimizer much harder. Simple code often optimizes better than hand-crafted complex code.

???


<a name="Rper-Comp"></a>
### PER.11: Move computation from run time to compile time
### 계산을 런 타임에서 컴파일 타임으로 옮겨라.

???


<a name="Rper-alias"></a>
### PER.12: Eliminate redundant aliases
### 불필요한 별칭을 제거해라. 
???


<a name="Rper-indirect"></a>
### PER.13: Eliminate redundant indirections  
### 불필요한 간접 접근을 제거해라.  

???


<a name="Rper-alloc"></a>
### PER.14: Minimize the number of allocations and deallocations  
### 할당과 반납의 수를 최소화해라.  

???


<a name="Rper-alloc0"></a>
### PER.15: Do not allocate on a critical branch  

???


<a name="Rper-compact"></a>
### PER.16: Use compact data structures  
### 간결한(?) 데이타 구조체을 사용해라.  

**Reason**: Performance is typically dominated by memory access times.

???


<a name="Rper-struct"></a>
### PER.17: Declare the most used member of a time critical struct first  
### 빠른 처리를 요구하는 구조체에서 가장 많이 쓰이는 멤버를 먼저 선언해라.  

???


<a name="Rper-space"></a>
### PER.18: Space is time  
### 공간은 시간이다.  

**Reason**: Performance is typically dominated by memory access times.

???


<a name="Rper-access"></a>
### PER.19: Access memory predictably  
### 예측할 수 있는 메모리 접근을 해라. 

**Reason**: Performance is very sensitive to cache performance and cache algorithms favor simple (usually linear) access to adjacent data.

???


<a name="Rper-context"></a>
### PER.30: Avoid context switches on the critical path  
### 크리티컬 패스에서 문맥 전환을 피해라.  

???
