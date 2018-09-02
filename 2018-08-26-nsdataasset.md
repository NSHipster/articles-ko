---
title: NSDataAsset
author: Mattt
translator: 김필권
category: Cocoa
excerpt: "네트워크 리퀘스트를 빠르게 할 수 있는 기술은 압축과 스트리밍, 캐싱과 미리불러오기, 커넥션 풀링과 멀티플렉싱, 연기(deferring)와 백그라운딩 등 여러가지가 있습니다.
그 중에서도 모든 것을 꿰뚫는 한 가지는 _첫 화면에서 리퀘스트를 만들지 마라_ 입니다."
status:
  swift: 4.2
---

웹에서 속도는 사치가 아니라 생존의 문제입니다.

최근 몇 년간의 사용자 연구에 의하면 페이지 로딩 시간(우리 눈이 인지 가능한 400 밀리초 이상)은 전환 및 참여율에 부정적인 영향을 가져올 수 있다고 합니다.
웹 페이지를 불러올 때 매 초 10%의 사용자가 다시 뒤로 돌아간다고 생각하면 됩니다.

구글, 아마존 그리고 넷플릭스 같은 인터넷 회사에게는 매 초가 연간 몇십억 달러의 손실을 의미하기 떄문에 그들이 웹을 조금이라도 빠르게 만들기 위해 엔지니어링에 노력을 기울이는 것은 놀랄 일이 아닙니다.

네트워크 리퀘스트를 빠르게 할 수 있는 기술은 압축과 스트리밍, 캐싱과 미리불러오기, 커넥션 풀링과 멀티플렉싱, 연기(deferring)와 백그라운딩 등 여러가지가 있습니다.
그 중에서도 모든 것을 꿰뚫는 한 가지는 _첫 화면에서 리퀘스트를 만들지 마라_ 입니다.

사용되기 이전에 어느정도 다운받는 앱의 경우엔 이런 측면에서 웹 페이지보다 더 나은 이점을 가진다고 볼 수 있습니다.
이 주의 NSHipster에서는 Asset Catalog를 이용해서 앱의 첫 실행 경험을 향상시킬 수 있는 방법에 대해 알아보겠습니다.

---

Asset Catalog는 현재 기기의 특성에 맞게 리소스를 모아줍니다. 한 이미지가 있다면 이를 기기(아이폰, 아이패드, 애플워치, 애플TV, 맥)와 화면 해상도(`@2x` / `@3x`) 또는 색상 영역(sRGB / P3)에 맞춰서 다른 파일을 제공할 수 있습니다.
그리고 메모리나 Metal의 버전 정보에 따라 다양성을 제공할 수도 있습니다. 사용자 입장에선 이름으로 에셋을 요청하면 가장 적절한 하나의 파일이 자동으로 제공된다고 생각하면 됩니다.

편한 API를 제공하는 것을 넘어서 Asset Catalog는 [app thinning](https://help.apple.com/xcode/mac/current/#/devbbdc5ce4f)을 제공해서 사용자의 기기에 최적화된 적은 용량의 앱을 얻을 수 있게 해줍니다.

이미지는 가장 흔한 에셋의 종류 중 하나입니다. 이제 iOS 9과 macOS El Capitan에서 JSON, XML 그리고 다른 데이터 파일이 [`NSDataAsset`](https://developer.apple.com/documentation/uikit/nsdataasset)를 통해 에셋에 참여할 수 있게 되었습니다.


## Asset Catalog로 데이터를 저장하고 검색하는 방법

예를 들어서 디지털 색상 팔레트 iOS 앱을 만든다고 가정해보겠습니다.

회색의 다양한 그림자를 구별하기 위해 우리는 색상의 목록과 그에 대응하는 이름을 불러와야 합니다. 보통 우리는 이를 처음 실행 시에 서버에서 다운받겠지만 그것은 [불리한 네트워크 조건](https://nshipster.com/network-link-conditioner/)에서는 앱의 기능을 차단하는 매우 나쁜 사용자 경험을 제공할 가능성이 있습니다.
우리가 필요로 하는 데이터는 유동적이지 않기 때문에 Asset Catalog를 통해 앱 번들에 포함하는건 어떨까요?

### 1단계. Asset Catalog에 새로운 데이터 셋을 추가합니다

Xcode에서 새로운 앱 프로젝트를 만들 때 Asset Catalog는 자동으로 생성됩니다.
Asset Catalog 편집기를 열기 위해 프로젝트 네비게이터에 있는 `Assets.xcassets` 를 선택합니다.
하단에 있는 <kbd>+</kbd> 아이콘을 누르고 "New Data Set"을 선택합니다.

{% asset add-new-data-set.png %}

이렇게 하면 `Assets.xcassets` 에 `.dataset` 익스텐션이 있는 새로운 서브 디렉토리가 생길 것입니다.

> 기본적으로 파인더는 이 번들들을 모두 디렉토리로 취급해서 컨텐츠를 필요에 맞게 검색하고 수정할 수 있게 해줍니다.

### 2단계. 데이터 파일을 추가합니다

파인더를 열고 데이터 파일을 Xcode의 데이터셋 에셋의 빈 필드에 드래그 앤 드롭합니다.

{% asset asset-catalog-any-any-universal.png %}

그 이후에 Xcode는 파일을 복사해서 `.dataset` 서브 디렉토리에 넣고 `contents.json` 메타데이터 파일을 파일 이름과 [Universal Type Identifier](https://en.wikipedia.org/wiki/Uniform_Type_Identifier)를 같이 업데이트합니다. 아래는 `contents.json` 파일입니다.

```json
{
  "info": {
    "version": 1,
    "author": "xcode"
  },
  "data": [
    {
      "idiom": "universal",
      "filename": "colors.json",
      "universal-type-identifier": "public.json"
    }
  ]
}
```

### 3단계. NSDataAsset을 통해 데이터 접근하기

이제 아래와 같은 코드로 파일의 데이터에 접근할 수 있습니다.

```swift
guard let asset = NSDataAsset(name: "NamedColors") else {
    fatalError("Missing data asset: NamedColors")
}

let data = asset.data
```

색상 앱에서는 이를 뷰 컨트롤러의 `viewDidLoad()` 메소드에서 호출할 것이고 테이블 뷰에 모델 객체의 배열을 디코딩하기 위한 결과 데이터를 사용할 것입니다.

```swift
let decoder = JSONDecoder()
self.colors = try! decoder.decode([NamedColor].self, from: asset.data)
```

## 합치기

일반적으로 데이터 셋은 Asset Catalog의 앱 용량 줄이기의 이점을 받지는 못합니다. (예를 들어 대부분의 JSON 파일들은 기기에서 지원하는 Metal 버전이 무엇인지에 상관이 없기 때문이죠.)

하지만 우리의 색상 팔레트 앱은 넓은 색상 영역을 보여주기 위해 기기별로 다른 색상 목록을 제공해야 합니다.

그러기 위해선 Asset Catalog 편집기의 사이드바에 있는 에셋을 고르고 Attributes Inspector의 Gamut이라고 적힌 드랍다운 컨트롤을 클릭하세요.

{% asset select-color-gamut.png %}

각 영역에 맞춤식 데이터 파일을 제공한 이후에 `contents.json` 메타데이터 파일은 다음과 같이 생겼을 것입니다.

```json
{
  "info": {
    "version": 1,
    "author": "xcode"
  },
  "data": [
    {
      "idiom": "universal",
      "filename": "colors-srgb.json",
      "universal-type-identifier": "public.json",
      "display-gamut": "sRGB"
    },
    {
      "idiom": "universal",
      "filename": "colors-p3.json",
      "universal-type-identifier": "public.json",
      "display-gamut": "display-P3"
    }
  ]
}
```

## 데이터 최신으로 유지하기

Asset Catalog에서 데이터를 저장하고 검색하는 것은 어렵지 않은 일입니다.
정말로 어렵고 궁극적으로 가장 중요한 것은 데이터를 항상 최신으로 유지하는 것입니다.

`curl`, `rsync`, `sftp`, Dropbox, BitTorrent 또는 Filecoin으로 데이터를 항상 최신으로 유지하는 방법.
쉘 스크립트에서 작업하기 (그리고 원한다면 Xcode 빌드 페이즈에서 호출)
여러분의 빌드 시스템이 필요로 하는 파일을 사용해서 업데이트 (`Makefile`, `Rakefile`, `Fastfile` 또는 무언가)
Slack이나 Siri Shortcut에 등록해서 캐쥬얼하게 업데이트 _"시리야 데이터 에셋이 상하기 전에 업데이트해줘"_

**그러나 여러분이 여러분의 데이터를 동기화하기로 했으니 릴리즈 과정의 일부분이고 자동화되었는지 확인하세요.**

다음은 쉘 스크립트에서 `curl`을 이용해서 최신 데이터 파일을 다운로드 받는 예제입니다.

```shell
#!/bin/sh
CURL='/usr/bin/curl'
URL='https://example.com/path/to/data.json'
OUTPUT='./Assets.xcassets/Colors.dataset/data.json'

$CURL -fsSL -o $OUTPUT $URL
```

## 감싸기

Assets Catalog가 이미지 에셋의 손실 없는 압축을 제공하고 있지만 문서, Xcode 도움말, WWDC 세션 그 어디를 봐도 데이터 에셋에 대한 최적화가 이루어졌다는 말은 없습니다. (적어도 아직은요)

데이터 에셋이 수백 킬로바이트보다 크다면 여러분은 압축을 고려할 필요가 있습니다.
왜냐면 JSON, CSV 그리고 XML과 같은 텍스트 파일은 보통 60% ~ 80% 까지 압축이 되기 때문입니다.

이전의 쉘 스크립트에 다음과 같이 `gzip`만 사용하면 압축을 간단하게 추가할 수 있습니다.

```shell
#!/bin/sh
CURL='/usr/bin/curl'
GZIP='/usr/bin/gzip'
URL='https://example.com/path/to/data.json'
OUTPUT='./Assets.xcassets/Colors.dataset/data.json.gz'

$CURL -fsSL $URL | $GZIP -c > $OUTPUT
```

압축을 하기로 했다면 `"universal-type-identifier"` 필드가 다음과 같이 설정돼있는지 확인하세요.

```json
{
  "info": {
    "version": 1,
    "author": "xcode"
  },
  "data": [
    {
      "idiom": "universal",
      "filename": "colors.json.gz",
      "universal-type-identifier": "org.gnu.gnu-zip-archive"
    }
  ]
}
```

클라이언트 측면에선 에셋 카탈로그를 사용하기 전에 압축 해제해줘야 합니다.
`Gzip` 모듈이 있다면 다음과 같이 사용할 수 있습니다.

```swift
do {
    let data = try Gzip.decompress(data: asset.data)
} catch {
    fatalError(error.localizedDescription)
}
```

비슷한 내용이 앱 내에 여러군데에 있다면 `NSDataAsset` 의 익스텐션을 만들어서 좀 더 편안하게 작업할 수 있을 것입니다.

```swift
extension NSDataAsset {
    func decompressedData() throws -> Data {
        return try Gzip.decompress(data: self.data)
    }
}
```

> 또한 [Git Large File Storage (LFS)](https://git-lfs.github.com)을 사용해서 큰 사이즈의 데이터 에셋을 관리할 수도 있습니다.

---

모든 사용자가 WiFi와 LTE를 통해 빠르고 모두 연결된(유비쿼터스) 네트워크 접근을 즐기고 있다고 가정하고 싶지만, 이는 모두에게 해당되는 것도 아니고 항상 그런것도 아닙니다.

시간을 내서 여러분의 앱이 실행될 때 어떤 네트워크 호출을 하는지 확인하고 미리 불러오는 것이 도움이 될 지 고려해보세요.
처음 실행할 때 걸리는 시간은 앱의 첫 인상을 결정하며 오래걸릴수록 장기 사용자가 줄어들고 바로 삭제하는 사용자가 늘어날 것입니다.
