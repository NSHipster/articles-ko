---
title: TimeInterval, Date, and DateInterval
author: Mattt
translator: 김필권
category: Cocoa
excerpt: >
  Our limited understanding of time
  is reflected in ---
  or perhaps exacerbated by ---
  the naming of the Foundation date and time APIs.
  It's about time we got them straight.
status:
  swift: 4.2
---

마드리드의 Centro 지구와 Salamanca 지구의 사이에 있고 Buen Retiro Park에서 걸어갈 수 있는 거리에 위치한 Prado 미술관은 유럽에서 가장 유명한 화가들의 작품을 다양하게 보유하고 있습니다.
만약 여러분이 17세기 스페인 군주들의 초상화에 싫증이 났을 경우엔 1층 가장 북쪽에 있는 _Sala 002_ 방을 방문하는 것을 추천드립니다.
그곳엔 [프랑스 화가인 Simon Vouet이 바로크 시대동안 그렸던 그림들을 보실 수 있습니다](https://www.museodelprado.es/en/the-collection/art-work/time-defeated-by-hope-and-beauty/ebaeb191-f3ff-43b1-9207-fb36a3e5ad5a).

이 그림을 보면 두 젊은 여성들이 갈고리와 창을 들고 늙은 남자를 위협하는 이유와 천사들이 뒤에서 웃으며 구경하고 있는 이유에 대해 궁금하실 것입니다.
그것은 당연히 우화적인 이유입니다. 근처에 있는 설명글을 읽고나면 여러분은 이 작품이 _시간이 희망과 아름다움에 패배한다_ 는 것을 느끼셨을 것입니다.
늙은 남자는 왜 시간을 의미할까요?
손에 있는 모래시계와 발쪽에 있는 낫을 보면 이해하실 것입니다.

잠시동안 작품앞에 서서 시간의 수수께끼같은 속성에 대해 생각해보세요.

우리가 시간을 얼마나 제한적이게 생각하고 있는지는 Foundation의 날짜와 시간 API의 이름에서 드러납니다. (혹은 더 악화됐을수도 있겠습니다)

이제 이를 바로잡을 시간입니다.

---

초(Seconds)는 시간의 근본적인 단위입니다.
그리고 그들은 고정된 기간의 단위를 표현합니다.

달(Months)은 다양한 기간을 가지고 있으며(_9월은 30일이 있죠_), 년(Years)도 그렇습니다(_400년마다 71개의 년이 53주를 가집니다_). 특정 년에는 추가적인 날이 존재하고 여름에는 시간 절약을 위해 1시간을 더 얻거나 덜 얻습니다. (_고마워요 벤자민 프랭클린_)
그리고 어떤 기이한 날에는 1분에 61초, 1시간에 3601초, 하루에 1209601초가 들기도 합니다.

`TimeInterval` (옛날 이름은 `NSTimeInterval`) 은 기간을 초로 나타낸 값을 `Double` 타입으로 가지고 있는 typealias입니다.
시간의 기간을 다루는 API의 반환 타입이나 파라미터인 TimeInterval을 본 적 있을 것입니다.
두배로 정밀한 부동 소수점 수인 `TimeInterval` 은 분수로도 나타낼 수 있습니다.
(밀리세컨드를 넘어선 정밀도를 필요로 한다면요)

## 날짜와 시간 (Date and Time)

시간을 표현하는 Foundation 타입의 이름이 `Date` 라는 것은 불행한 일입니다.
구어체로 말할 때를 생각해보면 "날짜"는 "시간"과 구별해서 말하는 편이며 전자는 달력의 어떤 날을 의미하고 후자는 하루의 어떤 시간을 의미합니다.
하지만 `Date` 는 완전히 달력과는 다르며 그 이름과는 반대로 시간의 절대적인 지점을 표현합니다.

{% info %}

왜 `NSTime` 이 아니라 `NSDate` 일까요?
저희의 추측은 EOF(Enterprise Objects Framework)가 Java와 Objective-C 모두를 목표로 삼았을 때 이 API를 설계한 사람이 [`java.util.date`의 대항마](https://docs.oracle.com/javase/7/docs/api/java/util/Date.html)로 만들려고 했다는 것입니다.

{% endinfo %}

`Date` 에 대한 혼란의 또 다른 이유는 시간의 절대적인 지점을 나타내는 것에도 불구하고 [참조 날짜 이후의 시간 간격으로 정의된다는 것](https://github.com/apple/swift-corelibs-foundation/blob/master/Foundation/Date.swift#L17-L20)입니다.

```swift
public struct Date : ReferenceConvertible, Comparable, Equatable {
    public typealias ReferenceType = NSDate

    fileprivate var _time: TimeInterval

    // ...
}
```

이 경우에는 참조 날짜는 2001년 1월 1일 GMT(Greenwich Mean Time)의 첫번째 인스턴트입니다.

{% info %}

아직도 이유를 모르겠는 사실이 있습니다.
Apple은 왜 Unix Epoch(1970년 1월 1일)를 사용하지 않았을까요?
2001년은 Mac OS X의 첫 번째 릴리즈가 있었던 해였지만 `NSDate` 는 NeXT에도 존재했었습니다.
[Y2K](https://en.wikipedia.org/wiki/Year_2000_problem) 때문이었을까요?

{% endinfo %}

## Date Intervals와 Time Intervals

`DateInterval` 은 비교적 최근 추가된 내용입니다.
iOS 10과 macOS Sierra에 소개됐으며 이 타입은 두 시간의 절대적인 지점의 닫힌 간격을 나타냅니다. (기간을 초로 나타내는 `TimeInterval` 과는 대조적입니다.)

그래서 이건 어디에 쓰일까요?
다음과 같이 사용할 수 있을 것입니다.

### Calendar Unit의 Date Interval 구하기

시간의 특정 지점을 어떤 날의 시간으로 표현하기 위해선 calendar를 참고해야 합니다.
여기서 여러분은 특정 calendar unit의 범위를 일(day), 월(month) 또는 년(year)으로 구별할 수 있습니다.
`Calendar` 의 메소드인 `dateInterval(of:for:)` 는 이를 정말 쉽게 만들어줍니다.

```swift
let calendar = Calendar.current
let date = Date()
let dateInterval = calendar.dateInterval(of: .month, for: date)
```

우리의 `Calendar` 는 우리를 배신하는 경우가 없으니 걱정마십시오.
다음은 분위기를 깨지 않고 해가 떠있는 시간을 계산하는 예제입니다.

```swift
let dstComponents = DateComponents(year: 2018,
                                   month: 11,
                                   day: 4)
calendar.dateInterval(of: .day,
                      for: calendar.date(from: dstComponents)!)?.duration
// 90000 초
```

_지금은 {{ site.time | date: '%Y' }}년 입니다.
`secondsInDay = 86400`와 같이 하드코딩할 시기는 지났다고 생각합니다!_

## Date Intervals의 교차 지점 계산하기

이 예제에선 다시 Prado 미술관으로 돌아가서 그들의 광대한 Rubens의 그림 컬렉션을 존경해봅시다. 특히 여기 [Swift 프로그래밍의 신을 명백하게 묘사한 그림](https://www.museodelprado.es/coleccion/obra-de-arte/eolo/e447dadb-b93f-4ce5-84e9-e6ae1d95c6cd)은 꼭 보세요.

Vouet처럼 Rubens도 Baroque 전통의 그림을 그렸습니다.
이 둘은 동시대의 인물들이었고 우리는 `DateInterval` 의 도움을 받아서 그 둘이 역사에서 얼마나 겹치는지 알아보도록 하겠습니다.

```swift
import Foundation

let calendar = Calendar.current

// Simon Vouet
// 9 January 1590 – 30 June 1649
let vouet =
    DateInterval(start: calendar.date(from:
        DateComponents(year: 1590, month: 1, day: 9))!,
                 end: calendar.date(from:
                    DateComponents(year: 1649, month: 6, day: 30))!)

// Peter Paul Rubens
// 28 June 1577 – 30 May 1640
let rubens =
    DateInterval(start: calendar.date(from:
                            DateComponents(year: 1577, month: 6, day: 28))!,
                 end: calendar.date(from:
                            DateComponents(year: 1640, month: 5, day: 30))!)

let overlap = rubens.intersection(with: vouet)!

calendar.dateComponents([.year],
                        from: overlap.start,
                        to: overlap.end) // 50 years
```

우리의 계산에 의하면 이 둘은 50년동안 같은 시대를 공유했습니다.

심지어 `DateIntervalFormatter` 를 사용하면 그 시간대를 더 이쁘게 표현할 수도 있습니다.

```swift
let formatter = DateIntervalFormatter()
formatter.timeStyle = .none
formatter.dateTemplate = "%Y"
formatter.string(from: overlap)
// "1590 – 1640"
```

_아름답네요._
이 코드는 출력해서 액자에 담아서 [_The Judgement of Paris_](https://www.museodelprado.es/en/the-collection/art-work/the-judgement-of-paris/f8b061e1-8248-42ae-81f8-6acb5b1d5a0a) 작품 옆에 걸어놔도 될 정도로 아름답습니다.

---

사실 우리는 아직 시간이 _무엇인지_ (또는 그것이 실제로 존재하는지조차) 알지 못합니다.
하지만 저는 우리 개발자들이 Foundation의 `Date` API에서 아름다움을 발견하고 언젠가는 이해가 부족한 부분들을 배워서 극복하는 것을 희망합니다.

여기까지 이 주의 글이었습니다.
다음 시간에 뵙겠습니다.
