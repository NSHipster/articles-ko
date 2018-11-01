---
title: numericCast(_:)
author: Mattt
translator: 김필권
category: Swift
excerpt: >
  코드가 컴파일되도록 하는 것은 코드를 올바르게 짜는 것과는 다른 일입니다.
  하지만 때론 전자를 추구하는 것이 후자를 이루는 궁극적인 방법이 될 때도 있습니다.
status:
  swift: 4.2
---

누구나 프로그래밍을 설명할 때 주로 사용하는 비유가 있을 것입니다.

목공부터 뜨개질, 정원 가꾸기까지 다양하게 비유할 것입니다.
게다가 프로그래밍은 문제 해결, 스토리텔링, 예술 작품으로도 비유할 수 있을 것입니다.
작문에 비유한다면 그 프로그램이 시인지 산문인지 문제는 의심의 여지도 없습니다.
또는 프로그래밍이 음악이라면 분명히 재즈일 것입니다.

지금 우리가 하는 얘기는 중동지역의 이야기인 _천일야화_(The Thousand and One Nights)와 가장 비슷하다고 생각합니다. 천일야화의 아무 이야기 하나를 읽어보면 초자연적인 존재인 지니(<dfn>jinn</dfn>, <dfn>djinn</dfn>, <dfn>genies</dfn> 또는 🧞‍)를 볼 수 있을 것입니다.
이 존재를 뭐라고 부르든 우리는 이 존재가 소원을 이뤄주고 그에 필연적으로 따라오는 불행을 알고 있습니다.

많은 방면에서 컴퓨터는 형이상학적 소원을 물리적으로 구체화해서 성취해줍니다.
지니처럼 컴퓨터는 우리의 의도가 무엇인지에 상관없이 기쁘게 받아들이고, 무엇이든 할 것입니다.
그리고 에러가 발생하기 전까지 우리는 그것에 대해 아무것도 할 수 없을 수도 있습니다.

Swift 개발자라면 정수형 변환 에러를 본 적이 있을 것입니다. 저는 이 에러를 볼 때마다 "이 경고들 좀 사라지고 내 코드도 컴파일되면 좋겠네"라고 생각합니다.

여러분도 그런 적이 있으시다면 `numericCast(_:)` 을 알기 딱 좋은 타이밍이십니다. `numericCast(_:)` 는 Swift 표준 라이브러리의 작은 유틸리티 기능이지만 우리가 원하는 딱 그 기능입니다.
기억하세요. 이건 그저 실현되는 것뿐입니다.

---

`numericCast(_:)` 가 [어떻게 생겼는지 알아보고](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Integers.swift#L3508-L3510) 이에 걸려있는 마법적인 상상을 풀어보겠습니다.

```swift
public func numericCast<T : BinaryInteger, U : BinaryInteger>(_ x: T) -> U {
  return U(x)
}
```

([`Never`](/never)에서도 배웠듯이 코드가 많다고 큰 충격을 주는 것이 아니고 코드가 적다고 주는 충격이 작은 게 아닙니다)

[`BinaryInteger`](https://developer.apple.com/documentation/swift/binaryinteger) 프로토콜은 숫자들이 언어 안에서 어떻게 작동되고 있는지 검사하기 위해 Swift 4에서 추가되었습니다.
`BinaryInteger` 는 signed, unsigned에 상관없이 모든 모양과 모든 사이즈의 정수를 다루는 하나의 통합된 인터페이스를 제공합니다.

정수형을 다른 타입으로 변형할 때는 그 타입으로 표현이 안 되는 값이라도 변형은 가능합니다.
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

숫자들이 Swift 4에서 바뀌면서 어떻게 작동하게 되었는지에 대한 더 자세한 정보는 [SE-0104: "Protocol-oriented integers"](https://github.com/apple/swift-evolution/blob/master/proposals/0104-improved-integers.md)를 확인해주세요.

이 주제는 또한 [Flight School Guide to Numbers](https://gumroad.com/l/swift-numbers)에서도 길게 다뤄집니다.

{% endinfo %}


## 문자 그대로 생각하기, 비판적으로 생각하기

더 알아보기 전에 정수 리터럴에 대해 얘기하는 시간을 가져보겠습니다.

[이전 글에서도 얘기 나눴듯이](https://nshipster.com/swift-literals/) Swift는 값을 표현하는 편리하고 확장성있는 방법을 제공합니다.
Swift의 타입 추론을 사용하면 때로는 "그냥 되는" 경우가 있는데 이런 경우엔 정말 좋습니다. 하지만 "그냥 안되는" 경우라면 우리를 매우 혼란스럽게 할 것입니다.

다음과 같이 signed 정수의 배열과 unsigned 정수의 배열이 있고 같은 값으로 초기화됐다고 해봅시다.

```swift
let arrayOfInt: [Int] = [1, 2, 3]
let arrayOfUInt: [UInt] = [1, 2, 3]
```

그들이 보기에는 같아보임에도 불구하고 우리는 다음과 같은 상황을 마주하게 될 것입니다.

```swift
arrayOfInt as [UInt] // 에러: `[Int]` 타입을 `[UInt]` 타입으로 강제로 바꿀 수 없습니다
```

이 이슈를 해결하는 한 가지 방법은 `map(_:)` 메소드에 `numericCast` 함수를 인자로 넘기는 것입니다.

```swift
arrayOfInt.map(numericCast) as [UInt]
```

다음은 `UInt` range-checked initializer를 직접 보내는 방법입니다.

```swift
arrayOfInt.map(UInt.init)
```

이번엔 조금 다른 값으로 같은 예제를 살펴보겠습니다.

```swift
let arrayOfNegativeInt: [Int] = [-1, -2, -3]
arrayOfNegativeInt.map(numericCast) as [UInt] // 🧞‍ Fatal error: 음수는 표현할 수 없습니다
```

컴파일 시간 타입 기능의 런타임과 같은 `numericCast(_:)` 는 `as!` 보다는 `as` 나 `as?` 에 가깝습니다.

대신에 exact conversion initializer(`init?(exactly:)`)를 넘겼을 경우를 비교해보겠습니다.

```swift
let arrayOfNegativeInt: [Int] = [-1, -2, -3]
arrayOfNegativeInt.map(UInt.init(exactly:)) // [nil, nil, nil]
```

`numericCast(_:)` 는 무딘 도구라서 사용하기로 마음먹었을 때 무엇을 넘겨줘야하는지 이해하는 것이 중요합니다.

## The Cost of Being Right

Swift에서 정수 값에 `Int` 를 사용하도록(그리고 부동 소숫점 값에 `Double`을 사용하도록) 가이드를 주는 것은 _정말로_ 좋은 이유가 아니라면 구체적인 타입을 사용하지 않게 하기 위해서 입니다.
`Collection` 의 `count` 값이 정의상으로 무조건 0보다 클지라도 우리는 API에서 잘 못 보낼 수도 있는 경우를 생각해서 `UInt` 대신에 `Int` 를 사용해야 합니다.
동일한 이유로 아주 적은 숫자를 나타내는 경우에 8 비트 공간에 모든 값이 들어갈지라도 `Int` 를 사용하는 것이 항상 더 나은 표현 방법입니다. 예를 들면 [weekday numbers](/datecomponents)가 있겠네요.

이러한 경우는 C API와 Swift간의 의사소통에서도 볼 수 있습니다.

오래돼고 낮은 단계의 C API들은 아키텍쳐 의존 타입 정의와 미세하게 조정된 값들로 가득 차 있습니다.
그들은 스스로 관리가 가능합니다.
하지만 헤더에서 포인터와 같이 상호 운용성 문제로 올라가게 되면 이것은 브레이크 포인트가 될 수도 있습니다. (디버깅의 그것을 말하는게 아닙니다.)

`numericCast(_:)` 는 더 이상 빨간 줄을 보기 싫고 그냥 다 해결됐으면 좋겠으면 하는 우리를 위해 존재합니다.

## 컴파일의 무작위 행동

[공식 문서의 예제](https://developer.apple.com/documentation/swift/2884564-numericcast)는 우리에게 아주 친근한 내용입니다.

[SE-0202](https://github.com/apple/swift-evolution/blob/master/proposals/0202-random-unification.md)에 앞서, Swift에서 숫자를 생성하는 표준 예제는 `Darwin` 프레임워크를 가져와서 `arc4random_uniform(3)` 함수를 사용합니다.

```c
uint32_t arc4random_uniform(uint32_t __ upper_bound)
```

Swift에서 `arc4random` 를 사용하려면 두 번의 타입 변형을 필요로 합니다.
첫 번째는 `Int` 값을 `UInt32` 로 변형하고, 다음은 `UInt32` 를 `Int` 로 다시 변형하는 것입니다.

```swift
import Darwin

func random(in range: Range<Int>) -> Int {
    return Int(arc4random_uniform(UInt32(range.count))) + range.lowerBound
}
```

_끔찍하네요._

`numericCast(_:)` 를 사용하면 코드를 더 가독성 있게 만들 수 있습니다.

```swift
import Darwin

func random(in range: Range<Int>) -> Int {
    return numericCast(arc4random_uniform(numericCast(range.count))) + range.lowerBound
}
```

지니의 소원을 기억하세요. 우리는 우리의 소원을 항상 경계해야 합니다.

더 자세히 살펴보면 `numericCast(_:)` 의 예제에는 치명적인 결함이 있습니다.
바로 _`UInt32.max` 를 넘는 값이 오면 문제가 생긴다는 것이죠!_

```swift
random(in: 0..<0x1_0000_0000) // 🧞‍ Fatal error: 넘겨진 값을 표현할 비트가 충분하지 않습니다. (Not enough bits to represent the passed value)
```

[표준 라이브러리의 구현 방식](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Integers.swift#L2537-L2560)에 따르면 이제 `Int.random(in: 0...10)` 을 사용할 수 있게 되었고 여기선 range-checked conversion이 아닌 clamping conversion을 사용하는 것을 알 수 있습니다.
그리고 `arc4random_uniform` 같은 간편한 함수를 delegate하는 대신에 [무작위 바이트의 버퍼에서 값을 뽑아냅니다](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Random.swift#L156-L177).

---

코드가 컴파일되도록 하는 것은 코드를 올바르게 짜는 것과는 다른 일입니다.
하지만 때론 전자를 추구하는 것이 후자를 이루는 궁극적인 방법이 될 때도 있습니다.
현명하게 사용한다면 `numericCast(_:)` 는 이슈를 빠르게 해결할 수 있는 간편한 도구가 될 것입니다.
또한 기존 타입 initializer보다 잠재적인 오작동을 명확하게 알려주는 이점이 있습니다.

프로그래밍은 궁극적으로 우리가 원하는 것을 _정확하게_ 표현해내는 것이라고 생각합니다. 종종 골치 아픈 세부 사항이 있지만요.
CPU에는 지니같은 "올바른 일을 해라"같은 지시사항은 존재하지 않습니다.
(만일 있다고 해도 [실제로 믿을만 할까요](https://github.com/FixIssue/FixCode)?)
다행히도 Swift는 다른 많은 언어들보다 안전하고 간결한 방식으로 이러한 작업을 수행할 수 있습니다.
이보다 더 많은 것을 원하는 사람이 있을까요?
