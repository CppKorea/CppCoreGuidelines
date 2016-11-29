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
