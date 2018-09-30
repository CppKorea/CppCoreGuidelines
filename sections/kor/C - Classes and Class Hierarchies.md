

복사와 이동 규칙들 : 

* [C.60: 복사연산을 `virtual`로 만들지 말아라. 매개변수는 `const&`로 받고, `const&`로 반환하지 말아라](#Rc-copy-assignment)
* [C.61: 복사 연산은 복사를 수행해야 한다](#Rc-copy-semantic)
* [C.62: 복사 연산은 자기 대입에 안전하게 작성하라](#Rc-move-self)
* [C.63: 이동 연산은 `virtual`로 만들지 말아라, 매개변수는 `&&`를 사용하고, `const&`로 반환하지 말아라](#Rc-move-assignment)
* [C.64: 이동 연산은 이동을 수행해야 하며, 원본 객체를 유효한 상태로 남겨놓아야 한다](#Rc-move-semantic)
* [C.65: 이동 연산은 자기 대입에 안전하게 작성하라](#Rc-copy-self)
* [C.66: 이동 연산은 `noexcept`로 만들어라](#Rc-move-noexcept)
* [C.67: 기본 클래스에 대한 복사를 제한하라, 대신 복사가 필요하다면 가상 `clone`함수를 제공하라](#Rc-copy-virtual)


다른 기본 연산들에 대한 규칙 :

* [C.80: 기본 의미론을 명시적으로 사용하려면 `=default` 키워드를 사용하라](#Rc-default)
* [C.81: 기본 동작을 (대안을 원하지 않고) 금지하고 싶다면 `=delete`를 사용하라](#Rc-delete)
* [C.82: 생성자 또는 소멸자에서 가상 함수를 호출하지 말아라](#Rc-ctor-virtual)
* [C.83: 값 타입들에는, `noexcept` swap함수를 제공하는 것을 고려하라](#Rc-swap)
* [C.84: `swap` 연산은 실패해선 안된다](#Rc-swap-fail)
* [C.85: `swap` 연산은 `noexcept`로 작성하라](#Rc-swap-noexcept)
* [C.86: `==`연산자는 피연산자 타입들에 대칭적이고, `noexcept`로 만들어라](#Rc-eq)
* [C.87: 기본 클래스에 있는 `==`에 주의하라](#Rc-eq-base)
* [C.89: `hash`는 `noexcept`로 작성하라](#Rc-hash)



## <a name="SS-defop"></a>C.defop: 기본 연산들

기본적으로, 언어에서 기본적인 의미를 담는 기본 연산들을 제공한다.
그러나, 프로그래머는 기본적으로 제공되는 것들을 막거나 바꿀 수 있다.


### <a name="Rc-zero"></a>C.20: 기본 연산을 정의하지 않아도 되면 그렇게 하라

##### 근거
가장 단순하고, 가장 명료한 의미를 준다.

##### 예
```c++
    struct Named_map {
    public:
        // ... no default operations declared ...
    private:
        string name;
        map<int, int> rep;
    };

    Named_map nm;        // default construct
    Named_map nm2 {nm};  // copy construct
```
`std::map` 과 `string` 은 모든 특수한 함수들을 갖고 있다, 추가적인 작업이 필요없다.

##### 참고 사항
"the rule of zero"로 알려져 있다.

##### 시행하기
시행할 수 없더라도, 좋은 정적 분석기는 이 규칙에 맞는 가능한 개선사항들을 알려주는 패턴들을 찾을 수 있다.
예를 들면, 포인터와 크기를 멤버로 갖는 클래스가 있고 소멸자에서 그 포인터를 `delete` 한다면 아마도 `vector` 로 바꿀 수 있을 것이다.



### <a name="Rc-five"></a>C.21: 기본 연산을 정의 하거나 `=delete` 로 선언했다면, 나머지 모두 정의하거나 `=delete`하라

##### 근거
특별한 함수들의 의미론들은 밀접하게 연관되어 있다. 만약 한 함수가 기본 제공 함수가 아니어야 한다면(non-default), 다른 함수들도 수정이 필요하다.

##### 잘못된 예
```c++
    struct M2 {   // bad: incomplete set of default operations
    public:
        // ...
        // ... no copy or move operations ...
        ~M2() { delete[] rep; }
    private:
        pair<int, int>* rep;  // zero-terminated set of pairs
    };

    void use()
    {
        M2 x;
        M2 y;
        // ...
        x = y;   // the default assignment
        // ...
    }
```
여기서는, 소멸자에 대한 "특별한 주의"가 필요하다고 한다면, 복사와 이동 할당(둘 다 묵시적으로 객체를 소멸할 것이다)이 정확하게 동작할 가능성은 적다. (여기서는, 두번 `delete`를 시도할 것이다)

##### 참고 사항
기본 생성자를 중요하게 생각하는지에 달려있는데, 이것은 "the rule of five" 혹은 "the rule of six" 이라고 알려져 있다.

##### 참고 사항
다른 것은 정의 하더라도 기본 연산의 기본 구현이 필요하다면, `=default` 을 사용하여 해당 함수에 대한 의도를 표현하라.
기본 연산을 원하지 않는다면, `=delete`를 써서 제한하라.

##### 참고 사항
컴파일러는 이 규칙을 강제하고 이상적으로는 위반사항이 발생하면 경고한다.

##### 참고 사항
클래스에 묵시적으로 생성된 복사 연산에 의존하는 것은 더 이상 사용되지 않는다.

##### 시행하기
(쉬움) 클래스는 특별한 함수들에 대한 선언(`=delete`도 포함하여)을 모두 갖거나 갖지 말아야 한다.



### <a name="Rc-matched"></a>C.22: 기본 연산들을 일관성 있도록 하라

##### 근거
기본 연산들은 개념적으로 잘 짜여진 집합이다. 연산들의 의미는 서로 연관되어 있다.
사용자는 복사/이동 생성과 복사/이동 할당이 논리적으로 동일하고, 생성자와 소멸자가 리소스 관리에 대해 일관적으로 동작하며, 복사와 이동이 생성자와 소멸자가 동작하는 방식을 반영한다는 것을 기대 할 것이다.


##### 잘못된 예
```c++
    class Silly {   // BAD: Inconsistent copy operations
        class Impl {
            // ...
        };
        shared_ptr<Impl> p;
    public:
        Silly(const Silly& a) : p{a.p} { *p = *a.p; }   // deep copy
        Silly& operator=(const Silly& a) { p = a.p; }   // shallow copy
        // ...
    };
```
이 연산들은 복사 연산에 대한 의미가 일치하지 않는다. 이런 동작은 혼란을 야기하고 버그를 만들 것이다.

##### 시행하기

* (어려움) 복사/이동 생성자와 이에 대응하는 복사/이동 할당 연산자는 동일한 레벨에서 동일한 멤버 변수를 변경하는 것이 좋다.
* (어려움) 복사/이동 생성자에서 변경하는 멤버 변수들은 다른 생성자들에서도 초기화 하는 것이 좋다.
* (어려움) 복사/이동 생성자는 멤버 변수에 대해 깊은 복사를 수행하고 나서, 소멸자는 멤버 변수를 수정해야 한다.
* (어려움) 소멸자가 멤버 변수를 변경하면, 그 멤버 변수들은 복사/이동 생성자 혹은 할당 연산자에서 쓰여지는 것이 좋다.


## <a name="SS-dtor"></a>C.dtor: 소멸자
"이 클래스에 소멸자가 필요할까?"라는 것은 설계 측면에서 굉장히 강력한 질문이다.

대부분의 클래스들에 대해서 대답은 "no"인데, 그 이유는 해당 클래스가 자원들을 가지고 있지 않거나 소멸과정이 [the rule of zero](#Rc-zero)에 의해 처리되기 때문이다.

요컨대, 클래스의 멤버들이 스스로의 소멸을 관리한다는 것이다.      
만약 대답이 "yes"라면, 그 클래스 설계의 대부분은 [the rule of five](#Rc-five)를 따르게 된다. 

### <a name="Rc-dtor"></a>C.30: 객체가 없어질 때, 명시적인 동작이 필요할 경우 소멸자를 정의하라

##### 근거
소멸자는 암묵적으로 객체의 생명주기의 마지막에 호출된다.
기본 소멸자로 충분하다면 그것을 사용하라.
단순하게 멤버의 소멸자를 호출하는 것이 아닌 코드가 필요할 경우 소멸자를 정의하라.

##### 예
```c++
    template<typename A>
    struct final_action {   // 약간 단순화된 클래스
        A act;
        final_action(A a) :act{a} {}
        ~final_action() { act(); }
    };

    template<typename A>
    final_action<A> finally(A act)   // 타입 추론
    {
        return final_action<A>{act};
    }

    void test()
    {
        // 최종 동작을 지정
        auto act = finally([]{ cout << "Exit test\n"; });  
        // ...
        if (something) 
            return;   // 여기서 소멸자를 통해 호출된다
        // ...
    } // 여기서 소멸자를 통해 호출된다
```

`Final_action` 의 목적은 소멸할 때 실행할 코드(보통 람다)를 얻는 것이다.


##### 참고 사항
사용자 정의 소멸자가 필요한 클래스에는 보통 두 종류가 있다.
* 리소스를 사용하는 클래스가 소멸자가 없는 경우, 예컨대 `vector` 혹은 트랜잭션 코드
* A class that exists primarily to execute an action upon destruction, such as a tracer or `final_action`.

##### 잘못된 예
```c++
    class Foo {   // 좋지 않다; 기본 소멸자를 사용하라
    public:
        // ...
        ~Foo() { s = ""; i = 0; vi.clear(); }  // clean up
    private:
        string s;
        int i;
        vector<int> vi;
    };
```
기본 소멸자가 더 잘 동작하고, 더 효과적이며, 틀리지 않는다.

##### 참고 사항
기본 소멸자가 필요하지만, 생성되지 않도록 되어 있다면 (예, 이동 생성자를 정의한 경우), `=default` 를 사용하라.

##### 시행하기
포인터나 참조와 같은 "암묵적인 자원"이 될 수 있는 것들을 찾아보라. 
모든 데이터 멤버가 소멸자를 갖고 있더라도, 사용자 지정 소멸자가 있는 클래스들을 찾아보라.


### <a name="Rc-dtor-release"></a>C.31: 클래스에 의해 얻어진 모든 리소스는 소멸자에서 해제되어야 한다

##### 근거
리소스 누수를 막는다, 특히 에러가 발생한 상황에서 그렇다.

##### 참고 사항
클래스로 표현되는 리소스들이 기본 연산 집합을 갖고 있을 때 소멸자에서의 리소스 해제가 자동으로 발생한다.

##### 예
```c++
    class X {
        ifstream f;   // 파일을 열었을 수도 있다.
        // ... 다른 기본 연산은 정의되지 않았거나, = deleted 되었다 ...
    };
```
`X`의 `ifstream` 은 `X`가 소멸될 때 묵시적으로 열었을 수 있는 파일을 닫는다.  


##### 잘못된 예
```c++
    class X2 {     // 잘못되었다.
        FILE* f;   // 파일을 가지고 있을 수도 있다.
        // ... 다른 기본 연산은 정의되지 않았거나, = deleted 되었다 ...
    };
```
`X2` 에서는 파일 핸들 누수가 생길 것이다.  

##### 참고 사항
닫지 않은 소켓은 어떨까? 소멸자, 닫기, 정리 연산은 [실패하지 않는 것이 좋다](#Rc-dtor-fail).
그럼에도 불구 하고 발생한다면, 정말 좋은 해결책을 찾기 힘든 문제를 만나는 것이다.

초심자들은 소멸자를 작성할 때 왜 소멸자가 호출되고, 예외를 던짐으로써 "처리 거부"를 할 수 없는지 알지 못할 것이다. 이에 대해서는 [discussion](#Sd-never-fail)을 보라.

문제를 악화시키는 것은, 많은 "닫기/해제" 연산들이 재시도 할 수 없도록 되어있는 것이다.
이 문제를 풀려는 시도는 많았지만, 일반적인 해결책은 알려지지 않았다.
해결책이 없다면, 닫기/해제에 대한 실패를 디자인 오류로 간주하고 종료시키는 것을 고려해 보라.

##### 참고 사항
클래스가 소유하고 있지 않은 객체에 대한 포인터나 참조를 갖고 있을 수 있다.
명백하게, 이 객체들은 클래스의 소멸자에서 `delete`되지 않아야 한다.
예를 들면:

```c++
    Preprocessor pp { /* ... */ };
    Parser p { pp, /* ... */ };
    Type_checker tc { p, /* ... */ };
```
`p`는 `pp`를 참조하지만, 소유하고 있지 않다.

##### 시행하기

* (쉬움) 클래스가 소유자인 포인터나 참조 멤버 변수를 갖고 있다면 (가령, `gsl::owner`를 사용하여 소유하는 경우), 소멸자에서 참조되는 것이 좋다.
* (어려움) 소유권에 대해 명시적으로 기술하지 않은 경우, 포인터나 참조 멤버 변수들이 소유자 인지 판단하라. (예, 생성자를 보라)



### <a name="Rc-dtor-ptr"></a>C.32: 클래스가 날 포인터(`T*`)나 참조(`T&`)를 갖고 있을 때, 소유하고 있는 것인지 고려해 보라

> 역주 : 날 포인터(raw pointer) [관련 이슈](https://github.com/CppKorea/CppCoreGuidelines/issues/88)

##### 근거
소유권에 대해서 상세하지 않은 코드는 많이 있다.

##### 예
```
    ???
```

##### 참고 사항
`T*` 혹은 `T&` 가 소유를 의미한다면, **소유한다는** 표시를 하라. `T*` 에 소유의 의미가 없다면 `ptr` 로 표시하는 것을 고려하라.
이것은 문서화와 분석에 도움이 될 것이다.

##### 시행하기
저수준 일반 포인터나 멤버 참조의 초기화를 살펴보고, 할당이 사용되는지 보라.

Look at the initialization of raw member pointers and member references and see if an allocation is used.

### <a name="Rc-dtor-ptr2"></a>C.33: 클래스가 포인터 멤버를 소유하고 있다면, 소멸자를 정의하거나 `=delete` 로 선언하라


##### 근거
소유된 객체는 그것을 소유한 객체가 소멸될 때 `삭제`되어야 한다.

##### 예
포인터 멤버는 리소스일 것이다. [`T*`는 리소스가 아니어야 한다](#Rr-ptr), 이는 오래된 코드에서는 일반적이다.
가능한 `T*`를 소유자라고 고려하고, 의심해보라.

```c++
    template<typename T>
    class Smart_ptr {
        T* p;   // BAD: *p 의 소유가 불분명하다
        // ...
    public:
        // ... 사용자가 복사 연산을 정의하지 않았다 ...
    };

    void use(Smart_ptr<int> p1)
    {
        // error: p2.p 에 누수가 발생한다.
        //      (nullptr가 아니거나 다른 코드에서 소유하지 않는다면)
        auto p2 = p1;
    }
```

소멸자를 정의 한다면, [모든 기본 연산들](#Rc-five)을 정의하거나 삭제해야 한다.

```c++
    template<typename T>
    class Smart_ptr2 {
        T* p;   // BAD: *p 의 소유가 불분명하다
        // ...
    public:
        // ... 사용자가 복사 연산을 정의하지 않았다 ...
        ~Smart_ptr2() { delete p; }  // p 가 자원을 소유하고 있었다!
    };

    void use(Smart_ptr<int> p1)
    {
        auto p2 = p1;   // error: 소멸자가 2번 호출된다.
    }
```
기본 복사 연산은 단지 `p1.p` 를 `p2.p` 로 복사하고, `p1.p` 가 두번 소멸되게 만들 것이다. 소유권을 명시하라:

```c++
    template<typename T>
    class Smart_ptr3 {
        owner<T>* p;   // OK: 명시적으로 *p 의 소유권을 가진다. 
        // ...
    public:
        // ...
        // ... 복사와 이동 연산들 ...
        ~Smart_ptr3() { delete p; }
    };

    void use(Smart_ptr3<int> p1)
    {
        auto p2 = p1;   // error: 소멸자가 2번 호출된다
    }
```

##### 참고 사항
보통 소멸자를 사용하는 가장 단순한 방법은 포인터를 스마트 포인터(가령, `std::unique_ptr`)로 교체하고, 컴파일러가
적절한 소멸자를 암묵적으로 호출하게 만들도록 놔두는 것이다.

##### 참고 사항
왜 소유하고 있는 모든 포인터를 "스마트 포인터"로 사용하도록 하지 않는가?
가끔 중요하지 않은 코드 변경을 만들게 되고, ABI 에 영향을 줄 수 있다.

##### 시행하기
* 포인터 데이터 맴버를 갖는 클래스를 의심하라.
* `owner<T>` 를 갖는 클래스는 기본 연산들을 정의 해야한다.


### <a name="Rc-dtor-ref"></a>C.34: 클래스가 참조 멤버를 소유하고 있다면, 소멸자를 정의하거나 `=delete` 로 선언하라

##### 근거
참조 멤버는 클래스가 소유한 리소스일 수도 있다.
그래서는 안되지만, 오래된 코드에선 흔히 발견된다.
[포인터 멤버들과 소멸자](#Rc-dtor-ptr)를 보라.
또, 복사가 복사 손실로 이어질 수도 있다.

##### 잘못된 예
```c++
    class Handle {  // 굉장히 의심스럽다 
        Shape& s;   // 중복 바인딩을 막기 위해서 포인터보다는 참조자를 사용하라
                    // BAD: *p 의 소유가 불분명하다
        // ...
    public:
        Handle(Shape& ss) : s{ss} { /* ... */ }
        // ...
    };
```
`Handle` 이 `Shape` 를 소멸할 책임이 있는지에 대한 문제는 [the pointer case](#Rc-dtor-ptr)과 동일하다.
만약 `Handle` 이 `s` 로 참조되는 객체를 소유한다면, 소멸자가 있어야 한다.

##### 예
```c++
    class Handle {        // OK
        // 중복 바인딩을 막기 위해서 포인터보다는 참조자를 사용하라
        owner<Shape&> s;  
        // ...
    public:
        Handle(Shape& ss) : s{ss} { /* ... */ }
        ~Handle() { delete &s; }
        // ...
    };
```
`Handle` 이 `Shape` 를 소유하는지와는 별개로, 기본 복사 동작에 대해 의심해야 한다:

```c++
    // Handle이 Circle을 소유하지 않는다면 누수가 발생할 수 있다.
    Handle x { *new Circle{p1, 17} };

    Handle y { *new Triangle{p1, p2, p3} };
    
    // 기본 대입 연산은 *x.s = *y.s 를 시도할 것이다.
    x = y;     
```
코드에 사용된 `x = y` 는 굉장히 의심스럽다.
`Triangle` 을 `Circle` 에 할당한다?
`Shape` 에 [복사 대입을 `=deleted`](#Rc-copy-virtual) 하지 않았다면, `Triangle` 의 `Shape` 부분만 `Circle` 로 복사된다.

##### 참고 사항
모든 소유하는 참조는 "스마트 포인터"로 대체하도록 하는 것은 어떤가?
참조를 스마트 포인터로 변경하는 것은 코드 변경을 의미한다.
아직 스마트 참조는 없다. ABI 에 영향을 줄 수도 있다.

##### 시행하기
* 참조 데이터 멤버를 갖는 클래스는 의심스럽다.
* `owner<T>` 참조를 갖는 클래스는 기본 연산들을 정의하는 것이 좋다.


### <a name="Rc-dtor-virtual"></a>C.35: 기본 클래스의 소멸자는 `public` `virtual` 이거나, `protected` non-`virtual`이어야 한다.

##### 근거
미정의 동작(undefined behavior)을 막기 위함이다.

만약 소멸자가 `public` 이면, 호출하는 코드는 파생 클래스가 기본 클래스의 포인터를 통해 소멸될 것이라 생각한다. 그리고 기본 클래스의 소멸자가 `virtual`이 아니면 결과는 미정의 동작으로 이어진다.

만약 소멸자가 `protected`라면, 호출하는 코드는 기본 클래스의 포인터를 통해서 소멸시킬 수 없고, 따라서 소멸자는 `virtual`이 아니어도 문제가 없다. `private`가 아닌 `protected`여야 하는 이유는 파생 클래스의 소멸자가 호출할 수 있어야 하기 때문이다.

일반적으로, 기본 클래스의 작성자는 소멸 과정에서 어떤 동작이 적합한지 알 수 없다.  


##### 토의

[토론](#Sd-dtor)을 함께 읽어보라.

##### 잘못된 예
```c++
    struct Base {  // BAD: virtual 소멸자가 없다
        virtual f();
    };

    struct D : Base {
        string s {"a resource needing cleanup"};
        ~D() { /* ... 정리 작업을 한다 ... */ }
        // ...
    };

    void use()
    {
        unique_ptr<Base> p = make_unique<D>();
        // ...
    } 
    // p 의 소멸은 ~Base()를 호출하지만, ~D() 는 호출하지 않는다.
    // 따라서 D::s 에 누수가 발생하고, 다른 자원들도 누수될 것이다.

```

##### 참고 사항

가상(`virtual`) 함수는 파생 클래스에 대한 인터페이스를 제공한다. 이 인터페이스를 통해 파생 클래스에 대해 신경을 쓰지 않게 된다.   
만약 인터페이스가 소멸을 지원한다면, 그 과정은 안전해야만 한다.

##### 참고 사항
소멸자는 private이 아니어야 한다. 만약 그럴 경우 해당 타입을 사용하지 못하게 될 것이다 : 

```c++
    class X {
        ~X();   // private 소멸자
        // ...
    };

    void use()
    {
        X a;                        // error: 소멸시킬 수 없다
        auto p = make_unique<X>();  // error: 소멸시킬 수 없다
    }
```
##### 예외 사항
protected virtual 소멸자를 원하지 않는 경우를 상상해볼 수 있다. 파생 타입의 객체가 기본 타입 포인터를 통해 (그 자신이 아닌) *다른* 객체의 소멸을 하도록 허용해야 하는 경우가 그러하다. 하지만 아직까지 그런 사례를 볼 수 없었다.

##### 시행하기
* 가상 함수를 하나라도 가지는 클래스는 `public` 하고 `virtual`한 소멸자를 가져야 한다. 또는 `protected`이고 non-`virtual`한 소멸자를 가져야 한다.


### <a name="Rc-dtor-fail"></a>C.36: 소멸자는 실패해선 안된다

##### 근거
일반적으로 소멸자가 실패할 때 에러 없는 코드를 작성하는 방법을 알 수 없다. 표준 라이브러리에서 다루는 모든 클래스들은 예외를 던지지 않는 소멸자를 요구한다.

##### 예
```c++
    class X {
    public:
        ~X() noexcept;
        // ...
    };

    X::~X() noexcept
    {
        // ...
        if (cannot_release_a_resource) terminate();
        // ...
    }
```
##### 참고 사항

소멸자에서의 실패를 다루기 위해 실패할 염려가 없는 방법(scheme)을 많이 고안해 왔다. 이에 대해선 일반적인 방법으로 성공한 예가 없다.

이것은 정말 현실적인 문제가 될 수 있다: 예를 들면, 닫지 않은 소켓은 어떤가?  
소멸자를 작성하는 사람은 왜 소멸자가 호출되고 예외를 던짐으로써 "동작을 거부하는 것"을 할 수 없는지 모른다.

[토론](#Sd-dtor)을 보라.    
문제를 악화시키는 것은, 많은 "close/release" 연산이 재시도할 수 없게 되어있는 것이다.
가능하다면, close/failure에 대한 실패를 근본적인 디자인 오류로 간주하고 종료시켜라.

##### 참고 사항
소멸자를 `noexcept`로 선언하라. 이것은 소멸자가 정상적으로 완료했거나 프로그램을 종료한다는 것을 보장한다.

##### 참고 사항
만약 자원이 해제될 수 없고 프로그램이 실패하지 않는다면, 어떤 방법으로든 시스템의 나머지 부분에서 실패 했다는 신호를 보내도록 하라.
(전역 상태 변수를 수정하고 프로그램의 다른 부분이 그것을 확인하고 아마도 문제를 처리할 수 있을 것이다)

이 방식은 특별한 목적이 있고, 에러가 발생하기 쉽다는 것을 충분히 이해하라.

예시로 "닫히지 않는 연결"을 고려해보자.
어쩌면 연결의 반대편에 문제가 있을 수 있고, 이때 양쪽의 연결을 담당하는 코드만이 문제를 처리할 수 있다.
소멸자가 (어떤 방법으로) 시스템의 담당(responsible) 부분에 메세지를 보내고, 연결이 닫힌 것으로 간주한 뒤, 정상적으로 반환할 수도 있다. 

##### 참고 사항
소멸자가 실패할 수도 있는 연산을 사용한다면, 예외를 잡을 수 있고, 어떤 경우에는 성공적으로 완료할 수 있다.
(가령, 예외를 던진 메커니즘과 다른 정리(clean-up) 메커니즘을 사용하는 것이다)

##### 시행하기
(쉬움) 소멸자는 `noexcept`로 선언되어야 한다.


### <a name="Rc-dtor-noexcept"></a>C.37: 소멸자를 `noexcept`로 작성하라

##### 근거
[소멸자는 실패해선 안된다](#Rc-dtor fail).   
만약 소멸자가 예외로 인해 종료되려고 한다면, 좋지 않은 디자인 오류로 보고 종료하는 편이 나을 것이다.

##### 참고 사항

A destructor (either user-defined or compiler-generated) is implicitly declared `noexcept` (independently of what code is in its body) if all of the members of its class have `noexcept` destructors.

##### 시행하기

(쉬움) 소멸자는 `noexcept`로 선언되어야 한다.




## <a name="SS-ctor"></a>C.ctor: 생성자
> [원문](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#cctor-constructors)  

생성자는 객체가 생성되는(초기화되는) 방법을 정의 한다.


### <a name="Rc-ctor"></a>C.40: 클래스가 불변조건을 가지면 생성자를 정의하라
> 역주 : 불변조건(invariant)

##### 근거
이는 생성자가 존재하는 이유이다.

##### 예
```
    class Date {  // Date 클래스는 유효한 날짜를 표현한다
                  // 범위 : 1900년 1월 1일 ~ 2100년 12월 31일
        Date(int dd, int mm, int yy)
            :d{dd}, m{mm}, y{yy}
        {
            if (!is_valid(d, m, y)) 
                throw Bad_date{};  // 불변조건을 강제한다
        }
        // ...
    private:
        int d, m, y;
    };
```
생성자에서 `Ensures`로 불변조건을 표현하는 것은 좋은 생각이다.


##### 참고 사항
생성자는 클래스가 불변조건이 아니더라도 편의를 위해 사용될 수 있다.  
예:
```
    struct Rec {
        string s;
        int i {0};
        Rec(const string& ss) : s{ss} {}
        Rec(int ii) :i{ii} {}
    };

    Rec r1 {7};
    Rec r2 {"Foo bar"};
```

##### 참고 사항
C++11 초기화 리스트 규칙은 많은 생성자의 필요성을 제거한다. 
예:  
```
    struct Rec2{
        string s;
        int i;
        Rec2(const string& ss, int ii = 0) :s{ss}, i{ii} {}   // redundant
    };

    Rec r1 {"Foo", 7};
    Rec r2 {"Bar"};
```
`Rec2` 생성자는 중복적이다.
또한, `int`에 대한 기본값은 [member initializer](#Rc-in-class initializer)를 사용하는 편이 났다.

##### 함께 보기
[유효한 객체를 생성하라](#Rc-complete).  
[생성자가 던지는 예외](#Rc-throw).

##### 시행하기
* 사용자 정의 복사 연산이 있지만 소멸자가 없는 클래스를 표시해보라 (사용자 정의 복사는 클래스가 불변조건을 가진다는 것을 알려준다)



### <a name="Rc-complete"></a>C.41: 생성자는 완전히 초기화된 객체를 생성해야 한다

##### 근거
생성자는 클래스에 대한 불변조건을 설정한다. 클래스 사용자는 생성된 객체가 사용가능하다는 것을 가정할 수 있어야 한다.

##### 잘못된 예
```
    class X1 {
        FILE* f;   // 다른 함수에 앞서 init()을 호출한다
        // ...
    public:
        X1() {}
        void init();   // 멤버 f 초기화
        void read();   // 멤버 f 로부터 읽는다
        // ...
    };

    void f()
    {
        X1 file;
        file.read();   // crash 또는 bad read 가 발생한다.
        // ...
        file.init();   // 초기화 하기엔 너무 늦었다
        // ...
    }
```
컴파일러는 주석을 읽지 않는다.

##### 예외 사항
생성자만으로 유효한 객체를 쉽게 만들 수 없다면 [팩토리 함수를 사용하라](#C factory)

##### 참고 사항
생성자가 유효한 객체를 만들기 위해 자원을 얻는다면, 리소스는 [소멸자에 의해 해제](#Rc-release)되어야 한다.
생성자에서 자원을 얻고 소멸자에서 자원을 해제하는 것을 [RAII](Rr-raii) ("Resource Acquisitions Is Initialization") 라고 한다.


### <a name="Rc-throw"></a>C.42: 생성자가 유효한 객체를 생성하지 못한다면, 예외를 던지도록 하라

##### 근거
유효하지 않은 객체를 남겨두는 것은 문제를 일으킬 것이다.

##### 예
```c++
    class X2 {
        FILE* f;   // 다른 함수에 앞서 init()을 호출한다
        // ...
    public:
        X2(const string& name)
            :f{fopen(name.c_str(), "r")}
        {
            if (f == nullptr) throw runtime_error{"could not open" + name};
            // ...
        }

        void read();      // 멤버 f 로부터 읽는다
        // ...
    };

    void f()
    {
        X2 file {"Zeno"}; // file이 열려있지 않으면 예외를 던진다
        file.read();      // 문제 없다
        // ...
    }
```

##### 잘못된 예
```c++
    class X3 {     // bad: 생성자가 유효하지 않은 객체를 남겨놓을 수 있다
        FILE* f;   // 다른 함수에 앞서 init()을 호출한다
        bool valid;
        // ...
    public:
        X3(const string& name)
            :f{fopen(name.c_str(), "r")}, valid{false}
        {
            if (f) valid = true;
            // ...
        }

        void is_valid() { return valid; }
        void read();   // 멤버 f 로부터 읽는다
        // ...
    };

    void f()
    {
        X3 file {"Heraclides"};
        file.read();   // crash 또는 bad read가 발생한다!
        // ...
        if (is_valid()) {
            file.read();
            // ...
        }
        else {
            // ... error를 처리한다 ...
        }
        // ...
    }
```
##### 참고 사항
변수를 정의할 때는 (가령, 스택에 혹은 다른 객체의 멤버로써) 에러코드가 리턴되는 명시적인 함수 호출은 없다.
유효하지 않은 객체를 남겨두고 사용하기 전에 지속적으로 `is_valid()` 함수를 호출해야 하는 것은 번거롭고, 에러가 발생하기 쉬우며, 비효율적 이다.

##### 예외 사항
타이밍의 관점에서 볼 때 (추가적인 툴 지원 없이) 예외 처리가 충분하게 예측 가능하지 않은 실시간 시스템(비행기 제어를 생각해 보라)과 같은 영역이 있다. 이런 경우엔 `is_valid()` 와 같은 방법이 반드시 사용되어야 한다. 이와 같은 경우 [RAII](#Rc-raii)처럼 동작하도록 하기 위해 지속적이고 즉각적으로 `is_valid()` 로 확인하라.

##### 대안
"생성자 이후 초기화" 혹은 "두 단계 초기화"를 사용해야 할 것 같다면, 그렇게 하지 않도록 해보라. 정말로 그렇게 해야 한다면 [팩토리 함수](#Rc-factory)를 검토하라.


##### 참고 사항
사람들이 생성자에서 초기화를 수행하지 않고 `init()`함수를 사용해온 이유 중 하나는 코드의 중복을 막기 위함이었다.
[대리 생성자](#Rc-delegating)와 [기본 멤버 초기화](#Rc-in-class-initializer)가 이런 작업을 더 잘 해낼 수 있다.

또 다른 이유로는 객체가 필요할 때까지 초기화를 지연시키는 것이다; 이러한 해법은 보통 [변수가 적절하게 초기화되기 전까지는 해당 변수를 선언하지 않는 것이다](#Res-init). 


##### 시행하기
* (쉬움) 모든 생성자는 모든 멤버 변수를 초기화 해야 한다. (명시적으로든, 생성자를 호출하도록 위임하든, 또는 기본 생성자를 통해서라도)
* (알수없음)  생성자가 `Ensures` 계약을 갖고 있다면, 그 계약이 사후 조건인지 확인하라.


### <a name="Rc-default0"></a>C.43: 클래스가 기본 생성자를 갖도록 하라

##### 근거
많은 언어나 라이브러리들이 기본 생성자를 필요로 한다.  
예를 들면, `T a[10]` 나 `std::vector<T> v(10)` 는 기본 생성자들이 각 요소를 초기화 한다.

##### 잘못된 예
```c++
    class Date { // BAD: 기본 생성자가 없다
    public:
        Date(int dd, int mm, int yyyy);
        // ...
    };

    vector<Date> vd1(1000);   // Date의 기본 값이 필요하다

    // 대안: 기본값 제공하기
    vector<Date> vd2(1000, Date{ Month::october, 7, 1885 });   
```
기본 생성자는 다른 사용자 정의 생성자가 없을 때만 자동으로 생성된다. 이런 코드와 같은 경우엔 `vd1`을 초기화 하는 것은 불가능하다.

자연스러은 기본 날짜는 없다, 때문에 이 예는 사소하지 않다. (대부분의 사람들에게 태초의 시간은 필요없다)
대부분의 달력 시스템에서 `{0,0,0}` 은 유효한 날짜가 아니다. 이것은 부동 소수점의 `NaN` 같은 것을 만드는 것이다. 
하지만, 대부분의 현실적인 `Date` 클래스는 "첫째 날" (가령. 1970년 1월 1일이 많이 쓰인다)을 갖기 때문에 이것을 기본으로 사용하는 것이 일반적이다.


##### 예
```c++
    class Date {
    public:
        Date(int dd, int mm, int yyyy);
        Date() = default; // 함께 보기 : C.45 
        // ...
    private:
        int dd = 1;
        int mm = 1;
        int yyyy = 1970;
        // ...
    };

    vector<Date> vd1(1000);
```

##### 참고 사항
클래스의 모든 멤버들이 기본 생성자들을 가지고 있을 경우 묵시적으로 기본 생성자를 가진다 : 
```c++
    struct X {
        string s;
        vector v;
    };

    X x; // X{{}, {}}를 의미한다; 빈 string과 빈 vector를 생성한다
```

기본 내장(built-in) 타입들은 적절하게 기본 생성되지 않을 수도 있다:
```c++
    struct X {
       string s;
       int i;
    };

    void f()
    {
       // x.s 은 빈 string로 초기화 되었다 
       // x.i 은 초기화되지 않았다
       X x;    

       cout << x.s << ' ' << x.i << '\n';
       ++x.i;
    }
```

정적으로 할당된 내장 타입 객체들은 `0`으로 초기화 된다. 하지만 지역 변수들은 그렇지 않다.  
컴파일러의 최적화 빌드는 내장 타입 지역 변수들을 초기화하지 않을 수 있다는 점에 주의하라. 따라서, 위의 예시와 같은 코드가 나타난다면, 미정의 동작을 일으킬 수 있다.

초기화를 하고자 한다면, 명시적 기본 생성이 도움이 될 것이다:

```c++
    struct X {
       string s;
       int i {};   // 기본 초기화 (i는 0 이 된다)
    };
```

##### 시행하기
* 기본 생성자가 없는 클래스들에 표시를 남겨라.


### <a name="Rc-default00"></a>C.44: 기본 생성자는 되도록 단순하고 예외를 던지지 않도록 하라

##### 근거
실패할 수 있는 연산없이 "기본"적인 값을 설정할 수 있다는 것은 에러 처리를 단순화 하고, 이동 연산을 추측 할 수 있도록 한다.

##### 잘못된 예
```c++
    // elem은 공간에 대한 포인터다 - new를 사용해 원소들이 할당된다.
    template<typename T>
    class Vector0 {
    public:
        Vector0() : Vector0{0} {}
        Vector0(int n) :
            elem{new T[n]}, space{elem + n}, last{elem} {}
        // ...
    private:
        own<T*> elem;
        T* space;
        T* last;
    };
```
이것은 일반적이지만, 에러 이후 `Vector0` 를 공백으로 만드는 것은 할당과 관련이 있고, 실패할 수 있다.
또, 기본 `Vector` 를 `{ new T[0], 0, 0}` 으로 표현하는 것 역시 낭비처럼 보인다

예를 들면, `Vector0 v(100)`은 100 만큼 할당하는 비용이 든다.

##### 예
```c++
    // elem은 nullptr이거나, new를 사용해 할당된 공간을 가리킨다.
    template<typename T>
    class Vector1 {
    public:
        // {nullptr, nullptr, nullptr}과 동일하다. 
        // 예외를 던지지 않는다.
        Vector1() noexcept {}

        Vector1(int n) : 
            elem{new T[n]}, space{elem + n}, last{elem} {}
        // ...
    private:
        own<T*> elem = nullptr;
        T* space = nullptr;
        T* last = nullptr;
    };
```
`{nullptr, nullptr, nullptr}`는 `Vector1{}` 를 만드는 비용을 줄여준다(cheap). 하지만 이는 특별한 경우이고 실행시간 평가가 필요하다.
에러를 발견하고 `Vector1`를 비우는 것은 간단하다.

##### 시행하기
* 예외를 던지는 기본 생성자에는 표시를 남겨라.


### <a name="Rc-default"></a>C.45: 멤버를 초기화 하기만 하는 기본 생성자는 정의하지 마라; 대신 멤버들이 스스로 초기화 하도록 하라

##### 근거
멤버들에게 초기화를 위임하면, 컴파일러가 효율적인 코드를 생성한다.

##### 잘못된 예
```c++
    class X1 { // BAD: 멤버 초기화를 사용하지 않는다
        string s;
        int i;
    public:
        X1() :s{"default"}, i{1} { }
        // ...
    };
```
##### 예
```
    class X2 {
        string s = "default";
        int i = 1;
    public:
        // 컴파일러가 생성한 기본 생성자를 사용한다.
        // ...
    };
```
##### 시행하기

(쉬움) 명시적인 기본 생성자는 초기화 이외의 동작을 해야할 때 쓰는 것이 좋다.


### <a name="Rc-explicit"></a>C.46: By default, declare single-argument constructors explicit

##### 근거

To avoid unintended conversions.

##### 잘못된 예
```c++
    class String {
        // ...
    public:
        String(int);   // BAD
        // ...
    };

    String s = 10;   // surprise: string of size 10
```
##### 예외 사항

If you really want an implicit conversion from the constructor argument type to the class type, don't use `explicit`:
```c++
    class Complex {
        // ...
    public:
        Complex(double d);   // OK: we want a conversion from d to {d, 0}
        // ...
    };

    Complex z = 10.7;   // unsurprising conversion
```
##### 함께 보기
[Discussion of implicit conversions](#Ro-conversion).

##### 시행하기

(쉬움) Single-argument constructors should be declared `explicit`. Good single argument non-`explicit` constructors are rare in most code based. Warn for all that are not on a "positive list".


### <a name="Rc-order"></a>C.47: Define and initialize member variables in the order of member declaration

##### 근거

To minimize confusion and errors. That is the order in which the initialization happens (independent of the order of member initializers).

##### 잘못된 예
```c++
    class Foo {
        int m1;
        int m2;
    public:
        Foo(int x) :m2{x}, m1{++x} { }   // BAD: misleading initializer order
        // ...
    };

    Foo x(1); // surprise: x.m1 == x.m2 == 2
```
##### 시행하기

(쉬움) A member initializer list should mention the members in the same order they are declared.

##### 함께 보기
[Discussion](#Sd-order)


### <a name="Rc-in-class-initializer"></a>C.48: Prefer in-class initializers to member initializers in constructors for constant initializers

##### 근거

Makes it explicit that the same value is expected to be used in all constructors. Avoids repetition. Avoids maintenance problems. It leads to the shortest and most efficient code.

##### 잘못된 예
```c++
    class X {   // BAD
        int i;
        string s;
        int j;
    public:
        X() :i{666}, s{"qqq"} { }   // j is uninitialized
        X(int ii) :i{ii} {}         // s is "" and j is uninitialized
        // ...
    };
```
How would a maintainer know whether `j` was deliberately uninitialized (probably a poor idea anyway) and whether it was intentional to give `s` the default value `""` in one case and `qqq` in another (almost certainly a bug)? The problem with `j` (forgetting to initialize a member) often happens when a new member is added to an existing class.

##### 예
```c++
    class X2 {
        int i {666};
        string s {"qqq"};
        int j {0};
    public:
        X2() = default;        // all members are initialized to their defaults
        X2(int ii) :i{ii} {}   // s and j initialized to their defaults
        // ...
    };
```
##### 대안
We can get part of the benefits from default arguments to constructors, and that is not uncommon in older code. However, that is less explicit, causes more arguments to be passed, and is repetitive when there is more than one constructor:
```c++
    class X3 {   // BAD: inexplicit, argument passing overhead
        int i;
        string s;
        int j;
    public:
        X3(int ii = 666, const string& ss = "qqq", int jj = 0)
            :i{ii}, s{ss}, j{jj} { }   // all members are initialized to their defaults
        // ...
    };
```
##### 시행하기

* (쉬움) Every constructor should initialize every member variable (either explicitly, via a delegating ctor call or via default construction).
* (쉬움) Default arguments to constructors suggest an in-class initializer may be more appropriate.


### <a name="Rc-initialize"></a>C.49: Prefer initialization to assignment in constructors

##### 근거

An initialization explicitly states that initialization, rather than assignment, is done and can be more elegant and efficient. Prevents "use before set" errors.

##### 좋은 예
```c++
    class A {   // Good
        string s1;
    public:
        A() : s1{"Hello, "} { }    // GOOD: directly construct
        // ...
    };
```
##### 잘못된 예
```c++
    class B {   // BAD
        string s1;
    public:
        B() { s1 = "Hello, "; }   // BAD: default constructor followed by assignment
        // ...
    };

    class C {   // UGLY, aka very bad
        int* p;
    public:
        C() { cout << *p; p = new int{10}; }   // accidental use before initialized
        // ...
    };
```
### <a name="Rc-factory"></a>C.50: Use a factory function if you need "virtual behavior" during initialization

##### 근거

If the state of a base class object must depend on the state of a derived part of the object, we need to use a virtual function (or equivalent) while minimizing the window of opportunity to misuse an imperfectly constructed object.

##### 잘못된 예
```c++
    class B {
    public:
        B()
        {
            // ...
            f();   // BAD: virtual call in constructor
            // ...
        }

        virtual void f() = 0;

        // ...
    };
```
##### 예
```c++
    class B {
    protected:
        B() { /* ... */ }              // create an imperfectly initialized object

        virtual void PostInitialize()  // to be called right after construction
        {
            // ...
            f();    // GOOD: virtual dispatch is safe
            // ...
        }

    public:
        virtual void f() = 0;

        template<class T>
        static shared_ptr<T> Create()  // interface for creating objects
        {
            auto p = make_shared<T>();
            p->PostInitialize();
            return p;
        }
    };

    class D : public B { /* ... */ };            // some derived class

    shared_ptr<D> p = D::Create<D>();  // creating a D object
```
By making the constructor `protected` we avoid an incompletely constructed object escaping into the wild.
By providing the factory function `Create()`, we make construction (on the free store) convenient.

##### 참고 사항

Conventional factory functions allocate on the free store, rather than on the stack or in an enclosing object.

##### 함께 보기
[Discussion](#Sd-factory)

### <a name="Rc-delegating"></a>C.51: Use delegating constructors to represent common actions for all constructors of a class

##### 근거

To avoid repetition and accidental differences.

##### 잘못된 예
```c++
    class Date {   // BAD: repetitive
        int d;
        Month m;
        int y;
    public:
        Date(int ii, Month mm, year yy)
            :i{ii}, m{mm} y{yy}
            { if (!valid(i, m, y)) throw Bad_date{}; }

        Date(int ii, Month mm)
            :i{ii}, m{mm} y{current_year()}
            { if (!valid(i, m, y)) throw Bad_date{}; }
        // ...
    };
```
The common action gets tedious to write and may accidentally not be common.

##### 예
```c++
    class Date2 {
        int d;
        Month m;
        int y;
    public:
        Date2(int ii, Month mm, year yy)
            :i{ii}, m{mm} y{yy}
            { if (!valid(i, m, y)) throw Bad_date{}; }

        Date2(int ii, Month mm)
            :Date2{ii, mm, current_year()} {}
        // ...
    };
```
##### 함께 보기
If the "repeated action" is a simple initialization, consider [an in-class member initializer](#Rc-in-class-initializer).

##### 시행하기

(중간) Look for similar constructor bodies.

### <a name="Rc-inheriting"></a>C.52: Use inheriting constructors to import constructors into a derived class that does not need further explicit initialization

##### 근거

If you need those constructors for a derived class, re-implementing them is tedious and error prone.

##### 예

`std::vector` has a lot of tricky constructors, so if I want my own `vector`, I don't want to reimplement them:
```c++
    class Rec {
        // ... data and lots of nice constructors ...
    };

    class Oper : public Rec {
        using Rec::Rec;
        // ... no data members ...
        // ... lots of nice utility functions ...
    };
```
##### 잘못된 예
```c++
    struct Rec2 : public Rec {
        int x;
        using Rec::Rec;
    };

    Rec2 r {"foo", 7};
    int val = r.x;   // uninitialized
```
##### 시행하기

Make sure that every member of the derived class is initialized.

## <a name="SS-copy"></a>C.copy: 복사와 이동
값 타입들은 일반적으로 복사 가능해야 한다. 하지만 클래스 계층에서의 인터페이스들은 그렇지 않아야 한다.  
리소스 핸들의 경우, 복사가 가능할 수도, 그렇지 않을 수도 있다.  
타입들은 논리적인 또는 성능 상의 이유로 이동하도록 정의될 수 있다. 


### <a name="Rc-copy-assignment"></a>C.60: 복사연산을 `virtual`로 만들지 말아라. 매개변수는 `const&`로 받고, `const&`로 반환하지 말아라

##### 근거
이렇게 하는 것이 간단하고 효율적이다. r-value를 위해 최적화하길 원한다면, `&&`를 받는 대입 연산을 오버로드하여 제공하라. ([F.24](#Rf-pass-ref-ref)를 참조하라)

##### 예
```c++
    class Foo {
    public:
        Foo& operator=(const Foo& x)
        {
            // GOOD: 자기대입 검사를 할 필요가 없다. (성능은 어찌되었든)
            auto tmp = x;
            std::swap(*this, tmp);
            return *this;
        }
        // ...
    };

    Foo a;
    Foo b;
    Foo f();

    a = b;    // l-value 대입 : 복사
    a = f();  // r-value 대입 : 이동일수도 있다
```
##### 참고 사항
`swap`함수의 구현은 [강한 예외 안전성 보장](???)을 가능하게 한다.

##### 예
하지만 만약 임시 사본을 만들지 않음으로써 훨씬 더 좋은 성능을 얻을 수 있다면 어떨까? 
크고 같은 크기의 `Vector`들의 대입이 빈번한 영역을 위한 간단한 `Vector`를 생각해보라.  
이 경우, `swap`구현 기법에 의한 원소들의 사본은 상당한 비용 증가를 야기할 수 있다.
```c++
    template<typename T>
    class Vector {
    public:
        Vector& operator=(const Vector&);
        // ...
    private:
        T* elem;
        int sz;
    };

    Vector& Vector::operator=(const Vector& a)
    {
        if (a.sz > sz) {
			// ... swap함수 기법을 사용한다. 이러면 최상의 구현이 된다 ...
            return *this
        }
		// ... *a.elem으로부터 elem으로 sz만큼 원소들을 복사한다 ...
        if (a.sz < sz) {
            // ... destroy the surplus elements in *this* and adjust size ...
        }
        return *this;
    }
```
대상 원소들에 직접 쓰기 연산을 함으로써, `swap`기법이 제공하는 강한 예외 보장 대신 [기본적인 예외 보장](#???)만 얻게 될 것이다.  
[자기 대입](#Rc-copy-self)에 주의하라.


##### 대안s
만약 당신이 `virtual` 대입 연산자가 필요하다고 생각한다면, 그리고 어째서 그것이 문제를 야기할 수 있는지 이해한다면, 그 함수는 `operator=`라고 부르지 마라. 이름을 부여해서 `virtual void assign(const Foo&)`로 만들어라. 
[복사 생성 vs. `clone()`](#Rc-copy-virtual)를 참조하라. 


##### 시행하기
* (쉬움) 대입 연산자는 가상함수여서는 안된다. 드래곤들만큼 위험하다!
* (쉬움) 대입 연산자는 `T&`를 반환하면 안된다. 연쇄적인 호출을 위해선, 컨테이너로의 객체 대입과 코드 작성을 방해하는 `const T&`를 사용하지 말아라.
* (중간) 대입 연산자는 (암시적으로나 명시적으로나) 모든 기본 클래스와 멤버들의 대입 연산자를 호출해야 한다.  
해당 타입이 포인터 문맥이나 값 문맥을 가지는지 확인하기 위해 소멸자를 확인하라. 


### <a name="Rc-copy-semantic"></a>C.61: 복사 연산은 복사를 수행해야 한다.

##### 근거
그렇게 하는 것이 일반적으로 생각되는 의미론이다. `x = y`가 수행된 후에는, `x == y`인 결과를 가져야 한다.
복사 후에는 `x`와 `y`가 독립적인 객체들일 수 있다. (값 의미론, 비-포인터 빌트인 타입들과 표준 라이브러리 타입들의 동작하는 방식) 또는 공유된 객체를 참조한다(포인터 의미론, 포인터들이 동작하는 방식).


##### 예
```c++
    class X {   // OK:  값 의미론
    public:
        X();
        X(const X&);     // X를 복사한다
        void modify();   // X의 값을 변경한다
        // ...
        ~X() { delete[] p; }
    private:
        T* p;
        int sz;
    };

    bool operator==(const X& a, const X& b)
    {
        return a.sz == b.sz && equal(a.p, a.p + a.sz, b.p, b.p + b.sz);
    }

    X::X(const X& a)
        :p{new T[a.sz]}, sz{a.sz}
    {
        copy(a.p, a.p + sz, a.p);
    }

    X x;
    X y = x;
    if (x != y) throw Bad{};
    x.modify();
    if (x == y) throw Bad{};   // 값 의미론으로 가정한다
```
##### 예
```c++
    class X2 {  // OK: 포인터 의미론
    public:
        X2();
        X2(const X&) = default; // 얕은 복사
        ~X2() = default;
        void modify();          // X의 값을 변경한다
        // ...
    private:
        T* p;
        int sz;
    };

    bool operator==(const X2& a, const X2& b)
    {
        return a.sz == b.sz && a.p == b.p;
    }

    X2 x;
    X2 y = x;
    if (x != y) throw Bad{};
    x.modify();
    if (x != y) throw Bad{};  // 포인터 의미론으로 가정한다
```
##### 참고 사항
"스마트 포인터"를 만들고 있지 않다면 복사 의미론을 선호하라. 값 의미론은 가장 간단하며, 표준 라이브러리의 기능들이 기대하는 것이다.   

##### 시행하기
(특별히 없음)


### <a name="Rc-copy-self"></a>C.62: 복사 연산은 자기 대입에 안전하게 작성하라


##### 근거
`x = x`의 수행이 `x`의 값을 바꾼다면, 사람들은 놀랄 것이며 안좋은 에러들이 발생할 수 있다 (종종 자원 누수를 포함하기도 한다).

##### 예
표준 라이브러리 컨테이너들은 자기 대입을 우아하고 효율적인 방법으로 처리한다.

```c++
    std::vector<int> v = {3, 1, 4, 1, 5, 9};
    v = v;
    // v의 값은 여전히 {3, 1, 4, 1, 5, 9} 그대로다
```
##### 참고 사항
멤버들로부터 생성된 기본 대입 연산은 자기 대입에 안전하다.

```c++
    struct Bar {
        vector<pair<int, int>> v;
        map<string, int> m;
        string s;
    };

    Bar b;
    // ...
    b = b;   // 정확하고, 효율적이다
```
##### 참고 사항
자기 대입을 명시적으로 검사함으로써 처리할 수도 있을 것이다. 하지만 종종 그런 검사 없이도 우아하고 빠르게 동작하도록 할 수 있다 (가령, [`swap` 사용법](#Rc-swap)).

```c++
    class Foo {
        string s;
        int i;
    public:
        Foo& operator=(const Foo& a);
        // ...
    };

    // OK, 하지만 비용이 든다
    Foo& Foo::operator=(const Foo& a)   
    {
        if (this == &a) return *this;
        s = a.s;
        i = a.i;
        return *this;
    }
```
이 방법은 분명 안전하고 효율적이다.
하지만, 만약 백만번 마다 한번씩 자기 대입을 한다면 어떻겠는가?  
그 말은 백만번이나 장황한 검사를해야 한다는 것과 같다 (하지만 자기 대입의 결과는 반드시 자신과 같아야 하기 때문에, 컴퓨터의 분기 예측은 매번 맞아떨어질 것이다.  
이런 코드를 고려해보자 :

```c++
    // 간단하고, 아마도 훨씬 나을 것이다.
    Foo& Foo::operator=(const Foo& a)   
    {
        s = a.s;
        i = a.i;
        return *this;
    }
```
`std::string`은 자기 대입에 안전하고, `int` 역시 안전하다. (희소하게 발생하는) 자기 대입에 대해서만 비용이 발생하게 된다. 

##### 시행하기
* (쉬움) 대입 연산자들은 `if (this == &a) return *this;`와 같은 패턴이 있어선 안된다.  
???

### <a name="Rc-move-assignment"></a>C.63: 이동 연산은 `virtual`로 만들지 말아라, 매개변수는 `&&`를 사용하고, `const&`로 반환하지 말아라

##### 근거
간단하고, 효율적이다. 

##### 함께 보기
[복사 대입을 위한 규칙들](#Rc-copy-assignment).

##### 시행하기
[복사 대입](#Rc-copy-assignment)에서와 동일하다.  
* (쉬움) 대입 연산자는 가상 함수여서는 안된다. 드래곤들만큼 위험하다!
* (쉬움) 대입 연산자는 `T&`를 반환하면 안된다. 연쇄적인 호출을 위해선, 컨테이너로의 객체 대입과 코드 작성을 방해하는 `const T&`를 사용하지 말아라.
* (중간) 이동 연산자는 (암시적으로나 명시적으로나) 모든 기본 클래스와 멤버들의 이동 연산자를 호출해야 한다.  


### <a name="Rc-move-semantic"></a>C.64: 이동 연산은 이동을 수행해야 하며, 원본 객체를 유효한 상태로 남겨놓아야 한다

##### 근거
그것이 일반적으로 기대되는 동작(semantics)이다.  `x = std::move(y)`를 수행한 후에는, `x`의 값은 `y`여야 하며, `y`는 유효한 상태여야 한다.

##### 예
```c++
    template<typename T>
    class X {   // OK: 값 의미론
    public:
        X();
        X(X&& a);          // X를 이동한다
        void modify();     // X의 값을 변경한다
        // ...
        ~X() { delete[] p; }
    private:
        T* p;
        int sz;
    };


    X::X(X&& a)
        :p{a.p}, sz{a.sz}  // 값을 가져간다
    {
        a.p = nullptr;     // empty 상태가 된다
        a.sz = 0;
    }

    void use()
    {
        X x{};
        // ...
        X y = std::move(x);
        x = X{};   // OK
    } // OK: x 는 소멸 가능하다
```
##### 참고 사항
이상적으로는, 이동연산을 해준 객체는 해당 타입의 기본 값이어야 한다. 그렇지 않아야 하는 이유가 있지 않는한 기본 값을 가지도록 확실히 하라. 하지만, 모든 타입들이 기본 값을 가지는 것은 아니며, 또 일부 타입들에서는 기본 값을 만드는 것이 비싼 비용을 필요로 할 수도 있다. 표준에서 요구하는 것은, 이동연산을 해준 객체가 파괴될 수 있다는 것뿐이다.  
종종, 쉽고 비용이 들지 않는 방법을 쓸수도 있다 : 표준 라이브러리는 객체로부터 이동을 받을 수 있다고 가정한다. 이동을 해주는 객체는 유효한 상태로 (필요하다면 명시하여) 남겨놓아라.  

##### 참고 사항
이 가이드라인을 적용하지 않아야 할 예외적인 이유가 있지 않는 한, `x = std::move(y); y = z;`를 사용하라. 전통적인 의미론에 부합한다.


##### 시행하기
(자유선택) 이동 연산에서 멤버들의 대입을 확인해보라. 기본 생성자가 있다면, 그 대입 연산들을 기본 생성자를 사용한 초기화와 비교해보라.  


### <a name="Rc-move-self"></a>C.65: 이동 연산은 자기 대입에 안전하게 작성하라

##### 근거
만약 `x = x`가 `x`의 값을 바꾼다면, 사람들은 놀랄 것이고 안좋은 에러들이 발생할 수 있다. 사람들은 주로 자기 대입을 이동연산으로 작성하지 않지만, 그럴 수도 있다. 가령, `std::swap`은 이동 연산들로 구현되었고 만약 당신이 우연히  `a`와 `b`가 같은 객체를 참조하는 상황에서 `swap(a, b)`를 사용한다면, 자기-이동의 실패는 심각하거나 찾기 어려운(subtle) 에러가 될 수 있다.


##### 예
```c++
    class Foo {
        string s;
        int i;
    public:
        Foo& operator=(Foo&& a);
        // ...
    };

    Foo& Foo::operator=(Foo&& a)       // OK, 하지만 비용이 든다
    {
        if (this == &a) return *this;  // 이 라인은 무의미하다
        s = std::move(a.s);
        i = a.i;
        return *this;
    }
```
백만번에 한번 발생하는 `if (this == &a) return *this;`에 대한 논쟁이 있다. [자기 대입](#Rc-copy-self)에서 논의한 검사에 대한 이야기는 자기 이동에 더 관련이 있다. 


##### 참고 사항
`if (this == &a) return *this;`을 쓰지 않는 방법은 알려진 것이 없다. 이동 대입 연산에서 검사를 수행하고 정확한 결과를 얻으라.(가령, `x=x`를 수행한 뒤에 `x`가 변화하지 않는다.)  

##### 참고 사항
ISO 표준은 표준 라이브러리 컨테이너들에 대해 오직 "유효하지만 명시되지는 않은" 상태만을 보장한다. 이것은 10여년간의 실험적인 사용이나 상용 환경에서 문제가 되지 않았다. 만약 반례를 찾게 된다면 작성자에게 연락하라. 이 규칙은 주의를 필요로 하며 완전히 안전해야 한다.

##### 예
여기 검사 없이 포인터를 이동하는 방법이 있다.(마치 이동 대입을 구현한 코드라고 상상해보라.):
```c++
    // move from other.ptr to this->ptr
    T* temp = other.ptr;
    other.ptr = nullptr;
    delete ptr;
    ptr = temp;
```
##### 시행하기
* (중간) 이러한 자기 대입의 경우, 이동 대입 연산자는 대입 받는 객체의 포인터 멤버를 `delete`된 상태 또는 `nullptr`로 남겨놓아서는 안된다.
* (자유선택) 표준 라이브러리 컨테이너들의 사용법을 보라(`string`을 포함한다). 그리고 일반적인(객체 수명에 민감하지 않은) 사용에 그 컨테이너들이 안전하다고 생각하라. 


### <a name="Rc-move-noexcept"></a>C.66: 이동 연산은 `noexcept`로 만들어라

##### 근거
예외를 던지는 이동 연산은 대다수의 사람들의 타당한 가정을 무너뜨린다.
예외를 던지지 않는 이동은 표준 라이브러리와 언어 특징들에 의해 더 효율적으로 사용될 수 있다. 

##### 예
```c++
    template<typename T>
    class Vector {
        // ...
        Vector(Vector&& a) noexcept :
            elem{a.elem}, sz{a.sz} 
        { 
            a.sz = 0; 
            a.elem = nullptr; 
        }
        
        Vector& operator=(Vector&& a) noexcept { 
            elem = a.elem; 
            sz = a.sz; 
            a.sz = 0; 
            a.elem = nullptr; 
        }
        // ...
    public:
        T* elem;
        int sz;
    };
```
이 복사 연산들은 예외를 던지지 않는다.

##### 잘못된 예
```c++
    template<typename T>
    class Vector2 {
        // ...
        Vector2(Vector2&& a) { *this = a; }             // 그냥 복사 연산
        Vector2& operator=(Vector2&& a) { *this = a; }  // 그냥 복사 연산
        // ...
    public:
        T* elem;
        int sz;
    };
```
이 `Vector2`는 비 효율적일 뿐만 아니라, 벡터가 메모리 할당을 요구하기 때문에 예외를 던질 수 있다. 

##### 시행하기
(쉬움) 이동연산은 `noexcept`로 표시되어야 한다.


### <a name="Rc-copy-virtual"></a>C.67: 기본 클래스에 대한 복사를 제한하라, 대신 복사가 필요하다면 가상 `clone`함수를 제공하라


##### 근거
복사손실(slicing)을 피하기 위함이다. 일반적인 복사 연산은 파생 클래스 객체에서 기본 클래스 부분만 복사할 것이다. 

##### 잘못된 예
```c++
    class B { // BAD: 기본 클래스가 복사를 제한하지 않는다
        int data;
        // ... 복사 연산에 대한 정의가 없으므로, 기본 동작을 사용한다 ...
    };

    class D : public B {
        string moredata; // 데이터 멤버가 추가되었다
        // ...
    };

    auto d = make_unique<D>();

    // 이런, 객체가 절단된다; 
    // d.moredata를 잃어버리고 d.data만 가지게 된다.
    auto b = make_unique<B>(d);
```

##### 예
```c++
    class B { // GOOD: 기본 클래스가 복사를 제한한다
        B(const B&) = delete;
        B& operator=(const B&) = delete;
        virtual unique_ptr<B> clone() { return /* B object */; }
        // ...
    };

    class D : public B {
        string moredata; // 데이터 멤버가 추가되었다
        unique_ptr<B> clone() override { return /* D object */; }
        // ...
    };

    auto d = make_unique<D>();
    auto b = d.clone(); // OK, 깊은 복사
```
##### 참고 사항
스마트 포인터를 반환하는 것이 좋다. 하지만 날 포인터(raw pointer)와 달리 반환 타입이 공변적이지 않다. (예를 들면, `D::clone`함수는 `unique_ptr<D>`를 반환할 수 없다.)
Don't let this tempt you into returning an owning raw pointer; this is a minor drawback compared to the major robustness benefit delivered by the owning smart pointer.

##### 예외 사항
공변성 있는 반환 타입이 필요하다면, `owner<derived*>`를 반환하라. [C.130](#Rh-copy)을 보라.

##### 시행하기
가상 함수를 가진 클래스는 (컴파일러가 생성하였거나 프로그래머가 작성한) 복사 생성자나 복사 대입 연산자가 없어야 한다. 


## C.other: 다른 기본 연산 규칙들
언어가 제공하는 기본 구현 연산들에 더해서, 
정의가 필요할 정도로 기초적인 몇몇 연산들이 있다:
비교, `swap`, 그리고 `hash`


### <a name="Rc-default"></a>C.80: 기본 의미론을 명시적으로 사용하려면 `=default` 키워드를 사용하라 


##### 근거
컴파일러가 더 정확한 기본 의미론을 알고 있으며, 이보다 나은 코드를 작성할 수 없다. 


##### 예
```c++
    class Tracer {
        string message;
    public:
        Tracer(const string& m) : message{m} { 
            cerr << "entering " << message << '\n'; 
        }
        ~Tracer() { 
            cerr << "exiting " << message << '\n'; 
        }

        Tracer(const Tracer&) = default;
        Tracer& operator=(const Tracer&) = default;
        Tracer(Tracer&&) = default;
        Tracer& operator=(Tracer&&) = default;
    };
```
소멸자를 정의했기 때문에, 우리는 복사, 이동 연산들을 정의해야만 한다. 이를 위해선 `=default`가 가장 최선이고, 간단한 방법이다.  

##### 잘못된 예
```c++
    class Tracer2 {
        string message;
    public:
        Tracer2(const string& m) : message{m} {
            cerr << "entering " << message << '\n'; 
        }
        ~Tracer2() { 
            cerr << "exiting " << message << '\n'; 
        }

        Tracer2(const Tracer2& a) : message{a.message} {}
        Tracer2& operator=(const Tracer2& a) { message = a.message; }
        Tracer2(Tracer2&& a) :message{a.message} {}
        Tracer2& operator=(Tracer2&& a) { message = a.message; }
    };
```
복사와 이동 연산들의 함수들을 일일이 작성하는 것은 번거롭고, 지루하며, 에러에 취약하다. 컴파일러가 이 작업을 더 잘 할수있다.


##### 시행하기
* (중간) 특별한 연산들은 중복성을 피하기 위해 컴파일러가 만든 함수들과 같은 접근성, 의미론을 가져서는 안된다.  


### <a name="Rc-delete"></a>C.81: C.81: 기본 동작을 (대안을 원하지 않고) 금지하고 싶다면 `=delete`를 사용하라

##### 근거
드물게 기본 연산들이 바람직하지 않은 경우도 있다.


##### 예
```c++
    class Immortal {
    public:
        // 소멸이 금지되었다
        ~Immortal() = delete;   
        // ...
    };

    void use()
    {
        Immortal ugh;   // error: ugh은 소멸될 수 없다
        Immortal* p = new Immortal{};
        delete p;       // error: *p를 소멸시킬 수 없다
    }
```
##### 예
`unique_ptr`는 이동 가능하지만, 복사는 불가능하다. 이 클래스의 복사를 막기 위해, 복사 연산들은 삭제되었다. l-value로부터 복사 연산을 막기 위해서는 `=delete`가 필요하다:
```c++
    template <class T, 
              class D = default_delete<T>> 
    class unique_ptr {
    public:
        // ...
        constexpr unique_ptr() noexcept;
        explicit unique_ptr(pointer p) noexcept;
        // ...
        // 이동 생성자
        unique_ptr(unique_ptr&& u) noexcept;   
        // ...
        // l-value 복사를 금지한다
        unique_ptr(const unique_ptr&) = delete; 
        // ...
    };

    unique_ptr<int> make();   // "무엇인가" 만든 뒤에 이동으로 반환한다

    void f()
    {
        unique_ptr<int> pi {};
        auto pi2 {pi};      // error: l-value로 생성할 수 없다.
        auto pi3 {make()};  // OK, 이동 생성: make()의 결과는 r-value이다
    }
```
##### 시행하기
기본 연산을 제거하는 것은 해당 클래스에 부합하는 근거가 있어야 한다. 
정말 이유가 있는지 의심하라. 
하지만 사람이 보기에 문맥적으로 타당하다고 단언할 수 있도록 하라.   


### <a name="Rc-ctor-virtual"></a>C.82: 생성자 또는 소멸자에서 가상 함수를 호출하지 말아라


##### 근거
호출된 함수는 파생 클래스에서 오버라이드 하는 함수가 아니라, 생성된 객체의 함수이다. 
이러한 동작은 혼란을 일으킬 수 있다. 
나쁘게는, 생성자와 소멸자 내부에서 발생하는 구현되지 않은 순수 가상 함수에 대한 직접 또는 간접호출이 비정의된 동작을 일으킨다.  

##### 잘못된 예
```c++
    class base {
    public:
        virtual void f() = 0;   // 구현되지 않았다
        virtual void g();       // 기본 버전을 구현하였다
        virtual void h();       // 기본 버전을 구현하였다
    };

    class derived : public base {
    public:
        void g() override;   // 파생 구현을 제공한다
        void h() final;      // 파생 구현을 제공한다

        derived()
        {
            // BAD: 구현되지 않은 가상 함수를 호출한다
            f();

            // BAD: will call derived::g, not dispatch further virtually
            g();

            // GOOD: 유효 범위의(visible) 함수를 명시적으로 호출한다
            derived::g();

            // ok, 문제 없다. h함수는 final이다
            h();
        }
    };
```
특정하게 명시적으로 한정된 함수는 `virtual`로 선언되었다고 하더라도 가상호출이 발생하지 않음을 기억하라.


##### 함께 보기
정의되지 않은 동작의 위험이 없이 파생 클래스의 함수를 호출하는 효과를 얻기 위해서는 [팩토리 함수들](#Rc-factory) 참고하라. 


##### 참고 사항
There is nothing inherently wrong with calling virtual functions from constructors and destructors.
The semantics of such calls is type safe.
However, experience shows that such calls are rarely needed, easily confuse maintainers, and become a source of errors when used by novices.

##### 시행하기
* 생성자와 소멸자에서의 가상 함수 호출에는 표시를 남겨라.




### <a name="Rc-swap"></a>C.83: 값 형식 타입들에는, `noexcept` swap함수를 제공하는 것을 고려하라.

##### 근거
`swap`함수는 객체 대입을 구현할 때 원활하게 객체를 이동하는 것에서, 에러가 발생하지 않는 것을 보장하는 함수를 제공하는 것까지 몇몇 함수들(idioms)을 구현하는데 유용하다. swap함수을 이용해서 복사 대입을 구현하는 것을 고려하라. [소멸자, 자원해제, 그리고 swap은 실패해선 안된다]("#Re-never-fail)를 확인하라.


##### 좋은 예
```c++
    class Foo {
        // ...
    public:
        void swap(Foo& rhs) noexcept
        {
            m1.swap(rhs.m1);
            std::swap(m2, rhs.m2);
        }
    private:
        Bar m1;
        int m2;
    };
```
호출자들의 편의를 위해서 같은 네임스페이스에 비-멤버 `swap`함수를 제공하라.

```c++
    void swap(Foo& a, Foo& b)
    {
        a.swap(b);
    }
```
##### 시행하기
* (쉬움) 가상 함수들이 없는 클래스는 `swap`멤버 함수 선언이 있어야 한다. 
* (쉬움) 클래스가 `swap` 멤버함수를 가지고 있다면, 그 함수는 `noexcept`로 선언되어야 한다.



### <a name="Rc-swap-fail"></a>C.84: `swap`연산은 실패하지 않도록 작성하라

##### 근거
`swap`연산은 많은 경우 실패하지 않을 것으로 전제하고 사용된다. 또한 실패 가능성이 있는 `swap`연산으로는 정확하게 동작하도록 프로그램이 작성되기 어렵다. 표준 라이브러리의 컨테이너들과 알고리즘들은 swap연산의 타입이 실패하면 정확하게 동작하지 않을 것이다.  


##### 잘못된 예
```c++
    void swap(My_vector& x, My_vector& y)
    {
        auto tmp = x;   // copy elements
        x = y;
        y = tmp;
    }
```
이 경우는 느릴 뿐만 아니라, `tmp`내의 원소들에 메모리 할당이 발생하면, 이 `swap` 연산은 예외를 던지고 이를 사용하는 STL 알고리즘들이 실패할 수 있다.

##### 시행하기
* (쉬움) 클래스에 `swap` 멤버 함수가 있으면, `noexcept`로 선언되어야 한다.


### <a name="Rc-swap-noexcept"></a>C.85: `swap`연산은 `noexcept`로 작성하라

##### 근거
[`swap`연산은 실패하지 않도록 작성하라](#Rc-swap-fail).
`swap`연산이 예외를 던지면서 종료하려 한다면, 그것은 잘못된 설계 오류이며 프로그램을 종료하는게 낫다.

##### 시행하기
* (쉬움) 클래스에 `swap` 멤버 함수가 있으면, `noexcept`로 선언되어야 한다.


### <a name="Rc-eq"></a>C.86: `==`연산자는 피연산자 타입들에 대칭적이고, `noexcept`로 만들어라.  

##### 근거
피연산자들에 비대칭적인 처리는 기대에 부합하지 않고, 형변환이 가능한 경우 에러를 유발할 수 있다. 
`==`는 기본적인 연산이며 프로그래머들이 이 연산을 사용할 때 연산 실패에 대한 고민이 없어야 한다.


##### 예
```c++
    class X {
        string name;
        int number;
    };

    bool operator==(const X& a, const X& b) noexcept { 
        return a.name == b.name 
                && a.number == b.number; 
    }
```
##### 잘못된 예
```c++
    class B {
        string name;
        int number;

        bool operator==(const B& a) const { 
            return name == a.name 
                    && number == a.number; 
        }

        // ...
    };
```
`B`의 비교 연산은 두번째 피연산자에 대해 형변환을 용인하지만, 첫번째 피연산자에 대해서는 그렇지 않다.

##### 참고 사항
만약 클래스가 `double`타입의 `NaN`처럼 실패 상태를 가진다면, 실패 상태와의 비교에서 예외를 던지도록 하는 것이 적합할 수도 있다.
다른 방법으로는 실패 상태끼리의 비교는 동등하게 보고, 적합한 상태와 실패 상태의 비교에서는 거짓으로 판정할 수 있다.    


#### Note
이 규칙은 모든 일반 비교 연산자들에도 적용된다 : `!=`, `<`, `<=`, `>`, `>=`.


##### 시행하기
* 인자의 타입이 다른 `operator==()`에는 표시를 남겨라. 다른 비교 연산자들도 마찬가지다 : `!=`, `<`, `<=`, `>`, `>=`.
* 멤버인  `operator==()`함수들에는 표시를 남겨라. 다른 비교 연산자들도 마찬가지다 : `!=`, `<`, `<=`, `>`, `>=`.


### <a name="Rc-eq-base"></a>C.87: 기본 클래스에 있는 `==`에 주의하라

##### 근거
계층 구조에서 잘못 사용하기 어렵고 유용한 `==`를 작성하는 것은 어려운 일이다. 

##### 잘못된 예
```c++
    class B {
        string name;
        int number;
        virtual bool operator==(const B& a) const
        {
             return name == a.name 
                    && number == a.number;
        }
        // ...
    };
```
`B`의 비교 연산은 두번째 피연산자에 대해서 타입 변환을 허용하지만, 첫번째 피연산자에 대해서는 허용하지 않는다.
```c++
    class D :B {
        char character;
        virtual bool operator==(const D& a) const
        {
            return name == a.name 
                    && number == a.number 
                    && character == a.character;
        }
        // ...
    };

    B b = ...
    D d = ...
    b == d;    // name과 number를 비교한다. d의 character는 무시한다.
    d == b;    // error: == 연산자가 정의되지 않았다
    D d2;
    d == d2;   // name과 number, character를 비교한다.
    B& b2 = d2;
    b2 == d;   // name과 number를 비교한다. d2와 d의 character는 무시한다
```
물론 계층 구조 안에서 `==`가 동작하도록 하는 방법들이 있지만, 단순한(naive) 방법들은 고려하지 말아라.  

#### Note
이 규칙은 모든 일반 비교연산자에 대해서도 동일하다 : `!=`, `<`, `<=`, `>`, `>=`

##### 시행하기
* 가상 함수인 `operator==()`에는 표시를 남겨라. 다른 비교 연산자들도 동일하다: `!=`, `<`, `<=`, `>`, `>=`.

### <a name="Rc-hash"></a>C.89:`hash`는 `noexcept`로 작성하라 

##### 근거
해시 컴테이너들의 사용자들은 hash를 간접적으로 사용하며, 해시값을 위한 단순한 접근이 throw하지 않을 것으로 기대한다.  
이는 표준 라이브러리의 요구사항이다.  

##### 잘못된 예
```c++
    template<>
    struct hash<My_type> {  // 정말정말 안좋은 해시 특수화

        using result_type = size_t;
        using argument_type = My_type;

        size_t operator() (const My_type & x) const
        {
            size_t xs = x.s.size();
            // "이런 이단 같으니!"
            if (xs < 4) 
                throw Bad_My_type{};
    
            return hash<size_t>()(x.s.size()) ^ trim(x.s);
        }
    };

    int main()
    {
        unordered_map<My_type, int> m;
        My_type mt{ "asdfg" };
        m[mt] = 7;
        cout << m[My_type{ "asdfg" }] << '\n';
    }
```
`hash` 특수화를 정의할 때는, 간단하게 `^` (xor)와 함께 표준 라이브러리의 `hash` 특수화와 통합되도록 하라.  
비 전문가들을 위해선 이 방법이 더 적합하다.


##### 시행하기
* 예외를 던지는 `hash`들에는 표시를 남겨라.



## <a name="SS-containers"></a>C.con: Containers and other resource handles
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#SS-containers)

A container is an object holding a sequence of objects of some type; `std::vector` is the archetypical container.
A resource handle is a class that owns a resource; `std::vector` is the typical resource handle; its resource is its sequence of elements.


컨테이너 규칙 요약

* [C.100: 컨테이너를 정의할때는 STL을 따르라](#Rcon-stl)
* [C.101: 값 문맥을 적용하라](#Rcon-val)
* [C.102: move 연산을 제공하라](#Rcon-move)
* [C.103: 초기화 리스트 생성자를 지원하라](#Rcon-init)
* [C.104: 공백 값으로 설정하는 기본 생성자를 지원하라](#Rcon-empty)
* [C.105: 생성자와 '확장' 생성자를 지원하라](#Rcon-val)
* ???
* [C.109: 리소스 핸들이 포인터 문맥을 따를 경우에는, `*` 과 `->` 연산자를 제공하라](#rcon-ptr)

### 같이 보기
[Resources](#S-resource)


## <a name="SS-lambdas"></a>C.lambdas: 함수 객체와 람다 표현식
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#SS-lambdas)


A function object is an object supplying an overloaded `()` so that you can call it.
A lambda expression (colloquially often shortened to "a lambda") is a notation for generating a function object.
Function objects should be cheap to copy (and therefore [passed by value](#Rf-in)).

요약:

* [F.50: Use a lambda when a function won't do (to capture local variables, or to write a local function)](#Rf-capture-vs-overload)
* [F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms](#Rf-reference-capture)
* [F.53: Avoid capturing by reference in lambdas that will be used nonlocally, including returned, stored on the heap, or passed to another thread](#Rf-value-capture)
* [ES.28: Use lambdas for complex initialization, especially of `const` variables](#Res-lambda-init)



## <a name="SS-hier"></a>C.hier:  클래스 계층 구조 (OOP)
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#SS-hier)

A class hierarchy is constructed to represent a set of hierarchically organized concepts (only).
Typically base classes act as interfaces.
There are two major uses for hierarchies, often named implementation inheritance and interface inheritance.

Class hierarchy rule summary:

* [C.120: Use class hierarchies to represent concepts with inherent hierarchical structure](#Rh-domain)
* [C.121: If a base class is used as an interface, make it a pure abstract class](#Rh-abstract)
* [C.122: Use abstract classes as interfaces when complete separation of interface and implementation is needed](#Rh-separation)

Designing rules for classes in a hierarchy summary:

* [C.126: An abstract class typically doesn't need a constructor](#Rh-abstract-ctor)
* [C.127: A class with a virtual function should have a virtual or protected destructor](#Rh-dtor)
* [C.128: Use `override` to make overriding explicit in large class hierarchies](#Rh-override)
* [C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance](#Rh-kind)
* [C.130: Redefine or prohibit copying for a base class; prefer a virtual `clone` function instead](#Rh-copy)
* [C.131: Avoid trivial getters and setters](#Rh-get)
* [C.132: Don't make a function `virtual` without reason](#Rh-virtual)
* [C.133: Avoid `protected` data](#Rh-protected)
* [C.134: Ensure all non-`const` data members have the same access level](#Rh-public)
* [C.135: Use multiple inheritance to represent multiple distinct interfaces](#Rh-mi-interface)
* [C.136: Use multiple inheritance to represent the union of implementation attributes](#Rh-mi-implementation)
* [C.137: Use `virtual` bases to avoid overly general base classes](#Rh-vbase)
* [C.138: Create an overload set for a derived class and its bases with `using`](#Rh-using)
* [C.139: Use `final` sparingly](#Rh-final)
* [C.140: Do not provide different default arguments for a virtual function and an overrider](#Rh-virtual-default-arg)

Accessing objects in a hierarchy rule summary:

* [C.145: Access polymorphic objects through pointers and references](#Rh-poly)
* [C.146: Use `dynamic_cast` where class hierarchy navigation is unavoidable](#Rh-dynamic_cast)
* [C.147: Use `dynamic_cast` to a reference type when failure to find the required class is considered an error](#Rh-ptr-cast)
* [C.148: Use `dynamic_cast` to a pointer type when failure to find the required class is considered a valid alternative](#Rh-ref-cast)
* [C.149: Use `unique_ptr` or `shared_ptr` to avoid forgetting to `delete` objects created using `new`](#Rh-smart)
* [C.150: Use `make_unique()` to construct objects owned by `unique_ptr`s](#Rh-make_unique)
* [C.151: Use `make_shared()` to construct objects owned by `shared_ptr`s](#Rh-make_shared)
* [C.152: Never assign a pointer to an array of derived class objects to a pointer to its base](#Rh-array)

### <a name="Rh-domain"></a>C.120: Use class hierarchies to represent concepts with inherent hierarchical structure (only)

##### 근거

Direct representation of ideas in code eases comprehension and maintenance. Make sure the idea represented in the base class exactly matches all derived types and there is not a better way to express it than using the tight coupling of inheritance.

Do *not* use inheritance when simply having a data member will do. Usually this means that the derived type needs to override a base virtual function or needs access to a protected member.

##### 예
```
    ??? Good old Shape example?
```
##### 잘못된 예

Do *not* represent non-hierarchical domain concepts as class hierarchies.
```c++
    template<typename T>
    class Container {
    public:
        // list operations:
        virtual T& get() = 0;
        virtual void put(T&) = 0;
        virtual void insert(Position) = 0;
        // ...
        // vector operations:
        virtual T& operator[](int) = 0;
        virtual void sort() = 0;
        // ...
        // tree operations:
        virtual void balance() = 0;
        // ...
    };
```
Here most overriding classes cannot implement most of the functions required in the interface well.
Thus the base class becomes an implementation burden.
Furthermore, the user of `Container` cannot rely on the member functions actually performing a meaningful operations reasonably efficiently;
it may throw an exception instead.
Thus users have to resort to run-time checking and/or
not using this (over)general interface in favor of a particular interface found by a run-time type inquiry (e.g., a `dynamic_cast`).

##### 시행하기

* Look for classes with lots of members that do nothing but throw.
* Flag every use of a nonpublic base class `B` where the derived class `D` does not override a virtual function or access a protected member in `B`, and `B` is not one of the following: empty, a template parameter or parameter pack of `D`, a class template specialized with `D`.

### <a name="Rh-abstract"></a>C.121: If a base class is used as an interface, make it a pure abstract class

##### 근거

A class is more stable (less brittle) if it does not contain data.
Interfaces should normally be composed entirely of public pure virtual functions and a default/empty virtual destructor.

##### 예
```c++
    class my_interface {
    public:
        // ...only pure virtual functions here ...
        virtual ~my_interface() {}   // or =default
    };
```
##### 잘못된 예
```c++
    class Goof {
    public:
        // ...only pure virtual functions here ...
        // no virtual destructor
    };

    class Derived : public Goof {
        string s;
        // ...
    };

    void use()
    {
        unique_ptr<Goof> p {new Derived{"here we go"}};
        f(p.get()); // use Derived through the Goof interface
        g(p.get()); // use Derived through the Goof interface
     } // leak
```
The `Derived` is `delete`d through its `Goof` interface, so its `string` is leaked.
Give `Goof` a virtual destructor and all is well.


##### 시행하기

* Warn on any class that contains data members and also has an overridable (non-`final`) virtual function.

### <a name="Rh-separation"></a>C.122: Use abstract classes as interfaces when complete separation of interface and implementation is needed

##### 근거

Such as on an ABI (link) boundary.

##### 예
```c++
    struct Device {
        virtual void write(span<const char> outbuf) = 0;
        virtual void read(span<char> inbuf) = 0;
    };

    class D1 : public Device {
        // ... data ...

        void write(span<const char> outbuf) override;
        void read(span<char> inbuf) override;
    };

    class D2 : public Device {
        // ... different data ...

        void write(span<const char> outbuf) override;
        void read(span<char> inbuf) override;
    };
```
A user can now use `D1`s and `D2`s interchangeably through the interface provided by `Device`.
Furthermore, we can update `D1` and `D2` in a ways that are not binarily compatible with older versions as long as all access goes through `Device`.

##### 시행하기
```
    ???
```

## C.hierclass: Designing classes in a hierarchy:

### <a name="Rh-abstract-ctor"></a>C.126: An abstract class typically doesn't need a constructor

##### 근거

An abstract class typically does not have any data for a constructor to initialize.

##### 예
```
    ???
```
##### 예외 사항

* A base class constructor that does work, such as registering an object somewhere, may need a constructor.
* In extremely rare cases, you might find it reasonable for an abstract class to have a bit of data shared by all derived classes
  (e.g., use statistics data, debug information, etc.); such classes tend to have constructors. But be warned: Such classes also tend to be prone to requiring virtual inheritance.

##### 시행하기

Flag abstract classes with constructors.


### <a name="Rh-dtor"></a>C.127: A class with a virtual function should have a virtual or protected destructor

##### 근거

A class with a virtual function is usually (and in general) used via a pointer to base. Usually, the last user has to call delete on a pointer to base, often via a smart pointer to base, so the destructor should be public and virtual. Less commonly, if deletion through a pointer to base is not intended to be supported, the destructor should be protected and nonvirtual; see [C.35](#Rc-dtor-virtual).

##### 잘못된 예
```c++
    struct B {
        virtual int f() = 0;
        // ... no user-written destructor, defaults to public nonvirtual ...
    };

    // bad: class with a resource derived from a class without a virtual destructor
    struct D : B {
        string s {"default"};
    };

    void use()
    {
        auto p = make_unique<D>();
        // ...
    } // calls B::~B only, leaks the string
```
##### 참고 사항

There are people who don't follow this rule because they plan to use a class only through a `shared_ptr`: `std::shared_ptr<B> p = std::make_shared<D>(args);` Here, the shared pointer will take care of deletion, so no leak will occur from an inappropriate `delete` of the base. People who do this consistently can get a false positive, but the rule is important -- what if one was allocated using `make_unique`? It's not safe unless the author of `B` ensures that it can never be misused, such as by making all constructors private and providing a factory function to enforce the allocation with `make_shared`.

##### 시행하기

* A class with any virtual functions should have a destructor that is either public and virtual or else protected and nonvirtual.
* Flag `delete` of a class with a virtual function but no virtual destructor.

### <a name="Rh-override"></a>C.128: Virtual functions should specify exactly one of `virtual`, `override`, or `final`

##### 근거

Readability. Detection of mistakes. Writing explicit `virtual`, `override`, or `final` is self-documenting and enables the compiler to catch mismatch of types and/or names between base and derived classes. However, writing more than one of these three is both redundant and a potential source of errors.

Use `virtual` only when declaring a new virtual function. Use `override` only when declaring an overrider. Use `final` only when declaring an final overrider.

##### 잘못된 예
```c++
    struct B {
        void f1(int);
        virtual void f2(int) const;
        virtual void f3(int);
        // ...
    };

    struct D : B {
        void f1(int);        // warn: D::f1() hides B::f1()
        void f2(int) const;  // warn: no explicit override
        void f3(double);     // warn: D::f3() hides B::f3()
        // ...
    };

    struct D2 : B {
        virtual void f2(int) final;  // BAD; pitfall, D2::f does not override B::f
    };
```
##### 시행하기

* Compare names in base and derived classes and flag uses of the same name that does not override.
* Flag overrides with neither `override` nor `final`.
* Flag function declarations that use more than one of `virtual`, `override`, and `final`.

### <a name="Rh-kind"></a>C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance

##### 근거

 ??? Herb: I've become a non-fan of implementation inheritance -- seems most often an anti-pattern. Are there reasonable examples of it?

##### 예
```
    ???
```
##### 시행하기

???

### <a name="Rh-copy"></a>C.130: Redefine or prohibit copying for a base class; prefer a virtual `clone` function instead

##### 근거

Copying a base is usually slicing. If you really need copy semantics, copy deeply: Provide a virtual `clone` function that will copy the actual most-derived type and return an owning pointer to the new object, and then in derived classes return the derived type (use a covariant return type).

##### 예
```c++
    class base {
    public:
        virtual owner<base*> clone() = 0;
        virtual ~base() = 0;

        base(const base&) = delete;
        base& operator=(const base&) = delete;
    };

    class derived : public base {
    public:
        owner<derived*> clone() override;
        virtual ~derived() override;
    };
```
Note that because of language rules, the covariant return type cannot be a smart pointer. See also [C.67](#Rc-copy-virtual).

##### 시행하기

* Flag a class with a virtual function and a non-user-defined copy operation.
* Flag an assignment of base class objects (objects of a class from which another has been derived).

### <a name="Rh-get"></a>C.131: Avoid trivial getters and setters

##### 근거

A trivial getter or setter adds no semantic value; the data item could just as well be `public`.

##### 예
```c++
    class point {
        int x;
        int y;
    public:
        point(int xx, int yy) : x{xx}, y{yy} { }
        int get_x() { return x; }
        void set_x(int xx) { x = xx; }
        int get_y() { return y; }
        void set_y(int yy) { y = yy; }
        // no behavioral member functions
    };
```
Consider making such a class a `struct` -- that is, a behaviorless bunch of variables, all public data and no member functions.
```c++
    struct point {
        int x = 0;
        int y = 0;
    };
```
##### 참고 사항

A getter or a setter that converts from an internal type to an interface type is not trivial (it provides a form of information hiding).

##### 시행하기

Flag multiple `get` and `set` member functions that simply access a member without additional semantics.

### <a name="Rh-virtual"></a>C.132: 함수를 이유없이 `virtual`로 만들지 말아라

##### 근거
중첩된 `virtual`은 실행 시간과 객체의 코드 크기를 증가시킨다.
가상 함수는 오버라이드 될 수 있고, 그렇기 때문에 파생 클래스에서의 실수에 노출되어있다. 
가상 함수는 템플릿 계층구조에서 코드 복제를 야기한다.

##### 잘못된 예
```c++
    template<class T>
    class Vector {
    public:
        // ...
        // bad: 파생 클래스에서 다른 무슨 일을 하겠는가?
        virtual int size() const { return sz; }

    private:
        T* elem;   // the elements
        int sz;    // number of elements
    };
```
이러한 형태의 "vector"는 기본 클래스로 사용되는 것을 전혀 의도하지 않았다.

##### 시행하기
* 가상 함수를 가지지만 파생 클래스가 없으면 표시를 남겨라.
* 모든 멤버 함수가 가상 함수이고 구현을 가지고 있으면 표시를 남겨라.


### <a name="Rh-protected"></a>C.133: `protected` 데이터를 지양하라

##### 근거
`protected` 데이터는 복잡성과 에러의 원인이다.  
`protected` 데이터는 불변조건의 구문을 복잡하게 만든다.  

`protected` data inherently violates the guidance against putting data in base classes, which usually leads to having to deal virtual inheritance as well.

##### 예
```
    ???
```
##### 참고 사항
`protected` 멤버 함수는 허용된다


##### 시행하기
`protected` 데이터를 지닌 클래스들에 표시를 남겨라.


### <a name="Rh-public"></a>C.134: Ensure all non-`const` data members have the same access level

##### 근거

Prevention of logical confusion leading to errors.
If the non-`const` data members don't have the same access level, the type is confused about what it's trying to do.
Is it a type that maintains an invariant or simply a collection of values?

##### Discussion

The core question is: What code is responsible for maintaining a meaningful/correct value for that variable?

There are exactly two kinds of data members:

* A: Ones that don't participate in the object's invariant. Any combination of values for these members is valid.
* B: Ones that do participate in the object's invariant. Not every combination of values is meaningful (else there'd be no invariant). Therefore all code that has write access to these variables must know about the invariant, know the semantics, and know (and actively implement and enforce) the rules for keeping the values correct.

Data members in category A should just be `public` (or, more rarely, `protected` if you only want derived classes to see them). They don't need encapsulation. All code in the system might as well see and manipulate them.

Data members in category B should be `private` or `const`. This is because encapsulation is important. To make them non-`private` and non-`const` would mean that the object can't control its own state: An unbounded amount of code beyond the class would need to know about the invariant and participate in maintaining it accurately -- if these data members were `public`, that would be all calling code that uses the object; if they were `protected`, it would be all the code in current and future derived classes. This leads to brittle and tightly coupled code that quickly becomes a nightmare to maintain. Any code that inadvertently sets the data members to an invalid or unexpected combination of values would corrupt the object and all subsequent uses of the object.

Most classes are either all A or all B:

* *All public*: If you're writing an aggregate bundle-of-variables without an invariant across those variables, then all the variables should be `public`.
  [By convention, declare such classes `struct` rather than `class`](#Rc-struct)
* *All private*: If you're writing a type that maintains an invariant, then all the non-`const` variables should be private -- it should be encapsulated.

##### 예외 사항

Occasionally classes will mix A and B, usually for debug reasons. An encapsulated object may contain something like non-`const` debug instrumentation that isn't part of the invariant and so falls into category A -- it isn't really part of the object's value or meaningful observable state either. In that case, the A parts should be treated as A's (made `public`, or in rarer cases `protected` if they should be visible only to derived classes) and the B parts should still be treated like B's (`private` or `const`).

##### 시행하기

Flag any class that has non-`const` data members with different access levels.

### <a name="Rh-mi-interface"></a>C.135: 다중 상속을 복수의 인터페이스를 표현하기 위해 사용하라

##### 근거
모든 클래스들이 모든 인터페이스들을 지원하지는 않을 것이다. 그리고 모든 호출자(caller)들이 모든 연산들을 사용하길 원하지도 않을 것이다. (다중 상속은) 특별히 단일한(monolitic) 인터페이스들을 파생 클래스가 지원하는 동작의 "측면"들로 나눌때 사용하라. 


##### 예
```
    ???
```
##### 참고 사항

This is a very common use of inheritance because the need for multiple different interfaces to an implementation is common
and such interfaces are often not easily or naturally organized into a single-rooted hierarchy.

##### 참고 사항
Such interfaces are typically abstract classes.

##### 시행하기
???

### <a name="Rh-mi-implementation"></a>C.136: 다중 상속을 구현 속성들의 합집합(union)을 표현하기 위해 사용하라

##### 근거
???   
Herb: 여기서 구현 상속에 대한 두번째 언급이 있군요. 전 매우 부정적입니다. 하나의 구현 상속마저도요. 다수의 구현 상속은 절대 생각하지 마세요. -- 저는 정책 기반의(policy-based) 설계라도 정말로 정책 타입들을 상속할 필요가 있다고 생각하지 않습니다.
제가 좋은 예시들을 놓치고 있는 걸까요, 아니면 이것을 안티패턴으로 보고 제외시키는 것을 고려해야 할까요? 

##### 예
```
    ???
```
##### 참고 사항
이것은 상대적으로 드문 경우인데, 구현은 종종 단일루트(single-root) 계층으로 조직화될 수 있기 때문이다.

##### 시행하기
??? 
Herb: 정반대의 시행하기: 2개 이상의 (데이터 멤버가 있는)기본 클래스를 상속하는 타입에는 표시를 남겨라?  


### <a name="Rh-vbase"></a>C.137: Use `virtual` bases to avoid overly general base classes

##### 근거
 ???

##### 예
```
    ???
```
##### 참고 사항
???

##### 시행하기
???


### <a name="Rh-using"></a>C.138: Create an overload set for a derived class and its bases with `using`

##### 근거
???

##### 예
```
    ???
```
### <a name="Rh-final"></a>C.139: Use `final` sparingly

##### 근거

Capping a hierarchy with `final` is rarely needed for logical reasons and can be damaging to the extensibility of a hierarchy.
Capping an individual virtual function with `final` is error-prone as that `final` can easily be overlooked when defining/overriding a set of functions.

##### 잘못된 예
```c++
    class Widget { /* ... */ };

    class My_widget final : public Widget { /* ... */ };    // nobody will ever want to improve My_widget (or so you thought)

    class My_improved_widget : public My_widget { /* ... */ };  // error: can't do that
```
##### 잘못된 예
```c++
    struct Interface {
        virtual int f() = 0;
        virtual int g() = 0;
    };

    class My_implementation : public Interface {
        int f() override;
        int g() final;  // I want g() to be FAST!
        // ...
    };

    class Better_implementation : public My_implementation {
        int f();
        int g();
        // ...
    };

    void use(Interface* p)
    {
        int x = p->f();    // Better_implementation::f()
        int y = p->g();    // My_implementation::g() Surprise?
    }

    // ...

    use(new Better_interface{});
```
The problem is easy to see in a small example, but in a large hierarchy with many virtual functions, tools are required for reliably spotting such problems.
Consistent use of `override` would catch this.

##### 참고 사항

Claims of performance improvements from `final` should be substantiated.
Too often, such claims are based on conjecture or experience with other languages.

There are examples where `final` can be important for both logical and performance reasons.
One example is a performance-critical AST hierarchy in a compiler or language analysis tool.
New derived classes are not added every year and only by library implementers.
However, misuses are (or at least has been) far more common.

##### 시행하기

Flag uses of `final`.


## <a name="Rh-virtual-default-arg"></a>C.140: Do not provide different default arguments for a virtual function and an overrider

##### 근거

That can cause confusion: An overrider do not inherit default arguments..

##### 잘못된 예
```c++
    class base {
    public:
        virtual int multiply(int value, int factor = 2) = 0;
    };

    class derived : public base {
    public:
        int multiply(int value, int factor = 10) override;
    };

    derived d;
    base& b = d;

    b.multiply(10);  // these two calls will call the same function but
    d.multiply(10);  // with different arguments and so different results
```
##### 시행하기

Flag default arguments on virtual functions if they differ between base and derived declarations.




## C.hier-access: 계층 구조에서 객체 접근

### <a name="Rh-poly"></a>C.145: 다형성을 지닌 객체는 포인터나 참조자를 사용해서 접근하라

##### 근거
가상 함수를 가진 클래스가 있다면, 당신은 (일반적으로) 어떤 클래스가 실행될 함수를 제공할지 알 수 없다.

##### 예
```c++
    struct B { 
        int a; 
        virtual int f(); 
    };

    struct D : B { 
        int b; 
        int f() override; 
    };

    void use(B b)
    {
        D d;
        B b2 = d;   // 복사 손실(slice)
        B b3 = b;
    }

    void use2()
    {
        D d;
        use(d);   // 복사 손실(slice)
    }
```
양쪽의 `d` 모두 복사 손실로 잘려나간다.  


##### 예외 사항
객체가 정의된 범위 안에서는 이름이 있는 다형적 객체에 안전하게 접근할 수 있다. 단지 slice가 생기지 않도록 하라.
```c++
    void use3()
    {
        D d;
        d.f();   // OK
    }
```
##### 시행하기
모든 slicing에 표시를 남겨라


### <a name="Rh-dynamic_cast"></a>C.146: `dynamic_cast`는 클래스 계층 구조에서 탐색이 불가피할 때 사용하라

##### 근거
`dynamic_cast`는 실행시간에 검사된다.

##### 예
```c++
    struct B {   // 인터페이스
        virtual void f();
        virtual void g();
    };

    struct D : B {   // 확장된 인터페이스
        void f() override;
        virtual void h();
    };

    void user(B* pb)
    {
        if (D* pd = dynamic_cast<D*>(pb)) {
            // ... D의 인터페이스를 사용한다 ...
        }
        else {
            // ... B의 인터페이스를 사용한다 ...
        }
    }
```
##### 참고 사항
다른 모든 캐스팅처럼, `dynamic_cast`는 너무 자주 사용된다.

[캐스팅 보다는 가상 함수들을 사용하라](#???).  
가능한 한 클래스 계층을 탐색하는 것보다 [정적 다형성](#???)을 선호하라. 이렇게 하면 실행시간 결정이 필요없다. 그리고 충분히 편리하다.

##### 참고 사항
Some people use `dynamic_cast` where a `typeid` would have been more appropriate;
`dynamic_cast` is a general "is kind of" operation for discovering the best interface to an object,
whereas `typeid` is a "give me the exact type of this object" operation to discover the actual type of an object.
The latter is an inherently simpler operation that ought to be faster.
The latter (`typeid`) is easily hand-crafted if necessary (e.g., if working on a system where RTTI is -- for some reason -- prohibited),
the former (`dynamic_cast`) is far harder to implement correctly in general.

Consider:
```c++
    struct B {
        const char * name {"B"};
        virtual const char* id() const { return name; }
        // ...
    };

    struct D : B {
        const char * name {"D"};
        const char* id() const override { return name; }
        // ...
    };

    void use()
    {
        B* pb1 = new B;
        B* pb2 = new D;

        cout << pb1->id(); // "B"
        cout << pb2->id(); // "D"

        if (pb1->id() == pb2->id()) // *pb1 is the same type as *pb2
        if (pb2 == "D") {         // looks innocent
                D* pd = static_cast<D*>(pb1);
                // ...
        }
        // ...
    }
```
The result of `pb2 == "D"` is actually implementation defined.
We added it to warn of the dangers of home-brew RTTI.
This code may work as expected for years, just to fail on a new machine, new compiler, or a new linker that does not unify character literals.

If you implement your own RTTI, be careful.

##### 예외 사항
만약 당신의 구현 코드에 정말로 느린 `dynamic_cast`가 있다면, 대안을 찾아야 할 것이다. 
하지만, 정적으로 클래스를 결정할 수 없는 모든 대안은 명시적 캐스팅(일반적으로 `static_cast`)을 포함하고, 에러에 취약하다.  
당신만의 특별한 `dynamic_cast`를 만들수도 있을 것이다. 그러니, `dynamic_cast`가 정말로 당신이 생각하는 것 만큼 느리다는 것을 확실히하라. (근거 없는 루머들이 꽤 있다.) 그리고 `dynamic_cast`의 사용이 정말로 성능에 치명적이라는 것 또한 확인하라. 

We are of the opinion that current implementations of `dynamic_cast` are unnecessarily slow.
For example, under suitable conditions, it is possible to perform a `dynamic_cast` in [fast constant time](http://www.stroustrup.com/fast_dynamic_casting.pdf).
However, compatibility makes changes difficult even if all agree that an effort to optimize is worthwhile.

In very rare cases, if you have measured that the `dynamic_cast` overhead is material, you have other means to statically guarantee that a downcast will succeed (e.g., you are using CRTP carefully), and there is no virtual inheritance involved, consider tactically resorting `static_cast` with a prominent comment and disclaimer summarizing this paragraph and that human attention is needed under maintenance because the type system can't verify correctness. Even so, in our experience such "I know what I'm doing" situations are still a known bug source.

##### 시행하기
하향식 캐스팅(C언어 스타일을 포함해서)에 사용되는 `static_cast`에 표시를 남겨라. 


### <a name="Rh-ptr-cast"></a>C.147: 필요한 타입을 찾는 데 실패하는 것이 에러로 간주될 때는, `dynamic_cast`를 참조자 타입에 사용하라

##### 근거
참조자에 대한 캐스팅은 당신이 정상적인 객체를 얻는 것을 의도했음을 표현한다. 따라서 캐스팅은 반드시 성공해야만 한다. `dynamic_cast`는 만약 실패한다면 예외를 던질 것이다.

##### 예
```
    ???
```
##### 시행하기
???


### <a name="Rh-ref-cast"></a>C.148: Use `dynamic_cast` to a pointer type when failure to find the required class is considered a valid alternative

##### 근거

???

##### 예
```
    ???
```
##### 시행하기

???

### <a name="Rh-smart"></a>C.149: `new`를 사용해서 생성한 객체를 `delete`하지 않는 것을 예방하기 위해, `unique_ptr` 또는 `shared_ptr`를 사용하라

##### 근거
자원 누수를 방지한다.

##### 예
```c++
    void use(int i)
    {
        // bad: initialize local pointers with new
        auto p = new int {7};

        // ok: guarantee the release of the memory allocated for 9
        auto q = make_unique<int>(9);

        if (0 < i)  // maybe return and leak 
            return;
            
        delete p;   // too late
    }
```
##### 시행하기
* `new`를 사용한 일반(naked) 포인터의 초기화에 표시를 남겨라. 
* 지역 변수의 `delete`처리에 표시를 남겨라. 


### <a name="Rh-make_unique"></a>C.150: Use `make_unique()` to construct objects owned by `unique_ptr`s

##### 근거

 `make_unique` gives a more concise statement of the construction.
It also ensures exception safety in complex expressions.

##### 예
```c++
    unique_ptr<Foo> p {new<Foo>{7}};   // OK: but repetitive

    auto q = make_unique<Foo>(7);      // Better: no repetition of Foo

    // Not exception-safe: the compiler may interleave the computations of arguments as follows:
    //
    // 1. allocate memory for Foo,
    // 2. construct Foo,
    // 3. call bar,
    // 4. construct unique_ptr<Foo>.
    //
    // If bar throws, Foo will not be destroyed, and the memory allocated for it will leak.
    f(unique_ptr<Foo>(new Foo()), bar());

    // Exception-safe: calls to functions are never interleaved.
    f(make_unique<Foo>(), bar());
```
##### 시행하기

* Flag the repetitive usage of template specialization list `<Foo>`
* Flag variables declared to be `unique_ptr<Foo>`

### <a name="Rh-make_shared"></a>C.151: Use `make_shared()` to construct objects owned by `shared_ptr`s

##### 근거

 `make_shared` gives a more concise statement of the construction.
It also gives an opportunity to eliminate a separate allocation for the reference counts, by placing the `shared_ptr`'s use counts next to its object.

##### 예
```c++
    // OK: but repetitive; and separate allocations for the Foo and shared_ptr's use count
    shared_ptr<Foo> p {new<Foo>{7}};

    auto q = make_shared<Foo>(7);   // Better: no repetition of Foo; one object
```
##### 시행하기

* Flag the repetitive usage of template specialization list`<Foo>`
* Flag variables declared to be `shared_ptr<Foo>`

### <a name="Rh-array"></a>C.152: Never assign a pointer to an array of derived class objects to a pointer to its base

##### 근거

Subscripting the resulting base pointer will lead to invalid object access and probably to memory corruption.

##### 예
```c++
    struct B { int x; };
    struct D : B { int y; };

    void use(B*);

    D a[] = {{1, 2}, {3, 4}, {5, 6}};
    B* p = a;     // bad: a decays to &a[0] which is converted to a B*
    p[1].x = 7;   // overwrite D[0].y

    use(a);       // bad: a decays to &a[0] which is converted to a B*
```
##### 시행하기

* Flag all combinations of array decay and base to derived conversions.
* Pass an array as a `span` rather than as a pointer, and don't let the array name suffer a derived-to-base conversion before getting into the `span`


## <a name="SS-overload"></a>C.over: 오버로딩
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#SS-overload) 

You can overload ordinary functions, template functions, and operators.
You cannot overload function objects.

Overload rule summary:

* [C.160: 연산자를 정의할때는 관례적인 사용을 모방하라](#Ro-conventional)
* [C.161: 대칭적인 연산자들에는 비멤버 함수들을 사용하라](#Ro-symmetric)
* [C.162: 거의 동등한 연산들을 오버로드하라](#Ro-equivalent)
* [C.163: 거의 동등한 연산들만 오버로드하라](#Ro-equivalent-2)
* [C.164: 형변환 연산자들을 지양하라](#Ro-conversion)
* [C.165: Use `using` for customization points](#Ro-custom)
* [C.166: Overload unary `&` only as part of a system of smart pointers and references](#Ro-address-of)
* [C.167: Use an operator for an operation with its conventional meaning](#Ro-overload)
* [C.168: Define overloaded operators in the namespace of their operands](#Ro-namespace)
* [C.170: 람다를 오버로딩하는 기분이 든다면, 제네릭 람다를 사용하라](#Ro-lambda)

### <a name="Ro-conventional"></a>C.160: 연산자를 정의할때는 관례적인 사용을 모방하라

##### 근거
뜻밖의 의미가 없도록 한다.

##### 예
```c++
    class X {
    public:
        // ...
        X& operator=(const X&); // member function defining assignment

        // == needs access to representation
        friend bool operator==(const X&, const X&); 
        
        // after a=b we have a==b
        // ...
    };
```
Here, the conventional semantics is maintained: [Copies compare equal](#SS-copy).

##### 잘못된 예
```c++
    X operator+(X a, X b) { return a.v - b.v; }   // bad: makes + subtract
```
##### 참고 사항

Non-member operators should be either friends or defined in [the same namespace as their operands](#Ro-namespace).
[Binary operators should treat their operands equivalently](#Ro-symmetric).

##### 시행하기
Possibly impossible.


### <a name="Ro-symmetric"></a>C.161: 대칭적인 연산자들에는 비멤버 함수들을 사용하라

##### 근거

If you use member functions, you need two.
Unless you use a non-member function for (say) `==`, `a == b` and `b == a` will be subtly different.

##### 예
```c++
    bool operator==(Point a, Point b) { return a.x == b.x && a.y == b.y; }
```
##### 시행하기

Flag member operator functions.


### <a name="Ro-equivalent"></a>C.162: 거의 동등한 연산들을 오버로드하라

##### 근거

Having different names for logically equivalent operations on different argument types is confusing, leads to encoding type information in function names, and inhibits generic programming.

##### 예

Consider:
```c++
    void print(int a);
    void print(int a, int base);
    void print(const string&);
```
These three functions all print their arguments (appropriately). Conversely:
```c++
    void print_int(int a);
    void print_based(int a, int base);
    void print_string(const string&);
```
These three functions all print their arguments (appropriately). Adding to the name just introduced verbosity and inhibits generic code.

##### 시행하기

???


### <a name="Ro-equivalent-2"></a>C.163: 거의 동등한 연산들'만' 오버로드하라

##### 근거

Having the same name for logically different functions is confusing and leads to errors when using generic programming.

##### 예

Consider:
```c++
    void open_gate(Gate& g);   // remove obstacle from garage exit lane
    void fopen(const char* name, const char* mode);   // open file
```
The two operations are fundamentally different (and unrelated) so it is good that their names differ. Conversely:
```c++
    void open(Gate& g);   // remove obstacle from garage exit lane
    void open(const char* name, const char* mode ="r");   // open file
```
The two operations are still fundamentally different (and unrelated) but the names have been reduced to their (common) minimum, opening opportunities for confusion.
Fortunately, the type system will catch many such mistakes.

##### 참고 사항

Be particularly careful about common and popular names, such as `open`, `move`, `+`, and `==`.

##### 시행하기
???


### <a name="Ro-conversion"></a>C.164:형변환 연산자들을 지양하라

##### 근거
Implicit conversions can be essential (e.g., `double` to `int`) but often cause surprises (e.g., `String` to C-style string).

##### 참고 사항
Prefer explicitly named conversions until a serious need is demonstrated.
By "serious need" we mean a reason that is fundamental in the application domain (such as an integer to complex number conversion)
and frequently needed. Do not introduce implicit conversions (through conversion operators or non-`explicit` constructors)
just to gain a minor convenience.

##### 잘못된 예
```c++
    class String {   // handle ownership and access to a sequence of characters
        // ...
        String(czstring p); // copy from *p to *(this->elem)
        // ...
        operator zstring() { return elem; }
        // ...
    };

    void user(zstring p)
    {
        if (*p == "") {
            String s {"Trouble ahead!"};
            // ...
            p = s;
        }
        // use p
    }
```
`s`에 할당되고 `p`에 대입된 문자열이 사용될 수 있게 되기 전에 파괴된다.

##### 시행하기
모든 형변환 연산자에 표시를 남겨라.


### <a name="Ro-custom"></a>C.165: Use `using` for customization points

##### 근거
To find function objects and functions defined in a separate namespace to "customize" a common function.

##### 예
Consider `swap`. It is a general (standard library) function with a definition that will work for just about any type.
However, it is desirable to define specific `swap()`s for specific types.
For example, the general `swap()` will copy the elements of two `vector`s being swapped, whereas a good specific implementation will not copy elements at all.
```c++
    namespace N {
        My_type X { /* ... */ };
        void swap(X&, X&);   // optimized swap for N::X
        // ...
    }

    void f1(N::X& a, N::X& b)
    {
        std::swap(a, b);   // probably not what we wanted: calls std::swap()
    }
```
The `std::swap()` in `f1()` does exactly what we asked it to do: it calls the `swap()` in namespace `std`.
Unfortunately, that's probably not what we wanted.
How do we get `N::X` considered?
```c++
    void f2(N::X& a, N::X& b)
    {
        swap(a, b);   // calls N::swap
    }
```
But that may not be what we wanted for generic code.
There, we typically want the specific function if it exists and the general function if not.
This is done by including the general function in the lookup for the function:
```c++
    void f3(N::X& a, N::X& b)
    {
        using std::swap;  // make std::swap available
        swap(a, b);        // calls N::swap if it exists, otherwise std::swap
    }
```
##### 시행하기

Unlikely, except for known customization points, such as `swap`.
The problem is that the unqualified and qualified lookups both have uses.


### <a name="Ro-address-of"></a>C.166: Overload unary `&` only as part of a system of smart pointers and references

##### 근거

The `&` operator is fundamental in C++.
Many parts of the C++ semantics assumes its default meaning.

##### 예
```c++
    class Ptr { // a somewhat smart pointer
        Ptr(X* pp) :p(pp) { /* check */ }
        X* operator->() { /* check */ return p; }
        X operator[](int i);
        X operator*();
    private:
        T* p;
    };

    class X {
        Ptr operator&() { return Ptr{this}; }
        // ...
    };
```
##### 참고 사항

If you "mess with" operator `&` be sure that its definition has matching meanings for `->`, `[]`, `*`, and `.` on the result type.
Note that operator `.` currently cannot be overloaded so a perfect system is impossible.
We hope to remedy that: [n4477.pdf](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4477.pdf).  
Note that `std::addressof()` always yields a built-in pointer.

##### 시행하기

Tricky. Warn if `&` is user-defined without also defining `->` for the result type.


### <a name="Ro-namespace"></a>C.168: Define overloaded operators in the namespace of their operands

##### 근거

Readability.
Ability for find operators using ADL.
Avoiding inconsistent definition in different namespaces

##### 예
```c++
    struct S { };
    bool operator==(S, S);   // OK: in the same namespace as S, and even next to S
    S s;

    bool s == s;
```
This is what a default `==` would do, if we had such defaults.

##### 예
```c++
    namespace N {
        struct S { };
        bool operator==(S, S);   // OK: in the same namespace as S, and even next to S
    }

    N::S s;

    bool s == s;  // finds N::operator==() by ADL
```

##### 잘못된 예
```c++
    struct S { };
    S s;

    namespace N {
        S::operator!(S a) { return true; }
        S not_s = !s;
    }

    namespace M {
        S::operator!(S a) { return false; }
        S not_s = !s;
    }
```
Here, the meaning of `!s` differs in `N` and `M`.
This can be most confusing.
Remove the definition of `namespace M` and the confusion is replaced by an opportunity to make the mistake.

##### 참고 사항

If a binary operator is defined for two types that are defined in different namespaces, you cannot follow this rule.
For example:
```c++
    Vec::Vector operator*(const Vec::Vector&, const Mat::Matrix&);
```
This may be something best avoided.

##### 함께 보기

This is a special case of the rule that [helper functions should be defined in the same namespace as their class](#Rc-helper).

##### 시행하기

* Flag operator definitions that are not it the namespace of their operands


### <a name="Ro-overload"></a>C.167: Use an operator for an operation with its conventional meaning

##### 근거

Readability. Convention. Reusability. Support for generic code

##### 예
```c++
    void cout_my_class(const my_class& c) // confusing, not conventional,not generic
    {
        std::cout << /* class members here */;
    }

    std::ostream& operator<<(std::ostream& os, const my_class& c) //OK
    {
        return os << /* class members here */;
    }
```
By itself, `cout_my_class` would be OK, but it is not usable/composable with code that rely on the `<<` convention for output:
```c++
    My_class var { /* ... */ };
    // ...
    cout << "var = " << var << '\n';
```
##### 참고 사항

There are strong and vigorous conventions for the meaning most operators, such as

* comparisons (`==`, `!=`, `<`, `<=`, `>`, and `>=`),
* arithmetic operations (`+`, `-`, `*`, `/`, and `%`)
* access operations (`.`, `->`, unary `*`, and `[]`)
* assignment (`=`)

Don't define those unconventionally and don't invent your own names for them.

##### 시행하기
까다롭다. 의미론에 대한 통찰이 필요하다.


### <a name="Ro-lambda"></a>C.170: 람다를 오버로딩하는 기분이 든다면, 제네릭 람다를 사용하라


##### 근거
같은 이름으로 다른 람다 함수를 오버로드 할 수 없다. 

##### 예
```c++
    void f(int);
    void f(double);
    auto f = [](char);   // error: cannot overload variable and function

    auto g = [](int) { /* ... */ };
    auto g = [](double) { /* ... */ };   // error: cannot overload variables

    auto h = [](auto) { /* ... */ };   // OK
```
##### 시행하기
컴파일러가 람다 함수에 대한 오버로드 시도를 잡아낸다. 


## <a name="SS-union"></a>C.union: Unions
> [원문 링크](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#SS-union)

???

공용체(Unions) 규칙 요약:

* [C.180: `union`은 ???에 사용하라](#Ru-union)
* [C.181: `union` 그대로 사용하는 것을 지양하라](#Ru-naked)
* [C.182: 익명 `union`은 tagged union을 구현하는데 사용하라](#Ru-anonymous)
* ???

### <a name="Ru-union"></a>C.180: Use `union`s to ???

??? When should unions be used, if at all? What's a good future-proof way to re-interpret object representations of PODs?
??? variant

##### 근거
 ???

##### 예
```
    ???
```

##### 시행하기
???


### <a name="Ru-naked"></a>C.181: Avoid "naked" `union`s

##### 근거
Naked unions are a source of type errors.

##### 대안
Wrap them in a class together with a type field.

##### 대안
Use `variant`.

##### 예
```
    ???
```


##### 시행하기
???


### <a name="Ru-anonymous"></a>C.182: Use anonymous `union`s to implement tagged unions

##### 근거
???

##### 예
```
    ???
```
##### 시행하기

???
