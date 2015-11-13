# <a name="S-introduction"></a> In: Introduction

이 문서는 추후 강화될 내용과 ISO 기술명세까지 고려한 모던 C++, C++14를 위한 핵심 가이드라인 집합이며 C++ 프로그래머가 더 쉽고 효과적으로 유지보수가 좋은 코드를 작성하도록 하는 것을 목표로 두고 있습니다.
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

모든 C++ 프로그래머가 타겟이며 여기에 [programmers who might consider C](#S-cpl)도 포함됩니다.
>All C++ programmers. This includes [programmers who might consider C](#S-cpl).

## <a name="SS-aims"></a> In.aims: 목표
>## <a name="SS-aims"></a> In.aims: Aims

이 문서의 목적은 개발자들이 모던 C++(C++11, C++14, 이후 C++17까지)에 적응하는 것을 돕고 코드에 일정한 스타일을 적용하도록 하는 것입니다.
>The purpose of this document is to help developers to adopt modern C++ (C++11, C++14, and soon C++17) and to achieve a more uniform style across code bases.

우리는 이 규칙들이 모든 코드에 효과적으로 적용될 수 있다는 환상을 갖고 있진 않습니다.
오래된 시스템을 업그레이드하는 것은 어렵지만, 이 규칙을 사용하는 프로그램이 에러를 덜 만들고 더 유지보수가 쉬울 거라고 믿습니다. 아마도 이 규칙들이 초기 개발을 더 쉽고 빠르게 만들지도 모릅니다.
우리가 말할 수 있는 것은, 이 규칙들이 이전보다 더 좋은 성능, 더 관례적인 기술을 쓰는 코드를 만든다는 점이고 비용을 지불할 필요가 없는 규칙을 따른다는 것입니다.
("사용하지 않으면 지불하지 않아도 된다.", "추상화 방법을 적절히 사용하면 하위레벨 언어구조를 사용해서 직접 코딩한 정도의 성능을 얻을 것이다.")
오래된 코드를 동작시킬 때 이상이 생겨 새코드를 작성해야 할 때 이 규칙을 활용하는 것을 고려해보고 실현 가능할 정도로 이 규칙에 대해 알고 있어 보세요.
기억하세요:
>We do not suffer the delusion that every one of these rules can be effectively applied to every code base. Upgrading old systems is hard. However, we do believe that a program that uses a rule is less error-prone and more maintainable than one that does not. Often, rules also lead to faster/easier initial development.
As far as we can tell, these rules lead to code that performs as well or better than older, more conventional techniques; they are meant to follow the zero-overhead principle ("what you don't use, you don't pay for" or "when you use an abstraction mechanism appropriately, you get at least as good performance as if you had handcoded using lower-level language constructs").
Consider these rules ideals for new code, opportunities to exploit when working on older code, and try to approximate these ideas as closely as feasible.
Remember:

### <a name="R0"></a> In.0: 당황하지 마라!
>### <a name="R0"></a> In.0: Don't panic!

당신의 프로그램에 가이드라인 규칙을 적용했을때 어떤 영향을 미치는지 이해하기 위해선 시간이 필요할 것 입니다.
>Take the time to understand the implications of a guideline rule on your program.

가이드라인은 상위집합의 부분집합 원칙([Stroustrup05](#Stroustrup05))에 따라 디자인되었습니다.
신뢰성, 안정성, 성능 등을 위해 단순히 C++의 부분집합으로 정의하지는 않았습니다.
대신, 몇몇의 간단한 "확장"([library components](#S-gsl))을 사용하도록 강력히 추천합니다.
C++의 에러를 일으키기 쉬운 불필요한 특징을 사용하는 것 같은 규칙들은 제외하고 사용하실 수 있습니다. (의역)
>These guidelines are designed according to the "subset of a superset" principle ([Stroustrup05](#Stroustrup05)).
They do not simply define a subset of C++ to be used (for reliability, safety, performance, or whatever).
Instead, they strongly recommend the use of a few simple "extensions" ([library components](#S-gsl))
that make the use of the most error-prone features of C++ redundant, so that they can be banned (in our set of rules).

정적 타입 안전성, 리소스 안전성을 위해 RAII(Resource acquisition is initialization)를 통한 범위 체크 가능성, `nullptr` 역참조 피하기, 없는 메모리 참조 포인터 피하기, 예외에 대한 기계적인 사용과 부분적으로 안전성을 이루고 에러 원인인 애매한 코드를 최소화하기 위해 잘 정의된 인터페이스 뒤에 복잡성 숨기기를 이용하여 단순하게 하는것을 강조합니다.
>The rules emphasize static type safety and resource safety.
For that reason, they emphasize possibilities for range checking, for avoiding dereferencing `nullptr`, for avoiding dangling pointers, and the systematic use of exceptions (via RAII).
Partly to achieve that and partly to minimize obscure code as a source of errors, the rules also emphasize simplicity and the hiding of necessary complexity behind well-specified interfaces.

많은 규칙들은 권위적입니다.
우리는 대안도 없이 그저 "그렇게 하지 마라!"라고만 하는 규칙들을 싫어합니다.
몇몇 규칙들은 정확하고 기계적인 검증보다는 경험적으로만 지원될 수 있다는 것입니다. (?)
다른 규칙들은 일반적인 원리를 설명하지만 이런 일반적인 규칙들에 대해서, 좀더 자세하고 구체적인 규칙을 부분적으로 검증합니다.
>Many of the rules are prescriptive.
We are uncomfortable with rules that simply state "don't do that!" without offering an alternative.
One consequence of that is that some rules can be supported only by heuristics, rather than precise and mechanically verifiable checks.
Other rules articulate general principles. For these more general rules, more detailed and specific rules provide partial checking.

가이드라인은 C++의 핵심과 사용법을 언급합니다. 우리는 대규모 기구, 특수 응용 영역, 대규모 프로젝트에선 더 많은 규칙, 더 많은 제약, 더 많은 라이브러리 지원을 필요로 한다고 생각합니다.
예를 들어, 실시간 프로그래머는 전형적으로 공짜 스토어(동적 메모리)를 사용하면 안되고 라이브러리의 선택도 제한되듯이 말입니다.
가이드라인에 부록으로 더 구체적인 규칙을 추가하기를 기대합니다.
미화된 어셈블리 코드로 당신의 프로그래밍을 하위 레벨로 낮추지 말고, 당신의 작은 기본 라이브러리를 구성하고 사용하세요.
>These guidelines address a core of C++ and its use.
We expect that most large organizations, specific application areas, and even large projects will need further rules, possibly further restrictions, and further library support.
For example, hard real-time programmers typically can't use free store (dynamic memory) freely and will be restricted in their choice of libraries.
We encourage the development of such more specific rules as addenda to these core guidelines.
Build your ideal small foundation library and use that, rather than lowering your level of programming to glorified assembly code.

규칙들은 [gradual adoption](#S-modernizing)를 허용하도록 디자인했습니다.
>The rules are designed to allow [gradual adoption](#S-modernizing).

몇몇 규칙은 다양한 형태로 안전성을 증가시킬 목표가지지만 다른 몇몇 규칙은 사고 가능성을 줄일 목표를 가지고 있습니다. 이 둘을 모두 가지고 있는 규칙도 많습니다.
사고를 줄일 목적인 가이드라인은 C++에서 사용될 수 있는 것을 제한하고 있습니다.
그러나 사고를 줄일 방법이 두가지이고 하나는 에러가 발생하고 나머지 하나는 에러가 발생하지 않는다면, 후자를 프로그래머에게 가이드해드릴 것입니다.
>Some rules aim to increase various forms of safety while others aim to reduce the likelihood of accidents, many do both.
The guidelines aimed at preventing accidents often ban perfectly legal C++.
However, when there are two ways of expressing an idea and one has shown itself a common source of errors and the other has not, we try to guide programmers towards the latter.

## <a name="SS-non"></a> In.not: 목표가 아닌 것
>## <a name="SS-non"></a> In.not: Non-aims

규칙들을 최소로 하거나, 서로 관계 없이 할 의도는 없습니다.
특히 일반화된 규칙은 단순하지만 시행할 수 없기도 하고, 규칙의 의미를 이해하기가 어렵기도 합니다.
구체적인 규칙들은 종종 이해하기가 더 쉽고 시행하기도 쉽습니다. 하지만 일반화된 규칙들이 없다면 특수한 경우의 나열일 뿐입니다.
우리는 초보자들을 돕기 위한 규칙들, 전문가를 잘 지원하는 규칙들을 제공하려고 합니다.
어떤 규칙들은 완전히 실행이 가능하기도 하고, 또 어떤 것들은 경험치에 의존하기도 합니다.
>The rules are not intended to be minimal or orthogonal.
In particular, general rules can be simple, but unenforceable.
Also, it is often hard to understand the implications of a general rule.
More specialized rules are often easier to understand and to enforce, but without general rules, they would just be a long list of special cases.
We provide rules aimed at helping novices as well as rules supporting expert use.
Some rules can be completely enforced, but others are based on heuristics.

이 규칙들을 책처럼 진지하게 읽을 필요는 없습니다.
링크를 사용해서 규칙들을 둘러볼 수 있습니다.
하지만 링크는 규칙 위반 사항을 고치는 것을 돕기 위한 것입니다.
즉, 툴은 위반 사항을 찾고 위반한 규칙에 대한 링크를 반환합니다.
링크를 통해 원인, 위반에 대한 결과 예제, 그리고 수정 방안을 볼 수 있습니다.
>These rules are not meant to be read serially, like a book.
You can browse through them using the links.
However, their main intended use is to be targets for tools.
That is, a tool looks for violations and the tool returns links to violated rules.
The rules then provide reasons, examples of potential consequences of the violation, and suggested remedies.

이 가이드라인은 C++의 튜토리얼을 대신하지 않습니다.
당신의 능력에 맞는 튜토리얼이 필요하다면, [the references](#S-references)를 참조하세요.
>These guidelines are not intended to be a substitute for a tutorial treatment of C++.
If you need a tutorial for some given level of experience, see [the references](#S-references).

이 문서는 오래전 C++ 코드를 모던 코드로 어떻게 변환할지에 대한 가이드는 아닙니다.
구체적인 방법으로 새로운 코드에 대한 방법을 말하고 있을 뿐입니다.
그러니 코드를 현대적이고, 젊게만들고, 업그레이드하려면 [the modernization section](#S-modernizing)를 참조하세요.
한가지 중요한건, 이 규칙들은 점진적인 적응을 지원합니다. 한방에 모든 모든 코드를 변환시키는 것은 불가능하기 때문입니다.
>This is not a guide on how to convert old C++ code to more modern code.
It is meant to articulate ideas for new code in a concrete fashion.
However, see [the modernization section](#S-modernizing) for some possible approaches to modernizing/rejuvenating/upgrading.
Importantly, the rules support gradual adoption: It is typically infeasible to convert all of a large code base at once.

이 가이드라인은 언어의 기술적인 상세한 부분에 대해선 정확하지도 않고 완벽하지도 않습니다.
언어 정의 문제에 대해 우리가 할 마지막 말은, 일반화된 규칙에 대한 모든 예외와 모든 특징을 포함해서 ISO C++ 표준을 참조하라는 것입니다.
>These guidelines are not meant to be complete or exact in every language-technical detail.
For the final word on language definition issues, including every exception to general rules and every feature, see the ISO C++ standard.

여기 규칙들은 C++의 협소한 부분집합으로 코드를 작성하도록 요구하지 않습니다.
자바같이 C++의 부분집합을 정의하는 것은 *절대로* 요구하지 않습니다.
단 하나의 진짜 C++ 언어를 정의하려고 하지도 않습니다.
우리는 원하는 성능과 표현력에 가치를 두고 있습니다.
>The rules are not intended to force you to write in an impoverished subset of C++.
They are *emphatically* not meant to define a, say, Java-like subset of C++.
They are not meant to define a single "one true C++" language.
We value expressiveness and uncompromised performance.

여기 규칙들은 가치중립적이지 않습니다.
성능 손실없이 기존의 대부분 소스보다 코드를 더 쉽게 만들기, 더 정확하고 안전하게 만들고 에러, 복잡한 코드, 성능 저하와 연관성이 있는 C++ 코드를 금하기 위한 것입니다.
>The rules are not value-neutral.
They are meant to make code simpler and more correct/safer than most existing C++ code, without loss of performance.
They are meant to inhibit perfectly valid C++ code that correlates with errors, spurious complexity, and poor performance.

## <a name="SS-force"></a> In.force: 실행
>## <a name="SS-force"></a> In.force: Enforcement

이 규칙은 거대한 코드베이스를 관리할 수 없습니다.
모든 규칙은 작은 집합일때만 가능하거나, 또는 특수한 사용자 집단에서만 가능합니다.
그러나 우리는 많은 규칙을 필요로 하고, 모든 사람들이 사용할 수 있는 규칙이 필요합니다.
다른 사람들은 많은 규칙을 읽고 싶어하지 않고 기억할 수도 없습니다.
그래서 우리는 다양한 필요를 만족시킬 부분집합이 필요합니다.
임의로 부분집합을 만들면 혼란만 초래합니다. 코드를 보기 좋게 만들고, 현대화할 수 있게 도와주는 가이드라인이 필요합니다.
우리는 개인적인 선택과 관리 압박을 모두에게 떠넘기기보다는 모범사례로 남고 싶습니다.
이 모든 규칙을 사용하는 것이 이상적이고, 당신에게 최고의 혜택을 줄 것입니다.
>Rules with no enforcement are unmanageable for large code bases.
Enforcement of all rules is possible only for a small weak set of rules or for a specific user community.
But we want lots of rules, and we want rules that everybody can use.
But different people have different needs.
But people don't like to read lots of rules.
But people can't remember many rules.
So, we need subsetting to meet a variety of needs.
But arbitrary subsetting leads to chaos: We want guidelines that help a lot of people, make code more uniform, and strongly encourage people to modernize their code.
We want to encourage best practices, rather than leave all to individual choices and management pressures.
The ideal is to use all rules; that gives the greatest benefits.

이것은 상당한 딜레마를 덧붙인 격입니다.
그래서 우리는 툴을 사용해서 딜레마를 해결하고 싶습니다.
각 규칙은 시행을 위한 아이디어를 나열하는 **Enforcement** 단락을 가지고 있습니다.
시행은 코드 리뷰, 정적 분석, 컴파일러, 런타임 체크 등이 될 수도 있습니다.
인간은 느리고 쉽게 지루해 하기 때문에 가능하다면 우리는 "기계적인" 체크와 정적 체크를 선호합니다.
런타임 체크는 대안이 없을 때만 아주 드물게 기계적인 것에서 벗어날 수 있습니다.
난장판을 소개하고 싶지는 않습니다. - 당신이 원하는 것이 그것이라면, 어디 있는지 알 수 있게 하고 싶었습니다.
그래서 적당한 곳에, 관련된 규칙을 그룹으로 묶어 "프로필"이라고 불리우는 이름을 달았습니다.
규칙은 여러 프로필의 한 부분일 수 있고, 아닐 수도 있습니다.
처음으로 공통된 필요성(요구사항, 아이디어)에 해당하는 몇몇 프로필을 봅시다.
>This adds up to quite a few dilemmas.
We try to resolve those using tools.
Each rule has an **Enforcement** section listing ideas for enforcement.
Enforcement might be by code review, by static analysis, by compiler, or by run-time checks.
Wherever possible, we prefer "mechanical" checking (humans are slow and bore easily) and static checking.
Run-time checks are suggested only rarely where no alternative exists; we do not want to introduce "distributed fat" - if that's what you want, you know where to find it.
Where appropriate, we label a rule (in the **Enforcement** sections) with the name of groups of related rules (called "profiles").
A rule can be part of several profiles, or none.
For a start, we have a few profiles corresponding to common needs (desires, ideals):

* **types**: 타입없음 위반 (형변환, 유니언, 가변매개변수로 `U`를 `T`를 재해석하기)
* **bounds**: 범위없음 위반 (배열의 범위를 넘어서 접근하기)
* **lifetime**: 누수없음(`delete`, `delete []`에 실패), 잘못된 객체에 접근하기(`nullptr` 비참조, 없는 포인터 참조).

>* **types**: No type violations (reinterpreting a `T` as a `U` through casts/unions/varargs)
>* **bounds**: No bounds violations (accessing beyond the range of an array)
>* **lifetime**: No leaks (failing to `delete` or multiple `delete`) and no access to invalid objects (dereferencing `nullptr`, using a dangling reference).

프로필은 툴에서 사용할 생각으로 구성했고 사람들에게도 도움이 될 것입니다.
**Enforcement** 단락에서는 어떻게 시행할지 아는 사항에 대해서만 의견을 제시하지는 않았습니다. 몇몇 의견은 툴 개발자를 격려하려는 의도이기도 합니다.
>The profiles are intended to be used by tools, but also serve as an aid to the human reader.
We do not limit our comment in the **Enforcement** sections to things we know how to enforce; some comments are mere wishes that might inspire some tool builder.

## <a name="SS-struct"></a> In.struct: 이 문서의 구조
>## <a name="SS-struct"></a> In.struct: The structure of this document

규칙(가이드라인, 제안)은 여러 부분으로 나뉩니다.
>Each rule (guideline, suggestion) can have several parts:

* 규칙 - **no naked `new`**
* 규칙 번호 - **C.7** (클래스와 관련된 7번째 규칙)
  주요 단원이 순서대로 나열되지 않았기 때문에 참조번호는 문자로 시작함.
  규칙을 추가하거나 삭제할 때 혼란스럽지 않도록 숫자를 매길때 간격을 둠.
* **Reason**s (근거) - 프로그래머는 이해할 수 없는 규칙을 따르지 않으려 하기 때문에 근거를 두어 설명함.
* **Example**s - 내용만으로 규칙을 이해할 수 없기 때문에 좋은 예도 있고 나쁜 예를 보여줌.
* **Alternative**s - "이런 식으로 하지 마라"에 대한 대안을 제시함.
* **Exception**s - 단순하고 일반적인 규칙을 좋아하지만, 많은 규칙이 일반적이지 않게 적용될 수 있으므로 예외를 언급함.
* **Enforcement** - 어떻게 기계적으로 규칙을 체크할지에 대한 아이디어를 제공함.
* **See also**s - 이 문서 내에서나 외부에서 관련된 규칙이나 토의를 위한 참조들을 보여줌.
* **Note**s (의견) - 다른 분류에 들어갈 수 없는 의견들을 보여줌.
* **Discussion** - 더 확장된 근거, 규칙에 적용되지 않는 예제들에 대해 언급함.

>* The rule itself - e.g., **no naked `new`**
>* A rule reference number - e.g., **C.7** (the 7th rule related to classes).
  Since the major sections are not inherently ordered, we use a letter as the first part of a rule reference "number".
  We leave gaps in the numbering to minimize "disruption" when we add or remove rules.
>* **Reason**s (rationales) - because programmers find it hard to follow rules they don't understand
>* **Example**s - because rules are hard to understand in the abstract; can be positive or negative
>* **Alternative**s - for "don't do this" rules
>* **Exception**s - we prefer simple general rules. However, many rules apply widely, but not universally, so exceptions must be listed
>* **Enforcement** - ideas about how the rule might be checked "mechanically"
>* **See also**s - references to related rules and/or further discussion (in this document or elsewhere)
>* **Note**s (comments) - something that needs saying that doesn't fit the other classifications
>* **Discussion** - references to more extensive rationale and/or examples placed outside the main lists of rules

어떤 규칙들은 기계적으로 체크하기가 어렵지만 전문가들이 쉽게 위반 내용을 지적할 수 있는 최소한의 기준을 만족시키고 있습니다.
우리는 기계적인 툴이 시간이 지나면 전문가가 언급한 것을 추정해 낼 수 있고, 규칙들이 시간이 지남에 따라 더 정확하게 체크할 수 있을 정도로 수정될 것으로 기대하고 있습니다.
>Some rules are hard to check mechanically, but they all meet the minimal criteria that an expert programmer can spot many violations without too much trouble.
We hope that "mechanical" tools will improve with time to approximate what such an expert programmer notices.
Also, we assume that the rules will be refined over time to make them more precise and checkable.

규칙은 모든 대안이나 특수한 경우를 언급하는 것보다는 단순화가 목적입니다.
그런 정보는 **Alternative** 단락, [Discussion](#S-discussion) 절에서 찾을 수 있습니다.
만약 당신이 이 룰에 대해 이해가 되지 않거나 동의하지 않는 경우, **Alternative**를 방문하시고, 불안정하거나 빠진 부분에 대해 논의 할 것이 있으면 우리에게 이메일을 보내세요.
>A rule is aimed at being simple, rather than carefully phrased to mention every alternative and special case.
Such information is found in the **Alternative** paragraphs and the [Discussion](#S-discussion) sections.
If you don't understand a rule or disagree with it, please visit its **Discussion**.
If you feel that a discussion is missing or incomplete, send us an email.

이 문서는 언어 메뉴얼이 아닙니다.
기술적으로 완전히 완벽하기 보다는, 기존에 작성 된 코드에 대한 가이드로써 도움이 되기를 기대합니다.
추천할만한 내용은 [the references](#S-references)에서 찾을 수 있습니다.
>This is not a language manual.
It is meant to be helpful
rather than complete, fully accurate on technical details, or a guide to existing code.
Recommended information sources can be found in [the references](#S-references).

## <a name="SS-sec"></a> In.sec: 주 단원
>## <a name="SS-sec"></a> In.sec: Major sections

* [P: 철학](#S-philosophy)
* [I: 인터페이스](#S-interfaces)
* [F: 함수](#S-functions)
* [C: 클래스와 상속](#S-class)
* [Enum: 열거](#S-enum)
* [ES: 연산식과 구문](#S-expr)
* [E: 에러 처리](#S-errors)
* [R: 자원 관리](#S-resource)
* [T: 템플릿과 일반 프로그래밍](#S-templates)
* [CP: 동시성](#S-concurrency)
* [SL: 표준 라이브러리](#S-stdlib)
* [SF: 소스 파일](#S-source)
* [CPL: C스타일 프로그래밍](#S-cpl)
* [PRO: 프로필](#S-profile)
* [GSL: 가이드라인 지원 라이브러리](#S-gsl)
* [FAQ: 자주 묻는 질문의 답변들](#S-faq)

부가 단원:
>Supporting sections:

* [NL: 이름짓기와 배치 구조](#S-naming)
* [PER: 성능](#S-performance)
* [N: 비규칙과 미신](#S-not)
* [RF: 참고문헌](#S-references)
* [Appendix A: 라이브러리](#S-libraries)
* [Appendix B: 현대화 코드](#S-modernizing)
* [Appendix C: 토론](#S-discussion)
* [용어정리](#S-glossary)
* [할일: 분류되지 않은 규칙들](#S-unclassified)

이 단원들은 관련이 있진 않습니다.
>These sections are not orthogonal.

각각의 단원("P"는 "Philosophy"), 각각의 부단원("C.hier"는 "Class Hierarchies (OOP)")들은 검색, 참조 편의를 위해 약칭이 있습니다.
주 단원에 대한 약칭은 규칙번호에도 사용됩니다.("C.11"은 "Make concrete types regular")
>Each section (e.g., "P" for "Philosophy") and each subsection (e.g., "C.hier" for "Class Hierarchies (OOP)") have an abbreviation for ease of searching and reference.
The main section abbreviations are also used in rule numbers (e.g., "C.11" for "Make concrete types regular").
