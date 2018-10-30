---
title: numericCast(_:)
author: Mattt
translator: 김필권
category: Swift
excerpt: >
  Getting code to compile is different than doing things correctly.
  But sometimes it takes the former to ultimately get to the latter.
status:
  swift: 4.2
---

누구나 프로그래밍을 설명할 때 주로 사용하는 비유가 있을 것입니다.

목공부터 뜨개질, 정원 가꾸기까지 다양하게 비유할 것입니다.
게다가 프로그래밍은 문제 해결, 스토리텔링, 예술 작품으로도 비유할 수 있을 것입니다.
작문에 비유한다면 그 프로그램이 시인지 산문인지 문제는 의심의 여지도 없습니다.
또는 프로그래밍이 음악이라면 분명히 재즈일 것입니다.

지금 우리가 하고 있는 얘기는 중동지역의 이야기인 _천일야화_(The Thousand and One Nights)와 가장 비슷하다고 생각합니다. 천일야화의 아무 이야기 하나를 읽어보면 초자연적인 존재인 지니(<dfn>jinn</dfn>, <dfn>djinn</dfn>, <dfn>genies</dfn> 또는 🧞‍)를 볼 수 있을 것입니다.
이 존재를 뭐라고 부르든 우리는 이 존재가 소원을 이뤄주고 그에 필연적으로 따라오는 불행을 알고 있습니다.

많은 방면에서 컴퓨터는 형이상학적 소원을 물리적으로 구체화해서 성취해줍니다.
지니처럼 컴퓨터는 우리의 의도가 무엇인지에 상관없이 기쁘게 받아들이고 무엇이든 할 것입니다.
그리고 에러가 발생하기 전까지 우리는 그것에 대해 아무것도 할 수 없을 수도 있습니다.

Swift 개발자라면 정수형 변환 에러를 본 적이 있을 것입니다. 저는 이 에러를 볼 때 마다 "이 경고들 좀 사라지고 내 코드도 컴파일되면 좋겠네"라고 생각합니다.

여러분도 그런 적이 있으시다면 `numericCast(_:)` 을 알기 딱 좋은 타이밍이십니다. `numericCast(_:)` 는 Swift 표준 라이브러리의 작은 유틸리티 기능이지만 우리가 원하는 딱 그 기능입니다.
하지만 조심하세요 이건 그저 실현되는 것 뿐이니까요.

---

`numericCast(_:)` 가 [어떻게 생겼는지 알아보고](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Integers.swift#L3508-L3510) 이에 걸려있는 마법적인 상상을 풀어보겠습니다.

```swift
public func numericCast<T : BinaryInteger, U : BinaryInteger>(_ x: T) -> U {
  return U(x)
}
```

([`Never`](/never)에서도 배웠듯이 코드가 많다고 많은 충격을 주는 것이 아니고 코드가 적다고 주는 충격이 작은게 아닙니다)

[`BinaryInteger`](https://developer.apple.com/documentation/swift/binaryinteger) 프로토콜은 숫자들이 언어안에서 어떻게 작동되고 있는지 검사하기 위해 Swift 4에서 추가되었습니다.
`BinaryInteger` 는 signed, unsigned에 상관없이 모든 모양과 모든 사이즈의 정수를 다루는 하나의 통합된 인터페이스를 제공합니다.

정수형을 다른 타입으로 변형할 때는 그 타입으로 표현이 안되는 값이라도 변형은 가능합니다.
문제는 signed 정수를 unsigned 정수로 변형하려고 할 경우 (예를 들어 `-42` 를 `UInt` 로 변형하려고 할 때) 또는 바꾸고자하는 타입의 한계를 넘어가는 경우(예를 들어 `UInt8` 은 `0` 부터 `255` 까지밖에 표현하지 못합니다)에 발생합니다.

`BinaryInteger` 는 변형에 대한 네 가지 전략을 세웠습니다.

- **Range-Checked Conversion** ([`init(_:)`](https://developer.apple.com/documentation/swift/binaryinteger/2885704-init)): 한계를 넘어가는 런타임 에러를 일으킵니다

- **Exact Conversion** ([`init?(exactly:)`](https://developer.apple.com/documentation/swift/binaryinteger/2925955-init)): 한계를 넘어가는 값의 경우 `nil`을 반환합니다

- **Clamping Conversion** ([`init(clamping:)`](https://developer.apple.com/documentation/swift/binaryinteger/2886143-init)): 한계를 넘어가는 값의 경우 가장 가까운 표현가능한 타입을 사용합니다

- **Bit Pattern Conversion** ([`init(truncatingIfNeeded:)`](https://developer.apple.com/documentation/swift/binaryinteger/2925529-init)): 대상으로 하는 정수형의 너비로 잘라냅니다

올바른 변형 전략은 그것이 사용되는 상황에 의존합니다.
때로는 표현가능한 범위로 조정하는 것이 옳을 때도 있고 때로는 아무 값을 주지 않는 것이 옳을 때도 있습니다.
`numericCast(_:)` 의 경우엔 편의를 위해 range-checked conversion이 사용됩니다.
한계를 넘는 값을 이 함수를 호출하는데에 사용한다면 런타임 에러가 날 수 있다는 것이 단점입니다. (구체적으로는 `-0` 과 `-0none` 의 경우에 오버플로우에 걸립니다)

{% info %}


For more information about the changes to how numbers work in Swift 4, see [SE-0104: "Protocol-oriented integers"](https://github.com/apple/swift-evolution/blob/master/proposals/0104-improved-integers.md).


This subject is also discussed at length in the [Flight School Guide to Numbers](https://gumroad.com/l/swift-numbers).

{% endinfo %}


## Thinking Literally, Thinking Critically


Before we go any further, let's take a moment to talk about integer literals.


[As we've discussed in previous articles](https://nshipster.com/swift-literals/), Swift provides a convenient and extensible way to represent values in source code.
When used in combination with the language's use of type inference, things often "just work" ...which is nice and all, but can be confusing when things "just don't".


Consider the following example in which arrays of signed and unsigned integers are initialized from identical literal values:

```swift
let arrayOfInt: [Int] = [1, 2, 3]
let arrayOfUInt: [UInt] = [1, 2, 3]
```


Despite their seeming equivalence, we can't, for example, do this:

```swift
arrayOfInt as [UInt] // Error: Cannot convert value of type '[Int]' to type '[UInt]' in coercion
```


One way to reconcile this issue would be to pass the `numericCast` function as an argument to `map(_:)`:

```swift
arrayOfInt.map(numericCast) as [UInt]
```


This is equivalent to passing the `UInt` range-checked initializer directly:

```swift
arrayOfInt.map(UInt.init)
```


But let's take another look at that example, this time using slightly different values:

```swift
let arrayOfNegativeInt: [Int] = [-1, -2, -3]
arrayOfNegativeInt.map(numericCast) as [UInt] // 🧞‍ Fatal error: Negative value is not representable
```


As a run-time approximation of compile-time type functionality `numericCast(_:)` is closer to `as!` than `as` or `as?`.


Compare this to what happens if you instead pass the exact conversion initializer, `init?(exactly:)`:

```swift
let arrayOfNegativeInt: [Int] = [-1, -2, -3]
arrayOfNegativeInt.map(UInt.init(exactly:)) // [nil, nil, nil]
```


`numericCast(_:)`, like its underlying range-checked conversion, is a blunt instrument, and it's important to understand what tradeoffs you're making when you decide to use it.


## The Cost of Being Right


In Swift, the general guidance is to use `Int` for integer values (and `Double` for floating-point values) unless there's a _really_ good reason to use a more specific type.
Even though the `count` of a `Collection` is nonnegative by definition, we use `Int` instead of `UInt` because the cost of going back and forth between types when interacting with other APIs outweighs the potential benefit of a more precise type.
For the same reason, it's almost always better to represent even small numbers, like [weekday numbers](/datecomponents),
with an `Int`, despite the fact that any possible value would fit into an 8-bit integer with plenty of room to spare.


The best argument for this practice is a 5-minute conversation with a C API from Swift.


Older and lower-level C APIs are rife with architecture-dependent type definitions and finely-tuned value storage.
On their own, they're manageable.
But on top of all the other inter-operability woes like headers to pointers, they can be a breaking point for some (and I don't mean the debugging kind).


`numericCast(_:)` is there for when you're tired of seeing red and just want to get things to compile.


## Random Acts of Compiling


The [example in the official docs](https://developer.apple.com/documentation/swift/2884564-numericcast) should be familiar to many of us:


Prior to [SE-0202](https://github.com/apple/swift-evolution/blob/master/proposals/0202-random-unification.md), the standard practice for generating numbers in Swift (on Apple platforms) involved importing the `Darwin` framework and calling the `arc4random_uniform(3)` function:

```c
uint32_t arc4random_uniform(uint32_t __ upper_bound)
```


`arc4random` requires not one but two separate type conversions in Swift: first for the upper bound parameter (`Int` → `UInt32`) and second for the return value (`UInt32` → `Int`):

```swift
import Darwin

func random(in range: Range<Int>) -> Int {
    return Int(arc4random_uniform(UInt32(range.count))) + range.lowerBound
}
```


_Gross._


By using `numericCast(_:)`, we can make things a little more readable, albeit longer:

```swift
import Darwin

func random(in range: Range<Int>) -> Int {
    return numericCast(arc4random_uniform(numericCast(range.count))) + range.lowerBound
}
```


`numericCast(_:)` isn't doing anything here that couldn't otherwise be accomplished with type-appropriate initializers.
Instead, it serves as an indicator that the conversion is perfunctory --- the minimum of what's necessary to get the code to compile.


But as we've learned from our run-ins with genies, we should be careful what we wish for.


Upon closer inspection, it's apparent that the example usage of `numericCast(_:)` has a critical flaw:
_it traps on values that exceed `UInt32.max`!_

```swift
random(in: 0..<0x1_0000_0000) // 🧞‍ Fatal error: Not enough bits to represent the passed value
```


If we [look at the Standard Library implementation](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Integers.swift#L2537-L2560) that now lets us do `Int.random(in: 0...10)`, we'll see that it uses clamping, rather than range-checked, conversion.
And instead of delegating to a convenience function like `arc4random_uniform`, it [populates values from a buffer of random bytes](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Random.swift#L156-L177).

---


Getting code to compile is different than doing things correctly.
But sometimes it takes the former to ultimately get to the latter.
When used judiciously, `numericCast(_:)` is a convenient tool to resolve issues quickly.
It also has the added benefit of signaling potential misbehavior more clearly than a conventional type initializer.


Ultimately, programming is about describing _exactly_ what we want --- often with painstaking detail.
There's no genie-equivalent CPU instruction for "Do the Right Thing"
(and even if there was,
[would we really trust it](https://github.com/FixIssue/FixCode)?)
Fortunately for us, Swift allows us to do this in a way that's safer and more concise than many other languages.
And honestly, who could wish for anything more?
