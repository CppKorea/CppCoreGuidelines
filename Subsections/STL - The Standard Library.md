<a name="S-stdlib"></a>
# STL: The Standard Library
STL: 표준 라이브러리

Using only the bare language, every task is tedious (in any language).
Using a suitable library any task can be reasonably simple.
단순히 언어 자체만 사용하게 되면 모든 작업이 더디게 되고(어떤 언어라도), 반면 적절한 라이브러리를 사용하게 되면 어떤 작업도 상당히 단순해진다. 

Standard-library rule summary:
표준 라이브러리에 대한 기준 요약:

* [STL.1: Use libraries wherever possible](#Rstl-lib)
* STL.1: 가능한한 어디든지 라이브러리를 많이 사용한다 
* [STL2.: Prefer the standard library to other libraries](#Rstl-stl)
* STL.2: 가능하다면 표준 라이브러리를 사용한다
* ???

<a name="Rstl-lib"></a>
### STL.1:  Use libraries wherever possible
STL.1: 어디든지 라이브러리를 가능한한 많이 사용한다

**Reason**: Save time. Don't re-invent the wheel.
Don't replicate the work of others.
Benefit from other people's work when they make improvements.
Help other people when you make improvements.
이유: 시간을 절약하고,처음부터 다시 만들지 않아도 된다. 다른 사람들이 이미 작업해 놓은 바를 중복작업할 필요가 없고, 다른 사람들이 향후 개선을 하게 되면 그 혜택을 누릴 수 있다. 또한 내가 직접 개선함으로서 다른 사람들을 도울 수 있다.

**References**: ???

<a name="Rstl-stl"></a>
### STL2.: Prefer the standard library to other libraries
STL.2:가능하다면 표준 라이브러리를 사용한다

**Reason**. More people know the standard library.
It is more likely to be stable, well-maintained, and widely available than your own code or most other libraries.
이유: 많은 사람들이 표준 라이브러리를 알고 있으므로 스스로 작성한 코드나 다른 라이브러리 보다 더 안정적이고, 더 잘 관리되고, 더 광범위한 곳에 사용할 수 있다.


## STL.con: Containers

???

## STL.str: String

???

## STL.io: Iostream

???

### STL.???: Use character-level input only when you have to; _expr.low_.

### STL.???: When reading, always consider ill-formed input; _expr.low_.

## STL.regex: Regex

???

## STL:c: The C standard library

### STL.???: C-style strings

### STL.???: printf/scanf
