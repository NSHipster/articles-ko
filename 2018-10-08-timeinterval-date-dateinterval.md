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

마드리드의 Centro 지구와 Salamanca 지구의 사이에, Buen Retiro Park에서 걸어갈 수 있는 거리에 위치한 프라도 박물관은 유럽에서 가장 유명한 화가들의 작품을 다양하게 보유하고 있습니다.
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


It's unfortunate that the Foundation type representing time is named `Date`.
Colloquially, one typically distinguishes "dates" from "times" by saying that the former has to do with calendar days and the latter has more to do with the time of day.
But `Date` is entirely orthogonal from calendars, and contrary to its name represents an absolute point in time.

{% info %}


Why `NSDate` and not `NSTime`?
Our guess is that the originators of this API wanted to match its [counterpart in `java.util.date`](https://docs.oracle.com/javase/7/docs/api/java/util/Date.html) when <abbr title="Enterprise Objects Framework">EOF</abbr> targeted both Java and Objective-C.

{% endinfo %}


Another source of confusion for `Date` is that, despite representing an absolute point in time, it's [defined by a time interval since a reference date](https://github.com/apple/swift-corelibs-foundation/blob/master/Foundation/Date.swift#L17-L20):

```swift
public struct Date : ReferenceConvertible, Comparable, Equatable {
    public typealias ReferenceType = NSDate

    fileprivate var _time: TimeInterval

    // ...
}
```


The reference date, in this case, is the first instant of January 1, 2001, Greenwich Mean Time (GMT).

{% info %}


While we're on the subject of conjectural sidebars, does anyone know why Apple created a new standard instead of using, say, the Unix Epoch (January 1, 1970)? 2001 was the year that Mac OS X was first released, but `NSDate` pre-NSDates that from its NeXT days.
Was it perhaps a hedge against [Y2K](https://en.wikipedia.org/wiki/Year_2000_problem)?

{% endinfo %}


## Date Intervals and Time Intervals


`DateInterval` is a recent addition to Foundation.
Introduced in iOS 10 and macOS Sierra, this type represents a closed interval between two absolute points in time (again, in contrast to `TimeInterval`, which represents a duration in seconds).


So what is this good for?
Consider the following use cases:


### Getting the Date Interval of a Calendar Unit


In order to know the time of day for a point in time --- or what day it is in the first place --- you need to consult a calendar.
From there, you can determine the range of a particular calendar unit, like a day, month, or year.
The `Calendar` method `dateInterval(of:for:)` makes this really easy to do:

```swift
let calendar = Calendar.current
let date = Date()
let dateInterval = calendar.dateInterval(of: .month, for: date)
```


Because we're invoking `Calendar`, we can be confident in the result that we get back.
Look how it handles daylight saving transition without breaking a sweat:

```swift
let dstComponents = DateComponents(year: 2018,
                                   month: 11,
                                   day: 4)
calendar.dateInterval(of: .day,
                      for: calendar.date(from: dstComponents)!)?.duration
// 90000 seconds
```


_It's {{ site.time | date: '%Y' }}.
Don't you think that it's time you stopped hard-coding `secondsInDay = 86400`?_


## Calculating Intersections of Date Intervals


For this example, let's return to The Prado Museum and admire its extensive collection of paintings by Rubens --- particularly [this apparent depiction of the god of Swift programming](https://www.museodelprado.es/coleccion/obra-de-arte/eolo/e447dadb-b93f-4ce5-84e9-e6ae1d95c6cd).


Rubens, like Vouet, painted in the Baroque tradition.
The two were contemporaries, and we can determine the full extent of how they overlap in the history of art with the help of `DateInterval`:

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


According to our calculations, there was a period of 50 years where both painters were living.


We can even take things a step further and use `DateIntervalFormatter` to provide a nice representation of that time period:

```swift
let formatter = DateIntervalFormatter()
formatter.timeStyle = .none
formatter.dateTemplate = "%Y"
formatter.string(from: overlap)
// "1590 – 1640"
```


_Beautiful._
You might as well print this code out, frame it, and hang it next to [_The Judgement of Paris_](https://www.museodelprado.es/en/the-collection/art-work/the-judgement-of-paris/f8b061e1-8248-42ae-81f8-6acb5b1d5a0a).

---


The fact is, we still don't really know _what_ time is (or if it even actually exists).
But I'm hopeful that we, as developers, will find the beauty in Foundation's `Date` APIs, and in time, learn how to overcome our lack of understanding.


That does it for this week's article.
See you all next time.
