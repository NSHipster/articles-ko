---
title: NLLanguageRecognizer
author: Mattt
translator: 김필권
category: Cocoa
tags: language
excerpt: >
    머신 러닝은 애플 플랫폼에서 오랜 시간동안 자연어 처리의 심장이었습니다. 하지만 외부 개발자들이 직접 접근할 수 있게 된 것은 최근의 일입니다.
status:
    swift: 4.2
---

제가 여행할 때 가장 좋아하는 활동 중 하나는 사람들이 말하는 것을 듣고 그들이 어떤 언어를 말하고 있는지 추측하는 것입니다. 지난 몇 년간의 경험으로 저는 꽤 훌륭한 능력을 가지게 된 것 같습니다. (비록 제가 맞는지 확인하는 경우는 드물지만요)

운이 좋으면 저는 단어나 문장을 제가 잘 아는 언어와 비슷하게 알아들을 때도 있습니다. 그렇지 않으면 어떤 종류의 소리인지 듣고 음성 인벤토리를 구축하려고 노력합니다.

예를 들면 치경음을 발음하기 위해서 [`⟨r⟩`](https://en.wikipedia.org/wiki/Dental,_alveolar_and_postalveolar_trills)를 쓸까요? 아니면 [`⟨ɾ⟩`](https://en.wikipedia.org/wiki/Flap_consonant)를 쓸까요? 아니면 [`⟨ɹ⟩`](https://en.wikipedia.org/wiki/Alveolar_and_postalveolar_approximants)를 쓸까요?

모음들은 대부분 열릴까요 닫힐까요? 아니면 앞쪽에서 발음할까요 아니면 뒤쪽에서 발음할까요? 아니면 [`⟨ʇ⟩`](https://en.wikipedia.org/wiki/Dental_clicks)와 같은 비정상적인 소리일까요?

이것은 제가 생각하고 있는 내용입니다. 솔직히 말하자면 이 모든 일들은 언어의 인식 엄무를 위해서 무의식적으로 그리고 자동으로 일어납니다. 그리고 입력에서 출력까지에서 희미한 아이디어를 어떻게 얻는지 정도만 알려줍니다.

컴퓨터 연산도 비슷한 방식으로 작업됩니다. 오랜 기간의 훈련 이후에 머신 러닝 모델은 정형화된 탑다운 접근법에서 이전의 시도를 훨씬 넘어서는 텍스트의 언어 예측이 가능해졌습니다.

머신 러닝은 애플 플랫폼에서 오랜 시간동안 자연어 처리의 심장이었습니다. 하지만 외부 개발자들이 직접 접근할 수 있게 된 것은 최근의 일입니다.

---

새로나올 iOS 12와 macOS 10.4부터 자연어 프레임워크([Natural Language framework](https://developer.apple.com/documentation/naturallanguage))가 기존의 언어 API를 재정의하고 개발자들에게 새로운 기능을 제공합니다.

[`NLTagger`](https://developer.apple.com/documentation/naturallanguage/nltagger)는 [`NSLinguisticTagger`](https://nshipster.com/nslinguistictagger/)에 새로운 사고방식이 접목된 버전입니다.

[`NLTokenizer`](https://developer.apple.com/documentation/naturallanguage/nltokenizer)는 [`enumerateSubstrings(in:options:using:)`](https://developer.apple.com/documentation/foundation/nsstring/1416774-enumeratesubstrings) ([`CFStringTokenizer`](https://developer.apple.com/documentation/corefoundation/cfstringtokenizer-rf8))를 대체하게 될 것입니다.

[`NLLanguageRecognizer`](https://developer.apple.com/documentation/naturallanguage/nllanguagerecognizer)는 `NSLinguisticTagger`에서 `dominantLanguage`를 통해 제공되던 기능의 익스텐션을 제공하고 추가적인 예측과 힌트를 제공할 수 있는 기능도 포함돼있습니다.

## 자연어 텍스트의 언어 인식하기

다음은 `NLLanguageRecognizer`가 자연어 텍스트의 가장 확률이 높은 언어를 추측하는 방법입니다.

```swift
import NaturalLanguage

let string = """
私はガラスを食べられます。それは私を傷つけません。
"""

let recognizer = NLLanguageRecognizer()
recognizer.processString(string)
recognizer.dominantLanguage // ja
```

먼저 `NLLanguageRecognizer` 인스턴스를 만들고 `processString(_:)` 메소드에 스트링을 전달합니다. 여기서 `dominantLanguage` 속성은 예상되는 언어의 BCP-47 언어 태그가 달린 `NLLanguage` 객체를 반환합니다. (예를 들어 일본어(日本語 / Japanese)의 경우엔 `"ja"`를 반환합니다)

### 다중 언어 추측하기

여러분이 대학에서 언어학을 배우셨거나 고등학교때 라틴 클럽에 참여하셨었다면 변증법적인 라틴어와 현대 이탈리아어 사이의 _다중 언어간의 동음 이의어_ 에 대한 예제를 많이 보셨을 것입니다.

다음과 같은 문장으로 예를 들어보겠습니다.

> CANE NERO MAGNA BELLA PERSICA!

| Language | Translation                           |
| -------- | ------------------------------------- |
| Latin    | Sing, o Nero, the great Persian wars! |
| Italian  | The black dog eats a nice peach!      |

[Max Fisher](<https://en.wikipedia.org/wiki/Rushmore_(film)>)에게는 유감이지만 라틴어는 `NLLanguageRecognizer`가 지원하는 언어가 아닙니다. 따라서 이러한 혼란스러운 언어의 예는 거의 재미있지는 않을것입니다.

몇 가지 실험을 해보면 여러분은 `NLLanguageRecognizer`이 부정확한 결과를 내도록 하는 것이 어렵고 심지어 낮은 정확성을 보이는 경우도 적다는 것을 알게 될 것입니다. 하나의 어원을 가진 언어 가족간에 공유되는 것 외에도 각자의 단어도 95%의 확신을 얻게 되는 경우가 많습니다.

몇몇 시도와 에러 이후에 `NLLanguageRecognizer`에 [노르웨이의 부크몰 언어로 된 세계 인권 선언 중 첫 번째 챕터](https://www.ohchr.org/EN/UDHR/Pages/Language.aspx?LangID=nrr)의 꽤 긴 길이의 문자열을 전달하면 부정확한 추측을 하게 된다는 사실을 알아냈습니다.

```swift
let string = """
Alle mennesker er født frie og med samme menneskeverd og menneskerettigheter.
De er utstyrt med fornuft og samvittighet og bør handle mot hverandre i brorskapets ånd.
"""

let languageRecognizer = NLLanguageRecognizer()
languageRecognizer.processString(string)
recognizer.dominantLanguage // da (!)
```

> [세계 인권 선언](http://www.un.org/en/universal-declaration-human-rights/)은 전 세계 500개 이상의 언어로 번역된 가장 널리 알려진 문서입니다.
> 이러한 이유로 이 내용은 자연어 처리에 종종 쓰이곤 합니다.

덴마크어와 노르웨이의 부크몰은 시작하는 부분이 매우 비슷하게 생겼기 때문에 `NLLanguageRecognizer`가 제대로 추측하지 못하는 것은 놀랍지 않은 일입니다. (비교를 위해 같은 내용을 [덴마크어로](https://www.ohchr.org/EN/UDHR/Pages/Language.aspx?LangID=dns) 준비했습니다.)

`dominantLanguage`가 추측하는 언어가 얼마나 자신감 있는지는 `languageHypotheses(withMaximum:)` 메소드를 사용해서 확인할 수 있습니다.

```swift
languageRecognizer.languageHypotheses(withMaximum: 2)
```

| Language                | Confidence |
| ----------------------- | ---------- |
| Danish (`da`)           | 56%        |
| Norwegian Bokmål (`nb`) | 43%        |

글을 쓰고 있는 시점에는 [`languageHints`](https://developer.apple.com/documentation/naturallanguage/nllanguagerecognizer/3017455-languagehints) 속성이 문서화되지 않아서 어떤 방식으로 쓰이는지 확실하지 않습니다. 하지만 가중치가 적힌 사전을 전달한다면 가설을 강화하는 바람직한 효과를 갖는 것으로 보입니다.

```swift
languageRecognizer.languageHints = [.danish: 0.25, .norwegian: 0.75]
```

| Language                | Confidence (with Hints) |
| ----------------------- | ----------------------- |
| Danish (`da`)           | 30%                     |
| Norwegian Bokmål (`nb`) | 70%                     |

<br/>

그렇다면 문자열의 언어를 알면 무엇을 할 수 있을까요?

그래서 생각해볼만한 경우를 준비해왔습니다.

## 철자가 틀린 단어 검사할 때

`NLLanguageRecognizer`와 [`UITextChecker`](https://nshipster.com/uitextchecker/)를 조합하면 아무 문자열의 단어의 철자를 검사할 수 있습니다.

`NLLanguageRecognizer`를 만들고 `processString(_:)` 메소드로 초기화하는 것으로 시작합니다.

```swift
let string = """
Wenn ist das Nunstück git und Slotermeyer?
Ja! Beiherhund das Oder die Flipperwaldt gersput!
"""

let languageRecognizer = NLLanguageRecognizer()
languageRecognizer.processString(string)
let dominantLanguage = languageRecognizer.dominantLanguage! // de
```

그 다음엔 `rangeOfMisspelledWord(in:range:startingAt:wrap:language:)`의 파라미터인 `language`의 속성인 `dominantLanguage`에서 반환되는 `NLLanguage` 객체의 `rawValue`를 전달합니다.

```swift
let textChecker = UITextChecker()

let nsString = NSString(string: string)
let stringRange = NSRange(location: 0, length: nsString.length)
var offset = 0

repeat {
    let wordRange =
            textChecker.rangeOfMisspelledWord(in: string,
                                              range: stringRange,
                                              startingAt: offset,
                                              wrap: false,
                                              language: dominantLanguage.rawValue)
    guard wordRange.location != NSNotFound else {
        break
    }

    print(nsString.substring(with: wordRange))

    offset = wordRange.upperBound
} while true
```

[세상에서 가장 웃긴 농담](https://en.wikipedia.org/wiki/The_Funniest_Joke_in_the_World)을 전달해보면 다음의 언어들이 철자가 잘못됐다고 나올 것입니다.

- Nunstück
- Slotermeyer
- Beiherhund
- Flipperwaldt
- gersput

## 연설 합치기

`NLLanguageRecognizer`와 [`AVSpeechSynthesizer`](https://nshipster.com/avspeechsynthesizer/)를 콘서트에 사용하면 자연어 텍스트를 크게 들을 수도 있습니다.

```swift
let string = """
Je m'baladais sur l'avenue le cœur ouvert à l'inconnu
    J'avais envie de dire bonjour à n'importe qui.
N'importe qui et ce fut toi, je t'ai dit n'importe quoi
    Il suffisait de te parler, pour t'apprivoiser.
"""

let languageRecognizer = NLLanguageRecognizer()
languageRecognizer.processString(string)
let language = languageRecognizer.dominantLanguage!.rawValue // fr

let speechSynthesizer = AVSpeechSynthesizer()
let utterance = AVSpeechUtterance(string: string)
utterance.voice = AVSpeechSynthesisVoice(language: language)
speechSynthesizer.speak(utterance)
```

[Joe Dassin](https://itunes.apple.com/us/album/les-champs-%C3%A9lys%C3%A9es/311331439?i=311331447)의 서정적인 기교를 가지고 있지는 않지만 _인생도 그렇지 않습니까_.

---

이해되기 위해서는 우리가 먼저 이해해야 합니다. 그리고 자연어를 이해하기 위한 첫 번째 단계는 언어를 결정하는 것입니다.

`NLLanguageRecognizer`는 iOS와 macOS 전반에 걸친 지능형 특징을 담당하는 기능에 대한 강력한 새로운 인터페이스를 제공합니다. 사용자에 대한 새로운 이해를 얻기 위해 `NLLanguageRecognizer`을 여러분의 앱에 적용해보세요.
