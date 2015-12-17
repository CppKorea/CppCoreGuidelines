# 함수
># F: Functions

함수는 동작이나 계산을 명세하는데 이것들은 하나의 상태에서 다른 상태로 일관성 있게 넘어가는 시스템입니다. 이것은 프로그램의 기초적인 설계 단위입니다.
>A function specifies an action or a computation that takes the system from one consistent state to the next. It is the fundamental building block of programs.

함수의 이름은 의미있게 작성이 되어야 합니다. 그래야 인자의 요구사항, 인자간의 관계와 결과를 명확하게 기술할 수 있습니다. 함수의 구현은 사양서가 아닙니다. 함수가 무엇을 해야하는지, 어떻게 동작하는지를 생각해 보세요.
함수는 모든 인터페이스에서 가장 중요한 부분입니다. 인터페이스 규칙을 참고 하세요.
>It should be possible to name a function meaningfully, to specify the requirements of its argument, and clearly state the relationship between the arguments and the result. An implementation is not a specification. Try to think about what a function does as well as about how it does it.
Functions are the most critical part in most interfaces, so see the interface rules.

함수 규칙 정리:
>Function rule summary:

함수 정의 규칙:
>Function definition rules:

* [F.1: "패키지" meaningful operations as carefully named functions](#Rf-package)
* [F.2: 함수는 하나의 동작만 수행해야 한다](#Rf-logical)
* [F.3: 함수를 간결하고 단순하게 유지시켜야 한다](#Rf-single)
* [F.4: 만약 함수가 컴파일 시간에 평가되어야 한다면 `constexpr`로 선언하라](#Rf-constexpr)
* [F.5: 만약 함수가 매우 작고 성능이 중요하다면 인라인으로 선언하라](#Rf-inline)
* [F.6: 만약 함수가 예외를 발생시키지 않는다면 `noexcept`로 선언하라](#Rf-noexcept)
* [F.7: 일반적으로 사용되는 함수라면 스마트 포인터 보다는 `T*`를 인자로 받아라](#Rf-smart)
* [F.8: Prefer pure functions](#Rf-pure)

>* [F.1: "Package" meaningful operations as carefully named functions](#Rf-package)
>* [F.2: A function should perform a single logical operation](#Rf-logical)
>* [F.2: A function should perform a single logical operation](#Rf-logical)
>* [F.3: Keep functions short and simple](#Rf-single)
>* [F.4: If a function may have to be evaluated at compile time, declare it `constexpr`](#Rf-constexpr)
>* [F.5: If a function is very small and time critical, declare it inline](#Rf-inline)
>* [F.6: If your function may not throw, declare it `noexcept`](#Rf-noexcept)
>* [F.7: For general use, take `T*` arguments rather than a smart pointers](#Rf-smart)
>* [F.8: Prefer pure functions](#Rf-pure)

인자 전달 규칙:
>Argument passing rules:

* [F.15: 정보를 전달 할 때 단순하고 전통적인 방식을 선호하라](#Rf-conventional)
* [F.16: 객체 하나를 지정 할 때는 `T*` 나 `owner<T*>` 또는 스마트 포인터를 사용하라](#Rf-ptr)
* [F.17: "null"이 유효하지 않는 값을 의미할 때는 `not_null<T>`를 사용하라](#Rf-nullptr)
* [F.18: 반 개방 범위를 나타날 때는 `array_view<T>` 또는 `array_view_p<T>`를 사용하라](#Rf-range)
* [F.19: C언어 형 문자열을 지정 할 때는 `zstring` 또는 `not_null<zstring>`을 사용하라](#Rf-string)
* [F.20: 크기가 큰 객체를 매개변수로 사용 할 때는 `const T&`를 사용하라](#Rf-const-T-ref)
* [F.21: 크기가 작은 객체를 매개변수로 사용 할 때는 `T`를 사용하라](#Rf-T)
* [F.22: 입출력 매개변수는 `T&`를 사용하라](#Rf-T-re)
* [F.23: 값을 이동하는데 비용이 많이 드는 출력 매개변수는 `T&`를 사용하라 (only)](#Rf-T-return-out)
* [F.24: Use a `TP&&` parameter when forwarding (only)](#Rf-pass-ref-ref)
* [F.25: Use a `T&&` parameter together with `move` for rare optimization opportunities](#Rf-pass-ref-move)
* [F.26: 포인터가 필요한 곳에서 소유권을 이동 시킬 때는 `unique_ptr<T>`를 사용하라](#Rf-unique_ptr)
* [F.27: 소유권을 공유 할 때는 `shared_ptr<T>`을 사용하라](#Rf-shared_ptr)

>* [F.15: Prefer simple and conventional ways of passing information](#Rf-conventional)
>* [F.16: Use `T*` or `owner<T*>` or a smart pointer to designate a single object](#Rf-ptr)
>* [F.17: Use a `not_null<T>` to indicate "null" is not a valid value](#Rf-nullptr)
>* [F.18: Use an `array_view<T>` or an `array_view_p<T>` to designate a half-open sequence](#Rf-range)
>* [F.19: Use a `zstring` or a `not_null<zstring>` to designate a C-style string](#Rf-string)
>* [F.20: Use a `const T&` parameter for a large object](#Rf-const-T-ref)
>* [F.21: Use a `T` parameter for a small object](#Rf-T)
>* [F.22: Use `T&` for an in-out-parameter](#Rf-T-re)
>* [F.23: Use `T&` for an out-parameter that is expensive to move (only)](#Rf-T-return-out)
>* [F.24: Use a `TP&&` parameter when forwarding (only)](#Rf-pass-ref-ref)
>* [F.25: Use a `T&&` parameter together with `move` for rare optimization opportunities](#Rf-pass-ref-move)
>* [F.26: Use a `unique_ptr<T>` to transfer ownership where a pointer is needed](#Rf-unique_ptr)
>* [F.27: Use a `shared_ptr<T>` to share ownership](#Rf-shared_ptr)

값 전달 규칙:
>Value return rules:

* [F.40: Prefer return values to out-parameters](#Rf-T-return)
* [F.41: Prefer to return tuples to multiple out-parameters](#Rf-T-multi)
* [F.42: Return a `T*` to indicate a position (only)](#Rf-return-ptr)
* [F.43: Never (directly or indirectly) return a pointer to a local object](#Rf-dangle)
* [F.44: Return a `T&` when "returning no object" isn't an option](#Rf-return-ref)
* [F.45: Don't return a `T&&`](#Rf-return-ref-ref)

기타 규칙:
>Other function rules:

* [F.50: Use a lambda when a function won't do (to capture local variables, or to write a local function)](#Rf-capture-vs-overload)
* [F.51: Prefer overloading over default arguments for virtual functions](#Rf-default-arg)
* [F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms](#Rf-reference-capture)
* [F.53: Avoid capturing by reference in lambdas that will be used nonlocally, including returned, stored on the heap, or passed to another thread](#Rf-value-capture)

함수는 람다와 함수 객체와 매우 비슷합니다. 다음 섹션을 참조 하세요.
>Functions have strong similarities to lambdas and function objects so see also Section ???.


<a name="SS-fct-def"></a>
## 함수 정의
>## F.def: Function definitions

함수 정의란 함수의 선언과 함수의 몸체인 구현을 말합니다.
>A function definition is a function declaration that also specifies the function's implementation, the function body.


<a name="Rf-package"></a>
### F.1: "패키지" 
>### F.1: "Package" meaningful operations as carefully named functions

**근거**: 공용코드를 분해해 보면 코드를 더 읽기 쉽게 만들고 재사용률을 높이고 복잡한 코드에서 에러를 줄일 수 있습니다.
만약 어떤 동작이 잘 정의되어 있다면 그것을 코드로 분리하고 이름을 지어주세요.
>**Reason**: Factoring out common code makes code more readable, more likely to be reused, and limit errors from complex code.
If something is a well-specified action, separate it out from its  surrounding code and give it a name.

**예, 하지 말 것**:
>**Example, don't**:

	void read_and_print(istream& is)	// read and print and int
	{
		int x;
		if (is>>x)
			cout << "the int is " << x << '\n';
		else
			cerr << "no int on input\n";
	}


`read_and_print`는 거의 모든것이 잘못 되어 있습니다.
이 함수는 읽고, (고정된 `ostream`에) 씁니다. 이 함수는 에러 메시지를 (고정된 `ostream`에) 쓰고 `int`형만을 다룹니다.
재사용도 없고, 논리적으로 구분 될 수 있는 동작들은 뒤섞여 있고 지역변수는 사용이 끝난 후에도 논리적 범위에 남아 있습니다.
작은 예를 들어보면, 이것은 문제가 없어 보입니다. 하지만 입력을 처리하고 출력을 처리하고, 에러를 처리해야 한다면 더 복잡해 집니다.
>Almost everything is wrong with `read_and_print`.
>It reads, it writes (to a fixed `ostream`), it write error messages (to a fixed `ostream`), it handles only `int`s.
>There is nothing to reuse, logically separate operations are intermingled and local variables are in scope after the end of their logical use.
>For a tiny example, this looks OK, but if the input opeartion, the output operation, and the error handling had been more complicated the tangled 
mess could become hard to understand.

**참고 사항**: 한군데 이상에서 사용되는 사소하지 않은 람다를 작성한다면 함수에 이름을 짓고 (대부분 로컬이 아닌)변수에 할당하세요.
>**Note**: If you write a non-trivial lambda that potentially can be used in more than one place, give it a name by assigning it to a (usually non-local) variable.

**예**:
>**Example**:

	sort(a, b, [](T x, T y) { return x.valid() && y.valid() && x.value()<y.value(); });

람다에 이름을 짓게되면 표현식을 여러개의 논리적 부분으로 나눌 수 있고, 람다가 어떤 일을 하는지 가늠하게 해줍니다.
>Naming that lambda breaks up the expression into its logical parts and provides a strong hint to the meaning of the lambda.

	auto lessT = [](T x, T y) { return x.valid() && y.valid() && x.value()<y.value(); };
	
	sort(a, b, lessT);
	find_if(a,b, lessT);

유지보수나 성능을 고려하다면 짧은 코드가 항상 좋은것은 아닙니다.
>The shortest code is not always the best for performance or maintainability.

**예외 사항**: 반복문, 람다를 반복문으로 사용하는 경우는 거의 이름을 지어줄 필요가 없습니다.
그러나 수십줄에서 수십페이지가 되는 큰 반복문에는 문제가 있습니다.
[함수를 간결하게 유지하라](#Rf-single)는 규칙은 "반복문을 간결하게 유지하라"를 의미합니다.
비슷한 경우로, 콜백함수 인자로 사용되는 람다가 중요한 경우도 있습니다. 물론 재사용 될지는 알 수 없습니다.
>**Exception**: Loop bodies, including lambdas used as loop bodies, rarely needs to be named.
>However, large loop bodies (e.g., dozens of lines or dozens of pages) can be a problem.
>The rule [Keep functions short](#Rf-single) implies "Keep loop bodies short."
>Similarly, lambdas used as callback arguments are sometimes non-trivial, yet unlikely to be re-usable.

**시행하기**:
>**Enforcement**:

* [함수를 간결하게 유지하라](#Rf-single)를 참조하세요.
* 여러곳에서 사용되는 동일하거나 매우 비슷한 람다는 표시해 두세요.

>* See [Keep functions short](#Rf-single)
>* Flag identical and very similar lambdas used in different places.


<a name="Rf-logical"></a>
### F.2: 함수는 하나의 논리적 수행만 행해야 한다
>### F.2: A function should perform a single logical operation

**근거**: 하나의 논리적 수행만 행하는 함수는 이해하기 쉽고, 테스트하기가 쉽고, 재사용이 쉽습니다. 
>**Reason**: A function that performs a single operation is simpler to understand, test, and reuse.

**예**: 다음을 주목 하세요.
>**Example**: Consider

	void read_and_print()	// bad
	{
		int x;
		cin >> x;
		// 에러를 검사한다.
		cout << x << "\n";
	}

이 함수는 특정한 입력에 속박되어 있고 다른 쓰임세는 찾아 볼 수 없습니다. 대신에 함수를 의미있는 논리적 부분들로 나누고 매개변수화 하세요: 
>This is a monolith that is tied to a specific input and will never find a another (different) use. Instead, break functions up into suitable logical parts and parameterize:

	int read(istream& is)	// better
	{
		int x;
		is >> x;
		// 에러를 검사한다.
		return x;
	}

	void print(ostream& os, int x)
	{
		os << x << "\n";
	}

필요한 곳에서 두 함수를 결합할 수 있습니다:
>These can now be combined where needed:

	void read_and_print()
	{
		auto x = read(cin);
		print(cout, x);
	}

만약 요청이 있었다면, `read()`와 `print()`에서 사용하는 데이터형과 입력 메커니즘 등을 템플릿화 할 수 있다. 예:  
>If there was a need, we could further templatize `read()` and `print()` on the data type, the I/O mechanism, etc. Example:

	auto read = [](auto& input, auto& value)	// 더 나은 방법
	{
		input >> value;
		// 에러를 검사한다.
	}

	auto print(auto& output, const auto& value)
	{
		output << value << "\n";
	}

**시행하기**
>**Enforcement**:

* 함수가 두개 이상의 출력 매개변수를 가진다면 의심하세요. 여러개의 반환값을 저장 할 수 있는 `tuple`과 같은 것을 반환값으로 사용하세요. 
* 편집기의 한 화면에 다 나오지 않을 만큼 큰 함수는 의심하세요. 이런 함수는 이름을 잘 지어주고 더 작은 세부동작으로 나누세요.
* 7개 이상의 매개변수를 가진 함수는 의심하세요.

>* Consider functions with more than one "out" parameter suspicious. Use return values instead, including `tuple` for multiple return values.
>* Consider "large" functions that don't fit on one editor screen suspicious. Consider factoring such a function into smaller well-named suboperations.
>* Consider functions with 7 or more parameters suspicious.


<a name="Rf-single"></a>
### F.3: 함수를 단순하고 간결하게 유지하라
>### F.3: Keep functions short and simple

**근거**: 긴 함수는 읽기 어렵고 복잡하고, 변수는 최소범위를 넘어서서 사용되고 있을지 모른다. 복잡한 제어구조를 가진 함수는 더 길고 논리적 오류가 숨겨져 있을지도 모른다. 
>**Reason**: Large functions are hard to read, more likely to contain complex code, and more likely to have variables in larger than minimal scopes.
Functions with complex control stryuctures are more likely to be long and more likely to hide logical errors

**예**:
>**Example**: Consider

	double simpleFunc(double val, int flag1, int flag2)
		// simpleFunc: takes a value and calculates the expected ASIC output, given the two mode flags.
	{
  		double intermediate;
  		if (flag1 > 0) {
    		intermediate = func1(val);
    		if (flag2 % 2)
      			intermediate = sqrt(intermediate);
  		}
  		else if (flag1 == -1) {
    		intermediate = func1(-val);
    		if (flag2 % 2)
      			intermediate = sqrt(-intermediate);
    		flag1 = -flag1;
  		}
  		if (abs(flag2) > 10) {
   			intermediate = func2(intermediate);
  		}
  		switch (flag2 / 10) {
    	case 1: if (flag1 == -1) return finalize(intermediate, 1.171); break;
    	case 2: return finalize(intermediate, 13.1);
    	default: ;
  		}
  		return finalize(intermediate, 0.);
	}

이 함수는 너무 복잡하다 (그리고 너무 길다).
어떻게 모든 경우가 올바르게 처리되는지 알 수 있을까요?
게다가 이 함수는 다른 규칙들도 어기고 있습니다.
>This is too complex (and also pretty long).
How would you know if all possible alternatives have been correctly handled?
Yes, it break other rules also.

이렇게 바꿔 볼 수 있습니다:
>We can refactor:

	double func1_muon(double val, int flag)
	{
		// ???
	}
	
	double funct1_tau(double val, int flag1, int flag2)
	{
		// ???
	}
	
	double simpleFunc(double val, int flag1, int flag2)
		// simpleFunc: takes a value and calculates the expected ASIC output, given the two mode flags.
	{
  		if (flag1 > 0)
 			return func1_muon(val, flag2);
		if (flag1 == -1)
    		return func1_tau(-val, flag2);	// handled by func1_tau: flag1 = -flag1;
  		return 0.;
	}

**참고 사항**: "한 화면에 맞추기"는 "너무 크게 하지 않기"를 막는 좋은 실용적인 규칙이 되기도 합니다.
왠만하면 최대 다섯줄짜리 함수로 구현 할 수 있을지 고민해 봐야 합니다.
>**Note**: "It doesn't fit on a screen" is often a good practical definition of "far too large."
One-to-five-line functions should be considered normal.

**참고 사항**: 긴 함수는 응집성있고 의미있는 이름을 가진 작은 함수로 나누세요. 작고 간결한 함수는 함수 호출 비용이 중요한 곳에서 내제화되어 사용될 수 있습니다.
>**Note**: Break large functions up into smaller cohesive and named functions.
Small simple functions are easily inlined where the cost of a function call is significant.

**시행하기**:
>**Enforcement**:

* "한 화면에 맞지 않는" 함수는 표시해 두세요.
한 화면은 얼마나 클까요? 한 줄에 140자, 총 60줄에 맞추도록 노력하세요. 이 정도면 대략 책 한 페이지정도 되는데 보기 좋은 분량이 됩니다.
* 유달리 복잡해보이는 함수는 표시해 두세요. 얼마나 복잡해야 너무 복잡한 걸까요?
복잡성 지표를 사용하세요. Try "more that 10 logical path through."
Count a simple switch as one path. 

>* Flag functions that do not "fit on a screen."
How big is a screen? Try 60 lines by 140 characters; that's roughly the maximum that's comfortable for a book page.
>* Flag functions that are too complex. How complex is too complex?
You could use cyclomatic complexity. Try "more that 10 logical path through." Count a simple switch as one path.


<a name="Rf-constexpr"></a>
### F.4: 함수가 컴파일 타임에 평가되어야 한다면  `constexpr`로 선언하라
>### F.4: If a function may have to be evaluated at compile time, declare it `constexpr`

**근거**: `constexpr`는 컴파일러에게 컴파일 타임에 평가하라고 지시하는데 사용됩니다.
>**Reason**: `constexpr` is needed to tell the compiler to allow compile-time evaluation.

**예**: 유명한 팩토리얼:
>**Example**: The (in)famous factorial:

	constexpr int fac(int n)
	{
		constexpr int max_exp = 17;     // constexpr enables  this to be used in Expects
		Expects(0<=x && x<max_exp);		// prevent silliness and overflow
		int x = 1;
		for (int i=2; i<=n; ++i) x*= n;
		return x;
	}

이 예는  C++14 형태 입니다. C++11 형태를 사용하려면 	`fac()`의  functional formulation를 사용하세요.
>This is C++14. For C++11, use a functional formulation of `fac()`.

**참고 사항**: `constexpr`은 컴파일 타임 평가를 보장하지 않습니다;
이것은 프로그래머가 요구하거나 컴파일러가 최적화를 하기로 결정했을 때 상수 표현 인자에 대해서 컴파일 타임에 평가 될 수 있다는 것을 보장 할 뿐입니다.
>**Note**: `constexpr` does not guarantee compile-time evaluation;
it just guarantees that the function can be evaluated at compile time for constant expression arguments if the programmer requires it or the compiler decides to do so to optimize.

	constexpr int min(int x, int y) { return x<y?x:y;}
	
	void test(int v)
	{
		int m1 = min(-1,2);				// probably compile-time evaluation
		constexpr int m2 = min(-1,2);	// compile-time evaluation
		int m3 = min(-1,v);				// run-time evaluation
		constexpr int m4 = min(-1,2);	// error: connot evaluate at compile-time
	}
	
**참고 사항**: `constexpr` 함수는 결점이 없습니다: 부작용을 가질리 없습니다.
>**Note**: `constexpr` functions are pure: they can have no sideefects.

	int dcount = 0;
	constexpr int double(int v)
	{
		++dcount;		// error: attempted side effect from constexpr function
		return v+v;
	}

대체적으로 매우 유용 합니다.
>This is usually a very good thing.

**참고 사항**: 모든 함수를 `constexpr`로 만들지 마세요. 대부분의 계산은 런타임에 수행되는 것이 좋습니다.
>**Note**: Don't try to make all functions `constexpr`. Most computation is best done at run time.

**시행하기**: 불가능하고 불필요한 것.
만약 상수를 필요로하는 곳에서 `constexpr`로 선언되지 않은 함수가 호출 된다면 컴파일러는 에러를 출력합니다.
>**Enforcement**: Imposible and unnecessary.
The compiler gives an error if a non-`constexpr` function is called where a constant is required.


<a name="Rf-inline"></a>
### F.5: 만약 함수가 매우 짧고 수행시간이 중요하다면 `inline`으로 선언하라
>### F.5: If a function is very small and time critical, declare it `inline`

**근거**: 일부 최적화기는 프로그래머로부터 힌트를 받지 않아도 함수 인라인화를 잘 해내지만 그것에 의존하지는 마세요.
지난 40년간 우리는 컴파일러가 아무런 힌트가 없어도 사람보다 더 인라인화를 잘 할거라고 약속해 왔습니다.
우리는 계속 그 약속이 이뤄지길 기다리고 있습니다.
`inline`키워드를 지정하는 것은 컴파일러가 일을 더 잘 할 수 있도록 독려해주는 것 입니다.
>**Reason**: Some optimizers are good an inlining without hints from the programmer, but don't rely on it.
Measure! Over the last 40 years or so, we have been promised compilers that can inline better than humans without hints from humans.
We are still waiting.
Specifying `inline` encourages the compiler to do a better job.

**Exception**: Do not put an `inline` function in what is meant to be a stable interface unless you are really sure that it will not change.
An inline function is part of the ABI.

**참고 사항**: `constexpr`은 `inline`을 내포하고 합니다.
>**Note**: `constexpr` implies `inline`.

**참고 사항**: 클래스에 정의된 멤버 함수는 기본으로 인라인화 됩니다.
>**Note**: Member functions defined in-class are `inline` by default.

**예외 사항**: 템플릿 함수(템플릿 멤버 함수 포함)는 헤더에 정의 되어야 합니다. 따라서 인라인 함수가 됩니다.
>**Exception**: Template functions (incl. template member functions) must be in headers and therefore inline.

**수행하기**: `inline`함수가 3줄이상 된다면 표시해 두세요. (추가번역 필요)
>**Enforcement**: Flag `inline` functions that are more than three statements and could have been declared out of line (such as class member functions).
To fix: Declare the function out of line. [[NM: Certainly possible, but size-based metrics can be very annoying.]]


<a name="Rf-noexcept"></a>
### F.6: 만약 함수가 예외를 던지지 않는다면 `noexcept`로 선언하라.
>### F.6: If your function may not throw, declare it `noexcept`

**근거**: 예외를 던지지 않기로 계획 했다면, 프로그램은 오류를 처리하지 않을 것이기 때문에 가능하면 빨리 종료되어야 합니다. `noexcept`를 선언하면 최적화기가 선택하는 여러가지 대안들을 사전에 제거할 수 있도록 도와주게 됩니다.
>**Reason**: If an exception is not supposed to be thrown, the program cannot be assumed to cope with the error and should be terminated as soon as possible. Declaring a function `noexcept` helps optimizers by reducing the number of alternative execution paths. It also speeds up the exit after failure.

**예**: C언어로 구현이 되었거나 예외를 지원하지 않는 모든 함수에 `noexcept`를 추가하세요.
C++ 표준 라이브러리는 C 표준 라이브러리에 대해서 암시적으로 그렇게하고 있습니다.
>**Example**: Put `noexcept` on every function written completely in C or in any other language without exceptions.
The C++ standard library does that implicitly for all functions in the C standard library.

**참고 사항**: `constexpr` 함수는 예외를 던질 수 없기 때문에 `noexcept`를 사용할 필요가 없습니다.
>**Note**: `constexpr` functions cannot throw, so you don't need to use `noexcept` for those.

**예**: `noexcept`는 예외를 던질 수 있는 함수에서 사용할 수 있습니다.
>**Example**: You can use `noexcept` even on functions that can throw:

	vector<string> collect(istream& is) noexcept
	{
		vector<string> res;
		for(string s; is>>s; )
			res.push_back(s);
		return res;
	}

`collect()` 함수가 메모리를 모두 사용해 버리면 프로그램은 비정상적인 종료가 됩니다.
프로그램이 메모리 소진상태를 만들지 않기위해서 개선되지 않는다면 다음의 것을 할 수 밖에 없을 것입니다;
`terminate()`가 적절한 에러 로그 정보를 만들어 낼 것 입니다 (그러나 메모리가 이미 다 소진된 상태라면 현명한 무언가를 하기가 어려울 것 입니다).
>If `collect()` runs out of memory, the program crashes.
Unless the program is crafted to survive memory exhaustion, that may be just the right thing to do;
`terminate()` may generate suitable error log information (but after memory runs out it is hard to do anything clever).

**참고 사항**: 대부분의 프로그램에서, 대부분의 함수는 예외를 던질 수 있습니다
(예외를 던져서 실패상황을 보고하는 함수나 라이브러리를 사용하세요).
다만 모든 곳에 `noexcept` 덫을 놓지는 마세요.
`noexcept`는 저수준 함수에서 빈번하게 사용하기에 가장 유용합니다. 
>**Note**: In most programs, most functions can throw
(e.g., because they use `new`, call functions that do, or use library functions that reports failure by throwing),
so don't just springle `noexcept` all over the place.
`noexcept` is most useful for frequently used, low-level functions.

**참고 사항**: 소멸자, `swap` 함수, move 연산자 그리고 기본 생성자에서는 절대로 예를 던지면 안 됩니다.
>**Note**: Destructors, `swap` functions, move operations, and default constructors should never throw.


**시행하기**:
>**Enforcement**:

* 아직까지 예외를 던질 수 없는 함수가 있다면 표시해 두세요.
* 예외를 던지는 `swap`, move 연산자, 소멸자 그리고 기본 생성자가 있다면 표시해 두세요.

>* Flag functions that are not `noexcept`, yet cannot thow
* Flag throwing `swap`, `move`, destructors, and default constructors.


<a name="Rf-smart"></a>
### F.7: 범용으로 사용하려면 스마트포인터 대신에 `T*`형 인자를 사용하라.
>### F.7: For general use, take `T*` arguments rather than a smart pointers

**근거**: 스마트 포인터를 인자로 사용하면 소유권이 이전되거나 공유 됩니다.
스마트 포인터를 인자로 사용하면 함수 호출 시 스마트 포인터를 사용해야한다는 제약이 생깁니다.
공유 스마트 포인터를 인자로 사용하는 것은 (예, `std::shared_ptr`) 런타임시 추가 비용을 부담하게 됩니다.
>**Reason**: Passing a smart pointer transfers or shares ownership.
Passing by smart pointer restricts the use of a function to callers that use smart pointers.
Passing a shared smart pointer (e.g., `std::shared_ptr`) implies a run-time cost.

**예**:
>**Example**:

	void f(int*);		// accepts any int*
	void g(unique_ptr<int>);	// can only accept ints for which you want to transfer ownership
	void g(shared_ptr<int>);	// can only accept ints for which you are willing to share ownership

**참고 사항**: 우리는 정적분석으로 댕글링포인터 문제를 잡아 낼 수 있습니다. 그래서 굳이 댕글링포인터 문제를 피하기위해서 자원관리를 할 필요는 없습니다.
>**Note**: We can catch dangling pointers statically, so we don't need to rely on resource management to avoid violations from dangling pointers.

**더 보기**: [smart pointer use](#Rr-summary-smartptrs)에 대한 논의.
>**See also**: Discussion of [smart pointer use](#Rr-summary-smartptrs).

**시행하기**: 스마트 포인터 인자가 있다면 표시해 두세요.
>**Enforcement**: Flag smart pointer arguments.


<a name="Rf-pure"></a>
### F.8: 간결한 함수를 선호하라
>### F.8: Prefer pure functions

**근거**: 간결한 함수는 이유를 이해하기 쉽고, 최적화하기 쉽고(병렬화되어 있더라도), 메모이제이션하기 쉽습니다.
>**Reason**: Pure functions are easier to reason about, sometimes easier to optimize (and even parallelize), and sometimes can be memoized.

**예**:
>**Example**:

    template<class T>
	auto square(T t) { return t*t; }
	
**참고 사항**: `constexpr`는 간결한 함수 입니다.
>**Note**: `constexpr` functions are pure.
	
**시행하기**: 할게 없습니다(간결하기 때문에).
>**Enforcement**: not possible.


<a name="SS-call"></a>
## F.call: 인자전달
>## F.call: Argument passing

함수에 인자를 전달하고 값을 반환하는 다양한 방법이 있습니다.
>There are a variety of ways to pass arguments to a function and to return values.


<a name="Rf-conventional"></a>
### 규칙 F.15: 정보를 전달 할 때 단순하고 관습적인 방법을 선호하라
>### Rule F.15: Prefer simple and conventional ways of passing information

**근거**: "별나면서 교묘한" 기법은 깜짝놀랄만한 버그를 만들어내거나, 다른 프로그래머가 코드를 이해하는데 어렵게 만든다. 
정말로 일반적인 기법을 넘어서는 방법으로 최적화를 해야 한다면 꼭 필요한 개선사항이라는것을 확신할 수 있어야하고, 이식성이 없을 수 있기 때문에 문서나 주석을 남겨야 한다.
>**Reason**: Using "unusual and clever" techniques causes surprises, slows understanding by other programmers, and encourages bugs.
If you really feel the need for an optimization beyond the common techniques, measure to ensure that it really is an improvement,
and document/comment because the improvement may not be portable.

![Normal parameter passing table](./param-passing-normal.png "Normal parameter passing")

**"결과값"에 대해서**: 출력 매개변수보다는 반환값을 더 선호하라.
메모리 관리를 피하고 성능을 향상시키기 위해 move 연산자를 사용하는 표준 컨테이너와 같이 크기가 큰 객체에도 해당된다. 
다수의 값을 반환해야 한다면 [튜플 사용하기](#Rf-T-multi)  또는 다수의 멤버를 같은 데이터형을 사용하세요.
>**For an "output-only" value:** Prefer return values to output parameters.
This includes large objects like standard containers that use implicit move operations for performance and to avoid explicit memory management.
If you have multiple values to return, [use a tuple](#Rf-T-multi) or similar multi-member type.

**예**:
>**Example**:

	vector<const int*> find_all(const vector<int>&, int x);	// return pointers to elements with the value x
	
**나쁜 예**:
>**Example, bad**:

	void find_all(const vector<int>&, vector<const int*>& out, int x);	// place pointers to elements with value x in out

**예외 사항**:
>**Exceptions**:

* 예를 들어 상속 관계에서 사용되는 타입처럼 값이 아닌 타입은 `unique_ptr`또는 `shared_ptr`로 객체를 반환하세요. 
* move 사용하기에 비용이 많이 든다면 (예를들어, `array<BigPOD>`), 메모리를 할당하고 핸들을 반환하는 방법을 고려하거나 출력 매개변수로써 비상수 참조형을 사용하세요. 

>* For non-value types, such as types in an inheritance hierarchy, return the object by `unique_ptr` or `shared_ptr`.
* If a type is expensive to move (e.g., `array<BigPOD>`), consider allocating it on the free store and return a handle (e.g., `unique_ptr`), or passing it in a non-`const` reference to a target object to fill (to be used as an out-parameter).
* In the special case of allowing a caller to reuse an object that carries capacity (e.g., `std::string`, `std::vector`) across multiple calls to the function in an inner loop, treat it as an in/out parameter instead and pass by `&`. This one use of the more generally named "caller-allocated out" pattern.

**입출력 매개변수에 관하여:** 비상수 참조형으로 전달하세요. 이것은 호출자에게 객체가 수정될 것이라고 암시해주는 역활을 합니다.
>**For an "in-out" parameter:** Pass by non-`const` reference. This makes it clear to callers that the object is assumed to be modified.

**값으로 입력 매개변수를 사용하는 것에 관하여:** 값으로 복사, 전달하는데 비용이 적게 든다면 그렇게 하세요. 하지만 그외에는 `const&`를 사용하세요. 이것은 함수가 매개변수를 변경하지 않는다는 것을 암시하고 rvalue로 초기화 할 수 있게 해줍니다. 
"저렴한 복사 비용"이라는 것은 하드웨어 아키텍쳐에 따라 다릅니다.그러나 4바이트 또는 8바이트 (doubles, pointers, refereces)는 값으로 전달하는 것이 좋은 경우가 많습니다.
특히, 값으로 전달되는 객체는 함수에서 접근하는데 추가적인 작업이 필요없습니다. 
>**For an "input-only" value:** If the object is cheap to copy, pass by value.
Otherwise, pass by `const&`. It is useful to know that a function does not mutate an argument, and both allow initialization by rvalues.
What is "cheap to copy" depends on the machine architecture, but two or three words (doubles, pointers, references) are usually best passed by value.
In particular, an object passed by value does not require an extra reference to access from the function.

![Advanced parameter passing table](./param-passing-advanced.png "Advanced parameter passing")

For advanced uses (only), where you really need to optimize for rvalues passed to "input-only" parameters:

* If the function is going to unconditionally move from the argument, take it by `&&`.
* If the function is going to keep a copy of the argument, in addition to passing by `const&` add an overload that passes the parameter by `&&` and in the body `std::move`s it to its destination. (See [F.25](#Rf-pass-ref-move).)
* In special cases, such as multiple "input + copy" parameters, consider using perfect forwarding. (See [F.24](#Rf-pass-ref-ref).)

**Example**:

		int multiply(int, int); // just input ints, pass by value
 
		string& concatenate(string&, const string& suffix); // suffix is input-only but not as cheap as an int, pass by const&
		
		void sink(unique_ptr<widget>);	// input only, and consumes the widget
 
Avoid "esoteric techniques" such as:

* Passing arguments as `T&&` "for efficiency". Most rumors about performance advantages from passing by `&&` are false or brittle (but see [F.25](#Rf-pass-ref-move).)
* Returning `const T&` from assignments and similar operations.

**Example**: Assuming that `Matrix` has move operations (possibly by keeping its elements in a `std::vector`.

	Matrix operator+(const Matrix& a, const Matrix& b)
	{
		Matrix res;
		// ... fill res with the sum ...
		return res;
	}

	Matrix x = m1+m2;	// move constructor

	y = m3+m3;			// move assignment

**Note**: The (optional) return value optimization doesn't handle the assignment case.

**See also**: [implicit arguments](#Ri-explicit).

**Enforcement**: This is a philosophical guideline that is infeasible to check directly and completely.
However, many of the the detailed rules (F.16-F.45) can be checked,
such as passing a `const int&`, returning an `array<BigPOD>` by value, and returning a pointer to fre store alloced by the function.


<a name="Rf-ptr"></a>
### F.16: 객체 하나를 가리킬 때는 `T*`또는 `owner<T*>`를 사용하라 
>### F.16: Use `T*` or `owner<T*>` to designate a single object

**근거**: 전통적인 C, C++에서는 많은 경우에  단순히 `T*`가 사용되었다. 예를 들어,

>**Reason**: In traditional C and C++ code, "Plain `T*` is used for many weakly-related purposes, such as

* (함수 내에서 삭제되지 않는) 객체 한개를 식별한다
* (나중에 해제 될)동적할당 된 객체를 가리킨다
* `nullptr`를 유지한다
* (널문자로 끝나는 문자들) C언어 형태의 문자열을 식별한다
* 길이와 함께 배열을 식별한다
* 배열의 위치를 식별한다

>* Identify a (single) object (not to be deleted by this function)
* Point to an object allocated on the free store (and delete it later)
* Hold the `nullptr`
* Identify a C-style string (zero-terminated array of characters)
* Identify an array with a length specified separately
* Identify a location in an array

`T*`가 어떤 의미를 갖는지 애매한 경우 심각한 오류를 유발시킨다. 그래서 포인터를 사용된 의미에 맞게 구분하면 코드의 의미를 더 명확하게 할 수 있다. 디버깅에서  `owner<T*>` 와 `not_null<T>`는 좋은 도구로 사용된다. 예를들어, `not_null<T*>`는 (기계든 사람이든) 코드를 읽은 주체에게 역참조하기전에 `nullptr`인지 확인하는 것이 필요없다는걸 알려준다.
 
>Confusion about what meaning a `T*` is the source of many serious errors, so using separate names for pointers of these separate uses makes code clearer.
For debugging, `owner<T*>` and `not_null<T>` can be instrumented to check.
For example, `not_null<T*>` makes it obvious to a reader (human or machine) that a test for `nullptr` is not necessary before dereference.

**예**:
>**Example**: Consider

	int length(Record* p);

`length(r)`을 호출 할 때 `r==nullptr`을 검사해야 합니까? 아니면 `length()`를 구현할 때 `p==nullptr`을 검사해야 합니까?

>When I call `length(r)` should I test for `r==nullptr` first? Should the implementation of `length()` test for `p==nullptr`?

	int length(not_null<Record*> p);	// it is the caller's job to make sure p!=nullptr

	int length(Record* p);	// the implementor of length() must assume that p==nullptr is possible

**참고 사항**: `not_null<T>`은  `nullptr`은 아닐 거라고 가정을 합니다.; `T*`는 `nullptr`이 될 수도 있습니다; 둘 다 `T*`를 표현합니다. (그래서 실행시간에서의 오버헤드는 없습니다).
>**Note**: A `not_null<T>` is assumed not to be the `nullptr`; a `T*` may be the `nullptr`; both can be represented in memory as a `T*` (so no run-time overhead is implied).

**참고 사항**: `owner<T*>`는 소유권을 표현합니다. 
>**Note**: `owner<T*>` represents ownership.

**추가 사항**: `T*`는 하나의 요소를 가리키는 (unique_ptr<`T`>와 같은) 스마트 포인터로 부터 얻어진 `T`라고 가정합니다.
>**Also**: Assume that a `T*` obtained from a smart pointer to `T` (e.g., unique_ptr<`T`>) pointes to a single element.

**더 보기**:  [Support library](#S-support).
>**See also**: [Support library](#S-support).

**수행하기**:
>**Enforcement**:

* (단순) ((범위)) 포인터로 산술연산을 하면 포인터 변수의 타입의 값으로 연산이 되는 것에 주의하세요. 

>* (Simple) ((Bounds)) Warn for any arithmetic operation on an expression of pointer type that results in a value of pointer type.


<a name="Rf-nullptr"></a>
### F.17: "널"이 유효하지 않은 값을 의미한다면 `not_null<T>`을 사용하세요 
>### F.17: Use a `not_null<T>` to indicate that "null" is not a valid value

**근거**: 명확성. 널인지 확인 할 필요가 없다는 것을 명확히 해 줌.
>**Reason**: Clarity. Making it clear that a test for null isn't needed.

**예제**:
>**Example**:

	not_null<T*> check(T* p) { if (p) return not_null<T*>{p}; throw Unexpected_nullptr{}; }

	void computer(not_null<array_view<int>> p)
	{
		if (0<p.size()) {	// bad: redundant test
			// ...
		}
	}

**참고 사항**: `not_null`은 내장형 포인터 타입에만 사용되는 것은 아닙니다. `array_view`, `string_view`, `unique_ptr`, `shared_ptr`, 그리고 다른 포인터 형에도 사용 됩니다.
>**Note**: `not_null` is not just for built-in pointers. It works for `array_view`, `string_view`, `unique_ptr`, `shared_ptr`, and other pointer-like types.

**수행하기**:
>**Enforcement**:

* (단순) 함수 내에서 `nullptr`를 검사하지 않고 포인터를 역참조 한다면 경고하세요. `not_null`로 선언하도록 제안하세요.
* (단순) 함수 내에서 포인터가 역참조 될 때 `nullptr`를 검사할 때도 있고 검사하지 않을 때도 있다면 잘못되었다고 말하세요.
* (단순) 함수 내에서 `not_null` 포인터를 `nullptr`인지 검사한다면 경고하세요. 

>* (Simple) Warn if a raw pointer is dereferenced without being tested against `nullptr` (or equivalent) within a function, suggest it is declared `not_null` instead.
* (Simple) Error if a raw pointer is sometimes dereferenced after first being tested against `nullptr` (or equivalent) within the function and sometimes is not.
* (Simple) Warn if a `not_null` pointer is tested against `nullptr` within a function.


<a name="Rf-range"></a>
### F.18: Use an `array_view<T>` or an `array_view_p<T>` to designate a half-open sequence

**Reason**: Informal/non-explicit ranges are a source of errors

**Example**:

	X* find(array_view<X> r, const X& v)	// find v in r

	vector<X> vec;
	// ...
	auto p = find({vec.begin(),vec.end()},X{});	// find X{} in vec

**Note**: Ranges are extremely common in C++ code. Typically, they are implicit and their correct use is very hard to ensure. In particular, given a pair of arguments `(p,n)` designating an array [`p`:`p+n`), it is in general impossible to know if there really are n elements to access following `*p`. `array_view<T>` and `array_view_p<T>` are simple helper classes designating a [p:q) range and a range starting with p and ending with the first element for which a predicate is true, respectively.

**Note**: an `array_view<T>` object does not own its elements and is so small that it can be passed by value.

**Note**: Passing an `array_view` object as an argument is exactly as efficient as passing a pair of pointer arguments or passing a pointer and an integer count.

**See also**: [Support library](#S-support).

**Enforcement**: (Complex) Warn where accesses to pointer parameters are bounded by other parameters that are integral types and suggest they could use `array_view` instead.


<a name="Rf-string"></a>
### F.19: C언어 형식의 문자열을 가리킬 때는 `zstring`또는 `not_null<zstring>`을 사용하라
>### F.19: Use a `zstring` or a `not_null<zstring>` to designate a C-style string

**근거**: C언어 형식의 문자열은 어디에나 있습니다.
C언어 형식의 문자열은 관습에 따라 정의 됩니다: 0으로 끝나는 문자 배열의 집합.
함수들은 `nullptr`을 사용하는데 있어서 일관성이 없는데 이것들을 명백하게 할 필요가 있습니다.
>**Reason**: C-style strings are ubiquitous.
They are defined by convention: zero-terminated arrays of characters.
Functions are inconsistent in their use of `nullptr` and we must be more explicit.

**예**:
>**Example**: Consider

	int length(const char* p);

`length(s)`를 호출 할 때 `s==nullptr`을 검사해야 하나요? `length()`를 구현 할 때 `p==nullptr`을 검사해야 하나요?
>When I call `length(s)` should I test for `s==nullptr` first? Should the implementation of `length()` test for `p==nullptr`?

	int length(zstring p);	// it is the caller's job to make sure p!=nullptr

	int length(not_null<Zstring> p);	// the implementor of length() must assume that p==nullptr is possible

**참고 사항**: `zstring`은 소유권은 표현하지 않습니다.
>**Note**: `zstring` do not represent ownership.

**더 보기**: [Support library](#S-support).
>**See also**: [Support library](#S-support).


<a name="Rf-const-T-ref"></a>
### F.20: 크기가 큰 객체는 `const T&`형 매개변수를 사용하라
>### F.20: Use a `const T&` parameter for a large object

**근거**: 큰 객체를 복사하는데는 비용이 많이 들어갈 수 있습니다. `const T&`는 항상 비용이 적고 호출자가 의도하지 않게 값을 수정하는 오류를 저지르지 않게 막아 줍니다.
>**Reason**: Copying large objects can be expensive. A `const T&` is always cheap and protects the caller from unintended modification.

**예**:
>**Example**:

	void fct(const string& s);	// OK: pass by const reference; always checp

	void fct2(string s);		// bad: potentially expensive

**예외**: 소굴 (즉, 객체를 소멸시키거나 다른 소굴로 전달한다면) 이득이 있을까 ??? 
>**Exception**: Sinks (that is, a function that eventually destroys an object or passes it along to another sink), may benefit ???

**참고 사항**: 참조자는 유효한 객체를 가리키고 있다고 가정한다 (언어 규칙).
"널 참조"는 없다.
옵션 값을 사용하고자 하면 `std::optional`과 같은 포인터를 사용하거나 "값 없음"을 의미하는 특별한 값을 사용하세요. 
>**Note**: A reference may be assumed to refer to a valid object (language rule).
There in no (legitimate) "null reference."
If you need the notion of an optional value, use a pointer, `std::optional`, or a special value used to denote "no value."

**시행하기**:
>**Enforcement**:

* (단순) ((기초)) 매개변수에 `4*sizeof(int)`보다 큰 객체가 값으로 전달 된다면 경고 하세요.
그리고 `const` 참조형을 사용하도록 제안하세요. 

>* (Simple) ((Foundation)) Warn when a parameter being passed by value has a size greater than `4*sizeof(int)`.
Suggest using a `const` reference instead.


<a name="Rf-T"></a>
### F.21: 크기가 작은 객체에는 `T`형 매개변수를 사용하세요
>### F.21: Use a `T` parameter for a small object

**근거**: 그 어떤것도 단순함과 복사 안정성을 이길 수는 없습니다.
작은 객체의 경우 (최대 8또는 12바이트) 그 어떤 대안보다 더 빠릅니다.
>**Reason**: Nothing beats the simplicity and safety of copying.
For small objects (up to two or three words) is is also faster than alternatives.

**예**:
>**Example**:

	void fct(int x);		// OK: Unbeatable

	void fct(const int& x);	// bad: overhead on access in fct2()

	void fct(int& x);		// OK, but means something else; use only for an "out parameter"

**수행하기**:
>**Enforcement**:

* (단순) ((기초)) `3*sizeof(int)`보다 작은 크기의 객체가 `const` 참조형 매개변수로 전달된다면 경고하세요. 대신 값형으로 전달 할 것을 제안 하세요.

>* (Simple) ((Foundation)) Warn when a `const` parameter being passed by reference has a size less than `3*sizeof(int)`. Suggest passing by value instead.


<a name="Rf-T-ref"></a>
### F.22: 입출력 매개변수는 `T&`를 사용하라
>### F.22: Use a `T&` for an in-out-parameter

**근거**: 함수는 비상수 참조 인자에 값을 쓸 수 있고 그렇게 할 것이라고 가정하세요. 
>**Reason**: A called function can write to a non-`const` reference argument, so assume that it does.

**예**:
**Example**:

	void update(Record& r);	// assume that update writes to r
	
**예외사항**: 함수는 `T&` 인자로  정보를 전달할 수도 있지만 내보낼 수도 있습니다.
그러므로 `T&`는 입출력 매개변수가 됩니다. 그래서 그 자체로 문제가 되거나 에러의 원인이 되기도 합니다. 
>**Note**: A `T&` argument can pass information into a function as well as well as out of it.
Thus `T&` could be and in-out-parameter. That can in itself be a problem and a source of errors:

	void f(string& s)
	{
		s = "New York";	// non-obvious error
	}
	
	string g()
	{
		string buffer = ".................................";
		f(buffer);
		// ...
	}
	
여기서, `g()` 작성자는  `f()`에게 버퍼를 제공하고 있습니다. 그러나 `f()`는 버퍼를 덮어쓰기하고 있습니다 (문자들을 단순히 복사하는 것보다 비용이 조금 더는 곳에).
`g()` 작성자가 `버퍼`의 크기를 미리 가정해 놓고 사용한다면 잘못된 로직이 에러를 유발시킬 수 있다.  
>Here, the writer of `g()` is supplying a buffer for `f()` to fill,
but `f()` simply replaces it (at a somewhat higher cost than a simple copy of the characters).
If the writer of `g()` makes an assumption about the size of `buffer` a bad logic error can happen.

**수행하기**:
>**Enforcement**:

* (제한) ((기본)) 쓰기를 하지 않는 비상수 참조 인자를 가진 함수는 경고하세요.
* `T&`를 취하는 함수는 표시해두고 `T`로 대체하세요.

>* (Moderate) ((Foundation)) Warn about functions with non-`const` reference arguments that do *not* write to them.
* Flag functions that take a `T&` and replace the `T` referred to, rather what the contents of that `T`


<a name="Rf-T-return-out"></a>
### F.23: 값을 이동하는데 비용이 많이 드는 출력 매개변수는 `T&`를 사용하라
>### F.23: Use `T&` for an out-parameter that is expensive to move (only)

**근거**: 값을 반환하는 것은 `T&`(입출력 매개변수)보다 놓치는 경우가  거의 없고 잘못 사용하는 경우도 드물다; [더 보기](#Rf-return); [see also](#Rf-T-multi).
>**Reason**: A return value is harder to miss and harder to miuse than a `T&` (an in-out parameter); [see also](#Rf-return); [see also](#Rf-T-multi).

**예**:
>**Example**:

	struct Package {
		char header[16];
		char load[2024-16];
	};
	
	Package fill();			// Bad: large return value
	void fill(Package&);	// OK
	
	int val();				// OK
	val(int&);				// Bad: Is val reading its argument

**수행하기**: 반환되는 값의 크기의 상한값을 정하는 것은 어렵습니다. 
>**Enforcement**: Hard to choose a cutover value for the size of the value returned.


<a name="Rf-pass-ref-ref"></a>
### F.24: Use a `TP&&` parameter when forwarding (only)

**Reason**: When `TP` is a template type parameter, `TP&&` is a forwarding reference -- it both *ignores* and *preserves* const-ness and rvalue-ness. Therefore any code that uses a `T&&` is implicitly declaring that it itself doesn't care about the variable's const-ness and rvalue-ness (because it is ignored), but that intends to pass the value onward to other code that does care about const-ness and rvalue-ness (because it is preserved). When used as a parameter `TP&&` is safe because any temporary objects passed from the caller will live for the duration of the function call. A parameter of type `TP&&` should essentially always be passed onward via `std::forward` in the body of the function.

**Example**:

	template <class F, class... Args>
    inline auto invoke(F&& f, Args&&... args) {
        return forward<F>(f)(forward<Args>(args)...);
    }

**Enforcement**: Flag a function that takes a `TP&&` parameter (where `TP` is a template type parameter name) and uses it without `std::forward`.


<a name ="Rf-pass-ref-move"></a>
### F.25: Use a `T&&` parameter together with `move` for rare optimization opportunities

**Reason**: Moving from an object leaves an object in its moved-from state behind.
In general, moved-from objects are dangerous. The only guaranteed operation is destruction (more generally, member functions without preconditions).
The standard library additionally requires that a moved-from object can be assigned to.
If you have performance justification to optimize for rvalues, overload on `&&` and then `move` from the parameter ([example of such overloading](#)).

**Example**:

	void somefct(string&&);
	
	void user()
	{
		string s = "this is going to be fun!";
		// ...
		somefct(std::move(s));		// we don't need s any more, give it to somefct()
		//
		cout << s << '\n';			// Oops! What happens here?
	}

**Enforcement**:

* Flag all `X&&` parameters (where `X` is not a template type parameter name) and uses it without `std::move`.
* Flag access to moved-from objects


<a name="Rf-unique_ptr"></a>
### F.26: 포인터의 소유권을 이동해야 할 때는 `unique_ptr<T>`을 사용하라
>### F.26: Use a `unique_ptr<T>` to transfer ownership where a pointer is needed

**근거**: 가장 저렴한 비용으로 안전하게 포인터의 소유권을 이동시키는 방법은 `unique_ptr`을 사용하는 것이다.
>**Reason**: Using `unique_ptr` is the cheapest way to pass a pointer safely.

**예**:
>**Example**:

	unique_ptr<Shape> get_shape(istream& is)	// assemble shape from input stream
	{
		auto kind = read_header(is); // read header and identify the next shape on input
		switch (kind) {
		case kCicle:
			return make_unique<Circle>(is);
		case kTriangle:
			return make_unique<Triangle>(is);
		// ...
	}

**주의사항**: 인터페이스로써 클래스 상속 관계에 있는 객체를 이동시키는 것이라면 포인터를 전달하는 것이 낫다.
>**Note**: You need to pass a pointer rather than an object if what you are transferring is an object from a class hierarchy that is to be used through an interface (base class).

**수행하기**: (간단) 함수가 지역범위 내에서 할당한 포인터를 반환한다면 경고하세요. 대신 `unique_ptr` 나 `shared_ptr`를 사용하도록 권장하세요.
>**Enforcement**: (Simple) Warn if a function returns a locally-allocated raw pointer. Suggest using either `unique_ptr` or `shared_ptr` instead.


<a name="Rf-shared_ptr"></a>
### F.27: 소유권을 공유 할 때는 `shared_ptr<T>` 사용하라
>### F.27: Use a `shared_ptr<T>` to share ownership

**근거**: `std::shared_ptr`를 사용하여 소유권을 공유하는 방법은 표준 입니다. 마지막 소유자가 객체를 소멸 시킵니다.
>**Reason**: Using `std::shared_ptr` is the standard way to represent shared ownership. That is, the last owner deletes the object.

**예**:
**Example**:

	shared_ptr<Image> im { read_image(somewhere); };
	
	std::thread t0 {shade,args0,top_left,im};
	std::thread t1 {shade,args1,top_right,im};
	std::thread t2 {shade,args2,bottom_left,im};
	std::thread t3 {shade,args3,bottom_right,im};
	
	// detach treads
	// last thread to finish deletes the image

**주의사항**: 소유자가 단 하나인경우 `shared_ptr`보다는 `unique_ptr`을 사용하세요.
`shared_ptr`는 소유권을 공유할 때 사용 합니다. 
>**Note**: Prefer a `unique_ptr` over a `shared_ptr` if there is never more than one owner at a time.
`shared_ptr` is for shared ownership.

**대안**: 공유된 객체를 소유하고 있는 하나의 객체를 만들고 모든 사용자가 사용을 마치면 (아마도 암시적인 방법으로)소멸 시키세요.
>**Alternative**: Have a single object own the shared object (e.g. a scoped object) and destroy that (preferably implicitly) when all users have completd.

**수행하기**: (강요할 수 없음) 이것은 검출해 내기에 매우 복잡한 패턴입니다.
>**Enforcement**: (Not enforceable) This is a too complex pattern to reliably detect.


<a name="Rf-T-return"></a>
### F.40: 값 반환을 선호하라
>### F.40: Prefer return values to out-parameters

**근거**: 코드만으로도 문서화가 된다. `&`는 입/출력 또는 출력용 매개변수가 될 수 있다.
>**Reason**: It's self-documenting. A `&` parameter could be either in/out or out-only.

**예**:
>**Example**:

	void incr(int&);
	int incr();
	
	int i = 0;
	incr(i);
	i = incr(i);

 
**수행하기**: 값을 저장하기 전에 읽지 않는 비상수 매개변수는 표시해 두세요. 그런 경우 단순희 값을 반환하기만 하면 됩니다. 
**Enforcement**: Flag non-const reference parameters that are not read before being written to and are a type that could be cheaply returned.


<a name="Rf-T-multi"></a>
### F.41: 다수의 출력 매개변수는 tuple을 사용하라 
>### F.41: Prefer to return tuples to multiple out-parameters

**근거**: 값 반환은 코드 자체가 "출력용"이라는 문서화 역할을 한다.
예 그렇습니다, C++는 다수의 값을 반환할 수 있는데 `tuple`과 추가 편의를 제공하는 `tie`를 사용합니다.
>**Reason**: A return value is self-documenting as an "output-only" value.
And yes, C++ does have multiple return values, by convention of using a `tuple`, with the extra convenience of `tie` at the call site.

**예**:
>**Example**:

    int f( const string& input, /*output only*/ string& output_data ) { // BAD: output-only parameter documented in a comment
	    // ...
		output_data = something();
		return status;
	}

    tuple<int,string> f( const string& input ) { // GOOD: self-documenting
	    // ...
		return make_tuple(something(), status);
    }

사실, C++98의 표준 라이브러리에서는 `pair`가 두개의 `tuple` 요소를 갖는 것과 같기 때문에 편리한 이 기능을 사용하고 있었습니다.
예를 들어, `set<string> myset`가 주어졌다고 가정해 봅시다:
>In fact, C++98's standard library already used this convenient feature, because a `pair` is like a two-element `tuple`.
For example, given a `set<string> myset`, consider:

    // C++98
	result = myset.insert( “Hello” );
	if (result.second) do_something_with( result.first );    // workaround
	
C++11에서는 이렇게 쓸 수 있습니다, 결과값들을 이미 존재하는 지역변수에 넣으세요:
>With C++11 we can write this, putting the results directly in existing local variables:

	Sometype iter;                                          // default initialize if we haven't already
	Someothertype success;                                  // used these variables for some other purpose

	tie( iter, success ) = myset.insert( “Hello” );         // normal return value
	  if (success) do_something_with( iter );

**예외 사항**: `string`이나 `vector`와 같이 부가적인 능력이 있는 타입은 입/출력 매개변수로 사용하는 것이 "호출자가 출력 메모리를 할당"하는 패턴보다 유용할 때가 있다. 출력을 위한 용도로 사용하기 위해서 비상수 참조형으로 객체를 선언하고 피호출자는 객체가 제공하는 부가기능나 객체가 이미 가지고 있는 자원을 재사용할 수 있다. 이 기법은 반복문안에서 반복적으로 함수를 호출 하면서 문자열 값을 구하는 경우에 메모리 할당 횟수를 많이 줄여준다.
>**Exception**: For types like `string` and `vector` that carry additional capacity, it can sometimes be useful to treat it as in/out instead by using the "caller-allocated out" pattern, which is to pass an output-only object by reference to non-`const` so that when the callee writes to it the object can reuse any capacity or other resources that it already contains. This technique can dramatically reduce the number of allocations in a loop that repeatedly calls other functions to get string values, by using a single string object for the entire loop.

**주의 사항**: 어떤 경우는 `tuple` 대신에 `variant<T,error_code>`처럼 사용될 수 있는 `Value_or_error`와 같은 사용자 정의형으로 값을 반환하는 것이 유용할 때도 있다. 
>**Note**: In some cases it may be useful to return a specific, user-defined `Value_or_error` type along the lines of `variant<T,error_code>`,
rather than using the generic `tuple`.

**수행하기**:

    * 출력 매개변수는 반환값으로 대체되어야 한다.
    출력 매개변수는 함수가 값을 저장하고 비상수 멤버 함수형으로 호출하거나, 비상수형으로 전달한다.

>**Enforcement**:

    * Output parameters should be replaced by return values.
	An output parameter is one that the function writes to, invokes a non-`const` member function, or passes on as a non-`const`.


<a name="Rf-return-ptr"></a>
### F.42: 메모리 주소의 위치를 나타나는 경우에만 `T*`를 반환하라
>### F.42: Return a `T*` to indicate a position (only)

**근거**: 메모리 주소의 위치를 반환하는 경우는 포인터가 가장 좋다.
>**Reason**: That's what pointers are good for.
Returning a `T*` to transfer ownership is a misuse.

**주의 사항**: 호출자의 범위에 있지 않은 포인터는 반환하지 마세요.
>**Note**: Do not return a pointer to something that is not in the caller's scope.

**예**:
>**Example**:

		Node* find(Node* t, const string& s)	// find s in a binary tree of Nodes
		{
			if (t->name==s) return t;
			if (auto p = find(t->left,s)) return p;
			if (auto p = find(t->right,s)) return p;
			return nullptr;
		}

만약 `nullptr`가 아니라면 `find`가 반환하는 포인터는 `s`를 가지는 `node`를 가리킨다. 
중요한점은 이것은 객체를 가리키는 포인터의 소유권이 호출자까지 전달되지 않는다는 것이다.
>If it isn't the `nullptr`, the pointer returned by `find` indicates a `Node` holding `s`.
Importantly, that does not imply a transfer of ownership of the pointed-to object to the caller.

**주의 사항**: 메모리 주소의 위치는 이터레이터나 인덱스, 참조형으로 전달될 수 있다.
>**Note**: Positions can also be transferred by iterators, indices, and references.

**나쁜 예**:
>**Example, bad**:

	int* f()
	{
		int x = 7;
		// ...
		return &x;		// Bad: returns pointer to object that is about to be destroyed
	}

참조에도 똑같이 적용된다.
>This applies to references as well:

	int& f()
	{
		int x = 7;
		// ...
		return x;	// Bad: returns refence to object that is about to be destroyed
	}


**더 보기**: [discussion of dangling pointer prevention](#???).
>**See also**: [discussion of dangling pointer prevention](#???).

**수행하기**: 문제의 약간 다른 변형은 가리키고 있는 객체보다 더 오래 살아 있는 컨테이너에 위치한 포인터 입니다.
>**Enforcement**: A slightly diffent variant of the problem is placing pointers in a container that outlives the objects pointed to.

* 컴파일러는 지역범위로 반환되는 참조형을 잡아내는 경향이 있고 많은 경우에 있어서 지역범위로 반환되는 포인터를 잡아 낼 수 있습니다.
* 정적분석은 대부분의(거의?) 포인터 사용 패턴을 잡아 냅니다(댕글링 포인터 같은)

>* Compilers tend to catch return of reference to locals and could in many cases catch return of pointers to locals.
>* Static analysis can catch most (all?) common patterns of the use of pointers indicating positions (thus eliminating dangling pointers)


<a name="Rf-dangle"></a>
### F.43: Never (directly or indirectly) return a pointer to a local object

**Reason**: To avoid the crashes and data corruption that can result from the use of such a dangling pointer.

**Example**, bad: After the return from a function its local objects no longer exist:

	int* f()
	{
		int fx = 9;
		return &fx;	// BAD
	}
	
	void g(int* p)	// looks innocent enough
	{
		int gx;
		cout << "*p == " << *p << '\n';
		*p = 999;
		cout << "gx == " << gx << '\n';
	}
		
	void h()
	{
		int* p = f();
		int z = *p;		// read from abandoned stack frame (bad)
		g(p);			// pass pointer to abandoned stack frame to function (bad)
		
	}
	
Here on one popular implementation I got the output

	*p == 9
	cx == 999
	
I expected that because the call of `g()` reuses the stack space abandoned by the call of `f()` so `*p` refers to the space now occupied by `gx`.

Imagine what would happen if `fx` and `gx` were of different types.
Imagine what would happen if `fx` or `gx` was a type with an invariant.
Imagine what would happen if more that dangling pointer was passed around among a larger set of functions.
Imagine what a cracker could do with that dangling pointer.

Fortunately, most (all?) modern compilers catch and warn against this simple case.

**Note**: you can construct similar examples using references.

**Note**: This applies only to non-`static` local variables.
All `static` variables are (as their name indicates) statically allocated, so that pointers to them cannot dangle.

**Example**, bad: Not all examples of leaking a pointer to a local variable are that obvious:

	int* glob;		// global variables are bad in so many ways

	template<class T>
	void steal(T x)
	{
		glob = x();	// BAD
	}

	void f()
	{
		int i = 99;
		steal([&] { return &i; });
	}

	int main()
	{
		f();
		cout << *glob << '\n';
	}
	
Here I managed to read the location abandoned by the call of `f`.
The pointer stored in `glob` could be used much later and cause trouble in unpredictable ways.

**Note**: The address of a local variable can be "returned"/leaked by a return statement,
by a `T&` out-parameter, as a member of a returned object, as an element of a returned array, and more.

**Note**: Similar examples can be constructed "leaking" a pointer from an inner scope to an outer one;
such examples are handled equivalently to leaks of pointers out of a function.

**See also**: Another way of getting dangling pointers is [pointer invalidation](#???).
It can be detected/prevented with similar techniques.

**Enforcement**: Preventable through static analysis.


<a name="Rf-return-ref"></a>
### F.44: "객체를 반환하지 않기"가 선택사항이 아니라면 `T&`를 반환하라
>### F.44: Return a `T&` when "returning no object" isn't an option

**근거**: 언어가 `T&`는 객체를 가리키고 있다는 것을 보장하기 때문에 `nullptr`인지 시험하는 것은 필요없다.
>**Reason**: The language guarantees that a `T&` refers to an object, so that testing for `nullptr` isn't necessary.

**더 보기**: 참조형을 반환하는 것은 소유권 이전을 의미하지 않는다.
[discussion of dangling pointer prevention](#???) and [discussion of ownership](#???).

>**See also**: The return of a reference must not imply transfer of ownership:
>[discussion of dangling pointer prevention](#???) and [discussion of ownership](#???).

**예**:
>**Example**:

	???

**수행하기**: ???
>**Enforcement**: ???


<a name="Rf-return-ref-ref"></a>
### F.45: `T&&`를 반환하지 마라
>### F.45: Don't return a `T&&`

**근거**: 이것은 소멸된 임시 객체의 참조를 반환하라고 요청하는 것입니다. `&&`는 임시 객체에 붙는 자석 입니다. 임시 객체를 가리키는 참조는 함수호출보다 오래 살아 있기 때문에 피호줄자로 '하향이동' 시키는 편이 낫습니다. ([F.24](#RF-pass-ref-ref) 와 [F.25](#Rf-pass-ref-move)를 참조 하세요.) 그러나 임시객체의 참조를 호출자 범위로 '상향이동' 시키는 것은 좋지 않습니다. [F54](#Rf-local-ref-ref)를 보세요.
>**Reason**: It's asking to return a reference to a destroyed temporary object. A `&&` is a magnet for temporary objects. This is fine when the reference to the temporary is being passed "downward" to a callee, because the temporary is guaranteed to outlive the function call. (See [F.24](#RF-pass-ref-ref) and [F.25](#Rf-pass-ref-move).) However, it's not fine when passing such a reference "upward" to a larger caller scope. See also [F54](#Rf-local-ref-ref).

(참조 또는 퍼펙트 포워딩을 통하여)함수에 매개변수를 전달하고 값을 반환하려면 간단하게 `auto`를 반환형으로 사용하세요. 
>For passthrough functions that pass in parameters (by ordinary reference or by perfect forwarding) and want to return values, use simple `auto` return type deduction (not `auto&&`).

**나쁜 예**: `F`가 값을 반환한다면, 함수는 임시변수의 참조를 반환하게 됩니다.
>**Example; bad**: If `F` returns by value, this function returns a reference to a temporary.

	template<class F>
	auto&& wrapper(F f) {
		log_call(typeid(f)); // or whatever instrumentation
	    return f();
	}

**좋은 예**: 더 나은:
>**Example; good**: Better:
	
	template<class F>
	auto wrapper(F f) {
		log_call(typeid(f)); // or whatever instrumentation
	    return f();
	}

**예외사항**: `std::move` 와 `std::forward`는 `&&`를 반환하지만 형변환일 뿐이다. -- used by convention only in expression contexts where a reference to a temporary object is passed along within the same expression before the temporary is destroyed. 우리는 `&&`를 반환하는 다른 좋은 예제를 모릅니다. 
>**Exception**: `std::move` and `std::forward` do return `&&`, but they are just casts -- used by convention only in expression contexts where a reference to a temporary object is passed along within the same expression before the temporary is destroyed. We don't know of any other good examples of returning `&&`.

**수행하기**: `std::move` 와 `std::forward`를 제외하고 `&&`를 반환한다면 표시를 해 두세요.
>**Enforcement**: Flag any use of `&&` as a return type, except in `std::move` and `std::forward`.


<a name="Rf-capture-vs-overload"></a>
### F.50: 함수로 할 수 없을 때 람다를 사용하라 (지역변수를 캡쳐하거나 지역함수를 구현하기 위해서)
>### F.50: Use a lambda when a function won't do (to capture local variables, or to write a local function)

**근거**: 함수는 지역변수를 캡쳐할 수 없고, 지역범위로 선언될 수도 없습니다. 이런 기능이 필요하다면 가능한 곳에서 람다를 사용하고, 그렇지 않는 곳에서는 함수객체를 사용하세요. 한편 람다와 함수객체는 오버로드 할 수 없습니다; 만약 로버로드가 필요하다면 함수를 사용하세요(람다를 오버로드하면 굉장히 복잡한 문제를 발생시킵니다). 만약 둘다 사용할 수 있다면 함수를 사용하세요; 가장 단순한 툴을 사용하세요.
>**Reason**: Functions can't capture local variables or be declared at local scope; if you need those things, prefer a lambda where possible, and a handwritten function object where not. On the other hand, lambdas and function objects don't overload; if you need to overload, prefer a function (the workarounds to make lambdas overload are ornate). If either will work, prefer writing a function; use the simplest tool necessary.

**예**:
>**Example**:

	// writing a function that should only take an int or a string -- overloading is natural
	void f(int);
	void f(const string&);
	
	// writing a function object that needs to capture local state and appear
	// at statement or expression scope -- a lambda is natural
	vector<work> v = lots_of_work();
	for(int tasknum = 0; tasknum < max; ++tasknum) {
	    pool.run([=, &v]{
			/*
			...
			... process 1/max-th of v, the tasknum-th chunk
			...
			*/
		});
	}
	pool.join();

**예외사항**: 제네릭 람다는 함수 템플릿을 구현하는 간결한 방법을 제공하기 때문에 더 적은 문법으로 일반 함수 템플릿과 같은 기능을 할 수 있습니다. 그렇지만 앞으로 함수가 컨셉 매개변수를 가지게되면 이 장점은 사라질 것입니다.
>**Exception**: Generic lambdas offer a concise way to write function templates and so can be useful even when a normal function template would do equally well with a little more syntax. This advantage will probably disappear in the future once all functions gain the ability to have Concept parameters.

**수행하기**:
**Enforcement**:

    * Warn on use of a named non-generic lambda (e.g., `auto x = [](int i){ /*...*/; };`) that captures nothing and appears at global scope. Write an ordinary function instead.



<a name="Rf-default-args"></a>
### F.51: 가상함수의 기본인자도 오버로딩하라.
>### F.51: Prefer overloading over default arguments for virtual functions

혹시 다른 상황이 뭐가 있을까?
>??? possibly other situations?

**근거**: 가상함수 오버라이드는 기본인자를 상속하지 않기 때문에 예상치 못한 결과를 만들어 낸다.
>**Reason**: Virtual function overrides do not inherit default arguments, leading to surprises.

**나쁜 예**:
>**Example; bad**:

	class base {
	public:
		virtual int multiply(int value, int factor = 2) = 0;
	};

	class derived : public base {
	public:
		override int multiply(int value, int factor = 10);
	};
	
	derived d;
	base& b = d;
	
	b.multiply(10);	// these two calls will call the same function but
	d.multiply(10); // with different arguments and so different results

**수행하기**: 기본인자를 사용하는 가상함수는 표시를 해두세요.
>**Enforcement**: Flag all uses of default arguments in virtual functions.


<a name="Rf-reference-capture"></a>
### F.52: 지역범위에서 사용되는 변수는(알고리즘에 전달 되는것을 포함) 참조에 의한 캡쳐를 사용하라.
>### F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms

**근거**: 효율성과 정확도를 위해서 지역범위에서 람다를 사용할 때 참조에 의한 캡쳐를 원할 겁니다. 이 방법은 지역범위에서 병렬알고리즘을 호출 할 때도 해당됩니다.
>**Reason**: For efficiency and correctness, you nearly always want to capture by reference when using the lambda locally. This includes when writing or calling parallel algorithms that are local because they join before returning.

**예**: 아래 예는 간단한 3단계 병렬 파이프라인 입니다. 각 `stage` 객체는 작업 쓰레드와 큐를 캡슐화 하고 큐를 사용하는  `process` 함수, 그리고 소멸자는 쓰레드가 종료되기 전에 큐가 비어질 때까지 대기 합니다.  
>**Example**: This is a simple three-stage parallel pipeline. Each `stage` object encapsulates a worker thread and a queue, has a `process` function to enqueue work, and in its destructor automatically blocks waiting for the queue to empty before ending the thread.

	void send_packets( buffers& bufs ) {
	    stage encryptor  ([] (buffer& b){ encrypt(b); });
	    stage compressor ([&](buffer& b){ compress(b); encryptor.process(b); });
	    stage decorator  ([&](buffer& b){ decorate(b); compressor.process(b); });
	    for (auto& b : bufs) { decorator.process(b); }
	} // automatically blocks waiting for pipeline to finish

**수행하기** ???
>**Enforcement**: ???


<a name="Rf-value-capture"></a>
### F.53: 람다에서는 지역범위에서 사용되지 않는 변수를(반환값, 힙에 할당된 값 그리고 다른 쓰레드로 전달되는 값을 포함하여) 참조에 의한 캡쳐를 해서는 안된다. 
>### F.53: Avoid capturing by reference in lambdas that will be used nonlocally, including returned, stored on the heap, or passed to another thread

**근거**: 지역범위에 있는 포인터와 참조는 범위를 넘어서면 더 이상 존재하지 않는다.
참조의한 캡쳐를 가진 람다는 지역 객체의 참조를 저장하는 또 다른 공간일 뿐이고 지역범위를 넘어서면 더 이상 존재하지 않는다. 
>**Reason**: Pointers and references to locals shouldn't outlive their scope. Lambdas that capture by reference are just another place to store a reference to a local object, and shouldn't do so if they (or a copy) outlive the scope.

**예**:
>**Example**:

	{
		// ...
		
		// a, b, c are local variables
		background_thread.queue_work([=]{ process(a,b,c); });	// want copies of a, b, and c
	}

**수행하기**: ???
>**Enforcement**: ???

