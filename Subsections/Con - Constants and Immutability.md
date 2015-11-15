# <a name="S-const"></a> Con: Constants and Immutability

상수에 대해서는 경쟁상태가 아예 없다.
내부의 많은 객체가 값을 바꾸지 않는 프로그램이라면 해석하기가 쉬울 것이다.
인자로 넘어가는 객체의 상태가 바뀌지 않는 것을 보장한다면 그 인터페이스는 가독성이 굉장히 높을 것이다.
You can't have a race condition on a constant.
it is easier to reason about a program when many of the objects cannot change their values.
Interfaces that promises "no change" of objects passed as arguments greatly increase readability.

상수 규칙 요약:
Constant rule summary:

* [Con.1: By default, make objects immutable](#Rconst-immutable)
* [Con.2: By default, make member functions `const`](#Rconst-fct)
* [Con.3: By default, pass pointers and references to `const`s](#Rconst-ref)
* [Con.4: Use `const` to define objects with values that do not change after construction](#Rconst-const)
* [Con.5: Use `constexpr` for values that can be computed at compile time](#Rconst-constexpr)

### <a name="Rconst-immutable"></a> Con.1: 기본적으로 값이 안 바뀌는 객체를 만들어라.
>### <a name="Rconst-immutable"></a> Con.1: By default, make objects immutable

##### Reason

변하지 않는 객체는 해석하기가 더 쉽다. 그래서 값을 바꿀 필요가 있을 때에만 객체를 비`const`로 만든다.
>Immutable objects are easier to reason about, so make object non-`const` only when there is a need to change their value.

##### Example

    for (
    container
    ???

##### Enforcement

???

### <a name="Rconst-fct"></a> Con.2: 기본적으로 멤버 함수를 `const`로 만들어라.
>### <a name="Rconst-fct"></a> Con.2: By default, make member functions `const`

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Rconst-ref"></a> Con.3: 기본적으로 포인터, 참조자를 `const`인자로 넘겨라.
>### <a name="Rconst-ref"></a> Con.3: By default, pass pointers and references to `const`s

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Rconst-const"></a> Con.4: 생성 후에 값이 안 바뀐다면 `const` 객체를 사용하라.
>### <a name="Rconst-const"></a> Con.4: Use `const` to define objects with values that do not change after construction

##### Reason

 ???

##### Example

    ???

##### Enforcement

???

### <a name="Rconst-constexpr"></a> Con.5: 컴파일 시에 계산할 수 있는 값이라면 `constexpr`를 사용하라.
>### <a name="Rconst-constexpr"></a> Con.5: Use `constexpr` for values that can be computed at compile time

##### Reason

 ???

##### Example

    ???

##### Enforcement

???
