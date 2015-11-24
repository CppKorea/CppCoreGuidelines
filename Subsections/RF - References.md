# RF: 레퍼런스
# RF: References

C++을 보다 전문적으로 사용하기 위하여 많은 코딩 표준, 규칙들이 만들어 지고 있다.
Many coding standards, rules, and guidelines have been written for C++, and especially for specialized uses of C++.
Many

* 식별자들에 대한 스펠링과 같이 낮은 수준의 이슈들에 초점을 맞춘 것들
* C++의 초보자들에 의해 작성되어진 것들
* see "stopping programmers from doing unusual things" as their primary aim
* 심지어 10년도 넘은 소스코드를 많은 컴파일러로 이식하고자 할 때 적용하는 것들
* 수십년도 넘은 오래된 코드 베이스를 유지보수 하기 위한 것들
* 단일 애플리케이션 도메인에 적용하기 위한 것들
* 명백한 역효과를 내는 것들
* 자신의 프로젝트를 마무리 하기 위해 개발자 스스로 무시해도 된다 생각하는 것들
* focus on lower-level issues, such as the spelling of identifiers
* are written by C++ novices
* see "stopping programmers from doing unusual things" as their primary aim
* aim at protability across many compilers (some 10 years old)
* are written to preserve decades old code bases
* aims at a single application domain
* are downright counterproductive
* are ignored (must be ignored for programmers to get their work done well)

잘못된 코딩 표준은 차라리 없느니만 못하다.
A bad coding standard is worse than no coding standard.
그러나 적절한 가이드라인이라는 것은 표준이 없는것 보다 대부분 상황에서 좋다: "형식은 자유를 가져 오리라"
However an appropriate set of guidelines are much better than no standards: "Form is liberating."

우리가 원하는 대로 할 수 있고 원하지 않는 것은 허용되지 않도록, 소위 "완벽한 언어"가 될 수 있을까?
Why can't we just have a language that allows all we want and disallows all we don't want ("a perfect language")?
기본적으로, 툴 체인을 포함한 충분한 언어라는 것은 사용자들 그리고, 일반적인것 보다 더 많은 요구사항을 충족시켜야 하는 사용자들에게 기능을 제공해야 한다.
Fundamentally, because affordable languages (and their tool chains) also serve people with needs that differ from yours and serve more needs than you have today.
또한 사용자의 요구 사항은 시간에 따라 변하게 되고, 적용하기 위해서는 범용 언어가 필요해 지게 된다.
Also, your needs change over time and a general-purpose language is needed to allow you to adapt.
언어는 지금은 이상적이라고 생각되지만 조금만 지나면 점차 과도한 제약이 걸리게 된다.
A language that is ideal for today would be overly restrictive tomorrow.

코딩 가이드라인은 특정 요구 사항에 대한 언어 사용성을 조정한다.
Coding guidelines adapt the use of a language to specific needs.
그러므로, 모두를 위한 단일 코딩 스타일은 있을 수 없고,
Thus, there cannot be a single coding style for everybody.
각 개발조직 별로 제약사항을 추가하거나 확고한 스타일의 규칙을 더해야 한다
We expect different organizations to provide additions, typically with more restrictions and firmer style rules.

러퍼런스 섹션:
Reference sections:

* [RF.rules: 코딩 규칙](#SS-rules)
* [RF.books: 코딩 가이드라인 도서](#SS-books)
* [RF.C++: C++ 프로그래밍 (C++11/C++14)](#SS-C++)
* [RF.web: 웹사이트](#SS-web)
* [RS.video: 모던 C++관련 동영상](#SS-vid)
* [RF.man: 메뉴얼](#SS-man)


<a name="SS-rules"></a>
## RF.rules: 코딩규칙

* [Boost Library Requirements and Guidelines](http://www.boost.org/development/requirements.html").
???.
* [Bloomberg: BDE C++ Coding](https://raw.githubusercontent.com/wiki/bloomberg/bde/bdestds.pdf").
코드 구조 및 레이아웃에 중점을 둔.
Has a stong emphasis on code organization and layout.
* Facebook: ???
* [GCC Coding Conventions](https://gcc.gnu.org/codingconventions.html).
C++03과 약간 올드 하지만 그럭저럭 괜찮은.
C++03 and (reasonably) a bit backwards looking.
* [Google C++ Style Guide](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html").
Too timid and reflects its 1990s origins.
[A critique from 2014](https://www.linkedin.com/pulse/20140503193653-3046051-why-google-style-guide-for-c-is-a-deal-breaker).
구글은 자신들의 코드 베이스를 업데이트 하는데만 바쁘고, 공개한 가이드라인을 그들의 실제코드에 실제로 정확히 어떻게 반영하는지 모르겠다.
Google are busy updating their code base and we don't know how accurately the posted guideline reflects their actual code.
추천하고 있는것들은 계속 업데이트 되어 가고 있다.
This set of recommendations is evolving.
* [JSF++: JOINT STRIKE FIGHTER AIR VEHICLE C++ CODING STANDARDS](http://www.stroustrup.com/JSF-AV-rules.pdf).
Document Number 2RDU00001 Rev C. December 2005.
비행 조정 소프트웨어용.
For flight control software.
혹독한 리얼타임용.
For hard real time.
만일 프로그램이 실패할 경우 누군가가 죽는다던가 하는 상황 처럼, 매우 제한적인 상황일때 적용한다.
This means that it is necessarily very restrictive ("if the program fails somebody dies").
예를들어 메모리 오버플로나 파편화가 허용되지 않는 상황에서 비행기 착륙 후 메모리 할당이나 해제와 같은 상황이다.
For example, no free store allocation or deallocation may occur after the plane takes off (no memory overflow and no fragmentation allowed).
짧은 시간에 익셉션을 핸들링 할 수 있는 도구나 해당 상황에 대한 보장을 할 수 없기 때문에 익셉션은 사용할 수 없다. 
No exception may be used (because there was no available tool for guaranteeing that an exception would be handled within a fixed short time).
라이브러리는 매우 중요한 애플리케이션을 위해 승인된 것만을 사용해야 한다.
Libraries used have to have been approved for mission critical applications.
비야네 스타라우스트롭은 JSF++의 저자이기 때문에 가이드라인들과의 유사성에 그다지 놀랍지 않다.
Any similarities to this set of guidelines are unsurprising because Bjarne Stroustrup was an author of JSF++.
추천할만 하지만, 매우 특별한 상황에서만 임을 명심한다.
Recommended, but note its very specific focus.
* [Mozilla Portability Guide](https://developer.mozilla.org/en-US/docs/Mozilla/C%2B%2B_Portability_Guide).
이름에서 알 수 있듯, 오래되거나 많은 컴파일러에 대한 이식을 목표로 하고 있다.
As the name indicate, this aims for portability across many (old) compilers.
이와같이 한정된다.
As such, it is restrictive.
* [Geosoft.no: C++ Programming Style Guidelines](http://geosoft.no/development/cppstyle.html).
???.
* [Possibility.com: C++ Coding Standard](http://www.possibility.com/Cpp/CppCodingStandard.html).
???.
* [SEI CERT: Secure C++ Coding Standard](https://www.securecoding.cert.org/confluence/pages/viewpage.action?pageId=637).
예제와 예시가 곁들어진 보안에 민감한 코드 묶음들과 규칙의 묶음들은 내용이 매우 좋다.
A very nicely done set of rules (with examples and rationales) done for security-sensitive code.
일반적인 상황에서 규칙의 대부분을 적용할 수 있다.
Many of their rules apply generally.
* [High Integrity C++ Coding Standard](http://www.codingstandard.com/).
* [llvm](http://llvm.org/docs/CodingStandards.html).
Somewhat brief, pre-C++11, and (not unreasonably) adjusted to its domain.
* ???


<a name="SS-books"></a>
## RF.books: 코딩 가이드라인 도서

* Scott Meyers: Effective C++ (???). Addison-Wesley 2014. Beware of overly technical and overly definite rules.
* Sutter and Alexandrescu: C++ Coding Standards. Addison-Wesley 2005. More a set of meta-rules than a set of rules. Pre-C++11. Recommended.
* <a name="BS2005"></a>
Bjarne Stroustrup: [A rationale for semantically enhanced library languages](http://www.stroustrup.com/SELLrationale.pdf).
LCSD05. October 2005.
* Stroustrup: [A Tour of C++](http://www.stroustrup.com/Tour.html).
Addison Wesley 2014.
Each chapter ends with an advice section consisting of a set of recommendations.
* Stroustrup: [The C++ Programming Language (4th Edition)](http://www.stroustrup.com/4th.html).
Addison Wesley 2013.
Each chapter ends with an advice section consisting of a set of recommendations.
* Stroustrup: [Style Guide](http://www.stroustrup.com/Programming/PPP-style.pdf)
for [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html).
Mostly low-level naming and layout rules.
Primarily a teaching tool.


<a name="SS-C++"></a>
## RF.C++: C++ 프로그래밍 (C++11/C++14)

* TC++PL4
* Tour++
* Programming: Principles and Practice using C++


<a name="SS-web"></a>
## RF.web: 웹사이트

* [isocpp.org](http://www.isocpp.com)
* [Bjarne Stroustrup's home pages](http://www.stroustrup.com)
* [WG21](http://www.open-std.org/jtc1/sc22/wg21/)
* [Boost](http://www.boost.org)
* [Adobe open source](http://www.adobe.com/open-source.html)
* [Pogo libraries](http://pocoproject.org/)



<a name="SS-vid"></a>
## RS.video: 모던 C++관련 동영상

* Bjarne Stroustrup: [C++11 Style](http://channel9.msdn.com/Events/GoingNative/GoingNative-2012/Keynote-Bjarne-Stroustrup-Cpp11-Style). 2012.
* Bjarne Stroustrup: [The Essence of C++: With Examples in C++84, C++98, C++11, and C++14](http://channel9.msdn.com/Events/GoingNative/2013/Opening-Keynote-Bjarne-Stroustrup). 2013
* All the talks from [CppCon '14](https://isocpp.org/blog/2014/11/cppcon-videos-c9)
* Bjarne Stroustrup: [The essence of C++](https://www.youtube.com/watch?v=86xWVb4XIyE) at the University of Edinburgh. 2014.
* Sutter: ???
* ??? more ???


<a name="SS-man"></a>
## RF.man: 매뉴얼

* ISO C++ Standard C++11
* ISO C++ Standard C++14
* Palo Alto "Concepts" TR
* ISO C++ Concepts TS
* WG21 Ranges report
 
 
<a name="SS-ack"></a>
## Acknowledgements

규칙, 의견제시 정보 및 레퍼런스 제공 등 도움을 주신 분들에게 감사의 말씀을 전한다.
Thanks to the many people who contributed rules, suggestions, supporting information, references, etc.:

* Peter Juhl
* Neil MacIntosh
* Axel Naumann
* Andrew Pardoe
* Gabriel Dos Reis
* Zhuang, Jiangang (Jeff)
* Sergey Zubkov
