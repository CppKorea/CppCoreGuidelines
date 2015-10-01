# Enum: Enumerations

Enumerations are used to define sets of integer values and for defining types for such sets of values. There are two kind of enumerations, "plain" `enum`s and `class enum`s.

Enumeration rule summary:

* [Enum.1: Prefer enums over macros](#Renum-macro)
* [Enum.2: Use enumerations to represent sets of named constants](#Renum-set)
* [Enum.3: Prefer class enums over ``plain'' enums](#Renum-class)
* [Enum.4: Define operations on enumerations for safe and simple use](#Renum-oper)
* [Enum.5: Don't use ALL_CAPS for enumerators](#Renum-caps)
* [Enum.6: Use unnamed enumerations for ???](#Renum-unnamed)
* ???


<a name="Renum-macro"></a>
### Enum.1: Prefer enums over macros

**Reason**: Macros do not obey scope and type rules.

**Example**:

	???

**Enforcement**: ???


<a name="Renum-set"></a>
### Enum.2: Use enumerations to represent sets of named constants

**Reason**: ???

**Example**:

	???

**Enforcement**: ???


<a name="Renum-class"></a>
### Enum.3: Prefer class enums over ``plain'' enums

**Reason**: to minimize surprises

**Example**:

	???

**Enforcement**: ???


<a name="Renum-oper"></a>
### Enum.4: Define operations on enumerations for safe and simple use

**Reason**: Convenience of us and avoidance of errors.

**Example**:

	???

**Enforcement**: ???


<a name="Renum-caps"></a>
### Enum.5: Don't use ALL_CAPS for enumerators

**Reason**: Avoid clashes with macros

**Example**:

	???

**Enforcement**: ???


<a name="Renum-unnamed"></a>
### Enum.6: Use unnamed enumerations for ???

**Reason**: ???

**Example**:

	???

**Enforcement**: ???