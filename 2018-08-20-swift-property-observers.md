---
title: Swift Property Observers
author: Mattt
translator: 김필권
category: Swift
excerpt: "모던 소프트웨어 개발은 골드버그 기계의 정수라고 할 수 있을 정도로 복잡해졌습니다. 그러나 부작용을 생산하는 코드에 대한 의혹에도 불구하고 때로는 기술이 혼란스러운 것보다 명확해질 수 있는 기회가 존재합니다."
status:
  swift: 4.2
---

1930년대에 Rube Goldberg라는 이름은 누구나 알만한 이름이 되었고 ["자가 작동식 냅킨"](https://upload.wikimedia.org/wikipedia/commons/a/a9/Rube_Goldberg%27s_%22Self-Operating_Napkin%22_%28cropped%29.gif)처럼 환상적이게 복잡하며 만화에 그려질 것 같이 기발한 발명품을 뜻하는 말이었습니다. 같은 시간에 Albert Einstein은 Niels Bohr의 양자 기계의 우세한 이론에 대한 그의 [평론](https://en.wikipedia.org/wiki/EPR_paradox)에서 "양자 얽힘"이라는 단어를 유행하고 있었습니다.

몇 세기 후에 모던 소프트웨어 개발은 Goldberg 기계의 정수라고 할 수 있을 정도로 복잡해졌습니다.

소프트웨어 개발자로서 가능하면 코드를 얽히지 않게 하는 것이 좋습니다. 이는 [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle), [Principle of Least Astonishment](https://en.wikipedia.org/wiki/Principle_of_least_astonishment), 그리고 [Law of Demeter](https://en.wikipedia.org/wiki/Law_of_Demeter)와 같이 인상적인 가이드라인으로 성문화되어 있습니다. 그러나 부작용을 생산하는 코드에 대한 의혹에도 불구하고 때로는 기술이 혼란스러운 것보다 명확해질 수 있는 기회가 존재합니다.

그 중 하나가 이 주의 아티클의 주제인 Swift의 프로퍼티 옵저버입니다. 프로퍼티 옵저버는 모델-뷰-뷰모델(MVVM), 함수형 반응형 프로그래밍 (FRP) 같은 공식화된 해결책의 가볍고 내장된 대체재를 제공합니다.

---

Swift의 프로퍼티에는 두 가지 종류가 있습니다. 하나는 상태를 객체와 연결하는 <dfn>저장된 프로퍼티 (stored properties)</dfn>이고, 또 다른 하나는 그 상태에 기반해서 계산을 하는 <dfn>계산된 프로퍼티 (computed properties)</dfn> 입니다. 예를 들어 보겠습니다.

```swift
struct S {
    // 저장된 프로퍼티
    var stored: String = "stored"

    // 계산된 프로퍼티
    var computed: String {
        return "computed"
    }
}
```

저장된 프로퍼티를 선언했을 때는 <dfn>프로퍼티 옵저버</dfn>를 프로퍼티가 설정됐을 때 실행될 코드 블락과 함께 정의할 수 있는 선택지를 제공받습니다. `willSet` 옵저버는 새로운 값이 저장되기 전에 실행되고 `didSet` 옵저버는 이후에 실행됩니다. 그리고 그것들은 예전 값이 새로운 값과 같은지 여부에 상관없이 실행됩니다.

```swift
struct S {
    var stored: String {
        willSet {
            print("willSet이 호출되었습니다")
            print("stored가 지금은 \(self.stored)와 같습니다")
            print("stored가 곧 \(newValue)로 값이 변경될 것입니다")
        }

        didSet {
            print("didSet이 호출되었습니다")
            print("stored가 지금은 \(self.stored)와 같습니다")
            print("stored의 이전 값은 \(oldValue) 였습니다")
        }
    }
}
```

예를 들어 다음과 같은 코드를 실행하면 그 아래와 같은 텍스트가 출력될 것입니다.

```swift
var s = S(stored: "first")
s.stored = "second"
```

- <samp>willSet이 호출되었습니다</samp>
- <samp>stored가 지금은 first와 같습니다</samp>
- <samp>stored가 곧 second로 값이 변경될 것입니다</samp>
- <samp>didSet이 호출되었습니다</samp>
- <samp>stored가 지금은 second와 같습니다</samp>
- <samp>stored의 이전 값은 first 였습니다</samp>

> 중요한 사실은 옵저버는 이니셜라이져에서 프로퍼티를 설정할 떄는 실행되지 않는다는 것입니다.
> Swift 4.2에선 setter 호출을 `defer` 블록에서 작업할 수 있지만 [이것은 버그라서 곧 수정될 것입니다](https://twitter.com/jckarter/status/926459181661536256). 그러니 이러한 변화에는 따르지 않으셔도 됩니다.

---

Swift 프로퍼티 옵저버는 언어의 아주 초기부터 일부분을 차지하고 있었습니다. 그 이유를 잘 이해하기 위해 Objective-C에선 어떻게 작동하는지 빠르게 둘러보겠습니다.

## Objective-C에서의 프로퍼티

Objective-C에선 모든 프로퍼티는 어떤 의미에서는 계산된 프로퍼티입니다. 마침표 노테이션을 통해 프로퍼티에 접근할 때마다 그 호출은 getter 또는 setter를 발동하도록 번역됩니다. 이는 인스턴스 변수를 읽거나 작성하는 함수를 실행하는 메세지로 컴파일됩니다.

```objc
// 마침표 접근법
person.name = @"Johnny";

// ...은 다음과 같습니다
[person setName:@"Johnny"];

// ...는 이렇게 컴파일됩니다
objc_msgSend(person, @selector(setName:), @"Johnny");

// ...는 다음과 같이 구현됩니다
person->_name = @"Johnny";
```

부작용은 여러분이 보통 프로그래밍에서 피하고 싶은 무언가입니다. 왜냐하면 프로그램이 작동하는 방식을 추측하기 어렵게 만들기 때문입니다. 하지만 많은 Objective-C 개발자들이 필요에 따라 getter 또는 setter 메소드에 추가 행동을 주입하는 기능에 의존하게 되었습니다.

프로퍼티에 대한 Swift의 디자인은 이러한 패턴을 공식화하고 상태 접근을 데코레이트하는 부작용(저장된 프로퍼티)과 상태 접근을 리다이렉트하는 부작용 사이의 차이(계산된 프로퍼티)를 만듭니다. 저장된 프로퍼티의 경우 `willSet` 과 `didSet` 옵저버는 ivar 접근과 함께 포함시키지 않을 코드를 대체합니다. 계산된 프로퍼티의 경우 `get` 과 `set` 접근자는 Objective-C에서 `@dynamic` 프로퍼티로 구현할 코드를 대체합니다.

결과적으로 우리는 더 일관성있는 의미와 프로퍼티와 상호작용하는 KVO (Key-Value Observing) 및 KVC (Key-Value Coding)같은 매커니즘에 대한 더 나은 보증을 얻게됩니다.

---

그래서 Swift에서 프로퍼티 옵저버로 할 수 있는 일은 무엇일까요?
고려할만한 몇 가지 아이디어를 준비했습니다.

---

## 값을 정규화하거나 검증할 때

때로는 타입에 허용되는 값의 추가적인 제한 조건을 넣고 싶을 때가 있습니다.

예를 들면 만약 여러분이 정부 관료를 대하는 앱을 개발중이라면 여러분은 사용자가 문서에 필수 요소를 빼먹거나 유효하지 않은 값을 제출할 수 없도록 보증해야 할 수도 있습니다.

예를 들어 문서 폼에 이름이 악센트 없이 대문자만 필요로 한다면 `didSet` 프로퍼티 옵저버를 사용해서 자동으로 발음 구별 부호를 벗겨내고 새로운 값을 대문자로 만들 수 있습니다.

```swift
var name: String? {
    didSet {
        self.name = self.name?
                        .applyingTransform(.stripDiacritics,
                                            reverse: false)?
                        .uppercased()
    }
}
```

옵저버의 바디에서 프로퍼티를 설정하는 것은 (다행히도) 추가적인 콜백을 호출하지 않습니다. 그러니 여기서 무한 루프는 만들어지지 않습니다. 이는 이것이 `willSet` 옵저버로 작동하지 않는 이유와 동일합니다. 프로퍼티가 `newValue` 로 설정되면 콜백에 설정된 값은 즉시 덮어씌워집니다.

이 접근 방식은 일회성 문제에는 작동할 수 있어도 이와 같이 반복적으로 사용하는 것이 타입에서 공식화될 수 있는 비즈니스 로직의 강력한 지표입니다.

더 나은 디자인은 폼에 입력될 텍스트들의 필수 사항들이 압축돼있는 `NormalizedText` 를 만드는 것입니다.

```swift
struct NormalizedText {
    enum Error: Swift.Error {
        case empty
        case excessiveLength
        case unsupportedCharacters
    }

    static let maximumLength = 32

    var value: String

    init(_ string: String) throws {
        if string.isEmpty {
            throw Error.empty
        }

        guard let value = string.applyingTransform(.stripDiacritics,
                                                   reverse: false)?
                                .uppercased(),
              value.canBeConverted(to: .ascii)
        else {
             throw Error.unsupportedCharacters
        }

        guard value.count < NormalizedText.maximumLength else {
            throw Error.excessiveLength
        }

        self.value = value
    }
}
```

failable 이나 이니셜라이져를 던지는 것은 `didSet` 옵저버가 할 수 없는 방식으로 호출자에게 에러를 타나낼 수 있습니다. 이제 _[Llanfair­pwllgwyngyll­gogery­chwyrn­drobwll­llan­tysilio­gogo­goch](https://en.wikipedia.org/wiki/Llanfairpwllgwyngyll)_ 의 _Jøhnny_ 같은 트러블메이커가 와도 그를 위해 무언가를 해줄 수 있습니다! (말하자면, 허용가능한 범위의 예절로 에러를 얘기하는 것이 유효하지 않은 데이터를 제공하는 것보다는 낫다는 말입니다)

## 독립 상태 전파하기

프로퍼티 옵저버의 또 다른 잠재적 사용 방안은 뷰 컨트롤러의 독립 컴포넌트에 상태를 전파하는 것입니다.

`TrackViewController` 의 `Track` 모델로 예를 들어 보겠습니다.

```swift
struct Track {
    var title: String
    var audioURL: URL
}

class TrackViewController: UIViewController {
    var player: AVPlayer?

    var track: Track? {
        willSet {
            self.player?.pause()
        }

        didSet {
            guard let track = self.track else {
                return
            }

            self.title = track.title

            let item = AVPlayerItem(url: track.audioURL)
            self.player = AVPlayer(playerItem: item)
            self.player?.play()
        }
    }
}
```

뷰 컨트롤러의 `track` 프로퍼티가 설정되면 다음과 같은 상황이 자동으로 일어납니다.

1. 이전의 트랙(track)의 오디오는 일시정지됩니다.
2. 뷰 컨트롤러의 `title`은 새로운 트랙의 제목으로 설정됩니다.
3. 새로운 트랙의 오디오가 불러와지고 재생됩니다.

_끝내주지 않나요?_

[_Mousehunt_ 의 한 장면](https://www.youtube.com/watch?v=TVAhhVrpkwM)처럼 여러 관찰된 프로퍼티에 걸쳐서 이 동작을 연결시킬 수 있습니다.

---

일반적으로 부작용은 프로그래밍할 때 피하고 싶은 것들입니다. 왜냐하면 그들은 복잡항 행동에 대해 추측하기 어렵게 만들기 때문입니다. 다음 번에 이 새로운 도구를 사용할 때 꼭 기억하시길 바랍니다.

그럼에도 불구하고 이 추상화의 탑 꼭대기에서는 시스템의 혼란을 받아들이는 것이 유혹이 될 수도 있고 때로는 보람을 느낄 수도 있을 것입니다. 항상 규칙을 따르는 것은 아인슈타인이 아닌 보어스러운 것입니다.
