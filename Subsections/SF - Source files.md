#SF: Source files

Distinguish between declarations (used as interfaces) and definitions (used as implementations)
Use header files to represent interfaces and to emphasize logical structure.

Source file rule summary:

* [SF.1: Use a `.cpp` suffix for code files and `.h` for interface files](#Rs-suffix)
* [SF.2: A `.h` file may not contain object definitions or non-inline function definitions](#Rs-inline)
* [SF.3: Use `.h` files for all declarations used in multiple sourcefiles](#Rs-suffix)
* [SF.4: Include `.h` files before other declarations in a file](#Rs-include-order)
* [SF.5: A `.cpp` file must include the `.h` file(s) that defines its interface](#Rs-consistency)
* [SF.6: Use `using`-directives for transition, for foundation libraries (such as `std`), or within a local scope](#Rs-using)
* [SF.7: Don't put a `using`-directive in a header file](#Rs-using-directive)
* [SF.8: Use `#include` guards for all `.h` files](#Rs-guards)
* [SF.9: Avoid cyclic dependencies among source files](#Rs-cycles)

* [SF.20: Use `namespace`s to express logical structure](#Rs-namespace)
* [SF.21: Don't use an unnamed (anonymous) namespace in a header](#Rs-unnamed)
* [SF.22: Use an unnamed (anonymous) namespace for all internal/nonexported entities](#Rs-unnamed2)



<a name="Rs-suffix"></a>
### SF.1: Use a `.cpp` suffix for code files and `.h` for interface files

**Reason**: Convention

**Note**: The specific names `.h` and `.cpp` are not required (but recommended) and other names are in widespread use.
Examples are `.hh` and `.cxx`. Use such names equivalently.

**Example**:

	// foo.h:
	extern int a;	// a declaration
	extern void foo();
	
	// foo.cpp:
	int a;	// a definition
	void foo() { ++a; }

`foo.h` provides the interface to `foo.cpp`. Global variables are best avoided.

**Example**, bad:

	// foo.h:
	int a;	// a definition
	void foo() { ++a; }

`#include<foo.h>` twice in a program and you get a linker error for two one-definition-rule violations.
	

**Enforcement**:

* Flag non-conventional file names.
* Check that `.h` and `.cpp`` (and equivalents) follow the rules below.


<a name="Rs-inline"></a>
### SF.2: A `.h` file may not contain object definitions or non-inline function definitions

**Reason**: Including entities subject to the one-definition rule leads to linkage errors.

**Example**:

	???
	
**Alternative formulation**: A `.h` file must contain only:

* `#include`s of other `.h` files (possibly with include guards
* templates
* class definitions
* function declarations
* `extern` declarations
* `inline` function definitions
* `constexpr` definitions
* `const` definitions
* `using` alias definitions
* ???

**Enforcement**: Check the positive list above.


<a name="Rs-suffix"></a>
### SF.3: Use `.h` files for all declarations used in multiple sourcefiles

**Reason**: Maintainability. Readability.

**example, bad**:

	// bar.cpp:
	void bar() { cout << "bar\n"; }

	// foo.cpp:
	extern void bar();
	void foo() { bar(); }
	
A maintainer of `bar` cannot find all declarations of `bar` if its type needs changing.
The user of `bar` cannot know if the interface used is complete and correct. At best, error messages come (late) from the linker.

**Enforcement**:

* Flag declarations of entities in other source files not placed in a `.h`.


<a name="Rs-include-order"></a>
### SF.4: Include `.h` files before other declarations in a file

**Reason**: Minimize context dependencies and increase readability.

**Example**:

	#include<vector>
	#include<algorithms>
	#include<string>

	// ... my code here ...

**Example, bad**:

	#include<vector>
	#include<algorithms>
	#include<string>

	// ... my code here ...

**Note**: This applies to both `.h` and `.cpp` files.

**Exception**: Are there any in good code?

**Enforcement**: Easy.
	

<a name="Rs-consistency"></a>
### SF.5: A `.cpp` file must include the `.h` file(s) that defines its interface

**Reason** This enables the compiler to do an early consistency check.

**Example**, bad:

	// foo.h:
	void foo(int);
	int bar(long double);
	int foobar(int);
	
	// foo.cpp:
	void foo(int) { /* ... */ }
	int bar(double) { /* ... */ }
	double foobar(int);

Thw errors will not be caught until link time for a program calling `bar` or `foobar`.

**Example**:

	// foo.h:
	void foo(int);
	int bar(long double);
	int foobar(int);
	
	// foo.cpp:
	#include<foo.h>
		
	void foo(int) { /* ... */ }
	int bar(double) { /* ... */ }
	double foobar(int);		// error: wrong return type

The return-type error for `foobar` is now caught immediately when `foo.cpp` is compiled.
The argument-type error for `bar` cannot be caught until link time because of the possibility of overloading,
but systematic use of `.h` files increases the likelyhood that it is caught earlier by the programmer.

**Enforcement**: ???


<a name="Rs-using"></a>
### SF.6: Use `using`-directives for transition, for foundation libraries (such as `std`), or within a local scope

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rs-using-directive"></a>
### SF.7: Don't put a `using`-directive in a header file

**Reason** Doing so takes away an `#include`r's ability to effectively disambiguate and to use alternatives.

**Example**:

	???

**Enforcement**: ???


<a name="Rs-guards"></a>
### SF.8: Use `#include` guards for all `.h` files

**Reason**: To avoid files being `#include`d several times.

**Example**:

	// file foobar.h:
	#ifndef FOOBAR_H
	#define FOOBAR_H
	// ... declarations ...
	#endif // FOOBAR_H

**Enforcement**: Flag `.h` files without `#include` guards


<a name="Rs-cycles"></a>
### SF.9: Avoid cyclic dependencies among source files

**Reason**: Cycles complicates comprehension and slows down compilation.
Complicates conversion to use language-supported modules (when they become available).

**Note**: Eliminate cycles; don't just break them with `#include` guards.

**Example, bad**:

	// file1.h:
	#include "file2.h"
	
	// file2.h:
	#include "file3.h"
	
	// file3.h:
	#include "file1.h"

**Enforcement: Flag all cycles.


<a name="Rs-namespace"></a>
### SF.20: Use `namespace`s to express logical structure

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rs-unnamed"></a>
### SF.21: Don't use an unnamed (anonymous) namespace in a header

**Reason**: It is almost always a bug to mention an unnamed namespace in a header file.

**Example**:

	???

**Enforcement**:
* Flag any use of an anonymous namespace in a header file.



<a name="Rs-unnamed2"></a>
### SF.22: Use an unnamed (anonymous) namespace for all internal/nonexported entities

**Reason**:
nothing external can depend on an entity in a nested unnamed namespace.
Consider putting every definition in an implementation source file should be in an unnamed namespace unless that is defining an "external/exported" entity.

**Example**: An API class and its members can't live in an unnamed namespace; but any "helper" class or function that is defined in an implementation source file should be at an unnamed namespace scope.

	???

**Enforcement**:
* ???