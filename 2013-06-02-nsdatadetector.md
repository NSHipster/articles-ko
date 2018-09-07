---
title: NSDataDetector
author: Mattt
translator: 김필권
category: Cocoa
tags: nshipster
excerpt: >-
  Until humanity embraces RDF for our daily interactions,
  computers will have to work overtime
  to figure out what the heck we're all talking about.
status:
  swift: 4.2
---

문맥이 없는 문장은 아무 의미가 없습니다.

단어는 단어들 사이의 관계, 단어와 우리와의 관계 그리고 우리의 시간과 공간에 의해 무게를 갖게 됩니다.

주위에 어떤 문장이 있는지에 따라 의미가 다른 <dfn>endophoric</dfn> 또는 누가 어디서 말하는지에 따라 다른 <dfn>deictic</dfn>이라는 표현을 생각하면 되겠습니다.
그런 맥락에서 컴퓨터가 _"5분 안에 집에 도착해"_ 라는 말을 이해하는 것은 어려운 것임은 당연한 일입니다.
(그리고 날짜, 주소 및 기타 정보를 표현할 때 모호하게 표현하거나 변형하는 것은 말 할 것도 없습니다.)

좋든 나쁘든, 이것이 우리가 소통하는 방식입니다.
그리고 인간이 매일하는 상호작용에 [RDF](https://www.w3.org/RDF/)를 받아들이지 않는 이상, 컴퓨터는 우리가 뭐라고 말하는지 이해하는데 아주 많은 작업 시간을 들일 것입니다.

---

자연어를 구조화된 데이터로 바꾸는 것은 아주 큰 가치가 있습니다. 이는 우리의 달력, 주소록, 지도 그리고 리마인더에도 적용될 수 있습니다.
하지만 수동으로 입력하는 것은 사용자에게 강요하려는 선택 사항 중 제일 끝에 있어야 할 것입니다.

다른 플랫폼이었다면 이 업무를 잘 작동하는 다른 웹 서비스같은 것에 넘겼을 것입니다.
운좋게도 우리 Cocoa 개발자에겐 `NSDataDetector` 가 있습니다.

`NSDataDetector` 를 사용하면 날짜, 링크, 전화번호, 주소 그리고 대중 교통 정보를 자연어 문장에서 추출할 수 있습니다.

먼저 관심있는 결과 종류를 구체화한 detector를 만드세요.
그리고 `enumerateMatches(in:options:range:using:)` 메소드에 처리할 문장을 넘기세요.
다음 클로저는 각 결과에 한 번 실행될 것입니다.

```swift
let string = "123 Main St. / (555) 555-1234"

let types: NSTextCheckingResult.CheckingType = [.phoneNumber, .address]
let detector = try NSDataDetector(types: types.rawValue)
detector.enumerateMatches(in: string,
                          options: [],
                          range: range) { (result, _, _) in
    print(result)
}
```

```objc
NSString *string = @"123 Main St. / (555) 555-1234";

NSError *error = nil;
NSDataDetector *detector =
    [NSDataDetector dataDetectorWithTypes:NSTextCheckingTypeAddress |
                                          NSTextCheckingTypePhoneNumber
                                    error:&error];

[detector enumerateMatchesInString:string
                           options:kNilOptions
                             range:NSMakeRange(0, [string length])
                        usingBlock:
^(NSTextCheckingResult * result, NSMatchingFlags flags, BOOL * stop) {
  NSLog(@"%@", result);
}];
```

예상하셨듯이 위 코드를 실행하면 두 가지 결과가 만들어질 것입니다. 하나는 "123 Main St." 라는 주소고 다른 하나는 "(555) 555-1234" 라는 전화번호 입니다.

> `NSdataDetector`를 초기화할 때는 관심있는 딱 하나의 타입만을 지정하세요. 왜냐하면 필요없는 타입들이 과정을 느리게 만들기 때문입니다.

## 결과에서 의미있는 정보 뽑아내기

`NSDataDetector`는 `NSTextCheckingResult` 객체를 생성합니다.

한편으로는 `NSDataDetector`가 실제로 `NSRegularExpression`의 서브 클래스이기 때문에 맞는 말이고, 다른 한편으로는 패턴 매치와 탐지되는 데이터 사이의 겹치는 부분이 많지 않습니다.
그래서 여러분이 얻는 것은 어떤 상황에서 어떤 정보가 존재하는지에 대한 강력한 보장을 제공하지는 않는 오염된 API일 것입니다.

> 설상가상으로 `NSTextCheckingResult` 는 `NSSpellServer` 에서도 쓰입니다.
> _웩._

데이터 탐지기의 결과에 대한 정보를 얻고 싶다면 먼저 `resultType` 을 확인해야 합니다. `resultType` 을 이용하면 속성을 통해 정보에 직접적으로 접근할 수 있습니다. (링크, 전화번호 그리고 날짜 처럼요.) 또는 `components` 속성의 키값을 통해 간접적으로 접근할 수도 있습니다. (주소와 교통 정보 등)


다음은 `NSDataDetector` 의 결과 타입과 관련된 속성의 개요입니다.

<table>
  <thead>
    <tr>
      <th>Type</th>
      <th>Properties</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><tt>.link</tt></td>
      <td>
        <ul>
          <li><tt>.url</tt></li>
        </ul>
      </td>
    </tr>
    <tr>
      <td><tt>.phoneNumber</tt></td>
      <td>
        <ul>
          <li><tt>.phoneNumber</tt></li>
        </ul>
      </td>
    </tr>
    <tr>
      <td><tt>.date</tt></td>
      <td>
        <ul>
          <li><tt>.date</tt></li>
          <li><tt>.duration</tt></li>
          <li><tt>.timeZone</tt></li>
        </ul>
      </td>
    </tr>
    <tr>
      <td><tt>.address</tt></td>
      <td>
        <ul>
          <li><tt>.components</tt></li>
          <ul>
            <li><tt>.name</tt></li>
            <li><tt>.jobTitle</tt></li>
            <li><tt>.organization</tt></li>
            <li><tt>.street</tt></li>
            <li><tt>.city</tt></li>
            <li><tt>.state</tt></li>
            <li><tt>.zip</tt></li>
            <li><tt>.country</tt></li>
            <li><tt>.phone</tt></li>
          </ul>
        </ul>
      </td>
    </tr>
    <tr>
      <td><tt>.transitInformation</tt></td>
      <td>
        <ul>
          <li><tt>.components</tt></li>
          <ul>
            <li><tt>.airline</tt></li>
            <li><tt>.flight</tt></li>
          </ul>
        </ul>
      </td>
    </tr>
  </tbody>
</table>


## Data Detector Data Points


Let's put `NSDataDetector` through its paces.
That way, we'll not only have a complete example of how to use it to its full capacity but see what it's actually capable of.


The following text contains one of each of the type of data that `NSDataDetector` should be able to detect:

```swift
let string = """
   My flight (AA10) is scheduled for tomorrow night from 9 PM PST to 5 AM EST.
   I'll be staying at The Plaza Hotel, 768 5th Ave, New York, NY 10019.
   You can reach me at 555-555-1234 or me@example.com
"""
```


We can have `NSDataDetector` check for everything by passing `NSTextCheckingAllTypes` to its initializer.
The rest is a matter of switching over each `resultType` and extracting their respective details:

```swift
let detector = try NSDataDetector(types: NSTextCheckingAllTypes)
let range = NSRange(string.startIndex..<string.endIndex, in: string)
detector.enumerateMatches(in: string,
                          options: [],
                          range: range) { (match, flags, _) in
    guard let match = match else {
        return
    }

    switch match.resultType {
    case .date:
        let date = match.date
        let timeZone = match.timeZone
        let duration = match.duration
        print(date, timeZone, duration)
    case .address:
        if let components = match.components {
            let name = components[.name]
            let jobTitle = components[.jobTitle]
            let organization = components[.organization]
            let street = components[.street]
            let locality = components[.city]
            let region = components[.state]
            let postalCode = components[.zip]
            let country = components[.country]
            let phoneNumber = components[.phone]
            print(name, jobTitle, organization, street, locality, region, postalCode, country, phoneNumber)
        }
    case .link:
        let url = match.url
        print(url)
    case .phoneNumber:
        let phoneNumber = match.phoneNumber
        print(phoneNumber)
    case .transitInformation:
        if let components = match.components {
            let airline = components[.airline]
            let flight = components[.flight]
            print(airline, flight)
        }
    default:
        return
    }
}
```


When we run this code, we see that `NSDataDetector` is able to identify each of the types.


| Type                | Output                                                                     |
| ------------------- | -------------------------------------------------------------------------- |
| Date                | "2018-08-31 04:00:00 +0000", "America/Los_Angeles", 18000.0                |
| Address             | `nil`, `nil`, `nil` "768 5th Ave", "New York", "NY", "10019", `nil`, `nil` |
| Link                | "mailto:me@example.com"                                                    |
| Phone Number        | "555-555-1234"                                                             |
| Transit Information | `nil`, "10"                                                                |


Impressively, the date result correctly calculates the 6-hour duration of the flight, accommodating for the time zone change.
However, some information is missing, like the name of The Plaza Hotel in the address, and the airline in the transit information.


> Even after trying a handful of different representations ("American Airlines 10", "AA 10", "AA #10", "American Airlines (AA) #10") and airlines ("Delta 1226", "DL 1226")
> I still wasn't able to find an example that populated the `airline` property.
> If anyone knows what's up, [@ us](https://twitter.com/NSHipster/).


## Detect (Rough) Edges


Useful as `NSDataDetector` is, it's not a particularly _nice_ API to use.


With all of the charms of its parent class, [`NSRegularExpression`](https://nshipster.com/nsregularexpression/), the same, cumbersome initialization pattern of [NSLinguisticTagger](https://nshipster.com/nltagger/), and an [incomplete Swift interface](https://developer.apple.com/documentation/foundation/nstextcheckingtypes), `NSDataDetector` has an interface that only a mother could love.


But that's only the API itself.


In a broader context, you might be surprised to learn that a nearly identical API can be found in the `dataDetectorTypes` properties of `UITextView` and `WKWebView`.
_Nearly_ identical.


`UIDataDetectorTypes` and `WKDataDetectorTypes` are distinct from and incompatible with `NSTextCheckingTypes`, which is inconvenient but not super conspicuous.
But what's utterly inexplicable is that these APIs can detect [shipment tracking numbers](https://developer.apple.com/documentation/uikit/uidatadetectortypes/1648142-shipmenttrackingnumber) and [lookup suggestions](https://developer.apple.com/documentation/uikit/uidatadetectortypes/1648141-lookupsuggestion), neither of which are supported by `NSDataDetector`.
It's hard to imagine why shipment tracking numbers wouldn't be supported, which leads one to believe that it's an oversight.

---


Humans have an innate ability to derive meaning from language.
We can stitch together linguistic, situational and cultural information into a coherent interpretation at a subconscious level.
Ironically, it's difficult to put this process into words --- or code as the case may be.
Computers aren't hard-wired for understanding like we are.


Despite its shortcomings, `NSDataDetector` can prove invaluable for certain use cases.
Until something better comes along, take advantage of it in your app to unlock the structured information hiding in plain sight.
