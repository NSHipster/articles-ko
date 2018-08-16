---
title: "Password Rules / UITextInputPasswordRules"
author: Mattt
translator: 김필권
category: "Cocoa"
excerpt: 90년대의 해커 영화의 제목이나 방탈출의 해결법이 아닌 이상 비밀번호는 반드시 의미가 없어야 합니다.
hiddenlang: ""
status:
    swift: "4.2"
---

힙스터들이 _이것_ 에 과하게 장인정신을 가졌고 _저것_ 을 직접 만든것은 조금도 이상한 일이 아닙니다. 그것은 두꺼운 조각의 아보카도 토스트일 수도 있고, 작은 무지방 황금 우유일 수도 있으며 완벽한 한 잔의 푸어 오버 커피일 수도 있습니다. 이 모든 것들의 공통점은 완성하려면 인간의 손길이 필요하다는 것입니다.

앞의 것들과는 아주 대조적으로 좋은 비밀번호는 장인정신과는 정반대에 있습니다. 90년대의 해커 영화의 제목이나 방탈출의 해결법이 아닌 이상 비밀번호는 반드시 의미가 없어야 합니다.

그런 의미에서 iOS 12와 macOS Mojave의 Safari의 새로운 기능은 정말 반가운 기능입니다. 이제 비밀번호 생성이 더욱 강력하고 의미 없어지며 더 추측 불가능해집니다.

---

이상적인 비밀번호 정책은 간단합니다. 최소한의 글자수를 강제하고 (최소 8글자) 긴 글자수(64글자 또는 더 많이)도 허용해주는 것입니다.

더 정교한 시스템을 생각한다면 미리 선택된 보안 질문과 정기적인 비밀번호 기간 만료 또는 불가사의한 글자를 입력하는 기능을 넣어서 사람들의 비밀번호를 보호하세요.

> 하지만 저도 보안 전문가는 아니기 때문에 완벽한 방법은 아닙니다.
> 더 좋은 방법은 <abbr title="National Institute of Standards and Technology">NIST</abbr>에서 발행한 [Digital Identity Guidelines](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-63b.pdf)을 확인하는 것입니다. (2017년 6월에 발행되었습니다)

좋은 소식은 많은 단체와 회사들이 보안에 더 많은 신경을 쓰기 시작했다는 것입니다. 나쁜 소식은 이러한 변화를 위해 수백만의 사람들이 데이터 결함을 겪어야 한다는 것입니다. 그리고 알고 싶지 않은 사실이 있다면, 기업과 정부가 아무것도 하고있지 않기 때문에 앞서 말한 보안에 위반되는 패턴들이 당장은 사라지지 않을 거라는 것입니다.

## Automatic Strong Passwords

Safari의 AutoFill은 iOS 8부터 비밀번호 생성이 가능했지만 그 단점 중 하나는 생성된 비밀번호가 특정 서비스에 만족하는지는 보장되지 않는다는 점이었습니다.

Apple은 이 문제를 해결하는 것을 목표로 새로운 Automatic Strong Password 기능을 iOS 12와 macOS Mojave의 Safari에서 공개했습니다.

WebKit 엔지니어인 Daniel Bates는 <abbr title="Web Hypertext Application Technology Working Group">WHATWG</abbr>에 [이 제안](https://github.com/whatwg/html/issues/3518)을 3월 1일에 제출했습니다. 6월 6일에 WebKit 팀은 [Safari Technology Preview Release 58](https://webkit.org/blog/8327/safari-technology-preview-58-with-safari-12-features-is-now-available/)을 발표했고 이 버전에는 `passwordrules` 속성을 사용해서 강력한 비밀번호 생성하는 기능이 포함되어 있었습니다. 그리고 같은 시간 WWDC에서는 `UITextInputPasswordRules` API와 Security Code AutoFill, 연합 인증 (federated authentication)과 같은 비밀번호 관리 기능 `UITextInputPasswordRules` API가 포함된 iOS 12 베타 SDK가 릴리즈되었습니다.

## 비밀번호 규칙 (Password Rules)

비밀번호 규칙은 비밀번호 생성기의 레시피라고 생각하시면 됩니다. 비밀번호 생성기는 몇가지 간단한 규칙을 따르면서 무작위로 새롭고 보안적으로 튼튼하며 서비스 제공자들의 구체적인 요구에도 통과하는 비밀번호를 만들어줍니다.

비밀번호 규칙은 다음과 같은 모양의 하나 또는 그 이상의 키-밸류 쌍으로 이루어져 있습니다.

`required: lower; required: upper; required: digit; allowed: ascii-printable; max-consecutive: 3;`

### 키

하나의 키는 각각의 규칙을 구체적으로 가지고 있습니다.

- `required`: 비밀번호에서 요구로 하는 문자의 종류
- `allowed`: 비밀번호에서 사용할 수 있는 문자의 종류
- `max-consecutive`: 연속으로 사용할 수 있는 문자의 수
- `minlength`: 비밀번호 최대 길이
- `maxlength`: 비밀번호 최소 길이

`required`와 `allowed` 키는 아래에서 나열한 값 중 하나를 그것의 값으로 가집니다. `max-consecutive`, `minlength`, `maxlength` 키는 0이상의 정수를 값으로 가집니다.

### 문자 클래스

`required`와 `allowed` 키는 다음 목록 중 하나 이상의 값을 가집니다.

- `upper` (`A-Z`)
- `lower` (`a-z`)
- `digits` (`0-9`)
- `special` (`` -~!@#$%^&\*\_+=`|(){}[:;"'<>,.? ] `` and space)
- `ascii-printable` (U+0020 — 007f)
- `unicode` (U+0 — 10FFFF)

이 설정값 외에도, 중괄호로 둘러싸인 ASCII 문자열을 커스텀 문자열 클래스로 지정할 수도 있습니다. (예: `[abc]`)

---

Apple의 [Password Rules Validation Tool](https://developer.apple.com/password-rules/)은 우리가 다양한 규칙으로 실시간 피드백을 받을 수 있는 실험을 가능하게 해줍니다. 심지어 개발과 테스트를 하는 동안 사용할 수천개의 비밀번호를 생성하고 다운로드 받을 수도 있습니다!

{% asset password-rules-validation-tool.png alt="Password Rules Validation Tool" %}

비밀번호 규칙 문법에 대해 더 자세히 알고 싶다면 Apple의 ["Customizing Password AutoFill Rules"](https://developer.apple.com/documentation/security/password_autofill/customizing_password_autofill_rules)를 참고하세요.

---

## 비밀번호 규칙 지정하기

iOS에서는 `UITextField`의 `passwordRules` 속성을 `UITextInputPasswordRules` 객체를 사용해서 지정할 수 있습니다. (또한 `textContentType`을 `.newPassword`로 지정하셔야 합니다.)

```swift
let newPasswordTextField = UITextField()
newPasswordTextField.textContentType = .newPassword
newPasswordTextField.passwordRules = UITextInputPasswordRules(descriptor: "required: upper; required: lower; required: digit; max-consecutive: 2; minlength: 8;")
```

웹에서는 `<input>` 요소의 `passwordRules` 속성을 `type="password"` 와 함께 입력하시면 됩니다.

```html
<input type="password" passwordrules="required: upper; required: lower; required: special; max-consecutive: 3;"/>
```

> 만약 따로 지정하지 않는다면 기본 비밀번호 규칙은 `allowed: ascii-printable` 이 됩니다.
> 입력 폼에 비밀번호 확인 필드가 존재한다면 자동으로 이전 필드의 규칙을 가져올 것입니다.

## Swift에서 비밀번호 규칙 생성하기

적절한 추상 없이 문자열 기반으로 작업하는 것이 끔찍하다고 느끼신다면 혼자가 아니십니다.

다음은 Swift API로 비밀번호 규칙을 만드는 방법에 대해 요약입니다. ([Swift 패키지에서도 사용가능합니다!](https://github.com/NSHipster/PasswordRules))

```swift
enum PasswordRule {
    enum CharacterClass {
        case upper, lower, digits, special, asciiPrintable, unicode
        case custom(Set<Character>)
    }

    case required(CharacterClass)
    case allowed(CharacterClass)
    case maxConsecutive(UInt)
    case minLength(UInt)
    case maxLength(UInt)
}

extension PasswordRule: CustomStringConvertible {
    var description: String {
        switch self {
        case .required(let characterClass):
            return "required: \(characterClass)"
        case .allowed(let characterClass):
            return "allowed: \(characterClass)"
        case .maxConsecutive(let length):
            return "max-consecutive: \(length)"
        case .minLength(let length):
            return "minlength: \(length)"
        case .maxLength(let length):
            return "maxlength: \(length)"
        }
    }
}

extension PasswordRule.CharacterClass: CustomStringConvertible {
    var description: String {
        switch self {
        case .upper: return "upper"
        case .lower: return "lower"
        case .digits: return "digits"
        case .special: return "special"
        case .asciiPrintable: return "ascii-printable"
        case .unicode: return "unicode"
        case .custom(let characters):
            return "[" + String(characters) + "]"
        }
    }
}
```

With this in place, we can now specify a series of rules in code and use them to generate a string with valid password rules syntax:


```swift
let rules: [PasswordRule] = [ .required(.upper),
                              .required(.lower),
                              .required(.special),
                              .minLength(20) ]

let descriptor = rules.map{ "\($0.description);" }
                      .joined(separator: " ")

// "required: upper; required: lower; required: special; max-consecutive: 3;"
```

마음이 내키신다면 `UITextInputPasswordRules` 를 익스텐션으로 만들어서 `PasswordRule` 값의 배열을 변환하는 간편한 이니셜라이저를 제공하게 만들 수도 있습니다.

```swift
extension UITextInputPasswordRules {
    convenience init(rules: [PasswordRule]) {
        let descriptor = rules.map{ $0.description }
                              .joined(separator: "; ")

        self.init(descriptor: descriptor)
    }
}
```

---

여러분이 개인 보안에 관해서 감상적이거나 출신 대학교나 키우는 강아지 또는 좋아하는 스포츠 팀을 비밀번호에 넣는 것을 즐기는 타입이라면, 반드시 그 방식을 바꾸는 것을 추천드립니다.

개인적으로 말씀드리자면 저는 비밀번호 관리기가 하루라도 없는 것을 상상할 수 없습니다.

여러분이 이 글에 나온 예제들을 모두 따라해보셨다면 올해 말에 나올 iOS 12와 macOS Mojave의 Safari에서 발전된 기능을 완벽하게 활용할 수 있을 것입니다.
