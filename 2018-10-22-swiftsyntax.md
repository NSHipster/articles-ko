---
title: SwiftSyntax
author: Mattt
category: Swift
excerpt: >
  SwiftSyntax는 Swift 소스 코드를 파싱하고 분석하고 변형할 수 있게 해주는 Swift 라이브러리입니다.
  SwiftSynax를 사용해서 코드 형식기와 문법 하이라이터를 만들어봅시다.
status:
  swift: 4.2
---

[SwiftSyntax](https://github.com/apple/swift-syntax)는 Swift 소스 코드를 파싱하고 분석하고 변형할 수 있게 해주는 Swift 라이브러리입니다.
이것은 [libSyntax](https://github.com/apple/swift/tree/master/lib/Syntax) 라이브러리를 기반으로 하고 있고 [2017년 8월](https://github.com/apple/swift-syntax/commit/909d336aefacdcbdd45ec6130471644c1ae929f5)에 메인 Swift 언어 저장소로 빠져나왔습니다.

이 프로젝트의 목표는 [이처럼](https://github.com/apple/swift/blob/master/lib/Syntax/README.md#swift-syntax-and-structured-editing-library) <dfn>구조적 편집(structured editing)</dfn>을 위한 안정하고 정확하며 직관적인 도구를 제공하는 것입니다.

> 구조적 편집(structured editing)이란 무엇일까요?
> 그것은 소스 코드가 _표현_ 하는 내용이 아닌 _구조_ 를 정확히 아는 편집 전략입니다.
> 이는 identifier 바꿔치기, 전역 함수 호출을 메소드 호출로 변경하기 또는 모든 소스 파일을 규칙에 따라 인덴트 또는 포맷을 변경하기 등 다양한 방식으로 이룰 수 있습니다.

글을 쓰고 있는 지금은 SwiftSynax가 여전히 개발 중인 상태이고 API 변경이 아직 있습니다.
그러나 Swift 소스 코드에 프로그래밍적으로 오늘부터라도 사용할 수 있습니다.

지금은 [Swift Migrator](https://github.com/apple/swift/tree/master/lib/Migrator)에서 사용하고 있으며 도구를 적응시키기 위해 내부적으로나 외부적으로도 지속적인 노력이 있습니다.

## 어떻게 작동하나요?

SwiftSynax가 어떻게 작동하는지 이해하기 위해서는 뒤로 한 걸음 물러서서 Swift 컴파일러 구조를 봐야합니다.

{% asset swift-compilation-diagram.png %}

Swift 컴파일러는 Swift 코드를 실행 가능한 기계어로 만드는 일을 주로 합니다.
과정은 몇몇의 분리된 단계로 나눠지는데, 추상적인 문법 트리(AST)를 생성하는 [parser](https://github.com/apple/swift/tree/master/lib/Parse)로 시작합니다.
그 다음엔 의미적인 분석이 실행되어 타입 검사된 AST를 생성합니다. 이는 [Swift Intermediate Language(SIL)](https://github.com/apple/swift/blob/master/docs/SIL.rst)라는 더 낮은 단계의 언어가 됩니다. SIL은 변형되고 최적화되어 더 낮은 단계인 궁극적인 기계어로 컴파일된 [LLVM IR](http://llvm.org/docs/LangRef.html)가 됩니다.

이 논의에서 가장 중요한 점은 SwiftSynax가 컴파일 과정의 첫 번째 단계에서 생성된 AST에서 작동한다는 것입니다.
그것은 코드에 대한 의미적이거나 타입 정보를 알려줄 수 없다는 뜻입니다.

Swift 코드에 대한 상대적으로 더 완벽한 이해를 보여주는 [SourceKit](https://github.com/apple/swift/tree/master/tools/SourceKit)과는 대조됩니다.
이 추가적인 정보는 자동 완성이나 파일간 네비게이팅같은 기능을 구현하려고 할 땐 도움이 많이 됩니다.
하지만 코드 포맷팅이나 문법 하이라이팅처럼 순수하게 문법적인 단계에도 만족하는 사례들이 많이 있습니다.

### AST 쉽게 이해하기

추상 문법 트리(Abstract Syntax Tree)는 추상적으로 생각하면 어려울 수 있습니다.
그러니 직접 만들어보고 어떻게 생겼는지 확인해보겠습니다.

`1` 을 반환하는 함수 `one()` 을 정의하는 한 줄 짜리 Swift 파일이 있다고 해보겠습니다.

```swift
func one() -> Int { return 1 }
```

이 파일에서 `swiftc` 커맨드를 실행하고 거기에 `-frontend -emit-syntax` 인자를 넘겨보겠습니다.

```terminal
$ xcrun swiftc -frontend -emit-syntax ./One.swift
```

결과로 나오는 JSON 덩어리는 AST를 나타냅니다.
JSON의 포맷팅을 다시 하면 구조가 훨씬 깔끔해집니다.

<!-- ```json
{
    "kind": "SourceFile",
    "layout": [{
        "kind": "CodeBlockItemList",
        "layout": [{
          "kind": "CodeBlockItem",
          "layout": [{
              "kind": "FunctionDecl",
              "layout": [null, null, {
                  "tokenKind": {
                      "kind": "kw_func"
                  },
                  "leadingTrivia": [],
                  "trailingTrivia": [{
                      "kind": "Space",
                      "value": 1
                  }],
                  "presence": "Present"
              }, {
                  "tokenKind": {
                      "kind": "identifier",
                      "text": "one"
                  },
                  "leadingTrivia": [],
                  "trailingTrivia": [],
                  "presence": "Present"
              }, ...
``` -->

{% info %}

Python의 `json.tool` 모듈은 JSON을 포맷하는 더 편한 방식을 제공합니다.
이 모듈은 보통의 macOS라면 표준으로 들어있습니다.
사용하는 방법은 다음과 같습니다.

```terminal
$ xcrun swiftc -frontend -emit-syntax ./One.swift | python -m json.tool
```

{% endinfo %}

가장 상위 레벨에서 우리는 `SourceFile` 이 `CodeBlockItemList` 요소로 이루어져 있고 그 안에는 `CodeBlockItem` 파트가 있다는 것을 확인할 수 있습니다.
이 예제는 하나의 함수 정의(`FunctionDecl`)을 위한 `CodeBlockItem` 이 하나만 존재합니다. 이 함수 정의에는 함수 고유 정보, 파라미터 절 그리고 반환 절까지 포함돼 있습니다.

<dfn>trivia</dfn>라는 단어는 문법적으로 의미있지 않은 공백같은 것 무엇이든 설명하는데에 사용됩니다.
각 토큰은 하나 이상의 왼쪽(leading) 또는 오른쪽(trailing) trivia를 가지고 있습니다.
예를 들어 반환 절(`-> Int`)의 `Int` 다음에 있는 공백은 다음과 같은 trailing trivia로 나타낼 수 있습니다.

```json
{
  "kind": "Space",
  "value": 1
}
```

### 파일 시스템 제약

SwiftSynax는 추상 문법 트리를 `swiftc` 를 호출하는 시스템에 델리게이팅을 통해 생성합니다.
그러나 이를 위해서 처리할 코드를 파일과 연관시켜야하며 이는 코드로 문자열을 처리하는 작업에 유용합니다.

이 제약을 통과하기 위한 한 가지 방법은 임시 파일에 코드를 작성하고 컴파일러에 넘기는 것입니다.

예전에 [임시 파일을 작성하는 방법에 대한 글](https://nshipster.com/nstemporarydirectory/)도 썼었습니다. 하지만 요즘엔 더 나은 API가 [Swift Package Manager](https://github.com/apple/swift-package-manager)에 의해 제공됩니다.
`Package.swift` 파일에 다음과 같은 내용을 추가하고 `"Utility"` 디펜던시를 적절한 타겟에 추가하세요.

```swift
.package(url: "https://github.com/apple/swift-package-manager.git", from: "0.3.0"),
```

이제 `Base` 모듈을 추가하고 `TemporaryFile` API를 다음과 같이 사용하면 됩니다.

```swift
import Basic
import Foundation

let code: String

let tempfile = try TemporaryFile(deleteOnClose: true)
defer { tempfile.fileHandle.closeFile() }
tempfile.fileHandle.write(code.data(using: .utf8)!)

let url = URL(fileURLWithPath: tempfile.path.asString)
let sourceFile = try SyntaxTreeParser.parse(url)
```

## 이걸로 무엇을 할 수 있을까요?

이제 SwiftSynax가 어떻게 작동하는지에 대해 알았으니 어디에 사용할지에 대해 얘기해봅시다!

### Swift 코드 작성을 더 어렵게 만들기

SwiftSynax로 할 수 있는 가장 첫 번째 사례는 Swift 코드 작성을 더 어렵게 만드는 것입니다.

SwiftSynax의 `SyntaxFactory` API는 완전 새로운 Swift 코드를 생성할 수 있게 해줍니다.
불행히도 이를 프로그래밍적으로 하는 것은 공원을 걷는 것만큼 쉽지는 않습니다.

다음의 코드를 예로 들겠습니다.

```swift
import SwiftSyntax

let structKeyword = SyntaxFactory.makeStructKeyword(trailingTrivia: .spaces(1))

let identifier = SyntaxFactory.makeIdentifier("Example", trailingTrivia: .spaces(1))

let leftBrace = SyntaxFactory.makeLeftBraceToken()
let rightBrace = SyntaxFactory.makeRightBraceToken(leadingTrivia: .newlines(1))
let members = MemberDeclBlockSyntax { builder in
    builder.useLeftBrace(leftBrace)
    builder.useRightBrace(rightBrace)
}

let structureDeclaration = StructDeclSyntax { builder in
    builder.useStructKeyword(structKeyword)
    builder.useIdentifier(identifier)
    builder.useMembers(members)
}

print(structureDeclaration)
```

_휘유_
그래서 위 코드의 결과는 뭔가요?

```swift
struct Example {
}
```

_엄 청 나 네 요_

이 라이브러리가 [GYB](https://nshipster.com/swift-gyb/)의 모든 자리를 대체할 수 있지는 않을 것입니다. (사실 [libSyntax](https://github.com/apple/swift/blob/master/lib/Syntax/SyntaxKind.cpp.gyb)와 [SwiftSyntax](https://github.com/apple/swift-syntax/blob/master/Sources/SwiftSyntax/SyntaxKind.swift.gyb) 둘 모두 `gyb` 라는 GYB를 사용하기 위한 확장 인터페이스를 제공합니다.)

이 인터페이스는 정확도가 중요한 경우에 매우 유용할 수 있습니다.
예를 들어 Swift 컴파일러를 위한 [fuzzer](https://en.wikipedia.org/wiki/Fuzzing)를 구현하려고 SwiftSynax를 사용한다면 내부적인 스트레스 테스트를 위해 임의로 복잡하지만 표면적으로는 유효한 프로그램을 만들어야 할 것입니다.

## Swift 코드 다시 작성하기

[SwiftSynax의 README에 있는 예제](https://github.com/apple/swift-syntax#example)는 소스 파일의 정수 리터럴을 찾고 각 값을 하나 증가시키는 방법에 대해 보여줍니다.

그것을 보셨다면 여러분은 이미 표준 `swift-format` 도구를 만드는데에 SwiftSynax가 어떻게 사용되는지 추측하셨을 것입니다.

그러면 생산성이 낮은(그리고 시즌으로는 더 적절한 🎃) 소스 재작성은 어떤건지 알아볼까요?

```swift
import SwiftSyntax

public class ZalgoRewriter: SyntaxRewriter {
    public override func visit(_ token: TokenSyntax) -> Syntax {
        guard case let .stringLiteral(text) = token.tokenKind else {
            return token
        }

        return token.withKind(.stringLiteral(zalgo(text)))
    }
}
```

그래서 [`zalgo`](https://gist.github.com/mattt/b46ab5027f1ee6ab1a45583a41240033) 함수는 뭘까요?
몰라도 괜찮습니다...

아무튼 위 코드는 여러분 소스의 스트링 리터럴을 모두 다음과 같이 바꿔버릴 것입니다.

```swift
// Before 👋😄
print("Hello, world!")

// After 🦑😵
print("H͞͏̟̂ͩel̵ͬ͆͜ĺ͎̪̣͠ơ̡̼͓̋͝, w͎̽̇ͪ͢ǒ̩͔̲̕͝r̷̡̠͓̉͂l̘̳̆ͯ̊d!")
```

_무섭네요 그렇죠?_

## Swift 코드 하이라이트하기

SwiftSynax로 실제로 쓰일만한 Swift 문법 하이라이터를 만들어보겠습니다.

<dfn>syntax highlighter</dfn>는 소스 코드를 HTML 형식으로 표시하는 것에 더 적합한 방식으로 포맷을 지정하는 도구를 설명합니다.

[NSHipster는 Jekyll 위에 만들어졌습니다](https://github.com/NSHipster/nshipster.com). 그리고 Ruby 라이브러리인 [Rouge](https://github.com/jneen/rouge)를 사용해서 모든 글에 있는 예시 코드들을 색칠합니다.
그러나 Swift의 상대적으로 복잡한 문법과 빠른 성장으로 인해 생성된 HTML은 언제나 100% 옳지는 않습니다.

[정규식을 덕지덕지 붙이는 것](https://github.com/jneen/rouge/blob/master/lib/rouge/lexers/swift.rb) 대신에 우리는 [문법 하이라이터를 만들어서](https://github.com/NSHipster/SwiftSyntaxHighlighter) SwiftSynax가 언어를 이해하고 있는 이점을 사용해보겠습니다.

구현은 다소 간단합니다. `SyntaxRewriter` 의 서브클래스를 구현하고 각 토큰에 대해 호출되는 `visit(_:)` 메소드를 덮어씁니다.
각기 다른 종류의 토큰을 전환함으로써 우리는 [HTML 마크업의 하이라이터 토큰](https://github.com/jneen/rouge/wiki/List-of-tokens)으로 매핑할 수 있게 됩니다.

예를 들어 숫자 리터럴은 `<span>` 요소로 나타내고 클래스 이름을 `m` 으로 시작하는 값으로 설정합니다. (`mf` 는 실수, `mi` 는 정수 등)
다음은 `SyntaxRewriter` 서브클래스의 코드입니다.

```swift
import SwiftSyntax

class SwiftSyntaxHighlighter: SyntaxRewriter {
    var html: String = ""

    override func visit(_ token: TokenSyntax) -> Syntax {
        switch token.tokenKind {
        // ...
        case .floatingLiteral(let string):
            html += "<span class=\"mf\">\(string)</span>"
        case .integerLiteral(let string):
            if string.hasPrefix("0b") {
                html += "<span class=\"mb\">\(string)</span>"
            } else if string.hasPrefix("0o") {
                html += "<span class=\"mo\">\(string)</span>"
            } else if string.hasPrefix("0x") {
                html += "<span class=\"mh\">\(string)</span>"
            } else {
                html += "<span class=\"mi\">\(string)</span>"
            }
        // ...
        default:
            break
        }

        return token
    }
}
```

`SyntaxRewriter` 가 다양한 종류의 문법 요소에 접근하는 `visit(_:)` 에 특화돼 있습니다. 저는 이 각 요소 종류들에 접근하는 방버으로 `switch` 문을 사용하는 것이 쉬운 방법인 것을 알게되었습니다.
(`default` 에 아직 작업되지 않은 토큰을 출력하는 것은 정말 도움됐습니다)
이게 가장 우아한 구현 방식은 아니지만 라이브러리에 대한 제한적인 이해도에서 빠르게 시작할 수 있는 간편한 방법이라고 생각합니다.

아무튼 몇 시간의 개발을 한 이후에 저는 다양한 색상을 가진 코드를 생성할 수 있게 되었습니다.

{% asset swiftsyntaxhightlighter-example-output.png width=500 %}

이 프로젝트는 하나의 라이브러리와 커맨드 라인 툴로만 작업했습니다.
더 자세한 내용은 [링크](https://github.com/NSHipster/SwiftSyntaxHighlighter)에서 확인이 가능합니다.
여러분이 어떻게 생각하고 있는지 저에게 알려주세요!
