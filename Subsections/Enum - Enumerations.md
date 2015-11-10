# <a name="S-enum"></a> Enum: Enumerations

열거형은 특정 정수값 집합을 정하고 타입 정의를 위해 사용한다.
단순 `enum`과 `class enum` 두 종류 열거형이 있다.
>Enumerations are used to define sets of integer values and for defining types for such sets of values. There are two kind of enumerations, "plain" `enum`s and `class enum`s.

열거형 규칙 정리:
>Enumeration rule summary:

* [Enum.1: 메크로보다 enum을 선호하라.](#Renum-macro)
* [Enum.2: 상수 집합을 정의하는데 열거형을 사용하라.](#Renum-set)
* [Enum.3: Prefer class enums over "plain" enums](#Renum-class)
* [Enum.4: 단순 열거형보다 클래스 열거형을 선호하라.](#Renum-oper)
* [Enum.5: 열거형 상수에 전체 대문자를 사용하지 마라.](#Renum-caps)
* [Enum.6: 이름없는 열거형을 사용하라.](#Renum-unnamed)
* ???

### <a name="Renum-macro"></a> Enum.1: 메크로보다 enum을 선호하라.
>### <a name="Renum-macro"></a> Enum.1: Prefer enums over macros

##### Reason

메크로는 변수 유효영역과 타입 규칙이 없기 때문에.
>Macros do not obey scope and type rules.

##### Example

나쁜 코드:
>First some bad old code:

    // webcolors.h (third party header)
    #define RED   0xFF0000
    #define GREEN 0x00FF00
    #define BLUE  0x0000FF

    // productinfo.h
    // The following define product subtypes based on color
    #define RED    0
    #define PURPLE 1
    #define BLUE   2

    int webby = BLUE;   // webby == 2; probably not what was desired

`enum`을 사용해라:
>instead use an `enum`:

    enum class Webcolor { red = 0xFF0000, green = 0x00FF00, blue = 0x0000FF };
    enum class Productinfo { red = 0, purple = 1, blue = 2 };

    int webby = blue;   // error: be specific
    Webcolor webby = Webcolor::blue;

##### Enforcement

정수값을 정의하는 메크로를 더이상 사용하지 말자.
>Flag macros that define integer values

### <a name="Renum-set"></a> Enum.2: 상수 집합을 정의하는데 열거형을 사용하라.
>### <a name="Renum-set"></a> Enum.2: Use enumerations to represent sets of named constants

##### Reason

관련된 것을 묶을 수 있고 타입으로 이름 붙일 수 있다.
>An enumeration shows the enumerators to be related and can be a named type

##### Example

    enum class Webcolor { red = 0xFF0000, green = 0x00FF00, blue = 0x0000FF };

##### Enforcement

???

### <a name="Renum-class"></a> Enum.3: 단순 열거형보다 클래스 열거형을 선호하라.
>### <a name="Renum-class"></a> Enum.3: Prefer class enums over "plain" enums

##### Reason

뜻하지 않은 오류를 줄이기 위해서.
>To minimize surprises.

##### Example

    enum Webcolor { red = 0xFF0000, green = 0x00FF00, blue = 0x0000FF };
    enum Productinfo { red=0, purple=1, blue=2 };

    int webby = blue;   // error, ambiguous: be specific
    Webcolor webby = Webcolor::blue;

`enum class`를 사용해라:
>instead use an `enum class`:

    enum class Webcolor { red=0xFF0000, green=0x00FF00, blue=0x0000FF };
    enum class Productinfo { red=0, purple=1, blue=2 };

    int webby = blue;   // error: blue undefined in this scope
    Webcolor webby = Webcolor::blue;

##### Enforcement

???

### <a name="Renum-oper"></a> Enum.4: 안전하고 쉬운 사용을 위해 열거형에 대한 연산을 정의하라.
>### <a name="Renum-oper"></a> Enum.4: Define operations on enumerations for safe and simple use

##### Reason

사용 편의성과 에러를 줄이기 위해.
>Convenience of use and avoidance of errors.

##### Example

    ???

##### Enforcement

???

### <a name="Renum-caps"></a> Enum.5: 열거형 상수에 전체 대문자를 사용하지 마라.
>### <a name="Renum-caps"></a> Enum.5: Don't use `ALL_CAPS` for enumerators

##### Reason

메크로와 충돌을 피하기 위해.
>Avoid clashes with macros.

##### Example

    ???

##### Enforcement

???

### <a name="Renum-unnamed"></a> Enum.6: 이름없는 열거형을 사용하라.
>### <a name="Renum-unnamed"></a> Enum.6: Use unnamed enumerations for ???

##### Reason

???

##### Example

    ???

##### Enforcement

???
