# 부록 B: 코드 현대화하기

이상적으로,  우리는 모든 룰을 따라 코드를 짠다.
하지만 현실적으로, 우리는 다음과 같은 수많은 옛날 코드를 다루어야 한다.
>Ideally, we follow all rules in all code.
Realistically, we have to deal with a lot of old code

* 가이드라인이 형성되기 전이나, 알려지기 이전에 짜여진 어플리케이션 코드
* 오래됐거나 상이한 표준에 맞추어 짜여진 라이브러리
* 미처 현대화할 생각을 못했던 코드들

>* application code written before the guidelines were formuated or known
* libraries written to older/different standards
* code that we just haven't gotten around to modernizing

수백만줄에 달하는 코드가 있는데, 이걸 단 한 번에 모두 바꿔버린다는 생각은 일반적으로 비현실적이다. 그래서 단계적으로 코드 베이스를 현대화할 필요가 있다.
>If we have a million lines of new code, the idea of "just changing it all at once" is typically unrealistic.
Thus, we need a way of gradually modernizing a code base.

옛날 코드를 모던 스타일로 업그레이드하는 일은 좀 벅찬 일이다.
보통 옛날 코드들은 이해하기 아주 어렵도록 지저분한데, (적어도 현재의 사용범위 안에서만큼은) 작동이 정확히 된다.
일반적으로 그 프로그램을 처음 짠 프로그래머는 주변에 없고, 테스트 케이스는 완전치 않다.
코드가 지저분하면, 조그만 변화만 주려고 해도 엄청난 노력이 필요하고, 에러가 발생하는 확률도 급격히 증가한다.
대체로 지저분하고 오래된 코드들은 쓸데없이 느리게 도는데, 구식 컴파일러를 통해서만 컴파일되고 요즘 하드웨어의 혜택을 전혀 받지 못하기 때문이다.
그래서 많은 경우에, 메이저 업그레이드를 하기 위해서는 여러 프로그램들의 도움이 필요하게 된다.
>Upgrading older code to modern style can be a daunting task.
Often, the old code is both a mess (hard to understand) and working correctly (for the current range of uses).
Typically, the original programmer is not around and test cases incomplete.
The fact that the code is a mess dramatically increases to effort needed to make any change and the risk of introducing errors.
Often messy, old code runs unnecessarily slowly because it requires outdated compilers and cannot take advantage of modern hardware.
In many cases, programs support would be required for major upgrade efforts.

코드를 현대화하는 이유는 다음과 같다. 새 기능을 추가하는 것을 쉽게 하기 위해서, 유지보수성을 향상시키기 위해서, 성능(처리량과 응답속도)을 향상시키 위해서, 요즘의 하드웨어를 충분히 활용하기 위해서.
코드를 "보기 예쁘게 만들기"라든가 "모던 스타일을 따르기"는 그 자체로 코드 개선의 이유가 될 수는 없다.
구식 코드를 가지고 있다는 것은 (기회 비용을 포함한) 비용이 드는 일이지만, 코드에 변화를 주는 것은 또 그만큼의 리스크가 있는 일이다.
그래서, 코드 현대화를 통해 절감되는 비용은, 그에 따르는 리스크보다 커야만 한다.
>The purpose of modernizing code is to simplify adding new functionality, to ease maintenance, and to increase performance (throughput or latency), and to better utilize modern hardware.
Making code "look pretty" or "follow modern style" are not by themselves reasons for change.
There are risks implied by every change and costs (including the cost of lost opportunities) implied by having an outdated code base.
The cost reductions must outweigh the risks.


그렇다면 어떻게 해야할까?
>But how?


코드 현대화에 단 하나의 정답이 있는 것은 아니다.
코드에 따라, 업데이트에 대한 압박에 따라, 개발자의 배경에 따라, 사용가능한 툴에 따라 어떤 방법이 좋을지가 달라진다.
여기에서는 몇 가지 (아주 일반적인) 방법을 소개하고자 한다.
>There is no one approach to modernizing code.
How best to do it depends on the code, the pressure for updates, the backgrounds of the developers, and the available tool.
Here are some (very general) ideas:

* 가장 좋은 방법은 "모든 코드를 업그레이드 하기"다. 총 투입시간 대비 이득이 가장 크다.
하지만 대부분의 상황에서는 불가능 할 것이다.
* 모듈 하나씩 바꿔가는 식으로 코드를 컨버트할 수도 있다.하지만 인터페이스(특히 ABI)에 영향을 주는 룰들은 모듈 하나씩 바꿔가는 식으로 적용할 수가 없다. 예를 들어 [use `array_view`](#S-GSL), 같은 룰이 그렇다.
* 적용했을 때 가장 큰 이득을 얻을 수 있거나, 가장 적은 오류를 일으킬 것같은 룰부터 적용하여 코드를 컨버트하는 "상향식 접근법"도 있다.
* 인터페이스에 초점을 맞추어 시작할 수도 있다. 예를 들어, 어떤 시스템 자원도 누수되지 않고 어떤 포인터도 오용되지 않는 것을 확실하게 하는 것이다. 이 방법을 적용하면 전체 코드 베이스를 관통하여 이곳 저곳에 변화를 주어야겠지만, 매우 큰 효용을 얻을 수 있다.

>* The ideal is "just upgrade everything." That gives the most benefits for the shortest total time.
In most circumstances, it is also impossible.
* We could convert a code base module for module, but any rules that affects interfaces (especially ABIs), such as [use `array_view`](#S-GSL), cannot be done on a per-module basis.
* We could convert code "botton up" starting with the rules we estimate will give the greatest benefits and/or the least trouble in a given code base.
* We could start by focusing on the interfaces, e.g., make sure that no resources are lost and no pointer is misused.
This would be a set of changes across the whole code base, but would most likely have huge benefits.

어떤 방법을 선택하든, 코어 가이드라인을 충실히 따를 때 가장 큰 이득을 얻을 수 있다는 것을 유념하기 바란다.
이 가이드라인은 잘돌아가기를 기대하는 마음으로 아무거나 하나 골라서 적용해보는, 관계없는 규칙들의 마구잡이식 모음집이 아니다.
>Whichever way you choose, please note that the most advantages come with the highest conformance to the guidelines.
The guidelines are not a random set of unrelated rules where you can srandomly pick and choose with an expectation of success.

여러분들의 경험과 코드 현대화를 위해 사용하는 툴들을 대한 이야기를 진심으로 듣고 싶다.
코드 현대화는 분석 툴, 코드 변환툴 따위의 도움을 받을 때 더 신속하고, 단순하고, 안전하게 이루어질 수 있다.
>We would dearly love to hear about experience and about tools used.
Modernization can be much faster, simpler, and safer when supported with analysis tools and even code transformation tools.
