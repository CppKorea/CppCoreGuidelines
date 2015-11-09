# <a name="main"></a> C++ Core Guidelines

October 10, 2015

작성자:
>Editors:

* [Bjarne Stroustrup](http://www.stroustrup.com)
* [Herb Sutter](http://herbsutter.com/)

이 문서는 초안이다. 오픈소스 프로젝트였으면 0.6 버전이었을 것이다.
복사, 사용, 수정, 이 프로젝트에서 파생한 작업 생성은 MIT 라이센스를 지원한다.
이 프로젝트에 참여하는 것은 Contributor License에 동의함을 요구한다.
자세한 사항은 다음의 [LICENSE](LICENSE)를 보면 된다.
건설적인 작업을 기대하며 사용, 복사, 수정, 파생할 수 있도록 호의적인 사용자들에게 이 프로젝트를 사용할 수 있도록 허락한다.
>This document is a very early draft. It is inkorrekt, incompleat, and pÂµÃ¸oorly formatted.
Had it been an open source (code) project, this would have been release 0.6.
Copying, use, modification, and creation of derivative works from this project is licensed under an MIT-style license.
Contributing to this project requires agreeing to a Contributor License. See the accompanying [LICENSE](LICENSE) file for details.
We make this project available to "friendly users" to use, copy, modify, and derive from, hoping for constructive input.

개선을 위한 주석이나 제안은 언제나 환영이다.
이해를 증가시킬 목적, 라이브러리 집합과 언어를 개선할 목적으로 이 문서를 수정하고 확장할 계획이다.
의견을 주고 싶다면 우리의 목적과 접근법을 개략적으로 설명한 [the introduction](#S-introduction)를 참조하라.
공헌자 목록은 [here](#SS-ack) 여기서.
>Comments and suggestions for improvements are most welcome.
We plan to modify and extend this document as our understanding improves and the language and the set of available libraries improve.
When commenting, please note [the introduction](#S-introduction) that outlines our aims and general approach.
The list of contributors is [here](#SS-ack).

문제점:
>Problems:

* 규칙들은 완성도, 일관성, 시행가능성 등을 철저히 체크하지 않았다.
* (???)마크는 정보가 부족함을 나타낸다.
* 참조 단원을 업데이트해라; 많은 C++11 이전 소스는 너무 오래된 편이다.
* 거진 최신 해야할 일 목록은 [To-do: Unclassified proto-rules](#S-unclassified)를 보라.

>* The sets of rules have not been thoroughly checked for completeness, consistency, or enforceability.
>* Triple question marks (???) mark known missing information
>* Update reference sections; many pre-C++11 sources are too old.
>* For a more-or-less up-to-date to-do list see: [To-do: Unclassified proto-rules](#S-unclassified)

[Read an explanation of the scope and structure of this Guide](#S-abstract)를 읽거나 다음 단원으로 넘어가라:
>You can [Read an explanation of the scope and structure of this Guide](#S-abstract) or just jump straight in:

* [P: Philosophy](#S-philosophy)
* [I: Interfaces](#S-interfaces)
* [F: Functions](#S-functions)
* [C: Classes and class hierarchies](#S-class)
* [Enum: Enumerations](#S-enum)
* [ES: Expressions and statements](#S-expr)
* [E: Error handling](#S-errors)
* [R: Resource management](#S-resource)
* [T: Templates and generic programming](#S-templates)
* [CP: Concurrency](#S-concurrency)
* [SL: The Standard library](#S-stdlib)
* [SF: Source files](#S-source)
* [CPL: C-style programming](#S-cpl)
* [PRO: Profiles](#S-profile)
* [GSL: Guideline support library](#S-gsl)
* [FAQ: Answers to frequently asked questions](#S-faq)

부가적인 단원:
>Supporting sections:

* [NL: Naming and layout](#S-naming)
* [PER: Performance](#S-performance)
* [N: Non-Rules and myths](#S-not)
* [RF: References](#S-references)
* [Appendix A: Libraries](#S-libraries)
* [Appendix B: Modernizing code](#S-modernizing)
* [Appendix C: Discussion](#S-discussion)
* [Glossary](#S-glossary)
* [To-do: Unclassified proto-rules](#S-unclassified)

또는 구체적인 언어 특징을 살펴봐라.
>or look at a specific language feature

* [대입](#S-???)
* [클래스](#S-class)
* [생성자](#SS-ctor)
* [파생 클래스](#SS-hier)
* [소멸자](#SS-dtor)
* [예외](#S-errors)
* [`for`문](#S-???)
* [`inline`문](#S-class)
* [초기화](#S-???)
* [람다 표현식](#SS-lambdas)
* [연산자](#S-???)
* [`public`, `private`, and `protected`](#S-???)
* [`static_assert`](#S-???)
* [`struct`](#S-class)
* [`template`](#S-???)
* [`unsigned`](#S-???)
* [`virtual`](#SS-hier)

디자인, 프로그래밍 기술을 나타내고, 언어에 종속적이지 않은 규칙을 검토하고 기술하는데 사용하는 용어 정의.
>Definitions of terms used to express and discuss the rules, that are not language-technical, but refer to design and programming techniques

* error
* exception
* failure
* invariant
* leak
* precondition
* postcondition
* resource
* exception guarantee
