# C++ 핵심 가이드라인 한글화 프로젝트

- 본 프로젝트는 C++ 표준위원회에서 제작한 C++ 핵심 가이드라인을 한글화하는 프로젝트입니다.
- 번역을 시작하기 전에 번역하는 부분이 겹치지 않도록 Issue에 어떤 부분을 번역하는지 알려주시기 바랍니다.
- 원본 문서가 수시로 변경됩니다. 한글로 번역하신 내용 밑에 영어 원문을 인용 문구로 남겨주시기 바랍니다.
- 모든 문서는 Markdown 포맷을 기본으로 합니다.
- 원문의 의도만 바뀌지 않는다면 재밌게 의역해도 상관없습니다. 
- 번역이 완료된 이후에는 편집자분들 통해 편집 과정을 거치게 됩니다.
- 참여를 원하시는 분께서는 Chris Ohk(utilForever)에게 연락주시기 바랍니다.

## 번역을 도와주시는 분들

- Chris Ohk
- Seokjoon Yun
- Giyeon Bang
- Ilgook Jo
- Dongwoo Lee
- Joowon Ryoo
- Gun Son

>"C++은 작고 간단하면서도 안전한 언어이며, 많은 사람들이 사용할 수 있도록 고군분투하고 있다. (Within C++ is a smaller, simpler, safer language struggling to get out.)" 
>-- <cite>비야네 스트롭스트룹(Bjarne Stroustrup)</cite>

비야네 스트롭스트룹을 주도한 C++ 핵심 가이드라인은 많은 사람들이 공동으로 노력한 결과이며 C++ 언어 그 자체다. C++ 핵심 가이드라인은 수 년에 걸쳐 많은 조직에서 다양한 사람들의 토론과 디자인을 거친 결과다. C++ 핵심 가이드라인의 디자인은 범용성과 광범위한 채택을 장려하지만 여러분이 속한 조직의 요구에 맞게 복사하고 수정할 수 있다.
>The C++ Core Guidelines are a collaborative effort led by Bjarne Stroustrup, much like the C++ language itself. They are the result of many person-years of discussion and design across a number of organizations. Their design encourages general applicability and broad adoption but they can be freely copied and modified to meet your organization's needs.

The aim of the guidelines is to help people to use modern C++ effectively. By "modern C++" we mean C++11 and C++14 (and soon C++17). In other 
words, what would you like your code to look like in 5 years' time, given that you can start now? In 10 years' time?

The guidelines are focused on relatively higher-level issues, such as interfaces, resource management, memory management, and concurrency. Such 
rules affect application architecture and library design. Following the rules will lead to code that is statically type safe, has no resource 
leaks, and catches many more programming logic errors than is common in code today. And it will run fast - you can afford to do things right.

We are less concerned with low-level issues, such as naming conventions and indentation style. However, no topic that can help a programmer is 
out of bounds.

Our initial set of rules emphasize safety (of various forms) and simplicity. They may very well be too strict. We expect to have to introduce 
more exceptions to better accommodate real-world needs. We also need more rules.

You will find some of the rules contrary to your expectations or even contrary to your experience. If we haven't suggested you change your 
coding style in any way, we have failed! Please try to verify or disprove rules! In particular, we'd really like to have some of our rules 
backed up with measurements or better examples.

You will find some of the rules obvious or even trivial. Please remember that one purpose of a guideline is to help someone who is less 
experienced or coming from a different background or language to get up to speed.

The rules are designed to be supported by an analysis tool. Violations of rules will be flagged with references (or links) to the relevant rule. 
We do not expect you to memorize all the rules before trying to write code.

The rules are meant for gradual introduction into a code base. We plan to build tools for that and hope others will too.

Comments and suggestions for improvements are most welcome. We plan to modify and extend this document as our understanding improves and the 
language and the set of available libraries improve.
