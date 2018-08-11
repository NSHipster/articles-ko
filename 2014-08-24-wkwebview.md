---
title: WKWebView
author: Mattt
translator: 김필권
category: Cocoa
excerpt: iOS와 웹은 꽤 복잡한 관계를 맺고 있습니다.
  이는 10년 전 플랫폼이 처음 생겼을 때부터 시작되었습니다.
status:
  swift: 4.2
  reviewed: July 25, 2018
---

iOS와 웹은 꽤 복잡한 관계를 맺고 있습니다.
이는 10년 전 플랫폼이 처음 생겼을 때부터 시작되었습니다.

지금 와서 아이폰 초기 디자인을 보면 정해진 순서대로 디자인했다고 생각할 수 있지만, 우리의 사랑스러운 터치스크린 기기는 한때 여러 선택지 중 하나에 불과했습니다. 물리 키보드가 달린 초기 프로토타입도 있었으며 심지어 아이팟의 클릭 휠조차도 그때는 엄청난 도전이었습니다.

그래도 저는 하드웨어가 아닌 소프트웨어를 더 발전시키기로 한 것이 초기에 했던 결정 중 가장 의미 있는 결정이었다고 생각합니다.

아이폰에서 어떻게 macOS처럼 앱을 실행할 수 있었을까요? 혹은 어떻게 사파리로 웹페이지를 띄울 수 있었을까요?
iPhoneOS를 만들기로 한 것은 오늘날까지도 영향을 끼치는 광범위한 결정이었습니다.

WWDC 2007 키노트에서 스티브 잡스가 한 유명한 말들입니다.

> 사파리 엔진은 전부 아이폰 내부에 있습니다.
> 그러니 여러분은 Web 2.0과 Ajax 앱을 작성할 수 있습니다.
> 그것들은 아이폰의 앱과 같게 작동하고 보기에도 똑같습니다.
> 이 앱들은 아이폰 서비스에도 완벽하게 통합됩니다.
> 그들은 전화를 걸 수 있고 이메일도 보낼 수 있습니다.
> 게다가 구글 맵에서 나의 위치를 알아낼 수 있습니다.

요즘 아이폰이 모바일 웹에 큰 공을 세운 것과는 모순적으로 웹은 iOS에서 언제나 2급 시민이었습니다. `UIWebView`는 거대하고 투박하며 체로 가루를 치는 것처럼 메모리가 누수되었습니다.
그런 이유로 모바일 사파리에 뒤처져서 더 빠른 자바스크립트와 렌더링 엔진을 가지고 있음에도 불구하고 아무런 이득을 얻지 못했습니다.

하지만 이런 아쉬운 점들도 `WKWebView`와 `WebKit` 프레임워크가 나오면서 모두 바뀌었습니다.

---

`WKWebView`는 iOS 8과 macOS Yosemite에서 소개된 현대 WebKit API의 핵심이며, UIKit의 `UIWebView`와 AppKit의 `WebView`를 대체하고 두 플랫폼에 일관된 API를 제공합니다.

반응형 60fps 스크롤링, 내장 제스쳐, 앱과 웹페이간의 원활한 통신 그리고 사파리에서 쓰이는 것과 같은 자바스크립트 엔진을 자랑하는 `WKWebView`는 WWDC 2014에서 발표한 가장 의미 있는 발표입니다.

`UIWebView`와 `UIWebViewDelegate`로 이루어져 있던 클래스와 프로토콜은 WebKit 프레임워크에서 14개의 클래스와 3개의 프로토콜로 나누어졌습니다. 너무 복잡하다고 놀라지마세요! 이 새로운 아키텍쳐는 더 깔끔해졌으며 새로운 기능들로 가득합니다.

## UIWebView / WebView에서 WKWebView로 마이그레이션하기

`WKWebView`는 iOS 8 이후 선호되는 API입니다.
여러분의 앱이 _여전히_ 기존의 UIWebView / WebView를 사용중이라면 **`UIWebView`와 `WebView`가 iOS 12와 macOS Mojave에서 공식적으로 폐지 예정(deprecated)이라는 사실을** 꼭 기억하세요.
그러니 `WKWebView`로 최대한 빨리 업데이트해야 합니다.

더 나은 이해를 위해 `UIWebView`와 `WKWebView` API를 비교해봤습니다.

| UIWebView                              | WKWebView                                           |
| -------------------------------------- | --------------------------------------------------- |
| `var scrollView: UIScrollView { get }` | `var scrollView: UIScrollView { get }`              |
|                                        | `var configuration: WKWebViewConfiguration { get }` |
| `var delegate: UIWebViewDelegate?`     | `var UIDelegate: WKUIDelegate?`                     |
|                                        | `var navigationDelegate: WKNavigationDelegate?`     |
|                                        | `var backForwardList: WKBackForwardList { get }`    |

### 로딩

| UIWebView                                                                                                     | WKWebView                                                        |
| ------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| `func loadRequest(request: URLRequest)`                                                                       | `func load(_ request: URLRequest) -> WKNavigation?`              |
| `func loadHTMLString(string: String, baseURL: URL?)`                                                          | `func loadHTMLString(_: String, baseURL: URL?) -> WKNavigation?` |
| `func loadData(_ data: Data, mimeType: String, characterEncodingName: String, baseURL: URL) -> WKNavigation?` |                                                                  |
|                                                                                                               | `var estimatedProgress: Double { get }`                          |
|                                                                                                               | `var hasOnlySecureContent: Bool { get }`                         |
| `func reload()`                                                                                               | `func reload() -> WKNavigation?`                                 |
|                                                                                                               | `func reloadFromOrigin(Any?) -> WKNavigation?`                   |
| `func stopLoading()`                                                                                          | `func stopLoading()`                                             |
| `var request: URLRequest? { get }`                                                                            |                                                                  |
|                                                                                                               | `var URL: URL? { get }`                                          |
|                                                                                                               | `var title: String? { get }`                                     |

### 히스토리

| UIWebView                        | WKWebView                                                                    |
| -------------------------------- | ---------------------------------------------------------------------------- |
|                                  | `func goToBackForwardListItem(item: WKBackForwardListItem) -> WKNavigation?` |
| `func goBack()`                  | `func goBack() -> WKNavigation?`                                             |
| `func goForward()`               | `func goForward() -> WKNavigation?`                                          |
| `var canGoBack: Bool { get }`    | `var canGoBack: Bool { get }`                                                |
| `var canGoForward: Bool { get }` | `var canGoForward: Bool { get }`                                             |
| `var loading: Bool { get }`      | `var loading: Bool { get }`                                                  |

### 자바스크립트 작업

| UIWebView                                                               | WKWebView                                                                                                   |
| ----------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `func stringByEvaluatingJavaScriptFromString(script: String) -> String` |                                                                                                             |
|                                                                         | `func evaluateJavaScript(_ javaScriptString: String, completionHandler: ((AnyObject?, NSError?) -> Void)?)` |

### 기타사항

| UIWebView                                     | WKWebView                                       |
| --------------------------------------------- | ----------------------------------------------- |
| `var keyboardDisplayRequiresUserAction: Bool` |                                                 |
| `var scalesPageToFit: Bool`                   |                                                 |
|                                               | `var allowsBackForwardNavigationGestures: Bool` |

### 페이지네이션

`WKWebView`는 페이지네이션에 해당하는 API를 지금은 가지고 있지 않습니다.

- `var paginationMode: UIWebPaginationMode`
- `var paginationBreakingMode: UIWebPaginationBreakingMode`
- `var pageLength: CGFloat`
- `var gapBetweenPages: CGFloat`
- `var pageCount: Int { get }`

### `WKWebViewConfiguration`을 사용해서 리팩토링하기

다음의 `UIWebView`의 속성들은 독립적인 설정 객체로 나누어졌습니다. 이 설정 객체는 `WKWebView`를 초기화 할 때 사용합니다.

- `var allowsInlineMediaPlayback: Bool`
- `var allowsAirPlayForMediaPlayback: Bool`
- `var mediaTypesRequiringUserActionForPlayback: WKAudiovisualMediaTypes`
- `var suppressesIncrementalRendering: Bool`

---

## 자바스크립트와 스위프트 사이의 통신

`UIWebView` 이후의 가장 크게 바뀐 점 중 하나가 앱과 웹 컨텐츠 사이에 앞뒤로 데이터를 주고받는 방식입니다.

### 사용자 스크립트로 동작 끼워 넣기

`WKUserScript`는 문서를 불러오는 시작과 끝에 자바스크립트를 끼워 넣을 수 있게 해줍니다.
이 강력한 기능은 페이지 요청에 대해 안전하고 일관된 방식으로 웹 컨텐츠를 조작할 수 있습니다.

다음은 웹 페이지의 배경 색을 바꿀 수 있는 사용자 스크립트를 끼워 넣는 방법에 대한 간단한 예제입니다.

```swift
let source = """
    document.body.style.background = "#777";
"""

let userScript = WKUserScript(source: source,
                              injectionTime: .atDocumentEnd,
                              forMainFrameOnly: true)

let userContentController = WKUserContentController()
userContentController.addUserScript(userScript)

let configuration = WKWebViewConfiguration()
configuration.userContentController = userContentController
self.webView = WKWebView(frame: self.view.bounds,
                         configuration: configuration)
```

`WKUserScript` 객체를 만들 때 우리는 실행할 자바스크립트를 제공했고, 문서를 로딩하는 부분의 어떤 지점에 끼워 넣을지 지정했으며, 모든 프레임에서 사용할지 메인 프레임에서만 사용할지도 지정했습니다.
그 후에 `WKWebView`를 초기화할 때 `WKWebViewConfiguration` 객체를 이용해서 사용자 스크립트를 `WKUserContentController`에 추가했습니다.

이 예제는 더 의미 있는 방식으로 확장될 수 있습니다. 예를 들면 [모든 "the cloud"라는 단어를 "my butt"으로 변경](https://github.com/panicsteve/cloud-to-butt)하는 것이 있습니다.

### 메세지 핸들러

메세지 핸들러 도입으로 인해 웹에서 앱으로의 통신이 상당히 개선되었습니다.

`console.log`가 정보를 [Safari Web Inspector](https://developer.apple.com/safari/tools/)로 출력하는 것처럼, 웹 페이지의 정보는 다음과 같은 방법으로 앱에 전달할 수 있습니다.

```javascript
window.webkit.messageHandlers.<#name#>.postMessage()
```

이 API가 정말로 훌륭한 이유는 자바스크립트 객체들은 Objective-C나 스위프트 객체들에게 _자동으로 시리얼라이즈_ 된다는 점입니다.

핸들러의 이름은 `add(_:name)`에서 설정되고 이 메소드는 `WKScriptMessageHandler` 프로토콜을 따르는 핸들러를 등록할 수 있습니다.

```swift
class NotificationScriptMessageHandler: NSObject, WKScriptMessageHandler {
    func userContentController(_ userContentController: WKUserContentController,
                               didReceive message: WKScriptMessage)
    {
        print(message.body)
    }
}

let userContentController = WKUserContentController()
let handler = NotificationScriptMessageHandler()
userContentController.add(handler, name: "notification")
```

이제 앱에 객체가 생성됐을 경우와 같이 알림이 왔을 때, 다음과 같은 식으로 정보를 받을 수 있습니다.

```javascript
window.webkit.messageHandlers.notification.postMessage({ body: "..." });
```

> 웹 페이지 이벤트를 위한 훅을 만드는 사용자 스크립트를 추가합니다.
> 여기선 메세지 핸들러로 앱과 통신을 합니다.

같은 접근법으로 페이지에서 정보를 모아서 앱에서 표시하거나 분석할 수 있습니다.

예를 들면 만약 여러분이 NSHipster.co.kr 전용 브라우저를 만들고 싶다면 관련된 기사를 나열한 버튼을 만들 수도 있을 것입니다.

```javascript
// document.location.href == "https://nshipster.co.kr/wkwebview"
const showRelatedArticles = () => {
  let related = [];
  const elements = document.querySelectorAll("#related a");
  for (const a of elements) {
    related.push({ href: a.href, title: a.title });
  }

  window.webkit.messageHandlers.related.postMessage({ articles: related });
};
```

```swift
let js = "showRelatedArticles();"
self.webView?.evaluateJavaScript(js) { (_, error) in
    print(error)
}

// 미리 등록된 메세지 핸들러에서 결과를 받습니다
```

## 컨텐츠 차단 규칙

어떻게 사용하는지에 따라 자바스크립트와의 왕복 통신의 번거로움을 피할 수 있습니다.

iOS 11과 macOS High Sierra부터 [Safari Content Blocker app extension](https://developer.apple.com/library/archive/documentation/Extensions/Conceptual/ContentBlockingRules/CreatingRules/CreatingRules.html)처럼 `WKWebView`를 위한 컨텐츠 차단 규칙을 구체적으로 선언할 수 있게 되었습니다.

예를 들어 웹 뷰에 [Medium Readable Again](https://makemediumreadable.com)을 만들고 싶으면 다음과 같이 JSON으로 정의할 수 있습니다.

```swift
let json = """
[
    {
        "trigger": {
            "if-domain": "*.medium.com"
        },
        "action": {
            "type": "css-display-none",
            "selector": ".overlay"
        }
    }
]
"""
```

이 규칙을 `compileContentRuleList(forIdentifier:encodedContentRuleList:completionHandler:)`에 전달하고 컴플리션 핸들러의 결과로 오는 컨텐츠 규칙 목록으로 웹 뷰를 설정해주세요.

```swift
WKContentRuleListStore.default()
    .compileContentRuleList(forIdentifier: "ContentBlockingRules",
                            encodedContentRuleList: json)
{ (contentRuleList, error) in
    guard let contentRuleList = contentRuleList,
        error == nil else {
        return
    }

    let configuration = WKWebViewConfiguration()
    configuration.userContentController.add(contentRuleList)

    self.webView = WKWebView(frame: self.view.bounds,
                        configuration: configuration)
}
```

규칙을 선언적으로 선언함으로써 WebKit은 같은 작업을 자바스크립트를 삽입한 경우보다 더 효율적으로 실행할 수 있는 바이트코드로 작업을 컴파일 할 수 있습니다.

페이지 요소를 숨기는 것 외에도 컨텐츠 차단 규칙을 사용하여 페이지 자원이 로드되는 것을 방지하고 (이미지 또는 스크립트와 같이) 쿠키가 서버에 대한 요청에서 분리되고 페이지가 HTTPS를 통해 안전하게 불러오도록 할 수 있습니다.

## 스냅샷

iOS 11과 macOS High Sierra에서 시작해서 WebKit 프레임워크는 웹 페이지의 스크린샷을 찍는 내장 API를 제공합니다.

로딩이 끝난 이후에 화면에 보이는 뷰포트를 찍으려면 `webView(_:didFinish:)` 델리게이트 메소드를 구현하여 `takeSnapshot(with:completionHandler:)` 메소드를 다음과 같이 호출하면 됩니다.

```swift
func webView(_ webView: WKWebView,
            didFinish navigation: WKNavigation!)
{
    var snapshotConfiguration = WKSnapshotConfiguration()
    snapshotConfiguration.snapshotWidth = 1440

    webView.takeSnapshot(with: snapshotConfiguration) { (image, error) in
        guard let image = image,
            error == nil else {
            return
        }

        // ...
    }
}
```

예전엔 웹 페이지의 스크린샷을 찍는 것은 뷰 레이어와 그래픽 컨텍스트를 망친다는 것을 의미했습니다.
반대로 바뀐 API는 깨끗하고 하나의 메소드 옵션을 가지고 있기 때문에 환영받고 있습니다.

---

`WKWebView`는 웹을 진정한 일등 시민으로 만들어주었습니다.
그 어떤 네이티브 순정 주의자라도 WebKit이 제공하는 강력함과 유연성을 보면 놀랄 수밖에 없을겁니다.

실제로 우리가 사용하는 많은 수의 앱들은 까다로운 컨텐츠를 렌더링하기 위해 WebKit에 의존하고 있습니다.
앱 개발에도 모범 사례가 존재하는 것처럼 웹 뷰의 모범 사례는 사용자가 알아채지 못하고 사용할 수 있는지가 지표가 될 것입니다.
