#개요

># Abstract

이 문서는 C++를 적절히 사용하기위한 지침의 모음입니다.
사람들이 modern C++를 효율적으로 사용하는데 도움을 주는것이 그 목적입니다.
여기서 "modern C++"는 C++11, C++14 (그리고 곧 나올 C++17)을 뜻합니다.
즉, 지금 시작을 한경우 5년 뒤 당신의 코드가 어떻게 보여질것 같습니까 ?  10년 뒤에는요 ? (??)

> This document is a set of guidelines for using C++ well.
The aim of this document is to help people to use modern C++ effectively.
By "modern C++" we mean C++11 and C++14 (and soon C++17).
In other words, what would you like your code to look like in 5 years' time, given that you can start now? In 10 years' time?

가이드라인은 interface, 자원 관리, 메모리 관리, 동시성 같은 포괄적인 이슈에 대해 제시할 것입니다.
이런 규칙들은 응용 프로그램의 설계나 라이브러인에 영향을 미칩니다.
이런 규칙들은 요즘의 일반 코드보다 안전하고, 자원누수가 없으며, 프로그램적인 논리 오류를 잡기 쉽게 인도해 줄 것입니다.
그리고 성능적으로도 빨라집니다. - 지금 바로 해야겠죠 ?

> The guidelines are focused on relatively higher-level issues, such as interfaces, resource management, memory management, and concurrency.
Such rules affect application architecture and library design.
Following the rules will lead to code that is statically type safe,
has no resource leaks, and catches many more programming logic errors than is common in code today.
And it will run fast - you can afford to do things right.

이름 지정 규칙이나 들여쓰기 방식 같은 디테일한 이슈에 대해서는 되도록 다루지 않을 것입니다
하지만, 도움이 되는 사항에 대해서는 예외를 두도록 하겠습니다. (?)

> We are less concerned with low-level issues, such as naming conventions and indentation style.
However, no topic that can help a programmer is out of bounds.

초창기에 정한 규칙들은 단순성과 (다양한 형태의) 안전성을 강조하였습니다.
너무 엄격하다 할 수도 있습니다.
실제 필요로 하는 것보다 더 많은 예외들을 소개 할 수도 있습니다.
더 많은 규칙들을 제시하겠구요.

> Our initial set of rules emphasize safety (of various forms) and simplicity.
They may very well be too strict.
We expect to have to introduce more exceptions to better accommodate real-world needs.
We also need more rules.

아마 그동안의 알고있던 것들이나 경험에 반대되는 규칙이라 여기실 수도 있을 겁니다.
여러분의 코딩 스타일을 바꾸거나, 아니면 여러분이 알고 있는 것을 우리에게 제시해주기를 바랍니다.
우리가 맞음을 확인하거나 아니면 우리가 틀렸다는 것을 증명해주세요.
특히, 우리가 제시한 규칙보다 더 좋은 규칙이나 예제를 제시해주시면 감사하겠습니다.

> You will find some of the rules contrary to your expectations or even contrary to your experience.
If we haven't suggested you change your coding style in any way, we have failed!
Please try to verify or disprove rules!
In particular, we'd really like to have some of our rules backed up with measurements or better examples.

여러분은 미처 우리가 생각하지 못했던 디테일한 규칙들에 대해서도 찾을수 있을 것입니다.
이 가이드라인의 목적은 경험이 적거나 다른 배경지식이나 언어에 대한 지식을 가진 사람들이 빨리 익숙해지도록 도와주는 것입니다.

> You will find some of the rules obvious or even trivial.
Please remember that one purpose of a guideline is to help someone who is less experienced or coming from a different background or language to get up to speed.

규칙들은 분석툴에서 지원가능하도록 설계되었습니다.
규칙을 지키기 않으면 관련 규칙에 대한 참조 (또는 링크)로 표시되어 집니다. 
여러분이 코딩하기전에 모든 규칙에 대해서 기억하고 있는 것을 바라지는 않습니다.

> The rules are designed to be supported by an analysis tool.
Violations of rules will be flagged with references (or links) to the relevant rule.
We do not expect you to memorize all the rules before trying to write code.

규칙들을 코드 중심으로 조금씩 소개해 드리겠습니다.
또 관련된 툴들도 만들 계획에 있습니다. (?)

> The rules are meant for gradual introduction into a code base.
We plan to build tools for that and hope others will too.

규칙들이 좀 더 개선 될 수 있도록 많은 의견과 제안을 부탁드리겠습니다.
C++언어와 라이브러리들에 대해 더 이해하기 쉽고, 성능을 향상 시킬수 있도록 이 문서를 계속해서 수정하고 확장하겟습니다.

> Comments and suggestions for improvements are most welcome.
We plan to modify and extend this document as our understanding improves and the language and the set of available libraries improve.
