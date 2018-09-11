---
title: NSDataDetector
author: Mattt
translator: 김필권
category: Cocoa
tags: nshipster
excerpt: "인간이 매일하는 상호작용에 RDF를 받아들이지 않는 이상 컴퓨터는 우리가 말하는 것을 이해하는데 아주 많은 작업 시간을 들일 것입니다."
status:
  swift: 4.2
---

문맥이 없는 문장은 아무 것도 아닙니다.

단어는 단어들 사이의 관계, 단어와 우리와의 관계 그리고 우리의 시간과 공간에 의해 무게를 갖게 됩니다.

주위에 어떤 문장이 있는지에 따라 의미가 다른 <dfn>endophoric</dfn> 또는 누가 어디서 말하는지에 따라 다른 <dfn>deictic</dfn>이라는 표현을 생각하면 되겠습니다.
그런 맥락에서 컴퓨터가 _"5분 안에 집에 도착해"_ 라는 말을 이해하는 것은 어려운 것임은 당연한 일입니다.
(그리고 날짜, 주소 및 기타 정보를 표현할 때 모호하게 표현하거나 변형하는 것은 말 할 것도 없습니다.)

좋든 나쁘든 이것이 우리가 소통하는 방식입니다.
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

이제 `NSDataDetector`를 그것의 페이스에 맞춰보겠습니다.
그렇게하면 우리는 `NSDataDetector` 를 사용해서 전체 용량을 사용하는 방법에 대한 완벽한 예제를 얻게 될 뿐만 아니라 실제로 가능한 일들을 볼 수도 있습니다.

다음의 텍스트는 `NSDataDetector` 가 탐지할 수 있는 데이터의 종류를 모두 담고 있습니다.

```swift
let string = """
   My flight (AA10) is scheduled for tomorrow night from 9 PM PST to 5 AM EST.
   I'll be staying at The Plaza Hotel, 768 5th Ave, New York, NY 10019.
   You can reach me at 555-555-1234 or me@example.com
"""
```

`NSTextCheckingAllTypes` 를 초기화할 때 넘기면 `NSDataDetector` 가 모든 것을 확인하게 할 수 도 있습니다.
그 후엔 각 `resultType` 을 바꾸고 세부 사항을 추출하는 문제만 남습니다.

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

위의 코드를 실행하면 `NSDataDetector` 가 각각의 타입을 식별하는 것을 볼 수 있습니다.

| Type                | Output                                                                     |
| ------------------- | -------------------------------------------------------------------------- |
| Date                | "2018-08-31 04:00:00 +0000", "America/Los_Angeles", 18000.0                |
| Address             | `nil`, `nil`, `nil` "768 5th Ave", "New York", "NY", "10019", `nil`, `nil` |
| Link                | "mailto:me@example.com"                                                    |
| Phone Number        | "555-555-1234"                                                             |
| Transit Information | `nil`, "10"                                                                |

데이터 결과가 정확히 6시간동안의 비행을 계산하는 것은 인상적이었습니다. 시간대 변경까지 적용해서요! 하지만 주소의 The Plaza Hotel의 이름이나 교통 정보의 항공사같은 내용같은 정보는 제대로 인식하지 못했습니다.

> "American Airlines 10", "AA 10", "AA #10", "American Airlines (AA) #10"와 "Delta 1226", "DL 1226" 처럼 몇 가지 다른 표현을 시도했습니다.
> 하지만 여전히 `airline` 속성을 채우는 예는 찾을 수가 없었습니다.
> 아는 분이 있다면 [저희에게 알려주세요](https://twitter.com/NSHipster/).

## Detect (Rough) Edges

`NSDataDetector` 가 유용하긴 하지만 사용하기에 특별히 _좋은_ API는 아닙니다.

부모 클래스인 [`NSRegularExpression`](https://nshipster.com/nsregularexpression/)의 매력인 [NSLinguisticTagger](https://nshipster.com/nltagger/)의 성가신 초기화 패턴과 [불완전한 Swift 인터페이스](https://developer.apple.com/documentation/foundation/nstextcheckingtypes) 덕분에 `NSDataDetector` 는 부모만이 좋아할 인터페이스를 가지도록 만들었습니다.

하지만 그것은 API 자체일 뿐입니다.

더 넓은 맥락에서 `UITextView` 와 `WKWebView` 의 `dataDetectorTypes` 속성에서 비슷한 API가 발견된다는 것을 알면 깜짝 놀라실 것입니다.
_거의_ 똑같습니다.

`UIDataDetectorTypes` 와 `WKDataDetectorTypes` 는 `NSTextCheckingTypes` 와 다르며 서로 호환되지도 않습니다. 조금 불편하지만 그렇게 티나지는 않습니다.
하지만 완전히 설명할 수 없는 것은 이러한 API가 `NSDataDetector` 는 지원하지 않는 [배송 추적 번호](https://developer.apple.com/documentation/uikit/uidatadetectortypes/1648142-shipmenttrackingnumber)와 [lookup 제안](https://developer.apple.com/documentation/uikit/uidatadetectortypes/1648141-lookupsuggestion)을 탐지한다는 점입니다.
배송 추적 번호가 왜 지원되지 않는지 상상하긴 어려워서 못보고 넘긴 것으로 보입니다.

---

인간은 언어에서 의미를 추출해내는 타고난 능력을 가지고 있습니다.
우리는 잠재적인 단계에서 언어적, 상황적 그리고 문화적 정보를 일관된 해석으로 묶습니다.
역설적으로 이러한 과정을 단어로 만드는 것은 어려운 일입니다. 코드도 물론이구요.
우리가 이해하는 식으로 컴퓨터가 하기란 쉽지않은 일입니다.

앞서 설명한 단점에도 불구하고 `NSDataDetector` 는 특정 경우에 매우 귀중한 역할을 한다는 것을 증명했습니다.
더 나은 무언가가 나오기 전까지는 NSDataDetector를 사용해서 단조로운 내용안에 숨어있는 구조화된 정보를 찾아내는 것이 좋을 것입니다.
