---
title: 번들과 패키지
author: Mattt
translator: 김필권
category: Cocoa
excerpt: "선물이 오가는 시기네요! 오늘은 현대 컴퓨터 시스템이 선사한 가장 훌륭한 선물인 추상화에 대해 생각해보겠습니다."
status:
  swift: "4.2"
---

선물이 오가는 시기네요! 오늘은 현대 컴퓨터 시스템이 선사한 가장 훌륭한 선물인 _추상화_ 에 대해 생각해보겠습니다.

컴퓨터나 모바일 기기를 매일 사용하는 사람은 수십억 명입니다. 그들은 수백만 개의 CPU 트랜지스터와 SSD 셀렉터 그리고 LCD 픽셀에 대해서 생각하지 않습니다. 이 모든 것은 파일, 디렉터리, 앱과 도큐먼트 같은 추상화 덕분입니다.

그래서 이번 주엔 Apple 플랫폼의 가장 중요한 두 추상화인 `번들`과 `패키지`에 대해 알아보도록 하겠습니다. 🎁

---

분명한 개념을 가지고 있음에도 불구하고 "번들"과 "패키지"라는 용어는 자주 서로 바뀌어서 불립니다. 이러한 현상에는 분명 이것들의 이름이 비슷한 것도 한몫하겠지만 가장 큰 이유는 많은 번들들이 패키지이기도 하고 반대의 경우도 작용하기 때문이라고 생각합니다.

그러니 자세히 알아보기 전에 용어를 정의하고자 합니다.

- `번들`은 알려진 것들로 이뤄진 디렉터리이고 실행 가능한 코드와 그 코드가 사용하는 자원을 포함하고 있습니다.

- `패키지`는 파인더에서 봤을 때 파일처럼 보이는 디렉터리입니다.

다음 그림은 번들과 패키지의 관계를 나타냅니다. 여기엔 우리가 잘 아는 앱, 프레임워크, 플러그인 그리고 도큐먼트 등이 있습니다.

{% asset packages-and-bundles-diagram.svg %}

{% info %}
여전히 이 둘이 헷갈리신다면 이해하기 쉽게 비유로 설명하겠습니다.

_패키지_ 는 누군가가 봉인해둔 객체로 인식되는 하나의 _박스_(📦)라고 생각하면 됩니다.
_번들_ 은 비교하자면 _백팩_(🎒)이라고 할 수 있습니다. 이 가방엔 내가 원하는 무엇이든 담을 수 있는 특별한 주머니가 있고 이 주머니는 여러분이 어디를 가던 장소에 딱 맞게 바뀌어서 나올 것입니다.
만약 무언가가 _번들과 패키지 둘 다_ 라면 박스처럼 봉인돼있는데 백팩처럼 칸으로 나뉘어있는 캐리어의 형태를 가질 것입니다.
{% endinfo %}

## 번들

번들은 코드와 자원을 모으는 구조를 제공하여 **개발자 경험을 향상하는 것을** 가장 우선시합니다. 이 구조는 코드나 자원의 예측 가능한 로딩뿐만 아니라 지역화 같은 시스템 차원의 기능도 허용합니다.

번들은 다음과 같은 세 가지로 나눌 수 있습니다. 각각은 특정한 구조와 요구사항을 가지고 있습니다.

- **앱 번들** 은 실행될 수 있는 executable과 그 executable을 설명하는 `Info.plist` 파일 그리고 executable에서 사용되는 런치 이미지를 포함한 에셋과 자원, 인터페이스 파일, 스트링 파일 그리고 데이터 파일로 이루어져 있습니다.
- **프레임워크 번들** 은 동적 공유 라이브러리(Dynamic Shared Library)에서 사용되는 코드와 자원을 포함하고 있습니다.
- **로더블(Loadable) 번들** 은 앱의 기능성을 확장시켜주는 실행 가능한 코드와 자원을 포함하고 있고 플러그인을 예로 들 수 있습니다.

### 번들의 컨텐츠에 접근하기

여러분이 관심 있는 번들이 있다면 그것이 앱이든 플레이그라운드든 무엇이든 `Bundle.main` 속성 타입을 사용해서 접근할 수 있습니다.
대부분의 경우 `url(forResource:withExtension:)` (또는 비슷한 것 중 하나)를 사용해서 특정 자원의 위치를 알아낼 수 있습니다.

예를 들어 만약 여러분의 앱 번들이 `Photo.jpg` 라는 이름을 가진 파일을 포함하고 있으면 다음과 같이 URL을 만들어서 접근할 수 있습니다.

```swift
Bundle.main.url(forResource: "Photo", withExtension: "jpg")
```

{% info %}
혹은 여러분이 에셋 카탈로그(Asset Catalog)를 사용하고 있다면 미디어 라이브러리(<kbd>⇧</kbd><kbd>⌘</kbd><kbd>M</kbd>)에 드래그 앤 드롭하는 것으로 에디터에 이미지 리터럴을 생성할 수 있습니다.
{% endinfo %}

`Bundle`은 표준 번들 아이템의 위치를 제공하는 인스턴스 메소드와 프로퍼티를 제공하는 모든 것은 `URL`과 `String` 을 반환합니다.

| URL                            | Path                            | Description                                      |
| ------------------------------ | ------------------------------- | ------------------------------------------------ |
| `executableURL`                | `executablePath`                | The executable                                   |
| `url(forAuxiliaryExecutable:)` | `path(forAuxiliaryExecutable:)` | The auxiliary executables                        |
| `resourceURL`                  | `resourcePath`                  | The subdirectory containing resources            |
| `sharedFrameworksURL`          | `sharedFrameworksPath`          | The subdirectory containing shared frameworks    |
| `privateFrameworksURL`         | `privateFrameworksPath`         | The subdirectory containing private frameworks   |
| `builtInPlugInsURL`            | `builtInPlugInsPath`            | The subdirectory containing plug-ins             |
| `sharedSupportURL`             | `sharedSupportPath`             | The subdirectory containing shared support files |
| `appStoreReceiptURL`           |                                 | The App Store receipt                            |

### 앱 정보 가져오기

모든 앱 번들은 앱에 대한 정보가 담긴 `Info.plist` 파일을 가집니다.

`bundleURL`과 `bundleIdentifier`를 포함한 몇몇 메타 데이터는 번들의 인스턴스 프로퍼티를 통해 직접 접근할 수 있습니다.

```swift
import Foundation

let bundle = Bundle.main

bundle.bundleURL        // "/path/to/Example.app"
bundle.bundleIdentifier // "com.nshipster.example"
```

`infoDictionary` 프로퍼티에 접근할 수도 있고 사용자에게 보여주는 정보에 접근하려면 `localizedInfoDictionary` 프로퍼티를 사용하면 됩니다.

```swift
bundle.infoDictionary["CFBundleName"] // "Example"
bundle.localizedInfoDictionary["CFBundleName"] // "Esempio" (`it_IT` locale)
```

### 지역화된 스트링 가져오기

번들의 가장 중요한 기능 중 하나는 지역화입니다. 지역화된 에셋의 위치는 어느정도 컨벤션이 정해져서 강요돼있어서 시스템이 추상화하는 로직이 정해져있고 이를 알아내기만 하면 개발자도 가져올 수 있습니다.

예를 들어 번들은 여러분의 앱에서 사용되는 지역화된 스트링을 불러오는 역할을 맡고 있습니다. 이 정보는 `localizedString(forKey:value:table:)` 메소드를 사용해서 접근할 수 있습니다.

```swift
import Foundation

let bundle = Bundle.main
bundle.localizedString(forKey: "Hello, %@",
                       value: "Hello, ${username}",
                       table: nil)
```

그러나 `genstrings` 와 같은 도구를 사용해서 자동으로 추출해서 `.strings` 파일에 붙여주는 방식이 `NSLocalizedString` 보다 훨씬 더 좋습니다.

```swift
NSLocalizedString("Hello, %@", comment: "Hello, ${username}")
```

```terminal
$ find . \( -name "*.swift" !           \ # 모든 스위프트 파일을 찾는다
            ! -path "./Carthage/*"      \ # Carthage든 CocoaPods든
            ! -path "./Pods/*"          \ # 의존성 파일은 모두 무시한다
         \)    |                        \
  tr '\n' '\0' |                        \ # 공백으로 주소 정보를 다루기 위해
  xargs -0 genstrings -o .              \ # 구분자를 모두 NUL로 변경합니다
```

## 패키지

패키지는 관련있는 자원들을 하나의 유닛으로 압축시키고 연결시키는 작업을 통해 **사용자 경험을 향상하기 위해** 만들어졌습니다.

다음과 같은 조건이 맞다면 디렉터리는 파인더에 의해 만들어진 패키지라고 생각할 수 있습니다.

- 디렉터리에 `.app`, `.playground`, 또는 `.plugin`과 같은 특별한 확장자를 가지고 있는 파일이 있다.
- 디렉터리에 도큐먼트 타입으로 등록된 앱의 확장자가 존재한다.
- 디렉터리에 그 자체를 패키지<sup>\*</sup>로 보여지게하는 확장된 어트리뷰트가 존재한다.

### 패키지의 컨텐츠에 접근하기

우리는 파인더에서 컨트롤 클릭을 통해 선택된 아이템과 관련있는 모든 액션 메뉴를 볼 수 있습니다.
만약 아이템이 패키지라면 "Show Package Contents" 가 "Open" 바로 밑에 나올 것입니다.

{% asset show-package-contents.png %}

이 메뉴를 누르면 패키지 디렉터리가 새로운 파인더 윈도우에서 열립니다.

당연히 패키지의 컨텐츠를 코드로 접근할 수도 있습니다.
패키지에 가장 알맞은 방법은 패키지의 종류에 따라 다릅니다.

- 가장 쉬운 경우는 패키지가 번들 구조를 가지고 있을 때입니다. 이때는 이전 섹션에서 설명했던 [`Bundle`](https://developer.apple.com/documentation/foundation/bundle)를 사용하면 됩니다.
- 패키지가 도큐먼트라면 macOS에선 [`NSDocument`](https://developer.apple.com/documentation/appkit/nsdocument)를, iOS에선 [`UIDocument`](https://developer.apple.com/documentation/uikit/uidocument)를 사용하시면 됩니다.
- 디렉터리, 파일 그리고 심볼 링크들을 접근하려면 [`FileWrapper`](https://developer.apple.com/documentation/foundation/filewrapper)를 사용하면 되고 파일 설명을 읽으려면 [`FileHandler`](https://developer.apple.com/documentation/foundation/filehandle)를 사용하면 됩니다.

### 디렉터리가 패키지인지 결정하는 방법

디렉터리가 패키지인지 결정하는 방법이 파인더가 그들의 파일과 디렉터리를 어떻게 표현하려는 지에 달려있다고 하더라도 대부분 운영 체제에 의해 결정되는 것이고 이러한 서비스는 Uniform Type Identifiers를 관리하는 역할을 가집니다.

파일의 확장자를 내장된 시스템 패키지 타입으로 할지 아니면 등록된 도큐먼트 타입으로 설치된 앱에서 사용할지 결정하려고 한다면 Core Service의 함수인 `UTTypeCreatePreferredIdentifierForTag(_:_:_:)`와
`UTTypeConformsTo(_:_:)`를 사용하면 됩니다.

```swift
import Foundation
import CoreServices

func directoryIsPackage(_ url: URL) -> Bool {
    let filenameExtension: CFString = url.pathExtension as NSString
    guard let uti = UTTypeCreatePreferredIdentifierForTag(
                        kUTTagClassFilenameExtension,
                        filenameExtension, nil
                    )?.takeRetainedValue()
    else {
        return false
    }

    return UTTypeConformsTo(uti, kUTTypePackage)
}

let xcode = URL(fileURLWithPath: "/Applications/Xcode.app")
directoryIsPackage(xcode) // true
```

{% info %}

저희는 파일의 "패키지 비트(Package Bit)"라고 불리는 것을 설정하는 방법에 대해 설명한 문서를 찾지 못했지만 [CarbonCore/Finder.h](https://opensource.apple.com/source/CarbonHeaders/CarbonHeaders-8A428/Finder.h)에 따르면 이는 `com.apple.FinderInfo`의 확장 어트리뷰트인 `kHasBundle` (`0x2000`) 플래그를 설정하는 것으로 해낼 수 있다고 합니다.

```terminal
$ xattr -wx com.apple.FinderInfo /path/to/package \
  00 00 00 00 00 00 00 00 20 00 00 00 00 00 00 00 \
  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

{% endinfo %}

---

글에서 보았듯이 추상화를 하는 것의 이익을 얻는 것은 엔드 유저만이 아닙니다. Swift 같은 안전하고 표현이 풍부한 고급 프로그래밍 언어나 Foundation과 같은 편한 API를 쓰는 것에도 우리 개발자들은 추상화의 영향을 받아 더 나은 소프트웨어를 만들고 있습니다.

우리가 만나는 [leaky](https://en.wikipedia.org/wiki/Leaky_abstraction)하고 [inverted](https://en.wikipedia.org/wiki/Abstraction_inversion)한 추상화에 대해 불평하기 전에 한 발짝 물러서서 얼마나 많은 추상화가 우리의 일상의 많은 일을 가능하게 해준다는 사실을 생각한다면 어떨까요?
