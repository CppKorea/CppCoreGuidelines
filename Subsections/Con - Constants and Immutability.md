# Con: Constants and Immutability

You can't have a race condition on a constant.
it is easier to reason about a program when many of the objects cannot change threir values.
Interfaces that promises "no change" of objects passed as arguments greatly increase readability.

Constant rule summary:

* [Con.1: By default, make objects immutable](#Rconst-immutable)
* [Con.2: By default, make member functions `const`](#Rconst-fct)
* [Con.3: By default, pass pointers and references to `const`s](#Rconst-ref)
* [Con.4: Use `const` to define objects with values that do not change after construction](#Rconst-const)
* [Con.5: Use `constexpr` for values that can be computed at compile time](#Rconst-constexpr)


<a name="Rconst-immutable"></a>
### Con.1: By default, make objects immutable

**Reason**: Immutable objects are easier to reason about, so make object non-`const` only when there is a need to change their value.

**Example**:

	for (
	container
	???

**Enforcement**: ???


<a name="Rconst-fct"></a>
### Con.2: By default, make member functions `const`

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rconst-ref"></a>
### Con.3: By default, pass pointers and references to `const`s

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rconst-const"></a>
### Con.4: Use `const` to define objects with values that do not change after construction

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Rconst-constexpr"></a>
### Con.5: Use `constexpr` for values that can be computed at compile time

**Reason**: ???

**Example**:

	???

**Enforcement**: ???