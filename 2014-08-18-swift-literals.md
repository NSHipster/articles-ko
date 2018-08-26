---
title: Swift Literals
author: Mattt
translator: 김필권
category: Swift
tags: swift
excerpt: "리터럴은 소스 코드의 값을 표현한 것입니다. Swift가 제공하는 다양한 종류의 리터럴과 그들을 사용가능하게 만든 방법은 우리가 코드를 작성하고 생각하는 방법에 엄청난 영향을 줄 것입니다."
status:
  swift: 4.2
---

1911년, 언어학자인 [Franz Boas](https://en.wikipedia.org/wiki/Franz_Boas)는 [에스키모-알류트 언어](https://en.wikipedia.org/wiki/Eskimo–Aleut_languages)를 사용하는 사람들이 땅 위에 있는 눈과 떨어지고 있는 눈송이를 구별하기 위해 다른 단어를 사용한다는 사실을 알았습니다. 반대로 영어를 사용하는 사람들은 보통 그 둘을 모두 "snow"라고 부르지만 빗방울과 웅덩이를 구별하는 언어를 만들었습니다.

시간이 지나서 이 간단한 경험적인 관찰은 다음과 같은 끔찍하고 진부한 표현으로 변형되었습니다. "에스키모들은 눈을 표현하는 50가지 다른 단어를 사용한다." 불행히도 Boas의 관찰은 경험에 의거한 것이었고 결과로 나온 다음의 언어 상대성에 대한 약한 주장은 논란의 여지가 없습니다 : 언어는 의미적 개념을 서로 다른 방식으로 독립적인 단어로 나눕니다. 그것이 역사적인 사고였거나 문화에 대한 더 깊은 진실을 반영한 것인지는 불분명합니다. 이것은 추가 토론을 위한 주제가 될 것입니다.

오늘은 이 틀에서 Swift의 다양한 리터럴들이 어떻게 우리가 코드에 대해 생각하는 방식을 형성하는지 생각할 수 있을 것입니다.

---

<dfn>리터럴</dfn>은 넘버나 스트링같이 소스 코드의 값을 대표하는 것입니다.

Swift는 다음과 같은 종류의 리터럴을 제공합니다.

| Name                      | Default Inferred Type | Examples                          |
| ------------------------- | --------------------- | --------------------------------- |
| Integer                   | `Int`                 | `123`, `0b1010`, `0o644`, `0xFF`, |
| Floating-Point            | `Double`              | `3.14`, `6.02e23`, `0xAp-2`       |
| String                    | `String`              | `"Hello"`, `""" . . . """`        |
| Extended Grapheme Cluster | `Character`           | `"A"`, `"é"`, `"🇺🇸"`              |
| Unicode Scalar            | `Unicode.Scalar`      | `"A"`, `"´"`, `"\u{1F1FA}"`       |
| Boolean                   | `Bool`                | `true`, `false`                   |
| Nil                       | `Optional`            | `nil`                             |
| Array                     | `Array`               | `[1, 2, 3]`                       |
| Dictionary                | `Dictionary`          | `["a": 1, "b": 2]`                |

Swift의 리터럴을 이해하는데 가장 중요한 것은 그것들이 절대적인 타입이 아닌 값을 대표한다는 것입니다.

컴파일러가 리터럴을 만나면 자동으로 타입으로 추론하려고 합니다. 컴파일러는 그 리터럴 종류로 초기화할 수 있는 모든 타입을 찾아보고 다른 제약조건을 추가하면서 좁혀갑니다.

어떤 타입도 추론되지 않으면 Swift는 그 리터럴 종류에 대한 디폴트 타입을 초기화합니다. `Int`는 정수 리터럴, `String`은 스트링 리터럴처럼요.

```swift
57 // Integer literal
"Hello" // String literal
```

`nil` 리터럴의 경우엔 절대 자동으로 타입을 추론할 수 없기때문에 정의해주어야 합니다.

```swift
nil // ! cannot infer type
nil as String? // Optional<String>.none
```

배열과 딕셔너리 리터럴은 컬렉션에 연관된 타입들이 그것의 컨텐츠에 기반해서 추론됩니다. 그러나 크기가 크거나 중첩된 컬렉션의 타입을 추론하는 것은 복잡한 연산이고 여러분의 코드를 컴파일하는 총 시간을 엄청나게 증가시킬 것입니다. 정의에 분명한 타입을 추가하면 산뜻하게 유지할 수 있습니다.

```swift
// Explicit type in the declaration
// prevents expensive type inference during compilation
let dictionary: [String: [Int]] = [
    "a": [1, 2],
    "b": [3, 4],
    "c": [5, 6],
    // ...
]
```

### 플레이그라운드 리터럴

위에 나열된 표준 리터럴 외에도 플레이그라운드에서 코드에 대한 몇 가지 추가 리러털 유형이 있습니다.

| Name  | Default Inferred Type | Examples                                             |
| ----- | --------------------- | ---------------------------------------------------- |
| Color | `NSColor` / `UIColor` | `#colorLiteral(red: 1, green: 0, blue: 1, alpha: 1)` |
| Image | `NSImage` / `UIImage` | `#imageLiteral(resourceName: "icon")`                |
| File  | `URL`                 | `#fileLiteral(resourceName: "articles.json")`        |

Xcode나 iPad의 Swift 플레이그라운드에서 이 # 접두사 리터럴 표현식은 참조된 색상, 이미지 또는 파일의 시각적 표현을 제공하는 상호작용 컨트롤로 자동 대체됩니다.

```swift
// Code
#colorLiteral(red: 0.7477839589, green: 0.5598286986, blue: 0.4095913172, alpha: 1)

// Rendering
🏽
```

{% asset color-literal-picker.png %}

이 컨트롤은 새로운 값을 고를 경우도 편하게 만들어줍니다. RGBA 값이나 파일 주소를 입력하는 것 대신에 컬러 픽커와 파일 선택창을 제공해줍니다.

---

대부분의 프로그래밍 언어가 Boolean, 숫자, 스트링에 대한 리터럴을 가지고 있고, 배열, 딕셔너리 그리고 정규 표현식에 대한 리터럴도 많이 가지고 있습니다.

리터럴은 개발자들의 프로그래밍에 대한 멘탈 모델에 뿌리깊이 박혀있습니다. 우리 중 대부분은 컴파일러가 정확히 어떤 일을 하는지 제대로 생각하지는 않습니다.

이러한 필수 구성 요소들의 약칭을 사용하면 코드를 읽고 쓰기 쉽게 만들 수 있습니다.

## 리터럴이 작동하는 방식

리터럴은 단어와도 같습니다. 그들의 의미는 주위의 문맥에 따라서 바뀔 수 있기 때문입니다.

```swift
["h", "e", "l", "l", "o"] // Array<String>
["h" as Character, "e", "l", "l", "o"] // Array<Character>
["h", "e", "l", "l", "o"] as Set<Character>
```

위의 예제를 보면 스트링 리터럴을 포함한 배열 리터럴은 기본적으로 스트링의 배열로 초기화된다는 것을 볼 수 있습니다. 하지만 우리가 만약 첫 번째 요소를 `Character` 로 명확하게 지정한다면 리터럴은 문자의 배열로 초기화할 것입니다. 또 다른 방법으로는 배열 자체를 `Set<Character>` 로 캐스팅할 수도 있습니다.

_어떻게 이렇게 작동하는거죠?_

Swift에선 컴파일러가 <dfn>리터럴 표현 프로토콜</dfn>에 대응하게 구현된 타입들을 보며 리터럴을 어떻게 초기화할지 결정합니다.

| Literal                   | Protocol                                      |
| ------------------------- | --------------------------------------------- |
| Integer                   | `ExpressibleByIntegerLiteral`                 |
| Floating-Point            | `ExpressibleByFloatLiteral`                   |
| String                    | `ExpressibleByStringLiteral`                  |
| Extended Grapheme Cluster | `ExpressibleByExtendedGraphemeClusterLiteral` |
| Unicode Scalar            | `ExpressibleByUnicodeScalarLiteral`           |
| Boolean                   | `ExpressibleByBooleanLiteral`                 |
| Nil                       | `ExpressibleByNilLiteral`                     |
| Array                     | `ExpressibleByArrayLiteral`                   |
| Dictionary                | `ExpressibleByDictionaryLiteral`              |

프로토콜에 따르려면 타입이 반드시 필수 이니셜라이져를 구현해야 합니다. 예를 들면 `ExpressibleByIntegerLiteral` 프로토콜은 `init(integerLiteral:)`을 필수로 합니다.

이 접근에서 정말 훌륭한 점은 이것이 여러분만의 커스텀 타입에 리터럴 이니셜라이져를 추가하게 해준다는 것입니다.

## 커스텀 타입으로 리터럴 이니셜라이져 지원하기

적절한 경우에 리터럴로 초기화를 지원하면 커스텀 타입의 인체 공학을 향상시켜주고 내장돼있는 것과 같은 기분을 느끼게 해줍니다.

예를 들어 [fuzzy logic](https://en.wikipedia.org/wiki/Fuzzy_logic)을 지원하고 싶으면 다음과 같이 `Fuzzy` 타입을 구현해야 할 것입니다.

```swift
struct Fuzzy: Equatable {
    var value: Double

    init(_ value: Double) {
        precondition(value >= 0.0 && value <= 1.0)
        self.value = value
    }
}
```

`Fuzzy` 는 완전한 거짓(숫자 0)에서 완전한 참(숫자 1) 사이의 값을 표시합니다. 그 말은 숫자 1은 완전한 참, 0.8은 거의 참, 0.1은 거의 거짓임을 의미한다는 것입니다.

표준 Boolean 로직으로 더 편하게 작업하기 위해서 우리는 `Fuzzy` 가 `ExpressibleByBooleanLiteral` 프로토콜에 적응하도록 확장해야 합니다.

```swift
extension Fuzzy: ExpressibleByBooleanLiteral {
    init(booleanLiteral value: Bool) {
        self.init(value ? 1.0 : 0.0)
    }
}
```

> 실제로 Boolean 리터럴을 사용해서 타입을 초기화해야하는 경우는 많지 않습니다.
> 스트링, 정수 그리고 부동 소수점 리터럴이 더 보통의 경우입니다.

이렇게 하는 것이 `true`와 `false`의 기본적인 의미를 해치는 것은 아닙니다.
Doing so doesn't change the default meaning of `true` or `false`.
우리가 소개한 반-진실 개념이 기존의 코드를 부술 걱정은 하지 않아도 됩니다.
We don't have to worry about existing code breaking just because we introduced the concept of half-truths to our code base ("_view did appear animated... maybe?_").
`true` 또는 `false`가 `Fuzzy` 값을 초기화하는 유일한 상황은 컴파일러가 그 타입을 `Fuzzy` 라고 추론할 때만 입니다.

```swift
true is Bool // true
true is Fuzzy // false

(true as Fuzzy) is Fuzzy // true
(false as Fuzzy).value // 0.0
```

`Fuzzy` 는 하나의 `Double` 값으로 초기화하기 때문에 부동 소수점 리터럴을 사용해서도 값을 초기화할 수 있습니다.
Because `Fuzzy` is initialized with a single `Double` value, it's reasonable to allow values to be initialized with floating-point literals as well.
부동 소수점은 지원하지만 정수를 지원하지 않는 타입을 생각하는 것은 어렵습니다. 그러니 정수도 지원하도록 만들어보겠습니다. (하지만 그 반대는 참이 아닙니다. 정수는 지원하는데 부동 소수점을 지원하지 않는 경우는 정말 많습니다.)

```swift
extension Fuzzy: ExpressibleByIntegerLiteral {
    init(integerLiteral value: Int) {
        self.init(Double(value))
    }
}

extension Fuzzy: ExpressibleByFloatLiteral {
    init(floatLiteral value: Double) {
        self.init(value)
    }
}
```

이러한 프로토콜 적응을 통해 `Fuzzy` 타입은 이제 Swift 표준 라이브러리의 _bona fide(진실한)_ 멤버로 보이게 되었습니다.

```swift
let completelyTrue: Fuzzy = true
let mostlyTrue: Fuzzy = 0.8
let mostlyFalse: Fuzzy = 0.1
```

(이제 유일하게 남은 것은 표준 논리적 연산자를 구현하는 것입니다!)

편의성과 개발자 생산성이 여러분이 최적화하기 위한 목적이라면 커스텀 타입에 적합한 리터럴 프로콜을 구현하는 것을 고려해야합니다.

## 미래의 개발

리터럴은 미래의 언어를 위한 토론에서 빠지지 않는 주제입니다. Swift 5를 기대하는 이유는 우리가 코드를 작성하는 데에 있어서 엄청난 변화를 가져올 제안들이 몇가지 있기 때문입니다.

### 로우 스트링 리터럴

글을 작성하고 있는 지금은 [Swift 발전 제안 0200](https://github.com/apple/swift-evolution/blob/master/proposals/0200-raw-string-escaping.md) 가 리뷰 중입니다. 이것이 수락되면 Swift의 미래 버전은 "로우(raw)" 스트링 또는 이스케이프 시퀀스를 무시하는 스트링 리터럴을 지원하게 될 것입니다.

제안에 의하면 :

> 우리의 디자인은 스트링 구획 문자(delimiter)를 추가합니다.
> 하나 이상의 `#` (파운드, 숫자 기호, U+0023) 문자로 문자열 리터럴을 덧붙일 수 있습니다.
> 스트링의 시작에 있는 # 기호의 수는 스트링 끝에 있는 # 기호의 수와 반드시 똑같아야 합니다.

```swift
"This is a Swift string literal"

#"This is also a Swift string literal"#

####"So is this"####
```

이 제안([SE-0165](https://github.com/apple/swift-evolution/blob/master/proposals/0168-multi-line-string-literals.md))은 Swift 4의 새로운 멀티 라인 스트링 리터럴의 자연스러운 익스텐션입니다. 그리고 이는 JSON 이나 XML 같은 데이터 포맷을 작업하기 더 쉽게 만들어 줄 것입니다.

적어도 이 제안을 채택하면 `C:\Windows\All Users\Application Data` 와 같은 파일 경로를 다루는 Windows에서 Swift를 사용하는 데 있는 가장 큰 장애물을 제거할 수 있을 것입니다.

### Coercion을 통해서 리터럴 초기화하기

또 다른 최근 제안인 [SE-0213: Literal initialization via coercion](https://github.com/apple/swift-evolution/blob/master/proposals/0213-literal-init-via-coercion.md)는 이미 Swift 5에 구현돼 있습니다.

제안에 의하면 :

> `T(literal)` 는 가능하면 적절한 리터럴 프로토콜을 사용해서 `T`를 구성해야 합니다.
> 현재 리터럴 프로토콜을 준수하는 타입은 정규 이니셜라이져 규칙을 사용해서 타입 검사되고 있습니다. `UInt32(42)` 와 같은 표현이 있다면 타입 확인기는 사용가능한 이니셜라이져를 둘러보고 하나하나 맞춰보며 가장 최고의 해결책을 추론하려고 합니다.

Swift 4.2에선 `UInt64` 를 그것의 최대 값으로 초기화하는 것은 컴파일 할 때 결과적으로 오버플로우를 일으킵니다. 왜냐하면 컴파일러는 먼저 `Int` 를 리터럴 값으로 초기화하려고 하기 때문입니다.

```swift
UInt64(0xffff_ffff_ffff_ffff) // Swift 4.2에서 오버플로우
```

Swift 5부터 이 표현이 성공적으로 성공할 뿐만 아니라 더 빨라진다고 합니다.

---

언어 사용자가 사용할 수 있는 단어는 그들이 말한 것 뿐만 아니라 어떻게 생각하는지에 영향을 미칩니다. 같은 방식으로 프로그래밍 언어의 각 부분은 개발자의 작업 방식에 상당한 영향을 미칩니다.

Swift가 값의 의미 공간을 만드는 방식은 그러지 않는 언어들과는 다릅니다. 예를 들어 정수와 부동 소수점을 구별하는 것이나 스트링, 문자 그리고 유니코드의 개념을 나누는 것이 있습니다. 그러니 Swift 코드를 작성할 때 더 낮은 레벨에서 숫자와 문자열을 생각하는 것이 우연이 아닙니다.

Swift는 현재 문자열 리터럴과 정규 표현식을 구별하지 못하고 있기 때문에 다른 스크립트 언어와 비교해서 정규표현식 사용이 상대적으로 불편합니다.

특정 단어가 없거나 부족하다는 말은 어떤 아이디어가 있을 때 그것을 표현하기 어렵게 만듭니다. 우리는 "번역할 수 없는" 단어를 이해할 수 있기도 합니다. 예를 들면 포르투칼어의 ["Saudade"](https://en.wikipedia.org/wiki/Saudade), 한국어의 ["Han"](https://en.wikipedia.org/wiki/Han_%28cultural%29), 독일어의 ["Weltschmerz"](https://en.wikipedia.org/wiki/Weltschmerz) 처럼요.

우린 모두 사람이고 고통을 이해할 수 있기 때문입니다.

리터럴 초기화를 지원하는 모든 타입을 허용하면서 Swift는 우리에게 더 나은 세상의 일부가 되도록 초대합니다. 이를 활용해서 표준 라이브러리의 자연스러운 익스텐션같은 자신만의 코드를 만들 수 있습니다.
