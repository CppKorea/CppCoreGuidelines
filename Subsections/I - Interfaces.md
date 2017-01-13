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