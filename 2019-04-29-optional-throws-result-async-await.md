---
title: Optional, throws, Result, 그리고 async/await
author: jemmons
translator: 김필권
category: Swift
excerpt: "스위프트 에러 핸들링의 과거, 현재, 미래."
status:
  swift: 5.0
---

Swift 1 시절엔 에러 핸들링을 할 수 있는 방법이 그렇게 많지 않았습니다.
하지만 `Optional` 은 그때도 있었죠.
게다가 이 개념은 정말 멋졌습니다!
널(null) 확인을 명시적으로 만듦으로써, 함수에서 `nil` 을 반환하는 행동이 덜 꺼려지고 언어의 기능처럼 느껴지게 되었습니다.

다음은 Keychain 데이터를 가져올 때 어떠한 에러든 `nil` 을 반환하는 코드입니다. 어렵지 않게 만나볼 수 있는 코드죠.

```swift
func keychainData(service: String) -> Data? {
  let query: NSDictionary = [
    kSecClass: kSecClassGenericPassword,
    kSecAttrService: service,
    kSecReturnData: true
  ]
  var ref: CFTypeRef? = nil

  switch SecItemCopyMatching(query, &ref) {
  case errSecSuccess:
    return ref as? Data
  default:
    return nil
  }
}
```

이 코드에선 쿼리를 설정했고,
`SecItemCopyMatching` 에 빈 `inout` 참조를 넘겼으며,
반환된 스테이터스 코드에 기반하여,
제대로된 참조를 데이터로 반환하거나 에러를 `nil` 로 반환했습니다.

그리고 호출하는 곳에서 옵셔널을 벗겨낸 결과를 알려주면 됩니다.

```swift
if let myData = keychainData(service: "My Service") {
  <#do something with myData...#>
} else {
  fatalError("Something went wrong with... something?")
}
```

## Getting Results

결과가 쌍으로 나오는 경우엔 위에서 설명드린 것처럼 우아하게 해결할 수 있지만 치명적인 단점이 있습니다.
[옵셔널 코드](https://github.com/apple/swift/blob/swift-5.0-RELEASE/stdlib/public/core/Optional.swift#L122)를 살펴보겠습니다.
`Optional` 은 그저 어떤 값을 감싸고 있거나 아무것도 없는 것의 enum입니다.

```swift
enum Optional<Wrapped> {
  case some(Wrapped)
  case none
}
```

모든 일이 잘 풀리는 경우엔 우리가 만든 도구에서도 그저 그에 맞는 값을 반환하면 됩니다.
하지만 I/O 작업에서는 여러가지 이유로 일이 틀어질 수 있는데, `Optional` 은 일이 틀어졌다는 것만 알려줄 수 있습니다.
이 경우에 우리가 할 수 있는 것은 그저 `.none` 을 반환하는 것입니다.
(예를 들자면, `SecItemCopyMatching` 는 [아주 아주 다양한 방법](https://developer.apple.com/documentation/security/1542001-security_framework_result_codes)으로 일이 틀어질 수 있습니다.)

그러면 호출하는 부분은 일이 틀어진 이유를 알고 싶어도 그저 빈 `.none` 값만 받게 될 것입니다.
모든 최악의 경우가 `¯\_(ツ)_/¯` 같은 하나의 이유로 합쳐지면 강력한 소프트웨어를 만들기는 점점 어려워집니다.
이러한 상황은 어떻게 해결할 수 있을까요?

한 가지 방법은 언어단에서 제공하는 기능인 함수가 에러와 반환 값까지 발생하도록(throws) 하는 것입니다.
그리고 이 방법이 바로 Swift 2에서 추가된 `throws/throw` 와 `do/catch` 문법입니다.

그 방법말고 `Optional` 의 증명에 대해 조금 더 생각해봅시다. `Optional` 은 값 또는 `nil` 을 가지는데 이 `nil` 의 에러 표현력이 부족한 것이 문제라면, 해당 이슈가 발생하는 이유까지 들어있는 에러와 기존과 똑같은 값을 가지는 새로운 `Optional` 은 어떨까요?

바로 그겁니다!
그 새로운 `Optional` 은 새로운 이름을 받아서 `Result` 타입으로 세상에 나왔습니다.
게다가 [Swift 5 표준 라이브러리](https://github.com/apple/swift/blob/swift-5.0-RELEASE/stdlib/public/core/Result.swift#L16)에서도 사용이 가능합니다!

```swift
enum Result<Success, Failure: Error> {
  case success(Success)
  case failure(Failure)
}
```

`Result` 는 성공한 경우엔 값을, 실패한 경우엔 에러를 가지는 타입입니다.
그렇다면 위에서 만들었던 키체인 코드를 조금 더 발전시켜보도록 하겠습니다.

먼저 커스텀 `Error` 타입을 정의해서 `nil` 을 풍부하게 표현할 수 있도록 만듭니다.

```swift
enum KeychainError: Error {
  case notData
  case notFound(name: String)
  case ioWentBad
  <#...#>
}
```

다음은 `keychainData` 의 정의에 있는 `Data?` 대신에 `Result<Data, Error>` 를 사용합니다.
일이 틀어지지 않고 똑바로 실행된다면 `.success` 의 [associated value](https://docs.swift.org/swift-book/LanguageGuide/Enumerations.html#ID148)를 받게 될 것입니다.
`SecItemCopyMatching` 의 많고 다양한 재앙이 들어닥친다면 어떨까요?
이젠 `nil` 을 반환하는 것 대신에 개별 에러들이 `.failure` 에 감싸져서 나오는 것을 보게 될 것입니다.

```swift
func keychainData(service: String) -> Result<Data, Error> {
  let query: NSDictionary = [...]
  var ref: CFTypeRef? = nil

  switch SecItemCopyMatching(query, &ref) {
  case errSecSuccess:
    guard let data = ref as? Data else {
      return .failure(KeychainError.notData)
    }
    return .success(data)
  case errSecItemNotFound:
    return .failure(KeychainError.notFound(name: service))
  case errSecIO:
    return .failure(KeychainError.ioWentBad)
  <#...#>
  }
}

```

이제 호출부분에서 얻을 수 있는 정보가 많아졌네요!
그러면 `switch` 문을 반환된 값에 사용해서 성공과 실패 모두 개별적으로 다뤄보도록 하겠습니다.

```swift
switch keychainData(service: "My Service") {
case .success(let data):
  <#do something with data...#>
case .failure(KeychainError.notFound(let name)):
  print(""\(name)" not found in keychain.")
case .failure(KeychainError.io):
  print("Error reading from the keychain.")
case .failure(KeychainError.notData):
  print("Keychain is broken.")
<#...#>
}
```

모든 경우를 다룰 수 있다는 점에서 `Result` 는 `Optional` 의 상위 호환이라고도 할 수 있습니다. 어떻게 이 기능이 표준 라이브러리에 들어가는데 5년이나 걸렸을까요?

## Three's a Crowd

아쉽게도 `Result` _또한_ 저주를 받아 치명적인 단점을 가지고 있습니다. 지금까지는 하나의 함수에 하나의 호출만 있어서 나오지 않았습니다. `Result` 를 반환하는 함수를 두 개 더 만들어보겠습니다.

```swift
func makeAvatar(from user: Data) -> Result<UIImage, Error> {
  <#Return avatar made from user's initials...#>
  <#or return failure...#>
}

func save(image: UIImage) -> Result<Void, Error> {
  <#Save image and return success...#>
  <#or returns failure...#>
}
```

{% info %}
`save(image:)` 의 반환 타입 중 성공한 타입이 `Void` 인 것을 기억하세요.
생각해보면 당연하지만 성공할 때마다 특정한 값을 반환해야할 필요는 없습니다.
그저 성공 여부만 아는 것이 충분할 때도 있거든요.
{% endinfo %}

위 예제를 보면, 첫 번째 함수는 사용자 데이터를 바탕으로 아바타를 만들어주는 것이고, 두 번째 함수는 디스크에 이미지를 저장하는 것입니다.
구현 방식은 우리의 목적에 그렇게 문제가 되지 않습니다. 그저 그들이 `Result` 타입을 반환한다는 것만 알면됩니다.

이제는 키체인에서 가져온 사용자 데이터를 바탕으로  아바타를 만들고 그 아바타를 디스크에 저장하는 작업을 해보겠습니다. 물론 그 과정에서 발생하는 에러들도 모두 처리해야겠죠?

다음과 같이 시도해 볼 수 있겠습니다.

```swift
switch keychainData(service: "UserData") {
case .success(let userData):

  switch makeAvatar(from: userData) {
  case .success(let avatar):

    switch save(image: avatar) {
    case .success:
      break

    case .failure(FileSystemError.readOnly):
      print("Can't write to disk.")

    <#...#>
    }

  case .failure(AvatarError.invalidUserFormat):
    print("Unable to generate avatar from given user.")

  <#...#>
  }

case .failure(KeychainError.notFound(let name)):
  print(""\(name)" not found in keychain.")

<#...#>
}
```

이것 보세요. 두 함수를 추가했을 뿐인데 중첩문이 엄청나게 많아졌습니다. 이런 경우엔 에러 핸들링이 슬프고 복잡해집니다

## Falling Flat

다행히도 `Optional` 과 같이 [`Result` 내부에 `flatMap` 이 구현](https://github.com/apple/swift/blob/swift-5.0-RELEASE/stdlib/public/core/Result.swift#L96)되어 있기 때문에 해결이 가능합니다.
`Result` 에 `flatMap` 이 적용되면 `.success` 의 경우엔 주어진 값을 합쳐서 새로운 `Result` 를 반환합니다. 그러나 `.failure` 의 경우엔 `flatMap` 이 수정없이 `.failure` 을 반환합니다.

{% info %}
`flatMap` 과 같은 구현들을 때때로 [모나드](http://chris.eidhof.nl/post/monads-in-swift/)라고 부릅니다.
이것은 모나드가 모두 공통된 속성을 공유한다는 것을 아는데 도움을 줍니다.
단어 자체가 친숙하지 않고 무서울 수 있지만 괜찮습니다.
오늘 이해해야 할 것은 `flatMap` 단 하나니까요.
{% endinfo %}
{% warning %}
Swift 4.1에서 `flatMap` 이 `compactMap` 으로 대체되었다는 오해가 있습니다.
그렇지 않습니다!
[`Sequence` 에서 `Optional` 요소를 `flatMap` 하는 것처럼 특별한 케이스만 삭제되었습니다](https://github.com/apple/swift-evolution/blob/master/proposals/0187-introduce-filtermap.md#motivation).
{% endwarning %}

이러한 규칙을 지키며 에러를 보내기 때문에 각 단계의 `.failure` 를 생각하지 않고 `flatMap` 을 사용해서 연산을 합성할 수 있습니다. 이러한 방식은 우리의 코드에 중첩을 최소화해주고 에러 핸들링과 연산을 분리할 수 있게 해줍니다.

```swift
let result = keychainData(service: "UserData")
             .flatMap(makeAvatar)
             .flatMap(save)

switch result {
case .success:
  break

case .failure(KeychainError.notFound(let name)):
  print(""\(name)" not found in keychain.")
case .failure(AvatarError.invalidUserFormat):
  print("Unable to generate avatar from given user.")
case .failure(FileSystemError.readOnly):
  print("Can't write to disk.")
<#...#>
}
```

코드 품질이 향상됐다는 것엔 의심의 여지가 없네요. 하지만 이는 코드를 읽는 사람이 `.flatMap` 를 알아야해서 덜 직관적이게 될 것입니다.

{% warning %}
지금까지의 내용은 완벽한 결합성(Composability)의 최적화된 시나리오입니다. (첫 연산의 출력이 두 번째 연산의 입력이 되고, 두 번째 연산의 출력이 세 번째 연산의 입력이 되는 것처럼 이어지는 속성)
만약 연산자가 아무것도 받지 않는다면 어떨까요?
아니면 하나 이상의 값을 필수로 받는다면 어떨까요?
또는 각 연산이 입력으로 받는 값과 출력으로 내보내는 값들이 서로 타입이 다른 경우엔 어떨까요?
우아하지 않을 수 있지만 이런 경우에 `flatMap` 을 하면 해결이 가능합니다.
{% endwarning %}

비교를 위해 앞서 짠 코드를 Swift 2에 추가된 `do/catch` 방식으로 작성해보겠습니다.

```swift
do {
  let userData = try keychainData(service: "UserData")
  let avatar = try makeAvatar(from: userData)
  try save(image: avatar)

} catch KeychainError.notFound(let name) {
  print(""\(name)" not found in keychain.")

} catch AvatarError.invalidUserFormat {
  print("Not enough memory to create avatar.")

} catch FileSystemError.readOnly {
  print("Could not save avatar to read-only media.")
} <#...#>
```

첫 번째로 생각해야 할 내용은 두 코드가 얼마나 비슷한지 살펴보는 것입니다. 둘 다 위쪽에 연산하는 코드가 있고 아래쪽에 에러 핸들링 하는 부분이 있는 것이 비슷하네요.

{% info %}
두 코드가 비슷하게 생긴 것은 우연이 아닙니다. Swift의 에러 핸들링의 대부분이 [`Result` 와 비슷한 타입을 반환하거나 벗겨내는 일을 좀 더 쉽게 해주는 것](https://twitter.com/jckarter/status/608137115545669632)이었습니다.
계속 살펴보시죠...
{% endinfo %}

`Result` 버전은 `flatMap` 을 통해서 연산자들끼리 연결이 돼있는 반면에 `do/catch` 코드는 에러 핸들링이 일어나도록 하기 위해 `try` 를 계속 사용했습니다.
`Result` 버전은 내부 열거형을 알아야 하고 그를 통해 `switch` 문을 사용해야 했지만 `do/catch` 버전은 에러 자체에 더 집중할 수 있도록 해줍니다.

새로 생긴 에러 핸들링 문법은 `Result` 와 관련된 복잡한 내용들을 효과적이게 없애긴하지만 이를 사용하려면 열거형, associated value, 제네릭, flatMap, 모나드 등을 학습해야 합니다...
Swift 2에 추가된 에러 핸들링 방법은 `Result` 와 그외들을 학습하지 않아도 잘 사용할 수 있습니다.

지난 5년간 그것들을 다 익히셨다면 추가하지 않을 이유가 없겠죠?

## Error's Ups and Downs

다른 방법들도 그랬듯이 `do/catch` 방식에도 치명적인 단점이 존재합니다...

`throw` 는 마치 한 방향으로만 작동하는 `return` 같습니다. 우리는 호출한 곳에다가 에러를 보내는 것(up)만 할 수 있고 다른 함수에 에러를 파라미터로 보내는 것(down)은 할 수 없습니다.

이러한 "up"-only 방식은 일반적으로 우리가 원하는 내용입니다.
앞써 작성한 키체인 도구의 에러 핸들링을 살펴보면 데이터 혹은 에러를 보내기 위해서 `return` 과 `throw` 들이 난무하는 것을 볼 수 있습니다.

```swift
func keychainData(service: String) throws -> Data {
  let query: NSDictionary = [...]
  var ref: CFTypeRef? = nil

  switch SecItemCopyMatching(query, &ref) {
  case errSecSuccess:
    guard let data = ref as? Data else {
      throw KeychainError.notData
    }
    return data
  case errSecItemNotFound:
    throw KeychainError.notFound(name: service)
  case errSecIO:
    throw KeychainError.ioWentBad
  <#...#>
  }
}
```

하지만 만약 키체인에서 사용자 데이터를 가져오는 대신에 클라우드 서비스에서 가져온다면 어떨까요?
빠르고 믿을 수 있는 연결이 제공된다 해도 네트워크 상의 데이터 로딩은 디스크에서 읽어오는 것과 비교한다면 느릴 수 밖에 없습니다.
당연히 데이터가 오는 동안 앱이 멈추는 것은 원하지 않으시겠죠?
그러니 비동기로 만들어보겠습니다.

하지만 비동기로 만든다면 더 이상 호출한 곳으로 반환하는 것(up)을 하지 않을 것이라는 뜻입니다.
대신에 완료 시에 클로저를 다음으로(down) 호출할 것입니다.

```swift
func userData(for userID: String, completion: (Data) -> Void) {
  <#get data from the network#>
  // 시간이 지난 후 실행
  completion(myData)
}
```

오늘날의 네트워크 연산은 [다양한 종류의 에러](https://developer.apple.com/documentation/foundation/urlerror)로 인해 실패할 수 있습니다.
하지만 이 모든 에러를 `completion` 에 `throw` 로 넘길 수는 없습니다.
그러니 가장 좋은 선택지는 에러를 옵셔널 파라미터로 넘기는 것입니다.

```swift
func userData(for userID: String, completion: (Data?, Error?) -> Void) {
  <# Fetch data over the network... #>
  guard myError == nil else {
    completion(nil, myError)
  }
  completion(myData, nil)
}
```

하지만 이렇게 되면 호출하는 부분에서 우리가 예상했던 시나리오 뿐만 아니라 실행 불가능하다고 생각되는 시나리오에 대해서도 설명해야 합니다.

```swift
userData(for: "jemmons") { maybeData, maybeError in
  switch (maybeData, maybeError) {
  case let (data?, nil):
    <#do something with data...#>
  case (nil, URLError.timedOut?):
    print("Connection timed out.")
  case (nil, nil):
    fatalError("🤔Hmm. This should never happen.")
  case (_?, _?):
    fatalError("😱What would this even mean?")
  <#...#>
  }
}
```

원래 같았으면 "데이터 또는 nil"과 "에러 또는 nil"가 뒤섞인 것을 보는 대신에 그저 데이터 또는 에러일 경우만 관리할 수 있어서 도움이 많이 됩니다.

## Stop Me If You've Heard This One…

데이터 또는 에러요?
되게 친숙한 구도네요.
그렇다면 `Result` 를 써보는 것은 어떨까요?

```swift
func userData(for userID: String, completion: (Result<Data, Error>) -> Void) {
  // 모든 것이 잘 풀렸을 경우:
  completion(.success(myData))

  // 일이 잘 안 풀린 경우:
  completion(.failure(myError))
}
```

호출 부분은 다음처럼 될 것입니다

```swift
userData(for: "jemmons") { result in
  switch (result) {
  case (.success(let data)):
    <#do something with data...#>
  case (.failure(URLError.timedOut)):
    print("Connection timed out.")
  <#...#>
}
```

와!
코드에서 볼 수 있듯이 `Result` 타입은 "함수가 `throws` 로 표시될 때 리턴되는 것"에 대한 Swift의 추상적인 아이디어를 구체화할 때 사용됩니다.
우리는 이를 컴플리션 핸들러의 파라미터를 구체적인 타입으로 받는 비동기 연산을 다룰 때 사용할 수 있을 것입니다

{% info %}
추상적인 에러 핸들링과 구체적인 `Result` 사이의 이중성은 동전의 양 면처럼 서로 바꾸는 것이 아주 사소합니다.

```swift
Result { try somethingThatThrows() }
```

이 코드는 catch할 수 있는 추상적인 무언가를 어딘가에 넘길 수 있는 구체적인 result 타입으로 변경합니다.

```swift
try someResult.get()
```

이 코드는 구체적인 result 타입을 catch할 수 있는 추상적인 무언가로 만들어줍니다.
{% endinfo %}

`Result` 의 형태는 Swift 2 이후로 에러 핸들링으로 함축되었던 반면에, 이제는 공식적으로 Swift 5 표준 라이브러리에 주로 비동기 에러를 다루는 곳에서 쓰도록 추가됐습니다.
(공식적으로 발표되기 전에는 [몇몇 개발자들이 그들만의 버전](https://github.com/search?o=desc&q=result+language%3Aswift&s=&type=Repositories)을 개발했었습니다.)

앞에서 보았듯이 옵셔널 두 개 `(Value?, Error?)` 를 넘기는 것보다 낫다는 것은 의심할 여지가 없습니다.
하지만 아직 보여드리지 않은 경우인 둘 이상의 에러가 발생할 수 있는 호출에서 `Result` 를 사용해서 에러를 다루는 방법에 대해 보여드리겠습니다.

`flatMap` 은 동시에 변환을 반환하기 때문에 이것은 비동기 공간에서 문제가 생길 수 있습니다.
그러니 비동기 연산을 합성할 땐 사용할 수 없습니다.

```swift
userData(for: "jemmons") { userResult in
  switch userResult {
  case .success(let user):
    fetchAvatar(for: user) { avatarResult in

      switch avatarResult {
      case .success(let avatar):
        cloudSave(image: avatar) { saveResult in

          switch saveResult {
          case .success:
            // 모두 성공!

          case .failure(URLError.timedOut)
            print("Operation timed out.")
          <#...#>
        }
      }

      case .failure(AvatarError.invalidUserFormat):
        print("User not recognized.")
      <#...#>
    }
  }

  case .failure(URLError.notConnectedToInternet):
    print("No internet detected.")
  <#...#>
}
```

{% info %}
실제로 `flatMap` 과 같은 방식의 핸들링도 할 수 있는데, 이는 [Continuation Monad](https://en.wikipedia.org/wiki/Monad_%28functional_programming%29#Continuation_monad)라고 불립니다.
이는 지금 익히기엔 충분히 복잡해서 이것 하나만으로도 몇 개의 블로그 포스팅이 나올 것입니다.
{% endinfo %}

## Awaiting the Future

Swift가 동시 에러 처리에서 `Result` 중첩 문제를 정의하기 위해서 `do/catch` 문법을 사용한 것처럼, 비동기 에러 처리에 대해서도 비슷한 제안이 많습니다.

[async/await 제안](https://gist.github.com/lattner/429b9070918248274f25b714dcfc7619)은 그 중 하나입니다. 이 제안을 우리 코드에 적용해본다면 다음과 같이 만들 수 있습니다.

```swift
do {
  let user = try await userData(for: "jemmons")
  let avatar = try await fetchAvatar(for: user)
  try await cloudSave(image: avatar)

} catch AvatarError.invalidUserFormat {
  print("User not recognized.")

} catch URLError.timedOut {
  print("Operation timed out.")

} catch URLError.notConnectedToInternet {
    print("No internet detected.")
} <#...#>
```

여기까지 Swift 에러 핸들링의 어두운 과도기에 길을 밝혀준 `Result` 타입에 대해 알아보았습니다.
