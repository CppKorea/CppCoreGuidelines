# To-do: Unclassified proto-rules

This is our to-do list.
Eventually, the entries will become rules or parts of rules.
Aternatively, we will decide that no change is needed and delete the entry.

* No long-distance friendship
* Should physical design (what's in a file) and large-scale design (libraries, groups of libraries) be addressed?
* Namespaces
* How granular should namespaces be? All classes/functions designed to work together and released together (as defined in Sutter/Alexandrescu) or something narrower or wider?
* Should there be inline namespaces (a-la `std::literals::*_literals`)?
* Avoid implicit conversions
* Const member functions should be thread safe "¦ aka, but I don't really change the variable, just assign it a value the first time its called "¦ argh
* Always initialize variables, use initialization lists for member variables.
* Anyone writing a public interface which takes or returns void* should have their toes set on fire. Â Â That one has been a personal favourite of mine for a number of years. :)
* Use `const`'ness wherever possible: member functions, variables and (yippee) const_iterators
* Use `auto`
* `(size)` vs. `{initializers}` vs. `{Extent{size}}`
* Don't overabstract
* Never pass a pointer down the call stack
* falling through a function bottom
* Should there be guidelines to choose between polymorphisms? YES. classic (virtual functions, reference semantics) vs. Sean Parent style (value semantics, type-erased, kind of like std::function)  vs. CRTP/static? YES Perhaps even vs. tag dispatch?
* Speaking of virtual functions, should non-virtual interface be promoted? NO. (public non-virtual foo() calling private/protected do_foo())? Not a new thing, seeing as locales/streams use it, but it seems to be under-emphasized.
* should virtual calls be banned from ctors/dtors in your guidelines? YES. A lot of people ban them, even though I think it's a big strength of C++ that they are ??? -preserving (D disappointed me so much when it went the Java way). WHAT WOULD BE A GOOD EXAMPLE?
* Speaking of lambdas, what would weigh in on the decision between lambdas and (local?) classes in algorithm calls and other callback scenarios?
* And speaking of std::bind, Stephen T. Lavavej criticizes it so much I'm starting to wonder if it is indeed going to fade away in future. Should lambdas be recommended instead?
* What to do with leaks out of temporaries? : `p = (s1+s2).c_str();`
* pointer/iterator invalidation leading to dangling pointers

	void bad()
	{
		int* p = new int[700];
		    int* q = &p[7];
    	delete p;

    	vector<int> v(700);
    	int* q2 = &v[7];
    	v.resize(900);

  	  // ... use q and q2 ...
	}
	
* LSP
* private inheritance vs/and membership
* avoid static class members variables (race conditions, almost-global variables)

* Use RAII lock guards (`lock_guard`, `unique_lock`, `shared_lock`), never call `mutex.lock` and `mutex.unlock` directly (RAII)
* Prefer non-recursive locks (often used to work around bad reasoning, overhead)
* Join your threads! (because of `std::terminate` in destructor if not joined or detached... is there a good reason to detach threads?) -- ??? could support library provide a RAII wrapper for `std::thread`?
* If two or more mutexes must be acquired at the same time, use `std::lock` (or another deadlock avoidance algorithm?)
* When using a `condition_variable`, always protect the condition by a mutex (atomic bool whose value is set outside of the mutex is wrong!), and use the same mutex for the condition variable itself.
* Never use `atomic_compare_exchange_strong` with `std::atomic<user-defined-struct>` (differences in padding matter, while `compare_exchange_weak` in a loop converges to stable padding)
* individual `shared_future` objects are not thread-safe: two threads cannot wait on the same `shared_future` object (they can wait on copies of a `shared_future` that refer to the same shared state)
* individual `shared_ptr` objects are not thread-safe: a thread cannot call a non-const member function of `shared_ptr` while another thread accesses (but different threads can call non-const member functions on copies of a `shared_ptr` that refer to the same shared object)

* rules for arithmetic