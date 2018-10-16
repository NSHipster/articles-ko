---
title: macOS Dynamic Desktop
author: Mattt
translator: 김필권
category: ""
excerpt: >
  다크 모드는 macOS Mojave의 가장 유명한 기능 중 하나입니다. 특히 우리 개발자들에게는요.
  이 기능과 Night Shift를 통해 유추해본다면 Dynamic Desktop은 충분히 나올 수 있는 기능이었습니다.
status:
  swift: 4.2
---

다크 모드는 macOS Mojave의 가장 유명한 기능 중 하나입니다. 특히 우리 개발자들에게는요.

몇 년 전엔 어떤 팬이 Night Shift와 비슷하고 늦은 밤에 (또는 이른 아침) 개발하는 사람들의 눈의 피로를 줄여주는 기능을 만들기도 했습니다.

"시스템 환경설정 > 데스크탑 및 화면 보호기"에 가면 새로운 선택지인 "다이내믹"을 볼 수 있을 것입니다. 이 기능은 여러분의 위치와 시간에 따라 배경화면이 변하는 기능입니다.

{% asset desktop-and-screen-saver-preference-pane.png %}

결과는 감지하기 힘들지만 시간이 지나서 확인해보면 기분이 좋아집니다.
시간에 따라 배경화면을 보고있으면 자연과 조화돼서 마치 배경화면이 살아있는 것 같습니다.

_내부에선 정확히 어떤 일이 일어나는걸까요?_<br/>
그게 바로 이번주의 NSHipster의 주제입니다.

그에 대한 대답을 하기 위해 우리는 이미지 포맷을 깊숙히 파보며 약간의 리버스 엔지니어링과 구형 삼각법도 알아볼 것입니다.

---

Dynamic Desktop이 어떻게 작동하는지 이해하기 위한 가장 첫 번째는 다이내믹 이미지를 알아보는 것입니다.

macOS Mojave를 설치하셨다면 파인더를 열고 "이동 > 폴더로 이동"(<kbd>⇧</kbd><kbd>⌘</kbd><kbd>G</kbd>)을 선택하시고 "/Library/Desktop Pictures/"를 입력하세요.

{% asset go-to-library-desktop-pictures.png %}

이 폴더안에 있는 "Mojave.heic" 파일을 찾으시고 미리보기 앱으로 열어보세요.

{% asset mojave-heic.png %}

미리보기의 사이드 바를 보면 각자 사막의 다른 화면을 나타내고 있는 16개의 썸네일을 확인할 수 있을 것입니다.

{% asset mojave-dynamic-desktop-images.png %}

이제 "도구 > 속성 보기" (<kbd>⌘</kbd><kbd>I</kbd>) 를 선택하면 우리가 찾던 일반적인 정보를 볼 수 있습니다.

{% asset mojave-heic-preview-info.png %}

아쉽게도 이게 미리보기가 우리에게 제공할 수 있는 모든 정보입니다. (지금 이 글을 적고있는 지금까지는요.)
다음 패널인 "추가 정보"를 선택하면 우리의 주제에 대한 더 많은 정보를 얻을 수 있습니다.

|              |            |
| ------------ | ---------- |
| Color Model (색상 모델)  | RGB        |
| Depth (심도)  | 8          |
| Pixel Height (픽셀 높이) | 2,880      |
| Pixel Width (픽셀 너비) | 5,120      |
| Profile Name (프로파일 이름) | Display P3 |

{% info %}

`.heic` 파일 확장자는 <abbr title="High-Efficiency Image File Format">HEIF</abbr> 또는 <abbr title="High-Efficiency Video Compression">HEVC</abbr>를 사용해서 인코딩된 이미지들의 컨테이너를 의미합니다. 더 많은 정보는 [WWDC 2017 Session 503 "HEIF와 HEVC를 소개합니다"](https://developer.apple.com/videos/play/wwdc2017/503/)를 참고하세요.

{% endinfo %}

우리는 더 자세히 알고싶기 때문에 소매를 걷고 더 낮은 단계의 API로 들어가봅시다.

## CoreGraphics로 더 자세히 파헤치기

새로운 Xcode Playground를 생성해서 조사를 시작해보겠습니다.
간편함을 위해 시스템의 "Mojave.heic" 파일의 URL을 하드코딩하겠습니다.

```swift
import Foundation
import CoreGraphics

// macOS 10.14 Mojave가 필요합니다
let url = URL(fileURLWithPath: "/Library/Desktop Pictures/Mojave.heic")
```

다음은 `CGImageSource` 를 만들고 메타데이터를 복사해서 모든 태그들을 둘러보겠습니다.

```swift
let source = CGImageSourceCreateWithURL(url as CFURL, nil)!
let metadata = CGImageSourceCopyMetadataAtIndex(source, 0, nil)!
let tags = CGImageMetadataCopyTags(metadata) as! [CGImageMetadataTag]
for tag in tags {
    guard let name = CGImageMetadataTagCopyName(tag),
        let value = CGImageMetadataTagCopyValue(tag)
    else {
        continue
    }

    print(name, value)
}
```

위의 코드를 실행하면 두 가지 결과를 얻을 수 있습니다. `"True"` 값을 가지는 `hasXMP` 와 조금은 이해하기 힘든 값을 가지는 `solar` 입니다.

```
YnBsaXN0MDDRAQJSc2mvEBADDBAUGBwgJCgsMDQ4PEFF1AQFBgcICQoLUWlRelFh
UW8QACNAcO7vOubr3yO/1e+pmkOtXBAB1AQFBgcNDg8LEAEjQFRxqCKOFiAjwCR6
waUkDgHUBAUGBxESEwsQAiNAVZV4BI4c+CPAEP2uFrMcrdQEBQYHFRYXCxADI0BW
tALKmrjwIz/2ObLnx6l21AQFBgcZGhsLEAQjQFfTrJlEjnwjQByrLle1Q0rUBAUG
Bx0eHwsQBSNAWPrrmI0ISCNAKiwhpSRpc9QEBQYHISIjCxAGI0BgJff9KDpyI0BE
NTOsilht1AQFBgclJicLEAcjQGbHdYIVQKojQEq3fAg86lXUBAUGBykqKwsQCCNA
bTGmpC2YRiNAQ2WFOZGjntQEBQYHLS4vCxAJI0BwXfII2B+SI0AmLcjfuC7g1AQF
BgcxMjMLEAojQHCnF6YrsxcjQBS9AVBLTq3UBAUGBzU2NwsQCyNAcTcSnimmjCPA
GP5E0ASXJtQEBQYHOTo7CxAMI0BxgSADjxK2I8AoalieOTyE1AQFBgc9Pj9AEA0j
QHNWsnnMcWIjwEO+oq1pXr8QANQEBQYHQkNEQBAOI0ABZpkFpAcAI8BKYGg/VvMf
1AQFBgdGR0hAEA8jQErBKblRzPgjwEMGElBIUO0ACAALAA4AIQAqACwALgAwADIA
NAA9AEYASABRAFMAXABlAG4AcAB5AIIAiwCNAJYAnwCoAKoAswC8AMUAxwDQANkA
4gDkAO0A9gD/AQEBCgETARwBHgEnATABOQE7AUQBTQFWAVgBYQFqAXMBdQF+AYcB
kAGSAZsBpAGtAa8BuAHBAcMBzAHOAdcB4AHpAesB9AAAAAAAAAIBAAAAAAAAAEkA
AAAAAAAAAAAAAAAAAAH9
```

### Shining Light on Solar

대부분의 우리는 이 알 수 없는 글자의 벽을 보고선 조용히 맥북을 닫았을 것입니다. 하지만 몇 분은 아셨을 사실인데 이 텍스트는 [Base64-encoded](https://en.wikipedia.org/wiki/Base64)로 암호화된 것과 아주 비슷하게 생겼습니다.

우리의 가설을 실행에 옮길 시간입니다.

```swift
if name == "solar" {
    let data = Data(base64Encoded: value)!
    print(String(data: data, encoding: .ascii))
}
```

<samp>
bplist00Ò\u{01}\u{02}\u{03}...
</samp>

`bplist` 가 뭘까요?

놀랍게도 이건 [바이너리 프로퍼티 리스트](https://en.wikipedia.org/wiki/Property_list)의 [파일 서명](https://en.wikipedia.org/wiki/File_format#Magic_number)입니다.

이번엔 `PropertyListSerialization` 를 사용해보겠습니다...

```swift
if name == "solar" {
    let data = Data(base64Encoded: value)!
    let propertyList = try PropertyListSerialization
                            .propertyList(from: data,
                                          options: [],
                                          format: nil)
    print(propertyList)
}
```

```
(
    ap = {
        d = 15;
        l = 0;
    };
    si = (
        {
            a = "-0.3427528387535028";
            i = 0;
            z = "270.9334057827345";
        },
        ...
        {
            a = "-38.04743388682423";
            i = 15;
            z = "53.50908581251309";
        }
    )
)
```

_이제야 말이 통하네요!_

최상위 키는 두 가지입니다.

`ap` 키는 정수값을 가지는 `d` 와 `l` 키를 가지는 딕셔너리를 값으로 가지고 있습니다.

`si` 키는 정수와 실수 값을 가지는 딕셔너리의 배열을 값으로 가집니다.
중첩된 딕셔너리들의 키를 살펴보겠습니다. `i` 는 딱 보면 알 수 있듯이 0에서 15로 증가하는 인덱스 값입니다.
`a` 와 `z` 는 고도(altitude)와 방위각(azimuth)를 생각하시면 쉽습니다.

### 태양의 위치 계산하기

이 글을 쓰고 있는 시점엔 북반구에 있는 우리의 계절은 가을이며 날은 추워졌고 해가 짧아졌습니다. 남반구는 반대로 날이 따뜻해지고 해가 길어졌겠죠. 계절의 변화는 태양의 길이는 곧 우리가 어디에 있는지에 따라 다르다는 것을 생각하게 해줍니다.

좋은 소식은 천문학이 정확히 왜 그런지 알려줄 수 있다는 것입니다. 나쁜 소식은 그걸 설명하려면 아주 [복잡한](https://en.wikipedia.org/wiki/Position_of_the_Sun) 계산이 필요하다는 것입니다.

솔직히 말하자면 우리는 우리 자신을 이해시킬 필요가 없습니다. 우리는 그저 인터넷에서 찾은 내용을 포팅하면 됩니다.
몇번의 시도와 에러 끝에 저희는 [실제로 작동하는 코드](https://github.com/NSHipster/DynamicDesktop/blob/master/SolarPosition.playground)에 어느정도 도달한 것 같습니다. (PR은 환영입니다!)

```swift
import Foundation
import CoreLocation

// Apple Park, Cupertino, CA
let location = CLLocation(latitude: 37.3327, longitude: -122.0053)
let time = Date()

let position = solarPosition(for: location, at: time)
let formattedDate = DateFormatter.localizedString(from: time,
                                                    dateStyle: .medium,
                                                    timeStyle: .short)
print("Solar Position on \(formattedDate)")
print("\(position.azimuth)° Az / \(position.elevation)° El")
```

<samp>
2018년 10월 1일 12시의 태양 위치
180.73470025840783° Az / 49.27482549913847° El
</samp>

2018년 10월 1일 정오에 Apple Park의 태양빛은 남쪽에서 오고 수평선과 머리 바로 위의 사이에서 비치고 있었습니다.

하루 종일 태양의 위치를 추적한다면 우리는 Apple Watch의 "Solar" 페이스를 닮은 사인 그래프 모양을 얻게 될 것입니다.

{% asset solar-position-watch-faces.jpg %}

### XMP에 대한 이해 확장하기

좋습니다. 천문학은 충분한 것 같습니다.
이제 조금 지루한 내용으로 가보겠습니다. XML 메타데이터 표준입니다.

`hasXMP` 메타데이터 키를 기억하시나요? _그겁니다._

<abbr title="Extensible Metadata Platform">XMP</abbr> 또는 Extensible Metadata Platform은 메타데이터로 파일을 태깅하는 표준 형식입니다.
XMP는 어떻게 생겼을까요?
마음 단단히 먹으세요!

```swift
let xmpData = CGImageMetadataCreateXMPData(metadata, nil)
let xmp = String(data: xmpData as! Data, encoding: .utf8)!
print(xmp)
```

```xml
<x:xmpmeta xmlns:x="adobe:ns:meta/" x:xmptk="XMP Core 5.4.0">
   <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
      <rdf:Description rdf:about=""
            xmlns:apple_desktop="http://ns.apple.com/namespace/1.0/">
         <apple_desktop:solar>
            <!-- (Base64-Encoded Metadata) -->
        </apple_desktop:solar>
      </rdf:Description>
   </rdf:RDF>
</x:xmpmeta>
```

_웩_

우리는 `apple_desktop` 라는 이름이 우리가 만든 Dynamic Desktop 이미지에도 잘 작동하는지 확인할 필요가 있습니다.

말하자면 이제 시작이라는거죠.

## 자기만의 Dynamic Desktop 만들기

Dynamic Desktop을 표현하는 데이터 모델을 만들어보겠습니다.

```swift
struct DynamicDesktop {
    let images: [Image]

    struct Image {
        let cgImage: CGImage
        let metadata: Metadata

        struct Metadata: Codable {
            let index: Int
            let altitude: Double
            let azimuth: Double

            private enum CodingKeys: String, CodingKey {
                case index = "i"
                case altitude = "a"
                case azimuth = "z"
            }
        }
    }
}
```

각 Dynamic Desktop은 정렬된 이미지 시퀀스로 이루어져 있습니다. 이 이미지는 `CGImage` 객체가 저장된 이미지 데이터와 이전에 다뤘던 메타데이터로 이루어져 있습니다.
우리는 컴파일러가 자동으로 일을 마치게 하기 위해 `Metadata` 선언시에 `Codable` 을 사용합니다.
Base64로 인코딩된 바이너리 프로퍼티 리스트를 생성할 때 이를 활용할 것입니다.

### 이미지 경로에 작성하기

먼저 특정 URL로 `CGImageDestination` 를 생성합니다.
파일 타입은 `heic` 이고 원본 숫자는 포함돼있던 것과 동일합니다.

```swift
guard let imageDestination = CGImageDestinationCreateWithURL(
                                outputURL as CFURL,
                                AVFileType.heic as CFString,
                                dynamicDesktop.images.count,
                                nil
                             )
else {
    fatalError("Error creating image destination")
}
```

다음으로 dynamic desktop 객체의 각 이미지를 순환합니다.
`enumerated()` 메소드를 사용하면 우리는 반복문 속에서도 현재 `index` 를 알 수 있으니 첫 번째 이미지의 메타데이터 값을 설정해보겠습니다.

```swift
for (index, image) in dynamicDesktop.images.enumerated() {
    if index == 0 {
        let imageMetadata = CGImageMetadataCreateMutable()
        guard let tag = CGImageMetadataTagCreate(
                            "http://ns.apple.com/namespace/1.0/" as CFString,
                            "apple_desktop" as CFString,
                            "solar" as CFString,
                            .string,
                            try! dynamicDesktop.base64EncodedMetadata() as CFString
                        ),
            CGImageMetadataSetTagWithPath(
                imageMetadata, nil, "xmp:solar" as CFString, tag
            )
        else {
            fatalError("Error creating image metadata")
        }

        CGImageDestinationAddImageAndMetadata(imageDestination,
                                              image.cgImage,
                                              imageMetadata,
                                              nil)
    } else {
        CGImageDestinationAddImage(imageDestination,
                                   image.cgImage,
                                   nil)
    }
}
```

Core Graphics API의 정제되지 않은 특징 외에는 위의 코드는 꽤 쉽습니다.
추가적인 설명이 필요한 부분은 `CGImageMetadataTagCreate(_:_:_:_:_:)` 를 호출하는 부분뿐입니다.

이미지와 메타데이터가 이루어진 방식과 코드에서 표현되는 방식의 괴리때문에 우리는 `DynamicDesktop` 을 위한 `Encodable` 을 직접 만들어야 했습니다.

```swift
extension DynamicDesktop: Encodable {
    private enum CodingKeys: String, CodingKey {
        case ap, si
    }

    private enum NestedCodingKeys: String, CodingKey {
        case d, l
    }

    func encode(to encoder: Encoder) throws {
        var keyedContainer =
            encoder.container(keyedBy: CodingKeys.self)

        var nestedKeyedContainer =
            keyedContainer.nestedContainer(keyedBy: NestedCodingKeys.self,
                                           forKey: .ap)

        // 도와주세요: `l`과 `d`가 정확히 나타내는 값을 모르겠어요
        try nestedKeyedContainer.encode(0, forKey: .l)
        try nestedKeyedContainer.encode(self.images.count, forKey: .d)

        var unkeyedContainer =
            keyedContainer.nestedUnkeyedContainer(forKey: .si)
        for image in self.images {
            try unkeyedContainer.encode(image.metadata)
        }
    }
}
```

위의 Encodable이 완성되면 앞에서 언급했던 `base64EncodedMetadata()` 메소드를 다음과 같이 구현할 수 있을 것입니다.

```swift
extension DynamicDesktop {
    func base64EncodedMetadata() throws -> String {
        let encoder = PropertyListEncoder()
        encoder.outputFormat = .binary

        let binaryPropertyListData = try encoder.encode(self)
        return binaryPropertyListData.base64EncodedString()
    }
}
```

for-in 반복문이 끝나서 모든 이미지와 메타데이터가 작성되고나면 이제 `CGImageDestinationFinalize(_:)` 를 호출해서 이미지를 마무리하고 디스크에 저장하면 됩니다.

```swift
guard CGImageDestinationFinalize(imageDestination) else {
    fatalError("Error finalizing image")
}
```

모든 것이 예상했던 대로 작동한다면 이제 새로운 Dynamic Desktop의 주인이 된 것을 자랑스럽게 여기셔도 됩니다.
끝내주네요!

---

Mojave의 새로운 기능인 Dynamic Desktop을 사랑하며 윈도우 95에서 대유행을 했던 배경화면과 비슷한 것이 다시 나와서 정말 즐겁습니다.

여러분도 그렇게 생각하신다면 다음과 같은 아이디어를 시도해보세요.

### 사진에서 자동으로 Dynamic Desktop 생성하기

천체의 움직임처럼 엄청난 것이 시간과 장소라는 두 가지 입력의 방정식으로 축소될 수 있다는 것은 정말 충격적입니다.

이전의 예시처럼 이 정보는 하드 코딩돼있었지만 이미지에서 정보를 추출하는 것은 자동으로 할 수 있었습니다.

기본적으로 대부분의 핸드폰의 카메라는 사진을 찍을때마다 [Exif 메타데이터](https://en.wikipedia.org/wiki/Exif) 값도 같이 저장합니다.
이 메타데이터는 사진이 찍힌 시간과 기기의 위치 정보(GPS)를 포함할 수 있습니다.

이미지 메타데이터에서 시간과 위치 정보를 직접 읽으면 우리는 자동으로 태양의 위치를 계산할 수 있으며 사진 시리즈를 통해서 Dynamic Desktop를 만드는 과정을 간편하게 만들 수 있을 것입니다.

### iPhone의 타임 랩스 이용하기

iPhone XS를 좋은 곳에 사용하고 싶으신가요?
(더 정확히 말하자면, "오래된 iPhone을 팔기 전에 생산적인 활동에 사용해보고 싶으신가요?")

창문에 아이폰을 놓고, 충전기에 끼운 다음, 카메라를 타임 랩스 모드로 설정한 후에, "녹화" 버튼을 누르세요.
결과로 나온 비디오에서 키 프레임을 추출해내면 여러분만의 맞춤 제작 Dynamic Desktop을 만들 수 있을 것입니다.

[Skyflow](https://itunes.apple.com/us/app/skyflow-time-lapse-shooting/id937208291?mt=8)나 비슷한 앱을 사용하면 더 쉽게 지정된 간격으로 사진을 찍을 수 있습니다.

### GIS 데이터로 풍경 생성하기

아이폰과 하루라도 떨어져 있을 수 없거나 (슬프네요) 기억할만한 녹화거리가 없다면 (이것도 슬프네요), 현실을 창조해내면 됩니다. (가장 슬픈 얘기네요)

[Terragen](https://planetside.co.uk)같은 앱을 사용하면 현실같은 3D 풍경 사진을 만들 수 있습니다. 게다가 땅과 태양, 하늘까지 다 조정할 수 있습니다.

미국의 Geological Survey의 [National Map 웹사이트](https://viewer.nationalmap.gov/basic/)에서 다운로드 받아서 3D 렌더링 프로젝트의 템플릿에 사용하면 일이 더 쉬워질 것입니다.

### 이미 만들어진 Dynamic Desktop 다운로드받기

해야 할 일이 많고 이쁜 사진을 만드는 데에 쓸 시간이 없다면 항상 방법은 있습니다. 바로 다른 누군가에게 돈을 주고 사는 것이죠.

저희는 개인적으로 [24 Hour Wallpaper](https://www.jetsoncreative.com/24hourwallpaper/)의 팬입니다.

또 다른 추천할 것이 있다면 [트위터에서 태그해주세요](https://twitter.com/NSHipster/)!
