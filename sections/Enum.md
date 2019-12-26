# <a name="S-enum"></a>Enum: 열거형

> 역주 : 열거형(enumeration)  
> 역주 : 열거체(enumerator)  
> 역주 : 열거값(enumerator value)  

열거형은 정수 값의 집합을 정의하는데 사용되며, 그런 값들의 집합을 타입으로 정의하는데 사용된다.
열거형에는 두 종류가 있는데, "단순한" `enum`과 `class enum`이다.

열거형 규칙 요약:

* [Enum.1: 매크로보다 열거형을 사용하라](#Renum-macro)
* [Enum.2: 서로 관련있는 상수들의 집합을 표현하기 위해 열거형을 사용하라](#Renum-set)
* [Enum.3: 단순한 `enum`보다 `enum class`를 선호하라](#Renum-class)
* [Enum.4: 안전하고 단순한 사용을 위해 열거형들을 위한 연산을 정의하라](#Renum-oper)
* [Enum.5: 열거체들을 `ALL_CAPS`형태로 사용하지 마라](#Renum-caps)
* [Enum.6: 이름없는 열거형을 지양하라](#Renum-unnamed)
* [Enum.7: 필요할 때만 기본 타입을 명시하라](#Renum-underlying)
* [Enum.8: 열거체들의 값은 필요할때만 명시하라](#Renum-value)

### <a name="Renum-macro"></a>Enum.1: 매크로보다 열거형을 사용하라

##### Reason

매크로는 유효범위 또는 타입 규칙들을 무시한다. 전처리 과정에서 소멸되기 때문에 일반적으로 디버깅 도구에서는 확인할 수 없다.

##### Example

오래된, 나쁜 코드를 먼저 보자.

```c++
    // webcolors.h (3rd party header)
    #define RED   0xFF0000
    #define GREEN 0x00FF00
    #define BLUE  0x0000FF

    // productinfo.h
    // 여기서는 제품의 색에 따른 보조 타입을 정의한다
    #define RED    0
    #define PURPLE 1
    #define BLUE   2

    int webby = BLUE;   // webby == 2; 엉뚱한 값이 사용된다
```

`enum` 대신 이렇게 작성할 수 있다:

```c++
    enum class Web_color {
            red     = 0xFF0000,
            green   = 0x00FF00,
            blue    = 0x0000FF
        };

    enum class Product_info {
            red     = 0,
            purple  = 1,
            blue    = 2
        };

    int       webby = blue;   // 에러: enum class를 명시해야한다
    Web_color webby = Web_color::blue;
```

이름의 충돌을 막기 위해서 `enum class`를 사용할 수 있다

##### Enforcement

정수 값을 사용하는 매크로들을 지적하라

### <a name="Renum-set"></a>Enum.2: 서로 관련있는 상수들의 집합을 표현하기 위해 열거형을 사용하라

##### Reason

모든 열거형들은 열거체들이 특정한 타입으로 묶일 수 있다는 것을 의미한다

##### Example

```c++
    enum class Web_color { red = 0xFF0000, green = 0x00FF00, blue = 0x0000FF };
```

##### Note

열거형을 사용해 `switch`하는 것은 일반적이며, 비정상적인 `case`에 대해 컴파일러가 경고할 수 있다.

예를 들자면:

```c++
    enum class Product_info { red = 0, purple = 1, blue = 2 };

    void print(Product_info inf)
    {
        // default를 사용하지 않고,
        // case가 모든 enumerator들을 검출하지도 않는다.
        switch (inf) {
        case Product_info::red    :
            cout << "red";
            break;
        case Product_info::purple :
            cout << "purple";
            break;
        }
    }
```

이처럼 한개 모자란(off-by-one) `switch`구문은 보통 열거체가 추가되었거나 충분한 테스트가 없었기 때문에 발생한다.

##### Enforcement

* `case`가 모든 열거체들을 고려하지 않는 `switch`구문을 지적하라
* `case`가 일부 열거체만 사용하고, `default`를 사용하지 않는 경우 지적하라

### <a name="Renum-class"></a>Enum.3: 단순한 `enum`보다 `enum class`를 선호하라

##### Reason

오해의 소지가 없다. 전통적으로 사용된 enum은 int와 다를 것이 없다.

##### Example

```c++
    void Print_color(int color);

    enum Web_color { red = 0xFF0000, green = 0x00FF00, blue = 0x0000FF };
    enum Product_info { Red = 0, Purple = 1, Blue = 2 };

    Web_color webby = Web_color::blue;

    // 분명 이 둘중 하나는 버그를 일으킬 것이다
    Print_color(webby);
    Print_color(Product_info::Blue);
```

대신, `enum class` 사용하면:

```c++
    void Print_color(int color);

    enum class Web_color { red = 0xFF0000, green = 0x00FF00, blue = 0x0000FF };
    enum class Product_info { red = 0, purple = 1, blue = 2 };

    Web_color webby = Web_color::blue;
    Print_color(webby);  // Error: cannot convert Web_color to int.
    Print_color(Product_info::Red);  // Error: cannot convert Product_info to int.
```

##### Enforcement

(쉬움) 클래스가 아닌 `enum` 정의를 조심하라.

### <a name="Renum-oper"></a>Enum.4: 안전하고 단순한 사용을 위해 열거형들을 위한 연산을 정의하라

##### Reason

에러를 방지하는데 편리하다

##### Example

```c++
    enum class Day { mon, tue, wed, thu, fri, sat, sun };

    Day operator++(Day& d)
    {
        return d == Day::sun ? Day::mon : Day{++d};
    }

    Day today = Day::sat;
    Day tomorrow = ++today;
```

##### Enforcement

Flag repeated expressions cast back into an enumeration.

### <a name="Renum-caps"></a>Enum.5: 열거체들을 `ALL_CAPS`형태로 사용하지 마라

##### Reason

매크로들의 충돌을 방지한다

##### Example, bad

```c++
    // webcolors.h (third party header)
    #define RED   0xFF0000
    #define GREEN 0x00FF00
    #define BLUE  0x0000FF

    // productinfo.h
    // 매크로가 문법에러를 발생시킨다
    enum class Product_info { RED, PURPLE, BLUE };
```

##### Enforcement

대문자만으로 구성된 열거체들을 지적하라

### <a name="Renum-unnamed"></a>Enum.6: 이름없는 열거형을 지양하라

##### Reason

열거형에 이름을 붙일 수 없다면, 서로 상관없는 값들이다.

##### Example, bad

```c++
    enum { red = 0xFF0000, scale = 4, is_signed = 1 };
```

이런 코드는 정수 상수들을 정의하는 편리한 방법이 나타나기 전까진 일반적이었다.

##### Alternative

대신 `constexpr`를 사용하라 :

```c++
    constexpr int   red         = 0xFF0000;
    constexpr short scale       = 4;
    constexpr bool  is_signed   = true;
```

##### Enforcement

이름없는 열거형을 지적하라

### <a name="Renum-underlying"></a>Enum.7: 필요할 때만 기본 타입을 명시하라

##### Reason

기본 타입은 읽고 쓰기에 쉽다.
`int`는 기본 정수 타입이며, C 언어의 `enum`과 호환된다.

##### Example

```c++
    // 기본 타입(char)가 공간을 절약해준다
    enum class Direction : char { n, s, e, w,
                                  ne, nw, se, sw };  

    // 기본 타입(int)가 공간을 소모한다.(redundant)
    enum class Web_color : int { red   = 0xFF0000,
                                 green = 0x00FF00,
                                 blue  = 0x0000FF };  
```

##### Note

기본 타입을 정의하는 것은 전방 선언이 필요한 경우 필수적이다.

> 역주 : 전방 선언(forward declaration)

```c++
    // Forward declaration
    enum Flags : char;

    void f(Flags);

    // ....

    // Definition
    enum flags : char { /* ... */ };
```

##### Enforcement

????

### <a name="Renum-value"></a>Enum.8: 열거체들의 값은 필요할때만 명시하라

##### Reason

이는 가장 단순한 방법이며, 열거값의 중복을 방지한다. 열거체들이 기본값을 사용하면 연속된 값을 사용하게 되므로 `switch`구문의 구현에 좋다.

##### Example

```c++
    enum class Col1 { red, yellow, blue };
    enum class Col2 { red = 1, yellow = 2, blue = 2 }; // typo

    enum class Month { jan = 1, feb, mar, apr, may, jun,
                       jul, august, sep, oct, nov, dec };
                       // 달력은 1월부터 시작한다

    enum class Base_flag {
        dec = 1,
        oct = dec << 1,
        hex = dec << 2
    };
    // 비트들의 집합

```

(`Month` 와 같이) 관습적인 값이 필요한 경우, 또는 (`Base_flag`의 특정 비트 처럼)연속적인 값이 적합하지 않은 경우 값을 명시하라.

##### Enforcement

* 중복된 열거값에는 지적하라
* 연속적인 열거값을 명시하는 경우는 지적하라
