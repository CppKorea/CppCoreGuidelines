> "C++ 안에는 밖으로 빠져나오려고 애쓰고 있는 더 작고 단순하며 안전한 언어가 있다. (Within C++ is a smaller, simpler, safer language struggling to get out.)"
> -- <cite>비야네 스트롭스트룹(Bjarne Stroustrup)</cite>

# C++ 핵심 가이드라인 한글화 프로젝트

- 본 프로젝트는 C++ 표준위원회에서 제작한 C++ 핵심 가이드라인을 한글화하는 프로젝트입니다.
- 번역을 시작하기 전에 번역하는 부분이 겹치지 않도록 Issue에 어떤 부분을 번역하는지 알려주시기 바랍니다. 또한 원본 문서가 수시로 변경되므로, 한글로 번역하신 내용 밑에 영어 원문을 인용 문구로 남겨주시기 바랍니다.
- 모든 문서는 Markdown 포맷을 기본으로 합니다.
- 원문의 의도만 바뀌지 않는다면 재밌게 의역해도 상관없습니다.
- 번역이 완료된 이후에는 편집자분을 통해 편집 과정을 거치게 됩니다.
- 참여를 원하시는 분께서는 Chris Ohk(utilForever)에게 연락주시기 바랍니다.

## 프로젝트 진행 계획

- 1단계 : 1차 번역 작업 - C++ 핵심 가이드라인 (진행중, ~2015년 12월)
- 2단계 : 2차 번역 작업 - C++ 관련 문서 및 C++ 핵심 가이드라인 갱신 내용, 1차 교정 및 통합 문서 제작 (2016년 1월~)
- 3단계 : 2차 교정 및 PDF 문서 제작 (TBA)

## 프로젝트를 도와주시는 분

- Chris Ohk (utilForever)
- Seokjoon Yun (DevStarSJ)
- Giyeon Bang (GiyeonBang-rhathd)
- Byungwook Eric Ahn (bwahn)
- Eundoo Song (sedman)
- Jongdeok Kim (jenix21)
- Jihoon Song (jihoonsong)
- Junsung Jang (naxxster)
- Jonghee Yun (jongheeyun)
- Changjun Lee (yiatom)
- Sangbae Kang (typeon)
- Minjang Kim (minjang)
- Yonghyun Kim (drvoss)
- Hyungtae Kim (cavecafe)
- Junho Kim (h4wldev)
- Incheol Kang(kangic)

## C++ 핵심 가이드라인 1차 번역 진행 상황

- Abstract (100%)
- C++ Core Guildlines (100%)
- A - Architectural Ideas (100%)
- C - Classes and Class Hierarchies (40%)
- Con - Constants and Immutability (100%)
- CP - Concurrency and Parallelism (40%)
- CPL - C-Style programming (100%)
- E - Error Handling (100%)
- Enum - Enumerations (100%)
- ES - Expressions and Statements (80%)
- F - Functions (80%)
- GSL - Guideline support library (100%)
- I - Interfaces (100%)
- In - Introduction (100%)
- N - Non-Rules and myths (100%)
- NL - Naming and layout rules (100%)
- P - Philosophy (100%)
- PER - Performance (100%)
- PRO - Profiles (100%)
- R - Resource management (20%)
- RF - References (100%)
- SF - Source files (100%)
- STL - The Standard Library (100%)
- T - Templates and generic programming (100%)
- To-do - Unclassified proto-rules (0%)
- Appendix A - Libraries (100%)
- Appendix B - Modernizing code (100%)
- Appendix C - Discussion (5%)

## 개요

비야네 스트롭스트룹이 주도한 C++ 핵심 가이드라인은 많은 사람들이 공동으로 노력한 결과이며 C++ 언어 그 자체다.
C++ 핵심 가이드라인은 수 년에 걸쳐 많은 조직에서 다양한 사람들의 토론과 디자인을 거친 결과다.
C++ 핵심 가이드라인의 디자인은 범용성과 광범위한 채택을 장려하지만 여러분이 속한 조직의 요구에 맞게 복사하고 수정할 수 있다.

가이드라인을 만드는 목적은 사람들이 모던 C++을 효율적으로 사용하도록 도움을 주기 위해서다.
여기서 "모던 C++"이라는 말은 C++11과 C++14(그리고 이후에 나올 C++17)를 의미한다.
다시 말해서 여러분이 지금 시작할 수 있다고 고려한다면, 5년 뒤 여러분의 코드는 어떤 모습이겠는가? 10년 뒤는 어떻겠는가?

가이드라인은 인터페이스, 리소스 관리, 메모리 관리, 동시성과 같이 비교적 높은 수준의 문제에 초점을 맞추고 있다.
이러한 규칙은 애플리케이션 아키텍처와 라이브러리 디자인에 영향을 미친다.
여러분이 규칙을 지킨다면 정적으로 안전한 타입을 갖고, 리소스가 새는 일이 없고, 더 많은 프로그래밍 로직 오류를 잡는 코드를 만들 수 있다.
또한 속도가 빨라진다. 여러분은 제시된 규칙을 통해 코드를 바로잡을 수 있다.

우리는 이름 명명 규칙과 들여 쓰기와 같이 낮은 수준의 문제는 덜 고려하고 있다. 하지만 프로그래머를 도울 수 있는 주제라면 무엇이든 좋다.

우리의 초기 규칙은 (다양한 형태의) 안전함과 단순함을 강조한다. 이 규칙은 너무 엄격할 수 있다.
따라서 현실 세계에 알맞는 요구를 수용하기 위해 더 많은 예외 사항을 도입할 것으로 예상한다. 또한 우리는 더 많은 규칙이 필요하다.

여러분은 기대와 다르거나 경험한 내용과 다른 규칙을 찾을 수 있을 것이다. 만약 여러분이 어떤 방법으로든 코딩 스타일을 바꾸도록 제안하지 않았다면, 우리는 실패한 것이다!
그러니 규칙을 확인하거나 반증해보길 바란다! 특히, 성능 측정 또는 더 나은 예제로 뒷받침되는 규칙을 만들고 싶다.

당신은 심지어 당연하거나 사소한 규칙도 찾을 수 있을 것이다. 하지만 가이드라인의 목적 중 하나는 경험이 적거나 다른 언어를 배운 사람이 모던 C++을 빠르게 배울 수 있도록 돕기 위함을 기억하길 바란다.

우리는 분석 도구를 통해 지원할 수 있도록 규칙을 설계했다. 규칙을 위반하게 되면 관련 규칙에 대한 참조(또는 링크)를 표시한다.
우리는 여러분이 코드를 작성하기 전에 모든 규칙을 기억할 것이라고 기대하지 않는다.

규칙은 코드 베이스로의 점진적인 도입을 의미한다. 우리는 이를 위해 도구를 만들 계획이다.

개선을 위한 의견과 제안은 대부분 환영한다. 우리는 규칙에 대한 해석이 좀 더 나아지고 C++ 언어 및 라이브러리가 개선되는 대로 이 문서를 수정하고 확장할 계획이다.
