### <a name="Ri-abstract"></a> I.16: 클래스 상속보다 인터페이스로 추상 클래스(abstract class)를 선호하라.
>### <a name="Ri-abstract"></a> I.16: Prefer abstract classes as interfaces to class hierarchies

##### Reason

추상 클래스는 상태가 있는 부모클래스보다 더 안정적이다.
>Abstract classes are more likely to be stable than base classes with state.

##### Example, bad

You just knew that `Shape` would turn up somewhere :-)

	class Shape {	// bad: interface class loaded with data
	public:
		Point center() { return c; }
		virtual void draw();
		virtual void rotate(int);
		// ...
	private:
		Point c;
		vector<Point> outline;
		Color col;
	};

이 클래스는 상속받는 모든 자식 클래스에 센터를 계속하도록 강요할 것이다. -- center를 사용하지도 않을 거고 non-trivial한데도 불구하고 말이다. 모든 'Shape'가 'Color'를 가지지 않는다. 게다가 많은 'Shape'는 포인트의 열로 정의된 외각선이 없이도 표현된다. 추상클래스는 그런류의 클래스를 사용할 수 있도록 개발되었다.
>This will force every derived class to compute a center -- even if that's non-trivial and the center is never used. Similarly, not every `Shape` has a `Color`, and many `Shape`s are best represented without an outline defined as a sequence of `Point`s. Abstract classes were invented to discourage users from writing such classes:

	class Shape {	// better: Shape is a pure interface
	public:
		virtual Point center() =0;	// pure virtual function
		virtual void draw() =0;
		virtual void rotate(int) =0;
		// ...
		// ... no data members ...
	};

##### Enforcement

(Simple) 'C' 클래스 포인터가 부모클래스의 포인터를 가지고 있고 그 부모클래스가 데이터 멤버를 가진다면 경고하라.
>(Simple) Warn if a pointer to a class `C` is assigned to a pointer to a base of `C` and the base class contains data members.

### <a name="Ri-abi"></a> I.16: 컴파일러간 호환을 위한 바이너리 인터페이스(ABI)을 원한다면 C 스타일을 사용하라.
>### <a name="Ri-abi"></a> I.16: If you want a cross-compiler ABI, use a C-style subset

##### Reason

컴파일러는 제각각 클래스의 바이너리 레이아웃, 예외 처리, 함수명, 상세 실행방식이 다르게 구현되어 있다.
>Different compilers implement different binary layouts for classes, exception handling, function names, and other implementation details.

**Exception**: 상위레벨 C++ 타입만 사용해서 신경써서 인터페이스를 만들어야 한다. See ???.
>**Exception**: You can carefully craft an interface using a few carefully selected higher-level C++ types. See ???.

**Exception**: ~~ 몇몇 플랫폼 상에서 공통 ABI가 나타나고 있다.
>**Exception**: Common ABIs are emerging on some platforms freeing you from the more draconian restrictions.

##### Note

한종류만 사용한다면 C++ 기능을 풀로 사용할 수 있다는 것으로 새버전에 대해서도 재컴파일이 가능하는 의미이다.
>If you use a single compiler, you can use full C++ in interfaces. That may require recompilation after an upgrade to a new compiler version.

##### Enforcement

(Not enforceable) 어느 인터페이스가 ABI를 만족할지 신뢰할 정도로 구분해 낼 수 없다.
>(Not enforceable) It is difficult to reliably identify where an interface forms part of an ABI.