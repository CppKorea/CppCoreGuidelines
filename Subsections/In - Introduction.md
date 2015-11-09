# <a name="S-introduction"></a> In: Introduction

이 문서는 추후 강화될 내용과 ISO 기술명세까지 고려한 현대 C++, C++14를 위한 핵심 가이드라인 집합이다.
목표는 C++ 프로그래머가 더 쉽고 효과적으로 더 유지보수가 좋은 코드를 작성하도록 하는 것이다.
>This is a set of core guidelines for modern C++, C++14, and taking likely future enhancements and taking ISO Technical Specifications (TSs) into account.
The aim is to help C++ programmers to write simpler, more efficient, more maintainable code.

도입 요약:
>Introduction summary:

* [In.target: Target readership](#SS-readers)
* [In.aims: Aims](#SS-aims)
* [In.not: Non-aims](#SS-non)
* [In.force: Enforcement](#SS-force)
* [In.struct: The structure of this document](#SS-struct)
* [In.sec: Major sections](#SS-sec)

## <a name="SS-readers"></a> In.target: 타겟이 되는 독자층
>## <a name="SS-readers"></a> In.target: Target readership

모든 C++ 프로그래머. 여기에 [programmers who might consider C](#S-cpl)도 포함된다.
>All C++ programmers. This includes [programmers who might consider C](#S-cpl).

## <a name="SS-aims"></a> In.aims: 목표
>## <a name="SS-aims"></a> In.aims: Aims

이 문서의 목적은 개발자들이 현대 C++(C++11, C++14, 이후 C++17까지)에 적응하는 것을 돕고 코드에 일정한 스타일을 적용하도록 하는 것이다.
>The purpose of this document is to help developers to adopt modern C++ (C++11, C++14, and soon C++17) and to achieve a more uniform style across code bases.

우리는 여기 규칙들이 모든 코드에 효과적으로 적용될 수 있다는 환상을 갖고 있지는 않다.
오래된 시스템을 업그레이드하는 것은 어렵지만, 이 규칙을 사용하는 프로그램이 에러를 덜 만들고 더 유지보수가 쉬울 거라고 믿는다.
아마도 이런 규칙들이 초기 개발을 더 쉽고 빠르게 만들지도 모른다.
우리가 말할 수 있는 한, 이 규칙들이 이전보다 더 좋은 성능, 더 관례적인 기술을 쓰는 코드를 만든다는 점이고 오버헤드가 없는 규칙을 따른다는 것이다.
("사용하지 않으면 지불하지 않아도 된다.", "추상화 방법을 적절히 사용하면 하위레벨 언어구조를 사용해서 직접 코딩한 정도의 성능을 얻을 것이다.")
새 코드에 대한 이상으로, 오래된 코드를 동작시킬 때 활용할 기회로 이 규칙을 고려해보라. 그리고 실행 가능할 정도로 가깝게 이 아이디어를 가지고 추정해 보라.
기억해라:
>We do not suffer the delusion that every one of these rules can be effectively applied to every code base. Upgrading old systems is hard. However, we do believe that a program that uses a rule is less error-prone and more maintainable than one that does not. Often, rules also lead to faster/easier initial development.
As far as we can tell, these rules lead to code that performs as well or better than older, more conventional techniques; they are meant to follow the zero-overhead principle ("what you don't use, you don't pay for" or "when you use an abstraction mechanism appropriately, you get at least as good performance as if you had handcoded using lower-level language constructs").
Consider these rules ideals for new code, opportunities to exploit when working on older code, and try to approximate these ideas as closely as feasible.
Remember:

### <a name="R0"></a> In.0: 당황하지 마라.
>### <a name="R0"></a> In.0: Don't panic!

당신의 프로그램에 가이드라인 규칙을 적용했을때 어떤 영향을 미치는지 이해할려면 시간이 필요하다.
>Take the time to understand the implications of a guideline rule on your program.

가이드라인은 상위집합의 부분집합 원칙([Stroustrup05](#Stroustrup05))에 따라 디자인되었다.
단순히 C++의 부분집합으로 정의하지는 않았다.(신뢰성, 안정성, 성능 등을 위해)
대신, 몇몇 작은 "확장"([library components](#S-gsl))을 사용하도록 강하게 추천한다.
C++의 가장 에러를 일으키기 쉬운 특징을 사용하는 것을 중복하게 만드는 그런 규칙들은 금지할 수 있다. (? - 어렵다)
>These guidelines are designed according to the "subset of a superset" principle ([Stroustrup05](#Stroustrup05)).
They do not simply define a subset of C++ to be used (for reliability, safety, performance, or whatever).
Instead, they strongly recommend the use of a few simple "extensions" ([library components](#S-gsl))
that make the use of the most error-prone features of C++ redundant, so that they can be banned (in our set of rules).

정적 타입 안전성, 리소스 안전성을 강조한다.
이런 이유로 범위 체크 가능성, `nullptr` 역참조(?) 피하기, 없는 메모리 참조 포인터 피하기, 예외에 대한 기계적인 사용을 강조한다(RAII).
부분적으로 안전성을 이루기 위해, 부분적으로 에러원인인 애매한 코드를 최소화하기 위해, 단순성, 잘 정의된 인터페이스 뒤에 복잡성 숨기기를 강조한다.
>The rules emphasize static type safety and resource safety.
For that reason, they emphasize possibilities for range checking, for avoiding dereferencing `nullptr`, for avoiding dangling pointers, and the systematic use of exceptions (via RAII).
Partly to achieve that and partly to minimize obscure code as a source of errors, the rules also emphasize simplicity and the hiding of necessary complexity behind well-specified interfaces.

많은 규칙들은 권위적이다.
대안도 없이 "그걸 하지마라"라고 하는 규칙들을 불편해한다.
그런 결과로, 몇몇 규칙들은 정확하고 기계적인 검증보다는 경험적으로만 지원될 수 있다는 것이다.
다른 규칙들은 일반적인 원리를 설명한다.
이런 일반적인 규칙들에 대해서, 좀더 자세하고 구체적인 규칙이 부분적으로 검증한다.
>Many of the rules are prescriptive.
We are uncomfortable with rules that simply state "don't do that!" without offering an alternative.
One consequence of that is that some rules can be supported only by heuristics, rather than precise and mechanically verifiable checks.
Other rules articulate general principles. For these more general rules, more detailed and specific rules provide partial checking.

가이드라인은 C++의 핵심과 사용법을 언급한다.
대규모 기구, 특수 응용 영역, 대규모 프로젝트는 더 많은 규칙, 더 많은 제약, 더 많은 라이브러리 지원을 필요로 한다고 생각한다.
예를 들어, 실시간 프로그래머는 전형적으로 공짜 스토어(동적 메모리)를 사용하지 않는다. 그리고 라이브러리의 선택도 제한될 것이다.
가이드라인에 부록으로 더 구체적인 규칙을 추가하기를 기대한다.
미화된 어셈블리 코드로 당신의 프로그래밍을 하위레벨을 낮추지 말고, 당신의 작은 기본 라이브러리를 구성하고 사용해라.
>These guidelines address a core of C++ and its use.
We expect that most large organizations, specific application areas, and even large projects will need further rules, possibly further restrictions, and further library support.
For example, hard real-time programmers typically can't use free store (dynamic memory) freely and will be restricted in their choice of libraries.
We encourage the development of such more specific rules as addenda to these core guidelines.
Build your ideal small foundation library and use that, rather than lowering your level of programming to glorified assembly code.

규칙들은 [gradual adoption](#S-modernizing)를 허용하도록 디자인했다.
>The rules are designed to allow [gradual adoption](#S-modernizing).

몇몇 규칙은 다양한 형태로 안전성을 증가시킬 목표를 가진다. 반면에 다른 규칙은 사고 가능성을 줄일 목표를 가진다. 둘다 가지는 규칙도 많다.
사고를 줄일 목적인 가이드라인은 C++에 합법적인 것을 금지시키곤 한다.
그러나 아이디어를 표현할 두가지 방법이 있고 하나는 에러가 발생할 것처럼 보이고 하나는 안 보인다면, 후자를 프로그래머에게 가이드할 것이다.
>Some rules aim to increase various forms of safety while others aim to reduce the likelihood of accidents, many do both.
The guidelines aimed at preventing accidents often ban perfectly legal C++.
However, when there are two ways of expressing an idea and one has shown itself a common source of errors and the other has not, we try to guide programmers towards the latter.

## <a name="SS-non"></a> In.not: Non-aims

The rules are not intended to be minimal or orthogonal.
In particular, general rules can be simple, but unenforceable.
Also, it is often hard to understand the implications of a general rule.
More specialized rules are often easier to understand and to enforce, but without general rules, they would just be a long list of special cases.
We provide rules aimed at helping novices as well as rules supporting expert use.
Some rules can be completely enforced, but others are based on heuristics.

These rules are not meant to be read serially, like a book.
You can browse through them using the links.
However, their main intended use is to be targets for tools.
That is, a tool looks for violations and the tool returns links to violated rules.
The rules then provide reasons, examples of potential consequences of the violation, and suggested remedies.

These guidelines are not intended to be a substitute for a tutorial treatment of C++.
If you need a tutorial for some given level of experience, see [the references](#S-references).

This is not a guide on how to convert old C++ code to more modern code.
It is meant to articulate ideas for new code in a concrete fashion.
However, see [the modernization section](#S-modernizing) for some possible approaches to modernizing/rejuvenating/upgrading.
Importantly, the rules support gradual adoption: It is typically infeasible to convert all of a large code base at once.

These guidelines are not meant to be complete or exact in every language-technical detail.
For the final word on language definition issues, including every exception to general rules and every feature, see the ISO C++ standard.

The rules are not intended to force you to write in an impoverished subset of C++.
They are *emphatically* not meant to define a, say, Java-like subset of C++.
They are not meant to define a single "one true C++" language.
We value expressiveness and uncompromised performance.

The rules are not value-neutral.
They are meant to make code simpler and more correct/safer than most existing C++ code, without loss of performance.
They are meant to inhibit perfectly valid C++ code that correlates with errors, spurious complexity, and poor performance.

## <a name="SS-force"></a> In.force: Enforcement

Rules with no enforcement are unmanageable for large code bases.
Enforcement of all rules is possible only for a small weak set of rules or for a specific user community.
But we want lots of rules, and we want rules that everybody can use.
But different people have different needs.
But people don't like to read lots of rules.
But people can't remember many rules.
So, we need subsetting to meet a variety of needs.
But arbitrary subsetting leads to chaos: We want guidelines that help a lot of people, make code more uniform, and strongly encourage people to modernize their code.
We want to encourage best practices, rather than leave all to individual choices and management pressures.
The ideal is to use all rules; that gives the greatest benefits.

This adds up to quite a few dilemmas.
We try to resolve those using tools.
Each rule has an **Enforcement** section listing ideas for enforcement.
Enforcement might be by code review, by static analysis, by compiler, or by run-time checks.
Wherever possible, we prefer "mechanical" checking (humans are slow and bore easily) and static checking.
Run-time checks are suggested only rarely where no alternative exists; we do not want to introduce "distributed fat" - if that's what you want, you know where to find it.
Where appropriate, we label a rule (in the **Enforcement** sections) with the name of groups of related rules (called "profiles").
A rule can be part of several profiles, or none.
For a start, we have a few profiles corresponding to common needs (desires, ideals):

* **types**: No type violations (reinterpreting a `T` as a `U` through casts/unions/varargs)
* **bounds**: No bounds violations (accessing beyond the range of an array)
* **lifetime**: No leaks (failing to `delete` or multiple `delete`) and no access to invalid objects (dereferencing `nullptr`, using a dangling reference).

The profiles are intended to be used by tools, but also serve as an aid to the human reader.
We do not limit our comment in the **Enforcement** sections to things we know how to enforce; some comments are mere wishes that might inspire some tool builder.

## <a name="SS-struct"></a> In.struct: The structure of this document

Each rule (guideline, suggestion) can have several parts:

* The rule itself - e.g., **no naked `new`**
* A rule reference number - e.g., **C.7** (the 7th rule related to classes).
  Since the major sections are not inherently ordered, we use a letter as the first part of a rule reference "number".
  We leave gaps in the numbering to minimize "disruption" when we add or remove rules.
* **Reason**s (rationales) - because programmers find it hard to follow rules they don't understand
* **Example**s - because rules are hard to understand in the abstract; can be positive or negative
* **Alternative**s - for "don't do this" rules
* **Exception**s - we prefer simple general rules. However, many rules apply widely, but not universally, so exceptions must be listed
* **Enforcement** - ideas about how the rule might be checked "mechanically"
* **See also**s - references to related rules and/or further discussion (in this document or elsewhere)
* **Note**s (comments) - something that needs saying that doesn't fit the other classifications
* **Discussion** - references to more extensive rationale and/or examples placed outside the main lists of rules

Some rules are hard to check mechanically, but they all meet the minimal criteria that an expert programmer can spot many violations without too much trouble.
We hope that "mechanical" tools will improve with time to approximate what such an expert programmer notices.
Also, we assume that the rules will be refined over time to make them more precise and checkable.

A rule is aimed at being simple, rather than carefully phrased to mention every alternative and special case.
Such information is found in the **Alternative** paragraphs and the [Discussion](#S-discussion) sections.
If you don't understand a rule or disagree with it, please visit its **Discussion**.
If you feel that a discussion is missing or incomplete, send us an email.

This is not a language manual.
It is meant to be helpful, rather than complete, fully accurate on technical details, or a guide to existing code.
Recommended information sources can be found in [the references](#S-references).

## <a name="SS-sec"></a> In.sec: Major sections

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

Supporting sections:

* [NL: Naming and layout](#S-naming)
* [PER: Performance](#S-performance)
* [N: Non-Rules and myths](#S-not)
* [RF: References](#S-references)
* [Appendix A: Libraries](#S-libraries)
* [Appendix B: Modernizing code](#S-modernizing)
* [Appendix C: Discussion](#S-discussion)
* [Glossary](#S-glossary)
* [To-do: Unclassified proto-rules](#S-unclassified)

These sections are not orthogonal.

Each section (e.g., "P" for "Philosophy") and each subsection (e.g., "C.hier" for "Class Hierarchies (OOP)") have an abbreviation for ease of searching and reference.
The main section abbreviations are also used in rule numbers (e.g., "C.11" for "Make concrete types regular").
