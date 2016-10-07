#개요

># Abstract

이 문서는 C++를 올바로 사용하기 위한 가이드라인의 모음이며,
개발자들이 moden c++를 좀 더 효율적으로 사용할 수 있도록 돕기 위해 작성되었다.
"modern C++"이란 C++11, C++14 (그리고 곧 나올 C++17)을 뜻한다.
지금 개발중인 코드가 5년 후에는 어떤 모습이었으면 좋겠는가? 혹은 처음부터 다시 개발을 시작한다면 그 코드가 10년 후에 어떤 모습이었으면 좋겠는지 생각해 본적이 있는가?

> This document is a set of guidelines for using C++ well.
The aim of this document is to help people to use modern C++ effectively.
By "modern C++" we mean C++11 and C++14 (and soon C++17).
In other words, what would you like your code to look like in 5 years' time, given that you can start now? In 10 years' time?

본 가이드라인은 인터페이스, 자원 관리, 메모리 관리, 동시성와 같이 상대적으로 고수준의 내용을 다루고 있으며,
이러한 내용들은 대체로 응용 프로그램의 아키텍쳐나 라이브러리의 설계에 영향을 미친다.
본 가이드라인을 준수하면 코드가 정적인 타입 안정성을 가지도록 할 수 있으며, 리소스의 누수가 없고, 
흔히 저지르는 수 많은 프로그래밍 논리 오류를 잡아낼 수 있으며, 
더욱 빠르게 수행되는 코드를 작성할 수 있을 것이다. - 지금 바로 시작하길 바란다.

> The guidelines are focused on relatively higher-level issues, such as interfaces, resource management, memory management, and concurrency.
Such rules affect application architecture and library design.
Following the rules will lead to code that is statically type safe,
has no resource leaks, and catches many more programming logic errors than is common in code today.
And it will run fast - you can afford to do things right.

이름 지정 규칙이나 들여쓰기 스타일 같은 세부적인 내용에 대해서는 크게 언급하지 않을 것이나,
이 중 특별히 도움이 될만한 토픽이 있다면 다루도록 하겠다.  

> We are less concerned with low-level issues, such as naming conventions and indentation style.
However, no topic that can help a programmer is out of bounds.

초기의 규칙들은 (다양한 형태의) 안정성과 간결함을 과도하게 강조한 측면이 있다.
실용성을 위해 여러 예외 사항들을 소개하였으나 부족함이 있을 것이며, 더 많은 규칙이 필요하기도 하다.

> Our initial set of rules emphasize safety (of various forms) and simplicity.
They may very well be too strict.
We expect to have to introduce more exceptions to better accommodate real-world needs.
We also need more rules.

앞으로 소개할 규칙들 중에는 독자의 에상과 크게 어긋나거나 기존 경험과 전혀 다른 부분도 있을 것이다.
어떤식이던 기존의 코딩 스타일에 변화를 주지 못한다면, 이 문서는 실패한 것이나 다름이 없다.
올바른 규칙인지 검증해 보고, 다른 의견을 알려 주기 바란다.
특히, 더 나은 방법이나 사례가 있다면 꼭 알려주었으면 한다.

> You will find some of the rules contrary to your expectations or even contrary to your experience.
If we haven't suggested you change your coding style in any way, we have failed!
Please try to verify or disprove rules!
In particular, we'd really like to have some of our rules backed up with measurements or better examples.

일부 규칙 중에는 너무 뻔하거나 사소해 보이는 것도 있을 것이다.
하지만 이 가이드라인이 경험이 많지 않고, 다양한 배경을 가지고 있는 개발자들이 C++ 언어에 빨리 익숙해 질 수 있도록 돕기 위한 것임을 기억해 주었으면 한다.

> You will find some of the rules obvious or even trivial.
Please remember that one purpose of a guideline is to help someone who is less experienced or coming from a different background or language to get up to speed.

개별 규칙들은 분석 도구를 통해서 검증이 가능하도록 만들어졌다.
이러한 도구를 이용하면 어떤 규칙을 위배하였는지 그리고 그 규칙의 내용(혹은 링크)이 무엇인지를 확인할 수 있을 것이다.
모든 규칙을 완벽히 기억하고 코드를 작성할 수는 없는 노릇이다.

> The rules are designed to be supported by an analysis tool.
Violations of rules will be flagged with references (or links) to the relevant rule.
We do not expect you to memorize all the rules before trying to write code.

본 문서에서 다루고 있는 다양한 규칙들이 코드에 점진적으로 도입되길 희망하며, 이를 위한 도구를 만들 계획도 가지고 있다. 
다른 분들도 이러한 도구를 함꼐 만들어 주었으면 한다.

> The rules are meant for gradual introduction into a code base.
We plan to build tools for that and hope others will too.

다양한 의견과 제안은 언제든 대환영이다. 
규칙에 대한 이해도가 높아지고, 언어와 라이브러리가 개선될 때마다 이 문서를 지속적으로 수정하고 확장 해 나갈 것이다.

> Comments and suggestions for improvements are most welcome.
We plan to modify and extend this document as our understanding improves and the language and the set of available libraries improve.
