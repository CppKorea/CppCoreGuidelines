# PER: Performance

???should this section be in the main guide???

This section contains rules for people who needs high performance or low-latency.
That is, rules that relates to how to use as little time and as few resources as possible to achieve a task in a predictably short time.
The rules in this section are more restrictive and intrusive than what is needed for many (most) applications.
Do not blindly try to follow them in general code because achieving the goals of low latency requires extra work.

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

**Reason**: If there is no need for optimization, the main result of the effort will be more errors and higher maintenance costs.

**Note**: Some people optimize out of habit or because it's fun.

???


<a name="Rper-Knuth"></a>
### PER.2: Don't optimize prematurely

**Reason**: Elaborately optimized code is usually larger and harder to change than unoptimized code.

???


<a name="Rper-critical"></a>
### PER.3: Don't optimize something that's not performance critical

**Reason**: Optimizing a non-performance-critical part of a program has no effect on system performance.

**Note**: If your program spends most of its time waiting for the web or for a human, optimization of in-memory computation is problably useless.

???



<a name="Rper-simple"></a>
### PER.4: Don't assume that complicated code is necessarily faster than simple code

**Reason**: Simple code can be very fast. Optimizers sometimes do marvels with simple code

**Note**: ???

???


<a name="Rper-low"></a>
### PER.5: Don't assume that low-level code is necessarily faster than high-level code

**Reason**: Low-level code sometimes inhibits optimizations. Optimizers sometimes do marvels with high-level code

**Note**: ???

???


<a name="Rper-measure"></a>
### PER.6: Don't make claims about performance without measurements

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

**Reason**: Type violations, weak types (e.g. `void*`s), and low level code (e.g., manipulation of sequences as individual bytes)
make the job of the optimizer much harder. Simple code often optimizes better than hand-crafted complex code.

???


<a name="Rper-Comp"></a>
### PER.11: Move computation from run time to compile time

???


<a name="Rper-alias"></a>
### PER.12: Eliminate redundant aliases

???


<a name="Rper-indirect"></a>
### PER.13: Eliminate redundant indirections

???


<a name="Rper-alloc"></a>
### PER.14: Minimize the number of allocations and deallocations

???


<a name="Rper-alloc0"></a>
### PER.15: Do not allocate on a critical branch

???


<a name="Rper-compact"></a>
### PER.16: Use compact data structures

**Reason**: Performance is typically dominated by memory access times.

???


<a name="Rper-struct"></a>
### PER.17: Declare the most used member of a time critical struct first

???


<a name="Rper-space"></a>
### PER.18: Space is time

**Reason**: Performance is typically dominated by memory access times.

???


<a name="Rper-access"></a>
### PER.19: Access memory predictably

**Reason**: Performance is very sensitive to cache performance and cache algorithms favor simple (usually linear) access to adjacent data.

???


<a name="Rper-context"></a>
### PER.30: Avoid context switches on the critical path

???