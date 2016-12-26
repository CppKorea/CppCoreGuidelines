### <a name="Rp-leak"></a> P.8: 리소스를 누락시키지 마라.
>### <a name="Rp-leak"></a> P.8: Don't leak any resources

##### Reason

장시간 실행해야 하는 프로그램에서는 필수적이다. 효율성. 에러를 복구하는 능력.
>Essential for long-running programs. Efficiency. Ability to recover from errors.

##### Example, bad

    void f(char* name)
    {
        FILE* input = fopen(name, "r");
        // ...
        if (something) return;   // bad: if something == true, a file handle is leaked
        // ...
        fclose(input);
    }

[RAII](#Rr-raii)를 선호하라:
>Prefer [RAII](#Rr-raii):

    void f(char* name)
    {
        ifstream input {name};
        // ...
        if (something) return;   // OK: no leak
        // ...
    }

**See also**: [The resource management section](#S-resource)

##### Enforcement

* 포인터를 살펴봐라: 소유자와 비소유자로 구분해라.
  가능하다면 소유자를 표준라이브러리 리소스 핸들로 바꿔라.(위의 예제처럼.)
	또는 [the GSL](#S-gsl)에서 `owner`를 사용하는 것처럼 소유자를 표시하라.
* 생짜 `new`, `delete`를 찾아라.
* 잘 알려진 생짜 포인터를 반환하는 리소스 할당함수를 찾아라.
  (예를 들어 `fopen`, `malloc`, `strdup`)

>* Look at pointers: Classify them into non-owners (the default) and owners.
  Where feasible, replace owners with standard-library resource handles (as in the example above).
  Alternatively, mark an owner as such using `owner` from [the GSL](#S-gsl).
>* Look for naked `new` and `delete`
>* Look for known resource allocating functions returning raw pointers (such as `fopen`, `malloc`, and `strdup`)

### <a name="Rp-waste"></a> P.9: 시간이나 공간을 낭비하지 마라.
>### <a name="Rp-waste"></a> P.9: Don't waste time or space

##### Reason

이것이 C++이다.
>This is C++.

##### Note

개발속도, 안전한 리소스, 테스트 단순화 등의 목표를 수행하기 위해 제대로 소모하는 시간이나 공간은 낭비가 아니다.
>Time and space that you spend well to achieve a goal (e.g., speed of development, resource safety, or simplification of testing) is not wasted.

##### Example

쓸데없는 낭비에 대한 좋은 제안은 언제나 환영 ???
>??? more and better suggestions for gratuitous waste welcome ???

    struct X {
        char ch;
        int i;
        string s;
        char ch2;

        X& operator=(const X& a);
        X(const X&);
    };

    X waste(const char* p)
    {
    	if (p == nullptr) throw Nullptr_error{};
        int n = strlen(p);
        auto buf = new char[n];
        if (buf == nullptr) throw Allocation_error{};
        for (int i = 0; i < n; ++i) buf[i] = p[i];
        // ... manipulate buffer ...
        X x;
        x.ch = 'a';
        x.s = string(n);    // give x.s space for *ps
        for (int i = 0; i < x.s.size(); ++i) x.s[i] = buf[i];	// copy buf into x.s
        delete buf;
        return x;
    }

    void driver()
    {
        X x = waste("Typical argument");
        // ...
    }

예스, 이건 조크이기는 하지만, 실제 코드에서 이런 각각의 실수는 매번 보아 왔다.
`X`의 배치는 적어도 6바이트(다 많을지도 모르지만)의 낭비가 있다는 점을 주목하자.
복사연산을 그럴싸하게 정의해 두다보니 이동 의미가 없어져 버렸다. 그래서 반환 기능이 느려졌다.
`buf`의 `new`, `delete`는 불필요하게 중복적이다.
진짜 지역 문자열을 원했다면 `string` 지역변수를 사용했을 것이다.
몇가지 성능 버그가 있고 쓸데없이 복잡한 것도 문제다.
>Yes, this is a caricature, but we have seen every individual mistake in production code, and worse.
Note that the layout of `X` guarantees that at least 6 bytes (and most likely more) bytes are wasted.
The spurious definition of copy operations disables move semantics so that the return operation is slow.
The use of `new` and `delete` for `buf` is redundant; if we really needed a local string, we should use a local `string`.
There are several more performance bugs and gratuitous complication.

##### Note

낭비에 대한 각각의 예제는 별로 중요하지 않다. 그리고 중요했다면 이미 전문가들이 쉽게 제거했을 것이다.
그러나 코드 전체에 걸쳐 자유롭게 낭비가 퍼져버리면 중요해 질 수 있고,
낭비를 제거하기 위해서 전문가들을 항상 원하는데로 데려다 쓸 수는 없다.
이 규칙(더 세부적인 규칙)의 목적은 큰 낭비가 일어나기 전에 C++을 사용해서 제거하는 것이다.
그런 후에 알고리즘이나 요구사항과 관련된 낭비요소를 찾아볼 수 있다. 그건 이 영역 범위 밖이다.
>An individual example of waste is rarely significant, and where it is significant, it is typically easily eliminated by an expert.
However, waste spread liberally across a code base can easily be significant and experts are not always as available as we would like.
The aim of this rule (and the more specific rules that support it) is to eliminate most waste related to the use of C++ before it happens.
After that, we can look at waste related to algorithms and requirements, but that is beyond the scope of these guidelines.

##### Enforcement

많은 특정한 규칙은 단순한 총괄 목표, 쓸데없는 낭비 제거를 노리고 있다.
>Many more specific rules aim at the overall goals of simplicity and elimination of gratuitous waste.
