이 문서는 기존의 C++ 코드를 모던 C++ 코드로 변환하는 방법에 대해서 다루고 있는 것도 아니다.
다만, 새로운 C++ 코드에 대한 논리 정연한 생각들을 구체적으로 설명하고자 하였을 따름이다.
따라서 기존 코드를 현대적이며, 젊고 활기차게 업그레이드 하고 싶다면 [the modernization section](#S-modernizing)를 참조하기 바란다.
이 문서에 다루고 있는 규칙들을 점진적으로 적용해 갈 수 있다는 것은 이해하는 것은 매우 중요하다. 엄청난 양의 코드를 단번에 바꿀 수는 없는 노릇이다.
>This is not a guide on how to convert old C++ code to more modern code.
It is meant to articulate ideas for new code in a concrete fashion.
However, see [the modernization section](#S-modernizing) for some possible approaches to modernizing/rejuvenating/upgrading.
Importantly, the rules support gradual adoption: It is typically infeasible to convert all of a large code base at once.

이 가이드라인이 언어의 기술적인 세부사항을 완벽하고 정확하게 설명하지는 않는다. 
언어의 명세, 일반 규칙에 대한 예외 사항, 그외  세부 기능들에 대해서는 ISO C++ 표준을 참조하기 바란다.
>These guidelines are not meant to be complete or exact in every language-technical detail.
For the final word on language definition issues, including every exception to general rules and every feature, see the ISO C++ standard.

규칙들을 작성할 때 C++의 일부 기능만을 이용하여 코드를 작성하도록 의도하지 않았다.
마치 C++의 일부만을 때어놓은 Java를 사용하는 것처럼 강제하기 위함은 *절대로* 아니라는 것이다.
"단 하나의 올바른 C++" 언어라는 식의 정의 또한 의도하는 바가 아니다.
다만 가이드라인에 포함된 규칙을 통해서 C++라는 언어가 성능과 타협하지 않으면서도 풍부한 표현력을 지닌 언어라는 점을 강조하고자 하였다.
>The rules are not intended to force you to write in an impoverished subset of C++.
They are *emphatically* not meant to define a, say, Java-like subset of C++.
They are not meant to define a single "one true C++" language.
We value expressiveness and uncompromised performance.

세부 규칙들을 통해서 전달하고자 가치는 명확하다.
기존의 C++ 코드보다 더욱 간단하고, 올바르며, 안전한 코드를 성능 손실없이 작성할 수 있도록 돕는 것이다.
또한 유효한 C++ 코드이긴 하지만 에러를 유발할 가능성이 높고, 불필요하게 복잡하고, 성능도 좋지 않은 코드를 피할 수 있도록 돕는 것이다.
>The rules are not value-neutral.
They are meant to make code simpler and more correct/safer than most existing C++ code, without loss of performance.
They are meant to inhibit perfectly valid C++ code that correlates with errors, spurious complexity, and poor performance.

## <a name="SS-force"></a> In.force: 적용
>## <a name="SS-force"></a> In.force: Enforcement

다양한 규칙들을 적용하도록 사실상 강제화 하지 않고서는 방대한 코드에 이러한 규칙들이 모두 적용되길 기대하는 것은 어려운 일이다.
실상, 모든 규칙을 강제적으로 적용하는 것은 규칙의 수가 몇 개 되지 않거나 혹은 특수한 사용자 집단에서나 가능한 일일지도 모르겠다.
그러나, 개발자들은 여전히 다양한 규칙들을 기대하고 있으며, 이러한 규칙들에 대한 서로 다른 기대치를 가지고 있다고 생각한다. 
역설적이지만, 어떤 개발자도 이처럼 방대한 규칙을 모두 읽고 이해하고 싶어하지 않을 것이며, 각각의 규칙을 낱낱이 기억하고 싶지도 않을 것이다.
이런 이유로 다양한 기대치의 공통적인 부분만을 뽑아내려고도 해보았지만, 이처럼 임의의 규칙을 정하는 것 조차 혼돈을 초래할 것으로 생각했다.
우리는 더 많은 개발자들에게 도움이 되고, 코드를 좀더 간결하게 만들 수 있으며, 기존 코드를 현대화 할 수 있는 그런 가이드라인이 만들고 싶었다.
개인의 선택의 문제라거나 관리의 압박으로 인해 신경쓰지 않았던 부분들도 도외시 하지 않고 최선의 실용적인 예를 다루고자 하였다.
따라서, 바라건데는 모든 규칙들을 적용하는 것이 좋다고 생각한다. 그렇게 해야만 최고의 이득을 얻을 수 있을 것으로 생각하기 때문이다.

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

이는 꽤나 심각한 딜레마가 아닐 수 없는데, 우리는 이러한 딜레마의 해결책이 툴을 개발하는 것이라고 생각하였다.
각각의 규칙들은 적용 방법을 설명하고 있는 **Enforcement** 단락을 가지고 있는데, 
코드 리뷰, 정적 분석, 컴파일러, 런타임 체크 등의 방법을 나열하고 있다.
어떤 방식이 되던 우리는 "기계적"이며(사람은 느리기도 하거니와, 쉽게 지루해 할 수 있으므로) 일관된 방법으로 개별 규칙들이 적용될 수 있는 방법을 원했다.
이런 이유로 런타임 체크는 다른 대안이 없을 경우에 한해서만 언급하였다.
이 같은 내용들을 여기저기에 흩어 놓기 보다는 사용자가 원할 경우 손쉽게 찾을 수 있도록 하고 싶었기 때문에,
**Enfocement** 단락 내에 적절한 위치라고 생각되는 곳에 연관된 규칙들을 "프로필"이라는 단위로 이름을 붙여 나열 해 두었다.
하나의 규칙은 여러 프로필에 속할 수 있으며, 어떤 프로필에도 속하지 않은 규칙들도 있다.
자주 사용되는 프로필 몇 가지를 먼저 살펴보자.

>This adds up to quite a few dilemmas.
We try to resolve those using tools.
Each rule has an **Enforcement** section listing ideas for enforcement.
Enforcement might be by code review, by static analysis, by compiler, or by run-time checks.
Wherever possible, we prefer "mechanical" checking (humans are slow and bore easily) and static checking.
Run-time checks are suggested only rarely where no alternative exists; we do not want to introduce "distributed fat" - if that's what you want, you know where to find it.
Where appropriate, we label a rule (in the **Enforcement** sections) with the name of groups of related rules (called "profiles").
A rule can be part of several profiles, or none.
For a start, we have a few profiles corresponding to common needs (desires, ideals):

* **types**: 타입 위반 없음 (형변환, 유니언, 가변매개변수 통해 `T`를 `U`를 재해석 하는 등)
* **bounds**: 범위 위반 없음 (배열의 범위를 넘어서 접근 하는 등)
* **lifetime**: 누수 없음 (`delete`나 `delete []`등이 실패하는 등), 유효하지 않은 객체에 접근하지 않음(`nullptr`나 dangling된 참조를 통해 역참조 하는 등)

>* **types**: No type violations (reinterpreting a `T` as a `U` through casts/unions/varargs)
>* **bounds**: No bounds violations (accessing beyond the range of an array)
>* **lifetime**: No leaks (failing to `delete` or multiple `delete`) and no access to invalid objects (dereferencing `nullptr`, using a dangling reference).

프로필은 툴에서 활용할 용도로 만들었지만 직접 내용을 직접 읽어 보아도 적잖이 도움이 될 것이다.
**Enforcement** 단락을 단순히 우리가 알고 있는 적용 방법을 제시하는 것으로만 제한하고 싶은 생각은 없다.
이 단락을 통해서 일부 개발자는 툴 개발자에게 도움이 될만한 글을 남기고 싶을 수도 있을 것이다.
>The profiles are intended to be used by tools, but also serve as an aid to the human reader.
We do not limit our comment in the **Enforcement** sections to things we know how to enforce; some comments are mere wishes that might inspire some tool builder.

## <a name="SS-struct"></a> In.struct: 이 문서의 구조
>## <a name="SS-struct"></a> In.struct: The structure of this document

규칙(가이드라인, 제안)은 여러 부분으로 나뉜다.
>Each rule (guideline, suggestion) can have several parts:

* 규칙 자체 - 예. **`naked new를 사용하지 마라`**
* 규칙 번호 - 예. **C.7** (클래스와 관련된 7번째 규칙)
  주요 단원이 순서대로 나열되지 않았기 때문에 참조번호는 문자로 시작함.
  규칙을 추가하거나 삭제할 때 혼란스럽지 않도록 숫자를 매길때 간격을 둠.
* **Reason**s - 이해할 수 없는 규칙을 따르고 싶지는 않을 것이기 때문에 구체적인 이유를 설명함.
* **Example**s - 추상적인 표현만으로 내용을 이해하기 어려울 경우 구체적인 예를 기술. 좋은 예나 나쁜 예를 나타낼 수 있음.
* **Alternative**s - "이런 식으로 하지 마라"에 대한 대안을 제시함.
* **Exception**s - 단순하고 보편적인 규칙이 좋고, 일반적으로 대부분의 규칙들이 많은 경우에 적용 가능하지만 예외가 있을 경우 이를 나열함.
* **Enforcement** -  기계적으로 규칙을 확인하는 방법에 대한 아이디어를 기술.
* **See also**s - 연관된 규칙이나 관련 내용이 추가적으로 다루어지고 있는 부분에 대한 참조(이 문서나 혹은 이 문서외도 될 수 있음)
* **Note**s (의견) - 다른 곳에서 다루기에 적합하지 않은 부분에 대하여 추가 설명.
* **Discussion** - 규칙을 제시한 근본적인 이유를 담고 있는 다른 글에 대한 참조나 규칙의 주요 리스트에 포함되지 않는 예 등을 설명.

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

일부 규칙들은 기계적으로 확인하기는 어려울 수 있으나 전문성을 가진 프로그래머라면 손쉽게 위반 여부를 발견할 수 있는 것들도 있다.
규칙들은 시간이 지남에 따라 좀더 정밀하고, 기계적으로 확인 가능하도록 개선되기를 기대한다.
>Some rules are hard to check mechanically, but they all meet the minimal criteria that an expert programmer can spot many violations without too much trouble.
We hope that "mechanical" tools will improve with time to approximate what such an expert programmer notices.
Also, we assume that the rules will be refined over time to make them more precise and checkable.

각각의 규칙들이 가능한 모든 대안과 특별한 예외사항까지를 모두 언급하기를 바라지는 않는다. 이 보다 각각의 규칙들은 가능한 단순하게 유지되고,
지엽적인 내용이나 다른 대안은 **Alternative** 단락과 [Discussion](#S-discussion) 절에서 살펴 볼 수 있도록 하였으면 한다.
규칙을 이해할 수 없거나 동의하지 않는다면, **Discussion**을 먼저 살펴보기 바란다.
또한, 의구심을 가지고 있는 부분이 앞서 논의되지 않았거나, 불완전하다고 생각된다면 메일을 보내주었으면 한다.

>A rule is aimed at being simple, rather than carefully phrased to mention every alternative and special case.
Such information is found in the **Alternative** paragraphs and the [Discussion](#S-discussion) sections.
If you don't understand a rule or disagree with it, please visit its **Discussion**.
If you feel that a discussion is missing or incomplete, send us an email.

이 문서는 언어어 대한 메뉴얼이 아니다.
따라서 기술적인 세부사항을 완벽하고 정밀하게 다루기 보다는 기존에 작성 된 코드에 대한 가이드로써의 역할을 하였으면 한다.
추천할만한 내용은 [the references](#S-references)에서 찾을 수 있다.

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

각각의 단원들은 상호 직교성을 띄지는 않는다.
>These sections are not orthogonal.

각각의 단원("P"는 "Philosophy")과 부단원("C.hier"는 "Class Hierarchies (OOP)")은 검색, 참조의 편의를 위해 약칭을 가진다.
주 단원에 대한 약칭으로 규칙 번호를 사용하기도 한다.(즉, "Make concrete types regular"을 "C.11"로 나타내기도 한다.)
>Each section (e.g., "P" for "Philosophy") and each subsection (e.g., "C.hier" for "Class Hierarchies (OOP)") have an abbreviation for ease of searching and reference.
The main section abbreviations are also used in rule numbers (e.g., "C.11" for "Make concrete types regular").
