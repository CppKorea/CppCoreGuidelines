
# <a name="S-references"></a>RF: 레퍼런스

C++을 보다 전문적으로 사용하기 위하여 많은 코딩 표준, 규칙들이 만들어 지고 있다.

많은 규칙들이
* 식별자들의 스펠링같이 사소한 사항들에 초점을 맞춘다
* C++에 미숙한 사람들 의해 작성되었다
* "개발자를 올바르게 인도하는"것을 기본 목표로 한다
* 여러 컴파일러들(일부는 10년도 넘은) 간의 이식성을 위한 것이다
* 수십년된 오래된 코드 베이스를 유지하기 위해 작성되었다
* 하나의 분야에만 국한된다
* 명백히 역효과를 낳는다
* 무시된다 (그리고 프로그래머가 자신의 소임을 다하기 위해선 무시되어야만 한다)

잘못된 코딩 표준은 차라리 없느니만 못하다. 그러나 적절한 가이드라인이라는 것은 표준이 없는것 보다 훨씬 낫다: "형식은 자유를 가져 오리라"

우리가 원하는 대로 할 수 있고 원하지 않는 것은 허용되지 않는, 소위 "완벽한 언어"가 될 수 있을까? 기본적으로, 알맞은 언어는 (그리고 언어를 위한 툴 체인을 포함해서) 더 많은 요구사항을 충족시키고 기능을 제공해야 한다. 시간이 흐를수록 언어에 대한 요구사항은 변화하고, 이에 부응하기 위해서는 범용적인 언어가 필요하게 된다. 오늘은 이상적인 언어가 내일은 너무 제한적일 수 있다.

코딩 가이드라인은 특정 요구에 맞게 언어를 사용하는 것이다. 그러므로, 모두를 위한 단일 코딩 스타일은 있을 수 없다. 각 개발조직 별로 제약사항을 추가하거나 확고한 규칙을 더해서 사용해야 한다.

Reference sections:

* [RF.rules: Coding rules](#SS-rules)
* [RF.books: Books with coding guidelines](#SS-books)
* [RF.C++: C++ Programming (C++11/C++14/C++17)](#SS-Cplusplus)
* [RF.web: Websites](#SS-web)
* [RS.video: Videos about "modern C++"](#SS-vid)
* [RF.man: Manuals](#SS-man)
* [RF.core: Core Guidelines materials](#SS-core)

## <a name="SS-rules"></a>RF.rules: 코딩 규칙

* [Boost Library Requirements and Guidelines](http://www.boost.org/development/requirements.html).
  ???.
* [Bloomberg: BDE C++ Coding](https://github.com/bloomberg/bde/wiki/CodingStandards.pdf).
  코드 조직화와 배치에 중점을 두고 있다
* Facebook: ???
* [GCC Coding Conventions](https://gcc.gnu.org/codingconventions.html).
  C++03과 약간 올드 하지만 그럭저럭 괜찮다
* [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html).
  Geared toward C++03 and (also) older code bases. 구글 전문가들은 가이드라인을 개선하기 위해 활발히 협력하고 있고, 그들 또한 권할 수 있는 현대적이고 일반적인 규칙 집합이 될 수 있기를 기대한다
* [JSF++: JOINT STRIKE FIGHTER AIR VEHICLE C++ CODING STANDARDS](http://www.stroustrup.com/JSF-AV-rules.pdf).
  Document Number 2RDU00001 Rev C. December 2005.
  비행 제어 소프트웨어, 제한시간 내에 반응을 보장해야 하는 경우에 사용된다. 이는 매우 제약이 강하다는 것을 의미한다.("프로그램이 실패하면 누군가 죽는다거나").
  예를 들면, 비행기가 이륙한 이후로는 동적 할당이나 해제가 불가능하고 (메모리 오버플로우나 파편화가 허용되지 않는다), (예외가 고정된 짧은 시간 안에 처리된다는 것을 확실히 보장할 방법이 없기 때문에) 예외 기능이 사용되어선 안되는 것을 의미한다. 
  라이브러리는 특수 목적 애플리케이션을 위해 승인을 받아야만 한다.
  비야네 스트로스트롭은 JSF++의 저자이기 때문에 가이드라인들과의 유사성에 그다지 놀랍지 않다. 추천할만 하지만, 매우 특별한 상황에 대한 것임을 명심하라.
* [Mozilla Portability Guide](https://developer.mozilla.org/en-US/docs/Mozilla/C%2B%2B_Portability_Guide).
  이름에서 알 수 있듯, 많은 (오래된) 컴파일러에 대한 이식을 목표로 하고 있다. 따라서 한정적이다.
* [Geosoft.no: C++ Programming Style Guidelines](http://geosoft.no/development/cppstyle.html).
  ???.
* [Possibility.com: C++ Coding Standard](http://www.possibility.com/Cpp/CppCodingStandard.html).
  ???.
* [SEI CERT: Secure C++ Coding Standard](https://www.securecoding.cert.org/confluence/pages/viewpage.action?pageId=637).
  예제와 합리성이 곁들여진 규칙들이다. 보안에 민감한 코드를 위한 것이지만 대부분은 일반적으로 적용될 수 있다.
* [High Integrity C++ Coding Standard](http://www.codingstandard.com/).
* [llvm](http://llvm.org/docs/CodingStandards.html).
  Somewhat brief, pre-C++11, and (not unreasonably) adjusted to its domain.
* ???

## <a name="SS-books"></a>RF.books: 코딩 가이드라인이 있는 문헌

* [Meyers96](#Meyers96) Scott Meyers: *More Effective C++*. Addison-Wesley 1996.
* [Meyers97](#Meyers97) Scott Meyers: *Effective C++, Second Edition*. Addison-Wesley 1997.
* [Meyers01](#Meyers01) Scott Meyers: *Effective STL*. Addison-Wesley 2001.
* [Meyers05](#Meyers05) Scott Meyers: *Effective C++, Third Edition*. Addison-Wesley 2005.
* [Meyers15](#Meyers15) Scott Meyers: *Effective Modern C++*. O'Reilly 2015.
* [SuttAlex05](#SuttAlex05) Sutter and Alexandrescu: *C++ Coding Standards*. Addison-Wesley 2005. More a set of meta-rules than a set of rules. Pre-C++11.
* [Stroustrup05](#Stroustrup05) Bjarne Stroustrup: [A rationale for semantically enhanced library languages](http://www.stroustrup.com/SELLrationale.pdf).
  LCSD05. October 2005.
* [Stroustrup14](#Stroustrup05) Stroustrup: [A Tour of C++](http://www.stroustrup.com/Tour.html).
  Addison Wesley 2014.
  Each chapter ends with an advice section consisting of a set of recommendations.
* [Stroustrup13](#Stroustrup13) Stroustrup: [The C++ Programming Language (4th Edition)](http://www.stroustrup.com/4th.html).
  Addison Wesley 2013.
  Each chapter ends with an advice section consisting of a set of recommendations.
* Stroustrup: [Style Guide](http://www.stroustrup.com/Programming/PPP-style.pdf)
  for [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html).
  Mostly low-level naming and layout rules.
  Primarily a teaching tool.

## <a name="SS-Cplusplus"></a>RF.C++: C++ 프로그래밍 (C++11/C++14)

* [TC++PL4](http://www.stroustrup.com/4th.html):
A thorough description of the C++ language and standard libraries for experienced programmers.
* [Tour++](http://www.stroustrup.com/Tour.html):
An overview of the C++ language and standard libraries for experienced programmers.
* [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html):
A textbook for beginners and relative novices.

## <a name="SS-web"></a>RF.web: 웹사이트

* [isocpp.org](https://isocpp.org)
* [Bjarne Stroustrup's home pages](http://www.stroustrup.com)
* [WG21](http://www.open-std.org/jtc1/sc22/wg21/)
* [Boost](http://www.boost.org)<a name="Boost"></a>
* [Adobe open source](http://www.adobe.com/open-source.html)
* [Poco libraries](http://pocoproject.org/)
* Sutter's Mill?
* ???

## <a name="SS-vid"></a>RS.video: "모던 C++" 동영상

* Bjarne Stroustrup: [C++11 Style](http://channel9.msdn.com/Events/GoingNative/GoingNative-2012/Keynote-Bjarne-Stroustrup-Cpp11-Style). 2012.
* Bjarne Stroustrup: [The Essence of C++: With Examples in C++84, C++98, C++11, and C++14](http://channel9.msdn.com/Events/GoingNative/2013/Opening-Keynote-Bjarne-Stroustrup). 2013
* All the talks from [CppCon '14](https://isocpp.org/blog/2014/11/cppcon-videos-c9)
* Bjarne Stroustrup: [The essence of C++](https://www.youtube.com/watch?v=86xWVb4XIyE) at the University of Edinburgh. 2014.
* Bjarne Stroustrup: [The Evolution of C++ Past, Present and Future](https://www.youtube.com/watch?v=_wzc7a3McOs). CppCon 2016 keynote.
* Bjarne Stroustrup: [Make Simple Tasks Simple!](https://www.youtube.com/watch?v=nesCaocNjtQ). CppCon 2014 keynote.
* Bjarne Stroustrup: [Writing Good C++14](https://www.youtube.com/watch?v=1OEu9C51K2A). CppCon 2015 keynote about the Core Guidelines.
* Herb Sutter: [Writing Good C++14... By Default](https://www.youtube.com/watch?v=hEx5DNLWGgA). CppCon 2015 keynote about the Core Guidelines.
* CppCon 15
* ??? C++ Next
* ??? Meting C++
* ??? more ???

## <a name="SS-man"></a>RF.man: 매뉴얼

* ISO C++ Standard C++11.
* ISO C++ Standard C++14.
* [ISO C++ Standard C++17](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4606.pdf). Committee Draft.
* [Palo Alto "Concepts" TR](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3351.pdf).
* [ISO C++ Concepts TS](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
* [WG21 Ranges report](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf). Draft.


## <a name="SS-core"></a>RF.core: 핵심 가이드라인 자료들

This section contains materials that has been useful for presenting the core guidelines and the ideas behind them:

* [Our documents directory](https://github.com/isocpp/CppCoreGuidelines/tree/master/docs)
* Stroustrup, Sutter, and Dos Reis: [A brief introduction to C++'s model for type- and resource-safety](http://www.stroustrup.com/resource-model.pdf). A paper with lots of examples.
* Sergey Zubkov: [a Core Guidelines talk](https://www.youtube.com/watch?v=DyLwdl_6vmU)
and here are the [slides](http://2017.cppconf.ru/talks/sergey-zubkov). In Russian. 2017.
* Neil MacIntosh: [The Guideline Support Library: One Year Later](https://www.youtube.com/watch?v=_GhNnCuaEjo). CppCon 2016.
* Bjarne Stroustrup: [Writing Good C++14](https://www.youtube.com/watch?v=1OEu9C51K2A). CppCon 2015 keynote.
* Herb Sutter: [Writing Good C++14... By Default](https://www.youtube.com/watch?v=hEx5DNLWGgA). CppCon 2015 keynote.
* Peter Sommerlad: [C++ Core Guidelines - Modernize your C++ Code Base](https://www.youtube.com/watch?v=fQ926v4ZzAM). ACCU 2017.
* Bjarne Stroustrup: [No Littering!](https://www.youtube.com/watch?v=01zI9kV4h8c). Bay Area ACCU 2016.
It gives some idea of the ambition level for the Core Guidelines.

Note that slides for CppCon presentations are available (links with the posted videos).

Contributions to this list would be most welcome.

## <a name="SS-ack"></a> 감사의 말

규칙, 제안, 추가 정보, 참고사항 등 도움을 주신 분들에게 감사의 말씀을 전한다:

* Peter Juhl
* Neil MacIntosh
* Axel Naumann
* Andrew Pardoe
* Gabriel Dos Reis
* Zhuang, Jiangang (Jeff)
* Sergey Zubkov

그리고 Github에 있는 기여자 목록도 함께 봐주기를.
