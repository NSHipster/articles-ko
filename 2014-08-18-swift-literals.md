---
title: Swift Literals
author: Mattt
translator: 김필권
category: Swift
tags: swift
excerpt: >-
  Literals are representations of values in source code.
  The different kinds of literals that Swift provides ---
  and how it makes them available ---
  has a profound impact on how we write and think about code.
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


Literals are so ingrained in a developer's mental model of programming that most of us don't actively consider what the compiler is actually doing.


Having a shorthand for these essential building blocks makes code easier to both read and write.


## How Literals Work


Literals are like words: their meaning can change depending on the surrounding context.


```swift
["h", "e", "l", "l", "o"] // Array<String>
["h" as Character, "e", "l", "l", "o"] // Array<Character>
["h", "e", "l", "l", "o"] as Set<Character>
```


In the example above, we see that an array literal containing string literals is initialized to an array of strings by default.

However, if we explicitly cast the first array element as `Character`, the literal is initialized as an array of characters.

Alternatively, we could cast the entire expression as `Set<Character>` to initialize a set of characters.


_How does this work?_


In Swift, the compiler decides how to initialize literals by looking at all the visible types that implement the corresponding <dfn>literal expression protocol</dfn>.


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


To conform to a protocol, a type must implement its required initializer.

For example, the `ExpressibleByIntegerLiteral` protocol requires `init(integerLiteral:)`.


What's really great about this approach is that it lets you add literal initialization for your own custom types.


## Supporting Literal Initialization for Custom Types


Supporting initialization by literals when appropriate can significantly improve the ergonomics of custom types, making them feel like they're built-in.


For example, if you wanted to support [fuzzy logic](https://en.wikipedia.org/wiki/Fuzzy_logic), in addition to standard Boolean fare, you might implement a `Fuzzy` type like the following:


```swift
struct Fuzzy: Equatable {
    var value: Double

    init(_ value: Double) {
        precondition(value >= 0.0 && value <= 1.0)
        self.value = value
    }
}
```


A `Fuzzy` value represents a truth value that ranges between completely true and completely false over the numeric range 0 to 1 (inclusive).

That is, a value of 1 means completely true, 0.8 means mostly true, and 0.1 means mostly false.


In order to work more conveniently with standard Boolean logic, we can extend `Fuzzy` to adopt the `ExpressibleByBooleanLiteral` protocol.


```swift
extension Fuzzy: ExpressibleByBooleanLiteral {
    init(booleanLiteral value: Bool) {
        self.init(value ? 1.0 : 0.0)
    }
}
```


> In practice, there aren't many situations in which it'd be appropriate for a type to be initialized using Boolean literals.

> Support for string, integer, and floating-point literals are much more common.


Doing so doesn't change the default meaning of `true` or `false`.
We don't have to worry about existing code breaking just because we introduced the concept of half-truths to our code base ("_view did appear animated... maybe?_").

The only situations in which `true` or `false` initialize a `Fuzzy` value would be when the compiler could infer the type to be `Fuzzy`:


```swift
true is Bool // true
true is Fuzzy // false

(true as Fuzzy) is Fuzzy // true
(false as Fuzzy).value // 0.0
```


Because `Fuzzy` is initialized with a single `Double` value, it's reasonable to allow values to be initialized with floating-point literals as well.

It's hard to think of any situations in which a type would support floating-point literals but not integer literals, so we should do that too (however, the converse isn't true; there are plenty of types that work with integer but not floating point numbers).


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


With these protocols adopted, the `Fuzzy` type now looks and feels like a _bona fide_ member of Swift standard library.


```swift
let completelyTrue: Fuzzy = true
let mostlyTrue: Fuzzy = 0.8
let mostlyFalse: Fuzzy = 0.1
```


(Now the only thing left to do is implement the standard logical operators!)


If convenience and developer productivity is something you want to optimize for, you should consider implementing whichever literal protocols are appropriate for your custom types.


## Future Developments


Literals are an active topic of discussion for the future of the language.

Looking forward to Swift 5, there are a number of current proposals that could have terrific implications for how we write code.


### Raw String Literals


At the time of writing, [Swift Evolution proposal 0200](https://github.com/apple/swift-evolution/blob/master/proposals/0200-raw-string-escaping.md) is in active review.

If it's accepted, future versions of Swift will support "raw" strings, or string literals that ignores escape sequences.


From the proposal:


> Our design adds customizable string delimiters.

> You may pad a string literal with one or more `#` (pound, Number Sign, U+0023) characters [...]

> The number of pound signs at the start of the string (in these examples, zero, one, and four) must match the number of pound signs at the end of the string.


```swift
"This is a Swift string literal"

#"This is also a Swift string literal"#

####"So is this"####
```


This proposal comes as a natural extension of the new multi-line string literals added in Swift 4 ([SE-0165](https://github.com/apple/swift-evolution/blob/master/proposals/0168-multi-line-string-literals.md)), and would make it even easier to do work with data formats like JSON and XML.


If nothing else, adoption of this proposal could remove the largest obstacle to using Swift on Windows: dealing with file paths like `C:\Windows\All Users\Application Data`.


### Literal Initialization Via Coercion


Another recent proposal, [SE-0213: Literal initialization via coercion](https://github.com/apple/swift-evolution/blob/master/proposals/0213-literal-init-via-coercion.md) is already implemented for Swift 5.


From the proposal:


> `T(literal)` should construct `T` using the appropriate literal protocol if possible.

> Currently types conforming to literal protocols are type-checked using regular initializer rules, which means that for expressions like `UInt32(42)` the type-checker is going to look up a set of available initializer choices and attempt them one-by-one trying to deduce the best solution.


In Swift 4.2, initializing a `UInt64` with its maximum value results in a compile-time overflow because the compiler first tries to initialize an `Int` with the literal value.


```swift
UInt64(0xffff_ffff_ffff_ffff) // overflows in Swift 4.2
```


Starting in Swift 5, not only will this expression compile successfully, but it'll do so a little bit faster, too.


---


The words available to a language speaker influence not only what they say, but how they think as well.

In the same way, the individual parts of a programming language hold considerable influence over how a developer works.


The way Swift carves up the semantic space of values makes it different from languages that don't, for example, distinguish between integers and floating points or have separate concepts for strings, characters, and Unicode scalars.

So it's no coincidence that when we write Swift code, we often think about numbers and strings at a lower level than if we were hacking away in, say, JavaScript.


Along the same lines, Swift's current lack of distinction between string literals and regular expressions contributes to the relative lack of regex usage compared to other scripting languages.


That's not to say that having or lacking certain words makes it impossible to express certain ideas --- just a bit fuzzier.

We can understand "untranslatable" words like ["Saudade"](https://en.wikipedia.org/wiki/Saudade) in Portuguese, ["Han"](https://en.wikipedia.org/wiki/Han_%28cultural%29) in Korean, or ["Weltschmerz"](https://en.wikipedia.org/wiki/Weltschmerz) in German.


We're all human.

We all understand pain.


By allowing any type to support literal initialization, Swift invites us to be part of the greater conversation.

Take advantage of this and make your own code feel like a natural extension of the standard library.
