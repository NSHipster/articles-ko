---
title: Bundles and Packages
author: Mattt
translator: 김필권
category: Cocoa
excerpt: "In this season of giving, let's stop to consider one of the greatest gifts given to us by modern computer systems: the gift of abstraction."
status:
  swift: "4.2"
---

선물이 오가는 시기네요! 오늘은 현대 컴퓨터 시스템이 선사한 가장 훌륭한 선물인 _추상화라는 선물_ 에 대해 생각해보겠습니다.

컴퓨터나 모바일 기기를 매일 사용하는 사람은 수십억명입니다. 그들은 수백만개의 CPU 트랜지스터와 SSD 셀렉터 그리고 LCD 픽셀에 대해서 생각하지 않습니다. 이 모든 것은 파일, 디렉토리, 앱과 도큐먼트같은 추상화덕분입니다.

그래서 이번 주엔 Apple 플랫폼의 가장 중요한 두 추상화인 `번들`과 `패키지`에 대해 알아보도록 하겠습니 다. 🎁

---

분명한 개념을 가지고 있음에도 불구하고 "번들"과 "패키지"라는 용어는 자주 서로 바뀌어서 불립니다. 이러한 현상에는 분명 이것들의 이름이 비슷한 것도 한 몫 하겠지만 가장 큰 이유는 많은 번들들이 패키지이기도 하고 반대의 경우도 작용하기 때문이라고 생각합니다.

그러니 자세히 알아보기 전에 용어를 정의하고자 합니다.

- `번들`은 알려진 것들로 이뤄진 디렉토리이고 실행가능한 코드와 그 코드가 사용하는 자원을 포함하고 있습니다.

- `패키지`는 파인더에서 봤을 때 파일처럼 보이는 디렉토리입니다.

다음 그림은 번들과 패키지의 관계를 나타냅니다. 여기엔 우리가 잘 아는 앱, 프레임워크, 플러그인 그리고 도큐먼트 등이 있습니다.

{% asset packages-and-bundles-diagram.svg %}

{% info %}
여전히 이 둘을 나누는 기준에 대해 애매하다면 이해하기 쉽게 비유로 설명할 수 있습니다.

_패키지_ 를 누군가가 봉인해둬서 한 객체로 인식되는 하나의 _박스_(📦)라고 생각하면 됩니다.
_번들_ 은 비교하자면 _백팩_(🎒)에 더 가깝습니다. 여러분이 원하는 무엇이든 담을 수 있는 특별한 주머니가 있고 이 주머니에선 여러분이 학교, 직장, 체육관 등 가는 장소에 맞게 다른 설정이 적용돼 나올 것입니다.
만약 무언가가 _번들과 패키지 둘 다_ 라면 박스처럼 봉인돼있는데 백팩처럼 칸으로 나눠져있는 하나의 캐리어라고 할 수 있겠습니다.
{% endinfo %}

## 번들

번들은 코드와 자원을 모으는 구조를 제공하여 **개발자 경험을 향상시키는 것을** 가장 우선시 합니다. 이 구조는 코드나 자원의 예측 가능한 로딩뿐만 아니라 지역화같은 시스템 차원의 기능도 허용합니다.

번들은 다음과 같은 세 가지로 나눌 수 있습니다. 각각은 특정한 구조와 요구사항을 가지고 있습니다.

- **앱 번들** 은 실행될 수 있는 executable과 그 executable을 설명하는 `Info.plist` 파일 그리고 executable에서 사용되는 런치 이미지를 포함한 에셋과 자원, 인터페이스 파일, 스트링 파일 그리고 데이터 파일로 이루어져 있습니다.
- **프레임워크 번들** 은 동적 공유 라이브러리(Dynamic Shared Library)에서 사용되는 코드와 자원을 포함하고 있습니다.
- **로더블(Loadable) 번들** 은 앱의 기능성을 확장시켜주는 실행가능한 코드와 자원을 포함하고 있고 플러그인을 예로 들면 됩니다.

### 번들 컨텐츠에 접근하기

여러분이 관심있는 번들이 있다면 그것이 앱이든 플레이그라운드든 무엇이든 `Bundle.main` 속성 타입을 사용해서 접근이 가능합니다.
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

모든 앱 번들은 앱에 대한 정보가 담긴 `Info.plist` 파일을 필요로 합니다.

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

## Packages

Packages are primarily for **improving user experience**
by encapsulating and consolidating related resources into a single unit.

A directory is considered to be a package by the Finder
if any of the following criteria are met:

- The directory has a special extension like `.app`, `.playground`, or `.plugin`
- The directory has an extension that an app has registered as a document type
- The directory has an extended attribute designating it as a package <sup>\*</sup>

### Accessing the Contents of a Package

In Finder,
you can control-click to show a contextual menu
with actions to perform on a selected item.
If an item is a package,
"Show Package Contents" will appear at the top,
under "Open".

{% asset show-package-contents.png %}

Selecting this menu item will open a new Finder window
from the package directory.

You can, of course,
access the contents of a package programmatically, too.
The best option depends on the kind of package:

- If a package has bundle structure,
  it's usually easiest to use
  [`Bundle`](https://developer.apple.com/documentation/foundation/bundle)
  as described in the previous section.
- If a package is a document, you can use
  [`NSDocument`](https://developer.apple.com/documentation/appkit/nsdocument) on macOS
  and [`UIDocument`](https://developer.apple.com/documentation/uikit/uidocument) on iOS.
- Otherwise, you can use
  [`FileWrapper`](https://developer.apple.com/documentation/foundation/filewrapper)
  to navigate directories, files, and symbolic links,
  and [`FileHandler`](https://developer.apple.com/documentation/foundation/filehandle)
  to read and write to file descriptors.

### Determining if a Directory is a Package

Although it's up to the Finder how it wants to represent files and directories,
most of that is delegated to the operating system
and the services responsible for managing
Uniform Type Identifiers (<abbr title="Uniform Type Identifiers">UTI</abbr>).

If you want to determine whether a file extension
is one of the built-in system package types
or used by an installed app as a registered document type,
you can use the Core Services functions
`UTTypeCreatePreferredIdentifierForTag(_:_:_:)` and
`UTTypeConformsTo(_:_:)`:

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

We couldn't find any documentation describing
how to set the so-called "package bit" for a file,
but according to
[CarbonCore/Finder.h](https://opensource.apple.com/source/CarbonHeaders/CarbonHeaders-8A428/Finder.h),
this can be accomplished by setting the
`kHasBundle` (`0x2000`) flag
in the `com.apple.FinderInfo` extended attribute:

```terminal
$ xattr -wx com.apple.FinderInfo /path/to/package \
  00 00 00 00 00 00 00 00 20 00 00 00 00 00 00 00 \
  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

{% endinfo %}

---

As we've seen,
it's not just end-users that benefit from abstractions ---
whether it's the safety and expressiveness of
a high-level programming language like Swift
or the convenience of APIs like Foundation,
we as developers leverage abstraction to make great software.

For all that we may (rightfully) complain
about abstractions that are
[leaky](https://en.wikipedia.org/wiki/Leaky_abstraction) or
[inverted](https://en.wikipedia.org/wiki/Abstraction_inversion),
it's important to take a step back
and realize how many useful abstractions we deal with every day,
and how much they allow us to do.
