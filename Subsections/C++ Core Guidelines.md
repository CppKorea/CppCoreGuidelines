# C++ 핵심 안내서

2015-09-09

작성자:

* [Bjarne Stroustrup](http://www.stroustrup.com)
* [Herb Sutter](http://herbsutter.com/)

># C++ Core Guidelines
>
>September 9, 2015
>
>Editors:
>
>* [Bjarne Stroustrup](http://www.stroustrup.com)
* [Herb Sutter](http://herbsutter.com/)

이 문서는 아직 초안상태입니다. 완벽하지 않으며, 심지어 잘못됬을 수도 있습니다. 오픈소스 프로젝트로 0.6 버전까지 배포되었습니다. 복사, 사용, 수정, 파생 성과물은 MIT 라이선스를 지켜야 합니다. 이 프로젝트에 기여하려면 라이선스에 동의해야 합니다. 자세한 내용은 첨부된 라이선스 파일을 참조하십시오. 오픈 소스 프로젝트에 익숙한 이들이 쉽게 사용, 복사, 수정, 기여 할수 있도록 하겠습니다.  

> This document is a very early draft. It is inkorrekt, incompleat, and pÂµÃ¸oorly formatted.
Had it been an open source (code) project, this would have been release 0.6.
Copying, use, modification, and creation of derivative works from this project is licensed under an MIT-style license.
Contributing to this project requires agreeing to a Contributor License. See the accompanying LICENSE file for details.
We make this project available to "friendly users" to use, copy, modify, and derive from, hoping for constructive input.

개선사항에 대한 의견과 제안은 언제나 환영합니다. 계속해서 수정하고 확장하여 C++ 언어와 그 라이브러리들 및 개념적 이해를 돕고자 합니다. 의견을 주실 때에는 [소개](#S-introduction)에 있는 방법과 목적에 유의해 주시기 바랍니다. 기여한 이들의 목록은 [여기](#SS-ack)에 있습니다.

> Comments and suggestions for improvements are most welcome.
We plan to modify and extend this document as our understanding improves and the language and the set of available libraries improve.
When commenting, please note [the introduction](#S-introduction) that outlines our aims and general approach.
The list of contributors is [here](#SS-ack).

과제:

* 규칙들은 아직 완전성, 일관성, 가능성 등에 대해서 철저하게 검증된 것이 아닙니다.
* (???) 표시는 아직 관련정보를 기입하지 않았다는 것을 의미합니다.
* 참조 예제를 업데이트해야 합니다. ; C++11 이전의 오래된 소스코드들이 아직 많습니다.
* 앞으로 남아있는 해야할 일에 대한 리스트는 여기서 확인이 가능합니다. [To-do: 미분류 규칙초안](#S-unclassified)

>Problems:
>
>* The sets of rules have not been thoroughly checked for completeness, consistency, or enforceability.
* Triple question marks (???) mark known missing information
* Update reference sections; many pre-C++11 sources are too old.
* For a more-or-less up-to-date to-do list see: [To-do: Unclassified proto-rules](#S-unclassified)

[이 안내서의 범위와 전체적인 구조](#S-abstract)를 확인하시거나, 아니면 바로 아래 내용을 확인 할 수 있습니다.

> You can [Read an explanation of the scope and structure of this Guide](#S-abstract) or just jump straight in:

* [P: 철학](#S-philosophy)
* [I: 인터페이스](#S-interfaces)
* [F: 함수](#S-functions)
* [C: 클래스와 클레스 상관관계](#S-class)
* [Enum: 열거형](#S-enum)
* [ES: 표현식과 문장](#S-expr)
* [E: 오류 처리](#S-errors)
* [R: 자원 관리](#S-resource)
* [T: template와 제네릭 프로그래밍](#S-templates)
* [CP: 동시성](#S-concurrency)
* [STL: 표준 라이브러리](#S-stdlib)
* [SF: 소스파일](#S-source)
* [CPL: C스타일 프로그래밍](#S-cpl)
* [PRO: 프로파일](#S-profile)
* [GSL: 지원되는 라이브러리에 대한 안내서](#S-support)

>* [P: Philosophy](#S-philosophy)
* [I: Interfaces](#S-interfaces)
* [F: Functions](#S-functions)
* [C: Classes and class hierarchies](#S-class)
* [Enum: Enumerations](#S-enum)
* [ES: Expressions and statements](#S-expr)
* [E: Error handling](#S-errors)
* [R: Resource management](#S-resource)
* [T: Templates and generic programming](#S-templates)
* [CP: Concurrency](#S-concurrency)
* [STL: The Standard library](#S-stdlib)
* [SF: Source files](#S-source)
* [CPL: C-style programming](#S-cpl)
* [PRO: Profiles](#S-profile)
* [GSL: Guideline support library](#S-support)

참고할 만한 내용들:

>Supporting sections:

* [NL: 명명규칙 및 배치](#S-naming)
* [PER: 성능](#S-performance)
* [N: 규칙이 아닌 미신들](#S-not)
* [RF: 참조](#S-references)
* [Appendix A: 라이브러리](#S-libraries)
* [Appendix B: 모던C++ 스타일로 코딩하기](#S-modernizing)
* [Appendix C: 토론](#S-discussion)
* [To-do: 미분류 규칙초안](#S-unclassified)

>* [NL: Naming and layout](#S-naming)
* [PER: Performance](#S-performance)
* [N: Non-Rules and myths](#S-not)
* [RF: References](#S-references)
* [Appendix A: Libraries](#S-libraries)
* [Appendix B: Modernizing code](#S-modernizing)
* [Appendix C: Discussion](#S-discussion)
* [To-do: Unclassified proto-rules](#S-unclassified)

구체적인 C++ 특성들은 다음을 참조하세요.

>or look at a specific language feature

* [대입](#S-???)
* [`class`](#S-class)
* [생성자](#SS-ctor)
* [상속된 `class`](#SS-hier)
* [소멸자](#SS-ctor)
* [예외](#S-errors)
* [`for`](#S-???)
* [`inline`](#S-class)
* [초기화](#S-???)
* [람다식](#SS-lambda)
* [연산자](#S-???)
* [`public`, `private`, `protected`](#S-???)
* [`static_assert`](#S-???)
* [`struct`](#S-class)
* [`template`](#S-???)
* [`unsigned`](#S-???)
* [`virtual`](#S-hier)

>* [assignment](#S-???)
* [`class`](#S-class)
* [constructor](#SS-ctor)
* [derived `class`](#SS-hier)
* [destructor](#SS-ctor)
* [exception](#S-errors)
* [`for`](#S-???)
* [`inline`](#S-class)
* [initialization](#S-???)
* [lambda expression](#SS-lambda)
* [operator](#S-???)
* [`public`, `private`, and `protected`](#S-???)
* [`static_assert`](#S-???)
* [`struct`](#S-class)
* [`template`](#S-???)
* [`unsigned`](#S-???)
* [`virtual`](#S-hier)

용어의 정의는 규칙이 대한 설명을 위한 것이지, 프로그래밍 테크닉을 위한것은 아닙니다. 설계, 프로그래밍 테크닉에 대해서는 다음 사항을 참고하세요.

> Definitions of terms used to express and discuss the rules, that are not language-technical, but refer to design and programming techniques

* 오류(error)
* 예외(exception)
* 실패(failure)
* 불변(invariant) : 넣어야 할 값이 누락되는 경우
* 누수(leak)
* 선행조건(precondition)
* 결과조건(postcondition)
* 자원(resource)
* 예외 보증(exception guarantee)

>* error
* exception
* failure
* invariant
* leak
* precondition
* postcondition
* resource
* exception guarantee
