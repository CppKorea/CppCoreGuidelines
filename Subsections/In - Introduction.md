# <a name="S-introduction"></a> In: Introduction

이 문서는 modern C++, C++14 뿐 아니라 추후 개션될 것으로 보이는 내용과 ISO 기술 명세(Technical Specification까지를 고려한 핵심 가이드라인이며,
C++ 프로그래머가 더욱 간단하고, 효과적이며, 유지보수하기 좋은 코드를 작성하도록 돕기 위해 작성되었다.
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

## <a name="SS-readers"></a> In.target: 대상 독자층
>## <a name="SS-readers"></a> In.target: Target readership

모든 C++ 프로그래머가 대상이며 여기에 [programmers who might consider C](#S-cpl)도 포함된다.
>All C++ programmers. This includes [programmers who might consider C](#S-cpl).

## <a name="SS-aims"></a> In.aims: 목적
>## <a name="SS-aims"></a> In.aims: Aims

이 문서의 목적은 개발자들이 모던 C++(C++11, C++14, 이후 C++17까지)을 도입하는 것을 돕고, 코드 작성시에 더욱 일관된 스타일을 적용하도록 하기 위함이다.
>The purpose of this document is to help developers to adopt modern C++ (C++11, C++14, and soon C++17) and to achieve a more uniform style across code bases.

개별적인 규칙 하나하나가 모든 코드에 효과적으로 적용될 것이라고 생각하진 않는다.
오래된 시스템을 업그레이드하는 것은 어려운 작업이다. 그럼에도 이러한 규칙들을 적용하면 기존의 방식보다 에러를 유발할 가능성이 낮고, 유지보수도 편리한 코드를 작성할 수 있을 것이라 확신한다. 종종 이러한 규칙들이 초기 개발을 더욱 신속하고 쉽게 진행할 수 있도록 도와주기도 한다.
확실히 말할 수 있는 것은, 이러한 규칙들을 적용하면 이전 보다 더 잘 수행되는 코드를 만들 수 있으며, 제로-비용 원칙(Zero-overhead principle)을 따르고 있어서 더욱 보편 타당한 기법이라는 것이다. (제로-비용 원칙이란 "사용하지 않는 부분에 비용을 낭비하지 말라", "올바른 추상화 방법을 사용하여 적어도 저수준 언어로 하드코딩 한 것 만큼의 성능을 얻을 수 있도록 하라" 정도로 설명될 수 있다.)
새로운 코드를 작성하거나 기존 코드를 개선할 기회가 생겼을 때 규칙들을 적용해 보는 것이 좋다. 이런 기회에 적용 가능한 수준에서 규칙들을 활용해 보라.  
기억해라:
>We do not suffer the delusion that every one of these rules can be effectively applied to every code base. Upgrading old systems is hard. However, we do believe that a program that uses a rule is less error-prone and more maintainable than one that does not. Often, rules also lead to faster/easier initial development.
As far as we can tell, these rules lead to code that performs as well or better than older, more conventional techniques; they are meant to follow the zero-overhead principle ("what you don't use, you don't pay for" or "when you use an abstraction mechanism appropriately, you get at least as good performance as if you had handcoded using lower-level language constructs").
Consider these rules ideals for new code, opportunities to exploit when working on older code, and try to approximate these ideas as closely as feasible.
Remember:

### <a name="R0"></a> In.0: 당황하지 마라!
>### <a name="R0"></a> In.0: Don't panic!

개발 중인 프로그램에 가이드라인을 적용할 경우 어떤 영향이 있을지 이해하기 위해서 시간을 할애하기 바란다.  
>Take the time to understand the implications of a guideline rule on your program.

가이드라인은 상위집합의 부분집합 원칙([Stroustrup05](#Stroustrup05))에 따라 구성하였다.
신뢰성, 안정성, 성능 등을 고려하여 흔히 사용되는 C++의 일부 기능만을 이용하여 가이드라인을 정의하지 않았다.
대신, 몇가지 간단한 "확장" 컴포넌트([library components](#S-gsl))를 사용하도록 강력히 권고하고 있는데, 이를 이용하면 다소 장황하여 에러를 유발할 가능성이 높은 C++의 기능들을 배제할 수 있다.
>These guidelines are designed according to the "subset of a superset" principle ([Stroustrup05](#Stroustrup05)).
They do not simply define a subset of C++ to be used (for reliability, safety, performance, or whatever).
Instead, they strongly recommend the use of a few simple "extensions" ([library components](#S-gsl))
that make the use of the most error-prone features of C++ redundant, so that they can be banned (in our set of rules).

가이드라인에 포함된 규칙들은 정적 타입 안정성과 리소스 안정성에 주안점을 두고 있다.
이런 이유로 범위 확인 가능성, `nullptr`를 통한 역참조 회피 가능성, dangling 포인터 회피 가능성, 시스템적인 예외 사용(RAII를 통한) 등을 강조하고 있으며,
소스 코드의 오류 발생 가능성을 부분적으로 극복하거나 최소화 하는 방법들, 단순한 표현 방식, 더불어 올바르게 정의된 인터페이스를 통해 복잡도를 드러내지 않는 방법 등을 중요시 하고 있다.
>The rules emphasize static type safety and resource safety.
For that reason, they emphasize possibilities for range checking, for avoiding dereferencing `nullptr`, for avoiding dangling pointers, and the systematic use of exceptions (via RAII).
Partly to achieve that and partly to minimize obscure code as a source of errors, the rules also emphasize simplicity and the hiding of necessary complexity behind well-specified interfaces.

많은 규칙들은 규범적인 성격을 띄고 있다.
대안도 없이 그저 "그렇게 하지 마라!"라고만 하는 규칙들은 매우 불편할 수 밖에 없다.
일부 규칙들은 정밀하고 기계적으로 검증할 수 있다기 보다는 경험에 근거하여 제시된 규칙들도 있다. 
그외의 다른 규칙들은 보편적인 원칙을 논리적으로 정리한 것인데, 일반화의 수준이 높을수록 더욱 자세하게 설명하였고 부분적으로 나마 검증이 가능하도록 구체적으로 규칙을 규정하였다.

>Many of the rules are prescriptive.
We are uncomfortable with rules that simply state "don't do that!" without offering an alternative.
One consequence of that is that some rules can be supported only by heuristics, rather than precise and mechanically verifiable checks.
Other rules articulate general principles. For these more general rules, more detailed and specific rules provide partial checking.

가이드라인은 C++의 핵심과 그 사용법을 다루고 있다. 
C++을 사용하는 조직의 규모가 크거나, 매우 특화된 분야의 응용 프로그램을 개발하는 경우이거나 혹은 프로젝트의 규모가 방대한 경우라면 더 다양한 규칙과 제약, 그리고 더 많은 라이브러리가 필요할 것이다.
예를 들어, 고도의 실시간 응용 프로그램을 개발하는 경우라면 자유 저장소(free store, 동적 메모리)를 아무렇게나 사용하면 안되기 때문에, 라이브러리를 선택하는데 제약이 있을 수 밖에 없다.
이처럼 특화된 개발 분야에만 적용 할 수 있는 규칙들은 핵심 가이드라인의 부록으로 다루었으면 한다.
어셈블리 코드와 같이 저수준의 프로그래밍 방식을 고수하기 보다는 핵심을 담고 있는 소규모의 라이브러리를 만들고 사용하기 바란다.

>These guidelines address a core of C++ and its use.
We expect that most large organizations, specific application areas, and even large projects will need further rules, possibly further restrictions, and further library support.
For example, hard real-time programmers typically can't use free store (dynamic memory) freely and will be restricted in their choice of libraries.
We encourage the development of such more specific rules as addenda to these core guidelines.
Build your ideal small foundation library and use that, rather than lowering your level of programming to glorified assembly code.

개별적인 규칙들은 점진적으로 적용[gradual adoption](#S-modernizing)해 볼 수 있다.
>The rules are designed to allow [gradual adoption](#S-modernizing).

일부 규칙들은 다양한 방식으로 안전성을 높이기 위한 목적으로 만들어졌으며, 또 다른 일부는 문제 발생 가능성을 낮추거나 혹은 이 둘을 모두 고려하여 만들어진 규칙들도 있다.
규칙에 내포되어 있는 아이디어를 표현할 때, 통상 오류 발생 가능성이 높은 경우와 그렇지 않은 경우가 있다면, 가능한 오류 발생 가능성이 낮은 쪽을 택하였다.
>Some rules aim to increase various forms of safety while others aim to reduce the likelihood of accidents, many do both.
The guidelines aimed at preventing accidents often ban perfectly legal C++.
However, when there are two ways of expressing an idea and one has shown itself a common source of errors and the other has not, we try to guide programmers towards the latter.

## <a name="SS-non"></a> In.not: 목적이 아닌 것
>## <a name="SS-non"></a> In.not: Non-aims

최소한의 규칙만을 정의하고, 그러한 규칙들이 직교성을 띄도록 정밀하게 구성하려 하지는 않았다.
실제로, 범용적인 규칙 중 일부는 일면 간단해 보일 수도 있지만 실제로 적용하기가 매우 어려울 수 있으며, 그러한 규칙이 의미하는 함축적인 의미를 이해하기가 매우 어려울 수도 있다.
반대로, 구체적인 규칙들은 이해하기도 쉽고 적용하기도 쉽다. 하지만 범용적인 규칙들을 다루지 않고는 특별한 경우만을 장황하게 나열할 수 밖에 없을 것이다.
가이드라인에 포함된 규칙들은 초보 개발자뿐 아니라 전문가들에게 도움이 될 수 있도록 만들었다.
규칙 중 일부는 반드시 적용되어야만 하는 것들도 있고, 또 다른 일부는 필요에 따라 적용 여부를 결정하는 되는 것들도 있다.
>The rules are not intended to be minimal or orthogonal.
In particular, general rules can be simple, but unenforceable.
Also, it is often hard to understand the implications of a general rule.
More specialized rules are often easier to understand and to enforce, but without general rules, they would just be a long list of special cases.
We provide rules aimed at helping novices as well as rules supporting expert use.
Some rules can be completely enforced, but others are based on heuristics.

규칙들을 책처럼 너무 심각하게 받아들을 필요는 없으며, 링크를 통해서 가볍게 살펴보는 것도 좋다.
사실 이처럼 규칙을 정의한 본질적인 이유는 규칙들을 위배한 코드를 찾고, 그에 대한 링크 정보를 보여주는 툴을 만들 수 있도록 하기 위함이다.
또한, 규칙이 만들어진 이유와 규칙을 따르지 않았을 때 어떤 문제를 유발할 가능성이 있는지 그리고 그 문제를 어떻게 해결할 수 있는지를 알려주기 위함이다.

>These rules are not meant to be read serially, like a book.
You can browse through them using the links.
However, their main intended use is to be targets for tools.
That is, a tool looks for violations and the tool returns links to violated rules.
The rules then provide reasons, examples of potential consequences of the violation, and suggested remedies.

이 가이드라인이 C++의 튜토리얼을 대체하기 위한 목적으로 만들어진 것은 아니다.
개발 수준별 튜토리얼이 필요하다면, [the references](#S-references)를 참조하라.
>These guidelines are not intended to be a substitute for a tutorial treatment of C++.
If you need a tutorial for some given level of experience, see [the references](#S-references).

이 문서는 이전 C++ 코드를 모던 코드로 변환하는 방법에 대해 다루고 있는 것 또한 아니며,
새로운 코드에 대한 논리정연한 생각을 구체적으로 기술하고자 하였을 따름이다.
따라서 코드를 현대적이며, 젊고 활기차게, 업그레이드 하고 싶다면 [the modernization section](#S-modernizing)를 참조하라.
중요한건, 이 문서에 다루고 있는 규칙들은 점진적으로 적용이 가능하다는 것이다. 엄청난 양의 코드를 한번에 바꾸는 것은 그다지 현실적이지 않다.
>This is not a guide on how to convert old C++ code to more modern code.
It is meant to articulate ideas for new code in a concrete fashion.
However, see [the modernization section](#S-modernizing) for some possible approaches to modernizing/rejuvenating/upgrading.
Importantly, the rules support gradual adoption: It is typically infeasible to convert all of a large code base at once.

이 가이드라인을 통해서 언어의 기술적인 세부사항을 완벽하고 정확하게 설명하려 하지 않았다. 
언어의 명세, 일반 규칙에 대한 예외 사항, 그외 모든 세부 기능 들에 대해서는 ISO C++ 표준을 참조하기 바란다.
>These guidelines are not meant to be complete or exact in every language-technical detail.
For the final word on language definition issues, including every exception to general rules and every feature, see the ISO C++ standard.

규칙들을 통해서 C++의 일부 기능만을 이용하여 코드를 작성하도록 의도하기 위해 작성된 것도 아니다.
마치 Java와 같이 C++의 일부만을 사용하도록 강요하기 위함은 *절대로* 아니라는 것이다.
"단 하나의 올바른 C++" 언어라는 식의 정의 또한 의도하는 바가 아니다.
우리는 가이드라인에 포함된 규칙을 통해서 성능과 타협하지 않으면서도 풍부한 표현력을 지닌 언어라는 점을 강조하고자 하였다.
>The rules are not intended to force you to write in an impoverished subset of C++.
They are *emphatically* not meant to define a, say, Java-like subset of C++.
They are not meant to define a single "one true C++" language.
We value expressiveness and uncompromised performance.

규칙들을 통해서 설명하고자 하는 가치는 명확하다.
기존의 C++ 코드보다 더욱 간단하고 올바르며 안전한 코드를 성능 손실없이 작성할 수 있도록 돕고자 함이다.
또한 유효한 C++ 코드이긴 하지만 에러를 유발할 가능성이 높고, 불필요하게 복잡하고, 성능도 좋지 않은 코드를 피할 수 있도록 하기 위함이다.
>The rules are not value-neutral.
They are meant to make code simpler and more correct/safer than most existing C++ code, without loss of performance.
They are meant to inhibit perfectly valid C++ code that correlates with errors, spurious complexity, and poor performance.

## <a name="SS-force"></a> In.force: 적용
>## <a name="SS-force"></a> In.force: Enforcement

규칙들을 사용할 것을 사실상 강제화 하지 않고 방대한 코드에 이러한 규칙들이 적용되길 기대하는 것은 어렵다.
물론, 모든 규칙을 강제적으로 적용하는 것은 규칙의 수가 몇개 되지 않거나 혹은 특수한 사용자 집단에서나 가능한 일일지도 모르겠다.
그럼에도, 개발자들은 여전히 다양한 규칙들을 기대하고, 규칙에 대한 서로 다른 기대치를 가지고 있지만, 
방대한 규칙을 모두 읽고 싶어하지도 않을 것이며, 각각의 규칙을 낱낱이 기억하는 것도 불가능하다.
이런 이유로 우리는 다양한 기대치를 아우르는 공통분모를 뽑아내려고도 했지만, 이처럼 임의의 규칙을 정하는 것 조차 혼돈을 초래할 것으로 생각했다.
우리는 많은 개발자들에게 도움이 되고, 코드를 좀더 간결하게 만들고, 기존 코드를 현대화 할 수 있는 그런 가이드라인이 만들고 싶었다.
개인의 취향이라거나 관리의 압박으로 인해 도외시 하였던 부분들도 남겨두지 않고 최선의 실용적인 예를 다루고자 하였다.
따라서 이상적으로는 모든 규칙들을 적용하는 것이 좋다. 그렇게 해야만 최고의 이득을 얻을 수 있을 것으로 생각하기 때문이다.

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

이는 꽤나 심각한 딜레마이지만 우리는 이를 툴을 이용하여 해결하려 하였다.
개별 규칙들은 실제 적용을 위한 방법을 나열하고 있는 **Enforcement** 단락을 가지고 있는데, 
코드 리뷰, 정적 분석, 컴파일러, 런타임 체크 등을 기술하고 있다.
어떤 방식이던 우리는 "기계적"이며(사람은 느리기도 하거니와, 쉽게 지루해 할 수 있으므로) 일관된 방법으로 개별 규칙들이 적용될 수 있는 방법을 선호하며,
이런 이유로 런타임 체크는 다른 대안이 없을 경우에 한해서만 드물게 제시될 것이다.
관련 내용들을 여기저기에 흩어 놓기 보다는 사용자가 원할 경우 손쉽게 찾을 수 있도록 하고 싶었기 때문에,
**Enfocement** 단락 내에 적절하다고 생각되는 위치에 관련된 규칙들을 "프로필"이라는 단위로 이름을 붙여두었다.
하나의 규칙은 여러 프로필에 속할 수 있으며, 어떤 프로필에도 속하지 않은 규칙들도 있다.
먼저 아주 일반적인 프로필 몇 가지를 살펴보자.

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
* **lifetime**: 누수 없음 (`delete`나 `delete []`등이 실패하는 등)과 유효하지 않은 객체에 접근하지 않음(`nullptr`나 dangling된 참조를 통해 역참조 하는 등)

>* **types**: No type violations (reinterpreting a `T` as a `U` through casts/unions/varargs)
>* **bounds**: No bounds violations (accessing beyond the range of an array)
>* **lifetime**: No leaks (failing to `delete` or multiple `delete`) and no access to invalid objects (dereferencing `nullptr`, using a dangling reference).

프로필은 툴에서 사용될 수 있도록 만들어졌지만, 내용을 직접 읽어 보아도 도움이 될 것이다.
**Enforcement** 단락을 단순히 우리가 알고 있는 적용 방법을 제시하는 것으로만 제한하지는 않을 것이다.
아마 일부 개발자는 툴 개발자에게 도움이 될만한 글을 남기고 싶을 수도 있을 것이다.
>The profiles are intended to be used by tools, but also serve as an aid to the human reader.
We do not limit our comment in the **Enforcement** sections to things we know how to enforce; some comments are mere wishes that might inspire some tool builder.

## <a name="SS-struct"></a> In.struct: 이 문서의 구조
>## <a name="SS-struct"></a> In.struct: The structure of this document

개별 규칙(가이드라인, 제안)은 여러 부분으로 나뉜다.
>Each rule (guideline, suggestion) can have several parts:

* 규칙 자체 - 예. **`naked new를 사용하지 마라`**
* 규칙 번호 - 예. **C.7** (클래스와 관련된 7번째 규칙)
  주요 단원이 순서대로 나열되지 않았기 때문에 참조번호는 문자로 시작함.
  규칙을 추가하거나 삭제할 때 혼란스럽지 않도록 숫자를 매길때 간격을 둠.
* **Reason**s - 프로그래머는 이해할 수 없는 규칙을 따르지 않을 것이기 때문에 이유를 설명함.
* **Example**s - 추상적인 표현만으로 내용을 이해하기 어려울 수 있으므로 구체적인 예를 기술. 좋은 예나 나쁜 예를 나타냄.
* **Alternative**s - "이런 식으로 하지 마라"에 대한 대안을 제시함.
* **Exception**s - 단순하고 보편적인 규칙을 선호하고, 많은 규칙들이 널리 적용 가능하지만 예외가 있을 경우 이를 나열함.
* **Enforcement** -  기계적으로 규칙을 체크하는 방법에 대한 아이디어를 기술.
* **See also**s - 연관된 규칙이나 관련 내용이 추가적으로 다루어지고 있는 부분에 대한 참조(이 문서나 혹은 이 문서외도 될 수 있음)
* **Note**s (의견) - 다른 곳에서 다루기 적합하지 않은 부분에 대하여 추가 설명.
* **Discussion** - 규칙을 제시한 근본적인 이유에 대한 참조나 규칙의 주요 리스트에 포함되지 않는 예 등을 나열.

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

어떤 규칙들은 기계적으로 체크하기가 어렵지만 전문가들이 쉽게 위반 내용을 지적할 수 있는 최소한의 기준을 만족시키고 있다.
우리는 기계적인 툴이 시간이 지나면 전문가가 언급한 것을 추정해 낼 수 있고, 규칙들이 시간이 지남에 따라 더 정확하게 체크할 수 있을 정도로 수정될 것으로 기대하고 있다.
>Some rules are hard to check mechanically, but they all meet the minimal criteria that an expert programmer can spot many violations without too much trouble.
We hope that "mechanical" tools will improve with time to approximate what such an expert programmer notices.
Also, we assume that the rules will be refined over time to make them more precise and checkable.

규칙은 모든 대안이나 특수한 경우를 언급하는 것보다는 단순화가 목적이다.
그런 정보는 **Alternative** 단락, [Discussion](#S-discussion) 절에서 찾을 수 있다.
만약 당신이 이 룰에 대해 이해가 되지 않거나 동의하지 않는 경우, **Alternative**를 방문하고, 불안정하거나 빠진 부분에 대해 논의 할 것이 있으면 우리에게 이메일을 보내라.
>A rule is aimed at being simple, rather than carefully phrased to mention every alternative and special case.
Such information is found in the **Alternative** paragraphs and the [Discussion](#S-discussion) sections.
If you don't understand a rule or disagree with it, please visit its **Discussion**.
If you feel that a discussion is missing or incomplete, send us an email.

이 문서는 언어 메뉴얼이 아니다.
기술적으로 완전히 완벽하기 보다는, 기존에 작성 된 코드에 대한 가이드로써 도움이 되기를 기대한다.
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

이 단원들은 관련이 있진 않다.
>These sections are not orthogonal.

각각의 단원("P"는 "Philosophy"), 각각의 부단원("C.hier"는 "Class Hierarchies (OOP)")들은 검색, 참조 편의를 위해 약칭이 있다.
주 단원에 대한 약칭은 규칙번호에도 사용된다.("C.11"은 "Make concrete types regular")
>Each section (e.g., "P" for "Philosophy") and each subsection (e.g., "C.hier" for "Class Hierarchies (OOP)") have an abbreviation for ease of searching and reference.
The main section abbreviations are also used in rule numbers (e.g., "C.11" for "Make concrete types regular").
