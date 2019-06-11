# <a name="main"></a>C++ 핵심 가이드라인

> 2018년 9월 2일

편집자:

* [Bjarne Stroustrup](http://www.stroustrup.com)
* [Herb Sutter](http://herbsutter.com/)

이 문서는 오픈 소스 (코드) 프로젝트로, 지속적으로 개선되고 있으며 0.8 버전까지 배포되었다.
이 프로젝트의 복사, 사용, 수정, 파생 성과물과 관련된 저작권은 MIT와 유사한 라이선스를 따른다. 프로젝트에 기여하기 위해서는 기여자 라이선스(Contributor License)에 동의해야 한다. 보다 자세한 사항은 [LICENSE](../LICENSE)을 확인하라.
오픈 소스 프로젝트에 익숙한 사용자들이 사용, 복사, 수정, 파생 성과물들을 통해 건설적인 정보를 제공해주기 바란다.

개선 사항에 대한 의견과 제안은 대부분 환영한다. C++ 언어와 사용 가능한 라이브러리들이 발전하고, 이에 대한 이해가 깊어질수록 이 문서를 지속적으로 수정하고 확장할 것이다. 의견을 보내고 싶다면, [소개](#S-introduction)에 있는 목적과 방법을 참고하기 바란다.
기여자 목록은 [여기](#SS-ack)를 참고하기 바란다.

참고 사항:

* 이하의 규칙들은 완성도, 일관성, 시행 가능성 등에 대해서 아직 철저하게 검증되지 않았다
* (???)로 표시되어 있는 부분은 아직 관련 정보를 기입하지 않았음을 의미한다
* 참조(Reference) 부분은 갱신이 필요하다; C++11 이전의 오래된 소스 코드들이 아직 많이 있다.
* 앞으로 해야 할 일에 대한 최신 목록은 [To-do: 미분류 규칙 초안](./Unclassified.md)에서 확인할 수 있다

[이 가이드의 범위와 구조](#S-abstract)를 확인하거나, 읽고 싶은 부분을 클릭해 해당 내용으로 바로 이동할 수 있다.

* [In: 소개](./Instruction.md)
* [P: 철학](./Philosophy.md)
* [I: 인터페이스](./Interfaces.md)
* [F: 함수](./Functions.md)
* [C: 클래스와 클래스 계층 구조](./Class.md)
* [Enum: 열거형](./Enum.md)
* [R: 리소스 관리](./Resource.md)
* [ES: 표현식과 문장](./Expr.md)
* [Per: 성능](./Performance.md)
* [CP: 동시성과 병렬처리](./Concurrency.md)
* [E: 오류 처리](./Errors.md)
* [Con: 상수와 불변성](./Const.md)
* [T: 템플릿과 제너릭 프로그래밍](./Templates.md)
* [CPL: C 스타일 프로그래밍](./CPL.md)
* [SF: 소스 파일](./Source.md)
* [SL: 표준 라이브러리](./SL.md)

참고할 만한 내용:

* [A: 설계 아이디어](./Architecture.md)
* [NR: 규칙이 아닌 것들과 근거없는 이야기들](./Not.md)
* [RF: 참고 자료들](./References.md)
* [Pro: 프로파일](./Profile.md)
* [GSL: 가이드라인 지원 라이브러리](./GSL.md)
* [NL: 이름과 만듦새 규칙들](./Naming.md)
* [FAQ: 자주 묻는 질문에 대한 대답](./FAQ.md)
* [Appendix A: 라이브러리](./appendix/Libraries.md)
* [Appendix B: 모던 C++ 코드](./appendix/Modernizing.md)
* [Appendix C: 토론](./appendix/DIscussion.md)
* [Appendix D: 유용한 도구들](./appendix/Tools.md)
* [용어 해설](./Glossary.md)
* [To-do: 미분류 규칙](./Unclassified.md)

구체적인 언어 기능과 관련된 규칙들:

* assignment
  * [regular types](#Rc-regular)
  * [prefer initialization](#Rc-initialize)
  * [copy](#Rc-copy-semantic)
  * [move](#Rc-move-semantic)
  * [other operations](#Rc-matched)
  * [default](#Rc-eqdefault)
* `class`
  * [data](#Rc-org)
  * [invariant](#Rc-struct)
  * [members](#Rc-member)
  * [helpers](#Rc-helper)
  * [concrete types](#SS-concrete)
  * [ctors, =, and dtors](#S-ctor)
  * [hierarchy](#SS-hier)
  * [operators](#SS-overload)
* `concept`:
  * [rules](#SS-concepts)
  * [in generic programming](#Rt-raise)
  * [template arguments](#Rt-concepts)
  * [semantics](#Rt-low)
* constructor:
  * [invariant](#Rc-struct)
  * [establish invariant](#Rc-ctor)
  * [`throw`](#Rc-throw)
  * [default](#Rc-default0)
  * [not needed](#Rc-default)
  * [`explicit`](#Rc-explicit)
  * [delegating](#Rc-delegating)
  * [`virtual`](#Rc-ctor-virtual)
* derived `class`:
  * [when to use](#Rh-domain)
  * [as interface](#Rh-abstract)
  * [destructors](#Rh-dtor)
  * [copy](#Rh-copy)
  * [getters and setters](#Rh-get)
  * [multiple inheritance](#Rh-mi-interface)
  * [overloading](#Rh-using)
  * [slicing](#Rc-copy-virtual)
  * [`dynamic_cast`](#Rh-dynamic_cast)
* destructor:
  * [and constructors](#Rc-matched)
  * [when needed?](#Rc-dtor)
  * [may not fail](#Rc-dtor-fail)
* exception:
  * [errors](#S-errors)
  * [`throw`](#Re-throw)
  * [for errors only](#Re-errors)
  * [`noexcept`](#Re-noexcept)
  * [minimize `try`](#Re-catch)
  * [what if no exceptions?](#Re-no-throw-codes)
* `for`:
  * [range-for and for](#Res-for-range)
  * [for and while](#Res-for-while)
  * [for-initializer](#Res-for-init)
  * [empty body](#Res-empty)
  * [loop variable](#Res-loop-counter)
  * [loop variable type ???](#Res-???)
* function:
  * [naming](#Rf-package)
  * [single operation](#Rf-logical)
  * [no throw](#Rf-noexcept)
  * [arguments](#Rf-smart)
  * [argument passing](#Rf-conventional)
  * [multiple return values](#Rf-out-multi)
  * [pointers](#Rf-return-ptr)
  * [lambdas](#Rf-capture-vs-overload)
* `inline`:
  * [small functions](#Rf-inline)
  * [in headers](#Rs-inline)
* initialization:
  * [always](#Res-always)
  * [prefer `{}`](#Res-list)
  * [lambdas](#Res-lambda-init)
  * [in-class initializers](#Rc-in-class-initializer)
  * [class members](#Rc-initialize)
  * [factory functions](#Rc-factory)
* lambda expression:
  * [when to use](#SS-lambdas)
* operator:
  * [conventional](#Ro-conventional)
  * [avoid conversion operators](#Ro-conversion)
  * [lambdas](#Ro-lambda)
* `public`, `private`, and `protected`:
  * [information hiding](#Rc-private)
  * [consistency](#Rh-public)
  * [`protected`](#Rh-protected)
* `static_assert`:
  * [compile-time checking](#Rp-compile-time)
  * [concepts](#Rt-check-class)
* `struct`:
  * [for organizing data](#Rc-org)
  * [use if no invariant](#Rc-struct)
  * [no private members](#Rc-class)
* `template`:
  * [abstraction](#Rt-raise)
  * [containers](#Rt-cont)
  * [concepts](#Rt-concepts)
* `unsigned`:
  * [signed](#Res-mix)
  * [bit manipulation](#Res-unsigned)
* `virtual`:
  * [interfaces](#Ri-abstract)
  * [not `virtual`](#Rc-concrete)
  * [destructor](#Rc-dtor-virtual)
  * [never fail](#Rc-dtor-fail)

규칙들을 설명하는데 사용된 설계 개념들:

* Assertion: ???
* 오류(Error): ???
* 예외(Exception): 예외 보증(Exception Guarantee) (???)
* 실패(Failure): ???
* 불변 조건(invariant): ???
* 누수(Leak): ???
* 라이브러리(Library): ???
* 사전 조건(Precondition): ???
* 사후 조건(Postcondition): ???
* 리소스(Resource): ???

## <a name="S-abstract"></a>요약

이 문서는 C++를 올바르게 사용하기 위한 가이드라인의 모음이며, 개발자들이 모던 C++를 효율적으로 사용할 수 있도록 돕기 위해 작성되었다.
여기서 "모던 C++"는 실제로 사용되고 있는 ISO C++ 표준을 의미한다 (현재는 C++17을 의미하지만, 거의 모든 권장사항들은 C++14와 C++11에도 적용된다). 지금 개발중인 코드가 5년, 또는 10년 후에 어떤 모습이었으면 좋겠는가?

C++ 핵심 가이드라인은 인터페이스, 리소스 관리, 메모리 관리, 동시성와 같이 상대적으로 고수준의 내용에 대한 것이며, 대체로 이런 규칙들은 어플리케이션의 구조나 라이브러리 설계에 영향을 미친다. 
가이드라인에 나오는 규칙들을 준수하면 코드가 정적 타입 안정성을 가지도록 할 수 있으며, 리소스의 누수가 없고, 흔히 저지르는 수많은 프로그래밍 로직 오류를 잡아낼 수 있으며, 더욱 빠르게 수행되는 코드를 작성할 수 있을 것이다. 즉, 올바른 코드를 만들 수 있다.

이름 지정 규칙이나 들여쓰기 스타일과 같이 세부적인 내용에 대해서는 크게 언급하지 않을 것이다. 하지만, 프로그래머에게 도움을 줄 수 있는 주제가 있다면 다루도록 하겠다.

초기의 규칙들은 (다양한 형태의) 안정성과 간결함을 과도하게 강조한 측면이 있다.
실용성을 위해 여러 예외 사항들이 도입되기를 바란다. 또한 더 많은 규칙들이 필요하다.

규칙들 중에는 독자의 기대와 다르거나, 경험과는 정반대되는 것들도 있을 것이다.
어떤 형태로든 여러분의 코딩 스타일에 변화를 주지 못한다면, 우리는 실패한 것이다! 올바른 규칙인지 확인해 보고, 문제가 있다면 반박해주길 바란다!
특히, 측정 결과나 더 나은 예제들로 규칙들을 뒷받침해주길 바란다.

명백하거나 당연해 보이는 규칙도 일부 있을 것이다.
하지만 이 가이드라인은 숙련되지 않은 개발자들 또는 다른 배경 지식을 갖거나 다른 언어를 알고 있는 개발자들이 C++ 언어를 빨리 익힐 수 있도록 돕기 위한 문서임을 기억해 주었으면 한다.

대부분의 규칙들을 분석 도구들을 통해 지원할 수 있도록 설계되었다. 참조(또는 링크)를 통해 어떤 규칙을 위반했는지 파악할 수 있다. 모든 규칙을 알고 코드를 작성할 수는 없는 노릇이다. 
이러한 가이드라인은 무슨 문제가 발생했는지 파악할 수 있게 해주는 툴에 대한 명세라고 볼 수 있다.

규칙들이 코드에 점진적으로 도입되길 바라며, 이를 위한 툴을 만들 계획을 갖고 있다.
C++ 언어에 새로운 기능들이 추가되고 사용 가능한 라이브러리들이 많아지므로 이 문서를 지속적으로 수정하고 확장해 규칙들에 대한 이해를 돕고자 한다.

규칙에 대한 의견과 제안은 언제든 환영한다. 
C++ 언어와 사용 가능한 라이브러리들이 발전하고, 이에 대한 이해가 깊어질수록 이 문서를 지속적으로 수정하고 확장할 것이다.
