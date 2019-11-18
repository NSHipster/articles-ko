---
title: SwiftUI 없이 Xcode Preview 사용하기
author: Mattt
translator: 김필권
category: Xcode
excerpt: >-
  어마어마한 양의 iOS 코드를 다루는 일은 때론 기다림의 연속입니다.
  하지만 Xcode 11와 SwiftUI가 함께라면 더 이상 기다릴 필요가 없습니다.
status:
  swift: 5.1
---

어마어마한 양의 iOS 코드를 다루는 일은 때론 Xcode가 파일을 인덱싱하는 것을, Swift와 Objective-C 코드가 컴파일되기를, 시뮬레이터가 켜지고 앱이 실행되기를 기다리는 등 기다림의 연속입니다.

심지어 우리는 방금 추가한 오토 레이아웃 하나 때문에 앱의 특정 상태, 특정 화면을 보기 위해 계속해서 시간을 소비합니다.
만약 기대한 결과가 나오지 않는다면 다시 Xcode로 돌아가서 Content Hugging 우선순위를 조정하고 <kbd>⌘</kbd><kbd>R</kbd>을 눌러 모든 과정을 다시 시작합니다.

[이런 상황](https://xkcd.com/303/)을 즐기는 분도 계시겠지만 그렇지 않은 분에겐 [Joel Spolsky가 소개하는](https://www.joelonsoftware.com/2001/12/11/back-to-basics/) 옛날 농담이 생각날 수 있겠네요. 이 농담을 iOS식으로 변형해봤습니다.

> Shlemiel은 iOS 앱을 만드는 개발자가 되었습니다.
> 그는 첫 스프린트에서 화면 10개를 쳤습니다.
> 관리자는 _"끝내주네요! 당신은 정말 손이 빠르네요!"_ 라고 하며 비트코인을 주었습니다.
>
> 다음 스프린트에서 그는 다섯 화면을 쳤습니다.
> 관리자는 _"흠 저번보다 못하셨네요. 하지만 여전히 손은 빠르신 것 같습니다. 화면 5개도 빠른 거죠."_ 라고 하며 비트코인을 주었습니다.
> 
> 다음 스프린트에서는 화면을 하나밖에 못 쳤습니다.
> _"하나요?"_ 관리자는 소리쳤습니다.
> _"말이 된다고 생각하세요? 처음 스프린트에선 10배나 많은 일을 하셨잖아요. 무슨 일 생기셨나요?"_
> 
> _"어쩔 수 없었어요."_ Shlemiel이 말했습니다.
> <em>"스프린트가 진행될수록 `application(_:didFinishLaunchingWithOptions:)`의 코드가 커져서 어쩔 수가 없었다고요!"</em>

지난 몇 년간, 코드를 줄이는 것에 대한 개발은 발전해왔습니다. [`@IBInspectable` and `@IBDesignable`](/ibinspectable-ibdesignable/)과 [Xcode Playgrounds](/xcplayground/)도 그중 일부라 할 수 있죠.
드디어 Xcode 11과 함께 끝판왕이 나왔습니다. 바로 SwiftUI입니다.

---

{% warning %}

이 글에서 소개하는 기능은 다음과 같은 항목이 필요합니다.

- **Xcode 11**
- **macOS Catalina**
- **Debug** 설정의 **Deployment Target**이 **iOS 13**으로 설정된 앱

위 세 가지가 충족되지 않으면 여러분의 코드는 컴파일되지도, 실시간 렌더링 되지도 않을 것입니다.

{% endwarning %}

---

많은 분이 아직 SwiftUI를 보고만 있는 상태겠지만, 우리는 이 기술의 가능성을 통해 개발 프로세스를 더 빠르고 더 좋게 만들 것입니다.
_UIKit 앱의 코드 한 줄 바꾸지 않고 말이죠._

`UIButton`의 서브클래스로 테두리를 그리는 버튼이 있다면 코드는 다음과 같을 것입니다.

```swift
final class BorderedButton: UIButton {
    var cornerRadius: CGFloat { <#...#> }
    var borderWidth: CGFloat { <#...#> }
    var borderColor: UIColor? { <#...#> }
}
```

보통 우리가 만든 UI가 어떻게 작동하는지 테스트하기 위해서는 뷰의 어딘가에 추가한 후, 빌드, 실행 그리고 그 화면까지 가야했습니다.
하지만 Xcode 11를 사용하고 있다면 `BorderedButton`의 선언 밑에 다음과 같은 내용을 넣으면 실시간으로 미리보기를 볼 수 있습니다.

<div class="code-with-automatic-preview">

```swift
#if canImport(SwiftUI) && DEBUG
import SwiftUI

@available(iOS 13.0, *)
struct BorderedButton_Preview: PreviewProvider {
  static var previews: some View {
    UIViewPreview {
      let button = BorderedButton(frame: .zero)
      button.setTitle("Follow", for: .normal)
      button.tintColor = .systemOrange
      button.setTitleColor(.systemOrange, for: .normal)
      return button
    }.previewLayout(.sizeThatFits)
     .padding(10)
  }
}
#endif
```

<aside>
{% asset swiftui-preview-follow.png alt="SwiftUI preview with Follow button" %}
</aside>

</div>

<dfn>동적 대체(dynamic replacement)</dfn>라는 새로운 기능을 사용하면 Xcode는 새로운 컴파일 없이 여러분이 코드를 작성하고 있는 순간과 동시에 미리보기를 업데이트할 수 있게 됩니다.
이것은 이전에는 생각지도 못한 속도로 프로토타입을 생성할 수 있게 됐다는 의미입니다.

타이틀이 길어졌을 때 여러분의 버튼이 어떻게 바뀔지 궁금하신가요?
`setTitle(_:for:)`을 호출하는 부분에 원하는 만큼 입력하시고 내 버튼의 잠재력을 느껴보세요. 그것도 작성하던 파일을 벗어나지 않고요!

{% info %}

`UIViewPreview`는 `UIView` 서브클래스의 미리보기를 보여줄 수 있는 컨벤션을 지닌 커스텀하며 제네릭한 구조를 가집니다.
[소스](https://gist.github.com/mattt/ff6b58af8576c798485b449269d43607)는 gist에 올려두었으니 마음껏 가져가시고 프로젝트에 바로 적용해보세요.

SwiftUI를 사용하지 않는 앱에서 Xcode Preview를 작동하기 위해서는 조건적인 import를 통해 적절한 디펜던시 사용과 Deployment Target을 iOS 13으로 설정하는 작업이 필요합니다.
이런 경우엔 저 파일을 직접적으로 임베드하는 것이 최고입니다.

{% capture uiviewpreview %}

```swift
import UIKit

#if canImport(SwiftUI) && DEBUG
import SwiftUI
struct UIViewPreview<View: UIView>: UIViewRepresentable {
    let view: View

    init(_ builder: @escaping () -> View) {
        view = builder()
    }

    // MARK: - UIViewRepresentable

    func makeUIView(context: Context) -> UIView {
        return view
    }

    func updateUIView(_ view: UIView, context: Context) {
        view.setContentHuggingPriority(.defaultHigh, for: .horizontal)
        view.setContentHuggingPriority(.defaultHigh, for: .vertical)
    }
}
#endif
```

{% endcapture %}

{::nomarkdown}

<details>
<summary><code>UIViewPreview</code>의 모든 구현을 보려면 클릭하세요!</summary>
{{ uiviewpreview | markdownify }}
</details>
{:/}

{% endinfo %}

## 다양한 상태 미리보기

`FavoriteButton`이라는 버튼이 앱에 있다고 해보겠습니다.
아마도 `BorderedButton`의 먼 사촌이겠죠?
그리고 기본 상태에선 "Favorite"라는 제목과 <span title="Heart">♡</span> 아이콘을 가질 것입니다.
만약 `isFavorited` 속성이 `true`가 된다면, 타이틀이 "Unfavorite"로 바뀌고 아이콘은 <span title="Heart with slash">♡̸</span>가 될 것입니다.

SwiftUI의 `Group`을 사용한다면 두 개의 `UIViewPreview` 인스턴스도 동시에 볼 수 있습니다.

<div class="code-with-automatic-preview">

```swift
Group {
  UIViewPreview {
    let button = FavoriteButton(frame: .zero)
    return button
  }
  UIViewPreview {
    let button = FavoriteButton(frame: .zero)
    button.isFavorited = true
    return button
  }
}.previewLayout(.sizeThatFits)
 .padding(10)
```

<aside>
{% asset swiftui-preview-favorite-unfavorite.png alt="SwiftUI previews with Favorite and Unfavorite buttons" %}
</aside>

</div>

{% info %}

그룹 끝에 붙어있는 `previewLayout`과 `padding` 메소드는 `Group`의 각 멤버에게 적용됩니다.
미리보기를 원하는 방식으로 보기 위해 [여러 가지 `View` 메소드](https://developer.apple.com/documentation/swiftui/view)가 제공됩니다.

{% endinfo %}

## 다크 모드에서 미리보기

iOS 13에서 소개된 [다크 모드](/dark-mode/)에서도 여러분의 커스텀 뷰가 잘 나오는지 확인해야겠죠?

다크 모드를 확인할 수 있는 가장 쉬운 방법은 `UIViewPreview`의 `ColorScheme`을 돌면서 하나씩 확인하는 것입니다.

<div class="code-with-automatic-preview">

```swift
ForEach(ColorScheme.allCases, id: \.self) { colorScheme in
    UIViewPreview {
      let button = BorderedButton(frame: .zero)
      button.setTitle("Subscribe", for: .normal)
      button.setImage(UIImage(systemName: "plus"), for: .normal)
      button.setTitleColor(.systemOrange, for: .normal)
      button.tintColor = .systemOrange
      return button
  }.environment(\.colorScheme, colorScheme)
   .previewDisplayName("\(colorScheme)")
}.previewLayout(.sizeThatFits)
 .background(Color(.systemBackground))
 .padding(10)
```

<aside>
{% asset swiftui-preview-color-schemes.png alt="SwiftUI previews with different color schemes" %}
</aside>

</div>

{% info %}

`ForEach`를 사용해서 미리보기를 렌더링할 때는 `previewDisplayName` 메소드를 사용해서 각 컬러 스킴의 이름을 출력해서 구별할 수 있게 하는 것이 좋습니다.

{% endinfo %}

## 동적 타입의 사이즈별 미리보기

다양한 [동적 타입 사이즈(Dynamic Type Size)](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/typography/)의 뷰를 미리 보는 것도 비슷한 접근법을 사용하면 됩니다.

<div class="code-with-automatic-preview">

```swift
ForEach(ContentSizeCategory.allCases, id: \.self) { sizeCategory in
  UIViewPreview {
      let button = BorderedButton(frame: .zero)
      button.setTitle("Subscribe", for: .normal)
      button.setImage(UIImage(systemName: "plus"), for: .normal)
      button.setTitleColor(.systemOrange, for: .normal)
      button.tintColor = .systemOrange
      return button
  }.environment(\.sizeCategory, sizeCategory)
   .previewDisplayName("\(sizeCategory)")
}.previewLayout(.sizeThatFits)
 .padding(10)
```

<aside>
{% asset swiftui-preview-content-size-categories.png alt="SwiftUI previews with different content size categories" %}
</aside>

</div>

## 서로 다른 지역별 미리보기

Xcode Preview는 앱을 다양한 언어로 로컬라이징할 때 그 진가가 발휘됩니다.
시뮬레이터의 언어와 지역 설정을 바꿨다가 되돌렸다가 하던 것에 비교하면 천지 차이입니다.

여러분의 앱이 기본적으로 영어를 지원하고 있는데 [오른쪽에서 왼쪽으로 쓰는 언어](https://en.wikipedia.org/wiki/Right-to-left)를 지원해야 하는 상황을 예로 들어보겠습니다.
아래와 같이 할 수 있겠네요.

<div class="code-with-automatic-preview">

```swift
let supportedLocales: [Locale] = [
  "en-US", // English (United States)
  "ar-QA", // Arabic (Qatar)
  "he-IL", // Hebrew (Israel)
  "ur-IN"  // Urdu (India)
].map(Locale.init(identifier:))

func localizedString(_ key: String, for locale: Locale) -> String? { <#...#> }

return ForEach(supportedLocales, id: \.identifier) { locale in
  UIViewPreview {
    let button = BorderedButton(frame: .zero)
    button.setTitle(localizedString("Subscribe", for: locale), for: .normal)
    button.setImage(UIImage(systemName: "plus"), for: .normal)
    button.setTitleColor(.systemOrange, for: .normal)
    button.tintColor = .systemOrange
    return button
  }.environment(\.locale, locale)
   .previewDisplayName(Locale.current.localizedString(forIdentifier: locale.identifier))
}.previewLayout(.sizeThatFits)
 .padding(10)
```

<aside>
{% asset swiftui-preview-locales.png alt="SwiftUI previews with different locales" %}
</aside>

</div>

{% info %}

`NSLocalizedString`을 통해서 로컬라이제이션 테스트를 제대로 하는 것은 쉽지 않습니다.
하지만 미리보기와 함께라면 원하는 텍스트를 직접 입력해서 테스트할 수 있습니다.

{% endinfo %}

## 서로 다른 기기에서 뷰 컨트롤러 미리보기

SwiftUI의 미리보기는 뷰에만 국한된 기능이 아닙니다. 이 기능은 뷰 컨트롤러에도 사용할 수 있습니다.
[커스텀 `UIViewControllerPreview` 타입](https://gist.github.com/mattt/ff6b58af8576c798485b449269d43607)을 생성하면 [iOS 13의 새로운 `UIStoryboard` 클래스 메소드](https://nshipster.com/ios-13/#remove-implicitly-unwrapped-optionals-from-view-controllers-initialized-from-storyboards)의 이점을 얻어서 서로 다른 기기에서 뷰 컨트롤러를 미리보기할 수 있습니다.

<div class="code-with-automatic-preview">

```swift
#if canImport(SwiftUI) && DEBUG
import SwiftUI

let deviceNames: [String] = [
    "iPhone SE",
    "iPad 11 Pro Max",
    "iPad Pro (11-inch)"
]

@available(iOS 13.0, *)
struct ViewController_Preview: PreviewProvider {
  static var previews: some View {
    ForEach(deviceNames, id: \.self) { deviceName in
      UIViewControllerPreview {
        UIStoryboard(name: "Main", bundle: nil)
            .instantiateInitialViewController { coder in
            ViewController(coder: coder)
        }!
      }.previewDevice(PreviewDevice(rawValue: deviceName))
        .previewDisplayName(deviceName)
    }
  }
}
#endif
```

<aside>
{% asset swiftui-preview-devices.png alt="SwiftUI previews with different devices" %}
</aside>

</div>

{% error %}

단점이 하나 있다면 가로 모드에서는 SwiftUI 기기 미리보기를 할 방법이 없다는 것입니다.
하지만 미리보기 레이아웃을 특정한 사이즈로 고정하면 가로 모드를 비슷하게 확인할 수 있습니다.
이 방법은 아이폰의 Safe Area, 아이패드의 Split View를 정확하게 반영하지 않기 때문에 정확하지는 않습니다.

{% enderror %}

---

대부분의 사람들이 SwiftUI를 실제로 앱에 적용하려면 몇 년 남았다고 생각하고 있을 것입니다. (자의든 타의든 말이죠)
하지만 macOS Catalina의 Xcode 11만 있다면 앞에서 설명한 방법을 통해서 즉시 얻을 수 있는 이득이 많습니다.

SwiftUI로 바뀔 때까지 시간을 죽이는 것은 매주 몇 시간씩 날릴 뿐만 아니라, 개발 흐름을 끊기지 않게 해줄 수 있는 가능성을 잃어버리는 것입니다.
뿐만 아니라 통합 테스팅의 편리함은 근본적으로 테스팅에 대한 인식을 바꿀 것입니다. 지금까진 가끔 _"있어서 좋네"_ 라고 생각하는 정도였다면 이젠 새로운 기본값이 될 거라 생각합니다.

추가로, 이 미리보기 기능은 크고 작은 팀을 위한 라이브 문서로도 활용될 수 있을 것입니다. 더 나아가선 디자인 시스템에 사용될 수도 있겠죠?

Xcode Preview가 iOS 개발에 있어서 판도를 얼마나 뒤엎을지 평가하는 것은 어렵습니다만, 이 기능을 우리 프로젝트에 붙인다면 그보다 더 행복할 순 없을 것입니다.

{% asset articles/swiftui-previews.css %}
