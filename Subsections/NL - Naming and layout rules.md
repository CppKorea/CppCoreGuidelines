# NL: Naming and layout rules

Consistent naming and layout are helpful. If for no other reason because it minimizes "my style is better than your style" arguments.
However, there are many, many, different styles around and people are passionate about them (pro and con).
Also, most real-world projects includes code from many sources, so standardizing on a single style for all code is often impossible.
We present a set of rules that you might use if you have no better ideas, but the real aim is consistency, rather than any particular rule set.
IDEs and tools can help (as well as hinder).

Naming and layout rules:

* [NL 1: Don't say in comments what can be clearly stated in code](#Rl-comments)
* [NL.2: State intent in comments](#Rl-comments-intent)
* [NL.3: Keep comments crisp](#Rl-comments-crisp)
* [NL.4: Maintain a consistent indentation style](#Rl-indent)
* [NL.5: Don't encode type information in names](#Rl-name-type)
* [NL.6: Make the length of a name roughly proportional to the length of its scope](#Rl-name-length)
* [NL.7: Use a consistent naming style](#Rl-name)
* [NL 9: Use ALL_CAPS for macro names only](Rl-space)
* [NL.10: Avoid CamelCase](#Rl-camel)
* [NL.15: Use spaces sparingly](#Rl-space)
* [NL.16: Use a conventional class member declaration order](#Rl-order)
* [NL.17: Use K&R-derived layout](#Rl-knr)
* [NL.18: Use C++-style declarator layout](#Rl-ptr)

Most of these rules are aesthetic and programmers hold strong opinions.
IDEs also tend to have defaults and a range of alternatives.These rules are suggested defaults to follow unless you have reasons not to.

More specific and detailed rules are easier to enforce.


<a name="Rl-comments"></a>
###  NL.1: Don't say in comments what can be clearly stated in code

**Reason**: Compilers do not read comments.
Comments are less precise than code.
Comments are not updates as consistently as code.

**Example, bad**:

	auto x = m*v1 + vv;	// multiply m with v1 and add the result to vv

**Enforcement**: Build an AI program that interprets colloqual English text and see if what is said could be better expressed in C++.


<a name="Rl-comments intent"></a>
###  NL.2: State intent in comments

**Reason**: Code says what is done, not what is supposed to be done. Often intent can be stated more clearly and concisely than the implementation.

**Example**:

	void stable_sort(Sortable& c)
		// sort c in the order determined by <, keep equal elements (as defined by ==) in their original relative order
	{
		// ... quite a few lines of non-trivial code ...
	}

**Note**: If the comment and the code disagrees, both are likely to be wrong.


<a name="Rl-comments crisp"></a>
### NL.3: Keep comments crisp

**Reason**: Verbosity slows down understanding and makes the code harder to read by spreading it around in the source file.

**Enforcement**: not possible.


<a name="Rl-indent"></a>
### NL.4: Maintain a consistent indentation style

**Reason**: Readability. Avoidance of "silly mistakes."

**Example, bad**:

	int i;
	for (i=0; i<max; ++i); // bug waiting to happen
	if (i==j)
		return i;

Enforcement: Use a tool.


<a name="Rl-name type"></a>
### NL.5 Don't encode type information in names

**Rationale**: If names reflects type rather than functionality, it becomes hard to change the types used to provide that functionality.
Names with types encoded are either verbose or cryptic.
Hungarian notation is evil (at least in a strongly statically-typed language).

**Example**:

	???

**Note**: Some styles distinguishes members from local variable, and/or from global variable.

	struct S {
		int m_;
		S(int m) :m_{abs(m)) { }
	};

This is not evil.

**Note**: Some styles distinguishes types from non-types.

	typename<typename T>
	class Hash_tbl {	// maps string to T
		// ...
	};
	
	Hash_tbl<int> index;

This is not evil.


<a name="Rl-name length"></a>
### NL.7: Make the length of a name roughly proportional to the length of its scope

**Rationale**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rl-name"></a>
### NL.8: Use a consistent naming style

**Rationale**: Consistence in naming and naming style increases readability.

**Note**: Where are many styles and when you use multiple libraries, you can't follow all their differences conventions.
Choose a "house style", but leave "imported" libraries with their original style.

**Example**, ISO Standard, use lower case only and digits, separate words with underscores:

	int
	vector
	my_map

Avoid double underscores `__`

**Example**: [Stroustrup](http://www.stroustrup.com/Programming/PPP-style.pdf):
ISO Standard, but with upper case used for your own types and concepts:

	int
	vector
	My_map

**Example**: CamelCase: capitalize each word in a multi-word identifier

	int
	vector
	MyMap
	myMap

Some conventions capitalize the first letter, some don't.

**Note**: Try to be consistent in your use of acronyms, lengths of identifiers:

	int mtbf {12};
	int mean_time_between_failor {12};		// make up your mind

**Enforcement**: Would be possible except for the use of libraries with varying conventions.
	


<a name="Rl-space"></a>
### NL 9: Use ALL_CAPS for macro names only

**Reason**: To avoid confusing macros from names that obeys scope and type rules

**Example**:

	???
	
**Note**: This rule applies to non-macro symbolic constants

	enum bad { BAD, WORSE, HORRIBLE }; // BAD
	
**Enforcement**:

* Flag macros with lower-case letters
* Flag ALL_CAPS non-macro names



<a name="Rl-camel"></a>
### NL.10: Avoid CamelCase

**Reason**: The use of underscores to separate parts of a name is the originale C and C++ style and used in the C++ standard library.
If you prefer CamelCase, you have to choose among different flavors of camelCase.

**Example**:

	???
	
**Enforcement**: Impossible.


<a name="Rl-space"></a>
### NL.15: Use spaces sparingly

**Reason**: Too much space makes the text larger and distracts.

**Example, bad**:

	#include < map >
	
	int main ( int argc , char * argv [ ] )
	{
		// ...
	}
	
**Example**:

	#include<map>
	
	int main(int argc, char* argv[])
	{
		// ...
	}

**Note**: Some IDEs have their own opinios and adds distracting space.

**Note**: We value well-placed whitespace as a significant help for readability. Just don't overdo it.



<a name="Rl-order"></a>
### NL.16: Use a conventional class member declaration order

**Reason**: A conventional order of members improves readability.

When declaring a class use the following order

* types: classes, enums, and aliases (`using`)
* constructors, assignments, destructor
* functions
* data

Used the `public` before `protected` before `private` order.

Private types and functions can be placed with private data.

**Example**:

	???

**Enforcement**: Flag departures from the suggested order. There will be a lot of old code that doesn't follow this rule.


<a name="Rl-knr"></a>
### NL.17: Use K&R-derived layout

**Reason**: This is the original C and C++ layout. It preserves vertical space well. It distinguishes different language constructs (such as functions and classes well).

**Note**: In the context of C++, this style is often called "Stroustrup".

**Example**:

	struct Cable {
		int x;
		// ...
	};
	
	double foo(int x)
	{
		if (0<x) {
			// ...
		}
		
		switch (x) {
		case 0:
			// ...
			break;
		case amazing:
			// ...
			break;
		default:
			// ...
			break;
		}
		
		if (0<x)
			++x;
			
		if (x<0)
			something();
		else
			something_else();
		
		return some_value;
	}

**Note**: a space between `if` and `(`

**Note**: Use separate lines for each statement, the branches of an `if`, and the body of a `for`.

**Note** the `{` for a `class` and a `struct` in *not* on a separate line, but the `{` for a function is.

**Note**: Capitalize the names of your user-defined types to distinguish them from standards-library types.

**Note**: Do not capitalize function names.
		
**Enforcement**: If you want enforcement, use an IDE to reformat.
 

<a name="Rl-ptr"></a>
### NL.18: Use C++-style declarator layout

**Reason**: The C-style layout emphasizes use in expressions and grammar, whereas the C++-style emphasizes types.
The use in expressions argument doesn't hold for references.

**Example**:

	T& operator[](size_t);	// OK
	T &operator[](size_t);	// just strange
	T & operator[](size_t);	// undecided

**Enforcement**: Impossible in the face of history.